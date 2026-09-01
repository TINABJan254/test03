---
title: SQL常见面试题总结（1）
description: SQL常见面试题总结第一篇，涵盖SELECT检索数据、WHERE条件过滤、ORDER BY排序、DISTINCT去重、LIMIT分页等基础查询操作及牛客真题解析。
category: 数据库
tag:
  - 数据库基础
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL面试题,SELECT查询,WHERE条件,ORDER BY排序,DISTINCT去重,LIMIT分页,SQL基础
---

> Các câu hỏi từ: [Niuke Tiba - SQL Must-Knows](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=298)

## Truy xuất dữ liệu

`SELECT` dùng để truy vấn dữ liệu từ database.

### Truy xuất tất cả ID từ bảng Customers

Cho bảng `Customers` như sau:

| cust_id |
| ------- |
| A       |
| B       |
| C       |

Viết câu lệnh SQL để truy xuất tất cả `cust_id` từ bảng `Customers`.

Đáp án:

```sql
SELECT cust_id
FROM Customers
```

### Truy xuất và liệt kê danh sách các sản phẩm đã được đặt hàng

Bảng `OrderItems` chứa column `prod_id` không rỗng đại diện cho ID sản phẩm, chứa tất cả các sản phẩm đã được đặt hàng (một số đã được đặt hàng nhiều lần).

| prod_id |
| ------- |
| a1      |
| a2      |
| a3      |
| a4      |
| a5      |
| a6      |
| a7      |

Viết câu lệnh SQL để truy xuất và liệt kê danh sách đã loại bỏ trùng lặp (distinct) của tất cả sản phẩm đã được đặt hàng (`prod_id`).

Đáp án:

```sql
SELECT DISTINCT prod_id
FROM OrderItems
```

Kiến thức: `DISTINCT` dùng để trả về các giá trị duy nhất khác nhau trong column.

### Truy xuất tất cả các column

Hiện có bảng `Customers` (bảng chứa column `cust_id` đại diện cho ID khách hàng, `cust_name` đại diện cho tên khách hàng)

| cust_id | cust_name |
| ------- | --------- |
| a1      | andy      |
| a2      | ben       |
| a3      | tony      |
| a4      | tom       |
| a5      | an        |
| a6      | lee       |
| a7      | hex       |

Cần viết câu lệnh SQL để truy xuất tất cả các column.

Đáp án:

```sql
SELECT cust_id, cust_name
FROM Customers
```

## Sắp xếp dữ liệu truy xuất

`ORDER BY` dùng để sắp xếp tập kết quả theo một hoặc nhiều column. Mặc định sắp xếp các bản ghi theo thứ tự tăng dần (ASC), nếu cần sắp xếp theo thứ tự giảm dần, có thể sử dụng từ khóa `DESC`.

### Truy xuất tên khách hàng và sắp xếp

Cho bảng `Customers`, `cust_id` đại diện cho ID khách hàng, `cust_name` đại diện cho tên khách hàng.

| cust_id | cust_name |
| ------- | --------- |
| a1      | andy      |
| a2      | ben       |
| a3      | tony      |
| a4      | tom       |
| a5      | an        |
| a6      | lee       |
| a7      | hex       |

Truy xuất tất cả tên khách hàng (`cust_name`) từ bảng `Customers`, và hiển thị kết quả theo thứ tự từ Z đến A.

Đáp án:

```sql
SELECT cust_name
FROM Customers
ORDER BY cust_name DESC
```

### Sắp xếp theo ID khách hàng và ngày tháng

Cho bảng `Orders`:

| cust_id | order_num | order_date          |
| ------- | --------- | ------------------- |
| andy    | aaaa      | 2021-01-01 00:00:00 |
| andy    | bbbb      | 2021-01-01 12:00:00 |
| bob     | cccc      | 2021-01-10 12:00:00 |
| dick    | dddd      | 2021-01-11 00:00:00 |

Viết câu lệnh SQL để truy xuất ID khách hàng (`cust_id`) và số đơn hàng (`order_num`) từ bảng `Orders`, đầu tiên sắp xếp kết quả theo ID khách hàng, sau đó sắp xếp giảm dần theo ngày đặt hàng.

Đáp án:

```sql
# Sắp xếp theo tên column
# Chú ý: order_date giảm dần, chứ không phải order_num
SELECT cust_id, order_num
FROM Orders
ORDER BY cust_id,order_date DESC
```

Kiến thức: Khi `ORDER BY` sắp xếp trên nhiều column, column sắp xếp trước đặt ở phía trước, column sắp xếp sau đặt ở phía sau. Đồng thời, các column khác nhau có thể có quy tắc sắp xếp khác nhau.

### Sắp xếp theo số lượng và giá cả

Giả sử có một bảng `OrderItems`:

| quantity | item_price |
| -------- | ---------- |
| 1        | 100        |
| 10       | 1003       |
| 2        | 500        |

Viết câu lệnh SQL để hiển thị số lượng (`quantity`) và giá cả (`item_price`) trong bảng `OrderItems`, và sắp xếp theo số lượng từ nhiều đến ít, giá cả từ cao đến thấp.

Đáp án:

```sql
SELECT quantity, item_price
FROM OrderItems
ORDER BY quantity DESC,item_price DESC
```

### Kiểm tra câu lệnh SQL

Cho bảng `Vendors`:

| vend_name |
| --------- |
| 海底捞    |
| 小龙坎    |
| 大龙燚    |

Câu lệnh SQL dưới đây có vấn đề gì không? Hãy sửa lại cho đúng để nó có thể chạy chính xác và trả về kết quả được sắp xếp ngược (giảm dần) theo `vend_name`.

```sql
SELECT vend_name,
FROM Vendors
ORDER vend_name DESC
```

Sau khi sửa:

```sql
SELECT vend_name
FROM Vendors
ORDER BY vend_name DESC
```

Kiến thức:

- Dấu phẩy dùng để ngăn cách giữa các column với nhau.
- ORDER BY phải có từ BY, cần viết đầy đủ và đặt đúng vị trí.

## Lọc dữ liệu

`WHERE` có thể lọc dữ liệu trả về.

Các toán tử dưới đây có thể được sử dụng trong mệnh đề `WHERE`:

| Toán tử  | Mô tả                                                         |
| :------ | :----------------------------------------------------------- |
| =       | Bằng                                                         |
| <>      | Không bằng. **Ghi chú:** Trong một số phiên bản SQL, toán tử này có thể viết là != |
| >       | Lớn hơn                                                         |
| <       | Nhỏ hơn                                                         |
| >=      | Lớn hơn hoặc bằng                                                     |
| <=      | Nhỏ hơn hoặc bằng                                                     |
| BETWEEN | Trong một khoảng phạm vi nào đó                                                 |
| LIKE    | Tìm kiếm theo mẫu (pattern)                                                 |
| IN      | Chỉ định nhiều giá trị có thể có cho một column                                   |

### Trả về sản phẩm có giá cố định

Cho bảng `Products`:

| prod_id | prod_name      | prod_price |
| ------- | -------------- | ---------- |
| a0018   | sockets        | 9.49       |
| a0019   | iphone13       | 600        |
| b0018   | gucci t-shirts | 1000       |

【Câu hỏi】Truy xuất ID sản phẩm (`prod_id`) và tên sản phẩm (`prod_name`) từ bảng `Products`, chỉ trả về các sản phẩm có giá là 9.49 USD.

Đáp án:

```sql
SELECT prod_id, prod_name
FROM Products
WHERE prod_price = 9.49
```

### Trả về sản phẩm có giá cao hơn

Cho bảng `Products`:

| prod_id | prod_name      | prod_price |
| ------- | -------------- | ---------- |
| a0018   | sockets        | 9.49       |
| a0019   | iphone13       | 600        |
| b0019   | gucci t-shirts | 1000       |

【Câu hỏi】Viết câu lệnh SQL để truy xuất ID sản phẩm (`prod_id`) và tên sản phẩm (`prod_name`) từ bảng `Products`, chỉ trả về các sản phẩm có giá từ 9 USD trở lên.

Đáp án:

```sql
SELECT prod_id, prod_name
FROM Products
WHERE prod_price >= 9
```

### Trả về sản phẩm và sắp xếp theo giá

Cho bảng `Products`:

| prod_id | prod_name | prod_price |
| ------- | --------- | ---------- |
| a0011   | egg       | 3          |
| a0019   | sockets   | 4          |
| b0019   | coffee    | 15         |

【Câu hỏi】Viết câu lệnh SQL để trả về tên (`prod_name`) và giá (`prod_price`) của tất cả sản phẩm có giá từ 3 USD đến 6 USD trong bảng `Products`, sau đó sắp xếp kết quả theo giá.

Đáp án:

```sql
SELECT prod_name, prod_price
FROM Products
WHERE prod_price BETWEEN 3 AND 6
ORDER BY prod_price

# Hoặc
SELECT prod_name, prod_price
FROM Products
WHERE prod_price >= 3 AND prod_price <= 6
ORDER BY prod_price
```

### Trả về nhiều sản phẩm hơn

