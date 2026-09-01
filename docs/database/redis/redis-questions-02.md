---
title: Redis常见面试题总结(下)
description: 最新Redis面试题总结（下）：深度剖析Redis事务原理、性能优化（pipeline/Lua/bigkey/hotkey）、缓存穿透/击穿/雪崩解决方案、慢查询与内存碎片、Redis Sentinel与Cluster集群详解。助你轻松应对后端技术面试！
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis面试题,Redis事务,Redis性能优化,Redis缓存穿透,Redis缓存击穿,Redis缓存雪崩,Redis bigkey,Redis hotkey,Redis慢查询,Redis内存碎片,Redis集群,Redis Sentinel,Redis Cluster,Redis pipeline,Redis Lua脚本
---

## Redis Transaction (Giao dịch)

### Redis Transaction là gì?

Có thể hiểu Redis Transaction là **tính năng đóng gói nhiều lệnh request thành một gói, sau đó thực thi các lệnh đã đóng gói theo đúng thứ tự mà không bị ngắt quãng giữa chừng.**

Trong phát triển thực tế, Redis Transaction rất ít khi được sử dụng và tính năng tương đối hạn chế, không nên nhầm lẫn với Transaction của cơ sở dữ liệu quan hệ (RDBMS).

Ngoài việc không đáp ứng tính Atomicity và Durability, mỗi lệnh trong Transaction đều tương tác qua mạng với Redis Server, gây lãng phí tài nguyên. Do đó không khuyến nghị dùng Redis Transaction trong lập trình hàng ngày.

### Cách sử dụng Redis Transaction?

Redis thực hiện tính năng Transaction thông qua các lệnh **`MULTI`**, **`EXEC`**, **`DISCARD`** và **`WATCH`**.

```bash
> MULTI
OK
> SET PROJECT "JavaGuide"
QUEUED
> GET PROJECT
QUEUED
> EXEC
1) OK
2) "JavaGuide"
```

1. Bắt đầu Transaction (`MULTI`);
2. Đưa lệnh vào hàng đợi (FIFO);
3. Thực thi Transaction (`EXEC`).

Dùng `DISCARD` để hủy Transaction. Dùng `WATCH key` để giám sát Key, nếu Key bị client/session khác sửa đổi trước khi `EXEC` thì toàn bộ Transaction sẽ bị hủy.

### Redis Transaction có hỗ trợ Atomicity không?

Không! Khi xảy ra lỗi trong quá trình thực thi Transaction (trừ các lệnh bị lỗi cú pháp khi enqueue), các lệnh khác vẫn tiếp tục được thực thi bình thường. Redis Transaction **không hỗ trợ thao tác Rollback (Hoàn tác)**.

### Redis Transaction có hỗ trợ Durability không?

Không! Chế độ AOF `always` tuy có thể cơ bản đáp ứng Durability nhưng làm giảm hiệu năng nghiêm trọng nên không ai dùng trên production.

### Giải quyết nhược điểm của Redis Transaction như thế nào?

Từ phiên bản 2.6, Redis hỗ trợ **Lua Script**. Một đoạn Lua Script được coi như một lệnh thực thi Atomic duy nhất. (Từ Redis 7.0 có thêm **Redis Functions**).

## ⭐️Tối ưu hóa hiệu năng Redis

### Sử dụng thao tác Batch để giảm lượt truyền qua mạng (Network RTT)

Một câu lệnh Redis gồm 4 bước: 1. Gửi lệnh -> 2. Đưa vào hàng đợi -> 3. Thực thi -> 4. Trả kết quả.
Tổng thời gian của bước 1 và 4 gọi là **Round Trip Time (RTT)**.

- **Các lệnh Batch nguyên bản**: `MGET`, `MSET`, `HMGET`, `HMSET`, `SADD`... (Chú ý trên Redis Cluster: các Key cần nằm trên cùng một Hash Slot bằng cách dùng Hashtag `{...}`).
- **Pipeline**: Đóng gói nhiều lệnh bất kỳ thành 1 đợt gửi qua mạng. Phù hợp các lệnh độc lập nhau (không Atomic).
- **Lua Script**: Gộp nhiều lệnh thành 1 câu lệnh Atomic duy nhất có hỗ trợ logic kiểm tra.

