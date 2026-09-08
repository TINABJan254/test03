---
title: Chi tiết Quản lý bộ nhớ trong Hệ điều hành: Phân trang, Phân đoạn, Thay thế trang, Swap và OOM
description: Tổng hợp toàn diện kiến thức quản lý bộ nhớ trong Hệ điều hành, phân tích VSZ/RSS/PSS, phân mảnh bộ nhớ, Buddy System, phân trang đa cấp, TLB, Page Fault, cơ chế Swap và OOM Killer.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Quản lý bộ nhớ
head:
  - - meta
    - name: keywords
      content: Quản lý bộ nhớ, Phân trang, Phân đoạn, Bộ nhớ ảo, Buddy System, Slab, TLB, Page Fault, Swap, OOM Killer, VSZ, RSS
---

Quản lý bộ nhớ (Memory Management) là một trong những nhiệm vụ phức tạp và quan trọng nhất của hệ điều hành.

Hệ điều hành phải trả lời các câu hỏi: Làm sao để nhiều tiến trình cùng chạy mà không ghi đè dữ liệu của nhau? Khi bộ nhớ vật lý RAM bị đầy thì xử lý ra sao? Các chỉ số VSZ, RSS trong lệnh `top` có ý nghĩa gì? Tại sao ứng dụng Java bị Linux OOM Killer tiêu diệt?

Bài viết này sẽ làm sáng tỏ toàn bộ hệ thống quản lý bộ nhớ.

---

## 1. Phân biệt các chỉ số bộ nhớ: VSZ, RSS và PSS

Khi dùng lệnh `ps aux` hoặc `top` trong Linux, chúng ta thường thấy:

- **VSZ (Virtual Memory Size - Kích thước bộ nhớ ảo)**: Tổng dung lượng không gian địa chỉ ảo mà tiến trình đã xin cấp phát (bao gồm code, heap, stack, các file thư viện `.so` ánh xạ vào). Nó không phản ánh lượng RAM thực tế đang chiếm dụng.
- **RSS (Resident Set Size - Kích thước bộ nhớ thường trú)**: Dung lượng bộ nhớ vật lý **RAM thực tế** đang được cấp phát cho tiến trình. Lưu ý: RSS bao gồm cả các trang bộ nhớ dùng chung (Shared Libraries) với các tiến trình khác.
- **PSS (Proportional Set Size - Kích thước tỷ lệ)**: Dung lượng RAM thực tế riêng của tiến trình cộng với phần bộ nhớ chia sẻ được chia đều theo tỷ lệ số tiến trình cùng dùng.

---

## 2. Sự tiến hóa của các mô hình quản lý bộ nhớ

### 1. Phân bổ liên tục (Contiguous Allocation)
- Cấp cho mỗi tiến trình một vùng nhớ vật lý liên tục.
- *Nhược điểm nghiêm trọng*: Gây ra hiện tượng **Phân mảnh ngoại vi (External Fragmentation)** (tổng dung lượng RAM còn trống đủ lớn nhưng bị chia cắt thành các mảnh vụn nhỏ rời rạc không thể cấp cho tiến trình mới).

### 2. Mô hình Phân trang (Paging)
- Chia không gian bộ nhớ ảo của tiến trình thành các khối có kích thước cố định gọi là **Trang (Page)** (kích thước phổ biến là 4 KB).
- Chia bộ nhớ vật lý RAM thành các khối có kích thước tương đương gọi là **Khung trang (Page Frame)**.
- Các trang ảo có thể được nạp vào các khung trang vật lý **rời rạc bất kỳ** trong RAM thông qua **Bảng trang (Page Table)**.
- *Ưu điểm*: Loại bỏ hoàn toàn phân mảnh ngoại vi, chỉ còn phân mảnh nội vi nhỏ bên trong trang cuối cùng.

### 3. Mô hình Phân đoạn (Segmentation)
- Chia bộ nhớ theo logic ngữ nghĩa của chương trình: Đoạn mã (Code Segment), Đoạn dữ liệu (Data Segment), Đoạn Stack, Đoạn Heap. Mỗi đoạn có độ dài linh hoạt khác nhau.

---

## 3. Thuật toán quản lý bộ nhớ vật lý trong Linux Kernel

Linux sử dụng kết hợp 2 cơ chế:

1. **Hệ thống Buddy (Buddy System)**: Quản lý cấp phát các trang bộ nhớ vật lý theo lũy thừa của 2 ($2^0, 2^1, 2^2, \dots, 2^{11}$ trang, tương đương từ 4KB đến 4MB). Giúp ghép các mảnh bộ nhớ kề nhau (buddies) lại khi giải phóng để chống phân mảnh.
2. **Slab / Slub Allocator**: Quản lý cấp phát các đối tượng dữ liệu nhỏ trong kernel (vài byte đến vài KB, như `task_struct`, `inode`, `sk_buff`) để tránh lãng phí cả một trang 4KB của Buddy System.

---

## 4. Swap và OOM Killer trong Linux

### Cơ chế Swap (Bộ nhớ hoán đổi)
- Khi bộ nhớ RAM vật lý bắt đầu cạn kiệt, Kernel sẽ chọn các trang bộ nhớ ít sử dụng (Inactive Anonymous Pages) và di chuyển chúng xuống ổ đĩa cứng (phân vùng Swap / Swapfile) để giải phóng RAM cho các tiến trình đang hoạt động.
- *Tác dụng phụ*: Tốc độ truy xuất ổ đĩa chậm hơn RAM hàng nghìn lần, nên khi Swap hoạt động mạnh sẽ làm hệ thống bị chậm nghiêm trọng (Thrashing). Trong môi trường Kubernetes / Database hiệu năng cao, người ta thường tắt Swap (`swapoff -a`).

### Cơ chế OOM Killer (Out of Memory Killer)
- Khi bộ nhớ RAM và Swap đều cạn kiệt hoàn toàn và Kernel không thể cấp phát thêm bộ nhớ:
- Kernel sẽ kích hoạt tiến trình **OOM Killer**: Quét toàn bộ hệ thống, tính toán điểm số phạt (`oom_score`) dựa trên dung lượng RAM chiếm dụng và quyền hạn của tiến trình, sau đó chọn tiến trình có điểm cao nhất và phát tín hiệu `SIGKILL` để tiêu diệt tiến trình đó ngay lập tức nhằm cứu sống toàn bộ hệ điều hành.

<!-- @include: @article-footer.snippet.md -->
