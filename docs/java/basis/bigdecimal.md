---
title: Giải thích chi tiết BigDecimal
description: Phân tích chi tiết cách sử dụng BigDecimal: giải quyết vấn đề mất độ chính xác của số thực, nắm vững phép tính cộng trừ nhân chia, quy tắc làm tròn RoundingMode, phương thức so sánh compareTo, áp dụng cho các kịch bản độ chính xác cao như tính toán tài chính.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: BigDecimal,độ chính xác số thực,phép tính thập phân,chế độ làm tròn RoundingMode,so sánh BigDecimal,tính toán tiền tệ,mất độ chính xác
---

Trong 《Sổ tay phát triển Java của Alibaba》 có đề cập: “Để tránh mất độ chính xác, có thể sử dụng `BigDecimal` để tiến hành tính toán số thực”.

Phép tính số thực thực sự lại có nguy cơ mất độ chính xác sao? Đúng vậy!

Code ví dụ:

```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

**Tại sao khi tính toán số thực `float` hoặc `double` lại có nguy cơ mất độ chính xác?**

Điều này có liên quan lớn đến cơ chế máy tính lưu trữ số thập phân. Chúng ta biết máy tính sử dụng hệ nhị phân, và độ rộng bit khi biểu diễn một số trên máy tính là hữu hạn. Nhiều số thập phân khi chuyển sang nhị phân sẽ bị lặp vô hạn, chỉ có thể làm tròn thành số bit hữu hạn, do đó tồn tại nguy cơ mất độ chính xác. Tuy nhiên, các giá trị có thể biểu diễn dưới dạng số thập phân nhị phân hữu hạn như 0.5, 0.25 có thể được biểu diễn chính xác.

Ví dụ số 0.2 trong hệ thập phân không thể chuyển đổi chính xác thành số nhị phân:

```java
// Quá trình chuyển 0.2 sang số nhị phân là liên tục nhân 2 cho đến khi không còn phần thập phân,
// trong quá trình tính toán này, phần nguyên thu được xếp từ trên xuống dưới chính là kết quả nhị phân.
0.2 * 2 = 0.4 -> 0
0.4 * 2 = 0.8 -> 0
0.8 * 2 = 1.6 -> 1
0.6 * 2 = 1.2 -> 1
0.2 * 2 = 0.4 -> 0 (xảy ra lặp lại)
...
```

Về nhiều nội dung hơn về số thực, khuyên bạn nên đọc bài viết [Cơ sở hệ thống máy tính (4) Số thực](http://kaito-kidd.com/2018/08/08/computer-system-float-point/).

## Giới thiệu BigDecimal

`BigDecimal` có thể biểu diễn chính xác số thập phân, và cung cấp các phép toán có thể chỉ định rõ độ chính xác và quy tắc làm tròn. Tuy nhiên, khi sử dụng `MathContext` với độ chính xác hữu hạn, thực hiện phép chia cần làm tròn, hoặc chuyển đổi kết quả sang `float`, `double` thì vẫn có thể xảy ra làm tròn.

Trong thực tế, hầu hết các kịch bản nghiệp vụ cần kết quả tính toán thập phân chính xác (như các kịch bản liên quan đến tiền bạc) đều được thực hiện thông qua `BigDecimal`.

Trong 《Sổ tay phát triển Java của Alibaba》 có đề cập: **Đánh giá giá trị bằng nhau giữa các số thực, kiểu dữ liệu nguyên thủy không được dùng == để so sánh, kiểu dữ liệu đóng gói Wrapper không được dùng equals để đánh giá.**

![](https://oss.javaguide.cn/javaguide/image-20211213101646884.png)

Nguyên nhân cụ thể đã được giới thiệu chi tiết ở trên rồi nên không nhắc lại ở đây.

Muốn giải quyết vấn đề mất độ chính xác khi tính toán số thực này, có thể dùng trực tiếp `BigDecimal` để định nghĩa giá trị số thập phân, sau đó tiến hành các thao tác tính toán số thập phân là được.

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

System.out.println(x.compareTo(y));// 0
```

