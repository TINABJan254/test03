---
title: Chi tiết I/O Multiplexing: Nguyên lý và sự khác biệt giữa select, poll, epoll
description: Phân tích chuyên sâu về 5 mô hình I/O trong hệ điều hành, làm rõ hai giai đoạn của Network I/O, cơ chế hoạt động của select, poll, epoll, giải quyết bài toán C10K và phân biệt Level-Triggered (LT) vs Edge-Triggered (ET).
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
  - I/O Multiplexing
  - Lập trình mạng
head:
  - - meta
    - name: keywords
      content: I/O Multiplexing, Đa ghép kênh I/O, select, poll, epoll, Level-Triggered, Edge-Triggered, C10K, Netty, Redis, Nginx, Blocking I/O, Non-blocking I/O
---

Tại sao **Redis** là một tiến trình đơn luồng (Single-threaded) nhưng lại có thể xử lý hơn 100,000 requests mỗi giây? Tại sao **Nginx** và **Netty** có thể duy trì hàng triệu kết nối đồng thời với lượng RAM và CPU cực kỳ thấp?

Bí mật cốt lõi nằm ở cơ chế **I/O Multiplexing (Đa ghép kênh I/O)**, đặc biệt là cơ chế **`epoll`** trên Linux.

Bài viết này sẽ giải thích chi tiết toàn bộ kiến trúc của I/O Multiplexing.

---

## 1. Hai giai đoạn của Network I/O

Một thao tác đọc dữ liệu từ Socket mạng (`read()`) luôn trải qua 2 giai đoạn:

1. **Giai đoạn 1: Chờ dữ liệu sẵn sàng trên mạng (Wait for Data Ready)**: Dữ liệu từ mạng Internet truyền tới Card mạng, Card mạng kích hoạt DMA chuyển dữ liệu vào Socket Receive Buffer trong Kernel.
2. **Giai đoạn 2: Sao chép dữ liệu từ Kernel sang User Space (Copy Data from Kernel to User Buffer)**: CPU sao chép dữ liệu từ bộ đệm Kernel vào mảng byte của chương trình ứng dụng.

---

## 2. Năm mô hình I/O kinh điển trong Hệ điều hành

![Năm mô hình I/O trong hệ điều hành](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/five-io-models-comparison.png)

1. **Blocking I/O (I/O chặn truyền thống)**: Luồng gọi `read()` và bị treo (ngủ) ở cả 2 giai đoạn cho đến khi có dữ liệu.
2. **Non-blocking I/O (I/O không chặn)**: Luồng gọi `read()`, nếu chưa có dữ liệu thì Kernel trả về lỗi `EAGAIN` / `EWOULDBLOCK` ngay lập tức. Ứng dụng phải liên tục chạy vòng lặp bận (Busy Polling) hỏi lại -> Tốn CPU vô ích.
3. **I/O Multiplexing (Đa ghép kênh I/O - select/poll/epoll)**: Một luồng duy nhất gọi `epoll_wait()` để giám sát trạng thái của hàng nghìn Socket. Khi có ít nhất một Socket có dữ liệu, kernel sẽ đánh thức luồng dậy để xử lý.
4. **Signal-driven I/O (I/O hướng tín hiệu)**: Cài đặt Signal Handler `SIGIO`. Khi có dữ liệu, Kernel phát tín hiệu báo cho ứng dụng.
5. **Asynchronous I/O (AIO - I/O bất đồng bộ hoàn toàn)**: Ứng dụng khởi tạo thao tác I/O rồi tiếp tục làm việc khác; Kernel tự động làm cả 2 giai đoạn và thông báo khi dữ liệu đã nằm sẵn trong User Buffer (như IOCP trên Windows, io_uring trên Linux mới).

---

## 3. So sánh chi tiết: `select`, `poll` và `epoll`

![So sánh select, poll và epoll](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/select-poll-epoll-comparison.png)

### 1. `select` (Ra đời năm 1983)
- Sử dụng mảng Bitmap `fd_set` có kích thước cố định (mặc định giới hạn tối đa **1024 File Descriptors**).
- **Quy trình**: Mỗi lần gọi, ứng dụng phải sao chép toàn bộ mảng `fd_set` từ User sang Kernel. Kernel duyệt vòng lặp $O(N)$ toàn bộ mảng để kiểm tra. Khi có sự kiện, Kernel trả về mảng đã sửa; ứng dụng lại phải chạy vòng lặp $O(N)$ duyệt lại từ đầu để tìm xem fd nào có dữ liệu.
- *Độ phức tạp*: $O(N)$. Kém hiệu quả khi số lượng kết nối lớn.

