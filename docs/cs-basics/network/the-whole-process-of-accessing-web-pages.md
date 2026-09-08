---
title: Từ lúc nhập URL đến khi trang web hiển thị thực sự đã diễn ra những gì? (Tổng hợp)
description: Phân tích toàn diện chuỗi xử lý mạng từ khi nhập URL đến khi trang web hiển thị, xâu chuỗi DNS, TCP, TLS, HTTP, CDN, Gateway và Render trình duyệt.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Nhập URL, Phân giải DNS, Bắt tay TCP, Bắt tay TLS, HTTP Request, Render trình duyệt, Toàn bộ quy trình
---

"Từ lúc nhập URL trên thanh địa chỉ của trình duyệt đến khi trang web hiển thị đầy đủ, toàn bộ quá trình đã diễn ra những gì?"

Đây là một trong những câu hỏi phỏng vấn kinh điển và toàn diện nhất trong ngành kỹ thuật phần mềm. Nó kiểm tra mức độ hiểu biết có hệ thống của ứng viên về toàn bộ kiến trúc mạng máy tính, hệ điều hành và trình duyệt.

Chuỗi xử lý hoàn chỉnh trải qua các giai đoạn chính:

```text
1. Phân tích URL (URL Parsing)
2. Phân giải tên miền (DNS Lookup)
3. Thiết lập kết nối TCP (3-way Handshake)
4. Bắt tay bảo mật TLS (TLS Handshake - nếu là HTTPS)
5. Gửi HTTP Request và nhận HTTP Response
6. Xử lý phía Server (Gateway, Load Balancer, Microservices, Database)
7. Trình duyệt Render trang web (DOM, CSSOM, Render Tree, Paint)
8. Ngắt hoặc duy trì kết nối TCP (Keep-Alive / 4-way Teardown)
```

---

## 1. Phân tích URL và Kiểm tra Cache trình duyệt

Khi người dùng nhập `https://www.javaguide.cn/index.html`:
1. **Phân tích URL**: Trình duyệt bóc tách các thành phần:
   - Giao thức (Protocol): `https` (Cổng mặc định 443)
   - Tên miền (Host): `www.javaguide.cn`
   - Đường dẫn tài nguyên (Path): `/index.html`
2. **Kiểm tra Cache tài nguyên**: Trình duyệt kiểm tra xem tài nguyên `/index.html` có trong bộ nhớ đệm (Browser Cache) hay không (dựa trên `Cache-Control`, `Expires`). Nếu có bản cache còn hạn (Strong Cache), trình duyệt đọc thẳng từ bộ nhớ (Disk/Memory Cache) và bỏ qua việc gửi request qua mạng.
3. **Kiểm tra HSTS (HTTP Strict Transport Security)**: Nếu trang web nằm trong danh sách HSTS, trình duyệt tự động chuyển hướng sang HTTPS mà không gửi HTTP request ban đầu.

---

## 2. Phân giải tên miền (DNS Lookup)

Để kết nối tới server, trình duyệt cần biết địa chỉ IP của tên miền `www.javaguide.cn`. Quá trình tìm kiếm IP diễn ra theo thứ tự phân tầng:

1. **Browser Cache**: Trình duyệt kiểm tra cache DNS của chính nó.
2. **OS Cache & File `hosts`**: Kiểm tra cache DNS của hệ điều hành và file `hosts` cục bộ.
3. **Router Cache**: Kiểm tra cache trên Router mạng cục bộ.
4. **Local DNS Server (ISP)**: Gửi truy vấn DNS đệ quy tới máy chủ DNS của nhà mạng ISP (hoặc Public DNS như 8.8.8.8).
5. **Truy vấn lặp qua hệ thống DNS phân cấp**:
   - Local DNS hỏi **Root DNS Server** -> Nhận địa chỉ của **TLD DNS Server (`.cn`)**.
   - Local DNS hỏi **TLD DNS Server (`.cn`)** -> Nhận địa chỉ của **Authoritative DNS Server (`javaguide.cn`)**.
   - Local DNS hỏi **Authoritative DNS Server** -> Nhận về địa chỉ IP đích (hoặc địa chỉ CNAME của CDN).
6. Local DNS trả IP về cho trình duyệt và lưu cache theo thời gian sống TTL.

---

## 3. Thiết lập kết nối TCP (Bắt tay 3 bước - 3-way Handshake)

Có được địa chỉ IP và cổng (443), trình duyệt khởi tạo kết nối TCP tới server thông qua Socket:

1. **Bước 1 (SYN)**: Client gửi gói tin `SYN=1, seq=x` tới Server, Client chuyển sang trạng thái `SYN_SENT`.
2. **Bước 2 (SYN-ACK)**: Server nhận được SYN, gửi lại gói tin `SYN=1, ACK=1, seq=y, ack=x+1`, Server chuyển sang trạng thái `SYN_RCVD`.
3. **Bước 3 (ACK)**: Client nhận SYN-ACK, gửi lại gói tin `ACK=1, seq=x+1, ack=y+1`, Client và Server cùng chuyển sang trạng thái `ESTABLISHED`.

