---
title: MySQL备份与恢复详解：mysqldump、XtraBackup、binlog和PITR
description: MySQL备份与恢复详解，讲解 mysqldump、MySQL Shell、Percona XtraBackup、binlog、PITR、RTO/RPO、恢复演练和常见备份误区。
category: 数据库
tag:
  - MySQL
  - 备份恢复
head:
  - - meta
    - name: keywords
      content: MySQL备份,MySQL恢复,mysqldump,mysqlbinlog,MySQL Shell,Percona XtraBackup,binlog,PITR,全量备份,增量备份,逻辑备份,物理备份,RTO,RPO
---

Sự cố cơ sở dữ liệu đáng sợ nhất trên production thường không phải là MySQL process bị sập.

Process sập còn có thể khởi động lại, Master sập còn có thể chuyển sang Slave. Điều thực sự rắc rối là dữ liệu bị xóa, bị script chạy lỗi làm sai lệch, ổ đĩa bị hỏng, hoặc khi migration phát hiện thiếu một loạt bảng. Lúc này Master-Slave Replication, redo log, undo log đều không cứu nổi, thứ duy nhất có thể cứu vãn là phương án Sao lưu (Backup) và Khôi phục (Restore).

Bài viết này tập trung vào Sao lưu & Khôi phục MySQL. Các câu lệnh được hiệu đối theo MySQL 8.4 LTS và tham khảo tài liệu MySQL 9.7 mới nhất; các phiên bản khác nhau có thể thay đổi tên tham số, yêu cầu quyền hạn và tính tương thích của công cụ.

## Backup thực sự giải quyết vấn đề gì?

Hãy làm rõ vài khái niệm dễ bị nhầm lẫn:

- **Crash Recovery (Phục hồi sau sập)**: Sau khi MySQL bị tắt đột ngột, InnoDB dựa vào redo log, undo log để đưa database về trạng thái nhất quán. Giải quyết vấn đề nhất quán Storage Engine sau khi process sập / restart máy.
- **Master-Slave Replication / High Availability Failover (Sao chép Master-Slave / Chuyển đổi HA)**: Khi Master không khả dụng, chuyển traffic sang Slave hoặc Master mới. Giải quyết vấn đề tính khả dụng của dịch vụ, nhưng câu lệnh ghi sai sẽ lập tức được đồng bộ sang Slave.
- **Backup & Restore (Sao lưu & Khôi phục)**: Phục hồi từ một bản sao dữ liệu lịch sử, sau đó replay binlog đến một mốc thời gian chỉ định. Giải quyết các vấn đề mất dữ liệu, xóa/sửa nhầm, hỏng đĩa, migration xuyên môi trường và lưu trữ kiểm toán.

Master-Slave Replication không thể thay thế cho Backup!

Nếu một lệnh `DROP TABLE` thực thi thành công trên Master, nó chắc chắn sẽ đồng bộ ngay sang Slave. Replication càng nhanh thì sai lầm lan truyền càng nhanh. Giá trị của Backup nằm ở việc giữ lại một trạng thái lịch sử độc lập, cho bạn cơ hội quay trở lại trước thời điểm xảy ra sự cố.

## RTO và RPO quyết định chiến lược Backup

Chiến lược Backup không thể chỉ hỏi "mỗi ngày backup mấy lần". Câu hỏi thực tế hơn là 2 chỉ số:

- **RPO (Recovery Point Objective - Mục tiêu điểm khôi phục)**: Chấp nhận mất tối đa dữ liệu trong bao lâu?
- **RTO (Recovery Time Objective - Mục tiêu thời gian khôi phục)**: Chấp nhận mất tối đa bao lâu để khôi phục dịch vụ?

Nếu nghiệp vụ chấp nhận mất 1 ngày dữ liệu, mỗi ngày 1 lần Full Backup có thể đủ dùng. Nếu các dữ liệu như đơn hàng, thanh toán, kho vận chỉ được phép mất vài phút, thì chỉ Full Backup thôi là không đủ mà còn phải lưu giữ binlog để làm Incremental Restore (Khôi phục tăng lượng). Nếu DB có hàng trăm GB, việc restore file SQL có thể chạy rất lâu, mà RTO lại yêu cầu khôi phục trong 30 phút, thì phải nghiêm túc cân nhắc Physical Backup, pre-heat Slave, diễn tập khôi phục và quy trình failover.

