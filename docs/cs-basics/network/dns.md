---
title: Chi tiết Hệ thống tên miền DNS (Tầng ứng dụng)
description: Phân tích chi tiết kiến trúc phân tầng và quy trình phân giải DNS, bao gồm truy vấn đệ quy / lặp, bộ nhớ đệm cache, máy chủ có thẩm quyền (Authoritative Server), cổng tầng ứng dụng và các điểm tối ưu hiệu năng.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: DNS, Phân giải tên miền, Recursive Query, Iterative Query, Cache, Authoritative DNS, Cổng 53, UDP
---

Sau khi nhập tên miền vào thanh địa chỉ của trình duyệt, trước khi thực sự phát ra HTTP Request, thông thường phải trải qua bước phân giải DNS.

DNS giải quyết **bài toán ánh xạ giữa tên miền và địa chỉ IP**. Nhìn bề ngoài nó chỉ là "dịch tên miền thành IP", nhưng phía sau là cả một hệ thống bao gồm Local Cache, Recursive Query (Truy vấn đệ quy), Iterative Query (Truy vấn lặp), Authoritative Server (Máy chủ có thẩm quyền), Root Server (Máy chủ gốc) và cơ chế chuyển đổi UDP/TCP.

Bài viết này chủ yếu trả lời các câu hỏi:

1. Tại sao DNS lại cần thiết kế phân tầng?
2. Một quy trình phân giải tên miền hoàn chỉnh trải qua những bước nào?
3. Sự khác nhau giữa Truy vấn đệ quy (Recursive Query) và Truy vấn lặp (Iterative Query)?
4. Tại sao DNS thường chạy trên UDP, trường hợp nào chuyển sang TCP?

![Tổng quan hệ thống DNS phân giải tên miền thành địa chỉ IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/dns-overview.png)

Trong thực tế, có một trường hợp trình duyệt không cần dùng đến DNS mà vẫn biết được ánh xạ giữa tên miền và IP: Trình duyệt và hệ điều hành duy trì một danh sách file `hosts` cục bộ. Thông thường hệ thống sẽ kiểm tra xem tên miền cần truy cập có trong file `hosts` hay không, nếu có sẽ trực tiếp lấy IP tương ứng. Nếu trong file `hosts` không có bản ghi tương ứng, lúc này DNS mới chính thức vào cuộc.

Hiện tại thiết kế của DNS áp dụng cấu trúc cơ sở dữ liệu phân tán, phân tầng. **DNS là giao thức tầng ứng dụng, thông thường chạy trên giao thức UDP, cổng là 53**. Khi dữ liệu phản hồi vượt quá giới hạn độ dài gói tin UDP (512 byte, EDNS0 có thể mở rộng lớn hơn) hoặc khi thực hiện truyền vùng (Zone Transfer giữa các máy chủ DNS), DNS sẽ chuyển sang dùng giao thức TCP để đảm bảo tính toàn vẹn dữ liệu.

![Tổng quan các tầng giao thức TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/network-protocol-overview.png)

## Các loại DNS Server

DNS có thể được mô tả từ 2 góc độ: Phân tầng thẩm quyền (Authoritative Hierarchy) gồm Root Server, TLD Server và Authoritative Server của từng vùng cụ thể; phía truy vấn gồm Stub Resolver, Recursive Resolver và Forwarder.

