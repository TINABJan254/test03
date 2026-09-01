---
title: Số tiền trong Java nên dùng long hay BigDecimal?
description: Hướng dẫn lựa chọn kiểu số tiền trong Java: giải thích rõ kịch bản áp dụng giữa việc dùng long lưu trữ đơn vị tiền tệ nhỏ nhất và dùng BigDecimal để tính toán chính xác, cũng như cách làm tròn, tràn số, chuyển đổi đơn vị và thiết kế field trong database.
category: Java
tag:
  - Java cơ bản
  - Tính toán số tiền trong Java
head:
  - - meta
    - name: keywords
      content: kiểu số tiền Java,long lưu số tiền,Long lưu xu xu/xu,BigDecimal tính số tiền,độ chính xác số tiền,làm tròn số tiền,DECIMAL,BIGINT
---

Ở phần bình luận của một bài viết thảo luận về kiểu dữ liệu cho field số tiền, tôi thấy có vài đáp án hoàn toàn khác nhau: Có người kiên quyết dùng `Long` lưu đơn vị xu (cent/đồng nhỏ nhất), có người nói các kịch bản như lãi suất, tỷ giá phải dùng `BigDecimal`, lại có người nhắc đến việc truyền trực tiếp chuỗi trong interface.

Những cách nói này không thảo luận về cùng một vấn đề. Dùng `Long` lưu đơn vị nhỏ nhất là nói về **cách lưu trữ** số tiền; lãi suất và tỷ giá dùng `BigDecimal` là nói về **cách tính toán** số tiền; interface truyền chuỗi thường chỉ là **định dạng truyền dữ liệu**.

Số tiền đã xác định đơn vị nhỏ nhất có thể dùng `long` để lưu trữ. Trong quá trình tính toán cần giữ lại số thập phân, hoặc muốn chỉ định rõ phương thức làm tròn thì sử dụng `BigDecimal`. Hai kiểu dữ liệu này có thể xuất hiện cùng nhau trong cùng một hệ thống.

Phương án Long được đề cập dưới đây đều chỉ việc dùng số nguyên để lưu đơn vị tiền tệ nhỏ nhất. Khi mã Java tham gia tính toán thường dùng kiểu nguyên thủy `long`, khi cần biểu diễn giá trị null mới dùng Wrapper Class `Long`.

| Tiêu chí so sánh | `long` | `BigDecimal` |
| --- | --- | --- |
| **Cách biểu diễn** | Số nguyên với đơn vị nhỏ nhất cố định | Số thập phân có `scale` |
| **Công dụng phổ biến** | Số tiền đơn hàng, số dư, số tiền ghi sổ đã xác định đơn vị nhỏ nhất | Tính toán chiết khấu, thuế phí, lãi suất, tỷ giá |
| **Rủi ro chính** | Nhầm lẫn đơn vị, tràn số âm thầm, khó mở rộng độ chính xác | Phương thức khởi tạo, quy tắc làm tròn, khác biệt `scale` |
| **Kiểu Database phổ biến** | `BIGINT` | `DECIMAL(p, s)` |

## Tại sao số tiền không được dùng double?

`double` và `float` lưu trữ số thực nhị phân. Nhiều số thập phân hữu hạn chữ số, khi chuyển sang nhị phân sẽ biến thành số thập phân vô hạn tuần hoàn, chỉ có thể lấy một giá trị biểu diễn gần đúng nhất.

```java
double a = 1.0;
double b = 0.9;

System.out.println(a - b);
// 0.09999999999999998

System.out.println(0.1 + 0.1 + 0.1);
// 0.30000000000000004
```

Sai số này không liên quan đến bản thực thi Java, bất kỳ ngôn ngữ nào áp dụng số thực nhị phân IEEE 754 cũng đều gặp phải. Tính toán số tiền thông thường yêu cầu kết quả tuân theo độ chính xác thập phân và quy tắc làm tròn rõ ràng, giá trị gần đúng rất khó đáp ứng yêu cầu này.

`new BigDecimal(0.1)` có thể hiển thị đầy đủ giá trị gần đúng lưu trong `double`:

```java
System.out.println(new BigDecimal(0.1));
// 0.1000000000000000055511151231257827021181583404541015625
```

Đây cũng là lý do tại sao đối tượng số tiền không nên khởi tạo từ `double`. Một giá trị vốn đã sinh ra sai số, khi chuyển sang `BigDecimal` cũng sẽ không tự động khôi phục lại thành số thập phân ban đầu.

## Những loại số tiền nào thích hợp dùng Long?

Nếu quy định nghiệp vụ về số tiền Việt Nam Đồng / RMB được quy đổi chuẩn xác về đơn vị nhỏ nhất (ví dụ xu/đồng), thì `19.99`元 có thể lưu thành `1999`分 (xu). Các phép cộng trừ đều hoàn thành trên số nguyên, không sinh ra sai số số thập phân.

```java
long priceCents = 1_999L;
long shippingCents = 500L;
long totalCents = Math.addExact(priceCents, shippingCents);
```

