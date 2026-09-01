---
title: Spring常见面试题总结
description: Spring框架核心面试题详解，涵盖IoC容器、AOP原理、Bean生命周期、依赖注入等Spring核心知识点。
category: 框架
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: Spring面试题,Spring框架,Bean生命周期,IoC,AOP,依赖注入,事务,Spring常见问题
---

Bài viết này chủ yếu muốn thông qua một số câu hỏi để giúp mọi người hiểu sâu hơn về Spring, vì vậy sẽ không liên quan quá nhiều đến code!

Rất nhiều câu hỏi dưới đây bản thân tôi trong quá trình sử dụng Spring cũng không chú ý tới, mà cũng là tạm thời tra cứu rất nhiều tài liệu và sách vở để bổ sung. Trên mạng cũng có rất nhiều bài viết tổng hợp về các câu hỏi/câu hỏi phỏng vấn thường gặp về Spring, tôi cảm thấy đa số là copy lẫn nhau, hơn nữa nhiều câu hỏi cũng không hay lắm, một số câu trả lời cũng có vấn đề. Do đó, bản thân tôi đã dành một tuần thời gian rảnh rỗi để tổng hợp lại, hy vọng sẽ có ích cho mọi người.

## Nền tảng Spring

### Spring Framework là gì?

Spring là một framework phát triển Java mã nguồn mở, nhẹ (lightweight), nhằm mục đích nâng cao hiệu suất phát triển của lập trình viên cũng như khả năng bảo trì của hệ thống.

Chúng ta thường nói Spring Framework là tập hợp của nhiều module, sử dụng các module này có thể rất tiện lợi trong việc hỗ trợ chúng ta phát triển, ví dụ như Spring hỗ trợ IoC (Inversion of Control: Điều khiển đảo ngược) và AOP (Aspect-Oriented Programming: Lập trình hướng khía cạnh), có thể rất tiện lợi trong việc truy cập cơ sở dữ liệu, tích hợp các component bên thứ ba (email, task, scheduling, cache,...), hỗ trợ unit test khá tốt, hỗ trợ phát triển ứng dụng RESTful Java.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/38ef122122de4375abcd27c3de8f60b4.png)

Tư tưởng cốt lõi nhất của Spring là không phát minh lại bánh xe, mở hộp là dùng ngay (out of the box), nâng cao hiệu suất phát triển.

Spring dịch ra có nghĩa là "mùa xuân", có thể thấy mục tiêu và sứ mệnh của nó là mang lại mùa xuân cho các lập trình viên Java! Thật cảm động!

🤐 Nói thêm một chút: **Sự phổ biến của một ngôn ngữ thường cần một ứng dụng sát thủ (killer app), và Spring chính là một framework ứng dụng sát thủ của hệ sinh thái Java.**

Các tính năng cốt lõi do Spring cung cấp chủ yếu là IoC và AOP. Học Spring, nhất định phải hiểu rõ tư tưởng cốt lõi của IoC và AOP!

- Trang chủ Spring: <https://spring.io/>
- Địa chỉ GitHub: <https://github.com/spring-projects/spring-framework>

### Spring bao gồm những module nào?

**Phiên bản Spring 4.x**:

![Các module chính của Spring 4.x](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/jvme0c60b4606711fc4a0b6faf03230247a.png)

**Phiên bản Spring 5.x**:

![Các module chính của Spring 5.x](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/20200831175708.png)

Trong phiên bản Spring 5.x, component Portlet trong module Web đã bị loại bỏ (deprecated), đồng thời thêm vào component WebFlux dùng cho xử lý phản ứng bất đồng bộ (asynchronous reactive processing).

Mối quan hệ phụ thuộc giữa các module của Spring như sau:

![Mối quan hệ phụ thuộc giữa các module của Spring](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/20200902100038.png)

#### Core Container

Module cốt lõi của Spring Framework, cũng có thể coi là module nền tảng, chủ yếu cung cấp sự hỗ trợ cho tính năng IoC Dependency Injection. Tất cả các tính năng khác của Spring về cơ bản đều cần phụ thuộc vào module này, chúng ta có thể thấy điều đó từ sơ đồ quan hệ phụ thuộc các module của Spring ở trên.

- **spring-core**: Các class tiện ích cốt lõi cơ bản của Spring Framework.
- **spring-beans**: Cung cấp sự hỗ trợ cho các tính năng tạo, cấu hình và quản lý Bean.
- **spring-context**: Cung cấp sự hỗ trợ cho các tính năng đa ngôn ngữ (internationalization), lan truyền event, load tài nguyên,...
- **spring-expression**: Cung cấp sự hỗ trợ cho ngôn ngữ biểu thức (Spring Expression Language) SpEL, chỉ phụ thuộc vào module core, không phụ thuộc vào các module khác, có thể sử dụng độc lập.

#### AOP

- **spring-aspects**: Module này cung cấp sự hỗ trợ tích hợp với AspectJ.
- **spring-aop**: Cung cấp việc thực thi lập trình hướng khía cạnh (AOP).
- **spring-instrument**: Cung cấp tính năng thêm agent cho JVM. Cụ thể, nó cung cấp một weaving agent cho Tomcat, có khả năng truyền các file class cho Tomcat giống như các file này được load bởi ClassLoader. Nếu chưa hiểu cũng không sao, kịch bản sử dụng của module này rất hạn chế.

#### Data Access/Integration

Danh sách các module dưới đây chủ yếu dựa trên Spring Framework 5.x. Spring Framework hiện đại đã loại bỏ tích hợp của một số công nghệ cũ, khi sử dụng thực tế nên lấy danh sách module chính thức của phiên bản Spring mục tiêu làm chuẩn.

- **spring-jdbc**: Cung cấp sự trừu tượng hóa JDBC đối với truy cập cơ sở dữ liệu. Các cơ sở dữ liệu khác nhau có API độc lập riêng để thao tác với cơ sở dữ liệu, còn chương trình Java chỉ cần tương tác với JDBC API, từ đó che đi sự ảnh hưởng của cơ sở dữ liệu.
- **spring-tx**: Cung cấp sự hỗ trợ cho transaction.
- **spring-orm**: Trong Spring Framework 5.x cung cấp sự hỗ trợ cho các công nghệ ORM như Hibernate, JPA,...; các phiên bản Spring sớm hơn còn từng cung cấp tích hợp iBATIS.
- **spring-oxm**: Cung cấp trừu tượng hóa OXM (Object-to-XML Mapping). Các phiên bản Spring khác nhau hỗ trợ các triển khai cụ thể khác nhau, ví dụ JAXB; Castor, XMLBeans, JiBX,... thuộc về tích hợp ở các phiên bản cũ.
- **spring-jms**: Dịch vụ tin nhắn (messaging service). Từ Spring Framework 4.1 trở đi, nó còn cung cấp sự kế thừa cho module spring-messaging.

#### Spring Web

- **spring-web**: Cung cấp một số hỗ trợ nền tảng nhất cho việc thực hiện các tính năng Web.
- **spring-webmvc**: Cung cấp sự thực thi cho Spring MVC.
- **spring-websocket**: Cung cấp sự hỗ trợ cho WebSocket, WebSocket có thể cho phép client và server giao tiếp hai chiều.
- **spring-webflux**: Cung cấp sự hỗ trợ cho WebFlux. WebFlux là một web framework phản ứng (reactive), phi bất đồng bộ (non-blocking) được giới thiệu trong Spring Framework 5.0, có thể chạy trên Netty, hoặc trên các Servlet container hỗ trợ I/O phi bất đồng bộ. Việc ứng dụng có phi bất đồng bộ end-to-end hay không còn phụ thuộc vào truy cập dữ liệu và các cuộc gọi downstream khác có chứa thao tác blocking hay không.

#### Messaging

**spring-messaging** là một module mới được thêm vào từ Spring 4.0, nhiệm vụ chính là tích hợp một số ứng dụng truyền tin nhắn cơ bản cho Spring Framework.

#### Spring Test

Team Spring khuyến khích phát triển hướng kiểm thử (TDD). Nhờ có sự giúp đỡ của Inversion of Control (IoC), unit test và integration test trở nên đơn giản hơn.

Module test của Spring hỗ trợ khá tốt cho các framework test thường dùng như JUnit (framework unit test), TestNG (tương tự JUnit), Mockito (chủ yếu dùng để Mock object), PowerMock (giải quyết các vấn đề của Mockito như không thể mock method final, static, private),...

### ⭐️Mối quan hệ giữa Spring, Spring MVC, Spring Boot là gì?

Rất nhiều người không phân biệt rõ ràng giữa Spring, Spring MVC và Spring Boot! Ở đây xin giới thiệu ngắn gọn về ba cái này, thực ra rất đơn giản, không có gì quá cao siêu.

Spring bao gồm nhiều module tính năng (như vừa đề cập ở trên), trong đó quan trọng nhất là module Spring-Core (chủ yếu cung cấp sự hỗ trợ cho tính năng IoC Dependency Injection), việc thực hiện tính năng của các module khác trong Spring (như Spring MVC) về cơ bản đều cần phụ thuộc vào module này.

Sơ đồ dưới đây tương ứng với phiên bản Spring 4.x. Spring 5.0 đã giới thiệu WebFlux dùng cho xử lý reactive, và từng bước loại bỏ hỗ trợ liên quan đến Portlet; cấu thành module của các phiên bản Spring hiện đại vui lòng tham khảo tài liệu chính thức.

