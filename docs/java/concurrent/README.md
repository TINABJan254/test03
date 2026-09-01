---
title: Java Lập Trình Đồng Thời: Thread, Lock, JMM, CAS, AQS, Thread Pool và Virtual Thread
description: Lộ trình học và phỏng vấn về lập trình đồng thời Java, bao gồm Thread, Lock, synchronized, ReentrantLock, JMM, CAS, AQS, ThreadLocal, Thread Pool, CompletableFuture và Virtual Thread.
category: Java
tag:
  - Java
  - Java并发
  - Java面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java并发,Java锁,synchronized,ReentrantLock,JMM,CAS,AQS,ThreadLocal,线程池,CompletableFuture,并发容器,Atomic,虚拟线程
---

Lập trình đồng thời Java là một trong những module quan trọng nhất và dễ gây nhầm lẫn nhất trong phát triển backend và phỏng vấn. Học lập trình đồng thời không chỉ đơn giản là thuộc lòng API, mà cần hiểu vòng đời Thread, cơ chế Lock, mô hình bộ nhớ, thao tác nguyên tử, Thread Pool và các công cụ đồng thời trong cùng một mạch kiến thức liên kết.

## Dành cho ai

- Lập trình viên backend muốn học lập trình đồng thời Java một cách hệ thống.
- Những bạn đang chuẩn bị phỏng vấn về Thread, Lock, JMM, CAS, AQS, Thread Pool, v.v.
- Lập trình viên đã sử dụng đa luồng trong dự án nhưng chưa nắm vững các vấn đề về deadlock, tham số Thread Pool, rò rỉ bộ nhớ ThreadLocal, v.v.
- Kỹ sư muốn hiểu về concurrent container, CompletableFuture, Virtual Thread trong thực tế.

## Trọng tâm học tập

- Tạo Thread, vòng đời Thread, chuyển ngữ cảnh, thread safety và các vấn đề đồng thời phổ biến.
- Phạm vi áp dụng của `synchronized`, `volatile`, `ReentrantLock`, khóa loại trừ lẫn nhau, khóa đọc-ghi, optimistic lock, pessimistic lock.
- JMM, happens-before, sắp xếp lại lệnh, visibility, atomicity và ordering.
- Tư tưởng cốt lõi của CAS, lớp nguyên tử Atomic, AQS, concurrent container và blocking queue.
- Tham số cốt lõi Thread Pool, chiến lược từ chối, hàng đợi tác vụ, cấu hình tham số và thực tiễn sản xuất.
- Cách sử dụng và rủi ro của CompletableFuture, ThreadLocal, Virtual Thread trong dự án thực tế.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Java Concurrent (Phần 1)](./java-concurrent-questions-01.md): Xây dựng danh sách câu hỏi nền tảng về Thread, Lock và thread safety.
2. [Tổng hợp câu hỏi phỏng vấn Java Concurrent (Phần 2)](./java-concurrent-questions-02.md) và [Tổng hợp câu hỏi phỏng vấn Java Concurrent (Phần 3)](./java-concurrent-questions-03.md): Tiếp tục bổ sung JMM, CAS, AQS, Thread Pool và công cụ đồng thời.
3. [Giải thích chi tiết Java Lock](./java-lock.md), [Giải thích Optimistic Lock và Pessimistic Lock](./optimistic-lock-and-pessimistic-lock.md), [Giải thích CAS](./cas.md), [Giải thích JMM (Java Memory Model)](./jmm.md): Xây dựng hệ thống Lock trước, rồi hiểu ngữ nghĩa cốt lõi của kiểm soát đồng thời.
4. [Giải thích AQS](./aqs.md), [Tìm hiểu nguyên lý và ứng dụng AQS qua ReentrantLock](./reentrantlock.md): Hiểu sâu về Lock Java và synchronizer.
5. [Giải thích chi tiết Java Thread Pool](./java-thread-pool-summary.md) và [Thực tiễn tốt nhất với Java Thread Pool](./java-thread-pool-best-practices.md): Nắm vững cơ sở hạ tầng đồng thời phổ biến nhất trong sản xuất.

## Bài viết cốt lõi

### Câu hỏi phỏng vấn về đồng thời

