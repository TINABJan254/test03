---
title: Chi tiết Bloom Filter (Nguyên lý, Triển khai và Ứng dụng chống Cache Penetration)
description: Phân tích chuyên sâu về Bộ lọc Bloom (Bloom Filter): Nguyên lý mảng bit và nhiều hàm băm, đặc tính nhận định sai (False Positive), cách cài đặt bằng Java, thư viện Google Guava và RedisBloom trong hệ thống phân tán.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
  - Redis
head:
  - - meta
    - name: keywords
      content: Bloom Filter, Mảng bit, BitMap, BitSet, False Positive, Xác suất nhận định sai, Hàm băm, Guava BloomFilter, RedisBloom, Chống Cache Penetration, Khử trùng dữ liệu
---

# Bộ lọc Bloom (Bloom Filter)

**Bloom Filter (Bộ lọc Bloom)** là một cấu trúc dữ liệu xác suất tiết kiệm không gian bộ nhớ tuyệt vời, được phát minh bởi Burton Howard Bloom vào năm 1970.

Bloom Filter được thiết kế chuyên biệt để giải quyết bài toán: **Kiểm tra sự tồn tại của một phần tử trong tập dữ liệu khổng lồ (hàng trăm triệu đến hàng tỷ phần tử) với dung lượng RAM siêu nhỏ và chấp nhận một tỷ lệ sai số nhỏ có thể kiểm soát được.**

Nội dung chính:
1. Bloom Filter là gì?
2. Nguyên lý hoạt động của Bloom Filter.
3. Kịch bản ứng dụng thực tế (Chống Cache Penetration, Lọc thư rác, Khử trùng lặp).
4. Tự tay lập trình Bloom Filter trong Java.
5. Sử dụng Google Guava BloomFilter.
6. Bloom Filter phân tán trong Redis (RedisBloom).

---

## 1. Bloom Filter là gì?

Bloom Filter được cấu tạo từ hai thành phần cơ bản:
1. **Một mảng các bit (Bit Array / BitMap)** có độ dài $m$, ban đầu tất cả các bit đều bằng **`0`**.
2. **Một tập hợp $k$ hàm băm độc lập ($h_1, h_2, \dots, h_k$)**, mỗi hàm băm sẽ ánh xạ một phần tử vào một chỉ số trong khoảng $[0, m - 1]$.

So với các cấu trúc dữ liệu lưu trữ đối tượng thực tế như `HashSet` hay `HashMap`, Bloom Filter **hoàn toàn không lưu trữ bản thân phần tử**, mà nó chỉ lưu trữ các "dấu vết bit" đã được bật lên 1.
- Để lưu trữ **1 triệu phần tử**, Bloom Filter chỉ tiêu tốn khoảng **1.2 MB RAM**!

![Cấu trúc Mảng bit trong Bloom Filter](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/bloom-filter-bit-table.png)

---

## 2. Nguyên lý hoạt động

### 1. Thao tác Thêm một phần tử (Insert / Add)
Khi muốn thêm một phần tử $x$ vào Bloom Filter:
1. Lần lượt đưa $x$ qua $k$ hàm băm để tính toán ra $k$ vị trí chỉ số: $i_1 = h_1(x), i_2 = h_2(x), \dots, i_k = h_k(x)$.
2. Đặt các bit tại các vị trí $i_1, i_2, \dots, i_k$ trong mảng bit thành **`1`** (`bits.set(index, true)`).

### 2. Thao tác Kiểm tra một phần tử (Query / Contains)
Khi muốn kiểm tra phần tử $y$ có tồn tại trong tập hợp hay không:
1. Lần lượt tính $k$ giá trị băm của $y$: $j_1 = h_1(y), j_2 = h_2(y), \dots, j_k = h_k(y)$.
2. Kiểm tra giá trị các bit tại các vị trí đó:
   - **Nếu có ÍT NHẤT MỘT bit bằng `0`**: Chắc chắn 100% phần tử $y$ **CHƯA BAO GIỜ** được thêm vào tập hợp!
   - **Nếu TẤT CẢ $k$ bit đều bằng `1`**: Phần tử $y$ **CÓ THỂ** đã tồn tại trong tập hợp (vẫn có một tỷ lệ nhỏ xảy ra False Positive).

![Sơ đồ nguyên lý hoạt động của Bloom Filter](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/bloom-filter-simple-schematic-diagram.png)

> ### ⭐️ Quy tắc vàng của Bloom Filter:
> - **Nếu Bloom Filter báo KHÔNG TỒN TẠI $\rightarrow$ Chắc chắn 100% KHÔNG TỒN TẠI.**
> - **Nếu Bloom Filter báo CÓ TỒN TẠI $\rightarrow$ Chỉ mang tính xác suất (Có thể có, có thể do trùng lặp bit ngẫu nhiên).**