So sánh Pipeline và Redis Transaction:
- Transaction là thao tác Atomic (ngăn lệnh khác xen vào), Pipeline không phải Atomic.
- Transaction gửi từng lệnh sang server khi enqueue, Pipeline đóng gói gửi 1 lần duy nhất.

### Vấn đề Key tập trung hết hạn cùng lúc

Khi lượng lớn Key tập trung hết hạn cùng lúc, thread quét dọn định kỳ của Redis chạy trên main thread có thể làm chậm thời gian phản hồi request của client.

**Giải pháp**:
1. Đặt thời gian hết hạn ngẫu nhiên (Random Jitter).
2. Bật cơ chế Lazy Free: Đặt `lazyfree-lazy-expire yes` trong `redis.conf`.

### Big Key (Key dung lượng lớn)

#### Big Key là gì?
- Kiểu String: Value > 1MB.
- Kiểu Composite (List, Hash, Set, Zset...): Số lượng phần tử > 5000.

#### Tac hại của Big Key:
1. Nghẽn client timeout: Thao tác trên Big Key tốn thời gian làm ngắt quãng main thread.
2. Nghẽn băng thông mạng: Truy cập Big Key tốn băng thông cực lớn.
3. Nghẽn thread làm việc khi xóa: Lệnh `DEL` trên Big Key làm nghẽn main thread.

#### Tìm Big Key như thế nào?
1. Dùng lệnh CLI: `redis-cli -p 6379 --bigkeys -i 3` (dùng `-i 3` để nghỉ 3 giây giữa các lần SCAN tránh ảnh hưởng DB).
2. Dùng lệnh `SCAN` kết hợp `STRLEN`, `HLEN`, `LLEN`, `SCARD`, `ZCARD` hoặc `MEMORY USAGE` (Redis 4.0+).
3. Phân tích file RDB bằng công cụ offline (`rdb_bigkeys`, `redis-rdb-tools`).
4. Dùng tính năng phân tích Key trên Cloud (Alibaba Cloud Redis...).

#### Xử lý Big Key như thế nào?
- Chia nhỏ Big Key (ví dụ Hash lớn chia thành nhiều Hash bằng Hashing).
- Xóa bất đồng bộ bằng `UNLINK` (Redis 4.0+).
- Bật cơ chế Lazy Free (`lazyfree-lazy-server-del yes`...).

### Hot Key (Key được truy cập cực nhiều)

#### Hot Key là gì?
Key có số lượt truy cập vượt trội so với các Key khác (ví dụ: tin sốc trên mạng xã hội, sản phẩm Flash Sale).

#### Tác hại của Hot Key:
Chiếm dụng CPU và băng thông của 1 node Redis. Nếu request vượt quá tải, node Redis bị sập sẽ đẩy toàn bộ traffic đè bẹp Database đằng sau.

#### Tìm Hot Key như thế nào?
1. Lệnh CLI: `redis-cli --hotkeys` (cần cấu hình maxmemory policy là LFU).
2. Dùng lệnh `MONITOR` (chú ý: tốn hiệu năng, chỉ bật ngắn hạn).
3. Dùng thư viện phát hiện Hot Key (ví dụ JD Hotkey).
4. Dự đoán trước dựa trên nghiệp vụ (Sự kiện Flash Sale).

#### Xử lý Hot Key như thế nào?
- Read-Write Splitting (Đọc từ Read Replica).
- Phân tán Key trên Redis Cluster.
- Dùng L2 Local Cache (Caffeine ở JVM local memory).

### Slow Query Command (Lệnh truy vấn chậm)

Các lệnh O(N) như `KEYS *`, `HGETALL`, `LRANGE`, `SMEMBERS`, `SINTER`/`SUNION`/`SDIFF`... nên thay bằng `HSCAN`, `SSCAN`, `ZSCAN`.

