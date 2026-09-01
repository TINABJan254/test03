---
title: MongoDB 专题：文档模型、索引、副本集、分片、事务与常见面试题
description: MongoDB 面试与 NoSQL 学习路线，涵盖文档模型、集合、索引、副本集、分片、事务、聚合、读写关注、存储引擎和常见面试题。
category: 数据库
tag:
  - MongoDB
  - NoSQL
  - 后端面试
sitemap:
  changefreq: weekly
  priority: 0.85
head:
  - - meta
    - name: keywords
      content: MongoDB,MongoDB面试题,NoSQL,文档数据库,MongoDB索引,副本集,分片,聚合,事务,后端面试
---

MongoDB là một Document Database điển hình, phù hợp với dữ liệu bán cấu trúc (semi-structured), mô hình field linh hoạt và các kịch bản nghiệp vụ lặp nhanh (fast iteration). Khi học MongoDB, nên tập trung hiểu sâu về Document Model, Index, Replica Set, Sharding, Aggregation và Transaction, thay vì chỉ coi nó đơn giản là một "Database có thể lưu JSON".

## Phù hợp với ai

- Lập trình viên Backend muốn tìm hiểu khái niệm cốt lõi về MongoDB và Document Database.
- Bạn đọc đang chuẩn bị các câu hỏi phỏng vấn liên quan đến MongoDB, NoSQL, Document Model.
- Kỹ sư cần lựa chọn giữa RDBMS (Cơ sở dữ liệu quan hệ) và Document Database.
- Người đọc đã từng tiếp xúc với MongoDB nhưng chưa nắm vững Index, Replica Set, Sharding và cơ chế Transaction.

## Trọng tâm học tập

- Database, Collection, Document trong MongoDB khác biệt gì so với Database, Table, Row trong RDBMS?
- Document Model phù hợp với những cấu trúc dữ liệu nào, khi nào không nên dùng MongoDB?
- Index, Aggregation Pipeline, Replica Set và Sharding trong MongoDB lần lượt giải quyết vấn đề gì?
- Hiểu như thế nào về Read Concern, Write Concern, Transaction và Consistency Semantics?
- Khi phỏng vấn, làm thế nào để trả lời câu hỏi MongoDB từ góc độ "Model, Query, Index, High Availability, Scalability, Kịch bản áp dụng"?

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản phổ biến](../nosql.md): Đầu tiên hiểu phân loại NoSQL, kịch bản áp dụng và sự khác biệt với RDBMS.
2. [Tổng hợp câu hỏi phỏng vấn MongoDB phổ biến (Phần 1)](./mongodb-questions-01.md): Học các khái niệm cơ bản của MongoDB, Document Model, Index và Query.
3. [Tổng hợp câu hỏi phỏng vấn MongoDB phổ biến (Phần 2)](./mongodb-questions-02.md): Tiếp tục tìm hiểu Replica Set, Sharding, Transaction, Aggregation và thực tiễn Production.
4. Sau đó quay lại [Hệ thống kiến thức Database](../), đặt định vị của MongoDB so sánh cùng với MySQL, Redis, Elasticsearch.

## Bài viết cốt lõi

- [Tổng hợp câu hỏi phỏng vấn MongoDB phổ biến (Phần 1)](./mongodb-questions-01.md): Bao phủ các khái niệm cơ bản của MongoDB, Document Model, Collection, Query, Index và các vấn đề sử dụng thường gặp.
- [Tổng hợp câu hỏi phỏng vấn MongoDB phổ biến (Phần 2)](./mongodb-questions-02.md): Bao phủ Replica Set, Sharding, Transaction, Aggregation, Read/Write Concern, Storage Engine và thực tiễn Production.
- [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản phổ biến](../nosql.md): Giúp hiểu định vị của MongoDB trong hệ thống NoSQL, cũng như sự khác biệt với Key-Value, Column-Family, Graph Database.

## Câu hỏi tần suất cao

- MongoDB và MySQL khác nhau như thế nào?
- Document Model của MongoDB phù hợp với những kịch bản nghiệp vụ nào?
- MongoDB Index có những loại nào? Thiết kế Index như thế nào?
- MongoDB Replica Set đảm bảo High Availability như thế nào?
- MongoDB Sharding giải quyết vấn đề gì? Lựa chọn Shard Key như thế nào?
- MongoDB có hỗ trợ Transaction không? Có phù hợp với kịch bản Transaction phức tạp không?
- Aggregation Pipeline của MongoDB phù hợp giải quyết những vấn đề nào?
- Read Concern và Write Concern lần lượt kiểm soát cái gì?
- Kịch bản nào không phù hợp sử dụng MongoDB?
- Sự khác biệt về định vị giữa MongoDB, Redis, Elasticsearch là gì?

## Chuyên đề liên quan

- [Hệ thống kiến thức Database](../)
- [Tổng hợp câu hỏi phỏng vấn NoSQL cơ bản phổ biến](../nosql.md)
- [Chuyên đề MySQL](../mysql/)
- [Chuyên đề Redis](../redis/)

<!-- @include: @article-footer.snippet.md -->
