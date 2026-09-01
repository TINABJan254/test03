---
title: Java 9 新特性概览
description: 解析 Java 9 的模块化系统与 jlink 等更新，理解对运行时镜像与库使用的影响。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 9,JDK9,模块化,JPMS,jlink,集合工厂方法,新 API
---

**Java 9** được phát hành vào ngày 21 tháng 9 năm 2017. Là một phiên bản mới được phát hành sau Java 8 3 năm rưỡi, Java 9 mang lại nhiều thay đổi lớn, trong đó thay đổi quan trọng nhất là việc giới thiệu hệ thống module của nền tảng Java, ngoài ra còn có các cải tiến như Collection, `Stream` stream,...

JDK 9 không phải là bản LTS (Hỗ trợ dài hạn). Các phiên bản LTS hiện tại do Oracle niêm yết bao gồm JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

Bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 222: Java Shell Tool (JShell)](https://openjdk.org/jeps/222)
- [JEP 261: Module System (Hệ thống module)](https://openjdk.org/jeps/261)
- [JEP 248: G1 Becomes the Default Garbage Collector (G1 trở thành Bộ thu gom rác mặc định)](https://openjdk.org/jeps/248)
- [JEP 254: Compact Strings (Chuỗi nén / Chuỗi thu gọn)](https://openjdk.org/jeps/254)
- [JEP 193: Variable Handles (Biến handle)](https://openjdk.org/jeps/193)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 222: Java Shell Tool (JShell)

JShell là một công cụ tiện ích mới được bổ sung trong Java 9. Nó cung cấp một công cụ tương tác dòng lệnh thời gian thực cho Java tương tự như Python.

Trong JShell có thể nhập trực tiếp các biểu thức và xem kết quả thực thi của chúng ngay lập tức.

![](https://oss.javaguide.cn/java-guide-blog/image-20210816083417616.png)

**JShell mang lại cho chúng ta những lợi ích gì?**

1. Giảm bớt rào cản xuất ra dòng lệnh Java "Hello World!" đầu tiên, giúp nâng cao sự hào hứng học tập của người mới.
2. Khi xử lý các logic nhỏ đơn giản, xác minh các vấn đề nhỏ đơn giản, nó hiệu quả hơn so với IDE (không phải để thay thế IDE, đối với việc xác minh logic phức tạp thì IDE phù hợp hơn, cả hai bổ sung cho nhau).
3. ……

**Mã nguồn JShell khác gì với mã nguồn có thể biên dịch thông thường?**

1. Ngay khi câu lệnh được nhập xong, JShell sẽ biên dịch và thực thi mã ở nền sau, sau đó lập tức trả về kết quả, người dùng không cần phải chạy `javac` và `java` thủ công.
2. JShell hỗ trợ khai báo lại biến, khai báo phía sau sẽ đè lên khai báo phía trước.
3. JShell hỗ trợ các biểu thức độc lập ví dụ như phép cộng thông thường `1 + 1`.
4. ……

## JEP 261: Module System (Hệ thống module)

Hệ thống module là một phần của [Jigsaw Project](https://openjdk.java.net/projects/jigsaw/), đưa thực tiễn phát triển module vào nền tảng Java, giúp cho mã nguồn của chúng ta có khả năng tái sử dụng tốt hơn!

**Hệ thống module là gì?** Định nghĩa chính thức là:

> A uniquely named, reusable group of related packages, as well as resources (such as images and XML files) and a module descriptor。

Nói một cách đơn giản, bạn có thể coi một module như một tập hợp các package, tài nguyên và file mô tả module (`module-info.java`) được đặt tên duy nhất và có thể tái sử dụng.

Bất kỳ file jar nào, chỉ cần thêm một file mô tả module (`module-info.java`), là có thể nâng cấp thành một module.

![](https://oss.javaguide.cn/java-guide-blog/module-structure.png)

Sau khi giới thiệu hệ thống module, JDK được tái tổ chức thành 94 module. Ứng dụng Java có thể thông qua **công cụ [jlink](http://openjdk.java.net/jeps/282) mới được bổ sung** (Jlink là công cụ dòng lệnh mới được phát hành cùng với Java 9. Nó cho phép các nhà phát triển tạo image runtime nhẹ, tùy chỉnh của riêng họ cho các ứng dụng Java dựa trên module), tạo ra runtime image tùy chỉnh chỉ chứa các module JDK mà ứng dụng phụ thuộc. Điều này có thể giảm đáng kể dung lượng của môi trường chạy Java (JRE).

Chúng ta có thể thông qua từ khóa `exports` để kiểm soát những package nào có thể mở ra cho bên ngoài sử dụng, cũng như những package này có thể mở cho những module nào.

```java
module my.module {
    //exports công khai tất cả thành viên public của package được chỉ định
    exports com.my.package.name;
}

module my.module {
    // exports ... to xuất định hướng package được chỉ định cho module được chỉ định
    exports com.my.package.name to com.specific.module;
}
```

Nếu muốn tìm hiểu sâu hơn về tính năng module hóa của Java 9, bạn có thể tham khảo các bài viết dưới đây:

- [《Project Jigsaw: Module System Quick-Start Guide》](https://openjdk.java.net/projects/jigsaw/quick-start)
- [《Java 9 Modules: part 1》](https://stacktraceguru.com/java9/module-introduction)
- [Giải mã Java 9 (2. Hệ thống module hóa)](http://www.cnblogs.com/IcanFixIt/p/6947763.html)

## JEP 248: G1 Becomes the Default Garbage Collector (G1 trở thành Bộ thu gom rác mặc định)

Ở thời Java 8, Bộ thu gom rác mặc định là Parallel Scavenge (thế hệ trẻ) + Parallel Old (thế hệ già). Đến Java 9, Bộ thu gom rác CMS đã bị loại bỏ (deprecated), **G1 (Garbage-First Garbage Collector)** đã trở thành Bộ thu gom rác mặc định.

G1 được giới thiệu từ Java 7, qua biểu hiện xuất sắc của 2 phiên bản đã trở thành Bộ thu gom rác mặc định.

## JEP 193: Variable Handles (Biến handle)

Variable Handle là tham chiếu đến một biến hoặc một nhóm biến, bao gồm các trường static, trường non-static, phần tử mảng và các thành phần trong cấu trúc dữ liệu ngoài heap,...

Ý nghĩa của VarHandle tương tự như `MethodHandle` đã có, được biểu thị bởi lớp Java `java.lang.invoke.VarHandle`, có thể sử dụng các phương thức tìm kiếm của instance `java.lang.invoke.MethodHandles.Lookup` để tạo đối tượng `VarHandle`.

Sự xuất hiện của `VarHandle` thay thế cho một số thao tác của `java.util.concurrent.atomic` và `sun.misc.Unsafe`. Đồng thời cung cấp một chuỗi các thao tác rào cản bộ nhớ (memory barrier) chuẩn hóa, dùng để kiểm soát thứ tự bộ nhớ chi tiết hơn. Nó vượt trội hơn các API hiện có về tính an toàn, tính khả dụng và hiệu năng.

## Tăng cường API

Không phải tất cả các thay đổi API đều được phát hành thông qua JEP (Java Enhancement Proposal).

Trong quy trình phát triển JDK: **JEP** thường được dùng cho các thay đổi lớn, ví dụ giới thiệu tính năng ngôn ngữ mới, cơ chế JVM mới hoặc tái cấu trúc thư viện quy mô lớn. Còn các thao tác như bổ sung một vài phương thức factory trong lớp hiện có như `List.of()`, thường được coi là bảo trì thư viện thông thường. Chúng được các nhà phát triển JDK trực tiếp nộp và xem xét thông qua các thẻ ticket của **JBS (JDK Bug System)**, sau đó phát hành trực tiếp cùng với phiên bản.

### Tăng cường Collection

Đã bổ sung thêm các phương thức factory như `List.of()`, `Set.of()`, `Map.of()` và `Map.ofEntries()` để tạo collection bất biến (có một chút phong cách tham khảo Guava):

```java
List.of("Java", "C++");
Set.of("Java", "C++");
Map.of("Java", 1, "C++", 2);
```

Collection được tạo bằng `of()` là collection bất biến, không thể thực hiện các thao tác thêm, xóa, thay thế, sắp xếp,... nếu không sẽ báo ngoại lệ `java.lang.UnsupportedOperationException`.

### Tăng cường Stream

Trong `Stream` đã bổ sung các phương thức mới `ofNullable()`, `dropWhile()`, `takeWhile()` cũng như phương thức nạp chồng (overload) của phương thức `iterate()`.

Phương thức `ofNullable()` trong Java 9 có thể tạo `Stream` đơn phần tử hoặc rỗng dựa trên một giá trị có thể là `null`. Java 8 đã có thể tạo stream rỗng thông qua `Stream.empty()`, nhưng không có phương thức tiện lợi để chuyển trực tiếp giá trị cho phép null thành stream.

```java
Stream<String> stringStream = Stream.ofNullable("Java");
System.out.println(stringStream.count());// 1
Stream<String> nullStream = Stream.ofNullable(null);
System.out.println(nullStream.count());//0
```

Phương thức `takeWhile()` có thể lấy lần lượt các phần tử đáp ứng điều kiện từ `Stream`, cho đến khi gặp phần tử không đáp ứng điều kiện thì kết thúc việc lấy.

```java
List<Integer> integerList = List.of(11, 33, 66, 8, 9, 13);
integerList.stream().takeWhile(x -> x < 50).forEach(System.out::println);// 11 33
```

Tác dụng của phương thức `dropWhile()` ngược lại với `takeWhile()`.

```java
List<Integer> integerList2 = List.of(11, 33, 66, 8, 9, 13);
integerList2.stream().dropWhile(x -> x < 50).forEach(System.out::println);// 66 8 9 13
```

Phương thức nạp chồng mới của phương thức `iterate()` cung cấp một tham số `Predicate` (điều kiện phán đoán) để quyết định khi nào kết thúc vòng lặp

```java
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
}
// 新增加的重载方法
public static<T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next) {

}
```

So sánh cách sử dụng của cả hai như sau, phương thức nạp chồng `iterate()` mới linh hoạt hơn một chút.

```java
// 使用原始 iterate() 方法输出数字 1~10
Stream.iterate(1, i -> i + 1).limit(10).forEach(System.out::println);
// 使用新的 iterate() 重载方法输出数字 1~10
Stream.iterate(1, i -> i <= 10, i -> i + 1).forEach(System.out::println);
```

### Tăng cường Optional

Trong lớp `Optional` đã bổ sung các phương thức mới như `ifPresentOrElse()`, `or()` và `stream()`

Phương thức `ifPresentOrElse()` nhận hai tham số `Consumer` và `Runnable`, nếu `Optional` không rỗng sẽ gọi tham số `Consumer`, nếu rỗng sẽ gọi tham số `Runnable`.

```java
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)

Optional<Object> objectOptional = Optional.empty();
objectOptional.ifPresentOrElse(System.out::println, () -> System.out.println("Empty!!!"));// Empty!!!
```

Phương thức `or()` nhận một tham số `Supplier`, nếu `Optional` rỗng sẽ trả về giá trị `Optional` được chỉ định bởi tham số `Supplier`.

```java
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)

Optional<Object> objectOptional = Optional.empty();
objectOptional.or(() -> Optional.of("java")).ifPresent(System.out::println);//java
```

### Tăng cường String

Ở Java 8 và các phiên bản trước đó, `String` luôn được lưu trữ bằng `char[]`. Sau Java 9, triển khai của `String` đổi sang dùng mảng `byte[]` để lưu trữ chuỗi, giúp tiết kiệm không gian.

```java
public final class String implements java.io.Serializable,Comparable<String>, CharSequence {
    // Annotation @Stable biểu thị biến bị sửa đổi tối đa một lần, gọi là "ổn định".
    @Stable
    private final byte[] value;
}
```

### Tăng cường Interface

Java 9 cho phép sử dụng phương thức private trong interface. Như vậy, việc sử dụng interface trở nên linh hoạt hơn, có phần giống như một phiên bản đơn giản hóa của abstract class.

```java
public interface MyInterface {
    private void methodPrivate(){
    }
}
```

### Tăng cường IO

Trước Java 9, chúng ta chỉ có thể khai báo biến trong khối `try-with-resources`:

```java
try (Scanner scanner = new Scanner(new File("testRead.txt"));
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) {
    // omitted
}
```

Sau Java 9, trong câu lệnh `try-with-resources` có thể sử dụng biến effectively-final.

```java
final Scanner scanner = new Scanner(new File("testRead.txt"));
PrintWriter writer = new PrintWriter(new File("testWrite.txt"));
try (scanner; writer) {
    // omitted
}
```

**Biến effectively-final là gì?** Nói một cách đơn giản là biến không được bổ sung từ khóa `final` nhưng giá trị chưa từng bị thay đổi sau khi khởi tạo.

Đúng như mã nguồn ở trên minh họa, ngay cả khi biến `writer` không được khai báo rõ ràng là `final`, nhưng sau khi được gán giá trị lần đầu tiên nó sẽ không thay đổi nữa, do đó, nó chính là biến effectively-final.

### Process API (API Tiến trình)

Java 9 bổ sung interface `java.lang.ProcessHandle` để thực hiện việc quản lý các tiến trình native, đặc biệt thích hợp để quản lý các tiến trình chạy trong thời gian dài.

```java
// Lấy tiến trình JVM hiện tại đang chạy
ProcessHandle currentProcess = ProcessHandle.current();
// In id của tiến trình
System.out.println(currentProcess.pid());
// In thông tin của tiến trình
System.out.println(currentProcess.info());
```

Tổng quan interface `ProcessHandle`:

![](https://oss.javaguide.cn/java-guide-blog/image-20210816104614414.png)

### Các tăng cường API khác

**Reactive Streams (Luồng phản ứng)**

Trong lớp `java.util.concurrent.Flow` của Java 9 đã bổ sung thêm các interface cốt lõi của quy chuẩn Reactive Streams.

`Flow` bao gồm 4 interface cốt lõi là `Flow.Publisher`, `Flow.Subscriber`, `Flow.Subscription` và `Flow.Processor`. Java 9 cũng cung cấp `SubmissionPublisher` như một triển khai của `Flow.Publisher`.

Về giải thích chi tiết hơn về Reactive Streams trong Java 9, khuyên bạn nên xem bài viết [Giải mã Java 9 (17. Reactive Streams) - Lâm Bản Thác](https://www.cnblogs.com/IcanFixIt/p/7245377.html).

## Khác

- **Cải tiến Logging API cho nền tảng**: Java 9 cho phép cấu hình cùng một triển khai log cho JDK và ứng dụng. Bổ sung thêm `System.LoggerFinder` dùng để quản lý triển khai logger mà JDK sử dụng. JVM ở thời điểm runtime chỉ có một instance `LoggerFinder` trên toàn hệ thống. Chúng ta có thể thông qua việc thêm triển khai `System.LoggerFinder` của riêng mình để giúp JDK và ứng dụng sử dụng các framework ghi log khác như SLF4J.
- **Tăng cường lớp `CompletableFuture`**: Bổ sung một vài phương thức mới (`completeAsync`, `orTimeout`,...).
- **Tăng cường Nashorn Engine**: Nashorn là JavaScript engine được giới thiệu từ Java 8, Java 9 đã thực hiện một số tăng cường cho Nashorn, triển khai một số tính năng mới của ES6 (đã bị loại bỏ trong Java 11).
- **Tính năng mới của I/O Stream**: Bổ sung phương thức mới để đọc và sao chép dữ liệu chứa trong `InputStream`.
- **Cải tiến hiệu năng bảo mật ứng dụng**: Java 9 bổ sung thêm 4 thuật toán băm SHA-3 là SHA3-224, SHA3-256, SHA3-384 và SHA3-512.
- **Cải tiến Method Handle**: Method Handle được giới thiệu từ Java 7, Java 9 đã bổ sung thêm nhiều phương thức static hơn trong lớp `java.lang.invoke.MethodHandles` để tạo các loại Method Handle khác nhau.
- ……

## Tham khảo

- Java version history: <https://en.wikipedia.org/wiki/Java_version_history>
- Release Notes for JDK 9 and JDK 9 Update Releases : <https://www.oracle.com/java/technologies/javase/9-all-relnotes.html>
- 《Phân tích sâu tính năng mới của Java》- Geek Time - JShell: Làm thế nào để xác minh nhanh các vấn đề nhỏ đơn giản?
- New Features in Java 9: <https://www.baeldung.com/new-java-9>
- Java – Try with Resources: <https://www.baeldung.com/java-try-with-resources>

<!-- @include: @article-footer.snippet.md -->
