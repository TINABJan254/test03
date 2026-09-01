---
title: Redis持久化机制详解
description: 深入解析Redis三种持久化机制RDB快照、AOF日志和混合持久化的工作原理、配置方法和优缺点对比，帮助你选择适合业务场景的持久化策略。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis持久化,RDB,AOF,混合持久化,bgsave,数据恢复,Redis备份,fork子进程
---

Khi sử dụng Cache, chúng ta thường cần đưa dữ liệu trong RAM xuống lưu trữ bền vững trên đĩa cứng (Persistence). Mục đích chính là để tái sử dụng dữ liệu sau khi khởi động lại máy/gặp sự cố crash, hoặc để đồng bộ dữ liệu (như Slave khôi phục từ Master trong Redis Cluster qua file RDB).

Khác với Memcached, Redis hỗ trợ 3 cơ chế Persistence:

- 快照 (Snapshotting - **RDB**)
- 只追加文件 (Append-Only File - **AOF**)
- RDB và AOF 混合持久化 (Hybrid Persistence - từ Redis 4.0)

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis4.0-persitence.png)

Bảng khác biệt giữa các phiên bản Redis:

| Phiên bản | Mặc định | Tính năng quan trọng |
| -------------- | -------------- | ----------------------- |
| **Redis 4.0** | RDB | Đưa vào RDB + AOF Hybrid Persistence |
| **Redis 6.0** | RDB | AOF vẫn cần bật thủ công |
| **Redis 7.0** | RDB | Đưa vào Multi-Part AOF |
| **Redis 7.2+** | RDB | Tiếp tục tối ưu hiệu năng Persistence |

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-persistence-flow.png)

## RDB Persistence

### RDB Persistence là gì?

Redis tạo bản sao Snapshot dữ liệu bộ nhớ tại một **thời điểm nhất định**. File RDB rất nhỏ gọn, thích hợp cho việc backup, khôi phục thảm họa hoặc đồng bộ Master-Slave.

Cấu hình mặc định trong `redis.conf`:

```clojure
# Cấu hình mặc định Redis 7.0
save 3600 1 300 100 60 10000
```
- Trong 3600s có ít nhất 1 key bị sửa đổi.
- Trong 300s có ít nhất 100 key bị sửa đổi.
- Trong 60s có ít nhất 10000 key bị sửa đổi.

### Tạo Snapshot RDB có gây nghẽn Main Thread không?

Redis cung cấp 2 câu lệnh tạo RDB Snapshot:

- `save`: Thực thi đồng bộ, **gây nghẽn Main Thread** cho đến khi hoàn thành.
- `bgsave`: `fork` ra một child process chạy ngầm để tạo file RDB.

#### Phân tích chi phí hiệu năng `fork`

Mặc dù `bgsave` chạy ở child process không ngắt câu lệnh của main thread, nhưng **bản thân thao tác `fork` gây nghẽn Main Thread** và tiêu tốn thêm tài nguyên bộ nhớ:

| Dung lượng Dataset | Độ trễ `fork` | RAM tốn thêm (Copy-on-Write) | Mức độ rủi ro |
| ---------- | --------- | ---------------- | -------- |
| < 1GB | < 10ms | ~10MB (Copy Page Table) | Thấp |
| 1-10GB | 10-100ms | 10-100MB | Trung bình |
| 10-50GB | 100ms-1s | 100-500MB | Cao |
| > 50GB | > 1s | > 500MB | Cực cao |

#### Cơ chế Copy-on-Write (COW)

Khi child process được `fork`, nó chia sẻ các trang nhớ (Page 4KB) với parent process. Khi parent process sửa đổi dữ liệu trên RAM, hệ điều hành Kernel sẽ sao chép lại trang nhớ đó (Copy-on-Write). Nếu lượng ghi quá lớn trên dataset khổng lồ, COW sẽ tiêu tốn thêm rất nhiều RAM.

#### Nguy cơ nổ bộ nhớ do THP (Transparent Huge Pages)

Mặc định Linux bật THP với kích thước trang nhớ lên tới 2MB. Nếu bị sửa đổi dù chỉ 10 bytes, Kernel phải copy cả 2MB trang nhớ (phóng đại thêm 512 lần). Dưới tải ghi cao, điều này có thể lập tức làm cạn kiệt RAM và kích hoạt **OOM Killer giết chết tiến trình Redis**.

**Giải pháp**: Tắt THP trước khi chạy Redis bằng lệnh `echo never > /sys/kernel/mm/transparent_hugepage/enabled` hoặc thêm `redis-server --disable-thp yes` (từ Redis 6.0+).

#### Đề xuất trên môi trường Production

1. Giám sát rủi ro fork: `redis-cli INFO memory | grep -E "(used_memory|used_memory_rss)"`. Tỷ lệ `RSS/USED` khi fork nên < 2.
2. Thiết lập `maxmemory` và `maxmemory-policy` để dành không gian cho COW.
3. Tránh gọi `BGSAVE` thủ công vào giờ cao điểm.
4. Chuyển thao tác Persistence sang các Node Replica (Slave) trong kiến trúc Master-Slave.

## AOF Persistence

### AOF Persistence là gì?

AOF (Append-Only File) ghi lại toàn bộ các câu lệnh làm thay đổi dữ liệu trong Redis vào file nhật ký. AOF không bật mặc định, cần cài đặt:

```bash
appendonly yes
```

