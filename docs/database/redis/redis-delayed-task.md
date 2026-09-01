---
title: 如何基于Redis实现延时任务？
description: 详解基于Redis实现延时任务的两种方案：过期事件监听和Redisson延时队列，分析各方案的优缺点、可靠性问题和适用场景。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis延时任务,延时队列,过期事件监听,Redisson DelayedQueue,订单超时,定时任务
---

Triển khai tính năng Delayed Task (Nhiệm vụ trì hoãn) dựa trên Redis thường có 2 giải pháp:

1. **Lắng nghe sự kiện hết hạn Key (Redis Expire Event Listener)**
2. **Hàng chờ trì hoãn tích hợp sẵn của Redisson (Redisson DelayedQueue)**

Khi phỏng vấn, bạn có thể nói rằng mình đã xem xét cả 2 giải pháp nhưng nhận thấy giải pháp lắng nghe sự kiện hết hạn Key tồn tại nhiều nhược điểm, do đó đã chọn giải pháp Redisson DelayedQueue.

### Nguyên lý của việc lắng nghe sự kiện hết hạn Key trong Redis?

Redis 2.0 đưa vào tính năng Publish/Subscribe (Pub/Sub). Trong Pub/Sub có khái niệm **channel (kênh)** tương tự như **topic (chủ đề)** trong Message Queue.
- Nhà phát hành (Publisher) dùng `PUBLISH` đẩy tin nhắn vào channel.
- Người đăng ký (Subscriber / Consumer) dùng `SUBSCRIBE` đăng ký nhận tin từ channel.

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-pub-sub.png)

Redis có các channel mặc định của hệ thống. Trong đó, `__keyevent@0__:expired` là channel mặc định phát sự kiện khi có Key bị hết hạn. Tính năng này được Redis gọi là **keyspace notifications**.

Bằng cách lắng nghe channel này, chúng ta lấy được Key bị hết hạn và kích hoạt xử lý Delayed Task.

### Nhược điểm của việc lắng nghe sự kiện hết hạn Key?

**1. Tính thời gian kém (Độ trễ cao)**

Sự kiện hết hạn KHÔNG ĐƯỢC PHÁT NGAY KHI KEY CHẠM MỐC HẾT HẠN, mà chỉ được phát khi Redis thực sự thực hiện XÓA Key đó khỏi bộ nhớ (thông qua cơ chế Định kỳ / Lười xóa). Do đó có độ trễ rất cao.

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-timing-of-expired-events.png)

**2. Dễ mất tin nhắn**

Pub/Sub của Redis không hỗ trợ Persistence (lưu trữ bền vững). Nếu lúc phát sự kiện không có Subscriber nào online listening channel, tin nhắn sẽ bị hủy bỏ vĩnh viễn.

**3. Tiêu thụ trùng lặp giữa nhiều Service Instance**

Pub/Sub của Redis hoạt động theo mô hình Broadcast (Quảng bá). Khi phát 1 tin nhắn, tất cả các instance ứng dụng cùng subscribe channel đó đều nhận được tin nhắn, dẫn tới việc xử lý lặp lại tin nhắn.

### Nguyên lý và ưu điểm của Redisson Delayed Queue?

Redisson cung cấp `RDelayedQueue` dựa trên **Sorted Set (Zset)** của Redis.

Các nhiệm vụ trì hoãn được đưa vào Sorted Set với trọng số `score` là timestamp hết hạn của nhiệm vụ. Redisson chạy định kỳ lệnh `ZRANGEBYSCORE` để quét các phần tử đã hết hạn trong Sorted Set, chuyển các phần tử này sang một Blocking Queue (Hàng chờ chặn) để các Consumer rút ra xử lý.

Ưu điểm của Redisson Delayed Queue:
1. **Hạn chế tối đa mất tin nhắn**: Dữ liệu trong Sorted Set được lưu trữ trên Redis và được bảo vệ bởi cơ chế Persistence (RDB/AOF).
2. **Không lo tiêu thụ trùng lặp**: Tất cả các Client Consumer đều rút nhiệm vụ từ cùng một Blocking Queue duy nhất, đảm bảo tính duy nhất.

*(Lưu ý: Với các hệ thống lớn đòi hỏi độ tin cậy tuyệt đối, vẫn nên ưu tiên sử dụng Delayed Message của các Message Queue chuyên dụng như RabbitMQ hay RocketMQ).*

<!-- @include: @article-footer.snippet.md -->
