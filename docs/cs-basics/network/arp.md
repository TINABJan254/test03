---
title: Chi tiết giao thức ARP (Tầng mạng)
description: Giải thích chi tiết cơ chế phân giải địa chỉ và quy trình gói tin của ARP, kết hợp bảng ARP Table, cơ chế Broadcast hỏi / Unicast trả lời, các kiểu tấn công ARP Spoofing và chiến lược phòng thủ.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: ARP, Phân giải địa chỉ, IP sang MAC, Broadcast, Unicast, Bảng ARP, ARP Spoofing
---

Địa chỉ IP chịu trách nhiệm định tuyến ở tầng mạng, nhưng khi khung dữ liệu (Data Frame) thực sự được chuyển tiếp trong mạng cục bộ (LAN), thiết bị bắt buộc phải biết địa chỉ MAC của thiết bị ở chặng kế tiếp (Next Hop).

Giao thức ARP giải quyết chính xác bài toán chuyển đổi này: **Đã biết địa chỉ IP đích, làm thế nào để tìm ra địa chỉ MAC tương ứng**. Nhìn bề ngoài có vẻ đơn giản, nhưng ARP là chiếc cầu nối giữa Tầng Mạng (Network Layer) và Tầng Liên kết dữ liệu (Data Link Layer), đồng thời là nền tảng để hiểu về giao tiếp LAN, chuyển tiếp Gateway và tấn công ARP Spoofing.

Bài viết này chủ yếu trả lời các câu hỏi:

1. ARP nằm ở vị trí nào trong chồng giao thức (Protocol Stack)?
2. ARP hoàn thành phân giải địa chỉ thông qua cơ chế "Hỏi bằng Broadcast, Trả lời bằng Unicast" như thế nào?
3. Bảng ARP (ARP Table) có tác dụng gì, khi cache hết hạn sẽ ảnh hưởng ra sao?
4. Các cuộc tấn công ARP phổ biến diễn ra như thế nào và cách phòng thủ?

## Địa chỉ MAC

Trước khi tìm hiểu ARP, chúng ta cần hiểu rõ về địa chỉ MAC.

MAC là viết tắt của **Media Access Control Address (Địa chỉ điều khiển truy cập môi trường)**, dùng để định danh duy nhất một giao diện mạng (Network Interface) ở tầng liên kết dữ liệu trong mạng cục bộ. Địa chỉ MAC thuộc về Card mạng / Network Interface chứ không phải là ID vĩnh viễn của toàn bộ thiết bị vật lý; một máy tính hay router có nhiều cổng mạng thì mỗi cổng sẽ có một địa chỉ MAC riêng biệt.

