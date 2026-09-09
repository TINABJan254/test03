---
title: Tổng hợp câu hỏi phỏng vấn Bảng băm: Xung đột băm, Rehash và Java HashMap
description: Cẩm nang tổng hợp câu hỏi phỏng vấn về Bảng băm (Hash Table), phân tích chuyên sâu về Hàm băm, Xung đột băm, Separate Chaining, Open Addressing, Hệ số tải (Load Factor), Rehash, cấu trúc Java HashMap và các bài toán thuật toán tần suất cao.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Bảng băm, Hash Table, HashMap, Hàm băm, Xung đột băm, Separate Chaining, Open Addressing, Load Factor, Rehash, Java Collections, Câu hỏi phỏng vấn Cấu trúc dữ liệu
---

Bảng băm (**Hash Table** hay **Bảng phân tán**) có giá trị vô cùng to lớn trong các buổi phỏng vấn kỹ thuật, bởi vì nó vừa gắn liền với các thuật toán tìm kiếm/đếm tần suất nhanh chóng, vừa là nền tảng cốt lõi của `HashMap` trong Java, các hệ thống Cache, khử trùng lặp dữ liệu và phân mảnh định tuyến (Sharding) trong hệ thống phân tán.

Bản chất cốt lõi của bảng băm là: **Làm sao để ánh xạ một Key bất kỳ sang chỉ số mảng (Index) trong thời gian cực nhanh, đồng thời vẫn duy trì hiệu suất truy vấn ổn định khi xảy ra xung đột, mở rộng dung lượng và đối mặt với các tập dữ liệu bất lợi.**

Tổng quan nội dung bài viết:
1. Bảng băm là gì?
2. Bảng băm định vị từ Key sang chỉ số mảng như thế nào?
3. Xung đột băm (Hash Collision), Hệ số tải (Load Factor) và Rehash giải quyết những bài toán nào?
4. Mối liên hệ giữa Java `HashMap` và Bảng băm nguyên lý?
5. Bảng băm được ứng dụng như thế nào trong giải thuật và kiến trúc hệ thống?

![Sơ đồ cấu trúc ánh xạ Key sang chỉ số mảng thông qua hàm băm](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/hash-table.png)

## Bảng băm là gì?

Bảng băm là cấu trúc dữ liệu dùng để lưu trữ các cặp quan hệ **Key - Value** (Khóa - Giá trị). Các khái niệm quen thuộc như `Map`, `Dictionary`, `Associative Array` về bản chất đều có thể được cài đặt dựa trên Bảng băm.

Nếu Key là các số nguyên liên tục, ví dụ mã sinh viên từ `0` đến `999`, ta chỉ cần dùng mảng thông thường là có thể truy cập `students[id]` với thời gian $O(1)$. Nhưng trong thực tế nghiệp vụ, Key rất đa dạng: có thể là chuỗi ký tự, UUID, mã đơn hàng, URL hoặc các đối tượng tùy biến (Custom Objects). Nhiệm vụ của bảng băm là:
1. Thông qua **Hàm băm (Hash Function)** biến đổi các Key thuộc đủ loại kiểu dữ liệu và độ dài khác nhau thành một **Số nguyên (Hash Code)**.
2. Ánh xạ số nguyên đó vào phạm vi **Chỉ số mảng (Index)**.

Có thể nhìn nhận Bảng băm qua 3 tầng cấu trúc:
1. **Mảng (Array / Bucket Array)**: Vị trí lưu trữ dữ liệu thực tế, thường được gọi là các Thùng (Buckets).
2. **Hàm băm (Hash Function)**: Đảm nhận chuyển đổi Key thành giá trị băm.
3. **Chiến lược giải quyết xung đột (Collision Resolution)**: Quyết định cách lưu trữ tiếp theo khi có nhiều Key khác nhau cùng bị ánh xạ vào một Bucket duy nhất.

Vì vậy, bảng băm không phải là "hoàn toàn không cần tìm kiếm", mà nó sử dụng hàm băm để thu hẹp phạm vi tìm kiếm xuống mức tối đa: Ở điều kiện lý tưởng, chỉ cần đúng 1 phép tính là định vị được bucket mục tiêu; khi có xung đột, ta chỉ cần so sánh một số lượng rất ít phần tử bên trong bucket đó.

## Tại sao chúng ta cần Bảng băm?

