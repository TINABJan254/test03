---
title: Tổng hợp các giao thức tầng ứng dụng thường gặp: HTTP, WebSocket, SMTP, FTP, SSH, DNS, v.v.
description: Tổng hợp khái niệm cốt lõi và kịch bản điển hình của các giao thức tầng ứng dụng phổ biến, tập trung so sánh mô hình giao tiếp và ranh giới năng lực giữa HTTP và WebSocket.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Giao thức tầng ứng dụng, HTTP, WebSocket, DNS, SMTP, FTP, Đặc tính, Kịch bản
---

<!-- @include: @article-header.snippet.md -->

Các giao thức tầng ứng dụng rất phong phú, những cái tên như HTTP, WebSocket, SMTP, POP3/IMAP, FTP, Telnet, SSH, RTP, DNS thường xuyên xuất hiện cùng nhau.

Chúng ta không cần phải học chi tiết triển khai của tất cả các giao thức này, nhưng nếu chỉ nhớ tên, rất dễ bị nhầm lẫn ở các điểm: "Công dụng, Giao thức truyền tải tầng dưới (TCP hay UDP), Kịch bản sử dụng điển hình".

Bài viết này chủ yếu trả lời các câu hỏi:

1. Các giao thức HTTP, WebSocket, SMTP, FTP, SSH, DNS lần lượt giải quyết vấn đề gì?
2. Các giao thức này thường chạy trên TCP hay UDP, cổng mặc định và kịch bản sử dụng là gì?
3. Những giao thức nào dễ bị nhầm lẫn nhất, cách phân biệt trong phỏng vấn và thực tế?

## HTTP: Giao thức truyền siêu văn bản (HyperText Transfer Protocol)

**HTTP (HyperText Transfer Protocol)** là giao thức tầng ứng dụng dùng để truyền tải siêu văn bản và nội dung đa phương tiện, kịch bản sử dụng phổ biến nhất là giao tiếp giữa Web Browser và Web Server.

![Tổng quan HTTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-overview.png)

Khi chúng ta truy cập một trang web trên trình duyệt, trình duyệt gửi HTTP Request tới máy chủ, máy chủ xử lý rồi trả về HTTP Response. Các tài nguyên HTML, CSS, JavaScript, hình ảnh, video trong trang phần lớn đều được tải qua HTTP.

HTTP sử dụng mô hình Client-Server: Client gửi HTTP Request, Server trả về HTTP Response.