---

## 4. Bắt tay bảo mật TLS (Nếu là HTTPS)

Đối với giao thức HTTPS, sau khi kết nối TCP được thiết lập, hai bên tiến hành bắt tay TLS (TLS 1.2 hoặc TLS 1.3) để thiết lập kênh mã hóa:

1. **ClientHello**: Client gửi các phiên bản TLS hỗ trợ, danh sách Cipher Suite và `Client Random`.
2. **ServerHello + Certificate**: Server chọn phiên bản TLS, chọn Cipher Suite, gửi `Server Random` và gửi Chứng chỉ số CA.
3. **Xác thực chứng chỉ & Trao đổi khóa**:
   - Client xác minh chứng chỉ số với Root CA tin cậy.
   - Hai bên trao đổi khóa thông qua **ECDHE** (hoặc RSA trong các hệ thống cũ) để tính toán ra `PreMasterSecret`.
4. **Sinh khóa phiên (Session Key)**: Client và Server kết hợp `Client Random`, `Server Random` và `PreMasterSecret` để tạo ra các khóa đối xứng (Symmetric Keys).
5. **Finished**: Hai bên gửi thông điệp mã hóa để xác nhận kênh truyền bảo mật đã sẵn sàng.

---

## 5. Gửi HTTP Request và Nhận HTTP Response

1. **Client gửi HTTP Request**: Trình duyệt đóng gói HTTP Request (gồm Request Line: `GET /index.html HTTP/1.1`, Request Headers: `Host`, `User-Agent`, `Accept`, `Cookie`, v.v.), mã hóa qua TLS rồi gửi qua kết nối TCP.
2. **Phía Server tiếp nhận và xử lý**:
   - **CDN / Edge Node**: Nếu sử dụng CDN, node mạng gần nhất có thể trả về tài nguyên tĩnh ngay lập tức.
   - **Reverse Proxy & Load Balancer (Nginx / F5)**: Tiếp nhận kết nối, giải mã TLS (TLS Termination), cân bằng tải request tới các cụm máy chủ Backend.
   - **API Gateway**: Thực hiện xác thực (Auth), Rate Limiting, Logging, Routing.
   - **Microservices Application (Spring Boot)**: Nhận request, thực thi logic nghiệp vụ qua Service, truy vấn Cache (Redis), truy vấn Cơ sở dữ liệu (MySQL).
3. **Server trả về HTTP Response**: Server đóng gói HTTP Response (Status Line: `HTTP/1.1 200 OK`, Response Headers: `Content-Type: text/html`, `Set-Cookie`, `Cache-Control`, cùng Response Body là mã HTML) và gửi về cho trình duyệt.

---

## 6. Trình duyệt Render trang web

Trình duyệt nhận luồng dữ liệu byte HTML và thực hiện quy trình dựng hình (Critical Rendering Path):

1. **Parse HTML -> Tạo DOM Tree**: Trình duyệt phân tích cú pháp HTML và xây dựng cây mô hình tài liệu (DOM Tree).
2. **Parse CSS -> Tạo CSSOM Tree**: Khi gặp thẻ `<link>` hoặc `<style>`, trình duyệt tải và phân tích CSS để tạo cây mô hình kiểu dáng (CSSOM Tree).
3. **Xử lý JavaScript**: Khi gặp thẻ `<script>`, trình duyệt tạm dừng parse HTML để tải và thực thi JavaScript (trừ khi có thuộc tính `async` hoặc `defer`), JS có thể thao tác thay đổi DOM và CSSOM.
4. **Tạo Render Tree**: Kết hợp DOM Tree và CSSOM Tree để tạo thành cây biểu diễn hình ảnh (Render Tree - chỉ chứa các phần tử thực sự hiển thị trên màn hình).
5. **Layout / Reflow (Tính toán bố cục)**: Tính toán vị trí, kích thước chính xác của từng phần tử trên màn hình thiết bị.
6. **Painting (Vẽ)**: Chuyển đổi các node trong Render Tree thành các pixel thực tế trên màn hình (vẽ màu sắc, viền, bóng, hình ảnh).
7. **Compositing (Tổng hợp lớp)**: Ghép các lớp (layers) lại với nhau và hiển thị lên màn hình người dùng.

---

## 7. Đóng hoặc Duy trì kết nối TCP

- Nếu có header `Connection: keep-alive` (mặc định trong HTTP/1.1 và HTTP/2), kết nối TCP được giữ lại để tiếp tục tải các tài nguyên phụ (hình ảnh, CSS, JS, font) hoặc các request tiếp theo.
- Nếu kết nối cần đóng, hai bên tiến hành **Bắt tay 4 bước (4-way Teardown)**:
  1. Client gửi `FIN` -> Server gửi `ACK`.
  2. Server gửi `FIN` -> Client gửi `ACK` và bước vào trạng thái `TIME_WAIT` (chờ 2 MSL) trước khi đóng hẳn kết nối.

<!-- @include: @article-footer.snippet.md -->
