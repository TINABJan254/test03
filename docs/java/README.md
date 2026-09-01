---
title: Java 知识体系：基础、集合、并发、JVM、IO 与新特性
description: Java 面试与知识体系学习路线，涵盖 Java 基础、集合源码、并发编程、JVM、IO/NIO 和 Java 新特性，适合校招、社招和 Java 后端面试复习。
category: Java
tag:
  - Java
  - Java基础
  - Java面试
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: Java,Java基础,Java集合,Java并发,JVM,Java IO,Java NIO,Java新特性,Java面试题,Java后端面试
---

<!-- @include: @small-advertisement.snippet.md -->

Bộ **Hệ thống kiến thức Java** này hướng đến việc học và ôn tập phỏng vấn Java backend, được sắp xếp theo thứ tự "Cú pháp cơ bản -> Container tập hợp (Collection) -> Lập trình đồng thời (Concurrency) -> IO/NIO -> JVM -> Tính năng mới".

Nếu thời gian của bạn có hạn, nên xem trước tổng hợp câu hỏi phỏng vấn về Java cơ bản, Collection, Concurrency và JVM để nhanh chóng thiết lập danh sách câu hỏi tần suất cao; nếu muốn củng cố nền tảng một cách hệ thống, bạn có thể đọc theo thứ tự chuyên đề bên dưới.

## Phù hợp với ai

- Lập trình viên backend đang học Java một cách hệ thống.
- Các bạn sinh viên/lập trình viên đang chuẩn bị cho các buổi phỏng vấn Java backend tuyển dụng sinh viên (校招), tuyển dụng người có kinh nghiệm (社招), hoặc tại các công ty vừa và lớn.
- Độc giả muốn kết nối và ôn tập tổng hợp Java cơ bản, Collection, Concurrency, JVM, IO và các tính năng mới.
- Kỹ sư đã làm dự án Java nhưng chưa hiểu hệ thống về nguyên lý tầng dưới, thiết kế mã nguồn (source code) và thực tiễn kỹ thuật.

## Trọng tâm học tập

- Cú pháp cơ bản Java, hướng đối tượng (OOP), Exception, Generics, Reflection, Proxy, Serialization và các cơ chế cốt lõi khác.
- Ranh giới sử dụng, triển khai mã nguồn và các câu hỏi phỏng vấn phổ biến về List, Map, Queue, Container đồng thời (Concurrent Collection).
- Thread Java, Lock, JMM, CAS, AQS, Thread Pool, CompletableFuture và Virtual Thread.
- Vùng nhớ JVM, Class Loading, Garbage Collection (GC), cấu hình tham số, công cụ giám sát và truy vết sự cố trên môi trường thực tế (production).
- Các mô hình BIO, NIO, AIO, IO, cũng như các Design Pattern liên quan đến IO như Decorator, Adapter.
- Các tính năng mới quan trọng từ Java 8 đến Java 26, và những tính năng nào thực sự ảnh hưởng đến công việc lập trình hàng ngày.

## Thứ tự đọc đề xuất

1. [Chuyên đề Java Cơ bản](./basis/): Nắm vững cú pháp cơ bản, hướng đối tượng, Generics, Reflection, Proxy, Serialization,...
2. [Chuyên đề Java Collection](./collection/): Hiểu cách sử dụng và mã nguồn của các container phổ biến như ArrayList, LinkedList, HashMap, ConcurrentHashMap.
3. [Chuyên đề Java Concurrency](./concurrent/): Học hệ thống về Thread, Lock, JMM, CAS, AQS, Thread Pool và các lớp tiện ích đồng thời.
4. [Chuyên đề JVM](./jvm/): Hiểu về vùng nhớ, Class Loading, Garbage Collection, tham số JVM và truy vết sự cố production.
5. [Chuyên đề Java IO](./io/): Bổ sung kiến thức về BIO, NIO, AIO, Reactor, I/O multiplexing và các Design Pattern trong IO.
6. [Chuyên đề Java Tính năng mới](./new-features/): Phân loại theo phiên bản các tính năng then chốt như Lambda, Stream, Module system, var, Record, Virtual Thread,...

