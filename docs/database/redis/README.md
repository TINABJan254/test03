---
title: Redis 专题：缓存、数据结构、持久化、集群、阻塞与工程实践
description: Redis 面试与缓存学习路线，涵盖缓存穿透、缓存击穿、缓存雪崩、读写策略、Redis 数据结构、持久化、阻塞问题、延时任务和集群。
category: 数据库
tag:
  - Redis
  - 缓存
  - 后端面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Redis,Redis面试题,缓存,缓存穿透,缓存击穿,缓存雪崩,Redis数据结构,Redis持久化,Redis集群,Redis阻塞,Redis跳表,Redis延时任务,Redis消息队列,后端面试
---

Redis là một trong những giải pháp Cache và In-Memory Data Store hiệu năng cao được sử dụng phổ biến nhất trong phát triển backend. Khi học Redis, chúng ta không nên chỉ dừng lại ở các lệnh (command) và Data Type, mà còn phải hiểu rõ các chiến lược đọc ghi Cache, Data Structure tầng dưới, Persistence, nguyên nhân nghẽn (blocking), quản lý bộ nhớ, Replication và Cluster trong thực tiễn kỹ thuật.

## Phù hợp với ai

- Lập trình viên backend muốn học một cách hệ thống về nguyên lý Redis, thiết kế Cache và thực tiễn kỹ thuật.
- Các bạn chuẩn bị phỏng vấn liên quan tới Redis Data Structure, Persistence, Cluster, High Availability, Cache Consistency.
- Độc giả đã sử dụng Redis trong dự án nhưng chưa nắm vững về Cache Exception, nghẽn (blocking), mảnh bộ nhớ (Memory Fragmentation) và cơ chế Cluster.
- Các kỹ sư cần triển khai Delayed Task, Message Queue, Leaderboard, Shopping Cart... dựa trên Redis.

## Trọng tâm học tập

- Các Data Structure phổ biến của Redis phù hợp với kịch bản nghiệp vụ nào, encoding tầng dưới ảnh hưởng thế nào đến hiệu năng?
- Cache Penetration, Cache Breakdown, Cache Avalanche và Cache Consistency nên được thiết kế như thế nào?
- RDB, AOF, AOF Rewrite và Hybrid Persistence trong Redis Persistence có điểm gì khác nhau?
- Tại sao Redis có thể bị nghẽn (blocked), định vị slow command, Big Key, Hot Key và tác động của Persistence ra sao?
- Master-Slave Replication, Sentinel và Cluster giải quyết những vấn đề gì, các bước chuyển đổi sự cố (Failover) then chốt là gì?
- Khi dùng Redis làm Delayed Task, Message Queue có những ranh giới khả năng và rủi ro về độ tin cậy nào?

## Thứ tự đọc khuyến nghị

1. [Tóm tắt câu hỏi phỏng vấn phổ biến về Cache cơ bản](./cache-basics.md): Hiểu trước kịch bản sử dụng Cache, Cache Exception và vấn đề Consistency.
2. [Tóm tắt câu hỏi phỏng vấn Redis phổ biến (Tập 1)](./redis-questions-01.md), [Tóm tắt câu hỏi phỏng vấn Redis phổ biến (Tập 2)](./redis-questions-02.md): Xây dựng danh sách câu hỏi tần suất cao về Redis.
3. [Chi tiết 5 loại Data Type cơ bản của Redis](./redis-data-structures-01.md), [Chi tiết 3 loại Data Type đặc biệt của Redis](./redis-data-structures-02.md): Nắm vững hệ thống Data Structure và kịch bản áp dụng.
4. [Chi tiết 3 chiến lược đọc ghi Cache phổ biến](./3-commonly-used-cache-read-and-write-strategies.md), [Chi tiết cơ chế Persistence của Redis](./redis-persistence.md): Bổ sung khả năng Cache Consistency và khôi phục dữ liệu.
5. [Tóm tắt các nguyên nhân nghẽn phổ biến của Redis](./redis-common-blocking-problems-summary.md), [Chi tiết Redis Cluster](./redis-cluster.md): Đưa Redis vào môi trường production để thấu hiểu.

## Bài viết cốt lõi

### Cache cơ bản & Chiến lược đọc ghi

