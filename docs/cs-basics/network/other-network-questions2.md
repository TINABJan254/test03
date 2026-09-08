---
title: Tổng hợp câu hỏi phỏng vấn Mạng máy tính thường gặp (Phần 2)
description: Tổng hợp chi tiết các câu hỏi phỏng vấn mạng máy tính tần suất cao, bao gồm TCP, UDP, Socket, IP, ARP, NAT, WebSocket và các câu hỏi tình huống thực tế.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Phỏng vấn Mạng máy tính, TCP, UDP, Socket, ARP, NAT, Bắt tay 3 bước, 4 bước ngắt kết nối, WebSocket
---

<!-- @include: @article-header.snippet.md -->

Phần 2 của bộ câu hỏi phỏng vấn Mạng máy tính tập trung vào: **Tầng Giao vận (TCP, UDP, Socket), Tầng Mạng (IP, ARP, NAT) và Giao tiếp thời gian thực (WebSocket)**.

---

## 1. So sánh TCP và UDP

### Câu 1: So sánh sự khác nhau giữa TCP và UDP?

| Tiêu chí | TCP | UDP |
| --- | --- | --- |
| **Hướng kết nối** | Hướng kết nối (Bắt tay 3 bước trước khi truyền) | Không kết nối (Gửi trực tiếp) |
| **Độ tin cậy** | Tin cậy 100% (Không mất gói, không trùng, đúng thứ tự) | Không đảm bảo tin cậy (Nỗ lực tối đa Best-effort) |
| **Dạng truyền dữ liệu** | Hướng luồng byte (Byte Stream) | Hướng gói tin (Datagram) |
| **Kích thước Header** | 20 đến 60 byte | Cố định 8 byte |
| **Kiểm soát luồng & Tắc nghẽn** | Có (Sliding Window, Congestion Control) | Không có |
| **Kịch bản sử dụng** | Truyền file (FTP), Web (HTTP), Email, SSH, Database | Streaming video, Âm thanh thời gian thực (VoIP, RTP), Game online, DNS, DHCP |

---

## 2. Quản lý kết nối TCP

### Câu 2: Tại sao kết nối TCP cần Bắt tay 3 bước mà ngắt kết nối lại cần Bắt tay 4 bước?
- **Thiết lập kết nối (3 bước)**: Cờ `SYN` và `ACK` của Server có thể gộp chung vào một gói tin duy nhất (`SYN+ACK`) vì lúc này hai bên chưa có dữ liệu nào cần truyền dở dang.
- **Ngắt kết nối (4 bước)**: Do tính chất Full-Duplex, khi Client gửi `FIN` (đóng chiều gửi), Server có thể vẫn còn dữ liệu đang xử lý dở cần gửi nốt cho Client. Vì vậy Server phải gửi `ACK` trước (bước 2), sau khi gửi hết dữ liệu mới gửi `FIN` của mình (bước 3).

### Câu 3: Trạng thái TIME_WAIT là gì? Tại sao cần 2MSL?
- Là trạng thái của bên chủ động đóng kết nối sau khi gửi gói ACK cuối cùng.
- Kéo dài **2MSL** để:
  1. Đảm bảo nếu gói ACK cuối bị mất, bên kia gửi lại `FIN` thì mình vẫn còn sống để gửi lại `ACK`, giúp bên kia đóng kết nối êm đẹp.
  2. Cho phép toàn bộ các gói tin trôi nổi của kết nối cũ trên mạng tiêu biến hết, không làm sai lệch kết nối mới tạo sau đó.

---

## 3. Tầng Mạng: IP, ARP và NAT

### Câu 4: ARP hoạt động như thế nào?
- Khi host cần tìm địa chỉ MAC của một IP đích trong cùng mạng LAN: Gửi gói **ARP Request** dưới dạng **Broadcast** (`FF-FF-FF-FF-FF-FF`). Host có IP trùng khớp sẽ phản hồi gói **ARP Reply** chứa MAC của mình dưới dạng **Unicast**.

### Câu 5: NAT là gì và giải quyết vấn đề gì?
- NAT (Network Address Translation) chuyển đổi địa chỉ IP Private trong mạng LAN sang IP Public của Router khi gói tin ra Internet và ngược lại, giúp giải quyết tình trạng cạn kiệt địa chỉ IPv4 và che giấu mạng nội bộ.

---

## 4. Giao tiếp thời gian thực: WebSocket vs Polling

### Câu 6: WebSocket khác gì so với HTTP Polling và Long Polling?
- **Short Polling**: Client định kỳ mỗi vài giây gửi một HTTP Request lên hỏi server -> Tốn băng thông, tiêu hao CPU vì liên tục bắt tay HTTP.
- **Long Polling**: Client gửi request, Server treo kết nối cho đến khi có dữ liệu mới trả về -> Vẫn tốn chi phí đóng mở kết nối liên tục.
- **WebSocket**: Nâng cấp kết nối từ HTTP sang kết nối TCP hai chiều bền vững (Full-Duplex), Server có thể chủ động đẩy tin nhắn (Push) xuống Client bất cứ lúc nào với header nhị phân siêu nhẹ (chỉ 2 byte).

<!-- @include: @article-footer.snippet.md -->
