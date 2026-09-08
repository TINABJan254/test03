---
title: Chi tiết Giao tiếp liên tiến trình (IPC - Inter-Process Communication)
description: Phân tích toàn diện các cơ chế giao tiếp liên tiến trình trong hệ điều hành: Pipe (Đường ống ẩn danh/có tên), Message Queue, Shared Memory (Bộ nhớ chia sẻ), Semaphore, Signal, Socket mạng và Binder trong Android.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - IPC
head:
  - - meta
    - name: keywords
      content: IPC, Giao tiếp liên tiến trình, Pipe, Anonymous Pipe, Named Pipe, Message Queue, Shared Memory, Semaphore, Signal, Socket, Binder
---

Mỗi tiến trình trong hệ điều hành đều sở hữu một không gian địa chỉ ảo độc lập và được bảo vệ nghiêm ngặt.

Tuy nhiên, trong thực tế các tiến trình thường xuyên cần phải phối hợp, trao đổi dữ liệu hoặc đồng bộ trạng thái với nhau (ví dụ: lệnh pipe `cat file | grep text` trong Shell, giao tiếp giữa Web Server và Database, microservices trên cùng máy chủ).

Các cơ chế cho phép các tiến trình trao đổi dữ liệu với nhau được gọi chung là **IPC (Inter-Process Communication - Giao tiếp liên tiến trình)**.

Bài viết này sẽ phân tích chi tiết 7 phương thức IPC phổ biến nhất.

---

## 1. Pipe (Đường ống)

### 1. Anonymous Pipe (Đường ống vô danh / Đường ống ẩn danh)
- Được sử dụng rất phổ biến trong dòng lệnh Linux thông qua ký tự `|`:
  ```bash
  cat log.txt | grep "ERROR" | wc -l
  ```
- **Đặc điểm**:
  - Là kênh truyền dữ liệu **đơn công bán phần (Half-Duplex)**: Dữ liệu chỉ chảy theo một chiều từ đầu ghi sang đầu đọc.
  - Tồn tại trong bộ nhớ RAM (dưới dạng một file buffer đặc biệt trong VFS kernel), không có file thực trên đĩa.
  - **Chỉ dùng được giữa các tiến trình có quan hệ cha - con** (hoặc anh em) được tạo thông qua `fork()`. Khi tiến trình kết thúc, pipe tự động biến mất.

### 2. Named Pipe (FIFO - Đường ống có tên)
- Được tạo bằng lệnh `mkfifo my_pipe` hoặc hàm `mkfifo()`.
- Tồn tại dưới dạng một file đặc biệt loại `p` trên hệ thống tệp tin.
- **Cho phép 2 tiến trình bất kỳ (không cần quan hệ cha con) giao tiếp với nhau** bằng cách cùng mở file FIFO đó để đọc và ghi.

---

## 2. Message Queue (Hàng đợi thông điệp)

- Là một danh sách liên kết các thông điệp được lưu trữ và quản lý bên trong bộ nhớ của Kernel (như POSIX Message Queue hoặc System V MQ).
- **Ưu điểm**:
  - Dữ liệu được đóng gói thành từng **Message độc lập có kiểu (Type) và độ dài xác định**, tránh được vấn đề dính gói của Pipe dạng stream.
  - Hỗ trợ truyền bất đồng bộ: Bên gửi ghi thông điệp vào hàng đợi rồi tiếp tục làm việc khác, bên nhận có thể đọc sau.
- **Nhược điểm**: Mỗi lần gửi/nhận đều phải sao chép dữ liệu giữa User space và Kernel space (mất 2 lần copy dữ liệu).

---

## 3. Shared Memory (Bộ nhớ chia sẻ - IPC nhanh nhất)

- Hai hoặc nhiều tiến trình cùng ánh xạ một vùng bộ nhớ vật lý chung vào không gian địa chỉ ảo của chính mình (sử dụng `shmget` / `shmat` hoặc `mmap`).
- **Ưu điểm vượt trội**: **Là phương thức IPC có tốc độ nhanh nhất!**
  - Dữ liệu được ghi trực tiếp vào RAM, tiến trình kia có thể nhìn thấy ngay lập tức mà **không cần thông qua Kernel sao chép dữ liệu (Zero Kernel Copy)**.
