---
title: Tóm tắt kiến thức Mạng máy tính (Theo giáo trình Tạ Hi Nhân)
description: Tổng hợp cô đọng toàn bộ kiến thức cốt lõi môn Mạng máy tính từ Tầng Vật lý, Tầng Liên kết dữ liệu, Tầng Mạng, Tầng Giao vận đến Tầng Ứng dụng.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Mạng máy tính, Tạ Hi Nhân, Tóm tắt kiến thức, Tầng vật lý, Tầng liên kết, Tầng mạng, Tầng giao vận, Tầng ứng dụng
---

<!-- @include: @article-header.snippet.md -->

Giáo trình *Mạng máy tính* (tác giả Tạ Hi Nhân) là cuốn sách giáo khoa kinh điển được giảng dạy tại hầu hết các trường đại học chuyên ngành Công nghệ Thông tin.

Bài viết này tổng hợp cô đọng toàn bộ kiến thức trọng tâm của 5 tầng mạng từ giáo trình để phục vụ ôn thi tốt nghiệp, thi cao học và phỏng vấn kỹ thuật.

---

## Chương 1: Tổng quan về Mạng máy tính

- **3 loại mạng máy tính**: Mạng viễn thông, Mạng truyền hình cáp và Mạng máy tính (Internet).
- **Các thành phần của Internet**:
  - Phần rìa (Edge): Các thiết bị đầu cuối (Host, PC, Server, Smartphone) chạy các ứng dụng mạng.
  - Phần lõi (Core): Các bộ định tuyến (Router) và mạng lưới liên kết chuyển mạch gói (Packet Switching).
- **3 phương thức chuyển mạch**: Chuyển mạch kênh (Circuit Switching), Chuyển mạch thông báo (Message Switching), Chuyển mạch gói (Packet Switching).
- **Các chỉ số hiệu năng quan trọng**:
  - **Tốc độ (Rate / Bit rate)**: Số bit truyền qua trong 1 giây (bps, kbps, Mbps, Gbps).
  - **Băng thông (Bandwidth)**: Độ rộng dải tần số tín hiệu hoặc tốc độ truyền dữ liệu tối đa trên đường truyền.
  - **Thông lượng (Throughput)**: Lượng dữ liệu thực tế truyền qua mạng trong một đơn vị thời gian.
  - **Độ trễ (Delay)**: Gồm Trễ truyền dẫn (Transmission Delay = Kích thước gói / Băng thông), Trễ lan truyền (Propagation Delay = Khoảng cách / Tốc độ sóng), Trễ xử lý (Processing Delay) và Trễ xếp hàng (Queueing Delay).
  - **RTT (Round-Trip Time)**: Thời gian truyền vòng từ khi gửi dữ liệu đến khi nhận được xác nhận từ bên nhận.

---

## Chương 2: Tầng Vật lý (Physical Layer)

- Nhiệm vụ: Truyền tải trong suốt luồng bit trên phương tiện truyền dẫn vật lý.
- **Định lý Nyquist**: Giới hạn tốc độ truyền dữ liệu tối đa trên kênh truyền lý tưởng không có nhiễu: $C = 2W \log_2 V$ (bps).
- **Định lý Shannon**: Giới hạn tốc độ truyền dữ liệu tối đa trên kênh truyền có nhiễu Gaussian: $C = W \log_2(1 + S/N)$ (bps).
- **Các kỹ thuật đa ghép kênh (Multiplexing)**:
  - FDM (Ghép kênh phân chia theo tần số)
  - TDM (Ghép kênh phân chia theo thời gian)
  - WDM (Ghép kênh phân chia theo bước sóng ánh sáng)
  - CDM / CDMA (Ghép kênh phân chia theo mã)

---

## Chương 3: Tầng Liên kết dữ liệu (Data Link Layer)

- Đơn vị dữ liệu: **Khung (Frame)**.
- **3 bài toán cơ bản của tầng liên kết dữ liệu**:
  1. **Đóng khung (Framing)**: Xác định ranh giới bắt đầu và kết thúc của một Frame.
  2. **Truyền trong suốt (Transparency)**: Sử dụng kỹ thuật Byte Stuffing hoặc Bit Stuffing để tránh việc dữ liệu bên trong trùng với cờ đóng khung.
  3. **Kiểm tra lỗi (Error Detection)**: Sử dụng mã kiểm tra dư thừa tuần hoàn **CRC (Cyclic Redundancy Check)** để phát hiện bit lỗi.