![Các module chính của Spring](https://oss.javaguide.cn/github/javaguide/jvme0c60b4606711fc4a0b6faf03230247a.png)

Spring MVC là một module rất quan trọng trong Spring, chủ yếu trao cho Spring khả năng nhanh chóng xây dựng các ứng dụng Web theo kiến trúc MVC. MVC là viết tắt của Model (Mô hình), View (Giao diện), Controller (Bộ điều khiển), tư tưởng cốt lõi của nó là tổ chức code bằng cách tách biệt business logic, dữ liệu và hiển thị.

![](https://oss.javaguide.cn/java-guide-blog/image-20210809181452421.png)

Sử dụng Spring để phát triển thì các cấu hình quá phiền phức, ví dụ như khi bật một số tính năng của Spring, cần phải sử dụng XML hoặc Java để cấu hình tường minh. Thế là, Spring Boot ra đời!

Spring nhằm mục đích đơn giản hóa việc phát triển ứng dụng doanh nghiệp J2EE. Spring Boot nhằm mục đích đơn giản hóa việc phát triển Spring (giảm thiểu file cấu hình, mở hộp dùng ngay!).

Spring Boot chỉ đơn giản hóa cấu hình, nếu bạn cần xây dựng ứng dụng Web kiến trúc MVC, bạn vẫn cần sử dụng Spring MVC làm MVC framework, chỉ có điều Spring Boot giúp bạn đơn giản hóa rất nhiều cấu hình của Spring MVC, thực sự đạt được mở hộp dùng ngay!

## Spring IoC

### ⭐️IoC là gì?

IoC (Inversion of Control) tức Điều khiển đảo ngược / Đảo ngược điều khiển. Nó là một tư tưởng chứ không phải một triển khai kỹ thuật. Nó mô tả vấn đề tạo và quản lý object trong lĩnh vực phát triển Java.

Ví dụ: Hiện tại class A phụ thuộc vào class B

- **Phương thức phát triển truyền thống**: Thường là trong class A thủ công dùng từ khóa new để new một object B ra.
- **Phương thức phát triển sử dụng tư tưởng IoC**: Không thông qua từ khóa new để tạo object, mà thông qua IoC container (Spring Framework) để giúp chúng ta khởi tạo object. Chúng ta cần object nào, trực tiếp lấy từ IoC container ra là được.

Tương quan so sánh giữa hai phương thức phát triển trên: Chúng ta "mất đi một quyền hạn" (quyền hạn tạo, quản lý object), từ đó cũng nhận được một lợi ích (không cần phải suy nghĩ về một loạt các việc như tạo, quản lý object nữa).

**Tại sao gọi là Điều khiển đảo ngược (Inversion of Control)?**

- **Điều khiển**: Chỉ quyền hạn tạo (khởi tạo, quản lý) object.
- **Đảo ngược**: Quyền điều khiển được giao cho môi trường bên ngoài (IoC container).

![Minh họa IoC](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration.png)

### ⭐️IoC giải quyết vấn đề gì?

Tư tưởng của IoC chính là hai bên không phụ thuộc lẫn nhau, mà do container bên thứ ba quản lý các tài nguyên liên quan. Như vậy có lợi ích gì?

1. Độ phụ thuộc hay mức độ phụ thuộc giữa các object giảm xuống;
2. Tài nguyên trở nên dễ quản lý hơn; ví dụ như bạn dùng Spring container cung cấp thì rất dễ dàng có thể thực hiện một Singleton.

Ví dụ: Hiện có một thao tác đối với User, sử dụng cấu trúc hai tầng Service và Dao để phát triển.

Trong trường hợp không sử dụng tư tưởng IoC, nếu tầng Service muốn sử dụng triển khai cụ thể của tầng Dao, cần thông qua từ khóa new trong `UserServiceImpl` để new thủ công class triển khai cụ thể `UserDaoImpl` của `IUserDao` (không thể new trực tiếp class interface).

Rất hoàn hảo, phương thức này cũng có thể thực hiện được, nhưng chúng ta hãy tưởng tượng kịch bản sau:

Trong quá trình phát triển đột nhiên nhận được một yêu cầu mới, phát triển một class triển khai cụ thể khác cho interface `IUserDao`. Vì tầng Service phụ thuộc vào triển khai cụ thể của `IUserDao`, nên chúng ta cần sửa object new trong `UserServiceImpl`. Nếu chỉ có một class tham chiếu đến triển khai cụ thể của `IUserDao`, có thể cảm thấy không sao, sửa lại cũng không tốn nhiều sức, nhưng nếu có rất nhiều nơi đều tham chiếu đến triển khai cụ thể của `IUserDao`, một khi cần thay đổi phương thức triển khai của `IUserDao`, việc sửa đổi đó sẽ vô cùng đau đầu.

![IoC&Aop-ioc-illustration-dao-service](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao-service.png)

Sử dụng tư tưởng IoC, chúng ta giao quyền điều khiển object (tạo, quản lý) cho IoC container quản lý, khi sử dụng chúng ta trực tiếp "hỏi xin" IoC container là được.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao.png)

### Spring Bean là gì?

Nói một cách đơn giản, Bean đại diện cho những object được quản lý bởi IoC container.

Chúng ta cần nói cho IoC container biết cần giúp chúng ta quản lý những object nào, điều này được định nghĩa thông qua metadata cấu hình (configuration metadata). Metadata cấu hình có thể là file XML, annotation hoặc class configuration Java.

```xml
<!-- Constructor-arg with 'value' attribute -->
<bean id="..." class="...">
   <constructor-arg value="..."/>
</bean>
```

Sơ đồ dưới đây hiển thị đơn giản cách IoC container sử dụng metadata cấu hình để quản lý object.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/062b422bd7ac4d53afd28fb74b2bc94d.png)

Hai package `org.springframework.beans` và `org.springframework.context` là nền tảng thực hiện IoC, nếu muốn nghiên cứu mã nguồn liên quan đến IoC thì có thể tìm hiểu hai package này.

### Các annotation khai báo một class thành Bean là gì?

- `@Component`: Annotation dùng chung, có thể đánh dấu bất kỳ class nào thành component của `Spring`. Nếu một Bean không biết thuộc về tầng nào, có thể sử dụng annotation `@Component` để đánh dấu.
- `@Repository`: Tương ứng với tầng lưu trữ (persistence layer) tức tầng Dao, chủ yếu dùng cho các thao tác liên quan đến cơ sở dữ liệu.
- `@Service`: Tương ứng với tầng dịch vụ (service layer), chủ yếu liên quan đến một số logic phức tạp, cần dùng đến tầng Dao.
- `@Controller`: Tương ứng với tầng điều khiển Spring MVC, chủ yếu dùng để nhận request của người dùng và gọi tầng `Service` để trả về dữ liệu cho trang phía frontend.

### Sự khác biệt giữa @Component và @Bean là gì?

- Annotation `@Component` tác động lên class, còn annotation `@Bean` tác động lên method.
- `@Component` thường tự động phát hiện và tự động cấu hình (auto-wire) vào Spring container thông qua quét classpath (chúng ta có thể sử dụng annotation `@ComponentScan` để định nghĩa path cần quét, từ đó tìm ra các class được đánh dấu cần lắp ráp để tự động đưa vào bean container của Spring). Annotation `@Bean` thường được chúng ta sử dụng trong method có gắn annotation đó để định nghĩa và tạo ra bean này, `@Bean` nói cho Spring biết đây là một instance của một class nào đó, khi tôi cần dùng nó thì trả lại cho tôi.
- Annotation `@Bean` có khả năng tùy biến mạnh hơn annotation `@Component`, hơn nữa trong nhiều trường hợp chúng ta chỉ có thể đăng ký bean thông qua annotation `@Bean`. Ví dụ khi chúng ta tham chiếu class trong thư viện bên thứ ba cần đưa vào `Spring` container, thì chỉ có thể thực hiện thông qua `@Bean`.

Ví dụ sử dụng annotation `@Bean`:

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

Code ở trên tương đương với cấu hình XML dưới đây:

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

Ví dụ dưới đây không thể thực hiện thông qua `@Component`:

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

### Các annotation để inject Bean là gì?

`@Autowired` do Spring cung cấp, cũng như `@Resource` và `@Inject` do chuẩn Jakarta cung cấp, đều có thể dùng để inject Bean.

| Annotation   | Package                                        | Nguồn gốc                              |
| ------------ | ---------------------------------------------- | -------------------------------------- |
| `@Autowired` | `org.springframework.beans.factory.annotation` | Spring 2.5+                            |
| `@Resource`  | `jakarta.annotation` (Spring 6+)               | Jakarta Annotations / JSR-250          |
| `@Inject`    | `jakarta.inject` (Spring 6+)                   | Jakarta Dependency Injection / JSR-330 |

`@Autowired` và `@Resource` được sử dụng phổ biến hơn.

### ⭐️Sự khác biệt giữa @Autowired và @Resource là gì?

`@Autowired` là annotation tích hợp sẵn của Spring, logic inject mặc định là **khớp theo kiểu (byType) trước, nếu tồn tại nhiều Bean cùng kiểu, thì mới thử lọc theo tên (byName)**.

Cụ thể như sau:

1. Ưu tiên dựa vào kiểu của interface / class để tìm kiếm Bean phù hợp trong Spring container. Nếu chỉ tìm thấy 1 Bean phù hợp với kiểu, inject trực tiếp, không cần xét đến tên;
2. Nếu tìm thấy nhiều Bean cùng kiểu (ví dụ một interface có nhiều class triển khai), thì sẽ thử khớp thông qua **tên thuộc tính hoặc tên tham số** với tên của Bean (tên Bean mặc định là tên class viết thường chữ cái đầu, trừ khi chỉ định tường minh qua `@Bean(name = "...")` hoặc `@Component("...")`).

Khi một interface tồn tại nhiều class triển khai:

- Nếu tên thuộc tính khớp với tên của một Bean nào đó, thì inject Bean đó;
- Nếu tên thuộc tính không khớp với bất kỳ tên Bean nào, sẽ ném ra `NoUniqueBeanDefinitionException`, lúc này cần thông qua `@Qualifier` để chỉ định tường minh tên Bean cần inject.

Ví dụ minh họa:

```java
// Interface SmsService có hai class triển khai: SmsServiceImpl1, SmsServiceImpl2 (đều do Spring quản lý)

// Báo lỗi: byType khớp được nhiều Bean, và tên thuộc tính "smsService" không khớp với tên mặc định của hai class triển khai (smsServiceImpl1, smsServiceImpl2)
@Autowired
private SmsService smsService;

// Đúng: tên thuộc tính "smsServiceImpl1" khớp với tên mặc định của class triển khai SmsServiceImpl1
@Autowired
private SmsService smsServiceImpl1;

// Đúng: thông qua @Qualifier chỉ định tường minh tên Bean "smsServiceImpl1"
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;
```