Một bộ kết hợp phổ biến:

- Mỗi ngày hoặc mỗi tuần làm Full Backup 1 lần.
- Bật binlog và giữ log đủ dài theo yêu cầu RPO.
- File backup và binlog không đặt trên cùng một ổ đĩa / cùng một fault domain.
- Định kỳ restore bản backup sang máy mới để ghi nhận thời gian thực tế.

## Các phương pháp Backup

Phân loại theo việc MySQL có đang phục vụ hay không:

| Loại | Mô tả | Kịch bản áp dụng |
| ---- | -------------------------------------- | ---------------------------------------- |
| Cold Backup (Sao lưu lạnh) | Tắt MySQL rồi copy data file | Hệ thống nhỏ, bảo trì có window thời gian dư dả |
| Warm Backup (Sao lưu ấm) | Backup khi MySQL chạy nhưng có thể khóa bảng | Yêu cầu khả dụng vừa phải, chấp nhận ảnh hưởng ngắn |
| Hot Backup (Sao lưu nóng) | Backup khi MySQL chạy, tối đa không block đọc ghi | Production, DB lớn, window bảo trì rất ngắn |

Phân loại theo nội dung file backup:

| Loại | Mô tả | Công cụ tiêu biểu |
| -------- | -------------------------------- | ------------------------------------------- |
| Logical Backup (Sao lưu logic) | Export các nội dung logic SQL, CSV | `mysqldump`, MySQL Shell dump utilities |
| Physical Backup (Sao lưu vật lý) | Copy data file, log file vật lý | Percona XtraBackup, MySQL Enterprise Backup |

Phân loại theo phạm vi backup:

| Loại | Mô tả |
| -------- | -------------------------------- |
| Full Backup (Sao lưu toàn lượng) | Backup toàn bộ dữ liệu tại một thời điểm |
| Incremental Backup (Sao lưu tăng lượng) | Backup dữ liệu/log thay đổi sau lần backup trước |
| Differential Backup (Sao lưu sai biệt) | Backup dữ liệu thay đổi sau lần Full Backup gần nhất |

## Dùng mysqldump làm Logical Backup

`mysqldump` là công cụ Logical Backup tự có của MySQL, nó export ra các câu SQL để rebuild lại cấu trúc và dữ liệu bảng. Ưu điểm: đơn giản,通用, dễ xem, thích hợp migration. Nhược điểm: dữ liệu lớn backup chậm, restore càng chậm vì phải thực thi lại từng câu SQL, ghi dữ liệu và build Index.

Lệnh Full Backup InnoDB khuyên dùng cho production:

```bash
mysqldump \
  --host=127.0.0.1 \
  --user=backup_user \
  --password \
  --all-databases \
  --single-transaction \
  --routines \
  --events \
  --triggers \
  --source-data=2 \
  > mysql-full-backup.sql
```

Giải thích tham số:

- `--single-transaction`: Mở một Transaction đọc nhất quán trước khi backup, phù hợp bảng InnoDB.
- `--routines` và `--events`: Export cả Stored Procedure, Function và Event.
- `--triggers`: Export các Trigger.
- `--source-data=2`: Ghi tên file binlog hiện tại và vị trí (position) vào dump file dưới dạng ghi chú SQL.

Nếu chỉ backup 1 database:

```bash
mysqldump \
  --host=127.0.0.1 \
  --user=backup_user \
  --password \
  --single-transaction \
  --routines \
  --events \
  --triggers \
  --source-data=2 \
  --databases order_db \
  > order_db.sql
```

Lệnh Restore:

```bash
mysql --host=127.0.0.1 --user=root --password < mysql-full-backup.sql
```

## MySQL Shell Dump Utilities: Logical Backup song song

Nếu muốn giữ tính khả thi của Logical Backup nhưng thấy `mysqldump` đơn thread quá chậm, có thể dùng MySQL Shell Dump Utilities:

```javascript
util.dumpInstance("/backup/mysql/instance", { threads: 8 });
util.dumpSchemas(["order_db"], "/backup/mysql/order_db", { threads: 8 });
util.loadDump("/backup/mysql/order_db", { threads: 8 });
```

