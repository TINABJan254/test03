---
title: Spring 事务详解
description: Spring事务管理详解，涵盖@Transactional注解、事务传播行为、隔离级别、事务失效场景及回滚规则。
category: 框架
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: Spring事务,@Transactional,事务传播,隔离级别,事务失效,回滚规则,声明式事务,AOP事务
---

Cách đây một thời gian đã hứa với độc giả bài phân tích tổng kết về **Spring Transaction** cuối cùng cũng đến rồi. Phần nội dung này tương đối quan trọng, bất kể là đối với công việc hay phỏng vấn, tuy nhiên tài liệu tham khảo tốt trên mạng tương đối ít.

## Transaction là gì?

**Transaction là một nhóm các thao tác về mặt logic, hoặc là đều thực thi, hoặc là đều không thực thi.**

Tin rằng mọi người đều thuộc lòng câu nói trên rồi, dưới đây tôi kết hợp với thực tế phát triển hàng ngày của chúng ta để nói một chút.

Mỗi method nghiệp vụ trong hệ thống của chúng ta có thể bao gồm nhiều thao tác cơ sở dữ liệu mang tính nguyên tử (atomic), ví dụ method `savePerson()` dưới đây có hai thao tác cơ sở dữ liệu mang tính nguyên tử. Những thao tác cơ sở dữ liệu nguyên tử này có quan hệ phụ thuộc lẫn nhau, chúng hoặc là đều thực thi, hoặc là đều không thực thi.

```java
  public void savePerson() {
    personDao.save(person);
    personDetailDao.save(personDetail);
  }
```

Ngoài ra, cần đặc biệt lưu ý rằng: **Transaction có hiệu lực hay không thì việc database engine có hỗ trợ transaction hay không là mấu chốt. Ví dụ cơ sở dữ liệu MySQL thường dùng mặc định sử dụng engine `innodb` hỗ trợ transaction. Tuy nhiên, nếu chuyển database engine sang `myisam`, thì chương trình cũng không còn hỗ trợ transaction nữa!**

Ví dụ kinh điển nhất và thường được mang ra nói về transaction chính là chuyển tiền. Giả sử Tiểu Minh muốn chuyển cho Tiểu Hồng 1000 VNĐ, việc chuyển tiền này sẽ liên quan đến hai thao tác mấu chốt là:

> 1. Giảm số dư của Tiểu Minh đi 1000 VNĐ.
> 2. Tăng số dư của Tiểu Hồng thêm 1000 VNĐ.

Lỡ như ở giữa hai thao tác này đột nhiên xuất hiện lỗi như hệ thống ngân hàng bị sập hoặc sự cố mạng, dẫn đến số dư của Tiểu Minh bị giảm mà số dư của Tiểu Hồng chưa được tăng, như vậy là không đúng. Transaction chính là đảm bảo hai thao tác mấu chốt này hoặc là đều thành công, hoặc là đều thất bại.

