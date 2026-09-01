---
title: NoSQL基础常见面试题总结
description: NoSQL数据库基础面试题和知识总结，包括NoSQL与SQL的区别、NoSQL的优势、四种NoSQL数据库类型（键值、文档、图形、宽列）及其代表产品Redis、MongoDB、Neo4j等的应用场景。
category: 数据库
tag:
  - NoSQL
  - MongoDB
  - Redis
head:
  - - meta
    - name: keywords
      content: NoSQL,Redis,MongoDB,HBase,Cassandra,键值数据库,文档数据库,图数据库,宽列存储,SQL与NoSQL区别
---

## NoSQL là gì?

NoSQL (viết tắt của Not Only SQL) dùng để chỉ các database phi quan hệ (non-relational), chủ yếu hướng tới các kiểu lưu trữ dữ liệu dạng Key-Value, Document, Graph và Wide-Column. Đồng thời, NoSQL Database bản chất đã hỗ trợ các đặc tính phân tán, dư thừa dữ liệu (data redundancy) và Sharding (phân mảnh dữ liệu), nhằm cung cấp giải pháp lưu trữ dữ liệu có tính khả mở (scalability), High Availability và High Performance.

Một hiểu lầm phổ biến là NoSQL Database hoặc non-relational database không thể lưu trữ tốt dữ liệu quan hệ. NoSQL Database hoàn toàn có thể lưu trữ dữ liệu quan hệ — chỉ là chúng lưu trữ theo cách khác với Relational Database.

Các đại diện tiêu biểu của NoSQL Database: HBase, Cassandra, MongoDB, Redis.