## Các bài viết cốt lõi

### Java Cơ bản

- [Chuyên đề Java Cơ bản](./basis/): Đi từ cú pháp cơ bản đến cơ chế cốt lõi và các câu hỏi phỏng vấn Java thường gặp.
- [Tổng hợp câu hỏi phỏng vấn Java cơ bản thường gặp (Phần 1)](./basis/java-basic-questions-01.md): Bao gồm đặc trưng ngôn ngữ Java, cú pháp cơ bản, hướng đối tượng và các lớp thường dùng.
- [Tổng hợp câu hỏi phỏng vấn Java cơ bản thường gặp (Phần 2)](./basis/java-basic-questions-02.md): Tiếp tục tổng hợp Exception, Generics, Reflection, Annotation và các chi tiết thường gặp.
- [Tổng hợp câu hỏi phỏng vấn Java cơ bản thường gặp (Phần 3)](./basis/java-basic-questions-03.md): Bổ sung kiến thức cơ bản nâng cao và các điểm dễ mắc lỗi.
- [Giải thích chi tiết Truyền giá trị (Pass-by-value) trong Java](./basis/why-there-only-value-passing-in-java.md): Làm rõ mối quan hệ giữa truyền giá trị, biến tham chiếu và việc sửa đổi đối tượng.
- [Giải thích chi tiết Java Serialization](./basis/serialization.md): Hiểu cơ chế Serialization, serialVersionUID, rủi ro bảo mật và giải pháp thay thế.
- [Giải thích chi tiết Java Reflection](./basis/reflection.md) và [Giải thích chi tiết Pattern Proxy trong Java](./basis/proxy.md): Nắm vững các cơ chế tầng dưới phổ biến trong framework.

### Java Collection

- [Chuyên đề Java Collection](./collection/): Chuỗi bài viết kết nối Collection framework, lưu ý khi sử dụng và phân tích mã nguồn phổ biến.
- [Tổng hợp câu hỏi phỏng vấn Java Collection thường gặp (Phần 1)](./collection/java-collection-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn Java Collection thường gặp (Phần 2)](./collection/java-collection-questions-02.md): Bao gồm các vấn đề tần suất cao về List, Set, Map, Queue và Concurrent Collection.
- [Tổng hợp lưu ý khi sử dụng Java Collection](./collection/java-collection-precautions-for-use.md): Tổng hợp các lưu ý liên quan đến kiểm tra rỗng, duyệt (traversal), mở rộng dung lượng (capacity expansion), thread safety và hiệu năng của Collection.
- [Phân tích mã nguồn ArrayList](./collection/arraylist-source-code.md), [Phân tích mã nguồn HashMap](./collection/hashmap-source-code.md), [Phân tích mã nguồn ConcurrentHashMap](./collection/concurrent-hash-map-source-code.md): Hiểu sự đánh đổi trong thiết kế container phổ biến từ góc độ mã nguồn.

### Java Concurrency

- [Chuyên đề Java Concurrency](./concurrent/): Xoay quanh Thread, Lock, Memory Model, Thread Pool và các công cụ đồng thời.
- [Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (Phần 1)](./concurrent/java-concurrent-questions-01.md)、[Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (Phần 2)](./concurrent/java-concurrent-questions-02.md)、[Tổng hợp câu hỏi phỏng vấn Java Concurrency thường gặp (Phần 3)](./concurrent/java-concurrent-questions-03.md): Xây dựng danh sách câu hỏi phỏng vấn về Concurrency.
- [Giải thích chi tiết JMM (Java Memory Model)](./concurrent/jmm.md): Hiểu về Visibility, Atomicity, Ordering và happens-before.
- [Giải thích chi tiết CAS](./concurrent/cas.md), [Giải thích chi tiết AQS](./concurrent/aqs.md), [Giải thích chi tiết Java Thread Pool](./concurrent/java-thread-pool-summary.md): Nắm vững các trọng tâm tầng dưới của lập trình đồng thời.
- [Tổng hợp câu hỏi thường gặp về Virtual Thread](./concurrent/virtual-thread.md): Hiểu tác động của Project Loom đối với mô hình đồng thời.

