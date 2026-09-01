---
title: MySQL事务隔离级别详解
description: 详解MySQL四种事务隔离级别（读未提交、读已提交、可重复读、串行化）的特点与区别，分析脏读、不可重复读、幻读等并发问题，以及InnoDB如何通过MVCC和锁机制解决幻读。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL事务隔离级别,读未提交,读已提交,可重复读,串行化,脏读,不可重复读,幻读,MVCC,间隙锁
---

> Bài viết này do [SnailClimb](https://github.com/Snailclimb) và [guang19](https://github.com/guang19) cùng hoàn thành.

Về giới thiệu tổng quan cơ bản về Transaction, vui lòng xem bài viết: [Tổng kết kiến thức & câu hỏi phỏng vấn MySQL phổ biến](./mysql-questions-01.md#MySQL-事务)

## Tổng kết mức độ cô lập Transaction

Chuẩn SQL định nghĩa 4 mức độ cô lập Transaction để cân bằng giữa tính cô lập (Isolation) của Transaction và hiệu năng đồng thời. Mức độ càng cao, tính nhất quán của dữ liệu càng tốt, nhưng hiệu năng đồng thời có thể càng thấp. 4 mức độ này là:

- **READ UNCOMMITTED (Đọc chưa commit)**: Mức độ cô lập thấp nhất, cho phép đọc các thay đổi dữ liệu chưa commit, có thể dẫn đến Dirty Read, Phantom Read hoặc Non-repeatable Read. Mức độ này rất ít khi dùng trong thực tế vì khả năng bảo đảm tính nhất quán dữ liệu quá yếu.
- **READ COMMITTED (Đọc đã commit)**: Cho phép đọc dữ liệu đã commit của các Transaction đồng thời, có thể ngăn chặn Dirty Read, nhưng Phantom Read hoặc Non-repeatable Read vẫn có thể xảy ra. Đây là mức độ cô lập mặc định của hầu hết cơ sở dữ liệu (như Oracle, SQL Server).
- **REPEATABLE READ (Có thể đọc lặp lại)**: Các lần đọc cùng một field cho kết quả nhất quán, trừ khi dữ liệu được sửa đổi bởi chính Transaction đó. Có thể ngăn chặn Dirty Read và Non-repeatable Read, nhưng Phantom Read vẫn có thể xảy ra. Mức độ cô lập mặc định của Storage Engine MySQL InnoDB chính là REPEATABLE READ. Hơn nữa, InnoDB ở mức độ này thông qua cơ chế MVCC (Multiversion Concurrency Control) và Next-Key Locks (Gap Lock + Row Lock) đã giải quyết vấn đề Phantom Read ở mức độ rất lớn.
- **SERIALIZABLE (Chuỗi hóa / Tuần tự hóa)**: Mức độ cô lập cao nhất, tuân thủ hoàn toàn tính cô lập ACID. Tất cả các Transaction được thực thi lần lượt từng Transaction một, hoàn toàn không tạo ra can thiệp lẫn nhau, ngăn chặn Dirty Read, Non-repeatable Read và Phantom Read.

| Mức độ cô lập | Dirty Read | Non-Repeatable Read | Phantom Read |
| ---------------- | ----------------- | -------------------------------- | ---------------------- |
| READ UNCOMMITTED | √ | √ | √ |
| READ COMMITTED | × | √ | √ |
| REPEATABLE READ | × | × | √ (Chuẩn) / ≈× (InnoDB) |
| SERIALIZABLE | × | × | × |

**Truy vấn mức độ mặc định:**

Mức độ cô lập mặc định của Storage Engine MySQL InnoDB là **REPEATABLE READ**. Có thể xem thông qua lệnh:

- Trước MySQL 8.0: `SELECT @@tx_isolation;`
- Từ MySQL 8.0 trở đi: `SELECT @@transaction_isolation;`

```bash
mysql> SELECT @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
```

**Cách InnoDB REPEATABLE READ xử lý Phantom Read:**

Trong định nghĩa chuẩn mức độ cô lập SQL, REPEATABLE READ không thể ngăn chặn Phantom Read. Nhưng cách thực thi của InnoDB thông qua các cơ chế sau đã tránh được Phantom Read ở mức độ lớn:

- **Snapshot Read (Đọc ảnh chụp)**: Câu SELECT thông thường, thực thi thông qua cơ chế **MVCC**. Khi Transaction khởi chạy sẽ tạo một snapshot dữ liệu, các lần Snapshot Read sau đó đều đọc phiên bản dữ liệu này, từ đó tránh việc nhìn thấy các hàng mới được chèn bởi Transaction khác (Phantom Read) hoặc hàng bị sửa đổi (Non-repeatable Read).
- **Current Read (Đọc hiện tại)**: Các thao tác như `SELECT ... FOR UPDATE`, `SELECT ... LOCK IN SHARE MODE`, `INSERT`, `UPDATE`, `DELETE`. InnoDB sử dụng **Next-Key Lock** để khóa các bản ghi Index quét được và phạm vi khoảng trống (Gap) giữa chúng, ngăn chặn Transaction khác chèn bản ghi mới vào phạm vi này, từ đó tránh Phantom Read. Next-Key Lock là tổ hợp của Record Lock và Gap Lock.

Đáng chú ý là, mặc dù thông thường cho rằng mức độ cô lập càng cao thì tính đồng thời càng kém, nhưng Storage Engine InnoDB đã tối ưu hóa mức REPEATABLE READ thông qua cơ chế MVCC. Đối với nhiều kịch bản chỉ đọc hoặc đọc nhiều ghi ít phổ biến, hiệu năng của nó **so với READ COMMITTED có thể không có chênh lệch đáng kể**. Tuy nhiên, trong các kịch bản ghi đậm đặc và xung đột đồng thời cao, cơ chế Gap Lock của RR có thể mang lại nhiều chờ khóa (lock wait) hơn so với RC.

Ngoài ra, trong một số kịch bản đặc biệt như Transaction phân tán (XA Transactions) yêu cầu tính nhất quán nghiêm ngặt, InnoDB có thể yêu cầu hoặc khuyến nghị sử dụng mức độ cô lập SERIALIZABLE để đảm bảo tính nhất quán của dữ liệu toàn cục.

Trích sách "MySQL Technical Innards: InnoDB Storage Engine (2nd Edition)" chương 7.7:

> Storage Engine InnoDB cung cấp hỗ trợ cho XA Transaction và thông qua XA Transaction để hỗ trợ việc thực thi Transaction phân tán. Transaction phân tán đề cập đến việc cho phép nhiều tài nguyên transaction (transactional resources) độc lập tham gia vào một transaction toàn cục. Tài nguyên transaction thông thường là hệ quản trị cơ sở dữ liệu quan hệ, nhưng cũng có thể là các loại tài nguyên khác. Transaction toàn cục yêu cầu tất cả các transaction tham gia bên trong hoặc là cùng commit, hoặc là cùng rollback, điều này lại nâng cao yêu cầu ACID ban đầu của transaction. Ngoài ra, khi sử dụng transaction phân tán, mức độ cô lập transaction của Storage Engine InnoDB bắt buộc phải thiết lập thành SERIALIZABLE.

## Diễn tập tình huống thực tế

Dưới đây tôi sẽ sử dụng 2 lệnh MySQL terminal để mô phỏng tình huống Dirty Read của nhiều thread (nhiều Transaction) trên cùng một dữ liệu.

Trong cấu hình mặc định của MySQL terminal, các Transaction đều tự động commit (autocommit), tức là sau khi thực thi SQL sẽ lập tức COMMIT. Nếu muốn chủ động bắt đầu một Transaction cần dùng lệnh: `START TRANSACTION`.

Chúng ta có thể thiết lập mức độ cô lập bằng lệnh:

```sql
SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL [READ UNCOMMITTED|READ COMMITTED|REPEATABLE READ|SERIALIZABLE]
```

Một số câu lệnh kiểm soát đồng thời sử dụng trong thao tác thực tế bên dưới:

- `START TRANSACTION` | `BEGIN`: Chủ động bắt đầu một Transaction.
- `COMMIT`: Commit Transaction, làm cho tất cả sửa đổi với database trở thành vĩnh viễn.
- `ROLLBACK`: Rollback sẽ kết thúc Transaction của người dùng và hủy bỏ tất cả các sửa đổi chưa commit đang tiến hành.

### Dirty Read (Đọc chưa commit)

![](<https://oss.javaguide.cn/github/javaguide/2019-31-1%E8%84%8F%E8%AF%BB(%E8%AF%BB%E6%9C%AA%E6%8F%90%E4%BA%A4)%E5%AE%9E%E4%BE%8B.jpg>)

### Tránh Dirty Read (Đọc đã commit)

![](https://oss.javaguide.cn/github/javaguide/2019-31-2%E8%AF%BB%E5%B7%B2%E6%8F%90%E4%BA%A4%E5%AE%9E%E4%BE%8B.jpg)

### Non-repeatable Read (Đọc không lặp lại)

Vẫn là sơ đồ READ COMMITTED ở trên, mặc dù đã tránh được Dirty Read, nhưng lại xuất hiện vấn đề Non-repeatable Read khi Transaction chưa kết thúc.

![](https://oss.javaguide.cn/github/javaguide/2019-32-1%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB%E5%AE%9E%E4%BE%8B.jpg)

### Repeatable Read (Có thể đọc lặp lại)

![](https://oss.javaguide.cn/github/javaguide/2019-33-2%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB.jpg)

### Phantom Read (Đọc ảo)

#### Diễn tập kịch bản xuất hiện Phantom Read

![](https://oss.javaguide.cn/github/javaguide/phantom_read.png)

SQL script 1 ở lần query đầu tiên các bản ghi lương = 500 chỉ có 1 bản ghi. SQL script 2 insert 1 bản ghi lương = 500 và commit; SQL script 1 trong cùng Transaction dùng Current Read query lại phát hiện xuất hiện 2 bản ghi lương = 500, đây chính là Phantom Read.

Ghi chú: Ví dụ này bản chất là do ngữ nghĩa đọc khác nhau giữa lần đầu Snapshot Read và lần sau Current Read. Ở mức RR, MVCC có thể đảm bảo Snapshot Read không bị Phantom Read, Next-Key Lock có thể ràng buộc Current Read; nhưng khi trộn lẫn Snapshot Read và Current Read trong cùng một Transaction, kết quả nhìn thấy ở hai lần đọc có thể khác nhau.

#### Phương pháp giải quyết Phantom Read

Có nhiều cách giải quyết Phantom Read, nhưng tư tưởng cốt lõi của chúng là khi một Transaction đang thao tác trên dữ liệu một bảng nào đó, Transaction khác không được phép thêm mới hoặc xóa dữ liệu trong bảng đó nữa. Các phương pháp giải quyết Phantom Read chủ yếu có các cách sau:

1. Điều chỉnh mức độ cô lập Transaction thành `SERIALIZABLE`.
2. Ở mức độ cô lập Repeatable Read, thêm Table Lock cho bảng mà Transaction thao tác.
3. Ở mức độ cô lập Repeatable Read, thêm `Next-key Lock (Record Lock + Gap Lock)` cho bảng mà Transaction thao tác.

### Tham khảo

- 《MySQL 技术内幕：InnoDB 存储引擎》
- <https://dev.MySQL.com/doc/refman/5.7/en/>
- [Mysql 锁：灵魂七拷问](https://tech.youzan.com/seven-questions-about-the-lock-of-MySQL/)
- [Innodb 中的事务隔离级别和锁的关系](https://tech.meituan.com/2014/08/20/innodb-lock.html)

<!-- @include: @article-footer.snippet.md -->
