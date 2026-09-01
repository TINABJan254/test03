---
title: MongoDB常见面试题总结（上）
description: MongoDB常见面试题总结上篇，详解MongoDB基础概念、存储结构、数据类型、副本集高可用、分片集群水平扩展等核心知识点，助力后端面试准备。
category: 数据库
tag:
  - NoSQL
  - MongoDB
head:
  - - meta
    - name: keywords
      content: MongoDB面试题,文档数据库,BSON,副本集,分片集群,MongoDB索引,WiredTiger,聚合管道
---

> Một phần nhỏ nội dung có tham khảo mô tả từ tài liệu chính thức của MongoDB, xin được lưu ý tại đây.

## Cơ sở MongoDB

### MongoDB là gì?

MongoDB là một hệ thống Cơ sở dữ liệu NoSQL mã nguồn mở dựa trên **Lưu trữ file phân tán (Distributed File Storage)**, được viết bằng **C++**. MongoDB cung cấp phương thức lưu trữ **Hướng tài liệu (Document-oriented)**, thao tác khá đơn giản và dễ dàng, hỗ trợ mô hình hóa dữ liệu "**No-schema (Không có lược đồ cố định)**", có thể lưu trữ các kiểu dữ liệu tương đối phức tạp, là một **Document Database** vô cùng phổ biến.

Trong trường hợp tải cao (High Load), MongoDB hỗ trợ tự nhiên cho Mở rộng theo chiều ngang (Horizontal Scaling) và Tính sẵn sàng cao (High Availability), có thể thêm nhiều node/instance một cách rất thuận tiện để đảm bảo hiệu năng và tính sẵn sàng của dịch vụ. Trong nhiều kịch bản, MongoDB có thể dùng để thay thế RDBMS truyền thống hoặc lưu trữ Key/Value, nhằm cung cấp giải pháp lưu trữ dữ liệu hiệu năng cao, tính sẵn sàng cao, có khả năng mở rộng cho các ứng dụng Web.

### Cấu trúc lưu trữ của MongoDB là gì?

Cấu trúc lưu trữ của MongoDB khác với RDBMS truyền thống, chủ yếu bao gồm 3 đơn vị sau:

- **Document (Tài liệu)**: Đơn vị cơ bản nhất trong MongoDB, cấu thành từ các cặp Key-Value BSON, tương tự như dòng (Row) trong RDBMS.
- **Collection (Tập hợp)**: Một Collection có thể chứa nhiều Document, tương tự như bảng (Table) trong RDBMS.
- **Database (Cơ sở dữ liệu)**: Một Database có thể chứa nhiều Collection, có thể tạo nhiều Database trong MongoDB, tương tự như Database trong RDBMS.

