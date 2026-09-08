---
title: HTTP 1.0 vs HTTP 1.1: Long Connection, Caching, Host Header và các khác biệt cốt lõi (Tầng ứng dụng)
description: So sánh chi tiết sự khác biệt giao thức giữa HTTP/1.0 và HTTP/1.1, bao gồm Persistent Connection, Pipelining, Caching, cải tiến Status Code, Host Header và tối ưu băng thông.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: HTTP/1.0, HTTP/1.1, Long Connection, Pipelining, Cache, Status Code, Host Header, Tối ưu băng thông
---

HTTP/1.0 và HTTP/1.1 tuy chỉ cách nhau một phiên bản nhỏ, nhưng chúng có những khác biệt rất rõ ràng về tái sử dụng kết nối (Connection Reuse), bộ nhớ đệm (Caching), Host Header, mã trạng thái và tối ưu hóa băng thông.

Những khác biệt này ảnh hưởng trực tiếp tới cách trình duyệt gửi request, cách máy chủ tái sử dụng kết nối, cách cache hoạt động và cách máy chủ ảo (Virtual Host) vận hành.

Bài viết này chủ yếu trả lời các câu hỏi:

1. So với HTTP/1.0, HTTP/1.1 đã bổ sung những mã trạng thái phổ biến nào?
2. Cơ chế Caching của HTTP/1.0 và HTTP/1.1 khác nhau như thế nào?
3. Tại sao HTTP/1.1 mặc định hỗ trợ Persistent Connection (Kết nối dài / Keep-Alive)?
4. Header `Host` và các tính năng tối ưu băng thông giải quyết vấn đề gì?

![Tổng quan HTTP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/http-overview.png)

## Mã phản hồi trạng thái (Status Code)

HTTP/1.0 chỉ định nghĩa 16 mã trạng thái. HTTP/1.1 đã bổ sung thêm rất nhiều mã trạng thái mới (chỉ riêng mã lỗi đã thêm 24 loại). Ví dụ:
- `100 Continue`: Cho phép Client gửi trước header để xác nhận Server có sẵn sàng tiếp nhận Request Body dung lượng lớn hay không trước khi truyền tải thực sự.
- `206 Partial Content`: Mã chỉ định phản hồi thành công cho một Range Request (yêu cầu một phần dữ liệu).
- `409 Conflict`: Request xung đột với trạng thái hiện tại của tài nguyên.
- `410 Gone`: Tài nguyên được yêu cầu đã bị xóa vĩnh viễn trên server.

## Cơ chế Caching (Bộ nhớ đệm)

Công nghệ Caching giúp tránh việc người dùng phải tương tác liên tục với Origin Server, tiết kiệm băng thông mạng đáng kể và giảm độ trễ phản hồi.

### HTTP/1.0

HTTP/1.0 cung cấp cơ chế Cache tương đối đơn giản:
- Server sử dụng header `Expires` để đánh dấu thời gian hết hạn của tài nguyên (dựa trên thời gian tuyệt đối của Server).
- Trong lần phản hồi đầu tiên, Server trả về header `Last-Modified` ghi lại thời điểm tài nguyên được chỉnh sửa lần cuối trên server.
- Trong các request tiếp theo, Client gửi kèm header `If-Modified-Since` chứa mốc thời gian của `Last-Modified` trước đó để hỏi Server: "Sau thời điểm này, tài nguyên có bị sửa đổi không?".
- Nếu tài nguyên chưa bị sửa, Server trả về `304 Not Modified` (không kèm Body), báo cho Client tiếp tục dùng bản Cache cục bộ.
- Nếu tài nguyên đã bị sửa, Server trả về `200 OK` kèm nội dung tài nguyên mới.

![HTTP/1.0 sử dụng Expires và Last-Modified để kiểm tra Cache](./images/http-vs-https/HTTP1.0cache1.png)

![HTTP/1.0 trả về 304 Not Modified khi trúng Cache](./images/http-vs-https/HTTP1.0cache2.png)

Nhược điểm của HTTP/1.0: `Expires` dựa vào đồng hồ của Server. Nếu đồng hồ giữa Client và Server bị lệch, việc kiểm tra hạn Cache sẽ không chính xác.

### HTTP/1.1

HTTP/1.1 bổ sung cơ chế Caching linh hoạt và mạnh mẽ hơn rất nhiều:
- Đưa vào `Cache-Control` với `max-age` (tính theo thời gian tương đối tính bằng giây, khắc phục nhược điểm lệch đồng hồ của `Expires`).
- Đưa vào **Entity Tag (`ETag`)**: Một chuỗi băm đại diện cho phiên bản nội dung của tài nguyên.
- Client dùng `If-None-Match` gửi kèm giá trị `ETag` lên để so khớp. `ETag` giúp nhận diện chính xác sự thay đổi nội dung (thay vì chỉ dựa vào mốc thời gian giây của `Last-Modified`).