Trong thực tế phát triển, chúng tôi vẫn khuyến nghị nên chỉ định tên tường minh thông qua annotation `@Qualifier` thay vì phụ thuộc vào tên của biến.

`@Resource` có nguồn gốc từ chuẩn **JSR-250**. Từ JDK 6 đến JDK 10, `javax.annotation.Resource` từng đi kèm với JDK; từ JDK 11 trở đi cần khai báo riêng dependency API. Dự án Spring 5 / Java EE 8 thường dùng `javax.annotation-api`, dự án Spring 6 / Jakarta EE 9 trở lên dùng `jakarta.annotation-api`.

Logic xử lý của Spring đối với `@Resource` (trường hợp không có tham số) như sau:

1. **Khớp theo tên (byName):** Mặc định lấy tên field (Field Name) làm tên bean để tìm kiếm trong container. Nếu tìm thấy Bean có tên đó, inject trực tiếp.
2. **Fallback về khớp theo kiểu (byType):** Nếu **không** tìm thấy Bean cùng tên, Spring sẽ lùi lại thử tìm kiếm theo **kiểu** của field. **Đánh giá kết quả khớp theo kiểu**:
   - **Tìm thấy 1 Bean**: Inject thành công.
   - **Tìm thấy 0 Bean**: Ném ra exception (`NoSuchBeanDefinitionException`).
   - **Tìm thấy >1 Bean**: Ném ra exception (`NoUniqueBeanDefinitionException`).

`@Resource` có hai thuộc tính khá quan trọng và thường dùng trong phát triển hàng ngày: `name` (tên), `type` (kiểu).

```java
public @interface Resource {
    String name() default "";
    Class<?> type() default Object.class;
}
```

Nếu chỉ chỉ định thuộc tính `name` thì phương thức inject là `byName`, nếu chỉ chỉ định thuộc tính `type` thì phương thức inject là `byType`, nếu đồng thời chỉ định thuộc tính `name` và `type` (không khuyến khích làm vậy) thì phương thức inject là `byType`+`byName`.

```java
// Báo lỗi, byName và byType đều không khớp được bean
@Resource
private SmsService smsService;
// Inject đúng bean tương ứng với object SmsServiceImpl1
@Resource
private SmsService smsServiceImpl1;
// Inject đúng bean tương ứng với object SmsServiceImpl1 (khuyên dùng cách này)
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

**Tóm tắt đơn giản lại**:

- `@Autowired` là annotation do Spring cung cấp, `@Resource` là annotation do chuẩn Jakarta Annotations/JSR-250 cung cấp.
- Phương thức inject mặc định của `@Autowired` là `byType` (khớp theo kiểu), phương thức inject mặc định của `@Resource` là `byName` (khớp theo tên).
- Khi một interface có nhiều class triển khai, cả `@Autowired` và `@Resource` đều cần thông qua tên mới có thể khớp chính xác đến Bean tương ứng. `Autowired` có thể thông qua annotation `@Qualifier` để chỉ định tên tường minh, `@Resource` có thể thông qua thuộc tính `name` để chỉ định tên tường minh.
- `@Autowired` hỗ trợ sử dụng trên constructor, method, field và parameter. `@Resource` chủ yếu dùng để inject trên field và method, không hỗ trợ sử dụng trên constructor hay parameter.

Xét tới việc ngữ nghĩa của `@Resource` rõ ràng hơn (ưu tiên tên), đồng thời là chuẩn Java, có thể giảm sự phụ thuộc chặt chẽ vào Spring Framework, chúng tôi thường **khuyến khích sử dụng `@Resource` hơn**, đặc biệt là trong các kịch bản cần inject theo tên. Còn `@Autowired` kết hợp với constructor injection có ưu thế trong việc thực hiện tính bất biến (immutability) và tính bắt buộc của dependency injection, cũng là một thực hành rất tốt.

### Có những cách inject Bean nào?

Các cách thường gặp của Dependency Injection (DI):

1. Constructor injection: Inject dependency thông qua constructor của class.
2. Setter injection: Inject dependency thông qua method Setter của class.
3. Field injection: Trực tiếp sử dụng annotation (như `@Autowired` hoặc `@Resource`) trên field của class để inject dependency.

Ví dụ Constructor injection:

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
```

Ví dụ Setter injection:

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
```

Ví dụ Field injection:

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    //...
}
```

### ⭐️Constructor injection hay Setter injection?

Trang chính thức của Spring có câu trả lời cho câu hỏi này: <https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html#beans-setter-injection>.

Ở đây tôi chủ yếu trích xuất và tổng hợp hoàn thiện đề xuất chính thức của Spring.

**Chính thức Spring khuyến nghị Constructor injection**, ưu điểm của cách inject này như sau:

1. Tính toàn vẹn của phụ thuộc: Đảm bảo tất cả các dependency bắt buộc đều được inject khi tạo object, tránh được rủi ro NullPointerException.
2. Tính bất biến (Immutability): Giúp tạo ra các object bất biến, nâng cao tính an toàn đa luồng (thread safety).
3. Đảm bảo khởi tạo: Component đã được khởi tạo hoàn chỉnh trước khi sử dụng, giảm thiểu các lỗi tiềm ẩn.
4. Tiện lợi cho testing: Trong unit test, có thể truyền trực tiếp các dependency mock thông qua constructor mà không cần phụ thuộc vào Spring container để inject.

Constructor injection thích hợp cho việc xử lý các **dependency bắt buộc**, còn **Setter injection** thì thích hợp hơn với các **dependency tùy chọn**, những dependency này có thể có giá trị mặc định hoặc được thiết lập động trong vòng đời của object. Mặc dù `@Autowired` có thể dùng cho method Setter để xử lý dependency bắt buộc, nhưng Constructor injection vẫn là lựa chọn tốt hơn.

Trong một số trường hợp (ví dụ class bên thứ ba không cung cấp method Setter), Constructor injection có thể là **lựa chọn duy nhất**.

### ⭐️Bean có những scope (phạm vi tác dụng) nào?

Scope của Bean trong Spring thường có các loại sau:

- **singleton**: Trong IoC container chỉ có duy nhất một instance bean. Bean trong Spring mặc định đều là singleton, đây là ứng dụng của Singleton Design Pattern.
- **prototype**: Mỗi lần lấy đều sẽ tạo một instance bean mới. Nghĩa là, gọi `getBean()` hai lần liên tiếp sẽ nhận được hai instance Bean khác nhau.
- **request** (chỉ dùng cho ứng dụng Web): Mỗi một request HTTP đều tạo ra một bean mới (request bean), bean này chỉ có hiệu lực trong request HTTP hiện tại.
- **session** (chỉ dùng cho ứng dụng Web): Mỗi một request HTTP đến từ session mới đều tạo ra một bean mới (session bean), bean này chỉ có hiệu lực trong session HTTP hiện tại.
- **application/global-session** (chỉ dùng cho ứng dụng Web): Mỗi ứng dụng Web khi khởi động sẽ tạo một Bean (application bean), bean này chỉ có hiệu lực trong thời gian chạy của ứng dụng hiện tại.
- **websocket** (chỉ dùng cho ứng dụng Web): Mỗi phiên WebSocket tạo ra một bean mới.

**Cấu hình scope của bean như thế nào?**

Cách XML:

```xml
<bean id="..." class="..." scope="singleton"></bean>
```

Cách Annotation:

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```

### ⭐️Bean có thread-safe không?

Bean trong Spring Framework có thread-safe hay không phụ thuộc vào scope và state (trạng thái) của nó.

Ở đây chúng ta lấy hai scope thường dùng nhất là prototype và singleton để giới thiệu. Hầu như mọi kịch bản scope của Bean đều sử dụng singleton mặc định, nên chỉ cần tập trung chú ý vào scope singleton.

Dưới scope prototype, mỗi lần xin từ container đều tạo một instance bean mới, có thể giảm xác suất chia sẻ ở cấp độ container, nhưng bản thân scope không cung cấp đảm bảo thread-safe: nếu bên gọi chia sẻ cùng một instance prototype cho nhiều thread, sự tranh chấp tài nguyên vẫn có thể xảy ra. Dưới scope singleton, trong IoC container chỉ có duy nhất một instance bean, dễ xuất hiện vấn đề tranh chấp state được chia sẻ hơn (phụ thuộc vào Bean có state hay không).

Ví dụ Bean có state (stateful Bean):

```java
// Định nghĩa một class giỏ hàng, trong đó chứa một List lưu trữ các sản phẩm trong giỏ hàng của user
@Component
public class ShoppingCart {
    private List<String> items = new ArrayList<>();

    public void addItem(String item) {
        items.add(item);
    }

    public List<String> getItems() {
        return items;
    }
}
```

Tuy nhiên, phần lớn các Bean thực tế đều là không có state (stateless, không định nghĩa biến thành viên có thể thay đổi) (như Dao, Service), trong trường hợp này, Bean là thread-safe.

Ví dụ Bean không có state (stateless Bean):

```java
// Định nghĩa một user service, nó chỉ chứa business logic mà không lưu trữ bất kỳ state nào.
@Component
public class UserService {

    public User findUserById(Long id) {
        //...
    }
    //...
}
```

Đối với vấn đề thread-safe của singleton Bean có state, ba cách giải quyết thường gặp là:

1. **Tránh biến thành viên có thể thay đổi**: Cố gắng thiết kế Bean thành không có state.
2. **Sử dụng `ThreadLocal`**: Lưu biến thành viên có thể thay đổi trong `ThreadLocal`, đảm bảo độc lập giữa các thread.
3. **Sử dụng cơ chế đồng bộ (synchronization)**: Sử dụng `synchronized` hoặc `ReentrantLock` để kiểm soát đồng bộ, đảm bảo thread-safe.

Ở đây lấy `ThreadLocal` làm ví dụ, minh họa kịch bản `ThreadLocal` lưu trữ thông tin đăng nhập của user:

```java
public class UserThreadLocal {

    private UserThreadLocal() {}

    private static final ThreadLocal<SysUser> LOCAL = ThreadLocal.withInitial(() -> null);

    public static void put(SysUser sysUser) {
        LOCAL.set(sysUser);
    }

