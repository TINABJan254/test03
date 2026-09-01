---
title: MySQL数据同步到Elasticsearch详解：常见方案与一致性处理
description: MySQL数据同步到Elasticsearch方案详解，对比应用层双写、定时同步、Canal、Debezium和Flink CDC，并介绍全量同步、增量同步、消息乱序、幂等、删除、重建和数据对账。
category: 数据库
tag:
  - MySQL
  - Elasticsearch
  - 数据同步
head:
  - - meta
    - name: keywords
      content: MySQL同步Elasticsearch,MySQL同步ES,MySQL ES同步,Canal同步ES,Flink CDC同步ES,Debezium,CDC,binlog,全量同步,增量同步,数据一致性
---

MySQL giỏi xử lý Transaction (Giao dịch), Elasticsearch (ES) giỏi tìm kiếm toàn văn (Full-text Search) và truy vấn phức tạp. Nhiều hệ thống sẽ coi MySQL làm nguồn dữ liệu chuẩn (Authority Data Source), sau đó chuẩn hóa các field cần thiết thành Document để ghi vào Elasticsearch.

Sau khi áp dụng mô hình này, rắc rối thường tập trung vào các khâu:

- Lần đầu tiên đưa lên hệ thống, làm sao import lượng dữ liệu lịch sử từ MySQL sang ES?
- Sau đó MySQL phát sinh thao tác `INSERT`, `UPDATE`, `DELETE`, làm sao liên tục cập nhật sang ES?
- Khi tác vụ đồng bộ bị tiêu thụ lặp lại, đảo lộn thứ tự hoặc ngắt kết nối, làm sao tránh dữ liệu cũ đè dữ liệu mới?
- Khi cấu trúc Index thay đổi hoặc dữ liệu không nhất quán, làm sao Rebuild và chuyển đổi mượt mà (smooth switch)?

Một phương án đồng bộ hoàn chỉnh thường phải cân nhắc đồng thời **Đồng bộ toàn lượng (Full Sync), Đồng bộ tăng lượng (Incremental Sync), Khôi phục khi thất bại và Đối soát dữ liệu**. Chỉ giải quyết một khâu rất dễ bị tắc nghẽn khi lên hệ thống hoặc khi xử lý sự cố.

## Xác định rõ mục tiêu đồng bộ

Đồng bộ dữ liệu từ MySQL sang Elasticsearch thường chia thành 2 loại:

- **Đồng bộ toàn lượng (Full Sync)**: Đọc dữ liệu hiện có trong MySQL để xây dựng một bản ES Index hoàn chỉnh. Thường dùng khi lần đầu đưa lên hệ thống, Rebuild Index hoặc sửa chữa dữ liệu.
- **Đồng bộ tăng lượng (Incremental Sync)**: Liên tục bắt các thao tác `INSERT`, `UPDATE`, `DELETE` sau khi MySQL commit để cập nhật Document tương ứng trong ES.

Hai loại này thường được kết hợp sử dụng. Ví dụ: trước tiên import toàn lượng 10 triệu bản ghi lịch sử, sau đó từ vị trí binlog position ghi nhận lúc bắt đầu tác vụ toàn lượng để tiếp tục tiêu thụ tăng lượng cho tới khi đuổi kịp dữ liệu mới nhất.

Ngoài ra cần lưu ý, ghi thành công vào ES không có nghĩa dữ liệu lập tức có thể search ra ngay. Elasticsearch thông qua cơ chế `refresh` làm cho Document mới ghi có thể tìm thấy được, do đó độ trễ end-to-end bao gồm các khâu: bắt thay đổi, xếp hàng message, xử lý dữ liệu, ghi vào ES và `refresh`.

