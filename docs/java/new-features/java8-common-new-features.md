---
title: Java8 新特性实战
description: 实战讲解 Java 8 的核心新特性，包括 Lambda、Stream、Optional、日期时间 API 与接口默认方法等。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 8,Lambda,Stream API,Optional,Date/Time API,默认方法,函数式接口
---

> Bài viết này đến từ đóng góp của [cowbi](https://github.com/cowbi)~

<!-- markdownlint-disable MD024 -->

JDK 8 được phát hành vào ngày 18 tháng 3 năm 2014, đây là một phiên bản LTS (Long Term Support - Hỗ trợ dài hạn), cũng là một trong những phiên bản được sử dụng rộng rãi và lâu dài nhất trong hệ sinh thái Java. Các phiên bản LTS hiện tại do Oracle niêm yết bao gồm JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

JDK 8 đã giới thiệu nhiều tính năng mới quan trọng. Bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- Biểu thức Lambda (Lambda expression)
- Stream API
- Lớp Optional
- Date-Time API
- Phương thức mặc định trong Interface (Default methods)
- Functional interface (Interface chức năng)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 24:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Oracle đã phát hành Java 8 (JDK 1.8) vào năm 2014, và kể từ đó nó đã được sử dụng rộng rãi trong thời gian dài trong hệ sinh thái Java. Nhiều lập trình viên vẫn chưa hiểu rõ về một số tính năng mới của nó, đặc biệt là các nhà phát triển đã quen với các phiên bản trước Java 8, ví dụ như tôi.

Để không bị tụt lại quá xa so với đội ngũ, việc tổng hợp lại các tính năng mới này là rất cần thiết. So với JDK 7, nó có nhiều thay đổi hoặc tối ưu hóa, ví dụ trong `interface` có thể có phương thức `static` và có thể chứa thân phương thức (body), điều này làm đảo lộn nhận thức trước đây; cấu trúc dữ liệu `java.util.HashMap` đã bổ sung thêm cây đỏ đen (Red-Black Tree); cùng với biểu thức Lambda nổi tiếng,... Bài viết này không thể chia sẻ tất cả các tính năng mới với mọi người, mà chỉ liệt kê các tính năng mới thường dùng để giải thích chi tiết. Để biết thêm nội dung liên quan, vui lòng xem [Giới thiệu tính năng mới của Java 8 trên trang chủ](https://www.oracle.com/java/technologies/javase/8-whats-new.html).

## Interface

Mục đích thiết kế ban đầu của `interface` là hướng tới sự trừu tượng, nâng cao tính mở rộng. Điều này cũng để lại một chút đáng tiếc: khi `interface` bị sửa đổi, các lớp triển khai (implement) nó cũng bắt buộc phải sửa đổi theo.

Để giải quyết vấn đề không tương thích giữa việc sửa đổi interface và các triển khai hiện có, phương thức trong `interface` mới có thể sử dụng từ khóa `default` hoặc `static` để bổ sung, từ đó có thể chứa thân phương thức, và các lớp triển khai không nhất thiết phải ghi đè (`@Override`) phương thức này.

Trong một `interface` có thể có nhiều phương thức được bổ sung bởi các từ khóa này. Sự khác biệt giữa 2 từ khóa này chủ yếu cũng là sự khác biệt giữa phương thức thông thường và phương thức `static`.

1. Phương thức được bổ sung từ khóa `default` là phương thức thể hiện (instance method) thông thường, có thể dùng `this` để gọi, có thể được lớp con kế thừa và ghi đè.
2. Phương thức được bổ sung từ khóa `static`, về cách sử dụng thì giống như phương thức `static` của lớp thông thường. Nhưng nó không thể được lớp con kế thừa, mà chỉ có thể gọi bằng `Interface`.

Chúng ta hãy xem một ví dụ thực tế.

```java
public interface InterfaceNew {
    static void sm() {
        System.out.println("interface提供的方式实现");
    }
    static void sm2() {
        System.out.println("interface提供的方式实现");
    }

    default void def() {
        System.out.println("interface default方法");
    }
    default void def2() {
        System.out.println("interface default2方法");
    }
    //须要实现类重写
    void f();
}

public interface InterfaceNew1 {
    default void def() {
        System.out.println("InterfaceNew1 default方法");
    }
}
```

Nếu có một lớp vừa triển khai interface `InterfaceNew` vừa triển khai interface `InterfaceNew1`, cả hai đều có `def()`, và interface `InterfaceNew` cũng như `InterfaceNew1` không có mối quan hệ kế thừa với nhau, lúc này bắt buộc phải ghi đè `def()`. Nếu không, khi biên dịch sẽ báo lỗi.

```java
public class InterfaceNewImpl implements InterfaceNew , InterfaceNew1{
    public static void main(String[] args) {
        InterfaceNewImpl interfaceNew = new InterfaceNewImpl();
        interfaceNew.def();
    }

    @Override
    public void def() {
        InterfaceNew1.super.def();
    }

    @Override
    public void f() {
    }
}
```

**Trong Java 8, interface và abstract class có điểm gì khác nhau?**

Nhiều bạn cho rằng: "Vì interface cũng có thể có triển khai phương thức riêng, dường như không có nhiều khác biệt với abstract class."

Thực ra chúng vẫn có sự khác biệt:

1. Sự khác biệt giữa interface và class, dường như là điều hiển nhiên, chủ yếu bao gồm:

   - Interface hỗ trợ đa triển khai (multi-implementation), Class chỉ hỗ trợ đơn kế thừa (single-inheritance)
   - Các phương thức thể hiện không có thân phương thức trong interface mặc định định nghĩa ẩn là `public abstract`, các trường (field) mặc định định nghĩa ẩn là `public static final`; ngoài ra, interface còn có thể khai báo các phương thức `default`, `static`,... Các thành viên của abstract class có thể sử dụng nhiều từ khóa truy cập hơn
2. Phương thức của interface giống như một plugin mở rộng. Trong khi phương thức của abstract class là để kế thừa.

Như đã đề cập ở phần đầu, các phương thức được bổ sung `default` và `static` trong interface nhằm giải quyết vấn đề không tương thích giữa việc sửa đổi interface và triển khai hiện có, chứ không phải để thay thế `abstract class`. Về mặt sử dụng, những nơi cần dùng abstract class vẫn phải dùng abstract class, đừng vì tính năng mới của interface mà thay thế nó.

**Hãy nhớ rằng interface mãi mãi không giống như class.**

## Functional Interface (Interface Chức Năng)

**Định nghĩa**: Còn gọi là interface SAM, tức Single Abstract Method interfaces, có và chỉ có một phương thức trừu tượng, nhưng có thể có nhiều phương thức không trừu tượng.

Trong Java 8 có một package chuyên dụng chứa các functional interface là `java.util.function`. Tất cả các interface thuộc package này đều có annotation `@FunctionalInterface`, hỗ trợ lập trình hàm (functional programming).

Trong các package khác cũng có các functional interface, một số trong đó không có annotation `@FunctionalInterface`, nhưng chỉ cần phù hợp với định nghĩa của functional interface thì đó là functional interface, không liên quan đến việc có annotation `@FunctionalInterface` hay không. Annotation chỉ đóng vai trò ép buộc quy chuẩn định nghĩa khi biên dịch. Nó được ứng dụng rộng rãi trong biểu thức Lambda.

## Biểu thức Lambda (Lambda Expression)

Tiếp theo là thảo luận về biểu thức Lambda nổi tiếng. Đây là tính năng mới quan trọng nhất thúc đẩy việc phát hành Java 8, là thay đổi lớn nhất kể từ Generics và Annotation.

Sử dụng biểu thức Lambda có thể làm cho mã nguồn trở nên gọn gàng và súc tích hơn, giúp Java hỗ trợ lập trình hàm cơ bản.

> Biểu thức Lambda là một hàm ẩn danh (anonymous function), Java 8 cho phép truyền hàm làm tham số vào phương thức.

### Cú pháp

```java
(parameters) -> expression hoặc
(parameters) ->{ statements; }
```

### Thực hành Lambda

Chúng ta sử dụng các ví dụ thường gặp để cảm nhận sự tiện lợi do Lambda mang lại.

#### Thay thế Inner Class ẩn danh (Anonymous Inner Class)

Trước đây, cách duy nhất để truyền tham số động vào phương thức là sử dụng inner class. Ví dụ:

**1.`Runnable` Interface**

```java
new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("The runable now is using!");
            }
}).start();
//Dùng lambda
new Thread(() -> System.out.println("It's a lambda function!")).start();
```

**2.`Comparator` Interface**

```java
List<Integer> strings = Arrays.asList(1, 2, 3);

Collections.sort(strings, new Comparator<Integer>() {
@Override
public int compare(Integer o1, Integer o2) {
    return Integer.compare(o1, o2);}
});

//Lambda
Collections.sort(strings, (Integer o1, Integer o2) -> Integer.compare(o1, o2));
//Phân tách ra
Comparator<Integer> comparator = (Integer o1, Integer o2) -> Integer.compare(o1, o2);
Collections.sort(strings, comparator);
```

**3.`Listener` Interface**

```java
JButton button = new JButton();
button.addItemListener(new ItemListener() {
@Override
public void itemStateChanged(ItemEvent e) {
   e.getItem();
}
});
//lambda
button.addItemListener(e -> e.getItem());
```

**4. Interface tự định nghĩa**

3 ví dụ trên là những ví dụ thường gặp nhất trong quá trình phát triển của chúng ta, từ đó cũng có thể cảm nhận được sự tiện lợi và thanh thoát mà Lambda mang lại. Nó chỉ giữ lại đoạn mã thực sự được sử dụng và bỏ qua tất cả mã thừa. Vậy nó có yêu cầu gì đối với interface không? Chúng ta thấy rằng các anonymous inner class này chỉ ghi đè một phương thức của interface, và dĩ nhiên chỉ có một phương thức cần ghi đè. Đó chính là **functional interface** mà chúng ta đã đề cập ở trên, nghĩa là chỉ cần tham số của phương thức là functional interface thì đều có thể sử dụng biểu thức Lambda.

```java
@FunctionalInterface
public interface Comparator<T>{
    int compare(T o1, T o2);
}

@FunctionalInterface
public interface Runnable{
    void run();
}
```

Chúng ta tự định nghĩa một functional interface:

```java
@FunctionalInterface
public interface LambdaInterface {
 void f();
}
//Sử dụng
public class LambdaClass {
    public static void forEg() {
        lambdaInterfaceDemo(()-> System.out.println("自定义函数式接口"));
    }
    //Tham số functional interface
    static void lambdaInterfaceDemo(LambdaInterface i){
        i.f();
    }
}
```

#### Duyệt Tập Hợp (Iteration)

```java
void lamndaFor() {
        List<String> strings = Arrays.asList("1", "2", "3");
        //foreach truyền thống
        for (String s : strings) {
            System.out.println(s);
        }
        //Lambda foreach
        strings.forEach((s) -> System.out.println(s));
        //hoặc
        strings.forEach(System.out::println);
     //map
        Map<Integer, String> map = new HashMap<>();
        map.forEach((k,v)->System.out.println(v));
}
```

#### Tham Chiếu Phương Thức (Method Reference)

Java 8 cho phép sử dụng từ khóa `::` để truyền tham chiếu phương thức hoặc constructor. Dù thế nào đi nữa, kiểu trả về của biểu thức phải là functional-interface.

```java
public class LambdaClassSuper {
    LambdaInterface sf(){
        return null;
    }
}

public class LambdaClass extends LambdaClassSuper {
    public static LambdaInterface staticF() {
        return null;
    }

    public LambdaInterface f() {
        return null;
    }

    void show() {
        //1. Gọi phương thức static, kiểu trả về bắt buộc phải là functional-interface
        LambdaInterface t = LambdaClass::staticF;

        //2. Gọi phương thức thể hiện (instance method)
        LambdaClass lambdaClass = new LambdaClass();
        LambdaInterface lambdaInterface = lambdaClass::f;

        //3. Gọi phương thức trên lớp cha (superclass)
        LambdaInterface superf = super::sf;

        //4. Gọi constructor
        LambdaInterface tt = LambdaClassSuper::new;
    }
}
```

#### Truy Cập Biến (Accessing Variables)

```java
int i = 0;
Collections.sort(strings, (Integer o1, Integer o2) -> o1 - i);
//i =3;
```

Biểu thức Lambda có thể tham chiếu đến biến cục bộ bên ngoài, nhưng biến đó bắt buộc phải là `final` hoặc effectively final (không bị gán lại giá trị sau khi khởi tạo). Trình biên dịch sẽ không tự động thêm từ khóa `final` cho biến.

## Stream

Java đã bổ sung thêm package `java.util.stream`, nó tương tự như các stream trước đây. Trước đây chúng ta tiếp xúc nhiều nhất là stream tài nguyên, ví dụ `java.io.FileInputStream`, nhập tệp từ nơi này sang nơi khác thông qua stream, nó chỉ là người vận chuyển nội dung, không thực hiện bất kỳ thao tác *CRUD* nào đối với nội dung tệp.

`Stream` vẫn không lưu trữ dữ liệu, điểm khác biệt là nó có thể truy xuất (Retrieve) và xử lý logic dữ liệu tập hợp, bao gồm lọc, sắp xếp, thống kê, đếm,... Có thể tưởng tượng nó tương tự như câu lệnh SQL.

Nguồn dữ liệu của nó có thể là `Collection`, `Array`,... Vì các tham số phương thức của nó đều thuộc kiểu functional interface, nên nó thường được kết hợp sử dụng với Lambda.

### Các loại Stream

1. stream - Luồng nối tiếp (Serial stream)
2. parallelStream - Luồng song song (Parallel stream), có thể thực thi đa luồng (multi-threaded)

### Các phương thức thường dùng

Tiếp theo chúng ta xem các phương thức thường dùng trong `java.util.stream.Stream`:

```java
/**
* 返回一个串行流
*/
default Stream<E> stream()

/**
* 返回一个并行流
*/
default Stream<E> parallelStream()

/**
* 返回T的流
*/
public static<T> Stream<T> of(T t)

/**
* 返回其元素是指定值的顺序流。
*/
public static<T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}


/**
* 过滤，返回由与给定predicate匹配的该流的元素组成的流
*/
Stream<T> filter(Predicate<? super T> predicate);

/**
* 此流的所有元素是否与提供的predicate匹配。
*/
boolean allMatch(Predicate<? super T> predicate)

/**
* 此流任意元素是否有与提供的predicate匹配。
*/
boolean anyMatch(Predicate<? super T> predicate);

/**
* 返回一个 Stream的构建器。
*/
public static<T> Builder<T> builder();

/**
* 使用 Collector对此流的元素进行归纳
*/
<R, A> R collect(Collector<? super T, A, R> collector);

/**
 * 返回此流中的元素数。
*/
long count();

/**
* 返回由该流的不同元素（根据 Object.equals(Object) ）组成的流。
*/
Stream<T> distinct();

/**
 * 遍历
*/
void forEach(Consumer<? super T> action);

/**
* 用于获取指定数量的流，截短长度不能超过 maxSize 。
*/
Stream<T> limit(long maxSize);

/**
* 用于映射每个元素到对应的结果
*/
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

/**
* 根据提供的 Comparator进行排序。
*/
Stream<T> sorted(Comparator<? super T> comparator);

/**
* 丢弃此流中的前 n 个元素，返回由剩余元素组成的新流。
*/
Stream<T> skip(long n);

/**
* 返回一个包含此流的元素的数组。
*/
Object[] toArray();

/**
* 使用提供的 generator函数返回一个包含此流的元素的数组，以分配返回的数组，以及分区执行或调整大小可能需要的任何其他数组。
*/
<A> A[] toArray(IntFunction<A[]> generator);

/**
* 合并流
*/
public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
```

### Thực hành

Bài viết này liệt kê cách sử dụng các phương thức mang tính đại diện của `Stream`, để biết thêm nhiều cách sử dụng khác bạn nên tham khảo API.

```java
@Test
public void test() {
  List<String> strings = Arrays.asList("abc", "def", "gkh", "abc");
    //返回符合条件的stream
    Stream<String> stringStream = strings.stream().filter(s -> "abc".equals(s));
    //计算流符合条件的流的数量
    long count = stringStream.count();

    //forEach遍历->打印元素
    strings.stream().forEach(System.out::println);

    //limit 获取到1个元素的stream
    Stream<String> limit = strings.stream().limit(1);
    //toArray 比如我们想看这个limitStream里面是什么，比如转换成String[],比如循环
    String[] array = limit.toArray(String[]::new);

    //map 对每个元素进行操作返回新流
    Stream<String> map = strings.stream().map(s -> s + "22");

    //sorted 排序并打印
    strings.stream().sorted().forEach(System.out::println);

    //Collectors collect 把abc放入容器中
    List<String> collect = strings.stream().filter(string -> "abc".equals(string)).collect(Collectors.toList());
    //把list转为string，各元素用，号隔开
    String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(","));

    //对数组的统计，比如用
    List<Integer> number = Arrays.asList(1, 2, 5, 4);

    IntSummaryStatistics statistics = number.stream().mapToInt((x) -> x).summaryStatistics();
    System.out.println("列表中最大的数 : "+statistics.getMax());
    System.out.println("列表中最小的数 : "+statistics.getMin());
    System.out.println("平均数 : "+statistics.getAverage());
    System.out.println("所有数之和 : "+statistics.getSum());

    //concat 合并流
    List<String> strings2 = Arrays.asList("xyz", "jqx");
    Stream.concat(strings2.stream(),strings.stream()).count();

    //注意 一个Stream只能操作一次，不能断开，否则会报错。
    Stream stream = strings.stream();
    //第一次使用
    stream.limit(2);
    //第二次使用
    stream.forEach(System.out::println);
    //报错 java.lang.IllegalStateException: stream has already been operated upon or closed

    //可以在同一条流水线中连续调用
    strings.stream().limit(2).forEach(System.out::println);
}
```

### Thực thi trì hoãn (Lazy Execution)

Khi thực thi phương thức trả về `Stream`, nó không thực thi ngay lập tức, mà đợi đến khi một phương thức không trả về `Stream` được gọi mới thực thi. Bởi vì khi lấy được `Stream` thì không thể sử dụng trực tiếp, mà cần phải xử lý thành một kiểu dữ liệu thông thường. `Stream` ở đây có thể hình dung như một luồng nhị phân, lấy được cũng không đọc hiểu ngay được.

Dưới đây chúng ta hãy phân tách phương thức `filter`.

```java
@Test
public void laziness(){
  List<String> strings = Arrays.asList("abc", "def", "gkh", "abc");
  Stream<Integer> stream = strings.stream().filter(new Predicate() {
      @Override
      public boolean test(Object o) {
        System.out.println("Predicate.test 执行");
        return true;
        }
      });

   System.out.println("count 执行");
   stream.count();
}
/*-------执行结果--------*/
count 执行
Predicate.test 执行
Predicate.test 执行
Predicate.test 执行
Predicate.test 执行
```

Theo thứ tự thực thi thì lẽ ra phải in 4 lần 「`Predicate.test` 执行」 trước, rồi mới in 「`count` 执行」. Kết quả thực tế lại hoàn toàn ngược lại. Điều này cho thấy phương thức trong filter không thực thi ngay lập tức, mà đợi sau khi gọi phương thức `count()` mới thực thi.

Trên đây đều là các ví dụ về `Stream` nối tiếp. `parallelStream` song song có cách sử dụng tương tự như luồng nối tiếp. Khác biệt chính là `parallelStream` có thể thực thi đa luồng, được triển khai dựa trên framework ForkJoin. Khi có thời gian mọi người có thể tìm hiểu về framework `ForkJoin` và `ForkJoinPool`. Ở đây có thể hiểu đơn giản là nó được triển khai thông qua thread pool, từ đó sẽ liên quan đến các vấn đề an toàn luồng (thread safety), tiêu tốn tài nguyên luồng,... Dưới đây chúng ta qua mã nguồn để trải nghiệm việc thực thi đa luồng của luồng song song.

```java
@Test
public void parallelStreamTest(){
   List<Integer> numbers = Arrays.asList(1, 2, 5, 4);
   numbers.parallelStream() .forEach(num->System.out.println(Thread.currentThread().getName()+">>"+num));
}
//执行结果
main>>5
ForkJoinPool.commonPool-worker-2>>4
ForkJoinPool.commonPool-worker-11>>1
ForkJoinPool.commonPool-worker-9>>2
```

Từ kết quả chúng ta thấy, for-each đã sử dụng đa luồng.

### Tóm tắt nhỏ

Từ mã nguồn và các ví dụ, chúng ta có thể tóm tắt một số đặc điểm của stream:

1. Thông qua lập trình chuỗi (chaining) đơn giản, nó giúp thuận tiện trong việc xử lý lại dữ liệu sau khi đã duyệt qua.
2. Các tham số phương thức đều thuộc kiểu functional interface
3. Một Stream chỉ có thể thao tác một lần, sau khi thao tác xong sẽ đóng lại, tiếp tục sử dụng stream này sẽ báo lỗi.
4. Stream không lưu dữ liệu, không thay đổi nguồn dữ liệu

## Optional

Trong [Tài liệu hướng dẫn phát triển của Alibaba về giới thiệu Optional](https://share.weiyun.com/ThuqEbD5) có viết như sau:

> Phòng tránh NPE, là tu dưỡng cơ bản của lập trình viên, cần chú ý các kịch bản phát sinh NPE:
>
> 1） Khi kiểu trả về là kiểu dữ liệu cơ bản, return đối tượng thuộc kiểu dữ liệu đóng gói (wrapper class), việc tự động mở hộp (auto-unboxing) có khả năng tạo ra NPE.
>
> Ví dụ phản diện: public int f() { return đối tượng Integer; }, nếu là null, tự động mở hộp sẽ ném ra NPE.
>
> 2） Kết quả truy vấn cơ sở dữ liệu có thể là null.
>
> 3） Ngay cả khi phần tử trong collection isNotEmpty, phần tử dữ liệu lấy ra cũng có thể là null.
>
> 4） Khi gọi từ xa (RPC/API) trả về đối tượng, nhất thiết phải thực hiện kiểm tra null để phòng tránh NPE.
>
> 5） Đối với dữ liệu lấy từ Session, nên kiểm tra NPE để tránh con trỏ null.
>
> 6） Gọi liên hoàn obj.getA().getB().getC()；chuỗi gọi liên tiếp rất dễ tạo ra NPE.
>
> Ví dụ chính diện: Sử dụng lớp Optional của JDK8 để phòng tránh vấn đề NPE.

Ở đây khuyên bạn nên sử dụng `Optional` để biểu thị rõ ràng "có thể không có kết quả", nhằm giảm thiểu rủi ro NPE (`java.lang.NullPointerException`). Optional hoặc là chứa một giá trị không `null`, hoặc là rỗng, chứ không lưu trữ `null` bên trong. Dưới đây chúng ta qua mã nguồn để từng bước hé mở bức màn về `Optional`.

Giả sử có lớp `Zoo`, bên trong có thuộc tính `Dog`, yêu cầu lấy `age` của `Dog`.

```java
class Zoo {
   private Dog dog;
}

class Dog {
   private int age;
}
```

Cách giải quyết NPE truyền thống như sau:

```java
Zoo zoo = getZoo();
if(zoo != null){
   Dog dog = zoo.getDog();
   if(dog != null){
      int age = dog.getAge();
      System.out.println(age);
   }
}
```

Từng tầng kiểm tra đối tượng không null, có người nói cách này rất xấu và không thanh lịch, nhưng tôi không nghĩ vậy. Trái lại tôi thấy rất gọn gàng, dễ đọc, dễ hiểu. Bạn thấy sao?

`Optional` được triển khai như sau:

```java
Optional.ofNullable(zoo).map(o -> o.getDog()).map(d -> d.getAge()).ifPresent(age ->
    System.out.println(age)
);
```

Có phải đã gọn gàng hơn rất nhiều không?

### Làm thế nào để tạo một Optional

Trong ví dụ trên `Optional.ofNullable` là một trong những cách tạo Optional. Chúng ta xem ý nghĩa của nó và các phương thức mã nguồn tạo Optional khác.

```java
/**
* Common instance for {@code empty()}. 全局EMPTY对象
*/
private static final Optional<?> EMPTY = new Optional<>();

/**
* Optional维护的值
*/
private final T value;

/**
* 如果value是null就返回EMPTY，否则就返回of(T)
*/
public static <T> Optional<T> ofNullable(T value) {
   return value == null ? empty() : of(value);
}
/**
* 返回 EMPTY 对象
*/
public static<T> Optional<T> empty() {
   Optional<T> t = (Optional<T>) EMPTY;
   return t;
}
/**
* 返回Optional对象
*/
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}
/**
* 私有构造方法，给value赋值
*/
private Optional(T value) {
  this.value = Objects.requireNonNull(value);
}
/**
* 所以如果of(T value) 的value是null，会抛出NullPointerException异常，这样貌似就没处理NPE问题
*/
public static <T> T requireNonNull(T obj) {
  if (obj == null)
         throw new NullPointerException();
  return obj;
}
```

Khác biệt chính giữa phương thức `ofNullable` và phương thức `of` là: khi value là `null`, `ofNullable` trả về Optional rỗng, còn `of` sẽ ném ra `NullPointerException`. Khi `null` biểu thị "không có giá trị" hợp lệ thì dùng `ofNullable`; khi tham số theo quy ước bắt buộc phải khác `null` thì có thể dùng `of` để sớm bộc lộ lỗi.

**Sự khác biệt giữa `map()` và `flatMap()` là gì?**

Cả `map` và `flatMap` đều áp dụng một hàm lên từng phần tử trong collection, nhưng điểm khác biệt là `map` trả về một collection mới, còn `flatMap` ánh xạ từng phần tử thành một collection, cuối cùng làm phẳng (flatten) collection này.

Trong kịch bản thực tế, nếu `map` trả về mảng, thì kết quả cuối cùng thu được là một mảng hai chiều, việc sử dụng `flatMap` chính là để làm phẳng mảng hai chiều này thành mảng một chiều.

```java
public class MapAndFlatMapExample {
    public static void main(String[] args) {
        List<String[]> listOfArrays = Arrays.asList(
                new String[]{"apple", "banana", "cherry"},
                new String[]{"orange", "grape", "pear"},
                new String[]{"kiwi", "melon", "pineapple"}
        );

        List<String[]> mapResult = listOfArrays.stream()
                .map(array -> Arrays.stream(array).map(String::toUpperCase).toArray(String[]::new))
                .collect(Collectors.toList());

        System.out.println("Using map:");
        mapResult.forEach(arrays-> System.out.println(Arrays.toString(arrays)));

        List<String> flatMapResult = listOfArrays.stream()
                .flatMap(array -> Arrays.stream(array).map(String::toUpperCase))
                .collect(Collectors.toList());

        System.out.println("Using flatMap:");
        System.out.println(flatMapResult);
    }
}

```

Kết quả chạy:

```plain
Using map:
[[APPLE, BANANA, CHERRY], [ORANGE, GRAPE, PEAR], [KIWI, MELON, PINEAPPLE]]

Using flatMap:
[APPLE, BANANA, CHERRY, ORANGE, GRAPE, PEAR, KIWI, MELON, PINEAPPLE]
```

Cách hiểu đơn giản nhất là `flatMap()` có thể trải rộng (flatten) kết quả của `map()`.

Trong `Optional`, khi sử dụng `map()`, nếu hàm ánh xạ trả về một giá trị thông thường, nó sẽ bọc giá trị đó trong một `Optional` mới. Còn khi sử dụng `flatMap`, nếu hàm ánh xạ trả về một `Optional`, nó sẽ làm phẳng `Optional` được trả về này chứ không bọc thành `Optional` lồng nhau nữa.

Dưới đây là mã ví dụ so sánh:

```java
public static void main(String[] args) {
        int userId = 1;

        // 使用flatMap的代码
        String cityUsingFlatMap = getUserById(userId)
                .flatMap(OptionalExample::getAddressByUser)
                .map(Address::getCity)
                .orElse("Unknown");

        System.out.println("User's city using flatMap: " + cityUsingFlatMap);

        // 不使用flatMap的代码
        Optional<Optional<Address>> optionalAddress = getUserById(userId)
                .map(OptionalExample::getAddressByUser);

        String cityWithoutFlatMap;
        if (optionalAddress.isPresent()) {
            Optional<Address> addressOptional = optionalAddress.get();
            if (addressOptional.isPresent()) {
                Address address = addressOptional.get();
                cityWithoutFlatMap = address.getCity();
            } else {
                cityWithoutFlatMap = "Unknown";
            }
        } else {
            cityWithoutFlatMap = "Unknown";
        }

        System.out.println("User's city without flatMap: " + cityWithoutFlatMap);
    }
```

Sử dụng đúng `flatMap` trong `Stream` và `Optional` có thể giảm bớt rất nhiều mã nguồn không cần thiết.

### Kiểm tra value có phải null hay không

```java
/**
* value是否为null
*/
public boolean isPresent() {
    return value != null;
}
/**
* 如果value不为null执行consumer.accept
*/
public void ifPresent(Consumer<? super T> consumer) {
   if (value != null)
    consumer.accept(value);
}
```

### Lấy value

```java
/**
* Return the value if present, otherwise invoke {@code other} and return
* the result of that invocation.
* 如果value != null 返回value，否则返回other的执行结果
*/
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}

/**
* 如果value != null 返回value，否则返回T
*/
public T orElse(T other) {
    return value != null ? value : other;
}

/**
* 如果value != null 返回value，否则抛出参数返回的异常
*/
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
}
/**
* value为null抛出NoSuchElementException，不为空返回value。
*/
public T get() {
  if (value == null) {
      throw new NoSuchElementException("No value present");
  }
  return value;
}
```

### Lọc giá trị

```java
/**
* 1. 如果是empty返回empty
* 2. predicate.test(value)==true 返回this，否则返回empty
*/
public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
}
```

### Tóm tắt nhỏ

Sau khi xem xong mã nguồn `Optional` có thể nhận thấy rằng, `of()` yêu cầu tham số phải khác `null`, `get()` ném ra `NoSuchElementException` khi Optional rỗng, còn `flatMap()` dùng để làm phẳng kết quả ánh xạ trả về Optional, chứ không phải là phương thức nên tránh sử dụng. Thông thường nên chọn `of()` hoặc `ofNullable()` tùy thuộc vào việc giá trị có cho phép thiếu hay không, và ưu tiên sử dụng các phương thức như `orElse`, `orElseGet`, `orElseThrow` để xử lý giá trị rỗng. Cuối cùng, tổng hợp lại cách dùng các phương thức tần suất cao của `Optional`.

```java
Optional.ofNullable(zoo).map(o -> o.getDog()).map(d -> d.getAge()).filter(v->v==1).orElse(3);
```

## Date-Time API

Đây là sự bổ sung mạnh mẽ cho `java.util.Date`, giải quyết hầu hết các điểm nhói lòng của lớp Date:

1. Không an toàn với luồng (Not thread-safe)
2. Xử lý múi giờ (time zone) phiền phức
3. Định dạng và tính toán thời gian rắc rối
4. Thiết kế có khuyết tật, lớp Date chứa đồng thời cả ngày và giờ; ngoài ra còn có java.sql.Date dễ gây nhầm lẫn.

Chúng ta hãy so sánh sự khác biệt giữa java.util.Date và Date mới qua các ví dụ thời gian thường dùng. Đã đến lúc phải sửa lại đoạn mã dùng `java.util.Date` rồi.

### Các lớp chính trong java.time

`java.util.Date` chứa cả ngày và giờ, trong khi `java.time` tách biệt chúng ra:

```java
LocalDateTime.class //日期+时间 format: yyyy-MM-ddTHH:mm:ss.SSS
LocalDate.class //日期 format: yyyy-MM-dd
LocalTime.class //时间 format: HH:mm:ss
```

### Định dạng (Formatting)

**Trước Java 8:**

```java
public void oldFormat(){
    Date now = new Date();
    //format yyyy-MM-dd
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    String date  = sdf.format(now);
    System.out.println(String.format("date format : %s", date));

    //format HH:mm:ss
    SimpleDateFormat sdft = new SimpleDateFormat("HH:mm:ss");
    String time = sdft.format(now);
    System.out.println(String.format("time format : %s", time));

    //format yyyy-MM-dd HH:mm:ss
    SimpleDateFormat sdfdt = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String datetime = sdfdt.format(now);
    System.out.println(String.format("dateTime format : %s", datetime));
}
```

**Sau Java 8:**

```java
public void newFormat(){
    //format yyyy-MM-dd
    LocalDate date = LocalDate.now();
    System.out.println(String.format("date format : %s", date));

    //format HH:mm:ss
    LocalTime time = LocalTime.now().withNano(0);
    System.out.println(String.format("time format : %s", time));

    //format yyyy-MM-dd HH:mm:ss
    LocalDateTime dateTime = LocalDateTime.now();
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String dateTimeStr = dateTime.format(dateTimeFormatter);
    System.out.println(String.format("dateTime format : %s", dateTimeStr));
}
```

### Chuyển chuỗi thành định dạng ngày (Parsing String to Date)

**Trước Java 8:**

```java
//已弃用
Date date = new Date("2021-01-26");
//替换为
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
Date date1 = sdf.parse("2021-01-26");
```

**Sau Java 8:**

```java
LocalDate date = LocalDate.of(2021, 1, 26);
LocalDate.parse("2021-01-26");

LocalDateTime dateTime = LocalDateTime.of(2021, 1, 26, 12, 12, 22);
LocalDateTime.parse("2021-01-26T12:12:22");

LocalTime time = LocalTime.of(12, 12, 22);
LocalTime.parse("12:12:22");
```

**Trước Java 8** việc chuyển đổi đều phải nhờ vào lớp `SimpleDateFormat`, còn **Sau Java 8** chỉ cần phương thức `of` hoặc `parse` của `LocalDate`, `LocalTime`, `LocalDateTime`.

### Tính toán ngày tháng

Dưới đây chỉ lấy ví dụ **ngày sau 1 tuần**, các đơn vị khác (năm, tháng, ngày, 1/2 ngày, giờ,...) cũng tương tự. Ngoài ra, các đơn vị này đều được định nghĩa trong enum _java.time.temporal.ChronoUnit_.

**Trước Java 8:**

```java
public void afterDay(){
     //一周后的日期
     SimpleDateFormat formatDate = new SimpleDateFormat("yyyy-MM-dd");
     Calendar ca = Calendar.getInstance();
     ca.add(Calendar.DATE, 7);
     Date d = ca.getTime();
     String after = formatDate.format(d);
     System.out.println("一周后日期：" + after);

   //算两个日期间隔多少天，计算间隔多少年，多少月方法类似
     String dates1 = "2021-12-23";
   String dates2 = "2021-02-26";
     SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
     Date date1 = format.parse(dates1);
     Date date2 = format.parse(dates2);
     int day = (int) ((date1.getTime() - date2.getTime()) / (1000 * 3600 * 24));
     System.out.println(dates1 + "和" + dates2 + "相差" + day + "天");
     //结果：2021-02-26和2021-12-23相差300天
}
```

**Sau Java 8:**

```java
public void pushWeek(){
     //一周后的日期
     LocalDate localDate = LocalDate.now();
     //方法1
     LocalDate after = localDate.plus(1, ChronoUnit.WEEKS);
     //方法2
     LocalDate after2 = localDate.plusWeeks(1);
     System.out.println("一周后日期：" + after);

     //算两个日期间隔多少天，计算间隔多少年，多少月
     LocalDate date1 = LocalDate.parse("2021-02-26");
     LocalDate date2 = LocalDate.parse("2021-12-23");
     Period period = Period.between(date1, date2);
     System.out.println("date1 到 date2 相隔："
                + period.getYears() + "年"
                + period.getMonths() + "月"
                + period.getDays() + "天");
   //打印结果是 “date1 到 date2 相隔：0年9月27天”
     //这里period.getDays()得到的天是抛去年月以外的天数，并不是总天数
     //如果要获取纯粹的总天数应该用下面的方法
     long day = date2.toEpochDay() - date1.toEpochDay();
     System.out.println(date1 + "和" + date2 + "相差" + day + "天");
     //打印结果：2021-02-26和2021-12-23相差300天
}
```

### Lấy ngày chỉ định

Ngoài việc tính toán ngày tháng phiền phức, việc lấy một ngày cụ thể cũng rất rắc rối, ví dụ lấy ngày cuối cùng của tháng này, ngày đầu tiên của tháng.

**Trước Java 8:**

```java
public void getDay() {

        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        //获取当前月第一天：
        Calendar c = Calendar.getInstance();
        c.set(Calendar.DAY_OF_MONTH, 1);
        String first = format.format(c.getTime());
        System.out.println("first day:" + first);

        //获取当前月最后一天
        Calendar ca = Calendar.getInstance();
        ca.set(Calendar.DAY_OF_MONTH, ca.getActualMaximum(Calendar.DAY_OF_MONTH));
        String last = format.format(ca.getTime());
        System.out.println("last day:" + last);

        //当年最后一天
        Calendar currCal = Calendar.getInstance();
        Calendar calendar = Calendar.getInstance();
        calendar.clear();
        calendar.set(Calendar.YEAR, currCal.get(Calendar.YEAR));
        calendar.roll(Calendar.DAY_OF_YEAR, -1);
        Date time = calendar.getTime();
        System.out.println("last day:" + format.format(time));
}
```

**Sau Java 8:**

```java
public void getDayNew() {
    LocalDate today = LocalDate.now();
    //获取当前月第一天：
    LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
    // 取本月最后一天
    LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());
    //取下一天：
    LocalDate nextDay = lastDayOfThisMonth.plusDays(1);
    //当年最后一天
    LocalDate lastday = today.with(TemporalAdjusters.lastDayOfYear());
    //2021年最后一个周日，如果用Calendar是不得烦死。
    LocalDate lastMondayOf2021 = LocalDate.parse("2021-12-31").with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
}
```

Trong `java.time.temporal.TemporalAdjusters` còn rất nhiều thuật toán tiện lợi, ở đây sẽ không dẫn mọi người xem API nữa, đều rất đơn giản, nhìn qua là hiểu ngay.

### JDBC và Java 8

Hiện tại quan hệ tương ứng giữa kiểu thời gian JDBC và kiểu thời gian Java 8 là:

1. `Date` ---> `LocalDate`
2. `Time` ---> `LocalTime`
3. `Timestamp` ---> `LocalDateTime`

Trước JDBC 4.2, thường sử dụng `java.sql.Date`, `java.sql.Time` và `java.sql.Timestamp` để biểu diễn các kiểu thời gian SQL này.

### Múi giờ (Time Zone)

> Múi giờ: Việc phân chia múi giờ chính thức được chia theo mỗi 15° kinh độ thành 1 múi giờ, toàn cầu có tổng cộng 24 múi giờ, mỗi múi giờ chênh lệch 1 giờ. Nhưng để tiện cho hành chính, thường chia 1 quốc gia hoặc 1 tỉnh thành vào cùng một múi giờ.

Đối tượng `java.util.Date` về bản chất lưu trữ số millisecond đã trôi qua từ 0 giờ ngày 1 tháng 1 năm 1970 (GMT) đến thời điểm mà đối tượng Date biểu diễn. Nghĩa là bất kể new Date ở múi giờ nào, số millisecond nó ghi lại đều giống nhau, không liên quan đến múi giờ. Nhưng khi sử dụng nên chuyển đổi nó thành giờ địa phương, điều này liên quan đến quốc tế hóa thời gian. Bản thân `java.util.Date` không hỗ trợ quốc tế hóa, cần phải nhờ vào `TimeZone`.

```java
//北京时间：Wed Jan 27 14:05:29 CST 2021
Date date = new Date();

SimpleDateFormat bjSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//北京时区
bjSdf.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
System.out.println("毫秒数:" + date.getTime() + ", 北京时间:" + bjSdf.format(date));

//东京时区
SimpleDateFormat tokyoSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
tokyoSdf.setTimeZone(TimeZone.getTimeZone("Asia/Tokyo"));  // 设置东京时区
System.out.println("毫秒数:" + date.getTime() + ", 东京时间:" + tokyoSdf.format(date));

//如果直接print会自动转成当前时区的时间
System.out.println(date);
//Wed Jan 27 14:05:29 CST 2021
```

Trong tính năng mới đã giới thiệu `java.time.ZonedDateTime` để biểu diễn thời gian kèm múi giờ. Nó có thể coi là `LocalDateTime + ZoneId`.

```java
//当前时区时间
ZonedDateTime zonedDateTime = ZonedDateTime.now();
System.out.println("当前时区时间: " + zonedDateTime);

//东京时间
ZoneId zoneId = ZoneId.of(ZoneId.SHORT_IDS.get("JST"));
ZonedDateTime tokyoTime = zonedDateTime.withZoneSameInstant(zoneId);
System.out.println("东京时间: " + tokyoTime);

// ZonedDateTime 转 LocalDateTime
LocalDateTime localDateTime = tokyoTime.toLocalDateTime();
System.out.println("东京时间转当地时间: " + localDateTime);

//LocalDateTime 转 ZonedDateTime
ZonedDateTime localZoned = localDateTime.atZone(ZoneId.systemDefault());
System.out.println("本地时区时间: " + localZoned);

//打印结果
当前时区时间: 2021-01-27T14:43:58.735+08:00[Asia/Shanghai]
东京时间: 2021-01-27T15:43:58.735+09:00[Asia/Tokyo]
东京时间转当地时间: 2021-01-27T15:43:58.735
当地时区时间: 2021-01-27T15:43:58.735+08:00[Asia/Shanghai]
```

### Tóm tắt nhỏ

Thông qua việc so sánh sự khác biệt giữa `Date` mới và cũ ở trên, tất nhiên chỉ liệt kê một phần sự khác biệt về chức năng, nhiều chức năng hơn nữa còn cần tự bạn khám phá. Tóm lại date-time-api mang lại sự tiện lợi lớn cho các thao tác ngày tháng. Trong công việc hàng ngày khi gặp các thao tác kiểu date, điều đầu tiên nên cân nhắc là date-time-api, khi thật sự không giải quyết được mới cân nhắc đến Date cũ.

## Tổng kết

Các tính năng mới của Java 8 mà chúng ta đã tổng hợp gồm có:

- Interface & functional Interface
- Lambda
- Stream
- Optional
- Date time-api

Đây đều là những tính năng thường dùng trong quá trình phát triển. Tổng hợp lại mới thấy chúng thực sự rất tuyệt, mà bản thân lại không áp dụng sớm hơn. Lúc nào cũng cảm thấy học tính năng mới của Java 8 tương đối phiền phức, nên cứ dùng cách triển khai cũ. Thực ra những tính năng mới này chỉ mất vài ngày là có thể nắm vững, một khi đã nắm vững thì hiệu suất sẽ tăng lên rất nhiều. Thực ra tiền lương của chúng ta tăng lên cũng là tiền học hỏi, không học hỏi rốt cuộc sẽ bị đào thải, khủng hoảng tuổi 35 sẽ đến sớm hơn.

<!-- @include: @article-footer.snippet.md -->
