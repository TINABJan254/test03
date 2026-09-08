---
title: Chi tiết Tiến trình và Luồng: Phân biệt, Trạng thái, Giao tiếp, Context Switch và Virtual Thread
description: Tổng hợp câu hỏi phỏng vấn tần suất cao về Tiến trình và Luồng, từ góc nhìn Hệ điều hành làm rõ khái niệm, mô hình tài nguyên, chuyển đổi trạng thái, PCB/TCB, fork/exec/wait, mô hình luồng, Context Switch và mối quan hệ với Virtual Thread trong Java.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Tiến trình và Luồng
  - Java Concurrency
head:
  - - meta
    - name: keywords
      content: Tiến trình, Luồng, Phân biệt tiến trình và luồng, Trạng thái tiến trình, Trạng thái luồng, PCB, TCB, fork, exec, wait, clone, pthread, Context Switch, Mô hình luồng, Java Virtual Thread, Phỏng vấn Hệ điều hành
---

Tiến trình (Process) và Luồng (Thread) là hai khái niệm nền tảng nhất trong hệ điều hành, nhưng cũng là hai khái niệm dễ bị học vẹt máy móc nhất.

Khi phỏng vấn về sự khác biệt giữa chúng, nhiều câu trả lời chỉ dừng lại ở câu: *"Tiến trình là đơn vị cơ bản để phân bổ tài nguyên, còn luồng là đơn vị cơ bản để CPU lập lịch"*. Câu nói này đúng để mở đầu, nhưng hoàn toàn chưa đủ.

Khi người phỏng vấn tiếp tục hỏi sâu: Tại sao các tiến trình mặc định bị cô lập với nhau? Các luồng chia sẻ những tài nguyên gì? Sau khi gọi `fork()`, tiến trình cha và con có những gì giống nhau và những gì tách biệt? Tại sao gọi `fork()` trong chương trình đa luồng lại dễ sinh lỗi? Virtual Thread trong Java có phải là luồng của hệ điều hành không?

Bài viết này sẽ làm sáng tỏ toàn bộ các vấn đề trên theo một mạch tư duy xuyên suốt: **Tiến trình đại diện cho ranh giới tài nguyên và sự cô lập, còn Luồng đại diện cho một luồng thực thi được CPU lập lịch.**

---

## 1. Chương trình, Tiến trình và Luồng là gì?

![Mối quan hệ giữa Chương trình, Tiến trình và Luồng](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/relationship-between-program-process-and-thread.png)

- **Chương trình (Program)**: Là một tập hợp các chỉ lệnh và dữ liệu tĩnh được lưu trữ trên ổ đĩa (ví dụ một file thực thi `.exe`, một file `.jar`). Nó chưa chạy, chỉ là một file tĩnh.
- **Tiến trình (Process)**: Khi hệ điều hành nạp chương trình vào bộ nhớ RAM, thiết lập không gian địa chỉ ảo, bảng mô tả tệp (File Descriptor Table) và cấp phát tài nguyên, một thực thể đang chạy được hình thành - đó là **Tiến trình**. Cùng một chương trình có thể chạy nhiều lần tạo thành nhiều tiến trình độc lập (ví dụ mở 2 cửa sổ Chrome hoặc Terminal).
- **Luồng (Thread)**: Là một dòng thực thi bên trong tiến trình. Một tiến trình có ít nhất một luồng (Main Thread). Các luồng trong cùng một tiến trình chia sẻ chung toàn bộ tài nguyên của tiến trình đó, nhưng mỗi luồng sở hữu riêng ngữ cảnh thực thi của mình (Ngăn xếp Stack, Tập thanh ghi Registers, Con trỏ lệnh Program Counter). Hệ điều hành thực sự lập lịch và phân bổ thời gian CPU cho các Luồng.

> **Tóm gọn**: Tiến trình đại diện cho **Ranh giới tài nguyên**, Luồng đại diện cho **Thực thi và Lập lịch**.

