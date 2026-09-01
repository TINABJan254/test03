---
title: Java 10 新特性概览
description: 概览 JDK 10 的主要更新，重点介绍 var 类型推断与其他平台改进。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 10,JDK10,var 局部变量类型推断,垃圾回收改进,性能
---

**Java 10** được phát hành vào ngày 20 tháng 3 năm 2018, đây là một phiên bản không phải LTS (Hỗ trợ dài hạn), Oracle chỉ cung cấp hỗ trợ trong 6 tháng.

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 286: Local-Variable Type Inference (Suy luận kiểu biến cục bộ)](https://openjdk.org/jeps/286)
- [JEP 304: Garbage-Collector Interface (Interface bộ thu gom rác)](https://openjdk.org/jeps/304)
- [JEP 307: Parallel Full GC for G1 (Parallel Full GC cho G1)](https://openjdk.org/jeps/307)
- [JEP 310: Application Class-Data Sharing (Chia sẻ dữ liệu class ứng dụng)](https://openjdk.org/jeps/310)
- [JEP 317: Experimental Java-Based JIT Compiler (Bộ biên dịch JIT dựa trên Java mang tính thử nghiệm)](https://openjdk.org/jeps/317)

## JEP 286: Local-Variable Type Inference

Do có rất nhiều nhà phát triển Java mong muốn giới thiệu suy luận kiểu biến cục bộ trong Java, nên ở thời Java 10 nó đã tới, đúng như kỳ vọng của mọi người!

Java 10 cung cấp từ khóa `var` để khai báo biến cục bộ.

```java
var id = 0;
var codefx = new URL("https://mp.weixin.qq.com/");
var list = new ArrayList<>();
var list = List.of(1, 2, 3);
var map = new HashMap<String, String>();
var p = Paths.of("src/test/java/Java9FeaturesTest.java");
var numbers = List.of("a", "b", "c");
for (var n : numbers)
    System.out.print(n+ " ");
```

`var` chỉ có thể dùng cho khai báo biến cục bộ có bộ khởi tạo (initializer), cũng có thể dùng cho biến cục bộ trong vòng lặp `for` cơ bản hoặc nâng cao, cũng như biến tài nguyên trong `try`-with-resources. Nó không thể dùng cho trường (field), tham số phương thức hoặc kiểu trả về.

```java
var count = null; //❌编译不通过，不能声明为 null
var r = () -> Math.random();//❌编译不通过，不能声明为 Lambda表达式
var array = {1, 2, 3};//❌编译不通过，不能声明数组
```

`var` không làm thay đổi sự thật rằng Java là một ngôn ngữ kiểu tĩnh (statically typed language), trình biên dịch chịu trách nhiệm suy luận ra kiểu.

Ngoài ra, trong Scala và Kotlin đã có từ khóa `val` (từ khóa kết hợp `final var`).

## JEP 304: Garbage-Collector Interface

Trong cấu trúc JDK sớm hơn, các component cấu thành nên triển khai Bộ thu gom rác (GC) bị phân tán ở các phần khác nhau của codebase. Java 10 thông qua việc giới thiệu một tập hợp các interface Bộ thu gom rác thuần túy để tách biệt mã nguồn của các Bộ thu gom rác khác nhau.

## JEP 307: Parallel Full GC for G1

Từ Java 9, G1 đã trở thành Bộ thu gom rác mặc định. G1 được thiết kế như một Bộ thu gom rác có độ trễ thấp, nhằm tránh thực hiện Full GC, nhưng Full GC của G1 trong Java 9 vẫn sử dụng đơn luồng (single-thread) để hoàn thành thuật toán Mark-Sweep (đánh dấu - dọn dẹp), điều này có thể dẫn đến việc Bộ thu gom rác kích hoạt Full GC khi không thể thu hồi bộ nhớ.

Để giảm thời gian dừng ứng dụng do Full GC gây ra, từ Java 10 trở đi, Full GC của G1 được đổi thành sử dụng nhiều luồng làm việc song song (parallel worker threads) để thực hiện đánh dấu, dọn dẹp và nén (Mark, Sweep, Compact). Thay đổi này rút ngắn thời gian dừng của Full GC, chứ không trực tiếp giảm số lần kích hoạt Full GC.

## JEP 310: Application Class-Data Sharing (Mở rộng tính năng CDS)

Java 5 đã giới thiệu cơ chế Chia sẻ dữ liệu class (Class Data Sharing, gọi tắt là CDS), cho phép tiền xử lý một nhóm các class hệ thống thành file lưu trữ chia sẻ (shared archive file), để tiến hành ánh xạ bộ nhớ lúc runtime, từ đó giảm thời gian khởi động của chương trình Java và dung lượng bộ nhớ chiếm dụng của nhiều JVM. Việc đưa các class ứng dụng vào lưu trữ chia sẻ AppCDS trước đây chỉ được cung cấp như một tính năng thương mại trong Oracle JDK.

Java 10 mở rộng thêm và mở rộng công khai AppCDS trên cơ sở tính năng CDS hiện có, cho phép đưa các class ứng dụng vào lưu trữ chia sẻ. Quy trình điển hình là trước tiên tạo danh sách class ứng dụng, sau đó dựa trên danh sách class để tạo lưu trữ chia sẻ, các lần khởi động tiếp theo sẽ tải lưu trữ đó thông qua ánh xạ bộ nhớ; bản thân văn bản danh sách class không phải là bộ nhớ đệm mà JVM tải trực tiếp ở các lần khởi động sau.

## JEP 317: Experimental Java-Based JIT Compiler (Bộ biên dịch JIT dựa trên Java mang tính thử nghiệm)

Graal là một bộ biên dịch JIT được viết bằng ngôn ngữ Java, là nền tảng của bộ biên dịch Ahead-of-Time (AOT) thử nghiệm được giới thiệu trong JDK 9.

HotSpot VM của Oracle đi kèm với hai JIT compiler được triển khai bằng C++: C1 và C2. Trong Java 10 (Linux/x64, macOS/x64), mặc định HotSpot vẫn sử dụng C2, nhưng bằng cách thêm tham số `-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler` vào lệnh java là có thể thay thế C2 bằng Graal.

## Tăng cường API

Không phải tất cả các thay đổi API đều được phát hành thông qua JEP (Java Enhancement Proposal).

Trong quy trình phát triển JDK: **JEP** thường được dùng cho các thay đổi lớn, ví dụ giới thiệu tính năng ngôn ngữ mới (như `var`), cơ chế JVM mới (như ZGC) hoặc tái cấu trúc thư viện quy mô lớn. Còn các thao tác như bổ sung một vài phương thức static trong lớp hiện có như `List.copyOf()`, thường được coi là bảo trì thư viện thông thường. Chúng được các nhà phát triển JDK trực tiếp nộp và xem xét thông qua các thẻ ticket của **JBS (JDK Bug System)**, sau đó phát hành trực tiếp cùng với phiên bản.

### Tăng cường Collection

`List`, `Set`, `Map` cung cấp phương thức static `copyOf()` trả về một bản sao bất biến của collection tham số đầu vào.

```java
static <E> List<E> copyOf(Collection<? extends E> coll) {
    return ImmutableCollections.listCopy(coll);
}
```

Collection được tạo bằng `copyOf()` là collection bất biến, không thể thực hiện các thao tác thêm, xóa, thay thế, sắp xếp,... nếu không sẽ báo ngoại lệ `java.lang.UnsupportedOperationException`. IDEA cũng sẽ có gợi ý tương ứng.

![Collection được tạo bằng `copyOf()` là collection bất biến](https://oss.javaguide.cn/java-guide-blog/image-20210816154125579.png)

Hơn nữa, trong `java.util.stream.Collectors` đã bổ sung thêm phương thức static, dùng để thu thập các phần tử trong stream thành collection bất biến.

```java
var list = new ArrayList<>();
list.stream().collect(Collectors.toUnmodifiableList());
list.stream().collect(Collectors.toUnmodifiableSet());
```

### Tăng cường Optional

`Optional` bổ sung một phương thức không tham số `orElseThrow()`, như một phiên bản đơn giản hóa của `orElseThrow(Supplier<? extends X> exceptionSupplier)` có tham số, mặc định ném ra ngoại lệ NoSuchElementException khi không có giá trị.

```java
Optional<String> optional = Optional.empty();
String result = optional.orElseThrow();
```

## Khác

- **Kiểm soát cục bộ luồng (Thread-Local Handshakes)**: Trong Java 10 kiểm soát luồng giới thiệu khái niệm JVM Safepoint, sẽ cho phép thực hiện callback luồng mà không cần chạy JVM Safepoint toàn cục, do bản thân luồng hoặc luồng JVM thực thi, đồng thời duy trì luồng ở trạng thái block. Cách thức này giúp cho việc dừng một luồng đơn lẻ trở nên khả thi, thay vì chỉ có thể bật hoặc dừng tất cả các luồng.
- **Phân bổ Heap trên thiết bị lưu trữ dự phòng**: Trong Java 10 sẽ giúp JVM có thể sử dụng heap phù hợp với các cơ chế lưu trữ thuộc các loại khác nhau, thực hiện phân bổ bộ nhớ heap trên thiết bị bộ nhớ tùy chọn.
- ……

## Tham khảo

- Java 10 Features and Enhancements : <https://howtodoinjava.com/java10/java10-features/>
- Guide to Java10 : <https://www.baeldung.com/java-10-overview>
- 4 Class Data Sharing : <https://docs.oracle.com/javase/10/vm/class-data-sharing.htm#JSJVM-GUID-7EAA3411-8CF0-4D19-BD05-DF5E1780AA91>

<!-- @include: @article-footer.snippet.md -->
