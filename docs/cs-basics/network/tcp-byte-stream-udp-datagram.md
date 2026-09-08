---
title: Tại sao TCP là hướng luồng byte, còn UDP là hướng gói tin? (Tầng giao vận)
description: Phân tích sự khác biệt cốt lõi giữa Byte Stream của TCP và Message/Datagram của UDP, làm rõ hiện tượng dính gói / tách gói (TCP Packet Sticking / Splitting) và cách phân định ranh giới gói tin ở tầng ứng dụng.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: TCP, UDP, Byte Stream, Luồng byte, Datagram, Gói tin, Dính gói, Tách gói, Packet Sticking
---

Trong các cuộc phỏng vấn mạng máy tính, đây là một câu hỏi rất phổ biến:

**"Tại sao nói TCP là giao thức hướng luồng byte (Byte Stream-oriented), trong khi UDP lại là giao thức hướng gói tin (Message/Datagram-oriented)?"**

Sự khác biệt này quyết định trực tiếp tới cách chúng ta lập trình Socket và thiết kế giao thức ở tầng ứng dụng (Application Layer Protocol Design).

Bài viết này chủ yếu trả lời các câu hỏi:

1. Thế nào là "hướng luồng byte" và thế nào là "hướng gói tin"?
2. Tại sao TCP không bảo toàn ranh giới thông điệp (Message Boundary), còn UDP lại bảo toàn?
3. Hiện tượng "dính gói / phân mảnh gói" (TCP Packet Sticking / Splitting) thực chất là gì?
4. Tầng ứng dụng giải quyết bài toán phân định ranh giới thông điệp trong TCP bằng cách nào?

---

## 1. UDP: Hướng gói tin (Message/Datagram-oriented)

**UDP bảo toàn nguyên vẹn ranh giới thông điệp do ứng dụng gửi xuống.**

- Khi tầng ứng dụng gọi hàm `sendto()` gửi một thông điệp (Message) có độ dài $N$ byte xuống UDP:
  - UDP không chia nhỏ hay gộp thông điệp, mà chỉ thêm Header UDP 8 byte vào đầu rồi chuyển thẳng xuống tầng IP thành một IP Datagram.
- Phía nhận (Receiver):
  - Mỗi lần gọi hàm `recvfrom()` sẽ đọc được **chính xác một thông điệp trọn vẹn** do bên gửi phát đi.
  - Số lần gọi `sendto()` ở bên gửi tương ứng $1:1$ với số lần gọi `recvfrom()` ở bên nhận.

Nói cách khác: **UDP có ranh giới thông điệp rõ ràng (Preserves Message Boundaries).**

---

## 2. TCP: Hướng luồng byte (Byte Stream-oriented)

**TCP xem dữ liệu cần truyền tải là một dòng chảy liên tục của các byte nhị phân không có điểm đầu và điểm cuối.**

- Khi tầng ứng dụng gọi `write()` hoặc `send()` ghi dữ liệu vào Socket TCP:
  - TCP chỉ xem đây là một chuỗi các byte được đưa vào **Bộ đệm gửi (Send Buffer)**.
  - TCP không quan tâm một thông điệp ứng dụng bắt đầu ở đâu và kết thúc ở đâu.
  - Tùy thuộc vào kích thước MSS (Maximum Segment Size), kích thước cửa sổ trượt (Sliding Window) và thuật toán Nagle, TCP sẽ tự động quyết định cắt luồng byte thành các đoạn (Segment) có kích thước phù hợp để gửi đi.
- Phía nhận (Receiver):
  - Các byte nhận được từ mạng được xếp vào **Bộ đệm nhận (Receive Buffer)** theo đúng số thứ tự (Sequence Number).
  - Tầng ứng dụng gọi `read()` hoặc `recv()` để đọc một số lượng byte tùy ý từ bộ đệm.
  - Một lần `send()` 1000 byte ở bên gửi có thể được bên nhận đọc bằng 1 lần `recv()` 1000 byte, hoặc 2 lần `recv()` 500 byte, hoặc 10 lần `recv()` 100 byte!

Nói cách khác: **TCP không bảo toàn ranh giới thông điệp của tầng ứng dụng.**

---

## 3. Vấn đề "Dính gói" (Packet Sticking) và "Tách gói" (Packet Splitting)

