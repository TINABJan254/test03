---
title: Chuyên đề Mạng máy tính: Mô hình phân tầng, HTTP, HTTPS, DNS, TCP, UDP, ARP và NAT
description: Lộ trình học tập và phỏng vấn Mạng máy tính, bao gồm mô hình phân tầng OSI / TCP-IP, HTTP, HTTPS, DNS, TCP, UDP, ARP, NAT, an toàn mạng và các câu hỏi phỏng vấn thường gặp.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
  - TCP/IP
  - HTTP
sidebar: false
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Mạng máy tính, Câu hỏi phỏng vấn Mạng máy tính, Mô hình 7 tầng OSI, TCP/IP, HTTP, HTTPS, DNS, TCP, UDP, ARP, NAT, Phỏng vấn Backend
---

**Chuyên đề Mạng máy tính** này hướng tới việc học tập và ôn tập phỏng vấn Backend, được sắp xếp theo trình tự: "Mô hình phân tầng -> Tầng ứng dụng -> Tầng giao vận -> Tầng mạng -> Bảo mật".

## Phù hợp với ai

- Các lập trình viên Backend đang học Mạng máy tính một cách có hệ thống.
- Các bạn đang chuẩn bị cho các kỳ phỏng vấn mạng của đợt tuyển dụng sinh viên (Campus), người có kinh nghiệm (Social), các công ty vừa và lớn.
- Những độc giả chỉ học vẹt rời rạc các điểm kiến thức về HTTP, HTTPS, TCP, DNS, Socket.
- Các kỹ sư muốn kết nối kiến thức mạng với RPC, API Gateway, Load Balancing và System Design.

## Trọng tâm học tập

- Giá trị cốt lõi của phân tầng mạng là chia nhỏ bài toán giao tiếp phức tạp, mỗi tầng chỉ giải quyết trách nhiệm của riêng mình.
- HTTP, HTTPS, DNS là các kiến thức tầng ứng dụng được sử dụng thường xuyên nhất trong lập trình Backend.
- Các điểm thi tần suất cao của TCP tập trung vào Quản lý kết nối, Truyền tải tin cậy, Kiểm soát tắc nghẽn (Congestion Control), TIME_WAIT và Keepalive.
- Kiến thức tầng mạng như ARP, NAT giúp hiểu rõ giao tiếp trong mạng cục bộ (LAN), truy cập mạng nội bộ/mạng ngoài và xử lý sự cố (troubleshooting).
- Trong phỏng vấn, cần có khả năng xâu chuỗi một request hoàn chỉnh qua các khâu: giao thức, kết nối, mã hóa, phân giải tên miền và truyền tải.

## Thứ tự đọc khuyến nghị

1. [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (Phần 1)](./other-network-questions.md) và [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (Phần 2)](./other-network-questions2.md): Thiết lập danh sách câu hỏi tần suất cao trước.
2. [Chi tiết Mô hình 7 tầng OSI và Mô hình 4 tầng TCP/IP](./osi-and-tcp-ip-model.md): Hiểu về phân tầng mạng và trách nhiệm từng tầng.
3. [Từ lúc nhập URL đến khi trang web hiển thị thực sự đã xảy ra những gì?](./the-whole-process-of-accessing-web-pages.md): Dùng chuỗi hoàn chỉnh để xâu chuỗi DNS, TCP, HTTP và xử lý của trình duyệt.
4. [HTTP vs HTTPS](./http-vs-https.md), [RSA và ECDHE trong bắt tay HTTPS](./https-rsa-vs-ecdhe.md), [Tổng hợp HTTP Status Code thường gặp](./http-status-codes.md): Bổ sung các câu hỏi tần suất cao ở tầng ứng dụng.
5. [Bắt tay 3 bước và Bắt tay 4 bước của TCP (3-way Handshake & 4-way Teardown)](./tcp-connection-and-disconnection.md), [Đảm bảo độ tin cậy truyền tải của TCP](./tcp-reliability-guarantee.md), [Chi tiết TCP TIME_WAIT](./tcp-time-wait.md): Trọng tâm chinh phục TCP.

## Các bài viết cốt lõi

### Tổng quan & Nền tảng

- [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (Phần 1)](./other-network-questions.md): Bao quát mô hình mạng, HTTP, HTTPS, DNS và các câu hỏi cơ bản.
- [Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (Phần 2)](./other-network-questions2.md): Tiếp tục tổng hợp TCP, UDP, Socket, an toàn mạng và các câu hỏi tần suất cao.
- [Chi tiết Mô hình 7 tầng OSI và Mô hình 4 tầng TCP/IP](./osi-and-tcp-ip-model.md): Hiểu mô hình mạng, phân tầng giao thức và quy trình đóng gói dữ liệu (encapsulation).
- [Từ lúc nhập URL đến khi trang web hiển thị thực sự đã xảy ra những gì?](./the-whole-process-of-accessing-web-pages.md): Dùng một request để xâu chuỗi các điểm kiến thức mạng phổ biến.

