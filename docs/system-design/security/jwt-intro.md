---
title: JWT là gì? Cấu trúc JWT, giải mã và xác thực đăng nhập chi tiết
description: JWT (JSON Web Token) là gì? Bài viết này giải thích cấu trúc 3 phần Header, Payload, Signature, giải mã Base64Url, xác thực chữ ký, thời hạn hết hạn và quy trình đăng nhập xác thực, cùng các rủi ro an toàn thường gặp.
category: Thiết kế hệ thống
tag:
  - Bảo mật
head:
  - - meta
    - name: keywords
      content: JWT là gì,Giải mã JWT,JWT Token,Xác thực JWT,JSON Web Token,Token authentication,Stateless,Header Payload Signature,Thuật toán ký,Đăng nhập xác thực,CSRF
---

<!-- @include: @article-header.snippet.md -->

## JWT là gì?

JWT（JSON Web Token）là một định dạng biểu diễn tuyên bố (Claims) nhỏ gọn và an toàn trên URL được định nghĩa bởi [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519). Sau khi đăng nhập thành công, phía server có thể ghi các tuyên bố như định danh người dùng, phạm vi quyền hạn và thời gian hết hạn vào JWT; phía client trong các request tiếp theo sẽ mang theo nó, server xác thực chữ ký và các tuyên bố liên quan trước khi quyết định có cho phép truy cập hay không.

JWT có thể mang các tuyên bố cần thiết cho việc xác thực, do đó phía server không nhất thiết phải lưu trữ trạng thái phiên làm việc (Session) như phương pháp Session truyền thống. Tuy nhiên, các yêu cầu như thu hồi token, thay đổi quyền hạn và chủ động đăng xuất vẫn có thể cần trạng thái ở phía server, không thể chỉ dựa vào việc "sử dụng JWT" mà cho rằng hệ thống hoàn toàn vô trạng (Stateless).

Phần Header và Payload của JWT chỉ được mã hóa dạng Base64Url, bất kỳ ai lấy được token đều có thể giải mã, do đó không được ghi các thông tin nhạy cảm như mật khẩu, số CMND/CCCD vào Payload. Chữ ký (Signature) dùng để kiểm tra xem nội dung có bị thay đổi/tâm lý chỉnh sửa hay không, chứ không chịu trách nhiệm mã hóa nội dung.

Nếu client đặt JWT làm Bearer Token hiển thị trong Header `Authorization`, trình duyệt sẽ không tự động đính kèm nó giống như Cookie, do đó có thể giảm rủi ro CSRF truyền thống. Tuy nhiên, điều này phụ thuộc vào phương thức truyền và lưu trữ thông tin xác thực, chứ không phải bản thân định dạng JWT; nếu đặt JWT trong Cookie, vẫn cần có các biện pháp phòng chống CSRF.

Bài viết [Phân tích ưu nhược điểm của JWT](./advantages-and-disadvantages-of-jwt.md) giới thiệu chi tiết về ưu điểm và hạn chế của việc sử dụng JWT trong xác thực danh tính.

Dưới đây là định nghĩa về JWT theo [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519):

> JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is used as the payload of a JSON Web Signature (JWS) structure or as the plaintext of a JSON Web Encryption (JWE) structure, enabling the claims to be digitally signed or integrity protected with a Message Authentication Code (MAC) and/or encrypted. ——[JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)

## JWT gồm những phần nào?

