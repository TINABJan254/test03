---
title: MySQL常见面试题总结
description: MySQL高频面试题精讲：基础架构、InnoDB引擎、索引原理、B+树、事务ACID、MVCC、redo/undo/binlog日志、行锁/表锁、慢查询优化，一文速通大厂必考点！
category: 数据库
tag:
  - MySQL
  - 大厂面试
head:
  - - meta
    - name: keywords
      content: MySQL面试题,MySQL基础架构,InnoDB存储引擎,MySQL索引,B+树索引,事务隔离级别,redo log,undo log,binlog,MVCC,行级锁,慢查询优化
---

## Cơ sở MySQL

### Cơ sở dữ liệu quan hệ (Relational Database) là gì?

Đúng như tên gọi, Cơ sở dữ liệu quan hệ (RDB, Relational Database) là loại cơ sở dữ liệu được xây dựng dựa trên mô hình quan hệ. Mô hình quan hệ thể hiện mối liên hệ (1-1, 1-nhiều, nhiều-nhiều) giữa các dữ liệu được lưu trữ trong cơ sở dữ liệu.

Trong cơ sở dữ liệu quan hệ, dữ liệu của chúng ta được lưu trữ trong các bảng (table) khác nhau (ví dụ: bảng người dùng), mỗi hàng trong bảng lưu trữ một bản ghi dữ liệu (ví dụ: thông tin của một người dùng).