Bảng `OrderItems` chứa: số đơn hàng `order_num`, số lượng sản phẩm `quantity`

| order_num | quantity |
| --------- | -------- |
| a1        | 105      |
| a2        | 1100     |
| a2        | 200      |
| a4        | 1121     |
| a5        | 10       |
| a2        | 19       |
| a7        | 5        |

【Câu hỏi】Truy xuất tất cả các số đơn hàng (`order_num`) khác nhau và không trùng lặp từ bảng `OrderItems`, trong đó mỗi đơn hàng phải chứa từ 100 sản phẩm trở lên.

Đáp án:

```sql
SELECT order_num
FROM OrderItems
GROUP BY order_num
HAVING SUM(quantity) >= 100
```

## Lọc dữ liệu nâng cao

Các toán tử `AND` và `OR` được dùng để lọc bản ghi dựa trên nhiều hơn một điều kiện, cả hai có thể kết hợp sử dụng. `AND` bắt buộc cả 2 điều kiện đều thỏa mãn, `OR` chỉ cần 1 trong 2 điều kiện thỏa mãn là được.

### Truy xuất tên nhà cung cấp

Bảng `Vendors` có các field tên nhà cung cấp (`vend_name`), quốc gia nhà cung cấp (`vend_country`), bang nhà cung cấp (`vend_state`)

| vend_name | vend_country | vend_state |
| --------- | ------------ | ---------- |
| apple     | USA          | CA         |
| vivo      | CNA          | shenzhen   |
| huawei    | CNA          | xian       |

【Câu hỏi】Viết câu lệnh SQL để truy xuất tên nhà cung cấp (`vend_name`) từ bảng `Vendors`, chỉ trả về các nhà cung cấp ở bang California (cần lọc theo quốc gia [USA] và bang [CA], phòng trường hợp quốc gia khác cũng có bang CA)

Đáp án:

```sql
SELECT vend_name
FROM Vendors
WHERE vend_country = 'USA' AND vend_state = 'CA'
```

### Truy xuất và liệt kê danh sách sản phẩm đã đặt hàng

Bảng `OrderItems` chứa tất cả các sản phẩm đã đặt hàng (một số đã được đặt hàng nhiều lần).

| prod_id | order_num | quantity |
| ------- | --------- | -------- |
| BR01    | a1        | 105      |
| BR02    | a2        | 1100     |
| BR02    | a2        | 200      |
| BR03    | a4        | 1121     |
| BR017   | a5        | 10       |
| BR02    | a2        | 19       |
| BR017   | a7        | 5        |

【Câu hỏi】Viết câu lệnh SQL để tìm tất cả các đơn hàng đã đặt mua sản phẩm `BR01`, `BR02` hoặc `BR03` với số lượng ít nhất 100. Bạn cần trả về số đơn hàng (`order_num`), ID sản phẩm (`prod_id`) và số lượng (`quantity`) từ bảng `OrderItems`, và lọc theo ID sản phẩm và số lượng.

Đáp án:

```sql
SELECT order_num, prod_id, quantity
FROM OrderItems
WHERE prod_id IN ('BR01', 'BR02', 'BR03') AND quantity >= 100
```

### Trả về tên và giá của tất cả sản phẩm có giá từ 3 USD đến 6 USD

Cho bảng `Products`:

| prod_id | prod_name | prod_price |
| ------- | --------- | ---------- |
| a0011   | egg       | 3          |
| a0019   | sockets   | 4          |
| b0019   | coffee    | 15         |

【Câu hỏi】Viết câu lệnh SQL để trả về tên (`prod_name`) và giá (`prod_price`) của tất cả sản phẩm có giá từ 3 USD đến 6 USD, sử dụng toán tử AND, sau đó sắp xếp kết quả tăng dần theo giá.

Đáp án:

```sql
SELECT prod_name, prod_price
FROM Products
WHERE prod_price >= 3 and prod_price <= 6
ORDER BY prod_price
```

### Kiểm tra câu lệnh SQL

Bảng nhà cung cấp `Vendors` có các field tên nhà cung cấp `vend_name`, quốc gia nhà cung cấp `vend_country`, bang/tỉnh nhà cung cấp `vend_state`

| vend_name | vend_country | vend_state |
| --------- | ------------ | ---------- |
| apple     | USA          | CA         |
| vivo      | CNA          | shenzhen   |
| huawei    | CNA          | xian       |

【Câu hỏi】Sửa lại câu lệnh SQL dưới đây để nó trả về kết quả đúng.

```sql
SELECT vend_name
FROM Vendors
ORDER BY vend_name
WHERE vend_country = 'USA' AND vend_state = 'CA';
```

Sau khi sửa:

```sql
SELECT vend_name
FROM Vendors
WHERE vend_country = 'USA' AND vend_state = 'CA'
ORDER BY vend_name
```

Mệnh đề `ORDER BY` bắt buộc phải đặt sau mệnh đề `WHERE`.

## Lọc bằng ký tự đại diện (Wildcard)

Ký tự đại diện SQL (Wildcard) bắt buộc phải sử dụng cùng với toán tử `LIKE`

Trong SQL, có thể sử dụng các ký tự đại diện sau:

| Ký tự đại diện | Mô tả |
| :------------------------------- | :------------------------- |
| `%`                              | Đại diện cho 0 hoặc nhiều ký tự |
| `_`                              | Chỉ thay thế duy nhất 1 ký tự |
| `[charlist]`                     | Bất kỳ 1 ký tự đơn nào trong danh sách ký tự |
| `[^charlist]` hoặc `[!charlist]` | Bất kỳ 1 ký tự đơn nào KHÔNG nằm trong danh sách ký tự |

### Truy xuất tên sản phẩm và mô tả (Phần 1)

Cho bảng `Products` như sau:

| prod_name | prod_desc      |
| --------- | -------------- |
| a0011     | usb            |
| a0019     | iphone13       |
| b0019     | gucci t-shirts |
| c0019     | gucci toy      |
| d0019     | lego toy       |

【Câu hỏi】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về các sản phẩm mà mô tả chứa từ `toy`.

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc LIKE '%toy%'
```

### Truy xuất tên sản phẩm và mô tả (Phần 2)

Cho bảng `Products` như sau:

| prod_name | prod_desc      |
| --------- | -------------- |
| a0011     | usb            |
| a0019     | iphone13       |
| b0019     | gucci t-shirts |
| c0019     | gucci toy      |
| d0019     | lego toy       |

【Câu hỏi】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về các sản phẩm mà mô tả KHÔNG xuất hiện từ `toy`, cuối cùng sắp xếp kết quả theo "tên sản phẩm".

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc NOT LIKE '%toy%'
ORDER BY prod_name
```

### Truy xuất tên sản phẩm和mô tả (Phần 3)

Cho bảng `Products` như sau:

| prod_name | prod_desc        |
| --------- | ---------------- |
| a0011     | usb              |
| a0019     | iphone13         |
| b0019     | gucci t-shirts   |
| c0019     | gucci toy        |
| d0019     | lego carrots toy |

