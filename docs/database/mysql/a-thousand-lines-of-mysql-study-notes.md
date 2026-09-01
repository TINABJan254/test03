---
title: 一千行 MySQL 学习笔记
description: 一千行MySQL学习笔记精华总结，涵盖数据库操作、表管理、SQL语法、索引、视图、存储过程、触发器等核心知识点，适合快速查阅和复习。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL学习笔记,MySQL命令大全,SQL语法,数据库操作,表操作,索引,视图,存储过程,触发器
---

> Bài viết gốc: <https://shockerli.net/post/1000-line-mysql-note/>, JavaGuide đã định dạng lại và thêm mục lục.

Tổng kết rất tuyệt vời, khuyến nghị lưu lại để tra cứu khi cần.

### Thao tác cơ bản

```sql
/* Windows Service */
-- Khởi động MySQL
				net start mysql
-- Tạo Windows Service
				sc create mysql binPath= mysqld_bin_path(Chú ý: giữa dấu = và giá trị có khoảng trắng)
/* Kết nối và ngắt kết nối Server */
-- Kết nối MySQL
				mysql -h Địa_chỉ -P Cổng -u Tên_đăng_nhập -p Mật_khẩu
-- Hiển thị các thread đang chạy
				SHOW PROCESSLIST
-- Hiển thị thông tin biến hệ thống
				SHOW VARIABLES
```

### Thao tác Cơ sở dữ liệu

```sql
/* Thao tác Cơ sở dữ liệu */
-- Xem database hiện tại
    SELECT DATABASE();
-- Hiển thị thời gian hiện tại, user, phiên bản database
    SELECT now(), user(), version();
-- Tạo database
    CREATE DATABASE[ IF NOT EXISTS] tên_db tùy_chọn_db
    Tùy chọn db:
        CHARACTER SET charset_name
        COLLATE collation_name
-- Xem các database hiện có
    SHOW DATABASES[ LIKE 'PATTERN']
-- Xem thông tin database hiện tại
    SHOW CREATE DATABASE tên_db
-- Sửa tùy chọn của database
    ALTER DATABASE tên_db tùy_chọn
-- Xóa database
    DROP DATABASE[ IF EXISTS] tên_db
        Đồng thời xóa thư mục liên quan và nội dung thư mục đó
```

### Thao tác Bảng

