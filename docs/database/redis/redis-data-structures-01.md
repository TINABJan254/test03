---
title: Redis 5 种基本数据类型详解
description: 详解Redis五种基本数据类型String、List、Set、Hash、Zset的使用方法和应用场景，深入分析SDS、跳表、压缩列表等底层数据结构实现原理。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis数据类型,String,List,Set,Hash,Zset,SDS,跳表,压缩列表,Redis命令
---

Redis có 5 Data Type cơ bản: String (Chuỗi), List (Danh sách), Set (Tập hợp), Hash (Bảng băm), Zset (Tập hợp có thứ tự).

5 Data Type này được cung cấp trực tiếp cho người dùng sử dụng. Tầng dưới của chúng phụ thuộc vào 8 loại Data Structure: SDS (Simple Dynamic String), LinkedList (Cấu trúc liên kết kép), Dict (Bảng băm/Từ điển), SkipList (Bảng nhảy), Intset (Tập hợp số nguyên), ZipList (Danh sách nén), QuickList (Danh sách nhanh).

Bảng mã hóa cấu trúc dữ liệu tầng dưới tương ứng:

| String | List                         | Hash          | Set          | Zset              |
| :----- | :--------------------------- | :------------ | :----------- | :---------------- |
| SDS    | LinkedList/ZipList/QuickList | Dict, ZipList | Dict, Intset | ZipList, SkipList |

Từ Redis 3.2, List dùng QuickList (kết hợp của LinkedList và ZipList). Từ Redis 7.0, ZipList được thay thế bằng ListPack.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720181630203.png)

## String (Chuỗi)

### Giới thiệu

String là Data Type đơn giản và được sử dụng phổ biến nhất trong Redis.

String là kiểu Binary-safe, dùng để lưu trữ mọi kiểu dữ liệu như chuỗi văn bản, số nguyên, số thực, ảnh (mã hóa base64), object đã serialize.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124403897.png)

Redis tự xây dựng cấu trúc **Simple Dynamic String (SDS)** thay vì dùng chuỗi C nguyên bản. SDS giúp lưu được dữ liệu nhị phân, lấy độ dài chuỗi trong O(1) và an toàn không lo Buffer Overflow.

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| ------------------------------- | -------------------------------- |
| SET key value | Đặt giá trị cho key chỉ định |
| SETNX key value | Đặt giá trị cho key chỉ khi key chưa tồn tại |
| GET key | Lấy giá trị của key chỉ định |
| MSET key1 value1 key2 value2 …… | Đặt giá trị cho nhiều key cùng lúc |
| MGET key1 key2 ... | Lấy giá trị của nhiều key cùng lúc |
| STRLEN key | Trả về độ dài chuỗi của key |
| INCR key | Tăng giá trị số của key lên 1 |
| DECR key | Giảm giá trị số của key đi 1 |
| EXISTS key | Kiểm tra key có tồn tại hay không |
| DEL key (Chung) | Xóa key chỉ định |
| EXPIRE key seconds (Chung) | Đặt thời gian hết hạn (TTL) cho key |

Các thao tác cơ bản:

```bash
> SET key value
OK
> GET key
"value"
> EXISTS key
(integer) 1
> STRLEN key
(integer) 5
> DEL key
(integer) 1
```

Bộ đếm Counter:

```bash
> SET number 1
OK
> INCR number # Tăng số lên 1
(integer) 2
> DECR number # Giảm số đi 1
(integer) 1
```

### Kịch bản áp dụng

- **Lưu trữ dữ liệu thông thường**: Cache Session, Token, URL hình ảnh, Object đã serialize.
- **Bộ đếm (Counter)**: Đếm số lượng request theo đơn vị thời gian (Rate Limiting đơn giản), lượt xem trang.
- **Distributed Lock**: Lệnh `SETNX key value` để triển khai khóa phân tán đơn giản.

## List (Danh sách)

### Giới thiệu

