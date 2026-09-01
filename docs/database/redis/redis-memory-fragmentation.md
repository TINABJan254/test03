---
title: Redis内存碎片详解
description: 深入解析Redis内存碎片产生的原因、判断方法和优化方案，包括内存碎片率计算、jemalloc分配器原理、自动内存碎片清理配置等。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis内存碎片,内存碎片率,jemalloc,内存分配,activedefrag,内存优化,Redis内存管理
---

## Mảnh bộ nhớ (Memory Fragmentation) là gì?

Có thể hiểu đơn giản mảnh bộ nhớ là những khoảng bộ nhớ rỗng không thể tái sử dụng để lưu trữ dữ liệu khác.

Ví dụ: Hệ điều hành cấp cho bạn 32 bytes bộ nhớ liên tục, nhưng dữ liệu thực tế chỉ dùng 24 bytes. 8 bytes thừa ra nếu sau đó không thể phân bổ cho dữ liệu khác thì được gọi là mảnh bộ nhớ.

![](https://oss.javaguide.cn/github/javaguide/memory-fragmentation.png)

Mảnh bộ nhớ không ảnh hưởng trực tiếp đến tốc độ xử lý của Redis nhưng làm gia tăng dung lượng RAM tiêu thụ.

## Tại sao lại phát sinh mảnh bộ nhớ trong Redis?

Dưới đây là 2 nguyên nhân chính:

**1. Bộ nhớ xin cấp phát từ hệ điều hành lớn hơn dung lượng dữ liệu thực tế.**

Hàm `zmalloc` do Redis tự viết khi cấp phát bộ nhớ ngoài dung lượng `size` chỉ định sẽ cộng thêm `PREFIX_SIZE`.

Ngoài ra, Redis mặc định sử dụng bộ phân phối bộ nhớ **jemalloc**. jemalloc phân chia các block bộ nhớ theo các kích thước cố định (8, 16, 32 bytes...). Khi ứng dụng xin 17 bytes, jemalloc sẽ cấp phát ngay block 32 bytes, làm lãng phí 15 bytes rỗng.

![](https://oss.javaguide.cn/github/javaguide/database/redis/6803d3929e3e46c1b1c9d0bb9ee8e717.png)

**2. Thường xuyên sửa đổi hoặc xóa dữ liệu trong Redis.**

Khi một dữ liệu bị xóa, Redis thông thường không lập tức trả ngay phần bộ nhớ đó về cho hệ điều hành.

![](https://oss.javaguide.cn/github/javaguide/redis-docs-memory-optimization.png)

## Làm thế nào để kiểm tra thông tin mảnh bộ nhớ trong Redis?

Dùng lệnh `info memory` để xem thông tin liên quan tới bộ nhớ:

![](https://oss.javaguide.cn/github/javaguide/redis-info-memory.png)

Công thức tính tỷ lệ mảnh bộ nhớ (`mem_fragmentation_ratio`):

$$\text{mem\_fragmentation\_ratio} = \frac{\text{used\_memory\_rss}}{\text{used\_memory}}$$

- `used_memory_rss`: Bộ nhớ vật lý thực tế mà hệ điều hành cấp cho Redis.
- `used_memory`: Bộ nhớ mà allocator của Redis đăng ký sử dụng để lưu dữ liệu.

Tỷ lệ `mem_fragmentation_ratio` càng lớn thì mảnh bộ nhớ càng nghiêm trọng.

Thông thường, khi **`mem_fragmentation_ratio > 1.5`** nghĩa là cần phải dọn dẹp mảnh bộ nhớ (ví dụ lưu 2GB data nhưng tiêu tốn hơn 3GB RAM vật lý).

Xem nhanh chỉ số mảnh bộ nhớ qua lệnh:

```bash
> redis-cli -p 6379 info | grep mem_fragmentation_ratio
```

## Dọn dẹp mảnh bộ nhớ trong Redis như thế nào?

Từ phiên bản Redis 4.0-RC3+, Redis hỗ trợ tính năng dọn dẹp mảnh bộ nhớ tự động (`activedefrag`).

Bật tính năng này bằng lệnh `config set`:

```bash
config set activedefrag yes
```

Cấu hình điều kiện tự động dọn dẹp:

```bash
# Dung lượng mảnh bộ nhớ đạt 500MB mới bắt đầu dọn dẹp
config set active-defrag-ignore-bytes 500mb
# Tỷ lệ mảnh bộ nhớ vượt 1.5 mới bắt đầu dọn dẹp
config set active-defrag-threshold-lower 50
```

Giới hạn phần trăm CPU cho việc dọn dẹp để tránh ảnh hưởng tới hiệu năng:

```bash
# CPU tối thiểu dành cho dọn dẹp mảnh bộ nhớ là 20%
config set active-defrag-cycle-min 20
# CPU tối đa dành cho dọn dẹp mảnh bộ nhớ là 50%
config set active-defrag-cycle-max 50
```

Ngoài ra, việc khởi động lại Instance Redis cũng giúp thu hồi toàn bộ mảnh bộ nhớ. Trong mô hình Cluster/Sentinel High Availability, bạn có thể hoán đổi Node Master thành Slave rồi tiến hành Safe Restart.

<!-- @include: @article-footer.snippet.md -->