```sql
/* Thao tác Bảng */
-- Tạo bảng
    CREATE [TEMPORARY] TABLE[ IF NOT EXISTS] [tên_db.]tên_bảng ( định_nghĩa_cấu_trúc_bảng )[ tùy_chọn_bảng]
        Mỗi field bắt buộc phải có kiểu dữ liệu
        Field cuối cùng không có dấu phẩy
        TEMPORARY là bảng tạm, tự biến mất khi kết thúc session
        Định nghĩa field:
            tên_field kiểu_dữ_liệu [NOT NULL | NULL] [DEFAULT giá_trị_mặc_định] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']
-- Tùy chọn bảng
    -- Tập ký tự
        CHARSET = charset_name
        Nếu bảng không đặt thì dùng charset của database
    -- Storage Engine
        ENGINE = engine_name
        Các engine phổ biến: InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
        File bảng MyISAM: .frm (định nghĩa bảng), .MYD (dữ liệu), .MYI (index)
        File bảng InnoDB: .frm (định nghĩa bảng), tablespace dữ liệu và log file
        SHOW ENGINES -- Hiển thị thông tin trạng thái Storage Engine
        SHOW ENGINE tên_engine {LOGS|STATUS} -- Hiển thị log hoặc trạng thái của engine
    -- Số khởi tạo tự tăng
    	AUTO_INCREMENT = số_hàng
    -- Thư mục file dữ liệu
        DATA DIRECTORY = 'thư_mục'
    -- Thư mục file index
        INDEX DIRECTORY = 'thư_mục'
    -- Ghi chú bảng
        COMMENT = 'string'
    -- Tùy chọn phân vùng (Partition)
        PARTITION BY ...
-- Xem tất cả các bảng
    SHOW TABLES[ LIKE 'pattern']
    SHOW TABLES FROM tên_db
-- Xem cấu trúc bảng
    SHOW CREATE TABLE tên_bảng (Thông tin chi tiết hơn)
    DESC tên_bảng / DESCRIBE tên_bảng / EXPLAIN tên_bảng / SHOW COLUMNS FROM tên_bảng [LIKE 'PATTERN']
    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']
-- Sửa bảng
    -- Sửa tùy chọn của bản thân bảng
        ALTER TABLE tên_bảng tùy_chọn_bảng
        ví dụ: ALTER TABLE tên_bảng ENGINE=MYISAM;
    -- Đổi tên bảng
        RENAME TABLE tên_bảng_cũ TO tên_bảng_mới
        RENAME TABLE tên_bảng_cũ TO tên_db.tên_bảng_mới (Có thể chuyển bảng sang database khác)
    -- Sửa cấu trúc field của bảng
        ALTER TABLE tên_bảng tên_thao_tác
        -- tên_thao_tác
            ADD[ COLUMN] định_nghĩa_field       -- Thêm field
                AFTER tên_field          -- Thêm vào sau field chỉ định
                FIRST               -- Thêm vào vị trí đầu tiên
            ADD PRIMARY KEY(tên_field)   -- Tạo Primary Key
            ADD UNIQUE [tên_index] (tên_field)-- Tạo Unique Index
            ADD INDEX [tên_index] (tên_field) -- Tạo Normal Index
            DROP[ COLUMN] tên_field      -- Xóa field
            MODIFY[ COLUMN] tên_field thuộc_tính_field     -- Sửa thuộc tính field, không sửa được tên field
            CHANGE[ COLUMN] tên_field_cũ tên_field_mới thuộc_tính_field      -- Sửa tên field
            DROP PRIMARY KEY    -- Xóa Primary Key (cần xóa thuộc tính AUTO_INCREMENT trước)
            DROP INDEX tên_index -- Xóa Index
            DROP FOREIGN KEY khóa_ngoại    -- Xóa Foreign Key
-- Xóa bảng
    DROP TABLE[ IF EXISTS] tên_bảng ...
-- Xóa sạch dữ liệu bảng
    TRUNCATE [TABLE] tên_bảng
-- Sao chép cấu trúc bảng
    CREATE TABLE tên_bảng LIKE bảng_cần_sao_chép
-- Sao chép cấu trúc bảng và dữ liệu
    CREATE TABLE tên_bảng [AS] SELECT * FROM bảng_cần_sao_chép
-- Kiểm tra bảng có lỗi không
    CHECK TABLE tbl_name [, tbl_name] ... [option] ...
-- Tối ưu hóa bảng
    OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
-- Sửa chữa bảng
    REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]
-- Phân tích bảng
    ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```

### Thao tác Dữ liệu

```sql
/* Thao tác Dữ liệu */
-- Thêm (INSERT)
    INSERT [INTO] tên_bảng [(danh_sách_cột)] VALUES (danh_sách_giá_trị)[, (danh_sách_giá_trị), ...]
        -- Nếu danh sách giá trị chứa tất cả các field và đúng thứ tự thì có thể bỏ qua danh sách cột.
        -- Có thể insert nhiều bản ghi cùng lúc!
        REPLACE tương tự INSERT, khác biệt duy nhất là nếu trùng hàng (so sánh Primary Key / Unique Key) thì đè dữ liệu cũ, không trùng thì chèn mới.
    INSERT [INTO] tên_bảng SET tên_cột=giá_trị[, tên_cột=giá_trị, ...]
-- Truy vấn (SELECT)
    SELECT danh_sách_cột FROM tên_bảng[ mệnh_đề_khác]
-- Xóa (DELETE)
    DELETE FROM tên_bảng[ điều_kiện_xóa]
        Không có điều kiện sẽ xóa toàn bộ hàng
-- Sửa (UPDATE)
    UPDATE tên_bảng SET tên_cột=giá_trị_mới[, tên_cột=giá_trị_mới] [điều_kiện_cập_nhật]
```

### Tập ký tự và Mã hóa

```sql
/* Tập ký tự và Mã hóa */
SHOW VARIABLES LIKE 'character_set_%'   -- Xem tất cả các mục charset
    character_set_client        Mã hóa client gửi dữ liệu sang server
    character_set_results       Mã hóa server trả kết quả về client
    character_set_connection    Mã hóa tầng connection
SET tên_biến = giá_trị_biến
    SET character_set_client = gbk;
    SET character_set_results = gbk;
    SET character_set_connection = gbk;
SET NAMES GBK;  -- Tương đương hoàn thành 3 thiết lập trên
-- Collation (Tập hiệu đối)
    SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern']   Xem tất cả charset
    SHOW COLLATION [LIKE 'pattern']     Xem tất cả collation
```

