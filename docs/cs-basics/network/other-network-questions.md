---
title: Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (Phần 1)
description: Tổng hợp chi tiết các câu hỏi phỏng vấn mạng máy tính tần suất cao, bao gồm mô hình phân tầng OSI/TCP-IP, HTTP, HTTPS, DNS, Cookie/Session/Token và các kiến thức nền tảng.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Phỏng vấn Mạng máy tính, Câu hỏi phỏng vấn, OSI, TCP/IP, HTTP, HTTPS, DNS, Cookie, Session, Token
---

<!-- @include: @article-header.snippet.md -->

Phần 1 của bộ câu hỏi phỏng vấn Mạng máy tính tập trung vào các chủ đề: **Mô hình phân tầng, Tầng Ứng dụng (HTTP, HTTPS, DNS) và Quản lý phiên (Cookie/Session/Token)**.

---

## 1. Mô hình phân tầng và Khái niệm cơ bản

### Câu 1: Trình bày sự khác biệt giữa Mô hình 7 tầng OSI và Mô hình 4 tầng TCP/IP?
- **Mô hình OSI (7 tầng)**: Ứng dụng (Application), Trình diễn (Presentation), Phiên (Session), Giao vận (Transport), Mạng (Network), Liên kết dữ liệu (Data Link), Vật lý (Physical). Mang tính chuẩn hóa lý thuyết.
- **Mô hình TCP/IP (4 tầng)**: Ứng dụng (gộp 3 tầng trên của OSI), Giao vận, Mạng (Internet), Giao diện mạng (Link + Physical). Là tiêu chuẩn thực tế của Internet ngày nay.

### Câu 2: Tại sao kiến trúc mạng máy tính lại cần phải phân tầng?
1. **Độc lập và module hóa**: Mỗi tầng chỉ tập trung xử lý một nhóm nhiệm vụ chuyên biệt, che giấu chi tiết triển khai với các tầng khác.
2. **Linh hoạt và dễ bảo trì**: Thay đổi công nghệ ở một tầng không làm ảnh hưởng đến các tầng khác (nguyên tắc High Cohesion, Low Coupling).
3. **Chuẩn hóa và chia nhỏ bài toán**: Giúp việc thiết kế thiết bị phần cứng và phần mềm mạng của các hãng khác nhau dễ dàng tương thích với nhau.

---

## 2. Giao thức HTTP và HTTPS

### Câu 3: HTTP và HTTPS khác nhau như thế nào?
1. **Bảo mật**: HTTP truyền Plaintext (không mã hóa, không xác thực), HTTPS sử dụng TLS để mã hóa kênh truyền, bảo đảm tính toàn vẹn và xác thực danh tính server qua chứng chỉ số CA.
2. **Cổng mặc định**: HTTP là `80`, HTTPS là `443`.
3. **Chi phí và hiệu năng**: HTTPS tốn thêm tài nguyên CPU để mã hóa/giải mã và tăng độ trễ bắt tay TLS ban đầu.

### Câu 4: Quá trình bắt tay HTTPS diễn ra như thế nào?
- **Bắt tay RSA (TLS 1.2 cũ)**: Client gửi `ClientHello` -> Server gửi `ServerHello + Certificate` -> Client xác thực chứng chỉ, tạo `PreMasterSecret` mã hóa bằng Public Key của Server và gửi qua `ClientKeyExchange` -> Server dùng Private Key giải mã -> Hai bên phái sinh ra `MasterSecret` dùng cho mã hóa đối xứng.
- **Bắt tay ECDHE (TLS 1.2 / TLS 1.3 hiện đại)**: Hai bên trao đổi Public Key tạm thời của thuật toán Diffie-Hellman trên đường cong Elliptic và tự tính toán ra khóa dùng chung độc lập, đảm bảo **Forward Secrecy** (Bảo mật chuyển tiếp).

### Câu 5: HTTP/1.0, HTTP/1.1 và HTTP/2 khác nhau như thế nào?
- **HTTP/1.0**: Mặc định Short Connection, mỗi request là 1 kết nối TCP mới.
- **HTTP/1.1**: Mặc định Persistent Connection (Keep-Alive), bổ sung Host Header, Range Request, Caching (`Cache-Control`, `ETag`), Chunked Transfer.
- **HTTP/2**: Chuyển sang định dạng nhị phân (Binary Framing), hỗ trợ **Multiplexing** (đa ghép kênh song song trên 1 kết nối TCP), nén Header (HPACK), Server Push.
- **HTTP/3**: Chuyển sang chạy trên **QUIC (UDP)**, loại bỏ triệt để vấn đề Head-of-Line Blocking của TCP.

---

## 3. Quản lý trạng thái: Cookie, Session và Token (JWT)

### Câu 6: Phân biệt Cookie và Session?
- **Cookie**: Được lưu trữ tại Trình duyệt Client, gửi kèm trong Header `Cookie` của mỗi HTTP request, dễ bị can thiệp và giới hạn dung lượng khoảng 4KB.
- **Session**: Được lưu trữ tại Server (trong RAM, Database hoặc Redis), Client chỉ giữ `SessionID` (thường lưu trong Cookie). Bảo mật hơn nhưng tốn bộ nhớ server và khó scale phân tán nếu không dùng Centralized Session (Redis).

### Câu 7: Phân biệt Session và Token (JWT)?
- **Session**: Stateful (Server phải lưu trạng thái phiên), khó mở rộng khi cụm server scale out.
- **JWT (JSON Web Token)**: Stateless (Server không cần lưu trạng thái), toàn bộ thông tin user và quyền hạn được mã hóa và ký số trong Token. Client lưu token và gửi kèm trong header `Authorization: Bearer <token>`. Phù hợp tuyệt đối cho kiến trúc Microservices và RESTful API.

---

## 4. Hệ thống tên miền DNS

### Câu 8: DNS phân giải tên miền như thế nào? Phân biệt Recursive và Iterative Query?
- **Recursive Query (Truy vấn đệ quy)**: Client hỏi Local DNS Server, Local DNS có trách nhiệm hỏi các server khác đến cùng và trả kết quả cuối cùng về cho Client.
- **Iterative Query (Truy vấn lặp)**: Local DNS lần lượt hỏi Root Server -> TLD Server -> Authoritative Server, mỗi server chỉ trả về địa chỉ của server ở cấp tiếp theo để Local DNS tự đi hỏi tiếp.

### Câu 9: DNS sử dụng UDP hay TCP?
- Mặc định sử dụng **UDP cổng 53** cho các truy vấn thông thường vì tốc độ nhanh, gói tin nhỏ (< 512 bytes).
- Chuyển sang **TCP cổng 53** khi gói phản hồi vượt quá 512 bytes hoặc khi thực hiện truyền vùng (**Zone Transfer**) giữa máy chủ DNS Master và Slave.

<!-- @include: @article-footer.snippet.md -->