    public static SysUser get() {
        return LOCAL.get();
    }

    public static void remove() {
        LOCAL.remove();
    }
}
```

### ⭐️Bạn có hiểu vòng đời của Bean không?

1. **Tạo instance của Bean**: Container Bean đầu tiên sẽ tìm định nghĩa Bean trong file cấu hình, sau đó chọn chiến lược khởi tạo thích hợp (factory method, constructor auto-wire hoặc khởi tạo đơn giản) thông qua Java Reflection API để tạo instance của Bean.
2. **Gán giá trị / Điền thuộc tính Bean (populateBean)**: Thiết lập các thuộc tính và dependency liên quan cho Bean, ví dụ xử lý các annotation `@Autowired`, `@Value`, `@Resource` gắn trên field hoặc method Setter.
3. **Khởi tạo Bean (initializeBean)**:
   - Nếu Bean implement interface `BeanNameAware`, gọi method `setBeanName()`, truyền vào tên của Bean.
   - Nếu Bean implement interface `BeanClassLoaderAware`, gọi method `setBeanClassLoader()`, truyền vào instance đối tượng `ClassLoader`.
   - Nếu Bean implement interface `BeanFactoryAware`, gọi method `setBeanFactory()`, truyền vào instance đối tượng `BeanFactory`.
   - Tương tự như trên, nếu implement các interface `*.Aware` khác, thì gọi method tương ứng.
   - Nếu có đối tượng `BeanPostProcessor` liên quan đến Spring container đang load Bean này, thực thi method `postProcessBeforeInitialization()`.
   - Nếu Bean implement interface `InitializingBean`, thực thi method `afterPropertiesSet()`.
   - Nếu định nghĩa của Bean trong file cấu hình chứa thuộc tính `init-method`, thực thi method được chỉ định.
   - Nếu có đối tượng `BeanPostProcessor` liên quan đến Spring container đang load Bean này, thực thi method `postProcessAfterInitialization()`.
4. **Hủy Bean (destroy)**: Hủy không có nghĩa là lập tức tiêu hủy Bean, mà là ghi nhận lại method hủy của Bean trước, sau này khi cần hủy Bean hoặc hủy container, sẽ gọi các method này để giải phóng tài nguyên mà Bean nắm giữ.
   - Nếu Bean implement interface `DisposableBean`, thực thi method `destroy()`.
   - Nếu định nghĩa của Bean trong file cấu hình chứa thuộc tính `destroy-method`, thực thi method hủy Bean được chỉ định. Hoặc cũng có thể đánh dấu trực tiếp method thực thi trước khi hủy Bean thông qua annotation `@PreDestroy`.

Trong method `doCreateBean()` của `AbstractAutowireCapableBeanFactory` có thể thấy lần lượt thực thi 4 giai đoạn này:

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {

    // 1. Tạo instance của Bean
    BeanWrapper instanceWrapper = null;
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    Object exposedObject = bean;
    try {
        // 2. Gán giá trị / Điền thuộc tính Bean
        populateBean(beanName, mbd, instanceWrapper);
        // 3. Khởi tạo Bean
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }

    // 4. Hủy Bean - Đăng ký interface callback
    try {
        registerDisposableBeanIfNecessary(beanName, bean, mbd);
    }

    return exposedObject;
}
```

Interface `Aware` giúp Bean có thể lấy được tài nguyên từ Spring container.

Các interface `Aware` chính được cung cấp trong Spring là:

1. `BeanNameAware`: Inject beanName tương ứng với bean hiện tại;
2. `BeanClassLoaderAware`: Inject ClassLoader load bean hiện tại;
3. `BeanFactoryAware`: Inject tham chiếu đến `BeanFactory` container hiện tại.

Interface `BeanPostProcessor` là điểm mở rộng mạnh mẽ do Spring cung cấp để chỉnh sửa Bean.

```java
public interface BeanPostProcessor {

	// Xử lý trước khi khởi tạo
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	// Xử lý sau khi khởi tạo
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}
```

- `postProcessBeforeInitialization`: Thực thi sau khi Bean khởi tạo instance và inject thuộc tính xong, trước method `InitializingBean#afterPropertiesSet` cũng như method `init-method` tùy chỉnh;
- `postProcessAfterInitialization`: Tương tự như trên, nhưng thực thi sau method `InitializingBean#afterPropertiesSet` cũng như method `init-method` tùy chỉnh.

`InitializingBean` và `init-method` là các điểm mở rộng do Spring cung cấp cho việc khởi tạo Bean.

```java
public interface InitializingBean {
 // Logic khởi tạo
	void afterPropertiesSet() throws Exception;
}
```

Chỉ định method `init-method`, chỉ định method khởi tạo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="demo" class="com.chaycao.Demo" init-method="init"/>

</beans>
```

**Ghi nhớ như thế nào?**

1. Tổng thể có thể chia đơn giản thành 4 bước: Khởi tạo instance (Instantiation) —> Gán thuộc tính (Populate) —> Khởi tạo (Initialization) —> Hủy (Destruction).
2. Bước khởi tạo này liên quan đến khá nhiều bước, bao gồm Dependency Injection của interface `Aware`, xử lý trước và sau khởi tạo của `BeanPostProcessor`, cũng như thao tác khởi tạo của `InitializingBean` và `init-method`.
3. Bước hủy sẽ đăng ký các interface callback hủy liên quan, cuối cùng tiến hành hủy thông qua `DisposableBean` và `destroy-method`.

Cuối cùng, xin chia sẻ thêm một sơ đồ rõ ràng (nguồn hình: [Làm thế nào để ghi nhớ vòng đời Spring Bean](https://chaycao.github.io/2020/02/15/如何记忆Spring-Bean的生命周期.html)).

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/spring-bean-lifestyle.png)

## Spring AOP

### ⭐️Nói về hiểu biết của bản thân đối với AOP

AOP (Aspect-Oriented Programming: Lập trình hướng khía cạnh) có thể đóng gói những logic hoặc trách nhiệm không liên quan đến nghiệp vụ nhưng lại được các module nghiệp vụ cùng gọi (như xử lý transaction, quản lý log, kiểm soát phân quyền,...), giúp giảm code trùng lặp trong hệ thống, giảm độ gắn kết (coupling) giữa các module, và có lợi cho khả năng mở rộng cũng như bảo trì trong tương lai.

Spring AOP dựa trên Dynamic Proxy. Nếu object cần proxy implement một interface nào đó, Spring AOP sẽ sử dụng **JDK Proxy** để tạo object proxy; còn đối với object không implement interface, sẽ không thể sử dụng JDK Proxy để proxy nữa, lúc này Spring AOP sẽ sử dụng **Cglib** để sinh ra một class con của object được proxy làm proxy, như hình dưới đây:

![Quy trình Spring AOP](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

Tất nhiên bạn cũng có thể sử dụng **AspectJ**! Spring AOP đã tích hợp AspectJ, AspectJ được coi là AOP framework hoàn chỉnh nhất trong hệ sinh thái Java.

Một số thuật ngữ chuyên ngành liên quan đến AOP aspect programming:

| Thuật ngữ         | Ý nghĩa                                                                                             |
| :---------------- | :-------------------------------------------------------------------------------------------------- |
| Target (Mục tiêu) | Object được advice                                                                                  |
| Proxy (Ủy quyền)  | Object proxy được tạo ra sau khi áp dụng advice vào target object                                   |
| JoinPoint (Điểm nối)| Tất cả các method được định nghĩa trong class thuộc về target object đều là joinpoint              |
| Pointcut (Điểm cắt)| JoinPoint bị aspect chặn / tăng cường (Pointcut chắc chắn là JoinPoint, JoinPoint chưa chắc là Pointcut)|
| Advice (Thông báo/Tăng cường)| Logic / code tăng cường, tức là những việc cần làm sau khi chặn được JoinPoint của target object|
| Aspect (Khía cạnh)| Pointcut + Advice                                                                                   |
| Weaving (Dệt/Kết hợp)| Quá trình/hành động áp dụng advice vào target object, từ đó sinh ra proxy object                  |

### ⭐️Sự khác biệt giữa Spring AOP và AspectJ AOP là gì?

| Tính chất           | Spring AOP                                                | AspectJ                                    |
| ------------------- | --------------------------------------------------------- | ------------------------------------------ |
| **Cách thức tăng cường** | Tăng cường lúc runtime (dựa trên Dynamic Proxy)      | Tăng cường lúc compile, lúc load class (thao tác trực tiếp với bytecode) |
| **Hỗ trợ Pointcut** | Cấp method (trong phạm vi Spring Bean, không hỗ trợ method final và static) | Cấp method, field, constructor, static method,... |
| **Hiệu năng**       | Phụ thuộc vào proxy lúc runtime, có overhead nhất định, khi có nhiều aspect hiệu năng thấp hơn | Runtime không có overhead proxy, hiệu năng cao hơn |
| **Độ phức tạp**     | Đơn giản, dễ dùng, phù hợp hầu hết kịch bản              | Mạnh mẽ, nhưng tương đối phức tạp          |
| **Kịch bản sử dụng**| Nhu cầu AOP tương đối đơn giản trong ứng dụng Spring     | Nhu cầu AOP hiệu năng cao, độ phức tạp cao |

**Lựa chọn thế nào?**

- **Xem xét tính năng**: AspectJ hỗ trợ các kịch bản AOP phức tạp hơn, Spring AOP đơn giản dễ dùng hơn. Nếu bạn cần tăng cường các method `final`, method static, truy cập field, gọi constructor,... hoặc cần áp dụng logic tăng cường trên các object không do Spring quản lý, AspectJ là lựa chọn duy nhất.
- **Xem xét hiệu năng**: Khi số lượng aspect ít thì hiệu năng cả hai không khác biệt nhiều, nhưng khi số lượng aspect nhiều thì AspectJ có hiệu năng tốt hơn.

**Tóm lại một câu**: Kịch bản đơn giản ưu tiên dùng Spring AOP; kịch bản phức tạp hoặc nhu cầu hiệu năng cao thì chọn AspectJ.

### ⭐️Các loại Advice thường gặp trong AOP là gì?

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/aspectj-advice-types.jpg)

- **Before** (Advice trước): Kích hoạt trước khi method của target object được gọi
- **After** (Advice sau): Kích hoạt sau khi method của target object được gọi
- **AfterReturning** (Advice trả về): Kích hoạt sau khi method của target object gọi xong, sau khi trả về giá trị kết quả
- **AfterThrowing** (Advice ngoại lệ): Kích hoạt sau khi method của target object đang chạy ném ra / kích hoạt exception. AfterReturning và AfterThrowing loại trừ lẫn nhau. Nếu method gọi thành công không có exception thì sẽ có giá trị trả về; nếu method ném ra exception thì sẽ không có giá trị trả về.
- **Around** (Advice bao quanh): Kiểm soát bằng lập trình đối với việc gọi method của target object. Around advice là loại advice có phạm vi thao tác lớn nhất trong tất cả các loại advice, vì nó có thể lấy trực tiếp target object cũng như method sắp thực thi, nên around advice có thể tùy ý thực hiện công việc trước và sau khi gọi method của target object, thậm chí không gọi method của target object.

### Thứ tự thực thi của nhiều Aspect được kiểm soát như thế nào?

1. Thường sử dụng annotation `@Order` để định nghĩa trực tiếp thứ tự aspect

```java
// Giá trị càng nhỏ ưu tiên càng cao
@Order(3)
@Component
@Aspect
public class LoggingAspect implements Ordered {
```

**2. Implement interface `Ordered` và override method `getOrder`.**

```java
@Component
@Aspect
public class LoggingAspect implements Ordered {

