---
title: Chi tiết giao thức NAT (Tầng mạng)
description: Phân tích cơ chế chuyển đổi địa chỉ và ánh xạ cổng của NAT (NAPT), kết hợp truyền thông LAN/WAN và bảng chuyển đổi NAT Table để hiểu chi tiết thực tế trong mạng gia đình và doanh nghiệp.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: NAT, NAPT, Chuyển đổi địa chỉ, Ánh xạ cổng, LAN, WAN, Connection Tracking, DHCP
---

Rất nhiều thiết bị trong mạng gia đình, mạng công ty đều sử dụng dải địa chỉ IP Private (IP riêng tư) như `192.168.x.x`, `10.x.x.x`, `172.16.x.x` đến `172.31.x.x`. Những địa chỉ này không thể định tuyến trực tiếp trên mạng Internet công cộng, nhưng các thiết bị nội bộ vẫn có thể lướt web bình thường.

Phía sau điều kỳ diệu đó chính là **NAT (Network Address Translation)**. NAT thực hiện chuyển đổi giữa địa chỉ nội bộ (Private IP) và địa chỉ công cộng (Public IP), cho phép hàng trăm thiết bị nội bộ cùng chia sẻ một hoặc một số ít Public IP để giao tiếp ra bên ngoài.

Bài viết này chủ yếu trả lời các câu hỏi:

1. NAT chủ yếu giải quyết bài toán gì?
2. Bảng chuyển đổi NAT (NAT Translation Table) ghi lại ánh xạ địa chỉ và cổng như thế nào?
3. Khi máy nội bộ truy cập mạng ngoài, IP nguồn và Cổng nguồn thay đổi ra sao?
4. NAT mang lại những hạn chế gì, tại sao mạng ngoài chủ động truy cập vào máy nội bộ lại khó khăn?

## Bối cảnh và Tác dụng của NAT

Không gian địa chỉ IPv4 chỉ có $2^{32}$ địa chỉ (khoảng 4.3 tỷ), không đủ để cấp cho mỗi thiết bị trên toàn cầu một IP tĩnh công cộng duy nhất.

Để giải quyết vấn đề cạn kiệt IPv4:
- Bên trong mạng cục bộ (LAN), các thiết bị được cấp IP Private (do DHCP Server trên Router quản lý).
- Router đóng vai trò Gateway kết nối giữa LAN và WAN (Internet), giao diện WAN của Router được cấp một địa chỉ Public IP duy nhất từ nhà mạng ISP.
- Khi các thiết bị LAN gửi dữ liệu ra Internet, Router NAT sẽ thay thế IP Private nguồn bằng IP Public của Router, đồng thời gán một Port mới để phân biệt.

![Quy trình NAT chuyển đổi địa chỉ IP Private sang IP Public](https://oss.javaguide.cn/github/javaguide/cs-basics/network/nat-demo.png)

## Nguyên lý hoạt động chi tiết (NAPT - Network Address Port Translation)

Giả sử trong mạng LAN `10.0.0.0/24`, máy `10.0.0.1` muốn gửi HTTP Request tới Web Server `128.119.40.186:80`:

### Chiều gửi đi (LAN -> WAN)

1. Máy `10.0.0.1` chọn cổng nguồn ngẫu nhiên `3345`, gửi gói tin với `Src: 10.0.0.1:3345 -> Dest: 128.119.40.186:80` tới Router.
2. Router NAT nhận gói tin tại cổng LAN (`10.0.0.4`):
   - Router chọn một cổng Public rảnh rỗi trên giao diện WAN của mình, ví dụ `5001`.
   - Router sửa lại Header của gói tin: thay `10.0.0.1:3345` thành `138.76.29.7:5001`.
   - Router ghi một dòng vào **Bảng chuyển đổi NAT (NAT Table)**:
     ```text
     WAN Side: 138.76.29.7:5001  <--->  LAN Side: 10.0.0.1:3345
     ```
3. Router gửi gói tin đã sửa ra Internet tới Web Server `128.119.40.186:80`.

### Chiều nhận về (WAN -> LAN)

1. Web Server xử lý xong, gửi gói tin Response với `Src: 128.119.40.186:80 -> Dest: 138.76.29.7:5001`.
2. Router nhận được gói tin tại cổng WAN `138.76.29.7:5001`:
   - Router tra cứu Bảng NAT, tìm thấy ánh xạ `138.76.29.7:5001 -> 10.0.0.1:3345`.
   - Router sửa địa chỉ đích của gói tin thành `10.0.0.1:3345`.
3. Router chuyển tiếp gói tin vào mạng LAN tới máy `10.0.0.1`.

![Quy trình chuyển đổi địa chỉ hai chiều của NAT](https://oss.javaguide.cn/github/javaguide/cs-basics/network/nat-demo2.png)

## Đánh giá và Tổng kết

### Ưu điểm
- Tiết kiệm địa chỉ IPv4 công cộng một cách hiệu quả.
- Tăng cường an ninh mạng nội bộ: Bên ngoài Internet không thể nhìn thấy cấu trúc và địa chỉ IP thực của các máy bên trong mạng LAN.

### Hạn chế & Tranh cãi
- **Vi phạm nguyên tắc phân tầng mạng**: Router là thiết bị Tầng 3 (Network Layer) nhưng lại can thiệp và sửa đổi Port của Tầng 4 (Transport Layer).
- **Khó khăn cho giao tiếp ngang hàng (P2P)**: Các ứng dụng P2P (như BitTorrent, WebRTC, Game trực tuyến) cần các thiết bị kết nối trực tiếp với nhau. Khi cả hai máy đều nằm sau NAT, việc thiết lập kết nối đòi hỏi các kỹ thuật đục lỗ NAT phức tạp (NAT Traversal / STUN / TURN / ICE).
- **Không thay thế hoàn toàn cho Firewall**: NAT chỉ ẩn địa chỉ chứ không thay thế được các chính sách kiểm soát truy cập và tường lửa kiểm tra trạng thái (Stateful Firewall).

<!-- @include: @article-footer.snippet.md -->
