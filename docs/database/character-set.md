---
title: 字符集详解：字符集是什么？怎么用？
description: 详解字符集与字符编码原理，深入分析ASCII、GB2312、GBK、UTF-8、UTF-16等常见编码，解释MySQL中utf8与utf8mb4的区别以及emoji存储问题的解决方案。
category: 数据库
tag:
  - 数据库基础
head:
  - - meta
    - name: keywords
      content: 字符集,字符编码,UTF-8,UTF-16,GBK,GB2312,utf8mb4,ASCII,Unicode,MySQL字符集,emoji存储
---

Trong tập hợp bộ mã hóa ký tự (Character Encoding) của MySQL có 2 cách thực thi mã hóa UTF-8: **`utf8`** và **`utf8mb4`**.

Nếu sử dụng **`utf8`**, việc lưu trữ ký tự emoji và một số chữ Hán phức tạp, chữ Phồn thể sẽ bị lỗi.

Tại sao lại như vậy? Bài viết này sẽ giải đáp cho bạn từ gốc rễ.

## Character Set là gì?

Ký tự (Character) là tên gọi chung cho các loại chữ viết và ký hiệu, bao gồm chữ viết của các quốc gia, dấu câu, emoji, con số, v.v. **Character Set** (tập ký tự) chính là một tập hợp các ký tự. Có nhiều loại Character Set khác nhau, phạm vi ký tự mà mỗi Character Set có thể biểu diễn thường khác nhau, chẳng hạn như có một số Character Set không thể biểu diễn chữ Hán.

**Máy tính chỉ có thể lưu trữ dữ liệu nhị phân (binary), vậy các ký tự như tiếng Anh, chữ Hán, emoji... nên được lưu trữ như thế nào?**

Chúng ta phải ánh xạ tương ứng 1:1 giữa các ký tự này với dữ liệu nhị phân. Chẳng hạn như ký tự "a" tương ứng với "01100001", ngược lại "01100001" tương ứng với "a". Quá trình ánh xạ từ ký tự sang dữ liệu nhị phân được gọi là "**Character Encoding**" (Mã hóa ký tự), ngược lại quá trình giải mã dữ liệu nhị phân thành ký tự được gọi là "**Character Decoding**" (Giải mã ký tự).

## Character Encoding là gì?

Character Encoding là một phương pháp chuyển đổi qua lại giữa các ký tự trong Character Set với dữ liệu nhị phân trong máy tính, có thể xem như một quy tắc ánh xạ (mapping rule). Nói cách khác, mục đích của Character Encoding là để máy tính có thể lưu trữ và truyền tải các thông tin chữ viết khác nhau.

Mỗi Character Set đều có quy tắc mã hóa ký tự riêng của nó, các quy tắc mã hóa Character Set thường dùng bao gồm mã hóa ASCII, GB2312, GBK, GB18030, Big5, UTF-8, UTF-16, v.v.

## Có những Character Set phổ biến nào?

Các Character Set phổ biến gồm: ASCII, GB2312, GB18030, GBK, Unicode, ...

Điểm khác biệt chính giữa các Character Set khác nhau nằm ở:

- Phạm vi ký tự có thể biểu diễn
- Phương thức mã hóa

### ASCII

**ASCII** (**A**merican **S**tandard **C**ode for **I**nformation **I**nterchange - Mã chuẩn Mỹ dùng cho trao đổi thông tin) là một bộ Character Set chủ yếu dùng cho tiếng Anh Mỹ hiện đại (đây cũng chính là hạn chế của ASCII Character Set).

**Tại sao ASCII Character Set lại không tính đến tiếng Trung hay các ký tự khác?** Vì máy tính do người Mỹ phát minh, vào thời điểm đó, sự phát triển của máy tính vẫn đang ở thời kỳ phôi thai, chưa được sử dụng quy mô lớn ở các quốc gia khác. Do đó, khi Mỹ ban hành ASCII Character Set đã không xem xét đến việc tương thích với ngôn ngữ của các quốc gia khác.

Cho đến nay, ASCII Character Set định nghĩa tổng cộng 128 ký tự, trong đó có 33 ký tự điều khiển (như xuống dòng, xóa) không thể hiển thị.