    // ....

    @Override
    public int getOrder() {
        // Giá trị trả về càng nhỏ ưu tiên càng cao
        return 1;
    }
}
```

## Spring MVC

### Nói về hiểu biết của bản thân đối với Spring MVC?

MVC là viết tắt của Model (Mô hình), View (Giao diện), Controller (Bộ điều khiển), tư tưởng cốt lõi của nó là tổ chức code bằng cách tách biệt business logic, dữ liệu và hiển thị.

![](https://oss.javaguide.cn/java-guide-blog/image-20210809181452421.png)

Trên mạng có rất nhiều người nói MVC không phải là pattern thiết kế, mà chỉ là quy chuẩn thiết kế phần mềm, cá nhân tôi nghiêng về việc MVC cũng là một trong số rất nhiều pattern thiết kế. Trong dự án **[java-design-patterns](https://github.com/iluwatar/java-design-patterns)** có giới thiệu liên quan đến MVC.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/159b3d3e70dd45e6afa81bf06d09264e.png)

Muốn thực sự hiểu Spring MVC, trước tiên chúng ta hãy nhìn vào hai thời đại Model 1 và Model 2 khi chưa có Spring MVC.

**Thời đại Model 1**

Rất nhiều bạn học Java backend muộn có thể chưa từng tiếp xúc với việc phát triển ứng dụng JavaWeb trong thời đại Model 1. Trong mô hình Model 1, toàn bộ ứng dụng Web hầu như được cấu thành hoàn toàn bằng các trang JSP, chỉ dùng một số lượng nhỏ JavaBean để xử lý các thao tác như kết nối, truy cập cơ sở dữ liệu.

Trong mô hình này, JSP vừa là tầng điều khiển (Controller) vừa là tầng hiển thị (View). Rõ ràng, mô hình này tồn tại rất nhiều vấn đề. Ví dụ logic điều khiển và logic hiển thị trộn lẫn vào nhau, dẫn đến tỷ lệ tái sử dụng code cực thấp; hay ví dụ frontend và backend phụ thuộc lẫn nhau, khó tiến hành test bảo trì và hiệu suất phát triển cực kỳ thấp.

![mvc-mode1](https://oss.javaguide.cn/java-guide-blog/mvc-mode1.png)

**Thời đại Model 2**

Những bạn đã học Servlet và làm qua Demo liên quan chắc hẳn biết mô hình phát triển "Java Bean (Model) + JSP (View) + Servlet (Controller)", đây chính là mô hình phát triển JavaWeb MVC thời kỳ đầu.

- Model: Dữ liệu liên quan đến hệ thống, tức dao và bean.
- View: Hiển thị dữ liệu trong model, chỉ dùng để hiển thị.
- Controller: Nhận request của người dùng, và gửi request đến Model, cuối cùng trả dữ liệu cho JSP và hiển thị cho người dùng.

![](https://oss.javaguide.cn/java-guide-blog/mvc-model2.png)

Dưới mô hình Model 2 vẫn còn tồn tại rất nhiều vấn đề, mức độ trừu tượng hóa và đóng gói của Model 2 vẫn chưa đủ, khi sử dụng Model 2 để phát triển không thể tránh khỏi việc lặp lại chế tạo bánh xe, điều này làm giảm đáng kể tính bảo trì và tính tái sử dụng của chương trình.

Thế là, rất nhiều MVC framework liên quan đến phát triển JavaWeb đã ra đời như Struts2, nhưng Struts2 cồng kềnh hơn.

**Thời đại Spring MVC**

Cùng với sự phổ biến của framework phát triển mỏng nhẹ Spring, trong hệ sinh thái Spring đã xuất hiện Spring MVC framework, Spring MVC hiện là MVC framework xuất sắc nhất. So với Struts2, Spring MVC sử dụng đơn giản và tiện lợi hơn, hiệu suất phát triển cao hơn, và Spring MVC chạy nhanh hơn.

MVC là một pattern thiết kế, Spring MVC là một MVC framework rất xuất sắc. Spring MVC có thể giúp chúng ta tiến hành phát triển tầng Web gọn gàng hơn, và nó vốn tích hợp tự nhiên với Spring Framework. Dưới Spring MVC, chúng ta thường chia dự án backend thành tầng Service (xử lý nghiệp vụ), tầng Dao (thao tác cơ sở dữ liệu), tầng Entity (class thực thể), tầng Controller (tầng điều khiển, trả dữ liệu cho trang frontend).

### Các component cốt lõi của Spring MVC là gì?

Ghi nhớ những component dưới đây, bạn cũng ghi nhớ nguyên lý hoạt động của SpringMVC.

- **`DispatcherServlet`**: **Bộ xử lý trung tâm cốt lõi**, chịu trách nhiệm nhận request, phân phát, và phản hồi cho client.
- **`HandlerMapping`**: **Bộ ánh xạ handler**, dựa vào URL để tìm kiếm khớp `Handler` có thể xử lý, và sẽ đóng gói các interceptor liên quan đến request cùng với `Handler`.
- **`HandlerAdapter`**: **Bộ thích ứng handler**, dựa vào `Handler` mà `HandlerMapping` tìm được, thích ứng để thực thi `Handler` tương ứng;
- **`Handler`**: **Bộ xử lý request**, xử lý request thực tế.
- **`ViewResolver`**: **Bộ giải mã view**, dựa vào view logic / view mà `Handler` trả về, giải mã và render view thực sự, và truyền cho `DispatcherServlet` để phản hồi client.

### ⭐️Bạn có hiểu nguyên lý hoạt động của SpringMVC không?

**Nguyên lý Spring MVC như hình dưới đây:**

> Hình minh họa nguyên lý hoạt động của SpringMVC tôi không tự vẽ, mà tìm trực tiếp một hình rất rõ ràng trực quan trên mạng, nguồn gốc ban đầu không rõ.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/de6d2b213f112297298f3e223bf08f28.png)

**Giải thích quy trình (quan trọng):**

1. Client (trình duyệt) gửi request, `DispatcherServlet` chặn request.
2. `DispatcherServlet` dựa vào thông tin request gọi `HandlerMapping`. `HandlerMapping` dựa vào URL để tìm kiếm khớp `Handler` có thể xử lý (tức là bộ điều khiển `Controller` mà chúng ta thường nói), và đóng gói các interceptor liên quan đến request cùng với `Handler`.
3. `DispatcherServlet` gọi adapter `HandlerAdapter` để thực thi `Handler`.
4. Sau khi `Handler` xử lý xong request của người dùng, sẽ trả về một object `ModelAndView` cho `DispatcherServlet`, `ModelAndView` đúng như tên gọi, chứa thông tin về data model và view tương ứng. `Model` là object dữ liệu trả về, `View` là một `View` về mặt logic.
5. `ViewResolver` sẽ dựa vào `View` logic để tìm kiếm `View` thực tế.
6. `DispatcherServlet` truyền `Model` trả về cho `View` (render view).
7. Trả `View` về cho người gửi request (trình duyệt).

Quy trình trên là nguyên lý hoạt động của mô hình phát triển truyền thống (JSP, Thymeleaf,...). Tuy nhiên hiện nay phương thức phát triển chủ đạo là tách biệt frontend và backend (decoupled frontend/backend), trong trường hợp này khái niệm `View` của Spring MVC đã có một số thay đổi. Vì `View` thường do frontend framework (Vue, React,...) xử lý, backend không còn chịu trách nhiệm render trang nữa, mà chỉ chịu trách nhiệm cung cấp dữ liệu, do đó:

- Khi tách biệt frontend và backend, backend thường không còn trả về view cụ thể nữa, mà trả về **dữ liệu thuần túy** (thường là định dạng JSON), do frontend chịu trách nhiệm render và hiển thị.
- Phần `View` trong kịch bản tách biệt frontend/backend thường không cần thiết lập, method controller của Spring MVC chỉ cần trả về dữ liệu, không trả về `ModelAndView` nữa, mà trực tiếp trả về dữ liệu, Spring sẽ tự động chuyển đổi nó sang định dạng JSON. Tương ứng, `ViewResolver` cũng sẽ không được sử dụng nữa.

Làm thế nào để đạt được điều đó?

- Sử dụng annotation `@RestController` thay cho annotation `@Controller` truyền thống, như vậy tất cả các method mặc định sẽ trả về dữ liệu định dạng JSON, chứ không cố gắng giải mã view.
- Nếu bạn sử dụng `@Controller`, có thể kết hợp với annotation `@ResponseBody` để trả về JSON.

### Làm thế nào để xử lý exception tập trung (tống nhất)?

Khuyên dùng cách xử lý exception tập trung thông qua annotation, cụ thể sẽ sử dụng hai annotation `@ControllerAdvice` + `@ExceptionHandler`.

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<?> handleAppException(BaseException ex, HttpServletRequest request) {
      //......
    }

    @ExceptionHandler(value = ResourceNotFoundException.class)
    public ResponseEntity<ErrorReponse> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
      //......
    }
}
```

Cách xử lý exception này không phải là tạo proxy AOP cho `Controller`. Sau khi method `Controller` ném ra exception, Spring MVC sẽ thông qua chuỗi xử lý `HandlerExceptionResolver` để tìm kiếm method `@ExceptionHandler` có thể xử lý exception đó.

Method `getMappedMethod` trong `ExceptionHandlerMethodResolver` quyết định exception cụ thể được xử lý bởi method nào có gắn annotation `@ExceptionHandler`.

```java
@Nullable
  private Method getMappedMethod(Class<? extends Throwable> exceptionType) {
    List<Class<? extends Throwable>> matches = new ArrayList<>();
    // Tìm tất cả thông tin exception có thể xử lý. mappedMethods lưu trữ mối quan hệ tương ứng giữa exception và method xử lý exception
    for (Class<? extends Throwable> mappedException : this.mappedMethods.keySet()) {
      if (mappedException.isAssignableFrom(exceptionType)) {
        matches.add(mappedException);
      }
    }
    // Không rỗng nghĩa là có method xử lý exception
    if (!matches.isEmpty()) {
      // Sắp xếp theo mức độ khớp từ nhỏ đến lớn
      matches.sort(new ExceptionDepthComparator(exceptionType));
      // Trả về method xử lý exception
      return this.mappedMethods.get(matches.get(0));
    }
    else {
      return null;
    }
  }
```

Từ mã nguồn có thể thấy: **`getMappedMethod()` trước tiên sẽ tìm tất cả các thông tin method có thể khớp xử lý exception, sau đó tiến hành sắp xếp từ nhỏ đến lớn, cuối cùng lấy method khớp nhỏ nhất đó (tức là method có độ khớp cao nhất).**

## Những pattern thiết kế nào được sử dụng trong Spring Framework?

> Về phần giới thiệu chi tiết các pattern thiết kế dưới đây, có thể đọc bài viết [Giải thích chi tiết Pattern thiết kế trong Spring](https://javaguide.cn/system-design/framework/spring/spring-design-patterns-summary.html) do tôi viết.

- **Factory Pattern**: Spring sử dụng Factory Pattern thông qua `BeanFactory`, `ApplicationContext` để tạo object bean.
- **Proxy Pattern**: Thực hiện tính năng Spring AOP.
- **Singleton Pattern**: Các Bean trong Spring mặc định đều là singleton.
- **Template Method Pattern**: Các class thao tác với cơ sở dữ liệu kết thúc bằng Template trong Spring như `jdbcTemplate`, `hibernateTemplate`,... đều sử dụng Template Pattern.
- **Decorator/Wrapper Pattern**: Dự án của chúng ta cần kết nối nhiều cơ sở dữ liệu, và các khách hàng khác nhau trong mỗi lần truy cập sẽ truy cập các cơ sở dữ liệu khác nhau tùy theo nhu cầu. Pattern này cho phép chúng ta có thể chuyển đổi động các datasource khác nhau dựa trên nhu cầu của khách hàng.
- **Observer Pattern**: Mô hình xử lý event (event-driven) trong Spring là một ứng dụng rất kinh điển của Observer Pattern.
- **Adapter Pattern**: Tăng cường hoặc advice của Spring AOP sử dụng Adapter Pattern, trong Spring MVC cũng sử dụng Adapter Pattern để thích ứng `Controller`.
- ……

## ⭐️Phụ thuộc vòng (Circular Dependency) trong Spring

### Bạn có hiểu phụ thuộc vòng trong Spring không, giải quyết như thế nào?

Phụ thuộc vòng chỉ việc các object Bean tham chiếu vòng lẫn nhau, là hai hoặc nhiều Bean giữ tham chiếu đến nhau, ví dụ CircularDependencyA → CircularDependencyB → CircularDependencyA.

```java
@Component
public class CircularDependencyA {
    @Autowired
    private CircularDependencyB circB;
}

@Component
public class CircularDependencyB {
    @Autowired
    private CircularDependencyA circA;
}
```

Tự phụ thuộc của một object đơn lẻ cũng có thể xuất hiện phụ thuộc vòng, nhưng xác suất này cực kỳ thấp, thuộc về lỗi viết code.

```java
@Component
public class CircularDependencyA {
    @Autowired
    private CircularDependencyA circA;
}
```

Spring Framework có thể giải quyết một phần phụ thuộc vòng của Setter/field injection đối với singleton Bean thông qua cơ chế bộ nhớ đệm 3 tầng (three-level cache). Các kịch bản phụ thuộc vòng trong constructor, phụ thuộc vòng của prototype Bean,... không thể dựa vào cơ chế này để giải quyết.

Bộ nhớ đệm 3 tầng trong Spring thực chất là 3 Map, như sau:

```java
// Cache cấp 1
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// Cache cấp 2
/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

// Cache cấp 3
/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

Nói một cách đơn giản, bộ nhớ đệm 3 tầng của Spring bao gồm:

1. **Cache cấp 1 (singletonObjects)**: Lưu trữ Bean dạng hoàn chỉnh cuối cùng (đã khởi tạo instance, điền thuộc tính, khởi tạo), pool singleton, sinh ra cho "thuộc tính singleton của Spring". Thông thường chúng ta lấy Bean đều lấy từ đây, nhưng không phải tất cả các Bean đều nằm trong pool singleton, ví dụ prototype Bean thì không nằm trong đó.
2. **Cache cấp 2 (earlySingletonObjects)**: Lưu trữ Bean chuyển tiếp (bán thành phẩm, chưa điền thuộc tính), tức là đối tượng do `ObjectFactory` ở cache cấp 3 tạo ra, được sử dụng phối hợp với cache cấp 3, có thể ngăn ngừa trường hợp AOP khiến cho mỗi lần gọi `ObjectFactory#getObject()` đều sinh ra đối tượng proxy mới.
3. **Cache cấp 3 (singletonFactories)**: Lưu trữ `ObjectFactory`, method `getObject()` của `ObjectFactory` (cuối cùng gọi method `getEarlyBeanReference()`) có thể sinh ra object Bean nguyên bản hoặc object proxy (nếu Bean bị AOP aspect proxy). Cache cấp 3 chỉ có hiệu lực với singleton Bean.

Tiếp theo nói về quy trình Spring tạo Bean:

1. Đầu tiên tìm trong **Cache cấp 1 `singletonObjects`**, nếu tồn tại thì trả về;
2. Nếu không tồn tại hoặc object đang trong quá trình tạo, liền tìm trong **Cache cấp 2 `earlySingletonObjects`**;
3. Nếu vẫn chưa lấy được, liền tìm trong **Cache cấp 3 `singletonFactories`**, thông qua thực thi `getObject()` của `ObjectFactory` là có thể lấy được object đó, sau khi lấy thành công, xóa khỏi cache cấp 3, và thêm object đó vào cache cấp 2.

Trong cache cấp 3 lưu trữ `ObjectFactory`:

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```

Khi Spring tạo Bean, nếu cho phép phụ thuộc vòng, Spring sẽ bộc lộ sớm (early expose) object Bean vừa khởi tạo instance xong nhưng thuộc tính chưa khởi tạo hoàn tất, ở đây thông qua method `addSingletonFactory` để thêm một đối tượng `ObjectFactory` vào cache cấp 3:

```java
// AbstractAutowireCapableBeanFactory # doCreateBean #
public abstract class AbstractAutowireCapableBeanFactory ... {
	protected Object doCreateBean(...) {
        //...

        // Hỗ trợ phụ thuộc vòng: đưa ()->getEarlyBeanReference làm method getObject() của một đối tượng ObjectFactory vào cache cấp 3
		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
}
```

Vậy ở trên khi nói về quy trình Spring tạo Bean đã nói, nếu cache cấp 1, cấp 2 đều không lấy được object, sẽ đến cache cấp 3 thông qua method `getObject()` của `ObjectFactory` để lấy object.

```java
class A {
    // Sử dụng B
    private B b;
}
class B {
    // Sử dụng A
    private A a;
}
```

Lấy code phụ thuộc vòng ở trên làm ví dụ, toàn bộ quy trình giải quyết phụ thuộc vòng như sau:

- Khi Spring tạo A xong, phát hiện A phụ thuộc vào B, lại đi tạo B; B phụ thuộc vào A, lại đi tạo A;
- Khi B tạo A, lúc này A xảy ra phụ thuộc vòng, vì A lúc này vẫn chưa khởi tạo hoàn tất, nên trong **Cache cấp 1 và cấp 2** chắc chắn không có A;
- Lúc này sẽ đến cache cấp 3 gọi method `getObject()` để lấy **object bộc lộ sớm** của A, tức là gọi method `getEarlyBeanReference()` vừa thêm ở trên, sinh ra một **object bộc lộ sớm** của A;
- Sau đó xóa `ObjectFactory` này khỏi cache cấp 3, và đưa object bộc lộ sớm vào cache cấp 2, như vậy B sẽ inject object bộc lộ sớm này vào dependency, để hỗ trợ phụ thuộc vòng.

**Chỉ dùng bộ nhớ đệm 2 tầng có đủ không?** Trong trường hợp không có AOP, quả thực có thể chỉ cần sử dụng cache cấp 1 và cấp 2 để giải quyết vấn đề phụ thuộc vòng. Tuy nhiên, khi liên quan đến AOP, cache cấp 3 tỏ ra vô cùng quan trọng, vì nó đảm bảo rằng ngay cả khi trong quá trình tạo Bean có nhiều lần yêu cầu tham chiếu sớm, thì vẫn luôn chỉ trả về cùng một object proxy, từ đó tránh được vấn đề một Bean có nhiều object proxy.

**Cuối cùng tóm tắt lại cách Spring giải quyết bằng bộ nhớ đệm 3 tầng**:

Ở phần bộ nhớ đệm 3 tầng này, chủ yếu nhớ cách Spring hỗ trợ phụ thuộc vòng như thế nào là được, tức là nếu xảy ra phụ thuộc vòng, sẽ đến **Cache cấp 3 `singletonFactories`** lấy `ObjectFactory` được lưu trữ trong cache cấp 3 và gọi method `getObject()` của nó để lấy object bộc lộ sớm của đối tượng phụ thuộc vòng này (tuy chưa khởi tạo hoàn tất, nhưng có thể lấy được địa chỉ lưu trữ của object đó trên Heap), và đưa object bộc lộ sớm này vào cache cấp 2, như vậy khi phụ thuộc vòng sẽ không bị khởi tạo lặp lại nữa!

Tuy nhiên, cơ chế này cũng có một số nhược điểm, ví dụ tăng chi phí bộ nhớ (cần duy trì bộ nhớ đệm 3 tầng, tức là 3 Map), giảm hiệu năng (cần tiến hành nhiều lần kiểm tra và chuyển đổi). Nó chỉ áp dụng cho phụ thuộc vòng của Setter/field injection của một số singleton Bean, còn non-singleton Bean, phụ thuộc vòng trong constructor,... vẫn không thể giải quyết thông qua bộ nhớ đệm 3 tầng.

### @Lazy có thể giải quyết phụ thuộc vòng không?

`@Lazy` dùng để đánh dấu class có cần lazy loading (tải chậm / trì hoãn) hay không, có thể tác động lên class, method, constructor, parameter của method, biến thành viên.

Spring Boot 2.2 đã bổ sung **thuộc tính lazy loading toàn cục**, sau khi bật thì các bean toàn cục được thiết lập là lazy loading, khi cần mới tạo.

Cấu hình file configuration cho lazy loading toàn cục:

```properties
# Mặc định false
spring.main.lazy-initialization=true
```

Cấu hình bằng code cho lazy loading toàn cục:

```java
SpringApplication springApplication=new SpringApplication(Start.class);
springApplication.setLazyInitialization(true);
springApplication.run(args);
```

Nếu không thực sự cần thiết, cố gắng không dùng lazy loading toàn cục. Lazy loading toàn cục sẽ khiến Bean lần đầu tiên sử dụng bị load chậm hơn, và nó sẽ trì hoãn việc phát hiện vấn đề của ứng dụng (chỉ khi Bean được khởi tạo thì vấn đề mới xuất hiện).

Nếu một Bean không được đánh dấu là lazy loading, thì nó sẽ được tạo và khởi tạo trong quá trình Spring IoC container khởi động. Nếu một Bean được đánh dấu là lazy loading, thì nó sẽ không được khởi tạo instance ngay khi Spring IoC container khởi động, mà khi được request lần đầu tiên mới tạo. Điều này có thể giúp giảm thời gian khởi tạo khi ứng dụng khởi động, cũng có thể dùng để giải quyết vấn đề phụ thuộc vòng.

Vấn đề phụ thuộc vòng được giải quyết thông qua `@Lazy` như thế nào? Ở đây lấy một ví dụ, giả sử có hai Bean, A và B, giữa chúng xảy ra phụ thuộc vòng, có thể thêm `@Lazy` vào vị trí inject B của A, ví dụ tham số constructor `A(@Lazy B b)`. Lúc này việc trì hoãn giải mã là dependency B, chứ không phải đơn giản là đánh dấu `@Lazy` lên constructor hoặc type của A.

- Đầu tiên Spring sẽ đi tạo Bean A, khi tạo cần inject thuộc tính B;
- Vì tại vị trí inject B của A có đánh dấu `@Lazy`, nên Spring sẽ tạo một object proxy giải mã trì hoãn của B, và inject object proxy đó vào A;
- Sau đó bắt đầu thực thi khởi tạo instance, khởi tạo B, khi inject thuộc tính A trong B, lúc này A đã tạo xong rồi, có thể inject A vào.

Từ quy trình loading ở trên có thể thấy: Điểm mấu chốt của `@Lazy` trong việc giải quyết phụ thuộc vòng nằm ở việc sử dụng object proxy.

- **Trường hợp không có `@Lazy`**: Khi Spring container khởi tạo `A` sẽ ngay lập tức thử tạo `B`, mà trong quá trình tạo `B` lại thử tạo `A`, cuối cùng dẫn đến phụ thuộc vòng (tức đệ quy vô tận, cuối cùng ném ra exception).
- **Trường hợp sử dụng `@Lazy`**: Spring không lập tức tạo `B`, mà sẽ inject một object proxy của `B`. Vì lúc này `B` vẫn chưa thực sự được khởi tạo, nên việc khởi tạo `A` có thể hoàn thành thuận lợi. Đến khi instance `A` thực tế gọi method của `B`, object proxy mới kích hoạt việc khởi tạo thực sự của `B`.

Proxy tại vị trí inject của `@Lazy` có thể phá vỡ chuỗi phụ thuộc vòng ở một mức độ nhất định, bao gồm cả một số kịch bản constructor injection. Nhưng đây không phải là xóa bỏ phụ thuộc vòng từ mặt thiết kế, trong quan hệ phụ thuộc phức tạp cũng có thể sinh ra các vấn đề khởi tạo tiềm ẩn hơn, do đó thực hành tốt nhất vẫn là cố gắng tránh phụ thuộc vòng từ mặt thiết kế.

### SpringBoot có cho phép phụ thuộc vòng xảy ra không?

Trước SpringBoot 2.6.x, mặc định cho phép phụ thuộc vòng, nghĩa là code của bạn xuất hiện vấn đề phụ thuộc vòng thì trong trường hợp thông thường cũng không báo lỗi. Từ SpringBoot 2.6.x trở đi, chính thức không còn khuyến khích viết code tồn tại phụ thuộc vòng, đề xuất lập trình viên tự viết code nên giảm bớt các phụ thuộc lẫn nhau không cần thiết. Đây thực ra cũng là điều chúng ta nên làm nhất, bản thân phụ thuộc vòng đã là một khuyết điểm về thiết kế, chúng ta không nên quá phụ thuộc vào Spring mà bỏ qua quy chuẩn và chất lượng viết code, biết đâu trong một phiên bản SpringBoot tương lai sẽ hoàn toàn cấm code phụ thuộc vòng.

Sau SpringBoot 2.6.x, nếu bạn không muốn refactor code phụ thuộc vòng, cũng có thể áp dụng các phương pháp dưới đây:

- Trong file cấu hình toàn cục thiết lập cho phép phụ thuộc vòng tồn tại: `spring.main.allow-circular-references=true`. Cách đơn giản thô bạo nhất, không quá khuyến khích.
- Thêm annotation `@Lazy` trên Bean dẫn đến phụ thuộc vòng, đây là một cách tương đối khuyến khích. `@Lazy` dùng để đánh dấu class có cần lazy loading / trì hoãn loading hay không, có thể tác động lên class, method, constructor, parameter của method, biến thành viên.
- ……

## ⭐️Spring Transaction

Về phần giới thiệu chi tiết Spring Transaction, có thể xem bài viết [Giải thích chi tiết Spring Transaction](https://javaguide.cn/system-design/framework/spring/spring-transaction.html) do tôi viết.

### Có mấy cách quản lý transaction trong Spring?

- **Transaction lập trình (Programmatic Transaction)**: Hardcode trong mã nguồn (khuyên dùng trong hệ thống phân tán): Quản lý transaction thủ công thông qua `TransactionTemplate` hoặc `TransactionManager`, phạm vi transaction quá lớn sẽ xuất hiện việc transaction chưa commit dẫn đến timeout, do đó transaction phải có độ mịn (granularity) nhỏ hơn khóa (lock).
- **Transaction khai báo (Declarative Transaction)**: Cấu hình trong file XML hoặc dựa trực tiếp trên annotation (khuyên dùng cho ứng dụng đơn khối (monolith) hoặc hệ thống nghiệp vụ đơn giản): Thực tế được thực hiện thông qua AOP (cách dùng hoàn toàn bằng annotation dựa trên `@Transactional` được sử dụng nhiều nhất).

### Có những hành vi lan truyền transaction (Propagation) nào trong Spring Transaction?

**Hành vi lan truyền transaction là để giải quyết vấn đề transaction giữa các method thuộc tầng nghiệp vụ gọi lẫn nhau**.

Khi một method transaction được gọi bởi một method transaction khác, phải chỉ định transaction nên lan truyền như thế nào. Ví dụ: Method có thể tiếp tục chạy trong transaction hiện có, hoặc cũng có thể mở một transaction mới, và chạy trong transaction của chính nó.

Các giá trị hành vi lan truyền transaction đúng như sau:

**1. `TransactionDefinition.PROPAGATION_REQUIRED`**

Hành vi lan truyền transaction được sử dụng nhiều nhất, annotation `@Transactional` mà chúng ta thường dùng mặc định sử dụng hành vi lan truyền transaction này. Nếu hiện tại đã có transaction, thì tham gia vào transaction đó; nếu hiện tại không có transaction, thì tạo một transaction mới.

**2. `TransactionDefinition.PROPAGATION_REQUIRES_NEW`**

Tạo một transaction mới, nếu hiện tại đã có transaction, thì hoãn (suspend) transaction hiện tại. Nghĩa là bất kể method bên ngoài có mở transaction hay không, method bên trong được trang bị `Propagation.REQUIRES_NEW` sẽ mở transaction riêng của mình, và các transaction được mở độc lập với nhau, không can thiệp lẫn nhau.

**3. `TransactionDefinition.PROPAGATION_NESTED`**

Nếu hiện tại đã có transaction, thì tạo một transaction làm transaction lồng nhau (nested transaction) của transaction hiện tại để chạy; nếu hiện tại không có transaction, thì giá trị này tương đương với `TransactionDefinition.PROPAGATION_REQUIRED`.

**4. `TransactionDefinition.PROPAGATION_MANDATORY`**

Nếu hiện tại đã có transaction, thì tham gia vào transaction đó; nếu hiện tại không có transaction, thì ném ra exception. (mandatory: bắt buộc)

Cái này rất ít khi sử dụng.

Ngoài ra 3 loại hành vi lan truyền transaction khác cũng là cấu hình hợp lệ, cần hiểu dựa trên việc có tồn tại transaction bên ngoài hay không:

- **`TransactionDefinition.PROPAGATION_SUPPORTS`**: Nếu hiện tại đã có transaction, thì tham gia vào transaction đó; nếu hiện tại không có transaction, thì tiếp tục chạy theo cách không có transaction.
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**: Chạy theo cách không có transaction, nếu hiện tại đã có transaction, thì hoãn transaction hiện tại.
- **`TransactionDefinition.PROPAGATION_NEVER`**: Chạy theo cách không có transaction, nếu hiện tại đã có transaction, thì ném ra exception.

### Có những mức độ cô lập (Isolation Level) nào trong Spring Transaction?

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

### Bạn có hiểu annotation @Transactional(rollbackFor = Exception.class) không?

`Exception` chia thành exception runtime (`RuntimeException`) và exception non-runtime. Quản lý transaction đối với ứng dụng doanh nghiệp là cực kỳ quan trọng, ngay cả khi xuất hiện trường hợp exception, nó cũng có thể đảm bảo tính nhất quán của dữ liệu.

Khi annotation `@Transactional` tác động lên class, tất cả các method public của class đó sẽ có thuộc tính transaction của loại đó, đồng thời chúng ta cũng có thể sử dụng annotation này ở cấp độ method để ghi đè (override) định nghĩa ở cấp độ class.

Mặc định chiến lược rollback của annotation `@Transactional` là chỉ khi gặp `RuntimeException` (exception runtime) hoặc `Error` mới rollback transaction, mà không rollback `Checked Exception` (exception kiểm tra). Đó là vì Spring cho rằng `RuntimeException` và Error là các lỗi không thể dự đoán trước, còn Checked Exception là lỗi có thể dự đoán trước, có thể xử lý thông qua business logic.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/spring-transactional-rollbackfor.png)

Nếu muốn sửa đổi chiến lược rollback mặc định, có thể sử dụng thuộc tính `rollbackFor` và `noRollbackFor` của annotation `@Transactional` để chỉ định những exception nào cần rollback, những exception nào không cần rollback. Ví dụ, nếu muốn cho tất cả các exception đều rollback transaction, có thể sử dụng annotation như sau:

```java
@Transactional(rollbackFor = Exception.class)
public void someMethod() {
// some business logic
}
```

Nếu muốn cho một số exception cụ thể không rollback transaction, có thể sử dụng annotation như sau:

```java
@Transactional(noRollbackFor = CustomException.class)
public void someMethod() {
// some business logic
}
```

## Spring Data JPA

JPA quan trọng ở thực chiến, ở đây chỉ tổng kết một phần nhỏ điểm kiến thức.

### Làm thế nào để dùng JPA không lưu trữ (non-persistent) một field vào cơ sở dữ liệu?

Giả sử chúng ta có một class như dưới đây:

```java
@Entity(name="USER")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "ID")
    private Long id;

    @Column(name="USER_NAME")
    private String userName;

    @Column(name="PASSWORD")
    private String password;

    private String secrect;

}
```

Nếu chúng ta muốn field `secrect` không được lưu trữ (persistent), tức không được cơ sở dữ liệu lưu lại thì làm thế nào? Chúng ta có thể áp dụng một số cách dưới đây:

```java
static String transient1; // not persistent because of static
final String transient2 = "Satish"; // not persistent because of final
transient String transient3; // not persistent because of transient
@Transient
String transient4; // not persistent because of @Transient
```

Thông thường sử dụng hai cách sau nhiều hơn, cá nhân tôi sử dụng cách annotation nhiều hơn.

### Tính năng Audit của JPA làm gì? Có tác dụng gì?

Tính năng Audit (kiểm toán) chủ yếu giúp chúng ta ghi lại hành vi cụ thể của thao tác cơ sở dữ liệu, ví dụ một record nào đó do ai tạo, tạo lúc mấy giờ, người sửa đổi cuối cùng là ai, thời gian sửa đổi cuối cùng là khi nào.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@MappedSuperclass
@EntityListeners(value = AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate
    @Column(updatable = false)
    @JsonIgnore
    private Instant createdAt;

    @LastModifiedDate
    @JsonIgnore
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    @JsonIgnore
    private String createdBy;

    @LastModifiedBy
    @JsonIgnore
    private String updatedBy;
}
```

- `@CreatedDate`: Thể hiện field này là field thời gian tạo, khi entity này được insert, sẽ thiết lập giá trị

- `@CreatedBy`: Thể hiện field này là người tạo, khi entity này được insert, sẽ thiết lập giá trị