Giả sử cần kiểm tra xem một URL đã được crawler thu thập hay chưa. Cách trực tiếp nhất là lưu tất cả URL đã cào vào một danh sách, mỗi khi có URL mới lại quét từ đầu đến cuối danh sách. Khi dữ liệu nhỏ thì không vấn đề gì, nhưng khi danh sách lên tới hàng triệu URL, việc quét tuyến tính $O(n)$ sẽ khiến hệ thống bị quá tải hoàn toàn.

Ý tưởng của bảng băm là **Dùng không gian đánh đổi thời gian (Space-time tradeoff)**: Cấp phát một mảng dung lượng vừa đủ, dùng hàm băm phân tán các URL vào các bucket khác nhau. Khi truy vấn, thay vì duyệt toàn bộ danh sách, ta chỉ cần tính mã băm của URL và nhảy thẳng tới bucket tương ứng để kiểm tra trong thời gian $O(1)$.

Đó là lý do bảng băm là lựa chọn số 1 cho các bài toán: Tìm kiếm nhanh, Đếm tần suất, Khử trùng lặp và Đánh chỉ mục Cache. Bảng băm không quan tâm tới quan hệ thứ tự lớn nhỏ giữa các phần tử, mà chỉ tập trung vào câu hỏi: *"Cho trước một Key, làm sao tìm ra Value nhanh nhất có thể?"*.

## Hàm băm (Hash Function) cần giải quyết vấn đề gì?

Mục tiêu của hàm băm không phải là làm cho Key trở nên huyền bí, mà là **phân tán các Key càng đồng đều càng tốt vào các bucket trong mảng**. Một hàm băm tốt cần thỏa mãn 3 tiêu chí:

| Tiêu chí | Ý nghĩa |
| :--- | :--- |
| **Tính ổn định (Deterministic)** | Cùng một Key khi tính toán nhiều lần phải luôn cho ra cùng một giá trị băm |
| **Tốc độ tính toán nhanh** | Bản thân hàm băm phải có chi phí tính toán rất thấp, nếu không sẽ làm mất đi lợi thế thời gian $O(1)$ |
| **Phân bố đồng đều (Uniformity)** | Các Key khác nhau phải được phân tán ngẫu nhiên và đồng đều, giảm thiểu tối đa hiện tượng xung đột băm |

> **Lưu ý quan trọng**: Hàm băm trong cấu trúc dữ liệu và Hàm băm trong Mật mã học (Cryptographic Hash như SHA-256) là hai phạm trù khác nhau. Hàm băm cấu trúc dữ liệu ưu tiên tốc độ và độ phân tán; còn hàm băm mật mã học ưu tiên tính bảo mật, chống va chạm (Collision Resistance) và tính một chiều không thể đảo ngược.

Trong Java, khi sử dụng đối tượng tùy biến làm Key trong `HashMap`, hai phương thức `hashCode()` và `equals()` bắt buộc phải tuân thủ nghiêm ngặt giao ước (Contract):
- Nếu hai đối tượng `equals()` bằng nhau thì `hashCode()` của chúng **bắt buộc phải bằng nhau**.
- Nếu hai đối tượng có `hashCode()` bằng nhau thì chúng **chưa chắc đã `equals()` bằng nhau** (đây chính là nguyên nhân dẫn tới xung đột băm).

---

## Bảng băm hoạt động như thế nào?

Khi thực hiện chèn một cặp Key-Value (`put`), bảng băm thực hiện các bước sau:
1. Tính giá trị băm của Key: `hash = hash(key)`.
2. Ánh xạ giá trị băm thành chỉ số mảng: `index = hash & (table.length - 1)` (khi kích thước mảng là lũy thừa của 2).
3. Nếu vị trí `table[index]` đang rỗng (null), tạo node mới và đặt trực tiếp vào vị trí đó.
4. Nếu vị trí `table[index]` đã có dữ liệu (xảy ra xung đột), xử lý theo chiến lược giải quyết xung đột (Separate Chaining hoặc Open Addressing).

```java
int index = hash(key) & (table.length - 1);
```

Trong Java `HashMap`, hàm `hash(key)` không dùng trực tiếp mã `hashCode()` gốc của Object mà thực hiện thêm một phép dịch bit và XOR (Perturbation Function):
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
Phép toán này làm cho các bit bậc cao (16 bit đầu) cũng tham gia vào việc tính toán chỉ số ở các bit bậc thấp, giúp giảm thiểu đáng kể xung đột khi dung lượng mảng còn nhỏ.

