---
title: MySQL 专题：索引、事务、日志、备份恢复、MVCC 与性能优化
description: MySQL 面试与性能优化学习路线，涵盖索引、索引失效、事务隔离级别、MVCC、binlog、redo log、undo log、备份恢复、数据同步、执行计划和 SQL 优化。
category: 数据库
tag:
  - MySQL
  - 数据库
  - 后端面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: MySQL,MySQL面试题,MySQL索引,索引失效,事务隔离级别,MVCC,binlog,redo log,undo log,MySQL备份,MySQL恢复,MySQL同步ES,Canal,Flink CDC,执行计划,SQL执行过程,MySQL性能优化,后端面试
---

MySQL là một trong những cơ sở dữ liệu quan hệ được sử dụng phổ biến nhất trong phát triển backend, và cũng là chuyên đề dễ bị truy vấn sâu nhất trong các buổi phỏng vấn database. Khi học MySQL, nên tập trung vào các trục chính: "làm thế nào Index giúp truy vấn nhanh hơn, Transaction làm sao đảm bảo tính nhất quán, log làm sao đảm bảo phục hồi và sao chép (replication), dữ liệu làm sao đồng bộ sang các hệ thống dị thể như tìm kiếm, và Execution Plan làm sao để định vị SQL chậm".

## Dành cho ai

- Lập trình viên backend muốn học một cách hệ thống về nguyên lý và tối ưu hóa hiệu năng MySQL.
- Các bạn đang chuẩn bị cho các câu hỏi phỏng vấn liên quan đến MySQL Index, Transaction, MVCC, Log, Execution Plan.
- Độc giả đã có thể viết SQL thông thường nhưng chưa quen thuộc với phân tích SQL chậm, thiết kế Index và các vấn đề về Transaction.
- Kỹ sư cần xử lý các vấn đề về hiệu năng MySQL, tính nhất quán của dữ liệu, đồng bộ dữ liệu dị thể và thiết kế field trong dự án.

## Trọng tâm học tập

- B+ Tree Index, Clustered Index, Secondary Index, Covering Index và Index Lookup (回表) là gì?
- Những cách viết SQL nào sẽ dẫn đến Index invalidation (index失效), và làm thế nào để xác minh thông qua Execution Plan?
- Transaction isolation level trong MySQL ảnh hưởng thế nào đến Dirty Read, Non-repeatable Read và Phantom Read?
- InnoDB thực hiện Snapshot Read thông qua MVCC, undo log và Read View như thế nào?
- binlog, redo log, undo log giải quyết những vấn đề gì trong sao chép (replication), phục hồi sau sự cố (crash recovery) và hoàn tác transaction (rollback)?
- MySQL khôi phục về một thời điểm cụ thể (Point-in-Time Recovery - PITR) thông qua Full Backup, binlog như thế nào?
- Có những giải pháp nào để đồng bộ dữ liệu từ MySQL sang Elasticsearch? Làm thế nào để xử lý đồng bộ toàn bộ (full load), đồng bộ tăng lượng (incremental), lệch thứ tự và đối soát dữ liệu?
- Tối ưu hóa SQL chậm nên định vị từng tầng từ tạo bảng, Index, cách viết SQL và Execution Plan như thế nào?

## Thứ tự đọc đề xuất

1. [Tổng kết câu hỏi phỏng vấn MySQL phổ biến](./mysql-questions-01.md): Trước tiên hãy xây dựng danh sách các câu hỏi MySQL tần suất cao.
2. [Chi tiết về MySQL Index](./mysql-index.md), [Tổng kết các kịch bản MySQL Index bị vô hiệu hóa](./mysql-index-invalidation.md): Hiểu về nguyên lý Index và các kịch bản vô hiệu hóa phổ biến.
3. [Chi tiết về Mức độ cô lập Transaction trong MySQL](./transaction-isolation-level.md), [Cách Storage Engine InnoDB thực thi MVCC](./innodb-implementation-of-mvcc.md): Nắm vững Transaction và Consistent Read.
4. [Chi tiết về 3 loại Log lớn trong MySQL](./mysql-logs.md), [Chi tiết về Backup và Restore trong MySQL](./mysql-backup-and-restore.md), [Chi tiết về Đồng bộ dữ liệu từ MySQL sang Elasticsearch](./mysql-to-elasticsearch-sync.md): Hiểu vai trò của binlog trong commit, restore, replication và đồng bộ dữ liệu dị thể.
5. [Quy trình thực thi câu lệnh SQL trong MySQL](./how-sql-executed-in-mysql.md): Hiểu cách Connector, Analyzer, Optimizer, Executor và Storage Engine phối hợp với nhau.
6. [Phân tích Execution Plan trong MySQL](./mysql-query-execution-plan.md), [Tổng kết quy chuẩn và kiến nghị tối ưu hóa hiệu năng cao MySQL](./mysql-high-performance-optimization-specification-recommendations.md): Đưa các nguyên lý áp dụng vào SQL chậm và quy chuẩn kỹ thuật.

## Các bài viết cốt lõi

### Tổng quan và Quy chuẩn

- [Tổng kết câu hỏi phỏng vấn MySQL phổ biến](./mysql-questions-01.md): Xâu chuỗi các điểm kiến thức tần suất cao như Index, Transaction, Lock, Log, Storage Engine và tối ưu hóa SQL.
- [Tổng kết quy chuẩn và kiến nghị tối ưu hóa hiệu năng cao MySQL](./mysql-high-performance-optimization-specification-recommendations.md): Tổng kết các kiến nghị tối ưu hóa dưới góc độ tạo bảng, field, Index, SQL, Transaction và quy chuẩn phát triển.
- [Ghi chép học tập 1000 dòng MySQL](./a-thousand-lines-of-mysql-study-notes.md): Thích hợp để rà soát bổ sung kiến thức thiếu sót, ôn tập nhanh các điểm kiến thức phổ biến của MySQL.

