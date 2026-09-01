---
title: MySQL高性能优化规范建议总结
description: MySQL高性能优化规范建议总结，涵盖数据库命名规范、表设计规范、字段设计规范、索引设计规范、SQL编写规范等，帮助你构建高效稳定的数据库系统。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL优化规范,数据库设计规范,索引设计,SQL编写规范,慢查询优化,字段类型选择,表结构设计
---

> Tác giả: 听风 Bài gốc: <https://www.cnblogs.com/huchong/p/10219318.html>.
>
> JavaGuide đã được tác giả ủy quyền và hoàn thiện bổ sung nội dung.

## Quy chuẩn đặt tên cơ sở dữ liệu

- Tên của tất cả các đối tượng cơ sở dữ liệu bắt buộc phải sử dụng chữ cái viết thường và phân cách bằng dấu gạch dưới `_`.
- Tên đối tượng cấm sử dụng các từ khóa bảo lưu của MySQL (nếu tên bảng chứa từ khóa khi query phải bọc bằng dấu ngoặc kép/đơn).
- Đặt tên phải thấy tên hiểu nghĩa, tốt nhất không vượt quá 32 ký tự.
- Bảng tạm bắt buộc dùng tiền tố `tmp_` và hậu tố ngày. Bảng backup bắt buộc dùng tiền tố `bak_` và hậu tố ngày (timestamp).
- Tất cả các cột lưu trữ dữ liệu giống nhau phải thống nhất tên và kiểu dữ liệu (thường làm cột JOIN, nếu kiểu dữ liệu không thống nhất sẽ tự động chuyển đổi ngầm định làm vỡ Index).

## Quy chuẩn thiết kế cơ bản cơ sở dữ liệu

### Tất cả các bảng bắt buộc dùng Storage Engine InnoDB

Trừ khi có yêu cầu đặc biệt (InnoDB không đáp ứng như Columnar Storage, Spatial Data...), tất cả các bảng bắt buộc dùng Storage Engine InnoDB.

InnoDB hỗ trợ Transaction, Row-level Lock, khôi phục dữ liệu tốt hơn và hiệu năng cao hơn khi đồng thời lớn.

### Charset của Database và Bảng thống nhất dùng UTF8 (utf8mb4)

Khả năng tương thích tốt hơn, tránh lỗi font do chuyển đổi charset. Nếu có nhu cầu lưu trữ emoji, charset bắt buộc dùng `utf8mb4`.

Bài viết tham khảo: [Chi tiết Charset trong MySQL](../character-set.md).

### Tất cả các bảng và field đều phải thêm Comment (Ghi chú)

Sử dụng mệnh đề COMMENT để thêm ghi chú cho bảng và cột, bảo trì từ điển dữ liệu ngay từ đầu.

### Khống chế dung lượng bảng đơn dưới 5 triệu bản ghi

5 triệu không phải là giới hạn của MySQL, nhưng bảng quá lớn sẽ gây khó khăn lớn khi sửa cấu trúc bảng, backup, restore.

Có thể dùng lưu trữ lịch sử (dành cho log) hoặc Sharding (dành cho dữ liệu nghiệp vụ) để kiểm soát kích thước dữ liệu.

### Thận trọng khi sử dụng Partition Table trong MySQL

Bảng phân vùng về mặt vật lý là nhiều file, về mặt logic là một bảng. Chọn Partition Key cẩn thận vì truy vấn xuyên partition có thể chậm hơn.

Khuyên dùng phân bảng vật lý (Sharding) để quản lý Big Data.

### Các cột thường xuyên sử dụng cùng nhau nên đưa vào cùng một bảng

Tránh các thao tác JOIN phức tạp.

### Cấm tạo field dự phòng (Reserved Fields) trong bảng

- Đặt tên field dự phòng khó thấy tên hiểu nghĩa.
- Không xác định được kiểu dữ liệu lưu trữ nên không thể chọn kiểu dữ liệu phù hợp.
- Thay đổi kiểu field dự phòng về sau sẽ gây khóa bảng.

### Cấm lưu trữ file (hình ảnh, video) hoặc dữ liệu nhị phân lớn vào cơ sở dữ liệu

Lưu file vào DB ảnh hưởng nghiêm trọng đến hiệu năng và tốn dung lượng lưu trữ. Lưu file vào File Server/Object Storage, DB chỉ lưu đường dẫn URL.