`long` thích hợp cho các giá trị đã hoàn thành làm tròn như số tiền đơn hàng, số dư tài khoản, số tiền thanh toán, trong database có thể dùng `BIGINT`.

Khi lưu trữ trong database, tên field tốt nhất nên kèm theo đơn vị để nhìn trực quan hơn:

```sql
CREATE TABLE orders (
    id           BIGINT PRIMARY KEY,
    amount_cents BIGINT NOT NULL
);
```

Nếu dùng `amount`, `amount = 100` rốt cuộc biểu thị 100 VNĐ hay 100 xu, chỉ nhìn số thì không thể đánh giá. Đổi thành `amount_cents = 100` thì hoàn toàn khác.

**Cần lưu ý gì khi sử dụng long?**

Việc lưu đồng nhất số tiền theo đơn vị xu làm cố định độ chính xác ở 2 chữ số thập phân. Tỷ giá, lãi suất, thuế phí hoặc kết quả trung gian tính theo lưu lượng có thể cần 4, 6 hoặc nhiều chữ số thập phân hơn, những tính toán này không thể tiếp tục dùng "xu" để tính cứng.

Còn phải phòng ngừa tràn số (overflow). Phép `+` và `*` thông thường khi tràn số sẽ không báo lỗi, code số tiền nên chuyển sang dùng `Math.addExact()`, `Math.subtractExact()` và `Math.multiplyExact()`:

```java
long subtotalCents = Math.multiplyExact(unitPriceCents, quantity);
long balanceCents = Math.subtractExact(currentBalanceCents, paymentCents);
```

Phép nhân còn phải kiểm tra kết quả trung gian. Số tiền cuối cùng không vượt quá `Long.MAX_VALUE` không có nghĩa là một bước trung gian nào đó như `Đơn giá × Số lượng × Hệ số` cũng không bị tràn số.

Hệ thống đa tiền tệ cũng không thể giả định tất cả loại tiền đều có 2 chữ số thập phân. Số tiền ít nhất phải đi kèm với loại tiền tệ, số chữ số đơn vị nhỏ nhất do loại tiền tệ hoặc quy tắc nghiệp vụ quyết định, không thể suy ra từ một giá trị `long` đơn độc.

## Những loại số tiền nào thích hợp dùng BigDecimal?

Chiết khấu, thuế phí, lãi suất và đổi tỷ giá thường sinh ra kết quả trung gian vượt quá đơn vị nhỏ nhất của tiền tệ.

`BigDecimal` dùng số nguyên độ chính xác tùy ý và `scale` để biểu diễn số thập phân, có thể giữ lại các giá trị trung gian này, sau đó mới làm tròn ở vị trí quy định của nghiệp vụ.

```java
BigDecimal price = new BigDecimal("19.99");
BigDecimal discountRate = new BigDecimal("0.95");

BigDecimal discountedPrice = price.multiply(discountRate);
// 18.9905
```

Hằng số số tiền nên trực tiếp dùng chuỗi (String) để khởi tạo. Interface truyền sang là chuỗi thì chuyển trực tiếp thành `BigDecimal`, field database là `DECIMAL` thì ánh xạ trực tiếp thành `BigDecimal`, ở giữa không cần chuyển sang `double`.

**Nếu `divide()` không chia hết thì làm thế nào?**

Lúc này cần chỉ định số chữ số giữ lại và phương thức làm tròn.

Đoạn code bên dưới có nghĩa là giữ lại 2 chữ số thập phân và dùng `HALF_UP` (làm tròn 4 làm tròn xuống, 5 làm tròn lên). Nếu gọi trực tiếp `a.divide(b)`, chương trình sẽ throw `ArithmeticException`.

```java
BigDecimal a = new BigDecimal("10");
BigDecimal b = new BigDecimal("3");

System.out.println(a.divide(b, 2, RoundingMode.HALF_UP)); // 3.33
```

Còn một điểm cần lưu ý: `BigDecimal` là immutable class, kết quả tính toán phải dùng biến mới để nhận, hoặc gán lại giá trị. Lần gọi `add()` đầu tiên nếu không nhận giá trị trả về thì `amount` vẫn là `10.00`:

```java
BigDecimal amount = new BigDecimal("10.00");

amount.add(new BigDecimal("2.00"));
System.out.println(amount); // Vẫn là 10.00

amount = amount.add(new BigDecimal("2.00"));
System.out.println(amount); // 12.00
```

So sánh kích thước số tiền thông thường dùng `compareTo()`. `equals()` còn so sánh cả `scale`, do đó `1.0` và `1.00` khi gọi `equals()` kết quả là `false`:

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("1.00");