Mỗi câu lệnh ghi được gửi tới sẽ được ghi vào AOF Buffer `server.aof_buf`, sau đó đẩy sang file AOF (trong Kernel Buffer) rồi tùy theo chiến lược `appendfsync` để flush xuống đĩa cứng.

### Quy trình làm việc của AOF

1. **Command Append**: 追加 lệnh ghi vào AOF Buffer.
2. **File Write**: Ghi dữ liệu từ AOF Buffer sang Kernel Buffer bằng hàm `write()`.
3. **File Sync**: Gọi hàm `fsync()` ép buộc ghi dữ liệu từ Kernel Buffer xuống đĩa cứng.
4. **File Rewrite**: Tự động thu gọn AOF Rewrite khi file quá lớn.
5. **Load Data**: Khôi phục lại dữ liệu bằng cách chạy lại các lệnh trong AOF khi restart.

### 3 chiến lược `appendfsync`

1. `appendfsync always`: Main thread gọi `fsync` ngay sau mỗi câu lệnh ghi. An toàn tuyệt đối nhưng làm sụt giảm nghiêm trọng hiệu năng.
2. `appendfsync everysec` (Khuyên dùng): Main thread trả về ngay sau `write()`, thread ngầm `aof_fsync` gọi `fsync` 1 giây 1 lần. Cân bằng tuyệt vời giữa hiệu năng và an toàn (mất tối đa 1-2s dữ liệu nếu sập máy).
3. `appendfsync no`: Để hệ điều hành tự flush đĩa (thường là 30s một lần). Tốc độ nhanh nhất nhưng rủi ro mất dữ liệu không kiểm soát được.

### Tại sao AOF lại ghi log SAU KHI thực thi câu lệnh?

Khác với MySQL (WAL - ghi log trước khi sửa DB), Redis ghi AOF sau khi câu lệnh thực thi xong:
- Tránh tốn CPU kiểm tra cú pháp trước khi ghi log.
- Không làm ngắt quãng câu lệnh hiện tại.
- Rủi ro: Nếu sập ngay sau khi vừa sửa DB thì lệnh đó chưa kịp ghi vào AOF; Có thể gây nghẽn các lệnh phía sau nếu thao tác ghi log bị chậm.

### AOF Rewrite là gì?

Khi file AOF quá lớn, Redis chạy ngầm child process để thực hiện **AOF Rewrite** (tạo file AOF mới nhỏ gọn bằng cách quét trạng thái DB hiện tại mà không đọc file AOF cũ).

Cấu hình tự động rewrite:
- `auto-aof-rewrite-min-size 64mb` (kích thước tối thiểu để rewrite).
- `auto-aof-rewrite-percentage 100` (tăng gấp đôi kích thước so với lần rewrite trước thì kích hoạt).

Từ Redis 7.0+, cơ chế **Multi-Part AOF** chia AOF thành file BASE (snapshot toàn bộ), INCR (nhật ký tăng giọt) và file manifest quản lý.

## Hybrid Persistence (RDB + AOF)

Từ Redis 4.0, Redis hỗ trợ kết hợp RDB và AOF (**Hybrid Persistence**). Từ Redis 7.0+, tính năng này **mặc định được bật**:

```bash
appendonly yes
aof-use-rdb-preamble yes
```

### Nguyên lý làm việc

Khi AOF Rewrite diễn ra, Redis sẽ ghi nội dung dữ liệu dưới dạng **RDB Snapshot vào phần đầu của file AOF**, sau đó ghi tiếp các lệnh AOF tăng giọt vào phần đuôi file.

### Ưu điểm
- Khôi phục dữ liệu siêu nhanh (nhanh gấp 5 - 10 lần AOF thuần) nhờ phần đầu là RDB.
- Tránh mất nhiều dữ liệu nhờ phần đuôi là AOF tăng giọt.

## Lựa chọn giữa RDB và AOF

**RDB vượt trội hơn AOF ở điểm nào?**
- File nén nhỏ gọn, rất phù hợp cho Backup và Disaster Recovery.
- Tốc độ khôi phục cực nhanh (chỉ cần load snapshot binary).
- Tốc độ đồng bộ Master-Slave nhanh hơn.
- Không tiêu tốn I/O đĩa cứng của parent process (do child process làm hoàn toàn).

**AOF vượt trội hơn RDB ở điểm nào?**
- Độ an toàn dữ liệu cao hơn nhiều (chỉ mất tối đa 1-2s dữ liệu với `everysec`).
- Tương thích tốt qua các phiên bản Redis.
- File log dạng text dễ đọc, dễ sửa thủ công (nếu lỡ gõ `FLUSHALL` có thể mở file AOF xóa lệnh đó đi rồi khôi phục lại).

**Đề xuất chọn lựa theo kịch bản**:

| Kịch bản | Giải pháp khuyên dùng | Ghi chú |
| ---------------- | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Cache thuần túy** | **Tắt Persistence** hoặc RDB tần suất thấp | Tắt hoàn toàn để đạt hiệu năng cao nhất |
| **Dữ liệu quan trọng trung bình** | **RDB + AOF Hybrid Persistence** (`everysec`) | RDB tăng tốc khôi phục, AOF giữ an toàn 1s |
| **Dữ liệu cực kỳ quan trọng** | **RDB + AOF (Multi-Part AOF - Redis 7.0+)** | Redis làm cache, DB chính vẫn là MySQL |
| **Kiến trúc Master-Slave** | **Master tắt Persistence, Slave bật AOF** | Master cấm auto-restart để tránh ghi đè data rỗng lên Slave |

<!-- @include: @article-footer.snippet.md -->
