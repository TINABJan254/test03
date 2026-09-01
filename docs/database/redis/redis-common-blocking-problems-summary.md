---
title: Redis常见阻塞原因总结
description: 全面总结Redis常见的阻塞原因，包括O(n)复杂度命令、bigkey操作、AOF日志刷盘、RDB快照创建、主从同步等场景，帮助你排查和预防Redis性能问题。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis阻塞,Redis性能问题,O(n)命令,bigkey,AOF刷盘,RDB快照,主从同步,内存达上限
---

Bài viết này tổng hợp chi tiết các kịch bản có thể dẫn tới nghẽn (Blocking) trong Redis. Đây cũng là các yếu tố then chốt ảnh hưởng tới hiệu năng của Redis mà bạn cần đặc biệt chú ý trong quá trình phát triển và vận hành!

## Lệnh độ phức tạp O(N)

Đa số lệnh trong Redis có độ phức tạp $O(1)$, nhưng cũng có một số lệnh có độ phức tạp $O(N)$ như:

- `KEYS *`: Trả về tất cả các key phù hợp quy tắc.
- `HGETALL`: Trả về tất cả các cặp field-value trong một Hash.
- `LRANGE`: Trả về danh sách phần tử trong phạm vi chỉ định của List.
- `SMEMBERS`: Trả về tất cả phần tử trong Set.
- `SINTER` / `SUNION` / `SDIFF`: Tính tập giao/hợp/hiệu của các Set.

Do các lệnh này có độ phức tạp $O(N)$ và quét toàn bộ dữ liệu, khi $N$ tăng lên thời gian thực thi sẽ kéo dài và làm nghẽn client. Nếu cần duyệt dữ liệu, hãy thay thế bằng `HSCAN`, `SSCAN`, `ZSCAN`.

Ngoài ra, một số lệnh có độ phức tạp $O(\log N + M)$ trên Zset như `ZRANGE`, `ZREVRANGE`, `ZREMRANGEBYRANK`, `ZREMRANGEBYSCORE` khi $M$ (số phần tử lấy/xóa) và $N$ lớn cũng có thể gây ra hiện tượng nghẽn.

## Lệnh SAVE tạo RDB Snapshot

Redis cung cấp 2 lệnh tạo file RDB Snapshot:

- `save`: Thao tác lưu đồng bộ, **gây nghẽn Main Thread** của Redis.
- `bgsave`: Fork ra một child process chạy ngầm, không nghẽn Main Thread (lựa chọn mặc định).

Tránh sử dụng lệnh `save` trực tiếp trong môi trường production.

## AOF Persistence

### Nghẽn do Ghi log AOF

AOF ghi log SAU KHI câu lệnh đã được thực thi trên RAM. Do thao tác ghi log AOF diễn ra trên Main Thread của Redis, nếu I/O ghi log bị chậm sẽ **gây nghẽn việc thực thi các câu lệnh tiếp theo**.

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-aof-write-log-disc.png)

### Nghẽn do AOF Flush đĩa (fsync)

Khi cài đặt `appendfsync everysec`, thread ngầm `aof_fsync` sẽ gọi hàm `fsync` mỗi giây một lần. Nếu đĩa cứng bị nghẽn I/O nghiêm trọng khiến `fsync` bị treo, Main Thread khi gọi hàm `write()` để nạp dữ liệu mới vào AOF Buffer cũng sẽ bị **khóa nghẽn theo** để đợi `fsync` hoàn thành.

### Nghẽn do AOF Rewrite

Khi child process hoàn thành việc tạo file AOF mới trong quá trình `BGREWRITEAOF`, Server sẽ ghi nốt toàn bộ dữ liệu tích lũy từ AOF Rewrite Buffer vào đuôi file mới. Thao tác append dữ liệu cuối cùng này diễn ra trên Main Thread và có thể gây nghẽn ngắn hạn.

## Big Key (Key dung lượng lớn)

Key kiểu String > 1MB hoặc Composite Collection > 5000 phần tử.

Tác hại gây nghẽn của Big Key:
- **Nghẽn Client Timeout**: Thao tác đọc/sửa Big Key tốn thời gian làm nghẽn Main Thread, làm Client chờ lâu.
- **Nghẽn băng thông mạng**: Lấy Big Key tốn băng thông lớn (1MB/key * 1000 QPS = 1GB/s băng thông).
- **Nghẽn Worker Thread khi xóa**: Lệnh `DEL` trên Big Key khiến hệ điều hành mất nhiều thời gian thu hồi bộ nhớ và chèn các block RAM rỗng vào danh sách Free List, làm ngắt quãng Main Thread.

### Tìm Big Key

Sử dụng `redis-cli --bigkeys` nên chạy trên Node Slave (Replica) để tránh nghẽn Node Master. Hoặc dùng `SCAN` + `MEMORY USAGE`, phân tích file RDB offline bằng `rdb_bigkeys`.

### Xóa Big Key

Tuyệt đối không dùng `DEL` trực tiếp. Nên dùng `UNLINK` (xóa bất đồng bộ từ Redis 4.0+) hoặc chia nhỏ tập hợp để xóa theo batch (`SCAN` + `DEL`).

## Lệnh FLUSHDB / FLUSHALL

Lệnh `FLUSHDB` và `FLUSHALL` xóa toàn bộ Key trong Database và giải phóng RAM, gây ra hiện tượng nghẽn Main Thread tương tự như việc xóa Big Key. Nên sử dụng `FLUSHDB ASYNC` hoặc `FLUSHALL ASYNC`.

## Mở rộng/Thu hẹp Cluster (Resharding)

Việc di chuyển Slot giữa các Node trong Redis Cluster hiện tại là thao tác đồng bộ. Khi di chuyển một Key có dung lượng quá lớn (Big Key), cả 2 Node liên quan sẽ rơi vào trạng thái nghẽn, nghiêm trọng hơn có thể kích hoạt Failover không mong muốn trong Cluster.

## Swap Memory (Bộ nhớ ảo trên đĩa)

Khi RAM vật lý bị thiếu, hệ điều hành Linux sẽ đẩy một phần dữ liệu RAM của Redis xuống đĩa cứng (Swap Partition). Vì tốc độ đọc ghi đĩa chậm hơn RAM hàng ngàn lần, việc dính Swap sẽ làm hiệu năng của Redis bị sụt giảm thảm hại.

Phòng tránh Swap:
- Đảm bảo server đủ dung lượng RAM.
- Luôn thiết lập `maxmemory` cho mọi Instance Redis.
- Giảm ưu tiên Swap của hệ thống: `echo 10 > /proc/sys/vm/swappiness`.

## Tranh chấp CPU

Redis là ứng dụng CPU-intensive trên 1 thread chính. Tránh đặt Redis chung máy với các dịch vụ tiêu tốn nhiều CPU khác.

## Sự cố Mạng

Kết nối bị từ chối (Connection Refused), Network Latency cao, ngắt mềm card mạng (Soft Interrupt) cũng có thể làm nghẽn các thao tác Redis.

<!-- @include: @article-footer.snippet.md -->
