---
title: MySQL隐式转换造成索引失效
description: 深入分析MySQL中隐式类型转换导致索引失效的原因和场景，通过实际案例演示字符串与数字比较时的性能问题，并给出避免索引失效的最佳实践。
category: 数据库
tag:
  - MySQL
  - 性能优化
head:
  - - meta
    - name: keywords
      content: MySQL隐式转换,索引失效,类型转换,MySQL性能优化,数据类型不匹配,全表扫描,SQL优化
---

> Phiên bản MySQL sử dụng trong bài test này là `5.7.26`, theo sự cập nhật của phiên bản MySQL một số tính năng có thể có thay đổi, bài viết không đại diện quan điểm và kết luận chính xác cho mọi phiên bản MySQL.
>
> Bài viết gốc: <https://www.guitu18.com/post/2019/11/24/61.html>

## Lời nói đầu

Tối ưu hóa cơ sở dữ liệu là một nhiệm vụ đường dài và nặng nề, muốn làm tối ưu hóa bắt buộc phải hiểu sâu các tính năng khác nhau của cơ sở dữ liệu. Trong quá trình phát triển chúng ta thường gặp một số căn bệnh nan y nguyên nhân rất đơn giản nhưng hậu quả lại rất nghiêm trọng, các vấn đề này thường không dễ định vị, định vị mất thời gian công sức cuối cùng phát hiện ra là do một sơ suất rất nhỏ gây ra, hoặc do chưa hiểu về một tính năng kỹ thuật nào đó.

Ở góc độ cơ sở dữ liệu, phổ biến nhất có lẽ là vô hiệu hóa Index (rớt Index), và lúc đầu do lượng dữ liệu nhỏ nên chưa dễ phát hiện. Nhưng cùng với sự mở rộng nghiệp vụ và gia tăng lượng dữ liệu, vấn đề hiệu năng dần dần bộc lộ, nếu không xử lý kịp thời rất dễ gây ra hiệu ứng hòn tuyết lăn, cuối cùng làm database bị treo thậm chí sập. Có rất nhiều nguyên nhân gây ra vô hiệu hóa Index, bài viết này tôi ghi chép về **Chuyển đổi ngầm định (Implicit Conversion) gây vô hiệu hóa Index**.

## Chuẩn bị dữ liệu

Đầu tiên sử dụng Stored Procedure sinh 10 triệu bản ghi thử nghiệm trên bảng `test1`:
Bảng thử nghiệm có 7 field (bao gồm Primary Key `id`), `num1` và `num2` lưu các số thứ tự giống `id`, trong đó `num2` là kiểu chuỗi (VARCHAR).
`type1` và `type2` lưu kết quả chia lấy dư của id cho 5, `type2` không tạo Index.
`str1` và `str2` lưu chuỗi ngẫu nhiên 20 ký tự, `str1` NOT NULL, `str2` cho phép NULL.

```sql
-- Tạo bảng test
DROP TABLE IF EXISTS test1;
CREATE TABLE `test1` (
    `id` int(11) NOT NULL,
    `num1` int(11) NOT NULL DEFAULT '0',
    `num2` varchar(11) NOT NULL DEFAULT '',
    `type1` int(4) NOT NULL DEFAULT '0',
    `type2` int(4) NOT NULL DEFAULT '0',
    `str1` varchar(100) NOT NULL DEFAULT '',
    `str2` varchar(100) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `num1` (`num1`),
    KEY `num2` (`num2`),
    KEY `type1` (`type1`),
    KEY `str1` (`str1`),
    KEY `str2` (`str2`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
-- Tạo Stored Procedure
DROP PROCEDURE IF EXISTS pre_test1;
DELIMITER //
CREATE PROCEDURE `pre_test1`()
BEGIN
    DECLARE i INT DEFAULT 0;
    SET autocommit = 0;
    WHILE i < 10000000 DO
        SET i = i + 1;
        SET @str1 = SUBSTRING(MD5(RAND()),1,20);
        IF i % 100 = 0 THEN
            SET @str2 = NULL;
        ELSE
            SET @str2 = @str1;
        END IF;
        INSERT INTO test1 (`id`, `num1`, `num2`,
        `type1`, `type2`, `str1`, `str2`)
        VALUES (CONCAT('', i), CONCAT('', i),
        CONCAT('', i), i%5, i%5, @str1, @str2);
        IF i % 10000 = 0 THEN
            COMMIT;
        END IF;
    END WHILE;
END;
// DELIMITER ;
-- Thực thi Stored Procedure
CALL pre_test1();
```

Hình ảnh dữ liệu sinh ra:

![](https://oss.javaguide.cn/github/javaguide/mysqlindex-invalidation-caused-by-implicit-conversion-01.png)

## Thử nghiệm SQL

Hãy xem nhóm 4 câu SQL sau. Bảng dữ liệu của chúng ta có field `num1` kiểu `int`, `num2` kiểu `varchar`, nhưng dữ liệu lưu trữ đều là số giống `id`, cả 2 field đều đã tạo Index.

```sql
1: SELECT * FROM `test1` WHERE num1 = 10000;
2: SELECT * FROM `test1` WHERE num1 = '10000';
3: SELECT * FROM `test1` WHERE num2 = 10000;
4: SELECT * FROM `test1` WHERE num2 = '10000';
```

Thử nghiệm 4 câu SQL này cho kết quả chênh lệch rất lớn:
- Các câu 1, 2, 4 cơ bản ra kết quả tức thì (khoảng 0.001 ~ 0.005 giây). Trong lượng dữ liệu 10 triệu bản ghi, kết quả này cho thấy 3 câu SQL không có chênh lệch hiệu năng.
- Tuy nhiên câu SQL thứ 3 (`num2 = 10000`), thời gian thực thi mất từ **4.5 ~ 4.8 giây**.

Tại sao 3 và 4 lại chênh lệch lớn như vậy, còn 1 và 2 thì lại không có chênh lệch? Hãy xem Execution Plan của 4 câu SQL:

![](https://oss.javaguide.cn/github/javaguide/mysqlindex-invalidation-caused-by-implicit-conversion-02.png)

Có thể thấy các câu 1, 2, 4 đều dùng được Index, type là `ref`, `rows` quét chỉ là 1. Còn câu thứ 3 không dùng được Index nên là Full Table Scan, `rows` quét lên tới 10 triệu bản ghi!

Tra cứu tài liệu chính thức MySQL phát hiện đây là do **Chuyển đổi ngầm định (Implicit Conversion)** gây ra:

> Tài liệu chính thức: [12.2 Type Conversion in Expression Evaluation](https://dev.mysql.com/doc/refman/5.7/en/type-conversion.html?spm=5176.100239.blogcont47339.5.1FTben)
>
> Khi toán tử được sử dụng với các toán hạng khác kiểu, việc chuyển đổi kiểu sẽ xảy ra để làm cho các toán hạng tương thích. Một số chuyển đổi xảy ra ngầm định. Quy tắc số 7:
>
> 7. **Tất cả các trường hợp khác, cả hai tham số đều sẽ được chuyển đổi thành số thực dấu phẩy động (DOUBLE) rồi mới so sánh.**

Dựa theo quy tắc chính thức:

- Câu SQL thứ 2: `SELECT * FROM test1 WHERE num1 = '10000';`. Vế trái `num1` là kiểu int, vế phải là chuỗi `'10000'`. Việc chuyển đổi xảy ra ở vế phải (chuyển chuỗi hằng số thành số). Cột Index vế trái `num1` không bị biến đổi -> **Vẫn trúng Index**.
- Câu SQL thứ 3: `SELECT * FROM test1 WHERE num2 = 10000;`. Vế trái `num2` là kiểu varchar, vế phải là int `10000`. Do vế trái là cột truy vấn kiểu chuỗi, MySQL phải chuyển các chuỗi trong cột `num2` thành số để so sánh với `10000`. Vì các chuỗi như `'10000'`, `'10000a'`, `'010000'`, `' 10000'`... đều có thể chuyển đổi thành số `10000`, làm mất tính thứ tự nhị phân trên cây Index B+Tree -> **Vô hiệu hóa Index, chuyển sang Full Table Scan!**

Thử nghiệm chèn dữ liệu kiểm chứng:

```sql
INSERT INTO `test1` (`id`, `num1`, `num2`, `type1`, `type2`, `str1`, `str2`) VALUES ('10000001', '10000', '10000a', '0', '0', '2df3d9465ty2e4hd523', '2df3d9465ty2e4hd523');
INSERT INTO `test1` (`id`, `num1`, `num2`, `type1`, `type2`, `str1`, `str2`) VALUES ('10000002', '10000', '010000', '0', '0', '2df3d9465ty2e4hd523', '2df3d9465ty2e4hd523');
INSERT INTO `test1` (`id`, `num1`, `num2`, `type1`, `type2`, `str1`, `str2`) VALUES ('10000003', '10000', ' 10000', '0', '0', '2df3d9465ty2e4hd523', '2df3d9465ty2e4hd523');
```

Truy vấn với câu SQL 3:

![](https://oss.javaguide.cn/github/javaguide/mysqlindex-invalidation-caused-by-implicit-conversion-03.png)

Quy tắc chuyển đổi chuỗi sang số trong MySQL:
1. **Không bắt đầu bằng số**: Chuyển thành `0` (như `'abc'`, `'a123bc'` -> `0`).
2. **Bắt đầu bằng số**: Trích xuất từ ký tự đầu tiên đến ký tự không phải số đầu tiên (như `'123abc'` -> `123`, `'012abc'` -> `12`).

![](https://oss.javaguide.cn/github/javaguide/mysqlindex-invalidation-caused-by-implicit-conversion-04.png)

Query thử nghiệm: `SELECT * FROM test1 WHERE str1 = 1234;`

![](https://oss.javaguide.cn/github/javaguide/mysqlindex-invalidation-caused-by-implicit-conversion-05.png)

## Phân tích và Tóm tắt

Thông qua thử nghiệm trên, chúng ta rút ra:

1. Khi toán tử có **kiểu dữ liệu hai bên không nhất quán**, sẽ xảy ra **chuyển đổi ngầm định**.
2. Khi chuyển đổi ngầm định xảy ra trên **cột vế trái là kiểu số**, tác động hiệu năng không lớn.
3. Khi chuyển đổi ngầm định xảy ra trên **cột vế trái là kiểu chuỗi**, sẽ dẫn đến vô hiệu hóa Index, gây ra Full Table Scan hiệu năng cực kỳ kém.
4. Chuỗi chuyển thành số: không bắt đầu bằng số chuyển thành `0`, bắt đầu bằng số trích xuất đến ký tự không phải số đầu tiên.

Do đó, khi viết SQL cần rèn luyện thói quen tốt: Cột kiểu dữ liệu gì thì điều kiện vế phải viết đúng kiểu dữ liệu đó. Đặc biệt khi query cột chuỗi, vế phải bắt buộc phải bọc trong dấu ngoặc đơn/kép để xác định là chuỗi.

<!-- @include: @article-footer.snippet.md -->
