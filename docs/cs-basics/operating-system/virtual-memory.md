---
title: Chi tiết Bộ nhớ ảo (Virtual Memory): Chuyển đổi địa chỉ, TLB, Page Fault và Thuật toán thay thế trang
description: Phân tích toàn diện kiến trúc Bộ nhớ ảo: Cách MMU chuyển đổi địa chỉ ảo sang vật lý, vai trò của Bảng trang đa cấp (Multi-level Page Table), TLB Caching, quy trình xử lý Page Fault và các thuật toán thay thế trang (LRU, FIFO, LFU, Clock).
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Bộ nhớ ảo
head:
  - - meta
    - name: keywords
      content: Bộ nhớ ảo, Virtual Memory, MMU, Bảng trang, Page Table, TLB, Page Fault, Lỗi trang, Thuật toán thay thế trang, LRU, FIFO, Clock
---

Bộ nhớ ảo (Virtual Memory) là một trong những phát minh vĩ đại nhất của ngành khoa học máy tính.

Nhờ có Bộ nhớ ảo:
- Mỗi tiến trình đều cảm giác mình đang sở hữu độc quyền một không gian bộ nhớ riêng biệt, liên tục và khổng lồ (ví dụ 4GB trên hệ thống 32-bit, 128TB trên 64-bit), bất kể dung lượng RAM vật lý thực tế là bao nhiêu.
- Các tiến trình được cô lập an toàn, không thể ghi đè vào bộ nhớ của nhau hoặc can thiệp vào Kernel.

Bài viết này sẽ đi sâu vào nguyên lý vận hành bên dưới của Bộ nhớ ảo.

---

## 1. Cơ chế chuyển đổi địa chỉ: Từ Địa chỉ ảo sang Địa chỉ vật lý

Mỗi địa chỉ mà chương trình của bạn sử dụng (ví dụ con trỏ `0x7fff5fbff8ac` trong C/C++) đều là **Địa chỉ ảo (Virtual Address)**.

Bộ xử lý phần cứng **MMU (Memory Management Unit)** nằm trong CPU kết hợp với **Bảng trang (Page Table)** do hệ điều hành duy trì để thực hiện dịch địa chỉ:

```text
Địa chỉ ảo (Virtual Address)
┌───────────────────────────┬───────────────────────────┐
│   Số trang ảo (VPN)       │   Độ lệch trong trang (Offset)│
└─────────────┬─────────────┴─────────────┬─────────────┘
              │                           │
              ▼ (Tra cứu Page Table)      │
┌─────────────┴─────────────┐             │
│   Số khung vật lý (PFN)   │             │
└─────────────┬─────────────┘             │
              ▼                           ▼
┌───────────────────────────┬───────────────────────────┐
│   Số khung vật lý (PFN)   │   Độ lệch trong trang (Offset)│
└───────────────────────────┴───────────────────────────┘
Địa chỉ vật lý (Physical Address trong RAM)
```

- **Offset (Độ lệch)**: Được giữ nguyên không đổi vì kích thước trang ảo bằng kích thước khung trang vật lý (4KB = $2^{12}$ byte, nên 12 bit cuối là Offset).
- **VPN (Virtual Page Number)**: Được chuyển đổi thành **PFN (Physical Frame Number)** tương ứng trong RAM.

---

## 2. Bảng trang đa cấp (Multi-level Page Table)

Trên hệ điều hành 64-bit, nếu dùng bảng trang 1 cấp thì dung lượng lưu trữ bảng trang sẽ ngốn hàng chục GB RAM cho mỗi tiến trình.

Để tiết kiệm bộ nhớ, hệ điều hành sử dụng **Bảng trang đa cấp (Multi-level Page Table)** (trên x86-64 sử dụng 4 cấp hoặc 5 cấp trang: PGD -> P4D -> PUD -> PMD -> PTE).
- **Ưu điểm lớn nhất**: Chỉ những vùng địa chỉ ảo nào thực sự được tiến trình sử dụng thì mới cấp phát bảng trang cấp dưới, giúp tiết kiệm hơn 99% bộ nhớ cho bảng trang!