Xem Slow Log bằng lệnh:
- `slowlog-log-slower-than 10000` (đơn vị microsecond).
- `slowlog-max-len 128`.
- Lấy Slow Log: `SLOWLOG GET N`.

## ⭐️Sự cố trong Production

### Cache Penetration (Xuyên thấu Cache)

#### Là gì?
Lượng lớn request gửi tới các Key **KHÔNG TỒN TẠI ở cả Cache lẫn Database**, khiến request đâm thẳng vào Database gây sập hệ thống.

#### Giải pháp:
1. Validate tham số ở đầu vào.
2. Cache giá trị Null / Khống (với TTL ngắn khoảng 1 phút).
3. Dùng **Bloom Filter** (Bộ lọc Bloom) để chặn request không hợp lệ trước.
4. Rate Limiting (Giới hạn truy cập theo IP / User).

### Cache Breakdown (Đánh thủng Cache)

#### Là gì?
Request tập trung vào một **Hot Key TỒN TẠI trong Database nhưng VỪA HẾT HẠN trong Cache**, khiến lượng lớn request cùng lúc đánh thẳng vào Database.

#### Giải pháp:
1. Đặt Hot Key không bao giờ hết hạn (hoặc TTL rất dài).
2. Pre-heat Cache (Khởi động nạp trước Cache cho sản phẩm Flash Sale).
3. Sử dụng Mutex Lock (Khóa tương hổ): Chỉ cho 1 thread query Database và update Cache, các thread khác chờ.

### Cache Avalanche (Tuyết sơn Cache)

#### Là gì?
Lượng lớn Cache **HẾT HẠN CÙNG MỘT THỜI ĐIỂM** hoặc **Redis Instance bị sập**, khiến toàn bộ request đâm sầm vào Database gây sập DB.

#### Giải pháp:
1. Với sự cố Redis sập: Dùng Redis Cluster / Sentinel, kết hợp Multi-level Cache (Local Cache + Redis).
2. Với việc hết hạn cùng lúc: Thêm độ lệch ngẫu nhiên (Random Jitter) vào TTL khi SET Key, Pre-heat Cache.

### Làm sao đảm bảo tính nhất quán giữa Cache và Database?

Chiến lược phổ biến nhất: **Cache Aside Pattern (Mô hình Side-car Cache)**:
- **Đọc**: Kiểm tra Cache -> Nếu Hit trả về -> Nếu Miss thì đọc DB, ghi vào Cache rồi trả về.
- **Ghi**: **Cập nhật Database trước, sau đó Xóa Cache (Delete Cache)**.

Nếu thao tác xóa Cache bị thất bại: Dùng **Message Queue để Retry Async** việc xóa Cache cho đến khi thành công.

### Các trường hợp làm nghẽn Redis (Redis Blocking)

1. Thực thi các lệnh `O(N)` (`KEYS *`, `HGETALL`...).
2. Lệnh `SAVE` đồng bộ (thay bằng `BGSAVE`).
3. AOF fsync trên main thread hoặc đĩa cứng bị nghẽn I/O.
4. Thao tác xóa Big Key (dùng `UNLINK`).
5. Dùng `FLUSHDB` / `FLUSHALL`.
6. Migration Slot trong Cluster khi có Big Key.
7. Thiếu RAM phát sinh Swap đĩa.

## Redis Cluster

- **Redis Sentinel**: Giám sát Master-Slave, tự động Failover khi Master sập.
- **Redis Cluster**: Phân tán dữ liệu theo 16384 Hash Slot trên nhiều Node, tự động sharding và failover.

## Quy chuẩn sử dụng Redis

1. Luôn sử dụng Connection Pool.
2. Hạn chế tối đa lệnh O(N).
3. Dùng Batch Operation (MGET, Pipeline, Lua Script) để giảm RTT.
4. Không lạm dụng Redis Transaction.
5. Cấm bật `MONITOR` trong thời gian dài trên production.
6. Thiết lập TTL thích hợp cho mọi Key.

<!-- @include: @article-footer.snippet.md -->