### Tại sao Bloom Filter khó thực hiện thao tác Xóa (Delete)?
Do nhiều phần tử khác nhau có thể cùng chia sẻ chung một vị trí bit bằng 1 (xung đột băm). Nếu bạn xóa phần tử $A$ bằng cách đặt các bit của nó về `0`, bạn sẽ vô tình làm hỏng dấu vết của phần tử $B$ khác cũng dùng chung bit đó!

*(Để hỗ trợ xóa, người ta phải dùng biến thể Counting Bloom Filter với mỗi vị trí là một bộ đếm số nguyên thay vì 1 bit).*

---

## 3. Kịch bản ứng dụng thực tế

1. **Phòng chống Thủng bộ nhớ đệm (Cache Penetration)**:
   - Kẻ xấu liên tục gửi các request truy vấn các ID không hề tồn tại trong hệ thống nhằm làm sập Database.
   - Đưa toàn bộ các ID hợp lệ vào Bloom Filter đặt trước Cache. Khi request đến: Nếu Bloom Filter báo không tồn tại $\rightarrow$ Lập tức chặn request và trả về lỗi, hoàn toàn không chạm tới Database!
2. **Khử trùng lặp dữ liệu lớn**:
   - Web Crawler kiểm tra hàng tỷ URL đã được cào dữ liệu hay chưa.
3. **Danh sách đen (Blacklist)**:
   - Kiểm tra nhanh số điện thoại spam, email rác, địa chỉ IP độc hại.

---

## 4. Cài đặt Bloom Filter thuần trong Java

```java
import java.util.BitSet;

public class MyBloomFilter {
    private static final int DEFAULT_SIZE = 2 << 24; // 32MB bit array
    private static final int[] SEEDS = new int[]{3, 13, 46, 71, 91, 134};

    private final BitSet bits = new BitSet(DEFAULT_SIZE);
    private final SimpleHash[] funcs = new SimpleHash[SEEDS.length];

    public MyBloomFilter() {
        for (int i = 0; i < SEEDS.length; i++) {
            funcs[i] = new SimpleHash(DEFAULT_SIZE, SEEDS[i]);
        }
    }

    public void add(Object value) {
        for (SimpleHash f : funcs) {
            bits.set(f.hash(value), true);
        }
    }

    public boolean contains(Object value) {
        if (value == null) return false;
        for (SimpleHash f : funcs) {
            if (!bits.get(f.hash(value))) {
                return false; // Chắc chắn 100% không tồn tại
            }
        }
        return true; // Có thể tồn tại
    }

    public static class SimpleHash {
        private final int cap;
        private final int seed;

        public SimpleHash(int cap, int seed) {
            this.cap = cap;
            this.seed = seed;
        }

        public int hash(Object value) {
            int h;
            return (value == null) ? 0 : Math.abs((cap - 1) & seed * ((h = value.hashCode()) ^ (h >>> 16)));
        }
    }
}
```

---

## 5. Sử dụng Google Guava BloomFilter

Trong các ứng dụng Java đơn máy, khuyến nghị sử dụng trực tiếp thư viện **Google Guava**:

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
</dependency>
```

```java
// Tạo Bloom Filter dự kiến chứa 1.000.000 phần tử với tỷ lệ sai số mong muốn là 1% (0.01)
BloomFilter<Integer> filter = BloomFilter.create(
    Funnels.integerFunnel(),
    1000000,
    0.01
);

// Thêm phần tử
filter.put(1001);
filter.put(1002);

// Kiểm tra phần tử
System.out.println(filter.mightContain(1001)); // true
System.out.println(filter.mightContain(9999)); // false
```

---

## 6. Bloom Filter phân tán trong Redis (RedisBloom)

Trong kiến trúc Microservices phân tán, nhiều máy chủ cùng cần chia sẻ một Bloom Filter. Chúng ta sử dụng module **RedisBloom** trong Redis (tích hợp sẵn từ Redis 8):

### Các lệnh CLI thông dụng:
- `BF.RESERVE {key} {error_rate} {capacity}`: Khởi tạo Bloom Filter với sai số và sức chứa dự kiến.
- `BF.ADD {key} {item}`: Thêm một phần tử.
- `BF.EXISTS {key} {item}`: Kiểm tra một phần tử có tồn tại hay không (trả về 1 hoặc 0).
- `BF.MADD` / `BF.MEXISTS`: Thêm / Kiểm tra nhiều phần tử cùng lúc.

```shell
127.0.0.1:6379> BF.RESERVE user_filter 0.01 1000000
OK
127.0.0.1:6379> BF.ADD user_filter "user_1001"
(integer) 1
127.0.0.1:6379> BF.EXISTS user_filter "user_1001"
(integer) 1
127.0.0.1:6379> BF.EXISTS user_filter "user_9999"
(integer) 0
```

<!-- @include: @article-footer.snippet.md -->