### JVM và IO

- [Chuyên đề JVM](./jvm/): Xoay quanh vùng nhớ, Class Loading, GC, tham số, công cụ và truy vết sự cố production.
- [Giải thích chi tiết Vùng nhớ Java (Trọng tâm)](./jvm/memory-area.md): Hiểu về Program Counter, JVM Stack, Native Method Stack, Heap và Method Area.
- [Giải thích chi tiết Garbage Collection trong JVM (Trọng tâm)](./jvm/jvm-garbage-collection.md): Hiểu về việc xác định đối tượng còn sống, thuật toán thu gom rác và các Collector chủ đạo.
- [Giải thích chi tiết quá trình Class Loading](./jvm/class-loading-process.md) và [Giải thích chi tiết ClassLoader (Trọng tâm)](./jvm/classloader.md): Nắm vững vòng đời của Class và mô hình Parents Delegation.
- [Chuyên đề Java IO](./io/): Đi từ BIO, NIO, AIO đến các mô hình IO và Design Pattern trong IO.
- [Tổng hợp kiến thức cơ bản về Java IO](./io/io-basis.md), [Tổng hợp kiến thức cốt lõi về Java NIO](./io/nio-basis.md), [Giải thích chi tiết mô hình Java IO](./io/io-model.md): Bổ sung kiến thức tiền đề cho việc học lập trình mạng và middleware.

### Java Tính năng mới

- [Chuyên đề Java Tính năng mới](./new-features/): Phân loại theo phiên bản các tính năng ngôn ngữ, thư viện chuẩn và JVM quan trọng từ Java 8 trở đi.
- [Thực hành tính năng mới Java 8](./new-features/java8-common-new-features.md): Nắm vững Lambda, Stream, Optional, phương thức mặc định trong Interface và Date API mới.
- [Tổng quan tính năng mới Java 11 (Quan trọng)](./new-features/java11.md), [Tổng quan tính năng mới Java 17 (Quan trọng)](./new-features/java17.md), [Tổng quan tính năng mới Java 21 (Quan trọng)](./new-features/java21.md): Ưu tiên chú ý đến các tính năng khả dụng lâu dài trong phiên bản LTS.

## Câu hỏi tần suất cao

- Tại sao Java là truyền giá trị (Pass-by-value)? Rốt cuộc điều gì xảy ra khi truyền tham chiếu đối tượng dưới dạng tham số?
- `String`, `StringBuilder`, `StringBuffer` có điểm gì khác nhau?
- `equals()` và `hashCode()` có mối quan hệ như thế nào?
- Chọn `ArrayList` hay `LinkedList` như thế nào? Tại sao `HashMap` không an toàn với luồng (thread-safe)?
- `ConcurrentHashMap` có những thay đổi gì giữa JDK 7 và JDK 8?
- `synchronized` và `ReentrantLock` khác nhau như thế nào?
- JMM làm thế nào để đảm bảo tính hiển thị (Visibility), tính tuần tự (Ordering) và tính nguyên tử (Atomicity)?
- Cấu hình các tham số cốt lõi của Thread Pool như thế nào? Tại sao không nên trực tiếp sử dụng `Executors`?
- Vùng nhớ JVM được phân chia như thế nào? Vùng nào có thể xảy ra OOM?
- G1, ZGC, Shenandoah phù hợp với kịch bản nào?
- BIO, NIO, AIO khác nhau như thế nào? Mô hình Reactor giải quyết vấn đề gì?
- Trong Java 8, 11, 17, 21 những tính năng mới nào đáng để nắm vững nhất?

## Các chuyên đề liên quan

- [Cơ sở máy tính](../cs-basics/)
- [Thiết kế hệ thống](../system-design/)
- [Cơ sở dữ liệu](../database/)
- [Hệ thống kiến thức Hệ thống phân tán](../distributed-system/)
- [Hệ thống kiến thức Hệ thống hiệu năng cao](../high-performance/)

<!-- @include: @article-footer.snippet.md -->
