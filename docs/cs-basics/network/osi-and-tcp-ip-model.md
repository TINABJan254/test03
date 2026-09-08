---
title: Chi tiết Mô hình 7 tầng OSI và Mô hình 4 tầng TCP/IP
description: Phân tích chi tiết mô hình phân tầng và phân chia trách nhiệm của OSI và TCP/IP, kết hợp lịch sử và thực tiễn để so sánh sự khác biệt và sự đánh đổi kỹ thuật giữa hai mô hình.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Mô hình 7 tầng OSI, Mô hình 4 tầng TCP/IP, Mô hình phân tầng, Phân chia trách nhiệm, Protocol Stack, So sánh
---

Phân tầng mạng là bản đồ đầu tiên khi học Mạng máy tính. Nếu không có tấm bản đồ này, các khái niệm như HTTP, TCP, IP, Ethernet, DNS sẽ rất dễ bị lẫn lộn vào nhau, không phân biệt được ai phụ thuộc vào ai, ai chịu trách nhiệm phần nào.

Hai bộ mô hình phân tầng phổ biến là Mô hình 7 tầng OSI và Mô hình 4 tầng TCP/IP. Mô hình trước phù hợp để xây dựng khung khái niệm lý thuyết, còn mô hình sau gắn liền với việc triển khai thực tế của Internet.

Bài viết này chủ yếu trả lời các câu hỏi:

1. Từng tầng trong Mô hình 7 tầng OSI đảm nhận nhiệm vụ gì?
2. Mô hình 4 tầng TCP/IP và Mô hình 7 tầng OSI tương ứng với nhau như thế nào?
3. Tại sao mô hình OSI hoàn chỉnh về mặt lý thuyết nhưng lại không trở thành hiện thực triển khai chủ lưu của Internet?
4. Khi học một giao thức mạng cụ thể, tại sao trước tiên cần biết nó nằm ở tầng nào?

## Mô hình 7 tầng OSI

**Mô hình 7 tầng OSI** (Open Systems Interconnection) là mô hình phân tầng mạng do Tổ chức Tiêu chuẩn hóa Quốc tế (ISO) đề xuất. Cấu trúc tổng thể và chức năng của từng tầng được thể hiện như hình dưới đây:

![Phân chia chức năng các tầng trong mô hình 7 tầng OSI](https://oss.javaguide.cn/github/javaguide/cs-basics/network/osi-7-model.png)

Mỗi tầng đều tập trung làm một nhóm nhiệm vụ riêng biệt, và mỗi tầng đều cần sử dụng chức năng do tầng dưới cung cấp. Ví dụ: Tầng Giao vận (Transport Layer) cần sử dụng chức năng định tuyến (Routing) và đánh địa chỉ (Addressing) do Tầng Mạng (Network Layer) cung cấp, nhờ đó Tầng Giao vận mới biết phải chuyển dữ liệu tới đâu.

**Kiến trúc 7 tầng của OSI có khái niệm rõ ràng và lý thuyết rất hoàn chỉnh, nhưng nó tương đối phức tạp, kém thực tế, và một số chức năng bị trùng lặp ở nhiều tầng.**

Hình trên có thể hơi trừu tượng, hãy xem một bức ảnh sinh động dưới đây:

![Mô hình 7 tầng OSI 2](https://oss.javaguide.cn/github/javaguide/osi七层模型2.png)

**Mô hình 7 tầng OSI lý thuyết hoàn hảo như vậy, tại sao lại thua Mô hình 4 tầng TCP/IP?**

Quả thực, mô hình 7 tầng OSI thời đó từng được rất nhiều tập đoàn lớn, thậm chí chính phủ các quốc gia ủng hộ mạnh mẽ. Trong bối cảnh đó, tại sao nó lại thất bại? Chủ yếu do các nguyên nhân sau:

1. Các chuyên gia của OSI thiếu kinh nghiệm thực tế, họ thiếu động lực thương mại khi hoàn thiện tiêu chuẩn OSI.
2. Việc triển khai các giao thức của OSI quá phức tạp và hiệu suất vận hành rất thấp.
3. Chu kỳ xây dựng tiêu chuẩn của OSI quá dài, khiến các thiết bị sản xuất theo tiêu chuẩn OSI không kịp ra mắt thị trường (Đầu thập niên 90, dù toàn bộ tiêu chuẩn quốc tế OSI đã được ban hành, nhưng Internet dựa trên TCP/IP đã sớm vận hành thành công trên phạm vi toàn cầu).
4. Phân chia các tầng của OSI chưa thực sự hợp lý, một số chức năng bị lặp lại ở nhiều tầng khác nhau.

Mặc dù mô hình 7 tầng OSI thất bại trên thực tế, nhưng nó lại cung cấp nền tảng lý thuyết rất tuyệt vời. Để hiểu sâu hơn về phân tầng mạng, mô hình OSI vẫn là kiến thức rất cần thiết phải học.

Dưới đây là hình ảnh tổng kết mối quan hệ tương ứng giữa hai mô hình:

![Mối quan hệ tương ứng giữa Mô hình 7 tầng OSI và Mô hình 4 tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/osi-model-detail.png)

## Mô hình 4 tầng TCP/IP

**Mô hình 4 tầng TCP/IP** là mô hình đang được áp dụng rộng rãi nhất hiện nay. Chúng ta có thể xem mô hình TCP/IP là phiên bản tinh gọn của mô hình 7 tầng OSI, bao gồm 4 tầng sau:

1. Tầng Ứng dụng (Application Layer)
2. Tầng Giao vận (Transport Layer)
3. Tầng Mạng (Network Layer / Internet Layer)
4. Tầng Giao diện mạng (Network Interface Layer / Link Layer)

Cần lưu ý rằng chúng ta không thể ánh xạ hoàn toàn chính xác 100% giữa TCP/IP 4 tầng và OSI 7 tầng, nhưng có thể đối chiếu một cách đơn giản như sau:

![Phân chia chức năng các tầng trong mô hình 4 tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

### Tầng Ứng dụng (Application Layer)

**Tầng Ứng dụng nằm trên Tầng Giao vận, chủ yếu cung cấp dịch vụ trao đổi thông tin giữa các tiến trình ứng dụng trên hai thiết bị đầu cuối. Nó định nghĩa định dạng trao đổi thông tin, và message sẽ được giao cho Tầng Giao vận bên dưới để truyền tải.** Đơn vị dữ liệu trao đổi ở tầng ứng dụng được gọi là Thông điệp (Message / Báo văn).

![Quy trình phối hợp của mô hình mạng 5 tầng trong một lần truyền dữ liệu](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-five-layer-sample-diagram.png)

Giao thức tầng ứng dụng định nghĩa các quy tắc giao tiếp mạng, đối với các ứng dụng mạng khác nhau sẽ cần các giao thức tầng ứng dụng khác nhau. Trên Internet có rất nhiều giao thức tầng ứng dụng, ví dụ giao thức HTTP hỗ trợ Web, giao thức SMTP hỗ trợ Email, v.v.

**Các giao thức phổ biến ở Tầng Ứng dụng**:

![Các giao thức phổ biến ở tầng ứng dụng](https://oss.javaguide.cn/github/javaguide/cs-basics/network/application-layer-protocol.png)

- **HTTP (Hypertext Transfer Protocol)**: Dựa trên TCP, là giao thức dùng để truyền siêu văn bản và nội dung đa phương tiện, được thiết kế chủ yếu cho giao tiếp giữa trình duyệt Web và máy chủ Web. Khi duyệt web, các trang web được tải về thông qua các HTTP Request.
- **SMTP (Simple Mail Transfer Protocol)**: Dựa trên TCP, là giao thức dùng để gửi và chuyển tiếp email. Lưu ý ⚠️: SMTP chỉ phụ trách gửi mail, không phụ trách nhận mail. Để nhận mail từ mail server, cần sử dụng giao thức POP3 hoặc IMAP.
- **POP3 / IMAP (Giao thức nhận email)**: Dựa trên TCP, cả hai đều phụ trách nhận email. IMAP ra đời sau POP3, mạnh mẽ hơn cả về tính năng lẫn hiệu năng. IMAP hỗ trợ tìm kiếm, đánh dấu, phân loại thư mục, lưu trữ trên server và đồng bộ trạng thái mail trên nhiều thiết bị. Hầu hết các mail client và server hiện đại đều hỗ trợ IMAP.
- **FTP (File Transfer Protocol)**: Dựa trên TCP, là giao thức dùng để truyền tệp tin giữa các máy tính, che giấu sự khác biệt về hệ điều hành và cách lưu trữ tệp. Lưu ý ⚠️: FTP là giao thức không an toàn vì không mã hóa dữ liệu trong quá trình truyền tải. Khuyên dùng SFTP khi truyền dữ liệu nhạy cảm.
- **Telnet (Giao thức đăng nhập từ xa)**: Dựa trên TCP, dùng để đăng nhập từ terminal vào các server khác. Nhược điểm lớn nhất của Telnet là toàn bộ dữ liệu (bao gồm username và password) đều gửi dưới dạng Plaintext (văn bản thuần), tiềm ẩn rủi ro bảo mật lớn. Ngày nay Telnet hầu như đã bị thay thế hoàn toàn bởi giao thức SSH an toàn.
- **SSH (Secure Shell Protocol)**: Dựa trên TCP, cung cấp khả năng truy cập từ xa và truyền tệp an toàn thông qua cơ chế mã hóa và xác thực.
- **RTP (Real-time Transport Protocol)**: Thường chạy trên UDP (nhưng cũng hỗ trợ TCP). Cung cấp tính năng truyền dữ liệu thời gian thực end-to-end (âm thanh, video), không đảm bảo chất lượng truyền tải thời gian thực nếu không có cơ chế bù đắp (như WebRTC).
- **DNS (Domain Name System)**: Thường chạy trên UDP (cổng 53), giải quyết bài toán ánh xạ giữa Tên miền và Địa chỉ IP. Khi dữ liệu phản hồi vượt quá giới hạn hoặc khi thực hiện truyền vùng (Zone Transfer), DNS sẽ chuyển sang dùng TCP.

Chi tiết về các giao thức này xem tại bài viết [Tổng hợp các giao thức tầng ứng dụng thường gặp](./application-layer-protocol.md).

### Tầng Giao vận (Transport Layer)

**Nhiệm vụ chính của Tầng Giao vận là cung cấp dịch vụ truyền dữ liệu tổng quát cho giao tiếp giữa các tiến trình của hai thiết bị đầu cuối.** Tiến trình ứng dụng sử dụng dịch vụ này để truyền Message của tầng ứng dụng. "Tổng quát" ở đây có nghĩa là không nhắm tới một ứng dụng mạng cụ thể nào, mà nhiều ứng dụng đều có thể dùng chung dịch vụ của tầng giao vận.

**Các giao thức phổ biến ở Tầng Giao vận**:

![Các giao thức phổ biến ở tầng giao vận](https://oss.javaguide.cn/github/javaguide/cs-basics/network/transport-layer-protocol.png)

- **TCP (Transmission Control Protocol)**: Cung cấp dịch vụ truyền dữ liệu **hướng kết nối (Connection-oriented)** và **đáng tin cậy (Reliable)**.
- **UDP (User Datagram Protocol)**: Cung cấp dịch vụ truyền dữ liệu **không kết nối (Connectionless)**, **nỗ lực tối đa (Best-effort)** (không đảm bảo độ tin cậy của việc truyền dữ liệu), đơn giản và hiệu suất cao.

### Tầng Mạng (Network Layer)

**Tầng Mạng chịu trách nhiệm cung cấp dịch vụ giao tiếp cho các host khác nhau trên mạng chuyển mạch gói (packet-switched network).** Khi gửi dữ liệu, tầng mạng đóng gói Segment (của TCP) hoặc Datagram (của UDP) từ tầng giao vận thành Packet để truyền đi. Trong kiến trúc TCP/IP, vì tầng mạng dùng giao thức IP nên gói tin còn được gọi là IP Datagram.

⚠️ Lưu ý: **Đừng nhầm lẫn giữa "UDP User Datagram" của tầng giao vận và "IP Datagram" của tầng mạng**.

**Một nhiệm vụ quan trọng khác của tầng mạng là chọn đường đi phù hợp (Routing) để gói tin từ máy nguồn tìm được máy đích thông qua các Router trong mạng.**

Cần nhấn mạnh rằng chữ "Mạng" trong Tầng Mạng không phải là một mạng vật lý cụ thể, mà là tên gọi của tầng thứ 3 trong mô hình kiến trúc mạng máy tính.

Internet được kết nối từ rất nhiều mạng không đồng nhất (heterogeneous networks) thông qua các bộ định tuyến (Router). Giao thức tầng mạng mà Internet sử dụng là giao thức không kết nối IP (Internet Protocol) cùng nhiều giao thức định tuyến, do đó tầng mạng của Internet còn được gọi là **Tầng IP (IP Layer)**.

**Các giao thức phổ biến ở Tầng Mạng**:

![Các giao thức phổ biến ở tầng mạng](./images/network-model/nerwork-layer-protocol.png)

- **IP (Internet Protocol)**: Một trong những giao thức quan trọng nhất trong bộ TCP/IP, tác dụng chính là định nghĩa định dạng gói tin, định tuyến và đánh địa chỉ gói tin để chúng có thể truyền qua các mạng và đến đúng đích. Hiện tại gồm IPv4 và IPv6.
- **ARP (Address Resolution Protocol)**: Giải quyết vấn đề chuyển đổi giữa địa chỉ tầng mạng (IP) và địa chỉ tầng liên kết (MAC). Trong quá trình truyền vật lý, gói tin IP luôn cần biết chặng tiếp theo (Next Hop) phải đi đến đâu về mặt vật lý, ARP giải quyết bài toán chuyển đổi từ IP sang MAC.
- **ICMP (Internet Control Message Protocol)**: Giao thức truyền trạng thái mạng và thông báo lỗi, thường dùng để chẩn đoán mạng và khắc phục sự cố. Ví dụ công cụ `Ping` sử dụng ICMP để kiểm tra tính thông suốt của mạng.
- **NAT (Network Address Translation)**: Giao thức chuyển đổi địa chỉ mạng giữa mạng nội bộ (LAN) và mạng công cộng (WAN). Trong mạng LAN, các máy dùng IP nội bộ; khi ra mạng ngoài WAN, router NAT sẽ chuyển đổi sang một hoặc nhiều IP Public thống nhất.
- **OSPF (Open Shortest Path First)**: Giao thức định tuyến nội bộ (IGP) thuộc dạng động, dựa trên thuật toán trạng thái liên kết (Link-State), tính đến băng thông, độ trễ để chọn đường đi tối ưu.
- **RIP (Routing Information Protocol)**: Giao thức định tuyến nội bộ (IGP) dựa trên thuật toán Distance-Vector, dùng số bước nhảy (Hop Count) làm thước đo, chọn đường đi ít hop nhất.
- **BGP (Border Gateway Protocol)**: Giao thức định tuyến giữa các Autonomous System (AS) trên Internet để trao đổi thông tin tiếp cận tầng mạng (NLRI).

### Tầng Giao diện mạng (Network Interface Layer)

Chúng ta có thể xem Tầng Giao diện mạng là sự kết hợp của Tầng Liên kết dữ liệu (Data Link Layer) và Tầng Vật lý (Physical Layer).

1. Tầng Liên kết dữ liệu (Data Link Layer): **Có nhiệm vụ đóng gói IP Datagram từ tầng mạng thành Khung (Frame), và truyền các Frame trên đoạn liên kết giữa hai node kế cận. Mỗi Frame gồm dữ liệu và các thông tin điều khiển cần thiết (đồng bộ, địa chỉ, kiểm soát lỗi CRC, v.v.).**
2. **Tầng Vật lý (Physical Layer): Có nhiệm vụ truyền tải trong suốt luồng bit (Bitstream) giữa các node máy tính liền kề, che giấu tối đa sự khác biệt của môi trường truyền dẫn và thiết bị vật lý.**

Chức năng và giao thức quan trọng của Tầng Giao diện mạng:

![Chức năng và giao thức tầng giao diện mạng](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-interface-layer-protocol.png)

### Tổng kết

Tổng kết nhanh các giao thức và kỹ thuật cốt lõi ở từng tầng:

![Tổng quan các giao thức theo tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-protocol-overview.png)

**Giao thức Tầng Ứng dụng**:
- HTTP (Hypertext Transfer Protocol)
- SMTP (Simple Mail Transfer Protocol)
- POP3 / IMAP
- FTP (File Transfer Protocol)
- Telnet
- SSH (Secure Shell Protocol)
- RTP (Real-time Transport Protocol)
- DNS (Domain Name System)
- ……

**Giao thức Tầng Giao vận**:
- TCP
  - Cấu trúc Segment
  - Truyền dữ liệu tin cậy (Reliable Data Transfer)
  - Kiểm soát luồng (Flow Control)
  - Kiểm soát tắc nghẽn (Congestion Control)
- UDP
  - Cấu trúc Datagram
  - Không kết nối, đơn giản, nhanh

**Giao thức Tầng Mạng**:
- IP (IPv4, IPv6)
- ARP (Address Resolution Protocol)
- ICMP (Internet Control Message Protocol)
- NAT (Network Address Translation)
- OSPF, RIP, BGP (Định tuyến)
- ……

**Tầng Giao diện mạng**:
- Kiểm tra và phát hiện lỗi (Error Detection - CRC)
- Đa truy cập kênh truyền (CSMA/CD)
- Địa chỉ MAC
- Công nghệ Ethernet
- ……

## Lý do mạng cần phải phân tầng

Ở phần cuối bài viết, hãy cùng bàn luận: "Tại sao mạng lại phải phân tầng?".

Nói về phân tầng, hãy liên hệ với việc chúng ta phát triển một ứng dụng Backend: chúng ta thường chia hệ thống thành 3 tầng (hệ thống phức tạp còn nhiều tầng hơn):
1. Repository / DAO (Thao tác cơ sở dữ liệu)
2. Service (Thao tác nghiệp vụ)
3. Controller (Tương tác dữ liệu với Frontend)

**Hệ thống phức tạp cần phân tầng vì mỗi tầng cần tập trung vào một nhóm nhiệm vụ chuyên biệt. Phân tầng mạng cũng vậy, mỗi tầng chỉ tập trung xử lý một loại công việc.**

Cụ thể, có 3 lý do chính:

1. **Các tầng độc lập với nhau**: Mỗi tầng không cần quan tâm tầng khác được triển khai cụ thể ra sao, chỉ cần biết cách gọi chức năng do tầng dưới cung cấp (tương tự như gọi Interface/API).
2. **Nâng cao tính linh hoạt tổng thể**: Mỗi tầng đều có thể sử dụng công nghệ phù hợp nhất để triển khai, chỉ cần đảm bảo quy tắc giao tiếp và giao diện cung cấp không đổi. Điều này tương ứng với nguyên tắc **High Cohesion, Low Coupling** (Độ gắn kết cao, độ phụ thuộc thấp) trong công nghệ phần mềm.
3. **Chia nhỏ bài toán lớn**: Phân tầng giúp phân rã bài toán giao tiếp mạng phức tạp thành nhiều bài toán nhỏ, có ranh giới rõ ràng và đơn giản hơn để xử lý. Nhờ đó giúp hệ thống mạng dễ thiết kế, triển khai và chuẩn hóa.

Một câu nói kinh điển trong ngành khoa học máy tính:

> Mọi vấn đề trong khoa học máy tính đều có thể giải quyết bằng cách thêm một tầng trung gian gián tiếp (indirection layer), và toàn bộ hệ thống máy tính từ trên xuống dưới đều được thiết kế theo cấu trúc phân tầng chặt chẽ.

## Tài liệu tham khảo

- TCP/IP model vs OSI model: <https://fiberbit.com.tw/tcpip-model-vs-osi-model/>
- Data Encapsulation and the TCP/IP Protocol Stack: <https://docs.oracle.com/cd/E19683-01/806-4075/ipov-32/index.html>

<!-- @include: @article-footer.snippet.md -->