Nói cách khác, MongoDB lưu trữ các bản ghi dữ liệu dưới dạng Document (cụ thể hơn là [BSON Document](https://www.mongodb.com/docs/manual/core/document/#std-label-bson-document-format)), các Document này được tập hợp lại trong các Collection, và Database lưu trữ một hoặc nhiều Collection Document.

**So sánh thuật ngữ thường gặp giữa SQL và MongoDB**:

| SQL | MongoDB |
| ------------------------ | ------------------------------- |
| Table | Collection |
| Row | Document |
| Column | Field |
| Primary Key | ObjectId |
| Index | Index |
| Embedded Table | Embedded Document |
| Array | Array |

#### Document

Bản ghi trong MongoDB chính là một BSON Document, nó là một cấu trúc dữ liệu gồm các cặp Key-Value, tương tự như JSON Object, là đơn vị dữ liệu cơ bản trong MongoDB. Giá trị của field có thể bao gồm các Document khác, Array và Array của Document.

![MongoDB 文档](https://oss.javaguide.cn/github/javaguide/database/mongodb/crud-annotated-document..png)

Key của Document là chuỗi ký tự. Ngoại trừ một số ít trường hợp ngoại lệ, Key có thể sử dụng bất kỳ ký tự UTF-8 nào.

- Key không được chứa `\0` (Ký tự NULL). Ký tự này dùng để đánh dấu điểm kết thúc của Key.
- `.` và `$` có ý nghĩa đặc biệt, chỉ có thể sử dụng trong môi trường cụ thể.
- Key bắt đầu bằng dấu gạch dưới `_` được dành riêng (không bắt buộc nghiêm ngặt).

**BSON [bee-sahn]** là viết tắt của Binary [JSON](http://json.org/), là dạng biểu diễn nhị phân của JSON Document, hỗ trợ nhúng Document và Array vào các Document và Array khác, đồng thời còn chứa các phần mở rộng cho phép biểu diễn các kiểu dữ liệu không thuộc quy chuẩn JSON. Về nội dung quy chuẩn BSON, có thể tham khảo [bsonspec.org](http://bsonspec.org/), xem thêm [Các kiểu BSON](https://www.mongodb.com/docs/manual/reference/bson-types/).

Theo giới thiệu về BSON trên Wikipedia, tốc độ Duyệt (Traversal) của BSON tốt hơn JSON, đây cũng là lý do chính MongoDB lựa chọn BSON, tuy nhiên BSON cần nhiều dung lượng lưu trữ hơn.

> So với JSON, BSON tập trung vào việc nâng cao hiệu suất lưu trữ và quét (scan). Các phần tử lớn trong BSON Document có tiền tố là field độ dài để thuận tiện cho việc quét. Trong một số trường hợp, do sự tồn tại của tiền tố độ dài và Index mảng tường minh, dung lượng BSON sử dụng sẽ nhiều hơn JSON.

![BSON 官网首页](https://oss.javaguide.cn/github/javaguide/database/mongodb/bsonspec.org.png)

#### Collection

MongoDB Collection tồn tại trong Database, **không có cấu trúc cố định**, tức là **No-schema**, điều này có nghĩa là có thể INSERT dữ liệu với định dạng và kiểu khác nhau vào Collection. Tuy nhiên, thông thường, dữ liệu được INSERT vào Collection đều có tính liên quan nhất định.

![MongoDB 集合](https://oss.javaguide.cn/github/javaguide/database/mongodb/crud-annotated-collection.png)

Collection không cần phải tạo trước, khi Document đầu tiên được INSERT hoặc Index đầu tiên được tạo, nếu Collection đó chưa tồn tại thì sẽ tự động tạo một Collection mới.

Tên Collection có thể là bất kỳ chuỗi UTF-8 nào thỏa mãn các điều kiện sau:

- Tên Collection không được là chuỗi rỗng `""`.
- Tên Collection không được chứa `\0` (Ký tự NULL), ký tự này biểu thị điểm kết thúc của tên Collection.
- Tên Collection không được bắt đầu bằng "system.", đây là tiền tố dành riêng cho System Collection. Ví dụ Collection `system.users` lưu giữ thông tin người dùng của Database, Collection `system.namespaces` lưu giữ thông tin của tất cả Collection trong Database.
- Tên Collection bắt buộc phải bắt đầu bằng dấu gạch dưới hoặc chữ cái, và không được chứa `$`.

#### Database

Database dùng để lưu trữ tất cả Collection, và Collection lại dùng để lưu trữ tất cả Document. Trong một MongoDB có thể tạo nhiều Database, mỗi Database đều có Collection và phân quyền riêng.

MongoDB đã dành riêng một số Database đặc biệt:

- **admin** : Database admin chủ yếu lưu giữ root user và role. Ví dụ bảng `system.users` lưu user, bảng `system.roles` lưu role. Thông thường không khuyến nghị user thao tác trực tiếp trên Database này. Thêm một user vào Database này và cấp cho nó quyền role `dbAdminAnyDatabase` trên DB admin, user này sẽ tự động kế thừa quyền của tất cả Database. Một số lệnh Server-side đặc thù cũng chỉ có thể chạy từ Database này, ví dụ Shutdown Server.
- **local** : Database local sẽ không được Replicate (sao chép) sang các Shard khác, do đó có thể dùng để lưu trữ Collection bất kỳ của riêng một Server cục bộ. Thông thường không khuyến nghị người dùng trực tiếp dùng DB local để lưu trữ bất kỳ dữ liệu nào, cũng không khuyến nghị thực hiện thao tác CRUD, vì dữ liệu không thể Backup và Restore bình thường.
- **config** : Khi MongoDB sử dụng thiết lập Sharding, Database config có thể dùng để lưu giữ thông tin liên quan đến Shard.
- **test** : DB test được tạo mặc định, khi kết nối tới dịch vụ [mongod](https://mongoing.com/docs/reference/program/mongod.html), nếu không chỉ định Database cụ thể cần kết nối, mặc định sẽ kết nối tới DB test.

Tên Database có thể là bất kỳ chuỗi UTF-8 nào thỏa mãn các điều kiện sau:

- Không được là chuỗi rỗng `""`.
- Không được chứa `' '` (Khoảng trắng), `.`, `$`, `/`, `\` và `\0` (Ký tự NULL).
- Nên viết thường toàn bộ.
- Tối đa 64 bytes.

Tên Database cuối cùng sẽ biến thành file trong File System, đây chính là nguyên nhân vì sao lại có nhiều hạn chế như vậy.

### MongoDB có những đặc điểm gì?

- **Bản ghi dữ liệu được lưu trữ dưới dạng Document**: Bản ghi trong MongoDB là một BSON Document, cấu trúc dữ liệu gồm các cặp Key-Value, tương tự JSON Object, là đơn vị dữ liệu cơ bản.
- **Schema Free (Tự do lược đồ)**: Khái niệm Collection tương tự Table trong MySQL, nhưng nó không cần định nghĩa bất kỳ Schema nào, có thể thể hiện các đối tượng Domain Model phức tạp bằng ít đối tượng dữ liệu hơn.
- **Hỗ trợ nhiều phương thức truy vấn**: MongoDB Query API hỗ trợ các thao tác đọc ghi (CRUD) cũng như Data Aggregation, Text Search và Geospatial Query.
- **Hỗ trợ ACID Transaction**: Cơ sở dữ liệu NoSQL thông thường không hỗ trợ Transaction để đánh đổi lấy khả năng mở rộng và hiệu năng cao. Tuy nhiên cũng có ngoại lệ, MongoDB hỗ trợ Transaction. Tương tự RDBMS, Transaction trong MongoDB cũng có đầy đủ tính chất ACID. MongoDB natively hỗ trợ tính nguyên tố (Atomicity) cho Single Document, cũng mang tính chất Transaction. MongoDB 4.0 đã bổ sung hỗ trợ cho Multi-document Transaction, nhưng chỉ hỗ trợ ở chế độ triển khai Replica Set, tức là phạm vi tác động của Transaction bị giới hạn trong 1 Replica Set. MongoDB 4.2 giới thiệu Distributed Transaction, bổ sung hỗ trợ Multi-document Transaction trên Sharded Cluster, đồng thời gộp với hỗ trợ hiện có trên Replica Set.
- **Lưu trữ nhị phân hiệu quả**: Document lưu trữ trong Collection tồn tại dưới dạng cặp Key-Value. Key dùng để định danh duy nhất cho 1 Document, thông thường là kiểu ObjectId, Value tồn tại dưới dạng BSON. BSON = Binary JSON, là định dạng bổ sung thêm một số kiểu dữ liệu và mô tả Metadata trên nền tảng JSON.
- **Tích hợp sẵn tính năng nén dữ liệu**: Lưu trữ cùng một lượng dữ liệu tiêu tốn ít tài nguyên hơn.
- **Hỗ trợ mapreduce**: Hoàn thành các nhiệm vụ Aggregation phức tạp thông qua chia để trị (Divide and Conquer). Tuy nhiên từ MongoDB 5.0, map-reduce đã không còn được khuyến nghị chính thức nữa, giải pháp thay thế là [Aggregation Pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/). Aggregation Pipeline cung cấp hiệu năng và tính khả dụng tốt hơn map-reduce.
- **Hỗ trợ nhiều loại Index**: MongoDB hỗ trợ nhiều loại Index, bao gồm Single Field Index, Compound Index, Multikey Index, Hashed Index, Text Index, Geospatial Index, v.v., mỗi loại Index có trường hợp sử dụng khác nhau.
- **Hỗ trợ Failover**: Cung cấp tính năng tự động phục hồi sự cố, khi Node Primary bị lỗi sẽ tự động bầu chọn (Elect) một Node Primary mới từ các Node Secondary, đảm bảo Cluster hoạt động bình thường, việc này là hoàn toàn trong suốt (Transparent) đối với Client.
- **Hỗ trợ Sharded Cluster**: MongoDB hỗ trợ Cluster tự động phân chia dữ liệu, giúp Cluster lưu trữ nhiều dữ liệu hơn, có hiệu năng mạnh mẽ hơn. Khi INSERT và UPDATE dữ liệu, hệ thống có thể tự động Route và lưu trữ.
- **Hỗ trợ lưu trữ file lớn**: Dung lượng yêu cầu cho Single Document trong MongoDB không quá 16MB. Đối với các file lớn vượt quá 16MB, MongoDB cung cấp GridFS để lưu trữ, thông qua GridFS có thể chia nhỏ dữ liệu lớn thành các block, sau đó lưu trữ các Document nhỏ bóc tách này trong DB.

### MongoDB phù hợp với những kịch bản ứng dụng nào?

**Ưu thế của MongoDB nằm ở tính linh hoạt của Document Model và Storage Engine, tính khả mở của kiến trúc và sự hỗ trợ Index mạnh mẽ.**

Lựa chọn MongoDB nên cân nhắc đầy đủ ưu thế của MongoDB, kết hợp với nhu cầu thực tế của dự án để quyết định:

- Theo sự phát triển của dự án, việc lưu dữ liệu dạng giống JSON (BSON) có thỏa mãn nhu cầu dự án không? Bản ghi trong MongoDB là BSON Document, dạng Key-Value tương tự JSON Object, là đơn vị dữ liệu cơ bản.
- Có cần lưu trữ dữ liệu dung lượng lớn không? Có cần mở rộng theo chiều ngang nhanh chóng không? MongoDB hỗ trợ Sharded Cluster, có thể thêm nhiều node (instance) một cách rất thuận tiện, giúp Cluster lưu dữ liệu nhiều hơn, có hiệu năng mạnh mẽ hơn.
- Có cần nhiều loại Index hơn để đáp ứng các kịch bản ứng dụng khác nhau không? MongoDB hỗ trợ nhiều loại Index, bao gồm Single Field Index, Compound Index, Multikey Index, Hashed Index, Text Index, Geospatial Index, v.v., mỗi loại Index có trường hợp sử dụng khác nhau.
- ……

## Storage Engine trong MongoDB

### MongoDB hỗ trợ những Storage Engine nào?

Storage Engine (Động cơ lưu trữ) là component cốt lõi của Database, chịu trách nhiệm quản lý phương thức lưu trữ dữ liệu trong bộ nhớ (RAM) và trên đĩa (Disk).

Giống như MySQL, MongoDB cũng áp dụng **Kiến trúc Storage Engine dạng Pluggable**, hỗ trợ các loại Storage Engine khác nhau, các Storage Engine khác nhau giải quyết vấn đề của các kịch bản khác nhau. Khi tạo Database hoặc Collection có thể chỉ định Storage Engine.

> Kiến trúc Storage Engine dạng Pluggable giúp giải đóng gói (Decouple) giữa Server Layer và Storage Engine Layer, có thể hỗ trợ nhiều loại Storage Engine, ví dụ MySQL vừa hỗ trợ InnoDB Storage Engine cấu trúc B-Tree, vừa hỗ trợ RocksDB Storage Engine cấu trúc LSM.

Khi Storage Engine mới ra đời, mặc định sử dụng MMAPv1 Storage Engine, từ phiên bản MongoDB 4.x trở đi không còn hỗ trợ MMAPv1 Storage Engine nữa.

Hiện tại chủ yếu có 2 loại Storage Engine dưới đây:

- **WiredTiger Storage Engine**: Từ MongoDB 3.2 trở đi, Storage Engine mặc định là [WiredTiger Storage Engine](https://www.mongodb.com/docs/manual/core/wiredtiger/). Rất phù hợp với đại đa số Workload, khuyến nghị dùng cho các đợt triển khai mới. WiredTiger cung cấp các tính năng như Document-level Concurrency Model, Checkpoint và Nén dữ liệu (Data Compression).
- **In-Memory Storage Engine**: [In-Memory Storage Engine](https://www.mongodb.com/docs/manual/core/inmemory/) có sẵn trong MongoDB Enterprise. Nó không lưu Document trên đĩa mà giữ chúng trong bộ nhớ (RAM) để đạt được độ trễ dữ liệu (Data Latency) có thể dự đoán tốt hơn.

Ngoài ra, MongoDB 3.0 cung cấp **Pluggable Storage Engine API**, cho phép bên thứ ba phát triển Storage Engine cho MongoDB, điểm này cũng tương đối giống MySQL.

### WiredTiger dựa trên LSM Tree hay B+ Tree?

Hiện tại đại đa số các Database Storage Engine phổ biến đều dựa trên B/B+ Tree hoặc LSM (Log Structured Merge) Tree để triển khai. Đối với NoSQL Database, đại đa số (như HBase, Cassandra, RocksDB) đều dựa trên LSM Tree, MongoDB thì hơi khác một chút.

Như đã nói ở trên, từ MongoDB 3.2 trở đi, Storage Engine mặc định là WiredTiger Storage Engine. Trên trang chủ chính thức của WiredTiger Engine, chúng ta thấy WiredTiger sử dụng B+ Tree làm cấu trúc lưu trữ:

```plain
WiredTiger maintains a table's data in memory using a data structure called a B-Tree ( B+ Tree to be specific), referring to the nodes of a B-Tree as pages. Internal pages carry only keys. The leaf pages store both keys and values.
```

Ngoài ra, WiredTiger còn hỗ trợ [LSM(Log Structured Merge)](https://source.wiredtiger.com/3.1.0/lsm.html) Tree làm cấu trúc lưu trữ, MongoDB khi sử dụng WiredTiger làm Storage Engine thì mặc định sử dụng B+ Tree.

Nếu muốn tìm hiểu lý do MongoDB sử dụng B+ Tree, có thể xem bài viết này: [【Bác bỏ bài viết rập khuôn】Đừng phân tích bừa nữa, MongoDB sử dụng B+ Tree chứ không phải B Tree như các bạn nghĩ](https://zhuanlan.zhihu.com/p/519658576).

Khi sử dụng B+ Tree, WiredTiger lấy **Page** làm đơn vị cơ bản để đọc ghi dữ liệu trên đĩa. Mỗi node trong B+ Tree là một Page, có tổng cộng 3 loại Page:

- **root page (Node gốc)**: Node gốc của B+ Tree.
- **internal page (Node bên trong)**: Node Index trung gian không trực tiếp lưu trữ dữ liệu.
- **leaf page (Node lá)**: Node lá trực tiếp lưu trữ dữ liệu thực tế, chứa một Page Header, Block Header và dữ liệu thực tế (Key/Value), trong đó Page Header định nghĩa kiểu trang, kích thước dữ liệu tải (payload) thực tế trong trang, số lượng bản ghi trong trang, v.v.; Block Header định nghĩa Checksum của trang này, vị trí định địa chỉ của block trên đĩa, v.v.

Cấu trúc tổng thể như hình dưới:

![WiredTiger B+树整体结构](https://oss.javaguide.cn/github/javaguide/database/mongodb/mongodb-b-plus-tree-integral-structure.png)

Nếu muốn nghiên cứu sâu về WiredTiger Storage Engine, khuyến nghị đọc [Chuyên đề WiredTiger Storage Engine](https://mongoing.com/archives/category/wiredtiger%e5%ad%98%e5%82%a8%e5%bc%95%e6%93%8e%e7%b3%bb%e5%88%97) của Cộng đồng MongoDB Trung Quốc.

## Aggregation trong MongoDB

### Aggregation trong MongoDB có tác dụng gì?

Trong dự án thực tế, chúng ta thường cần gom nhiều Document hoặc thậm chí nhiều Collection lại với nhau để tính toán phân tích (như tính tổng, lấy giá trị lớn nhất) và trả về kết quả sau khi tính toán, quá trình này được gọi là **Thao tác Aggregation (Phép tổng hợp)**.

Theo giới thiệu từ tài liệu chính thức, chúng ta có thể sử dụng thao tác Aggregation để:

- Kết hợp các giá trị đến từ nhiều Document lại với nhau.
- Thực hiện một loạt các phép toán trên dữ liệu trong Collection.
- Phân tích sự thay đổi của dữ liệu theo thời gian.

### MongoDB cung cấp những phương thức thực thi Aggregation nào?

MongoDB cung cấp 2 phương thức thực thi Aggregation:

- **Aggregation Pipeline (Đường ống tổng hợp)**: Phương thức ưu tiên hàng đầu để thực thi thao tác Aggregation.
- **Single Purpose Aggregation Methods (Phương thức tổng hợp mục đích đơn lẻ)**: Tức là các hàm Aggregation có tác dụng đơn lẻ như `count()`, `distinct()`, `estimatedDocumentCount()`.

Đại đa số bài viết còn đề cập đến phương thức Aggregation **map-reduce**. Tuy nhiên từ MongoDB 5.0 trở đi, map-reduce đã không còn được khuyến nghị chính thức nữa, giải pháp thay thế là [Aggregation Pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/). Aggregation Pipeline cung cấp hiệu năng và tính khả dụng tốt hơn map-reduce.

MongoDB Aggregation Pipeline bao gồm nhiều Stage (Giai đoạn), mỗi Stage sẽ biến đổi Document khi Document đi qua Pipeline. Mỗi Stage nhận Output của Stage trước đó, xử lý dữ liệu tiếp theo, và gửi nó dưới dạng Input dữ liệu tới Stage tiếp theo.

Quy trình làm việc của mỗi Pipeline là:

1. Nhận một loạt các Document dữ liệu ban đầu
2. Thực hiện một loạt các phép toán trên các Document này
3. Output Document kết quả cho Stage tiếp theo

![管道的工作流程](https://oss.javaguide.cn/github/javaguide/database/mongodb/mongodb-aggregation-stage.png)

**Các Stage Operator thường dùng**:

| Operator | Mô tả ngắn gọn |
| --------- | ---------------------------------------------------------------------------------------------------- |
| \$match | Match operator, dùng để lọc tập hợp Document |
| \$project | Project operator, dùng để tái cấu trúc các field của từng Document, có thể trích xuất field, đổi tên field, thậm chí thao tác trên field cũ rồi thêm field mới |
| \$sort | Sort operator, dùng để sắp xếp Document dựa trên 1 hoặc nhiều field |
| \$limit | Limit operator, dùng để giới hạn số lượng Document trả về |
| \$skip | Skip operator, dùng để bỏ qua số lượng Document chỉ định |
| \$count | Count operator, dùng để thống kê số lượng Document |
| \$group | Group operator, dùng để gom nhóm tập hợp Document |
| \$unwind | Unwind operator, dùng để tách từng giá trị trong Array thành các Document riêng biệt |
| \$lookup | Lookup operator, dùng để JOIN với một Collection khác trong cùng DB và lấy các Document chỉ định, tương tự như populate |

Chi tiết các Operator khác xem tại tài liệu chính thức: <https://docs.mongodb.com/manual/reference/operator/aggregation/>

Stage Operator được dùng bên trong method `db.collection.aggregate`, nằm ở tầng đầu tiên của tham số mảng.

```sql
db.collection.aggregate( [ { 阶段操作符：表述 }, { 阶段操作符：表述 }, ... ] )
```

Dưới đây là một ví dụ trong tài liệu chính thức của MongoDB:

```sql
db.orders.aggregate([
   # Stage 1: $match stage lọc Document theo field status, và chuyển các Document có status bằng "A" sang Stage tiếp theo.
    { $match: { status: "A" } },
   # Stage 2: $group stage gom nhóm Document theo field cust_id để tính tổng số tiền của mỗi giá trị duy nhất cust_id.
    { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

## Transaction trong MongoDB

> Transaction trong MongoDB nếu muốn hiểu sâu nguyên lý thì khá tốn thời gian, bản thân tôi cũng chưa hiểu quá sâu. Do đó ở đây tôi chỉ giới thiệu ngắn gọn về Transaction trong MongoDB, các bạn muốn tìm hiểu nguyên lý có thể tự tìm kiếm tham khảo tài liệu liên quan.
>
> Dưới đây đề xuất một số bài viết để mọi người tham khảo:
>
> - [Nguyên lý Transaction trong MongoDB](https://mongoing.com/archives/82187)
> - [Thiết kế và triển khai mô hình nhất quán MongoDB](https://developer.aliyun.com/article/782494)
> - [Giới thiệu Transaction trên tài liệu chính thức MongoDB](https://www.mongodb.com/docs/upcoming/core/transactions/)

Khi giới thiệu NoSQL Database chúng ta cũng đã nói, NoSQL Database thông thường không hỗ trợ Transaction để đánh đổi lấy khả năng mở rộng và hiệu năng cao. Tuy nhiên cũng có ngoại lệ, MongoDB hỗ trợ Transaction.

Tương tự RDBMS, Transaction trong MongoDB cũng có đầy đủ tính chất ACID:

- **Atomicity (Tính nguyên tố)** (`Atomicity`): Transaction là đơn vị thực thi nhỏ nhất, không cho phép chia nhỏ. Tính nguyên tố của Transaction đảm bảo các hành động hoặc là hoàn thành toàn bộ, hoặc là hoàn toàn không có tác dụng;
- **Consistency (Tính nhất quán)** (`Consistency`): Trước và sau khi thực thi Transaction, dữ liệu giữ nguyên tính nhất quán, ví dụ trong nghiệp vụ chuyển tiền, bất kể Transaction có thành công hay không, tổng số tiền của người chuyển và người nhận phải không thay đổi;
- **Isolation (Tính cô lập)** (`Isolation`): Khi truy cập đồng thời Database, Transaction của một user không bị các Transaction khác can thiệp, giữa các Transaction đồng thời Database là độc lập. WiredTiger Storage Engine hỗ trợ các mức cô lập Read Uncommitted, Read Committed và Snapshot Isolation, MongoDB khi khởi động mặc định chọn Snapshot Isolation. Ở các Isolation Level khác nhau, trong vòng đời của 1 Transaction có thể xuất hiện các hiện tượng Dirty Read, Non-repeatable Read, Phantom Read, v.v.
- **Durability (Tính bền vững)** (`Durability`): Sau khi 1 Transaction được Commit, sự thay đổi của nó đối với dữ liệu trong Database là bền vững, ngay cả khi Database gặp sự cố cũng không nên gây bất kỳ ảnh hưởng nào tới nó.

Về giới thiệu chi tiết của Transaction bài viết này sẽ không nói nhiều nữa, ai quan tâm có thể xem bài viết [Tổng hợp câu hỏi phỏng vấn MySQL phổ biến](../mysql/mysql-questions-01.md) do tôi viết, trong đó có giới thiệu chi tiết.

MongoDB natively hỗ trợ Atomicity cho Single Document, cũng mang tính chất Transaction. Khi thảo luận về Transaction trong MongoDB, thông thường đang chỉ tới **Multi-document**. MongoDB 4.0 đã bổ sung hỗ trợ cho Multi-document ACID Transaction, nhưng chỉ hỗ trợ ở chế độ triển khai Replica Set, tức là phạm vi tác động của Transaction bị giới hạn trong 1 Replica Set. MongoDB 4.2 giới thiệu **Distributed Transaction**, bổ sung hỗ trợ Multi-document Transaction trên Sharded Cluster, đồng thời gộp với hỗ trợ hiện có trên Replica Set.

Theo giới thiệu từ tài liệu chính thức:

> Từ MongoDB 4.2 trở đi, Distributed Transaction và Multi-document Transaction mang cùng một ý nghĩa trong MongoDB. Distributed Transaction là chỉ Multi-document Transaction trên Sharded Cluster và Replica Set. Từ MongoDB 4.2 trở đi, Multi-document Transaction (dù trên Sharded Cluster hay Replica Set) cũng đều được gọi là Distributed Transaction.

Trong hầu hết các trường hợp, Multi-document Transaction sẽ tốn chi phí hiệu năng lớn hơn so với ghi Single Document. Đối với đại đa số kịch bản, [Denormalized Data Model (Embedded Document và Array)](https://www.mongodb.com/docs/upcoming/core/data-model-design/#std-label-data-modeling-embedding) vẫn là sự lựa chọn tốt nhất. Nói cách khác, mô hình hóa dữ liệu một cách thích hợp có thể giảm thiểu tối đa nhu cầu đối với Multi-document Transaction.

**Lưu ý**:

- Từ MongoDB 4.2 trở đi, Multi-document Transaction hỗ trợ Replica Set và Sharded Cluster, trong đó: Node Primary sử dụng WiredTiger Storage Engine, đồng thời Node Secondary sử dụng WiredTiger Storage Engine hoặc In-Memory Storage Engine. Trong MongoDB 4.0, chỉ có Replica Set sử dụng WiredTiger Storage Engine mới hỗ trợ Transaction.
- Trong MongoDB 4.2 và các phiên bản trước đây, bạn không thể tạo Collection bên trong Transaction. Từ MongoDB 4.4 trở đi, bạn có thể tạo Collection và Index bên trong Transaction. Để biết chi tiết xin tham khảo [Tạo Collection và Index trong Transaction](https://www.mongodb.com/docs/upcoming/core/transactions/#std-label-transactions-create-collections-indexes).

## Nén dữ liệu trong MongoDB

Nhờ WiredTiger Storage Engine (Storage Engine mặc định từ MongoDB 3.2 trở đi), MongoDB hỗ trợ nén toàn bộ Collection và Index. Nén sẽ tối đa hóa việc giảm dung lượng lưu trữ với cái giá phải trả là thêm CPU.

Mặc định, WiredTiger sử dụng thuật toán nén [Snappy](https://github.com/google/snappy) (Google mở nguồn, nhằm đạt được tốc độ rất cao và tỉ lệ nén hợp lý, tỉ lệ nén 3~5 lần) để dùng Block Compression cho tất cả Collection, dùng Prefix Compression cho tất cả Index.

Ngoài Snappy, đối với Collection còn có các thuật toán nén dưới đây:

- [zlib](https://github.com/madler/zlib): Thuật toán nén cao, tỉ lệ nén 5~7 lần
- [Zstandard](https://github.com/facebook/zstd) (gọi tắt là zstd): Thuật toán nén không mất dữ liệu tốc độ cao mở nguồn bởi Facebook, nhắm vào kịch bản nén thời gian thực mức zlib và tỉ lệ nén tốt hơn, cung cấp tỉ lệ nén cao hơn và mức sử dụng CPU thấp hơn, có sẵn từ MongoDB 4.2.

WiredTiger Log cũng được nén, mặc định cũng sử dụng thuật toán nén Snappy. Nếu bản ghi Log nhỏ hơn hoặc bằng 128 bytes, WiredTiger sẽ không nén bản ghi đó.

## Sự khác biệt giữa Amazon DocumentDB và MongoDB

Amazon DocumentDB (tương thích với MongoDB) là một dịch vụ Database nhanh chóng, tin cậy, hoàn toàn được quản lý (Fully-managed). Amazon DocumentDB có thể dễ dàng thiết lập, vận hành và mở rộng Database tương thích với MongoDB trên Cloud.

### Toán tử `$vectorSearch`

Amazon DocumentDB không hỗ trợ `$vectorSearch` làm operator độc lập. Thay vào đó, chúng tôi hỗ trợ `vectorSearch` bên trong operator `$search`. Để biết thêm thông tin, xin tham khảo [Tìm kiếm Vector Amazon DocumentDB](https://docs.aws.amazon.com/zh_cn/documentdb/latest/developerguide/vector-search.html).

### `OpCountersCommand`

Hành vi `OpCountersCommand` của Amazon DocumentDB lệch so với `opcounters.command` của MongoDB như sau:

- `opcounters.command` của MongoDB tính tất cả lệnh ngoại trừ Insert, Update và Delete, trong khi `OpCountersCommand` của Amazon DocumentDB cũng loại trừ lệnh `find`.
- Amazon DocumentDB tính các lệnh nội bộ (ví dụ `getCloudWatchMetricsV2`) vào `OpCountersCommand`.

### Quản lý Database và Collection

Amazon DocumentDB không hỗ trợ Database admin hoặc local, cũng không hỗ trợ Collection `system.*` hoặc `startup_log` của MongoDB.

### `cursormaxTimeMS`

Trong Amazon DocumentDB, `cursor.maxTimeMS` Reset counter của mỗi Request. Do đó nếu chỉ định `maxTimeMS` là 3000ms, truy vấn đó tốn 2800ms, còn mỗi Request `getMore` tiếp theo tốn 300ms, thì Cursor sẽ không Timeout. Cursor chỉ Timeout khi thao tác đơn lẻ (bất kể là truy vấn hay 1 Request `getMore` đơn lẻ) tốn thời gian vượt quá giá trị `maxTimeMS` chỉ định. Ngoài ra, Scanner kiểm tra thời gian thực thi của Cursor chạy theo chu kỳ mỗi 5 phút.

### explain()

Amazon DocumentDB mô phỏng MongoDB 4.0 API trên Engine DB chuyên dụng tận dụng hệ thống lưu trữ phân tán, chịu lỗi (fault-tolerant), tự phục hồi. Do đó Query Plan và Output của `explain()` giữa Amazon DocumentDB và MongoDB có thể khác nhau. Khách hàng muốn kiểm soát Query Plan có thể dùng operator `$hint` để ép buộc chọn Index ưu tiên.

### Giới hạn tên Field

Amazon DocumentDB không hỗ trợ dấu chấm "." trong tên Field của Document, ví dụ `db.foo.insert({'x.1':1})`.

Amazon DocumentDB cũng không hỗ trợ tiền tố $ trong tên Field.

Ví dụ, thử lệnh dưới đây trong Amazon DocumentDB hoặc MongoDB:

```shell
rs0:PRIMARY< db.foo.insert({"a":{"$a":1}})
```

MongoDB sẽ trả về:

```shell
WriteResult({ "nInserted" : 1 })
```

Amazon DocumentDB sẽ trả về lỗi:

```shell
WriteResult({
  "nInserted" : 0,
  "writeError" : {
    "code" : 2,
    "errmsg" : "Document can't have $ prefix field names: $a"
  }
})
```

## Tài liệu tham khảo

- Tài liệu chính thức MongoDB (Tài liệu tham khảo chính, lấy tài liệu chính thức làm chuẩn): <https://www.mongodb.com/docs/manual/>
- 《MongoDB Definitive Guide》
- Nguyên lý Transaction trong MongoDB - Cộng đồng MongoDB Trung Quốc: <https://mongoing.com/archives/82187>
- Transactions - Tài liệu chính thức MongoDB: <https://www.mongodb.com/docs/manual/core/transactions/>
- WiredTiger Storage Engine - Tài liệu chính thức MongoDB: <https://www.mongodb.com/docs/manual/core/wiredtiger/>
- Chuỗi bài viết WiredTiger Storage Engine 1: Phân tích cấu trúc dữ liệu cơ bản: <https://mongoing.com/topic/archives-35143>

<!-- @include: @article-footer.snippet.md -->
