---
title: JVM Chuyên đề: Vùng nhớ, Class Loading, Garbage Collection, Tối ưu tham số và Xử lý sự cố
description: Lộ trình học phỏng vấn JVM và tối ưu hiệu năng, bao gồm vùng nhớ Java, cấu trúc class file, class loading, garbage collection, tham số JVM, công cụ giám sát JDK và xử lý sự cố trực tuyến.
category: Java
tag:
  - Java
  - JVM
  - Java面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: JVM,JVM面试题,Java内存区域,类加载,类加载器,垃圾回收,GC,JVM参数,JDK监控工具,OOM,性能调优
---

JVM là nền tảng cốt lõi mà các lập trình viên Java backend không thể bỏ qua. Mục tiêu học JVM không phải là ghi nhớ lý thuyết, mà là có thể giải thích được cách đối tượng được tạo và thu hồi, class được nạp như thế nào, GC ảnh hưởng đến ứng dụng ra sao, tham số cấu hình như thế nào, và cách xử lý các vấn đề OOM, Full GC liên tục, CPU tăng đột biến trên môi trường production.

## Dành cho ai

- Các lập trình viên Java backend muốn học hệ thống JVM.
- Những bạn đang chuẩn bị câu hỏi phỏng vấn về bộ nhớ JVM, class loading, GC, tối ưu tham số và xử lý sự cố trực tuyến.
- Những độc giả đã tham gia vận hành dịch vụ trực tuyến nhưng chưa quen với GC log, heap dump, thread stack và các công cụ JDK.
- Các kỹ sư muốn tiếp tục đào sâu Spring, Netty, middleware hoặc tối ưu hiệu năng.

## Trọng tâm học tập

- Vùng nhớ runtime JVM, tạo đối tượng, truy cập đối tượng và các tình huống OOM.
- Cấu trúc class file, quá trình class loading, ClassLoader và mô hình Parent Delegation.
- Cơ bản về Garbage Collection, phán đoán sự tồn tại của đối tượng, thuật toán GC và các garbage collector chính.
- Tham số JVM, GC log, heap dump, thread stack và các công cụ giám sát/chẩn đoán JDK phổ biến.
- Lộ trình cơ bản xử lý sự cố trực tuyến: quan sát hiện tượng, thu thập chỉ số, phân tích bằng công cụ, xác định nguyên nhân và xác minh tối ưu.

## Thứ tự đọc đề xuất

1. Khi chuẩn bị phỏng vấn, hãy xem trước [Tổng hợp câu hỏi phỏng vấn JVM phổ biến](./jvm-interview-questions.md), tìm các module mình trả lời chưa đầy đủ; học hệ thống có thể bắt đầu từ bước tiếp theo.
2. [Hiểu JVM theo cách dễ hiểu nhất](./jvm-intro.md): Xây dựng cái nhìn tổng quan về JVM trước.
3. [Giải thích chi tiết vùng nhớ Java (quan trọng)](./memory-area.md): Hiểu vùng dữ liệu runtime và các tình huống OOM phổ biến.
4. [Giải thích chi tiết cấu trúc class file](./class-file-structure.md), [Giải thích chi tiết quá trình class loading](./class-loading-process.md), [Giải thích chi tiết ClassLoader (quan trọng)](./classloader.md): Nắm vững quá trình từ `.class` đến đối tượng có thể chạy được.
5. [Giải thích chi tiết Garbage Collection JVM (quan trọng)](./jvm-garbage-collection.md): Học hệ thống về cơ bản GC, thuật toán và garbage collector.
6. [Tổng hợp tham số JVM quan trọng nhất](./jvm-parameters-intro.md), [Tổng hợp công cụ giám sát và xử lý sự cố JDK](./jdk-monitoring-and-troubleshooting-tools.md), [Xử lý sự cố trực tuyến Java backend](./jvm-in-action.md): Bước vào cấu hình tham số và thực hành trực tuyến.

## Bài viết cốt lõi

### JVM Cơ bản và Bộ nhớ

- [Tổng hợp câu hỏi phỏng vấn JVM phổ biến](./jvm-interview-questions.md): Tổng hợp câu hỏi tần suất cao theo bộ nhớ, class loading, GC, công cụ tham số và xử lý sự cố trực tuyến.
- [Hiểu JVM theo cách dễ hiểu nhất](./jvm-intro.md): Hiểu vị trí và thành phần của JVM theo góc độ tổng quan.
- [Giải thích chi tiết vùng nhớ Java (quan trọng)](./memory-area.md): Giải thích Program Counter Register, VM Stack, Native Method Stack, Heap, Method Area và Direct Memory.
- [Giải thích chi tiết cấu trúc class file](./class-file-structure.md): Hiểu magic number, version number, constant pool, access flags, field table, method table và attribute table.

### Class Loading

- [Giải thích chi tiết quá trình class loading](./class-loading-process.md): Hệ thống hóa loading, verification, preparation, resolution, initialization, using và unloading.
- [Giải thích chi tiết ClassLoader (quan trọng)](./classloader.md): Hiểu Bootstrap ClassLoader, Extension ClassLoader (JDK 8)/Platform ClassLoader (JDK 9+), Application ClassLoader và mô hình Parent Delegation.

### Garbage Collection và Tối ưu

- [Giải thích chi tiết Garbage Collection JVM (quan trọng)](./jvm-garbage-collection.md): Hiểu phán đoán sự tồn tại của đối tượng, kiểu tham chiếu, thuật toán GC, generational collection và các garbage collector chính.
- [Tổng hợp tham số JVM quan trọng nhất](./jvm-parameters-intro.md): Tổng hợp các tham số liên quan đến kích thước heap, GC log, OOM dump, garbage collector và chẩn đoán.
- [Tổng hợp công cụ giám sát và xử lý sự cố JDK](./jdk-monitoring-and-troubleshooting-tools.md): Giới thiệu các công cụ jps, jstat, jmap, jstack, jcmd, JConsole, VisualVM, JMC, v.v.
- [Xử lý sự cố trực tuyến Java backend](./jvm-in-action.md): Bắt đầu từ xác nhận cảnh báo, cầm máu và giữ bằng chứng, kết nối quá trình xử lý sự cố CPU, bộ nhớ, GC, Thread Pool, connection pool, slow SQL, Redis và message tồn đọng.

## Câu hỏi thường gặp

- Vùng nhớ runtime JVM được phân chia như thế nào? Vùng nào là riêng tư của từng Thread?
- Heap và Method Area lưu trữ gì? Direct Memory có thể xảy ra OOM không?
- Đối tượng được tạo ra như thế nào? Có mấy cách định vị truy cập đối tượng?
- Quá trình class loading có những giai đoạn nào? Giai đoạn initialization được kích hoạt khi nào?
- Mô hình Parent Delegation là gì? Tại sao cần nó?
- Làm sao phán đoán đối tượng có thể được thu hồi không? Strong, Soft, Weak, Phantom reference khác nhau thế nào?
- Minor GC, Major GC, Full GC khác nhau như thế nào?
- G1, ZGC, Shenandoah phù hợp với những tình huống nào?
- Các tham số JVM thường dùng là gì? Làm sao giữ lại GC log và OOM dump trên production?
- CPU tăng đột biến, Full GC liên tục, memory leak nên xử lý như thế nào?

## Chủ đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề Java cơ bản](../basis/)
- [Chuyên đề lập trình đồng thời Java](../concurrent/)
- [Chuyên đề Java IO](../io/)
- [Hệ điều hành](../../cs-basics/operating-system/)


<!-- @include: @article-footer.snippet.md -->