### Kiểu dữ liệu (Kiểu cột)

```sql
/* Kiểu dữ liệu (Kiểu cột) */
1. Kiểu số
-- a. Số nguyên ----------
    Kiểu         Kích thước     Phạm vi (SIGNED)        UNSIGNED (Không dấu)
    tinyint     1 byte    -128 ~ 127              0 ~ 255
    smallint    2 byte    -32768 ~ 32767
    mediumint   3 byte    -8388608 ~ 8388607
    int         4 byte
    bigint      8 byte
    int(M)  M thể hiện độ rộng hiển thị
    - Mặc định có dấu (SIGNED), dùng UNSIGNED để đổi sang không dấu
    - 1 biểu thị true, 0 biểu thị false. MySQL không có kiểu bool thuần, dùng tinyint(1).
-- b. Số thực dấu phẩy động ----------
    float(đơn)     4 byte
    double(kép)    8 byte
    float(M, D)     double(M, D) - M là tổng số chữ số, D là số chữ số thập phân.
-- c. Số định điểm ----------
    decimal(M, D)   M là tổng số chữ số, D là số chữ số thập phân.
    Lưu giá trị chính xác, không mất độ chính xác.
2. Kiểu chuỗi
-- a. char, varchar ----------
    char    Chuỗi độ dài cố định, tốc độ nhanh, tốn dung lượng
    varchar Chuỗi độ dài biến đổi, tốc độ chậm hơn, tiết kiệm dung lượng
    char: tối đa 255 ký tự.
    varchar: tối đa 65535 byte (tùy charset, utf8 tối đa 21844 ký tự).
-- b. blob, text ----------
    blob: chuỗi nhị phân (tinyblob, blob, mediumblob, longblob)
    text: chuỗi văn bản (tinytext, text, mediumtext, longtext)
-- c. binary, varbinary ----------
    Tương tự char và varchar nhưng lưu chuỗi nhị phân byte.
3. Kiểu ngày tháng thời gian
    datetime    8 byte    1000-01-01 00:00:00 đến 9999-12-31 23:59:59
    date        3 byte    1000-01-01 đến 9999-12-31
    timestamp   4 byte    1970-01-01 00:00:01 UTC đến 2038-01-19 03:14:07 UTC
    time        3 byte    -838:59:59 đến 838:59:59
    year        1 byte    1901 đến 2155
4. Enum và Set
    enum(val1, val2...): chọn 1 trong danh sách. Lưu dạng smallint (2 byte).
    set(val1, val2...): chọn nhiều trong danh sách (tối đa 64 phần tử). Lưu dạng bigint (8 byte).
```

### Thuộc tính cột (Ràng buộc cột)

```sql
/* Thuộc tính cột (Ràng buộc cột) */
1. PRIMARY KEY: Chủ khóa, xác định duy nhất bản ghi, không null.
2. UNIQUE: Ràng buộc duy nhất, cho phép null.
3. NULL / NOT NULL: Cho phép hoặc không cho phép null.
4. DEFAULT: Giá trị mặc định.
5. AUTO_INCREMENT: Tự động tăng (phải là Index).
6. COMMENT: Ghi chú cột.
7. FOREIGN KEY: Khóa ngoại (InnoDB hỗ trợ).
    foreign key (cột_ngoại) references bảng_chính(cột_liên_kết) [ON DELETE cascade|set null|restrict] [ON UPDATE cascade|set null|restrict]
```

### Quy chuẩn tạo bảng

```sql
-- 1NF (Dạng chuẩn 1): Cột không thể chia nhỏ hơn được nữa.
-- 2NF (Dạng chuẩn 2): Đạt 1NF và không có phụ thuộc một phần (loại bỏ Primary Key phức hợp).
-- 3NF (Dạng chuẩn 3): Đạt 2NF và không có phụ thuộc bắc cầu.
```

### SELECT

```sql
SELECT [ALL|DISTINCT] select_expr FROM -> WHERE -> GROUP BY -> HAVING -> ORDER BY -> LIMIT
a. Gợi ý Index cho Optimizer:
   USE INDEX, IGNORE INDEX, FORCE INDEX
b. Các toán tử WHERE:
   =, <=>, <>, !=, <=, <, >=, >, !, &&, ||, IN, LIKE, BETWEEN AND, IS NULL, IS NOT NULL
c. GROUP BY & Hàm tổng hợp:
   COUNT(), SUM(), MAX(), MIN(), AVG(), GROUP_CONCAT()
d. HAVING: Lọc trên kết quả đã GROUP BY.
e. ORDER BY: ASC (tăng dần), DESC (giảm dần).
f. LIMIT: LIMIT start, count.
```

