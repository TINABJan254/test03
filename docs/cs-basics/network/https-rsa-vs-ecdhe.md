---
title: RSA và ECDHE trong bắt tay HTTPS: Khác biệt cốt lõi ở đâu? (Tầng ứng dụng)
description: So sánh sự khác biệt cốt lõi giữa trao đổi khóa RSA và ECDHE trong bắt tay TLS, làm rõ Forward Secrecy (Bảo mật chuyển tiếp), cấu trúc đặt tên Cipher Suite, thay đổi trong TLS 1.3 và các điểm phỏng vấn trọng tâm.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: HTTPS, RSA, ECDHE, TLS, Bắt tay, Forward Secrecy, Trao đổi khóa, Cipher Suite, TLS 1.3, PreMasterSecret
---

Nhiều người khi mới học về HTTPS thường có một ấn tượng khá đơn giản:

**HTTPS = HTTP + Mã hóa, Mã hóa = RSA. Do đó, HTTPS = Mã hóa RSA.**

Cách hiểu này xuất phát từ việc các tài liệu nhập môn trước đây thường lấy RSA làm ví dụ minh họa kinh điển.

Tuy nhiên, HTTPS chưa bao giờ đồng nghĩa với việc chỉ dùng mã hóa RSA. Ngay cả trong thời kỳ TLS 1.0, TLS 1.1, RSA cũng chỉ là một trong các phương án trao đổi khóa tùy chọn bên cạnh DHE. Đến chuẩn **TLS 1.3**, cơ chế trao đổi khóa RSA tĩnh (Static RSA Key Exchange) đã bị loại bỏ hoàn toàn, RSA hiện nay chủ yếu chỉ còn được dùng trong việc Ký số chứng chỉ (Certificate Signature) và Xác thực danh tính (Authentication).

Điểm khác biệt cốt lõi cần so sánh:

**Trong bắt tay RSA: Khóa phiên (Session Key Material) do Client tạo ra rồi dùng Public Key của Server mã hóa và gửi sang Server.**
**Trong bắt tay ECDHE: Khóa phiên không được truyền trực tiếp qua mạng, mà do Client và Server độc lập tự tính toán ra thông qua thuật toán trao đổi khóa Diffie-Hellman trên đường cong Elliptic.**

Bài viết này chủ yếu trả lời các câu hỏi:

1. Tại sao HTTPS không đồng nghĩa với mã hóa RSA?
2. Khóa phiên trong bắt tay RSA và ECDHE lần lượt được sinh ra như thế nào?
3. Tại sao ECDHE cung cấp được tính năng Forward Secrecy (Bảo mật chuyển tiếp)?
4. Tại sao TLS 1.3 lại loại bỏ hoàn toàn cơ chế trao đổi khóa RSA tĩnh?

![Khác biệt cốt lõi giữa trao đổi khóa RSA và ECDHE](https://oss.javaguide.cn/github/javaguide/cs-basics/network/https-rsa-ecdhe-rsa-and-ecdhe-key-exchange-core-differences.png)

## Hai bài toán cốt lõi của bắt tay TLS

Trong giao tiếp HTTPS, sau khi bắt tay hoàn tất, thuật toán thực sự mã hóa dữ liệu HTTP là các thuật toán mã hóa đối xứng như AES-GCM, ChaCha20-Poly1305, chứ không phải dùng RSA mã hóa từng request.

Quá trình bắt tay phải giải quyết 2 bài toán:

1. **Bài toán 1: Client và Server cần thỏa thuận ra một khóa phiên dùng chung (Session Key)** để mã hóa đối xứng dữ liệu nghiệp vụ về sau.
2. **Bài toán 2: Client cần xác thực Server thực sự là máy chủ chính chủ** (chống tấn công Man-in-the-Middle) thông qua Chứng chỉ số CA.

RSA và ECDHE giải quyết bài toán "Làm sao để có khóa phiên dùng chung" theo hai cách hoàn toàn khác nhau.

## Bắt tay RSA: Mã hóa gửi khóa bí mật

### Quy trình bắt tay TLS 1.2 với RSA

1. **ClientHello**: Trình duyệt gửi phiên bản TLS hỗ trợ, danh sách Cipher Suites và một số ngẫu nhiên `Client Random`.
2. **ServerHello + Certificate**: Server chọn phiên bản TLS, Cipher Suite, gửi số ngẫu nhiên `Server Random` và gửi Chứng chỉ số chứa Public Key RSA của Server.
3. **Client xác thực chứng chỉ**: Client kiểm tra chuỗi chứng chỉ CA, tên miền, hạn dùng. Nếu hợp lệ, Client trích xuất Public Key RSA của Server từ chứng chỉ.
4. **Client tạo và gửi PreMasterSecret**: Client tự sinh một chuỗi ngẫu nhiên 48 byte gọi là `PreMasterSecret`. Client dùng Public Key RSA của Server mã hóa chuỗi này và gửi qua thông điệp `Client Key Exchange`.
5. **Server giải mã**: Server dùng Private Key RSA của mình để giải mã gói tin và lấy được `PreMasterSecret`.
6. **Sinh Master Secret**: Lúc này cả hai bên đều có đủ 3 mảnh ghép:
   ```text
   Client Random
   Server Random
   PreMasterSecret
   ```
   Hai bên độc lập dùng hàm PRF (Pseudo-Random Function) kết hợp 3 giá trị trên để tính toán ra cùng một `Master Secret`, từ đó phái sinh ra các Symmetric Key dùng cho mã hóa dữ liệu.

Tóm gọn: **Trong RSA, khóa bí mật do Client tạo rồi đóng gói bằng Public Key của Server để gửi đi qua mạng.**

### Nhược điểm: Không có Forward Secrecy (Bảo mật chuyển tiếp)

Giả sử kẻ tấn công liên tục ghi lại và lưu trữ toàn bộ lưu lượng HTTPS mã hóa trên đường truyền trong nhiều năm.

Nếu một ngày nào đó trong tương lai, Private Key RSA của Server bị rò rỉ (do lộ mã nguồn, hacker tấn công server, v.v.):
- Kẻ tấn công có Private Key của Server.
- Kẻ tấn công mở lại các gói tin bắt tay đã lưu trong quá khứ, giải mã `Client Key Exchange` để lấy `PreMasterSecret`.
- Kết hợp với `Client Random` và `Server Random` (vốn truyền dạng plaintext), kẻ tấn công tính lại được `Master Secret`.
- **Toàn bộ dữ liệu lịch sử trong quá khứ bị giải mã hoàn toàn!**

Đây chính là lý do cơ chế RSA Key Exchange thiếu tính năng **Forward Secrecy**.

## Bắt tay ECDHE: Tự tính toán khóa chung với Forward Secrecy

**ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)** là thuật toán thỏa thuận khóa dựa trên đường cong Elliptic với các cặp khóa tạm thời (Ephemeral Key).