## Các phương thức phổ biến của BigDecimal

### Khởi tạo

Khi chúng ta sử dụng `BigDecimal`, để tránh mất độ chính xác, khuyến nghị dùng constructor `BigDecimal(String val)` hoặc phương thức static `BigDecimal.valueOf(double val)` để tạo đối tượng.

《Sổ tay phát triển Java của Alibaba》 cũng có đề cập đến phần nội dung này như hình dưới đây.

![](https://oss.javaguide.cn/javaguide/image-20211213102222601.png)

### Cộng trừ nhân chia

Phương thức `add` dùng để cộng hai đối tượng `BigDecimal`, `subtract` dùng để trừ hai đối tượng `BigDecimal`. Phương thức `multiply` dùng để nhân hai đối tượng `BigDecimal`, `divide` dùng để chia hai đối tượng `BigDecimal`.

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.add(b));// 1.9
System.out.println(a.subtract(b));// 0.1
System.out.println(a.multiply(b));// 0.90
System.out.println(a.divide(b));// Không thể chia hết, throw ArithmeticException
System.out.println(a.divide(b, 2, RoundingMode.HALF_UP));// 1.11
```

Lưu ý ở đây là nên căn cứ vào nghiệp vụ có cho phép làm tròn hay không để chọn overload của `divide`. Khi yêu cầu kết quả chính xác tuyệt đối, có thể dùng phiên bản không chỉ định quy tắc làm tròn; khi kết quả không thể biểu diễn chính xác sẽ throw `ArithmeticException`. Khi cho phép làm tròn, nên chỉ định rõ `scale` và `roundingMode`. `RoundingMode.UNNECESSARY` dùng để khẳng định kết quả không cần làm tròn, nếu thực tế cần làm tròn thì cũng sẽ throw `ArithmeticException`.

```java
public BigDecimal divide(BigDecimal divisor, int scale, RoundingMode roundingMode) {
    return divide(divisor, scale, roundingMode.oldMode);
}
```

Quy tắc giữ lại chữ số thập phân rất nhiều, ở đây liệt kê một số loại:

```java
public enum RoundingMode {
   // 2.4 -> 3 , 1.6 -> 2
   // -1.6 -> -2 , -2.4 -> -3
   UP(BigDecimal.ROUND_UP),
   // 2.4 -> 2 , 1.6 -> 1
   // -1.6 -> -1 , -2.4 -> -2
   DOWN(BigDecimal.ROUND_DOWN),
   // 2.4 -> 3 , 1.6 -> 2
   // -1.6 -> -1 , -2.4 -> -2
   CEILING(BigDecimal.ROUND_CEILING),
   // 2.5 -> 2 , 1.6 -> 1
   // -1.6 -> -2 , -2.5 -> -3
   FLOOR(BigDecimal.ROUND_FLOOR),
   // 2.4 -> 2 , 1.6 -> 2
   // -1.6 -> -2 , -2.4 -> -2
   HALF_UP(BigDecimal.ROUND_HALF_UP),
   //......
}
```

### So sánh lớn nhỏ

`a.compareTo(b)`: Trả về -1 biểu thị `a` nhỏ hơn `b`, 0 biểu thị `a` bằng `b`, 1 biểu thị `a` lớn hơn `b`.

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.compareTo(b));// 1
```

### Giữ lại bao nhiêu chữ số thập phân

Thông qua phương thức `setScale` thiết lập giữ lại bao nhiêu chữ số thập phân và quy tắc giữ lại. Quy tắc giữ lại có khá nhiều loại, không cần nhớ, IDEA sẽ gợi ý.

```java
BigDecimal m = new BigDecimal("1.255433");
BigDecimal n = m.setScale(3,RoundingMode.HALF_DOWN);
System.out.println(n);// 1.255
```

## Vấn đề so sánh giá trị bằng nhau của BigDecimal

Trong 《Sổ tay phát triển Java của Alibaba》 có đề cập:

![](https://oss.javaguide.cn/github/javaguide/java/basis/image-20220714161315993.png)

Ví dụ code xảy ra vấn đề khi `BigDecimal` dùng phương thức `equals()` để so sánh bằng giá trị:

```java
BigDecimal a = new BigDecimal("1");
BigDecimal b = new BigDecimal("1.0");
System.out.println(a.equals(b));// false
```

Đó là vì phương thức `equals()` không chỉ so sánh giá trị lớn nhỏ (value) mà còn so sánh cả độ chính xác (scale), trong khi phương thức `compareTo()` khi so sánh sẽ bỏ qua độ chính xác.

Scale của 1.0 là 1, scale của 1 là 0, do đó kết quả của `a.equals(b)` là false.

![](https://oss.javaguide.cn/github/javaguide/java/basis/image-20220714164706390.png)

Phương thức `compareTo()` có thể so sánh giá trị của hai `BigDecimal`, nếu bằng nhau trả về 0, nếu số thứ nhất lớn hơn số thứ hai trả về 1, ngược lại trả về -1.

```java
BigDecimal a = new BigDecimal("1");
BigDecimal b = new BigDecimal("1.0");
System.out.println(a.compareTo(b));// 0
```

## Chia sẻ Class tiện ích BigDecimal

Trên mạng có một utility class `BigDecimal` được khá nhiều người sử dụng, cung cấp nhiều phương thức static để đơn giản hóa các thao tác trên `BigDecimal`.

Tôi đã tiến hành cải tiến nhẹ và chia sẻ mã nguồn bên dưới:

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

/**
 * Utility class nhỏ giúp đơn giản hóa tính toán BigDecimal
 */
public class BigDecimalUtil {

    /**
     * Độ chính xác phép chia mặc định
     */
    private static final int DEF_DIV_SCALE = 10;

    private BigDecimalUtil() {
    }

    /**
     * Sử dụng BigDecimal tiến hành phép cộng, kết quả khi chuyển thành double vẫn có thể xảy ra làm tròn.
     *
     * @param v1 Số bị cộng
     * @param v2 Số cộng
     * @return Tổng hai tham số
     */
    public static double add(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.add(b2).doubleValue();
    }

    /**
     * Sử dụng BigDecimal tiến hành phép trừ, kết quả khi chuyển thành double vẫn có thể xảy ra làm tròn.
     *
     * @param v1 Số bị trừ
     * @param v2 Số trừ
     * @return Hiệu hai tham số
     */
    public static double subtract(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.subtract(b2).doubleValue();
    }

    /**
     * Sử dụng BigDecimal tiến hành phép nhân, kết quả khi chuyển thành double vẫn có thể xảy ra làm tròn.
     *
     * @param v1 Số bị nhân
     * @param v2 Số nhân
     * @return Tích hai tham số
     */
    public static double multiply(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.multiply(b2).doubleValue();
    }

    /**
     * Cung cấp phép chia (tương đối) chính xác, khi xảy ra trường hợp không chia hết thì chính xác tới
     * 10 chữ số sau dấu chấm thập phân, các chữ số sau đó làm tròn theo quy tắc RoundingMode.HALF_EVEN.
     *
     * @param v1 Số bị chia
     * @param v2 Số chia
     * @return Thương hai tham số
     */
    public static double divide(double v1, double v2) {
        return divide(v1, v2, DEF_DIV_SCALE);
    }

    /**
     * Cung cấp phép chia (tương đối) chính xác. Khi xảy ra trường hợp không chia hết, do tham số scale chỉ
     * định độ chính xác, các chữ số sau đó làm tròn theo quy tắc RoundingMode.HALF_EVEN.
     *
     * @param v1    Số bị chia
     * @param v2    Số chia
     * @param scale Biểu thị cần chính xác đến mấy chữ số sau dấu chấm thập phân.
     * @return Thương hai tham số
     */
    public static double divide(double v1, double v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                    "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.divide(b2, scale, RoundingMode.HALF_EVEN).doubleValue();
    }

    /**
     * Sử dụng quy tắc HALF_EVEN xử lý chữ số thập phân chỉ định.
     *
     * @param v     Số cần làm tròn theo quy tắc HALF_EVEN
     * @param scale Giữ lại mấy chữ số sau dấu chấm thập phân
     * @return Kết quả sau khi làm tròn
     */
    public static double round(double v, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                    "The scale must be a positive integer or zero");
        }
        BigDecimal b = BigDecimal.valueOf(v);
        BigDecimal one = new BigDecimal("1");
        return b.divide(one, scale, RoundingMode.HALF_EVEN).doubleValue();
    }

    /**
     * Chuyển thành Float, khi vượt quá độ chính xác hoặc phạm vi float có thể xảy ra làm tròn hoặc tràn số
     *
     * @param v Số cần chuyển đổi
     * @return Trả về kết quả chuyển đổi
     */
    public static float convertToFloat(double v) {
        BigDecimal b = BigDecimal.valueOf(v);
        return b.floatValue();
    }

    /**
     * Chuyển thành Int, không làm tròn; phần thập phân sẽ bị cắt bỏ, vượt quá phạm vi sẽ mất bit cao
     *
     * @param v Số cần chuyển đổi
     * @return Trả về kết quả chuyển đổi
     */
    public static int convertsToInt(double v) {
        BigDecimal b = BigDecimal.valueOf(v);
        return b.intValue();
    }

    /**
     * Chuyển thành Long, không làm tròn; phần thập phân sẽ bị cắt bỏ, vượt quá phạm vi sẽ mất bit cao
     *
     * @param v Số cần chuyển đổi
     * @return Trả về kết quả chuyển đổi
     */
    public static long convertsToLong(double v) {
        BigDecimal b = BigDecimal.valueOf(v);
        return b.longValue();
    }

    /**
     * Trả về giá trị lớn hơn trong hai số
     *
     * @param v1 Số thứ nhất cần so sánh
     * @param v2 Số thứ hai cần so sánh
     * @return Trả về giá trị lớn hơn trong hai số
     */
    public static double returnMax(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.max(b2).doubleValue();
    }

    /**
     * Trả về giá trị nhỏ hơn trong hai số
     *
     * @param v1 Số thứ nhất cần so sánh
     * @param v2 Số thứ hai cần so sánh
     * @return Trả về giá trị nhỏ hơn trong hai số
     */
    public static double returnMin(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.min(b2).doubleValue();
    }

    /**
     * So sánh chính xác hai số
     *
     * @param v1 Số thứ nhất cần so sánh
     * @param v2 Số thứ hai cần so sánh
     * @return Nếu hai số bằng nhau trả về 0, nếu số thứ nhất lớn hơn số thứ hai trả về 1, ngược lại trả về -1
     */
    public static int compareTo(double v1, double v2) {
        BigDecimal b1 = BigDecimal.valueOf(v1);
        BigDecimal b2 = BigDecimal.valueOf(v2);
        return b1.compareTo(b2);
    }

}
```

Issue liên quan: [Gợi ý thiết lập quy tắc giữ lại chữ số là RoundingMode.HALF_EVEN,#2129](https://github.com/Snailclimb/JavaGuide/issues/2129).

![RoundingMode.HALF_EVEN](https://oss.javaguide.cn/github/javaguide/java/basis/RoundingMode.HALF_EVEN.png)

## Tóm tắt

Nhiều số thập phân không thể biểu diễn chính xác bằng số nhị phân có độ rộng bit hữu hạn, do đó khi dùng `float` hoặc `double` để tính toán có nguy cơ mất độ chính xác.

Tuy nhiên, Java cung cấp `BigDecimal` để thao tác với số thực. Bản thực thi của `BigDecimal` sử dụng tới `BigInteger` (dùng để thao tác với số nguyên lớn), điểm khác biệt là `BigDecimal` đưa thêm vào khái niệm vị trí chữ số thập phân (scale).

<!-- @include: @article-footer.snippet.md -->