### Đừng bị trói buộc quá mức bởi chuẩn hóa Database (Normal Forms)

Thông thường thiết kế DB cần đạt chuẩn 3NF, nhưng đôi khi để tăng hiệu năng query ta có thể giảm yêu cầu chuẩn hóa bằng cách lưu trữ một số dữ liệu dư thừa (Denormalization / 反范式). Tuy nhiên phải điều độ.

### Cấm thực hiện Stress Test trên Production Database

### Cấm từ môi trường Dev/Test kết nối trực tiếp vào Production Database

Hiểm họa an toàn cực lớn!

## Quy chuẩn thiết kế field

### Ưu tiên lựa chọn kiểu dữ liệu nhỏ nhất đáp ứng đủ nhu cầu

Kích thước lưu trữ càng nhỏ, dung lượng chiếm dụng càng ít, hiệu năng càng tốt.

**a. Một số chuỗi có thể chuyển thành kiểu số để lưu trữ, ví dụ chuyển địa chỉ IP thành số nguyên.**

Số liên tục cho hiệu năng tốt hơn và tốn ít dung lượng hơn.

MySQL cung cấp 2 hàm xử lý địa chỉ IP:
- `INET_ATON()`: Chuyển IP thành số nguyên unsigned (4-8 byte);
- `INET_NTOA()`: Chuyển số nguyên IP thành địa chỉ IP chuỗi.

**b. Dữ liệu không âm (như ID tự tăng, IP dạng số, tuổi) nên ưu tiên dùng số nguyên UNSIGNED.**

UNSIGNED gấp đôi phạm vi dương so với SIGNED:

```sql
SIGNED INT -2147483648 ~ 2147483647
UNSIGNED INT 0 ~ 4294967295
```

**c. Các kiểu số nhỏ (tuổi, cờ trạng thái 0/1) ưu tiên dùng TINYINT.**

### Tránh sử dụng kiểu dữ liệu TEXT, BLOB

Memory temporary table không hỗ trợ TEXT, BLOB. Nếu query chứa các field này, khi sắp xếp bắt buộc phải dùng disk temporary table làm giảm hiệu năng.

Nếu bắt buộc dùng, khuyên bạn nên tách cột TEXT/BLOB sang một bảng mở rộng riêng.

### Tránh sử dụng kiểu ENUM

Sửa giá trị ENUM phải dùng ALTER TABLE. Thao tác ORDER BY trên ENUM hiệu suất kém.

### Cố gắng định nghĩa tất cả các cột là NOT NULL

Trừ khi có lý do đặc biệt, nên để cột là NOT NULL. Cột NULL tốn thêm không gian lưu trữ và cần xử lý đặc biệt khi so sánh/tính toán.

### Tuyệt đối không dùng Chuỗi (String) để lưu trữ Ngày tháng (Date)

Nên cân nhắc DATETIME, TIMESTAMP hoặc Numeric Timestamp (Timestamp dạng số).

| Kiểu | Kích thước | Định dạng | Phạm vi | Múi giờ |
| ------------ | -------- | ------------------------------ | ------------------------------------------------------------ | -------------- |
| DATETIME | 5~8 byte | YYYY-MM-DD hh:mm:ss[.fraction] | 1000-01-01 00:00:00 ～ 9999-12-31 23:59:59 | Không |
| TIMESTAMP | 4~7 byte | YYYY-MM-DD hh:mm:ss[.fraction] | 1970-01-01 00:00:01 ～ 2038-01-19 03:14:07 | Có |
| Numeric Timestamp | 4 byte | Số thuần túy như 1578707612 | Sau 1970-01-01 00:00:01 | Không |

### Dữ liệu tiền tệ tài chính bắt buộc dùng DECIMAL

- FLOAT, DOUBLE: Số thực dấu phẩy động không chính xác.
- DECIMAL: Số thực định điểm chính xác, không mất độ chính xác khi tính toán.

### Bảng đơn không chứa quá nhiều field

Nếu một bảng có quá nhiều field, nên cân nhắc tách thành nhiều bảng.

## Quy chuẩn thiết kế Index

### Giới hạn số Index trên mỗi bảng (khuyên dùng <= 5 Index)

Index không phải càng nhiều càng tốt! Index tăng hiệu năng query nhưng giảm hiệu năng insert/update.

### Cấm sử dụng Full-text Index trong kịch bản OLTP