### Tầng ứng dụng (Application Layer)

- [Tổng hợp các giao thức tầng ứng dụng thường gặp](./application-layer-protocol.md): Tổng hợp các giao thức HTTP, WebSocket, SMTP, FTP, SSH, DNS, v.v.
- [HTTP vs HTTPS](./http-vs-https.md): Hiểu mã hóa HTTPS, chứng chỉ (certificate), xác thực danh tính và bảo toàn tính toàn vẹn dữ liệu.
- [RSA và ECDHE trong bắt tay HTTPS](./https-rsa-vs-ecdhe.md): Phân biệt các phương thức trao đổi khóa khác nhau và Forward Secrecy (Tính bảo mật chuyển tiếp).
- [HTTP 1.0 vs HTTP 1.1](./http1.0-vs-http1.1.md): Hiểu sự khác biệt về Persistent Connection, Cache, Host header, v.v.
- [Tổng hợp HTTP Status Code thường gặp](./http-status-codes.md): Nắm vững ngữ nghĩa và kịch bản sử dụng các mã trạng thái từ 1xx đến 5xx.
- [Chi tiết Hệ thống tên miền DNS](./dns.md): Hiểu phân giải tên miền, Recursive Query, Iterative Query và Caching.
- [Đã có HTTP, tại sao còn cần RPC?](./http-vs-rpc.md): Làm rõ mối quan hệ tầng thứ giữa HTTP và RPC.

### Tầng giao vận, Tầng mạng & Bảo mật

- [Bắt tay 3 bước và Bắt tay 4 bước của TCP](./tcp-connection-and-disconnection.md): Nắm vững thiết lập, ngắt kết nối và các trạng thái then chốt.
- [Đảm bảo độ tin cậy truyền tải của TCP](./tcp-reliability-guarantee.md): Hiểu số thứ tự (Sequence Number), ACK, Retransmission, Flow Control và Congestion Control.
- [Chi tiết TCP TIME_WAIT](./tcp-time-wait.md): Hiểu tác dụng, ảnh hưởng và ranh giới tối ưu hóa của trạng thái TIME_WAIT.
- [Sự khác biệt giữa TCP Keepalive và HTTP Keep-Alive là gì?](./tcp-keepalive-vs-http-keepalive.md): Phân biệt cơ chế duy trì kết nối ở tầng giao vận và kết nối dài ở tầng ứng dụng.
- [Tại sao TCP là hướng luồng byte (Byte Stream), còn UDP là hướng gói tin (Datagram)?](./tcp-byte-stream-udp-datagram.md): Hiểu ranh giới dữ liệu giữa TCP và UDP.
- [Chi tiết giao thức ARP](./arp.md), [Chi tiết giao thức NAT](./nat.md), [Tổng hợp các thủ đoạn tấn công mạng thường gặp](./network-attack-means.md): Bổ sung kiến thức tầng mạng và kiến thức an toàn bảo mật.

## Câu hỏi tần suất cao

- Mô hình 7 tầng OSI và mô hình 4 tầng TCP/IP khác nhau như thế nào?
- Từ lúc nhập URL đến khi trang web hiển thị, phần mạng đã diễn ra những gì?
- HTTP và HTTPS khác nhau như thế nào? Quy trình bắt tay HTTPS diễn ra ra sao?
- Khác biệt cốt lõi giữa HTTP 1.0, 1.1 và 2.0 là gì?
- Các mã trạng thái HTTP (Status Code) phổ biến đại diện cho điều gì?
- Tại sao bắt tay 3 bước (3-way Handshake) và bắt tay 4 bước (4-way Teardown) của TCP không thể bớt đi bước nào?
- TCP đảm bảo truyền tải tin cậy bằng cách nào? Phân biệt Flow Control và Congestion Control?
- Tại sao trạng thái TIME_WAIT lại tồn tại? Khi xuất hiện lượng lớn TIME_WAIT thì điều tra và xử lý thế nào?
- TCP Keepalive và HTTP Keep-Alive khác nhau như thế nào?
- DNS, ARP, NAT lần lượt giải quyết những vấn đề gì?

## Chuyên đề liên quan

- [Hệ thống kiến thức Cơ sở máy tính](../)
- [Chuyên đề Hệ điều hành](../operating-system/)
- [Hệ thống kiến thức Hệ thống phân tán](../../distributed-system/)
- [Chuyên đề RPC](../../distributed-system/rpc/)
- [Hệ thống kiến thức Hệ thống hiệu năng cao](../../high-performance/)

<!-- @include: @article-footer.snippet.md -->