- [Tổng hợp câu hỏi phỏng vấn Java Concurrent (Phần 1)](./java-concurrent-questions-01.md): Bao gồm nền tảng Thread, thread safety, Lock và các vấn đề đồng thời phổ biến.
- [Tổng hợp câu hỏi phỏng vấn Java Concurrent (Phần 2)](./java-concurrent-questions-02.md): Tiếp tục tổng hợp các kiến thức cốt lõi như JMM, volatile, CAS, AQS.
- [Tổng hợp câu hỏi phỏng vấn Java Concurrent (Phần 3)](./java-concurrent-questions-03.md): Bổ sung Thread Pool, công cụ đồng thời, CompletableFuture và Virtual Thread.

### Lock, mô hình bộ nhớ và Synchronizer

- [Giải thích chi tiết Java Lock](./java-lock.md): Từ khóa loại trừ lẫn nhau, khóa đọc-ghi, spin lock đến `synchronized`, `ReentrantLock`, AQS, xây dựng hệ thống Lock Java.
- [Giải thích Optimistic Lock và Pessimistic Lock](./optimistic-lock-and-pessimistic-lock.md): Hiểu các chiến lược xử lý xung đột đồng thời khác nhau.
- [Giải thích CAS](./cas.md): Hiểu compare-and-swap, vấn đề ABA và chi phí spin.
- [Giải thích JMM (Java Memory Model)](./jmm.md): Nắm vững visibility, atomicity, ordering và happens-before.
- [Giải thích AQS](./aqs.md): Hiểu sync queue, chế độ độc quyền/chia sẻ và cơ chế bên dưới của các synchronizer phổ biến.
- [Tìm hiểu nguyên lý và ứng dụng AQS qua ReentrantLock](./reentrantlock.md): Hiểu sâu AQS thông qua ReentrantLock.

### Công cụ đồng thời và thực tiễn kỹ thuật

- [Giải thích chi tiết Java Thread Pool](./java-thread-pool-summary.md): Hiểu tham số cốt lõi, hàng đợi tác vụ, chiến lược từ chối và luồng thực thi.
- [Thực tiễn tốt nhất với Java Thread Pool](./java-thread-pool-best-practices.md): Tổng hợp khuyến nghị về cô lập Thread Pool, cấu hình tham số và giám sát trong môi trường sản xuất.
- [Tổng hợp các Concurrent Container phổ biến trong Java](./java-concurrent-collections.md): Tổng hợp các container như ConcurrentHashMap, CopyOnWriteArrayList, BlockingQueue.
- [Tổng hợp lớp Atomic](./atomic-classes.md): Hiểu cập nhật nguyên tử kiểu cơ bản, mảng, tham chiếu và trường.
- [Giải thích ThreadLocal](./threadlocal.md): Hiểu biến cục bộ của Thread, ThreadLocalMap và rủi ro rò rỉ bộ nhớ.
- [Giải thích CompletableFuture](./completablefuture-intro.md): Nắm vững lập lịch bất đồng bộ, xử lý ngoại lệ và sử dụng Thread Pool.
- [Tổng hợp câu hỏi phổ biến về Virtual Thread](./virtual-thread.md): Hiểu vị trí, trường hợp áp dụng và giới hạn sử dụng của Virtual Thread.

## Câu hỏi thường gặp

- Thread và tiến trình khác nhau như thế nào? Thread có những trạng thái nào?
- Thread safety là gì? Làm thế nào để xác định deadlock?
- Khóa loại trừ lẫn nhau, khóa đọc-ghi, spin lock khác nhau như thế nào?
- `synchronized` và `ReentrantLock` khác nhau như thế nào?
- `volatile` có đảm bảo tính nguyên tử không? Nó giải quyết vấn đề gì?
- JMM là gì? Quy tắc happens-before có tác dụng gì?
- CAS có ưu nhược điểm gì? Vấn đề ABA được giải quyết như thế nào?
- Tư tưởng cốt lõi của AQS là gì? Những công cụ nào dựa trên AQS?
- Làm thế nào để cấu hình tham số cốt lõi của Thread Pool? Chọn chiến lược từ chối như thế nào?
- Tại sao không nên trực tiếp dùng `Executors` để tạo Thread Pool?
- Tại sao `ThreadLocal` có thể bị rò rỉ bộ nhớ?
- Thread Pool mặc định của CompletableFuture có rủi ro gì?
- Virtual Thread có phù hợp với tác vụ CPU-intensive không?

## Chủ đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chủ đề Java Collection](../collection/)
- [Chủ đề JVM](../jvm/)
- [Chủ đề Java IO](../io/)
- [Hệ điều hành](../../cs-basics/operating-system/)


<!-- @include: @article-footer.snippet.md -->
