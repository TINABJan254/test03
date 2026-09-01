---
title: Redis 3 种特殊数据类型详解
description: 详解Redis三种特殊数据类型Bitmap、HyperLogLog、GEO的使用方法和应用场景，包括签到统计、UV统计、附近的人等典型业务场景实现。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis特殊数据类型,Bitmap,HyperLogLog,GEO,位图,基数统计,地理位置,签到统计,UV统计
---

Ngoài 5 Data Type cơ bản, Redis còn hỗ trợ 3 Data Type đặc biệt: Bitmap, HyperLogLog, GEO.

## Bitmap (Mảng bit / 位图)

### Giới thiệu

Bitmap không phải là một Data Type thực tế riêng biệt, mà là một tập hợp các thao tác hướng bit được định nghĩa trên kiểu String. Do String là binary-safe blob và có độ dài tối đa 512MB, nên nó thích hợp để lưu trữ tới $2^{32}$ bit khác nhau.

Bitmap lưu trữ dãy số nhị phân liên tục (0 và 1). Chỉ cần 1 bit để biểu diễn giá trị hoặc trạng thái của một phần tử, key là tên tập hợp, vị trí chỉ số trong mảng gọi là `offset` (độ lệch). 8 bit tạo thành 1 byte nên Bitmap cực kỳ tiết kiệm dung lượng lưu trữ.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720194154133.png)

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| ------------------------------------- | ---------------------------------------------------------------- |
| SETBIT key offset value | Đặt giá trị (0 hoặc 1) tại vị trí offset chỉ định |
| GETBIT key offset | Lấy giá trị tại vị trí offset chỉ định |
| BITCOUNT key start end | Đếm số lượng bit có giá trị 1 từ start đến end |
| BITOP operation destkey key1 key2 ... | Thực hiện phép toán logic (AND, OR, XOR, NOT) trên các Bitmap |

Ví dụ thao tác cơ bản:

```bash
> SETBIT mykey 7 1
(integer) 0
> SETBIT mykey 7 0
(integer) 1
> GETBIT mykey 7
(integer) 0
> SETBIT mykey 6 1
(integer) 0
> SETBIT mykey 8 1
(integer) 0
> BITCOUNT mykey
(integer) 2
```

### Kịch bản áp dụng

- **Lưu trữ thông tin trạng thái binary (0/1)**: Điểm danh người dùng (Check-in), người dùng Active hàng ngày, trạng thái tương tác (đã bấm Like video hay chưa).

## HyperLogLog (Thống kê Cardinality / 基数统计)

### Giới thiệu

HyperLogLog (HLL) là thuật toán xác suất thống kê Cardinality (số lượng phần tử không trùng lặp) nổi tiếng. Redis chỉ tốn đúng **12KB** dung lượng RAM để lưu trữ và ước tính tới $2^{64}$ phần tử khác nhau.

HLL chấp nhận sai số nhỏ (mặc định khoảng `0.81%`) để đánh đổi lấy việc tiết kiệm tối đa dung lượng bộ nhớ.

Redis tự động tối ưu lưu trữ HLL theo 2 dạng:
- **Ma trận thưa (Sparse)**: Khi số lượng đếm ít, chiếm dung lượng cực nhỏ.
- **Ma trận dày (Dense)**: Khi số lượng đạt ngưỡng, chiếm cố định 12KB.

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720194154133.png)

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| ----------------------------------------- | ---------------------------------------------------------------- |
| PFADD key element1 element2 ... | Thêm một hoặc nhiều phần tử vào HyperLogLog |
| PFCOUNT key1 key2 ... | Lấy số lượng phần tử duy nhất ước tính từ một hoặc nhiều HLL |
| PFMERGE destkey sourcekey1 sourcekey2 ... | Gộp nhiều HLL vào destkey |

Ví dụ thao tác cơ bản:

```bash
> PFADD hll foo bar zap
(integer) 1
> PFADD hll zap zap zap
(integer) 0
> PFCOUNT hll
(integer) 3
> PFADD some-other-hll 1 2 3
(integer) 1
> PFCOUNT hll some-other-hll
(integer) 6
> PFMERGE desthll hll some-other-hll
"OK"
> PFCOUNT desthll
(integer) 6
```

### Kịch bản áp dụng

- **Thống kê số lượng phần tử khổng lồ (hàng triệu, hàng tỷ)**: Thống kê UV (Unique Visitor) truy cập website/app hàng ngày, số IP độc lập truy cập bài viết.

## Geospatial (Chỉ mục địa lý / GEO)

### Giới thiệu

Geospatial Index (gọi tắt là GEO) dùng để lưu trữ thông tin vị trí địa lý (kinh độ, vĩ độ), tầng dưới dựa trên **Sorted Set**.

GEO hỗ trợ tính khoảng cách giữa 2 vị trí, lấy danh sách các vị trí xung quanh một tọa độ cho trước...

![](https://oss.javaguide.cn/github/javaguide/database/redis/image-20220720194359494.png)

### Các lệnh phổ biến

| Lệnh | Giới thiệu |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| GEOADD key longitude1 latitude1 member1 ... | Thêm thông tin kinh độ vĩ độ của phần tử vào GEO |
| GEOPOS key member1 member2 ... | Trả về thông tin kinh vĩ độ của các phần tử chỉ định |
| GEODIST key member1 member2 M/KM/FT/MI | Trả về khoảng cách giữa 2 phần tử (mét, km, feet, dặm) |
| GEORADIUS key longitude latitude radius distance | Lấy các phần tử nằm trong bán kính distance tính từ tọa độ chỉ định |
| GEORADIUSBYMEMBER key member radius distance | Lấy các phần tử nằm trong bán kính distance tính từ 1 phần tử có sẵn |

Thao tác cơ bản:

```bash
> GEOADD personLocation 116.33 39.89 user1 116.34 39.90 user2 116.35 39.88 user3
3
> GEOPOS personLocation user1
116.3299986720085144
39.89000061669732844
> GEODIST personLocation user1 user2 km
1.4018
```

Dữ liệu tọa độ trong GEO được chuyển đổi thành số nguyên bằng thuật toán GeoHash và dùng làm `score` cho Sorted Set tầng dưới.

Xóa phần tử trong GEO dùng lệnh của Sorted Set: `ZREM personLocation user1`.

### Kịch bản áp dụng

- **Quản lý dữ liệu không gian địa lý**: Tìm người ở gần ("Nearby People"), tìm quán ăn/cửa hàng xung quanh.

## Tóm tắt

| Data Type | Mô tả |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Bitmap | Xem như mảng bit 0/1 với chỉ số offset. Tiết kiệm dung lượng tối đa khi lưu trữ các trạng thái binary (0/1). |
| HyperLogLog | Thống kê số lượng phần tử không trùng lặp (Cardinality) dung lượng khổng lồ với đúng 12KB RAM (chấp nhận sai số 0.81%). |
| Geospatial index | Lưu trữ và tính toán khoảng cách tọa độ địa lý, dựa trên Sorted Set. |

<!-- @include: @article-footer.snippet.md -->