### UNION

```sql
SELECT ... UNION [ALL|DISTINCT] SELECT ...
-- UNION ALL không loại bỏ trùng lặp (nhanh hơn), UNION loại bỏ trùng lặp.
```

### Subquery (Truy vấn con)

- FROM subquery: Bắt buộc đặt tên alias (`SELECT * FROM (SELECT * FROM tb) AS temp;`).
- WHERE subquery: Trả về giá trị đơn (scalar), trả về 1 cột (column - dùng IN/NOT IN/EXISTS/NOT EXISTS), hoặc 1 hàng (ROW).

### JOIN (Truy vấn liên kết)

- INNER JOIN: Liên kết trong (chỉ trả về hàng thỏa mãn điều kiện ở cả 2 bảng).
- LEFT JOIN: Liên kết trái (bảng trái giữ nguyên, bảng phải không có thì NULL).
- RIGHT JOIN: Liên kết phải (bảng phải giữ nguyên, bảng trái không có thì NULL).
- NATURAL JOIN: Tự động liên kết theo cột cùng tên.

### TRUNCATE

`TRUNCATE TABLE tbl_name`: Xóa bảng và tạo lại bảng mới. Trái với DELETE, TRUNCATE reset giá trị `AUTO_INCREMENT` và không thể rollback.

### Backup và Restore

```sql
-- Export 1 bảng / nhiều bảng / 1 database:
mysqldump -u_username -p_password db_name tbl_name > backup.sql
mysqldump -u_username -p_password --database db_name > backup.sql

-- Import:
mysql -u_username -p_password db_name < backup.sql
-- hoặc trong mysql client:
source backup.sql;
```

### View (Chế độ xem / 视图)

Virtual Table tạo từ câu SELECT.
`CREATE VIEW view_name AS SELECT ...;`
Thuật toán View: MERGE, TEMPTABLE, UNDEFINED.

### Transaction (Giao dịch)

- `START TRANSACTION;` / `BEGIN;`
- `COMMIT;`
- `ROLLBACK;`
- ACID: Atomicity, Consistency, Isolation, Durability.
- `SAVEPOINT savepoint_name;`, `ROLLBACK TO SAVEPOINT savepoint_name;`
- `SET autocommit = 0|1;`

### Lock Table (Khóa bảng)

- `LOCK TABLES tbl_name READ|WRITE;`
- `UNLOCK TABLES;`

### Trigger (Cò kích / 触发器)

`CREATE TRIGGER trigger_name BEFORE|AFTER INSERT|UPDATE|DELETE ON tbl_name FOR EACH ROW trigger_stmt;`
Sử dụng `OLD` và `NEW` để truy cập dữ liệu cũ và mới.

### SQL Programming & Stored Procedure / Function

- Biến cục bộ: `DECLARE var_name type DEFAULT value;`
- Biến toàn cục: `SET @var = value;`
- Điều kiện: `IF ... THEN ... ELSE ... END IF;`, `CASE ... WHEN ... END`
- Vòng lặp: `WHILE ... DO ... END WHILE;` (dùng `LEAVE` để thoát, `ITERATE` để tiếp tục)
- Hàm tự định nghĩa: `CREATE FUNCTION func_name(...) RETURNS type ...`
- Stored Procedure: `CREATE PROCEDURE sp_name(IN|OUT|INOUT var_name type) ...`, gọi bằng `CALL sp_name(...)`.

### Quản lý User và Phân quyền

```sql
-- Tạo User:
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
-- Đổi mật khẩu:
SET PASSWORD FOR 'username'@'host' = PASSWORD('new_password');
-- Gán quyền:
GRANT ALL PRIVILEGES ON db_name.* TO 'username'@'host';
-- Xem quyền:
SHOW GRANTS FOR 'username'@'host';
-- Thu hồi quyền:
REVOKE ALL PRIVILEGES ON db_name.* FROM 'username'@'host';
-- Xóa User:
DROP USER 'username'@'host';
-- Refresh quyền:
FLUSH PRIVILEGES;
```

### Bảo trì Bảng

```sql
ANALYZE TABLE tbl_name;
CHECK TABLE tbl_name;
OPTIMIZE TABLE tbl_name;
```

<!-- @include: @article-footer.snippet.md -->
