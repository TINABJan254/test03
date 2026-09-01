---
title: MySQL三大日志(binlog、redo log和undo log)详解
description: 深入解析MySQL三大日志binlog、redo log和undo log的作用与原理，详解两阶段提交保证数据一致性的机制，以及日志在崩溃恢复和主从复制中的应用。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL日志,binlog,redo log,undo log,两阶段提交,崩溃恢复,主从复制,WAL,事务日志
---

> Bài viết đến từ đóng góp của kênh 程序猿阿星 (Programmer Ah Xing), JavaGuide đã bổ sung và hoàn thiện.

## Lời nói đầu

MySQL Log chủ yếu bao gồm Error Log, Query Log, Slow Query Log, Transaction Log, Binary Log. Trong đó, các log tương đối quan trọng phải kể đến Binary Log binlog (log lưu trữ/phục hồi), Transaction Log redo log (log ghi lại để thực hiện lại) và undo log (log hoàn tác).

![](https://oss.javaguide.cn/github/javaguide/01.png)

Hôm nay chúng ta cùng trò chuyện về redo log, binlog, Two-Phase Commit (cam kết hai giai đoạn) và undo log.

## redo log

redo log (log ghi lại để thực hiện lại) là tính năng riêng có của Storage Engine InnoDB, nó giúp MySQL sở hữu khả năng phục hồi sau sự cố (Crash Recovery).

Ví dụ khi instance MySQL bị sập hoặc ngắt điện bất ngờ, khi khởi động lại, Storage Engine InnoDB sẽ sử dụng redo log để khôi phục dữ liệu, đảm bảo tính bền vững (Durability) và tính toàn vẹn của dữ liệu.

![](https://oss.javaguide.cn/github/javaguide/02.png)

Trong MySQL, dữ liệu được quản lý theo đơn vị trang (page). Khi bạn query một bản ghi, MySQL sẽ load một trang dữ liệu từ đĩa cứng vào bộ nhớ, trang dữ liệu được load ra gọi là data page và được đưa vào `Buffer Pool`.

Các lần query sau đó đều tìm trong `Buffer Pool` trước, nếu không trúng (cache miss) mới ra đĩa cứng load, giúp giảm chi phí I/O đĩa, nâng cao hiệu năng.

Khi update dữ liệu bảng cũng vậy, nếu phát hiện dữ liệu cần update đã có trong `Buffer Pool`, sẽ update trực tiếp trong `Buffer Pool`.

Sau đó sẽ ghi nội dung "đã sửa đổi những gì trên data page nào" vào `redo log buffer`, tiếp theo mới flush xuống đĩa cứng ghi vào file redo log.

![](https://oss.javaguide.cn/github/javaguide/03.png)

> Ghi chú hình ảnh: Ở bước 4 "Clear redo log buffe flush down redo log", từ buffe là buffer.

Lý tưởng nhất là mỗi khi Transaction commit sẽ thực hiện flush đĩa, nhưng thực tế thời điểm flush đĩa được tiến hành theo chiến lược.

> Mẹo nhỏ: Mỗi bản ghi redo log gồm "Table Space No + Data Page No + Offset + Modified Length + Modified Data".

### Thời điểm Flush đĩa (刷盘时机)

Trong Storage Engine InnoDB, **redo log buffer** (vùng nhớ tạm redo log) là một vùng bộ nhớ tạm thời dùng để lưu trữ redo log. Để đảm bảo tính bền vững của Transaction và tính nhất quán của dữ liệu, InnoDB sẽ flush dữ liệu log từ buffer này xuống file redo log trên đĩa cứng vào các thời điểm nhất định. Các thời điểm này gồm 6 trường hợp:

1. **Khi Transaction commit (Cốt lõi nhất)**: Khi Transaction commit, redo log trong log buffer sẽ được flush xuống đĩa (thông qua tham số `innodb_flush_log_at_trx_commit` để kiểm soát).
2. **Khi dung lượng redo log buffer không đủ**: Đây là chiến lược quản lý dung lượng chủ động của InnoDB nhằm tránh việc thread người dùng bị chặn do buffer bị đầy.
   - Khi dung lượng đã dùng của redo log buffer vượt quá **50%** tổng dung lượng, thread chạy ngầm sẽ **chủ động** flush phần log này xuống đĩa.
   - Nếu vì Transaction quá lớn hoặc I/O bận rộn làm buffer bị **đầy hoàn toàn**, tất cả các thread người dùng cố gắng ghi log mới đều sẽ bị **chặn (blocked)** và ép buộc thực hiện một lần flush đĩa đồng bộ cho đến khi có bộ nhớ trống.
3. **Khi kích hoạt Checkpoint**: Checkpoint là cơ chế cốt lõi để rút ngắn thời gian Crash Recovery. Khi Checkpoint được kích hoạt, InnoDB cần flush tất cả dirty page trước Checkpoint này xuống đĩa. Theo nguyên tắc **Write-Ahead Logging (WAL)**, trước khi data page được ghi xuống đĩa, redo log tương ứng của nó phải được ghi xuống đĩa trước.
4. **Thread ngầm định kỳ flush**: InnoDB có một master thread chạy ngầm, khoảng mỗi 1 giây thực hiện công việc định kỳ, trong đó bao gồm flush redo log buffer xuống đĩa.
5. **Tắt server bình thường**: Trong quá trình tắt server MySQL bình thường, InnoDB sẽ thực hiện flush đĩa lần cuối để đưa tất cả log còn lại trong redo log buffer xuống đĩa.
6. **Khi chuyển đổi file binlog**: Khi bật binlog với cấu hình `innodb_flush_log_at_trx_commit=1` và `sync_binlog=1`, khi file binlog đầy hoặc chuyển đổi file binlog sẽ kích hoạt flush redo log.

Chiến lược flush đĩa `innodb_flush_log_at_trx_commit` có 3 giá trị:

- **0**: Khi thiết lập là 0, đại diện mỗi lần Transaction commit không thực hiện flush đĩa. Cách này hiệu năng cao nhất nhưng không an toàn nhất, nếu MySQL bị sập có thể mất dữ liệu trong 1 giây gần nhất.
- **1**: Khi thiết lập là 1, đại diện mỗi lần Transaction commit đều thực hiện flush đĩa. Cách này hiệu năng thấp nhất nhưng an toàn nhất, chỉ cần commit thành công thì redo log chắc chắn đã ở trên đĩa.
- **2**: Khi thiết lập là 2, đại diện mỗi lần Transaction commit chỉ ghi redo log từ log buffer vào page cache của hệ điều hành. Hiệu năng và độ an toàn nằm ở giữa 0 và 1.

Giá trị mặc định là 1. Để đảm bảo tính Durability của Transaction, bắt buộc phải đặt là 1.

Ngoài ra, master thread chạy ngầm của InnoDB mỗi 1 giây sẽ ghi nội dung trong `redo log buffer` vào `page cache`, sau đó gọi `fsync` để flush đĩa.

![](https://oss.javaguide.cn/github/javaguide/04.png)

Có nghĩa là, một bản ghi redo log của Transaction chưa commit cũng có thể bị flush xuống đĩa.

**Tại sao?**

Vì trong quá trình Transaction thực thi, bản ghi redo log được ghi vào `redo log buffer`, những bản ghi redo log này sẽ được thread ngầm flush xuống đĩa.

![](https://oss.javaguide.cn/github/javaguide/05.png)

Ngoài việc thread ngầm luân chuyển 1 giây/lần, khi `redo log buffer` chiếm gần 50% `innodb_log_buffer_size`, thread ngầm cũng sẽ chủ động flush đĩa.

Dưới đây là sơ đồ quy trình của các chiến lược flush đĩa:

#### innodb_flush_log_at_trx_commit=0

![](https://oss.javaguide.cn/github/javaguide/06.png)

Khi là 0, nếu MySQL sập có thể mất 1 giây dữ liệu.

#### innodb_flush_log_at_trx_commit=1

![](https://oss.javaguide.cn/github/javaguide/07.png)

Khi là 1, chỉ cần Transaction commit thành công thì bản ghi redo log chắc chắn đã nằm trên đĩa cứng, không lo mất dữ liệu.

#### innodb_flush_log_at_trx_commit=2

![](https://oss.javaguide.cn/github/javaguide/09.png)

Khi là 2, chỉ cần Transaction commit thành công, nội dung trong `redo log buffer` chỉ được ghi vào `page cache`. Nếu chỉ riêng MySQL sập thì không mất dữ liệu, nhưng nếu cả máy OS sập thì có thể mất 1 giây dữ liệu.

### Nhóm file Log (Log File Group)

Các file redo log lưu trên đĩa không phải chỉ có một file, mà tồn tại dưới dạng một **Log File Group** (nhóm file log), mỗi file redo log đều có dung lượng bằng nhau.

Ví dụ cấu hình 1 nhóm gồm 4 file, dung lượng mỗi file là 1GB, toàn bộ Log File Group có thể ghi 4GB nội dung.

Nó áp dụng dạng mảng vòng (circular buffer), ghi từ đầu đến cuối rồi quay lại đầu ghi đè tiếp.

![](https://oss.javaguide.cn/github/javaguide/10.png)

Trong **Log File Group** có 2 con trỏ quan trọng: `write pos` và `checkpoint`.

- **write pos** là vị trí ghi hiện tại, vừa ghi vừa dịch về sau.
- **checkpoint** là vị trí cần xóa hiện tại, cũng dịch về sau.

![](https://oss.javaguide.cn/github/javaguide/11.png)

Nếu `write pos` đuổi kịp `checkpoint`, đại diện **Log File Group** đã đầy, lúc này không thể ghi thêm bản ghi redo log mới, MySQL phải dừng lại để xóa bớt bản ghi và đẩy `checkpoint` lên trước.

![](https://oss.javaguide.cn/github/javaguide/12.png)

Lưu ý từ MySQL 8.0.30 trở đi, nhóm file log có sự thay đổi: Biến `innodb_redo_log_capacity` thay thế cho `innodb_log_files_in_group` và `innodb_log_file_size`. Số lượng file trong nhóm cố định là 32 file, kích thước mỗi file là `innodb_redo_log_capacity / 32`.

### Tóm tắt redo log

Tại sao không flush trực tiếp data page đã sửa đổi xuống đĩa mà phải thông qua redo log?

Kích thước data page là `16KB`, việc flush data page rất tốn thời gian vì đó là ghi ngẫu nhiên (random write) trên đĩa cứng.

Còn redo log chỉ chiếm vài chục byte cho mỗi bản ghi và là ghi tuần tự (sequential write), do đó tốc độ flush đĩa cực kỳ nhanh, giúp khả năng xử lý đồng thời của database mạnh mẽ hơn nhiều.

## binlog

redo log là log vật lý (physical log) ghi lại "đã sửa đổi những gì trên data page nào", thuộc về Storage Engine InnoDB.

Còn binlog là log logic (logical log) ghi lại logic câu lệnh ban đầu, thuộc về tầng `MySQL Server`.

Bất kể dùng Storage Engine nào, chỉ cần phát sinh update dữ liệu bảng đều tạo ra binlog.

binlog phục vụ cho **Backup dữ liệu, Master-Slave Replication, Master-Master Replication**, giúp đồng bộ dữ liệu và đảm bảo tính nhất quán dữ liệu giữa các node.

![](https://oss.javaguide.cn/github/javaguide/01-20220305234724956.png)

### Định dạng ghi (binlog_format)

- **statement**: Ghi lại câu SQL nguyên văn.
- **row**: Ghi lại dữ liệu chi tiết của từng hàng trước và sau thay đổi (khuyên dùng để đảm bảo tính chính xác và nhất quán).
- **mixed**: Kết hợp giữa statement và row, MySQL tự phán đoán câu SQL có gây ra sai lệch dữ liệu hay không để chọn format thích hợp.

### Cơ chế ghi

Trong quá trình thực thi Transaction, ghi vào `binlog cache` trước, khi Transaction commit mới ghi `binlog cache` vào file binlog.

Tham số `sync_binlog` kiểm soát flush đĩa:
- **0**: Mỗi lần commit chỉ `write` vào page cache, hệ điều hành tự quyết định khi nào `fsync`.
- **1**: Mỗi lần commit đều `fsync` xuống đĩa.
- **N (N > 1)**: Tích lũy N transaction `write` rồi mới `fsync`.

![](https://oss.javaguide.cn/github/javaguide/04-20220305234747840.png)

## Two-Phase Commit (Cam kết hai giai đoạn)

redo log mang lại khả năng Crash Recovery cho InnoDB.
binlog đảm bảo tính nhất quán dữ liệu cho kiến trúc MySQL Cluster (Master-Slave).

Trong quá trình thực thi câu lệnh update, redo log được ghi trong suốt quá trình Transaction, còn binlog chỉ được ghi khi commit Transaction.

Để giải quyết vấn đề không nhất quán logic giữa hai loại log, InnoDB sử dụng cơ chế **Two-Phase Commit**:
1. Phase 1: Ghi redo log ở trạng thái `prepare`.
2. Ghi binlog.
3. Phase 2: Chuyển redo log sang trạng thái `commit`.

![](https://oss.javaguide.cn/github/javaguide/04-20220305234956774.png)

Nếu sập khi binlog chưa ghi xong: redo log ở trạng thái `prepare` và không có binlog tương ứng -> MySQL sẽ rollback Transaction.

Nếu sập ở giai đoạn redo log `commit`: redo log ở trạng thái `prepare` nhưng tìm thấy binlog tương ứng -> MySQL coi Transaction là hoàn chỉnh và commit để phục hồi dữ liệu.

## undo log

Mỗi sửa đổi dữ liệu của Transaction đều được ghi vào undo log. Khi phát sinh lỗi hoặc thực hiện rollback, MySQL dùng undo log để khôi phục dữ liệu về trạng thái trước khi Transaction bắt đầu.

undo log là log logic, phục vụ cho **Atomicity (Rollback)** và **MVCC (Snapshot Read)**.

undo log được chia thành:
- `insert undo log`: Có thể xóa ngay sau khi commit.
- `update undo log`: Phục vụ MVCC, được đưa vào history list để purge thread dọn dẹp sau.

## Tóm tắt

- MySQL InnoDB dùng **redo log** để đảm bảo tính **Durability** (Bền vững).
- Dùng **undo log** để đảm bảo tính **Atomicity** (Nguyên tố / Rollback).
- Dùng **binlog** cho Backup, Master-Slave Replication và đảm bảo tính nhất quán dữ liệu.

## Tham khảo

- 《MySQL 实战 45 讲》
- 《从零开始带你成为 MySQL 实战优化高手》
- 《MySQL 是怎样运行的：从根儿上理解 MySQL》
- 《MySQL 技术 Innodb 存储引擎》

<!-- @include: @article-footer.snippet.md -->