- **Nhược điểm**: Cần phải kết hợp thêm các cơ chế đồng bộ như **Semaphore** hoặc **Mutex** để tránh xung đột dữ liệu (Race Condition) khi nhiều tiến trình cùng ghi đồng thời.

---

## 4. Semaphore (Đèn hiệu)

- Semaphore thực chất không dùng để truyền khối lượng lớn dữ liệu, mà là một **bộ đếm nguyên (Integer Counter)** dùng để **Đồng bộ hóa (Synchronization)** và kiểm soát truy cập vào tài nguyên dùng chung giữa các tiến trình.
- Cung cấp 2 thao tác nguyên tử:
  - **P (wait / acquire)**: Giảm giá trị Semaphore đi 1. Nếu giá trị $< 0$, tiến trình bị chặn và đưa vào hàng đợi chờ.
  - **V (signal / release)**: Tăng giá trị Semaphore lên 1. Đánh thức một tiến trình đang bị chặn trong hàng đợi dậy.

---

## 5. Signal (Tín hiệu)

- Là cơ chế thông báo sự kiện bất đồng bộ duy nhất ở cấp độ hệ điều hành.
- Được dùng để thông báo cho một tiến trình biết một sự kiện khẩn cấp vừa xảy ra.
- *Ví dụ phổ biến*:
  - Nhấn `Ctrl + C` trên terminal gửi tín hiệu `SIGINT` (ngắt tiến trình).
  - Lệnh `kill -9 <pid>` gửi tín hiệu `SIGKILL` (ép buộc kernel chấm dứt tiến trình ngay lập tức).
  - Lệnh `kill -15 <pid>` gửi tín hiệu `SIGTERM` (yêu cầu tiến trình tự dọn dẹp và dừng êm đẹp - Graceful Shutdown).

---

## 6. Socket (Ổ cắm mạng)

- Là phương thức IPC duy nhất hỗ trợ **giao tiếp giữa các tiến trình trên các máy tính khác nhau qua mạng** (sử dụng TCP/IP hoặc UDP).
- Trong phạm vi cùng một máy chủ Linux, có thể sử dụng **Unix Domain Socket (UDS)**: Bỏ qua toàn bộ việc đóng gói giao thức TCP/IP, truyền dữ liệu trực tiếp trong bộ nhớ kernel, cho hiệu năng cao hơn nhiều so với Socket TCP loopback `127.0.0.1`.

---

## 7. Bảng so sánh tổng kết các phương thức IPC

| Phương thức IPC | Tốc độ | Phạm vi giao tiếp | Dạng dữ liệu | Cơ chế đồng bộ | Kịch bản phù hợp |
| --- | --- | --- | --- | --- | --- |
| **Anonymous Pipe** | Trung bình | Cha - Con trên cùng máy | Luồng byte (Byte Stream) | Tự động chặn khi đầy/rỗng | Lệnh nối tiếp trong Shell (`|`) |
| **Named Pipe (FIFO)** | Trung bình | Bất kỳ tiến trình nào cùng máy | Luồng byte | Tự động chặn | 2 tiến trình độc lập đơn giản |
| **Message Queue** | Trung bình | Cùng máy chủ | Thông điệp có cấu trúc | Kernel tự quản lý | Giao tiếp bất đồng bộ có ranh giới |
| **Shared Memory** | **Nhanh nhất** | Cùng máy chủ | Vùng nhớ tùy ý | **Cần kết hợp Semaphore/Lock** | Truyền tải dữ liệu lớn, hiệu năng cực cao |
| **Semaphore** | Nhanh | Cùng máy chủ | Bộ đếm (Số nguyên) | Nguyên tử (Atomic) | Đồng bộ, khóa tài nguyên chung |
| **Signal** | Nhanh | Cùng máy chủ | Mã tín hiệu số nguyên | Bất đồng bộ | Xử lý sự cố, ngắt/dừng tiến trình |
| **Socket / UDS** | Trung bình/Nhanh | Cùng máy hoặc **Khác máy qua mạng** | Byte Stream hoặc Datagram | Tùy chọn TCP/UDP | Giao tiếp mạng, Microservices |

<!-- @include: @article-footer.snippet.md -->