  `@LastModifiedDate`, `@LastModifiedBy` tương tự.

### Các annotation quan hệ liên kết giữa các entity là gì?

- `@OneToOne`: Một - Một.
- `@ManyToMany`: Nhiều - Nhiều.
- `@OneToMany`: Một - Nhiều.
- `@ManyToOne`: Nhiều - Một.

Sử dụng `@ManyToOne` và `@OneToMany` cũng có thể thể hiện quan hệ liên kết Nhiều - Nhiều.

## Spring Security

Spring Security quan trọng ở thực chiến, ở đây chỉ tổng kết một phần nhỏ điểm kiến thức.

### Có những method nào kiểm soát quyền truy cập request?

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/image-20220728201854641.png)

- `permitAll()`: Cho phép truy cập vô điều kiện bất kỳ hình thức nào, bất kể bạn đã đăng nhập hay chưa đăng nhập.
- `anonymous()`: Cho phép truy cập ẩn danh, tức là chưa đăng nhập mới có thể truy cập.
- `denyAll()`: Từ chối vô điều kiện bất kỳ hình thức truy cập nào.
- `authenticated()`: Chỉ cho phép user đã xác thực (xác thực thành công) truy cập.
- `fullyAuthenticated()`: Chỉ cho phép user xác thực đầy đủ truy cập, không chấp nhận xác thực ẩn danh hoặc xác thực remember-me.
- `hasRole(String)`: Chỉ cho phép role được chỉ định truy cập.
- `hasAnyRole(String)`: Chỉ định một hoặc nhiều role, user thỏa mãn một trong số đó là có thể truy cập.
- `hasAuthority(String)`: Chỉ cho phép user có quyền hạn (authority) được chỉ định truy cập
- `hasAnyAuthority(String)`: Chỉ định một hoặc nhiều quyền hạn, user thỏa mãn một trong số đó là có thể truy cập.
- `hasIpAddress(String)`: Chỉ cho phép user có IP được chỉ định truy cập.

