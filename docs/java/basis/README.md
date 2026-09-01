---
title: Java Cơ Bản: Cú pháp, Hướng đối tượng, Generics, Reflection, Proxy và Serialization
description: Lộ trình phỏng vấn và học tập Java cơ bản, bao gồm cú pháp cơ bản, hướng đối tượng, từ khóa, truyền giá trị, generics, reflection, proxy, serialization, SPI, Unsafe và syntactic sugar.
category: Java
tag:
  - Java
  - Java基础
  - Java面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java基础,Java基础面试题,Java关键字,Java值传递,Java泛型,Java反射,Java代理,Java序列化,Java SPI,Java Unsafe,Java语法糖
---

Java cơ bản là nền tảng tiên quyết để học tiếp về Collections, Concurrency, JVM, Spring và các middleware khác. Phần này không chỉ đơn thuần là ghi nhớ cú pháp, mà quan trọng hơn là hiểu rõ mô hình đối tượng của Java, cách truyền tham số, type erasure trong generics, gọi reflection, dynamic proxy, giới hạn serialization và cơ chế mở rộng của framework.

## Dành cho ai

- Người mới bắt đầu học Java một cách có hệ thống, muốn kết nối cú pháp cơ bản với các cơ chế cốt lõi.
- Các bạn đang chuẩn bị phỏng vấn Java cơ bản, muốn nhanh chóng bổ sung kiến thức còn thiếu.
- Lập trình viên đã làm dự án Java nhưng chưa hiểu sâu về reflection, proxy, generics, SPI, serialization.
- Kỹ sư muốn bổ sung kiến thức nền trước khi học sâu về Collections, Concurrency, JVM, Spring source code.

## Trọng tâm học tập

- Cú pháp cơ bản Java, hướng đối tượng, ngoại lệ, các lớp thông dụng, từ khóa và chi tiết coding.
- Mối quan hệ giữa truyền giá trị, biến tham chiếu, tính bất biến của đối tượng và lời gọi phương thức.
- Generics, wildcard, type erasure và ảnh hưởng của chúng đến collections, thiết kế API và reflection.
- Các cơ chế phổ biến ở tầng đáy của framework: reflection, dynamic proxy, SPI.
- Các kiến thức dễ gây lỗi trong dự án và phỏng vấn: serialization, `BigDecimal`, `Unsafe`, syntactic sugar.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Java cơ bản (Phần 1)](./java-basic-questions-01.md): Ôn lại cú pháp cơ bản Java, hướng đối tượng và các lớp thông dụng.
2. [Tổng hợp câu hỏi phỏng vấn Java cơ bản (Phần 2)](./java-basic-questions-02.md) và [Tổng hợp câu hỏi phỏng vấn Java cơ bản (Phần 3)](./java-basic-questions-03.md): Bổ sung kiến thức về ngoại lệ, generics, reflection, annotation và các điểm dễ nhầm lẫn.
3. [Tổng hợp từ khóa Java](./java-keyword-summary.md) và [Giải thích chi tiết về truyền giá trị trong Java](./why-there-only-value-passing-in-java.md): Làm rõ các hiểu lầm phổ biến về khái niệm cơ bản.
4. [Giải thích Generics & Wildcard](./generics-and-wildcards.md), [Giải thích chi tiết Reflection trong Java](./reflection.md), [Giải thích chi tiết Proxy Pattern trong Java](./proxy.md): Hiểu các khả năng phổ biến ở tầng đáy của framework.
5. [Giải thích chi tiết Serialization trong Java](./serialization.md), [Giải thích chi tiết SPI trong Java](./spi.md), [Giải thích chi tiết lớp ma thuật Unsafe trong Java](./unsafe.md): Tiếp tục bổ sung kiến thức mở rộng trong thực hành kỹ thuật và đọc source code.

## Bài viết cốt lõi

### Câu hỏi phỏng vấn cơ bản

