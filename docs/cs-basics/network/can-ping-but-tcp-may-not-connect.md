---
title: Ping thông nhưng kết nối TCP thất bại: Tại sao? (Tổng hợp)
description: Phân tích nguyên nhân tại sao lệnh Ping thành công nhưng kết nối TCP lại thất bại, làm rõ sự khác biệt giữa ICMP và TCP cùng các điểm kiểm tra khi xử lý sự cố mạng.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: Ping, TCP, ICMP, Tường lửa, Cổng, Troubleshooting, Xử lý sự cố mạng
---

Trong công việc phát triển và vận hành hệ thống thực tế, chúng ta thường gặp tình huống:

**Chạy lệnh `ping 192.168.1.100` thấy phản hồi rất mượt mà (0% packet loss), nhưng khi kết nối MySQL hoặc gọi API HTTP trên máy đó thì lập tức báo lỗi Timeout hoặc Connection Refused.**

Tại sao lại có hiện tượng "Ping thì thông mà TCP lại không kết nối được"?

Bài viết này sẽ làm rõ bản chất và các nguyên nhân phổ biến nhất.

---

## 1. Bản chất: Ping và TCP sử dụng hai giao thức hoàn toàn khác nhau

Điểm cốt lõi đầu tiên cần phân biệt: **Ping không sử dụng giao thức TCP**.

- **Lệnh `Ping`**: Sử dụng giao thức **ICMP (Internet Control Message Protocol)** ở Tầng Mạng (Network Layer). ICMP không có khái niệm "Cổng" (Port) và không cần thiết lập kết nối (No Handshake). Lệnh Ping chỉ gửi gói tin `ICMP Echo Request` (Type 8) và đợi nhận lại `ICMP Echo Reply` (Type 0) từ tầng mạng của hệ điều hành đích.
- **Kết nối TCP**: Là giao thức ở Tầng Giao vận (Transport Layer), hoạt động dựa trên cặp `(IP, Port)` và bắt buộc phải thực hiện bắt tay 3 bước (**TCP 3-way Handshake**).

Do đó: **Ping thông chỉ chứng minh được đường truyền mạng ở Tầng IP giữa hai máy là thông suốt, hoàn toàn không đảm bảo dịch vụ ứng dụng trên cổng TCP tương ứng đang hoạt động bình thường!**

---

## 2. Các nguyên nhân phổ biến khiến Ping thông nhưng TCP không kết nối được

### Nguyên nhân 1: Tiến trình dịch vụ phía Server chưa khởi động hoặc chưa lắng nghe cổng

- **Hiện tượng**: Gửi request TCP bị trả về gói tin `RST` ngay lập tức hoặc báo lỗi `Connection refused`.
- **Giải thích**: Máy chủ đích đang bật (nên phản hồi Ping), nhưng ứng dụng phía trên (như MySQL cổng 3306, Tomcat cổng 8080) chưa được start, hoặc tiến trình bị crash.
- **Cách kiểm tra trên Server**:
  ```bash
  # Kiểm tra xem cổng có đang ở trạng thái LISTEN không
  netstat -tunlp | grep 8080
  # hoặc
  ss -tunlp | grep 8080
  ```

### Nguyên nhân 2: Dịch vụ chỉ lắng nghe trên `127.0.0.1` (Localhost) thay vì `0.0.0.0`

- **Hiện tượng**: Đứng trên chính máy server gọi `curl http://127.0.0.1:8080` thì được, nhưng máy khác trong mạng gọi `http://server_ip:8080` thì không kết nối được.
- **Giải thích**: Cấu hình `bind-address` trong file cấu hình ứng dụng đang để là `127.0.0.1`, khiến dịch vụ chỉ chấp nhận kết nối nội bộ từ loopback interface.
- **Cách khắc phục**: Đổi cấu hình lắng nghe sang `0.0.0.0` (Listen on all interfaces).

### Nguyên nhân 3: Tường lửa (Firewall / Security Group) chặn cổng TCP

- **Hiện tượng**: Gọi TCP bị treo chờ đến khi báo lỗi `Connection timed out`.
- **Giải thích**:
  - Tường lửa trên Server (`iptables`, `firewalld`, `ufw`, Windows Firewall) hoặc Security Group trên Cloud (AWS, Alibaba Cloud, GCP) cho phép gói tin ICMP đi qua (nên Ping được) nhưng đã chặn (DROP) các gói tin TCP trên cổng tương ứng.
- **Cách kiểm tra**:
  ```bash
  # Dùng telnet hoặc nc (netcat) kiểm tra trực tiếp cổng TCP
  telnet server_ip 8080
  nc -zv server_ip 8080
  ```

### Nguyên nhân 4: Hàng đợi kết nối TCP của Server bị đầy (SYN Queue / Accept Queue)

- **Hiện tượng**: Thỉnh thoảng kết nối được, lúc cao điểm lại bị timeout hoặc mất kết nối.
- **Giải thích**: Khi server chịu tải quá cao, số lượng request đến ồ ạt làm tràn hàng đợi `backlog` (SYN Queue hoặc Accept Queue), kernel Linux sẽ âm thầm drop các gói tin `SYN` mới đến.

---

## 3. Tổng kết quy trình xử lý sự cố (Troubleshooting Checklist)

Khi gặp lỗi không kết nối được:
1. `ping target_ip`: Kiểm tra tính thông suốt ở Tầng IP.
2. `telnet target_ip port` hoặc `nc -zv target_ip port`: Kiểm tra kết nối TCP trên cổng cụ thể.
3. Trên server đích: Chạy `ss -tunlp` kiểm tra tiến trình có đang LISTEN trên `0.0.0.0:port` không.
4. Kiểm tra Firewall cục bộ (`iptables -L -n`) và Security Group trên Cloud.

<!-- @include: @article-footer.snippet.md -->