### 2. `poll` (Ra đời năm 1997)
- Sử dụng mảng cấu trúc danh sách liên kết `struct pollfd`.
- **Cải tiến**: Loại bỏ giới hạn 1024 kết nối của `select`.
- **Nhược điểm vẫn còn**: Vẫn phải sao chép mảng từ User sang Kernel và vẫn phải duyệt vòng lặp $O(N)$ mỗi lần kiểm tra.

### 3. `epoll` (Ra đời trong Linux 2.6 - Vua hiệu năng)
- `epoll` giải quyết triệt để các điểm nghẽn của `select`/`poll` bằng 3 hàm API thông minh:
  1. `epoll_create()`: Tạo một đối tượng epoll trong kernel.
  2. `epoll_ctl()`: Đăng ký, sửa, xóa Socket cần theo dõi.
  3. `epoll_wait()`: Chờ các sự kiện I/O xảy ra.

#### Cơ chế kỳ diệu bên trong của `epoll`:
- **Quản lý Socket bằng Cây đỏ đen (Red-Black Tree)**: Mỗi Socket chỉ cần đăng ký vào cây đỏ đen 1 lần qua `epoll_ctl()` (thao tác thêm/xóa $O(\log N)$), không bao giờ phải sao chép lại toàn bộ danh sách mỗi lần gọi như select/poll.
- **Cơ chế Callback ngắt phần cứng**: Khi một Socket có dữ liệu đến từ mạng, trình điều khiển Card mạng kích hoạt hàm Callback để đưa Socket đó vào **Danh sách liên kết sẵn sàng (Ready List)**.
- **Trả về danh sách $O(1)$**: Khi ứng dụng gọi `epoll_wait()`, Kernel chỉ kiểm tra Ready List. Nếu có phần tử, Kernel trả về chính xác danh sách các Socket đang có dữ liệu; ứng dụng xử lý ngay lập tức mà không phải duyệt vòng lặp $O(N)$!

| Tiêu chí | `select` | `poll` | `epoll` |
| --- | --- | --- | --- |
| **Giới hạn số lượng kết nối** | Cố định 1024 (trên hệ thống 32-bit) | Không giới hạn (bị giới hạn bởi File Descriptor của OS) | Không giới hạn (chỉ phụ thuộc vào dung lượng RAM) |
| **Cấu trúc dữ liệu trong Kernel** | Mảng Bitmap cố định | Danh sách liên kết `pollfd` | **Cây đỏ đen (RB-Tree) + Danh sách sẵn sàng (Ready List)** |
| **Sao chép dữ liệu giữa User & Kernel** | Sao chép toàn bộ mỗi lần gọi | Sao chép toàn bộ mỗi lần gọi | **Chỉ truyền khi thêm/xóa bằng `epoll_ctl`** |
| **Độ phức tạp thời gian** | $O(N)$ | $O(N)$ | **$O(1)$ (Chỉ xử lý các kết nối thực sự có sự kiện)** |

---

## 4. Phân biệt Level-Triggered (LT) và Edge-Triggered (ET) trong epoll

- **Level-Triggered (LT - Kích hoạt theo mức / Mặc định)**:
  - Miễn là trong Socket Receive Buffer còn dữ liệu chưa đọc hết, mỗi lần gọi `epoll_wait()` sẽ **liên tục thông báo sự kiện**.
  - Lập trình đơn giản, an toàn, khó bị sót dữ liệu.
- **Edge-Triggered (ET - Kích hoạt theo cạnh viền / Hiệu năng cao)**:
  - Chỉ thông báo sự kiện **đúng một lần duy nhất** khi trạng thái của Socket thay đổi (từ không có dữ liệu thành có dữ liệu mới đến).
  - *Yêu cầu bắt buộc khi dùng ET*:
    1. Socket bắt buộc phải cấu hình ở chế độ **Non-blocking (Không chặn)**.
    2. Khi nhận được thông báo sự kiện, ứng dụng phải **dùng vòng lặp `while` gọi `read()` liên tục cho đến khi gặp lỗi `EAGAIN` / `EWOULDBLOCK`** để đọc sạch toàn bộ dữ liệu trong bộ đệm (nếu không dữ liệu còn sót lại sẽ bị kẹt vĩnh viễn!).
  - Được Nginx và Netty sử dụng để đạt hiệu năng tối đa.

<!-- @include: @article-footer.snippet.md -->
