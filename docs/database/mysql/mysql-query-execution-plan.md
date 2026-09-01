---
title: MySQL执行计划分析
description: 详解MySQL EXPLAIN执行计划的各列含义，包括id、select_type、type、key、rows、Extra等关键字段解读，帮助你分析SQL性能瓶颈并进行针对性优化。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL执行计划,EXPLAIN,查询优化器,SQL性能分析,索引命中,type访问类型,Extra字段,慢查询优化
---

Bước đầu tiên để tối ưu hóa SQL là đọc hiểu Kế hoạch thực thi (Execution Plan) của câu lệnh SQL. Trong bài viết này, chúng ta sẽ cùng nhau tìm hiểu kiến thức liên quan đến Execution Plan `EXPLAIN` trong MySQL.

> **Ghi chú phiên bản**: Nội dung bài viết dựa trên MySQL 5.7+ và 8.0+. Các cột `filtered` và `partitions` có từ MySQL 5.7+, các tính năng `EXPLAIN ANALYZE` và Hash Join yêu cầu MySQL 8.0.18+ và 8.0.20+.

## Execution Plan là gì?

**Execution Plan (Kế hoạch thực thi)** đề cập đến cách thức thực thi cụ thể của một câu lệnh SQL sau khi được **MySQL Query Optimizer (Bộ tối ưu hóa truy vấn MySQL)** tối ưu hóa.

Execution Plan thường dùng trong các kịch bản phân tích hiệu năng SQL, tối ưu hóa SQL... Thông qua kết quả của `EXPLAIN`, có thể biết được các thông tin như thứ tự truy vấn các bảng dữ liệu, kiểu thao tác truy vấn dữ liệu, những Index nào có thể trúng, những Index nào thực tế trúng, mỗi bảng dữ liệu có bao nhiêu hàng bản ghi bị quét...

## Làm thế nào để lấy Execution Plan?

MySQL cung cấp lệnh `EXPLAIN` để lấy thông tin liên quan đến Execution Plan.

Cần lưu ý rằng, câu lệnh `EXPLAIN` chuẩn sẽ không thực sự thực thi các câu lệnh liên quan, mà thông qua Query Optimizer để phân tích câu lệnh, tìm ra phương án truy vấn tối ưu và hiển thị thông tin tương ứng.

MySQL 8.0.18 đưa vào `EXPLAIN ANALYZE`, nó sẽ **thực sự thực thi** truy vấn và output thời gian thực tế cũng như số hàng ở từng bước, tin cậy hơn so với dữ liệu ước tính của `EXPLAIN` chuẩn, thích hợp để định vị sâu các câu SQL chậm trong môi trường test:

```sql
mysql> EXPLAIN ANALYZE SELECT * FROM users WHERE age = 25\G
*************************** 1. row ***************************
EXPLAIN: -> Covering index lookup on users using idx_age_score_name (age=25)
(cost=1.52 rows=12) (actual time=0.0272..0.0344 rows=12 loops=1)
```

Ngoài ra, `EXPLAIN FORMAT=JSON` có thể output dữ liệu mô hình chi phí của Optimizer (`query_cost`), phản ánh chi phí thực tế ở từng bước tốt hơn dạng bảng, đặc biệt hữu ích khi tối ưu hóa JOIN nhiều bảng hoặc Subquery:

```sql
mysql> EXPLAIN FORMAT=JSON SELECT * FROM users WHERE age = 25\G
*************************** 1. row ***************************
EXPLAIN: {
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.52"
    },
    "table": {
      "table_name": "users",
      "access_type": "ref",
      "key": "idx_age_score_name",
      "rows_examined_per_scan": 12,
      "filtered": "100.00",
      "using_index": true
    }
  }
}
```

`EXPLAIN` hỗ trợ các câu lệnh `SELECT`, `DELETE`, `INSERT`, `REPLACE` cũng như `UPDATE`. Chúng ta thường dùng nhất để phân tích câu lệnh `SELECT`, cú pháp đơn giản như sau:

```sql
EXPLAIN SELECT câu_lệnh_truy_vấn;
```

Hãy cùng xem Execution Plan của một câu lệnh query đơn giản:

**Ví dụ 1: Truy vấn đơn bảng (dùng Index)**

```sql
-- Cấu trúc bảng: users(id, age, score, name, address), Composite Index idx_age_score_name(age, score, name)
mysql> EXPLAIN SELECT * FROM users WHERE age = 25;
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys       | key                 | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | users | NULL       | ref  | idx_age_score_name  | idx_age_score_name  | 5       | const |   12 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------+------+----------+-------------+
```

**Ví dụ 2: Truy vấn UNION (kịch bản id là NULL)**

```sql
mysql> EXPLAIN SELECT * FROM users WHERE id = 1 UNION SELECT * FROM users WHERE id = 2;
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type  | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY      | users      | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
|  2 | UNION        | users      | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
|  3 | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
```

Có thể thấy kết quả Execution Plan gồm 12 cột, ý nghĩa từng cột tóm tắt ở bảng sau:

| **Tên cột** | **Ý nghĩa** |
| ------------- | -------------------------------------------- |
| id | Mã định danh chuỗi truy vấn SELECT |
| select_type | Loaị truy vấn tương ứng với từ khóa SELECT |
| table | Tên bảng được sử dụng |
| partitions | Partition khớp, với bảng chưa phân vùng giá trị là NULL |
| type | Phương thức truy cập bảng (Access Type) |
| possible_keys | Index có thể sử dụng |
| key | Index thực tế sử dụng |
| key_len | Độ dài tối đa của Index được chọn |
| ref | Cột hoặc hằng số so sánh với Index khi dùng truy vấn bằng Index |
| rows | Số hàng ước tính cần đọc |
| filtered | Tỷ lệ phần trăm bản ghi giữ lại sau khi lọc qua điều kiện ở tầng Server |
| Extra | Thông tin bổ sung |

## Phân tích kết quả EXPLAIN như thế nào?

Để phân tích kết quả thực thi câu lệnh `EXPLAIN`, chúng ta cần hiểu rõ các field quan trọng trong Execution Plan.

### id

Định danh `SELECT`, dùng để xác định thứ tự thực thi của từng câu lệnh `SELECT`.

Quy tắc đọc cột `id`:

- **id bằng nhau**: Thực thi từ trên xuống dưới (thường xuất hiện trong kịch bản JOIN nhiều bảng).
- **id khác nhau**: Giá trị id càng lớn thì mức ưu tiên thực thi càng cao (Subquery thực thi trước câu truy vấn bên ngoài).
- **id là NULL**: Biểu thị đây là tập kết quả của UNION RESULT hoặc DERIVED, không cần thực thi truy vấn riêng biệt.

**Ví dụ**:

```sql
mysql> EXPLAIN SELECT * FROM users WHERE id = 1
    -> UNION
    -> SELECT * FROM users WHERE id = 2\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: users
         type: const
*************************** 2. row ***************************
           id: 2
  select_type: UNION
        table: users
         type: const
*************************** 3. row ***************************
           id: NULL
  select_type: UNION RESULT
        table: <union1,2>
         type: ALL
        Extra: Using temporary
```

Hàng thứ 3 có `id = NULL`, table = `<union1,2>`, biểu thị đây là kết quả gộp của 2 truy vấn trước.

### select_type

Loại truy vấn, chủ yếu dùng để phân biệt truy vấn thông thường, truy vấn hợp nhất (UNION), truy vấn con (Subquery)... Các giá trị phổ biến:

- **SIMPLE**: Truy vấn đơn giản, không chứa UNION hay Subquery.
- **PRIMARY**: Mệnh đề SELECT ngoài cùng nếu truy vấn chứa Subquery hoặc phần khác.
- **SUBQUERY**: Mệnh đề SELECT đầu tiên trong Subquery.
- **UNION**: Mệnh đề SELECT xuất hiện sau từ khóa UNION trong câu lệnh UNION.
- **DERIVED**: Subquery xuất hiện trong mệnh đề FROM sẽ đánh dấu là DERIVED (bảng phái sinh).
- **UNION RESULT**: Kết quả của truy vấn UNION.