## Phương thức kết nối (Connection Management)

- **HTTP/1.0 mặc định sử dụng Short Connection (Kết nối ngắn)**: Mỗi lần Client và Server thực hiện một HTTP Request-Response là một lần thiết lập và ngắt kết nối TCP. Một trang HTML chứa nhiều file JS, CSS, hình ảnh sẽ khiến trình duyệt liên tục mở và đóng hàng loạt kết nối TCP, gây lãng phí lớn tài nguyên mạng cho việc bắt tay (SYN) và ngắt kết nối (FIN).
- **HTTP/1.1 mặc định sử dụng Persistent Connection (Kết nối dài / Keep-Alive)**: Kết nối TCP sẽ được giữ mở sau khi hoàn thành một request để phục vụ cho các request tiếp theo giữa Client và Server.

Để đóng kết nối trong HTTP/1.1, client hoặc server gửi header `Connection: close`. Trong HTTP/1.0, nếu muốn dùng kết nối dài phải chỉ định rõ `Connection: Keep-Alive`.

## Xử lý Host Header (Hỗ trợ Virtual Host)

Trong HTTP/1.0, request gửi lên không bắt buộc phải có thông tin tên miền máy chủ (chỉ gửi `GET /home.html HTTP/1.0`). Khi một địa chỉ IP vật lý lưu trữ nhiều website khác nhau (Virtual Hosting với nhiều tên miền trỏ về 1 IP), server sẽ không biết client đang muốn truy cập website nào.

HTTP/1.1 bắt buộc mọi request phải có header `Host`:

```plain
GET /home.html HTTP/1.1
Host: example1.org
```

Nhờ đó, một Web Server với một IP duy nhất có thể phân biệt chính xác và phục vụ nhiều tên miền khác nhau trên cùng một cổng.

## Tối ưu hóa băng thông

### Range Request (Yêu cầu một phần dữ liệu / Hỗ trợ Resume Download)

HTTP/1.1 hỗ trợ Range Request thông qua header `Range`, cho phép Client chỉ yêu cầu tải về một đoạn byte cụ thể của file:

```http
GET /image.jpg HTTP/1.1
Host: example.com
Range: bytes=0-1023
```

Server phản hồi với mã `206 Partial Content`:

```http
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1023/146515
Content-Length: 1024
... (1024 byte dữ liệu nhị phân)
```

Tính năng này là nền tảng cho việc tải file đa luồng (Multi-part download), xem video streaming (tua video) và tiếp tục tải khi bị đứt mạng (Resume Download).

### Mã trạng thái 100 Continue

Khi Client muốn gửi một Request Body lớn (ví dụ upload file hàng trăm MB), Client có thể gửi trước Request Header kèm `Expect: 100-continue`. Nếu Server kiểm tra quyền hạn và dung lượng thấy hợp lệ, Server sẽ trả về `100 Continue`, lúc này Client mới bắt đầu truyền tải Body thực sự. Nếu Server từ chối (ví dụ 401 hoặc 413), Client không cần tốn băng thông truyền file vô ích.

![HTTP/1.1 sử dụng 100 Continue](./images/http-vs-https/HTTP1.1continue1.png)

![Client nhận 100 Continue và gửi tiếp Request Body](./images/http-vs-https/HTTP1.1continue2.png)

### Nén và Chunked Transfer Encoding

HTTP/1.1 phân biệt rõ giữa Content-Encoding (mã hóa nội dung End-to-End, như `gzip`, `br`) và Transfer-Encoding (mã hóa truyền tải Hop-by-Hop, như `chunked`). Cơ chế `Transfer-Encoding: chunked` cho phép server truyền dữ liệu động theo từng khối mà không cần biết trước tổng kích thước `Content-Length`.

## Tổng kết các khác biệt chính

1. **Kết nối**: HTTP/1.0 mặc định Short Connection; HTTP/1.1 mặc định Persistent Connection (Keep-Alive).
2. **Trạng thái**: HTTP/1.1 bổ sung nhiều Status Code mới (`100 Continue`, `206 Partial Content`, `409 Conflict`, `410 Gone`, v.v.).
3. **Caching**: HTTP/1.0 chủ yếu dùng `Expires` và `Last-Modified`; HTTP/1.1 bổ sung `Cache-Control` (`max-age`), `ETag` và `If-None-Match`.
4. **Băng thông**: HTTP/1.1 hỗ trợ Range Request (`206 Partial Content`), cơ chế `100 Continue`, nén `Accept-Encoding` và truyền từng khối `Chunked`.
5. **Host Header**: HTTP/1.1 bắt buộc có `Host` header để hỗ trợ Virtual Hosting trên cùng 1 địa chỉ IP.

<!-- @include: @article-footer.snippet.md -->