![Sơ đồ minh họa Transaction](https://oss.javaguide.cn/github/javaguide/mysql/%E4%BA%8B%E5%8A%A1%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

```java
public class OrdersService {
  private AccountDao accountDao;

  public void setOrdersDao(AccountDao accountDao) {
    this.accountDao = accountDao;
  }

  @Transactional(propagation = Propagation.REQUIRED,
                isolation = Isolation.DEFAULT, readOnly = false, timeout = -1)
  public void accountMoney() {
    // Tài khoản Tiểu Hồng cộng thêm 1000
    accountDao.addMoney(1000,xiaohong);
    // Mô phỏng exception đột nhiên xuất hiện, ví dụ trong ngân hàng có thể đột nhiên mất điện,...
    // Nếu không cấu hình quản lý transaction sẽ dẫn đến tài khoản Tiểu Hồng cộng 1000 mà tài khoản Tiểu Minh chưa trừ tiền
    int i = 10 / 0;
    // Tài khoản Tiểu Vương trừ đi 1000
    accountDao.reduceMoney(1000,xiaoming);
  }
}
```

Ngoài ra, 4 tính chất lớn ACID của cơ sở dữ liệu là nền tảng của transaction, dưới đây cùng tìm hiểu ngắn gọn.

## Bạn có hiểu các tính chất của Transaction (ACID) không?

1. **Atomicity** (Tính nguyên tử): Transaction là đơn vị thực thi nhỏ nhất, không cho phép chia nhỏ. Tính nguyên tử của transaction đảm bảo các hành động hoặc là hoàn thành toàn bộ, hoặc là hoàn toàn không có tác dụng;
2. **Consistency** (Tính nhất quán): Trước và sau khi thực thi transaction, dữ liệu giữ được tính nhất quán, ví dụ trong nghiệp vụ chuyển tiền, bất kể transaction có thành công hay không, tổng số tiền của người chuyển và người nhận phải không đổi;
3. **Isolation** (Tính cô lập): Khi truy cập đồng thời cơ sở dữ liệu, transaction của một user không bị can thiệp bởi transaction của user khác, cơ sở dữ liệu giữa các transaction đồng thời là độc lập;
4. **Durability** (Tính bền vững): Sau khi một transaction được commit, sự thay đổi dữ liệu trong cơ sở dữ liệu của nó là bền vững, ngay cả khi cơ sở dữ liệu gặp sự cố cũng không nên có bất kỳ ảnh hưởng nào tới nó.

🌈 Ở đây xin bổ sung thêm một điểm: **Chỉ khi đảm bảo tính bền vững, tính nguyên tử, tính cô lập của transaction, thì tính nhất quán mới được đảm bảo. Nghĩa là A, I, D là phương tiện, C là mục đích!** Chắc hẳn mọi người cũng giống như tôi, từng bị khái niệm ACID làm hiểu lầm một thời gian dài! Tôi cũng xem bài giảng công khai của thầy Châu Chí Minh [《Chuyên đề Kiến trúc Phần mềm của Châu Chí Minh》](https://time.geekbang.org/opencourse/intro/100064201) mới hiểu rõ (hãy đọc nhiều sách hay!!!).

![AID->C](https://oss.javaguide.cn/github/javaguide/mysql/AID->C.png)

Ngoài ra, DDIA tức tác giả cuốn [《Designing Data-Intensive Application (Thiết kế ứng dụng lấy dữ liệu làm trung tâm)》](https://book.douban.com/subject/30329536/) đã viết trong cuốn sách của mình như sau:

> Atomicity, isolation, and durability are properties of the database, whereas consistency (in the ACID sense) is a property of the application. The application may rely on the database’s atomicity and isolation properties in order to achieve consistency, but it’s not up to the database alone.
>
> Dịch ra có nghĩa là: Tính nguyên tử, tính cô lập và tính bền vững là các thuộc tính của cơ sở dữ liệu, còn tính nhất quán (theo nghĩa ACID) là thuộc tính của ứng dụng. Ứng dụng có thể dựa vào tính nguyên tử và tính cô lập của cơ sở dữ liệu để đạt được tính nhất quán, nhưng điều đó không chỉ phụ thuộc vào cơ sở dữ liệu. Do đó, chữ cái C không thuộc về ACID.

Rất tiến cử cuốn sách 《Designing Data-Intensive Application》 này, rất đáng đọc nhiều lần! Trên Douban có gần 90% người xem xong cuốn sách này đã đánh giá 5 sao. Ngoài ra, bản dịch tiếng Trung đã được mở nguồn trên GitHub, địa chỉ: [https://github.com/Vonng/ddia](https://github.com/Vonng/ddia) .

## Nói chi tiết về sự hỗ trợ Transaction của Spring

> ⚠️ Nhắc lại một lần nữa: Chương trình của bạn có hỗ trợ transaction hay không trước tiên phụ thuộc vào cơ sở dữ liệu, ví dụ nếu sử dụng MySQL, nếu bạn chọn engine innodb, thì xin chúc mừng bạn, có thể hỗ trợ transaction. Tuy nhiên, nếu cơ sở dữ liệu MySQL của bạn sử dụng engine myisam, thì xin lỗi, từ gốc đã không hỗ trợ transaction rồi.

Ở đây đề cập thêm một điểm kiến thức rất quan trọng: **MySQL đảm bảo tính nguyên tử như thế nào?**

Chúng ta biết nếu muốn đảm bảo tính nguyên tử của transaction, thì cần phải **rollback** các thao tác đã thực thi khi xảy ra exception, trong MySQL, cơ chế phục hồi được thực hiện thông qua **Undo Log (log rollback)**, tất cả các sửa đổi do transaction tiến hành đều sẽ được ghi vào undo log này trước, sau đó mới thực thi các thao tác liên quan. Nếu trong quá trình thực thi gặp exception, chúng ta trực tiếp tận dụng thông tin trong **Undo Log** để rollback dữ liệu về trạng thái trước khi sửa đổi là được! Hơn nữa, undo log sẽ được lưu bền vững xuống đĩa trước dữ liệu. Như vậy đảm bảo ngay cả khi gặp trường hợp cơ sở dữ liệu đột nhiên sập,... khi user khởi động lại cơ sở dữ liệu lần nữa, cơ sở dữ liệu vẫn có thể thông qua tra cứu undo log để rollback các transaction chưa hoàn thành trước đó.

### Spring hỗ trợ hai phương thức quản lý Transaction

#### Quản lý Transaction lập trình (Programmatic Transaction Management)

Quản lý transaction thủ công thông qua `TransactionTemplate` hoặc `TransactionManager`, trong ứng dụng thực tế rất ít khi sử dụng, nhưng có ích cho việc hiểu nguyên lý quản lý transaction của Spring.

Mã nguồn ví dụ sử dụng `TransactionTemplate` để quản lý transaction lập trình như sau:

```java
@Autowired
private TransactionTemplate transactionTemplate;
public void testTransaction() {

        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {

                try {

                    // .... Code nghiệp vụ
                } catch (Exception e){
                    // Rollback
                    transactionStatus.setRollbackOnly();
                }

            }
        });
}
```

Mã nguồn ví dụ sử dụng `TransactionManager` để quản lý transaction lập trình như sau:

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void testTransaction() {

  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
          try {
               // .... Code nghiệp vụ
              transactionManager.commit(status);
          } catch (Exception e) {
              transactionManager.rollback(status);
          }
}
```

#### Quản lý Transaction khai báo (Declarative Transaction Management)

Khuyên dùng (độ xâm nhập code là nhỏ nhất), thực tế được thực hiện thông qua AOP (cách dùng hoàn toàn bằng annotation dựa trên `@Transactional` được sử dụng nhiều nhất).

Mã nguồn ví dụ sử dụng annotation `@Transactional` để quản lý transaction như sau:

```java
@Transactional(propagation = Propagation.REQUIRED)
public void aMethod() {
  //do something
  B b = new B();
  C c = new C();
  b.bMethod();
  c.cMethod();
}
```

### Giới thiệu Interface quản lý Transaction trong Spring

Trong Spring Framework, 3 interface quan trọng nhất liên quan đến quản lý transaction như sau:

- **`PlatformTransactionManager`**: Trình quản lý transaction (nền tảng), cốt lõi của chiến lược Spring transaction.
- **`TransactionDefinition`**: Thông tin định nghĩa transaction (mức cô lập transaction, hành vi lan truyền, timeout, read-only,...).
- **`TransactionStatus`**: Trạng thái chạy của transaction.

Chúng ta có thể xem interface **`PlatformTransactionManager`** như là người quản lý transaction ở cấp trên, còn hai interface **`TransactionDefinition`** và **`TransactionStatus`** có thể xem là sự mô tả về transaction.

**`PlatformTransactionManager`** sẽ dựa theo định nghĩa của **`TransactionDefinition`** như thời gian timeout của transaction, mức cô lập, hành vi lan truyền,... để tiến hành quản lý transaction, còn interface **`TransactionStatus`** thì cung cấp một số method để lấy trạng thái tương ứng của transaction như có phải transaction mới hay không, có thể rollback hay không,...

#### PlatformTransactionManager: Interface quản lý Transaction

**Spring không quản lý transaction trực tiếp, mà cung cấp nhiều loại trình quản lý transaction**. Interface của trình quản lý transaction trong Spring là: **`PlatformTransactionManager`**.

Thông qua interface này, Spring cung cấp trình quản lý transaction tương ứng cho các nền tảng như: JDBC (`DataSourceTransactionManager`), Hibernate (`HibernateTransactionManager`), JPA (`JpaTransactionManager`),... nhưng việc thực thi cụ thể là công việc của bản thân từng nền tảng.

**Các triển khai cụ thể của interface `PlatformTransactionManager` như hình dưới đây:**

![](./images/spring-transaction/PlatformTransactionManager.png)

Trong interface `PlatformTransactionManager` định nghĩa 3 method:

```java
package org.springframework.transaction;

import org.springframework.lang.Nullable;

public interface PlatformTransactionManager {
    // Lấy transaction
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
    // Commit transaction
    void commit(TransactionStatus var1) throws TransactionException;
    // Rollback transaction
    void rollback(TransactionStatus var1) throws TransactionException;
}

```

**Nói thêm ở đây một chút. Tại sao lại cần định nghĩa hay nói cách khác là trừu tượng hóa ra interface `PlatformTransactionManager` này?**

Chủ yếu là vì muốn trừu tượng hóa hành vi quản lý transaction ra, sau đó các nền tảng khác nhau đi implement nó, như vậy chúng ta có thể đảm bảo hành vi cung cấp cho bên ngoài không đổi, tiện cho chúng ta mở rộng.

Tôi từng chia sẻ trong [Tinh cầu Kiến thức](https://javaguide.cn/about-the-author/zhishixingqiu-two-years.html) của tôi: **"Tại sao chúng ta phải dùng interface?"** .

> Cuốn sách 《Design Patterns》 (bản GOF) từ nhiều năm trước đã đề cập rằng hãy lập trình dựa trên interface chứ không dựa trên triển khai, bạn thực sự biết tại sao phải lập trình dựa trên interface chưa?
>
> Nhìn vào mã nguồn của các framework và project mã nguồn mở, interface là thành phần cấu thành quan trọng không thể thiếu của chúng. Để hiểu tại sao dùng interface, trước tiên phải hiểu rõ interface cung cấp tính năng gì. Chúng ta có thể hiểu interface là một giao ước cung cấp một loạt danh sách tính năng, bản thân interface không cung cấp tính năng, nó chỉ định nghĩa hành vi. Nhưng ai muốn dùng thì phải implement tôi trước, tuân thủ giao ước của tôi, rồi sau đó tự mình đi thực hiện tính năng tôi định nghĩa cần thực hiện.
>
> Ví dụ, dự án trước của tôi có nhu cầu gửi tin nhắn SMS, vì vậy chúng tôi định nghĩa một interface, interface chỉ có 2 method:
>
> 1. Gửi SMS 2. Method xử lý kết quả gửi.
>
> Ban đầu chúng tôi dùng dịch vụ SMS Aliyun, sau đó chúng tôi implement interface này hoàn thành một dịch vụ SMS Aliyun. Sau đó, chúng tôi đột nhiên lại chuyển sang nền tảng dịch vụ SMS khác, lúc này chúng tôi chỉ cần implement lại interface này là được. Như vậy đảm bảo hành vi chúng tôi cung cấp cho bên ngoài không đổi. Hầu như không cần thay đổi code gì, chúng tôi đã dễ dàng hoàn thành sự thay đổi nhu cầu, nâng cao tính linh hoạt và tính mở rộng của code.
>
> Khi nào dùng interface? Khi module tính năng bạn muốn thực hiện thiết kế các hành vi trừu tượng, ví dụ dịch vụ gửi SMS, dịch vụ lưu trữ kho ảnh,...

#### TransactionDefinition: Thuộc tính Transaction

Interface trình quản lý transaction **`PlatformTransactionManager`** thông qua method **`getTransaction(TransactionDefinition definition)`** để có được một transaction, parameter trong method này là class **`TransactionDefinition`**, class này định nghĩa một số thuộc tính transaction cơ bản.

**Thuộc tính transaction là gì?** Thuộc tính transaction có thể hiểu là một số cấu hình cơ bản của transaction, mô tả chiến lược transaction được áp dụng vào method như thế nào.

`TransactionDefinition` chủ yếu bao gồm 4 phương diện cấu hình transaction sau:

- Mức cô lập (Isolation level)
- Hành vi lan truyền (Propagation behavior)
- Có phải read-only hay không
- Timeout của transaction

Ngoài ra, method `getName()` có thể trả về tên transaction. Quy tắc rollback không thuộc về bản thân `TransactionDefinition`; `TransactionAttribute` được Spring dùng trong transaction khai báo thừa kế từ `TransactionDefinition`, và bổ sung thêm các năng lực như quy tắc rollback lên trên đó.

```java
package org.springframework.transaction;

import org.springframework.lang.Nullable;

public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    int TIMEOUT_DEFAULT = -1;
    // Trả về hành vi lan truyền của transaction, giá trị mặc định là REQUIRED.
    int getPropagationBehavior();
    // Trả về mức cô lập của transaction, giá trị mặc định là DEFAULT
    int getIsolationLevel();
    // Trả về thời gian timeout của transaction, giá trị mặc định là -1. Nếu vượt quá thời gian này mà transaction chưa hoàn thành, sẽ tự động rollback.
    int getTimeout();
    // Trả về có phải transaction read-only hay không, giá trị mặc định là false
    boolean isReadOnly();

    @Nullable
    String getName();
}
```

#### TransactionStatus: Trạng thái Transaction

Interface `TransactionStatus` dùng để ghi lại trạng thái của transaction, interface này định nghĩa một nhóm method, dùng để lấy hoặc phán đoán thông tin trạng thái tương ứng của transaction.

Method `PlatformTransactionManager.getTransaction(…)` trả về một object `TransactionStatus`.

**Nội dung interface TransactionStatus như sau:**

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // Có phải transaction mới hay không
    boolean hasSavepoint(); // Có điểm phục hồi (savepoint) hay không
    void setRollbackOnly(); // Thiết lập thành chỉ rollback
    boolean isRollbackOnly(); // Có phải chỉ rollback hay không
    boolean isCompleted(); // Đã hoàn thành hay chưa
}
```

### Giải thích chi tiết Thuộc tính Transaction

Trong phát triển nghiệp vụ thực tế, mọi người nhìn chung đều sử dụng annotation `@Transactional` để bật transaction, nhiều người không rõ các parameter trong annotation này có ý nghĩa gì, có tác dụng gì. Để sử dụng quản lý transaction tốt hơn trong dự án, rất khuyến khích đọc kỹ nội dung dưới đây.

#### Hành vi lan truyền Transaction (Propagation)

**Hành vi lan truyền transaction là để giải quyết vấn đề transaction giữa các method thuộc tầng nghiệp vụ gọi lẫn nhau**.

Khi một method transaction được gọi bởi một method transaction khác, phải chỉ định transaction nên lan truyền như thế nào. Ví dụ: Method có thể tiếp tục chạy trong transaction hiện có, hoặc cũng có thể mở một transaction mới, và chạy trong transaction của chính nó.

Lấy một ví dụ: Chúng ta trong method `aMethod()` của class A gọi method `bMethod()` của class B. Lúc này liên quan đến vấn đề transaction giữa các method tầng nghiệp vụ gọi lẫn nhau. Nếu `bMethod()` của chúng ta nếu xảy ra exception cần rollback, cấu hình hành vi lan truyền transaction như thế nào mới khiến cho `aMethod()` cũng rollback theo? Lúc này cần đến kiến thức về hành vi lan truyền transaction, nếu bạn chưa biết thì nhất định phải xem kỹ.

Code hành vi lan truyền dưới đây đều là các đoạn mã minh họa bỏ qua import và một phần chi tiết triển khai, trong đó `Propagation.xxx` là giữ chỗ cần thay thế, chứ không phải class hoàn chỉnh có thể biên dịch trực tiếp.

```java
@Service
class A {
    @Autowired
    B b;
    @Transactional(propagation = Propagation.xxx)
    public void aMethod() {
        //do something
        b.bMethod();
    }
}

@Service
class B {
    @Transactional(propagation = Propagation.xxx)
    public void bMethod() {
       //do something
    }
}
```

Trong định nghĩa `TransactionDefinition` bao gồm các hằng số thể hiện hành vi lan truyền như sau:

```java
public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    ......
}
```

Tuy nhiên, để tiện sử dụng, Spring định nghĩa tương ứng một enum class: `Propagation`

```java
package org.springframework.transaction.annotation;

import org.springframework.transaction.TransactionDefinition;

public enum Propagation {

    REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),

    SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),

    MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),

    REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),

    NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),

    NEVER(TransactionDefinition.PROPAGATION_NEVER),

    NESTED(TransactionDefinition.PROPAGATION_NESTED);

    private final int value;

    Propagation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }

}

```

**Các giá trị hợp lệ của hành vi lan truyền transaction đúng như sau**:

**1. `TransactionDefinition.PROPAGATION_REQUIRED`**

Hành vi lan truyền transaction được sử dụng nhiều nhất, annotation `@Transactional` mà chúng ta thường dùng mặc định sử dụng hành vi lan truyền transaction này. Nếu hiện tại đã có transaction, thì tham gia vào transaction đó; nếu hiện tại không có transaction, thì tạo một transaction mới. Nghĩa là:

- Nếu method bên ngoài không bật transaction, method bên trong được trang bị `Propagation.REQUIRED` sẽ mở transaction riêng của mình, và các transaction được mở độc lập với nhau, không can thiệp lẫn nhau.
- Nếu method bên ngoài bật transaction và là `Propagation.REQUIRED`, tất cả các method bên trong và method bên ngoài được trang bị `Propagation.REQUIRED` đều thuộc về cùng một transaction, chỉ cần một method rollback, toàn bộ transaction đều rollback.

Lấy một ví dụ: Nếu `aMethod()` và `bMethod()` ở trên của chúng ta đều sử dụng hành vi lan truyền `PROPAGATION_REQUIRED`, cả hai sử dụng cùng một transaction, chỉ cần một trong số các method rollback, toàn bộ transaction đều rollback.

```java
@Service
class A {
    @Autowired
    B b;
    @Transactional(propagation = Propagation.REQUIRED)
    public void aMethod() {
        //do something
        b.bMethod();
    }
}
@Service
class B {
    @Transactional(propagation = Propagation.REQUIRED)
    public void bMethod() {
       //do something
    }
}
```

**2. `TransactionDefinition.PROPAGATION_REQUIRES_NEW`**

Tạo một transaction mới, nếu hiện tại đã có transaction, thì hoãn transaction hiện tại. Nghĩa là bất kể method bên ngoài có mở transaction hay không, method bên trong được trang bị `Propagation.REQUIRES_NEW` sẽ mở transaction riêng của mình, và các transaction được mở độc lập với nhau, không can thiệp lẫn nhau.

Lấy một ví dụ: Nếu `bMethod()` ở trên của chúng ta sử dụng hành vi lan truyền transaction `PROPAGATION_REQUIRES_NEW`, `aMethod` vẫn dùng `PROPAGATION_REQUIRED`. Nếu `aMethod()` xảy ra exception rollback, `bMethod()` không bị rollback theo, vì `bMethod()` đã mở một transaction độc lập. Tuy nhiên, nếu `bMethod()` ném ra exception chưa được bắt và exception này thỏa mãn quy tắc rollback transaction, `aMethod()` cũng sẽ rollback, vì exception này bị cơ chế quản lý transaction của `aMethod()` phát hiện.

```java
@Service
class A {
    @Autowired
    B b;
    @Transactional(propagation = Propagation.REQUIRED)
    public void aMethod() {
        //do something
        b.bMethod();
    }
}

@Service
class B {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void bMethod() {
       //do something
    }
}
```

**3. `TransactionDefinition.PROPAGATION_NESTED`**:

Nếu hiện tại đã có transaction, thì tạo một transaction làm transaction lồng nhau của transaction hiện tại để thực thi; nếu hiện tại không có transaction, thì thực thi thao tác tương tự `TransactionDefinition.PROPAGATION_REQUIRED`. Nghĩa là:

- Trong trường hợp method bên ngoài bật transaction, ở bên trong mở một transaction mới làm transaction lồng nhau tồn tại.
- Nếu method bên ngoài không có transaction, thì mở riêng một transaction, tương tự `PROPAGATION_REQUIRED`.

Transaction lồng nhau đại diện bởi `TransactionDefinition.PROPAGATION_NESTED` thể hiện dưới dạng quan hệ cha con, lý niệm cốt lõi của nó là transaction con không tự commit độc lập, phụ thuộc vào transaction cha, chạy trong transaction cha; khi transaction cha commit, transaction con cũng commit theo, và dĩ nhiên, khi transaction cha rollback, transaction con cũng rollback theo;

> Khác với `TransactionDefinition.PROPAGATION_REQUIRES_NEW` ở chỗ: `PROPAGATION_REQUIRES_NEW` là transaction độc lập, không phụ thuộc vào transaction bên ngoài, thể hiện dưới quan hệ ngang hàng, thực thi xong lập tức commit, không liên quan đến transaction bên ngoài;

Transaction con cũng có đặc tính riêng của mình, có thể rollback độc lập, không gây ra rollback cho transaction cha, nhưng với điều kiện cần xử lý exception của transaction con, tránh exception bị transaction cha nhận biết dẫn đến rollback transaction bên ngoài;

Lấy một ví dụ:

- Nếu `aMethod()` rollback, `bMethod()` với tư cách là transaction lồng nhau sẽ rollback theo.
- Nếu `bMethod()` rollback, `aMethod()` có rollback hay không phải xem exception của `bMethod()` có được xử lý hay không:

  - Exception của `bMethod()` không được xử lý, tức bên trong `bMethod()` không xử lý exception, và `aMethod()` cũng không xử lý exception, vậy `aMethod()` sẽ nhận biết exception dẫn đến tổng thể rollback.

    ```java
    @Service
    class A {
        @Autowired
        B b;
        @Transactional(propagation = Propagation.REQUIRED)
        public void aMethod (){
            //do something
            b.bMethod();
        }
    }

    @Service
    class B {
        @Transactional(propagation = Propagation.NESTED)
        public void bMethod (){
           //do something and throw an exception
        }
    }
    ```

  - `bMethod()` xử lý exception hoặc `aMethod()` xử lý exception, `aMethod()` sẽ không rollback.

    ```java
    @Service
    class A {
        @Autowired
        B b;
        @Transactional(propagation = Propagation.REQUIRED)
        public void aMethod (){
            //do something
            try {
                b.bMethod();
            } catch (Exception e) {
                System.out.println("Phương thức rollback");
            }
        }
    }

    @Service
    class B {
        @Transactional(propagation = Propagation.NESTED)
        public void bMethod() {
           //do something and throw an exception
        }
    }
    ```

**4. `TransactionDefinition.PROPAGATION_MANDATORY`**

Nếu hiện tại đã có transaction, thì tham gia vào transaction đó; nếu hiện tại không có transaction, thì ném ra exception. (mandatory: bắt buộc)

Cái này rất ít khi dùng, không lấy ví dụ nữa.

**3 loại hành vi lan truyền dưới đây có phương thức xử lý khác nhau đối với transaction hiện có, không thể hiểu theo cùng một quy tắc rollback.**

- **`TransactionDefinition.PROPAGATION_SUPPORTS`**: Nếu hiện tại đã có transaction, thì tham gia vào transaction đó, các thao tác trong đó sẽ tham gia vào việc commit hoặc rollback của transaction đó; nếu hiện tại không có transaction, thì chạy theo cách không có transaction.
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**: Luôn chạy theo cách không có transaction; nếu hiện tại đã có transaction, trước tiên hoãn nó lại. Do đó, các thao tác thực thi trong ranh giới lan truyền này không chịu sự kiểm soát rollback của transaction ngoài bị hoãn.
- **`TransactionDefinition.PROPAGATION_NEVER`**: Chỉ cho phép chạy theo cách không có transaction; nếu phát hiện hiện tại đã có transaction, trực tiếp ném ra exception.

Xem thêm nội dung về hành vi lan truyền transaction tại bài viết này: [《太难了~面试官让我结合案例讲讲自己对 Spring 事务传播行为的理解。》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486668&idx=2&sn=0381e8c836442f46bdc5367170234abb&chksm=cea24307f9d5ca11c96943b3ccfa1fc70dc97dd87d9c540388581f8fe6d805ff548dff5f6b5b&token=1776990505&lang=zh_CN#rd)

#### Mức độ cô lập Transaction (Isolation Level)

Trong interface `TransactionDefinition` định nghĩa 5 hằng số thể hiện mức độ cô lập:

```java
public interface TransactionDefinition {
    ......
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    ......
}
```

Giống như phần hành vi lan truyền transaction, để tiện sử dụng, Spring cũng định nghĩa tương ứng một enum class: `Isolation`

```java
public enum Isolation {

  DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),

  READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),

  READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),

  REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),

  SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);

  private final int value;

  Isolation(int value) {
    this.value = value;
  }

  public int value() {
    return this.value;
  }

}
```

Dưới đây tôi sẽ lần lượt giới thiệu từng mức độ cô lập transaction:

- **`TransactionDefinition.ISOLATION_DEFAULT`**: Sử dụng mức độ cô lập mặc định của cơ sở dữ liệu phía backend, MySQL mặc định áp dụng mức cô lập `REPEATABLE_READ`, Oracle mặc định áp dụng mức cô lập `READ_COMMITTED`.
- **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`**: Mức cô lập thấp nhất, rất ít khi sử dụng mức cô lập này, vì nó cho phép đọc thay đổi dữ liệu chưa được commit, **có thể dẫn đến đọc bẩn (dirty read), đọc ảo (phantom read) hoặc đọc không lặp lại (non-repeatable read)**.
- **`TransactionDefinition.ISOLATION_READ_COMMITTED`**: Cho phép đọc dữ liệu đã commit của các transaction đồng thời (concurrent), **có thể ngăn chặn đọc bẩn, nhưng đọc ảo hoặc đọc không lặp lại vẫn có thể xảy ra**.
- **`TransactionDefinition.ISOLATION_REPEATABLE_READ`**: Kết quả của nhiều lần đọc trên cùng một field đều nhất quán, trừ khi dữ liệu bị sửa đổi bởi chính transaction đó, **có thể ngăn chặn đọc bẩn và đọc không lặp lại, nhưng đọc ảo vẫn có thể xảy ra.**
- **`TransactionDefinition.ISOLATION_SERIALIZABLE`**: Mức cô lập cao nhất, tuân thủ hoàn toàn mức cô lập ACID. Tất cả các transaction được thực thi tuần tự từng cái một, như vậy giữa các transaction hoàn toàn không thể phát sinh can thiệp, nghĩa là **mức này có thể ngăn chặn đọc bẩn, đọc không lặp lại cũng như đọc ảo**. Tuy nhiên điều này sẽ ảnh hưởng nghiêm trọng đến hiệu năng của chương trình. Thông thường cũng không dùng đến mức này.

Bài viết liên quan: [Giải thích chi tiết Mức cô lập Transaction trong MySQL](https://javaguide.cn/database/mysql/transaction-isolation-level.html).

#### Thuộc tính Timeout của Transaction

Cái gọi là timeout của transaction chính là thời gian tối đa cho phép thực thi của một transaction, nếu vượt quá thời gian này mà transaction chưa hoàn thành, sẽ tự động rollback transaction. Trong `TransactionDefinition` dùng giá trị int để thể hiện thời gian timeout, đơn vị là giây, giá trị mặc định là -1, thể hiện thời gian timeout phụ thuộc vào hệ thống transaction bên dưới hoặc không có thời gian timeout.

#### Thuộc tính Read-Only của Transaction

```java
package org.springframework.transaction;

import org.springframework.lang.Nullable;

public interface TransactionDefinition {
    ......
    // Trả về có phải transaction read-only hay không, giá trị mặc định là false
    boolean isReadOnly();

}
```

Đối với transaction chỉ có các truy vấn đọc dữ liệu, có thể chỉ định loại transaction là readonly, tức transaction chỉ đọc. Transaction chỉ đọc không liên quan đến sửa đổi dữ liệu, cơ sở dữ liệu sẽ cung cấp một số phương tiện tối ưu hóa, thích hợp sử dụng trong các method có nhiều thao tác truy vấn cơ sở dữ liệu.

Rất nhiều người sẽ thắc mắc, tại sao một thao tác truy vấn dữ liệu của tôi lại phải bật sự hỗ trợ của transaction?

Lấy innodb của MySQL làm ví dụ, theo mô tả trên trang chủ chính thức [https://dev.mysql.com/doc/refman/5.7/en/innodb-autocommit-commit-rollback.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-autocommit-commit-rollback.html):

> MySQL mặc định đối với mỗi kết nối mới thành lập đều bật mode `autocommit`. Dưới mode này, mỗi một câu lệnh `sql` gửi tới MySQL server đều được xử lý trong một transaction riêng biệt, sau khi thực thi xong sẽ tự động commit transaction, và mở một transaction mới.

Tuy nhiên, nếu bạn thêm annotation `@Transactional` cho method, tất cả SQL do method này thực thi sẽ được đặt trong một transaction. Sau khi khai báo transaction chỉ đọc, Spring sẽ truyền gợi ý chỉ đọc cho hệ thống transaction bên dưới; có tối ưu hóa hay không và tối ưu như thế nào phụ thuộc vào cơ sở dữ liệu, driver và trình quản lý transaction, nó cũng không đảm bảo thao tác ghi chắc chắn thất bại.

Nếu không thêm `Transactional`, mỗi câu `sql` sẽ mở một transaction riêng biệt, ở giữa bị transaction khác sửa dữ liệu, đều sẽ đọc được giá trị mới nhất theo thời gian thực.

Xin chia sẻ giải đáp của người khác về thuộc tính read-only của transaction:

- Nếu bạn thực thi một câu lệnh truy vấn đơn lẻ trong một lần, thì không cần thiết bật hỗ trợ transaction, cơ sở dữ liệu mặc định hỗ trợ tính nhất quán khi đọc trong thời gian thực thi SQL;
- Nếu bạn thực thi nhiều câu lệnh truy vấn trong một lần, ví dụ truy vấn thống kê, truy vấn báo cáo, trong kịch bản này, nhiều SQL truy vấn bắt buộc phải đảm bảo tính nhất quán khi đọc của tổng thể, nếu không, sau khi câu SQL trước truy vấn, trước khi câu SQL sau truy vấn, dữ liệu bị user khác thay đổi, thì truy vấn thống kê tổng thể lần đó sẽ xuất hiện trạng thái đọc dữ liệu không nhất quán, lúc này nên bật hỗ trợ transaction.

#### Quy tắc Rollback của Transaction

Những quy tắc này định nghĩa những exception nào sẽ dẫn đến rollback transaction còn những exception nào không. Mặc định, transaction chỉ khi gặp exception runtime (class con của `RuntimeException`) mới rollback, `Error` cũng dẫn đến rollback transaction, tuy nhiên khi gặp Checked Exception sẽ không rollback.

![](./images/spring-transaction/roollbackFor.png)

Nếu bạn muốn rollback loại exception cụ thể mà bạn định nghĩa, có thể làm như sau:

```java
@Transactional(rollbackFor= MyException.class)
```

### Giải thích chi tiết cách dùng annotation @Transactional

#### Phạm vi tác dụng của `@Transactional`

1. **Method**: Khuyên dùng annotation trên method. Proxy class trong Spring 6 mặc định còn hỗ trợ các method `protected` và package-private; proxy interface yêu cầu method là method `public` định nghĩa trong interface. Pattern proxy trong các phiên bản sớm hơn thường chỉ hỗ trợ method `public`.
2. **Class**: Nếu annotation này dùng trên class, thể hiện các method trong class đó thỏa mãn quy tắc hiển thị của proxy ở trên đều áp dụng ngữ nghĩa transaction giống nhau.
3. **Interface**: Không khuyên dùng trên interface.

#### Các parameter cấu hình thường gặp của `@Transactional`

Mã nguồn annotation `@Transactional` như sau, trong đó chứa cấu hình các thuộc tính transaction cơ bản:

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

  @AliasFor("transactionManager")
  String value() default "";

  @AliasFor("value")
  String transactionManager() default "";

  Propagation propagation() default Propagation.REQUIRED;

  Isolation isolation() default Isolation.DEFAULT;

  int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

  boolean readOnly() default false;

  Class<? extends Throwable>[] rollbackFor() default {};

  String[] rollbackForClassName() default {};

  Class<? extends Throwable>[] noRollbackFor() default {};

  String[] noRollbackForClassName() default {};

}
```

**Tóm tắt các parameter cấu hình thường gặp của `@Transactional` (chỉ liệt kê 5 cái tôi thường dùng):**

| Tên thuộc tính | Giải thích |
| :--- | :--- |
| propagation | Hành vi lan truyền của transaction, giá trị mặc định là REQUIRED, các giá trị có thể chọn đã giới thiệu ở trên |
| isolation | Mức cô lập của transaction, giá trị mặc định áp dụng DEFAULT, các giá trị có thể chọn đã giới thiệu ở trên |
| timeout | Thời gian timeout của transaction, giá trị mặc định là -1 (không timeout). Nếu vượt quá thời gian này mà transaction chưa hoàn thành, sẽ tự động rollback. |
| readOnly | Chỉ định transaction có phải read-only hay không, giá trị mặc định là false. |
| rollbackFor | Dùng để chỉ định các loại exception có thể kích hoạt rollback transaction, và có thể chỉ định nhiều loại exception. |

#### Nguyên lý annotation Transaction `@Transactional`

Một câu hỏi có thể bị hỏi khi phỏng vấn về AOP. Nói ngắn gọn nhé!

Chúng ta biết, **cơ chế hoạt động của `@Transactional` là dựa trên AOP, AOP lại sử dụng Dynamic Proxy để thực hiện. Nếu target object implement interface, mặc định sẽ áp dụng Dynamic Proxy của JDK, nếu target object không implement interface, sẽ sử dụng CGLIB Dynamic Proxy.**

🤐 Nói thêm một chút: Method `createAopProxy()` quyết định xem dùng JDK hay Cglib để làm Dynamic Proxy, mã nguồn như sau:

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

  @Override
  public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
      Class<?> targetClass = config.getTargetClass();
      if (targetClass == null) {
        throw new AopConfigException("TargetSource cannot determine target class: " +
            "Either an interface or a target is required for proxy creation.");
      }
      if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
        return new JdkDynamicAopProxy(config);
      }
      return new ObjenesisCglibAopProxy(config);
    }
    else {
      return new JdkDynamicAopProxy(config);
    }
  }
  .......
}
```

Nếu một class hoặc một method public trong class được đánh dấu annotation `@Transactional`, Spring container khi khởi động sẽ tạo cho nó một class proxy, khi gọi method public có gắn annotation `@Transactional`, thực tế gọi là method `invoke()` trong class `TransactionInterceptor`. Tác dụng của method này chính là mở transaction trước method mục tiêu, trong quá trình thực thi method nếu gặp exception thì rollback transaction, sau khi method gọi xong thì commit transaction.

> Inside method `invoke()` in class `TransactionInterceptor`, it actually calls `invokeWithinTransaction()` of class `TransactionAspectSupport`. Due to major refactoring in new Spring versions and use of reactive programming, source code is omitted here.

#### Vấn đề tự gọi (Self-Invocation) trong Spring AOP

Khi một method được đánh dấu annotation `@Transactional`, trình quản lý transaction của Spring chỉ có hiệu lực khi được gọi bởi method của class khác, chứ không có hiệu lực khi gọi giữa các method trong cùng một class.

Điều này do nguyên lý hoạt động của Spring AOP quyết định. Vì Spring AOP sử dụng Dynamic Proxy để thực hiện quản lý transaction, nó sẽ sinh ra object proxy cho method mang annotation `@Transactional` lúc runtime, và áp dụng logic transaction ở trước và sau cuộc gọi method. Nếu method đó được class khác gọi thì object proxy của chúng ta sẽ chặn cuộc gọi method và xử lý transaction. Tuy nhiên khi gọi nội bộ trong các method khác của cùng một class, object proxy của chúng ta không thể chặn được cuộc gọi nội bộ này, do đó transaction cũng bị mất hiệu lực.

`method1()` trong class `MyService` gọi `method2()` sẽ dẫn đến transaction của `method2()` bị mất hiệu lực.

```java
@Service
public class MyService {

private void method1() {
     method2();
     //......
}
@Transactional
 public void method2() {
     //......
  }
}
```

Cách giải quyết là tránh tự gọi trong cùng một class hoặc sử dụng AspectJ thay thế cho Spring AOP proxy.

[issue #2091](https://github.com/Snailclimb/JavaGuide/issues/2091) bổ sung một ví dụ:

```java
@Service
public class MyService {

private void method1() {
     // Cần cấu hình @EnableAspectJAutoProxy(exposeProxy = true) trước
     ((MyService) AopContext.currentProxy()).method2();
     //......
}
@Transactional
 public void method2() {
     //......
  }
}
```

Code trên chỉ khi bật `exposeProxy` (ví dụ cấu hình `@EnableAspectJAutoProxy(exposeProxy = true)`) mới có thể thông qua `AopContext.currentProxy()` để lấy object proxy hiện tại. Như vậy gọi `method2()` sẽ đi qua proxy, annotation transaction mới có hiệu lực. Vì cách viết này khiến code nghiệp vụ phụ thuộc vào AOP context, nên thông thường khuyên bạn nên tách trách nhiệm class để tránh tự gọi.

#### Tóm tắt lưu ý khi sử dụng `@Transactional`

- Giới hạn hiển thị method của `@Transactional` phụ thuộc vào loại proxy và phiên bản Spring: Proxy class của Spring 6 mặc định hỗ trợ method `public`, `protected` và package-private, proxy interface yêu cầu method là method `public` định nghĩa trong interface; pattern proxy của các phiên bản sớm hơn thường chỉ hỗ trợ method `public`;
- Tránh gọi method gắn annotation `@Transactional` trong cùng một class, như vậy sẽ dẫn đến transaction bị mất hiệu lực;
- Thiết lập đúng các thuộc tính `rollbackFor` và `propagation` của `@Transactional`, nếu không transaction có thể rollback thất bại;
- Class chứa method gắn annotation `@Transactional` phải do Spring quản lý, nếu không sẽ không có hiệu lực;
- Database bên dưới phải hỗ trợ cơ chế transaction, nếu không sẽ không có hiệu lực;
- ……

## Tham khảo

- [Tổng kết] Các parameter của @Transactional trong quản lý Spring Transaction: [http://www.mobabel.net/spring 事务管理中 transactional 的参数/](http://www.mobabel.net/spring事务管理中transactional的参数/)
- Tài liệu chính thức Spring: [https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html)
- 《Lập trình Nâng cao Spring 5》
- Nắm vững cách dùng @transactional trong Spring: [https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html)
- Đặc tính lan truyền của Spring Transaction: [https://github.com/love-somnus/Spring/wiki/Spring 事务的传播特性](https://github.com/love-somnus/Spring/wiki/Spring事务的传播特性)
- [Giải thích chi tiết Hành vi lan truyền Transaction trong Spring](https://segmentfault.com/a/1190000013341344): [https://segmentfault.com/a/1190000013341344](https://segmentfault.com/a/1190000013341344)
- Phân tích toàn diện Quản lý transaction lập trình và Quản lý transaction khai báo của Spring: [https://www.ibm.com/developerworks/cn/education/opensource/os-cn-spring-trans/index.html](https://www.ibm.com/developerworks/cn/education/opensource/os-cn-spring-trans/index.html)

<!-- @include: @article-footer.snippet.md -->
