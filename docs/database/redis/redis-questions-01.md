---
title: Redis常见面试题总结(上)
description: 最新Redis面试题总结（上）：深入讲解Redis基础、五大常用数据结构、单线程模型原理、持久化机制、内存淘汰与过期策略、分布式锁与消息队列实现。适合准备后端面试的开发者！
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis面试题,Redis基础,Redis数据结构,Redis线程模型,Redis持久化,Redis内存管理,Redis性能优化,Redis分布式锁,Redis消息队列,Redis延时队列,Redis缓存策略,Redis单线程,Redis多线程,Redis过期策略,Redis淘汰策略
---

## Redis Cơ bản

### Redis là gì?

[Redis](https://redis.io/) (**RE**mote **DI**ctionary **S**erver) là một cơ sở dữ liệu NoSQL mã nguồn mở phát triển bằng ngôn ngữ C. Khác với các cơ sở dữ liệu truyền thống, dữ liệu của Redis được lưu trữ trong bộ nhớ RAM (In-Memory Database, có hỗ trợ Persistence), do đó tốc độ đọc ghi cực kỳ nhanh, được ứng dụng rộng rãi trong mảng Distributed Cache. Ngoài ra, Redis lưu trữ dữ liệu dưới dạng Key-Value.

Để đáp ứng các kịch bản nghiệp vụ khác nhau, Redis tích hợp sẵn nhiều Data Type (như String, Hash, List, Set, Sorted Set, Bitmap, HyperLogLog, GEO). Đồng thời, Redis cũng hỗ trợ Transaction, Persistence, Lua script, mô hình Publish/Subscribe, và nhiều giải pháp Cluster có sẵn (Redis Sentinel, Redis Cluster).

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-overview-of-data-types-2023-09-28.jpg)

### ⭐️Tại sao Redis lại nhanh như vậy?

Redis thực hiện rất nhiều tối ưu hóa hiệu năng bên trong, quan trọng nhất gồm 4 điểm sau:

1. **Thao tác hoàn toàn trên RAM (Memory-Based Storage)**: Đây là nguyên nhân chính. Đọc ghi dữ liệu Redis diễn ra trên RAM với tốc độ truy cập ở mức nanosecond, trong khi đọc ghi đĩa của DB truyền thống ở mức millisecond (chênh lệch hàng nghìn lần).
2. **Mô hình I/O hiệu quả (I/O Multiplexing & Single-Threaded Event Loop)**: Redis sử dụng vòng lặp sự kiện đơn thread kết hợp kỹ thuật I/O Multiplexing, cho phép một thread duy nhất xử lý đồng thời các sự kiện I/O trên nhiều kết nối mạng, tránh được context switch và tranh chấp khóa của mô hình multi-thread.
3. **Cấu trúc dữ liệu nội bộ được tối ưu (Optimized Data Structures)**: Redis cung cấp nhiều Data Type với mã hóa tầng dưới được tối ưu cao (ziplist, quicklist, skiplist, hashtable...). Redis sẽ tự động chọn mã hóa thích hợp dựa trên kích thước dữ liệu để đạt cân bằng tốt nhất giữa hiệu năng và dung lượng.
4. **Giao thức truyền thông đơn giản, hiệu quả (RESP Protocol)**: Redis sử dụng giao thức RESP (REdis Serialization Protocol) do chính mình thiết kế. Giao thức này đơn giản, hiệu năng parse tốt và binary-safe.