### table

Tên bảng dùng trong truy vấn. Ngoài tên bảng thông thường, cũng có thể là các giá trị sau:

- **`<unionM,N>`** : Hàng này tham chiếu kết quả UNION của các hàng có id M và N;
- **`<derivedN>`** : Hàng này tham chiếu kết quả bảng phái sinh tạo ra từ bảng có id N.
- **`<subqueryN>`** : Hàng này tham chiếu kết quả Subquery đã vật hóa (materialized subquery) tạo ra từ bảng có id N.

### type (Quan trọng)

Loại thực thi truy vấn, mô tả cách thức truy vấn được thực thi. **Thứ tự từ tối ưu nhất đến kém nhất**:

`system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL`

**Quy tắc phán đoán kinh nghiệm về hiệu năng**:

- **Tốt** (tối thiểu đạt được): `system`, `const`, `eq_ref`, `ref`, `range`
- **Cần chú ý**: `index_merge`, `index` (quét toàn bộ Index, với lượng dữ liệu lớn vẫn có rủi ro hiệu năng)
- **Cần tối ưu**: `ALL` (quét toàn bộ bảng - Full Table Scan)

**Lưu ý**: Thứ tự này phản ánh **hiệu quả truy cập đơn bảng**, không đại diện cho hiệu năng tổng thể của truy vấn. Ví dụ `type=ref` kết hợp với Index Lookup (回表) số lượng lớn có thể chậm hơn `type=index` dùng Covering Index.

Ý nghĩa cụ thể của các loại phổ biến:

- **system**: Trong bảng chỉ có 1 hàng bản ghi (hoặc bảng rỗng) và Storage Engine có thể thống kê chính xác số hàng.
- **const**: Trong bảng có tối đa 1 hàng bản ghi khớp, chỉ cần 1 lần tìm kiếm là ra, thường dùng khi Primary Key hoặc Unique Index làm điều kiện truy vấn.
- **eq_ref**: Khi JOIN bảng, hàng ở bảng trước chỉ tương ứng 1 hàng ở bảng hiện tại. Thường dùng khi Primary Key hoặc Unique Index không null làm điều kiện JOIN.
- **ref**: Sử dụng Normal Index làm điều kiện truy vấn, kết quả có thể tìm thấy nhiều hàng khớp.
- **index_merge**: Khi mệnh đề WHERE chứa nhiều điều kiện phạm vi và mỗi điều kiện có thể dùng Index khác nhau, MySQL sẽ gộp kết quả quét từ nhiều Index. Cột key liệt kê các Index được dùng, cột Extra hiển thị thuật toán gộp:

  - `Using union(...)`: Lấy hợp các kết quả Index (điều kiện OR)
  - `Using sort_union(...)`: Sắp xếp kết quả Index rồi lấy hợp (điều kiện OR, cột Index không liên tục)
  - `Using intersection(...)`: Lấy giao các kết quả Index (điều kiện AND)

  **Ví dụ**:

  ```sql
  -- Điều kiện OR kích hoạt index merge union
  EXPLAIN SELECT * FROM employees WHERE emp_no = 10001 OR dept_no = 'd001';
  -- Extra: Using union(PRIMARY,dept_no_index)
  ```

- **range**: Thực hiện truy vấn phạm vi trên cột Index, cột key thể hiện Index nào được dùng.
- **index**: Full Index Scan, truy vấn duyệt toàn bộ cây Index. Tương tự như ALL nhưng overhead thường thấp hơn.
- **ALL**: Full Table Scan (Quét toàn bộ bảng).

### possible_keys

Cột possible_keys biểu thị các Index mà MySQL có thể sử dụng khi thực thi truy vấn. Nếu cột này là NULL biểu thị không có Index nào có thể dùng.

