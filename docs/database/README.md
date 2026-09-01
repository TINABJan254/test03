---
title: 数据库知识体系：SQL、MySQL、Redis、MongoDB 与 Elasticsearch
description: 数据库面试与知识体系学习路线，涵盖 SQL、MySQL 索引、事务、日志、MVCC、执行计划、Redis 缓存、MongoDB 和 Elasticsearch。
category: 数据库
tag:
  - 数据库
  - MySQL
  - Redis
sitemap:
  changefreq: weekly
  priority: 0.95
head:
  - - meta
    - name: keywords
      content: 数据库,数据库面试题,SQL,MySQL,Redis,MongoDB,Elasticsearch,MySQL索引,MySQL事务,MySQL日志,MVCC,Redis缓存,Redis持久化,Redis集群,后端面试
---

<!-- @include: @small-advertisement.snippet.md -->

Trang **Hệ thống kiến thức Database** này dành cho việc học backend, thực hành dự án và ôn tập phỏng vấn, sắp xếp các bài viết liên quan đến Database trên trang web theo thứ tự "Database cơ bản -> SQL -> MySQL -> Redis -> NoSQL và Tìm kiếm".

Nếu bạn có thời gian có hạn, khuyến nghị nên xem trước [Tổng hợp câu hỏi phỏng vấn thường gặp về Database cơ bản](./basis.md), [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql/sql-syntax-summary.md), [Tổng hợp câu hỏi phỏng vấn thường gặp về MySQL](./mysql/mysql-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn thường gặp về Redis (Phần 1)](./redis/redis-questions-01.md), để nhanh chóng xây dựng danh sách câu hỏi tần suất cao.

## Phù hợp với ai

- Các lập trình viên backend đang học một cách hệ thống về Database cơ bản, SQL, MySQL và Redis.
- Các bạn học sinh/sinh viên và ứng viên đang chuẩn bị cho phỏng vấn backend (Campus Recruitment, Social Recruitment, các công ty vừa và lớn).
- Các kỹ sư muốn củng cố năng lực về Index, Transaction, Log, Execution Plan, Cache Consistency, Redis Persistence và Cluster.
- Các độc giả đã từng viết CRUD nghiệp vụ, nhưng chưa thực sự nắm vững nguyên lý tầng dưới của Database, Performance Optimization và lựa chọn NoSQL.

## Trọng tâm học tập

- RDBMS và NoSQL thích hợp giải quyết vấn đề gì, ranh giới lựa chọn phổ biến nằm ở đâu?
- Cú pháp Query, Aggregation, JOIN, Subquery và Transaction trong SQL nên được nắm vững như thế nào?
- MySQL Index, Transaction Isolation, MVCC, 3 loại Log lớn và Execution Plan kết nối thành một đường dây chính như thế nào?
- Tại sao Redis lại nhanh, làm sao để hiểu các Data Structure thường dùng, Cache Strategy, Persistence, vấn đề Blocking và cơ chế Cluster?
- Các hệ thống NoSQL/Search như MongoDB, Elasticsearch thường hỏi những điểm nào trong phỏng vấn backend và lựa chọn công nghệ?

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn thường gặp về Database cơ bản](./basis.md) và [Tổng hợp câu hỏi phỏng vấn thường gặp về NoSQL cơ bản](./nosql.md): Đầu tiên hãy hiểu về phân loại Database, Transaction, Normal Form, loại NoSQL và kịch bản ứng dụng điển hình.
2. [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql/sql-syntax-summary.md): Củng cố các kỹ năng cơ bản về SQL như Query, Filter, Sort, Aggregation, JOIN, Subquery, Insert, Update và Delete.
3. [Chuyên đề MySQL](./mysql/): Tập trung học về Index, Transaction Isolation, MVCC, 3 loại Log lớn, quá trình thực thi và Execution Plan.
4. [Chuyên đề Redis](./redis/): Tập trung học về Cache cơ bản, Data Structure, Cache Read/Write Strategy, Persistence, vấn đề Blocking, Memory Fragmentation và Cluster.
5. [Chuyên đề MongoDB](./mongodb/) và [Tổng hợp câu hỏi phỏng vấn thường gặp về Elasticsearch](./elasticsearch/elasticsearch-questions-01.md): Bổ sung kiến thức về Document Database và Search Engine dựa theo yêu cầu vị trí công việc.