### hasRole và hasAuthority có khác nhau không?

Có thể đọc bài viết này của 松哥 (Tùng Ca): [Spring Security 中的 hasRole 和 hasAuthority 有区别吗？](https://mp.weixin.qq.com/s/GTNOa2k9_n_H0w24upClRw), giới thiệu khá chi tiết.

### ⭐️Mã hóa mật khẩu như thế nào?

Nếu chúng ta cần lưu các dữ liệu nhạy cảm như mật khẩu vào cơ sở dữ liệu, cần phải thông qua hàm băm một chiều tự thích ứng (adaptive one-way hash function) để mã hóa trước rồi mới lưu, chứ không sử dụng mã hóa có thể đảo ngược.

Spring Security cung cấp triển khai nhiều thuật toán mã hóa mật khẩu, mở hộp dùng ngay. Interface của các class triển khai này là `PasswordEncoder`; nếu cần tùy chỉnh phương án mã hóa mật khẩu, cũng cần implement interface `PasswordEncoder`.

Interface `PasswordEncoder` có hai method abstract bắt buộc phải implement là `encode()` và `matches()`, cùng với một method mặc định `upgradeEncoding()` có thể override khi cần.

```java
public interface PasswordEncoder {
    // Mã hóa một chiều mật khẩu nguyên bản
    String encode(CharSequence var1);
    // So sánh mật khẩu nguyên bản và mật khẩu lưu trong cơ sở dữ liệu
    boolean matches(CharSequence var1, String var2);
    // Phán đoán mật khẩu đã mã hóa có cần nâng cấp mã hóa hay không, mặc định trả về false
    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/image-20220728183540954.png)

Chính thức khuyến nghị sử dụng hàm một chiều tự thích ứng có thể điều chỉnh hệ số công việc (work factor), và tinh chỉnh thời gian thực thi dựa trên hiệu năng hệ thống, ví dụ bcrypt, PBKDF2, scrypt hoặc Argon2.

### Thay đổi thuật toán mã hóa hệ thống đang sử dụng một cách thanh lịch như thế nào?

Nếu trong quá trình phát triển, chúng ta đột nhiên phát hiện thuật toán mã hóa hiện tại không thể đáp ứng nhu cầu của chúng ta, cần thay đổi sang một thuật toán mã hóa khác, lúc này nên làm thế nào?

Cách làm khuyên dùng là thông qua `DelegatingPasswordEncoder` tương thích với nhiều phương án mã hóa mật khẩu khác nhau, để thích ứng với các nhu cầu nghiệp vụ khác nhau.

Từ tên cũng có thể thấy, `DelegatingPasswordEncoder` thực ra là một class proxy, chứ không phải là một thuật toán mã hóa hoàn toàn mới, việc nó làm là proxy cho các class triển khai thuật toán mã hóa được đề cập ở trên. Từ Spring Security 5.0 trở đi, mặc định dựa trên `DelegatingPasswordEncoder` để tiến hành mã hóa mật khẩu.

## Tham khảo

- 《Thiết kế bên trong kỹ thuật Spring (Spring Technical Inner Principles)》
- 《Học sâu Spring từ con số 0》: <https://juejin.cn/book/6857911863016390663>
- <http://www.cnblogs.com/wmyskxz/p/8820371.html>
- <https://www.journaldev.com/2696/spring-interview-questions-and-answers>
- <https://www.edureka.co/blog/interview-questions/spring-interview-questions/>
- <https://www.cnblogs.com/clwydjgs/p/9317849.html>
- <https://howtodoinjava.com/interview-questions/top-spring-interview-questions-with-answers/>
- <http://www.tomaszezula.com/2014/02/09/spring-series-part-5-component-vs-bean/>
- <https://stackoverflow.com/questions/34172888/difference-between-bean-and-autowired>

<!-- @include: @article-footer.snippet.md -->