![](https://oss.javaguide.cn/github/javaguide/database/mongodb/sql-nosql-tushi.png)

## SQL và NoSQL khác nhau như thế nào?

| | SQL Database | NoSQL Database |
| :----------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Model lưu trữ dữ liệu | Lưu trữ có cấu trúc, dạng bảng với các dòng (row) và cột (column) cố định | Lưu trữ phi cấu trúc (hoặc bán cấu trúc). Document: JSON document; Key-Value: cặp key-value; Wide-column: bảng chứa các row và dynamic column; Graph: node và edge |
| Lịch sử phát triển | Được phát triển từ những năm 1970, trọng tâm là giảm dư thừa dữ liệu | Được phát triển từ cuối những năm 2000, trọng tâm là nâng cao tính khả mở (Scalability), giảm chi phí lưu trữ dữ liệu quy mô lớn |
| Ví dụ | Oracle, MySQL, Microsoft SQL Server, PostgreSQL | Document: MongoDB, CouchDB; Key-Value: Redis, DynamoDB; Wide-column: Cassandra, HBase; Graph: Neo4j, Amazon Neptune, Giraph |
| Thuộc tính ACID | Cung cấp các thuộc tính Atomicity, Consistency, Isolation, Durability (ACID) | Thường không hỗ trợ Transaction ACID (đã đánh đổi để lấy Scalability và High Performance), một số ít hỗ trợ như MongoDB. Tuy nhiên sự hỗ trợ Transaction ACID của MongoDB vẫn có điểm khác biệt so với MySQL. |
| Hiệu năng | Hiệu năng thường phụ thuộc vào subsystem của ổ đĩa. Để đạt hiệu năng tối ưu, thường cần tối ưu câu lệnh query, Index và cấu trúc bảng. | Hiệu năng thường được quyết định bởi quy mô cluster phần cứng tầng dưới, latency mạng và ứng dụng gọi đến. |
| Khả năng mở rộng | Scale Up (dùng máy chủ mạnh hơn để mở rộng theo chiều dọc), Read-Write Splitting (phân tách đọc/ghi), Sharding (phân kho phân bảng) | Scale Out (mở rộng hàng ngang bằng cách thêm các máy chủ, thường dựa trên cơ chế Sharding) |
| Mục đích sử dụng | Lưu trữ dữ liệu cho các dự án cấp doanh nghiệp thông thường | Mục đích sử dụng rộng rãi, ví dụ Graph Database hỗ trợ phân tích và duyệt mối quan hệ giữa các dữ liệu kết nối, Key-Value Database có thể xử lý mở rộng lượng dữ liệu khổng lồ và sự thay đổi trạng thái cực cao |
| Cú pháp truy vấn | Ngôn ngữ truy vấn có cấu trúc (SQL) | Cú pháp truy cập dữ liệu có thể khác nhau tùy thuộc vào từng database |

## NoSQL Database có những ưu thế gì?

NoSQL Database rất thích hợp cho nhiều ứng dụng hiện đại, chẳng hạn như ứng dụng Mobile, Web và Game, nơi cần một database linh hoạt, có thể mở rộng, hiệu năng cao và giàu tính năng để mang lại trải nghiệm người dùng tuyệt vời.

- **Tính linh hoạt:** NoSQL Database thường cung cấp một kiến trúc linh hoạt để đạt được tốc độ phát triển và lặp (iteration) nhanh hơn, nhiều hơn. Data Model linh hoạt khiến NoSQL Database trở thành lựa chọn lý tưởng cho dữ liệu bán cấu trúc (semi-structured) và phi cấu trúc (unstructured).
- **Khả năng mở rộng (Scalability):** NoSQL Database thường được thiết kế để mở rộng hàng ngang (Scale Out) thông qua việc sử dụng cluster phần cứng phân tán, thay vì mở rộng theo chiều dọc (Scale Up) bằng cách thêm các máy chủ đắt đỏ và mạnh mẽ.
- **Hiệu năng cao:** NoSQL Database được tối ưu hóa cho các Data Model và Access Pattern (mô hình truy cập) cụ thể, điều này giúp đạt được hiệu năng cao hơn so meo với việc cố gắng sử dụng Relational Database để hoàn thành tính năng tương tự.
- **Tính năng mạnh mẽ:** NoSQL Database cung cấp các API và Data Type mạnh mẽ, được xây dựng chuyên biệt cho các Data Model tương ứng của chúng.

## NoSQL Database có những loại nào?

NoSQL Database chủ yếu có thể chia thành 4 loại dưới đây:

- **Key-Value**: Key-Value Database là một loại database tương đối đơn giản, trong đó mỗi mục đều bao gồm Key và Value. Đây là loại NoSQL Database cực kỳ linh hoạt, vì application có thể kiểm soát hoàn toàn nội dung được lưu trữ trong field value mà không có bất kỳ hạn chế nào. Redis và DynamoDB là hai Key-Value Database rất phổ biến.
- **Document**: Dữ liệu trong Document Database được lưu trữ trong các tài liệu (document) tương tự như đối tượng JSON (JavaScript Object Notation), rất rõ ràng và trực quan. Mỗi document chứa các cặp field và value. Các value này thường có thể là nhiều loại khác nhau, bao gồm string, number, boolean, array hoặc object, v.v., và cấu trúc của chúng thường đồng nhất với đối tượng mà developer sử dụng trong code. MongoDB là một Document Database rất phổ biến.
- **Graph**: Graph Database nhằm mục đích xây dựng và vận hành dễ dàng các ứng dụng hoạt động với các tập dữ liệu có độ kết nối cao. Các case study điển hình của Graph Database bao gồm mạng xã hội, Recommendation Engine, Fraud Detection và Knowledge Graph. Neo4j và Giraph là hai Graph Database rất phổ biến.
- **Wide-Column**: Wide-Column Database rất thích hợp cho việc lưu trữ lượng dữ liệu cực kỳ lớn. Cassandra và HBase là hai Wide-Column Database rất phổ biến.

Hình ảnh dưới đây trích từ [Tài liệu chính thức của Microsoft | Relational Data vs NoSQL Data](https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data).

![NoSQL Data Model](https://oss.javaguide.cn/github/javaguide/database/mongodb/types-of-nosql-datastores.png)

## Tham khảo

- NoSQL 是什么？- MongoDB 官方文档：<https://www.mongodb.com/zh-cn/nosql-explained>
- 什么是 NoSQL? - AWS：<https://aws.amazon.com/cn/nosql/>
- NoSQL vs. SQL Databases - MongoDB 官方文档：<https://www.mongodb.com/zh-cn/nosql-explained/nosql-vs-sql>

<!-- @include: @article-footer.snippet.md -->