## Các bài viết cốt lõi

### Database cơ bản và SQL

Phần này thích hợp để xây dựng nhận thức chung về Database trước, tập trung hiểu loại Database, Transaction semantics, Character Set, SQL cơ bản và các bài tập Query thường gặp.

- [Tổng hợp câu hỏi phỏng vấn thường gặp về Database cơ bản](./basis.md): Hệ thống lại các khái niệm cơ bản về Database, đặc tính Transaction, Concurrency Control, Normal Form và các câu hỏi phỏng vấn thường gặp.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về NoSQL cơ bản](./nosql.md): Hiểu các loại NoSQL như Key-Value, Document, Column-Family, Graph Database và kịch bản áp dụng.
- [Giải thích chi tiết về Character Set: Character Set là gì? Sử dụng như thế meo?](./character-set.md): Hiểu về Character Set, Encoding, nguyên nhân lỗi font (garbled text) và cài đặt Character Set trong MySQL.
- [Chuyên đề SQL](./sql/): Giảng từ cơ bản cú pháp SQL đến các câu hỏi phỏng vấn SQL thường gặp.
- [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql/sql-syntax-summary.md): Bao gồm Query, Filter, Sort, Aggregation, Grouping, JOIN, Subquery và Data Modification.

### MySQL

MySQL là một trong những trọng tâm kiến thức RDBMS cốt lõi nhất trong lập trình backend. Khi học, khuyến nghị nên kết nối "Index -> Execution Plan -> Transaction -> MVCC -> Log -> Performance Optimization" lại với nhau.

- [Chuyên đề MySQL](./mysql/): Chuỗi liên kết về MySQL Index, Transaction, MVCC, Log, Execution Plan và Performance Optimization.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về MySQL](./mysql/mysql-questions-01.md): Nhanh chóng xây dựng danh sách câu hỏi MySQL tần suất cao.
- [Giải thích chi tiết về MySQL Index](./mysql/mysql-index.md): Hiểu Data Structure của Index, Leftmost Prefix Rule, Covering Index, Table Lookups (thẻ quay lại bảng) và Nguyên tắc thiết kế Index.
- [Giải thích chi tiết về Transaction Isolation Level trong MySQL](./mysql/transaction-isolation-level.md): Hiểu Dirty Read, Non-repeatable Read, Phantom Read và sự đánh đổi giữa các Isolation Level khác nhau.
- [Cách InnoDB Storage Engine thực thi MVCC](./mysql/innodb-implementation-of-mvcc.md): Hiểu Read View, các field ẩn, undo log và Snapshot Read.
- [Giải thích chi tiết 3 loại Log lớn của MySQL](./mysql/mysql-logs.md): Hiểu vai trò và mối quan hệ giữa binlog, redo log, undo log.
- [Giải thích chi tiết Đồng bộ dữ liệu từ MySQL sang Elasticsearch](./mysql/mysql-to-elasticsearch-sync.md): So sánh Dual-write ở tầng application, đồng bộ định kỳ, Canal, Debezium và Flink CDC, hiểu về vấn đề đồng bộ toàn bộ (Full Data), đồng bộ tăng cường (Incremental Data) và Data Consistency.
- [Phân tích Execution Plan của MySQL](./mysql/mysql-query-execution-plan.md): Nắm vững các field thường gặp trong EXPLAIN và điểm bắt đầu phân tích Slow SQL.

### Redis