![Giao thức HTTP](https://oss.javaguide.cn/github/javaguide/450px-HTTP-Header.png)

Cần lưu ý rằng HTTP là giao thức tầng ứng dụng, bản thân nó không trực tiếp chịu trách nhiệm truyền tải tin cậy. Các phiên bản HTTP khác nhau có sự phụ thuộc tầng dưới khác nhau:

- **HTTP/1.1**: Dựa trên TCP.
- **HTTP/2**: Thường cũng dựa trên TCP, nhưng đưa vào các khả năng Multiplexing (Đa ghép kênh), Nén Header (HPACK), v.v.
- **HTTP/3**: Dựa trên QUIC (mà QUIC chạy trên UDP), chủ yếu dùng để giảm thiểu chi phí thiết lập kết nối và giải tỏa ảnh hưởng của vấn đề Head-of-Line Blocking (Nghẽn đầu hàng) của TCP.

Trong HTTP/1.1, tính năng Keep-Alive (kết nối dài) được bật mặc định. Nhờ đó, cùng một kết nối TCP có thể được tái sử dụng cho nhiều HTTP Request, tránh việc mỗi request đều phải thiết lập lại kết nối TCP, giảm thiểu chi phí bắt tay 3 bước (3-way handshake).

Dưới góc độ tái sử dụng kết nối:
- Keep-Alive của HTTP/1.1 giải quyết việc "tái sử dụng 1 kết nối TCP cho nhiều request", nhưng các request trên cùng kết nối vẫn có thể chịu ảnh hưởng nghẽn đầu hàng ở tầng HTTP (xử lý tuần tự).
- HTTP/2 đưa vào Multiplexing trên 1 kết nối TCP, có thể truyền song song nhiều request và response dạng nhị phân (Binary Frames), loại bỏ nghẽn đầu hàng ở tầng HTTP. Tuy nhiên vì tầng dưới vẫn là TCP, nếu một gói TCP bị mất, toàn bộ dữ liệu trên kết nối đó vẫn bị tạm dừng chờ retransmit.
- HTTP/3 dựa trên QUIC (chạy trên UDP), QUIC tự triển khai cơ chế truyền tin cậy và đa luồng độc lập trên từng Stream, giải quyết triệt để vấn đề nghẽn đầu hàng ở tầng TCP.

Ngoài ra, HTTP là một **giao thức phi trạng thái (Stateless Protocol)**. Server về bản chất không tự động ghi nhớ "request trước đó do ai gửi, đang ở trạng thái nào". Vì vậy trong phát triển Web thực tế, chúng ta thường phải dựa vào Cookie, Session, Token (bao gồm JWT) để duy trì trạng thái đăng nhập và phiên làm việc (Session) của người dùng.

## WebSocket: Giao thức giao tiếp song công toàn phần (Full-Duplex)

**WebSocket** là giao thức giao tiếp Full-Duplex dựa trên kết nối TCP, cho phép Client và Server gửi và nhận dữ liệu đồng thời trên cùng một kết nối duy nhất.

![Tổng quan WebSocket](https://oss.javaguide.cn/github/javaguide/cs-basics/network/websocket-overview.png)

Đặc điểm điển hình của nó là: **Sau khi kết nối được thiết lập, Server cũng có thể chủ động đẩy tin nhắn (Push Message) tới Client**. Điều này bù đắp hoàn hảo cho sự thiếu hụt của mô hình Request-Response truyền thống của HTTP trong các kịch bản thời gian thực (Real-time).

Giao thức WebSocket ra đời năm 2008, trở thành tiêu chuẩn quốc tế năm 2011, các trình duyệt hiện đại đều đã hỗ trợ. Không chỉ trên trình duyệt, rất nhiều ngôn ngữ lập trình, framework và máy chủ backend đều hỗ trợ WebSocket.

Bản chất WebSocket vẫn là giao thức tầng ứng dụng. Nó thường khởi đầu bằng một HTTP Request để yêu cầu nâng cấp giao thức (Protocol Upgrade), sau khi nâng cấp thành công (HTTP 101), giữa Client và Server sẽ duy trì một kết nối bền vững để truyền dữ liệu hai chiều.

![Sơ đồ WebSocket](https://oss.javaguide.cn/github/javaguide/system-design/web-real-time-message-push/1460000042192394.png)

Các kịch bản ứng dụng phổ biến của WebSocket:
- Bình luận / Danmaku video trực tiếp
- Đẩy thông báo thời gian thực (Real-time Notification)
- Game online đối kháng thời gian thực
- Chỉnh sửa tài liệu cộng tác nhiều người (Collaborative Editing)
- Chăm sóc khách hàng trực tuyến / Chat, nhắn tin trực tiếp
- Cập nhật giá chứng khoán, tỷ số thể thao trực tiếp

Quy trình hoạt động của WebSocket gồm các bước:
1. Client gửi một HTTP Request tới Server, header chứa `Upgrade: websocket`, `Connection: Upgrade`, `Sec-WebSocket-Key`, thể hiện mong muốn nâng cấp kết nối sang WebSocket.
2. Server nhận được request, nếu hỗ trợ WebSocket sẽ phản hồi mã `101 Switching Protocols` với header `Upgrade: websocket`, `Connection: Upgrade`, `Sec-WebSocket-Accept`, xác nhận nâng cấp thành công.
3. Sau khi nâng cấp, kết nối WebSocket được thiết lập, hai bên có thể truyền dữ liệu hai chiều.
4. Dữ liệu WebSocket được truyền dưới dạng các Khung (Frame). Một thông điệp hoàn chỉnh có thể được chia thành nhiều Frame để gửi, bên nhận sẽ lắp ráp lại.
5. Client hoặc Server đều có thể chủ động gửi Close Frame để đóng kết nối TCP.

Ngoài ra, WebSocket thường kết hợp với **cơ chế Heartbeat** (Ping/Pong frame hoặc heartbeat ở tầng ứng dụng) để kiểm tra kết nối còn sống hay không, tránh tình trạng kết nối "chết giả" (Dead connection).

## SMTP: Giao thức truyền thư đơn giản (Simple Mail Transfer Protocol)

**SMTP (Simple Mail Transfer Protocol)** là giao thức tầng ứng dụng chạy trên TCP, chủ yếu dùng để **gửi và chuyển tiếp email**.

![Tổng quan SMTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/smtp-overview.png)

Lưu ý điểm dễ nhầm lẫn:
**SMTP phụ trách gửi email và chuyển tiếp giữa các Mail Server; POP3 / IMAP phụ trách việc người dùng nhận email từ Mail Server về thiết bị.**

Tức là: email từ server của bạn chuyển tới server của người nhận dùng SMTP; còn người nhận dùng phần mềm (Outlook, Thunderbird, Mail app) để xem thư trong hộp thư thì dùng POP3 hoặc IMAP.

![Giao thức SMTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/what-is-smtp.png)

Các cổng phổ biến của SMTP:

| Cổng | Công dụng phổ biến | Giải thích |
| --- | --- | --- |
| 25 | Chuyển tiếp email giữa các Mail Server | Dùng cho chuyển tiếp MTA tới MTA, nhiều nhà cung cấp Cloud/ISP chặn cổng 25 chiều out để chống Spam |
| 587 | Client gửi (submit) email | Cổng Message Submission chuẩn, thường kết hợp STARTTLS và xác thực tài khoản |
| 465 | Gửi email với TLS ngầm định (Implicit TLS) | Kết nối client trực tiếp thiết lập kênh mã hóa TLS |

### Quy trình gửi Email

Ví dụ email của tôi là `<dabai@cszhinan.com>`, tôi muốn gửi thư tới `<xiaoma@qq.com>`:
1. Tôi soạn thư qua Mail Client hoặc Webmail.
2. Mail Client thông qua giao thức SMTP gửi thư lên Mail Server phụ trách tên miền `cszhinan.com`.
3. Mail Server gửi tra cứu bản ghi MX của tên miền `qq.com` để tìm địa chỉ Mail Server của QQ.
4. Mail Server gửi dùng SMTP chuyển tiếp thư sang QQ Mail Server.
5. QQ Mail Server nhận thư và lưu vào hộp thư người nhận.
6. Người dùng `<xiaoma@qq.com>` thông qua giao thức POP3 hoặc IMAP tải/đọc thư từ QQ Mail Server về máy.

## POP3 / IMAP: Giao thức nhận Email

**POP3 và IMAP đều là giao thức dùng để nhận email**, chạy trên nền TCP ở tầng ứng dụng.

![Tổng quan POP3/IMAP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/pop3-imap-overview.png)

- **POP3 (Post Office Protocol 3)**: Thiết kế đơn giản, mô hình phổ biến là tải toàn bộ email từ server về lưu cục bộ trên máy. Thích hợp khi chỉ dùng 1 thiết bị duyệt mail, nhưng trải nghiệm đồng bộ trên nhiều thiết bị rất kém.
- **IMAP (Internet Message Access Protocol)**: Hiện đại và phổ biến hơn. Nó hỗ trợ quản lý mail trực tiếp trên server, đồng bộ trạng thái (đã đọc, chưa đọc, thùng rác, thư mục, gắn cờ) trên mọi thiết bị (điện thoại, máy tính, web).

So sánh nhanh:

| Giao thức | Công dụng chính | Đặc điểm |
| --- | --- | --- |
| POP3 | Nhận email | Thiên về tải về máy cục bộ, đồng bộ đa thiết bị yếu |
| IMAP | Nhận và quản lý email | Hỗ trợ đồng bộ đa thiết bị, tìm kiếm, phân loại, gắn cờ |
| SMTP | Gửi và chuyển tiếp email | Phụ trách toàn bộ chuỗi gửi/chuyển tiếp thư |

## FTP: Giao thức truyền tệp (File Transfer Protocol)

**FTP (File Transfer Protocol)** là giao thức tầng ứng dụng chạy trên TCP, dùng để truyền file giữa client và server.

![Tổng quan FTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/ftp-overview.png)

Điểm đặc biệt của FTP là nó thường thiết lập **hai kết nối TCP song song**:
1. **Kết nối điều khiển (Control Connection - Cổng 21)**: Dùng để truyền lệnh và phản hồi (đăng nhập, đổi thư mục, xóa file, v.v.).
2. **Kết nối dữ liệu (Data Connection - Cổng 20 hoặc cổng động)**: Dùng để truyền nội dung tệp tin hoặc danh sách thư mục thực sự.

![Quy trình hoạt động của FTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/ftp.png)

FTP có 2 chế độ truyền dữ liệu:
- **Chế độ chủ động (Active Mode - PORT)**: Client gửi lệnh qua kết nối điều khiển báo cho Server biết cổng mình đang lắng nghe, Server chủ động tạo kết nối tới cổng đó của Client để truyền dữ liệu. Nếu Client nằm sau NAT/Firewall, kết nối này rất dễ bị chặn và thất bại.
- **Chế độ bị động (Passive Mode - PASV)**: Client yêu cầu Server mở một cổng dữ liệu ngẫu nhiên, sau đó Client chủ động kết nối tới cổng đó của Server. Vì chiều kết nối luôn là từ Client ra Server nên dễ dàng đi qua NAT/Firewall hơn, do đó thực tế chế độ Passive được dùng phổ biến hơn nhiều.

Lưu ý: FTP truyền Plaintext (không mã hóa). Khi truyền dữ liệu nhạy cảm nên dùng:
- **SFTP (SSH File Transfer Protocol)**: Truyền file an toàn dựa trên nền SSH.
- **FTPS (FTP over TLS/SSL)**: FTP được bọc lớp mã hóa TLS.

## Telnet: Giao thức đăng nhập từ xa (Plaintext)

**Telnet** chạy trên TCP (cổng 23), cho phép người dùng kết nối từ xa vào server qua terminal và thực thi lệnh.

Nhược điểm lớn nhất: **Truyền Plaintext (không mã hóa)**. Toàn bộ tài khoản, mật khẩu và nội dung lệnh đều có thể bị bắt trọn nếu bị nghe lén (Sniffing). Vì vậy ngày nay Telnet đã bị thay thế hoàn toàn bằng SSH trong môi trường production.

## SSH: Giao thức vỏ bảo mật (Secure Shell)

**SSH (Secure Shell)** chạy trên TCP (cổng 22 mặc định), cung cấp cơ chế đăng nhập từ xa, thực thi lệnh và truyền file an toàn thông qua mã hóa và xác thực.

```bash
ssh user@server_ip
```

Ngoài đăng nhập từ xa, SSH còn hỗ trợ:
- Thực thi lệnh từ xa (Remote Command Execution)
- Chuyển tiếp cổng (Port Forwarding)
- Tạo đường hầm mạng (SSH Tunneling / Proxy)
- X11 Forwarding
- Truyền file an toàn qua SFTP hoặc SCP

Cơ chế xác thực phổ biến của SSH gồm mật khẩu (Password Authentication) và khóa công khai (Public Key Authentication). Môi trường production luôn khuyến nghị dùng SSH Key và tắt đăng nhập bằng mật khẩu thông thường.

## RTP: Giao thức truyền tải thời gian thực (Real-time Transport Protocol)

**RTP (Real-time Transport Protocol)** dùng để truyền tải âm thanh, video thời gian thực, thường chạy trên nền UDP.

![Tổng quan RTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/rtp-overview.png)

RTP thường kết hợp cùng RTCP:
- **RTP**: Phụ trách truyền tải gói tin media thực tế.
- **RTCP (RTP Control Protocol)**: Phụ trách truyền thông tin thống kê kiểm soát chất lượng (tỷ lệ mất gói, độ trễ RTT, Jitter) để hai bên điều chỉnh tốc độ bit phù hợp.

Trong WebRTC, RTP/RTCP là nền tảng cốt lõi kết hợp với SRTP (mã hóa), FEC (sửa lỗi tiến), NACK (yêu cầu gửi lại gói mất) để đảm bảo chất lượng gọi video/audio thời gian thực.

## DNS: Hệ thống tên miền (Domain Name System)

**DNS (Domain Name System)** giải quyết việc ánh xạ giữa Tên miền (Domain Name) và Địa chỉ IP.

DNS thường chạy trên UDP (cổng 53). Ưu tiên UDP vì gói tin DNS truy vấn và phản hồi thường nhỏ, không cần tốn chi phí bắt tay 3 bước của TCP. Khi gói phản hồi vượt quá dung lượng (hoặc khi Zone Transfer), DNS sẽ chuyển sang dùng TCP.

Hiện nay còn có các giải pháp DNS bảo mật như **DoH (DNS over HTTPS)** và **DoT (DNS over TLS)** để tránh bị nhà mạng hay kẻ tấn công theo dõi/giả mạo truy vấn DNS.

## Tổng kết cổng các giao thức tầng ứng dụng phổ biến

| Giao thức | Cổng mặc định | Giao thức tầng giao vận | Công dụng chính |
| --- | ---: | --- | --- |
| HTTP | 80 | TCP | Truy cập Web |
| HTTPS | 443 | TCP / QUIC | Truy cập Web mã hóa |
| WebSocket | 80 / 443 | TCP | Giao tiếp hai chiều thời gian thực |
| SMTP | 25 / 465 / 587 | TCP | Gửi và chuyển tiếp Email |
| POP3 | 110 / 995 | TCP | Nhận Email |
| IMAP | 143 / 993 | TCP | Nhận và đồng bộ Email |
| FTP | 20 / 21 | TCP | Truyền tệp tin |
| SSH | 22 | TCP | Đăng nhập từ xa và truyền file an toàn |
| Telnet | 23 | TCP | Đăng nhập từ xa văn bản thuần |
| DNS | 53 | UDP / TCP | Phân giải tên miền |
| RTP | Cổng động (chẵn), RTCP dùng cổng lẻ liền kề | UDP chủ yếu | Truyền tải âm thanh/video thời gian thực |

## Tài liệu tham khảo

- 《Computer Networking: A Top-Down Approach》
- RFC 6455: The WebSocket Protocol
- RFC 9110: HTTP Semantics
- RFC 8446: TLS 1.3
- RFC 9000: QUIC
- RFC 3550: RTP: A Transport Protocol for Real-Time Applications
- RFC 6891: Extension Mechanisms for DNS (EDNS(0))

<!-- @include: @article-footer.snippet.md -->
