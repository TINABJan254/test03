---
title: Chuyên đề Hệ điều hành: Tiến trình luồng, Quản lý bộ nhớ, File System, I/O Multiplexing, Linux và Shell
description: Lộ trình học tập và phỏng vấn Hệ điều hành, bao gồm Tiến trình & Luồng, Giao tiếp liên tiến trình (IPC), Khóa & Đồng bộ, Deadlock, Bộ nhớ ảo, Zero-Copy, I/O Multiplexing (select, poll, epoll), Hệ thống tệp tin, Linux cơ bản, Lập trình Shell và các câu hỏi phỏng vấn thường gặp.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
  - Shell
sidebar: false
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Hệ điều hành, Câu hỏi phỏng vấn Hệ điều hành, Tiến trình, Luồng, IPC, Khóa và đồng bộ, Mutex, Semaphore, Condition Variable, Futex, Deadlock, Quản lý bộ nhớ, Bộ nhớ ảo, Zero-Copy, I/O Multiplexing, select, poll, epoll, File System, Linux, Shell, Phỏng vấn Backend
---

**Chuyên đề Hệ điều hành** này hướng tới việc học tập và ôn tập phỏng vấn Backend, tổng hợp toàn diện các kiến thức nền tảng: Hệ điều hành cơ bản, Tiến trình & Luồng, Giao tiếp liên tiến trình (IPC), Khóa & Đồng bộ, Quản lý bộ nhớ, Bộ nhớ ảo, Zero-Copy, I/O Multiplexing, Hệ thống tệp tin, Linux và Shell.

## Phù hợp với ai

- Các lập trình viên Backend đang học Hệ điều hành một cách có hệ thống.
- Các bạn đang chuẩn bị cho các kỳ phỏng vấn Hệ điều hành của đợt tuyển dụng sinh viên (Campus), người có kinh nghiệm (Social), các công ty công nghệ lớn.
- Những độc giả chỉ học vẹt rời rạc các định nghĩa về Tiến trình, Luồng, Deadlock, Quản lý bộ nhớ, Lệnh Linux.
- Các kỹ sư muốn xây dựng nền tảng vững chắc cho Lập trình đa luồng (Java Concurrency), JVM, Cơ sở dữ liệu và Lập trình mạng.

## Trọng tâm học tập

- Hệ điều hành chịu trách nhiệm quản lý CPU, Bộ nhớ, File, I/O và Tiến trình, là nền tảng cốt lõi để hiểu cách phần mềm vận hành bên dưới.
- Tiến trình, Luồng và Giao tiếp liên tiến trình (IPC) là các khái niệm gốc rễ của lập trình bất đồng bộ, hiệu năng máy chủ và điều tra sự cố.
- Khóa & Đồng bộ, Deadlock, Chuyển đổi ngữ cảnh (Context Switch), Lập lịch CPU là các chủ đề phỏng vấn tần suất rất cao.
- Quản lý bộ nhớ, Bộ nhớ ảo, Phân trang (Paging), Thay thế trang (Page Replacement) giúp hiểu sâu cơ chế của JVM, Cơ sở dữ liệu và Cache.
- Zero-Copy, I/O Multiplexing (epoll) là chìa khóa để làm chủ các thành phần hiệu năng cao như Kafka, RocketMQ, Redis, Nginx, Netty.
- Linux và Shell là kỹ năng thực chiến không thể thiếu trong phát triển Backend, triển khai ứng dụng, xử lý sự cố và tự động hóa.

## Thứ tự đọc khuyến nghị

