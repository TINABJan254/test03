---
title: MongoDB常见面试题总结（下）
description: MongoDB常见面试题总结下篇，深入讲解MongoDB各类索引（单字段、复合、多键、文本、地理位置、TTL）的原理、使用场景和查询优化技巧。
category: 数据库
tag:
  - NoSQL
  - MongoDB
head:
  - - meta
    - name: keywords
      content: MongoDB索引,复合索引,多键索引,文本索引,地理位置索引,TTL索引,MongoDB查询优化,索引设计
---

## Index trong MongoDB

### Index trong MongoDB có tác dụng gì?

Tương tự như RDBMS, trong MongoDB cũng có Index. Mục đích của Index chủ yếu dùng để nâng cao hiệu suất truy vấn, nếu không có Index, MongoDB bắt buộc phải thực thi **Collection Scan**, tức là quét từng Document trong Collection để lựa chọn Document khớp với câu lệnh truy vấn. Nếu truy vấn tồn tại Index phù hợp, MongoDB có thể sử dụng Index đó để giới hạn số lượng Document mà nó bắt buộc phải kiểm tra. Đồng thời, MongoDB có thể sử dụng thứ tự sắp xếp trong Index để trả về kết quả đã được sắp xếp.

Mặc dù Index có thể rút ngắn đáng kể thời gian truy vấn, nhưng việc sử dụng Index và bảo trì Index đều phải trả giá. Khi thực thi thao tác ghi (Write), ngoài việc phải cập nhật Document, còn bắt buộc phải cập nhật Index, điều này chắc chắn sẽ ảnh hưởng đến hiệu năng ghi. Do đó, khi có số lượng thao tác ghi lớn mà thao tác đọc ít, hoặc khi không cần xem xét hiệu năng thao tác đọc, đều không khuyến nghị tạo Index.

### MongoDB hỗ trợ những loại Index nào?

**MongoDB hỗ trợ nhiều loại Index, bao gồm Single Field Index, Compound Index, Multikey Index, Hashed Index, Text Index, Geospatial Index, v.v., mỗi loại Index có trường hợp sử dụng khác nhau.**

- **Single Field Index:** Index tạo trên 1 field đơn lẻ, thứ tự sắp xếp khi tạo Index không quan trọng, MongoDB có thể duyệt từ đầu hoặc từ cuối.
- **Compound Index:** Index tạo trên nhiều field, cũng có thể gọi là Composite Index hay Joint Index.
- **Multikey Index**: Một field trong MongoDB có thể là Array, khi tạo Index trên field loại này thì đó là Multikey Index. MongoDB sẽ tạo Index cho từng giá trị của Array. Có nghĩa là bạn có thể làm điều kiện truy vấn theo các giá trị bên trong Array, lúc này vẫn sẽ chạy qua Index.
- **Hashed Index**: Index theo giá trị Hash của dữ liệu, dùng cho Hashed Sharded Cluster.
- **Text Index:** Hỗ trợ truy vấn Text Search đối với nội dung chuỗi. Text Index có thể bao gồm bất kỳ field nào có giá trị là chuỗi hoặc mảng phần tử chuỗi. Một Collection chỉ có thể có 1 Text Search Index, nhưng Index đó có thể bao phủ nhiều field. Mặc dù MongoDB hỗ trợ Full-text Index, nhưng hiệu năng kém, tạm thời không khuyến nghị sử dụng.
- **Geospatial Index:** Index dựa trên kinh độ vĩ độ (Coordinates), phù hợp cho truy vấn vị trí 2D và 3D.
- **Unique Index**: Đảm bảo field Index sẽ không lưu giữ các giá trị trùng lặp. Nếu Collection đã tồn tại Document vi phạm Unique Constraint của Index thì việc tạo Unique Index ở Background sẽ thất bại.
- **TTL Index**: TTL Index cung cấp một cơ chế hết hạn (Expiration Mechanism), cho phép thiết lập thời gian hết hạn cho từng Document, khi 1 Document đạt đến thời gian hết hạn đã đặt trước thì sẽ bị xóa.
- ……

### Thứ tự của các field trong Compound Index có ảnh hưởng gì không?

Thứ tự của các field trong Compound Index cực kỳ quan trọng, ví dụ Compound Index trong hình dưới được cấu thành từ `{userid:1, score:-1}`, thì Compound Index này đầu tiên sẽ sắp xếp tăng dần theo `userid`; sau đó trong từng giá trị `userid`, lại tiếp tục sắp xếp giảm dần theo `score`.

