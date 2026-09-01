---
title: Java 11 新特性概览（重要）
description: 总结 JDK 11 的更新，关注新 HTTP 客户端与字符串增强等实用特性。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 11,JDK11,LTS,HTTP 客户端,字符串 API,移除特性
---

Java 11 được phát hành chính thức vào ngày 25 tháng 9 năm 2018, đây là một phiên bản rất quan trọng! Java 11 là phiên bản Hỗ trợ dài hạn (Long-Term-Support) đầu tiên sau Java 8. Theo lộ trình hỗ trợ hiện tại của Oracle, Extended Support của Java 11 sẽ kéo dài tới tháng 1 năm 2032.

Dưới đây là hình ảnh biểu đồ thời gian hỗ trợ Oracle JDK chính thức do Oracle cung cấp.

![Biểu đồ thời gian hỗ trợ Oracle JDK chính thức do Oracle cung cấp](https://oss.javaguide.cn/github/javaguide/java/new-features/4c1611fad59449edbbd6e233690e9fa7.png)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 321: HTTP Client (Standard)](https://openjdk.org/jeps/321)
- [JEP 323: Local-Variable Syntax for Lambda Parameters](https://openjdk.org/jeps/323)
- [JEP 330: Launch Single-File Source-Code Programs](https://openjdk.org/jeps/330)
- [JEP 333: ZGC: A Scalable Low-Latency Garbage Collector (Experimental)](https://openjdk.org/jeps/333)

## JEP 321: HTTP Client（HTTP Client, Phiên bản chuẩn）

Java 11 đã chuẩn hóa HTTP Client API từng được giới thiệu trong Java 9 và cập nhật trong Java 10, trong quá trình ươm tạo ở hai phiên bản trước, HTTP Client hầu như đã được viết lại hoàn toàn, và hiện tại hỗ trợ hoàn toàn bất đồng bộ không chặn (asynchronous non-blocking).

Hơn nữa, trong Java 11, package name của HTTP Client đã được đổi từ `jdk.incubator.http` thành `java.net.http`, API này cung cấp ngữ nghĩa yêu cầu và phản hồi không chặn thông qua `CompletableFuture`. Việc sử dụng cũng rất đơn giản như sau:

```java
var request = HttpRequest.newBuilder()
    .uri(URI.create("https://javastack.cn"))
    .GET()
    .build();
var client = HttpClient.newHttpClient();

// Đồng bộ
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());

// Bất đồng bộ
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println);
```

## JEP 333: ZGC（Bộ thu gom rác độ trễ thấp có thể mở rộng, Thử nghiệm）

**ZGC tức Z Garbage Collector**, là một Bộ thu gom rác độ trễ thấp, có khả năng mở rộng.

ZGC được thiết kế chủ yếu để đáp ứng các mục tiêu sau:

- Thời gian dừng GC không vượt quá 10ms
- Vừa có thể xử lý heap nhỏ vài trăm MB, vừa có thể xử lý heap lớn vài TB
- Khả năng thông lượng của ứng dụng không giảm quá 15% (so với thuật toán thu hồi G1)
- Thuận tiện cho việc giới thiệu các tính năng GC mới trên nền tảng này và đặt nền móng cho việc sử dụng con trỏ màu (colored pointers) cũng như rào cản tải (load barriers) để tối ưu hóa
- Hiện tại chỉ hỗ trợ nền tảng Linux/x64

ZGC hiện tại **đang ở giai đoạn thử nghiệm**, chỉ hỗ trợ nền tảng Linux/x64. Lưu ý: ZGC trở thành tính năng chính thức trong Java 15, và giới thiệu Generational ZGC trong Java 21.

Tương tự như ParNew trong CMS và G1, ZGC cũng áp dụng thuật toán Đánh dấu - Sao chép (Mark-Copy), tuy nhiên ZGC đã thực hiện cải tiến lớn đối với thuật toán này.

Trong ZGC, tình trạng Stop The World xảy ra ít hơn rất nhiều!

Chi tiết có thể xem: [《Khám phá và thực tiễn về Bộ thu gom rác thế hệ mới ZGC》](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)

## JEP 323: Local-Variable Syntax for Lambda Parameters（Cú pháp biến cục bộ cho tham số Lambda）

Từ Java 10, tính năng then chốt suy luận kiểu biến cục bộ đã được giới thiệu. Suy luận kiểu cho phép sử dụng từ khóa var làm kiểu của biến cục bộ thay vì kiểu thực tế, trình biên dịch suy luận ra kiểu dựa trên giá trị được gán cho biến.

Trong Java 10 có một vài hạn chế đối với từ khóa var:

- Chỉ có thể dùng trên biến cục bộ
- Phải khởi tạo khi khai báo
- Không thể dùng làm tham số phương thức
- Không thể sử dụng trong biểu thức Lambda

Từ Java 11 cho phép các nhà phát triển sử dụng var để khai báo tham số trong biểu thức Lambda.

```java
// Cả hai cách dưới đây là tương đương
Consumer<String> consumer = (var i) -> System.out.println(i);
Consumer<String> consumer = (String i) -> System.out.println(i);
```

## JEP 330: Launch Single-File Source-Code Programs（Khởi chạy chương trình mã nguồn đơn file）

Điều này có nghĩa là chúng ta có thể chạy mã nguồn Java của một file đơn lẻ. Tính năng này cho phép sử dụng trình thông dịch Java để thực thi trực tiếp mã nguồn Java. Mã nguồn được biên dịch trong bộ nhớ, sau đó được thực thi bởi trình thông dịch, không cần tạo file `.class` trên đĩa nữa. Ràng buộc duy nhất là tất cả các class liên quan phải được định nghĩa trong cùng một file Java.

Rất hữu ích đối với người mới bắt đầu học Java và muốn thử nghiệm các chương trình đơn giản, hơn nữa có thể sử dụng cùng với jshell, ở một mức độ nào đó nâng cao khả năng sử dụng Java để viết các chương trình script.

## Tăng cường API

Không phải tất cả các thay đổi API đều được phát hành thông qua JEP (Java Enhancement Proposal).

Trong quy trình phát triển JDK: **JEP** thường được dùng cho các thay đổi lớn, ví dụ giới thiệu tính năng ngôn ngữ mới (như `var`), cơ chế JVM mới (như ZGC) hoặc tái cấu trúc thư viện quy mô lớn. Còn các thao tác như bổ sung một vài phương thức trong lớp hiện có như `String.isBlank()`, thường được coi là bảo trì thư viện thông thường. Chúng được các nhà phát triển JDK trực tiếp nộp và xem xét thông qua các thẻ ticket của **JBS (JDK Bug System)**, sau đó phát hành trực tiếp cùng với phiên bản.

### Tăng cường String

Java 11 bổ sung một chuỗi các phương thức xử lý chuỗi:

```java
//Kiểm tra chuỗi có trống/chỉ chứa khoảng trắng hay không
" ".isBlank();//true
//Loại bỏ khoảng trắng ở đầu và cuối chuỗi
" Java ".strip();// "Java"
//Loại bỏ khoảng trắng ở đầu chuỗi
" Java ".stripLeading();   // "Java "
//Loại bỏ khoảng trắng ở cuối chuỗi
" Java ".stripTrailing();  // "Java"
//Lặp lại chuỗi bao nhiêu lần
"Java".repeat(3);             // "JavaJavaJava"
//Trả về tập hợp chuỗi được phân tách bởi ký tự kết thúc dòng.
"A\nB\nC".lines().count();    // 3
"A\nB\nC".lines().collect(Collectors.toList());
```

### Tăng cường Optional

Bổ sung phương thức `isEmpty()` để kiểm tra đối tượng `Optional` được chỉ định có rỗng hay không.

```java
var op = Optional.empty();
System.out.println(op.isEmpty());//Kiểm tra đối tượng Optional được chỉ định có rỗng hay không
```

## Các tính năng mới khác

- **Bộ thu gom rác mới Epsilon**: Một triển khai GC hoàn toàn thụ động, phân bổ tài nguyên bộ nhớ có hạn, giảm thiểu tối đa mức chiếm dụng bộ nhớ và thời gian độ trễ thông lượng bộ nhớ
- **Heap Profiling chi phí thấp**: Java 11 cung cấp một phương pháp lấy mẫu phân bổ heap Java chi phí thấp, có thể lấy thông tin đối tượng Java được phân bổ trên heap, và có thể truy cập thông tin heap thông qua JVMTI
- **Giao thức TLS 1.3**: Java 11 bao hàm triển khai quy chuẩn Transport Layer Security (TLS) 1.3 (RFC 8446), thay thế cho TLS được bao hàm trong các phiên bản trước bao gồm TLS 1.2, đồng thời cải tiến các chức năng TLS khác như OCSP stapling extensions (RFC 6066, RFC 6961), cũng như session hash và extended master secret extension (RFC 7627), thực hiện nhiều nâng cao về mặt an toàn và hiệu năng
- **Flight Recorder (Java Flight Recorder)**: Flight Recorder trước đây là một công cụ phân tích của bản JDK thương mại, nhưng trong Java 11, mã nguồn của nó đã được đưa vào codebase công khai, nhờ đó tất cả mọi người đều có thể sử dụng chức năng này.
- ......

## Tham khảo

- JDK 11 Release Notes：<https://www.oracle.com/java/technologies/javase/11-relnote-issues.html>
- Java 11 – Features and Comparison：<https://www.geeksforgeeks.org/java-11-features-and-comparison/>

<!-- @include: @article-footer.snippet.md -->