1. [Tổng hợp câu hỏi phỏng vấn Hệ điều hành thường gặp (Phần 1)](./operating-system-basic-questions-01.md): Thiết lập danh sách câu hỏi tần suất cao về nền tảng OS, Tiến trình, Luồng, Deadlock, Quản lý bộ nhớ.
2. [Tổng hợp câu hỏi phỏng vấn Hệ điều hành thường gặp (Phần 2)](./operating-system-basic-questions-02.md): Tiếp tục bổ sung File System, I/O, Linux.
3. [Chi tiết Tiến trình và Luồng: Phân biệt, Trạng thái, Giao tiếp, Context Switch và Virtual Thread](./process-and-thread.md): Hiểu có hệ thống về Process, Thread, PCB/TCB, fork/exec/wait, mô hình luồng và Context Switch.
4. [Chi tiết Ngắt, Ngoại lệ và System Call: Từ cửa ngõ Kernel đến Page Fault](./interrupt-exception-syscall.md): Lấy `read()` làm sợi dây xâu chuỗi ngắt phần cứng, ngoại lệ đồng bộ, system call, tín hiệu, page fault và chuyển đổi luồng.
5. [Chi tiết Lập lịch CPU và Tải hệ thống (CPU Load)](./cpu-scheduling-and-load.md): Hiểu các thuật toán lập lịch, CFS/EEVDF, load average, CPU utilization và tư duy điều tra sự cố production.
6. [Chi tiết Giao tiếp liên tiến trình (IPC): Pipe, Message Queue, Shared Memory, Socket và Binder](./ipc.md): So sánh các phương thức IPC phổ biến.
7. [Chi tiết Khóa và Cơ chế đồng bộ trong OS: Mutex, Semaphore, Condition Variable, Spinlock và Futex](./os-lock-and-sync.md): Hiểu ranh giới trách nhiệm của Critical Section, Mutex, Semaphore, Spinlock, Futex.
8. [Chi tiết Deadlock: Bốn điều kiện cần, Điều tra Deadlock Java và Xử lý Deadlock Database](./dead-lock.md): Làm rõ đồ thị phân bổ tài nguyên, 4 điều kiện cần của deadlock, công cụ dò tìm deadlock trong Java và retry transaction trong Database.
9. [Chi tiết Quản lý bộ nhớ trong OS: Phân trang, Phân đoạn, Thay thế trang, Swap và OOM](./memory-management.md): Hiểu cấp phát bộ nhớ, phân mảnh, bảng trang, thu hồi trang và cơ chế OOM.
10. [Chi tiết Bộ nhớ ảo: Chuyển đổi địa chỉ, TLB, Page Fault và Thay thế trang](./virtual-memory.md): Xâu chuỗi địa chỉ ảo, bảng trang, TLB, page fault và thuật toán thay thế trang.
11. [Chi tiết Hệ thống tệp tin trong OS: Inode, VFS, Page Cache và Cơ chế ghi nhật ký (Journaling)](./file-system.md): Hiểu tệp, thư mục, inode, VFS, Page Cache, fsync và phục hồi dữ liệu.
12. [Chi tiết I/O Multiplexing: Nguyên lý và sự khác biệt giữa select, poll, epoll](./io-multiplexing.md): Hiểu cơ chế nhân kernel xử lý hàng triệu kết nối đồng thời trên một luồng duy nhất.
13. [Chi tiết Zero-Copy: mmap, sendfile và splice](./zero-copy.md): Làm rõ các bước sao chép dữ liệu và kịch bản ứng dụng trong Java NIO, Kafka, Netty.
14. [Tổng hợp kiến thức Linux cơ bản](./linux-intro.md): Nắm vững cấu trúc thư mục, quyền tệp, các lệnh thông dụng và kỹ năng khắc phục sự cố.
15. [Tổng hợp kiến thức Lập trình Shell cơ bản](./shell-intro.md): Học biến, điều kiện, vòng lặp, hàm và cách viết script tự động hóa.

## Các bài viết cốt lõi

