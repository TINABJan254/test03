---
title: TCP Keepalive và HTTP Keep-Alive có gì khác nhau? (Tầng giao vận vs Tầng ứng dụng)
description: So sánh toàn diện sự khác biệt giữa TCP Keepalive (bảo tồn kết nối tầng giao vận) và HTTP Keep-Alive (kết nối dài tầng ứng dụng), làm rõ nguyên lý hoạt động, cấu hình kernel và kịch bản thực tế.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: TCP Keepalive, HTTP Keep-Alive, Kết nối dài, Heartbeat, Tầng giao vận, Tầng ứng dụng
---

Cùng mang chữ "Keep-Alive", nhưng **TCP Keepalive** và **HTTP Keep-Alive** là hai khái niệm hoàn toàn khác nhau ở hai tầng mạng khác nhau.

Bài viết này sẽ làm rõ sự khác biệt bản chất giữa chúng.

---

## 1. Bảng so sánh tổng quan

| Tiêu chí | TCP Keepalive | HTTP Keep-Alive |
| --- | --- | --- |
| **Tầng mạng** | **Tầng Giao vận (Transport Layer - Layer 4)** | **Tầng Ứng dụng (Application Layer - Layer 7)** |
| **Mục đích chính** | **Kiểm tra kết nối có còn sống hay không** (Phát hiện Dead Peer / Đứt dây mạng / Crash) | **Tái sử dụng kết nối TCP cho nhiều HTTP Request** (Tránh lặp lại bắt tay 3 bước) |
| **Cơ chế hoạt động** | Gửi gói tin thăm dò trống (Probe Packet với Sequence Number cũ) khi đường truyền nhàn rỗi | Giữ kết nối TCP mở sau khi hoàn thành một HTTP Response |
| **Cấu hình** | Cấu hình trong Linux Kernel (`tcp_keepalive_time`, `intvl`, `probes`) | Cấu hình qua Header HTTP (`Connection: keep-alive`) hoặc Web Server (Nginx, Tomcat) |
| **Mặc định** | Mặc định thường tắt trong Socket (phải bật cờ `SO_KEEPALIVE`), thời gian chờ nhàn rỗi mặc định rất dài (2 tiếng) | Mặc định bật trong HTTP/1.1 |

---

## 2. TCP Keepalive (Tầng 4) hoạt động như thế nào?

Giả sử Client và Server đã thiết lập một kết nối TCP. Sau đó cả hai bên không gửi thêm dữ liệu nào nữa.
Nếu dây mạng của Client bị rút, hoặc máy Client bị mất nguồn đột ngột:
- Phía Server không nhận được gói `FIN`, nên Server vẫn giữ kết nối ở trạng thái `ESTABLISHED` mãi mãi, gây rò rỉ tài nguyên bộ nhớ và Socket.

**TCP Keepalive giải quyết vấn đề này:**
1. Khi kết nối nhàn rỗi quá thời gian `tcp_keepalive_time` (mặc định 7200 giây = 2 giờ), Kernel Server sẽ tự động gửi một gói tin thăm dò (Probe Segment).
2. Gói thăm dò này không chứa dữ liệu thực tế, số thứ tự `seq = current_seq - 1`.
3. Phía Client nếu còn sống sẽ phản hồi lại gói `ACK`.
4. Nếu sau `tcp_keepalive_probes` lần gửi thăm dò (mỗi lần cách nhau `tcp_keepalive_intvl` giây) mà không nhận được phản hồi, Server sẽ tự động đóng Socket và giải phóng tài nguyên.

---

## 3. HTTP Keep-Alive (Tầng 7) hoạt động như thế nào?

Trong HTTP/1.0, mỗi lần tải một tài nguyên (HTML, ảnh, CSS, JS), trình duyệt phải tạo một kết nối TCP mới và đóng lại ngay sau đó.

**HTTP Keep-Alive (Persistent Connection) giải quyết vấn đề này:**
- Sau khi tải xong file HTML, kết nối TCP không bị đóng lại.
- Trình duyệt tiếp tục gửi request tải tiếp file `style.css` và `logo.png` trên chính kết nối TCP đã mở sẵn.
- Giúp giảm độ trễ đáng kể do không phải lặp lại bắt tay 3 bước TCP (tiết kiệm ít nhất 1 RTT cho mỗi tài nguyên).

---

## 4. Tổng kết

- **HTTP Keep-Alive**: Là để **tiết kiệm chi phí mở kết nối** bằng cách tái sử dụng kết nối TCP cho nhiều request ứng dụng.
- **TCP Keepalive**: Là để **dọn dẹp các kết nối chết** bằng cách định kỳ thăm dò trạng thái hoạt động của đối phương ở tầng mạng.

<!-- @include: @article-footer.snippet.md -->