List trong Redis chính là việc triển khai cấu trúc dữ liệu Linked List kép (Doubly Linked List), hỗ trợ duyệt và tìm kiếm hai chiều.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124413287.png)

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| --------------------------- | ------------------------------------------ |
| RPUSH key value1 value2 ... | Thêm một hoặc nhiều phần tử vào đuôi (bên phải) List |
| LPUSH key value1 value2 ... | Thêm một hoặc nhiều phần tử vào đầu (bên trái) List |
| LSET key index value | Đặt giá trị tại vị trí index trong List |
| LPOP key | Phân tán và lấy ra phần tử đầu tiên (ngoài cùng bên trái) |
| RPOP key | Phân tán và lấy ra phần tử cuối cùng (ngoài cùng bên phải) |
| LLEN key | Lấy số lượng phần tử của List |
| LRANGE key start end | Lấy danh sách phần tử trong phạm vi start và end |

Mô hình Queue (FIFO) bằng `RPUSH/LPOP` hoặc `LPUSH/RPOP`:

```bash
> RPUSH myList value1
(integer) 1
> RPUSH myList value2 value3
(integer) 3
> LPOP myList
"value1"
```

Mô hình Stack (LIFO) bằng `RPUSH/RPOP` hoặc `LPUSH/LPOP`:

```bash
> RPUSH myList2 value1 value2 value3
(integer) 3
> RPOP myList2
"value3"
```

![](https://oss.javaguide.cn/github/javaguide/database/redis/redis-list.png)

### Kịch bản áp dụng

- **Dòng thông tin (News Feed)**: Danh sách bài viết mới nhất, hoạt động mới nhất (`LPUSH`, `LRANGE`).
- **Message Queue đơn giản**: Dùng List làm hàng chờ tin nhắn đơn giản.

## Hash (Bảng băm)

### Giới thiệu

Hash trong Redis là bảng ánh xạ field-value kiểu String, cực kỳ thích hợp để lưu trữ Object (thao tác trực tiếp trên từng field của Object).

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124421703.png)

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| ----------------------------------------- | -------------------------------------------------------- |
| HSET key field value | Đặt giá trị cho field trong Hash |
| HSETNX key field value | Đặt giá trị cho field chỉ khi field chưa tồn tại |
| HMSET key field1 value1 field2 value2 ... | Đặt giá trị cho nhiều cặp field-value cùng lúc |
| HGET key field | Lấy giá trị của field chỉ định trong Hash |
| HMGET key field1 field2 ... | Lấy giá trị của nhiều field cùng lúc |
| HGETALL key | Lấy toàn bộ các cặp field-value trong Hash |
| HEXISTS key field | Kiểm tra field có tồn tại trong Hash không |
| HDEL key field1 field2 ... | Xóa một hoặc nhiều field trong Hash |
| HLEN key | Lấy số lượng field trong Hash |
| HINCRBY key field increment | Thực hiện phép cộng/trừ số nguyên trên field |

Ví dụ lưu trữ Object:

```bash
> HMSET userInfoKey name "guide" description "dev" age 24
OK
> HEXISTS userInfoKey name
(integer) 1
> HGET userInfoKey name
"guide"
> HGETALL userInfoKey
1) "name"
2) "guide"
3) "description"
4) "dev"
5) "age"
6) "24"
```

### Kịch bản áp dụng

- **Lưu trữ dữ liệu Object**: Thông tin người dùng, thông tin sản phẩm, bài viết, thông tin giỏ hàng (Shopping Cart).

## Set (Tập hợp)

### Giới thiệu

