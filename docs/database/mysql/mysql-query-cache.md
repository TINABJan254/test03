---
title: MySQL查询缓存详解
description: 深入解析MySQL查询缓存的工作原理、配置管理及其优缺点，分析为什么MySQL 8.0移除了查询缓存功能，以及生产环境中的最佳实践建议。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL查询缓存,Query Cache,MySQL缓存机制,缓存失效,MySQL 8.0,查询性能优化,MySQL内存管理
---

Cache (Bộ nhớ tạm) là một phương tiện tối ưu hóa hiệu năng hệ thống hiệu quả và thực tế, từ hệ điều hành cho đến các phần mềm ứng dụng và Web Service đều áp dụng rộng rãi cơ chế cache.

Tuy nhiên, các DBA có kinh nghiệm đều khuyến nghị nên tắt tính năng Query Cache (Bộ nhớ tạm truy vấn) tự có của MySQL trong môi trường production. Hơn nữa, từ MySQL 5.7.20 trở đi đã mặc định không khuyến nghị dùng Query Cache. Đến phiên bản MySQL 8.0 và về sau, MySQL đã trực tiếp xóa bỏ hoàn toàn tính năng Query Cache.

Tại sao lại như vậy? Query Cache thực sự "gà mờ" đến thế sao?

Mang theo các câu hỏi dưới đây, chúng ta hãy cùng đi vào nội dung bài viết:

- MySQL Query Cache là gì? Phạm vi áp dụng?
- Quy tắc cache của MySQL là gì?
- Ưu nhược điểm của MySQL Query Cache là gì?
- MySQL Query Cache ảnh hưởng thế nào đến hiệu năng?

## Giới thiệu MySQL Query Cache

Kiến trúc tổng thể của MySQL như hình dưới:

![](https://oss.javaguide.cn/github/javaguide/mysql/mysql-architecture.png)

Để tăng tốc độ phản hồi cho các câu lệnh truy vấn hoàn toàn giống nhau, MySQL Server sẽ tiến hành tính toán Hash trên câu lệnh query để thu được một giá trị Hash. MySQL Server không xử lý bất kỳ biến đổi nào trên SQL, SQL bắt buộc phải hoàn toàn giống hệt nhau thì giá trị Hash mới giống nhau. Sau khi có giá trị Hash, nó tìm kiếm trong Query Cache xem có kết quả tương ứng hay không.

- Nếu trúng (hit), trực tiếp trả kết quả về cho client mà không cần parse hay execute câu lệnh.
- Nếu không trúng (miss), lưu giá trị Hash và kết quả truy vấn vào Query Cache để dùng cho lần sau.

Nghĩa là, **khi một câu lệnh query (SELECT) tới MySQL Server, nó sẽ tới Query Cache kiểm tra trước, nếu từng thực thi rồi thì trả thẳng kết quả về client.**

![](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

## Quản lý và Cấu hình MySQL Query Cache

Thông qua lệnh `show variables like '%query_cache%';` có thể xem thông tin liên quan đến Query Cache.

Trước phiên bản 8.0, thông tin in ra có dạng:

```bash
mysql> show variables like '%query_cache%';
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| have_query_cache             | YES     |
| query_cache_limit            | 1048576 |
| query_cache_min_res_unit     | 4096    |
| query_cache_size             | 599040  |
| query_cache_type             | ON      |
| query_cache_wlock_invalidate | OFF     |
+------------------------------+---------+
6 rows in set (0.02 sec)
```

Từ phiên bản 8.0 trở đi, thông tin in ra như sau:

```bash
mysql> show variables like '%query_cache%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| have_query_cache | NO    |
+------------------+-------+
1 row in set (0.01 sec)
```

Giải thích các thông số:

- **`have_query_cache`**: MySQL Server có hỗ trợ Query Cache hay không, YES là có, NO là không.
- **`query_cache_limit`**: Kích thước kết quả truy vấn tối đa được cache, nếu lớn hơn giá trị này sẽ không cache.
- **`query_cache_min_res_unit`**: Đơn vị khối (block) cấp phát tối thiểu của Query Cache (byte). Điều chỉnh thích hợp giúp tối ưu bộ nhớ.
- **`query_cache_size`**: Dung lượng bộ nhớ cấp phát cho Query Cache (byte), phải là bội số của 1024. Đặt là 0 nghĩa là vô hiệu hóa Query Cache.
- **`query_cache_type`**: Loại Query Cache, mặc định là ON.
- **`query_cache_wlock_invalidate`**: Nếu bảng bị khóa, có trả về dữ liệu trong cache hay không.

Các giá trị có thể của `query_cache_type`:

- 0 hoặc OFF: Tắt tính năng Query Cache.
- 1 hoặc ON: Bật tính năng Query Cache, ngoại trừ các câu lệnh bắt đầu bằng `SELECT SQL_NO_CACHE`.
- 2 hoặc DEMAND: Bật tính năng Query Cache, nhưng chỉ cache các câu lệnh bắt đầu bằng `SELECT SQL_CACHE`.

**Khuyến nghị**:
Nên vô hiệu hóa Query Cache bằng cách đặt `query_cache_size=0` thay vì chỉ dựa vào `query_cache_type`, vì `query_cache_size=0` sẽ bỏ qua hoàn toàn việc cấp phát bộ nhớ và kiểm tra cache.

Các lệnh dọn dẹp cache thủ công:

- `flush query cache;`: Dọn dẹp mảnh bộ nhớ (fragmentation) của Query Cache.
- `reset query cache;`: Xóa tất cả các query khỏi Query Cache.
- `flush tables;`: Đóng tất cả các bảng đang mở và xóa sạch nội dung trong Query Cache.

## Cơ chế Cache trong MySQL

### Quy tắc Cache

- Query Cache lưu câu lệnh truy vấn và kết quả vào RAM dưới dạng key-value. Key được tính toán Hash từ văn bản câu SQL, Database hiện tại, character set và protocol version...
- Kết quả cache được chia sẻ giữa các session.
- Câu SQL bắt buộc phải hoàn toàn giống hệt nhau mới trúng cache (chữ hoa chữ thường, khoảng trắng, database, protocol, charset...).
- Không cache kết quả của Subquery, chỉ cache kết quả cuối cùng.
- Các hàm không xác định sẽ không bao giờ được cache (`now()`, `curdate()`, `last_insert_id()`, `rand()`...).
- Không cache các query phát sinh cảnh báo (Warnings).
- Kết quả vượt quá `query_cache_limit` (mặc định 1MB) không được cache.
- Khi dữ liệu hoặc cấu trúc của bảng thay đổi, tất cả cache liên quan đến bảng đó đều bị vô hiệu hóa.
- Không cache các query dùng `SQL_NO_CACHE`.

Ví dụ tùy chọn `SELECT`:

```sql
SELECT SQL_CACHE id, name FROM customer; # Được cache
SELECT SQL_NO_CACHE id, name FROM customer; # Không được cache
```

### Quản lý bộ nhớ trong cơ chế Cache

Query Cache được lưu hoàn toàn trong bộ nhớ RAM, sử dụng cơ chế Memory Pool tự quản lý cấp phát và giải phóng. Đơn vị cơ bản là các block độ dài biến đổi.

Theo thời gian chạy đồng thời đọc ghi, các block bị giải phóng hỗn loạn rải rác dẫn đến phát sinh lượng lớn mảnh bộ nhớ (fragmentation), làm tăng tần suất dọn dẹp bộ nhớ.

## Ưu nhược điểm của MySQL Query Cache

**Ưu điểm:**

- Khi trúng cache, trả thẳng kết quả từ RAM mà không cần parse, optimize hay tương tác với Storage Engine, tiết kiệm chi phí I/O đĩa và CPU. Tuy nhiên ưu điểm này chỉ đúng trong kịch bản tĩnh đọc nhiều ghi ít và concurrency thấp.

**Nhược điểm:**

- Query Cache phụ thuộc vào một khóa độc chiếm toàn cục duy nhất (`LOCK_query_cache`). Trong môi trường concurrency cao, hàng ngàn query tranh chấp khóa này gây ra nghẽn cổ chai hiệu năng nghiêm trọng.
- Vấn đề vô hiệu hóa cache: Nếu bảng bị sửa đổi thường xuyên (ghi/cập nhật dữ liệu, sửa cấu trúc bảng, sửa Index), tỷ lệ vô hiệu hóa vô cùng cao.
- Chênh lệch khoảng trắng, chữ hoa chữ thường cũng khiến SQL bị coi là câu lệnh mới, gây tiêu tốn tài nguyên bộ nhớ.

## Tác động của MySQL Query Cache đến hiệu năng

Khi bật Query Cache, cả thao tác đọc lẫn ghi đều tốn thêm overhead:

- Thao tác đọc phải lấy khóa `LOCK_query_cache` để kiểm tra cache.
- Thao tác ghi phải ghi kết quả vào cache và cấp phát bộ nhớ.
- Khi ghi vào bảng, bắt buộc phải lấy khóa độc chiếm để vô hiệu hóa tất cả cache liên quan đến bảng đó.
- Transaction dài trong InnoDB làm trầm trọng thêm vấn đề tranh chấp khóa.

Xem trạng thái Query Cache bằng lệnh:

```sql
SHOW STATUS LIKE 'Qcache%';
```

Công thức tính tỷ lệ trúng cache:

```
Tỷ lệ trúng = Qcache_hits / (Qcache_hits + Qcache_inserts + Qcache_not_cached)
```

Nếu tỷ lệ trúng lâu dài dưới 50%, khuyến nghị nên tắt Query Cache.

## Tóm tắt

Query Cache chỉ thích hợp cho kịch bản dữ liệu tĩnh, ít cập nhật (ví dụ hệ thống blog).

Đối với các hệ thống cập nhật thường xuyên, Query Cache mang lại ít tác dụng và có thể làm giảm hiệu năng hệ thống.

Trong dự án thực tế, **mạnh mẽ khuyến nghị sử dụng Local Cache (như Caffeine) hoặc Distributed Cache (như Redis)** mang lại hiệu năng tốt hơn và tính tổng quát cao hơn.

## Tham khảo

- 《高性能 MySQL》
- MySQL 缓存机制：<https://zhuanlan.zhihu.com/p/55947158>
- RDS MySQL 查询缓存（Query Cache）的设置和使用 - 阿里元云数据库 RDS 文档:<https://help.aliyun.com/document_detail/41717.html>
- 8.10.3 The MySQL Query Cache - MySQL 官方文档：<https://dev.mysql.com/doc/refman/5.7/en/query-cache.html>

<!-- @include: @article-footer.snippet.md -->