![Mối quan hệ bảng trong cơ sở dữ liệu quan hệ](https://oss.javaguide.cn/java-guide-blog/5e3c1a71724a38245aa43b02_99bf70d46cc247be878de9d3a88f0c44.png)

Hầu hết các cơ sở dữ liệu quan hệ đều sử dụng SQL để thao tác với dữ liệu. Hơn nữa, hầu hết các cơ sở dữ liệu quan hệ đều hỗ trợ 4 tính chất của Transaction (ACID).

**Có những cơ sở dữ liệu quan hệ phổ biến nào?**

MySQL, PostgreSQL, Oracle, SQL Server, SQLite (việc lưu trữ lịch sử trò chuyện cục bộ của WeChat chính là sử dụng SQLite) ...

### SQL là gì?

SQL là Ngôn ngữ truy vấn có cấu trúc (Structured Query Language), được thiết kế chuyên biệt để tương tác với cơ sở dữ liệu, mục đích là cung cấp một phương pháp đơn giản và hiệu quả để đọc và ghi dữ liệu từ cơ sở dữ liệu.

Hầu như tất cả các cơ sở dữ liệu quan hệ chủ lưu đều hỗ trợ SQL, tính áp dụng rất cao. Ngoài ra, một số cơ sở dữ liệu NoSQL cũng tương thích với SQL hoặc sử dụng ngôn ngữ truy vấn tương tự như SQL.

SQL có thể giúp chúng ta:

- Tạo cơ sở dữ liệu, bảng dữ liệu, field mới;
- Thêm, xóa, sửa, truy vấn dữ liệu trong cơ sở dữ liệu;
- Tạo view, function, stored procedure mới;
- Tiến hành phân tích dữ liệu đơn giản đối với dữ liệu trong cơ sở dữ liệu;
- Kết hợp với Hive, Spark SQL để làm Big Data;
- Kết hợp với SQLFlow để làm Machine Learning;
- ...

### MySQL là gì?

![](https://oss.javaguide.cn/github/javaguide/csdn/20210327143351823.png)

**MySQL là một cơ sở dữ liệu quan hệ, chủ yếu dùng để lưu trữ bền vững (persistence) các dữ liệu trong hệ thống của chúng ta như thông tin người dùng.**

Do MySQL là hệ quản trị cơ sở dữ liệu mã nguồn mở miễn phí và khá trưởng thành, nên MySQL được sử dụng rộng rãi trong nhiều loại hệ thống. Bất kỳ ai cũng có thể tải về và tùy chỉnh theo nhu cầu cá nhân dưới giấy phép GPL (General Public License). Cổng (port) mặc định của MySQL là **3306**.

### ⭐️MySQL có những ưu điểm gì?

Câu hỏi này bản chất là đang hỏi về lý do khiến MySQL trở nên phổ biến đến vậy.

Sự thành công của MySQL có thể quy cho ưu thế tổng hợp ở 3 khía cạnh: **Hệ sinh thái, Tính năng và Vận hành bảo trì (O&M)**.

**Thứ nhất, từ góc độ hệ sinh thái và chi phí, rào cản phòng thủ của nó rất sâu.**

- **Mã nguồn mở miễn phí:** Đây là đá tảng giúp nó được phổ biến rộng rãi. Bất kỳ công ty hay cá nhân nào cũng có thể sử dụng miễn phí, giảm thiểu đáng kể ngưỡng kỹ thuật và chi phí ban đầu.
- **Cộng đồng lớn mạnh, hệ sinh thái hoàn thiện:** Qua hàng chục năm phát triển, MySQL sở hữu cộng đồng cực kỳ năng động và hệ sinh thái phong phú. Điều này có nghĩa là dù bạn gặp bất kỳ vấn đề gì, hầu như đều có thể tìm thấy giải pháp trên mạng; đồng thời, tất cả các ngôn ngữ lập trình, framework, ORM tool, hệ thống giám sát chủ lưu trên thị trường đều hỗ trợ hoàn hảo cho MySQL. Tài liệu của nó cũng rất phong phú, tài nguyên học tập dễ dàng có được.

**Thứ hai, từ góc độ tính năng kỹ thuật cốt lõi, nó rất mạnh mẽ và cân bằng.**

- **Hỗ trợ Transaction mạnh mẽ:** Đây là chỗ đứng cốt lõi của nó với tư cách là cơ sở dữ liệu quan hệ. Đáng chú ý là, mức độ cô lập REPEATABLE-READ mặc định của InnoDB, thông qua cơ chế MVCC và Next-Key Lock, đã tránh được vấn đề Phantom Read ở mức độ lớn, điều mà ở nhiều cơ sở dữ liệu khác cần mức độ cô lập cao hơn mới làm được, dung hòa được hiệu năng và tính nhất quán. Chi tiết có thể đọc bài viết: [Chi tiết mức độ cô lập Transaction trong MySQL](https://javaguide.cn/database/mysql/transaction-isolation-level.html).
- **Hiệu năng xuất sắc và tính mở rộng cao:** Bản thân MySQL đã trải qua thử thách khắc nghiệt của các nghiệp vụ Internet quy mô cực lớn, hiệu năng đơn máy rất ấn tượng. Quan trọng hơn, xoay quanh việc mở rộng theo chiều ngang (horizontal scaling), nó đã hình thành một bộ giải pháp kiến trúc rất trưởng thành, như Master-Slave replication, Read-Write splitting, cũng như Sharding (phân kho phân bảng) thông qua middleware. Điều này giúp nó có thể chống đỡ nghiệp vụ ở mọi quy mô từ công ty khởi nghiệp đến các nền tảng Internet lớn.

**Thứ ba, từ góc độ vận hành và sử dụng, nó rất "thân thiện".**

- **Dùng được ngay, dễ tiếp cận:** So với các cơ sở dữ liệu thương mại lớn như Oracle, việc cài đặt, cấu hình và sử dụng hàng ngày của MySQL đều rất đơn giản, trực quan, đường cong học tập bằng phẳng, rất thân thiện với lập trình viên và DBA sơ cấp.
- **Chi phí bảo trì thấp:** Nhờ tính đơn giản và cộng đồng khổng lồ, việc tìm kiếm nhân tài vận hành bảo trì và giải pháp liên quan tương đối dễ dàng, chi phí bảo trì tổng thể thấp hơn.

Đáng đề cập là trong những năm gần đây, đà phát triển của PostgreSQL rất mạnh mẽ. Trên mạng xuất hiện nhiều bài viết chỉ trích MySQL, tác giả cho rằng bất kỳ hành vi chỉ trích hay tâng bốc mù quáng bên nào cũng đều không nên.

Tác giả cũng từng viết bài chia sẻ góc nhìn về hai đại diện cơ sở dữ liệu quan hệ này: [MySQL bị đẩy xuống thứ hai rồi sao?](https://mp.weixin.qq.com/s/APWD-PzTcTqGUuibAw7GGw).

## Các kiểu field trong MySQL

Các kiểu field trong MySQL có thể chia thành 3 nhóm lớn:

- **Kiểu số (Numeric)**: Số nguyên (TINYINT, SMALLINT, MEDIUMINT, INT và BIGINT), Số thực dấu phẩy động (FLOAT và DOUBLE), Số định điểm (DECIMAL), Kiểu dữ liệu Bit (BIT)
- **Kiểu chuỗi (String)**: CHAR, VARCHAR, TINYTEXT, TEXT, MEDIUMTEXT, LONGTEXT, BINARY, TINYBLOB, BLOB, MEDIUMBLOB, LONGBLOB..., thường dùng nhất là CHAR và VARCHAR.
- **Kiểu ngày tháng thời gian (Date & Time)**: YEAR, TIME, DATE, DATETIME và TIMESTAMP...

Hình ảnh dưới đây tổng kết rất tốt các kiểu field phổ biến trong MySQL:

![Tổng kết các kiểu field phổ biến trong MySQL](https://oss.javaguide.cn/github/javaguide/mysql/summary-of-mysql-field-types.png)

Các kiểu field MySQL khá nhiều, ở đây sẽ chọn ra một số kiểu được sử dụng tần suất cao trong phát triển hàng ngày và thường gặp trong phỏng vấn để giới thiệu chi tiết. Nếu không có giải thích đặc biệt, đối tượng hướng đến đều là Storage Engine InnoDB.

Ngoài ra, nên đọc chương 4 của cuốn "High Performance MySQL (3rd Edition)", có giới thiệu chi tiết về tối ưu hóa kiểu field MySQL.

### ⭐️Thuộc tính UNSIGNED của kiểu số nguyên có tác dụng gì?

Kiểu số nguyên trong MySQL có thể sử dụng thuộc tính tùy chọn `UNSIGNED` để biểu thị số nguyên không dấu không cho phép giá trị âm. Sử dụng thuộc tính `UNSIGNED` có thể tăng giới hạn trên của số nguyên dương lên gấp đôi, vì nó không cần lưu trữ giá trị âm.

Ví dụ, kiểu `TINYINT UNSIGNED` có phạm vi giá trị là 0 ~ 255, trong khi `TINYINT` thông thường có phạm vi giá trị là -128 ~ 127. `INT UNSIGNED` có phạm vi là 0 ~ 4,294,967,295, trong khi `INT` thông thường là -2,147,483,648 ~ 2,147,483,647.

Đối với cột ID tự tăng từ 0, việc sử dụng thuộc tính `UNSIGNED` rất phù hợp vì không cho phép số âm và có giới hạn trên lớn hơn, cung cấp nhiều giá trị ID có sẵn hơn.

### Sự khác biệt giữa CHAR và VARCHAR là gì?

CHAR và VARCHAR là hai kiểu chuỗi được sử dụng phổ biến nhất, sự khác biệt chính giữa chúng là: **CHAR là chuỗi độ dài cố định (fixed-length), VARCHAR là chuỗi độ dài biến đổi (variable-length).**

CHAR khi lưu trữ sẽ tự động điền khoảng trắng ở bên phải để đạt được độ dài chỉ định, khi truy xuất sẽ xóa khoảng trắng; VARCHAR khi lưu trữ cần dùng 1 hoặc 2 byte bổ sung để ghi lại độ dài chuỗi, khi truy xuất không cần xử lý khoảng trắng.

CHAR phù hợp hơn cho việc lưu trữ chuỗi có độ dài ngắn hoặc độ dài xấp xỉ bằng nhau, chẳng hạn như mật khẩu mã hóa thuật toán Bcrypt, MD5, số CMND/CCCD. Kiểu VARCHAR phù hợp cho việc lưu trữ chuỗi có độ dài không cố định hoặc chênh lệch lớn, như biệt danh người dùng, tiêu đề bài viết.

M trong CHAR(M) và VARCHAR(M) đều đại diện cho số lượng ký tự tối đa có thể lưu trữ, bất kể là chữ cái, chữ số hay ký tự đa byte, mỗi ký tự chỉ tính là 1.

### Sự khác biệt giữa VARCHAR(100) và VARCHAR(10) là gì?

VARCHAR(100) và VARCHAR(10) đều là kiểu độ dài biến đổi, biểu thị có thể lưu trữ tối đa 100 ký tự và 10 ký tự. Do đó, VARCHAR(100) có thể đáp ứng nhu cầu lưu trữ ký tự phạm vi rộng hơn, có tính mở rộng nghiệp vụ tốt hơn. Còn VARCHAR(10) khi lưu trữ vượt quá 10 ký tự thì bắt buộc phải sửa cấu trúc bảng.

Tuy nhiên, dù VARCHAR(100) và VARCHAR(10) có phạm vi số ký tự có thể lưu trữ khác nhau, nhưng khi lưu cùng một chuỗi ký tự, dung lượng lưu trữ chiếm dụng trên đĩa cứng thực tế là như nhau, đây là điểm mà nhiều người dễ hiểu lầm.

Mặc dù vậy, VARCHAR(100) sẽ tiêu tốn nhiều bộ nhớ (RAM) hơn. Điều này là do khi kiểu VARCHAR thao tác trong bộ nhớ, thông thường sẽ cấp phát khối bộ nhớ kích thước cố định để lưu giá trị, tức là sử dụng độ dài được định nghĩa trong kiểu ký tự. Ví dụ khi thực hiện sắp xếp (sort), VARCHAR(100) sẽ được tiến hành theo độ dài 100, do đó tiêu tốn nhiều bộ nhớ hơn.

### Sự khác biệt giữa DECIMAL và FLOAT/DOUBLE là gì?

Sự khác biệt giữa DECIMAL và FLOAT/DOUBLE là: **DECIMAL là số định điểm (fixed-point), FLOAT/DOUBLE là số thực dấu phẩy động (floating-point). DECIMAL có thể lưu trữ giá trị số thập phân chính xác, FLOAT/DOUBLE chỉ có thể lưu trữ giá trị số thập phân xấp xỉ.**

DECIMAL dùng để lưu trữ các số thập phân có yêu cầu độ chính xác cao, chẳng hạn như dữ liệu liên quan đến tiền tệ, có thể tránh tổn thất độ chính xác do số thực dấu phẩy động gây ra.

Trong Java, kiểu DECIMAL của MySQL tương ứng với class Java `java.math.BigDecimal`.

### Tại sao không khuyến nghị sử dụng TEXT và BLOB?

Kiểu TEXT tương tự như CHAR (0-255 byte) và VARCHAR (0-65,535 byte), nhưng có thể lưu trữ chuỗi dài hơn, tức là dữ liệu văn bản dài, ví dụ nội dung blog.

| Kiểu | Dung lượng lưu trữ | Mục đích sử dụng |
| ---------- | -------------------- | -------------- |
| TINYTEXT | 0-255 byte | Chuỗi văn bản thông thường |
| TEXT | 0-65,535 byte | Chuỗi văn bản dài |
| MEDIUMTEXT | 0-16,772,150 byte | Dữ liệu văn bản khá lớn |
| LONGTEXT | 0-4,294,967,295 byte | Dữ liệu văn bản cực lớn |

Kiểu BLOB chủ yếu dùng để lưu trữ đối tượng nhị phân lớn (Binary Large Object), như hình ảnh, âm thanh, video...

| Kiểu | Dung lượng lưu trữ | Mục đích sử dụng |
| ---------- | ---------- | ------------------------ |
| TINYBLOB | 0-255 byte | Chuỗi nhị phân văn bản ngắn |
| BLOB | 0-65KB | Chuỗi nhị phân |
| MEDIUMBLOB | 0-16MB | Dữ liệu văn bản dài dạng nhị phân |
| LONGBLOB | 0-4GB | Dữ liệu văn bản cực lớn dạng nhị phân |

Trong phát triển hàng ngày, rất ít khi sử dụng kiểu TEXT (thỉnh thoảng dùng), còn kiểu BLOB thì cơ bản không dùng. Nếu phạm vi độ dài dự kiến có thể đáp ứng bằng VARCHAR, khuyến nghị tránh sử dụng TEXT.

Quy chuẩn database thường không khuyến nghị sử dụng kiểu BLOB và TEXT, hai kiểu này có một số nhược điểm và hạn chế như:

- Không thể có giá trị mặc định (DEFAULT value).
- Khi sử dụng bảng tạm (temporary table), không thể sử dụng memory temporary table mà chỉ có thể tạo temporary table trên đĩa (được đề cập trong sách High Performance MySQL).
- Hiệu suất truy xuất thấp hơn.
- Không thể trực tiếp tạo Index, cần chỉ định độ dài prefix.
- Có thể tiêu tốn băng thông mạng và I/O đĩa lớn.
- Có thể khiến các thao tác DML trên bảng bị chậm đi.
- ...

### ⭐️Sự khác biệt giữa DATETIME và TIMESTAMP là gì? Lựa chọn thế nào?

Kiểu DATETIME không có thông tin múi giờ (timezone), TIMESTAMP có liên quan đến múi giờ.

TIMESTAMP chỉ cần 4 byte dung lượng lưu trữ, nhưng DATETIME cần tiêu tốn 8 byte dung lượng lưu trữ. Tuy nhiên, điều này cũng gây ra một vấn đề: TIMESTAMP biểu thị phạm vi thời gian nhỏ hơn.

- DATETIME: '1000-01-01 00:00:00.000000' đến '9999-12-31 23:59:59.999999'
- TIMESTAMP: '1970-01-01 00:00:01.000000' UTC đến '2038-01-19 03:14:07.999999' UTC

Ưu thế cốt lõi của `TIMESTAMP` nằm ở khả năng xử lý múi giờ tích hợp sẵn. Database chịu trách nhiệm lưu trữ UTC và tự động chuyển đổi dựa trên múi giờ của session, giúp đơn giản hóa việc phát triển ứng dụng cần xử lý đa múi giờ. Nếu ứng dụng cần xử lý đa múi giờ hoặc muốn database tự động quản lý chuyển đổi múi giờ, `TIMESTAMP` là lựa chọn tự nhiên (chú ý hạn chế phạm vi thời gian của nó, tức là sự cố năm 2038).

Nếu kịch bản ứng dụng không liên quan đến chuyển đổi múi giờ, hoặc muốn ứng dụng hoàn toàn kiểm soát logic múi giờ, đồng thời cần biểu thị thời gian sau năm 2038, `DATETIME` là lựa chọn an toàn và vững chắc hơn.

Về so sánh chi tiết giữa hai kiểu và kiến nghị lựa chọn kiểu lưu trữ ngày tháng, vui lòng tham khảo bài viết: [Khuyến nghị lưu trữ dữ liệu kiểu thời gian trong MySQL](./some-thoughts-on-database-storage-time.md).

### Sự khác biệt giữa NULL và '' là gì?

`NULL` và `''` (chuỗi rỗng) là hai giá trị hoàn toàn khác nhau, chúng đại diện cho ý nghĩa khác nhau và có hành vi khác nhau trong cơ sở dữ liệu. `NULL` đại diện cho dữ liệu bị thiếu hoặc chưa xác định, còn `''` biểu thị một chuỗi rỗng đã biết là tồn tại. Sự khác biệt chính của chúng như sau:

1. **Ý nghĩa**:
   - `NULL` đại diện cho một giá trị không xác định, nó không bằng bất kỳ giá trị nào, bao gồm cả chính nó. Do đó, kết quả của `SELECT NULL = NULL` là `NULL`, chứ không phải `true` hay `false`. `NULL` có nghĩa là thông tin bị thiếu hoặc chưa biết. Mặc dù `NULL` không bằng bất kỳ giá trị nào, nhưng trong một số thao tác, hệ thống database sẽ coi các giá trị `NULL` thuộc cùng một nhóm để xử lý, ví dụ: `DISTINCT`, `GROUP BY`, `ORDER BY`. Cần lưu ý rằng việc các thao tác này coi giá trị `NULL` thuộc cùng một nhóm xử lý không có nghĩa là các giá trị `NULL` bằng nhau. Chúng chỉ được xử lý đặc biệt trong các thao tác nhất định để đảm bảo kết quả chính xác và nhất quán.
   - `''` biểu thị một chuỗi rỗng, là một giá trị đã biết.
2. **Dung lượng lưu trữ**:
   - Dung lượng lưu trữ của `NULL` phụ thuộc vào cách thực thi của database, thông thường cần một chút không gian để đánh dấu giá trị đó là rỗng.
   - Dung lượng lưu trữ của `''` thông thường rất nhỏ, vì nó chỉ lưu trữ cờ hiệu chuỗi rỗng, không cần lưu trữ ký tự thực tế.
3. **Phép toán so sánh**:
   - Bất kỳ giá trị nào so sánh với `NULL` (ví dụ `=`, `!=`, `>`, `<`...) kết quả đều là `NULL`, biểu thị kết quả không xác định. Để phán đoán một giá trị có phải `NULL` hay không, bắt buộc phải dùng `IS NULL` hoặc `IS NOT NULL`.
   - `''` có thể tiến hành phép toán so sánh như các chuỗi khác. Ví dụ kết quả của `'' = ''` là `true`.
4. **Hàm tổng hợp (Aggregate Functions)**:
   - Hầu hết các hàm tổng hợp (như `SUM`, `AVG`, `MIN`, `MAX`) sẽ bỏ qua giá trị `NULL`.
   - `COUNT(*)` sẽ thống kê tất cả các hàng, bao gồm các hàng chứa giá trị `NULL`. `COUNT(column_name)` sẽ thống kê các hàng có giá trị khác `NULL` trong cột chỉ định.
   - Chuỗi rỗng `''` sẽ được hàm tổng hợp tính vào. Ví dụ `SUM` sẽ coi nó là 0, `MIN` và `MAX` coi nó là một chuỗi rỗng.

Sau khi xem phần giới thiệu trên, tin rằng bạn cũng đã có câu trả lời cho một câu hỏi phỏng vấn tần suất cao khác: "Tại sao MySQL không khuyến nghị sử dụng `NULL` làm giá trị mặc định của cột?".

### ⭐️Kiểu Boolean được biểu thị như thế nào?

Trong MySQL không có kiểu Boolean chuyên biệt, `BOOL` và `BOOLEAN` là từ đồng nghĩa của `TINYINT(1)`, thông thường dùng 0 để biểu thị false, khác 0 biểu thị true. `BIT(1)` là kiểu bit field, cũng có thể lưu trữ 0 hoặc 1, nhưng nó không phải là ánh xạ thực tế của `BOOL`/`BOOLEAN`.

### ⭐️Lưu trữ số điện thoại nên dùng INT hay VARCHAR?

Lưu trữ số điện thoại, **mạnh mẽ khuyến nghị sử dụng kiểu VARCHAR**, chứ không nên dùng INT hay BIGINT. Lý do chính như sau:

1. **Tính tương thích định dạng và tính toàn vẹn:**
   - Số điện thoại có thể chứa số 0 ở đầu (như mã vùng điện thoại cố định), tiền tố mã quốc gia ('+'), thậm chí có dấu phân cách ('-' hoặc khoảng trắng). Kiểu số như INT hay BIGINT sẽ tự động làm mất các thông tin định dạng quan trọng này (ví dụ số 0 ở đầu sẽ bị xóa, '+' và '-' không thể lưu trữ).
   - VARCHAR có thể lưu nguyên văn các loại định dạng số điện thoại, dù là số di động 11 chữ số trong nước hay số quốc tế có mã quốc gia đều có thể tương thích hoàn hảo.
2. **Tính phi đại số (Non-arithmetic):** Số điện thoại tuy trông giống như số, nhưng chúng ta không bao giờ thực hiện các phép toán đại số trên đó (như tính tổng, trung bình cộng). Bản chất nó là một định danh (identifier), giống một chuỗi ký tự hơn. Dùng VARCHAR phù hợp hơn với bản chất dữ liệu.
3. **Tính linh hoạt khi truy vấn:**
   - Trong nghiệp vụ thường cần truy vấn theo đầu số (prefix), ví dụ tìm tất cả người dùng có đầu số "138". Sử dụng kiểu VARCHAR kết hợp với câu lệnh SQL như `LIKE '138%'` vừa trực quan vừa hiệu quả.
   - Nếu dùng kiểu số, việc thực hiện khớp prefix tương tự thường cần chuyển đổi hàm phức tạp (như CAST hoặc SUBSTRING), hoặc dùng truy vấn phạm vi (như `WHERE phone >= 13800000000 AND phone < 13900000000`), điều này không những viết rườm rà mà còn có thể không lợi dụng được Index hiệu quả, dẫn đến giảm hiệu năng.
4. **Yêu cầu lưu trữ mã hóa (cực kỳ quan trọng):**
   - Do yêu cầu an toàn dữ liệu và tuân thủ quyền riêng tư, thông tin cá nhân nhạy cảm như số điện thoại thông thường bắt buộc phải mã hóa khi lưu vào database.
   - Dữ liệu sau khi mã hóa (ciphertext) là một chuỗi ký tự dài (thường gồm chữ cái, chữ số, ký tự đặc biệt, hoặc qua mã hóa Base64/Hex), kiểu INT hay BIGINT hoàn toàn không thể lưu trữ loại ciphertext này. Chỉ có các kiểu như VARCHAR, TEXT hay BLOB mới đáp ứng được.

**Về việc lựa chọn độ dài VARCHAR:**

- **Nếu không lưu trữ mã hóa (mạnh mẽ KHÔNG khuyến nghị!):** Tính đến số quốc tế và các ký tự định dạng có thể có, VARCHAR(20) đến VARCHAR(32) thông thường là một phạm vi an toàn, đủ để bao phủ hầu hết định dạng số điện thoại toàn cầu.
- **Nếu tiến hành lưu trữ mã hóa (cách làm chuẩn được khuyến nghị):** Độ dài phải được tính toán và thiết lập chính xác dựa trên độ dài tối đa của ciphertext do thuật toán mã hóa được chọn tạo ra, cũng như phương thức encoding (như Base64 làm độ dài tăng khoảng 1/3). Thông thường sẽ cần độ dài VARCHAR lớn hơn, ví dụ VARCHAR(128), VARCHAR(256) hoặc dài hơn.

Bảng tổng kết so sánh:

| Tiêu chí so sánh | Kiểu VARCHAR (Khuyên dùng) | Kiểu INT/BIGINT (Không khuyên dùng) | Ghi chú / Giải thích |
| ---------------- | --------------------------------- | ---------------------------- | --------------------------------------------------------------------------- |
| **Tính tương thích định dạng** | ✔ Lưu được số 0 ở đầu, "+", "-", khoảng trắng | ✘ Tự động mất số 0 ở đầu, không lưu được ký tự | VARCHAR lưu nguyên văn mọi định dạng số điện thoại, INT/BIGINT chỉ hỗ trợ số thuần túy |
| **Tính toàn vẹn** | ✔ Không làm mất thông tin định dạng | ✘ Mất thông tin định dạng | Ví dụ "013800012345" lưu vào INT thành 13800012345, dấu "+" cũng không lưu được |
| **Tính phi đại số** | ✔ Phù hợp lưu trữ "định danh" | ✘ Chỉ phù hợp làm phép toán số học | Bản chất số điện thoại là chuỗi định danh, VARCHAR sát với thực tế sử dụng hơn |
| **Tính linh hoạt truy vấn** | ✔ Hỗ trợ `LIKE '138%'`... | ✘ Truy vấn prefix bất tiện hoặc hiệu năng kém | Dùng VARCHAR truy vấn theo đầu số/prefix hiệu quả, kiểu số phải chuyển đổi rườm rà |
| **Hỗ trợ lưu trữ mã hóa** | ✔ Lưu được ciphertext mã hóa (chữ cái, ký tự...) | ✘ Không thể lưu ciphertext | Chipertext sau mã hóa là chuỗi/nhị phân, chỉ VARCHAR, TEXT, BLOB mới tương thích |
| **Khuyến nghị độ dài** | 15~20 (chưa mã hóa), mã hóa tùy tình huống | Không có ý nghĩa | Chưa mã hóa VARCHAR(15~20) thông dụng, sau mã hóa phụ thuộc thuật toán và encoding |

## Kiến trúc cơ bản của MySQL

> Đề xuất phối hợp đọc bài viết [Quy trình thực thi câu lệnh SQL trong MySQL](./how-sql-executed-in-mysql.md) để hiểu kiến trúc cơ bản của MySQL. Ngoài ra, "Quy trình thực thi một câu lệnh SQL trong MySQL" cũng là câu hỏi phỏng vấn rất thường gặp.

Dưới đây là sơ đồ kiến trúc tóm tắt của MySQL, từ sơ đồ bạn có thể thấy rõ một câu lệnh SQL từ client được thực thi bên trong MySQL như thế nào:

![](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

Từ sơ đồ trên có thể thấy, MySQL chủ yếu được cấu thành từ các phần sau:

- **Connector (Bộ kết nối):** Xác thực danh tính và phân quyền (khi đăng nhập MySQL).
- **Query Cache (Bộ nhớ tạm truy vấn):** Khi thực thi câu lệnh query, sẽ kiểm tra cache trước (đã bị loại bỏ từ bản MySQL 8.0 do tính thực tế không cao).
- **Analyzer (Bộ phân tích):** Nếu không trúng cache, câu SQL sẽ đi qua Analyzer. Analyzer sẽ kiểm tra xem câu SQL muốn làm gì và syntax của câu SQL có đúng hay không.
- **Optimizer (Bộ tối ưu hóa):** Thực thi theo phương án mà MySQL cho là tối ưu nhất.
- **Executor (Bộ thực thi):** Thực thi câu lệnh, sau đó nhận dữ liệu trả về từ Storage Engine. Trước khi thực thi câu lệnh sẽ kiểm tra xem có quyền truy cập hay không, nếu không có quyền sẽ báo lỗi.
- **Pluggable Storage Engine (Động cơ lưu trữ dạng cắm rút):** Chủ yếu chịu trách nhiệm lưu trữ và đọc dữ liệu, sử dụng kiến trúc cắm rút, hỗ trợ nhiều Storage Engine như InnoDB, MyISAM, Memory... InnoDB là Storage Engine mặc định của MySQL, trong hầu hết các kịch bản sử dụng InnoDB là lựa chọn tốt nhất.

## Storage Engine trong MySQL

Cốt lõi của MySQL nằm ở Storage Engine, muốn học sâu về MySQL nhất định phải nghiên cứu sâu về Storage Engine của MySQL.

### MySQL hỗ trợ những Storage Engine nào? Mặc định dùng loại nào?

MySQL hỗ trợ nhiều Storage Engine, bạn có thể thông qua lệnh `SHOW ENGINES` để xem tất cả các Storage Engine mà MySQL hỗ trợ.

![Xem tất cả Storage Engine do MySQL cung cấp](https://oss.javaguide.cn/github/javaguide/mysql/image-20220510105408703.png)

Từ hình trên chúng ta có thể thấy, Storage Engine mặc định hiện tại của MySQL là InnoDB. Hơn nữa, trong tất cả các Storage Engine chỉ có InnoDB là Storage Engine hỗ trợ Transaction.

Phiên bản MySQL sử dụng ở đây là 8.x, giữa các phiên bản MySQL khác nhau có thể có sự khác biệt.

Trước MySQL 5.5.5, MyISAM là Storage Engine mặc định của MySQL. Từ phiên bản 5.5.5 trở đi, InnoDB là Storage Engine mặc định của MySQL.

Bạn có thể thông qua lệnh `SELECT VERSION();` để xem phiên bản MySQL của mình.

```bash
mysql> SELECT VERSION();
+-----------+
| VERSION() |
+-----------+
| 8.0.27    |
+-----------+
1 row in set (0.00 sec)
```

Bạn cũng có thể thông qua lệnh `SHOW VARIABLES LIKE '%storage_engine%'` để xem trực tiếp Storage Engine mặc định hiện tại của MySQL.

```bash
mysql> SHOW VARIABLES  LIKE '%storage_engine%';
+---------------------------------+-----------+
| Variable_name                   | Value     |
+---------------------------------+-----------+
| default_storage_engine          | InnoDB    |
| default_tmp_storage_engine      | InnoDB    |
| disabled_storage_engines        |           |
| internal_tmp_mem_storage_engine | TempTable |
+---------------------------------+-----------+
4 rows in set (0.00 sec)
```

Nếu bạn muốn tìm hiểu sâu về từng Storage Engine và sự khác biệt giữa chúng, khuyến nghị đọc tài liệu chính thức tương ứng của MySQL:

- Giới thiệu chi tiết Storage Engine InnoDB: <https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html> .
- Giới thiệu chi tiết các Storage Engine khác: <https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html> .

![](https://oss.javaguide.cn/github/javaguide/mysql/image-20220510155143458.png)

### Bạn có hiểu về kiến trúc Storage Engine của MySQL không?

Storage Engine của MySQL sử dụng **kiến trúc cắm rút (Pluggable Architecture)**, hỗ trợ nhiều Storage Engine, thậm chí chúng ta có thể thiết lập các Storage Engine khác nhau cho các bảng database khác nhau để thích ứng với nhu cầu của các kịch bản khác nhau. **Storage Engine là dựa trên bảng (table), chứ không phải dựa trên cơ sở dữ liệu (database).**

Sơ đồ dưới đây thể hiện kiến trúc MySQL với Storage Engine có thể cắm rút:

![MySQL architecture diagram showing connectors, interfaces, pluggable storage engines, the file system with files and logs.](https://oss.javaguide.cn/github/javaguide/mysql/mysql-architecture.png)

Bạn cũng có thể dựa theo interface chuẩn định nghĩa bởi MySQL để viết một Storage Engine của riêng mình. Các Storage Engine không do chính thức cung cấp này được gọi là Storage Engine bên thứ ba. InnoDB phổ biến nhất hiện nay ban đầu cũng là một Storage Engine bên thứ ba, sau này vì quá xuất sắc nên đã được Oracle trực tiếp thâu tóm.

Tài liệu chính thức của MySQL cũng có giới thiệu cách viết một Storage Engine tùy chỉnh: <https://dev.mysql.com/doc/internals/en/custom-engine.html> .

### ⭐️Sự khác biệt giữa MyISAM và InnoDB là gì?

Trước MySQL 5.5, MyISAM là Storage Engine mặc định của MySQL, từng rất hoàng kim.

Mặc dù hiệu năng của MyISAM khá tốt, các tính năng cũng không tệ (như Full-text Index, nén, hàm không gian...). Nhưng MyISAM không hỗ trợ Transaction và Row-level Lock, nhược điểm lớn nhất là sau khi crash không thể recovery an toàn.

Từ phiên bản MySQL 5.5 trở đi, InnoDB là Storage Engine mặc định của MySQL.

Đi vào trọng tâm! Dưới đây chúng ta hãy so sánh đơn giản giữa hai loại:

**1. Có hỗ trợ Row-level Lock hay không**

MyISAM chỉ có Table-level Lock (khóa cấp bảng), trong khi InnoDB hỗ trợ Row-level Lock (khóa cấp hàng) và Table-level Lock, mặc định là Row-level Lock.

Có nghĩa là, MyISAM hễ khóa là khóa cả bảng, điều này trong tình huống ghi đồng thời (concurrent write) là cực kỳ kém hiệu quả! Đây cũng là lý do tại sao InnoDB có hiệu năng vượt trội hơn hẳn khi ghi đồng thời!

**2. Có hỗ trợ Transaction hay không**

MyISAM không cung cấp hỗ trợ Transaction.

InnoDB cung cấp hỗ trợ Transaction, thực thi 4 mức độ cô lập chuẩn SQL, có khả năng commit và rollback transaction. Ngoài ra, mức độ cô lập REPEATABLE-READ mặc định của InnoDB có thể giải quyết vấn đề Phantom Read (dựa trên MVCC và Next-Key Lock).

Về giới thiệu chi tiết Transaction trong MySQL, có thể xem bài viết: [Chi tiết về mức độ cô lập Transaction trong MySQL](./transaction-isolation-level.md).

**3. Có hỗ trợ Foreign Key hay không**

MyISAM không hỗ trợ, còn InnoDB hỗ trợ Foreign Key.

Foreign Key rất có ích cho việc duy trì tính nhất quán của dữ liệu, nhưng gây tổn hại nhất định đến hiệu năng. Do đó, thông thường trong các dự án thực tế, chúng ta không khuyến nghị sử dụng Foreign Key ở tầng database, chỉ cần ràng buộc ở code nghiệp vụ là được!

Cẩm nang phát triển Java của Alibaba cũng quy định rõ ràng cấm sử dụng Foreign Key.

![](https://oss.javaguide.cn/github/javaguide/mysql/image-20220510090309427.png)

Tóm lại: Thông thường không khuyến nghị dùng Foreign Key ở tầng database, tầng ứng dụng có thể xử lý được.

**4. Có hỗ trợ khôi phục an toàn sau sự cố (Crash Recovery) hay không**

MyISAM không hỗ trợ, còn InnoDB hỗ trợ.

Database sử dụng InnoDB sau khi gặp sự cố bất ngờ (crash) và khởi động lại sẽ đảm bảo khôi phục database về trạng thái trước khi crash. Quy trình khôi phục này phụ thuộc vào `redo log`.

**5. Có hỗ trợ MVCC hay không**

MyISAM không hỗ trợ, còn InnoDB hỗ trợ.

MVCC có thể xem là một bước nâng cấp của Row-level Lock, giúp giảm thiểu thao tác khóa, nâng cao hiệu năng.

**6. Thực thi Index khác nhau**

Mặc dù cả MyISAM và InnoDB đều sử dụng B+Tree làm cấu trúc dữ liệu Index, nhưng cách thực thi của hai bên khác nhau.

Trong InnoDB, file dữ liệu chính là file Index. So với MyISAM (file Index và file dữ liệu tách rời), file dữ liệu bảng của InnoDB bản thân nó được tổ chức theo cấu trúc Index B+Tree, vùng data của node lá lưu trữ toàn bộ bản ghi dữ liệu hoàn chỉnh.

Chi tiết khác biệt khuyến nghị xem bài viết: [Chi tiết về MySQL Index](./mysql-index.md).

**7. Chênh lệch về hiệu năng**

Hiệu năng của InnoDB mạnh mẽ hơn MyISAM, dù ở chế độ hỗn hợp đọc ghi hay chế độ chỉ đọc, theo sự gia tăng số nhân CPU, khả năng đọc ghi của InnoDB tăng trưởng theo đường thẳng. MyISAM do đọc ghi không thể đồng thời nên khả năng xử lý không liên quan đến số nhân CPU.

![So sánh hiệu năng InnoDB và MyISAM](https://oss.javaguide.cn/github/javaguide/mysql/innodb-myisam-performance-comparison.png)

**8. Chiến lược và cơ chế cache dữ liệu khác nhau**

InnoDB sử dụng Buffer Pool để cache trang dữ liệu (data page) và trang Index (index page), MyISAM sử dụng Key Cache chỉ cache trang Index mà không cache trang dữ liệu.

**Tóm lại**:

- InnoDB hỗ trợ Row-level Lock, MyISAM không hỗ trợ (chỉ hỗ trợ Table-level Lock).
- MyISAM không cung cấp Transaction. InnoDB cung cấp Transaction với 4 mức độ cô lập.
- MyISAM không hỗ trợ Foreign Key, InnoDB hỗ trợ.
- MyISAM không hỗ trợ MVCC, InnoDB hỗ trợ.
- Cách thực thi Index B+Tree của MyISAM và InnoDB khác nhau.
- MyISAM không hỗ trợ Crash Recovery an toàn, InnoDB hỗ trợ.
- Hiệu năng của InnoDB mạnh hơn MyISAM.

Sơ đồ so sánh các Storage Engine phổ biến của MySQL:

![So sánh các Storage Engine phổ biến của MySQL](https://oss.javaguide.cn/github/javaguide/mysql/comparison-of-common-mysql-storage-engines.png)

### Lựa chọn MyISAM hay InnoDB?

Hầu hết thời gian chúng ta đều sử dụng Storage Engine InnoDB. Trong một số kịch bản đọc chiếm đa số (read-intensive), dùng MyISAM cũng phù hợp nếu dự án không bận tâm đến việc thiếu Transaction hay Crash Recovery.

Trích sách "High Performance MySQL":

> Đừng dễ dàng tin vào những kinh nghiệm truyền miệng như "MyISAM nhanh hơn InnoDB", kết luận này thường không tuyệt đối. Trong nhiều kịch bản, tốc độ của InnoDB có thể bỏ xa MyISAM, đặc biệt là khi dùng Clustered Index hoặc dữ liệu cần truy cập đều nằm trong bộ nhớ.

Do đó, đối với các hệ thống nghiệp vụ phát triển hàng ngày, gần như không có lý do gì để dùng MyISAM nữa, hãy trung thành sử dụng InnoDB mặc định!

## ⭐️MySQL Index

Các câu hỏi liên quan đến MySQL Index rất nhiều và cực kỳ quan trọng, bài viết chi tiết: [Chi tiết về MySQL Index](./mysql-index.md).

### Index là gì?

**Index là một cấu trúc dữ liệu dùng để truy vấn và tìm kiếm dữ liệu nhanh chóng, bản chất có thể xem là một cấu trúc dữ liệu đã được sắp xếp.**

Tác dụng của Index tương tự như mục lục của một cuốn sách. Ví dụ: khi tra từ điển, nếu không có mục lục, ta chỉ có thể lật từng trang một để tìm từ cần tra, tốc độ rất chậm; nếu có mục lục, ta chỉ cần vào mục lục tìm vị trí của từ rồi lật trực tiếp đến trang đó.

Cấu trúc dữ liệu bên dưới của Index có nhiều loại, phổ biến như: B-Tree, B+Tree, Hash, Red-Black Tree. Trong MySQL, dù là InnoDB hay MyISAM đều sử dụng B+Tree làm cấu trúc dữ liệu Index.

**Ưu điểm của Index:**

1. **Tốc độ truy vấn tăng vọt (mục đích chính)**: Thông qua Index, database có thể **giảm đáng kể lượng dữ liệu cần quét**, định vị trực tiếp đến bản ghi thỏa mãn điều kiện, từ đó đẩy nhanh tốc độ truy xuất dữ liệu, giảm số lần I/O đĩa.
2. **Đảm bảo tính duy nhất của dữ liệu**: Thông qua việc tạo **Unique Index**, có thể đảm bảo giá trị của một cột (hoặc tổ hợp nhiều cột) trong bảng là duy nhất, như ID người dùng, email... Primary Key bản thân nó cũng là một Unique Index.
3. **Đẩy nhanh sắp xếp và nhóm**: Nếu cột liên quan trong mệnh đề ORDER BY hoặc GROUP BY của query có tạo Index, database thường có thể tận dụng trực tiếp đặc tính đã sắp xếp của Index, tránh thao tác sắp xếp bổ sung, từ đó nâng cao hiệu năng.

**Nhược điểm của Index:**

1. **Tốn thời gian tạo và bảo trì**: Tạo Index bản thân nó cần thời gian, đặc biệt là thao tác trên bảng lớn. Quan trọng hơn, khi thực hiện các thao tác **thêm, xóa, sửa (thao tác DML)** dữ liệu trong bảng, không những phải thao tác bản thân dữ liệu mà Index liên quan cũng phải cập nhật và bảo trì động, việc này sẽ **làm giảm hiệu suất thực thi của các thao tác DML này**.
2. **Chiếm dụng không gian lưu trữ**: Index bản chất cũng là cấu trúc dữ liệu, cần lưu trữ dưới dạng file vật lý (hoặc cấu trúc bộ nhớ), do đó sẽ **chiếm thêm một khoảng dung lượng đĩa**. Index càng nhiều, càng lớn thì dung lượng chiếm dụng càng nhiều.
3. **Có thể bị dùng sai hoặc vô hiệu hóa**: Nếu thiết kế Index không hợp lý, hoặc viết câu lệnh query không tốt, Optimizer của database có thể sẽ không chọn sử dụng Index (hoặc chọn sai Index), dẫn đến hiệu năng bị giảm sút.

**Vậy dùng Index có chắc chắn nâng cao hiệu năng truy vấn không?**

**Không nhất thiết.** Hầu hết trường hợp, dùng Index hợp lý quả thực nhanh hơn nhiều so với Full Table Scan (quét toàn bảng). Nhưng cũng có ngoại lệ:

- **Lượng dữ liệu quá nhỏ**: Nếu dữ liệu trong bảng rất ít (ví dụ chỉ vài trăm hàng), Full Table Scan có thể nhanh hơn việc tìm kiếm qua Index, vì bản thân việc đi qua Index cũng có overhead.
- **Tỷ lệ kết quả query quá lớn**: Nếu dữ liệu cần query chiếm phần lớn cả bảng (ví dụ trên 20%-30%), Optimizer có thể cho rằng Full Table Scan kinh tế hơn, vì chi phí Index Lookup (回表) nhiều lần (I/O ngẫu nhiên) có thể cao hơn một lần Full Table Scan tuần tự.
- **Bảo trì Index không tốt hoặc thông tin thống kê bị lỗi thời**: Dẫn đến Optimizer đưa ra phán đoán sai.

### Tại sao Index lại nhanh?

Lý do cốt lõi khiến Index nhanh là vì nó **giảm đáng kể số lần I/O đĩa**.

Bản chất của nó là một **cấu trúc dữ liệu đã được sắp xếp**, giống như mục lục sách, giúp ta không phải lật từng trang (Full Table Scan).

Trong MySQL, cấu trúc dữ liệu này là **B+Tree**. Cấu trúc B+Tree chủ yếu tối ưu hóa ở 2 phương diện:

1. Đặc điểm của B+Tree là "lùn và béo", một bảng hàng chục triệu dữ liệu, chiều cao cây Index có thể chỉ từ 3-4 tầng. Điều này có nghĩa là tối đa chỉ cần **3-4 lần I/O đĩa** là có thể định vị chính xác dữ liệu muốn tìm, trong khi Full Table Scan có thể cần hàng ngàn hàng vạn lần.
2. Các node lá của B+Tree **được nối với nhau bằng danh sách liên kết (linked list)**. Sau khi tìm được điểm đầu, có thể men theo linked list **đọc tuần tự (sequential read)** tiếp, điều này rất thân thiện với đĩa cứng và còn kích hoạt tính năng read-ahead (đọc trước).

### Cấu trúc dữ liệu bên dưới của MySQL Index là gì?

Trong MySQL, cả Storage Engine MyISAM và InnoDB đều sử dụng B+Tree làm cấu trúc dữ liệu Index, bài viết chi tiết: [Chi tiết về MySQL Index](https://javaguide.cn/database/mysql/mysql-index.html).

### Tại sao InnoDB không sử dụng Hash làm cấu trúc dữ liệu cho Index?

Hash Index dựa trên Hash Table. Ưu điểm của nó là khi thực hiện **truy vấn bằng (equality query) chính xác**, về mặt lý thuyết độ phức tạp thời gian là **O(1)**, tốc độ cực nhanh, ví dụ `WHERE id = 123`.

Tuy nhiên, nó có một số nhược điểm chí mạng đối với cơ sở dữ liệu tổng quát:

1. **Không hỗ trợ truy vấn phạm vi (Range Query):** Đây là nguyên nhân chính. Đặc điểm của hàm Hash là nó ánh xạ các giá trị đầu vào liền kề (như `id=100` và `id=101`) đến các vị trí hoàn toàn không liền kề trong Hash Table. Sự phá vỡ thứ tự này khiến chúng ta không thể xử lý các truy vấn phạm vi như `WHERE age > 30` hay `BETWEEN 100 AND 200`. Để hoàn thành truy vấn này, Hash Index chỉ có thể thoái hóa thành Full Table Scan.
2. **Không hỗ trợ sắp xếp:** Tương tự, vì giá trị Hash là vô trật tự, nên không thể tận dụng Hash Index để tối ưu hóa mệnh đề `ORDER BY`.
3. **Không hỗ trợ truy vấn một phần cột Index:** Đối với Composite Index, ví dụ `(col1, col2)`, Hash Index bắt buộc phải dùng tất cả các cột Index để query, nó không thể dùng riêng `col1` để đẩy nhanh truy vấn.
4. **Vấn đề xung đột Hash (Hash Collision):** Khi các key khác nhau tạo ra cùng giá trị Hash, cần thêm linked list hoặc open addressing để giải quyết, làm giảm hiệu năng.

Xét thấy truy vấn phạm vi và sắp xếp là các thao tác cực kỳ phổ biến trong database, một cấu trúc Index không hỗ trợ các tính năng này rõ ràng không thể làm kiểu Index mặc định, tổng quát được.

### Tại sao InnoDB không sử dụng B-Tree làm cấu trúc dữ liệu cho Index?

B-Tree và B+Tree đều là các cây tìm kiếm cân bằng đa đường (multi-way balanced search tree) xuất sắc, rất phù hợp cho lưu trữ đĩa vì chúng rất "lùn và béo", tối đa hóa việc tận dụng mỗi lần I/O đĩa.

Tuy nhiên B+Tree là phiên bản nâng cấp của B-Tree, nó thực hiện một số tối ưu hóa then chốt cho kịch bản database:

1. **Hiệu suất I/O cao hơn:** Trong B+Tree, chỉ các node lá mới lưu trữ dữ liệu (hoặc con trỏ dữ liệu), còn các node không phải lá chỉ lưu key của Index. Vì các node không phải lá không lưu dữ liệu nên chúng có thể chứa nhiều key Index hơn. Điều này có nghĩa là "fan-out" của B+Tree lớn hơn, với cùng lượng dữ liệu, B+Tree thông thường sẽ lùn hơn B-Tree, đồng nghĩa số lần I/O đĩa cần thiết để tìm dữ liệu ít hơn.
2. **Hiệu năng truy vấn ổn định hơn:** Trong B+Tree, bất kỳ truy vấn nào cũng bắt buộc phải đi từ node gốc đến node lá mới tìm thấy dữ liệu, nên độ dài đường đi truy vấn là cố định. Còn trong B-Tree, nếu may mắn có thể tìm thấy dữ liệu ngay ở node không phải lá, nhưng nếu không may mắn vẫn phải đi tới node lá, dẫn đến hiệu năng truy vấn không ổn định.
3. **Cực kỳ thân thiện với truy vấn phạm vi:** Đây là ưu thế cốt lõi nhất của B+Tree. Tất cả các node lá của nó được nối với nhau qua một doubly linked list. Khi chúng ta thực hiện truy vấn phạm vi (như `WHERE id > 100`), chỉ cần thông qua cấu trúc cây tìm thấy node lá `id=100`, sau đó có thể dọc theo linked list quét tuần tự về sau mà không cần quay ngược lên các node tầng trên. Điều này làm hiệu suất truy vấn phạm vi tăng lên rất nhiều.

### Covering Index là gì?

Nếu một Index chứa (hoặc bao phủ - cover) giá trị của tất cả các field cần query, ta gọi đó là **Covering Index (索引覆盖 / 覆盖索引)**.

Trong Storage Engine InnoDB, node lá của Secondary Index chứa giá trị của Primary Key. Điều này có nghĩa là khi dùng Secondary Index để query, database sẽ tìm thấy giá trị Primary Key tương ứng trước, sau đó thông qua Primary Key Index để định vị và truy xuất dữ liệu hàng hoàn chỉnh. Quy trình này được gọi là "Index Lookup" (回表 - huíbǐao).

**Covering Index chính là khi các field cần query vừa hay chính là các field của Index, vậy thì dựa vào Index đó là có thể查 ra dữ liệu trực tiếp mà không cần Index Lookup (回表).**

### Giải thích Composite Index trong MySQL và Quy tắc Tiền tố Trái nhất (Leftmost Prefix Rule)

Tạo Index bằng nhiều field trong bảng gọi là **Composite Index (联合索引 / 组合索引)**.

Tạo Composite Index với 2 field `score` và `name`:

```sql
ALTER TABLE `cus_order` ADD INDEX id_score_name(score, name);
```

Quy tắc Tiền tố Trái nhất (Leftmost Prefix Matching Rule) đề cập đến việc khi sử dụng Composite Index, MySQL sẽ dựa theo thứ tự các field trong Index, từ trái sang phải khớp lần lượt các field trong điều kiện query. Nếu điều kiện query khớp với field bên trái nhất trong Index, MySQL sẽ dùng Index để lọc dữ liệu.

Quy tắc Tiền tố Trái nhất sẽ khớp liên tục sang phải cho đến khi gặp truy vấn phạm vi (như `>`, `<`) mới dừng lại. Đối với các truy vấn phạm vi `>=`, `<=`, `BETWEEN` và LIKE khớp tiền tố thì không dừng khớp.

Giả sử có một Composite Index `(column1, column2, column3)`, tất cả tiền tố từ trái sang phải là `(column1)`, `(column1, column2)`, `(column1, column2, column3)` (tạo 1 Composite Index tương đương tạo 3 Index), tất cả các query chứa các cột này đều sẽ đi qua Index chứ không Full Table Scan.

Khi dùng Composite Index, chúng ta có thể đặt field có độ phân biệt (selectivity) cao ở ngoài cùng bên trái để lọc được nhiều dữ liệu hơn.

Ví dụ hiệu quả của Quy tắc Tiền tố Trái nhất:

1. Tạo bảng `student` gồm 3 field `id`, `name`, `class`:

```sql
CREATE TABLE `student` (
  `id` int NOT NULL,
  `name` varchar(100) DEFAULT NULL,
  `class` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `name_class_idx` (`name`,`class`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

2. Thử nghiệm 3 câu SQL:

![](https://oss.javaguide.cn/github/javaguide/database/mysql/leftmost-prefix-matching-rule.png)

```sql
# Có thể trúng Index
SELECT * FROM student WHERE name = 'Anne Henry';
EXPLAIN SELECT * FROM student WHERE name = 'Anne Henry' AND class = 'lIrm08RYVk';
# Không thể trúng Index
SELECT * FROM student WHERE class = 'lIrm08RYVk';
```

Câu hỏi phỏng vấn phổ biến: Nếu có `Composite Index (a, b, c)`, query `a=1 AND c=1` có đi qua Index không? `c=1` thì sao? `b=1 AND c=1` thì sao? `b=1 AND a=1 AND c=1` thì sao?

1. Query `a=1 AND c=1`: Theo quy tắc tiền tố trái nhất, query có thể dùng phần tiền tố của Index. Do đó, query chỉ dùng Index trên `a=1`, sau đó thực hiện lọc `c=1` trên kết quả.
2. Query `c=1`: Do query không chứa cột bên trái nhất `a`, toàn bộ Index không được sử dụng.
3. Query `b=1 AND c=1`: Tương tự trường hợp 2, toàn bộ Index không được sử dụng.
4. Query `b=1 AND a=1 AND c=1`: Query này dùng được Index. Khi Optimizer phân tích SQL, đối với Composite Index, nó sẽ tự động sắp xếp lại thứ tự điều kiện query thành `a=1 AND b=1 AND c=1`.

MySQL 8.0.13 giới thiệu Index Skip Scan (ISS), có thể nâng cao hiệu quả truy vấn trong một số kịch bản nhất định khi không tuân thủ quy tắc tiền tố trái nhất.

### SELECT * có dẫn đến Index bị vô hiệu hóa không?

`SELECT *` không trực tiếp làm Index bị vô hiệu hóa (nếu không đi qua Index phần lớn là do phạm vi query WHERE quá rộng), nhưng nó có thể mang lại một số vấn đề hiệu năng khác như lãng phí truyền tải mạng, không thể dùng Covering Index.

### Những field nào phù hợp để tạo Index?

- **Field không phải NULL**: Dữ liệu field Index nên cố gắng không phải NULL, vì đối với field là NULL, database khó tối ưu hóa hơn. Nếu field hay được query nhưng không tránh được NULL, khuyến nghị dùng các giá trị ngắn có ngữ nghĩa rõ ràng như 0, 1, true, false để thay thế.
- **Field được truy vấn tần suất cao**: Field ta tạo Index nên là field có thao tác query cực kỳ thường xuyên.
- **Field làm điều kiện query**: Field dùng làm điều kiện WHERE nên được cân nhắc tạo Index.
- **Field thường xuyên cần sắp xếp**: Index đã được sắp xếp sẵn, query có thể lợi dụng tính năng sắp xếp của Index để tăng tốc.
- **Field thường xuyên dùng để JOIN**: Các field dùng để JOIN thường là cột Foreign Key, việc tạo Index giúp nâng cao hiệu quả JOIN đa bảng.

### Nguyên nhân khiến Index bị vô hiệu hóa (Index Invalidation)?

1. Tạo Composite Index nhưng điều kiện query không tuân thủ Quy tắc Tiền tố Trái nhất;
2. Thực hiện tính toán, hàm, chuyển đổi kiểu dữ liệu trên cột Index;
3. Truy vấn LIKE bắt đầu bằng `%` như `LIKE '%abc'`;
4. Trong điều kiện query sử dụng `OR`, mà trước hoặc sau `OR` có một cột không có Index;
5. Phạm vi giá trị của `IN` quá lớn dẫn đến bỏ Index, chuyển sang Full Table Scan (kịch bản vô hiệu hóa của `NOT IN` tương tự `IN`);
6. Xảy ra [Chuyển đổi ngầm định (Implicit Conversion)](https://javaguide.cn/database/mysql/index-invalidation-caused-by-implicit-conversion.html);

## MySQL Query Cache

Query Cache trong MySQL là cache kết quả truy vấn. Khi thực thi câu lệnh query, sẽ kiểm tra cache trước, nếu có kết quả sẽ trả về trực tiếp.

Trong `my.cnf` thêm cấu hình sau và khởi động lại MySQL để bật Query Cache:

```properties
query_cache_type=1
query_cache_size=600000
```

Lệnh bật Query Cache trong MySQL:

```properties
set global  query_cache_type=1;
set global  query_cache_size=600000;
```

Query Cache đòi hỏi điều kiện rất nghiêm ngặt, bất kỳ khác biệt nhỏ nào cũng khiến cache miss.

**Các trường hợp Query Cache không trúng:**

1. Bất kỳ sự khác biệt ký tự nào giữa 2 query đều làm cache miss.
2. Nếu query chứa hàm tự định nghĩa, stored function, user variable, temporary table, system table trong sys database thì kết quả không được cache.
3. Khi dữ liệu hoặc cấu trúc bảng thay đổi, tất cả cache liên quan đến bảng đó đều bị hủy.

Từ MySQL 5.6, Query Cache mặc định bị tắt. Từ MySQL 8.0, Query Cache đã bị loại bỏ hoàn toàn (tham khảo: [MySQL 8.0: Retiring Support for the Query Cache](https://dev.mysql.com/blog-archive/mysql-8-0-retiring-support-for-the-query-cache/)).

![MySQL 8.0: Retiring Support for the Query Cache](https://oss.javaguide.cn/github/javaguide/mysql/mysql8.0-retiring-support-for-the-query-cache.png)

## ⭐️MySQL Log

Đáp án của phần này có thể xem thêm trong tài liệu khóa học.

## ⭐️MySQL Transaction

### Transaction là gì?

Thử tưởng tượng kịch bản chèn nhiều dữ liệu liên quan vào database, quá trình này có thể gặp sự cố:
- Database đột ngột bị sập giữa chừng.
- Client mất kết nối do sự cố mạng.
- Nhiều thread cùng ghi vào database đè lên thay đổi của nhau.

Để đảm bảo tính nhất quán của dữ liệu, **Transaction (Giao dịch/Giao thức)** là cơ chế hàng đầu được trừu tượng hóa để giải quyết các vấn đề này.

**Transaction là gì?** Tóm lại, **Transaction là một tập hợp các thao tác về mặt logic, hoặc là cùng thực thi thành công, hoặc là cùng không thực thi.**

Ví dụ điển hình nhất của Transaction là chuyển tiền. Chuyển 1000 VNĐ từ tài khoản A sang B gồm 2 thao tác:
1. Giảm 1000 VNĐ ở tài khoản A.
2. Tăng 1000 VNĐ ở tài khoản B.

Transaction coi 2 thao tác này là một thể thống nhất logic, cùng thành công hoặc cùng thất bại.

![Sơ đồ Transaction](https://oss.javaguide.cn/github/javaguide/mysql/%E4%BA%8B%E5%8A%A1%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

### Database Transaction là gì?

Khi nói về Transaction mà không chỉ định **Distributed Transaction** (Transaction phân tán), thường ta đang nói về **Database Transaction**.

Database Transaction giúp nhiều thao tác SQL cấu thành một thể thống nhất logic: **hoặc là tất cả đều thành công, hoặc là tất cả đều không thực thi**.

```sql
# Bắt đầu một transaction
START TRANSACTION;
# Các câu lệnh SQL
SQL1,SQL2...
## Commit transaction
COMMIT;
```

![Sơ đồ Database Transaction](https://oss.javaguide.cn/github/javaguide/mysql/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

Các cơ sở dữ liệu quan hệ (MySQL, SQL Server, Oracle...) có 4 tính chất **ACID**:

![ACID](https://oss.javaguide.cn/github/javaguide/mysql/ACID.png)

1. **Tính nguyên tố (Atomicity)**: Transaction là đơn vị thực thi nhỏ nhất, không thể chia cắt. Đảm bảo các hành động hoặc là hoàn thành tất cả, hoặc không có tác dụng gì.
2. **Tính nhất quán (Consistency)**: Trước và sau khi thực thi Transaction, dữ liệu giữ được tính nhất quán.
3. **Tính cô lập (Isolation)**: Khi truy cập đồng thời, Transaction của một người dùng không bị can thiệp bởi Transaction khác.
4. **Tính bền vững (Durability)**: Sau khi Transaction được commit, thay đổi dữ liệu là vĩnh viễn, kể cả khi database gặp sự cố cũng không ảnh hưởng.

🌈 Bổ sung: **Chỉ khi đảm bảo tính Durability, Atomicity và Isolation thì Consistency mới được đảm bảo. Tức A, I, D là phương tiện, C là mục đích!**

![AID->C](https://oss.javaguide.cn/github/javaguide/mysql/AID-%3EC.png)

Trích sách DDIA ("Designing Data-Intensive Applications"):

> Atomicity, isolation, and durability are properties of the database, whereas consistency (in the ACID sense) is a property of the application. The application may rely on the database's atomicity and isolation properties in order to achieve consistency, but it's not up to the database alone.

![](https://oss.javaguide.cn/github/javaguide/books/ddia.png)

### Các vấn đề do Transaction đồng thời (Concurrent Transactions) gây ra?

#### Dirty Read (Đọc bẩn)

Một Transaction đọc dữ liệu và sửa đổi dữ liệu đó, sửa đổi này hiển thị với Transaction khác dù chưa commit. Transaction thứ hai đọc dữ liệu chưa commit này, nhưng Transaction đầu tiên đột ngột rollback. Dữ liệu mà Transaction thứ hai đọc được chính là dữ liệu bẩn (Dirty Data).

![Dirty Read](https://oss.javaguide.cn/github/javaguide/database/mysql/concurrency-consistency-issues-dirty-reading.png)

#### Lost Update (Mất cập nhật / 丢失修改)

Transaction 1 đọc A=20, Transaction 2 cũng đọc A=20. Transaction 1 sửa A=A-1, Transaction 2 sau đó cũng sửa A=A-1. Kết quả cuối cùng A=19, sửa đổi của Transaction 1 bị mất.

![Lost Update](https://oss.javaguide.cn/github/javaguide/database/mysql/concurrency-consistency-issues-missing-modifications.png)

#### Non-repeatable Read (Đọc không lặp lại)

Trong cùng một Transaction, đọc cùng một dữ liệu nhiều lần cho kết quả khác nhau do Transaction khác sửa đổi hoặc xóa dữ liệu đó giữa các lần đọc.

![Non-repeatable Read](https://oss.javaguide.cn/github/javaguide/database/mysql/concurrency-consistency-issues-unrepeatable-read.png)

#### Phantom Read (Đọc ảo)

Transaction 1 thực hiện truy vấn phạm vi, Transaction 2 insert thêm dữ liệu mới vào phạm vi đó. Transaction 1 truy vấn lại phạm vi đó thấy xuất hiện các bản ghi mới mà trước đó không có.

![Phantom Read](https://oss.javaguide.cn/github/javaguide/database/mysql/concurrency-consistency-issues-phantom-read.png)

### Sự khác biệt giữa Non-repeatable Read và Phantom Read là gì?

- Non-repeatable Read: Trong cùng Transaction, cùng một bản ghi bị sửa hoặc xóa bởi Transaction khác.
- Phantom Read: Trong cùng Transaction, cùng một truy vấn phạm vi xuất hiện thêm bản ghi mới chèn vào hoặc biến mất.

Để tránh Phantom Read khi insert, ngoài Record Lock cần phụ thuộc thêm Gap Lock (kết hợp thành Next-Key Lock).

### Các phương thức kiểm soát Transaction đồng thời?

Kiểm soát Transaction đồng thời trong MySQL có 2 phương thức: **Lock** và **MVCC**.
- Lock (Khóa) là cơ chế kiểm soát bi quan (Pessimistic).
- MVCC (Multiversion Concurrency Control) là cơ chế kiểm soát quan (Optimistic).

Các loại Lock:
- **Shared Lock (S Lock / 读锁)**: Khóa chia sẻ.
- **Exclusive Lock (X Lock / 写锁)**: Khóa độc chiếm.

Về khóa theo hạt (granularity): Table-level Lock và Row-level Lock.

**MVCC** dựa trên 3 yếu tố: **Field ẩn (Hidden columns), Read View, Undo Log**.

Bài viết chi tiết: [Cách Storage Engine InnoDB thực thi MVCC](./innodb-implementation-of-mvcc.md).

### Các mức độ cô lập Transaction chuẩn SQL?

- **READ UNCOMMITTED (Đọc chưa commit)**: Cho phép đọc dữ liệu chưa commit, có thể bị Dirty Read, Non-repeatable Read, Phantom Read.
- **READ COMMITTED (Đọc đã commit)**: Cho phép đọc dữ liệu đã commit, chống Dirty Read, vẫn bị Non-repeatable Read, Phantom Read.
- **REPEATABLE READ (Có thể đọc lặp lại)**: Các lần đọc cùng field trong Transaction cho kết quả như nhau, chống Dirty Read và Non-repeatable Read. InnoDB mặc định dùng mức này và giải quyết phần lớn Phantom Read nhờ MVCC + Next-Key Lock.
- **SERIALIZABLE (Chuỗi hóa / Tuần tự hóa)**: Mức cô lập cao nhất, các Transaction thực thi tuần tự.

| Mức độ cô lập | Dirty Read | Non-Repeatable Read | Phantom Read |
| ---------------- | ----------------- | -------------------------------- | ---------------------- |
| READ UNCOMMITTED | √ | √ | √ |
| READ COMMITTED | × | √ | √ |
| REPEATABLE READ | × | × | √ (Chuẩn) / ≈× (InnoDB) |
| SERIALIZABLE | × | × | × |

### Mức độ cô lập mặc định của MySQL là gì?

MySQL InnoDB mặc định là **REPEATABLE READ**.

Xem bằng lệnh:
- MySQL < 8.0: `SELECT @@tx_isolation;`
- MySQL >= 8.0: `SELECT @@transaction_isolation;`

```sql
mysql> SELECT @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

### Mức độ cô lập của MySQL có phải dựa hoàn toàn vào Lock không?

Kết hợp cả Lock và MVCC. SERIALIZABLE dựa trên Lock, READ COMMITTED và REPEATABLE READ dựa trên MVCC.

## MySQL Lock

### Table-level Lock và Row-level Lock khác nhau như thế nào?

MyISAM chỉ hỗ trợ Table-level Lock. InnoDB hỗ trợ cả Table-level Lock và Row-level Lock (mặc định Row-level Lock).

- **Table-level Lock:** Khóa toàn bộ bảng, chi phí thấp, không bị Deadlock, nhưng độ tranh chấp cao.
- **Row-level Lock:** Khóa trên các field có Index, độ tranh chấp thấp, hiệu năng cao, nhưng chi phí khóa cao hơn và có thể bị Deadlock.

### Lưu ý khi sử dụng Row-level Lock?

Row-level Lock của InnoDB là khóa trên Index. Nếu câu SQL `UPDATE`/`DELETE` không trúng Index hoặc bị vô hiệu hóa Index, MySQL sẽ quét toàn bộ bảng và khóa tất cả các hàng!

### InnoDB có những loại Row Lock nào?

- **Record Lock**: Khóa bản ghi đơn lẻ.
- **Gap Lock**: Khóa khoảng giữa các bản ghi (không bao gồm bản ghi).
- **Next-Key Lock**: Record Lock + Gap Lock (khóa phạm vi bao gồm bản ghi), dùng để giải quyết Phantom Read.

Ở mức REPEATABLE READ, nếu query trên Unique Index hoặc Primary Key, Next-Key Lock sẽ được tối ưu hạ cấp xuống Record Lock.

### Shared Lock và Exclusive Lock?

- **Shared Lock (S Lock)**: Khóa đọc.
- **Exclusive Lock (X Lock)**: Khóa ghi.

| | S Lock | X Lock |
| :--- | :----- | :--- |
| S Lock | Không xung đột | Xung đột |
| X Lock | Xung đột | Xung đột |

SQL chủ động thêm khóa:
```sql
# S Lock (MySQL 5.7 & 8.0)
SELECT ... LOCK IN SHARE MODE;
# S Lock (MySQL 8.0)
SELECT ... FOR SHARE;
# X Lock
SELECT ... FOR UPDATE;
```

### Intention Lock (Khóa ý định) có tác dụng gì?

Intention Lock là Table-level Lock giúp kiểm tra nhanh xem trong bảng có hàng nào đang bị khóa row hay không mà không cần duyệt từng hàng.

- **Intention Shared Lock (IS Lock)**: Ý định thêm S Lock.
- **Intention Exclusive Lock (IX Lock)**: Ý định thêm X Lock.

| | IS Lock | IX Lock |
| ----- | ----- | ----- |
| IS Lock | Tương thích | Tương thích |
| IX Lock | Tương thích | Tương thích |

| | IS Lock | IX Lock |
| ---- | ----- | ----- |
| S Lock | Tương thích | Xung đột |
| X Lock | Xung đột | Xung đột |

### Khác biệt giữa Snapshot Read và Current Read?

- **Snapshot Read (Đọc ảnh chụp / Consistent Non-locking Read)**: `SELECT` thông thường. Đọc phiên bản lịch sử qua MVCC mà không cần chờ giải phóng X Lock.
- **Current Read (Đọc hiện tại / Consistent Locking Read)**: `SELECT ... FOR UPDATE`, `SELECT ... LOCK IN SHARE MODE`, `INSERT`, `UPDATE`, `DELETE`. Đọc bản ghi mới nhất và chủ động thêm khóa.

### AUTO-INC Lock (Khóa tự tăng)?

Cấu hình `innodb_autoinc_lock_mode` (0: Traditional, 1: Consecutive, 2: Interleaved - mặc định từ MySQL 8.0).

## ⭐️Tối ưu hóa hiệu năng MySQL

### Có nên trực tiếp lưu file (như hình ảnh) vào MySQL không?

Không nên. Nên dùng Object Storage (OSS, MinIO, S3...) và chỉ lưu URL file trong database.

### MySQL lưu địa chỉ IP như thế nào?

Dùng `INET_ATON()` chuyển IP thành số nguyên unsigned int (4 byte) để lưu trữ và `INET_NTOA()` để đọc ra.

### Làm thế nào để phân tích hiệu năng SQL?

Dùng lệnh `EXPLAIN` để xem Execution Plan (Kế hoạch thực thi).

Cột trong EXPLAIN: `id`, `select_type`, `table`, `partitions`, `type`, `possible_keys`, `key`, `key_len`, `ref`, `rows`, `filtered`, `Extra`.

### Phân tách Đọc-Ghi (Read-Write Splitting) và Sharding (Phân kho phân bảng)?

Tham khảo: [Chi tiết Read-Write Splitting và Sharding](../../high-performance/read-and-write-separation-and-library-subtable.md).

### Tối ưu hóa Deep Pagination (Phân trang sâu)?

Tham khảo: [Tối ưu hóa Deep Pagination](../../high-performance/deep-pagination-optimization.md).

### Tách dữ liệu Nóng - Lạnh (Cold-Hot Data Separation)?

Tham khảo: [Chi tiết tách dữ liệu Nóng - Lạnh](../../high-performance/data-cold-hot-separation.md).

### Tổng kết quy trình Tối ưu hóa hiệu năng MySQL?

1. Định vị SQL chậm (Slow Query Log, Performance Schema, EXPLAIN).
2. Tối ưu hóa Index, cấu trúc bảng và câu lệnh SQL.
3. Kiến trúc nâng cao: Read-Write Splitting, Sharding, Cache (Redis), Cold-Hot Separation.
4. Cấu hình Connection Pool, phần cứng.

## Tài liệu tham khảo học tập MySQL

- [Sách tham khảo](../../books/database.md#mysql)
- Các series bài viết hướng dẫn MySQL.

<!-- @include: @article-footer.snippet.md -->
