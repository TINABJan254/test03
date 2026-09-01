---
title: Java 17（JDK 17）新特性：密封类、switch 模式匹配与随机数 API
description: Java 17（JDK 17）新特性详解，涵盖密封类、switch 模式匹配、新随机数 API、外部函数与内存 API，以及 LTS 支持周期和升级时需要关注的变化。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 17,JDK 17,JDK17新特性,Java17新特性,LTS,密封类,switch模式匹配,随机数 API,外部函数与内存 API,JEP
---

Java 17 (JDK 17) được phát hành chính thức vào ngày 14 tháng 9 năm 2021, là phiên bản Hỗ trợ dài hạn (LTS) được Oracle xác nhận.

Theo lộ trình hỗ trợ Java SE được Oracle cập nhật vào tháng 4 năm 2026, Premier Support của Oracle JDK 17 kéo dài tới tháng 9 năm 2026, Extended Support kéo dài tới tháng 9 năm 2029. Chu kỳ cập nhật miễn phí và hỗ trợ thương mại của các bản phân phối JDK khác nhau là không giống nhau, môi trường sản xuất (production) vẫn phải dựa trên bản phân phối thực tế được sử dụng.

![](https://oss.javaguide.cn/github/javaguide/java/new-features/4c1611fad59449edbbd6e233690e9fa7.png)

Khi nâng cấp từ JDK 11 lên JDK 17, khuyên bạn nên tập trung chú ý vào Sealed Class, Switch Pattern Matching, API số giả ngẫu nhiên mới, cũng như các thay đổi về tính tương thích do việc đóng gói mạnh các API nội bộ JDK mang lại. Phiên bản Java tối thiểu của Spring 6.x và Spring Boot 3.x cũng là Java 17.

JDK 17 có tổng cộng 14 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 356: Enhanced Pseudo-Random Number Generators (Trình tạo số giả ngẫu nhiên được tăng cường)](https://openjdk.java.net/jeps/356)
- [JEP 398: Deprecate the Applet API for Removal (Đánh dấu loại bỏ Applet API để chuẩn bị xóa)](https://openjdk.java.net/jeps/398)
- [JEP 406: Pattern Matching for switch (Preview) (Khớp mẫu switch, Xem trước)](https://openjdk.java.net/jeps/406)
- [JEP 407: Remove RMI Activation (Loại bỏ cơ chế kích hoạt RMI)](https://openjdk.java.net/jeps/407)
- [JEP 409: Sealed Classes (Sealed Class, Chính thức)](https://openjdk.java.net/jeps/409)
- [JEP 410: Remove the Experimental AOT and JIT Compiler (Loại bỏ bộ biên dịch AOT và JIT thử nghiệm)](https://openjdk.java.net/jeps/410)
- [JEP 411: Deprecate the Security Manager for Removal (Đánh dấu loại bỏ Security Manager để chuẩn bị xóa)](https://openjdk.java.net/jeps/411)
- [JEP 412: Foreign Function & Memory API (Incubator) (Hàm ngoại và Memory API, Lần ươm tạo thứ 1)](https://openjdk.java.net/jeps/412)
- [JEP 414: Vector API (Second Incubator) (Vector API, Lần ươm tạo thứ 2)](https://openjdk.java.net/jeps/414)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 16:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Đọc thêm: [Tài liệu OpenJDK Java 17](https://openjdk.java.net/projects/jdk/17/).

## JEP 356: Enhanced Pseudo-Random Number Generators (Trình tạo số giả ngẫu nhiên được tăng cường)

Trước JDK 17, chúng ta có thể nhờ vào `Random`, `ThreadLocalRandom` và `SplittableRandom` để tạo số ngẫu nhiên. Tuy nhiên, 3 lớp này đều có khuyết điểm riêng, và thiếu sự hỗ trợ của các thuật toán giả ngẫu nhiên phổ biến.

Java 17 đã bổ sung các kiểu interface và triển khai mới cho trình tạo số giả ngẫu nhiên (pseudorandom number generator, PRNG, còn gọi là trình tạo bit ngẫu nhiên xác định), giúp các nhà phát triển dễ dàng trao đổi và sử dụng các thuật toán PRNG khác nhau trong ứng dụng.

> [PRNG](https://ctf-wiki.org/crypto/streamcipher/prng/intro/) được dùng để tạo ra chuỗi số tiệm cận với chuỗi số ngẫu nhiên tuyệt đối. Nói chung, PRNG sẽ phụ thuộc vào một giá trị ban đầu, còn gọi là seed (hạt giống), để tạo ra chuỗi số giả ngẫu nhiên tương ứng. Chỉ cần seed được xác định, số ngẫu nhiên do PRNG tạo ra là hoàn toàn xác định, do đó chuỗi số ngẫu nhiên được tạo ra không phải là thực sự ngẫu nhiên.

Ví dụ mã nguồn:

```java
RandomGeneratorFactory<RandomGenerator> l128X256MixRandom = RandomGeneratorFactory.of("L128X256MixRandom");
// 使用时间戳作为随机数种子
RandomGenerator randomGenerator = l128X256MixRandom.create(System.currentTimeMillis());
// 生成随机数
randomGenerator.nextInt(10);
```

## JEP 398: Deprecate the Applet API for Removal (Đánh dấu loại bỏ Applet API để chuẩn bị xóa)

Applet API dùng để viết các chương trình Java nhỏ chạy trên trình duyệt Web, nhiều năm trước đã bị đào thải, không còn lý do gì để sử dụng nữa.

Applet API đã bị đánh dấu loại bỏ trong Java 9 ([JEP 289](https://openjdk.java.net/jeps/289)), nhưng không phải để xóa ngay.

## JEP 406: Pattern Matching for switch (Khớp mẫu switch, Xem trước)

Đúng như `instanceof`, `switch` cũng nối tiếp bổ sung chức năng tự động chuyển đổi khớp kiểu.

Ví dụ mã nguồn `instanceof`:

```java
// Old code
if (o instanceof String) {
    String s = (String)o;
    ... use s ...
}

// New code
if (o instanceof String s) {
    ... use s ...
}
```

Ví dụ mã nguồn `switch`:

```java
// Old code
static String formatter(Object o) {
    String formatted = "unknown";
    if (o instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (o instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (o instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (o instanceof String s) {
        formatted = String.format("String %s", s);
    }
    return formatted;
}

// New code
static String formatterPatternSwitch(Object o) {
    return switch (o) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> o.toString();
    };
}

```

Việc kiểm tra giá trị `null` cũng được tối ưu hóa.

```java
// Old code
static void testFooBar(String s) {
    if (s == null) {
        System.out.println("oops!");
        return;
    }
    switch (s) {
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}

// New code
static void testFooBar(String s) {
    switch (s) {
        case null         -> System.out.println("Oops");
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}
```

## JEP 407: Remove RMI Activation (Loại bỏ cơ chế kích hoạt RMI)

Xóa cơ chế kích hoạt RMI (Remote Method Invocation Activation), đồng thời giữ lại phần còn lại của RMI. Cơ chế kích hoạt RMI đã lỗi thời và không còn được sử dụng nữa.

## JEP 409: Sealed Classes (Sealed class / Class niêm phong)

Sealed class được [JEP 360](https://openjdk.java.net/jeps/360) đề xuất xem trước và tích hợp vào Java 15. Trong JDK 16, Sealed class đã được cải tiến (kiểm tra tham chiếu nghiêm ngặt hơn và quan hệ kế thừa của Sealed class), được [JEP 397](https://openjdk.java.net/jeps/397) đề xuất xem trước lại.

Trong bài [Tổng quan tính năng mới Java 14 & 15](./java14-15.md), tôi đã giới thiệu chi tiết về Sealed class, ở đây không giới thiệu thêm nữa.

## JEP 410: Remove the Experimental AOT and JIT Compiler (Loại bỏ bộ biên dịch AOT và JIT thử nghiệm)

Trong [JEP 295](https://openjdk.java.net/jeps/295) của Java 9, đã giới thiệu bộ biên dịch Ahead-of-Time (AOT) thử nghiệm, biên dịch các class Java thành mã native trước khi khởi động virtual machine.

Trong Java 17, xóa bộ biên dịch Ahead-of-Time (AOT) và Just-In-Time (JIT) thử nghiệm, vì bộ biên dịch này từ khi đưa ra ít khi được sử dụng, công sức bảo trì nó rất lớn. Giữ lại JVM Compiler Interface (JVMCI) cấp Java thử nghiệm, để các nhà phát triển có thể tiếp tục sử dụng các phiên bản bộ biên dịch được build bên ngoài để biên dịch JIT.

## JEP 411: Deprecate the Security Manager for Removal (Đánh dấu loại bỏ Security Manager để chuẩn bị xóa)

Loại bỏ Security Manager để xóa trong các phiên bản tương lai.

Security Manager có lịch sử từ Java 1.0, trong nhiều năm, nó không phải là phương pháp chính để bảo vệ mã Java client, cũng hiếm khi dùng để bảo vệ mã server. Để thúc đẩy Java phát triển tiến lên, Java 17 loại bỏ Security Manager, để xóa cùng với Applet API cũ ([JEP 398](https://openjdk.java.net/jeps/398)).

## JEP 412: Foreign Function & Memory API (Hàm ngoại và Memory API, Lần ươm tạo thứ 1)

Các chương trình Java có thể thông qua API này để tương tác với mã và dữ liệu bên ngoài Java runtime. Thông qua việc gọi hàm ngoại (tức mã ngoài JVM) hiệu quả và truy cập bộ nhớ ngoài (tức bộ nhớ không do JVM quản lý) an toàn, API này giúp chương trình Java có thể gọi thư viện native và xử lý dữ liệu native, chứ không nguy hiểm và dễ vỡ như JNI.

Foreign Function & Memory API thực hiện vòng ươm tạo thứ nhất trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Vòng ươm tạo thứ hai do [JEP 419](https://openjdk.org/jeps/419) đề xuất và tích hợp vào Java 18, bản xem trước do [JEP 424](https://openjdk.org/jeps/424) đề xuất và tích hợp vào Java 19.

Trong bài [Tổng quan tính năng mới Java 19](./java19.md), tôi đã giới thiệu chi tiết về Foreign Function & Memory API, ở đây không giới thiệu thêm nữa.

## JEP 414: Vector API (Vector API, Lần ươm tạo thứ 2)

Vector API ban đầu do [JEP 338](https://openjdk.java.net/jeps/338) đề xuất, và được tích hợp vào Java 16 dưới dạng Incubator API. Vòng ươm tạo thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và tích hợp vào Java 17, vòng ươm tạo thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và tích hợp vào Java 18, vòng thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và tích hợp vào Java 19.

API ươm tạo này cung cấp một vòng lặp ban đầu của API để biểu thị các tính toán vector, các tính toán này khi runtime được biên dịch một cách đáng tin cậy thành các lệnh phần cứng vector tối ưu trên kiến trúc CPU được hỗ trợ, từ đó đạt được hiệu năng vượt trội hơn so với các tính toán vô hướng (scalar) tương đương, tận dụng tối đa công nghệ Single Instruction Multiple Data (SIMD) (một loại lệnh có sẵn trên hầu hết các CPU hiện đại). Mặc dù HotSpot hỗ trợ tự động vector hóa, tuy nhiên tập hợp thao tác vô hướng có thể chuyển đổi bị hạn chế và dễ chịu ảnh hưởng bởi việc thay đổi mã nguồn. API này sẽ giúp các nhà phát triển dễ dàng sử dụng Java để viết các thuật toán vector hiệu năng cao có thể di trú.

Trong bài [Tổng quan tính năng mới Java 18](./java18.md), tôi đã giới thiệu chi tiết về Vector API, ở đây sẽ không giới thiệu thêm nữa.

<!-- @include: @article-footer.snippet.md -->