![Cấu trúc JWT](https://oss.javaguide.cn/javaguide/system-design/jwt/jwt-composition.png)

JWT thường gồm 3 phần được phân cách bởi dấu chấm (`.`):

- **Header (Phần đầu)**: Mô tả dữ liệu siêu (Metadata) của JWT, chứa loại token và thuật toán ký. Header sau khi được mã hóa Base64Url sẽ trở thành phần thứ nhất của JWT.
- **Payload (Tải trọng / Nguồn dữ liệu)**: Chứa các tuyên bố (Claims) cần truyền đi, như `sub` (Subject - Chủ thể), `jti` (JWT ID). Payload sau khi được mã hóa Base64Url sẽ trở thành phần thứ hai của JWT.
- **Signature (Chữ ký)**: Được tính toán dựa trên Header đã mã hóa, Payload đã mã hóa, thuật toán ký và khóa ký. Thuật toán HS256 sử dụng khóa bí mật chia sẻ (Symmetric Secret Key), còn các thuật toán bất đối xứng như RS256, ES256 sử dụng khóa riêng (Private Key) để ký và khóa công khai (Public Key) để xác thực.

JWT thường có dạng như thế này: `xxxxx.yyyyy.zzzzz`.

Ví dụ:

```plain
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Bạn có thể giải mã đoạn JWT ví dụ này trên [jwt.io](https://jwt.io/), sau khi giải mã bạn sẽ thấy 3 phần Header, Payload và Signature. Trong môi trường thực tế, token thật chứa thông tin định danh và quyền hạn người dùng, tuyệt đối không sao chép vào các công cụ trực tuyến của bên thứ ba.

Cả Header và Payload đều là dữ liệu JSON, còn Signature được tính toán từ Header đã mã hóa, Payload đã mã hóa và khóa ký.

![](https://oss.javaguide.cn/javaguide/system-design/jwt/jwt.io.png)

### Sự khác biệt giữa Giải mã JWT (Parse) và Xác thực JWT (Verify)?

Giải mã JWT chỉ thực hiện giải mã Base64Url cho Header và Payload, không cần khóa bí mật. Bất kỳ ai lấy được token đều có thể hoàn thành việc giải mã, do đó kết quả giải mã không chứng minh được token có đáng tin hay không.

Xác thực JWT sẽ sử dụng thuật toán và khóa bí mật được chỉ định để kiểm tra Signature, đồng thời kiểm tra các tuyên bố như `exp`, `nbf`, `iss`, `aud`... Chỉ khi chữ ký và tất cả các tuyên bố theo yêu cầu nghiệp vụ đều vượt qua kiểm tra, phía server mới có thể tin tưởng thông tin danh tính và quyền hạn trong token.

### Header

Header thường gồm hai phần:

- `typ` (Type): Loại token, tức là JWT.
- `alg` (Algorithm): Thuật toán ký, ví dụ HS256.

Ví dụ:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Header dạng JSON sau khi được mã hóa Base64Url sẽ trở thành phần thứ nhất của JWT.

### Payload

Payload cũng là dữ liệu JSON, trong đó chứa các Claims (tuyên bố).

Claims được chia làm 3 loại:

- **Registered Claims (Tuyên bố đã đăng ký)**: Các tuyên bố được định nghĩa sẵn, khuyến nghị sử dụng nhưng không bắt buộc.
- **Public Claims (Tuyên bố công khai)**: Các tuyên bố do bên phát hành JWT tự định nghĩa, nhưng để tránh xung đột, nên định nghĩa chúng trong [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml).
- **Private Claims (Tuyên bố riêng)**: Các tuyên bố do bên phát hành JWT tự định nghĩa theo nhu cầu dự án, phù hợp hơn với các kịch bản thực tế.

Dưới đây là một số tuyên bố đã đăng ký thường gặp:

- `iss` (issuer): Bên phát hành JWT.
- `iat` (issued at time): Thời điểm phát hành JWT.
- `sub` (subject): Chủ thể của JWT.
- `aud` (audience): Bên nhận JWT.
- `exp` (expiration time): Thời gian hết hạn của JWT.
- `nbf` (not before time): Thời điểm có hiệu lực của JWT, JWT không được chấp nhận xử lý trước thời điểm này.
- `jti` (JWT ID): Mã định danh duy nhất của JWT.

Ví dụ:

```json
{
  "uid": "ff1212f5-d8d1-4496-bf41-d2dda73de19a",
  "sub": "1234567890",
  "name": "John Doe",
  "exp": 15323232,
  "iat": 1516239022,
  "scope": ["admin", "user"]
}
```

Mặc định phần Payload không được mã hóa, **tuyệt đối không lưu trữ thông tin riêng tư/nhạy cảm trong Payload!!!**

Payload dạng JSON sau khi mã hóa Base64Url sẽ trở thành phần thứ hai của JWT.

### Signature

Phần Signature là chữ ký cho hai phần trước, có tác dụng phòng chống việc JWT (chủ yếu là payload) bị chỉnh sửa trái phép.

Quá trình tạo chữ ký này cần sử dụng:

- Header + Payload.
- Khóa ký lưu trữ ở phía server. Khi sử dụng thuật toán bất đối xứng, khóa riêng (Private Key) không được để rò rỉ.
- Thuật toán ký.

Công thức tính chữ ký như sau:

```plain
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

Sau khi tính ra chữ ký, ghép 3 phần Header, Payload, Signature thành một chuỗi, nối với nhau bằng dấu chấm (`.`), chuỗi này chính là JWT.

## Làm thế nào để xác thực danh tính dựa trên JWT?

Trong ứng dụng xác thực danh tính dựa trên JWT, server tạo JWT thông qua Payload, Header và khóa bí mật rồi gửi JWT cho client. Client cần lưu trữ token an toàn tùy theo dạng ứng dụng và mô hình đe dọa bảo mật, các request gửi đi sau đó sẽ đính kèm token này.

![Sơ đồ xác thực danh tính JWT](https://oss.javaguide.cn/github/javaguide/system-design/jwt/jwt-authentication%20process.png)

Các bước đơn giản hóa như sau:

1. Người dùng gửi username, password và mã xác nhận lên server để đăng nhập hệ thống;
2. Nếu thông tin kiểm tra chính xác, phía server trả về Token đã được ký, tức là JWT;
3. Client sau khi nhận Token sẽ lưu trữ an toàn; ứng dụng trình duyệt có thể dùng BFF giữ token ở phía server, hoặc dùng Cookie được bảo vệ tùy theo kịch bản;
4. Mỗi lần người dùng gửi request lên backend sau đó đều mang theo JWT này trong Header;
5. Phía server kiểm tra JWT và lấy ra thông tin liên quan đến người dùng từ trong token.

Hai lời khuyên:

1. Đừng mặc định lưu JWT trong `localStorage` hoặc `sessionStorage`. Bất kỳ script độc hại nào trong trang cùng nguồn (Same-origin) đều có thể đọc Web Storage, chỉ một lỗi XSS cũng có thể làm rò rỉ token. Khi dùng Cookie, nên thiết lập các thuộc tính `HttpOnly`, `Secure` và `SameSite` phù hợp, đồng thời làm tốt công tác phòng chống CSRF.
2. Cách làm phổ biến khi không dùng Cookie là đặt JWT vào trường `Authorization` trong HTTP Header (`Authorization: Bearer Token`).

Dự án **[spring-security-jwt-guide](https://github.com/Snailclimb/spring-security-jwt-guide)** là một ví dụ đơn giản về xác thực danh tính dựa trên JWT, bạn nào quan tâm có thể xem thử.

## Làm thế nào để phòng chống JWT bị chỉnh sửa?

Sau khi có chữ ký được xác thực chính xác, ngay cả khi JWT bị rò rỉ hoặc chặn lại, kẻ tấn công cũng không thể sửa đổi Header hoặc Payload để tạo ra chữ ký hợp lệ nếu không biết khóa ký bí mật. Tuy nhiên, chữ ký không cung cấp tính bảo mật thông tin (không mã hóa) và cũng không thể ngăn kẻ tấn công phát lại (replay attack) trực tiếp token hợp lệ đã bị trộm.

Tại sao lại như vậy? Vì sau khi server nhận được JWT, nó sẽ giải mã ra Header, Payload và Signature. Server sẽ dựa trên Header, Payload và khóa bí mật để tạo lại một Signature mới. Lấy Signature mới tạo so sánh với Signature trong JWT, nếu giống nhau chứng tỏ Header và Payload chưa bị thay đổi.

Tuy nhiên, nếu khóa bí mật phía server bị rò rỉ, kẻ tấn công có thể sửa đổi Header và Payload, sau đó tạo lại một Signature mới hoàn toàn hợp lệ.

Khóa ký bí mật phải được bảo quản cẩn thận, đồng thời xây dựng cơ chế xoay vòng (rotation) và thu hồi.

## Làm thế nào để tăng cường độ an toàn cho JWT?

1. Sử dụng các thư viện mã nguồn mở thành숙, không tự viết logic mã hóa/giải mã và kiểm tra JWT.
2. Phía server cố định danh sách thuật toán được phép, không được tin tưởng trực tiếp `alg` trong JWT Header để chọn thuật toán xác thực; khóa HMAC phải có đủ độ ngẫu nhiên và độ dài.
3. Xác thực tất cả các tuyên bố liên quan đến ứng dụng hiện tại, bao gồm `iss`, `aud`, `exp` và `nbf`, đồng thời thiết lập giới hạn độ lệch đồng hồ (clock skew) rõ ràng.
4. Sử dụng `typ` rõ ràng và quy tắc kiểm tra loại trừ lẫn nhau cho các JWT dùng vào mục đích khác nhau (như ID Token, Access Token), tránh việc một loại token bị thay thế vào kịch bản khác.
5. Tuyệt đối không lưu trữ thông tin riêng tư trong Payload chưa được mã hóa, cũng như không coi Claim nhận được nhưng chưa xác thực là đầu vào đáng tin cậy.
6. Lựa chọn phương thức lưu trữ token an toàn dựa trên loại client, giới hạn thời gian hiệu lực, phạm vi quyền hạn và bên nhận token; các kịch bản rủi ro cao cần cân nhắc thu hồi, phát hiện phát lại (replay detection) hoặc ràng buộc người gửi (sender-constrained).
7. Khóa bí mật phải được bảo quản cẩn thận và hỗ trợ xoay vòng. Các yêu cầu an toàn đầy đủ hơn có thể tham khảo [RFC 8725: JSON Web Token Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725.html).

<!-- @include: @article-footer.snippet.md -->