Bởi vì TCP là hướng luồng byte, nên ở tầng ứng dụng thường gặp 2 hiện tượng:

1. **Dính gói (Packet Sticking)**: Bên gửi phát đi 2 thông điệp độc lập `Msg1` và `Msg2` liên tiếp. Bên nhận khi gọi `read()` một lần lại đọc được cả `Msg1` và `Msg2` dính liền nhau.
2. **Tách gói / Phân mảnh (Packet Splitting)**: Bên gửi phát đi 1 thông điệp lớn `Msg1`. Bên nhận đọc lần 1 chỉ được nửa đầu của `Msg1`, lần 2 mới đọc được nửa sau.

> **Lưu ý**: Dưới góc độ của giao thức TCP ở tầng giao vận, đây là hành vi hoàn toàn chuẩn xác và tối ưu của một luồng byte liên tục, không phải là "lỗi của TCP". Nhưng dưới góc độ của tầng ứng dụng cần phân biệt từng thông điệp độc lập, ứng dụng phải tự chịu trách nhiệm phân định ranh giới.

---

## 4. Các giải pháp phân định ranh giới gói tin ở tầng ứng dụng

Để giải quyết bài toán dính gói/tách gói trong TCP, các giao thức tầng ứng dụng (như HTTP, Redis RESP, Dubbo, RPC) thường sử dụng 3 giải pháp kinh điển sau:

### Giải pháp 1: Sử dụng độ dài cố định (Fixed-Length Framing)
- Quy định mỗi thông điệp gửi đi luôn có độ dài cố định là $N$ byte (ví dụ 128 byte). Nếu dữ liệu ngắn hơn thì chèn thêm byte trống.
- Bên nhận cứ tích lũy đủ $N$ byte trong bộ đệm là bóc tách thành 1 thông điệp.
- *Nhược điểm*: Gây lãng phí băng thông nếu dữ liệu thực tế nhỏ hơn $N$.

### Giải pháp 2: Sử dụng ký tự phân cách (Delimiter-based Framing)
- Chèn một ký tự đặc biệt ở cuối mỗi thông điệp để làm dấu hiệu kết thúc (ví dụ ký tự xuống dòng `
` hoặc `
`).
- Bên nhận quét luồng byte đến khi gặp ký tự phân cách thì bóc tách thông điệp.
- *Ví dụ thực tế*: Giao thức dòng lệnh HTTP Header, giao thức Redis RESP, giao thức POP3/SMTP.
- *Nhược điểm*: Dữ liệu trong nội dung thông điệp không được chứa ký tự phân cách (hoặc phải dùng cơ chế Escape phức tạp).

### Giải pháp 3: Header chứa độ dài thông điệp (Length-Field Based Framing - Khuyên dùng)
- Cấu trúc thông điệp gồm 2 phần: **Header** (chứa trường `Length` cố định 2 hoặc 4 byte) + **Body** (chứa dữ liệu thực tế dài đúng `Length` byte).
- Bên nhận đọc trước số byte của Header để biết độ dài `Length`, sau đó chờ bộ đệm tích lũy đủ `Length` byte của Body thì cắt trọn vẹn thông điệp.
- *Ví dụ thực tế*: Header `Content-Length` trong HTTP, giao thức nhị phân của gRPC / Dubbo / Netty (`LengthFieldBasedFrameDecoder`).

---

## 5. Tổng kết

| Tiêu chí | TCP (Hướng luồng byte) | UDP (Hướng gói tin) |
| --- | --- | --- |
| Ranh giới dữ liệu | Không có ranh giới, là luồng byte liên tục | Bảo toàn nguyên vẹn ranh giới từng gói tin |
| Ghép/Tách gói tin | Tự động ghép/tách tùy theo MSS và Buffer | Giữ nguyên từng gói độc lập |
| Số lần gửi & nhận | Không có tương ứng $1:1$ giữa `send()` và `recv()` | Tương ứng chính xác $1:1$ giữa `sendto()` và `recvfrom()` |
| Xử lý ở tầng ứng dụng | Ứng dụng phải tự thiết kế cơ chế phân định ranh giới (Length-field, Delimiter) | Ứng dụng nhận gói tin nguyên vẹn trực tiếp |

<!-- @include: @article-footer.snippet.md -->
