---
title: Tổng hợp HTTP Status Code thường gặp (Tầng ứng dụng)
description: Tổng hợp ý nghĩa và kịch bản sử dụng các mã trạng thái HTTP phổ biến, nhấn mạnh các điểm dễ nhầm lẫn như 201/204, 301/302, 401/403, 500/502 để nâng cao hiệu quả thiết kế API và gỡ lỗi.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: HTTP Status Code, 2xx, 3xx, 4xx, 5xx, Chuyển hướng, Mã lỗi, 201 Created, 204 No Content
---

HTTP Status Code là phần tóm tắt kết quả xử lý mà Server trả về cho Client. Nhìn vào mã trạng thái, về cơ bản có thể phán đoán được ngay request thành công, bị chuyển hướng, client bị lỗi hay server bị lỗi.

Các mã trạng thái nhìn bề ngoài chỉ là con số, nhưng rất nhiều mã dễ bị nhầm lẫn: ví dụ 301 và 302, 401 và 403, 500 và 502, 201 và 204.

Bài viết này chủ yếu trả lời các câu hỏi:

1. Các nhóm 1xx, 2xx, 3xx, 4xx, 5xx lần lượt đại diện cho loại kết quả nào?
2. Các mã thành công phổ biến như 200, 201, 204 khác nhau như thế nào?
3. Các mã lỗi Client phổ biến như 400, 401, 403, 404 nên hiểu ra sao?
4. Các mã lỗi Server phổ biến như 500, 502, 503, 504 thường mang ý nghĩa gì?

![Các mã trạng thái HTTP thường gặp](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-status-code.png)

### 1xx Informational (Mã trạng thái thông tin)

So với các nhóm khác, nhóm 1xx bạn rất ít khi gặp trong thực tế, thể hiện server đã nhận được request và đang tiếp tục xử lý.

### 2xx Success (Mã trạng thái thành công)

- **200 OK**: Request đã được xử lý thành công. Ví dụ: gửi một HTTP Request truy vấn dữ liệu user lên server, server trả về đúng dữ liệu user. Đây là status code phổ biến nhất.
- **201 Created**: Request đã được xử lý thành công và tạo mới một hoặc nhiều tài nguyên trên server. Ví dụ: gọi POST request để tạo một user mới.
- **202 Accepted**: Server đã tiếp nhận request nhưng chưa xử lý xong (xử lý bất đồng bộ). Ví dụ: gửi request tạo báo cáo thống kê hoặc export Excel tốn nhiều thời gian, server nhận việc và trả về 202 để client biết tác vụ đang chạy nền.
- **204 No Content**: Server đã xử lý thành công request nhưng không trả về bất kỳ nội dung nào trong Response Body. Ví dụ: gửi request DELETE để xóa một user, server xóa thành công và không cần trả về body.

🐛 Đính chính (Xem: [issue#2458](https://github.com/Snailclimb/JavaGuide/issues/2458)): Mã `201 Created` chính xác hơn là tạo mới một hoặc nhiều tài nguyên, tham khảo RFC 9110: <https://httpwg.org/specs/rfc9110.html#status.201>.

![Định nghĩa mã 201 Created trong RFC 9110](https://oss.javaguide.cn/github/javaguide/cs-basics/network/rfc9110-201-created.png)

Đặc biệt nhắc thêm về mã **204 No Content**, trong học tập/công việc bình thường ít để ý:

Đặc tả [HTTP RFC 2616 mục 10.2.5](https://tools.ietf.org/html/rfc2616#section-10.2.5) nêu rõ:

> The server has fulfilled the request but does not need to return an entity-body, and might want to return updated metainformation...
> The 204 response MUST NOT include a message-body, and thus is always terminated by the first empty line after the header fields.

Nói một cách đơn giản, mã 204 mô tả tình huống Client gửi HTTP Request và chỉ cần quan tâm kết quả xử lý thành công hay không (kết quả boolean true/false), không cần dữ liệu trả về trong body.

### 3xx Redirection (Mã trạng thái chuyển hướng)

- **301 Moved Permanently**: Tài nguyên đã được chuyển hướng vĩnh viễn sang URL mới. Trình duyệt và Search Engine sẽ tự động cập nhật và cache URL mới.
- **302 Found**: Tài nguyên tạm thời được chuyển sang URL khác. Lần truy cập sau client vẫn nên gửi request tới URL gốc.
- **304 Not Modified**: Tài nguyên chưa bị thay đổi kể từ lần request trước (dựa trên header `If-Modified-Since` hoặc `If-None-Match`), Client có thể tiếp tục sử dụng bản cache cục bộ.

### 4xx Client Error (Mã lỗi từ phía Client)

- **400 Bad Request**: Request gửi lên có lỗi cú pháp, tham số không hợp lệ hoặc method không đúng.
- **401 Unauthorized**: Chưa được xác thực danh tính (chưa đăng nhập hoặc token không hợp lệ) mà đã cố truy cập tài nguyên yêu cầu xác thực.
- **403 Forbidden**: Server hiểu request nhưng từ chối thực hiện (thường do không đủ quyền hạn, ví dụ tài khoản thường cố truy cập API của Admin).
- **404 Not Found**: Không tìm thấy tài nguyên được yêu cầu trên server.
- **409 Conflict**: Request xung đột với trạng thái hiện tại của tài nguyên trên server (ví dụ: tạo tài khoản với username đã tồn tại).

### 5xx Server Error (Mã lỗi từ phía Server)

- **500 Internal Server Error**: Đã xảy ra lỗi nội bộ phía server (thường do code phía backend phát sinh Exception mà chưa được bắt và xử lý).
- **502 Bad Gateway**: Server đóng vai trò Gateway hoặc Proxy nhận được phản hồi không hợp lệ từ Upstream Server phía sau.
- **503 Service Unavailable**: Server hiện tại đang bị quá tải hoặc đang bảo trì, tạm thời không thể xử lý request.
- **504 Gateway Timeout**: Server đóng vai trò Gateway hoặc Proxy không nhận được phản hồi kịp thời từ Upstream Server trong khoảng thời gian quy định (Timeout).

### Tài liệu tham khảo

- <https://www.restapitutorial.com/httpstatuscodes.html>
- <https://developer.mozilla.org/en-US/docs/Web/HTTP/Status>
- <https://en.wikipedia.org/wiki/List_of_HTTP_status_codes>

<!-- @include: @article-footer.snippet.md -->