![Ẩn dụ về nhà máy để phân biệt tiến trình và luồng](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/wechat-factory-process-thread.png)

---

## 2. Tiến trình sở hữu những tài nguyên nào?

Một tiến trình sở hữu các tài nguyên sau:
- **Không gian địa chỉ ảo (Virtual Address Space)**: Bao gồm Đoạn mã (Code/Text Segment), Đoạn dữ liệu (Data Segment), Bộ nhớ Heap, Ngăn xếp Stack, Vùng ánh xạ bộ nhớ (Memory Mapped Region).
- **Bảng tệp đang mở (File Descriptor Table)**: Quản lý các file, Socket mạng, Pipe, thiết bị ngoại vi đang mở.
- **Thông tin bảo mật và định danh**: User ID (UID), Group ID (GID), quyền hạn bảo mật.
- **Ngữ cảnh chạy**: Biến môi trường (Environment Variables), Thư mục làm việc hiện tại (Working Directory), Bộ xử lý tín hiệu (Signal Handlers).

**Các tiến trình mặc định hoàn toàn cô lập với nhau**: Tiến trình A không thể tự ý đọc/ghi vào không gian bộ nhớ của tiến trình B. Sự cô lập này đảm bảo an toàn cho hệ điều hành, nhưng cũng làm cho việc giao tiếp (IPC) và chuyển đổi tiến trình tốn nhiều chi phí hơn so với luồng.

---

## 3. Các trạng thái của Tiến trình

Mô hình 5 trạng thái kinh điển của tiến trình:

![Sơ đồ chuyển đổi trạng thái của tiến trình](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/state-transition-of-process.png)

1. **Mới tạo (New / Created)**: Tiến trình đang được khởi tạo, chưa vào hàng đợi sẵn sàng.
2. **Sẵn sàng (Ready)**: Đã có đủ mọi tài nguyên cần thiết, chỉ chờ CPU cấp phát thời gian để chạy.
3. **Đang chạy (Running)**: Đang thực thi lệnh trên CPU.
4. **Bị chặn / Chờ (Blocked / Waiting)**: Đang chờ một sự kiện nào đó hoàn thành (chờ I/O đĩa, nhận dữ liệu mạng Socket, chờ giải phóng Khóa Lock, Timer).
5. **Kết thúc (Terminated / Exit)**: Tiến trình chạy xong hoặc bị hủy, hệ điều hành thu hồi tài nguyên.

---

## 4. PCB và TCB là gì?

- **PCB (Process Control Block - Khối điều khiển tiến trình)**: Cấu trúc dữ liệu trong Kernel lưu trữ toàn bộ thông tin quản lý của một tiến trình (PID, trạng thái, con trỏ thanh ghi CPU, thông tin lập lịch, con trỏ bảng trang bộ nhớ, danh sách file descriptor). Trong Linux, PCB được biểu diễn bằng cấu trúc `struct task_struct`.
- **TCB (Thread Control Block - Khối điều khiển luồng)**: Cấu trúc dữ liệu lưu trữ thông tin riêng của từng luồng (TID, trạng thái luồng, tập thanh ghi riêng, con trỏ đỉnh ngăn xếp Stack Pointer, độ ưu tiên).

Khi xảy ra **Context Switch (Chuyển đổi ngữ cảnh)**, hệ điều hành sẽ lưu trạng thái thanh ghi CPU của luồng hiện tại vào TCB/PCB, sau đó nạp trạng thái thanh ghi của luồng tiếp theo từ TCB/PCB vào CPU để tiếp tục chạy.

---

## 5. Các hàm `fork()`, `exec()`, `wait()` trong Linux

![Chuỗi gọi hàm fork, exec, wait](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/fork-exec-wait-call-chain.png)