- **Root DNS Server (Máy chủ DNS gốc)**: Cung cấp thông tin chuyển hướng (referral) tới các máy chủ TLD tương ứng.
- **Top-Level Domain (TLD) DNS Server (Máy chủ tên miền cấp cao nhất)**: Quản lý các đuôi tên miền như `com`, `org`, `net`, `edu` hoặc tên miền quốc gia như `vn`, `uk`, `jp`. TLD Server trả về thông tin máy chủ Authoritative của tên miền mục tiêu.
- **Authoritative DNS Server (Máy chủ DNS có thẩm quyền)**: Lưu trữ dữ liệu của một hoặc nhiều DNS Zone và đưa ra câu trả lời có thẩm quyền cho các truy vấn trong vùng đó.
- **Recursive Resolver (Trình phân giải đệ quy / Local DNS Server)**: Do mạng nội bộ, ISP hoặc các dịch vụ Public DNS (như 8.8.8.8, 1.1.1.1) cung cấp. Nó nhận truy vấn từ Client, kiểm tra cache trước, nếu không có sẽ thay mặt Client lần lượt hỏi Root Server, TLD Server và Authoritative Server.

**Trên thế giới thực sự chỉ có 13 máy chủ gốc thôi sao?** Đây là một hiểu lầm kỹ thuật phổ biến.

**Thực tế hoàn toàn không phải như vậy.**

Hệ thống máy chủ gốc về mặt logic có 13 định danh tên máy chủ gốc từ `a.root-servers.net` đến `m.root-servers.net`, do 12 tổ chức độc lập vận hành. Con số 13 này liên quan đến giới hạn kích thước gói tin UDP trong chuẩn DNS ban đầu, nhưng không có nghĩa là toàn cầu chỉ có 13 cỗ máy vật lý.

