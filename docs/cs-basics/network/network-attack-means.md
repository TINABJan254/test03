---
title: Tổng hợp các thủ đoạn tấn công mạng thường gặp (Bảo mật mạng)
description: Tổng hợp các kiểu tấn công TCP/IP phổ biến và tư duy phòng thủ trong thực tế, bao gồm DDoS, IP Spoofing, SYN Flood, ARP Spoofing, Man-in-the-Middle (MITM), DNS Spoofing và Port Scanning.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Tấn công mạng, DDoS, IP Spoofing, ARP Spoofing, Man-in-the-Middle, SYN Flood, Quét cổng, Bảo mật mạng
---

> Bài viết được tổng hợp và hoàn thiện từ bài viết [Các thủ đoạn tấn công TCP/IP thường gặp - 暖蓝笔记 - 2021](https://mp.weixin.qq.com/s/AZwWrOlLxRSSi-ywBgZ0fA).

Chồng giao thức TCP/IP sinh ra nhằm phục vụ mục tiêu kết nối và liên thông mạng, nhiều cơ chế ở thời kỳ đầu thiết kế đã không lường hết quy mô và cường độ tấn công phức tạp ngày nay.

Các hình thức tấn công như IP Spoofing, SYN Flood, DDoS, ARP Spoofing, DNS Hijacking nhìn bề ngoài khác nhau nhưng về bản chất đều lợi dụng các giả định tin cậy ngầm định, điểm nghẽn tiêu hao tài nguyên hoặc chuỗi phân giải trong giao thức mạng.

Bài viết này chủ yếu trả lời các câu hỏi:

1. Các thủ đoạn tấn công TCP/IP thường gặp lợi dụng cơ chế nào?
2. IP Spoofing, SYN Flood, DDoS diễn ra cụ thể ra sao?
3. Ảnh hưởng và hậu quả của các cuộc tấn công này là gì?
4. Những tư duy và giải pháp phòng ngự cơ bản trong thực tế?

---

## 1. IP Spoofing (Giả mạo IP)

### Khái niệm
IP Spoofing là kỹ thuật kẻ tấn công giả mạo địa chỉ IP nguồn trong gói tin IP, khiến máy nhận tin rằng gói tin được gửi đến từ một máy chủ hợp lệ hoặc có đặc quyền đáng tin cậy.

### Kịch bản tấn công
Giả sử người dùng hợp lệ (`1.1.1.1`) đã thiết lập kết nối TCP với Server. Kẻ tấn công có thể giả mạo IP nguồn là `1.1.1.1` và gửi một gói tin `RST` tới Server. Nếu gói tin `RST` này đoán trúng bộ 4 thông số (4-tuple) và số thứ tự `Sequence Number` hợp lệ, Server sẽ lập tức ngắt kết nối TCP của người dùng hợp lệ.

![Kẻ tấn công giả mạo IP nguồn gửi gói RST ngắt kết nối](https://oss.javaguide.cn/p3-juejin/7547a145adf9404aa3a05f01f5ca2e32~tplv-k3u1fbpfcp-zoom-1.png)

### Cách phòng thủ
- **Ingress Filtering (Lọc gói tin ở cổng vào)**: Được triển khai tại các Router biên mạng theo chuẩn BCP 38, kiểm tra xem địa chỉ IP nguồn của gói tin có khớp với dải mạng con thực tế đang đi vào hay không; nếu bất thường sẽ lập tức hủy bỏ.
- **Egress Filtering (Lọc gói tin ở cổng ra)**: Ngăn chặn các máy nội bộ phát tán gói tin giả mạo IP nguồn ra bên ngoài Internet.

---

## 2. SYN Flood (Tấn công tràn ngập SYN)

### Bản chất của SYN Flood
SYN Flood là một trong những dạng tấn công DoS/DDoS kinh điển nhất trên Internet, nhằm làm cạn kiệt tài nguyên bộ nhớ Kernel của máy chủ mục tiêu.

SYN Flood khai thác cơ chế Bắt tay 3 bước của TCP: Kẻ tấn công liên tục gửi hàng triệu gói tin `SYN` với IP nguồn giả mạo hoặc không bao giờ phản hồi gói `ACK` bước 3.
- Phía Server nhận được `SYN`, gửi lại `SYN+ACK` và cấp phát một mục trong **Hàng đợi bán kết nối (SYN Queue)** ở trạng thái `SYN_RCVD`.
- Do không bao giờ nhận được `ACK` phản hồi, Server phải giữ trạng thái bán kết nối này, khởi động Timer và liên tục gửi lại `SYN+ACK` thăm dò đến khi timeout.
- Khi hàng triệu yêu cầu bán kết nối tràn ngập, **SYN Queue bị đầy hoàn toàn**, Server không còn khả năng tiếp nhận bất kỳ kết nối hợp lệ nào của người dùng bình thường.

![SYN Flood làm cạn kiệt tài nguyên bán kết nối của Server](https://oss.javaguide.cn/p3-juejin/2b3d2d4dc8f24890b5957df1c7d6feb8~tplv-k3u1fbpfcp-zoom-1.png)

### Cách phòng thủ SYN Flood
1. **SYN Cookie (`net.ipv4.tcp_syncookies = 1`)**: Khi SYN Queue bị đầy, Server không lưu trạng thái trong bộ nhớ mà mã hóa thông tin kết nối vào số thứ tự ISN của gói `SYN+ACK`. Khi Client gửi ACK hợp lệ, Server mới giải mã và cấp phát tài nguyên kết nối.
2. **Tăng dung lượng hàng đợi bán kết nối**: Tăng tham số `net.ipv4.tcp_max_syn_backlog`.
3. **Giảm số lần truyền lại SYN+ACK**: Giảm `net.ipv4.tcp_synack_retries` (ví dụ từ 5 lần xuống 2 lần) để nhanh chóng giải phóng các kết nối không phản hồi.
4. **Sử dụng Tường lửa / Thiết bị chống DDoS chuyên dụng**: Kiểm tra tính hợp lệ của IP nguồn (SYN Proxy, TCP Check).

---

## 3. DDoS (Distributed Denial of Service - Tấn công từ chối dịch vụ phân tán)

### Bản chất của DDoS
DDoS là hình thức tấn công mà kẻ tấn công điều khiển một mạng lưới hàng trăm nghìn thiết bị bị nhiễm mã độc (gọi là **Botnet / Mạng máy tính ma**) đồng loạt phát lưu lượng truy cập khổng lồ (hàng trăm Gbps tới Tbps) nhắm vào một mục tiêu duy nhất, làm tê liệt băng thông đường truyền hoặc làm sập hoàn toàn hệ thống xử lý của máy chủ.

![Tổng quan tấn công DDoS qua Botnet](https://oss.javaguide.cn/p3-juejin/bcf506720d5e46be959a43a0e1c3132e~tplv-k3u1fbpfcp-zoom-1.png)

### Các loại tấn công DDoS phổ biến
1. **Tấn công tầng mạng / băng thông (Volumetric Attacks)**:
   - **UDP Flood, ICMP Flood**: Gửi hàng loạt gói tin UDP/Ping khổng lồ làm nghẽn toàn bộ băng thông đường truyền Internet của nạn nhân.
   - **NTP / DNS Amplification (Tấn công khuếch đại)**: Lợi dụng các máy chủ NTP/DNS công cộng có tính năng khuếch đại: gửi một truy vấn nhỏ 50 byte với IP nguồn giả mạo là IP nạn nhân, khiến máy chủ phản hồi gói tin khổng lồ 4000 byte dội thẳng về nạn nhân (hệ số khuếch đại gấp 50-80 lần).
2. **Tấn công tầng giao vận (Protocol Attacks)**: SYN Flood, ACK Flood.
3. **Tấn công tầng ứng dụng (Application Layer - Layer 7 Attacks)**:
   - **HTTP Flood (CC Attack)**: Gửi hàng triệu HTTP GET/POST request hợp lệ nhắm vào các trang xử lý nặng (như tìm kiếm, truy vấn DB phức tạp) làm cạn kiệt CPU/RAM của Web Server và Database.
   - **Slowloris (Tấn công kết nối chậm)**: Giữ hàng nghìn kết nối HTTP mở bằng cách gửi header cực kỳ chậm chạp từng byte một, làm cạn kiệt Connection Pool của Web Server (như Apache).

### Cách phòng thủ DDoS
- Sử dụng dịch vụ **Cloud DDoS Protection / Anycast Scrubbing Center** (như Cloudflare, AWS Shield, Alibaba Cloud Anti-DDoS): Phân tán và lọc sạch lưu lượng tấn công tại các node biên trước khi đến máy chủ gốc.
- Bật **Rate Limiting** (Giới hạn tần suất request theo IP) tại API Gateway / WAF / Nginx.
- Tối ưu hóa Cache (Redis/CDN) để giảm tải cho Database.

---

## 4. ARP Spoofing (Giả mạo ARP)

### Bản chất
Trong mạng cục bộ (LAN), giao thức ARP không có cơ chế xác thực danh tính. Kẻ tấn công trong cùng mạng LAN có thể liên tục phát các gói tin **ARP Reply giả mạo (Gratuitous ARP)**:
- Nói với Nạn nhân: *"Địa chỉ IP của Gateway là tôi (gán MAC của kẻ tấn công)"*.
- Nói với Gateway: *"Địa chỉ IP của Nạn nhân là tôi (gán MAC của kẻ tấn công)"*.

Hậu quả: Toàn bộ lưu lượng Internet của Nạn nhân sẽ bị chuyển tiếp qua máy kẻ tấn công, hình thành thế tấn công **Man-in-the-Middle (Nghe lén / Đánh cắp dữ liệu)** hoặc ngắt kết nối mạng của nạn nhân (DoS).

![Tấn công ARP Spoofing trong mạng LAN](https://oss.javaguide.cn/p3-juejin/5ce347076d29486c87cf863fe940ba0a~tplv-k3u1fbpfcp-zoom-1.png)

### Cách phòng thủ
- **Bật tính năng Dynamic ARP Inspection (DAI)** trên Switch mạng doanh nghiệp (kết hợp với DHCP Snooping).
- Cấu hình gán cứng IP-MAC tĩnh (**Static ARP Binding**) trên máy tính và Router.
- Sử dụng giao thức mã hóa HTTPS/SSH để kẻ tấn công dù có bắt được gói tin cũng không thể đọc được nội dung.

---

## 5. Man-in-the-Middle (Tấn công kẻ trung gian - MITM)

### Bản chất
Kẻ tấn công đứng giữa kênh giao tiếp giữa Client và Server, bí mật chặn bắt, nghe lén hoặc sửa đổi thông tin trao đổi giữa hai bên mà không bên nào hay biết.

### Cách phòng thủ
- **Bắt buộc sử dụng HTTPS (TLS 1.2 / TLS 1.3)**: Kênh truyền được mã hóa đối xứng, ngăn chặn việc đọc trộm nội dung.
- **Sử dụng Chứng chỉ số CA hợp lệ** và kiểm tra chuỗi chứng chỉ chặt chẽ (Trình duyệt sẽ cảnh báo đỏ khi chứng chỉ bị giả mạo).
- **HSTS (HTTP Strict Transport Security)**: Bắt buộc trình duyệt chỉ kết nối qua HTTPS, chống tấn công SSL Stripping.

---

## 6. Port Scanning (Quét cổng mạng)

### Khái niệm
Kỹ thuật do kẻ tấn công (hoặc chuyên gia bảo mật) sử dụng các công cụ như `Nmap` để gửi các gói tin thăm dò tới một loạt các cổng trên máy chủ nhằm phát hiện những dịch vụ nào đang mở (Open Ports), hệ điều hành và phiên bản phần mềm đang chạy để tìm kiếm lỗ hổng đã biết.

### Các kỹ thuật quét phổ biến
- **TCP Connect Scan**: Hoàn thành bắt tay 3 bước trọn vẹn (Dễ bị Firewall ghi log).
- **TCP SYN Scan (Half-open Scan)**: Chỉ gửi `SYN`, nếu nhận được `SYN+ACK` thì xác định cổng mở rồi gửi ngay `RST` để ngắt kết nối (Khó bị phát hiện hơn).
- **UDP Scan**: Gửi gói UDP trống; nếu nhận lại ICMP Port Unreachable thì cổng đóng, nếu không phản hồi thì cổng có thể đang mở.

### Cách phòng thủ
- Đóng toàn bộ các cổng không sử dụng, chỉ mở cổng cần thiết (`80`, `443`, `22`).
- Đổi các cổng mặc định nhạy cảm (ví dụ đổi cổng SSH `22` sang một cổng ngẫu nhiên).
- Dùng Firewall / Security Group giới hạn IP được phép truy cập vào các cổng quản trị nội bộ.

<!-- @include: @article-footer.snippet.md -->