【Câu hỏi】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về các sản phẩm mà mô tả đồng thời xuất hiện cả `toy` và `carrots`. Có nhiều cách để thực hiện việc này, nhưng đối标志 bài tập thử thách này, hãy sử dụng `AND` và hai so sánh `LIKE`.

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc LIKE '%toy%' AND prod_desc LIKE "%carrots%"
```

### Truy xuất tên产品和mô tả (Phần 4)

Cho bảng `Products` như sau:

| prod_name | prod_desc        |
| --------- | ---------------- |
| a0011     | usb              |
| a0019     | iphone13         |
| b0019     | gucci t-shirts   |
| c0019     | gucci toy        |
| d0019     | lego toy carrots |

【Câu hỏi】Viết câu lệnh SQL để truy xuất tên sản phẩm (`prod_name`) và mô tả (`prod_desc`) từ bảng `Products`, chỉ trả về các sản phẩm mà mô tả xuất hiện cả `toy` và `carrots` theo **thứ tự trước sau**. Gợi ý: Chỉ cần sử dụng `LIKE` với ba ký hiệu `%`.

Đáp án:

```sql
SELECT prod_name, prod_desc
FROM Products
WHERE prod_desc LIKE '%toy%carrots%'
```

## Tạo Field tính toán (Calculated Fields)

### Alias (Tên giả/Tên thay thế)

Cách dùng phổ biến của Alias là đổi tên column của bảng trong kết quả truy xuất (để phù hợp với yêu cầu báo cáo cụ thể hoặc nhu cầu khách hàng). Cho bảng `Vendors` đại diện cho thông tin nhà cung cấp, `vend_id` ID nhà cung cấp, `vend_name` tên nhà cung cấp, `vend_address` địa chỉ nhà cung cấp, `vend_city` thành phố nhà cung cấp.

| vend_id | vend_name     | vend_address | vend_city |
| ------- | ------------- | ------------ | --------- |
| a001    | tencent cloud | address1     | shenzhen  |
| a002    | huawei cloud  | address2     | dongguan  |
| a003    | aliyun cloud  | address3     | hangzhou  |
| a003    | netease cloud | address4     | guangzhou |

【Câu hỏi】Viết câu lệnh SQL để truy xuất `vend_id`, `vend_name`, `vend_address` và `vend_city` từ bảng `Vendors`, đổi tên `vend_name` thành `vname`, đổi tên `vend_city` thành `vcity`, đổi tên `vend_address` thành `vaddress`, và sắp xếp kết quả tăng dần theo tên nhà cung cấp.

Đáp án:

```sql
SELECT vend_id, vend_name AS vname, vend_address AS vaddress, vend_city AS vcity
FROM Vendors
ORDER BY vname
# Từ khóa AS có thể bỏ qua
SELECT vend_id, vend_name vname, vend_address vaddress, vend_city vcity
FROM Vendors
ORDER BY vname
```

### Giảm giá

Cửa hàng ví dụ của chúng ta đang thực hiện chương trình khuyến mãi giảm giá, tất cả sản phẩm đều giảm 10%. Bảng `Products` chứa `prod_id` ID sản phẩm, `prod_price` giá sản phẩm.

【Câu hỏi】Viết câu lệnh SQL để trả về `prod_id`, `prod_price` và `sale_price` từ bảng `Products`. `sale_price` là một field tính toán chứa giá khuyến mãi. Gợi ý: Có thể nhân với 0.9 để có được 90% giá gốc (tức giảm giá 10%).

Đáp án:

```sql
SELECT prod_id, prod_price, prod_price * 0.9 AS sale_price
FROM Products
```

Lưu ý: `sale_price` là tên đặt cho kết quả tính toán, chứ không phải tên column gốc.

## Sử dụng Function để xử lý dữ liệu

### Tên đăng nhập của khách hàng

Cửa hàng của chúng ta đã lên sóng, đang tạo tài khoản khách hàng. Tất cả người dùng đều cần tên đăng nhập, tên đăng nhập mặc định là sự kết hợp giữa tên và thành phố sinh sống của họ.

Cho bảng `Customers` như sau:

| cust_id | cust_name | cust_contact | cust_city |
| ------- | --------- | ------------ | --------- |
| a1      | Andy Li   | Andy Li      | Oak Park  |
| a2      | Ben Liu   | Ben Liu      | Oak Park  |
| a3      | Tony Dai  | Tony Dai     | Oak Park  |
| a4      | Tom Chen  | Tom Chen     | Oak Park  |
| a5      | An Li     | An Li        | Oak Park  |
| a6      | Lee Chen  | Lee Chen     | Oak Park  |
| a7      | Hex Liu   | Hex Liu      | Oak Park  |

【Câu hỏi】Viết câu lệnh SQL để trả về ID khách hàng (`cust_id`), tên khách hàng (`cust_name`) và tên đăng nhập (`user_login`), trong đó tên đăng nhập hoàn toàn là chữ in hoa, được ghép từ 2 ký tự đầu tiên của người liên hệ (`cust_contact`) và 3 ký tự đầu tiên của thành phố (`cust_city`). Gợi ý: Cần sử dụng Function, Cắt/Ghép chuỗi và Alias.

Đáp án:

```sql
SELECT cust_id, cust_name, UPPER(CONCAT(SUBSTRING(cust_contact, 1, 2), SUBSTRING(cust_city, 1, 3))) AS user_login
FROM Customers
```

Kiến thức:

- Hàm cắt chuỗi `SUBSTRING()`: Cắt chuỗi, `SUBSTRING(str, n, m)` (n biểu thị vị trí bắt đầu cắt, m biểu thị số ký tự cần cắt) biểu thị trả về chuỗi str cắt m ký tự bắt đầu từ ký tự thứ n;
- Hàm nối chuỗi `CONCAT()`: Nối hai hoặc nhiều chuỗi thành một chuỗi, SELECT CONCAT(A, B): Nối chuỗi A và B.
- Hàm in hoa `UPPER()`: Chuyển đổi chuỗi chỉ định thành chữ in hoa.

### Trả về số đơn hàng和ngày đặt hàng của tất cả đơn hàng tháng 01 năm 2020

Cho bảng đơn hàng `Orders` như sau:

| order_num | order_date          |
| --------- | ------------------- |
| a0001     | 2020-01-01 00:00:00 |
| a0002     | 2020-01-02 00:00:00 |
| a0003     | 2020-01-01 12:00:00 |
| a0004     | 2020-02-01 00:00:00 |
| a0005     | 2020-03-01 00:00:00 |

【Câu hỏi】Viết câu lệnh SQL để trả về số đơn hàng (`order_num`) và ngày đặt hàng (`order_date`) của tất cả đơn hàng trong tháng 01 năm 2020, và sắp xếp tăng dần theo ngày đặt hàng.

Đáp án:

```sql
SELECT order_num, order_date
FROM Orders
WHERE month(order_date) = '01' AND YEAR(order_date) = '2020'
ORDER BY order_date
```

Cũng có thể dùng ký tự đại diện để làm:

```sql
SELECT order_num, order_date
FROM Orders
WHERE order_date LIKE '2020-01%'
ORDER BY order_date
```

Kiến thức:

- Định dạng ngày: `YYYY-MM-DD`
- Định dạng giờ: `HH:MM:SS`

Các hàm thường dùng liên quan đến xử lý ngày tháng và thời gian:

| Hàm           | Mô tả                          |
| --------------- | ------------------------------ |
| `ADDDATE()`     | Thêm một khoảng thời gian ngày (ngày, tuần, v.v.)       |
| `ADDTIME()`     | Thêm một khoảng thời gian giờ (giờ, phút, v.v.)       |
| `CURDATE()`     | Trả về ngày hiện tại                   |
| `CURTIME()`     | Trả về giờ hiện tại                   |
| `DATE()`        | Trả về phần ngày của datetime         |
| `DATEDIFF`      | Tính khoảng cách chênh lệch giữa 2 ngày               |
| `DATE_FORMAT()` | Trả về chuỗi ngày/giờ đã được định dạng   |
| `DAY()`         | Trả về phần ngày (day) của date         |
| `DAYOFWEEK()`   | Trả về thứ trong tuần tương ứng với date |
| `HOUR()`        | Trả về phần giờ của time         |
| `MINUTE()`      | Trả về phần phút của time         |
| `MONTH()`       | Trả về phần tháng của date         |
| `NOW()`         | Trả về ngày và giờ hiện tại             |
| `SECOND()`      | Trả về phần giây của time           |
| `TIME()`        | Trả về phần giờ (time) của datetime     |
| `YEAR()`        | Trả về phần năm của date         |

## Tổng合dữ liệu (Aggregating Data)

Các hàm liên quan đến tổng hợp dữ liệu:

| Hàm     | Mô tả            |
| --------- | ---------------- |
| `AVG()`   | Trả về giá trị trung bình của một column |
| `COUNT()` | Trả về số dòng của một column   |
| `MAX()`   | Trả về giá trị lớn nhất của một column |
| `MIN()`   | Trả về giá trị nhỏ nhất của một column |
| `SUM()`   | Trả về tổng các giá trị của một column   |

### Xác定sản phẩm已售出的总数

Bảng `OrderItems` đại diện cho sản phẩm đã bán out, `quantity` đại diện cho số lượng sản phẩm bán out.

| quantity |
| -------- |
| 10       |
| 100      |
| 1000     |
| 10001    |
| 2        |
| 15       |

【Câu hỏi】Viết câu lệnh SQL để xác định tổng số lượng sản phẩm đã bán out.

Đáp án:

```sql
SELECT Sum(quantity) AS items_ordered
FROM OrderItems
```

### Xác定tổng số lượng已售出 của mã sản phẩm BR01

Bảng `OrderItems` đại diện cho sản phẩm đã bán out, `quantity` đại diện cho số lượng sản phẩm bán out, mã sản phẩm là `prod_id`.

| quantity | prod_id |
| -------- | ------- |
| 10       | AR01    |
| 100      | AR10    |
| 1000     | BR01    |
| 10001    | BR010   |

【Câu hỏi】Sửa câu lệnh đã tạo, xác định tổng số lượng đã bán của mã sản phẩm (`prod_id`) là "BR01".

Đáp án:

```sql
SELECT Sum(quantity) AS items_ordered
FROM OrderItems
WHERE prod_id = 'BR01'
```

### Xác定giá của sản phẩm đắt nhất không vượt quá 10 USD trong bảng Products

Cho bảng `Products` như sau, `prod_price` đại diện cho giá sản phẩm.

| prod_price |
| ---------- |
| 9.49       |
| 600        |
| 1000       |

【Câu hỏi】Viết câu lệnh SQL để xác định giá của sản phẩm đắt nhất (`prod_price`) không vượt quá 10 USD trong bảng `Products`. Đặt tên cho field tính toán được là `max_price`.

Đáp án:

```sql
SELECT Max(prod_price) AS max_price
FROM Products
WHERE prod_price <= 10
```

## Gom nhóm dữ liệu (Grouping Data)

`GROUP BY`:

- Mệnh đề `GROUP BY` gom nhóm các bản ghi vào các dòng tổng hợp (summary rows).
- `GROUP BY` trả về 1 bản ghi cho mỗi nhóm.
- `GROUP BY` thường liên quan đến các hàm Aggregation như `COUNT`, `MAX`, `SUM`, `AVG`, v.v.
- `GROUP BY` có thể gom nhóm theo 1 hoặc nhiều column.
- Sau khi `GROUP BY` sắp xếp theo field gom nhóm, `ORDER BY` có thể dùng field tổng hợp để sắp xếp.

`HAVING`:

- `HAVING` dùng để lọc kết quả tổng hợp của `GROUP BY`.
- `HAVING` bắt buộc phải dùng kết hợp với `GROUP BY`.
- `WHERE` và `HAVING` có thể nằm trong cùng một câu truy vấn.

`HAVING` vs `WHERE`:

- `WHERE`: Lọc các dòng chỉ định, phía sau KHÔNG thể thêm hàm Aggregation (hàm gom nhóm).
- `HAVING`: Lọc nhóm (group), bắt buộc phải dùng kết hợp với `GROUP BY`, không thể dùng riêng lẻ.

### Trả về số lượng dòng của từng số đơn hàng

Bảng `OrderItems` chứa từng sản phẩm của từng đơn hàng

| order_num |
| --------- |
| a002      |
| a002      |
| a002      |
| a004      |
| a007      |

【Câu hỏi】Viết câu lệnh SQL để trả về số lượng dòng (`order_lines`) của từng số đơn hàng (`order_num`), và sắp xếp kết quả tăng dần theo `order_lines`.

Đáp án:

```sql
SELECT order_num, Count(order_num) AS order_lines
FROM OrderItems
GROUP BY order_num
ORDER BY order_lines
```

Kiến thức:

1. `COUNT(*)` hay `COUNT(tên_cột)` đều được, điểm khác biệt là `COUNT(tên_cột)` chỉ thống kê số dòng không NULL;
2. `ORDER BY` thực thi cuối cùng, nên có thể sử dụng Alias của column;
3. Gom nhóm tổng hợp nhất định đừng quên thêm `GROUP BY`, nếu không sẽ chỉ có 1 dòng kết quả.

### Sản phẩm có chi phí thấp nhất của từng nhà cung cấp

Cho bảng `Products`, chứa field `prod_price` đại diện cho giá sản phẩm, `vend_id` đại diện cho ID nhà cung cấp

| vend_id | prod_price |
| ------- | ---------- |
| a0011   | 100        |
| a0019   | 0.1        |
| b0019   | 1000       |
| b0019   | 6980       |
| b0019   | 20         |

【Câu hỏi】Viết câu lệnh SQL để trả về field có tên là `cheapest_item`, field này chứa sản phẩm có chi phí thấp nhất của từng nhà cung cấp (sử dụng `prod_price` trong bảng `Products`), sau đó sắp xếp kết quả tăng dần từ chi phí thấp nhất đến cao nhất.

Đáp án:

```sql
SELECT vend_id, Min(prod_price) AS cheapest_item
FROM Products
GROUP BY vend_id
ORDER BY cheapest_item
```

### Trả về số đơn hàng của tất cả các đơn có tổng số lượng sản phẩm từ 100 trở lên

`OrderItems` đại diện cho bảng sản phẩm đơn hàng, bao gồm: số đơn hàng `order_num` và số lượng đơn hàng `quantity`.

| order_num | quantity |
| --------- | -------- |
| a1        | 105      |
| a2        | 1100     |
| a2        | 200      |
| a4        | 1121     |
| a5        | 10       |
| a2        | 19       |
| a7        | 5        |

【Câu hỏi】Xin hãy viết câu lệnh SQL để trả về tất cả các số đơn hàng có tổng số lượng sản phẩm từ 100 trở lên, kết quả cuối cùng sắp xếp tăng dần theo số đơn hàng.

Đáp án:

```sql
# Aggregation trực tiếp
SELECT order_num
FROM OrderItems
GROUP BY order_num
HAVING Sum(quantity) >= 100
ORDER BY order_num