Set trong Redis là một tập hợp không có thứ tự, các phần tử trong tập hợp là duy nhất không trùng lặp (tương tự `HashSet` trong Java). Set cung cấp các hàm tìm giao, hợp, hiệu cực kỳ mạnh mẽ.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124430264.png)

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| ------------------------------------- | ----------------------------------------- |
| SADD key member1 member2 ... | Thêm một hoặc nhiều phần tử vào Set |
| SMEMBERS key | Lấy tất cả phần tử trong Set |
| SCARD key | Lấy số lượng phần tử trong Set |
| SISMEMBER key member | Kiểm tra phần tử có nằm trong Set không |
| SINTER key1 key2 ... | Lấy tập giao (Intersection) của các Set |
| SINTERSTORE destination key1 key2 ... | Lưu tập giao của các Set vào destination |
| SUNION key1 key2 ... | Lấy tập hợp (Union) của các Set |
| SUNIONSTORE destination key1 key2 ... | Lưu tập hợp của các Set vào destination |
| SDIFF key1 key2 ... | Lấy tập hiệu (Difference) của các Set |
| SDIFFSTORE destination key1 key2 ... | Lưu tập hiệu của các Set vào destination |
| SPOP key count | Rút ngẫu nhiên và xóa 1 hoặc nhiều phần tử khỏi Set |
| SRANDMEMBER key count | Rút ngẫu nhiên 1 hoặc nhiều phần tử (không xóa) |

### Kịch bản áp dụng

- **Lưu trữ dữ liệu không trùng lặp**: Thống kê UV website, Like bài viết.
- **Tìm tập Giao, Hợp, Hiệu**: Bạn chung (Giao), Fan chung (Giao), Gợi ý bạn bè / âm nhạc (Hiệu / Hợp).
- **Lấy ngẫu nhiên**: Hệ thống quay số trúng thưởng, điểm danh ngẫu nhiên (`SPOP`, `SRANDMEMBER`).

## Sorted Set (Zset - Tập hợp có thứ tự)

### Giới thiệu

Sorted Set tương tự như Set, nhưng bổ sung thêm tham số trọng số `score`, giúp các phần tử trong tập hợp được sắp xếp có thứ tự theo `score`.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220719124437791.png)

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ZADD key score1 member1 score2 member2 ... | Thêm một hoặc nhiều phần tử kèm score vào Sorted Set |
| ZCARD KEY | Lấy số lượng phần tử trong Sorted Set |
| ZSCORE key member | Lấy giá trị score của phần tử chỉ định |
| ZRANGE key start end | Lấy danh sách phần tử từ start đến end (score từ thấp đến cao) |
| ZREVRANGE key start end | Lấy danh sách phần tử từ start đến end (score từ cao đến thấp) |
| ZREVRANK key member | Lấy thứ hạng của phần tử chỉ định (score từ lớn đến nhỏ) |

Ví dụ thao tác Zset:

```bash
> ZADD myZset 2.0 value1 1.0 value2
(integer) 2
> ZRANGE myZset 0 1
1) "value2"
2) "value1"
> ZREVRANGE myZset 0 1
1) "value1"
2) "value2"
```

### Kịch bản áp dụng

- **Bảng xếp hạng (Leaderboard)**: Bảng xếp hạng tặng quà livestream, bảng xếp hạng bước chân WeChat, xếp hạng game, bảng xếp hạng chủ đề hot.
- **Hàng chờ công việc theo độ ưu tiên (Priority Queue)**.

## Tóm tắt

| Data Type | Mô tả |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| String | Binary-safe, lưu chuỗi, số, ảnh mã hóa base64, object đã serialize. |
| List | Doubly Linked List, hỗ trợ duyệt hai chiều, thao tác push/pop ở hai đầu. |
| Hash | Bảng ánh xạ field-value kiểu String, cực kỳ thích hợp lưu trữ Object. |
| Set | Tập hợp không có thứ tự, phần tử duy nhất không trùng lặp. |
| Zset | Tập hợp có thứ tự dựa trên trọng số `score`. Tương tự kết hợp giữa HashMap và TreeSet. |

## Đọc thêm về Data Structure

- [Chi tiết Cấu trúc dữ liệu tuyến tính](../../cs-basics/data-structure/linear-data-structure.md)
- [Tóm tắt câu hỏi phỏng vấn Hash Table](../../cs-basics/data-structure/hash-table.md)
- [Tóm tắt câu hỏi phỏng vấn Skip List](../../cs-basics/data-structure/skip-list.md)

<!-- @include: @article-footer.snippet.md -->
