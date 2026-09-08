---
title: Một máy chủ tối đa có thể duy trì bao nhiêu kết nối TCP? (Tầng giao vận)
description: Phân tích các yếu tố giới hạn số lượng kết nối TCP tối đa trên một máy chủ, làm rõ vai trò của bộ tứ 4-tuple, File Descriptor, bộ nhớ RAM, CPU và các thông số kernel cần tối ưu.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Kết nối TCP, Số lượng kết nối tối đa, 4-tuple, File Descriptor, C10K, C1000K, Tối ưu hóa Linux
---

Một câu hỏi phỏng vấn kinh điển về hệ thống mạng và lập trình mạng hiệu năng cao:

**"Một máy chủ (Server) tối đa có thể duy trì bao nhiêu kết nối TCP cùng một lúc? Có phải tối đa chỉ là 65,535 kết nối không?"**

Câu trả lời ngắn gọn: **Con số 65,535 là giới hạn số cổng của Client khi kết nối tới một Server cụ thể, còn phía Server có thể duy trì hàng trăm nghìn, thậm chí hàng triệu kết nối TCP đồng thời (vấn đề C1000K), giới hạn thực tế phụ thuộc chủ yếu vào Bộ nhớ (RAM) và File Descriptor (FD).**

Bài viết này sẽ phân tích chi tiết từ lý thuyết đến thực tế.

---

## 1. Giới hạn lý thuyết: Bộ 4 thông số (4-Tuple)

Một kết nối TCP được định danh duy nhất bởi 4 thông số:

$$	ext{4-Tuple} = (	ext{IP nguồn}, 	ext{Cổng nguồn}, 	ext{IP đích}, 	ext{Cổng đích})$$

Đối với một máy chủ Server đang lắng nghe trên một cổng cố định (ví dụ Server IP: `1.2.3.4`, Cổng: `80`):
- `IP đích` cố định: `1.2.3.4`
- `Cổng đích` cố định: `80`
- `IP nguồn`: Có thể là bất kỳ IP nào của các Client trên Internet.
- `Cổng nguồn`: Mỗi Client có thể sử dụng các cổng từ 1 đến 65535 ($2^{16} - 1$).

Do đó, về mặt lý thuyết toán học:

$$	ext{Số kết nối tối đa} = 	ext{Số lượng IP Client} 	imes 	ext{Số lượng Port Client} pprox 2^{32} 	imes 2^{16} pprox 2^{48} pprox 280	ext{ nghìn tỷ kết nối!}$$

Rõ ràng con số 65,535 không phải là giới hạn của Server.

---

## 2. Các giới hạn thực tế trên hệ điều hành Linux

Trong thực tế, số lượng kết nối TCP trên một máy chủ bị giới hạn bởi các tài nguyên phần cứng và cấu hình hệ điều hành:

### Giới hạn 1: File Descriptor (Số lượng tệp mở tối đa)

Trong Linux, **"Mọi thứ đều là tệp" (Everything is a file)**. Mỗi kết nối TCP (Socket) khi được tạo ra tương ứng với một File Descriptor (FD).

Hệ điều hành mặc định giới hạn số lượng FD để bảo vệ tài nguyên:
- **Giới hạn trên mỗi tiến trình (User limit)**: Mặc định thường là 1024. Có thể kiểm tra bằng `ulimit -n`.
- **Giới hạn trên toàn hệ thống (System limit)**: Kiểm tra qua `cat /proc/sys/fs/file-max`.

Để hỗ trợ hàng triệu kết nối, quản trị viên cần tăng giới hạn này trong `/etc/security/limits.conf` và `/etc/sysctl.conf`:
```bash
# Tăng giới hạn số file descriptor
fs.file-max = 2097152
```

### Giới hạn 2: Bộ nhớ RAM (Tài nguyên quan trọng nhất)

Mỗi kết nối TCP tiêu tốn một lượng bộ nhớ RAM nhất định trong Kernel để duy trì:
- Cấu trúc dữ liệu `struct sock` (khoảng 1-2 KB).
- Bộ đệm gửi (Send Buffer - `wmem`) và Bộ đệm nhận (Receive Buffer - `rmem`).

Mặc định, một kết nối TCP có thể được cấp phát từ vài KB đến vài chục KB bộ đệm.
- Nếu mỗi kết nối tiêu tốn trung bình **3 KB RAM**: 1 triệu kết nối (1M connections) sẽ tốn khoảng **3 GB RAM**.
- Nếu mỗi kết nối tiêu tốn **10 KB RAM**: 1 triệu kết nối sẽ tốn khoảng **10 GB RAM**.

Ngày nay với các máy chủ 32GB / 64GB / 128GB RAM, việc duy trì **1 triệu đến 2 triệu kết nối TCP đồng thời** là hoàn toàn khả thi và phổ biến (như các hệ thống Push Notification, Gateway, Chat Server).

### Giới hạn 3: Mô hình xử lý I/O và CPU

Để xử lý hàng triệu kết nối đồng thời mà không làm sập CPU:
- Không thể dùng mô hình truyền thống "Mỗi kết nối một Thread/Process" (sẽ cạn kiệt bộ nhớ và chi phí Context Switch làm tê liệt CPU).
- Bắt buộc phải sử dụng mô hình **I/O Multiplexing (Đa ghép kênh I/O)** hướng sự kiện bất đồng bộ như **`epoll`** trên Linux (được triển khai trong Netty, Nginx, Redis).

---

## 3. Tổng kết

- Giới hạn lý thuyết của Server: Gần như vô hạn (dựa trên 4-tuple).
- Giới hạn của Client khi gọi tới 1 Server: Khoảng 65,535 kết nối (do giới hạn cổng nguồn của Client).
- Giới hạn thực tế của Server: Phụ thuộc vào **Bộ nhớ RAM** (dung lượng bộ đệm TCP) và cấu hình **File Descriptor (`fs.file-max`, `ulimit -n`)** cùng mô hình xử lý **`epoll`**.

<!-- @include: @article-footer.snippet.md -->
