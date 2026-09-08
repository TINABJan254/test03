---
title: Cơ chế đảm bảo truyền tải tin cậy của TCP (Tầng giao vận)
description: Phân tích toàn diện các cơ chế giúp TCP truyền tải tin cậy trên môi trường mạng IP không tin cậy: Đánh số Sequence Number, Xác nhận ACK, Truyền lại Retransmission (ARQ/SACK), Kiểm soát luồng Flow Control (Sliding Window) và Kiểm soát tắc nghẽn Congestion Control.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: TCP, Truyền tải tin cậy, Sequence Number, ACK, Retransmission, ARQ, SACK, Sliding Window, Flow Control, Congestion Control
---

Mạng IP bên dưới là một mạng chuyển mạch gói **không tin cậy và nỗ lực tối đa (Best-effort)**: Gói tin có thể bị mất, bị trùng lặp, bị đảo lộn thứ tự hoặc bị trễ.

Tuy nhiên, **TCP (Transmission Control Protocol)** lại cung cấp cho tầng ứng dụng một dịch vụ truyền dữ liệu **hoàn toàn tin cậy, không mất gói, không trùng lặp và đúng thứ tự**.

Làm thế nào TCP có thể làm được điều kỳ diệu đó?

Bài viết này sẽ phân tích 5 cơ chế cốt lõi tạo nên độ tin cậy của TCP:

```text
1. Đánh số thứ tự (Sequence Number) và Xác nhận (Acknowledgment - ACK)
2. Cơ chế truyền lại khi mất gói (Retransmission: Timeout & Fast Retransmit / SACK)
3. Kiểm tra tính toàn vẹn dữ liệu (Checksum)
4. Kiểm soát luồng (Flow Control - Cửa sổ trượt Sliding Window)
5. Kiểm soát tắc nghẽn (Congestion Control: Slow Start, Congestion Avoidance, Fast Recovery)
```

---

## 1. Đánh số thứ tự (Sequence Number) và Xác nhận (ACK)

- **Sequence Number (SEQ)**: Mọi byte dữ liệu gửi đi trong TCP đều được đánh số thứ tự tuần tự. Nhờ đó, phía nhận có thể:
  - Sắp xếp lại các gói tin bị đảo lộn thứ tự trên đường truyền về đúng vị trí ban đầu.
  - Phát hiện và loại bỏ các gói tin bị gửi trùng lặp (Duplicate Packets).
- **Acknowledgment (ACK)**: Bên nhận sau khi nhận được dữ liệu sẽ gửi lại số xác nhận `ack = N`, mang ý nghĩa: *"Tôi đã nhận đầy đủ dữ liệu từ byte 0 đến byte N-1, byte tiếp theo tôi đang chờ nhận là byte N"*.

---

## 2. Cơ chế truyền lại (Retransmission)

Khi một gói tin bị mất trên đường truyền mạng, TCP sử dụng 2 cơ chế để phát hiện và gửi lại dữ liệu:

### 1. Truyền lại theo thời gian chờ (Timeout Retransmission - RTO)
- Mỗi khi gửi một đoạn dữ liệu, TCP khởi động một bộ đếm thời gian **RTO (Retransmission Timeout)**.
- RTO được tính toán động dựa trên thời gian truyền vòng **RTT (Round Trip Time)** thực tế của mạng.
- Nếu hết thời gian RTO mà bên gửi chưa nhận được ACK xác nhận, bên gửi sẽ tự động truyền lại gói tin đó.

### 2. Truyền lại nhanh (Fast Retransmit) và SACK (Selective Acknowledgment)
- Nếu một gói tin bị mất trong khi các gói sau đó vẫn tới đích bình thường, bên nhận sẽ liên tục gửi lại **3 gói ACK trùng lặp (Duplicate ACKs)** cho gói tin bị thiếu.
- Bên gửi khi nhận được 3 Duplicate ACKs liên tiếp sẽ lập tức truyền lại gói tin bị thiếu ngay lập tức mà **không cần chờ đến khi hết hạn RTO**.
- **SACK (Selective ACK)**: Cho phép bên nhận thông báo chính xác trong Header những đoạn byte nào đã nhận được và đoạn nào bị thiếu, giúp bên gửi chỉ cần truyền lại đúng những đoạn bị mất thay vì phải gửi lại toàn bộ.

---

## 3. Kiểm tra mã lỗi (Checksum)

Mỗi TCP Segment đều có trường `Checksum` 16 bit bao phủ cả Header và Data. Phía nhận tính toán lại Checksum; nếu phát hiện dữ liệu bị lỗi/biến dạng do nhiễu vật lý, TCP sẽ âm thầm hủy gói tin đó để bên gửi truyền lại.

---

## 4. Kiểm soát luồng (Flow Control - Cửa sổ trượt Sliding Window)

**Mục đích**: Ngăn không cho bên gửi truyền dữ liệu quá nhanh khiến **Bộ đệm nhận (Receive Buffer) của bên nhận bị tràn**.

- Phía nhận thông báo dung lượng bộ đệm còn trống của mình cho bên gửi thông qua trường **Cửa sổ nhận (RWND - Receive Window)** trong TCP Header.
- Bên gửi duy trì một **Cửa sổ gửi (Send Window)** và chỉ được phép gửi tối đa số byte nằm trong phạm vi của `RWND`.
- Khi `RWND = 0`, bên gửi tạm dừng truyền dữ liệu và định kỳ gửi các gói tin thăm dò cửa sổ (Window Probe) để kiểm tra khi nào bên nhận giải phóng thêm bộ đệm.

---

## 5. Kiểm soát tắc nghẽn (Congestion Control)

**Mục đích**: Ngăn không cho các máy gửi quá nhiều dữ liệu vào mạng làm **tắc nghẽn các Router và đường truyền mạng Internet**.

Bên gửi duy trì một **Cửa sổ tắc nghẽn (CWND - Congestion Window)** và điều chỉnh kích thước của nó dựa trên 4 thuật toán kinh điển:

```text
Kích thước cửa sổ gửi thực tế = min(CWND, RWND)
```

1. **Khởi động chậm (Slow Start)**: Ban đầu đặt $CWND = 1	ext{ MSS}$. Sau mỗi RTT nhận đủ ACK, $CWND$ tăng gấp đôi (tăng theo cấp số nhân: $1 	o 2 	o 4 	o 8 	o \dots$) cho đến khi chạm ngưỡng $ssthresh$ (Slow Start Threshold).
2. **Tránh tắc nghẽn (Congestion Avoidance)**: Khi $CWND \ge ssthresh$, $CWND$ chuyển sang tăng tuyến tính (mỗi RTT chỉ tăng thêm $1	ext{ MSS}$).
3. **Phát hiện tắc nghẽn**:
   - Nếu xảy ra Timeout (tắc nghẽn nặng): Đặt $ssthresh = CWND / 2$, kéo $CWND$ tụt về $1	ext{ MSS}$ và quay lại Slow Start.
   - Nếu nhận 3 Duplicate ACKs (tắc nghẽn nhẹ): Kích hoạt Fast Recovery.
4. **Phục hồi nhanh (Fast Recovery)**: Đặt $ssthresh = CWND / 2$, đặt $CWND = ssthresh + 3	ext{ MSS}$, tiếp tục truyền lại nhanh và tăng tuyến tính mà không cần kéo $CWND$ về 1.

<!-- @include: @article-footer.snippet.md -->
