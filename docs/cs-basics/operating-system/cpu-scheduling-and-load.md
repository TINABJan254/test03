---
title: Chi tiết Lập lịch CPU và Tải hệ thống (CPU Load & Scheduling)
description: Phân tích toàn diện các thuật toán lập lịch CPU trong hệ điều hành (FCFS, SJF, RR, Priority, Linux CFS, EEVDF), làm rõ ý nghĩa của Load Average, CPU Utilization và cách điều tra sự cố hiệu năng cao trong production.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Hiệu năng hệ thống
head:
  - - meta
    - name: keywords
      content: Lập lịch CPU, Thuật toán lập lịch, CPU Load, Load Average, CPU Utilization, CFS, EEVDF, top, uptime, Troubleshooting
---

Khi máy chủ gặp sự cố hiệu năng, chúng ta thường chạy lệnh `top` hoặc `uptime` và thấy các chỉ số như:

```text
load average: 12.50, 8.30, 4.15
%Cpu(s): 85.2 us, 10.1 sy, 0.0 ni, 2.5 id, 1.8 wa
```

Ý nghĩa thực sự của **Load Average (Tải trung bình)** là gì? Tại sao có lúc CPU Utilization (Tỷ lệ sử dụng CPU) chỉ có 20% mà Load Average lại tăng vọt lên 50? Hệ điều hành phân bổ CPU cho các tiến trình/luồng dựa trên những thuật toán lập lịch nào?

Bài viết này sẽ giải thích chi tiết từ nguyên lý lập lịch đến thực tiễn vận hành.

---

## 1. Các thuật toán lập lịch CPU kinh điển

Lập lịch CPU (CPU Scheduling) là cơ chế của hệ điều hành quyết định tiến trình/luồng nào trong hàng đợi sẵn sàng (Ready Queue) sẽ được nhận CPU để thực thi tiếp theo.

### 1. Đến trước phục vụ trước (First-Come, First-Served - FCFS)
- Tiến trình nào yêu cầu CPU trước thì được phục vụ trước (Non-preemptive - Không chiếm quyền).
- *Nhược điểm*: Gặp **hiệu ứng đoàn tàu (Convoy Effect)**: Nếu một tiến trình tính toán nặng chạy trước, các tiến trình ngắn khác phải chờ đợi rất lâu, làm tăng thời gian chờ trung bình.

### 2. Tác vụ ngắn nhất trước (Shortest Job First - SJF / SRTF)
- Ưu tiên tiến trình có thời gian thực thi CPU ngắn nhất.
- Cho thời gian chờ trung bình tối ưu nhất về mặt lý thuyết.
- *Nhược điểm*: Khó dự đoán chính xác thời gian chạy của tiến trình tiếp theo; các tiến trình dài có thể bị **bỏ đói (Starvation)** nếu liên tục có các tiến trình ngắn đến.

### 3. Xoay vòng thời gian (Round Robin - RR)
- Mỗi tiến trình được cấp một khoảng thời gian cố định gọi là **Lượng tử thời gian (Time Slice / Quantum)** (ví dụ 10ms - 100ms).
- Khi hết Time Slice, tiến trình bị ngắt và đưa về cuối Ready Queue để nhường CPU cho tiến trình tiếp theo.
- Thuật toán công bằng, phổ biến nhất cho các hệ thống đa nhiệm (Time-sharing System).

### 4. Lập lịch theo độ ưu tiên (Priority Scheduling)
- Mỗi tiến trình được gán một mức độ ưu tiên (Priority). CPU luôn được cấp cho tiến trình có độ ưu tiên cao nhất.
- Áp dụng kỹ thuật **Lão hóa (Aging)**: Tăng dần độ ưu tiên của các tiến trình chờ lâu để chống hiện tượng bỏ đói (Starvation).

