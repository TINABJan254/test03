---
title: SQL语句在MySQL中的执行过程
description: 详解SQL语句在MySQL中的完整执行流程，从连接器身份认证、查询缓存、分析器语法解析、优化器生成执行计划到执行器调用存储引擎的全过程。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL执行流程,SQL执行过程,连接器,解析器,优化器,执行器,Server层,存储引擎,InnoDB
---

> Bài viết đến từ đóng góp của [木木匠 (Mu Mu Jiang)](https://github.com/kinglaw1204).

Bài viết này sẽ phân tích quy trình thực thi một câu lệnh SQL trong MySQL, bao gồm việc truy vấn SQL được luôn chuyển bên trong MySQL như thế nào, và câu lệnh SQL update được hoàn thành ra sao.

Trước khi phân tích, tôi sẽ dẫn bạn xem qua kiến trúc cơ bản của MySQL. Việc nắm rõ MySQL được cấu thành từ những component nào và tác dụng của chúng là gì sẽ giúp chúng ta dễ dàng hiểu và giải quyết các vấn đề này.

## I Phân tích kiến trúc cơ bản của MySQL

### 1.1 Tổng quan kiến trúc cơ bản MySQL

Sơ đồ dưới đây là sơ đồ kiến trúc tóm tắt của MySQL, từ sơ đồ bạn có thể thấy rất rõ câu lệnh SQL của người dùng được thực thi bên trong MySQL như thế nào:

![](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

Tóm tắt tác dụng của các component trong sơ đồ:

- **Connector (Bộ kết nối):** Xác thực danh tính và phân quyền (khi đăng nhập MySQL).
- **Query Cache (Bộ nhớ tạm truy vấn):** Khi thực thi câu lệnh query, sẽ kiểm tra cache trước (đã loại bỏ từ MySQL 8.0 do tính thực tế không cao).
- **Analyzer (Bộ phân tích):** Nếu không trúng cache, câu SQL sẽ đi qua Analyzer để xem câu SQL muốn làm gì và kiểm tra syntax có đúng hay không.
- **Optimizer (Bộ tối ưu hóa):** Thực thi theo phương án mà MySQL cho là tối ưu nhất.
- **Executor (Bộ thực thi):** Thực thi câu lệnh, sau đó nhận dữ liệu trả về từ Storage Engine.

Nói một cách đơn giản, MySQL chủ yếu chia thành Server layer và Storage Engine layer:

- **Server layer**: Bao gồm Connector, Query Cache, Analyzer, Optimizer, Executor... Tất cả các tính năng xuyên suốt các Storage Engine đều được thực thi ở tầng này như Stored Procedure, Trigger, View, Function..., và có một module log chung là `binlog`.
- **Storage Engine layer**: Chịu trách nhiệm lưu trữ và đọc dữ liệu, sử dụng kiến trúc cắm rút có thể thay thế, hỗ trợ nhiều Storage Engine như InnoDB, MyISAM, Memory... Trong đó engine InnoDB có module log riêng là `redo log`. **Storage Engine phổ biến nhất hiện nay là InnoDB, nó đã trở thành Storage Engine mặc định từ phiên bản MySQL 5.5.**

### 1.2 Giới thiệu các component cơ bản của Server layer

#### 1) Connector

Connector liên quan đến xác thực danh tính và phân quyền, giống như một người gác cổng cấp cao.

Chủ yếu chịu trách nhiệm cho người dùng đăng nhập cơ sở dữ liệu, tiến hành xác thực danh tính bao gồm kiểm tra tài khoản, mật khẩu, phân quyền... Sau khi tài khoản và mật khẩu thông qua, Connector sẽ tra cứu tất cả quyền hạn của user đó trong bảng phân quyền. Về sau việc kiểm tra logic phân quyền trong kết nối này sẽ phụ thuộc vào dữ liệu quyền hạn đọc được lúc này. Nghĩa là chỉ cần kết nối này chưa ngắt, dù admin có sửa quyền của user đó thì user hiện tại cũng không bị ảnh hưởng.

#### 2) Query Cache (Loại bỏ từ phiên bản MySQL 8.0)

Query Cache chủ yếu dùng để cache các câu lệnh SELECT đã thực thi cùng tập kết quả tương ứng.

Sau khi kết nối được thiết lập, khi thực thi câu SELECT, trước tiên sẽ kiểm tra Query Cache dưới dạng Key-Value trong RAM (Key là câu SQL, Value là tập kết quả). Nếu trúng Key, trực tiếp trả về cho client. Nếu không trúng, mới thực hiện các thao tác tiếp theo và sau khi xong cũng lưu kết quả vào cache.

Tuy nhiên không khuyến nghị dùng Query Cache vì trong kịch bản thực tế cache bị vô hiệu hóa rất thường xuyên (chỉ cần update 1 bảng thì tất cả Query Cache trên bảng đó đều bị xóa sạch).

Từ MySQL 8.0, tính năng Query Cache đã bị loại bỏ hoàn toàn.

#### 3) Analyzer

Nếu không trúng Query Cache, câu SQL sẽ đi vào Analyzer. Analyzer dùng để phân tích xem câu SQL muốn làm gì:

- **Bước 1: Phân tích từ vựng (Lexical Analysis)**: Một câu SQL gồm nhiều chuỗi ký tự, trước tiên phải bóc tách các từ khóa như `select`, bóc tách bảng cần query, bóc tách tên field, điều kiện query...
- **Bước 2: Phân tích cú pháp (Syntax Analysis)**: Chủ yếu phán đoán xem câu SQL bạn nhập có đúng syntax của MySQL hay không.

#### 4) Optimizer

Tác dụng của Optimizer là thực thi theo phương án mà nó cho là tối ưu nhất (chọn Index nào trong số nhiều Index, thứ tự JOIN giữa các bảng...).

#### 5) Executor

Khi đã chốt phương án thực thi, MySQL chuẩn bị bắt đầu thực thi. Trước tiên kiểm tra xem user có quyền truy cập hay không, nếu không có quyền sẽ trả về lỗi, nếu có quyền sẽ gọi interface của Storage Engine và nhận kết quả trả về từ Storage Engine.

## II Phân tích câu lệnh

### 2.1 Câu lệnh truy vấn (SELECT)

Một câu lệnh SQL query được thực thi như thế nào? Ví dụ câu lệnh:

```sql
select * from tb_student  A where A.age='18' and A.name=' 张三 ';
```

Quy trình thực thi:

- Xác thực danh tính và lấy quyền hạn qua Connector. (Trước MySQL 8.0 sẽ kiểm tra Query Cache trước).
- Analyzer phân tích từ vựng và cú pháp, bóc tách câu lệnh là query `select`, bảng là `tb_student`, lấy tất cả các cột, điều kiện `age='18'` và `name='张三'`. Kiểm tra xem có lỗi syntax hay không.
- Optimizer chốt phương án thực thi tối ưu (ví dụ chọn lọc theo age trước hay name trước).
- Kiểm tra quyền hạn -> Executor gọi interface của Storage Engine -> nhận kết quả trả về từ Storage Engine.

### 2.2 Câu lệnh cập nhật (UPDATE)

Ví dụ câu UPDATE:

```sql
update tb_student A set A.age='19' where A.name=' 张三 ';
```

Quy trình thực thi câu UPDATE dưới Storage Engine InnoDB:

- Query lấy bản ghi của "Trương Tam" (không qua Query Cache).
- Nhận dữ liệu, sửa `age` thành 19, gọi API của Storage Engine để ghi hàng dữ liệu này. Storage Engine InnoDB lưu dữ liệu trong bộ nhớ (`Buffer Pool`), đồng thời ghi vào `redo log` ở trạng thái `prepare`, sau đó báo cho Executor là đã thực thi xong, sẵn sàng commit.
- Executor nhận thông báo và ghi `binlog`, đồng thời làm mới cache của bảng này.
- Executor gọi interface của Storage Engine, commit `redo log` sang trạng thái `commit`.
- Cập nhật hoàn tất.

**Tại sao phải dùng cả 2 module log (redo log và binlog)?**

Vì ban đầu MySQL dùng MyISAM làm engine mặc định (không có redo log và không có khả năng Crash-Safe). InnoDB sau này được đưa vào dưới dạng plugin và dùng `redo log` để hỗ trợ Transaction và Crash-Safe.

**Tại sao redo log phải áp dụng Two-Phase Commit (Cam kết 2 giai đoạn)?**

- Nếu ghi `redo log` xong sập máy trước khi ghi `binlog`: khi restart máy dùng `redo log` khôi phục dữ liệu nhưng `binlog` không có bản ghi này, dẫn đến sai lệch dữ liệu khi backup hoặc Master-Slave Replication.
- Nếu ghi `binlog` xong sập máy trước khi ghi `redo log`: máy không có `redo log` nên không khôi phục được bản ghi, nhưng `binlog` lại có, dẫn đến sai lệch dữ liệu.

Cơ chế Two-Phase Commit (chuẩn bị `prepare` -> ghi `binlog` -> cam kết `commit`) giúp đảm bảo tính nhất quán dữ liệu giữa 2 log.

## III Tóm tắt

- MySQL chia làm Server layer (Connector, Query Cache, Analyzer, Optimizer, Executor, binlog) và Storage Engine layer (InnoDB với redo log, MyISAM...).
- Quy trình SELECT: Kiểm tra quyền -> Query Cache -> Analyzer -> Optimizer -> Kiểm tra quyền -> Executor -> Storage Engine.
- Quy trình UPDATE: Analyzer -> Kiểm tra quyền -> Executor -> Storage Engine -> redo log (prepare) -> binlog -> redo log (commit).

## IV Tham khảo

- 《MySQL 实战 45 讲》
- MySQL 5.6 参考手册:<https://dev.MySQL.com/doc/refman/5.6/en/>

<!-- @include: @article-footer.snippet.md -->