Hỗ trợ dump/load song song nhiều thread, nén dữ liệu và xem tiến độ.

## Dùng binlog làm PITR (Point-in-Time Recovery - Khôi phục theo thời điểm)

Full Backup chỉ khôi phục về mốc thời gian backup. Muốn khôi phục tới thời điểm sau đó phải dựa vào binlog.

Khôi phục theo mốc thời gian (datetime):

```bash
mysqlbinlog \
  --start-position=154 \
  --stop-datetime="2026-06-25 10:20:59" \
  binlog.000120 binlog.000121 \
  | mysql --binary-mode --host=127.0.0.1 --user=root --password
```

Khôi phục theo vị trí (position) chính xác:

```bash
mysqlbinlog \
  --start-position=154 \
  --stop-position=987654 \
  binlog.000120 \
  | mysql --binary-mode --host=127.0.0.1 --user=root --password
```

Giải mã binlog ROW format thành câu lệnh dễ đọc để kiểm tra:

```bash
mysqlbinlog --base64-output=DECODE-ROWS -vv binlog.000120 > binlog.000120.readable.sql
```

## Dùng Percona XtraBackup làm Physical Backup

Percona XtraBackup là công cụ Physical Backup mã nguồn mở phổ biến. Sao lưu trực tiếp các data file của InnoDB.

Quy trình Full Backup cơ bản:

```bash
# 1. Backup
xtrabackup --backup --target-dir=/data/backups/mysql/base

# 2. Prepare (đưa dữ liệu về trạng thái nhất quán)
xtrabackup --prepare --target-dir=/data/backups/mysql/base

# 3. Restore (stop MySQL, clear datadir, copy back)
systemctl stop mysqld
mv /var/lib/mysql /var/lib/mysql.bak.$(date +%F-%H%M%S)
install -d -o mysql -g mysql /var/lib/mysql
xtrabackup --copy-back --target-dir=/data/backups/mysql/base
chown -R mysql:mysql /var/lib/mysql
systemctl start mysqld
```

Xem thông tin binlog để làm PITR tiếp theo:

```bash
cat /data/backups/mysql/base/xtrabackup_binlog_info
```

## Logical Backup và Physical Backup chọn thế nào?

| Kịch bản | Phương pháp phù hợp |
| ---------------------------------- | ------------------------------------- |
| DB nhỏ, DB test, export 1 bảng, migration xuyên môi trường | `mysqldump` |
| DB dung lượng trung bình, muốn export/import song song | MySQL Shell Dump Utilities |
| DB lớn, window restore ngắn, chủ yếu là bảng InnoDB | XtraBackup hoặc MySQL Enterprise Backup |
| Cần khôi phục theo thời điểm (PITR) | Full Backup + binlog |

## Quy trình diễn tập khôi phục (Restore Drills)

1. Chuẩn bị 1 máy cô lập, cài MySQL cùng phiên bản.
2. Lấy bản Full Backup mới nhất và binlog tương ứng.
3. Restore Full Backup, ghi lại thời gian tốn.
4. Replay binlog đến mốc chỉ định, ghi lại thời gian tốn.
5. Kiểm tra xác minh số lượng bảng, dữ liệu then chốt, Stored Procedure, Trigger, Account, Permission.
6. Chạy test ứng dụng kết nối DB khôi phục để xác nhận dữ liệu chính xác.

## Các hiểu lầm phổ biến

- **Hiểu lầm 1: Có Slave là không cần Backup.** (Lệnh xóa nhầm trên Master sẽ đồng bộ ngay sang Slave).
- **Hiểu lầm 2: Chỉ backup dữ liệu, không backup binlog.** (Chỉ khôi phục được tới lúc Full Backup, mất hết ghi chép từ lúc đó tới thời điểm sự cố).
- **Hiểu lầm 3: Lưu file backup trên cùng 1 máy với DB.** (Hỏng đĩa là mất cả chì lẫn chài).
- **Hiểu lầm 4: Backup script không cảnh báo khi thất bại.**
- **Hiểu lầm 5: Không bao giờ diễn tập khôi phục.**

<!-- @include: @article-footer.snippet.md -->