- [Tổng hợp câu hỏi phỏng vấn Hệ điều hành thường gặp (Phần 1)](./operating-system-basic-questions-01.md): Bao quát nền tảng OS, tiến trình, luồng, deadlock, quản lý bộ nhớ.
- [Tổng hợp câu hỏi phỏng vấn Hệ điều hành thường gặp (Phần 2)](./operating-system-basic-questions-02.md): Bao quát hệ thống tệp tin, I/O, I/O multiplexing, Linux.
- [Chi tiết Tiến trình và Luồng: Phân biệt, Trạng thái, Giao tiếp, Context Switch và Virtual Thread](./process-and-thread.md): Làm rõ ranh giới tài nguyên, chuyển đổi trạng thái, cơ chế tạo tiến trình trong Linux và Virtual Thread của Java.
- [Chi tiết Ngắt, Ngoại lệ và System Call: Từ cửa ngõ Kernel đến Page Fault](./interrupt-exception-syscall.md): Làm rõ mối quan hệ giữa ngắt phần cứng, ngoại lệ đồng bộ, system call, tín hiệu và page fault.
- [Chi tiết Lập lịch CPU và Tải hệ thống](./cpu-scheduling-and-load.md): Làm rõ lập lịch tác vụ, CFS/EEVDF, load average, CPU utilization và các lệnh điều tra sự cố.
- [Chi tiết Giao tiếp liên tiến trình (IPC)](./ipc.md): Nguyên lý, ưu nhược điểm và tiêu chí lựa chọn các phương thức IPC.
- [Chi tiết Khóa và Cơ chế đồng bộ trong OS](./os-lock-and-sync.md): Critical Section, Mutex, Semaphore, Condition Variable, Spinlock, Futex, Memory Ordering.
- [Chi tiết Deadlock](./dead-lock.md): Điều kiện hình thành deadlock, đồ thị tài nguyên, công cụ phân tích trong Java, deadlock trong Database và chiến lược retry.
- [Chi tiết Quản lý bộ nhớ trong OS](./memory-management.md): VSZ/RSS/PSS, cấp phát liên tục, phân mảnh bộ nhớ, Buddy System, bảng trang, TLB, page fault, thu hồi trang và OOM.
- [Chi tiết Bộ nhớ ảo](./virtual-memory.md): Địa chỉ ảo, địa chỉ vật lý, phân trang, bảng trang đa cấp, TLB, page fault và thuật toán thay thế trang (LRU, FIFO, Clock).
- [Chi tiết Hệ thống tệp tin trong OS](./file-system.md): File, thư mục, Inode, Dentry, File Descriptor, VFS, Page Cache, fsync và cơ chế Journaling.
- [Chi tiết I/O Multiplexing: select, poll, epoll](./io-multiplexing.md): Hai giai đoạn của Network I/O, 5 mô hình I/O và sự khác biệt giữa select, poll, epoll (LT vs ET).
- [Chi tiết Zero-Copy: mmap, sendfile và splice](./zero-copy.md): Zero-Copy tiết kiệm những thao tác sao chép nào và ứng dụng trong Java NIO, Kafka, RocketMQ.
- [Tổng hợp kiến thức Linux cơ bản](./linux-intro.md): Cây thư mục Linux, quyền tệp (chmod/chown), các lệnh thông dụng, quản lý user và tiến trình.
- [Tổng hợp kiến thức Lập trình Shell cơ bản](./shell-intro.md): Biến Shell, câu lệnh điều kiện, vòng lặp, hàm, xử lý văn bản (grep, sed, awk) và script thực chiến.

## Câu hỏi tần suất cao

- Tiến trình và Luồng khác nhau như thế nào? Các luồng trong cùng tiến trình chia sẻ những tài nguyên gì?
- Có những phương thức giao tiếp liên tiến trình (IPC) nào? Mỗi phương thức phù hợp với kịch bản nào?
- Context Switch (Chuyển đổi ngữ cảnh) là gì? Context Switch quá thường xuyên gây ảnh hưởng gì?
- Mutex, Semaphore, Condition Variable, Spinlock và Futex lần lượt giải quyết những bài toán nào?
- 4 điều kiện cần để hình thành Deadlock là gì? Cách điều tra và xử lý Deadlock trong Java và Database?
- Bộ nhớ ảo (Virtual Memory) là gì? Phân trang (Paging) và Phân đoạn (Segmentation) khác nhau ra sao?
- TLB, Page Fault và Thay thế trang lần lượt giải quyết bài toán gì?
- Zero-Copy tại sao lại nhanh? Phân biệt `mmap`, `sendfile` và `splice`?
- Inode, Hard Link và Soft Link trong File System là gì?
- Sự khác biệt cốt lõi giữa `select`, `poll` và `epoll` là gì? (Cơ chế `epoll` giải quyết điểm nghẽn C10K như thế nào?)
- Hiểu thế nào về quyền tệp tin trong Linux? Các lệnh điều tra sự cố CPU/RAM/I/O thông dụng?
- Shell Script phù hợp để giải quyết những tác vụ tự động hóa nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức Cơ sở máy tính](../)
- [Chuyên đề Mạng máy tính](../network/)
- [Chuyên đề Cấu trúc dữ liệu](../data-structure/)
- [Lập trình đa luồng trong Java (Java Concurrency)](../../java/concurrent/java-concurrent-questions-01.md)
- [Chi tiết các vùng nhớ trong JVM (JVM Memory Area)](../../java/jvm/memory-area.md)

<!-- @include: @article-footer.snippet.md -->