---

## Các phương pháp giải quyết Xung đột băm (Hash Collision)

| Phương pháp | Ý tưởng thực hiện | Ứng dụng tiêu biểu | Điểm cần lưu ý |
| :--- | :--- | :--- | :--- |
| **Separate Chaining (Nối chuỗi)** | Mỗi ô mảng là đầu của một Danh sách liên kết hoặc Cây | Java `HashMap` | Nếu chuỗi quá dài sẽ làm giảm tốc độ truy vấn |
| **Open Addressing (Địa chỉ mở)** | Khi xung đột, tiếp tục dò tìm ô nhớ trống tiếp theo trong mảng | Python dict, Caching hiệu năng cao | Thao tác xóa phức tạp, nhạy cảm với hệ số tải cao |
| **Re-hashing (Băm kép)** | Khi xung đột, dùng một hàm băm thứ hai để tính bước nhảy | Lý thuyết bảng băm | Tăng thêm chi phí tính toán |

### 1. Phương pháp nối chuỗi (Separate Chaining)
Mỗi bucket trong mảng đóng vai trò là một đầu danh sách liên kết. Khi có xung đột, phần tử mới được thêm vào danh sách liên kết của bucket đó.
- **Ưu điểm**: Dễ cài đặt, thao tác xóa phần tử đơn giản.
- **Cải tiến trong JDK 8+**: Khi độ dài danh sách liên kết tại một bucket vượt quá ngưỡng **8** và dung lượng mảng $\ge 64$, danh sách liên kết sẽ được tự động chuyển đổi thành **Cây đỏ đen (Red-Black Tree)**, giúp hạ độ phức tạp tìm kiếm trong trường hợp xấu nhất từ $O(n)$ xuống $O(\log n)$.

### 2. Phương pháp địa chỉ mở (Open Addressing)
Tất cả các phần tử đều được lưu trực tiếp bên trong mảng, không dùng thêm danh sách liên kết ngoài. Khi vị trí $H_0$ bị chiếm dụng, hệ thống sẽ dò tìm vị trí trống tiếp theo theo các quy tắc:
- **Dò tìm tuyến tính (Linear Probing)**: $H_i = (H_0 + i) \pmod m$
- **Dò tìm bậc hai (Quadratic Probing)**: $H_i = (H_0 + c_1 i + c_2 i^2) \pmod m$
- **Băm kép (Double Hashing)**: $H_i = (H_0 + i \cdot \text{hash}_2(\text{key})) \pmod m$
- **Ưu điểm**: Tính cục bộ bộ nhớ (Memory Locality) tốt, thân thiện với CPU Cache.
- **Nhược điểm**: Hiện tượng tập hợp cụm (Clustering), thao tác xóa phần tử phải dùng cờ đánh dấu (Tombstone) thay vì xóa thật.

---

## Hệ số tải (Load Factor) và Cơ chế Rehash

**Hệ số tải (Load Factor)** phản ánh mức độ lấp đầy của bảng băm:

$$\text{Load Factor} = \frac{\text{Số lượng phần tử hiện có}}{\text{Tổng dung lượng mảng (Capacity)}}$$

Trong Java `HashMap`, giá trị mặc định của `DEFAULT_LOAD_FACTOR` là **`0.75`**.
- Khi số lượng phần tử vượt quá ngưỡng $\text{Threshold} = \text{Capacity} \times \text{Load Factor}$, bảng băm sẽ tự động kích hoạt quá trình **Mở rộng dung lượng (Resize / Rehash)**.
- Dung lượng mảng mới thường được gấp đôi lên ($2 \times \text{Old Capacity}$).
- Toàn bộ các phần tử cũ sẽ được phân bổ lại vào các bucket mới.

> **Tại sao giá trị mặc định lại là 0.75?**  
> `0.75` là sự cân bằng tối ưu giữa **Hiệu suất thời gian** và **Không gian bộ nhớ**. Nếu hệ số tải quá lớn (ví dụ 1.0), bộ nhớ được tiết kiệm nhưng tỷ lệ xung đột băm tăng cao, làm chậm truy vấn. Nếu hệ số tải quá nhỏ (ví dụ 0.5), xung đột rất ít nhưng sẽ lãng phí tới một nửa dung lượng mảng.