![Mặt sau router ghi địa chỉ MAC](https://oss.javaguide.cn/github/javaguide/cs-basics/network/router-back-will-indicate-mac-address.png)

Địa chỉ MAC thường dài 6 byte (48 bit), biểu diễn dưới dạng Hexadecimal (ví dụ: `00:1A:2B:3C:4D:5E`).

Địa chỉ MAC có một địa chỉ đặc biệt: `FF-FF-FF-FF-FF-FF` (toàn bộ bit 1), đại diện cho **Địa chỉ Broadcast (Địa chỉ quảng bá)**.

## Nguyên lý hoạt động của giao thức ARP

Tiền đề hoạt động của ARP là **Bảng ARP (ARP Table / ARP Cache)**.

Trong mạng LAN, mỗi thiết bị tự duy trì một Bảng ARP lưu trữ các ánh xạ giữa địa chỉ IP và địa chỉ MAC dưới dạng bộ ba `<IP, MAC, TTL>`. Trong đó TTL (Time To Live) là thời gian sống của bản ghi (thường là 20 phút), quá thời gian này bản ghi sẽ bị xóa để cập nhật mới.

Quy trình hoạt động của ARP được chia thành 2 kịch bản:
1. **Tìm kiếm MAC trong cùng mạng cục bộ (Same LAN)**;
2. **Tìm kiếm thiết bị ở mạng cục bộ khác (Different LAN / Qua Router)**.

### 1. Tìm kiếm MAC trong cùng mạng cục bộ

Giả sử Host A (`137.196.7.23`) muốn gửi gói tin IP cho Host B (`137.196.7.14`) trong cùng một mạng LAN.

Quy trình diễn ra theo thứ tự thời gian:

1. Host A tra cứu Bảng ARP của mình, phát hiện chưa có bản ghi nào cho IP của Host B.
2. Host A tạo một gói tin **ARP Request** (chứa IP nguồn, MAC nguồn, IP đích của B, và MAC đích tạm để trống/toàn 0).
3. Host A đóng gói ARP Request vào một Ethernet Frame với địa chỉ MAC đích là địa chỉ Broadcast `FF-FF-FF-FF-FF-FF` và gửi quảng bá ra toàn bộ mạng LAN.
4. Mọi thiết bị trong LAN đều nhận được Ethernet Frame này. Các thiết bị khác thấy IP đích trong ARP Request không phải của mình nên âm thầm hủy gói tin. Riêng Host B nhận thấy IP đích trùng với IP của mình:
   - Host B trích xuất thông tin IP và MAC của Host A để lưu vào Bảng ARP của chính mình (nhờ đó B biết luôn MAC của A mà không cần hỏi lại).
   - Host B tạo một gói tin **ARP Reply** chứa địa chỉ MAC của mình.
5. Host B gửi gói tin ARP Reply dưới dạng **Unicast (Đơn phát)** trực tiếp về địa chỉ MAC của Host A.
6. Host A nhận được ARP Reply từ B, lưu cặp `IP_B - MAC_B` vào Bảng ARP của mình và bắt đầu đóng gói Frame gửi dữ liệu thực sự.

![Tìm kiếm MAC trong cùng LAN qua ARP](./images/arp/arp_same_lan.png)

Tóm lại: ARP hoạt động theo nguyên tắc **Broadcast khi hỏi (Query), Unicast khi trả lời (Reply)**.

### 2. Tìm kiếm MAC qua các mạng khác nhau (Cross-LAN / Qua Router)

Khi Host A muốn gửi gói tin cho Host B nằm ở một mạng con (Subnet) khác qua Router:

1. Host A kiểm tra IP của B và Subnet Mask, nhận thấy B không cùng mạng LAN với mình.
2. Host A nhận ra gói tin phải được chuyển tiếp qua **Default Gateway (Cổng mặc định - tức giao diện Router trong mạng của A)**.
3. Host A sử dụng ARP để tìm kiếm **địa chỉ MAC của Router Gateway** (chứ không tìm MAC của B).
4. Host A đóng gói IP Datagram (với IP nguồn = A, IP đích = B) vào Ethernet Frame (với MAC nguồn = A, **MAC đích = MAC của Router Gateway**) và gửi cho Router.
5. Router nhận được Frame, bóc tách lấy gói tin IP, tra cứu Bảng định tuyến (Routing Table) và chuyển tiếp gói tin sang giao diện mạng kết nối với mạng con của B.
6. Giao diện mạng của Router dùng ARP để tìm địa chỉ MAC của Host B trong mạng con đích.
7. Router đóng gói lại Frame với **MAC nguồn = MAC của Router, MAC đích = MAC của Host B** và gửi tới B.

![Truyền thông tin khác LAN qua Router dùng ARP](./images/arp/arp_different_lan.png)

Lưu ý quan trọng: Trong suốt quá trình truyền gói tin qua nhiều chặng Router, **địa chỉ IP nguồn và IP đích ban đầu được giữ nguyên**, nhưng **địa chỉ MAC nguồn và MAC đích sẽ liên tục thay đổi ở mỗi chặng liên kết (Hop-by-Hop)**.

<!-- @include: @article-footer.snippet.md -->
