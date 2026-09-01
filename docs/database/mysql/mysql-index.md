---
title: MySQL索引详解
description: MySQL索引详解，深入剖析B+树索引结构、聚簇索引与二级索引的区别、联合索引与最左前缀原则、覆盖索引与索引下推优化，以及常见的索引失效场景。
category: 数据库
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: MySQL索引,B+树索引,聚簇索引,覆盖索引,联合索引,索引下推,回表查询,索引失效,最左前缀原则
---

> Cảm ơn [WT-AHA](https://github.com/WT-AHA) đã hoàn thiện bài viết này, PR liên quan: <https://github.com/Snailclimb/JavaGuide/pull/1648> .

Bất kỳ ai đã trải qua vài buổi phỏng vấn đều hiểu rõ rằng, điểm kiến thức Index cơ sở dữ liệu xuất hiện với tần suất cao đến mức vô lý trong phỏng vấn.

Ngoài việc rất quan trọng đối với chuẩn bị phỏng vấn, việc tận dụng tốt Index mang lại hiệu năng SQL cải thiện rất rõ rệt, là một phương tiện tối ưu hóa SQL có tỷ lệ chi phí/hiệu quả (P/P) rất cao.

## Giới thiệu về Index

**Index là một cấu trúc dữ liệu dùng để truy vấn và tìm kiếm dữ liệu nhanh chóng, bản chất có thể xem là một cấu trúc dữ liệu đã được sắp xếp.**

Tác dụng của Index tương tự như mục lục của một cuốn sách. Ví dụ: khi tra từ điển, nếu không có mục lục, ta chỉ có thể lật từng trang một để tìm từ cần tra, tốc độ rất chậm; nếu có mục lục, ta chỉ cần vào mục lục tìm vị trí của từ rồi lật trực tiếp đến trang đó.

Cấu trúc dữ liệu bên dưới của Index có nhiều loại, phổ biến như: B-Tree, B+Tree, Hash, Red-Black Tree. Trong MySQL, dù là InnoDB hay MyISAM đều sử dụng B+Tree làm cấu trúc dữ liệu Index.

## Ưu nhược điểm của Index

**Ưu điểm của Index:**

1. **Tốc độ truy vấn tăng vọt (mục đích chính)**: Thông qua Index, database có thể **giảm đáng kể lượng dữ liệu cần quét**, định vị trực tiếp đến bản ghi thỏa mãn điều kiện, từ đó đẩy nhanh tốc độ truy xuất dữ liệu, giảm số lần I/O đĩa.
2. **Đảm bảo tính duy nhất của dữ liệu**: Thông qua việc tạo **Unique Index**, có thể đảm bảo giá trị của một cột (hoặc tổ hợp nhiều cột) trong bảng là duy nhất, như ID người dùng, email... Primary Key bản thân nó cũng là một Unique Index.
3. **Đẩy nhanh sắp xếp và nhóm**: Nếu cột liên quan trong mệnh đề ORDER BY hoặc GROUP BY của query có tạo Index, database thường có thể tận dụng trực tiếp đặc tính đã sắp xếp của Index, tránh thao tác sắp xếp bổ sung, từ đó nâng cao hiệu năng.

**Nhược điểm của Index:**

1. **Tốn thời gian tạo và bảo trì**: Tạo Index bản thân nó cần thời gian, đặc biệt là thao tác trên bảng lớn. Quan trọng hơn, khi thực hiện các thao tác **thêm, xóa, sửa (thao tác DML)** dữ liệu trong bảng, không những phải thao tác bản thân dữ liệu mà Index liên quan cũng phải cập nhật và bảo trì động, việc này sẽ **làm giảm hiệu suất thực thi của các thao tác DML này**.
2. **Chiếm dụng không gian lưu trữ**: Index bản chất cũng là cấu trúc dữ liệu, cần lưu trữ dưới dạng file vật lý (hoặc cấu trúc bộ nhớ), do đó sẽ **chiếm thêm một khoảng dung lượng đĩa**. Index càng nhiều, càng lớn thì dung lượng chiếm dụng càng nhiều.
3. **Có thể bị dùng sai hoặc vô hiệu hóa**: Nếu thiết kế Index không hợp lý, hoặc viết câu lệnh query không tốt, Optimizer của database có thể sẽ không chọn sử dụng Index (hoặc chọn sai Index), dẫn đến hiệu năng bị giảm sút.

**Vậy dùng Index có chắc chắn nâng cao hiệu năng truy vấn không?**

**Không nhất thiết.** Hầu hết trường hợp, dùng Index hợp lý quả thực nhanh hơn nhiều so với Full Table Scan (quét toàn bảng). Nhưng cũng có ngoại lệ:

- **Lượng dữ liệu quá nhỏ**: Nếu dữ liệu trong bảng rất ít (ví dụ chỉ vài trăm hàng), Full Table Scan có thể nhanh hơn việc tìm kiếm qua Index, vì bản thân việc đi qua Index cũng có overhead.
- **Tỷ lệ kết quả query quá lớn**: Nếu dữ liệu cần query chiếm phần lớn cả bảng (ví dụ trên 20%-30%), Optimizer có thể cho rằng Full Table Scan kinh tế hơn, vì chi phí Index Lookup (回表) nhiều lần (I/O ngẫu nhiên) có thể cao hơn một lần Full Table Scan tuần tự.
- **Bảo trì Index không tốt hoặc thông tin thống kê bị lỗi thời**: Dẫn đến Optimizer đưa ra phán đoán sai.

## Lựa chọn cấu trúc dữ liệu bên dưới cho Index

### Hash Table (Bảng băm)

Hash Table là tập hợp các cặp key-value, thông qua key có thể nhanh chóng lấy ra value tương ứng, do đó Hash Table có thể truy xuất dữ liệu cực nhanh (tiệm cận O(1)).

**Tại sao có thể thông qua key lấy nhanh value?** Nguyên nhân nằm ở **thuật toán Hash** (hay thuật toán băm). Thông qua thuật toán Hash, chúng ta có thể nhanh chóng tìm thấy index tương ứng với key, tìm được index nghĩa là tìm được value tương ứng.

```java
hash = hashfunc(key)
index = hash % array_size
```

![](https://oss.javaguide.cn/github/javaguide/database/mysql20210513092328171.png)

Tuy nhiên! Thuật toán Hash có vấn đề **xung đột Hash (Hash Collision)**, tức là nhiều key khác nhau cuối cùng tạo ra cùng một index. Thông thường, giải pháp phổ biến là **Chaining (phương pháp chuỗi / 链地址法)**. Chaining là lưu trữ các dữ liệu xung đột Hash vào một linked list. Giống như `HashMap` trước JDK 1.8 giải quyết xung đột Hash bằng Chaining. Từ JDK 1.8 trở đi, `HashMap` đưa vào Red-Black Tree để nâng cao hiệu suất tìm kiếm khi linked list quá dài.

![](https://oss.javaguide.cn/github/javaguide/database/mysql20210513092224836.png)

Để giảm thiểu xung đột Hash, một hàm Hash tốt nên phân bố dữ liệu "đồng đều" trên toàn bộ tập hợp giá trị Hash có thể có.

Storage Engine InnoDB của MySQL không trực tiếp hỗ trợ Hash Index thông thường, tuy nhiên trong InnoDB tồn tại một "Adaptive Hash Index" (Chỉ mục Hash tự thích ứng) đặc biệt. Adaptive Hash Index không phải Hash Index thuần túy theo nghĩa truyền thống mà kết hợp đặc điểm của B+Tree và Hash Index để thích ứng tốt hơn với pattern truy cập dữ liệu và nhu cầu hiệu năng trong thực tế. Mỗi hash bucket của Adaptive Hash Index thực chất là một cấu trúc B+Tree nhỏ. Cấu trúc B+Tree này có thể lưu trữ nhiều cặp key-value chứ không chỉ một key. Điều này giúp giảm độ dài chuỗi xung đột Hash, nâng cao hiệu quả Index. Chi tiết bài viết: [MySQL các loại Buffer: Adaptive Hash Index](https://mp.weixin.qq.com/s/ra4v1XR5pzSWc-qtGO-dBg).

Đã vậy Hash Table nhanh như thế, **tại sao MySQL không dùng nó làm cấu trúc dữ liệu Index?** Chủ yếu vì Hash Index không hỗ trợ truy vấn thứ tự và truy vấn phạm vi (Range Query). Nếu chúng ta muốn sắp xếp dữ liệu hoặc truy vấn phạm vi thì Hash Index chịu bó tay. Hơn nữa, mỗi lần I/O chỉ lấy được 1 giá trị.

Thử tưởng tượng kịch bản:

```java
SELECT * FROM tb1 WHERE id < 500;
```

Trong truy vấn phạm vi này, B+Tree có ưu thế cực lớn, chỉ cần duyệt các node lá nhỏ hơn 500. Còn Hash Index định vị dựa theo thuật toán hash, lẽ nào phải tính hash từng giá trị từ 1 đến 499 để định vị? Đây là nhược điểm lớn nhất của Hash.

### Binary Search Tree (BST - Cây tìm kiếm nhị phân)

BST là cấu trúc dữ liệu dựa trên cây nhị phân, có đặc điểm:

1. Giá trị tất cả node ở cây con bên trái đều nhỏ hơn node gốc.
2. Giá trị tất cả node ở cây con bên phải đều lớn hơn node gốc.
3. Cây con trái và phải cũng lần lượt là cây tìm kiếm nhị phân.

Khi BST cân bằng (độ sâu cây con trái và phải chênh lệch không quá 1), độ phức tạp thời gian query là O(log2(N)), hiệu suất khá cao. Tuy nhiên khi BST không cân bằng, ví dụ trường hợp xấu nhất (chèn node đã sắp xếp), cây thoái hóa thành linked list tuyến tính (斜树), độ phức tạp thời gian thoái hóa thành O(N).

![斜树](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/oblique-tree.png)

Nghĩa là **hiệu năng BST cực kỳ phụ thuộc vào độ cân bằng của nó, dẫn đến nó không phù hợp làm cấu trúc dữ liệu Index bên dưới của MySQL.**

Để giải quyết vấn đề này và nâng cao hiệu quả truy vấn, người ta đã phát minh nhiều cấu trúc dữ liệu cải tiến trên nền BST như AVL Tree, B-Tree, B+Tree...

### AVL Tree (Cây AVL)

AVL Tree là cây tìm kiếm nhị phân tự cân bằng sớm nhất trong khoa học máy tính, tên gọi lấy từ chữ cái viết tắt của các nhà phát minh G.M. Adelson-Velsky và E.M. Landis. Đặc điểm của AVL Tree là đảm bảo chênh lệch chiều cao cây con trái và phải của bất kỳ node nào không quá 1, nên còn gọi là cây nhị phân cân bằng chiều cao. Thao tác tìm kiếm, chèn và xóa ở trường hợp trung bình và xấu nhất đều có độ phức tạp thời gian O(logn).

![](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/avl-tree.png)

AVL Tree sử dụng thao tác xoay (rotation: LL, RR, LR, RL) để duy trì cân bằng. Do cần xoay thường xuyên để duy trì cân bằng nên overhead tính toán lớn, làm giảm hiệu năng ghi của database. Hơn nữa, mỗi node AVL Tree chỉ lưu 1 dữ liệu, mỗi lần I/O đĩa chỉ đọc 1 node. Nếu dữ liệu phân bố ở nhiều node thì cần nhiều lần I/O đĩa. **I/O đĩa là thao tác tốn thời gian, khi thiết kế database Index ta cần ưu tiên giảm thiểu số lần I/O đĩa.**

Trong thực tế ứng dụng, AVL Tree không được dùng nhiều.

### Red-Black Tree (Cây đỏ đen)

Red-Black Tree là cây tìm kiếm nhị phân tự cân bằng, duy trì cân bằng thông qua đổi màu và xoay node khi chèn/xóa, có các đặc điểm:

1. Mỗi node hoặc là màu đỏ hoặc màu đen;
2. Node gốc luôn là màu đen;
3. Mỗi node lá đều là node rỗng màu đen (NIL node);
4. Nếu node là màu đỏ thì các node con của nó bắt buộc phải là màu đen;
5. Từ một node bất kỳ đến các node lá hoặc node con rỗng của nó, mọi đường đi phải chứa số lượng node đen bằng nhau (tức cùng chiều cao đen).

![红黑树](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree.png)

Không giống AVL Tree đòi hỏi cân bằng tuyệt đối, Red-Black Tree chỉ duy trì cân bằng tương đối. Do đó hiệu năng truy vấn của Red-Black Tree giảm nhẹ một chút vì chiều cao cây có thể cao hơn, dẫn đến cần nhiều lần I/O đĩa hơn để tìm dữ liệu — đây là lý do chính MySQL không chọn Red-Black Tree. Nhưng đổi lại, hiệu suất thao tác chèn và xóa được nâng cao rất nhiều vì Red-Black Tree chỉ cần O(1) lần xoay và đổi màu khi chèn/xóa để duy trì cân bằng.

**Red-Black Tree được ứng dụng khá rộng rãi trong bộ nhớ (RAM), như TreeMap, TreeSet và HashMap JDK 1.8.**

Về so sánh cơ bản giữa BST, AVL Tree, Red-Black Tree, B-Tree và B+Tree, có thể tham khảo thêm [Chi tiết cấu trúc cây](../../cs-basics/data-structure/tree.md) và [Chi tiết cây đỏ đen](../../cs-basics/data-structure/red-black-tree.md).

### B-Tree & B+Tree

B-Tree (cây tìm kiếm cân bằng đa đường) và B+Tree (biến thể B-Tree). Chữ B đại diện cho `Balanced`.

Hầu hết các hệ quản trị database và file system hiện nay đều sử dụng B-Tree hoặc B+Tree làm cấu trúc dữ liệu Index.

**Điểm giống và khác nhau giữa B-Tree và B+Tree:**

- B-Tree: tất cả các node đều lưu cả key và data. B+Tree: chỉ node lá mới lưu key và data, node trong (internal node) chỉ lưu key.
- B-Tree: các node lá độc lập. B+Tree: các node lá có con trỏ linked list nối các node lá kề nhau.
- B-Tree: quy trình tìm kiếm tương đương tìm kiếm nhị phân trên từng node, có thể kết thúc trước khi đến node lá. B+Tree: độ dài đường đi truy vấn cố định từ gốc đến lá, hiệu năng tìm kiếm rất ổn định.
- Truy vấn phạm vi: B-Tree cần duyệt trung thứ tự (in-order traversal) trên cây; B+Tree chỉ cần duyệt linked list ở node lá.

Tóm lại, B+Tree có số lần I/O đĩa ít hơn, hiệu năng tìm kiếm ổn định hơn và thích hợp hơn cho truy vấn phạm vi.

Trong MySQL, cả Storage Engine MyISAM và InnoDB đều sử dụng B+Tree làm cấu trúc dữ liệu Index, nhưng cách thực thi khác nhau:

> Trong MyISAM, vùng data ở node lá B+Tree lưu địa chỉ của bản ghi dữ liệu (Non-clustered Index / 非聚簇索引).
>
> Trong InnoDB, bản thân file dữ liệu chính là cấu trúc Index B+Tree. Node lá vùng data lưu trữ toàn bộ bản ghi dữ liệu hoàn chỉnh (Clustered Index / 聚簇索引). Key của Index này là Primary Key. Các Index khác là Secondary Index (辅助索引), vùng data lưu giá trị Primary Key chứ không lưu địa chỉ.

## Tổng kết các loại Index

Theo cấu trúc dữ liệu:

- BTree Index: Mặc định và phổ biến nhất (thực tế dùng B+Tree).
- Hash Index: Định vị 1 lần qua cặp key-value.
- RTree Index: Dùng cho kiểu dữ liệu spatial/geometry.
- Full-text Index: Phân tích từ văn bản (Search Engine như ElasticSearch thường thay thế).

Theo cách lưu trữ bên dưới:

- Clustered Index (聚簇索引 / 聚集索引): Cấu trúc Index và dữ liệu lưu cùng nhau (Primary Key Index của InnoDB).
- Non-Clustered Index (非聚簇索引 / 非聚集索引): Cấu trúc Index và dữ liệu tách rời nhau (Secondary Index, MyISAM).

Theo ứng dụng:

- Primary Key Index: Tăng tốc + duy nhất + không NULL + chỉ có 1 trên bảng.
- Normal Index (普通索引): Chỉ tăng tốc truy vấn.
- Unique Index (唯一索引): Tăng tốc + duy nhất (cho phép NULL).
- Covering Index (覆盖索引): Index chứa toàn bộ field cần query.
- Composite Index (联合索引): Index nhiều cột.
- Full-text Index (全文索引): Tìm kiếm văn bản.
- Prefix Index (前缀索引): Tạo index trên tiền tố chuỗi.

Tính năng mới trong MySQL 8.x:

- Invisible Index (隐藏索引): Optimizer không dùng nhưng vẫn bảo trì.
- Descending Index (降序索引): Thật sự hỗ trợ sắp xếp giảm dần.
- Function-based Index (函数索引): Tạo index trên hàm/biểu thức (MySQL 8.0.13+).

## Primary Key Index

Bảng dữ liệu sử dụng cột Primary Key làm Primary Key Index.

Một bảng chỉ có duy nhất một Primary Key, không được null, không được trùng lặp.

Trong InnoDB, nếu không khai báo Primary Key, MySQL sẽ kiểm tra cột Unique Index đầu tiên không null để làm Primary Key, nếu không có sẽ tự động tạo自增主键 6 byte ẩn.

![Primary Key Index](https://oss.javaguide.cn/github/javaguide/open-source-project/cluster-index.png)

## Secondary Index (Chỉ mục thứ cấp)

Secondary Index (Chỉ mục phụ / chỉ mục không phải Primary Key) lưu giá trị Primary Key ở node lá.

Unique Index, Normal Index, Prefix Index... đều thuộc Secondary Index.

1. **Unique Index (Unique Key)**: Ràng buộc duy nhất, cho phép NULL, một bảng có thể có nhiều Unique Index.
2. **Normal Index (Index)**: Tăng tốc truy vấn dữ liệu.
3. **Prefix Index (Prefix)**: Chỉ áp dụng cho kiểu chuỗi, tạo index trên tiền tố ký tự để giảm dung lượng.
4. **Full-text Index (Full Text)**: Tìm kiếm từ khóa trong dữ liệu văn bản lớn.

![Secondary Index](https://oss.javaguide.cn/github/javaguide/open-source-project/no-cluster-index.png)

## Clustered Index & Non-Clustered Index

### Clustered Index (聚簇索引)

#### Giới thiệu
Clustered Index là cấu trúc Index và dữ liệu được lưu cùng nhau. Primary Key Index trong InnoDB thuộc Clustered Index.

File `.ibd` của InnoDB chứa cả Index và dữ liệu của bảng. Node lá của B+Tree lưu trữ cả Index và toàn bộ dữ liệu tương ứng.

#### Ưu nhược điểm
**Ưu điểm**:
- **Tốc độ truy vấn cực nhanh**: Ít hơn 1 lần I/O đĩa so với Non-Clustered Index.
- **Tối ưu cho sắp xếp và truy vấn phạm vi**: Tốc độ sắp xếp và truy vấn phạm vi trên Primary Key rất nhanh.

**Nhược điểm**:
- **Phụ thuộc dữ liệu có thứ tự**: Cần sắp xếp khi chèn dữ liệu.
- **Chi phí update lớn**: Khi sửa đổi cột Index, chi phí chỉnh sửa node lá rất cao (Primary Key nên là immutable).

### Non-Clustered Index (非聚簇索引)

#### Giới thiệu
Non-Clustered Index là cấu trúc Index và dữ liệu tách rời nhau. Secondary Index thuộc Non-Clustered Index. MyISAM dùng Non-Clustered Index cho cả Primary Key và Secondary Index.

#### Ưu nhược điểm
**Ưu điểm**: Chi phí update nhỏ hơn Clustered Index vì node lá không lưu dữ liệu.

**Nhược điểm**:
- Phụ thuộc dữ liệu có thứ tự.
- Có thể phải thực hiện Index Lookup (回表)二次.

![File trong MySQL](https://oss.javaguide.cn/github/javaguide/database/mysql20210420165311654.png)

![Clustered Index & Non-Clustered Index](https://oss.javaguide.cn/github/javaguide/database/mysql20210420165326946.png)

#### Non-Clustered Index có bắt buộc phải Index Lookup (回表) không?
**Không nhất thiết.** Nếu tất cả field cần SELECT đều nằm trong Index (Covering Index), database sẽ lấy dữ liệu trực tiếp từ Index mà không cần Index Lookup (回表).

```sql
SELECT name FROM table WHERE name='guang19';
```

## Covering Index & Composite Index

### Covering Index (索引覆盖 / 覆盖索引)

Nếu một Index chứa (hoặc bao phủ) tất cả các field cần query, ta gọi đó là **Covering Index**.

![Covering Index](https://oss.javaguide.cn/github/javaguide/database/mysql20210420165341868.png)

Thử nghiệm Covering Index với SQL:

1. Tạo bảng `cus_order`:

```sql
CREATE TABLE `cus_order` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `score` int(11) NOT NULL,
  `name` varchar(11) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=100000 DEFAULT CHARSET=utf8mb4;
```

2. Tạo Stored Procedure chèn 100w bản ghi:

```sql
DELIMITER ;;
CREATE DEFINER=`root`@`%` PROCEDURE `BatchinsertDataToCusOder`(IN start_num INT,IN max_num INT)
BEGIN
      DECLARE i INT default start_num;
      WHILE i < max_num DO
          insert into `cus_order`(`id`, `score`, `name`)
          values (i,RAND() * 1000000,CONCAT('user', i));
          SET i = i + 1;
      END WHILE;
  END;;
DELIMITER ;
```

Thực thi:

```sql
CALL BatchinsertDataToCusOder(1, 1000000);
```

3. Phân tích với EXPLAIN:

```sql
SELECT `score`,`name` FROM `cus_order` ORDER BY `score` DESC;
```

Nếu chưa tạo Index, cột `Extra` hiển thị `Using filesort`.

![](https://oss.javaguide.cn/github/javaguide/mysql/not-using-covering-index-demo.png)

Tạo Composite Index:

```sql
ALTER TABLE `cus_order` ADD INDEX id_score_name(score, name);
```

Phân tích lại với EXPLAIN, cột `Extra` hiển thị `Using index`, chứng tỏ đã dùng Covering Index thành công.

![](https://oss.javaguide.cn/github/javaguide/mysql/using-covering-index-demo.png)

### Composite Index (联合索引)

Sử dụng nhiều field tạo Index gọi là **Composite Index**.

```sql
ALTER TABLE `cus_order` ADD INDEX id_score_name(score, name);
```

### Quy tắc Tiền tố Trái nhất (Leftmost Prefix Rule)

MySQL khớp từ trái sang phải theo thứ tự các field trong Composite Index. Gặp so sánh phạm vi (`>`, `<`) sẽ dừng khớp.

Ví dụ Composite Index `(column1, column2, column3)` hỗ trợ các tiền tố `(column1)`, `(column1, column2)`, `(column1, column2, column3)`.

Nên đặt field có độ phân biệt (selectivity) cao ở ngoài cùng bên trái.

Demo:

```sql
CREATE TABLE `student` (
  `id` int NOT NULL,
  `name` varchar(100) DEFAULT NULL,
  `class` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `name_class_idx` (`name`,`class`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

![](https://oss.javaguide.cn/github/javaguide/database/mysql/leftmost-prefix-matching-rule.png)

```sql
# Trúng Index
SELECT * FROM student WHERE name = 'Anne Henry';
EXPLAIN SELECT * FROM student WHERE name = 'Anne Henry' AND class = 'lIrm08RYVk';
# Không trúng Index
SELECT * FROM student WHERE class = 'lIrm08RYVk';
```

## Index Condition Pushdown (ICP - 索引下推)

**Index Condition Pushdown (ICP)** là tính năng tối ưu Index giới thiệu từ **MySQL 5.6**, cho phép Storage Engine thực hiện một phần điều kiện `WHERE` trong quá trình duyệt Index để lọc trước các bản ghi không thỏa mãn, giảm số lần Index Lookup (回表).

Giả sử bảng `user` có Composite Index `(zipcode, birthdate)`:

```sql
CREATE TABLE `user` (
  `id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `zipcode` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `birthdate` date NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_zipcode_birthdate` (`zipcode`,`birthdate`) ) ENGINE=InnoDB AUTO_INCREMENT=1001 DEFAULT CHARSET=utf8mb4;

SELECT * FROM user WHERE zipcode = '431200' AND MONTH(birthdate) = 3;
```

- Không có ICP: Storage Engine tìm `zipcode = '431200'`, Index Lookup (回表) lấy toàn bộ dữ liệu lên Server layer, Server layer mới lọc `MONTH(birthdate) = 3`.
- Có ICP: Storage Engine lọc luôn `MONTH(birthdate) = 3` ngay khi quét Index `zipcode = '431200'`, rồi mới Index Lookup (回表) những bản ghi thỏa mãn.

![](https://oss.javaguide.cn/github/javaguide/database/mysql/index-condition-pushdown.png)

![](https://oss.javaguide.cn/github/javaguide/database/mysql/index-condition-pushdown-graphic-illustration.png)

Sơ đồ kiến trúc MySQL:

![](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

ICP "đẩy" bớt công việc của Server layer xuống Storage Engine layer.

Phạm vi áp dụng ICP:
1. Áp dụng cho InnoDB và MyISAM.
2. Áp dụng cho range, ref, eq_ref, ref_or_null.
3. Đối với InnoDB chỉ dùng cho Secondary Index.
4. Subquery không dùng ICP.
5. Stored Procedure không dùng ICP.

## Khuyến nghị sử dụng Index đúng cách

### Chọn field phù hợp để tạo Index
- Field không phải NULL.
- Field được truy vấn tần suất cao.
- Field làm điều kiện WHERE.
- Field thường xuyên cần ORDER BY.
- Field thường xuyên dùng cho JOIN.

### Tránh để Index bị vô hiệu hóa
1. **Vi phạm logic B+Tree**: Vi phạm tiền tố trái nhất, tính toán/hàm trên cột Index, chuyển đổi ngầm định (Implicit Conversion), LIKE bắt đầu bằng `%`.
2. **Quyết định chi phí của Optimizer**: `SELECT *` gây tốn chi phí回表, điều kiện `OR` không có Index, danh sách `IN` quá dài.

Chi tiết: [Tổng kết các kịch bản MySQL Index bị vô hiệu hóa](https://javaguide.cn/database/mysql/mysql-index-invalidation.html).

### Thận trọng với field update thường xuyên
Sửa đổi cột Index tốn chi phí bảo trì Index B+Tree.

### Giới hạn số Index trên mỗi bảng
Khuyến nghị mỗi bảng không quá 5 Index.

### Ưu tiên Composite Index thay vì nhiều Single-column Index
Tiết kiệm dung lượng đĩa và nâng cao hiệu quả bảo trì.

### Tránh Redundant Index (Index dư thừa)
Ví dụ `(name, city)` và `(name)` là dư thừa.

### Dùng Prefix Index cho kiểu String
Tiết kiệm không gian lưu trữ.

### Xóa Index lâu ngày không sử dụng
Sử dụng view `sys.schema_unused_indexes` trong MySQL 5.7+.

### Biết cách dùng EXPLAIN phân tích SQL
Dùng `EXPLAIN` kiểm tra Execution Plan để biết SQL có trúng Index hay không.

| **Tên cột** | **Ý nghĩa** |
| ------------- | -------------------------------------------- |
| id | ID chuỗi truy vấn SELECT |
| select_type | Loại truy vấn SELECT |
| table | Bảng được sử dụng |
| partitions | Partition khớp |
| type | Phương thức truy cập bảng (Access Type) |
| possible_keys | Index có thể sử dụng |
| key | Index thực tế sử dụng |
| key_len | Độ dài Index được chọn |
| ref | Cột hoặc hằng số so sánh với Index |
| rows | Số hàng dự kiến phải đọc |
| filtered | Tỷ lệ phần trăm bản ghi giữ lại sau lọc |
| Extra | Thông tin bổ sung |

Chi tiết: [Phân tích Execution Plan trong MySQL](./mysql-query-execution-plan.md).

## Đọc thêm về cấu trúc dữ liệu

- [Chi tiết cấu trúc cây](../../cs-basics/data-structure/tree.md)
- [Chi tiết cây đỏ đen](../../cs-basics/data-structure/red-black-tree.md)

<!-- @include: @article-footer.snippet.md -->