### Index và Execution Plan

- [Chi tiết về MySQL Index](./mysql-index.md): Hiểu về cấu trúc dữ liệu Index, Clustered Index, Secondary Index, Covering Index, Quy tắc Tiền tố Trái nhất (Leftmost Prefix Rule) và thiết kế Index.
- [Tổng kết các kịch bản MySQL Index bị vô hiệu hóa](./mysql-index-invalidation.md): Tổng hợp các cách viết khiến Index bị vô hiệu hóa phổ biến và hướng định vị kiểm tra.
- [Chuyển đổi ngầm định trong MySQL gây vô hiệu hóa Index](./index-invalidation-caused-by-implicit-conversion.md): Tập trung vào vấn đề Index bị vô hiệu hóa do chuyển đổi kiểu dữ liệu ngầm định.
- [Phân tích Execution Plan trong MySQL](./mysql-query-execution-plan.md): Nắm vững các field quan trọng của EXPLAIN như type, key, rows, Extra.

### Transaction, MVCC, Log và Đồng bộ dữ liệu

- [Chi tiết về Mức độ cô lập Transaction trong MySQL](./transaction-isolation-level.md): Hiểu về Read Uncommitted, Read Committed, Repeatable Read, Serializable và các bất thường khi đọc đồng thời.
- [Cách Storage Engine InnoDB thực thi MVCC](./innodb-implementation-of-mvcc.md): Hiểu về các field ẩn, undo log, Read View và đánh giá tính nhìn thấy (visibility).
- [Chi tiết về 3 loại Log lớn trong MySQL](./mysql-logs.md): Hiểu vai trò, thời điểm ghi của binlog, redo log, undo log và Two-Phase Commit.
- [Chi tiết về Backup và Restore trong MySQL](./mysql-backup-and-restore.md): Hiểu về mysqldump, XtraBackup, binlog, PITR, RTO/RPO và diễn tập khôi phục.
- [Chi tiết về Đồng bộ dữ liệu từ MySQL sang Elasticsearch](./mysql-to-elasticsearch-sync.md): So sánh Dual Write tầng ứng dụng, đồng bộ định kỳ, Canal, Debezium và Flink CDC, đồng thời hiểu các vấn đề về full load, incremental và tính nhất quán cuối cùng (eventual consistency).

### Quy trình thực thi và Chi tiết kỹ thuật

- [Quy trình thực thi câu lệnh SQL trong MySQL](./how-sql-executed-in-mysql.md): Hiểu về sự phối hợp giữa Connector, Query Cache, Analyzer, Optimizer, Executor và Storage Engine.
- [Chi tiết về Query Cache trong MySQL](./mysql-query-cache.md): Hiểu cách thức hoạt động của Query Cache, nguyên nhân bị vô hiệu hóa và bối cảnh nó bị loại bỏ.
- [Primary Key tự tăng trong MySQL có nhất thiết phải liên tục không?](./mysql-auto-increment-primary-key-continuous.md): Hiểu mối quan hệ giữa việc cấp phát giá trị tự tăng, rollback, chèn hàng loạt (batch insert) và tính liên tục của Primary Key.
- [Khuyến nghị lựa chọn kiểu ngày tháng trong MySQL](./some-thoughts-on-database-storage-time.md): So sánh kịch bản áp dụng của các kiểu như DATE, DATETIME, TIMESTAMP.

## Các câu hỏi tần suất cao

- Tại sao MySQL lại khuyến nghị sử dụng Index B+ Tree?
- Clustered Index và Secondary Index khác nhau như thế nào? Index Lookup (回表) và Covering Index là gì?
- Quy tắc Tiền tố Trái nhất (Leftmost Prefix Rule) là gì? Những kịch bản nào sẽ dẫn đến Index bị vô hiệu hóa?
- Làm thế nào để phán đoán thông qua EXPLAIN xem một câu SQL có đi qua Index phù hợp hay không?
- 4 mức độ cô lập Transaction trong MySQL giải quyết các vấn đề gì?
- MVCC được thực thi như thế nào? Trong Read View có những field quan trọng nào?
- binlog, redo log, undo log có điểm gì khác nhau? Two-Phase Commit giải quyết vấn đề gì?
- MySQL thực hiện khôi phục theo thời điểm thông qua Full Backup và binlog như thế nào?
- Có những giải pháp nào để đồng bộ dữ liệu từ MySQL sang Elasticsearch? Làm thế nào để đảm bảo tính Idempotent và xử lý tin nhắn lệch thứ tự?
- Tối ưu hóa SQL chậm nên bắt đầu từ những khía cạnh nào?
- Tại sao Primary Key tự tăng không nhất thiết phải liên tục?
- Nên lựa chọn giữa DATETIME và TIMESTAMP như thế nào?

## Các chuyên đề liên quan

- [Hệ thống kiến thức Database](../)
- [Chuyên đề SQL](../sql/)
- [Hệ thống kiến thức hệ thống hiệu năng cao](../../high-performance/)
- [Hệ thống kiến thức hệ thống tính sẵn sàng cao](../../high-availability/)

<!-- @include: @article-footer.snippet.md -->