# Subquery
SELECT a.order_num
FROM (SELECT order_num, Sum(quantity) AS sum_num
    FROM OrderItems
    GROUP BY order_num
    HAVING sum_num >= 100) a
ORDER BY a.order_num
```

Kiến thức:

- `WHERE`: Lọc các dòng chỉ định, phía sau không thể thêm hàm Aggregation.
- `HAVING`: Lọc nhóm, kết hợp với `GROUP BY`, không thể dùng đơn lẻ.

### Tính tổng số tiền

Bảng `OrderItems` đại diện cho thông tin đơn hàng, bao gồm các field: số đơn hàng `order_num` và `item_price` giá bán sản phẩm, `quantity` số lượng sản phẩm.

| order_num | item_price | quantity |
| --------- | ---------- | -------- |
| a1        | 10         | 105      |
| a2        | 1          | 1100     |
| a2        | 1          | 200      |
| a4        | 2          | 1121     |
| a5        | 5          | 10       |
| a2        | 1          | 19       |
| a7        | 7          | 5        |

【Câu hỏi】Viết câu lệnh SQL để gom nhóm theo số đơn hàng, trả về tất cả số đơn hàng có tổng giá trị đơn hàng từ 1000 trở lên, kết quả cuối cùng sắp xếp tăng dần theo số đơn hàng.

Gợi ý: Tổng giá trị = item_price nhân với quantity

Đáp án:

```sql
SELECT order_num, Sum(item_price * quantity) AS total_price
FROM OrderItems
GROUP BY order_num
HAVING total_price >= 1000
ORDER BY order_num
```

### Kiểm tra câu lệnh SQL

Bảng `OrderItems` chứa số đơn hàng `order_num`

| order_num |
| --------- |
| a002      |
| a002      |
| a002      |
| a004      |
| a007      |

【Câu hỏi】Sửa đoạn code dưới đây cho đúng rồi thực thi

```sql
SELECT order_num, COUNT(*) AS items
FROM OrderItems
GROUP BY items
HAVING COUNT(*) >= 3
ORDER BY items, order_num;
```

Sau khi sửa:

```sql
SELECT order_num, COUNT(*) AS items
FROM OrderItems
GROUP BY order_num
HAVING items >= 3
ORDER BY items, order_num;
```

## Sử dụng Subquery (Truy vấn con)

Subquery (truy vấn con) là câu truy vấn SQL được lồng bên trong một câu truy vấn lớn hơn, còn gọi là Inner Query hoặc Inner Select, câu lệnh chứa Subquery cũng được gọi là Outer Query hoặc Outer Select. Nói một cách đơn giản, Subquery là việc lấy kết quả của một truy vấn `SELECT` (truy vấn con) làm nguồn dữ liệu hoặc điều kiện phán đoán cho một câu lệnh SQL khác (truy vấn chính).

Subquery có thể được nhúng vào các câu lệnh `SELECT`, `INSERT`, `UPDATE` và `DELETE`, cũng như sử dụng cùng các toán tử `=`, `<`, `>`, `IN`, `BETWEEN`, `EXISTS`, v.v.

Subquery thường được dùng sau mệnh đề `WHERE` và mệnh đề `FROM`:

- Khi dùng sau mệnh đề `WHERE`, tùy thuộc vào các toán tử khác nhau, Subquery có thể trả về dữ liệu đơn dòng đơn cột, nhiều dòng đơn cột, đơn dòng nhiều cột. Subquery là để trả về giá trị có thể làm điều kiện truy vấn cho mệnh đề WHERE.
- Khi dùng sau mệnh đề `FROM`, thông thường trả về dữ liệu nhiều dòng nhiều cột, tương đương với việc trả về một bảng tạm (temporary table), như vậy mới phù hợp với quy tắc sau `FROM` phải là một bảng. Cách làm này có thể thực hiện liên kết nhiều bảng.

> Lưu ý: Database MySQL bắt đầu hỗ trợ Subquery từ phiên bản 4.1, các phiên bản sớm hơn không hỗ trợ.

Cú pháp cơ bản của Subquery dùng trong mệnh đề `WHERE` như sau:

```sql
SELECT column_name [, column_name ]
FROM table1 [, table2 ]
WHERE column_name operator
(SELECT column_name [, column_name ]
FROM table1 [, table2 ]
[WHERE])
```

- Subquery cần phải đặt trong cặp ngoặc đơn `( )`.
- `operator` biểu thị toán tử dùng cho mệnh đề `WHERE`, có thể là toán tử so sánh (như `=`, `<`, `>`, `<>`, v.v.) hoặc toán tử logic (như `IN`, `NOT IN`, `EXISTS`, `NOT EXISTS`, v.v.), xác định cụ thể theo nhu cầu.

Cú pháp cơ bản của Subquery dùng trong mệnh đề `FROM` như sau:

```sql
SELECT column_name [, column_name ]
FROM (SELECT column_name [, column_name ]
      FROM table1 [, table2 ]
      [WHERE]) AS temp_table_name [, ...]