- [Tổng hợp câu hỏi phỏng vấn Java cơ bản (Phần 1)](./java-basic-questions-01.md): Bao gồm đặc điểm ngôn ngữ Java, cú pháp cơ bản, hướng đối tượng, các lớp thông dụng và các điểm dễ nhầm lẫn.
- [Tổng hợp câu hỏi phỏng vấn Java cơ bản (Phần 2)](./java-basic-questions-02.md): Tiếp tục tổng hợp về ngoại lệ, generics, reflection, annotation và các khả năng cơ bản.
- [Tổng hợp câu hỏi phỏng vấn Java cơ bản (Phần 3)](./java-basic-questions-03.md): Bổ sung các câu hỏi phỏng vấn cơ bản thiên về chi tiết và nâng cao hơn.

### Cơ chế ngôn ngữ

- [Tổng hợp từ khóa Java](./java-keyword-summary.md): Giải thích rõ các từ khóa `final`, `static`, `this`, `super`.
- [Giải thích chi tiết về truyền giá trị trong Java](./why-there-only-value-passing-in-java.md): Giải thích tại sao Java chỉ có truyền giá trị và ngữ nghĩa thực sự khi truyền biến tham chiếu.
- [Giải thích Generics & Wildcard](./generics-and-wildcards.md): Hiểu cú pháp generics, upper/lower bound wildcard, type erasure và các tình huống sử dụng phổ biến.
- [Giải thích chi tiết Syntactic Sugar trong Java](./syntactic-sugar.md): Tìm hiểu cách compiler xử lý enhanced for, autoboxing/unboxing, enum, Lambda và các syntactic sugar khác.

### Cơ chế tầng đáy của framework

- [Giải thích chi tiết Reflection trong Java](./reflection.md): Hiểu đối tượng Class, gọi reflection, chi phí hiệu năng và các tình huống sử dụng.
- [Giải thích chi tiết Proxy Pattern trong Java](./proxy.md): Nắm vững static proxy, JDK dynamic proxy và CGLIB proxy.
- [Giải thích chi tiết SPI trong Java](./spi.md): Hiểu cơ chế service discovery và mở rộng plugin.
- [Giải thích chi tiết Serialization trong Java](./serialization.md): Hiểu quy trình serialization, serialVersionUID, rủi ro bảo mật và các phương án thay thế.

### Chi tiết thực tiễn

- [Giải thích chi tiết BigDecimal](./bigdecimal.md): Nắm vững các lưu ý về tính toán tiền tệ, độ chính xác, chế độ làm tròn và cách khởi tạo.
- [Dùng long hay BigDecimal cho tiền tệ trong Java?](./money-long-vs-bigdecimal.md): Phân biệt tình huống lưu trữ và tính toán tiền tệ, nắm vững đơn vị nhỏ nhất, làm tròn, tràn số và thiết kế trường trong database.
- [Giải thích chi tiết lớp ma thuật Unsafe trong Java](./unsafe.md): Tìm hiểu bộ nhớ off-heap, CAS, field offset của đối tượng và các khả năng tầng đáy trong source code.

## Câu hỏi thường gặp

- Java là truyền giá trị hay truyền tham chiếu? Tại sao khi truyền đối tượng làm tham số có thể thay đổi được field?
- `==` và `equals()` khác nhau như thế nào? Tại sao override `equals()` phải override `hashCode()`?
- Nên chọn `String`, `StringBuilder` hay `StringBuffer`?
- `final`, `static`, `this`, `super` có tác dụng gì?
- Type erasure là gì? `List<String>` và `List<Integer>` khác nhau gì lúc runtime?
- Tại sao reflection lại chậm? Có những tình huống sử dụng điển hình nào?
- JDK dynamic proxy và CGLIB dynamic proxy khác nhau như thế nào?
- Tại sao không nên dùng trực tiếp Java native serialization?
- Tại sao `BigDecimal` khuyến nghị khởi tạo bằng String?
- Nên dùng `long` lưu đơn vị nhỏ nhất hay dùng `BigDecimal` cho tiền tệ?

## Chủ đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chủ đề Collections Java](../collection/)
- [Chủ đề Lập trình đồng thời Java](../concurrent/)
- [Chủ đề JVM](../jvm/)
- [Spring](../../system-design/framework/spring/)


<!-- @include: @article-footer.snippet.md -->
