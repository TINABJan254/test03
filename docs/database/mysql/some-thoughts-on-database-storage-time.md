---
title: MySQL日期类型选择建议
description: 深入对比MySQL中DATETIME和TIMESTAMP的区别，分析时区处理、存储空间、取值范围等差异，给出日期类型选择的最佳实践建议。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL时间存储,DATETIME,TIMESTAMP,时间戳,时区处理,日期类型选择,MySQL日期函数
---

Trong công việc phát triển phần mềm hàng ngày, việc lưu trữ thời gian là một nhu cầu cơ bản và rất phổ biến. Dù là ghi lại thời gian thao tác dữ liệu, thời gian giao dịch tài chính, thời gian xuất phát lịch trình, hay thời gian đặt hàng của người dùng..., thông tin thời gian đều gắn kết chặt chẽ với logic nghiệp vụ và tính năng hệ thống. Do đó, việc lựa chọn và sử dụng chính xác kiểu ngày tháng thời gian trong MySQL là cực kỳ quan trọng, thậm chí ảnh hưởng trực tiếp tới tính chính xác của nghiệp vụ và độ ổn định của hệ thống.

Bài viết này nhằm giúp lập trình viên xem xét lại và hiểu sâu về các phương thức lưu trữ thời gian khác nhau trong MySQL để đưa ra lựa chọn phù hợp nhất với kịch bản nghiệp vụ dự án.

## Không sử dụng Chuỗi (String) để lưu trữ Ngày tháng

Giống như nhiều người mới bắt đầu với database, tác giả ở giai đoạn đầu học tập cũng từng thử dùng kiểu Chuỗi (như `VARCHAR`) để lưu trữ ngày tháng, thậm chí từng cho rằng đây là một cách đơn giản trực quan. Dù sao thì định dạng `'YYYY-MM-DD HH:MM:SS'` nhìn rất rõ ràng dễ hiểu.

Tuy nhiên, đây là cách làm không chính xác, chủ yếu gây ra 2 vấn đề:

1. **Hiệu quả dung lượng kém**: So với các kiểu ngày tháng nội dựng tự có của MySQL, Chuỗi thường tốn nhiều dung lượng lưu trữ hơn để biểu diễn cùng một thông tin thời gian.
2. **Hiệu năng truy vấn và tính toán cực kỳ thấp**:
   - **Thao tác so sánh phức tạp và kém hiệu quả**: So sánh ngày dựa trên chuỗi phải so sánh từng ký tự theo thứ tự từ điển (ví dụ `'2024-05-01'` sẽ nhỏ hơn `'2024-1-10'`), hiệu năng thấp hơn nhiều so với so sánh giá trị số hoặc mốc thời gian của kiểu thời gian nguyên bản.
   - **Tính năng tính toán bị hạn chế**: Không thể lợi dụng trực tiếp các hàm ngày tháng phong phú do DB cung cấp (như tính khoảng cách giữa 2 ngày, cộng trừ ngày tháng...), bắt buộc phải convert định dạng trước làm tăng độ phức tạp.
   - **Hiệu năng Index kém**: Index trên cột chuỗi khi xử lý truy vấn phạm vi (như tìm dữ liệu trong một khoảng thời gian) có hiệu quả kém hơn nhiều so với Index kiểu thời gian nguyên bản.

## Lựa chọn giữa DATETIME và TIMESTAMP

`DATETIME` và `TIMESTAMP` là 2 kiểu dữ liệu rất phổ biến trong MySQL dùng để lưu thông tin ngày và giờ. Cả hai đều lưu được thời gian chính xác tới hàng giây (MySQL 5.6.4+ hỗ trợ độ chính xác tới hàng phần triệu giây - fractional seconds). Trong ứng dụng thực tế, chúng ta nên lựa chọn thế nào?

Cùng so sánh qua các tiêu chí quan trọng:

### Thông tin Múi giờ (Time Zone)