System.out.println(a.equals(b));         // false
System.out.println(a.compareTo(b) == 0); // true
```

Sự khác biệt này cũng ảnh hưởng đến `HashMap` và `HashSet`. Nếu dùng `BigDecimal` làm Key, tốt nhất nên thống nhất `scale` trước, nếu không `1.0` và `1.00` sẽ bị coi là 2 Key khác nhau.

## Long và BigDecimal có thể dùng chung với nhau không?

Có thể. Lấy ví dụ đơn giá sản phẩm `19.99`元, mua 3 món, chiết khấu `0.95`, khi tính toán sử dụng `BigDecimal`, số tiền thanh toán cuối cùng lại chuyển thành `long` với đơn vị xu:

```java
BigDecimal unitPrice = new BigDecimal("19.99");
BigDecimal discountRate = new BigDecimal("0.95");
long quantity = 3L;

BigDecimal payable = unitPrice
        .multiply(BigDecimal.valueOf(quantity))
        .multiply(discountRate)
        .setScale(2, RoundingMode.HALF_UP);

long payableCents = payable
        .movePointRight(2)
        .longValueExact();
```

Đoạn code này tính ra `payable` là `56.97`, `movePointRight(2)` chuyển nó thành `5697`. `longValueExact()` chỉ chấp nhận số nguyên trong phạm vi `long`, chỉ cần còn phần thập phân khác 0 hoặc giá trị vượt giới hạn thì sẽ throw `ArithmeticException`.

Chuyển đổi số tiền đừng dùng trực tiếp `longValue()`, nó sẽ bỏ mất phần thập phân:

```java
long amount = new BigDecimal("19.99").longValue();
System.out.println(amount); // 19
```

Nếu số tiền đầu vào tối đa chỉ có 2 chữ số thập phân, còn có thể dùng `RoundingMode.UNNECESSARY` để validate:

```java
public static long toCentsExact(BigDecimal amount) {
    return amount
            .setScale(2, RoundingMode.UNNECESSARY)
            .movePointRight(2)
            .longValueExact();
}

public static BigDecimal fromCents(long cents) {
    return BigDecimal.valueOf(cents, 2);
}
```

`19.9` có thể bù thành `19.90`, còn `19.999` sẽ trực tiếp throw ngoại lệ chứ không âm thầm cắt bỏ hay làm tròn.

Interface thanh toán nhận đơn vị xu thì truyền `5697`; nhận chuỗi đơn vị 元/đồng thì truyền `payable.toPlainString()`. Code chuyển đổi đặt ở lớp Interface Adapter, trong quá trình tính toán nghiệp vụ không nên chuyển đổi qua lại giữa các kiểu và đơn vị.

## Database nên dùng BIGINT hay DECIMAL?

Trong Java nếu dùng `long` lưu đơn vị nhỏ nhất, field database thông thường dùng `BIGINT`; trong Java dùng `BigDecimal`, field database thông thường dùng `DECIMAL(p, s)`.

```sql
CREATE TABLE settlement_detail (
    id               BIGINT PRIMARY KEY,
    payable_cents    BIGINT        NOT NULL,
    exchange_rate    DECIMAL(18, 8) NOT NULL,
    settlement_amount DECIMAL(18, 2) NOT NULL
);
```

MySQL xếp cả số nguyên và `DECIMAL` vào kiểu giá trị chính xác. Trong `DECIMAL(18, 2)`, `18` là tổng số chữ số có hiệu lực, `2` là số chữ số thập phân; nó có bao phủ được số tiền nghiệp vụ hay không phải tính ngược từ giá trị tối đa, không thể thấy field số tiền là áp chung một độ chính xác.

Đừng phụ thuộc vào việc MySQL tự động làm tròn khi ghi `DECIMAL`. Code Java nên gọi `setScale()` để làm tròn hoặc validate trước, rồi mới ghi kết quả vào database, như vậy giá trị lưu DB và kết quả tính toán của chương trình mới khớp nhau.

Field số tiền có dùng `DEFAULT 0` hay không phải xem ý nghĩa nghiệp vụ. Việc thiếu số tiền và số tiền bằng 0 không phải lúc nào cũng là một chuyện, tùy tiện thêm giá trị mặc định có thể che giấu việc thiếu dữ liệu truyền vào. `NOT NULL` thông thường nên giữ lại, còn giá trị mặc định nên do quy tắc domain quyết định.

## Tóm tắt

Số tiền đã hoàn thành làm tròn và cố định đơn vị nhỏ nhất thích hợp dùng `long` để lưu trữ; các tính toán cần giữ lại số thập phân như chiết khấu, thuế phí, lãi suất và tỷ giá nên dùng `BigDecimal`. Khi làm tròn phải viết rõ số chữ số giữ lại và `RoundingMode`.

Hai kiểu dữ liệu có thể dùng chung. Giai đoạn tính toán giữ lại `BigDecimal`, sau khi xác định số tiền cuối cùng, tiến hành dịch chuyển dấu chấm thập phân theo số chữ số đơn vị nhỏ nhất trước, rồi dùng `longValueExact()` chuyển thành số nguyên. Tên field phải viết rõ đơn vị, tính toán số nguyên phải kiểm tra tràn số, chuyển đổi kiểu dữ liệu và đơn vị nên tập trung đặt ở mã nguồn adapter interface hoặc database.

<!-- @include: @article-footer.snippet.md -->
