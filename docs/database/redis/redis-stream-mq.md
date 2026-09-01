---
title: 如何基于Redis实现消息队列？
description: 讲解 Redis 做消息队列的三种方式：List、Pub/Sub、Stream。对比生产级 MQ 核心能力，详解 Redis 5.0 Stream 的消费者组、ACK 机制及与 Kafka/RabbitMQ 的适用场景对比。
category: 数据库
tag:
  - Redis
  - 消息队列
head:
  - - meta
    - name: keywords
      content: Redis消息队列,Redis Stream,Redis List,Redis Pub/Sub,消息队列,消费者组,ACK机制,XREADGROUP,XADD,XACK
---

Kết luận trước: **Redis hoàn toàn có thể làm Message Queue, nhưng phụ thuộc vào kịch bản cụ thể. So với các MQ chuyên dụng (Kafka, RabbitMQ), Redis vẫn có những điểm hạn chế.**

Một Message Queue cấp Production cần các năng lực cốt lõi: Persistence (Lưu trữ bền vững), At-least-once delivery (Gửi ít nhất một lần), ACK mechanism (Xác nhận tiêu thụ), Message Retry (Thử lại khi thất bại), Consumer Group (Nhóm tiêu thụ), Accumulation (Khả năng tích tụ tin nhắn trên đĩa), Order Guarantee (Đảm bảo thứ tự) và Scalability (Khả năng mở rộng).

Redis cung cấp 3 giai đoạn triển khai Message Queue:

### Giai đoạn 1: Dùng Data Structure List (Thời kỳ đầu)

Trước Redis 2.0, muốn làm MQ với Redis chỉ có thể dùng List bằng các lệnh `RPUSH/LPOP` hoặc `LPUSH/RPOP`.

Để tránh việc phải dùng vòng lặp liên tục gọi `LPOP` tốn tài nguyên, Redis cung cấp lệnh đọc chặn `BLPOP` / `BRPOP` (với tham số Timeout).

**Nhược điểm chí mạng**: List không hỗ trợ Broadcast (một tin nhắn cho nhiều Consumer cùng đọc). Tin nhắn khi rút ra sẽ bị xóa khỏi List, nếu Consumer xử lý thất bại thì tin nhắn bị mất vĩnh viễn.

### Giai đoạn 2: Đưa vào mô hình Pub/Sub (Publish/Subscribe)

Từ Redis 2.0, mô hình Pub/Sub giới thiệu khái niệm **Channel (Kênh)**.
- Nhà phát hành dùng `PUBLISH` gửi tin vào Channel.
- Người tiêu thụ dùng `SUBSCRIBE` đăng ký nhận tin từ Channel.

Nhiều Consumer có thể cùng Subscribe 1 Channel để nhận tin nhắn dạng Broadcast.

**Nhược điểm chí mạng**: Pub/Sub hoạt động theo cơ chế "Fire-and-forget" (Bắn rồi quên). Không có Persistence (không lưu trữ tin nhắn), không có ACK mechanism, không hỗ trợ tích tụ message. Nếu Consumer không online lúc phát tin, tin nhắn sẽ bị mất vĩnh viễn.

### Giai đoạn 3: Redis 5.0 bổ sung Data Type Stream

Redis 5.0 đưa vào **Stream**, một cấu trúc log tin nhắn có thứ tự dựa trên **Radix Tree (Cây cơ số)**, hỗ trợ Native Consumer Group và cơ chế ACK.

Tại sao dùng Radix Tree?: Nén bộ nhớ tối đa do gộp các tiền tố trùng nhau của Message ID (ví dụ `1625000000000-0`), và hỗ trợ tra cứu khoảng (`XRANGE`) siêu nhanh trên hàng triệu tin nhắn.

Stream học tập các khái niệm cốt lõi của Kafka:
1. **Consumer Groups (Nhóm tiêu thụ)**: Cân bằng tải tiêu thụ tin nhắn giữa nhiều Consumer, tự động failover.
2. **Persistence**: Bảo vệ bằng RDB và AOF.
3. **Cơ chế ACK**: Consumer sau khi xử lý xong phải gọi `XACK`, nếu chưa ACK tin nhắn sẽ lưu ở `Pending List` (PEL).
4. **Message Claim**: Cho phép dùng `XCLAIM` chuyển giao các tin nhắn chưa ACK từ Consumer bị crash sang Consumer khác xử lý.

Cấu trúc của Stream:

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-stream-structure.png)

Các lệnh phổ biến của Stream: `XADD`, `XREAD`, `XREADGROUP`, `XRANGE`, `XACK`, `XPENDING`, `XCLAIM`.

So sánh giữa Redis Stream và các MQ chuyên dụng:

| Tiêu chí | Redis Stream | RabbitMQ | Kafka |
| :------------- | :------------------------- | :------------------------------- | :---------------------------------- |
| **Throughput (Thông lượng)** | Cao (hàng trăm ngàn QPS) | Trung bình (hàng chục ngàn QPS) | **Cực cao (hàng triệu QPS với Partition)** |
| **Độ trễ** | **Cực thấp (sub-millisecond)** | **Thấp (microsecond/millisecond)** | Trung bình (millisecond) |
| **Persistence** | Hỗ trợ (Bất đồng bộ RDB/AOF) | Hỗ trợ (Đĩa cứng) | **Hỗ trợ mạnh (Ghi tuần tự đĩa)** |
| **Tích tụ tin nhắn** | Trung bình (bị giới hạn bởi RAM) | Trung bình (giảm hiệu năng khi tích tụ lớn) | **Cực mạnh (lưu đĩa TB, hiệu năng ổn định)** |
| **ACK & Reliablity** | Trung bình (có rủi ro mất 1s AOF) | **Cao (Confirm mechanism chín mùi)** | **Cực cao (Multi-replica + Strict Consistency)** |
| **Chi phí vận hành** | Thấp (dùng lại hạ tầng Redis) | Trung bình (cụm Erlang) | Cao (phụ thuộc Zookeeper / KRaft) |

### Tóm tắt

**Có nên dùng Redis làm Message Queue không?**

- **Nếu nghiệp vụ đơn giản, lượng nhỏ, tối ưu hiệu năng, chấp nhận rủi ro mất dữ liệu cực nhỏ**: Dùng **Redis Stream** là lựa chọn tối ưu nhất vì tiết kiệm chi phí vận hành MQ riêng và tái sử dụng hạ tầng Redis sẵn có.
- **Nếu là nghiệp vụ tài chính, dữ liệu khổng lồ, yêu cầu tuyệt đối không mất tin nhắn**: Bắt buộc phải chọn **Kafka, RabbitMQ**.

Các bài viết chuyên đề liên quan:
- [Tóm tắt câu hỏi phỏng vấn Redis phổ biến (Tập 1)](./redis-questions-01.md)
- [Tóm tắt câu hỏi phỏng vấn Redis phổ biến (Tập 2)](./redis-questions-02.md)
- [Hướng dẫn triển khai Delayed Task bằng Redis](./redis-delayed-task.md)
- [Chi tiết cơ chế Persistence của Redis](./redis-persistence.md)

<!-- @include: @article-footer.snippet.md -->
