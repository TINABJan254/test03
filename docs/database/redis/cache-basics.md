---
title: 缓存基础常见面试题总结
description: 深入讲解缓存的核心思想、本地缓存与分布式缓存的区别、多级缓存架构设计。涵盖Caffeine、Redis等主流缓存方案，以及缓存一致性的解决方案。适合Java开发者学习缓存架构设计。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: 缓存,本地缓存,分布式缓存,多级缓存,Caffeine,Redis,缓存一致性,系统设计,Java缓存,Guava Cache
---

> **Các câu hỏi phỏng vấn liên quan**:
>
> - Tại sao phải dùng Cache?
> - Làm Local Cache như thế nào?
> - Tại sao cần Distributed Cache? / Tại sao không dùng trực tiếp Local Cache?
> - Tại sao phải dùng Multi-level Cache (Bộ nhớ tạm đa tầng)?
> - Multi-level Cache phù hợp kịch bản nghiệp vụ nào?

## Tư tưởng cơ bản của Cache

Tư tưởng cơ bản của Cache thực ra rất đơn giản, đó chính là việc vận dụng chiến lược tối ưu hiệu năng kinh điển: **Đổi không gian lấy thời gian (Space-for-Time Trade-off)**. Dùng thêm dung lượng lưu trữ để lưu trữ các dữ liệu có thể tính toán hoặc sử dụng lại nhiều lần, từ đó giảm thời gian tính toán hoặc lấy lại dữ liệu.

Ngoài Cache, các ví dụ khác về đổi không gian lấy thời gian:

- **Index (Chỉ mục)**: Tổ chức một số cột trong bảng thành cấu trúc dữ liệu riêng biệt. Dù tốn thêm dung lượng đĩa nhưng tăng tốc độ tìm kiếm cực lớn.
- **Dư thừa cột trong Database (Denormalization)**: Lưu trữ dư thừa dữ liệu thường xuyên JOIN vào cùng 1 bảng để giảm JOIN nhiều bảng.
- **CDN (Content Delivery Network)**: Phân phối tài nguyên tĩnh tới các node biên gần người dùng nhất để tăng tốc truy cập.

Tư tưởng Cache xuất hiện ở khắp mọi nơi: **CPU Cache** (L1/L2/L3) cache dữ liệu RAM để giải quyết chênh lệch tốc độ giữa CPU và RAM; RAM cache dữ liệu đĩa cứng để giải quyết I/O đĩa chậm.

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/cpu-cache.png)

Hệ điều hành dùng **TLB (Translation Lookaside Buffer)** để tăng tốc chuyển đổi địa chỉ ảo sang địa chỉ vật lý. Trình duyệt dùng Browser Cache để lưu các file tĩnh.

Cache trong phát triển phần mềm thường lưu trên **RAM** với tốc độ cực nhanh. Việc bổ sung một lớp Cache trên tầng Database là giải pháp cốt lõi để bảo vệ bộ lưu trữ tầng dưới và nâng cao throughput (thông lượng) của hệ thống.

## Phân loại Cache

### Local Cache (Bộ nhớ tạm cục bộ)

#### Local Cache là gì?

Local Cache nằm bên trong tiến trình ứng dụng. Ưu điểm lớn nhất là truy cập cực nhanh do nằm trong cùng bộ nhớ JVM/Process, không phát sinh chi phí truyền qua mạng.

