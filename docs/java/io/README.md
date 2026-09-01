---
title: Java IO Chuyên đề: BIO, NIO, AIO, Mô hình IO và Design Pattern
description: Lộ trình học Java IO và NIO, bao gồm BIO, NIO, AIO, blocking/non-blocking, đồng bộ/bất đồng bộ, I/O multiplexing, mô hình Reactor và các design pattern IO.
category: Java
tag:
  - Java
  - Java IO
  - Java面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java IO,Java NIO,BIO,NIO,AIO,IO模型,I/O多路复用,Reactor,Selector,Channel,Buffer,Java IO面试题
---

Java IO là nền tảng quan trọng để hiểu về đọc/ghi file, lập trình mạng, Netty, framework RPC và server hiệu năng cao. Khi học IO, bạn nên đồng thời tìm hiểu Java API, mô hình IO của hệ điều hành và các design pattern phổ biến, từ đó mới có thể kết nối được các khái niệm BIO, NIO, AIO, Selector, Channel, Buffer, Reactor với nhau.

## Dành cho ai

- Các lập trình viên backend muốn học hệ thống Java IO/NIO.
- Những bạn đang chuẩn bị câu hỏi phỏng vấn về BIO, NIO, AIO, I/O multiplexing, Reactor.
- Những độc giả muốn tiếp tục học Netty, RPC, message queue, database driver và các framework giao tiếp mạng khác.
- Các kỹ sư hay nhầm lẫn giữa các khái niệm blocking/non-blocking, đồng bộ/bất đồng bộ, Selector, Channel, Buffer.

## Trọng tâm học tập

- Hệ thống luồng IO Java, byte stream, character stream, buffered stream và các thao tác file phổ biến.
- Ứng dụng của các design pattern như Decorator, Adapter trong IO.
- Sự khác biệt về mô hình, trường hợp sử dụng, ưu và nhược điểm của BIO, NIO, AIO.
- Đồng bộ/bất đồng bộ, blocking/non-blocking, I/O multiplexing, Reactor và Proactor.
- Mối quan hệ phối hợp giữa Buffer, Channel, Selector và vai trò của chúng trong lập trình mạng.

## Thứ tự đọc đề xuất

1. [Tổng hợp kiến thức cơ bản Java IO](./io-basis.md): Trước tiên nắm vững hệ thống luồng IO, các lớp thường dùng và nền tảng đọc/ghi file.
2. [Tổng hợp design pattern Java IO](./io-design-patterns.md): Hiểu cách Decorator, Adapter và các design pattern khác được áp dụng vào IO API.
3. [Giải thích chi tiết mô hình Java IO](./io-model.md): Làm rõ BIO, NIO, AIO, đồng bộ/bất đồng bộ, blocking/non-blocking và multiplexing.
4. [Tổng hợp kiến thức cốt lõi Java NIO](./nio-basis.md): Học sâu về Buffer, Channel, Selector và mô hình lập trình NIO.

## Bài viết cốt lõi

- [Tổng hợp kiến thức cơ bản Java IO](./io-basis.md): Giới thiệu hệ thống byte stream, character stream, buffered stream, random access file và các lớp IO phổ biến.
- [Tổng hợp design pattern Java IO](./io-design-patterns.md): Giải thích ứng dụng của Decorator, Adapter và các design pattern khác trong IO.
- [Giải thích chi tiết mô hình Java IO](./io-model.md): Phân biệt BIO, NIO, AIO, đồng bộ/bất đồng bộ, blocking/non-blocking và I/O multiplexing.
- [Tổng hợp kiến thức cốt lõi Java NIO](./nio-basis.md): Hiểu về Buffer, Channel, Selector, SelectionKey và lập trình server NIO.

## Câu hỏi thường gặp

- Byte stream và character stream khác nhau như thế nào? Khi nào dùng buffered stream?
- Tại sao Java IO sử dụng rất nhiều Decorator pattern?
- BIO, NIO, AIO khác nhau như thế nào?
- Đồng bộ và bất đồng bộ, blocking và non-blocking có nghĩa là gì?
- I/O multiplexing giải quyết vấn đề gì?
- `select`, `poll`, `epoll` khác nhau như thế nào?
- Mô hình Reactor là gì? Khác gì so với Proactor?
- Buffer, Channel, Selector trong NIO đảm nhiệm vai trò gì?
- Tại sao Netty được xây dựng dựa trên NIO thay vì dùng BIO truyền thống?

## Chủ đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề lập trình đồng thời Java](../concurrent/)
- [Chuyên đề JVM](../jvm/)
- [Mạng máy tính](../../cs-basics/network/)
- [Netty](../../system-design/framework/netty.md)


<!-- @include: @article-footer.snippet.md -->