### Quy trình bắt tay ECDHE

1. **ClientHello**: Gửi `Client Random`, các Cipher Suite hỗ trợ và các thông số đường cong Elliptic hỗ trợ.
2. **ServerHello + Certificate + Server Key Exchange**:
   - Server gửi `Server Random`, chứng chỉ số.
   - Server tự sinh một cặp khóa tạm thời ECDHE (gồm Private Key tạm thời và Public Key tạm thời $Pub_S$).
   - Server gửi $Pub_S$ trong thông điệp `Server Key Exchange`, đồng thời dùng Private Key trong chứng chỉ để ký số lên $Pub_S$ (để chống giả mạo).
3. **Client Key Exchange**:
   - Client xác thực chứng chỉ và xác thực chữ ký trên $Pub_S$.
   - Client tự sinh một cặp khóa tạm thời ECDHE của riêng mình (Private Key tạm thời và Public Key tạm thời $Pub_C$).
   - Client gửi $Pub_C$ cho Server qua `Client Key Exchange`.
4. **Độc lập tính toán PreMasterSecret**:
   - Client dùng Private Key tạm thời của mình kết hợp với $Pub_S$ của Server để tính toán ra `PreMasterSecret`.
   - Server dùng Private Key tạm thời của mình kết hợp với $Pub_C$ của Client để tính toán ra `PreMasterSecret`.
   - Nhờ tính chất toán học của thuật toán Diffie-Hellman trên đường cong Elliptic, **cả hai bên đều tính ra kết quả `PreMasterSecret` giống hệt nhau** mà không bên nào phải truyền giá trị này qua mạng!
5. **Sinh Master Secret**: Kết hợp `Client Random`, `Server Random` và `PreMasterSecret` để tạo Symmetric Key bảo vệ dữ liệu.

### Tại sao ECDHE đạt được Forward Secrecy?

Các cặp khóa ECDHE được tạo mới cho từng phiên kết nối (Ephemeral) và bị hủy ngay sau khi phiên kết thúc (chỉ lưu trong RAM).

Dù Private Key của chứng chỉ Server có bị lộ trong tương lai:
- Kẻ tấn công chỉ có thể dùng nó để giả mạo danh tính Server trong các kết nối mới.
- Kẻ tấn công **không thể giải mã các phiên giao dịch trong quá khứ**, vì Private Key tạm thời ECDHE của các phiên cũ đã bị xóa vĩnh viễn khỏi bộ nhớ của Client và Server.

## Tại sao TLS 1.3 loại bỏ hoàn toàn Static RSA Key Exchange?

TLS 1.3 (RFC 8446) được thiết kế với 2 mục tiêu lớn: **Tăng tính bảo mật** và **Giảm độ trễ (1-RTT / 0-RTT)**.

1. **Bắt buộc Forward Secrecy**: Loại bỏ RSA Key Exchange để đảm bảo mọi kết nối TLS 1.3 đều có Forward Secrecy thông qua (EC)DHE hoặc PSK.
2. **Đơn giản hóa và tối ưu bắt tay**:
   - Trong TLS 1.2 ECDHE, mất 2 RTT để hoàn tất bắt tay.
   - Trong TLS 1.3, Client gửi luôn tham số khóa ECDHE của mình ngay trong `ClientHello` (`Key Share`), giúp hoàn tất bắt tay chỉ trong **1 RTT** (thậm chí 0-RTT khi khôi phục phiên).

## So sánh tổng kết

| Tiêu chí | Bắt tay RSA | Bắt tay ECDHE |
| --- | --- | --- |
| Cơ chế tạo khóa phiên | Client tạo `PreMasterSecret` và mã hóa gửi cho Server | Hai bên trao đổi Public Key tạm thời và tự tính toán độc lập |
| Forward Secrecy | ❌ Không có | ✅ Có (khóa tạm thời bị hủy sau phiên) |
| Vai trò của Chứng chỉ Server | Dùng để mã hóa `PreMasterSecret` | Dùng để Ký số và Xác thực danh tính |
| Tình trạng trong TLS 1.3 | ❌ Đã bị loại bỏ hoàn toàn | ✅ Là phương thức chủ đạo |
| Hiệu năng bắt tay | 2 RTT (TLS 1.2) | 2 RTT (TLS 1.2) / 1 RTT (TLS 1.3) |

<!-- @include: @article-footer.snippet.md -->
