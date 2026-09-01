---
title: SQL语法基础知识总结
description: SQL语法基础知识总结，系统讲解DDL数据定义、DML数据操作、DQL数据查询、DCL数据控制语言，涵盖表操作、约束、索引、事务、连接查询等核心知识点。
category: 数据库
tag:
  - 数据库基础
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL语法,DDL,DML,DQL,DCL,CREATE,SELECT,INSERT,UPDATE,DELETE,JOIN连接,子查询
---

> Bài viết này được tổng hợp và hoàn thiện từ 2 tài liệu dưới đây:
>
> - [SQL 语法速成手册](https://juejin.cn/post/6844903790571700231)
> - [MySQL 超全教程](https://www.begtut.com/mysql/mysql-tutorial.html)

## Khái niệm cơ bản

### Thuật ngữ Database

- `Database (Cơ sở dữ liệu)` - Container lưu trữ dữ liệu có tổ chức (thông thường là một file hoặc một tập hợp file).
- `Table (Bảng dữ liệu)` - Danh sách cấu trúc hóa của một loại dữ liệu cụ thể.
- `Schema (Lược đồ)` - Thông tin về bố cục và đặc tính của Database và Table. Schema định nghĩa dữ liệu được lưu trữ trong Table như thế nào, bao gồm lưu loại dữ liệu gì, dữ liệu phân tách ra sao, thông tin từng phần được đặt tên thế nào, v.v. Cả Database và Table đều có Schema.
- `Column (Cột / Trường / Field)` - Một field trong Table. Tất cả các Table đều được cấu thành từ một hoặc nhiều Column.
- `Row (Dòng / Bản ghi / Record)` - Một bản ghi trong Table.
- `Primary Key (Khóa chính)` - Một Column (hoặc một nhóm Column) có giá trị có thể định danh duy nhất cho từng Row trong Table.

### Cú pháp SQL

SQL (Structured Query Language), SQL tiêu chuẩn được quản lý bởi Ủy ban Tiêu chuẩn ANSI, do đó được gọi là ANSI SQL. Các DBMS đều có bản thực thi riêng của mình, chẳng hạn như PL/SQL, Transact-SQL, v.v.

#### Cấu trúc cú pháp SQL

![](https://oss.javaguide.cn/p3-juejin/cb684d4c75fc430e92aaee226069c7da~tplv-k3u1fbpfcp-zoom-1.png)

Cấu trúc cú pháp SQL bao gồm:

- **`Mệnh đề (Clause)`** - Là thành phần cấu thành nên câu lệnh và truy vấn. (Trong một số trường hợp, các mệnh đề này là tùy chọn).
- **`Biểu thức (Expression)`** - Có thể tạo ra bất kỳ giá trị vô hướng (scalar value) nào, hoặc tạo ra từ các Column và Row của Table.
- **`Vị ngữ (Predicate)`** - Chỉ định điều kiện cho logic 3 giá trị (3VL) trong SQL (true/false/unknown) hoặc giá trị Boolean cần đánh giá, đồng thời giới hạn phạm vi tác động của câu lệnh và truy vấn, hoặc thay đổi luồng chương trình.
- **`Truy vấn (Query)`** - Truy xuất dữ liệu dựa trên điều kiện cụ thể. Đây là một thành phần quan trọng của SQL.
- **`Câu lệnh (Statement)`** - Có thể tác động lâu dài lên Schema và dữ liệu, cũng như điều khiển Transaction, luồng chương trình, Connection, Session hoặc chẩn đoán.

#### Điểm mấu chốt cú pháp SQL

- **Câu lệnh SQL không分界面 chữ hoa chữ thường**, tuy nhiên tên Table, tên Column và giá trị có分界面 hay不，phụ thuộc vào DBMS cụ thể和cấu hình của nó. Ví dụ: `SELECT`, `select`, `Select` là giống nhau.
- **Nhiều câu lệnh SQL phải được分界面 bằng dấu chấm phẩy (`;`)**.
- Khi xử lý câu lệnh SQL, **tất cả khoảng trắng đều được bỏ qua**.

Câu lệnh SQL có thể viết trên 1 dòng, hoặc chia thành nhiều dòng.

```sql
-- 一行 SQL 语句

UPDATE user SET username='robot', password='robot' WHERE username = 'root';

-- 多行 SQL 语句
UPDATE user
SET username='robot', password='robot'
WHERE username = 'root';
```

SQL hỗ trợ 3 loại Comment:

```sql
## 注释1
-- 注释2
/* 注释3 */
```

### Phân loại SQL

#### Data Definition Language (DDL)

Data Definition Language (DDL - Ngôn ngữ định nghĩa dữ liệu) là tập hợp ngôn ngữ chịu trách nhiệm định nghĩa cấu trúc dữ liệu và định nghĩa các đối tượng Database trong SQL.

Chức năng chính của DDL là **định nghĩa các đối tượng Database**.

Các lệnh cốt lõi của DDL là `CREATE`, `ALTER`, `DROP`.

#### Data Manipulation Language (DML)

Data Manipulation Language (DML - Ngôn ngữ thao tác dữ liệu) được sử dụng cho các thao tác Database, thực hiện công việc truy cập vào các đối tượng và dữ liệu bên trong Database.

Chức năng chính của DML là **truy cập dữ liệu**, do đó cú pháp của nó chủ yếu là **đọc và ghi Database**.

Các lệnh cốt lõi của DML là `INSERT`, `UPDATE`, `DELETE`, `SELECT`. Bốn lệnh này được gọi chung là CRUD (Create, Read, Update, Delete), tức Thêm, Xóa, Sửa, Truy vấn.

#### Transaction Control Language (TCL)

Transaction Control Language (TCL - Ngôn ngữ điều khiển Transaction) dùng để **quản lý Transaction trong Database**. Chúng dùng để quản lý các thay đổi được tạo bởi các câu lệnh DML. Nó cũng cho phép gom nhóm các câu lệnh thành một Transaction logic.

Các lệnh cốt lõi của TCL là `COMMIT`, `ROLLBACK`.

#### Data Control Language (DCL)

Data Control Language (DCL - Ngôn ngữ điều khiển dữ liệu) là các câu lệnh có thể kiểm soát quyền truy cập dữ liệu, nó có thể kiểm soát quyền của tài khoản người dùng cụ thể đối概念 với các đối tượng Database như Table, View, Stored Procedure, User-Defined Function, v.v.

Các lệnh cốt lõi của DCL là `GRANT`, `REVOKE`.

DCL chủ yếu **kiểm soát quyền truy cập của người dùng**, do đó cách dùng của nó không quá phức tạp. Các quyền có thể kiểm soát bằng DCL gồm: `CONNECT`, `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE`, `USAGE`, `REFERENCES`.

Tùy theo DBMS khác nhau và Security Entity khác nhau, các kiểm soát quyền được hỗ trợ cũng sẽ khác nhau.

**Đầu tiên chúng ta cùng giới thiệu cách dùng các câu lệnh DML. Chức năng chính của DML là đọc和ghi Database để thực hiện Thêm, Xóa, Sửa, Truy vấn (CRUD).**

## Thao tác Thêm, Xóa, Sửa, Truy vấn (CRUD)

CRUD (Create, Read, Update, Delete) là các thao tác cơ bản nhất trong các thao tác Database cơ bản.

### INSERT dữ liệu

Câu lệnh `INSERT INTO` dùng để thêm các bản ghi mới vào Table.

**INSERT một dòng hoàn chỉnh**

```sql
# 插入一行
INSERT INTO user
VALUES (10, 'root', 'root', 'xxxx@163.com');
# 插入多行
INSERT INTO user
VALUES (10, 'root', 'root', 'xxxx@163.com'), (12, 'user1', 'user1', 'xxxx@163.com'), (18, 'user2', 'user2', 'xxxx@163.com');
```

**INSERT một phần dòng**

```sql
INSERT INTO user(username, password, email)
VALUES ('admin', 'admin', 'xxxx@163.com');
```

**INSERT dữ liệu được truy vấn ra**

```sql
INSERT INTO user(username)
SELECT name
FROM account;
```

### UPDATE dữ liệu

Câu lệnh `UPDATE` dùng để cập nhật bản ghi trong Table.

```sql
UPDATE user
SET username='robot', password='robot'
WHERE username = 'root';
```

### DELETE dữ liệu

- Câu lệnh `DELETE` dùng để xóa bản ghi trong Table.
- `TRUNCATE TABLE` có thể làm rỗng Table, tức là xóa tất cả các dòng. Lưu ý: Câu lệnh `TRUNCATE` không thuộc cú pháp DML mà là cú pháp DDL.

**Xóa dữ liệu chỉ định trong Table**

```sql
DELETE FROM user
WHERE username = 'robot';
```

**Làm rỗng dữ liệu trong Table**

```sql
TRUNCATE TABLE user;
```

### SELECT (Truy vấn) dữ liệu

Câu lệnh `SELECT` dùng để truy vấn dữ liệu từ Database.

`DISTINCT` dùng để trả về các giá trị duy nhất khác nhau. Nó tác động lên tất cả các Column, tức là giá trị của tất cả các Column đều giống nhau mới được tính là trùng nhau.

`LIMIT` giới hạn số dòng trả về. Có thể có 2 tham số: tham số thứ nhất là dòng bắt đầu (bắt đầu từ 0); tham số thứ 2 là tổng số dòng trả về.

- `ASC`: Tăng dần (mặc định)
- `DESC`: Giảm dần

**Truy vấn 1 column**

```sql
SELECT prod_name
FROM products;
```

**Truy vấn nhiều column**

```sql
SELECT prod_id, prod_name, prod_price
FROM products;
```

**Truy vấn tất cả column**

```sql
SELECT *
FROM products;
```

**Truy vấn các giá trị duy nhất (DISTINCT)**

```sql
SELECT DISTINCT
vend_id FROM products;
```

**Giới hạn kết quả truy vấn (LIMIT)**

```sql
-- 返回前 5 行
SELECT * FROM mytable LIMIT 5;
SELECT * FROM mytable LIMIT 0, 5;
-- 返回第 3 ~ 5 行
SELECT * FROM mytable LIMIT 2, 3;
```

## Sắp xếp (ORDER BY)

`ORDER BY` dùng để sắp xếp tập kết quả theo 1 hoặc nhiều Column. Mặc định sắp xếp bản ghi theo thứ tự tăng dần, nếu cần sắp xếp theo thứ tự giảm dần có thể dùng từ khóa `DESC`.

`ORDER BY` khi sắp xếp trên nhiều Column, Column nào sắp xếp trước thì đặt trước, Column nào sắp xếp sau thì đặt sau. Đồng thời, các Column khác nhau có thể áp dụng các quy tắc sắp xếp khác nhau.

```sql
SELECT * FROM products
ORDER BY prod_price DESC, prod_name ASC;
```

## Gom nhóm (GROUP BY)

**`GROUP BY`**:

- Mệnh đề `GROUP BY` gom các bản ghi thành các dòng tổng hợp.
- `GROUP BY` trả về 1 bản ghi cho mỗi nhóm.
- `GROUP BY` thông thường còn liên quan đến các hàm Aggregation như `COUNT`, `MAX`, `SUM`, `AVG`, v.v.
- `GROUP BY` có thể gom nhóm theo 1 Column hoặc nhiều Column.
- Sau khi `GROUP BY` sắp xếp theo field gom nhóm, `ORDER BY` có thể sắp xếp dựa theo field tổng hợp.

**Gom nhóm**

```sql
SELECT cust_name, COUNT(cust_address) AS addr_num
FROM Customers GROUP BY cust_name;
```

**Sắp xếp sau khi gom nhóm**

```sql
SELECT cust_name, COUNT(cust_address) AS addr_num
FROM Customers GROUP BY cust_name
ORDER BY cust_name DESC;
```

**`HAVING`**:

- `HAVING` dùng để lọc kết quả tổng hợp của `GROUP BY`.
- `HAVING` thông thường được dùng đi kèm với `GROUP BY`.
- `WHERE` và `HAVING` có thể xuất hiện trong cùng một câu truy vấn.

**Sử dụng WHERE和HAVING để lọc dữ liệu**

```sql
SELECT cust_name, COUNT(*) AS NumberOfOrders
FROM Customers
WHERE cust_email IS NOT NULL
GROUP BY cust_name
HAVING COUNT(*) > 1;
```

**So sánh `HAVING` vs `WHERE`**:

- `WHERE`: Lọc các dòng chỉ định, phía sau không được dùng hàm Aggregation (hàm gom nhóm). `WHERE` đứng trước `GROUP BY`.
- `HAVING`: Lọc nhóm, thông thường dùng đi kèm với `GROUP BY`, không thể sử dụng riêng lẻ. `HAVING` đứng sau `GROUP BY`.

## Subquery (Truy vấn con)

Subquery là câu truy vấn SQL được lồng bên trong một câu truy vấn lớn hơn, còn gọi là Inner Query hoặc Inner Select, câu lệnh chứa Subquery còn được gọi là Outer Query hoặc Outer Select. Nói một cách đơn giản, Subquery chính là lấy kết quả của một truy vấn `SELECT` (Subquery) làm nguồn dữ liệu hoặc điều kiện kiểm tra cho một câu lệnh SQL khác (Main Query).

Subquery có thể nhúng vào các câu lệnh `SELECT`, `INSERT`, `UPDATE` và `DELETE`, cũng như có thể kết hợp sử dụng với các toán tử như `=`, `<`, `>`, `IN`, `BETWEEN`, `EXISTS`, v.v.

Subquery thường được dùng phía sau mệnh đề `WHERE` và mệnh đề `FROM`:

- Khi dùng trong mệnh đề `WHERE`, tùy thuộc vào toán tử khác nhau, Subquery có thể trả về dữ liệu 1 dòng 1 cột, nhiều dòng 1 cột, 1 dòng nhiều cột. Subquery là trả về giá trị có thể làm điều kiện truy vấn cho mệnh đề `WHERE`.
- Khi dùng trong mệnh đề `FROM`, thông thường trả về dữ liệu nhiều dòng nhiều cột, tương đương với trả về 1 bảng tạm, như vậy mới tuân thủ quy tắc phía sau `FROM` là một Table. Cách làm này có thể thực hiện truy vấn kết hợp nhiều bảng.

> Lưu ý: Database MySQL từ phiên bản 4.1 mới bắt đầu hỗ trợ Subquery, các phiên bản trước đây không hỗ trợ.

Cú pháp cơ bản của Subquery dùng trong mệnh đề `WHERE` như sau:

```sql
select column_name [, column_name ]
from   table1 [, table2 ]
where  column_name operator
    (select column_name [, column_name ]
    from table1 [, table2 ]
    [where])
```

- Subquery cần đặt trong ngoặc đơn `( )`.
- `operator` biểu thị toán tử dùng cho mệnh đề WHERE.

Cú pháp cơ bản của Subquery dùng trong mệnh đề `FROM` như sau:

```sql
select column_name [, column_name ]
from (select column_name [, column_name ]
      from table1 [, table2 ]
      [where]) as temp_table_name
where  condition
```

Kết quả trả về của Subquery dùng trong `FROM` tương đương với 1 bảng tạm, do đó cần dùng từ khóa AS để đặt tên cho bảng tạm này.

**Subquery của Subquery (Subquery lồng多层)**

```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                  FROM orders
                  WHERE order_num IN (SELECT order_num
                                      FROM orderitems
                                      WHERE prod_id = 'RGAN01'));
```

Inner Query được thực thi trước Parent Query của nó, để kết quả của Inner Query có thể truyền cho Outer Query. Quá trình thực thi có thể tham khảo hình dưới:

![](https://oss.javaguide.cn/p3-juejin/c439da1f5d4e4b00bdfa4316b933d764~tplv-k3u1fbpfcp-zoom-1.png)

### Mệnh đề WHERE

- Mệnh đề `WHERE` dùng để lọc các bản ghi, tức thu hẹp phạm vi dữ liệu truy cập.
- Phía sau `WHERE` là một điều kiện trả về `true` hoặc `false`.
- `WHERE` có thể dùng chung với `SELECT`, `UPDATE`与`DELETE`.
- Các toán tử有 thể dùng trong mệnh đề `WHERE`.

| 运算符 | 描述 |
| ------- | ------------------------------------------------------ |
| = | Bằng |
| <> | Không bằng. Ghi chú: Trong một số phiên bản SQL toán tử này có thể viết là != |
| > | Lớn hơn |
| < | Nhỏ hơn |
| >= | Lớn hơn hoặc bằng |
| <= | Nhỏ hơn hoặc bằng |
| BETWEEN | Trong một khoảng |
| LIKE | Tìm kiếm theo pattern |
| IN | Chỉ định nhiều giá trị có thể có đối与1 column |

**Mệnh đề `WHERE` trong câu lệnh `SELECT`**

```ini
SELECT * FROM Customers
WHERE cust_name = 'Kids Place';
```

**Mệnh đề `WHERE` trong câu lệnh `UPDATE`**

```ini
UPDATE Customers
SET cust_name = 'Jack Jones'
WHERE cust_name = 'Kids Place';
```

**Mệnh đề `WHERE` trong câu lệnh `DELETE`**

```ini
DELETE FROM Customers
WHERE cust_name = 'Kids Place';
```

### IN与BETWEEN

- Toán tử `IN` dùng trong mệnh đề `WHERE`, có tác dụng chọn bất kỳ một giá trị nào在số các值cụ thể được chỉ定.
- Toán tử `BETWEEN` dùng在mệnh đề `WHERE`, có tác dụng chọn值nằm在một khoảng nhất定.

**Ví dụ IN**

```sql
SELECT *
FROM products
WHERE vend_id IN ('DLL01', 'BRS01');
```

**Ví dụ BETWEEN**

```sql
SELECT *
FROM products
WHERE prod_price BETWEEN 3 AND 5;
```

### AND, OR, NOT

- `AND`, `OR`, `NOT` là các chỉ thị xử lý logic đối与điều kiện lọc.
- `AND` có độ ưu tiên cao hơn `OR`, để làm rõ thứ tự xử lý có thể dùng `()`.
- Toán tử `AND` biểu thị cả 2 điều kiện bên trái和bên右đều phải thỏa mãn.
- Toán tử `OR` biểu thị thỏa mãn bất kỳ điều kiện nào bên trái hoặc bên右là được.
- Toán tử `NOT` dùng để phủ定một điều kiện.

**Ví dụ AND**

```sql
SELECT prod_id, prod_name, prod_price
FROM products
WHERE vend_id = 'DLL01' AND prod_price <= 4;
```

**Ví dụ OR**

```ini
SELECT prod_id, prod_name, prod_price
FROM products
WHERE vend_id = 'DLL01' OR vend_id = 'BRS01';
```

**Ví dụ NOT**

```sql
SELECT *
FROM products
WHERE prod_price NOT BETWEEN 3 AND 5;
```

### LIKE

- Toán tử `LIKE` dùng在mệnh đề `WHERE`, có tác dụng xác定chuỗi ký tự có khớp与pattern hay不.
- Chỉ khi field là值văn bản (text) mới dùng `LIKE`.
- `LIKE` hỗ trợ 2 tùy chọn khớp ký tự đại diện (wildcard): `%`和`_`.
- 不nên lạm dụng wildcard, wildcard nằm在đầu chuỗi thì truy vấn khớp sẽ rất chậm.
- `%` biểu thị bất kỳ ký tự nào xuất hiện số lần bất kỳ.
- `_` biểu thị bất kỳ ký tự nào xuất hiện đúng 1 lần.

**Ví dụ %**

```sql
SELECT prod_id, prod_name, prod_price
FROM products
WHERE prod_name LIKE '%bean bag%';
```

**Ví dụ \_**

```sql
SELECT prod_id, prod_name, prod_price
FROM products
WHERE prod_name LIKE '__ inch teddy bear';
```

## Phép JOIN (Kết nối bảng)

JOIN có nghĩa là "kết nối", đúng như tên gọi, mệnh đề SQL JOIN dùng để kết hợp 2 hoặc nhiều Table lại để thực hiện truy vấn.

Khi JOIN các Table cần chọn một field在từng Table和so sánh值của các field này, hai bản ghi có值giống nhau sẽ hợp nhất thành một. **Bản chất của việc JOIN các Table chính là hợp nhất các bản ghi từ các Table khác nhau để tạo成một Table mới. Tất nhiên, Table mới试只là tạm thời, nó chỉ tồn tại在suốt thời gian truy vấn này**.

Cú pháp cơ bản sử dụng `JOIN` để kết nối 2 Table như sau:

```sql
select table1.column1, table2.column2...
from table1
join table2
on table1.common_column1 = table2.common_column2;
```

`table1.common_column1 = table2.common_column2` là điều kiện kết nối, chỉ những bản ghi thỏa mãn điều kiện này mới hợp nhất thành một dòng. Bạn có thể sử dụng nhiều toán tử để kết nối Table, ví dụ `=`, `>`, `<`, `<>`, `<=`, `>=`, `!=`, `BETWEEN`, `LIKE` hoặc `NOT`, nhưng phổ变nhất là dùng `=`.

Khi 2 Table có các field trùng tên, để giúp Database Engine分界面đó là field của Table nào, khi viết tên field trùng tên cần phải thêm tên Table vào. Tất nhiên, nếu tên field viết ra là duy nhất在cả 2 Table则cũng có thể不cần dùng定dạng trên, chỉ cần viết tên field là được.

Ngoài ra, nếu tên field liên kết của 2 Table giống nhau, cũng có thể dùng mệnh đề `USING` để thay thế `ON`, lấy ví dụ:

```sql
# join....on
select c.cust_name, o.order_num
from Customers c
inner join Orders o
on c.cust_id = o.cust_id
order by c.cust_name;

# Nếu tên field liên kết của 2 Table giống nhau, cũng có thể dùng mệnh đề USING: join....using()
select c.cust_name, o.order_num
from Customers c
inner join Orders o
using(cust_id)
order by c.cust_name;
```

**Sự khác biệt giữa `ON`和`WHERE`**:

- Khi JOIN các Table, SQL sẽ dựa vào điều kiện kết nối để tạo ra một bảng tạm mới. `ON` chính là điều kiện kết nối, nó quyết定việc tạo ra bảng tạm.
- `WHERE` là sau khi bảng tạm đã được tạo ra, mới tiến hành lọc dữ liệu在bảng tạm đó để tạo ra tập kết quả cuối cùng, lúc này đã不còn JOIN-ON nữa.

Cho nên tóm lại là: **SQL đầu tiên dựa vào ON để tạo một bảng tạm, sau đó mới dựa vào WHERE để lọc bảng tạm方案**.

SQL cho phép thêm các từ khóa tu bổ在bên trái `JOIN`, từ đó tạo成các loại JOIN khác nhau như bảng dưới đây:

| 连接类型 | 说明 |
| ---------------------------------------- | --------------------------------------------------------------------------------------------- |
| INNER JOIN 内连接 | (Phương thức JOIN mặc定) Chỉ khi cả 2 Table đều tồn在bản ghi thỏa mãn điều kiện mới trả về dòng. |
| LEFT JOIN / LEFT OUTER JOIN 左(外)连接 | Trả về tất cả các dòng在Table bên trái, cho dù Table bên右不có dòng thỏa mãn điều kiện. |
| RIGHT JOIN / RIGHT OUTER JOIN 右(外)连接 | Trả về tất cả các dòng在Table bên右, cho dù Table bên trái不có dòng thỏa mãn điều kiện. |
| FULL JOIN / FULL OUTER JOIN 全(外)连接 | Chỉ cần một在các Table tồn在bản ghi thỏa mãn điều kiện là trả về dòng. |
| SELF JOIN | JOIN một Table与chính nó, giống như Table đó là 2 Table. Để分界面2 Table,在câu lệnh SQL cần đổi tên (Alias) cho ít nhất 1 Table. |
| CROSS JOIN | Cross Join, trả về Tích Descartes (Cartesian Product) của tập bản ghi từ 2 hoặc多Table kết连接. |

Hình dưới đây minh họa 7 cách dùng liên quan到LEFT JOIN, RIGHT JOIN, INNER JOIN, OUTER JOIN.

![](https://oss.javaguide.cn/p3-juejin/701670942f0f45d3a3a2187cd04a12ad~tplv-k3u1fbpfcp-zoom-1.png)

Nếu不thêm bất kỳ词tu bổ开, chỉ viết `JOIN`, mặc定sẽ là `INNER JOIN`

Đối与`INNER JOIN`, còn có một cách viết ẩn (implicit), gọi là "**Implicit Inner Join**", tức là不có词khóa `INNER JOIN`, sử dụng câu lệnh `WHERE` để thực hiện chức năng của Inner Join:

```sql
# Implicit Inner Join
select c.cust_name, o.order_num
from Customers c, Orders o
where c.cust_id = o.cust_id
order by c.cust_name;

# Explicit Inner Join
select c.cust_name, o.order_num
from Customers c inner join Orders o
using(cust_id)
order by c.cust_name;
```

## Hợp nhất tập kết quả (UNION)

Toán tử `UNION` kết hợp kết quả của 2或者多truy vấn lại与nhau和tạo ra một tập kết quả chứa各dòng trích xuất从các truy vấn tham gia到`UNION`.

Quy tắc cơ bản của `UNION`:

- Số lượng Column和thứ tự Column của tất cả各truy vấn必须giống nhau.
- Kiểu dữ liệu của các Column liên quan在mỗi truy vấn必须giống nhau或者tương thích.
- Thông常tên Column trả về được lấy从truy vấn đầu tiên.

Mặc定, toán tử `UNION` chọn各值duy nhất (loại bỏ trùng lặp). Nếu cho phép各值trùng lặp, hãy dùng `UNION ALL`.

```sql
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```

Tên Column在tập kết quả `UNION` luôn luôn bằng tên Column在câu lệnh `SELECT` đầu tiên của `UNION`.

So sánh `JOIN` vs `UNION`:

- Các Column của Table kết连接在`JOIN` có thể khác nhau,但在`UNION`, số量Column和thứ tự Column của tất cả各truy vấn必须giống nhau.
- `UNION` đặt各dòng sau khi truy vấn方案与nhau (đặt theo chiều dọc), còn `JOIN` đặt各Column sau khi truy vấn方案与nhau (đặt theo chiều ngang), tức là nó tạo成một tích Descartes.

## Hàm (Functions)

Hàm của各Database khác nhau常不giống nhau, do đó不có tính di动(portable). Chương này chủ yếu lấy各hàm在MySQL làm ví dụ.

### Xử lý văn bản (String Functions)

| 函数 | 说明 |
| -------------------- | ---------------------- |
| `LEFT()`、`RIGHT()` | Ký tự bên trái或者bên右 |
| `LOWER()`、`UPPER()` | Chuyển成chữ常或者chữ hoa |
| `LTRIM()`、`RTRIM()` | Loại bỏ khoảng trắng bên trái或者bên右 |
| `LENGTH()` | Độ dài, tính theo đơn vị byte |
| `SOUNDEX()` | Chuyển成值âm học (phonetic value) |

Trong đó, **`SOUNDEX()`** có thể chuyển một chuỗi ký tự成một pattern chữ和số mô tả biểu diễn phát âm của nó.

```sql
SELECT *
FROM mytable
WHERE SOUNDEX(col1) = SOUNDEX('apple')
```

### Xử lý Date与Time

- Định dạng Date: `YYYY-MM-DD`
- Định dạng Time: `HH:MM:SS`

| 函 数 | 说 明 |
| --------------- | ------------------------------ |
| `AddDate()` | Tăng thêm một date (ngày, tuần, v.v.) |
| `AddTime()` | Tăng thêm một time (giờ, phút, v.v.) |
| `CurDate()` | Trả về ngày hiện tại |
| `CurTime()` | Trả về giờ hiện tại |
| `Date()` | Trả về phần date của datetime |
| `DateDiff()` | Tính chênh lệch giữa 2 date |
| `Date_Add()` | Hàm tính toán date linh hoạt cao |
| `Date_Format()` | Trả về một chuỗi date或者time đã định dạng |
| `Day()` | Trả về phần số ngày của một date |
| `DayOfWeek()` | Trả về thứ在tuần tương ứng与một date |
| `Hour()` | Trả về phần số giờ của một time |
| `Minute()` | Trả về phần số phút của một time |
| `Month()` | Trả về phần số tháng của một date |
| `Now()` | Trả về date和time hiện tại |
| `Second()` | Trả về phần số giây của một time |
| `Time()` | Trả về phần time của datetime |
| `Year()` | Trả về phần số năm của một date |

### Xử lý số học (Numeric Functions)

| 函数 | 说明 |
| ------ | ------ |
| SIN() | Sin |
| COS() | Cos |
| TAN() | Tan |
| ABS() | Giá trị tuyệt đối |
| SQRT() | Căn bậc hai |
| MOD() | Phép chia lấy dư |
| EXP() | Hàm số mũ (Exponential) |
| PI() | Số Pi |
| RAND() | Số ngẫu nhiên |

### Aggregation (Hàm tổng hợp)

| 函 数 | 说 明 |
| --------- | ---------------- |
| `AVG()` | Trả về giá trị trung bình của cột |
| `COUNT()` | Trả về số dòng của cột |
| `MAX()` | Trả về giá trị lớn nhất của cột |
| `MIN()` | Trả về giá trị nhỏ nhất của cột |
| `SUM()` | Trả về tổng giá trị của cột |

`AVG()` sẽ bỏ qua各dòng NULL.

Sử dụng `DISTINCT` có thể làm cho hàm Aggregation chỉ tổng hợp各值duy nhất.

```sql
SELECT AVG(DISTINCT col1) AS avg_col
FROM mytable
```

**Tiếp theo, chúng ta cùng giới thiệu cách dùng câu lệnh DDL. Chức năng chính của DDL là định nghĩa各đối tượng Database (như Database, Table, View, Index, v.v.)**

## Định nghĩa dữ liệu (DDL)

### Cơ sở dữ liệu (DATABASE)

#### Tạo Database

```sql
CREATE DATABASE test;
```

#### Xóa Database

```sql
DROP DATABASE test;
```

#### Chọn (Chuyển sang) Database

```sql
USE test;
```

### Bảng dữ liệu (TABLE)

#### Tạo Table

**Tạo thông常**

```sql
CREATE TABLE user (
  id int(10) unsigned NOT NULL COMMENT 'Id',
  username varchar(64) NOT NULL DEFAULT 'default' COMMENT '用户名',
  password varchar(64) NOT NULL DEFAULT 'default' COMMENT '密码',
  email varchar(64) NOT NULL DEFAULT 'default' COMMENT '邮箱'
) COMMENT='用户表';
```

**Tạo bảng mới dựa在bảng đã有**

```sql
CREATE TABLE vip_user AS
SELECT * FROM user;
```

#### Xóa Table

```sql
DROP TABLE user;
```

#### Sửa đổi Table (ALTER TABLE)

**Thêm column**

```sql
ALTER TABLE user
ADD age int(3);
```

**Xóa column**

```sql
ALTER TABLE user
DROP COLUMN age;
```

**Sửa column**

```sql
ALTER TABLE `user`
MODIFY COLUMN age tinyint;
```

**Thêm Primary Key**

```sql
ALTER TABLE user
ADD PRIMARY KEY (id);
```

**Xóa Primary Key**

```sql
ALTER TABLE user
DROP PRIMARY KEY;
```

### View (Bảng ảo)

Định nghĩa:

- View là một bảng trực quan dựa在tập kết quả của câu lệnh SQL.
- View là bảng ảo, bản thân nó不chứa dữ liệu, do đó cũng不thể thực hiện thao tác Index在View. Thao tác在View giống hệt như thao tác在Table thông常.

Tác dụng:

- Đơn giản hóa各thao tác SQL phức tạp, ví dụ như各phép JOIN phức tạp;
- Chỉ sử dụng một phần dữ liệu của bảng thực tế;
- Bảo đảm tính an全của dữ liệu bằng cách chỉ cấp quyền truy cập View cho người dùng;
- Thay đổi định dạng和biểu diễn dữ liệu.

![mysql视图](https://oss.javaguide.cn/p3-juejin/ec4c975296ea4a7097879dac7c353878~tplv-k3u1fbpfcp-zoom-1.jpeg)

#### Tạo View

```sql
CREATE VIEW top_10_user_view AS
SELECT id, username
FROM user
WHERE id < 10;
```

#### Xóa View

```sql
DROP VIEW top_10_user_view;
```

### Index (Chỉ mục)

**Index là một cấu trúc dữ liệu dùng để truy vấn和tìm kiếm dữ liệu nhanh chóng, bản chất của nó có thể xem là một cấu trúc dữ liệu đã được sắp xếp.**

Tác dụng của Index tương tự như mục lục của một cuốn sách. Ví dụ: Khi chúng ta tra từ điển, nếu不có mục lục则chúng ta chỉ有thể lật từng trang một để tìm từ cần tra, tốc độ rất chậm. Nếu có mục lục已, chúng ta chỉ cần vào mục lục tìm vị trí của từ đó trước, sau đó lật thẳng到trang đó là xong.

**Ưu điểm**:

- Sử dụng Index có thể tăng tốc đáng kể tốc độ tìm kiếm dữ liệu (giảm đáng kể lượng dữ liệu cần quét), đây cũng là nguyên nhân chính yếu để tạo Index.
- Bằng cách tạo Unique Index, có thể đảm bảo tính duy nhất của từng dòng dữ liệu在Table.

**Nhược điểm**:

- Tạo Index和bảo trì Index tốn nhiều时间. Khi thực hiện Thêm, Xóa, Sửa dữ liệu在Table, nếu dữ liệu有Index则Index cũng cần sửa đổi动, làm giảm hiệu suất thực thi SQL.
- Index cần lưu trữ bằng file物理, cũng sẽ tiêu tốn dung量bộ nhớ nhất定.

Tuy nhiên, **sử dụng Index có chắc chắn nâng cao hiệu suất truy vấn不?**

Trong đại đa số trường hợp, truy vấn bằng Index đều nhanh hơn Full Table Scan. Tuy nhiên nếu dung量dữ liệu của Database不lớn则sử dụng Index cũng chưa chắc mang lại sự cải thiện lớn.

Về giới thiệu chi tiết của Index, xin xem bài viết [Chi tiết MySQL Index](https://javaguide.cn/database/mysql/mysql-index.html) do tôi viết.

#### Tạo Index

```sql
CREATE INDEX user_index
ON user (id);
```

#### Thêm Index

```sql
ALTER table user ADD INDEX user_index(id)
```

#### Tạo Unique Index

```sql
CREATE UNIQUE INDEX user_index
ON user (id);
```

#### Xóa Index

```sql
ALTER TABLE user
DROP INDEX user_index;
```

### Ràng buộc (Constraint)

SQL Constraint dùng để quy定các quy tắc dữ liệu在Table.

Nếu tồn在hành vi dữ liệu vi phạm Constraint, hành vi đó sẽ bị Constraint chặn ngắt.

Constraint có thể quy定khi tạo Table (qua câu lệnh CREATE TABLE), hoặc quy定sau khi Table已经được tạo (qua câu lệnh ALTER TABLE).

Các loại Constraint:

- `NOT NULL` - Chỉ定một Column不được lưu trữ值NULL.
- `UNIQUE` - Đảm bảo mỗi dòng của một Column必须có值duy nhất.
- `PRIMARY KEY` - Sự kết hợp giữa NOT NULL和UNIQUE. Đảm bảo một Column (hoặc sự kết hợp nhiều Column) có định danh duy nhất, giúp dễ dàng和nhanh chóng tìm thấy một bản ghi cụ thể在Table.
- `FOREIGN KEY` - Đảm bảo tính toàn vẹn tham chiếu (referential integrity) giữa dữ liệu在một Table和值在Table khác.
- `CHECK` - Đảm bảo值在Column phù hợp与điều kiện chỉ定.
- `DEFAULT` - Quy定值mặc定khi不gán值cho Column.

Sử dụng Constraint khi tạo Table:

```sql
CREATE TABLE Users (
  Id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '自增Id',
  Username VARCHAR(64) NOT NULL UNIQUE DEFAULT 'default' COMMENT '用户名',
  Password VARCHAR(64) NOT NULL DEFAULT 'default' COMMENT '密码',
  Email VARCHAR(64) NOT NULL DEFAULT 'default' COMMENT '邮箱地址',
  Enabled TINYINT(4) DEFAULT NULL COMMENT '是否有效',
  PRIMARY KEY (Id)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

**Tiếp theo, chúng ta cùng giới thiệu cách dùng câu lệnh TCL. Chức năng chính của TCL là quản lý Transaction在Database.**

## Quản lý Transaction

Không thể Rollback câu lệnh `SELECT`,和Rollback câu lệnh `SELECT` cũng不có ý nghĩa; Cũng不thể Rollback各câu lệnh `CREATE`和`DROP`.

**MySQL mặc定là Implicit Commit (Autocommit)**, mỗi khi thực thi 1 câu lệnh sẽ coi câu lệnh đó là 1 Transaction然后Commit. Khi xuất hiện câu lệnh `START TRANSACTION`, nó sẽ tắt Implicit Commit; Khi câu lệnh `COMMIT`或`ROLLBACK` thực thi xong, Transaction sẽ tự动đóng和khôi phục lại Implicit Commit.

Thông qua `SET autocommit=0` có thể hủy tự动Commit, cho到khi `SET autocommit=1` mới Commit; Cờ `autocommit` nhắm vào từng Connection cụ thể chứ不nhắm vào Server.

Các lệnh:

- `START TRANSACTION` - Lệnh dùng để đánh dấu điểm bắt đầu của Transaction.
- `SAVEPOINT` - Lệnh dùng để tạo Savepoint (điểm lưu).
- `ROLLBACK TO` - Lệnh dùng để Rollback về Savepoint chỉ定; Nếu不thiết lập Savepoint, nó sẽ Rollback về vị trí câu lệnh `START TRANSACTION`.
- `COMMIT` - Commit (Xác nhận) Transaction.

```sql
-- Bắt đầu Transaction
START TRANSACTION;

-- Thao tác INSERT A
INSERT INTO `user`
VALUES (1, 'root1', 'root1', 'xxxx@163.com');

-- Tạo Savepoint updateA
SAVEPOINT updateA;

-- Thao tác INSERT B
INSERT INTO `user`
VALUES (2, 'root2', 'root2', 'xxxx@163.com');

-- Rollback về Savepoint updateA
ROLLBACK TO updateA;

-- Commit Transaction, chỉ có thao tác A có hiệu lực
COMMIT;
```

**Tiếp theo, chúng ta cùng giới thiệu cách dùng câu lệnh DCL. Chức năng chính của DCL là kiểm soát quyền truy cập của người dùng.**

## Kiểm soát phân quyền (DCL)

Để cấp quyền cho tài khoản người dùng, có thể dùng lệnh `GRANT`. Để thu hồi quyền của người dùng, có thể dùng lệnh `REVOKE`. Ở đây lấy MySQL làm ví dụ để giới thiệu ứng dụng thực tế của kiểm soát phân quyền.

Cú pháp cấp quyền `GRANT`:

```sql
GRANT privilege,[privilege],.. ON privilege_level
TO user [IDENTIFIED BY password]
[REQUIRE tsl_option]
[WITH [GRANT_OPTION | resource_option]];
```

Giải thích ngắn gọn:

1. Phía sau từ khóa `GRANT` chỉ定một或者多quyền. Nếu cấp多quyền cho người dùng, mỗi quyền được ngăn cách bằng dấu phẩy.
2. `ON privilege_level` xác定cấp độ áp dụng quyền. MySQL hỗ trợ Global (`*.*`), Database (`database.*`), Table (`database.table`)和cấp độ Column. Nếu sử dụng cấp độ quyền Column,必须chỉ定danh sách Column ngăn cách bằng dấu phẩy phía sau mỗi quyền.
3. `user` là người dùng cần cấp quyền. Nếu người dùng已经tồn在, câu lệnh `GRANT` sẽ sửa đổi quyền của họ. Ngược lại, câu lệnh `GRANT` sẽ tạo một người dùng mới. Mệnh đề tùy chọn `IDENTIFIED BY` cho phép bạn đặt mật khẩu mới cho người dùng.
4. `REQUIRE tsl_option` chỉ定người dùng có bắt buộc必须kết连接与Database Server qua kết连接an全như SSL, X509 hay不.
5. Mệnh đề tùy chọn `WITH GRANT OPTION` cho phép bạn cấp quyền cho người dùng khác或者xóa各quyền mà bạn sở hữu从người dùng khác. Ngoài ra, bạn có thể dùng mệnh đề `WITH` để phân bổ tài nguyên của MySQL Database Server, ví dụ thiết lập số量Connection或者số câu lệnh người dùng có thể dùng mỗi giờ. Điều này rất hữu ích在môi trường chia sẻ như MySQL Shared Hosting.

Cú pháp thu hồi quyền `REVOKE`:

```sql
REVOKE   privilege_type [(column_list)]
        [, priv_type [(column_list)]]...
ON [object_type] privilege_level
FROM user [, user]...
```

Giải thích ngắn gọn:

1. Phía sau từ khóa `REVOKE` chỉ定danh sách各quyền cần thu hồi从người dùng. Bạn cần ngăn cách各quyền bằng dấu phẩy.
2. Chỉ定cấp độ quyền cần thu hồi在mệnh đề `ON`.
3. Chỉ定tài khoản người dùng cần thu hồi quyền在mệnh đề `FROM`.

`GRANT`和`REVOKE` có thể kiểm soát quyền truy cập在nhiều cấp độ:

- Toàn bộ Server, sử dụng `GRANT ALL`和`REVOKE ALL`;
- Toàn bộ Database, sử dụng `ON database.*`;
- Table cụ thể, sử dụng `ON database.table`;
- Column cụ thể;
- Stored Procedure cụ thể.

Tài khoản mới tạo不có bất kỳ quyền nào. Tài khoản được định nghĩa dưới dạng `username@host`, `username@%` sử dụng Hostname mặc定. Thông tin tài khoản của MySQL được lưu giữ在Database `mysql`.

```sql
USE mysql;
SELECT user FROM user;
```

Bảng dưới đây giải thích tất cả各quyền được phép dùng在câu lệnh `GRANT`与`REVOKE`:

| Privileges (Quyền) | Description (Mô tả) | Level (Cấp độ) | Global | Database | Table | Column | Procedure | Proxy |
| ----------------------- | ------------------------------------------------------------------------------------------------------- | -------- | ------ | -------- | -------- | --- | --- |
| ALL [PRIVILEGES] | Cấp tất cả các quyền ở cấp độ truy cập chỉ định (ngoại trừ GRANT OPTION) | | | | | | |
| ALTER | Cho phép người dùng sử dụng câu lệnh ALTER TABLE | X | X | X | | | |
| ALTER ROUTINE | Cho phép người dùng sửa hoặc xóa Stored Routine (Procedure/Function) | X | X | | | X | |
| CREATE | Cho phép người dùng tạo Database và Table | X | X | X | | | |
| CREATE ROUTINE | Cho phép người dùng tạo Stored Routine | X | X | | | | |
| CREATE TABLESPACE | Cho phép người dùng tạo, sửa hoặc xóa Tablespace và Log File Group | X | | | | | |
| CREATE TEMPORARY TABLES | Cho phép người dùng dùng CREATE TEMPORARY TABLE để tạo bảng tạm | X | X | | | | |
| CREATE USER | Cho phép người dùng dùng các câu lệnh CREATE USER, DROP USER, RENAME USER và REVOKE ALL PRIVILEGES. | X | | | | | |
| CREATE VIEW | Cho phép người dùng tạo hoặc sửa View. | X | X | X | | | |
| DELETE | Cho phép người dùng sử dụng DELETE | X | X | X | | | |
| DROP | Cho phép người dùng xóa Database, Table và View | X | X | X | | | |
| EVENT | Bật tính năng sử dụng Event của Event Scheduler. | X | X | | | | |
| EXECUTE | Cho phép người dùng thực thi Stored Routine | X | X | X | | | |
| FILE | Cho phép người dùng đọc bất kỳ file nào trong thư mục Database. | X | | | | | |
| GRANT OPTION | Cho phép người dùng có quyền cấp hoặc thu hồi quyền của tài khoản khác. | X | X | X | | X | X |
| INDEX | Cho phép người dùng tạo hoặc xóa Index. | X | X | X | | | |
| INSERT | Cho phép người dùng sử dụng câu lệnh INSERT | X | X | X | X | | |
| LOCK TABLES | Cho phép người dùng dùng LOCK TABLES trên các Table có quyền SELECT | X | X | | | | |
| PROCESS | Cho phép người dùng dùng câu lệnh SHOW PROCESSLIST để xem tất cả Process. | X | | | | | |
| PROXY | Bật Proxy người dùng. | | | | | | |
| REFERENCES | Cho phép người dùng tạo Foreign Key | X | X | X | X | | |
| RELOAD | Cho phép người dùng sử dụng các thao tác FLUSH | X | | | | | |
| REPLICATION CLIENT | Cho phép người dùng truy vấn vị trí của Master hoặc Slave Server | X | | | | | |
| REPLICATION SLAVE | Cho phép người dùng dùng Replication Slave để đọc Binary Log Event từ Master Server. | X | | | | | |
| SELECT | Cho phép người dùng sử dụng câu lệnh SELECT | X | X | X | X | | |
| SHOW DATABASES | Cho phép người dùng hiển thị tất cả Database | X | | | | | |
| SHOW VIEW | Cho phép người dùng sử dụng câu lệnh SHOW CREATE VIEW | X | X | X | | | |
| SHUTDOWN | Cho phép người dùng sử dụng lệnh mysqladmin shutdown | X | | | | | |
| SUPER | Cho phép người dùng sử dụng các thao tác quản trị khác như CHANGE MASTER TO, KILL, PURGE BINARY LOGS, SET GLOBAL và các lệnh mysqladmin | X | | | | | |
| TRIGGER | Cho phép người dùng sử dụng các thao tác TRIGGER. | X | X | X | | | |
| UPDATE | Cho phép người dùng sử dụng câu lệnh UPDATE | X | X | X | X | | |
| USAGE | Tương đương với "không có quyền" | | | | | | |

### Tạo tài khoản

```sql
CREATE USER myuser IDENTIFIED BY 'mypassword';
```

### Sửa tên tài khoản

```sql
UPDATE user SET user='newuser' WHERE user='myuser';
FLUSH PRIVILEGES;
```

### Xóa tài khoản

```sql
DROP USER myuser;
```

### Xem quyền

```sql
SHOW GRANTS FOR myuser;
```

### Cấp quyền

```sql
GRANT SELECT, INSERT ON *.* TO myuser;
```

### Thu hồi (Xóa) quyền

```sql
REVOKE SELECT, INSERT ON *.* FROM myuser;
```

### Đổi mật khẩu

```sql
SET PASSWORD FOR myuser = 'mypass';
```

## Stored Procedure (Thủ tục lưu trữ)

Stored Procedure có thể xem là một xử lý lô (batch processing) cho một chuỗi các thao tác SQL. Stored Procedure có thể được gọi bởi Trigger, Stored Procedure khác cũng như các ứng dụng Java, Python, PHP, v.v.

![mysql存储过程](https://oss.javaguide.cn/p3-juejin/60afdc9c9a594f079727ec64a2e698a3~tplv-k3u1fbpfcp-zoom-1.jpeg)

Lợi ích khi sử dụng Stored Procedure:

- Đóng gói code, bảo đảm tính an全nhất定;
- Tái sử dụng code;
- Do được biên dịch trước (Pre-compiled), do đó có hiệu năng rất cao.

Tạo Stored Procedure:

- Tạo Stored Procedure在Command Line (CLI) cần tự định nghĩa dấu分界面 (Delimiter), vì CLI dùng `;` làm dấu kết thúc câu lệnh, mà bên在Stored Procedure cũng chứa dấu chấm phẩy, do đó sẽ nhầm lẫn coi phần dấu chấm phẩy đó là dấu kết thúc gây ra lỗi cú pháp.
- Bao gồm 3 loại tham số: `IN`, `OUT`和`INOUT`.
- Gán值cho biến đều cần dùng câu lệnh `SELECT INTO`.
- Mỗi lần chỉ有thể gán值cho 1 biến,不hỗ trợ thao tác在Collection.

Cần lưu意rằng: **"Quy chuẩn phát triển Java của Alibaba" BẮT BUỘC NGHIÊM CẤM sử dụng Stored Procedure. Bởi vì Stored Procedure rất khó debug和mở rộng, lại càng不có tính di动(portability).**

![](https://oss.javaguide.cn/p3-juejin/93a5e011ade4450ebfa5d82057532a49~tplv-k3u1fbpfcp-zoom-1.png)

Còn về việc có nên sử dụng在dự án hay不vẫn phải xem nhu cầu thực tế của dự án, cân nhắc kỹ lợi ích和tác hại là được!

### Tạo Stored Procedure

```sql
DROP PROCEDURE IF EXISTS `proc_adder`;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_adder`(IN a int, IN b int, OUT sum int)
BEGIN
    DECLARE c int;
    if a is null then set a = 0;
    end if;

    if b is null then set b = 0;
    end if;

    set sum  = a + b;
END
;;
DELIMITER ;
```

### Sử dụng Stored Procedure

```less
set @b=5;
call proc_adder(2,@b,@s);
select @s as sum;
```

## Cursor (Con trỏ)

Cursor (Con trỏ) là một truy vấn Database được lưu trữ在DBMS Server, nó不phải là một câu lệnh `SELECT`, mà là tập kết quả được truy xuất ra bởi câu lệnh đó.

Sử dụng Cursor在Stored Procedure có thể di转duyệt (traverse) từng dòng qua một tập kết quả.

Cursor chủ yếu dùng在các ứng dụng tương tác,在đó người dùng cần cuộn màn hình在dữ liệu,和duyệt或者chỉnh sửa dữ liệu.

Các bước rõ ràng khi sử dụng Cursor:

- Trước khi sử dụng Cursor, bắt buộc必须khai báo (định nghĩa) nó. Quá trình này thực tế未truy xuất dữ liệu, nó chỉ định nghĩa câu lệnh `SELECT`和các tùy chọn Cursor sẽ dùng.

- Một khi đã khai báo, bắt buộc必须mở (OPEN) Cursor để sử dụng. Quá trình này sẽ dùng câu lệnh SELECT đã định nghĩa trước đó để truy xuất dữ liệu thực tế ra.

- Đối与Cursor đã đổ đầy dữ liệu, lấy (FETCH) từng dòng ra theo nhu cầu.

- Khi kết thúc sử dụng Cursor, bắt buộc必须đóng (CLOSE) Cursor, nếu có thể则giải phóng Cursor (tùy thuộc vào DBMS cụ thể).

```sql
DELIMITER $
CREATE  PROCEDURE getTotal()
BEGIN
    DECLARE total INT;
    -- Tạo các biến nhận dữ liệu Cursor
    DECLARE sid INT;
    DECLARE sname VARCHAR(10);
    -- Tạo biến tổng số
    DECLARE sage INT;
    -- Tạo biến cờ báo kết thúc
    DECLARE done INT DEFAULT false;
    -- Tạo Cursor
    DECLARE cur CURSOR FOR SELECT id,name,age from cursor_table where age>30;
    -- Chỉ định giá trị trả về khi vòng lặp Cursor kết thúc
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;
    SET total = 0;
    OPEN cur;
    FETCH cur INTO sid, sname, sage;
    WHILE(NOT done)
    DO
        SET total = total + 1;
        FETCH cur INTO sid, sname, sage;
    END WHILE;

    CLOSE cur;
    SELECT total;
END $
DELIMITER ;

-- Gọi Stored Procedure
call getTotal();
```

## Trigger (Bộ kích hoạt)

Trigger là một đối tượng Database liên quan到thao tác在Table, khi在Table chứa Trigger xuất hiện sự kiện chỉ定则đối tượng đó sẽ được gọi, tức là sự kiện thao tác Table sẽ kích hoạt việc thực thi của Trigger在Table đó.

Chúng ta có thể dùng Trigger để thực hiện Audit Trail (theo dõi kiểm toán), ghi lại各thay đổi vào một Table khác.

Ưu điểm khi sử dụng Trigger:

- SQL Trigger cung cấp một phương pháp khác để kiểm tra tính toàn vẹn dữ liệu.
- SQL Trigger có thể bắt各lỗi logic nghiệp vụ ở tầng Database.
- SQL Trigger cung cấp một phương pháp khác để chạy Scheduled Task. Bằng cách dùng SQL Trigger, bạn不cần phải chờ chạy Scheduled Task, vì trước或者sau khi sửa đổi dữ liệu在Table, Trigger sẽ tự动được gọi.
- SQL Trigger rất hữu ích cho việc kiểm toán各thay đổi dữ liệu在Table.

Nhược điểm khi sử dụng Trigger:

- SQL Trigger chỉ有thể cung cấp xác thực mở rộng和不thể thay thế toàn bộ xác thực. Bắt buộc必须hoàn成một số xác thực đơn giản ở tầng Application. Ví dụ, bạn có thể dùng JavaScript để validate input của người dùng ở Client,或者dùng ngôn ngữ script ở Server (như JSP, PHP, ASP.NET, Perl) để validate input ở Server.
- Việc gọi和thực thi SQL Trigger từ ứng dụng Client là不nhìn thấy được (invisible), do đó rất khó biết chuyện gì đang xảy ra ở tầng Database.
- SQL Trigger có thể làm tăng overhead (chi phí tài nguyên) cho Database Server.

MySQL不cho phép sử dụng câu lệnh CALL在Trigger, tức là不thể gọi Stored Procedure.

> Lưu ý: Trong MySQL, dấu chấm phẩy `;` là ký tự định danh kết thúc câu lệnh, khi gặp dấu chấm phẩy nghĩa là đoạn câu lệnh đó đã kết thúc, MySQL có thể bắt đầu thực thi. Do đó, trình thông dịch gặp dấu chấm phẩy trong hành động thực thi của Trigger liền bắt đầu thực thi, sau đó sẽ báo lỗi vì chưa tìm thấy END tương ứng với BEGIN.
>
> Lúc này sẽ cần dùng đến lệnh `DELIMITER` (DELIMITER có nghĩa là ký tự phân cách / định giới). Nó là một câu lệnh, không cần ký tự kết thúc, cú pháp là: `DELIMITER new_delimiter`. `new_delimiter` có thể đặt thành biểu tượng có độ dài 1 hoặc nhiều ký tự, mặc định là dấu chấm phẩy `;`, chúng ta có thể sửa nó thành biểu tượng khác, như `$` - `DELIMITER $`. Các câu lệnh sau đó kết thúc bằng dấu chấm phẩy trình thông dịch sẽ không phản ứng gì, chỉ khi gặp `$` mới coi là kết thúc câu lệnh. Lưu ý, sau khi dùng xong chúng ta cũng nên nhớ đổi nó về lại như cũ.

Trước phiên bản MySQL 5.7.2, có thể định nghĩa tối đa 6 Trigger cho mỗi Table.

- `BEFORE INSERT` - Kích hoạt trước khi INSERT dữ liệu vào Table.
- `AFTER INSERT` - Kích hoạt sau khi INSERT dữ liệu vào Table.
- `BEFORE UPDATE` - Kích hoạt trước khi UPDATE dữ liệu trong Table.
- `AFTER UPDATE` - Kích hoạt sau khi UPDATE dữ liệu trong Table.
- `BEFORE DELETE` - Kích hoạt trước khi DELETE dữ liệu khỏi Table.
- `AFTER DELETE` - Kích hoạt sau khi DELETE dữ liệu khỏi Table.

Tuy nhiên, từ phiên bản MySQL 5.7.2+ trở đi, có thể định nghĩa nhiều Trigger cho cùng một sự kiện和thời điểm kích hoạt.

**`NEW`与`OLD`**:

- Trong MySQL định nghĩa từ khóa `NEW`与`OLD`, dùng để biểu thị dòng dữ liệu在Table đã kích hoạt Trigger.
- Trong Trigger loại `INSERT`, `NEW` dùng để biểu thị dữ liệu mới sắp sửa (`BEFORE`)或者đã (`AFTER`) được INSERT;
- Trong Trigger loại `UPDATE`, `OLD` dùng để biểu thị dữ liệu cũ sắp sửa或者đã bị sửa, `NEW` dùng để biểu thị dữ liệu mới sắp sửa或者đã được sửa成;
- Trong Trigger loại `DELETE`, `OLD` dùng để biểu thị dữ liệu cũ sắp sửa或者đã bị xóa;
- Cách dùng: `NEW.columnName` (`columnName` là tên 1 column của Table tương ứng)

### Tạo Trigger

> Gợi意: Để hiểu各điểm mấu chốt của Trigger, cần thiết phải tìm hiểu lệnh tạo Trigger trước.

Lệnh `CREATE TRIGGER` dùng để tạo Trigger.

Cú pháp:

```sql
CREATE TRIGGER trigger_name
trigger_time
trigger_event
ON table_name
FOR EACH ROW
BEGIN
  trigger_statements
END;
```

Giải thích:

- `trigger_name`: Tên Trigger
- `trigger_time`: Thời điểm kích hoạt của Trigger. Giá trị là `BEFORE`或`AFTER`.
- `trigger_event`: Sự kiện lắng nghe của Trigger. Giá trị là `INSERT`, `UPDATE`或`DELETE`.
- `table_name`: Mục tiêu lắng nghe của Trigger. Chỉ定tạo Trigger在Table nào.
- `FOR EACH ROW`: Lắng nghe ở cấp độ dòng (Row-level), cách viết cố定在MySQL,各DBMS khác có thể khác.
- `trigger_statements`: Hành动thực thi của Trigger. Là danh sách một或者多câu lệnh SQL, mỗi câu lệnh在danh sách bắt buộc必须kết thúc bằng dấu chấm phẩy `;`.

Khi điều kiện kích hoạt của Trigger thỏa mãn, hành动thực thi nằm giữa `BEGIN`和`END` sẽ được thực thi.

Ví dụ:

```sql
DELIMITER $
CREATE TRIGGER `trigger_insert_user`
AFTER INSERT ON `user`
FOR EACH ROW
BEGIN
    INSERT INTO `user_history`(user_id, operate_type, operate_time)
    VALUES (NEW.id, 'add a user',  now());
END $
DELIMITER ;
```

### Xem Trigger

```sql
SHOW TRIGGERS;
```

### Xóa Trigger

```sql
DROP TRIGGER IF EXISTS trigger_insert_user;
```

## Bài viết đề xuất

- [Bắt buộc cho lập trình viên Backend: Hướng dẫn tối ưu hóa SQL hiệu năng cao! 35+ lời khuyên tối ưu GET ngay!](https://mp.weixin.qq.com/s/I-ZT3zGTNBZ6egS7T09jyQ)
- [Bắt buộc cho lập trình viên Backend: 30 lời khuyên viết SQL chất lượng cao](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486461&idx=1&sn=60a22279196d084cc398936fe3b37772&chksm=cea24436f9d5cd20a4fa0e907590f3e700d7378b3f608d7b33bb52cfb96f503b7ccb65a1deed&token=1987003517&lang=zh_CN#rd)

<!-- @include: @article-footer.snippet.md -->