Phía sau mỗi định danh máy chủ gốc đó, thông qua công nghệ **IP Anycast (Định tuyến Anycast)**, người ta đã triển khai hàng nghìn máy chủ vật lý phân tán trên khắp thế giới. BGP sẽ tự động định tuyến truy vấn đến node máy chủ phù hợp nhất trên đường truyền mạng. Số lượng và vị trí các node được cập nhật liên tục tại **[Root-Servers.org](https://root-servers.org/)**.

![Phân bổ các node máy chủ gốc toàn cầu trên Root-Servers.org](https://oss.javaguide.cn/github/javaguide/cs-basics/network/root-servers-org.png)

## Quy trình hoạt động của DNS

Quy trình truy vấn phân giải DNS được chia thành 2 chế độ:
- **Iterative (Lặp)**
- **Recursive (Đệ quy)**

Trong thực tế, mô hình kết hợp được áp dụng phổ biến nhất: Truy vấn từ Client đến Local DNS Server là Đệ quy, còn truy vấn từ Local DNS Server tới các Server bên ngoài là Lặp.

![Quy trình phân giải kết hợp giữa truy vấn đệ quy và truy vấn lặp](https://oss.javaguide.cn/github/javaguide/cs-basics/network/DNS-process.png)

Ví dụ: Máy trạm `cis.poly.edu` muốn biết IP của `gaia.cs.umass.edu`:
1. Máy trạm `cis.poly.edu` gửi yêu cầu DNS tới Local DNS Server `dns.poly.edu`.
2. `dns.poly.edu` kiểm tra cache cục bộ không thấy, gửi request lên Root DNS Server.
3. Root DNS Server thấy đuôi `edu`, trả về địa chỉ TLD DNS Server phụ trách vùng `.edu`.
4. `dns.poly.edu` gửi tiếp request hỏi TLD DNS Server của `.edu`.
5. TLD DNS Server của `.edu` nhận thấy tiền tố `umass.edu`, trả về địa chỉ Authoritative DNS Server `dns.cs.umass.edu`.
6. `dns.poly.edu` gửi request tới Authoritative DNS Server `dns.cs.umass.edu`.
7. Authoritative DNS Server `dns.cs.umass.edu` trả về chính xác địa chỉ IP của `gaia.cs.umass.edu`.
8. Local DNS Server `dns.poly.edu` lưu cache lại và trả IP về cho máy trạm `cis.poly.edu`.

Ngoài ra còn có mô hình thuần Đệ quy (Recursive Query hoàn toàn):

![Quy trình phân giải tên miền thuần đệ quy](https://oss.javaguide.cn/github/javaguide/cs-basics/network/DNS-process2.png)

Recursive Resolver sẽ lưu cache lại kết quả các bản ghi tài nguyên và TTL (Time To Live). Miễn là cache còn hạn TTL, các lần truy vấn tiếp theo sẽ trả về ngay lập tức mà không cần đi lại toàn bộ chuỗi máy chủ gốc.

## Định dạng gói tin DNS (DNS Packet Format)

![Cấu trúc các trường trong gói tin DNS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/DNS-packet.png)

Gói tin DNS gồm truy vấn (Query) và trả lời (Response), cấu trúc cơ bản gồm:
- **Transaction ID (Định danh - 16 bit)**: Dùng để ghép cặp request và response tương ứng.
- **Flags (Cờ - 16 bit)**: Xác định là Query hay Response, cờ Authoritative, cờ yêu cầu đệ quy (Recursion Desired), cờ hỗ trợ đệ quy (Recursion Available), mã phản hồi (RCODE).
- **Questions Count, Answer RRs, Authority RRs, Additional RRs**: Số lượng các bản ghi trong 4 phần tương ứng.
- **Question Section**: Tên miền cần tra cứu và loại truy vấn (Type A, AAAA, MX, CNAME, etc.).
- **Answer Section**: Chứa các bản ghi tài nguyên (RR) trả về cho tên miền được hỏi. Một tên miền có thể có nhiều bản ghi IP khác nhau (hỗ trợ DNS Load Balancing).
- **Authority Section**: Chứa bản ghi của các Authoritative Server khác.
- **Additional Section**: Chứa thông tin bổ trợ hữu ích.

## Các loại bản ghi DNS (DNS Resource Record - RR)

Mỗi bản ghi tài nguyên trong cơ sở dữ liệu DNS gồm một bộ 4 trường: `(Name, Value, Type, TTL)`.

![Bộ tứ trường trong bản ghi tài nguyên DNS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/20210506174303797.png)

- **TTL (Time to Live)**: Thời gian sống của bản ghi trong bộ nhớ cache.
- Ý nghĩa của `Name` và `Value` tùy thuộc vào `Type`:

![Ý nghĩa Name và Value theo từng loại bản ghi DNS](https://oss.javaguide.cn/github/javaguide/cs-basics/network/20210506170307897.png)

- **Type=A**: `Name` là tên miền, `Value` là địa chỉ IPv4 tương ứng.
- **Type=AAAA**: `Name` là tên miền, `Value` là địa chỉ IPv6 tương ứng.
- **Type=CNAME (Canonical Name)**: `Value` là tên miền chuẩn tắc (chính thức) của tên miền bí danh `Name`. Dùng để tạo bí danh trỏ tới một tên miền khác.
- **Type=NS (Name Server)**: `Name` là tên miền, `Value` là tên máy chủ Authoritative DNS biết cách phân giải IP cho tên miền đó.
- **Type=MX (Mail Exchange)**: `Value` là tên máy chủ Mail Server chính thức của tên miền `Name`.

Ví dụ về bản ghi `CNAME`:

```plain
NAME                    TYPE   VALUE
--------------------------------------------------
bar.example.com.        CNAME  foo.example.com.
foo.example.com.        A      192.0.2.23
```

Khi người dùng truy vấn `bar.example.com`, DNS Server sẽ lần theo CNAME và trả về địa chỉ IP `192.0.2.23` của `foo.example.com`.

## Tài liệu tham khảo

- Các loại DNS Server: <https://www.cloudflare.com/learning/dns/dns-server-types/>
- Định dạng trường bản ghi thông điệp DNS: <http://www.tcpipguide.com/free/t_DNSMessageResourceRecordFieldFormats-2.htm>
- Tìm hiểu các loại bản ghi DNS: <https://www.mustbegeek.com/understanding-different-types-of-record-in-dns-server/>

<!-- @include: @article-footer.snippet.md -->