Kiểu `DATETIME` lưu trữ **giá trị ngày giờ nguyên văn (literal value)**, bản thân nó **không chứa bất kỳ thông tin múi giờ nào**. Khi bạn insert một giá trị `DATETIME`, MySQL lưu trữ chính xác thời gian bạn cung cấp mà không thực hiện bất kỳ chuyển đổi múi giờ nào.

**Vậy sẽ có vấn đề gì?** Nếu ứng dụng cần hỗ trợ nhiều múi giờ, hoặc múi giờ của server/client có thể thay đổi, thì khi dùng `DATETIME`, ứng dụng phải tự xử lý việc chuyển đổi múi giờ. Nếu xử lý không khéo có thể dẫn tới hỗn loạn hiển thị hoặc tính toán thời gian.

`TIMESTAMP` **gắn liền với múi giờ**. Khi lưu trữ, MySQL chuyển đổi giá trị thời gian từ múi giờ session hiện tại thành UTC (Coordinated Universal Time) để lưu trữ nội bộ. Khi query trường `TIMESTAMP`, MySQL lại chuyển đổi thời gian UTC đã lưu về múi giờ hiện tại của session để hiển thị.

Điều này có nghĩa là, cùng 1 bản ghi trường `TIMESTAMP`, khi query ở các cấu hình múi giờ session khác nhau sẽ thấy thời gian hiển thị địa phương khác nhau, nhưng chúng đều tương ứng với 1 mốc thời gian tuyệt đối (thời gian UTC). Điều này rất hữu ích với các ứng dụng toàn cầu hóa, đa múi giờ.

Thử nghiệm thực tế:

Tạo bảng SQL:

```sql
CREATE TABLE `time_zone_test` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `date_time` datetime DEFAULT NULL,
  `time_stamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Chèn bản ghi (múi giờ session hiện tại là UTC+0):

```sql
INSERT INTO time_zone_test(date_time,time_stamp) VALUES(NOW(),NOW());
```

Query dữ liệu trong cùng session múi giờ:

```sql
SELECT date_time, time_stamp FROM time_zone_test;
```

Kết quả:

```plain
+---------------------+---------------------+
| date_time           | time_stamp          |
+---------------------+---------------------+
| 2020-01-11 09:53:32 | 2020-01-11 09:53:32 |
+---------------------+---------------------+
```

Đổi múi giờ session hiện tại sang UTC+8:

```sql
SET time_zone = '+8:00';
```

Query lại dữ liệu:

```bash
# Giá trị TIMESTAMP tự động chuyển đổi sang thời gian UTC+8
+---------------------+---------------------+
| date_time           | time_stamp          |
+---------------------+---------------------+
| 2020-01-11 09:53:32 | 2020-01-11 17:53:32 |
+---------------------+---------------------+
```

 các lệnh SQL xem và đặt múi giờ MySQL:

```sql
# Xem múi giờ session hiện tại
SELECT @@session.time_zone;
# Đặt múi giờ session hiện tại
SET time_zone = 'Europe/Helsinki';
SET time_zone = "+00:00";
# Xem múi giờ global
SELECT @@global.time_zone;
# Đặt múi giờ global
SET GLOBAL time_zone = '+8:00';
```

### Dung lượng chiếm dụng

Trong MySQL 5.6.4 trở về trước, dung lượng của DATETIME và TIMESTAMP là cố định (8 byte và 4 byte). Nhưng từ MySQL 5.6.4 trở đi, dung lượng của chúng thay đổi tùy theo độ chính xác của millisecond: DATETIME từ 5~8 byte, TIMESTAMP từ 4~7 byte.

### Phạm vi biểu diễn

`TIMESTAMP` biểu diễn phạm vi thời gian nhỏ hơn, chỉ tới năm 2038:

- `DATETIME`: '1000-01-01 00:00:00.000000' đến '9999-12-31 23:59:59.999999'
- `TIMESTAMP`: '1970-01-01 00:00:01.000000' UTC đến '2038-01-19 03:14:07.999999' UTC