---

## Tại sao độ phức tạp của Bảng băm là $O(1)$?

Độ phức tạp $O(1)$ của Bảng băm được hiểu theo **Ý nghĩa kỳ vọng trung bình (Average-case / Expected Time)**, chứ không phải đảm bảo tuyệt đối trong mọi trường hợp xấu nhất.

Khi hàm băm phân phối đều và hệ số tải được kiểm soát tốt, số lượng phần tử trong mỗi bucket chỉ là một hằng số nhỏ. Chi phí tính hash, định vị bucket và so sánh `equals` đều là hằng số.

Tuy nhiên, nếu bị tấn công băm (Hash Collision Attack) khi tất cả các Key đều bị cố ý băm về cùng một bucket, bảng băm dùng Separate Chaining thông thường sẽ bị suy thoái thành một danh sách liên kết đơn với thời gian truy vấn $O(n)$. Cơ chế chuyển đổi sang Red-Black Tree trong JDK 8 chính là giải pháp phòng vệ giúp giới hạn thời gian xấu nhất ở mức $O(\log n)$.

---

## Các câu hỏi phỏng vấn đào sâu về Java HashMap

| Câu hỏi đào sâu | Trọng tâm trả lời |
| :--- | :--- |
| **Tại sao dung lượng `HashMap` luôn là lũy thừa của 2?** | Để có thể thay thế phép chia lấy dư `%` bằng phép toán bit `hash & (length - 1)`, vừa tăng tốc độ tính toán vừa giúp phân tán lại phần tử khi resize cực nhanh (chỉ cần kiểm tra 1 bit mới). |
| **Tại sao JDK 8 đưa vào Red-Black Tree?** | Khi xảy ra xung đột nghiêm trọng (hoặc bị tấn công băm), cây đỏ đen giúp hạ độ phức tạp truy vấn từ $O(n)$ xuống $O(\log n)$. |
| **Tại sao `HashMap` không an toàn trong đa luồng (Thread-unsafe)?** | Nhiều luồng cùng `put` đồng thời có thể gây ghi đè dữ liệu mất mát (Data Overwrite), sai lệch biến đếm `size`, và trong JDK 7 từng gây ra vòng lặp vô tận (Infinite Loop / Deadlock) khi resize. Trong môi trường đa luồng cần dùng `ConcurrentHashMap`. |
| **Khi dùng Object tùy biến làm Key cần lưu ý gì?** | Bắt buộc phải Override đồng thời cả `equals()` và `hashCode()`, và đối tượng Key nên là Bất biến (Immutable) để tránh việc thay đổi thuộc tính làm thay đổi mã hash sau khi đã lưu vào Map. |

---

## Bài toán thuật toán kinh điển

### 1. Two Sum (LeetCode 1)
Sử dụng `HashMap` lưu lại giá trị đã duyệt qua để biến thao tác tìm kiếm số bù `target - nums[i]` từ $O(n)$ thành $O(1)$, đưa tổng thời gian giải thuật từ $O(n^2)$ xuống $O(n)$.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] {map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[] {-1, -1};
}
```

### 2. Subarray Sum Equals K (LeetCode 560 - Mảng tiền tố + Bảng băm)
Dùng `HashMap` lưu tần suất xuất hiện của các giá trị Tiền tố tổng (Prefix Sum) để tìm số lượng mảng con có tổng bằng $k$ chỉ trong một lần duyệt $O(n)$.

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    count.put(0, 1); // Khởi tạo tiền tố tổng rỗng bằng 0

    int sum = 0;
    int ans = 0;
    for (int num : nums) {
        sum += num;
        ans += count.getOrDefault(sum - k, 0);
        count.put(sum, count.getOrDefault(sum, 0) + 1);
    }
    return ans;
}
```

## Đề xuất bài tập luyện tập

- [LeetCode 1. Two Sum](https://leetcode.com/problems/two-sum/)
- [LeetCode 242. Valid Anagram](https://leetcode.com/problems/valid-anagram/)
- [LeetCode 49. Group Anagrams](https://leetcode.com/problems/group-anagrams/)
- [LeetCode 560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)
- [LeetCode 146. LRU Cache](https://leetcode.com/problems/lru-cache/)

<!-- @include: @article-footer.snippet.md -->