![复合索引](https://oss.javaguide.cn/github/javaguide/database/mongodb/mongodb-composite-index.png)

Trong Compound Index, sắp xếp theo phương thức nào quyết định Index đó có thể được áp dụng trong truy vấn hay không.

Sắp xếp chạy qua (tận dụng được) Compound Index:

```sql
db.s2.find().sort({"userid": 1, "score": -1})
db.s2.find().sort({"userid": -1, "score": 1})
```

Sắp xếp không chạy qua Compound Index:

```sql
db.s2.find().sort({"userid": 1, "score": 1})
db.s2.find().sort({"userid": -1, "score": -1})
db.s2.find().sort({"score": 1, "userid": -1})
db.s2.find().sort({"score": 1, "userid": 1})
db.s2.find().sort({"score": -1, "userid": -1})
db.s2.find().sort({"score": -1, "userid": 1})
```

Chúng ta có thể thông qua explain để phân tích:

```sql
db.s2.find().sort({"score": -1, "userid": 1}).explain()
```

### Compound Index có tuân theo quy tắc Tiền tố bên trái (Left-prefix Principle) không?

**Compound Index trong MongoDB tuân theo quy tắc Tiền tố bên trái**: Index sở hữu nhiều Key có thể đồng thời thu được tất cả các Index được cấu thành từ tiền tố của các Key này, nhưng không bao gồm các tập con khác ngoại trừ Left-prefix. Ví dụ như có 1 Index dạng `{a: 1, b: 1, c: 1, ..., z: 1}`, thì trên thực tế cũng tương đương với việc có một loạt các Index như `{a: 1}`, `{a: 1, b: 1}`, `{a: 1, b: 1, c: 1}`, v.v., nhưng sẽ không có Index không phải tiền tố bên trái như `{b: 1}`.

### TTL Index là gì?

TTL Index cung cấp một cơ chế hết hạn, cho phép thiết lập thời gian hết hạn `expireAfterSeconds` cho từng Document, khi 1 Document đạt đến thời gian hết hạn đặt trước thì sẽ bị xóa. Ngoại trừ thuộc属性 `expireAfterSeconds`, TTL Index giống hệt như Index thông thường.

Dữ liệu hết hạn rất hữu ích cho một số loại thông tin, ví dụ như Event Data do máy tạo ra, Log và Session info, những thông tin này chỉ cần lưu giữ trong Database một khoảng thời gian hữu hạn.

**Nguyên lý hoạt động của TTL Index:**

- MongoDB sẽ mở một Background Thread để đọc giá trị TTL Index này nhằm kiểm tra Document có hết hạn hay không, tuy nhiên không đảm bảo dữ liệu đã hết hạn sẽ bị xóa lập tức, vì Background Thread cứ 60 giây lại kích hoạt một nhiệm vụ xóa 1 lần, và nếu dung lượng dữ liệu xóa lớn thì có thể xuất hiện trường hợp lần xóa trước chưa hoàn thành mà nhiệm vụ lần sau đã bắt đầu, dẫn đến dữ liệu hết hạn cũng sẽ xuất hiện hiện tượng vượt quá thời gian lưu trữ dữ liệu từ 60 giây trở lên.
- Đối với Replica Set, tiến trình Background của TTL Index chỉ được bật trên Node Primary, ở Node Secondary sẽ luôn ở trạng thái Idle, việc xóa dữ liệu của Node Secondary được đồng bộ thông qua oplog sinh ra sau khi Node Primary xóa.

**Giới hạn của TTL Index:**

- TTL Index là Single Field Index. Compound Index không hỗ trợ TTL
- Field `_id` không hỗ trợ TTL Index.
- Không thể tạo TTL Index trên Capped Collection, vì MongoDB không thể xóa Document khỏi Capped Collection.
- Nếu một field đã tồn tại Non-TTL Index thì trên field đó không thể tạo thêm TTL Index.

### Covered Index Query là gì?

Theo giới thiệu từ tài liệu chính thức, Covered Query là câu truy vấn thỏa mãn:

- Tất cả các field truy vấn đều là một phần của Index.
- Tất cả các field trả về trong kết quả đều nằm trong cùng 1 Index.
- Trong truy vấn không có field nào bằng `null`.

Do tất cả các field xuất hiện trong truy vấn đều là một phần của Index, MongoDB không cần phải truy xuất toàn bộ Data Document để tìm các điều kiện khớp và trả về kết quả truy vấn sử dụng cùng 1 Index. Vì Index tồn tại trong bộ nhớ (RAM), việc lấy dữ liệu từ Index nhanh hơn rất nhiều so với đọc dữ liệu qua Scan Document.

Ví dụ: Chúng ta có Collection `users` như sau:

```json
{
   "_id": ObjectId("53402597d852426020000002"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "gender": "M",
   "name": "Tom Benzamin",
   "user_name": "tombenzamin"
}
```

Chúng ta tạo Compound Index trong Collection `users`, các field là `gender` và `user_name`:

```sql
db.users.ensureIndex({gender:1,user_name:1})
```

Bây giờ, Index này sẽ cover được truy vấn dưới đây:

```sql
db.users.find({gender:"M"},{user_name:1,_id:0})
```

Để cho Index chỉ định cover được truy vấn, bắt buộc phải chỉ định tường minh `_id: 0` để loại trừ field `_id` khỏi kết quả, vì Index không bao gồm field `_id`.

## High Availability trong MongoDB

### Replica Set (Cụm nhân bản)

#### Replica Set là gì?

Replica Set trong MongoDB còn gọi là Cụm bản sao, là một nhóm các tiến trình `mongod` duy trì cùng một tập hợp dữ liệu.

Client kết nối tới toàn bộ Replica Set MongoDB, Node Primary chịu trách nhiệm cho các thao tác ghi (Write) của toàn bộ Replica Set, Node Secondary có thể thực hiện thao tác đọc (Read), nhưng mặc định vẫn là Node Primary chịu trách nhiệm cho thao tác đọc của toàn bộ Replica Set. Khi Node Primary gặp sự cố, hệ thống tự động bầu chọn (Elect) một Node Primary mới từ các Node Secondary, đảm bảo Cluster hoạt động bình thường, việc này là hoàn toàn trong suốt (Transparent) đối với Client.

Thông thường, một Replica Set bao gồm 1 Node Primary, nhiều Node Secondary và 0 hoặc 1 Node Arbiter (Node trọng tài).

- **Node Primary**: Cổng vào thao tác ghi cho toàn bộ Cluster, tiếp nhận tất cả thao tác ghi, và ghi lại toàn bộ thay đổi của Collection vào Log thao tác, tức oplog. Sau khi Node Primary bị sập sẽ tự động bầu chọn Node Primary mới.
- **Node Secondary**: Đồng bộ dữ liệu từ Node Primary, bầu chọn node mới sau khi Node Primary bị sập. Tuy nhiên, Node Secondary có thể cấu hình Priority = 0 để ngăn nó trở thành Node Primary khi bầu chọn.
- **Node Arbiter (Node trọng tài)**: Node này dùng để tiết kiệm tài nguyên hoặc chịu lỗi nhiều Data Center (Multi-datacenter Disaster Recovery), chỉ chịu trách nhiệm bỏ phiếu khi bầu chọn Node Primary chứ không lưu dữ liệu, đảm bảo có node nhận được số phiếu tán thành chiếm đa số.

Hình dưới đây là một Replica Set 3 thành viên điển hình:

![](https://oss.javaguide.cn/github/javaguide/database/mongodb/replica-set-read-write-operations-primary.png)

Giữa Node Primary và Node Secondary đồng bộ dữ liệu thông qua **oplog (Log thao tác)**. oplog là một **Capped Collection (Tập hợp giới hạn)** đặc biệt thuộc DB local, dùng để lưu giữ Incremental Log sinh ra bởi thao tác ghi, tương tự Binlog trong MySQL.

> Capped Collection tương tự như hàng đợi vòng (Circular Queue) độ dài cố định, dữ liệu được Append tuần tự vào đuôi Collection, khi dung lượng Collection đạt đến giới hạn trên, nó sẽ ghi đè lên các Document cũ nhất trong Collection. Dữ liệu của Capped Collection sẽ được ghi tuần tự vào không gian cố định trên đĩa, do đó tốc độ I/O cực kỳ nhanh, nếu không tạo Index thì hiệu năng càng tốt hơn.

![](https://oss.javaguide.cn/github/javaguide/database/mongodb/replica-set-primary-with-two-secondaries.png)

Khi một thao tác ghi trên Node Primary hoàn thành, nó sẽ ghi một bản ghi log tương ứng vào Collection oplog, còn Node Secondary sẽ thông qua oplog này liên tục kéo Log mới về, thực hiện Replay tại cục bộ để đạt mục đích đồng bộ dữ liệu.

Replica Set có tối đa 1 Node Primary. Nếu Node Primary hiện tại không khả dụng, một cuộc bầu chọn sẽ quyết định ra Node Primary mới. Quy tắc bầu chọn node của MongoDB có thể đảm bảo sau khi Primary sập, node mới được chọn chắc chắn là node có dữ liệu đầy đủ nhất trong Cluster.

#### Tại sao phải dùng Replica Set?

- **Thực hiện Failover**: Cung cấp tính năng tự động phục hồi sự cố, khi Node Primary bị lỗi sẽ tự động bầu chọn Node Primary mới từ các Node Secondary, đảm bảo Cluster hoạt động bình thường, việc này hoàn toàn trong suốt đối với Client.
- **Thực hiện Phân tách đọc ghi (Read-Write Splitting)**: Chúng ta có thể thiết lập đọc dữ liệu trên Node Secondary, Node Primary chịu trách nhiệm ghi dữ liệu, như vậy sẽ thực hiện phân tách đọc ghi, giảm bớt áp lực đọc ghi quá lớn cho Node Primary. Trước phiên bản MongoDB 4.0, nếu áp lực Master DB không lớn thì không khuyến nghị phân tách đọc ghi, vì ghi sẽ làm Block đọc, trừ khi nghiệp vụ không quá quan tâm tới Response Time cũng như chấp nhận độ trễ thời gian nhất định khi đọc dữ liệu lịch sử.

### Sharded Cluster (Cụm分片)

#### Sharded Cluster là gì?

Sharded Cluster là phiên bản phân tán của MongoDB, so với Replica Set, dữ liệu trong Sharded Cluster được phân bố cân bằng trên các Shard khác nhau, không chỉ nâng cao đáng kể giới hạn dung lượng dữ liệu của toàn bộ Cluster mà còn phân tán áp lực đọc ghi sang các Shard khác nhau để giải quyết bài toán nghẽn hiệu năng của Replica Set.

Sharded Cluster của MongoDB bao gồm 3 phần dưới đây (Hình ảnh trích từ [Giới thiệu Sharded Cluster trong tài liệu chính thức](https://www.mongodb.com/docs/manual/sharding/)):

![](https://oss.javaguide.cn/github/javaguide/database/mongodb/sharded-cluster-production-architecture.png)

- **Config Servers**: Server cấu hình, bản chất là một Replica Set MongoDB, chịu trách nhiệm lưu trữ các loại Metadata và cấu hình của Cluster, như địa chỉ Shard, Chunk, v.v.
- **Mongos**: Dịch vụ Routing, không lưu dữ liệu cụ thể, lấy cấu hình Cluster từ Config và Forward Request tới Shard cụ thể, đồng thời tổng hợp kết quả của Shard trả về cho Client.
- **Shard**: Mỗi Shard là một tập con của toàn bộ dữ liệu, từ phiên bản MongoDB 3.6 trở đi, mỗi Shard bắt buộc phải triển khai dưới dạng kiến trúc Replica Set.

#### Tại sao phải dùng Sharded Cluster?

Theo sự tăng trưởng của dung lượng dữ liệu và Throughput của hệ thống, giải pháp thường gặp có 2 loại: Vertical Scaling (Mở rộng theo chiều dọc) và Horizontal Scaling (Mở rộng theo chiều ngang).

Vertical Scaling được thực hiện bằng cách tăng năng lực của 1 Server đơn lẻ, ví dụ dung lượng đĩa, RAM, CPU; Horizontal Scaling được thực hiện bằng cách lưu trữ dữ liệu sang nhiều Server, bổ sung thêm Server khi cần để tăng dung lượng.

Tương tự Redis Cluster, MongoDB cũng có thể thông qua Sharding để thực hiện **Horizontal Scaling**. Phương thức Horizontal Scaling linh hoạt hơn, có thể đáp ứng nhu cầu lưu trữ dữ liệu lớn hơn, hỗ trợ Throughput cao hơn. Đồng thời chi phí tổng thể cho Horizontal Scaling thấp hơn, chỉ cần các Server đơn lẻ cấu hình tương đối thấp là được, cái giá phải trả là làm tăng cơ sở hạ tầng triển khai và độ phức tạp khi bảo trì.

Nói cách khác khi bạn gặp phải các vấn đề dưới đây, có thể dùng Sharded Cluster để giải quyết:

- Dung lượng lưu trữ bị giới hạn bởi 1 máy đơn lẻ, tức tài nguyên đĩa gặp nghẽn (bottleneck).
- Năng lực đọc ghi bị giới hạn bởi 1 máy đơn lẻ, có thể là CPU, RAM hoặc Card mạng gặp nghẽn khiến năng lực đọc ghi không thể mở rộng.

#### Shard Key là gì?

**Shard Key (Khóa分片)** là tiền đề cho phân vùng dữ liệu, từ đó thực hiện phân phối dữ liệu sang các Server khác nhau, giảm bớt gánh nặng cho Server. Nói cách khác, Shard Key quyết định tình trạng phân bố của các Document trong Collection trên các Shard của Cluster.

Shard Key chính là một field bên trong Document, tuy nhiên field này không phải là field thông thường mà có các yêu cầu nhất định:

- Nó bắt buộc phải xuất hiện trong tất cả các Document.
- Nó bắt buộc phải là một Index của Collection, có thể là Single Index hoặc Index tiền tố của Compound Index, không thể là Multikey Index, Text Index hay Geospatial Index.
- Trước phiên bản MongoDB 4.2, giá trị field Shard Key của Document là bất biến (immutable). Từ phiên bản MongoDB 4.2 trở đi, trừ khi field Shard Key là field `_id` bất biến, nếu không bạn có thể UPDATE giá trị Shard Key của Document. Từ phiên bản MongoDB 5.0 trở đi, hệ thống đã triển khai Live Resharding, có thể thực hiện chọn lại hoàn toàn Shard Key.
- Kích thước của nó không được vượt quá 512 bytes.

#### Lựa chọn Shard Key như thế nào?

Lựa chọn Shard Key phù hợp có ảnh hưởng rất lớn đến hiệu quả Sharding, chủ yếu dựa trên 4 yếu tố dưới đây (Trích từ [Các lưu ý khi dùng Sharded Cluster - Tài liệu Tencent Cloud](https://cloud.tencent.com/document/product/240/44611)):

- **Cardinality (Số lượng giá trị phân biệt)**: Khuyến nghị Cardinality càng lớn càng tốt, nếu dùng Shard Key có Cardinality nhỏ, vì các giá trị dự phòng có hạn nên tổng số lượng Chunk sẽ có hạn, theo sự tăng lên của dữ liệu, kích thước Chunk sẽ ngày càng lớn, dẫn đến khi Horizontal Scaling việc di chuyển Chunk sẽ cực kỳ khó khăn. Ví dụ: Chọn tuổi (Age) làm Cardinality, phạm vi tối đa chỉ có 100 giá trị, theo sự tăng lên của dữ liệu, khi cùng một giá trị phân bố quá nhiều sẽ dẫn tới sự tăng trưởng của Chunk vượt quá phạm vi Chunk Size, gây ra Jumbo Chunk, từ đó không thể Migrate, dẫn tới phân bố dữ liệu không đồng đều, nghẽn hiệu năng.
- **Value Distribution (Phân bố giá trị)**: Khuyến nghị phân bố giá trị càng đồng đều càng tốt, Shard Key phân bố không đồng đều sẽ làm cho dung lượng dữ liệu của một số Chunk rất lớn, cũng sẽ gặp phải vấn đề phân bố dữ liệu không đồng đều và nghẽn hiệu năng như trên.
- **Query kèm Shard Key**: Khi truy vấn khuyến nghị kèm theo Shard Key, khi dùng Shard Key làm điều kiện truy vấn, `mongos` có thể định vị trực tiếp tới Shard cụ thể, nếu không `mongos` cần phải phân phối truy vấn tới tất cả các Shard rồi chờ Response trả về.
- **Tránh tăng hoặc giảm đơn điệu**: Shard Key tăng đơn điệu có việc di chuyển data file nhỏ, nhưng ghi (Write) sẽ bị tập trung, dẫn đến dung lượng dữ liệu của đoạn cuối cùng liên tục tăng lên và xảy ra Migrate liên tục, giảm đơn điệu cũng tương tự.

Tóm lại, khi lựa chọn Shard Key cần cân nhắc 4 điều kiện trên, cố gắng thỏa mãn càng nhiều điều kiện càng tốt để giảm thiểu ảnh hưởng của MoveChunks tới hiệu năng, từ đó đạt trải nghiệm hiệu năng tối ưu nhất.

#### Có những chiến lược Sharding nào?

MongoDB hỗ trợ 2 thuật toán Sharding để đáp ứng các nhu cầu truy vấn khác nhau (Trích từ [Giới thiệu Sharded Cluster MongoDB - Tài liệu Alibaba Cloud](https://help.aliyun.com/document_detail/64561.html)):

**1. Range-based Sharding (Sharding dựa trên khoảng)**:

![](https://oss.javaguide.cn/github/javaguide/database/mongodb/example-of-scope-based-sharding.png)

MongoDB chia dữ liệu thành các Chunk khác nhau theo phạm vi giá trị của Shard Key, mỗi Chunk chứa dữ liệu trong một khoảng phạm vi. Khi Shard Key có Cardinality lớn, tần suất thấp và giá trị không thay đổi đơn điệu thì Range Sharding sẽ hiệu quả hơn.

- Ưu điểm: `mongos` có thể nhanh chóng định vị dữ liệu Request cần, và Forward Request tới Node Shard tương ứng.
- Nhược điểm: Có thể dẫn tới dữ liệu phân bố không đồng đều trên các Node Shard, dễ tạo thành Hotspot đọc ghi, và không có tính chất phân tán ghi.
- Kịch bản áp dụng: Giá trị Shard Key không tăng hoặc giảm đơn điệu, Cardinality của Shard Key lớn và tần suất trùng lặp thấp, các kịch bản nghiệp vụ cần Range Query, v.v.

**2. Hash-based Sharding (Sharding dựa trên giá trị Hash)**:

![](https://oss.javaguide.cn/github/javaguide/database/mongodb/example-of-hash-based-sharding.png)

MongoDB tính toán giá trị Hash của 1 field làm giá trị Index, và chia dữ liệu thành các Chunk khác nhau dựa trên phạm vi giá trị Hash.

- Ưu điểm: Có thể phân bố dữ liệu đồng đều hơn giữa các Node Shard, có tính chất phân tán ghi.
- Nhược điểm: Không phù hợp thực hiện Range Query, khi thực hiện Range Query cần phân phối Read Request tới tất cả các Node Shard.
- Kịch bản áp dụng: Giá trị Shard Key có hiện tượng tăng hoặc giảm đơn điệu, Cardinality của Shard Key lớn và tần suất trùng lặp thấp, dữ liệu cần ghi phân phối ngẫu nhiên, độ ngẫu nhiên khi đọc dữ liệu lớn, v.v.

Ngoài 2 chiến lược Sharding trên, bạn còn có thể cấu hình **Compound Shard Key**, ví dụ bao gồm 1 Key có Cardinality thấp và 1 Key tăng đơn điệu.

#### Dữ liệu Sharding được lưu trữ như thế nào?

**Chunk (Khối)** là một khái niệm cốt lõi của Sharded Cluster trong MongoDB, bản chất của nó là một đơn vị dữ liệu logic được cấu thành từ một nhóm Document. Mỗi Chunk chứa dữ liệu Shard Key trong một phạm vi nhất định, không giao nhau và hợp lại thành toàn bộ dữ liệu (khái niệm **Phân hoạch - Partition** trong Toán rời rạc).

Sharded Cluster không ghi lại từng bản ghi dữ liệu nằm trên Shard nào, mà ghi lại Chunk nằm trên Shard nào cũng như Chunk đó bao gồm những dữ liệu nào.

Mặc định, kích thước tối đa của 1 Chunk là 64MB (có thể điều chỉnh, khoảng giá trị từ 1~1024 MB. Nếu không có nhu cầu đặc biệt khuyến nghị giữ giá trị mặc định), khi thực hiện INSERT, UPDATE, DELETE dữ liệu, nếu lúc此时 `mongos` nhận biết được kích thước của Chunk mục tiêu hoặc dung lượng dữ liệu bên trong vượt quá giới hạn上, sẽ kích hoạt **Chunk Splitting (Phân tách Chunk)**.

![Chunk 分裂](https://oss.javaguide.cn/github/javaguide/database/mongodb/chunk-splitting-shard-a.png)

Sự tăng trưởng của dữ liệu làm cho Chunk phân tách ngày càng多. Lúc này, số量Chunk在từng Shard有thể bị mất cân bằng. Component **Balancer (Bộ cân bằng)**在`mongos` sẽ thực hiện tự动cân bằng, cố gắng làm cho số量Chunk在các Shard giữ在mức cân bằng, quá trình限制gọi là **Rebalance (Tái cân bằng)**. Mặc定, Rebalance của Database和Collection được BẬT.

Như hình minh họa, theo sự INSERT của dữ liệu dẫn tới Chunk分裂, làm cho 2 Shard A B有3 Chunk, Shard C chỉ有1 Chunk, lúc限制sẽ Migrate 1 Chunk从B phân phối sang Shard C để thực hiện cân bằng dữ liệu Cluster.

![Chunk 迁移](https://oss.javaguide.cn/github/javaguide/database/mongodb/mongo-reblance-three-shards.png)

> Balancer是MongoDB的一个运行在 Config Server 的 Primary 节点上(自 MongoDB 3.4 版本起)的后台进程，它监控每个分片上 Chunk 数量，并在某个分片上 Chunk 数量达到阈值进行迁移。

Chunk chỉ分裂chứ不gộp方案, ngay cả khi值Chunk Size tăng上.

Thao tác Rebalance tương đối tốn tài nguyên hệ thống, chúng ta có thể giảm thiểu ảnh hưởng của nó到việc sử dụng bình常của MongoDB bằng各phương thức như thực hiện vào giờ thấp điểm nghiệp vụ, Pre-sharding或者thiết lập Time Window cho Rebalance.

#### Nguyên lý Migrate Chunk là gì?

Về giới thiệu chi tiết nguyên lý Migrate Chunk, khuyến nghị đọc bài viết [Một bài đọc hiểu Migrate Chunk trong MongoDB](https://mongoing.com/archives/77479) trên Cộng đồng MongoDB Trung Quốc.

## Tài liệu học tập đề xuất

- [Sổ tay tiếng Trung MongoDB | Bản dịch tiếng Trung tài liệu chính thức](https://docs.mongoing.com/) (Khuyến nghị): Dựa trên phiên bản 4.2, liên tục đồng bộ với phiên bản chính thức mới nhất.
- [Hướng dẫn cho người mới bắt đầu MongoDB - 7 ngày học MongoDB](https://mongoing.com/archives/docs/mongodb%e5%88%9d%e5%ad%a6%e8%80%85%e6%95%99%e7%a8%8b/mongodb%e5%a6%82%e4%bd%95%e5%88%9b%e5%bb%ba%e6%95%b0%e6%8d%ae%e5%ba%93%e5%92%8c%e9%9b%86%e5%90%88): Nhanh chóng nhập môn.
- [Thực chiến SpringBoot tích hợp MongoDB - 2022](https://www.cnblogs.com/dxflqm/p/16643981.html): Bài viết nhập môn MongoDB rất hay, chủ yếu xoay quanh việc sử dụng Java Client của MongoDB để giới thiệu các thao tác CRUD cơ bản.

## Tài liệu tham khảo

- Tài liệu chính thức MongoDB (Tài liệu tham khảo chính, lấy tài liệu chính thức làm chuẩn): <https://www.mongodb.com/docs/manual/>
- 《MongoDB Definitive Guide》
- Indexes - Tài liệu chính thức MongoDB: <https://www.mongodb.com/docs/manual/indexes/>
- MongoDB - Kiến thức Index - Lập trình viên Xiangzai - 2022: <https://fatedeity.cn/posts/database/mongodb-index-knowledge.html>
- MongoDB - Index: <https://www.cnblogs.com/Neeo/articles/14325130.html>
- Sharding - Tài liệu chính thức MongoDB: <https://www.mongodb.com/docs/manual/sharding/>
- Giới thiệu Sharded Cluster MongoDB - Tài liệu Alibaba Cloud: <https://help.aliyun.com/document_detail/64561.html>
- Các lưu ý khi dùng Sharded Cluster - Tài liệu Tencent Cloud: <https://cloud.tencent.com/document/product/240/44611>

<!-- @include: @article-footer.snippet.md -->