Redis vừa là Cache, vừa là một chủ đề kiểm tra middleware tần suất cao. Khi học đừng chỉ học thuộc lệnh, mà cần phải hiểu về Cache Strategy, Data Structure, Persistence, nguyên nhân Blocking và cơ chế Cluster.

- [Chuyên đề Redis](./redis/): Xoay quanh Cache, Data Structure, Persistence, Cluster, vấn đề Blocking và thực hành dự án.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về Cache cơ bản](./redis/cache-basics.md): Hiểu trường hợp sử dụng Cache, Cache Penetration, Cache Breakdown, Cache Avalanche và vấn đề Consistency.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về Redis (Phần 1)](./redis/redis-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn thường gặp về Redis (Phần 2)](./redis/redis-questions-02.md): Nhanh chóng xây dựng danh sách câu hỏi Redis tần suất cao.
- [Giải thích chi tiết 5 loại Data Structure cơ bản của Redis](./redis/redis-data-structures-01.md): Hiểu trường hợp ứng dụng của String, List, Hash, Set, Sorted Set.
- [Giải thích chi tiết cơ chế Persistence của Redis](./redis/redis-persistence.md): Hiểu RDB, AOF, AOF Rewrite và RDB-AOF Hybrid Persistence.
- [Giải thích chi tiết về Redis Cluster](./redis/redis-cluster.md): Hiểu Master-Slave Replication, Sentinel, Cluster, Hash Slot và Failover.

### NoSQL và Tìm kiếm

Phần này thích hợp bổ sung sau khi đã nắm vững RDBMS và Cache, dùng để hiểu ranh giới lựa chọn giữa Document Database, Search Engine và Non-relational Storage.

- [Chuyên đề MongoDB](./mongodb/): Sắp xếp lại Document Model, Index, Replica Set, Sharding, Transaction và các câu hỏi phỏng vấn thường gặp trong MongoDB.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về MongoDB (Phần 1)](./mongodb/mongodb-questions-01.md) và [Tổng hợp câu hỏi phỏng vấn thường gặp về MongoDB (Phần 2)](./mongodb/mongodb-questions-02.md): Hiểu các khái niệm cốt lõi và thực hành dự án với MongoDB.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về Elasticsearch](./elasticsearch/elasticsearch-questions-01.md): Hiểu Inverted Index, Shard, Replica, quy trình Write/Query và kịch bản Tìm kiếm.

## Câu hỏi tần suất cao

- RDBMS và NoSQL có điểm gì khác nhau? Lần lượt phù hợp với những kịch bản nào?
- Transaction ACID trong Database là gì? Các Isolation Level lần lượt giải quyết vấn đề gì?
- Thứ tự thực thi của WHERE, GROUP BY, HAVING, ORDER BY trong SQL được hiểu như thế meo?
- Tại sao MySQL Index lại có thể tăng tốc Query? Những trường hợp nào khiến Index bị thất hiệu (không hoạt động)?
- InnoDB làm thế nào để thực hiện Non-locking Consistent Read thông qua MVCC?
- binlog, redo log, undo log lần lượt giải quyết vấn đề gì?
- Làm thế nào để phân tích SQL Execution Plan thông qua EXPLAIN?
- Tại sao Redis lại nhanh? Xử lý Cache Penetration, Cache Breakdown, Cache Avalanche như thế nào?
- Redis Persistence, Master-Slave Replication, Sentinel và Cluster lần lượt giải quyết vấn đề gì?
- MongoDB và Elasticsearch lần lượt phù hợp với những kịch bản nào?

## Các chuyên đề liên quan

- [Hệ thống kiến thức hệ thống hiệu năng cao](../high-performance/)
- [Hệ thống kiến thức hệ thống tính sẵn sàng cao](../high-availability/)
- [Hệ thống kiến thức hệ thống phân tán](../distributed-system/)
- [Thiết kế hệ thống](../system-design/)

<!-- @include: @article-footer.snippet.md -->