---

## 3. Tăng tốc chuyển đổi địa chỉ: Bộ đệm TLB (Translation Lookaside Buffer)

Do bảng trang đa cấp cần nhiều lần truy cập bộ nhớ RAM để tìm ra địa chỉ vật lý (tốn từ 4 đến 5 lần đọc RAM), CPU trang bị một bộ nhớ đệm phần cứng siêu tốc độ gọi là **TLB (Translation Lookaside Buffer)**:
- TLB lưu trữ các ánh xạ `(VPN -> PFN)` được truy cập gần đây.
- **TLB Hit**: CPU dịch địa chỉ ngay lập tức trong 1 chu kỳ máy (khoảng 0.5 - 1 ns).
- **TLB Miss**: CPU mới phải đi lần từng cấp của Page Table trong RAM (tốn vài chục ns) rồi nạp vào TLB.

---

## 4. Ngoại lệ lỗi trang (Page Fault Exception)

Khi CPU cố gắng truy cập một địa chỉ ảo:
1. MMU tra cứu bảng trang và phát hiện bit hợp lệ `Present Bit = 0` (nghĩa là trang bộ nhớ này hiện chưa được nạp vào RAM vật lý, hoặc mới chỉ được cấp phát ảo mà chưa ghi dữ liệu).
2. CPU lập tức kích hoạt một **Ngoại lệ Lỗi trang (Page Fault Exception)** chuyển quyền điều khiển cho Kernel.
3. Trình xử lý Page Fault của Kernel thực hiện:
   - Kiểm tra xem địa chỉ ảo có hợp lệ không (nếu không hợp lệ -> gửi tín hiệu `SIGSEGV` báo lỗi Segmentation Fault).
   - Nếu hợp lệ: Cấp phát một khung trang vật lý trống trong RAM.
   - Đọc dữ liệu trang tương ứng từ ổ đĩa (tệp thực thi hoặc swap) nạp vào khung trang RAM.
   - Cập nhật lại Bảng trang (đặt `Present Bit = 1`).
4. Khôi phục lại trạng thái thanh ghi và cho CPU thực thi lại đúng chỉ lệnh vừa gây ra Page Fault. Lúc này trang đã có sẵn trong RAM và truy cập thành công!

---

## 5. Các thuật toán thay thế trang (Page Replacement Algorithms)

Khi xảy ra Page Fault nhưng bộ nhớ RAM đã đầy, hệ điều hành phải chọn một trang trong RAM để loại bỏ (hoặc hoán đổi xuống Swap) nhường chỗ cho trang mới:

1. **Thuật toán tối ưu (OPT - Optimal)**: Loại bỏ trang mà trong tương lai sẽ lâu nhất không được sử dụng. Đây là thuật toán lý tưởng nhất về mặt lý thuyết, nhưng không thể thực hiện trong thực tế vì không thể đoán trước tương lai (dùng làm thước đo chuẩn so sánh).
2. **FIFO (First-In, First-Out)**: Loại bỏ trang được nạp vào bộ nhớ sớm nhất. *Nhược điểm*: Gặp **nghịch lý Belady (Belady's Anomaly)**: Tăng thêm số khung trang RAM nhưng tỷ lệ lỗi trang lại tăng lên!
3. **LRU (Least Recently Used - Khuyên dùng)**: Loại bỏ trang có thời gian không được truy cập lâu nhất trong quá khứ (dựa trên nguyên lý cục bộ về thời gian Temporal Locality). Cho hiệu năng rất cao, là thuật toán phổ biến nhất trong OS, JVM, Redis và Database.
4. **Clock (Thuật toán đồng hồ / NRU)**: Sử dụng một bit tham chiếu (Reference bit) và một con trỏ quét vòng tròn như kim đồng hồ để xấp xỉ thuật toán LRU với chi phí phần cứng rất rẻ.

<!-- @include: @article-footer.snippet.md -->