Trong lập trình Unix/Linux:
- **`fork()`**: Tạo ra một tiến trình con mới bằng cách nhân bản tiến trình cha. Tiến trình cha và con tiếp tục chạy từ cùng vị trí lệnh, chỉ khác giá trị trả về: `fork()` trả về PID của con cho tiến trình cha, và trả về `0` cho tiến trình con. Linux áp dụng cơ chế **Copy-on-Write (COW)**: Ban đầu cha và con dùng chung các trang bộ nhớ vật lý dạng Read-Only; chỉ khi một trong hai bên thực hiện ghi (Write) dữ liệu, Kernel mới thực sự sao chép trang bộ nhớ đó sang vùng mới.
- **`exec()` (như `execve`)**: Thay thế toàn bộ không gian địa chỉ, mã lệnh, dữ liệu, heap, stack của tiến trình hiện tại bằng một chương trình mới. (Mô hình của Shell: Shell gọi `fork()` tạo tiến trình con, tiến trình con gọi `exec()` để chạy lệnh người dùng nhập).
- **`wait()` / `waitpid()`**: Tiến trình cha gọi để chờ tiến trình con kết thúc và thu hồi mã trạng thái thoát của con. Nếu con kết thúc mà cha chưa gọi `wait()`, tiến trình con trở thành **Zombie Process (Tiến trình ma)** chiếm giữ một PID trong bảng tiến trình.

---

## 6. Context Switch (Chuyển đổi ngữ cảnh) là gì?

Context Switch là quá trình CPU tạm dừng thực thi luồng/tiến trình hiện tại, lưu lại ngữ cảnh phần cứng (Registers, PC, SP) và nạp ngữ cảnh của luồng/tiến trình khác vào để chạy.

### Phân biệt Context Switch giữa Tiến trình và Luồng
- **Chuyển đổi giữa 2 tiến trình khác nhau**:
  - Tốn kém nhiều chi phí vì phải **chuyển đổi không gian địa chỉ ảo (đổi con trỏ Bảng trang CR3)**.
  - Việc đổi bảng trang làm vô hiệu hóa bộ nhớ đệm chuyển đổi địa chỉ **TLB (Translation Lookaside Buffer)**, khiến các truy cập bộ nhớ sau đó bị chậm do miss TLB.
- **Chuyển đổi giữa 2 luồng trong cùng tiến trình**:
  - Không cần đổi không gian địa chỉ ảo, không cần đổi bảng trang, không làm mất hiệu lực TLB.
  - Chỉ cần lưu và nạp lại tập thanh ghi và ngăn xếp Stack riêng của luồng. Chi phí nhẹ hơn rất nhiều!

---

## 7. Mối quan hệ giữa Luồng Hệ điều hành và Virtual Thread trong Java

- **Luồng Java truyền thống (Platform Thread)**: Có quan hệ **$1:1$ với Luồng Kernel của Hệ điều hành (OS Thread)**. Mỗi khi bạn `new Thread()`, JVM yêu cầu OS tạo một Kernel Thread tương ứng. Nhược điểm: Bộ nhớ Stack của OS thread tốn khoảng 1MB, chi phí Context Switch qua Kernel đắt đỏ, một máy chủ chỉ tạo được vài nghìn thread là cạn kiệt tài nguyên.
- **Java Virtual Thread (Project Loom - Java 21+)**: Là luồng mức người dùng (**User-mode Thread / Coroutine**), có quan hệ **$M:N$**. Hàng triệu Virtual Thread có thể chạy trên một số lượng nhỏ các Carrier Thread (OS Thread).
  - Khi một Virtual Thread gặp thao tác I/O bị chặn (Blocking I/O), JVM tự động tháo dỡ (unmount) nó khỏi Carrier Thread để nhường chỗ cho Virtual Thread khác chạy, mà không làm block OS Thread bên dưới.
  - Bộ nhớ của Virtual Thread siêu nhẹ (chỉ vài trăm byte trên Heap), giúp Java dễ dàng xử lý hàng triệu request đồng thời theo mô hình lập trình đồng bộ truyền thống (Thread-per-request).

<!-- @include: @article-footer.snippet.md -->