[JOIN type JOIN table_name ON condition]
WHERE condition;
```

- Kết quả Subquery dùng cho `FROM` trả về tương đương một bảng tạm, do đó cần dùng từ khóa AS để đặt tên cho bảng tạm đó.
- Subquery cần đặt trong cặp ngoặc đơn `( )`.
- Có thể chỉ định nhiều tên bảng tạm và sử dụng câu lệnh `JOIN` để kết nối các bảng này.

### Trả về danh sách khách hàng đã mua sản phẩm giá từ 10 USD trở lên

`OrderItems` đại diện cho bảng sản phẩm đơn hàng, chứa field số đơn hàng: `order_num`, giá đơn hàng: `item_price`; Bảng `Orders` đại diện cho bảng thông tin đơn hàng, chứa ID khách hàng: `cust_id` và số đơn hàng: `order_num`

Bảng `OrderItems`:

| order_num | item_price |
| --------- | ---------- |
| a1        | 10         |
| a2        | 1          |
| a2        | 1          |
| a4        | 2          |
| a5        | 5          |
| a2        | 1          |
| a7        | 7          |

Bảng `Orders`:

| order_num | cust_id |
| --------- | ------- |
| a1        | cust10  |
| a2        | cust1   |
| a2        | cust1   |
| a4        | cust2   |
| a5        | cust5   |
| a2        | cust1   |
| a7        | cust7   |

【Câu hỏi】Sử dụng Subquery, trả về danh sách khách hàng đã mua sản phẩm có giá từ 10 USD trở lên, kết quả không cần sắp xếp.

Đáp án:

```sql
SELECT cust_id
FROM Orders
WHERE order_num IN (SELECT DISTINCT order_num
    FROM OrderItems
    where item_price >= 10)
```

### Xác định những đơn hàng nào đã mua sản phẩm có prod_id là BR01 (Phần 1)

Bảng `OrderItems` đại diện cho bảng thông tin sản phẩm đơn hàng, `prod_id` là ID sản phẩm; Bảng `Orders` đại diện cho bảng đơn hàng có `cust_id` đại diện cho ID khách hàng và ngày đặt hàng `order_date`

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

【Câu hỏi】

Viết câu lệnh SQL, sử dụng Subquery để xác định những đơn hàng nào (trong `OrderItems`) đã mua sản phẩm có `prod_id` là "BR01", sau đó từ bảng `Orders` trả về ID khách hàng (`cust_id`) và ngày đặt hàng (`order_date`) tương ứng, sắp xếp kết quả tăng dần theo ngày đặt hàng.

Đáp án:

```sql
# Cách 1: Subquery
SELECT cust_id,order_date
FROM Orders
WHERE order_num IN
    (SELECT order_num
     FROM OrderItems
     WHERE prod_id = 'BR01' )
ORDER BY order_date;

# Cách 2: JOIN bảng
SELECT b.cust_id, b.order_date
FROM OrderItems a,Orders b
WHERE a.order_num = b.order_num AND a.prod_id = 'BR01'
ORDER BY order_date
```

### Trả về Email của tất cả khách hàng đã mua sản phẩm có prod_id là BR01 (Phần 1)

Bạn muốn biết ngày đặt hàng sản phẩm BR01, có bảng `OrderItems` đại diện cho bảng sản phẩm đơn hàng, `prod_id` là ID sản phẩm; Bảng `Orders` đại diện cho bảng đơn hàng có `cust_id` đại diện cho ID khách hàng và ngày đặt hàng `order_date`; Bảng `Customers` chứa Email khách hàng `cust_email` và ID khách hàng `cust_id`

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

Bảng `Customers` đại diện cho thông tin khách hàng, `cust_id` là ID khách hàng, `cust_email` là Email khách hàng

| cust_id | cust_email        |
| ------- | ----------------- |
| cust10  | <cust10@cust.com> |
| cust1   | <cust1@cust.com>  |
| cust2   | <cust2@cust.com>  |

【Câu hỏi】Trả về email (`cust_email` trong bảng `Customers`) của tất cả khách hàng đã mua sản phẩm có `prod_id` là `BR01`, kết quả không cần sắp xếp.

Gợi ý: Điều này liên quan đến câu lệnh `SELECT`, tầng trong cùng trả về `order_num` từ bảng `OrderItems`, tầng ở giữa trả về `cust_id` từ bảng `Orders`.

Đáp án:

```sql
# Cách 1: Subquery
SELECT cust_email
FROM Customers
WHERE cust_id IN (SELECT cust_id
    FROM Orders
    WHERE order_num IN (SELECT order_num
        FROM OrderItems
        WHERE prod_id = 'BR01'))

# Cách 2: JOIN bảng (INNER JOIN)
SELECT c.cust_email
FROM OrderItems a,Orders b,Customers c
WHERE a.order_num = b.order_num AND b.cust_id = c.cust_id AND a.prod_id = 'BR01'

# Cách 3: JOIN bảng (LEFT JOIN)
SELECT c.cust_email
FROM Orders a LEFT JOIN
  OrderItems b ON a.order_num = b.order_num LEFT JOIN
  Customers c ON a.cust_id = c.cust_id
WHERE b.prod_id = 'BR01'
```

### Trả về tổng số tiền của các đơn hàng khác nhau của từng khách hàng

Chúng ta cần một danh sách ID khách hàng, chứa tổng số tiền mà họ đã đặt hàng.

Bảng `OrderItems` đại diện cho thông tin đơn hàng, bảng `OrderItems` có số đơn hàng: `order_num` và giá bán sản phẩm: `item_price`, số lượng sản phẩm: `quantity`.

| order_num | item_price | quantity |
| --------- | ---------- | -------- |
| a0001     | 10         | 105      |
| a0002     | 1          | 1100     |
| a0002     | 1          | 200      |
| a0013     | 2          | 1121     |
| a0003     | 5          | 10       |
| a0003     | 1          | 19       |
| a0003     | 7          | 5        |

Bảng `Orders` có số đơn hàng: `order_num`, ID khách hàng: `cust_id`

| order_num | cust_id |
| --------- | ------- |
| a0001     | cust10  |
| a0002     | cust1   |
| a0003     | cust1   |
| a0013     | cust2   |

【Câu hỏi】

Viết câu lệnh SQL, trả về ID khách hàng (`cust_id` trong bảng `Orders`), và sử dụng Subquery để trả về `total_ordered` nhằm trả về tổng giá trị đơn hàng của từng khách hàng, sắp xếp kết quả theo số tiền từ lớn đến nhỏ.

Đáp án:

```sql
# Cách 1: Subquery
SELECT o.cust_id, SUM(tb.total_ordered) AS `total_ordered`
FROM (SELECT order_num, SUM(item_price * quantity) AS total_ordered
    FROM OrderItems
    GROUP BY order_num) AS tb,
  Orders o
WHERE tb.order_num = o.order_num
GROUP BY o.cust_id
ORDER BY total_ordered DESC;

# Cách 2: JOIN bảng
SELECT b.cust_id, Sum(a.quantity * a.item_price) AS total_ordered
FROM OrderItems a,Orders b
WHERE a.order_num = b.order_num
GROUP BY cust_id
ORDER BY total_ordered DESC
```

Về giới thiệu chi tiết cho cách viết 1 có thể tham khảo: [issue#2402：写法 1 存在的错误以及修改方法](https://github.com/Snailclimb/JavaGuide/issues/2402).

### Truy xuất tất cả tên sản phẩm和tổng số lượng bán tương ứng từ bảng Products

Truy xuất tất cả tên sản phẩm: `prod_name`, ID sản phẩm: `prod_id` trong bảng `Products`

| prod_id | prod_name |
| ------- | --------- |
| a0001   | egg       |
| a0002   | sockets   |
| a0013   | coffee    |
| a0003   | cola      |

`OrderItems` đại diện cho bảng sản phẩm đơn hàng, sản phẩm đơn hàng: `prod_id`, số lượng bán: `quantity`

| prod_id | quantity |
| ------- | -------- |
| a0001   | 105      |
| a0002   | 1100     |
| a0002   | 200      |
| a0013   | 1121     |
| a0003   | 10       |
| a0003   | 19       |
| a0003   | 5        |

【Câu hỏi】

Viết câu lệnh SQL, truy xuất tất cả tên sản phẩm (`prod_name`) từ bảng `Products`, cũng như column tính toán có tên `quant_sold`, trong đó chứa tổng số sản phẩm đã bán (sử dụng Subquery và `SUM(quantity)` trên bảng `OrderItems` để truy xuất).

Đáp án:

```sql
# Cách 1: Subquery
SELECT p.prod_name, tb.quant_sold
FROM (SELECT prod_id, Sum(quantity) AS quant_sold
    FROM OrderItems
    GROUP BY prod_id) AS tb,
  Products p
WHERE tb.prod_id = p.prod_id

# Cách 2: JOIN bảng
SELECT p.prod_name, Sum(o.quantity) AS quant_sold
FROM Products p,
  OrderItems o