### Hiệu năng

Do `TIMESTAMP` khi lưu trữ và truy xuất cần thực hiện chuyển đổi giữa UTC và múi giờ session hiện tại, quá trình này tốn thêm một chút overhead tính toán. `DATETIME` không liên quan tới chuyển đổi múi giờ nên xử lý đơn giản trực tiếp hơn, có thể mang lại lợi thế hiệu năng nhỏ trong kịch bản concurrency cực cao.

## Numeric Timestamp (Thời gian戳 dạng số) có phải lựa chọn tốt hơn?

Ngoài 2 kiểu trên, thực tế cũng thường dùng kiểu số nguyên (`INT` hoặc `BIGINT`) để lưu cái gọi là "Unix Timestamp" (tổng số giây hoặc millisecond tính từ 1970-01-01 00:00:00 UTC).

Phương thức lưu trữ này có các ưu điểm tương tự `TIMESTAMP`, việc sắp xếp và so sánh ngày có hiệu năng cao hơn, truyền dữ liệu xuyên hệ thống rất tiện vì chỉ là giá trị số. Nhược điểm là tính trực quan kém, không thể nhìn thấy ngay thời gian cụ thể.

Thao tác thực tế trong DB:

```sql
-- Chuyển chuỗi ngày giờ sang Unix Timestamp (giây)
mysql> SELECT UNIX_TIMESTAMP('2020-01-11 09:53:32');
+---------------------------------------+
| UNIX_TIMESTAMP('2020-01-11 09:53:32') |
+---------------------------------------+
|                            1578707612 |
+---------------------------------------+

-- Chuyển Unix Timestamp (giây) sang định dạng ngày giờ
mysql> SELECT FROM_UNIXTIME(1578707612);
+---------------------------+
| FROM_UNIXTIME(1578707612) |
+---------------------------+
| 2020-01-11 09:53:32       |
+---------------------------+
```

## Trong PostgreSQL không có DATETIME

Trong PostgreSQL (PG):
- `TIMESTAMP WITHOUT TIME ZONE` tương đương với `DATETIME` của MySQL.
- `TIMESTAMP WITH TIME ZONE` (`TIMESTAMPTZ`) tương đương với `TIMESTAMP` của MySQL.

## Tóm tắt

So sánh 3 cách lưu trữ thời gian:

| Kiểu | Dung lượng | Định dạng | Phạm vi | Có múi giờ không |
| ------------ | -------- | ------------------------------ | ------------------------------------------------------------ | -------------- |
| DATETIME | 5~8 byte | YYYY-MM-DD hh:mm:ss[.fraction] | 1000-01-01 00:00:00 ～ 9999-12-31 23:59:59 | Không |
| TIMESTAMP | 4~7 byte | YYYY-MM-DD hh:mm:ss[.fraction] | 1970-01-01 00:00:01 ～ 2038-01-19 03:14:07 | Có |
| Numeric Timestamp | 4/8 byte | Số thuần túy như 1578707612 | Sau 1970-01-01 00:00:01 | Không |

**Tóm tắt khuyến nghị lựa chọn:**

- Ưu thế cốt lõi của `TIMESTAMP` là khả năng tự động xử lý múi giờ. Nếu ứng dụng cần xử lý đa múi giờ, `TIMESTAMP` là lựa chọn tự nhiên (chú ý giới hạn năm 2038).
- Nếu kịch bản ứng dụng không liên quan chuyển đổi múi giờ, và cần biểu diễn thời gian sau năm 2038, `DATETIME` là lựa chọn thỏa đáng hơn.
- Nếu cực kỳ chú trọng hiệu năng so sánh hoặc cần truyền dữ liệu thời gian liên tục giữa các hệ thống, Numeric Timestamp là một tùy chọn mạnh mẽ.

<!-- @include: @article-footer.snippet.md -->