### key (Quan trọng)

Cột key biểu thị Index thực tế mà MySQL sử dụng. Nếu là NULL biểu thị không sử dụng Index.

### key_len

Cột key_len biểu thị độ dài tối đa của Index thực tế sử dụng. Khi dùng Composite Index, có thể là tổng độ dài của nhiều cột. Trong điều kiện đáp ứng nhu cầu thì càng ngắn càng tốt.

### rows

Cột rows biểu thị số hàng **ước tính** cần đọc để tìm thấy bản ghi dựa trên thông tin thống kê bảng và lựa chọn Index, giá trị càng nhỏ càng tốt.

Cần lưu ý đây là giá trị ước tính chứ không phải giá trị chính xác. Thống kê của InnoDB dựa trên lấy mẫu ngẫu nhiên trang Index:

- Số trang lấy mẫu kiểm soát bởi `innodb_stats_persistent_sample_pages` (mặc định 20 trang)
- Khi dữ liệu bảng biến động thường xuyên hoặc import số lượng lớn, giá trị ước tính có thể chênh lệch 10%～50% so với số hàng thực tế.

**Phương pháp xác minh**:

```sql
-- Số hàng ước tính trong Execution Plan
mysql> EXPLAIN SELECT * FROM users WHERE age = 25\G
rows: 12

-- Số hàng thực tế
mysql> SELECT COUNT(*) FROM users WHERE age = 25;
+----------+
| COUNT(*) |
+----------+
|       12 |
+----------+
```

### filtered

Cột filtered biểu thị tỷ lệ phần trăm (0~100) bản ghi giữ lại sau khi dữ liệu từ Storage Engine trả về được lọc qua điều kiện WHERE ở tầng Server. Công thức tính: `filtered = (số hàng sau khi lọc / số hàng Storage Engine trả về) * 100`.

- `filtered = 100`: Tất cả các hàng Storage Engine trả về đều thỏa mãn điều kiện WHERE (lý tưởng)
- `filtered < 100`: Một phần hàng bị tầng Server lọc bỏ, chứng tỏ Index chưa bao phủ tất cả điều kiện truy vấn.

### Extra (Quan trọng)

Cột này chứa thông tin bổ sung khi MySQL giải mã truy vấn:

- **Using filesort**: MySQL không thể lợi dụng Index để hoàn thành yêu cầu sắp xếp của ORDER BY hay GROUP BY, cần phải thực hiện một thao tác sắp xếp bổ sung sau khi nhận tập kết quả.
- **Using temporary**: MySQL cần tạo bảng tạm (Temporary Table) để lưu kết quả truy vấn, thường gặp trong ORDER BY và GROUP BY.
- **Using index**: Biểu thị truy vấn sử dụng Covering Index, không cần Index Lookup (回表), hiệu năng cực cao.
- **Using index condition**: Biểu thị Optimizer lựa chọn sử dụng tính năng Index Condition Pushdown (ICP).
- **Using where**: Tầng Server áp dụng lọc điều kiện WHERE bổ sung trên các hàng trả về từ Storage Engine.
- **Using join buffer (Block Nested Loop)**: Khi JOIN bảng, bảng được truy cập không dùng được Index, MySQL sẽ đọc dữ liệu bảng trước vào join buffer rồi duyệt匹配.
- **Using join buffer (hash join)**: MySQL 8.0.18 đưa vào thuật toán Hash Join, **chỉ dùng cho JOIN bằng** (`t1.id = t2.id`), từ 8.0.20 mặc định thay thế BNL.

Nhắc nhở: Khi cột Extra chứa `Using filesort` hoặc `Using temporary`, hiệu năng MySQL có thể có vấn đề, cần cố gắng tránh.

## Tham khảo

- <https://dev.mysql.com/doc/refman/8.0/en/explain-output.html>
- <https://dev.mysql.com/doc/refman/8.0/en/explain.html>
- <https://juejin.cn/post/6953444668973514789>

<!-- @include: @article-footer.snippet.md -->