Độ dài của một mã ASCII là 1 byte, tức là 8 bit. Ví dụ mã ASCII tương ứng với "a" là "01100001". Tuy nhiên, bit cao nhất là 0 chỉ được dùng làm bit kiểm tra (parity bit), 7 bit còn lại sử dụng sự kết hợp giữa 0 và 1, do đó ASCII Character Set có thể định nghĩa 128 ($2^7$) ký tự.

Do các ký tự có thể biểu diễn bởi mã ASCII quá ít, sau đó người ta đã mở rộng nó để có được **Extended ASCII Character Set**. Extended ASCII Character Set sử dụng 8 bit để biểu diễn 1 ký tự, do đó nó có thể định nghĩa 256 ($2^8$) ký tự.

![Mã hóa ký tự ASCII](https://oss.javaguide.cn/github/javaguide/csdn/c1c6375d08ca268690cef2b13591a5b4.png)

### GB2312

Như chúng ta đã nói ở trên, ASCII Character Set là tập ký tự dành riêng cho tiếng Anh Mỹ. Do đó, nhiều quốc gia đã tự xây dựng bộ Character Set phù hợp với ngôn ngữ của nước mình.

GB2312 Character Set là bộ tập ký tự thân thiện với chữ Hán, thu thập hơn 6.700 chữ Hán, về cơ bản bao phủ hầu hết các chữ Hán thường dùng. Tuy nhiên, GB2312 Character Set không hỗ trợ phần lớn các từ hiếm và chữ Phồn thể.

Đối với ký tự tiếng Anh, mã hóa GB2312 giống hệt mã ASCII, chỉ cần 1 byte. Đối với ký tự không phải tiếng Anh, cần mã hóa 2 byte.

### GBK

GBK Character Set có thể xem là phiên bản mở rộng của GB2312 Character Set, tương thích với GB2312, thu thập hơn 20.000 chữ Hán.

Chữ K trong GBK là chữ cái đầu tiên của từ "Kuo" trong pinyin Kuo Zhan (Mở rộng).

### GB18030

GB18030 hoàn toàn tương thích với GB2312 và GBK Character Set, đưa vào cả chữ viết của các dân tộc thiểu số tại Trung Quốc, đồng thời thu thập cả chữ Hán của Nhật Bản và Hàn Quốc. Đây là bộ Character Set chữ Hán toàn diện nhất cho đến nay, thu thập hơn 70.000 chữ Hán.

### BIG5

BIG5 chủ yếu dành cho tiếng Trung Phồn thể, thu thập hơn 13.000 chữ Hán.

### Unicode & UTF-8

Để phù hợp hơn với ngôn ngữ của từng quốc gia, rất nhiều Character Set đã ra đời.

Như chúng ta cũng đã nói ở trên, phạm vi ký tự và quy tắc mã hóa giữa các Character Set khác nhau có sự khác biệt. Điều này dẫn đến một vấn đề vô cùng nghiêm trọng: **Sử dụng phương thức mã hóa sai để xem một file chứa ký tự sẽ gây ra hiện tượng lỗi font (garbled text / rác chữ).**

Chẳng hạn nếu bạn dùng phương thức mã hóa UTF-8 để mở một file định dạng mã hóa GB2312 thì sẽ bị rác chữ. Ví dụ: Chữ Hán "牛" sau khi mã hóa GB2312 có giá trị Hex (thập lục phân) là "C5A3", nhưng khi giải mã "C5A3" bằng UTF-8 thì kết quả thu được lại là "ţ".

Bạn có thể tiến hành encode và decode online thông qua trang web này: <https://www.haomeili.net/HanZi/ZiFuBianMaZhuanHuan>

![](https://oss.javaguide.cn/github/javaguide/csdn/836c49b117ee4408871b0020b74c991d.png)

Như vậy chúng ta đã hiểu được bản chất của lỗi font (rác chữ): **Sử dụng các Character Set khác nhau hoặc không tương thích khi Encode và Decode**.

![](https://oss.javaguide.cn/javaguide/a8808cbabeea49caa3af27d314fa3c02-1.jpg)

Để giải quyết vấn đề này, mọi người đã nghĩ: "Giá như có một Character Set đưa tất cả ký tự trên thế giới vào trong đó thì tốt biết mấy!".

Sau đó, **Unicode** ra đời mang theo sứ mệnh này.

Unicode Character Set chứa hầu hết tất cả các ký tự đã biết trên thế giới. Tuy nhiên, Unicode Character Set không quy định cách lưu trữ các ký tự này như thế nào (tức là cách biểu diễn các ký tự này bằng dữ liệu nhị phân).

Tiếp theo, **UTF-8** (**8**-bit **U**nicode **T**ransformation **F**ormat) ra đời. Tương tự còn có UTF-16, UTF-32.

UTF-8 sử dụng từ 1 đến 4 byte để mã hóa từng ký tự, UTF-16 sử dụng 2 hoặc 4 byte để mã hóa từng ký tự, UTF-32 cố định 4 byte để mã hóa từng ký tự.

UTF-8 có thể tự động chọn độ dài mã hóa theo các ký hiệu khác nhau, chẳng hạn như ký tự tiếng Anh chỉ cần 1 byte là đủ, điểm này giống hệt ASCII Character Set. Do đó, đối với ký tự tiếng Anh, mã hóa UTF-8 và mã ASCII là giống nhau.

Quy tắc của UTF-32 là đơn giản nhất, tuy nhiên nhược điểm cũng khá rõ ràng: đối với các ký tự như chữ cái tiếng Anh, dung lượng tiêu tốn gấp 4 lần so với UTF-8.

**UTF-8** hiện là bộ mã hóa ký tự được sử dụng rộng rãi nhất.

![](https://oss.javaguide.cn/javaguide/1280px-Utf8webgrowth.svg.png)

## MySQL Character Set

MySQL hỗ trợ rất nhiều loại Character Set, ví dụ như GB2312, GBK, BIG5, nhiều loại Unicode Character Set (mã hóa UTF-8, mã hóa UTF-16, mã hóa UCS-2, mã hóa UTF-32, v.v.).

### Xem các Character Set được hỗ trợ

Bạn có thể xem thông qua câu lệnh `SHOW CHARSET`, hỗ trợ các mệnh đề LIKE và WHERE.

![](https://oss.javaguide.cn/javaguide/image-20211008164229671.png)

### Character Set mặc định

Trong MySQL 5.7, Character Set mặc định là `latin1`; trong MySQL 8.0, Character Set mặc định là `utf8mb4`.

### Các cấp độ của Character Set

Character Set trong MySQL có các cấp độ sau:

- `server` (cấp độ MySQL Instance)
- `database` (cấp độ Database)
- `table` (cấp độ Bảng)
- `column` (cấp độ Column / Trường)

Mức độ ưu tiên của chúng có thể hiểu đơn giản là tăng dần từ trên xuống dưới, tức là ưu tiên của `column` sẽ lớn hơn `table` và các cấp độ còn lại. Nếu chỉ định Character Set cấp MySQL Instance là `utf8mb4`, nhưng chỉ định Character Set của một bảng là `latin1`, thì tất cả các field của bảng này nếu không chỉ định riêng sẽ có mã hóa là `latin1`.

#### server

Các phiên bản MySQL khác nhau có giá trị mặc định cho Character Set cấp `server` khác nhau. Trong MySQL 5.7, giá trị mặc định là `latin1`; trong MySQL 8.0, giá trị mặc định là `utf8mb4`.

Tất nhiên cũng có thể thiết lập Character Set cấp `server` bằng cách chỉ định `--character-set-server` khi khởi động `mysqld`.

```bash
mysqld
mysqld --character-set-server=utf8mb4
mysqld --character-set-server=utf8mb4 \
  --collation-server=utf8mb4_0900_ai_ci
```

Hoặc nếu bạn khởi động MySQL bằng cách build từ source code, bạn có thể chỉ định option trong lệnh `cmake`:

```sh
cmake . -DDEFAULT_CHARSET=latin1
或者
cmake . -DDEFAULT_CHARSET=latin1 \
  -DDEFAULT_COLLATION=latin1_german1_ci
```

Ngoài ra, bạn cũng có thể thay đổi giá trị của `character_set_server` trong lúc runtime để đạt được mục đích sửa đổi Character Set cấp `server`.

Character Set cấp `server` là cài đặt global của MySQL Server, nó không chỉ làm Character Set mặc định khi tạo hoặc sửa đổi database (nếu không chỉ định Character Set khác), mà còn ảnh hưởng đến Character Set của kết nối giữa Client và Server, chi tiết có thể xem thêm tại [MySQL Connector/J 8.0 - 6.7 Using Character Sets and Unicode](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-charsets.html).

#### database

Character Set cấp `database` là giá trị chúng ta chỉ định khi tạo hoặc sửa đổi database:

```sql
CREATE DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]

ALTER DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]
```

Như đã nói ở trên, nếu khi thực thi các câu lệnh trên mà không chỉ định Character Set, MySQL sẽ sử dụng Character Set cấp `server`.

Có thể xem Character Set của một database bằng cách sau:

```sql
USE db_name;
SELECT @@character_set_database, @@collation_database;
```

```sql
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = 'db_name';
```

#### table

Character Set cấp `table` được chỉ định khi tạo bảng và sửa đổi bảng:

```sql
CREATE TABLE tbl_name (column_list)
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]]

ALTER TABLE tbl_name
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]
```

Nếu không chỉ định Character Set khi tạo bảng và sửa đổi bảng, Character Set cấp `database` sẽ được sử dụng.

#### column

Character Set cấp `column` cũng được chỉ định khi tạo bảng và sửa đổi bảng, chỉ có điều nó được định nghĩa ngay trong column. Dưới đây là một ví dụ:

```sql
CREATE TABLE t1
(
    col1 VARCHAR(5)
      CHARACTER SET latin1
      COLLATE latin1_german1_ci
);
```

Nếu không chỉ định Character Set cấp column, Character Set cấp bảng sẽ được sử dụng.

### Connection Character Set (Character Set kết nối)

Ở trên đã đề cập đến các cấp độ của Character Set, chúng liên quan đến việc lưu trữ. Còn Connection Character Set liên quan đến việc giao tiếp với MySQL Server.

Connection Character Set liên quan chặt chẽ với các biến dưới đây:

- `character_set_client`: Mô tả các câu lệnh SQL mà Client gửi cho Server sử dụng Character Set nào.
- `character_set_connection`: Mô tả Server khi nhận được câu lệnh SQL sẽ sử dụng Character Set nào để phiên dịch.
- `character_set_results`: Mô tả kết quả Server trả về cho Client sử dụng Character Set nào.

Giá trị của chúng có thể truy vấn bằng các câu lệnh SQL dưới đây:

```sql
SELECT * FROM performance_schema.session_variables
WHERE VARIABLE_NAME IN (
'character_set_client', 'character_set_connection',
'character_set_results', 'collation_connection'
) ORDER BY VARIABLE_NAME;
```

```sql
SHOW SESSION VARIABLES LIKE 'character\_set\_%';
```

Nếu muốn sửa đổi giá trị của các biến đã nêu ở trên, có các cách sau:

1. Sửa file cấu hình

```properties
[mysql]
# Chỉ dành riêng cho chương trình MySQL Client
default-character-set=utf8mb4
```

2. Sử dụng câu lệnh SQL

```sql
set names utf8mb4
# 或者一个个进行修改
# SET character_set_client = utf8mb4;
# SET character_set_results = utf8mb4;
# SET collation_connection = utf8mb4;
```

### Ảnh hưởng của JDBC đối với Connection Character Set

Không biết các bạn đã từng gặp trường hợp lưu trữ biểu tượng emoji bình thường, nhưng khi dùng phần mềm như Navicat để query thì phát hiện biểu tượng emoji biến thành dấu hỏi (?) chưa. Vấn đề này rất có thể là do JDBC Driver gây ra.

Dựa vào nội dung phía trên, chúng ta biết Connection Character Set cũng ảnh hưởng đến dữ liệu chúng ta lưu trữ, mà JDBC Driver sẽ ảnh hưởng đến Connection Character Set.

`mysql-connector-java` (JDBC Driver) chủ yếu ảnh hưởng đến Connection Character Set thông qua các thuộc tính này:

- `characterEncoding`
- `characterSetResults`

Lấy `DataGrip 2023.1.2` làm ví dụ, trong hộp thoại nâng cao cấu hình Data Source của nó, có thể thấy giá trị mặc định của `characterSetResults` là `utf8`, khi sử dụng `mysql-connector-java 8.0.25`, Connection Character Set cuối cùng sẽ được thiết lập thành `utf8mb3`. Do đó trong trường hợp này, biểu tượng emoji sẽ hiển thị dưới dạng dấu hỏi, và phiên bản Driver hiện tại chưa hỗ trợ đặt `characterSetResults` thành `utf8mb4`, nhưng đổi sang `mysql-connector-java driver 8.0.29` thì lại cho phép.

Cụ thể có thể tham khảo câu trả lời trên StackOverflow: [DataGrip MySQL stores emojis correctly but displays them as?](https://stackoverflow.com/questions/54815419/datagrip-mysql-stores-emojis-correctly-but-displays-them-as).

### Sử dụng UTF-8

Thông thường, chúng tôi khuyến nghị sử dụng UTF-8 làm phương thức mã hóa ký tự mặc định.

Tuy nhiên, ở đây có một "cạm bẫy" nhỏ.

Trong tập hợp mã hóa ký tự MySQL có 2 bản thực thi mã hóa UTF-8:

- **`utf8`**: Mã hóa `utf8` chỉ hỗ trợ từ 1 đến 3 byte. Trong mã hóa `utf8`, chữ Trung Quốc chiếm 3 byte, các con số, tiếng Anh, ký hiệu khác chiếm 1 byte. Nhưng ký hiệu emoji chiếm 4 byte, một số chữ viết phức tạp, chữ Phồn thể cũng chiếm 4 byte.
- **`utf8mb4`**: Bản thực thi đầy đủ của UTF-8, hàng chính chủ! Hỗ trợ tối đa sử dụng 4 byte để biểu diễn ký tự, do đó có thể dùng để lưu trữ ký hiệu emoji.

**Tại sao lại có 2 bản thực thi mã hóa UTF-8?** Nguyên nhân như sau:

![](https://oss.javaguide.cn/javaguide/image-20211008164542347.png)

Do đó, nếu bạn cần lưu trữ dữ liệu loại `emoji` hoặc một số chữ viết phức tạp, chữ Phồn thể vào database MySQL, CHARSET của database nhất định phải chỉ định là `utf8mb4` chứ không phải `utf8`, nếu không khi lưu trữ sẽ bị báo lỗi.

Cùng demo thử nhé! (Môi trường: MySQL 5.7+)

Câu lệnh tạo bảng như sau, chúng ta chỉ định CHARSET của database là `utf8`:

```sql
CREATE TABLE `user` (
  `id` varchar(66) CHARACTER SET utf8mb3 NOT NULL,
  `name` varchar(33) CHARACTER SET utf8mb3 NOT NULL,
  `phone` varchar(33) CHARACTER SET utf8mb3 DEFAULT NULL,
  `password` varchar(100) CHARACTER SET utf8mb3 DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Khi chúng ta thực thi câu lệnh INSERT dưới đây để chèn dữ liệu vào database, quả nhiên bị báo lỗi!

```sql
INSERT INTO `user` (`id`, `name`, `phone`, `password`)
VALUES
 ('A00003', 'guide哥😘😘😘', '181631312312', '123456');

```

Thông tin lỗi như sau:

```plain
Incorrect string value: '\xF0\x9F\x98\x98\xF0\x9F...' for column 'name' at row 1
```

## Tham khảo

- 字符集和字符编码（Charset & Encoding）：<https://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html>
- 十分钟搞清字符集和字符编码：<http://cenalulu.github.io/linux/character-encoding/>
- Unicode-维基百科：<https://zh.wikipedia.org/wiki/Unicode>
- GB2312-维基百科：<https://zh.wikipedia.org/wiki/GB_2312>
- UTF-8-维基百科：<https://zh.wikipedia.org/wiki/UTF-8>
- GB18030-维基百科: <https://zh.wikipedia.org/wiki/GB_18030>
- MySQL8 文档：<https://dev.mysql.com/doc/refman/8.0/en/charset.html>
- MySQL5.7 文档：<https://dev.mysql.com/doc/refman/5.7/en/charset.html>
- MySQL Connector/J 文档：<https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-charsets.html>

<!-- @include: @article-footer.snippet.md -->
