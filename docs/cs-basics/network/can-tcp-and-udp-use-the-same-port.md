---
title: TCP và UDP có thể dùng chung một số cổng (Port) không? (Tầng giao vận)
description: Phân tích cơ chế quản lý cổng và Socket trong kernel hệ điều hành, giải thích tại sao TCP và UDP có thể lắng nghe trên cùng một số hiệu cổng.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: TCP, UDP, Cổng, Port, Socket, Quản lý cổng, Tầng giao vận, Kernel
---

Một câu hỏi phỏng vấn rất thú vị:

**"Một tiến trình TCP và một tiến trình UDP có thể cùng bind và lắng nghe trên cùng một số hiệu cổng (ví dụ cổng 8888) trên cùng một máy chủ được không?"**

Câu trả lời ngắn gọn: **HOÀN TOÀN ĐƯỢC.**

Bài viết này sẽ giải thích chi tiết nguyên lý từ góc độ hệ điều hành và ngăn xếp mạng (Network Stack).

---

## 1. Bản chất: Cổng (Port) thuộc về Tầng Giao Vận của từng giao thức riêng biệt

Nhiều người lầm tưởng rằng "Cổng là tài nguyên dùng chung duy nhất của toàn bộ máy tính". Nhưng trên thực tế:

**Số hiệu cổng (Port) được quản lý độc lập bên trong cấu trúc dữ liệu của từng giao thức ở Tầng Giao vận (Transport Layer).**

Trong nhân hệ điều hành (Kernel):
- Kernel duy trì một bảng Socket riêng cho giao thức **TCP** (quản lý bởi TCP Stack).
- Kernel duy trì một bảng Socket riêng biệt cho giao thức **UDP** (quản lý bởi UDP Stack).

Khi một gói tin IP đi từ tầng dưới lên:
1. Header của gói tin IP có trường **Protocol** (giá trị `6` đại diện cho TCP, giá trị `17` đại diện cho UDP).
2. Tầng mạng nhìn vào trường này để phân phối gói tin tới đúng Module xử lý tương ứng:
   - Nếu là TCP -> Chuyển cho TCP Stack tra cứu trong Bảng Socket TCP.
   - Nếu là UDP -> Chuyển cho UDP Stack tra cứu trong Bảng Socket UDP.

Vì hai bảng này hoàn toàn tách biệt, nên việc tồn tại một Socket TCP bind cổng 8888 và một Socket UDP bind cổng 8888 không hề gây ra bất kỳ xung đột nào!

---

## 2. Định danh duy nhất của một kết nối mạng

Một kết nối mạng được định danh duy nhất bởi **Bộ 5 thông số (5-tuple)**:

$$	ext{5-Tuple} = (	ext{Giao thức}, 	ext{IP nguồn}, 	ext{Cổng nguồn}, 	ext{IP đích}, 	ext{Cổng đích})$$

Nhờ có thành phần **Giao thức (Protocol)** trong bộ 5 thông số, hệ điều hành luôn luôn phân biệt chính xác gói tin thuộc về kết nối TCP hay UDP mà không bao giờ bị nhầm lẫn.

---

## 3. Ví dụ thực tế trong các giao thức mạng tiêu chuẩn

Trong thực tế, có rất nhiều dịch vụ mạng tiêu chuẩn sử dụng cùng một số hiệu cổng cho cả TCP và UDP:

| Dịch vụ | Số hiệu cổng | TCP dùng làm gì | UDP dùng làm gì |
| --- | ---: | --- | --- |
| **DNS** | 53 | Dùng khi truyền vùng (Zone Transfer) hoặc gói phản hồi > 512 bytes | Dùng cho các truy vấn phân giải tên miền thông thường hàng ngày |
| **DHCP** | 67 / 68 | Ít dùng | Dùng để cấp phát IP động trong mạng LAN |
| **NTP** | 123 | Ít dùng | Đồng bộ thời gian mạng |
| **SNMP** | 161 / 162 | Giám sát mạng tin cậy | Giám sát mạng thông thường |

---

## 4. Hai tiến trình cùng là TCP có dùng chung một cổng được không?

- Mặc định: Hai tiến trình TCP khác nhau không thể cùng bind vào cùng một địa chỉ IP và Port (sẽ báo lỗi `Address already in use`).
- Tuy nhiên, trong Linux từ kernel 3.9 trở đi, hỗ trợ tùy chọn socket **`SO_REUSEPORT`**: Cho phép nhiều tiến trình (hoặc nhiều luồng) cùng bind vào cùng một cổng TCP/UDP, kernel sẽ tự động cân bằng tải (Load Balancing) các kết nối đến giữa các tiến trình này. Đây là kỹ thuật tối ưu hiệu năng quan trọng được các Web Server như Nginx áp dụng rộng rãi.

<!-- @include: @article-footer.snippet.md -->
