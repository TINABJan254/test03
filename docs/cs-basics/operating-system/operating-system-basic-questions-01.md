---
title: Tổng hợp câu hỏi phỏng vấn Hệ điều hành thường gặp (Phần 1)
description: Tổng hợp chi tiết các câu hỏi phỏng vấn Hệ điều hành tần suất cao: Nền tảng OS, Tiến trình vs Luồng, IPC, Deadlock và Quản lý bộ nhớ.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
head:
  - - meta
    - name: keywords
      content: Phỏng vấn Hệ điều hành, Tiến trình, Luồng, Deadlock, Quản lý bộ nhớ, Phân trang, Bộ nhớ ảo, IPC
---

<!-- @include: @article-header.snippet.md -->

Phần 1 của bộ câu hỏi phỏng vấn Hệ điều hành tập trung vào các chủ đề: **Nền tảng Hệ điều hành, Tiến trình & Luồng, Giao tiếp liên tiến trình (IPC), Deadlock và Quản lý bộ nhớ**.

---

## 1. Khái niệm cơ bản về Hệ điều hành

### Câu 1: Hệ điều hành là gì và đảm nhận những chức năng cốt lõi nào?
- Hệ điều hành (OS) là phần mềm hệ thống trung gian quản lý toàn bộ tài nguyên phần cứng (CPU, RAM, Ổ đĩa, Thiết bị ngoại vi) và cung cấp môi trường trừu tượng, an toàn để các ứng dụng chạy trên đó.
- **5 chức năng cốt lõi**:
  1. Quản lý tiến trình/luồng và lập lịch CPU.
  2. Quản lý bộ nhớ (Memory Management).
  3. Quản lý hệ thống tệp tin (File System).
  4. Quản lý thiết bị I/O và Driver phần cứng.
  5. Quản lý bảo mật, phân quyền và giao diện người dùng (Shell/GUI).

### Câu 2: Phân biệt User Mode (Chế độ người dùng) và Kernel Mode (Chế độ nhân)?
- **User Mode (Ring 3)**: Ứng dụng thông thường chạy ở chế độ này, chỉ có quyền truy cập hạn chế vào không gian địa chỉ riêng của mình và không được phép thực thi các lệnh phần cứng đặc quyền.
- **Kernel Mode (Ring 0)**: Mã nhân của hệ điều hành chạy ở chế độ này, có toàn quyền tối cao truy cập toàn bộ bộ nhớ và điều khiển phần cứng.
- Chuyển đổi giữa 2 chế độ thông qua: **System Call, Ngắt phần cứng (Hardware Interrupt) hoặc Ngoại lệ (Exception)**.

---

## 2. Tiến trình và Luồng (Process & Thread)

### Câu 3: So sánh sự khác nhau giữa Tiến trình và Luồng?

| Tiêu chí | Tiến trình (Process) | Luồng (Thread) |
| --- | --- | --- |
| **Bản chất** | Đơn vị cơ bản để **phân bổ tài nguyên** | Đơn vị cơ bản để **CPU lập lịch thực thi** |
| **Không gian địa chỉ** | Độc lập, cô lập nghiêm ngặt | Chia sẻ chung không gian địa chỉ của tiến trình |
| **Tài nguyên riêng** | Address Space, File Descriptors, PID, Page Table | Stack, Registers, Program Counter (PC), TID |
| **Chi phí tạo & chuyển đổi** | Lớn (phải nạp bảng trang, làm mất hiệu lực TLB) | Rất nhẹ (chỉ lưu/nạp thanh ghi và stack) |
| **Giao tiếp** | Phải dùng IPC (Pipe, Shared Memory, Socket) | Đọc/ghi trực tiếp vào biến chung trong Heap |
| **Độ an toàn** | Một tiến trình crash không làm ảnh hưởng tiến trình khác | Một luồng bị lỗi bộ nhớ có thể làm sập toàn bộ tiến trình |

### Câu 4: Có những phương thức Giao tiếp liên tiến trình (IPC) nào?
1. **Pipe (Đường ống ẩn danh / có tên FIFO)**: Kênh truyền luồng byte đơn công.
2. **Message Queue (Hàng đợi thông điệp)**: Hàng đợi các message có cấu trúc trong kernel.
3. **Shared Memory (Bộ nhớ chia sẻ)**: Phương thức nhanh nhất do không qua kernel copy dữ liệu.
4. **Semaphore (Đèn hiệu)**: Dùng để đồng bộ và khóa tài nguyên dùng chung.
5. **Signal (Tín hiệu)**: Cơ chế thông báo sự kiện bất đồng bộ.
6. **Socket (Ổ cắm mạng)**: Hỗ trợ giao tiếp xuyên qua mạng Internet.

---

## 3. Deadlock (Bế tắc)

### Câu 5: Bốn điều kiện cần để xảy ra Deadlock là gì?
1. **Loại trừ tương hỗ (Mutual Exclusion)**
2. **Giữ và chờ (Hold and Wait)**
3. **Không chiếm đoạt (No Preemption)**
4. **Chờ đợi vòng tròn (Circular Wait)**

### Câu 6: Làm thế nào để phòng ngừa và khắc phục Deadlock trong thực tế?
- **Phòng ngừa**: Quy định tất cả các luồng bắt buộc phải xin khóa theo **thứ tự tăng dần của ID tài nguyên** (phá vỡ điều kiện Chờ đợi vòng tròn).
- **Trong Java**: Dùng `tryLock(timeout)` để tự nhả khóa khi chờ quá lâu; dùng lệnh `jstack <pid>` để dò tìm deadlock.
- **Trong Database (MySQL)**: InnoDB tự động phát hiện chu trình deadlock và tự động rollback một transaction có chi phí nhỏ nhất.

---

## 4. Quản lý bộ nhớ và Bộ nhớ ảo

### Câu 7: Bộ nhớ ảo (Virtual Memory) là gì và mang lại lợi ích gì?
- Là kỹ thuật cho phép mỗi tiến trình nhìn thấy một không gian bộ nhớ ảo liên tục và độc lập.
- **Lợi ích**: Cô lập an toàn giữa các tiến trình, cho phép chạy các chương trình có dung lượng lớn hơn cả dung lượng RAM vật lý thực tế nhờ cơ chế phân trang và swap.

### Câu 8: Page Fault (Ngoại lệ lỗi trang) là gì?
- Khi CPU truy cập một địa chỉ ảo mà trang đó chưa được nạp vào RAM vật lý (`Present Bit = 0`), CPU phát sinh ngoại lệ Page Fault chuyển quyền cho Kernel.
- Kernel cấp phát khung trang RAM, nạp dữ liệu từ ổ đĩa vào RAM, cập nhật Bảng trang rồi cho CPU chạy lại chỉ lệnh ban đầu.

<!-- @include: @article-footer.snippet.md -->