![](https://oss.javaguide.cn/github/javaguide/database/redis/local-cache.png)

**Lưu ý**: Trong mô hình Cluster (cụm), nếu dùng Nginx cân bằng tải dạng **Round-Robin**, request của cùng một user sẽ rơi ngẫu nhiên vào các máy khác nhau làm tỷ lệ trúng (Hit rate) Local Cache rất thấp.
- Giải pháp: Dùng Consistent Hashing / Sticky Session trên Nginx, hoặc chỉ dùng Local Cache cho dữ liệu tĩnh hầu như không đổi toàn hệ thống (Cấu hình, Từ điển).

#### Các giải pháp Local Cache?

1. **`HashMap` và `ConcurrentHashMap` trong JDK**: Chỉ lưu trữ Key-Value đơn thuần, không có TTL (thời gian hết hạn) và Eviction policy (cơ chế loại bỏ).
2. **`Ehcache`, `Guava Cache`, `Spring Cache`**:
   - `Ehcache`: Khá nặng, hỗ trợ nhúng vào Hibernate/MyBatis và lưu đĩa cứng.
   - `Guava Cache`: Cung cấp API rất tiện lợi, hỗ trợ thiết lập TTL, cấu trúc sạch sẽ.
   - `Spring Cache`: Dùng Annotation rất gọn gàng nhưng dễ vướng lỗi Cache Penetration hay OOM nếu dùng không khéo.
3. **`Caffeine`**: "Ngôi sao mới" có hiệu năng vượt trội hơn Guava về mọi mặt, là lựa chọn thay thế hàng đầu cho Guava Cache.

Code tạo Local Cache bằng Caffeine:

```java
// Ví dụ tạo Local Cache bằng Caffeine
Cache<String, String> cache = Caffeine.newBuilder()
        // Hết hạn sau 60 ngày ghi
        .expireAfterWrite(60, TimeUnit.DAYS)
        // Dung lượng khởi tạo
        .initialCapacity(100)
        // Giới hạn số lượng bản ghi tối đa
        .maximumSize(500)
        // Bật tính năng thống kê
        .recordStats()
        .build();
```

#### Nhược điểm của Local Cache?

- **Phụ thuộc vào từng instance, không hỗ trợ tốt cho kiến trúc phân tán**: Mỗi server giữ bản cache riêng, không chia sẻ được dữ liệu với các máy khác.
- **Bị giới hạn dung lượng bởi RAM của máy app**.

### Distributed Cache (Bộ nhớ tạm phân tán)

#### Distributed Cache là gì?

Distributed Cache là một service In-Memory Database độc lập (như Redis). Các ứng dụng khác nhau có thể dùng chung một Distributed Cache.

![](https://oss.javaguide.cn/github/javaguide/database/redis/distributed-cache.png)

Tác hại khi đưa thêm Distributed Cache vào hệ thống:
- **Tăng độ phức tạp hệ thống**: Phải bảo trì tính nhất quán giữa Cache và DB, bảo trì High Availability cho Cache service.
- **Tăng chi phí hạ tầng**: Tốn thêm chi phí cho server RAM đắt đỏ.

#### Các giải pháp Distributed Cache?

Phổ biến nhất hiện nay là **Redis**. Các sản phẩm khác: Memcached, Tendis, Dragonfly, KeyDB. Lựa chọn ưu tiên hàng đầu vẫn là Redis.

### Multi-level Cache (Bộ nhớ tạm đa tầng)

#### Multi-level Cache là gì? Tại sao phải dùng?

Phổ biến nhất là mô hình **Local Cache (L1) + Distributed Cache (L2)**:
- **L1 (Local Cache - Caffeine)**: Nằm ngay trên JVM memory, tốc độ truy cập cực nhanh.
- **L2 (Distributed Cache - Redis)**: Nằm trên server riêng, dùng chung cho toàn bộ Cluster.

![](https://oss.javaguide.cn/javaguide/database/redis/multilevel-cache.png)

Kịch bản phù hợp dùng Multi-level Cache:
1. Dữ liệu cực kỳ ổn định, hiếm khi sửa đổi.
2. QPS siêu lớn (như kịch bản Flash Sale).

Quy trình đọc: L1 Miss -> Đọc L2 -> L2 Hit thì **Fill-back (nạp lại) vào L1 của instance hiện tại** rồi trả về -> L2 Miss thì đọc DB -> Đọc thành công ghi vào cả L1 và L2.

Framework hỗ trợ: [J2Cache](https://gitee.com/ld/J2Cache), [JetCache](https://github.com/alibaba/jetcache) (của Alibaba).

#### Đảm bảo tính nhất quán trong Multi-level Cache thế nào?

Thường đạt tính nhất quán cuối cùng (Eventual Consistency). Dùng Pub/Sub của Redis hoặc MQ Broadcast / Canal khi DB thay đổi để thông báo cho tất cả các JVM instance invalidate (xóa) L1 Local Cache tương ứng.

## Đọc thêm về Data Structure

- [Chi tiết Bloom Filter](../../cs-basics/data-structure/bloom-filter.md)
- [Tóm tắt câu hỏi phỏng vấn LRU Cache](../../cs-basics/data-structure/lru-cache.md)
- [Tóm tắt câu hỏi phỏng vấn Hash Table](../../cs-basics/data-structure/hash-table.md)

<!-- @include: @article-footer.snippet.md -->