### Cấm tạo Index riêng lẻ cho mọi cột

### Mọi bảng InnoDB bắt buộc phải có Primary Key

InnoDB lưu trữ dữ liệu theo thứ tự Primary Key (Clustered Index).
- Không dùng cột cập nhật thường xuyên làm Primary Key.
- Không dùng UUID, MD5, HASH, String làm Primary Key (không đảm bảo tính tăng dần tự nhiên).
- Primary Key nên dùng ID tự tăng.

### Khuyến nghị các cột tạo Index
- Cột trong mệnh đề WHERE của SELECT, UPDATE, DELETE.
- Cột trong ORDER BY, GROUP BY, DISTINCT.
- Cột dùng để JOIN đa bảng.

### Lựa chọn thứ tự cột trong Composite Index
- **Cột có độ phân biệt (Selectivity) cao nhất nằm ở ngoài cùng bên trái**: Tính bằng `count(distinct column) / count(*)`.
- **Cột được sử dụng thường xuyên nhất nằm ở bên trái**.

### Tránh tạo Redundant Index và Duplicate Index
- Duplicate Index: `primary key(id)`, `index(id)`, `unique index(id)`.
- Redundant Index: `index(a,b,c)`, `index(a,b)`, `index(a)`.

### Ưu tiên dùng Covering Index cho các truy vấn tần suất cao

### Tránh sử dụng Foreign Key (Khóa ngoại)
- Không khuyến nghị dùng Foreign Key (foreign key) ở tầng database, nhưng bắt buộc tạo Index trên các cột liên kết.
- Ràng buộc dữ liệu nên thực hiện ở tầng ứng dụng/nghiệp vụ.

## Quy chuẩn phát triển SQL

### Hạn chế tính toán trong Database, chuyển tính toán phức tạp lên tầng ứng dụng

Database dùng để lưu trữ và quản lý dữ liệu, không nên gánh hậu quả tính toán quá tải.

### Tối ưu hóa các câu SQL ảnh hưởng lớn tới hiệu năng

Dùng Slow Query Log để phát hiện các câu SQL cần tối ưu.

### Tận dụng Index sẵn có trên bảng

Tránh query `LIKE '%123%'`. Dùng `LEFT JOIN` hoặc `NOT EXISTS` thay cho `NOT IN`.

### Cấm dùng `SELECT *`, bắt buộc dùng `SELECT <danh_sách_field>`

- `SELECT *` tốn CPU, tốn băng thông truyền dữ liệu mạng.
- `SELECT *` không tận dụng được Covering Index.

### Cấm dùng câu lệnh INSERT không có danh sách cột

**Nên dùng**: `INSERT INTO t(c1, c2) VALUES ('a', 'b');`

### Khuyên dùng PreparedStatement cho các thao tác Database

Tái sử dụng Execution Plan, chống SQL Injection.

### Tránh chuyển đổi kiểu ngầm định

### Tránh dùng Subquery, tối ưu Subquery thành JOIN

### Tránh JOIN quá nhiều bảng (khuyên dùng <= 5 bảng)

### Giảm số lần tương tác với Database (gộp thao tác batch)

### Dùng `IN` thay cho `OR` cho cùng một cột (danh sách IN <= 500)

### Cấm dùng `ORDER BY RAND()` để sắp xếp ngẫu nhiên

### Mệnh đề WHERE cấm dùng hàm và tính toán trên cột Index

### Sử dụng `UNION ALL` thay cho `UNION` khi không có trùng lặp

### Chia nhỏ SQL phức tạp thành nhiều SQL nhỏ

## Quy chuẩn thao tác vận hành

### Thao tác ghi hàng loạt (UPDATE, DELETE, INSERT) > 1 triệu hàng phải chia nhỏ thực hiện nhiều đợt

Tránh trễ Master-Slave nghiêm trọng, tránh binlog quá lớn, tránh Transaction quá lớn gây khóa bảng kéo dài.

### Đối với bảng lớn dùng `pt-online-schema-change` để sửa cấu trúc bảng

Tránh khóa bảng khi thực hiện DDL trên bảng lớn.

### Cấm cấp quyền SUPER cho tài khoản ứng dụng

### Tài khoản ứng dụng tuân thủ nguyên tắc quyền hạn tối thiểu

## Đọc thêm

<!-- @include: @article-footer.snippet.md -->
