---
title: Tổng hợp câu hỏi phỏng vấn Hệ điều hành thường gặp (Phần 2)
description: Tổng hợp chi tiết các câu hỏi phỏng vấn Hệ điều hành tần suất cao: I/O Multiplexing (select, poll, epoll), Zero-Copy, File System (Inode, Soft/Hard Link), Lập lịch CPU và lệnh Linux thực chiến.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
head:
  - - meta
    - name: keywords
      content: Phỏng vấn Hệ điều hành, I/O Multiplexing, select, poll, epoll, Zero-Copy, Inode, Hard Link, Soft Link, Page Cache, Linux
---

<!-- @include: @article-header.snippet.md -->

Phần 2 của bộ câu hỏi phỏng vấn Hệ điều hành tập trung vào: **I/O Multiplexing, Zero-Copy, Hệ thống tệp tin và Kỹ năng Linux thực chiến**.

---

## 1. I/O Multiplexing (Đa ghép kênh I/O)

### Câu 1: So sánh sự khác nhau giữa `select`, `poll` và `epoll`?

| Tiêu chí | `select` | `poll` | `epoll` |
| --- | --- | --- | --- |
| **Giới hạn kết nối** | Mặc định 1024 fd | Không giới hạn | Không giới hạn (chỉ phụ thuộc RAM) |
| **Cấu trúc dữ liệu trong Kernel** | Mảng Bitmap cố định | Danh sách liên kết `pollfd` | **Cây đỏ đen (RB-Tree) + Ready List** |
| **Sao chép dữ liệu User-Kernel** | Sao chép toàn bộ mỗi lần gọi | Sao chép toàn bộ mỗi lần gọi | **Chỉ truyền khi thêm/xóa bằng `epoll_ctl`** |
| **Độ phức tạp** | $O(N)$ | $O(N)$ | **$O(1)$** |

### Câu 2: Phân biệt Level-Triggered (LT) và Edge-Triggered (ET) trong epoll?
- **LT (Level-Triggered - Mặc định)**: Miễn là bộ đệm còn dữ liệu, `epoll_wait()` sẽ liên tục báo sự kiện. An toàn, dễ lập trình.
- **ET (Edge-Triggered - Cạnh viền)**: Chỉ báo sự kiện 1 lần duy nhất khi có dữ liệu mới đến. Yêu cầu Socket phải cấu hình Non-blocking và ứng dụng phải dùng vòng lặp `while` đọc cạn dữ liệu cho đến khi gặp lỗi `EAGAIN`. Hiệu năng cao hơn, được Nginx/Netty sử dụng.

---

## 2. Zero-Copy (Không sao chép dữ liệu)

### Câu 3: Zero-Copy là gì? Tại sao `sendfile` lại nhanh hơn đọc ghi truyền thống?
- **Truyền thống (`read` + `write`)**: Mất **4 lần Context Switch** và **4 lần Copy dữ liệu** (2 lần DMA, 2 lần CPU Copy qua User Space).
- **`sendfile` kết hợp DMA Gather Copy**: Mất **2 lần Context Switch** và **0 lần CPU Copy** (Dữ liệu truyền thẳng từ Page Cache của Kernel sang Card mạng qua DMA). Tiết kiệm tối đa CPU và băng thông bộ nhớ.
- **Ứng dụng**: Apache Kafka dùng `FileChannel.transferTo()` (bên dưới là `sendfile`) để đạt thông lượng hàng triệu message/giây.

---

## 3. Hệ thống tệp tin (File System)

### Câu 4: Inode là gì? Xóa một file trong Linux diễn ra như thế nào?
- **Inode**: Lưu trữ toàn bộ Metadata của file (dung lượng, quyền, thời gian, con trỏ data blocks), không lưu tên file.
- **Quy trình xóa file**:
  - Giảm số lượng liên kết cứng `link count` trong Inode đi 1.
  - Khi `link count = 0` **VÀ** không còn tiến trình nào đang mở file descriptor tới file đó, Inode và các khối dữ liệu Data Block mới thực sự được Kernel đánh dấu giải phóng để ghi đè.

### Câu 5: Phân biệt Hard Link và Soft Link?
- **Hard Link**: Thư mục mới trỏ tới cùng số hiệu Inode với file gốc. Xóa file gốc thì hard link vẫn dùng bình thường. Không áp dụng được xuyên phân vùng đĩa.
- **Soft Link (Symbolic Link)**: File mới với Inode riêng, chứa chuỗi đường dẫn tới file gốc. Xóa file gốc thì soft link bị gãy (Broken link). Hoạt động được xuyên phân vùng đĩa.

---

## 4. Kỹ năng Linux thực chiến

### Câu 6: Khi CPU máy chủ tăng vọt lên 100%, bạn điều tra như thế nào?
1. Chạy lệnh `top` xem PID của tiến trình chiếm CPU cao nhất và xem tỷ lệ `%Cpu` (nếu `us` cao do code ứng dụng, nếu `wa` cao do nghẽn I/O đĩa).
2. Nếu là ứng dụng Java: Chạy `top -Hp <pid>` để tìm Thread ID (TID) đang ngốn CPU.
3. Đổi TID sang dạng Hex: `printf "%x\n" <tid>`.
4. Dùng `jstack <pid> | grep -A 30 <tid_hex>` để xem trực tiếp stack trace và dòng code Java đang chạy.

<!-- @include: @article-footer.snippet.md -->