### 5. Bộ lập lịch CFS và EEVDF trong Linux Kernel
- **CFS (Completely Fair Scheduler - Bộ lập lịch hoàn toàn công bằng)**: Là bộ lập lịch mặc định của Linux từ bản 2.6.23 đến 6.5.
  - Sử dụng khái niệm **Virtual Runtime (`vruntime`)**: Tiến trình chạy càng nhiều thì `vruntime` càng tăng; tiến trình có mức ưu tiên cao (`nice` thấp) thì `vruntime` tăng chậm hơn.
  - Sử dụng cấu trúc dữ liệu **Cây đỏ đen (Red-Black Tree)** để quản lý các tiến trình: Luôn chọn tiến trình có `vruntime` nhỏ nhất (nằm ở node ngoài cùng bên trái của cây) để chạy tiếp theo.
- **EEVDF (Earliest Eligible Virtual Deadline First)**: Thay thế CFS từ Linux Kernel 6.6 trở đi, bổ sung khái niệm Virtual Deadline để tối ưu hóa độ trễ phản hồi cho các tác vụ nhạy cảm với thời gian thực.

---

## 2. Phân biệt CPU Load Average và CPU Utilization

Đây là điểm nhầm lẫn phổ biến nhất khi tối ưu hóa hệ thống:

### 1. CPU Utilization (Tỷ lệ sử dụng CPU - %)
- Đo lường **tỷ lệ phần trăm thời gian CPU thực sự bận rộn thực thi lệnh** trong một khoảng thời gian quan sát.
- Gồm: `us` (User space - ứng dụng), `sy` (System/Kernel space - system call), `wa` (I/O Wait - chờ đĩa/mạng), `id` (Idle - rảnh rỗi).

### 2. CPU Load Average (Tải trung bình của hệ thống)
- Trong Linux, **Load Average là số lượng tiến trình trung bình đang ở trạng thái CÓ THỂ CHẠY (R - Running/Runnable) và KHÔNG THỂ NGẮT (D - Uninterruptible Sleep / Chờ I/O đĩa)** tại các mốc thời gian 1 phút, 5 phút và 15 phút.

$$	ext{Load Average} = 	ext{Số tiến trình đang chạy/chờ CPU} + 	ext{Số tiến trình đang chờ I/O đĩa không thể ngắt}$$

### Ý nghĩa thực tế:
- **Nếu máy có 4 Core CPU**:
  - `Load = 4.0`: Hệ thống đang vận hành ở mức 100% công suất lý tưởng (không có tiến trình nào phải xếp hàng chờ đợi).
  - `Load = 2.0`: Hệ thống còn 50% tài nguyên rảnh rỗi.
  - `Load = 12.0`: Trung bình có 8 tiến trình đang phải xếp hàng chờ đợi CPU hoặc I/O (hệ thống bị quá tải gấp 3 lần).

### Tại sao CPU Utilization thấp (ví dụ 10%) mà Load Average lại rất cao (ví dụ 30)?
- **Nguyên nhân chính**: Hệ thống bị **nghẽn cổ chai ở I/O đĩa (Disk I/O Bottleneck)**.
- Hàng chục tiến trình đang thực hiện đọc/ghi đĩa chậm, rơi vào trạng thái `D` (Uninterruptible Sleep). Lúc này CPU không phải tính toán gì nhiều nên CPU % rất thấp, nhưng Load Average vẫn tính các tiến trình `D` này vào hàng đợi khiến Load vọt lên rất cao!

---

## 3. Quy trình điều tra sự cố hiệu năng cao (CPU Troubleshooting Checklist)

Khi hệ thống báo động CPU cao:

1. Chạy `uptime` hoặc `top` xem Load Average và chỉ số `%Cpu(s)`:
   - Nếu `us` cao: Ứng dụng chạy vòng lặp vô tận, thuật toán nặng, hoặc Java JVM đang Full GC liên tục.
   - If `sy` cao: Ứng dụng gọi System Call quá nhiều, Context Switch quá thường xuyên.
   - If `wa` cao: Nghẽn I/O đĩa (Slow I/O, thiếu index database, ghi log quá mức).
2. Với ứng dụng Java bị CPU 100%:
   - Chạy `top -Hp <pid>` để tìm mã Thread ID (TID) đang ngốn nhiều CPU nhất.
   - Đổi TID sang dạng Hexadecimal: `printf "%x\n" <tid>`.
   - Dùng lệnh `jstack <pid> | grep -A 30 <tid_hex>` để soi chính xác dòng code Java đang chiếm CPU.

<!-- @include: @article-footer.snippet.md -->