- [Tóm tắt câu hỏi phỏng vấn phổ biến về Cache cơ bản](./cache-basics.md): Giải thích kịch bản ứng dụng Cache, Cache Penetration, Cache Breakdown, Cache Avalanche, Cache Consistency và Cache Eviction.
- [Chi tiết 3 chiến lược đọc ghi Cache phổ biến](./3-commonly-used-cache-read-and-write-strategies.md): So sánh các chiến lược phổ biến như Cache Aside, Read/Write Through, Write Behind.
- [Tóm tắt câu hỏi phỏng vấn Redis phổ biến (Tập 1)](./redis-questions-01.md) và [Tóm tắt câu hỏi phỏng vấn Redis phổ biến (Tập 2)](./redis-questions-02.md): Chuỗi kiến thức về Redis cơ bản, Thread Model, Data Structure, Persistence, Cluster và các vấn đề trong production.

### Data Structure & Ứng dụng tiêu biểu

- [Chi tiết 5 loại Data Type cơ bản của Redis](./redis-data-structures-01.md): Hiểu cấu trúc tầng dưới và kịch bản nghiệp vụ của String, List, Hash, Set, Sorted Set.
- [Chi tiết 3 loại Data Type đặc biệt của Redis](./redis-data-structures-02.md): Hiểu cách dùng và kịch bản áp dụng của Bitmap, HyperLogLog, Geospatial.
- [Tại sao Redis dùng Skiplist để triển khai Sorted Set](./redis-skiplist.md): Hiểu cấu trúc Skiplist, độ phức tạp truy vấn và lựa chọn triển khai Sorted Set.
- [Làm thế nào để triển khai Delayed Task dựa trên Redis?](./redis-delayed-task.md): So sánh các cách triển khai như Expire Event, Sorted Set, Stream.
- [Làm thế nào để triển khai Message Queue dựa trên Redis?](./redis-stream-mq.md): Hiểu sự khác biệt khi dùng List, Pub/Sub, Stream làm Message Queue.

### Persistence, Bộ nhớ & Cluster

- [Chi tiết cơ chế Persistence của Redis](./redis-persistence.md): Giải thích hệ thống về RDB, AOF, AOF Rewrite và Hybrid Persistence.
- [Chi tiết mảnh bộ nhớ (Memory Fragmentation) trong Redis](./redis-memory-fragmentation.md): Hiểu nguyên nhân phát sinh mảnh bộ nhớ, chỉ số quan sát và chiến lược dọn dẹp.
- [Tóm tắt các nguyên nhân nghẽn phổ biến của Redis](./redis-common-blocking-problems-summary.md): Tổng hợp nguồn gốc gây nghẽn từ slow command, Big Key, Persistence, Master-Slave Sync, CPU và mạng.
- [Chi tiết Redis Cluster](./redis-cluster.md): Hiểu Master-Slave Replication, Sentinel, Cluster, Slot Migration và Failover.

## Câu hỏi tần suất cao

- Tại sao Redis lại nhanh? Tại sao Single Thread vẫn hỗ trợ được High Concurrency?
- Các Data Type phổ biến của Redis phù hợp với kịch bản nghiệp vụ nào?
- Tại sao Sorted Set lại sử dụng Skiplist?
- Cache Penetration, Breakdown, Avalanche khác nhau như thế nào, xử lý ra sao?
- Cache và Database đảm bảo tính nhất quán (Consistency) như thế nào?
- RDB và AOF khác nhau thế nào? AOF Rewrite giải quyết vấn đề gì?
- Các nguyên nhân nghẽn phổ biến của Redis là gì? Định vị Big Key và slow command như thế nào?
- Master-Slave Replication, Sentinel và Cluster khác nhau thế nào?
- Redis triển khai Delayed Task như thế nào? Rủi ro về độ tin cậy nằm ở đâu?
- Redis Stream so với Message Queue truyền thống có những ranh giới nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức Database](../)
- [Hệ thống kiến thức High-Performance System](../../high-performance/)
- [Hệ thống kiến thức High-Availability System](../../high-availability/)
- [Chuyên đề Message Queue](../../high-performance/message-queue/)

<!-- @include: @article-footer.snippet.md -->