![](https://oss.javaguide.cn/github/javaguide/database/redis/why-redis-so-fast.png)

### Ngoài Redis, bạn còn biết giải pháp Distributed Cache nào khác không?

Các giải pháp Distributed Cache phổ biến:

- **Memcached**: Giải pháp thế hệ đầu. Hiện nay hầu hết các dự án đều chuyển sang dùng Redis.
- **Tendis**: Do Tencent mã nguồn mở dựa trên RocksDB, tương thích 100% giao thức Redis.
- **Dragonfly**: Được coi là in-memory DB nhanh nhất hiện nay, tương thích hoàn toàn API Redis và Memcached.
- **KeyDB**: Một nhánh đa luồng hiệu năng cao của Redis.

Tuy nhiên, lựa chọn hàng đầu cho Distributed Cache vẫn là Redis nhờ hệ sinh thái cực kỳ phát triển và tài liệu phong phú.

### So sánh điểm giống và khác nhau giữa Redis và Memcached

**Điểm giống nhau**:

1. Đều là In-memory Database, chủ yếu làm Cache.
2. Đều có Expire Policy (Chiến lược hết hạn).
3. Hiệu năng đều cực kỳ cao.

**Điểm khác nhau**:

1. **Data Type**: Redis hỗ trợ các kiểu dữ liệu phong phú hơn (String, List, Set, Hash, Zset...), Memcached chỉ hỗ trợ Key/Value đơn giản.
2. **Persistence**: Redis hỗ trợ lưu dữ liệu xuống đĩa (Crash Recovery), Memcached chỉ lưu trên RAM.
3. **Cluster Support**: Memcached không có Cluster native (phụ thuộc client sharding), Redis hỗ trợ Native Cluster từ phiên bản 3.0.
4. **Thread Model**: Memcached dùng mô hình multi-thread non-blocking I/O; Redis dùng mô hình single-thread I/O multiplexing (từ Redis 6.0 mới đưa vào multi-thread cho I/O read/write).
5. **Feature**: Redis hỗ trợ Pub/Sub, Lua script, Transaction..., Memcached thì không.
6. **Expire Eviction**: Memcached chỉ dùng Lazy Eviction, Redis dùng kết hợp cả Lazy Eviction và Periodic Eviction.

### ⭐️Tại sao nên dùng Redis?

1. **Tốc độ truy cập nhanh hơn**: Dữ liệu lưu trên RAM, tốc độ đọc nhanh gấp hàng chục đến hàng trăm lần đĩa cứng.
2. **Khả năng chịu High Concurrency**: QPS của MySQL thường khoảng 4k, còn Redis có thể dễ dàng đạt 5w+ thậm chí 10w+ trên 1 instance đơn lẻ.
3. **Tính năng toàn diện**: Ngoài Cache, Redis còn dùng cho Distributed Lock, Rate Limiting, Message Queue, Delayed Queue...

### ⭐️Tại sao nên dùng Redis mà không dùng Local Cache?

| Đặc tính | Local Cache | Redis |
| ------------ | ------------------------------------ | -------------------------------- |
| Tính nhất quán dữ liệu | Dễ bị lệch dữ liệu giữa nhiều server | Dữ liệu thống nhất toàn cục |
| Giới hạn bộ nhớ | Bị giới hạn bởi RAM của 1 máy server | Triển khai độc lập, RAM lớn hơn |
| Rủi ro mất dữ liệu | Mất dữ liệu khi server restart / crash | Hỗ trợ Persistence, an toàn hơn |
| Quản lý bảo trì | Phân tán, khó quản lý | Quản lý tập trung, công cụ phong phú |
| Tính năng | Hạn chế, chỉ lưu Key-Value đơn giản | Phong phú, nhiều Data Structure |

### Chiêm ngưỡng Redis Module

Từ phiên bản 4.0, Redis hỗ trợ mở rộng tính năng thông qua Module dạng file động `.so`:
- **RediSearch**: Module công cụ tìm kiếm.
- **RedisJSON**: Module xử lý dữ liệu JSON.
- **RedisBloom**: Module triển khai Bloom Filter.
- **RedisCell**: Module triển khai Distributed Rate Limiting.

## ⭐️Ứng dụng Redis

### Ngoài làm Cache, Redis còn làm được những gì?

- **Distributed Lock**: Triển khai khóa phân tán (thường dựa trên Redisson).
- **Rate Limiting (Giới hạn tốc độ)**: Dùng Redis + Lua script hoặc `RRateLimiter` của Redisson.
- **Message Queue**: Dùng Data Structure List hoặc Stream (có Ack, Consumer Group từ Redis 5.0).
- **Delayed Queue (Hàng chờ trì hoãn)**: Dùng Redisson DelayedQueue (dựa trên Sorted Set).
- **Distributed Session**: Lưu Session data dạng String hoặc Hash cho tất cả server truy cập.
- **Kịch bản nghiệp vụ phức tạp**: Dùng Bitmap thống kê Active User (UV), Sorted Set làm Leaderboard, HyperLogLog đếm lượt truy cập ngầm.

### Dùng Redis làm Message Queue có được không? Triển khai như thế nào?

- Nếu nghiệp vụ đơn giản, lượng nhỏ và chấp nhận rủi ro mất dữ liệu cực nhỏ -> dùng **Redis Stream** là lựa chọn tối ưu vì tiết kiệm chi phí triển khai MQ riêng.
- Nếu là nghiệp vụ tài chính, dữ liệu cực lớn, yêu cầu tuyệt đối không mất message -> bắt buộc dùng **Kafka, RabbitMQ**.

### Làm thế nào để triển khai Delayed Task dựa trên Redis?

1. **Redis Expire Event Listener**: Không khuyến nghị do tính thời gian kém chính xác và dễ mất message.
2. **Redisson DelayedQueue (dựa trên Sorted Set)**: Khuyên dùng, có persistence, không lo tiêu thụ trùng lặp.

## ⭐️Redis Data Types

### Các Data Type phổ biến trong Redis

- **5 loại cơ bản**: String, List, Set, Hash, Sorted Set (Zset).
- **3 loại đặc biệt**: HyperLogLog, Bitmap, Geospatial.

### Kịch bản áp dụng của String

String là Data Type đơn giản và phổ biến nhất, là Binary-safe:
- Cache dữ liệu (Session, Token, Object đã serialize, đường dẫn ảnh...).
- Bộ đếm Counter (Đếm số lần request, đếm lượt xem).
- Distributed Lock đơn giản (`SETNX`).

### Lưu trữ Object nên dùng String hay Hash tốt hơn?

- **String**: Lưu object đã serialize toàn bộ. Thích hợp khi object đơn giản, thường xuyên đọc ghi toàn bộ.
- **Hash**: Lưu từng field riêng lẻ của object. Thích hợp khi cần đọc/sửa từng field lẻ mà không muốn truyền lại toàn bộ object qua mạng, tiết kiệm RAM hơn.

### Cấu trúc tầng dưới của String là gì?

Redis không dùng chuỗi C thuần (kết thúc bằng `\0`) mà tự viết **SDS (Simple Dynamic String)**:

```c
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* Đã sử dụng */
    uint8_t alloc; /* Kích thước khả dụng */
    unsigned char flags;
    char buf[];
};
```

Ưu điểm của SDS:
1. **Tránh Buffer Overflow**: Kiểm tra `len` và tự động cấp phát bộ nhớ trước khi nối chuỗi.
2. **Lấy độ dài chuỗi O(1)**: Đọc trực tiếp biến `len`.
3. **Giảm số lần cấp phát bộ nhớ**: Cơ sở dự phòng bộ nhớ (pre-allocation) và giải phóng lười (lazy free).
4. **Binary Safe**: Không bị ngắt bởi ký tự `\0`.

### Dùng Redis làm Leaderboard (Bảng xếp hạng) như thế nào?

Sử dụng Data Type `Sorted Set` (Zset) với các lệnh `ZRANGE` (xếp tăng dần), `ZREVRANGE` (xếp giảm dần), `ZREVRANK` (lấy thứ hạng của phần tử).

### Tại sao Sorted Set lại dùng Skiplist mà không dùng Balanced Tree, Red-Black Tree hay B+ Tree?

- **Balanced Tree vs Skiplist**: Cả hai đều có độ phức tạp tìm kiếm O(log n). Tuy nhiên Balanced Tree phải liên tục xoay cây để giữ cân bằng tuyệt đối khi Insert/Delete (rất tốn CPU). Skiplist dùng cân bằng xác suất (probabilistic balance), việc chèn/xóa đơn giản và nhanh hơn nhiều.
- **Red-Black Tree vs Skiplist**: Skiplist dễ cài đặt hơn, không cần nhuộm màu và xoay cây. Range Query trên Skiplist hiệu quả hơn nhiều so với Red-Black Tree.
- **B+ Tree vs Skiplist**: B+ Tree tối ưu cho I/O đĩa cứng (Database). Redis là In-Memory DB nên Skiplist tiết kiệm RAM hơn và cài đặt đơn giản hơn nhiều.

### Kịch bản áp dụng của Set

Set là tập hợp không lặp lại và không có thứ tự:
- Loại bỏ trùng lặp (UV, Like bài viết).
- Tìm giao / hợp / hiệu giữa các tập hợp (Bạn chung, Fan chung, Gợi ý bạn bè).
- Lấy ngẫu nhiên phần tử (Quay số trúng thưởng bằng `SPOP` hoặc `SRANDMEMBER`).

### Dùng Bitmap thống kê Active User như thế nào?

Bitmap lưu chuỗi bit 0 và 1. Dùng ngày làm Key, `user_id` làm offset (vị trí bit), nếu active trong ngày thì `SETBIT` thành 1. Dùng `BITOP AND/OR` và `BITCOUNT` để thống kê tổng số user active trong khoảng thời gian.

### HyperLogLog phù hợp kịch bản nào?

HyperLogLog (HLL) là cấu trúc dữ liệu xác suất giúp ước tính Cardinality (số lượng phần tử không trùng lặp) của tập dữ liệu khổng lồ với dung lượng RAM cố định cực nhỏ (chỉ **12KB** trong Redis, sai số khoảng 0.81%). Thích hợp làm thống kê UV website hàng triệu/hàng tỷ lượt truy cập.

### Muốn kiểm tra phần tử có nằm trong tập dữ liệu khổng lồ hay không thì dùng gì?

Sử dụng **Bloom Filter** (Bộ lọc Bloom). Bloom Filter khẳng định phần tử KHÔNG tồn tại thì chắc chắn không tồn tại; nếu khẳng định CÓ tồn tại thì có một tỷ lệ sai số nhỏ (misclassification).

## ⭐️Cơ chế Persistence của Redis

Xem bài viết chi tiết riêng: [Chi tiết cơ chế Persistence của Redis](./redis-persistence.md).

## ⭐️Thread Model của Redis

### Mô hình Single Thread của Redis là gì?

Redis thiết kế dựa trên mô hình Reactor: **File Event Handler** chạy dạng đơn luồng (Single Thread), sử dụng **I/O Multiplexing** để cùng lúc lắng nghe nhiều Socket kết nối từ client.

Bộ xử lý sự kiện file gồm 4 phần: Sockets -> IO Multiplexing -> Event Dispatcher -> Event Handlers.

### Tại sao trước Redis 6.0 lại không dùng Multi-Thread?

1. Lập trình single-thread đơn giản và dễ bảo trì.
2. Nguồn nghẽn hiệu năng của Redis không phải ở CPU mà ở RAM và băng thông mạng.
3. Multi-thread phát sinh các vấn đề deadlock, context switch làm giảm hiệu năng.

*(Lưu ý: Redis 4.0 đã thêm multi-thread cho thao tác xóa bất đồng bộ các Big Key: `UNLINK`, `FLUSHALL ASYNC`, `FLUSHDB ASYNC`).*

### Tại sao từ Redis 6.0 lại đưa vào Multi-Thread?

Redis 6.0 đưa vào multi-thread **chỉ để xử lý đọc ghi I/O mạng** (tăng tốc độ đọc ghi gói tin mạng). Việc thực thi các lệnh (command execution) vẫn hoàn toàn là single-thread theo thứ tự, do đó không lo ngại vấn đề Thread-safe.

### Các Background Thread trong Redis

Redis có các thread chạy ngầm: `bio_close_file` (đóng file tạm), `bio_aof_fsync` (flush dữ liệu AOF xuống đĩa), `bio_lazy_free` (giải phóng RAM cho các Big Object đã xóa).

## ⭐️Quản lý bộ nhớ Redis

### Tác dụng của việc đặt Expire Time cho Key

1. Tránh nổ bộ nhớ (OOM) khi lưu trữ dữ liệu lâu dài.
2. Phù hợp các kịch bản dữ liệu tạm thời (Mã OTP 1 phút, Session/Token 1 ngày).

### Redis kiểm tra Key hết hạn như thế nào?

Redis duy trì một **Expire Dictionary** (`expires` dict trong cấu trúc `redisDb`) lưu trữ timestamp hết hạn của các Key dưới dạng UNIX epoch (millisecond).

### Chiến lược xóa Key hết hạn trong Redis

Redis kết hợp 2 chiến lược: **Periodic Eviction (Xóa định kỳ)** + **Lazy Eviction (Xóa lười)**.
- **Lazy Eviction**: Chỉ kiểm tra và xóa Key khi có request query tới Key đó (tiết kiệm CPU, tốn RAM).
- **Periodic Eviction**: Mỗi giây thực hiện `hz` lần (mặc định 10 lần/giây), mỗi lần rút ngẫu nhiên 20 Key kiểm tra, nếu đã hết hạn thì xóa.

### Xử lý thế nào khi lượng lớn Key tập trung hết hạn cùng lúc?

1. Thêm độ lệch ngẫu nhiên (random jitter) vào Expire Time khi cài đặt.
2. Bật cơ chế Lazy Free: Cấu hình `lazyfree-lazy-expire yes` trong `redis.conf` để xóa Key hết hạn bất đồng bộ ở background thread mà không làm nghẽn main thread.

### Chiến lược Eviction khi RAM chạm ngưỡng maxmemory

Khi RAM đạt ngưỡng `maxmemory`, Redis cung cấp các chiến lược Eviction:

1. **volatile-lru**: Loại bỏ Key ít sử dụng gần đây nhất (LRU) trong số các Key có đặt Expire Time.
2. **volatile-ttl**: Loại bỏ Key sắp hết hạn nhất.
3. **volatile-random**: Loại bỏ ngẫu nhiên Key trong số các Key có đặt Expire Time.
4. **allkeys-lru**: Loại bỏ Key LRU trong tất cả các Key.
5. **allkeys-random**: Loại bỏ ngẫu nhiên trong tất cả các Key.
6. **no-eviction** (Mặc định): Không loại bỏ, trả về lỗi khi ghi dữ liệu mới.
7. **volatile-lfu**: Loại bỏ Key ít tần suất sử dụng nhất (LFU) trong số các Key có đặt Expire Time (từ Redis 4.0).
8. **allkeys-lfu**: Loại bỏ Key LFU trong tất cả các Key (từ Redis 4.0).

## Tham khảo

- 《Redis 开发与运维》
- 《Redis 设计与实现》
- 《Redis 核心原理与实战》

<!-- @include: @article-footer.snippet.md -->