- **Giao thức CSMA/CD (Carrier Sense Multiple Access with Collision Detection)**:
  - Lắng nghe trước khi nói (Carrier Sense).
  - Vừa nói vừa nghe (Collision Detection).
  - Gặp xung đột thì dừng gửi, gửi tín hiệu gây nhiễu (Jamming Signal) và chờ ngẫu nhiên theo thuật toán Binary Exponential Backoff rồi thử lại.
- **Địa chỉ MAC**: Địa chỉ vật lý 48 bit (6 byte) gán cho Card mạng (NIC).

---

## Chương 4: Tầng Mạng (Network Layer)

- Đơn vị dữ liệu: **IP Datagram (Gói tin IP)**.
- Dịch vụ: Cung cấp dịch vụ truyền gói tin không kết nối, nỗ lực tối đa (Best-effort).
- **Địa chỉ IPv4**: Dài 32 bit, chia thành các lớp A, B, C, D, E và kỹ thuật phân chia mạng con không phân lớp **CIDR (Classless Inter-Domain Routing)** với Subnet Mask (ví dụ `/24`).
- **Các giao thức tầng mạng cốt lõi**:
  - **IP**: Định dạng gói tin, đánh địa chỉ và định tuyến.
  - **ARP**: Chuyển đổi từ địa chỉ IP sang địa chỉ MAC trong mạng cục bộ.
  - **ICMP**: Báo cáo lỗi và kiểm tra trạng thái mạng (Ping, Traceroute).
  - **IGMP**: Quản lý nhóm Multicast.
- **Các giao thức định tuyến**:
  - Định tuyến nội miền (IGP): **RIP** (Distance-Vector), **OSPF** (Link-State).
  - Định tuyến liên miền (EGP): **BGP-4** (Path-Vector).
- **NAT (Network Address Translation)**: Chuyển đổi giữa IP Private nội bộ và IP Public để tiết kiệm không gian địa chỉ IPv4.

---

## Chương 5: Tầng Giao vận (Transport Layer)

- Cung cấp dịch vụ giao tiếp logic End-to-End giữa các tiến trình ứng dụng.
- **UDP (User Datagram Protocol)**:
  - Không kết nối, không đảm bảo tin cậy, hướng gói tin (Datagram), header tối giản 8 byte, hỗ trợ 1-1, 1-nhiều, nhiều-nhiều.
- **TCP (Transmission Control Protocol)**:
  - Hướng kết nối, truyền tin cậy, hướng luồng byte (Byte Stream), header từ 20 đến 60 byte, chỉ hỗ trợ giao tiếp điểm-điểm (1-1).
  - **Bắt tay 3 bước (3-way Handshake)** và **Bắt tay 4 bước (4-way Teardown)**.
  - **5 cơ chế tin cậy**: Sequence Number, ACK, Retransmission (ARQ), Checksum, Flow Control (Sliding Window), Congestion Control (Slow Start, Congestion Avoidance, Fast Retransmit, Fast Recovery).

---

## Chương 6: Tầng Ứng dụng (Application Layer)

- **DNS (Domain Name System)**: Cổng 53 (UDP/TCP), phân giải tên miền thành địa chỉ IP qua hệ thống máy chủ phân cấp (Root, TLD, Authoritative, Recursive).
- **HTTP / HTTPS**: Cổng 80 / 443, giao thức truyền siêu văn bản Web.
- **FTP**: Cổng 21 (Control) và Cổng 20 (Data), truyền tệp tin.
- **Email**: SMTP (Cổng 25/587 gửi thư), POP3 (Cổng 110 nhận thư), IMAP (Cổng 143 đồng bộ thư).
- **DHCP**: Cổng 67/68 (UDP), tự động cấp phát địa chỉ IP cho thiết bị mới vào mạng.

<!-- @include: @article-footer.snippet.md -->