WHERE p.prod_id = o.prod_id
GROUP BY p.prod_name
```

## JOIN bảng

JOIN có nghĩa là "kết nối/liên kết". Đúng như tên gọi, mệnh đề SQL JOIN được dùng để liên kết hai hoặc nhiều bảng lại với nhau để thực hiện truy vấn.

Khi JOIN bảng, cần chọn một field trong mỗi bảng và so sánh giá trị của các field này, hai bản ghi có giá trị giống nhau sẽ gộp thành một. **Bản chất của việc JOIN bảng là gộp các bản ghi của các bảng khác nhau lại để tạo thành một bảng mới. Tất nhiên bảng mới này chỉ là tạm thời, nó chỉ tồn tại trong thời gian thực thi câu truy vấn này**.

Cú pháp cơ bản khi sử dụng `JOIN` để kết nối 2 bảng như sau:

```sql
SELECT table1.column1, table2.column2...
FROM table1
JOIN table2
ON table1.common_column1 = table2.common_column2;
```

`table1.common_column1 = table2.common_column2` là điều kiện JOIN, chỉ các bản ghi thỏa mãn điều kiện này mới được gộp thành một dòng. Bạn có thể sử dụng nhiều toán tử để JOIN bảng, ví dụ =, >, <, <>, <=, >=, !=, `between`, `like` hoặc `not`, nhưng phổ biến nhất là sử dụng =.

Khi hai bảng có các field cùng tên, để giúp database engine phân biệt đó là field của bảng nào, khi viết tên field cần thêm tên bảng phía trước. Tất nhiên nếu tên field được viết là duy nhất trong hai bảng thì cũng có thể không cần dùng định dạng trên, chỉ cần viết tên field.

Ngoài ra, nếu field liên kết của hai bảng có tên giống nhau, cũng có thể sử dụng mệnh đề `USING` để thay thế cho `ON`, ví dụ:

```sql
# join....on
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
ON c.cust_id = o.cust_id
ORDER BY c.cust_name

# Nếu 2 bảng có tên field liên kết giống nhau, cũng có thể dùng USING: JOIN....USING()
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
USING(cust_id)
ORDER BY c.cust_name
```

**Điểm khác biệt giữa `ON` và `WHERE`**:

- Khi JOIN bảng, SQL sẽ tạo ra một bảng tạm mới dựa trên điều kiện JOIN. `ON` chính là điều kiện JOIN, nó quyết định việc tạo ra bảng tạm.
- `WHERE` là sau khi bảng tạm đã được tạo ra, mới tiếp tục lọc dữ liệu trong bảng tạm để tạo ra tập kết quả cuối cùng, lúc này không còn JOIN-ON nữa.

Tóm lại: **SQL đầu tiên dựa vào ON để tạo ra một bảng tạm, sau đó mới dựa vào WHERE để lọc bảng tạm đó**.

SQL cho phép thêm một số từ khóa bổ trợ vào bên trái `JOIN`, từ đó tạo thành các loại JOIN khác nhau như bảng dưới đây:

| Loại JOIN | Giải thích |
| ---------------------------------------- | --------------------------------------------------------------------------------------------- |
| INNER JOIN 内连接 | (Phương thức JOIN mặc định) Chỉ khi cả 2 bảng đều tồn tại bản ghi thỏa mãn điều kiện mới trả về dòng. |
| LEFT JOIN / LEFT OUTER JOIN 左(外)连接 | Trả về tất cả các dòng trong bảng bên trái, ngay cả khi bảng bên phải không có dòng thỏa mãn điều kiện. |
| RIGHT JOIN / RIGHT OUTER JOIN 右(外)连接 | Trả về tất cả các dòng trong bảng bên phải, ngay cả khi bảng bên trái không có dòng thỏa mãn điều kiện. |
| FULL JOIN / FULL OUTER JOIN 全(外)连接 | Chỉ cần 1 trong các bảng tồn tại bản ghi thỏa mãn điều kiện là trả về dòng. |
| SELF JOIN | JOIN một bảng với chính nó, giống như bảng đó là 2 bảng khác nhau. Để phân biệt 2 bảng, trong câu SQL cần đặt lại tên (Alias) cho ít nhất 1 bảng. |
| CROSS JOIN | Trả về tích Cartesian (Cartesian Product) của tập bản ghi từ hai hoặc nhiều bảng JOIN. |

Hình dưới đây minh họa 7 cách dùng liên quan đến LEFT JOIN, RIGHT JOIN, INNER JOIN, OUTER JOIN.

![](https://oss.javaguide.cn/github/javaguide/csdn/d1794312b448516831369f869814ab39.png)

Nếu không thêm bất kỳ từ khóa bổ trợ nào, chỉ viết `JOIN`, thì mặc định là `INNER JOIN`

Đối với `INNER JOIN`, còn có một cách viết ẩn gọi là "**Implicit Inner Join**", tức là không có từ khóa `INNER JOIN`, mà sử dụng câu lệnh `WHERE` để thực hiện chức năng Inner Join.

```sql
# Implicit Inner Join (Inner Join ẩn)
SELECT c.cust_name, o.order_num
FROM Customers c,Orders o
WHERE c.cust_id = o.cust_id
ORDER BY c.cust_name

# Explicit Inner Join (Inner Join hiện)
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
USING(cust_id)
ORDER BY c.cust_name;
```

### Trả về tên khách hàng和số đơn hàng liên quan

Bảng `Customers` có các field tên khách hàng `cust_name`, ID khách hàng `cust_id`

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

Bảng thông tin đơn hàng `Orders`, chứa các field số đơn hàng `order_num`, ID khách hàng `cust_id`

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

【Câu hỏi】Viết câu lệnh SQL, trả về tên khách hàng (`cust_name`) trong bảng `Customers` và số đơn hàng (`order_num`) liên quan trong bảng `Orders`, và sắp xếp kết quả tăng dần theo tên khách hàng rồi đến số đơn hàng. Bạn có thể thử 2 cách viết khác nhau, một cách dùng cú pháp bằng đơn giản, cách kia dùng INNER JOIN.

Đáp án:

```sql
# Implicit Inner Join
SELECT c.cust_name, o.order_num
FROM Customers c,Orders o
WHERE c.cust_id = o.cust_id
ORDER BY c.cust_name,o.order_num

# Explicit Inner Join
SELECT c.cust_name, o.order_num
FROM Customers c
INNER JOIN Orders o
USING(cust_id)
ORDER BY c.cust_name,o.order_num;
```

### Trả về tên khách hàng, số đơn hàng liên quan và tổng giá trị của mỗi đơn hàng

Bảng `Customers` có các field, tên khách hàng: `cust_name`, ID khách hàng: `cust_id`

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

Bảng thông tin đơn hàng `Orders`, chứa các field, số đơn hàng: `order_num`, ID khách hàng: `cust_id`

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

Bảng `OrderItems` có các field, số đơn hàng sản phẩm: `order_num`, số lượng sản phẩm: `quantity`, giá sản phẩm: `item_price`

| order_num | quantity | item_price |
| --------- | -------- | ---------- |
| a1        | 1000     | 10         |
| a2        | 200      | 10         |
| a3        | 10       | 15         |
| a4        | 25       | 50         |
| a5        | 15       | 25         |
| a7        | 7        | 7          |

【Câu hỏi】Ngoài việc trả về tên khách hàng (`cust_name` trong bảng `Customers`) và số đơn hàng (`order_num` trong bảng `Orders`), hãy thêm column thứ 3 `OrderTotal` chứa tổng giá trị của mỗi đơn hàng, và sắp xếp kết quả tăng dần theo tên khách hàng rồi đến số đơn hàng.

```sql
# Cú pháp JOIN bằng đơn giản
SELECT c.cust_name, o.order_num, SUM(quantity * item_price) AS OrderTotal
FROM Customers c,Orders o,OrderItems oi
WHERE c.cust_id = o.cust_id AND o.order_num = oi.order_num
GROUP BY c.cust_name, o.order_num
ORDER BY c.cust_name, o.order_num
```

Chú ý, có thể có bạn sẽ viết như thế này:

```sql
SELECT c.cust_name, o.order_num, SUM(quantity * item_price) AS OrderTotal
FROM Customers c,Orders o,OrderItems oi
WHERE c.cust_id = o.cust_id AND o.order_num = oi.order_num
GROUP BY c.cust_name
ORDER BY c.cust_name,o.order_num
```

Điều này là sai! Chỉ gom nhóm trên `cust_name` tuy đúng ý bài toán nhưng không phù hợp với cú pháp `GROUP BY`.

Trong câu lệnh SELECT, nếu không có mệnh đề `GROUP BY`, thì `cust_name`, `order_num` sẽ trả về nhiều giá trị, trong khi `SUM(quantity * item_price)` chỉ trả về 1 giá trị. Thông qua `GROUP BY cust_name`, có thể làm cho `cust_name` và `SUM(quantity * item_price)` tương ứng 1:1 với nhau (gom nhóm), do đó tương tự cũng phải gom nhóm đối với `order_num`.

> **Nói một câu ngắn gọn: Các field trong SELECT hoặc là đều phải nằm trong hàm gom nhóm/mệnh đề GROUP BY, hoặc là không gom nhóm.**

### Xác定những đơn hàng nào已购买 prod_id 为 BR01 的产品（二）

Bảng `OrderItems` đại diện cho bảng thông tin sản phẩm đơn hàng, `prod_id` là ID sản phẩm; Bảng `Orders` đại diện cho bảng đơn hàng có `cust_id` đại diện cho ID khách hàng và ngày đặt hàng `order_date`

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

【Câu hỏi】

Viết câu lệnh SQL, sử dụng Subquery để xác định những đơn hàng nào (trong `OrderItems`) đã mua sản phẩm có `prod_id` là "BR01", sau đó từ bảng `Orders` trả về ID khách hàng (`cust_id`) và ngày đặt hàng (`order_date`) tương ứng, sắp xếp kết quả tăng dần theo ngày đặt hàng.

Gợi ý: Lần này sử dụng JOIN và cú pháp JOIN bằng đơn giản.

```sql
# Cách 1: Subquery
SELECT cust_id, order_date
FROM Orders
WHERE order_num IN (SELECT order_num
    FROM OrderItems
    WHERE prod_id = 'BR01')