![Chuyển tiếp giữa Full Sync và Incremental Sync từ MySQL sang ES](https://oss.javaguide.cn/github/javaguide/database/es/mysql-es-full-incremental-sync.webp)

## Định nghĩa Hợp đồng đồng bộ (Sync Contract) trước khi chọn công cụ

Cùng một công cụ đồng bộ, đưa vào các mô hình dữ liệu và yêu cầu nhất quán khác nhau có thể cho ra kết quả hoàn toàn khác nhau. Trước khi lựa chọn công cụ, hãy làm rõ các thỏa thuận sau:

- **Nguồn dữ liệu chuẩn (Authority Source)**: Thường lấy MySQL làm chuẩn, ES chỉ lưu trữ View tìm kiếm có thể Rebuild lại. Nghiệp vụ không được bỏ qua MySQL để sửa trực tiếp các field chuẩn trong ES.
- **Quy tắc tạo Document**: Làm rõ những bảng và field nào hợp thành 1 ES Document, `_id` của Document lấy từ đâu.
- **Ngữ nghĩa xóa (Delete Semantics)**: Xóa vật lý, xóa logic hay mất hiệu lực nghiệp vụ tương ứng với ES `delete`, giữ cờ xóa, hay sinh lại Document.
- **Thứ tự và Phiên bản (Order & Version)**: Xác định Partition Key và nguồn Version của cùng 1 Document để tránh việc retry hay concurrency làm giá trị cũ ghi đè giá trị mới.
- **Mục tiêu độ trễ và khôi phục**: Thỏa thuận độ trễ cho phép trong điều kiện bình thường, thời gian đuổi kịp dữ liệu sau khi ngắt kết nối.
- **Cấu hình tài nguyên DB nguồn**: Đánh giá số kết nối và I/O đĩa khi quét toàn lượng, quyết định đọc Master hay Read Replica, thiết lập giới hạn tốc độ (Rate Limit) cho tác vụ đồng bộ.

## Bảng so sánh các phương án phổ biến

| Phương án | Đồng bộ toàn lượng | Đồng bộ tăng lượng | Ưu điểm chính | Nhược điểm chính | Kịch bản áp dụng |
| ---------------- | ---------- | -------------- | ------------------------------------ | ------------------------------------ | -------------------------------------- |
| Application-level Double Write | Không hỗ trợ | Hỗ trợ | Dễ triển khai,链路 ngắn | Không thể commit nguyên tố, xâm nhập code nghiệp vụ, dễ đảo thứ tự | Dữ liệu nhỏ, chấp nhận không nhất quán ngắn |
| Local Message Table + MQ | Không hỗ trợ | Hỗ trợ | Đưa việc cập nhật nghiệp vụ và event vào cùng Transaction | Vẫn cần viết thêm phần import toàn lượng, sender/consumer | Hệ thống đã dùng kiến trúc Event-driven |
| Scheduled Sync (Định kỳ) | Hỗ trợ | Hỗ trợ theo điều kiện | Đơn giản, chi phí vận hành thấp | Độ trễ cao, thao tác DELETE cần xử lý riêng | Dữ liệu nhỏ, tần suất cập nhật thấp |
| Canal + Consumer | Cần xử lý riêng | Hỗ trợ | Hệ sinh thái phát triển tốt, dễ đẩy sang MQ | Cần tự bảo trì position, consumer và nối toàn lượng | Đã có hạ tầng Java/MQ, đồng bộ tăng lượng |
| Debezium + Kafka | Hỗ trợ Snapshot | Hỗ trợ | Hệ sinh thái Kafka Connect hoàn thiện, format nhất quán | Nhiều component, phụ thuộc Kafka Connect | Đã có Kafka Connect hoặc platform CDC |
| Flink CDC | Hỗ trợ | Hỗ trợ | Tự động nối Full và Incremental, phù hợp chuyển đổi phức tạp | Chi phí triển khai và quản lý State cao | Dữ liệu lớn, cần Stream Processing / JOIN nhiều bảng |

## Đồng bộ kép ở tầng ứng dụng (Application Double Write)

Đồng bộ kép ở tầng ứng dụng là việc sau khi code nghiệp vụ update MySQL thành công sẽ gọi tiếp Elasticsearch API để cập nhật Index.

![Viết kép đồng bộ tầng ứng dụng](https://oss.javaguide.cn/github/javaguide/database/es/es-mq-synchronization-synchronous-double-write.png)

Mô hình này không thể rollback thao tác trên ES khi MySQL commit, cũng không thể tạo ra Local Transaction giữa MySQL và ES. Do đó chỉ đáp ứng tính nhất quán cuối cùng (Eventual Consistency).

### Bù đắp lỗ hổng Transaction bằng Bảng tin nhắn cục bộ (Local Message Table / Outbox Pattern)

Khi yêu cầu độ tin cậy đồng bộ cao, có thể dùng **Transactional Outbox (Bảng tin nhắn cục bộ)**:

1. Trong cùng 1 MySQL Transaction, update bảng nghiệp vụ đồng thời insert 1 bản ghi vào bảng `outbox`.
2. Một tiến trình độc lập hoặc CDC component đọc bảng `outbox` để gửi event sang MQ.
3. ES Consumer xử lý message, thành công thì ghi nhận kết quả, thất bại thì retry hoặc đưa vào Dead Letter Queue (DLQ).

## Đồng bộ định kỳ (Scheduled Sync)

Nếu lượng dữ liệu không lớn, tần suất cập nhật thấp và nghiệp vụ chấp nhận độ trễ theo phút/ngày thì Scheduled Task là đủ dùng.

![Đồng bộ định kỳ](https://oss.javaguide.cn/github/javaguide/database/es/es-mq-synchronization-scheduled-task.png)

Query quét tăng lượng theo mốc thời gian:

```sql
SELECT id, title, content, updated_at
FROM article
WHERE (
        updated_at > :last_updated_at
        OR (updated_at = :last_updated_at AND id > :last_id)
      )
  AND updated_at <= :task_cutoff
ORDER BY updated_at, id
LIMIT :batch_size;
```

### Sử dụng Index Alias khi Rebuild toàn lượng

Không nên vừa xóa vừa ghi trực tiếp trên Index cũ mà người dùng đang search. Cách làm an toàn:

1. Tạo Index mới mang số phiên bản, ví dụ `articles_v2`, cấu hình trước `settings` và `mappings`.
2. Ghi toàn bộ dữ liệu MySQL vào Index mới, kiểm tra xác minh số lượng và nội dung.
3. Bù đắp các thay đổi tăng lượng phát sinh trong quá trình chạy tác vụ toàn lượng.
4. Sử dụng `_aliases` API để chuyển đổi nguyên tố (atomic switch) Business Alias từ Index cũ sang Index mới.

```http
POST /_aliases
{
  "actions": [
    { "remove": { "index": "articles_v1", "alias": "articles", "must_exist": true } },
    { "add": { "index": "articles_v2", "alias": "articles" } }
  ]
}
```

## CDC dựa trên binlog

CDC (Change Data Capture) sẽ đọc MySQL binlog và chuyển đổi các thay đổi dữ liệu đã commit thành các event cho phía hạ nguồn tiêu thụ.

Các cấu hình cần kiểm tra trước:
- MySQL đã bật binary log (`binlog_format=ROW`).
- Tài khoản CDC được cấp đủ quyền đọc replication và đọc bảng.
- Thời gian lưu giữ binlog đủ dài để bao phủ window khôi phục sự cố.

### Canal

[Canal](https://github.com/alibaba/canal) đóng vai trò giả lập giao thức MySQL Replica, gửi yêu cầu đọc binlog tới MySQL và parse thành các event thay đổi cấu trúc.

![Nguyên lý hoạt động của Canal](https://oss.javaguide.cn/github/javaguide/open-source-project/canal-overview.png)

Canal Server có nhiệm vụ đăng ký và parse log tăng lượng. Phía dưới có thể dùng Canal Client tiêu thụ trực tiếp hoặc đẩy sang Kafka / RocketMQ để dịch vụ đồng bộ ghi vào ES.

![Canal đồng bộ dữ liệu qua MQ](https://oss.javaguide.cn/github/javaguide/database/es/es-mq-synchronization-canal-with-mq.png)

### Flink CDC

MySQL Source của Flink CDC có thể đọc Table Snapshot trước, sau đó tiếp tục tiêu thụ binlog. Nó chia bảng thành các Chunk theo Partition Key để đọc song song snapshot, tự động kết nối giữa Full Snapshot và Incremental binlog reading mà không cần giữ khóa đọc toàn cục.

![Flink CDC tự động nối Snapshot và binlog](https://oss.javaguide.cn/github/javaguide/database/es/flink-cdc-incremental-snapshot.webp)

### Debezium

Debezium MySQL Connector thường chạy trên nền Kafka Connect. Khi khởi động lần đầu nó có thể thực thi Consistency Snapshot rồi tiếp tục gửi event dòng sang Kafka Topic từ position tương ứng.

## Các vấn đề bắt buộc phải xử lý trong môi trường Production

![Các chốt chặn đảm bảo tính nhất quán giữa MySQL và ES](https://oss.javaguide.cn/github/javaguide/database/es/mysql-es-consistency-guardrails.webp)

### 1. Dùng Primary Key của MySQL làm ES Document ID (`_id`)

Khi đồng bộ nên lấy Primary Key ổn định và duy nhất làm `_id` cho ES Document. Khi một bản ghi bị lặp lại thao tác `index` hoặc `upsert` sẽ ghi đè lên cùng 1 Document, giúp Consumer đạt tính Idempotent (Đồng sức / 幂等).

### 2. Đảm bảo thứ tự event cho cùng một Document

Dùng Partition Key theo Primary Key trên MQ để các event của cùng 1 ID đi vào cùng 1 Partition và được xử lý tuần tự. Sử dụng `version` và `version_type=external` trong Bulk API của ES để tránh dữ liệu cũ đè dữ liệu mới.

### 3. Xử lý chính xác thao tác xóa (DELETE)

Các câu `INSERT` và `UPDATE` có thể quy về `upsert`, còn `DELETE` bắt buộc phải chuyển đổi thành thao tác `delete` trong ES. Với xóa logic (Logical Delete), cần thỏa thuận rõ ràng là xóa hẳn Document hay cập nhật cờ `deleted=true`.

### 4. Xử lý Document kết hợp nhiều bảng (Multi-table Document)

Một Document hàng hóa có thể kết hợp dữ liệu từ bảng hàng hóa, bảng thương hiệu, bảng tồn kho... Khi thương hiệu đổi tên, cần tìm ra tất cả hàng hóa chịu ảnh hưởng để Rebuild lại Document của chúng.

### 5. Kiểm tra kết quả từng bản ghi khi ghi Bulk

Sử dụng ES Bulk API để giảm overhead mạng, nhưng cần duyệt từng item trong phản hồi để xử lý riêng các lỗi 429, timeout, hay chuyển lỗi mapping sang DLQ.

### 6. Quản lý Mapping và DDL Changes

Nên sử dụng Explicit Mapping (Mapping rõ ràng) và Index Template. Tránh để Dynamic Mapping tự động nhận diện sai kiểu dữ liệu hoặc gây nổ số lượng field (mapping explosion).

### 7. Giám sát độ trễ (Lag), thất bại và thời gian lưu giữ binlog

Giám sát khoảng cách giữa position tiêu thụ hiện tại và position mới nhất của binlog, số lượng message tích áp trên MQ, tỷ lệ ghi Bulk thành công vào ES.

### 8. Định kỳ đối soát (Data Reconciliation) và giữ khả năng Rebuild

Định kỳ so sánh tổng số bản ghi và checksum giữa MySQL và ES để phát hiện sai lệch ngầm. Giữ khả năng "Rebuild Index mới 1-click" mọi lúc.

### 9. Chuẩn bị Runbook xử lý sự cố

| Kịch bản sự cố | Cách xử lý | Việc KHÔNG được làm trực tiếp |
| --------------------------- | ----------------------------------------------------------------------- | -------------------------------------------- |
| ES tạm thời không khả dụng / trả về 429/5xx | Tạm dừng hoặc giới hạn tốc độ ghi, giữ message chưa xác nhận, retry theo backoff | Commit position khi chưa ghi thành công |
| Xung đột mapping / sai format field | Dừng retry vô ích, ghi log sự kiện gốc và nguyên nhân, sửa template rồi replay có định hướng | Retry vô hạn câu lệnh lỗi cố định |
| CDC task restart | Khôi phục từ Checkpoint / position binlog đã lưu, dựa vào idempotent ghi đè | Xóa state rồi start trực tiếp từ position mới nhất |
| Binlog cần thiết đã bị xóa quá hạn | Thực thi lại Consistency Snapshot hoặc Rebuild toàn lượng | Nhảy tới position mới nhất rồi tiếp tục chạy |

## Lựa chọn phương án nào?

- Dữ liệu nhỏ, ít cập nhật, chấp nhận độ trễ dài: Ưu tiên Scheduled Sync + Rebuild định kỳ.
- Đã có hệ thống Domain Event và MQ tin cậy: Dùng Local Message Table + MQ.
- Muốn bắt tăng lượng từ MySQL ổn định, team thạo Java & MQ: Canal + MQ + Consumer.
- Đã có platform Kafka Connect: Dùng Debezium.
- Dữ liệu lớn, cần tự động nối toàn lượng và tăng lượng, hoặc cần Stream Processing: Dùng Flink CDC.

## Tham khảo

- [MySQL 8.4：Setting Binary Log Format](https://dev.mysql.com/doc/refman/8.4/en/binary-log-setting.html)
- [Canal Documentation](https://github.com/alibaba/canal)
- [Flink CDC 3.6 Documentation](https://nightlies.apache.org/flink/flink-cdc-docs-release-3.6/)
- [Debezium MySQL Connector](https://debezium.io/documentation/reference/stable/connectors/mysql.html)
- [Elasticsearch Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

<!-- @include: @article-footer.snippet.md -->
