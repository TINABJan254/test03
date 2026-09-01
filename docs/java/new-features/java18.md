---
title: Java 18 新特性概览
description: 概览 JDK 18 的更新与预览特性，理解新 API 带来的改进。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 18,JDK18,预览特性,API 更新,JEP
---

Java 18 phát hành chính thức vào ngày 22 tháng 3 năm 2022, là phiên bản không phải Hỗ trợ dài hạn.

JDK 18 có tổng cộng 8 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 400: UTF-8 by Default (UTF-8 làm bộ ký tự mặc định)](https://openjdk.java.net/jeps/400)
- [JEP 408: Simple Web Server (Web Server đơn giản)](https://openjdk.java.net/jeps/408)
- [JEP 413: Code Snippets in Java API Documentation (Đoạn mã mẫu trong tài liệu API)](https://openjdk.java.net/jeps/413)
- [JEP 416: Reimplement Core Reflection with Method Handles (Tái cấu trúc Reflection cốt lõi bằng Method Handles)](https://openjdk.java.net/jeps/416)
- [JEP 417: Vector API (Third Incubator) (Vector API, Lần ươm tạo thứ 3)](https://openjdk.java.net/jeps/417)
- [JEP 418: Internet-Address Resolution SPI (Phân giải địa chỉ Internet SPI)](https://openjdk.java.net/jeps/418)
- [JEP 419: Foreign Function & Memory API (Second Incubator) (Hàm ngoại và Memory API, Lần ươm tạo thứ 2)](https://openjdk.java.net/jeps/419)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Đọc thêm:

- [Tài liệu OpenJDK Java 18](https://openjdk.java.net/projects/jdk/18/)
- [IntelliJ IDEA | Hỗ trợ tính năng Java 18](https://mp.weixin.qq.com/s/PocFKR9z9u7-YCZHsrA5kQ)

## JEP 400: UTF-8 by Default (UTF-8 làm bộ ký tự mặc định, Chính thức)

JDK cuối cùng đã thiết lập UTF-8 làm bộ ký tự mặc định.

Trong Java 17 và các phiên bản sớm hơn, bộ ký tự mặc định chỉ được xác định khi Java virtual machine chạy, phụ thuộc vào các yếu tố như hệ điều hành khác nhau, thiết lập khu vực khác nhau, do đó tồn tại rủi ro tiềm ẩn. Ví dụ một đoạn chương trình Java in văn bản ra console chạy bình thường trên Mac nhưng sang Windows sẽ bị vỡ font chữ (lỗi hiển thị ký tự), nếu bạn không sửa đổi bộ ký tự thủ công.

## JEP 408: Simple Web Server (Web Server đơn giản, Chính thức)

Từ Java 18 trở đi, bạn có thể sử dụng câu lệnh `jwebserver` để khởi động một Web Server tĩnh đơn giản.

```bash
$ jwebserver
Binding to loopback by default. For all interfaces use "-b 0.0.0.0" or "-b ::".
Serving /cwd and subdirectories on 127.0.0.1 port 8000
URL: http://127.0.0.1:8000/
```

Server này không hỗ trợ CGI và Servlet, chỉ giới hạn cho các file tĩnh.

## JEP 413: Code Snippets in Java API Documentation (Đoạn mã mẫu trong tài liệu API, Chính thức)

Trước Java 18, nếu chúng ta muốn chèn đoạn mã vào Javadoc có thể sử dụng `<pre>{@code ...}</pre>`.

```java
<pre>{@code
    lines of source code
}</pre>
```

Hiệu quả tạo ra bởi cách `<pre>{@code ...}</pre>` này tương đối bình thường.

Từ Java 18 trở đi, có thể thông qua thẻ `@snippet` để làm điều này.

```java
/**
 * The following code shows how to use {@code Optional.isPresent}:
 * {@snippet :
 * if (v.isPresent()) {
 *     System.out.println("v: " + v.get());
 * }
 * }
 */
```

Cách `@snippet` này tạo ra hiệu quả tốt hơn và sử dụng tiện lợi hơn một chút.

## JEP 416: Reimplement Core Reflection with Method Handles (Tái cấu trúc Reflection cốt lõi bằng Method Handles, Chính thức)

Java 18 đã cải tiến logic triển khai của `java.lang.reflect.Method`, `Constructor`, giúp hiệu năng tốt hơn, tốc độ nhanh hơn. Thay đổi này không thay đổi API liên quan, điều này có nghĩa là trong quá trình phát triển không cần thay đổi mã nguồn liên quan đến Reflection, vẫn có thể trải nghiệm Reflection với hiệu năng tốt hơn.

Trang chủ OpenJDK cung cấp kết quả benchmark hiệu năng Reflection giữa triển khai mới và cũ.

![Kết quả benchmark hiệu năng Reflection giữa triển khai mới và cũ](https://oss.javaguide.cn/github/javaguide/java/new-features/JEP416Benchmark.png)

## JEP 417: Vector API (Vector API, Lần ươm tạo thứ 3)

Vector API ban đầu do [JEP 338](https://openjdk.java.net/jeps/338) đề xuất, và được tích hợp vào Java 16 dưới dạng Incubator API. Vòng ươm tạo thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và tích hợp vào Java 17, vòng ươm tạo thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và tích hợp vào Java 18, vòng thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và tích hợp vào Java 19.

Tính toán vector được cấu thành bởi một chuỗi các thao tác trên vector. Vector API dùng để biểu thị tính toán vector, tính toán này khi runtime có thể biên dịch một cách đáng tin cậy thành các lệnh vector tối ưu trên kiến trúc CPU được hỗ trợ, từ đó thực hiện hiệu năng vượt trội hơn so với tính toán vô hướng (scalar) tương đương.

Mục tiêu của Vector API là cung cấp cho người dùng tính toán vector biểu thị phạm vi rộng rãi, ngắn gọn dễ dùng và độc lập với nền tảng.

Đây là tính toán vô hướng đơn giản trên các phần tử mảng:

```java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}
```

Đây là tính toán vector tương đương sử dụng Vector API:

```java
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_PREFERRED;

void vectorComputation(float[] a, float[] b, float[] c) {
    int i = 0;
    int upperBound = SPECIES.loopBound(a.length);
    for (; i < upperBound; i += SPECIES.length()) {
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i);
        var vb = FloatVector.fromArray(SPECIES, b, i);
        var vc = va.mul(va)
                   .add(vb.mul(vb))
                   .neg();
        vc.intoArray(c, i);
    }
    for (; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}

```

Trong JDK 18, hiệu năng của Vector API tiếp tục được tối ưu hóa.

## JEP 418: Internet-Address Resolution SPI (Phân giải địa chỉ Internet SPI, Chính thức)

Java 18 định nghĩa một SPI (service-provider interface) hoàn toàn mới, dùng để phân giải tên và địa chỉ chính, để `java.net.InetAddress` có thể sử dụng các trình phân giải bên thứ ba ngoài nền tảng.

## JEP 419: Foreign Function & Memory API (Hàm ngoại và Memory API, Lần ươm tạo thứ 2)

Các chương trình Java có thể thông qua API này để tương tác với mã và dữ liệu bên ngoài Java runtime. Thông qua việc gọi hàm ngoại (tức mã ngoài JVM) hiệu quả và truy cập bộ nhớ ngoài (tức bộ nhớ không do JVM quản lý) an toàn, API này giúp chương trình Java có thể gọi thư viện native và xử lý dữ liệu native, chứ không nguy hiểm và dễ vỡ như JNI.

Foreign Function & Memory API thực hiện vòng ươm tạo thứ nhất trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Vòng ươm tạo thứ hai do [JEP 419](https://openjdk.org/jeps/419) đề xuất và tích hợp vào Java 18, bản xem trước do [JEP 424](https://openjdk.org/jeps/424) đề xuất và tích hợp vào Java 19.

Trong bài [Tổng quan tính năng mới Java 19](./java19.md), tôi đã giới thiệu chi tiết về Foreign Function & Memory API, ở đây không giới thiệu thêm nữa.

<!-- @include: @article-footer.snippet.md -->