ORDER BY order_date

# Cách 2: JOIN bảng inner join
SELECT cust_id, order_date
FROM Orders o INNER JOIN
  (SELECT order_num
    FROM OrderItems
    WHERE prod_id = 'BR01') tb ON o.order_num = tb.order_num
ORDER BY order_date

# Cách 3: Phiên bản đơn giản hóa của cách 2
SELECT cust_id, order_date
FROM Orders
INNER JOIN OrderItems USING(order_num)
WHERE OrderItems.prod_id = 'BR01'
ORDER BY order_date
```

### Trả về Email của tất cả khách hàng已购买 prod_id 为 BR01 的产品（二）

Cho bảng `OrderItems` đại diện cho bảng thông tin sản phẩm đơn hàng, `prod_id` là ID sản phẩm; Bảng `Orders` đại diện cho bảng đơn hàng có `cust_id` đại diện cho ID khách hàng và ngày đặt hàng `order_date`; Bảng `Customers` chứa `cust_email` email khách hàng và `cust_id` ID khách hàng

Bảng `OrderItems`:

| prod_id | order_num |
| ------- | --------- |
| BR01    | a0001     |
| BR01    | a0002     |
| BR02    | a0003     |
| BR02    | a0013     |

Bảng `Orders`:

| order_num | cust_id | order_date          |
| --------- | ------- | ------------------- |
| a0001     | cust10  | 2022-01-01 00:00:00 |
| a0002     | cust1   | 2022-01-01 00:01:00 |
| a0003     | cust1   | 2022-01-02 00:00:00 |
| a0013     | cust2   | 2022-01-01 00:20:00 |

Bảng `Customers` đại diện cho thông tin khách hàng, `cust_id` là ID khách hàng, `cust_email` là email khách hàng

| cust_id | cust_email        |
| ------- | ----------------- |
| cust10  | <cust10@cust.com> |
| cust1   | <cust1@cust.com>  |
| cust2   | <cust2@cust.com>  |

【Câu hỏi】Trả về email (`cust_email` trong bảng `Customers`) của tất cả khách hàng đã mua sản phẩm có `prod_id` là BR01, kết quả không cần sắp xếp.

Gợi ý: Liên quan đến câu lệnh `SELECT`, tầng trong cùng trả về `order_num` từ `OrderItems`, tầng giữa trả về `cust_id` từ `Customers`, nhưng bắt buộc phải dùng cú pháp INNER JOIN.

```sql
SELECT cust_email
FROM Customers
INNER JOIN Orders using(cust_id)
INNER JOIN OrderItems using(order_num)
WHERE OrderItems.prod_id = 'BR01'
```

### Một cách khác để xác định khách hàng tốt nhất (Phần 2)

Bảng `OrderItems` đại diện cho thông tin đơn hàng, một cách khác để xác định khách hàng tốt nhất là xem họ đã chi bao nhiêu tiền, bảng `OrderItems` có số đơn hàng `order_num` và `item_price` giá bán sản phẩm, `quantity` số lượng sản phẩm

| order_num | item_price | quantity |
| --------- | ---------- | -------- |
| a1        | 10         | 105      |
| a2        | 1          | 1100     |
| a2        | 1          | 200      |
| a4        | 2          | 1121     |
| a5        | 5          | 10       |
| a2        | 1          | 19       |
| a7        | 7          | 5        |

Bảng `Orders` chứa các field số đơn hàng `order_num`, ID khách hàng `cust_id`

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

Bảng khách hàng `Customers` có các field `cust_id` ID khách hàng, `cust_name` tên khách hàng

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

【Câu hỏi】Viết câu lệnh SQL, trả về tên khách hàng và tổng số tiền của các khách hàng có tổng giá trị đơn hàng từ 1000 trở lên.

Gợi ý: Cần tính tổng (`item_price` nhân với `quantity`). Sắp xếp kết quả theo tổng số tiền, hãy sử dụng cú pháp `INNER JOIN`.

```sql
SELECT cust_name, SUM(item_price * quantity) AS total_price
FROM Customers
INNER JOIN Orders USING(cust_id)
INNER JOIN OrderItems USING(order_num)
GROUP BY cust_name
HAVING total_price >= 1000
ORDER BY total_price
```

## Tạo Advanced JOIN

### Truy xuất tên của từng khách hàng和tất cả các số đơn hàng (Phần 1)

Bảng `Customers` đại diện cho thông tin khách hàng chứa ID khách hàng `cust_id` và tên khách hàng `cust_name`

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |

Bảng `Orders` đại diện cho thông tin đơn hàng chứa số đơn hàng `order_num` và ID khách hàng `cust_id`

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

【Câu hỏi】Sử dụng INNER JOIN viết câu lệnh SQL, truy xuất tên của từng khách hàng (`cust_name` trong bảng `Customers`) và tất cả các số đơn hàng (`order_num` trong bảng `Orders`), cuối cùng trả về sắp xếp tăng dần theo tên khách hàng `cust_name`.

```sql
SELECT cust_name, order_num
FROM Customers
INNER JOIN Orders
USING(cust_id)
ORDER BY cust_name
```

### Truy xuất tên của từng khách hàng和tất cả các số đơn hàng (Phần 2)

Bảng `Orders` đại diện cho thông tin đơn hàng chứa số đơn hàng `order_num` và ID khách hàng `cust_id`

| order_num | cust_id  |
| --------- | -------- |
| a1        | cust10   |
| a2        | cust1    |
| a3        | cust2    |
| a4        | cust22   |
| a5        | cust221  |
| a7        | cust2217 |

Bảng `Customers` đại diện cho thông tin khách hàng chứa ID khách hàng `cust_id` và tên khách hàng `cust_name`

| cust_id  | cust_name |
| -------- | --------- |
| cust10   | andy      |
| cust1    | ben       |
| cust2    | tony      |
| cust22   | tom       |
| cust221  | an        |
| cust2217 | hex       |
| cust40   | ace       |

【Câu hỏi】Truy xuất tên của từng khách hàng (`cust_name` trong bảng `Customers`) và tất cả các số đơn hàng (`order_num` trong bảng `Orders`), liệt kê tất cả khách hàng kể cả khi họ chưa từng đặt đơn hàng nào. Cuối cùng trả về sắp xếp tăng dần theo tên khách hàng `cust_name`.

```sql
SELECT cust_name, order_num
FROM Customers
LEFT JOIN Orders
USING(cust_id)
ORDER BY cust_name
```

### Trả về tên sản phẩm和các số đơn hàng liên quan

Bảng `Products` là bảng thông tin sản phẩm chứa các field `prod_id` ID sản phẩm, `prod_name` tên sản phẩm

| prod_id | prod_name |
| ------- | --------- |
| a0001   | egg       |
| a0002   | sockets   |
| a0013   | coffee    |
| a0003   | cola      |
| a0023   | soda      |

Bảng `OrderItems` là bảng thông tin đơn hàng chứa các field `order_num` số đơn hàng và ID sản phẩm `prod_id`

| prod_id | order_num |
| ------- | --------- |
| a0001   | a105      |
| a0002   | a1100     |
| a0002   | a200      |
| a0013   | a1121     |
| a0003   | a10       |
| a0003   | a19       |
| a0003   | a5        |

【Câu hỏi】Sử dụng Outer Join (LEFT JOIN, RIGHT JOIN, FULL JOIN) để liên kết bảng `Products` và bảng `OrderItems`, trả về danh sách tên sản phẩm (`prod_name`) và các số đơn hàng (`order_num`) liên quan, sắp xếp tăng dần theo tên sản phẩm.

```sql
SELECT prod_name, order_num
FROM Products
LEFT JOIN OrderItems
USING(prod_id)
ORDER BY prod_name
```

### Trả về tên sản phẩm和tổng số đơn hàng của từng sản phẩm

Bảng `Products` là bảng thông tin sản phẩm chứa các field `prod_id` ID sản phẩm, `prod_name` tên sản phẩm

| prod_id | prod_name |
| ------- | --------- |
| a0001   | egg       |
| a0002   | sockets   |
| a0013   | coffee    |
| a0003   | cola      |
| a0023   | soda      |

Bảng `OrderItems` là bảng thông tin đơn hàng chứa các field `order_num` số đơn hàng và ID sản phẩm `prod_id`

| prod_id | order_num |
| ------- | --------- |
| a0001   | a105      |
| a0002   | a1100     |
| a0002   | a200      |
| a0013   | a1121     |
| a0003   | a10       |
| a0003   | a19       |
| a0003   | a5        |

【Câu hỏi】

Sử dụng OUTER JOIN liên kết bảng `Products` và bảng `OrderItems`, trả về tên sản phẩm (`prod_name`) và tổng số đơn hàng của từng sản phẩm (không phải số đơn hàng), sắp xếp tăng dần theo tên sản phẩm.

```sql
SELECT prod_name, COUNT(order_num) AS orders
FROM Products
LEFT JOIN OrderItems
USING(prod_id)
GROUP BY prod_name
ORDER BY prod_name
```

### Liệt kê nhà cung cấp和số lượng sản phẩm họ cung cấp

Cho bảng `Vendors` chứa `vend_id` (ID nhà cung cấp)

| vend_id |
| ------- |
| a0002   |
| a0013   |
| a0003   |
| a0010   |

Cho bảng `Products` chứa `vend_id` (ID nhà cung cấp) và `prod_id` (ID sản phẩm cung cấp)

| vend_id | prod_id              |
| ------- | -------------------- |
| a0001   | egg                  |
| a0002   | prod_id_iphone       |
| a00113  | prod_id_tea          |
| a0003   | prod_id_vivo phone   |
| a0010   | prod_id_huawei phone |

【Câu hỏi】Liệt kê nhà cung cấp (`vend_id` trong bảng `Vendors`) và số lượng sản phẩm họ có thể cung cấp, bao gồm cả nhà cung cấp không có sản phẩm. Bạn cần sử dụng OUTER JOIN và hàm Aggregation COUNT() để tính số lượng từng loại sản phẩm trong bảng `Products`, cuối cùng sắp xếp tăng dần theo vend_id.

Lưu ý: Column `vend_id` hiển thị ở nhiều bảng, do đó mỗi lần tham chiếu đến nó đều cần chỉ định tên bảng đầy đủ.

```sql
SELECT v.vend_id, COUNT(prod_id) AS prod_id
FROM Vendors v
LEFT JOIN Products p
USING(vend_id)
GROUP BY v.vend_id
ORDER BY v.vend_id
```

## Truy vấn kết hợp (Combined Query / UNION)

Toán tử `UNION` kết hợp kết quả của 2 hoặc nhiều truy vấn lại với nhau và tạo ra một tập kết quả chứa các dòng trích xuất từ các truy vấn tham gia `UNION`.

Quy tắc cơ bản của `UNION`:

- Số lượng column và thứ tự column của tất cả các truy vấn phải giống nhau.
- Data type của các column trong bảng liên quan ở mỗi truy vấn phải giống nhau hoặc tương thích.
- Tên column trả về thông thường lấy từ câu truy vấn đầu tiên.

Mặc định, toán tử `UNION` chọn các giá trị khác nhau (loại bỏ trùng lặp). Nếu cho phép các giá trị trùng lặp, hãy sử dụng `UNION ALL`.

```sql
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```

Tên column trong tập kết quả `UNION` luôn bằng tên column trong câu lệnh `SELECT` đầu tiên của `UNION`.

`JOIN` vs `UNION`:

- Các column trong các bảng JOIN có thể khác nhau, nhưng trong `UNION`, số lượng column và thứ tự column của tất cả truy vấn phải giống nhau.
- `UNION` đặt các dòng sau truy vấn lại với nhau (đặt theo chiều dọc), còn `JOIN` đặt các column sau truy vấn lại với nhau (đặt theo chiều ngang), tức tạo thành một tích Cartesian.

### Kết hợp 2 câu lệnh SELECT (Phần 1)

Bảng `OrderItems` chứa thông tin sản phẩm đơn hàng, field `prod_id` đại diện cho ID sản phẩm, `quantity` đại diện cho số lượng sản phẩm

| prod_id | quantity |
| ------- | -------- |
| a0001   | 105      |
| a0002   | 100      |
| a0002   | 200      |
| a0013   | 1121     |
| a0003   | 10       |
| a0003   | 19       |
| a0003   | 5        |
| BNBG    | 10002    |

【Câu hỏi】Kết hợp 2 câu lệnh `SELECT` lại với nhau để truy xuất ID sản phẩm (`prod_id`) và `quantity` từ bảng `OrderItems`. Trong đó, một câu lệnh `SELECT` lọc dòng có số lượng là 100, câu lệnh `SELECT` còn lại lọc sản phẩm có ID bắt đầu bằng BNBG, cuối cùng sắp xếp kết quả tăng dần theo ID sản phẩm.

```sql
SELECT prod_id, quantity
FROM OrderItems
WHERE quantity = 100
UNION
SELECT prod_id, quantity
FROM OrderItems
WHERE prod_id LIKE 'BNBG%'
ORDER BY prod_id;
```

> **Lưu ý**: Khi sử dụng `ORDER BY` trong truy vấn `UNION`, chỉ có thể sử dụng 1 lần sau câu lệnh `SELECT` cuối cùng, nó sẽ sắp xếp trên toàn bộ tập kết quả đã kết hợp.

### Kết hợp 2 câu lệnh SELECT (Phần 2)

Bảng `OrderItems` chứa thông tin sản phẩm đơn hàng, field `prod_id` đại diện cho ID sản phẩm, `quantity` đại diện cho số lượng sản phẩm.

| prod_id | quantity |
| ------- | -------- |
| a0001   | 105      |
| a0002   | 100      |
| a0002   | 200      |
| a0013   | 1121     |
| a0003   | 10       |
| a0003   | 19       |
| a0003   | 5        |
| BNBG    | 10002    |

【Câu hỏi】Kết hợp 2 câu lệnh `SELECT` lại với nhau để truy xuất ID sản phẩm (`prod_id`) và `quantity` từ bảng `OrderItems`. Trong đó một câu lệnh `SELECT` lọc dòng có số lượng 100, câu lệnh `SELECT` còn lại lọc ID sản phẩm bắt đầu bằng BNBG, cuối cùng sắp xếp kết quả tăng dần theo ID sản phẩm. Lưu ý: **Lần này chỉ sử dụng duy nhất một câu lệnh SELECT.**

Đáp án:

Yêu cầu chỉ dùng 1 câu lệnh SELECT, vậy thì dùng `OR` chứ không dùng `UNION` nữa.

```sql
SELECT prod_id, quantity
FROM OrderItems
WHERE quantity = 100 OR prod_id LIKE 'BNBG%'
ORDER BY prod_id;
```

### Kết hợp tên sản phẩm在 Products 表和 tên khách hàng在 Customers 表

Bảng `Products` chứa field `prod_name` đại diện cho tên sản phẩm

| prod_name |
| --------- |
| flower    |
| rice      |
| ring      |
| umbrella  |

Bảng `Customers` đại diện cho thông tin khách hàng, `cust_name` đại diện cho tên khách hàng

| cust_name |
| --------- |
| andy      |
| ben       |
| tony      |
| tom       |
| an        |
| lee       |
| hex       |

【Câu hỏi】Viết câu lệnh SQL, kết hợp tên sản phẩm (`prod_name`) trong bảng `Products` và tên khách hàng (`cust_name`) trong bảng `Customers` rồi trả về, sau đó sắp xếp kết quả tăng dần theo tên sản phẩm.

```sql
# Tên column trong tập kết quả UNION luôn bằng tên column trong câu lệnh SELECT đầu tiên của UNION.
SELECT prod_name
FROM Products
UNION
SELECT cust_name
FROM Customers
ORDER BY prod_name
```

### Kiểm tra câu lệnh SQL

Bảng `Customers` chứa các field `cust_name` tên khách hàng, `cust_contact` người liên hệ, `cust_state` bang khách hàng, `cust_email` email khách hàng

| cust_name | cust_contact | cust_state | cust_email        |
| --------- | ------------ | ---------- | ----------------- |
| cust10    | 8695192      | MI         | <cust10@cust.com> |
| cust1     | 8695193      | MI         | <cust1@cust.com>  |
| cust2     | 8695194      | IL         | <cust2@cust.com>  |

【Câu hỏi】Sửa lại câu lệnh SQL bị lỗi dưới đây

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'MI'
ORDER BY cust_name;
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'IL'ORDER BY cust_name;
```

Sau khi sửa:

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'MI'
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'IL'
ORDER BY cust_name;
```

Khi sử dụng `UNION` cho truy vấn kết hợp, chỉ được dùng 1 mệnh đề `ORDER BY`, nó bắt buộc phải nằm sau câu lệnh `SELECT` cuối cùng.

Hoặc dùng trực tiếp `OR` để làm:

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state = 'MI' or cust_state = 'IL'
ORDER BY cust_name;
```

<!-- @include: @article-footer.snippet.md -->
