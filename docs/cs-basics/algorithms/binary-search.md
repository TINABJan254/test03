---
title: Tổng hợp câu hỏi phỏng vấn Tìm kiếm nhị phân: Biên trái, Biên phải, Đáp án nhị phân và Java Template
description: Cẩm nang chuyên sâu về Tìm kiếm nhị phân (Binary Search): Nguyên lý tính đơn điệu, kỹ thuật xử lý Biên trái (lowerBound), Biên phải (upperBound), Tìm kiếm nhị phân trên không gian đáp án (Binary Search on Answer) và các bài toán LeetCode kinh điển.
category: Cơ sở máy tính
tag:
  - Thuật toán
  - LeetCode
head:
  - - meta
    - name: keywords
      content: Tìm kiếm nhị phân, Binary Search, Biên trái, Biên phải, lowerBound, upperBound, Đáp án nhị phân, Binary Search on Answer, Java Binary Search, LeetCode Binary Search
---

Điểm dễ gây mất điểm nhất của **Tìm kiếm nhị phân (Binary Search)** không nằm ở mặt ý tưởng, mà nằm ở **Xử lý các điều kiện biên (Boundary Conditions)**.

Chỉ cần bạn chưa xác định rõ ràng ý nghĩa của các biến `left`, `right`, `mid`, điều kiện dừng vòng lặp (`left <= right` hay `left < right`), và quy tắc cập nhật cận biên (`mid`, `mid + 1` hay `mid - 1`), bạn sẽ rất dễ viết ra một đoạn code bị rơi vào **Vòng lặp vô tận (Infinite Loop)** hoặc **Bỏ sót đáp án chính xác**.

Khi phỏng vấn, để nhận diện xem một bài toán có thể áp dụng Binary Search hay không, hãy nhớ quy tắc vàng: **Không gian tìm kiếm của bài toán có tồn tại Tính đơn điệu (Monotonicity) hay không?**
Mảng được sắp xếp tăng dần chỉ là trường hợp trực quan nhất của tính đơn điệu; các bài toán tìm Tốc độ tối thiểu, Sức chứa nhỏ nhất, hay Số ngày ngắn nhất đều có thể áp dụng tìm kiếm nhị phân trên chính **Không gian giá trị của đáp án**.

---

## Trọng tâm đánh giá trong phỏng vấn

- Viết chuẩn xác Template tìm kiếm nhị phân cơ bản mà không bị lỗi tràn số nguyên.
- Xử lý mượt mà bài toán tìm **Biên trái (Left Bound)** và **Biên phải (Right Bound)** khi mảng có phần tử trùng lặp.
- Nhận diện và giải quyết thành thạo dạng bài **Tìm kiếm nhị phân trên không gian đáp án (Binary Search on Answer)**.
- Giải thích rành mạch tại sao vòng lặp chắc chắn sẽ kết thúc và không bao giờ bỏ sót nghiệm.
- Phân tích chuẩn xác độ phức tạp thời gian là **$O(\log n)$** và độ phức tạp không gian thường là **$O(1)$**.

---

## Khi nào nên nghĩ tới Tìm kiếm nhị phân?

Đừng bao giờ giới hạn suy nghĩ rằng Binary Search chỉ dùng được khi đề bài cho một "mảng số đã sắp xếp". Cốt lõi của Binary Search là **Tính đơn điệu**:

| Dạng đơn điệu | Ví dụ cụ thể | Cách kiểm tra loại trừ một nửa |
| :--- | :--- | :--- |
| **Mảng đơn điệu** | Tìm `target` trong mảng tăng dần | So sánh `nums[mid]` với `target` để loại bỏ hẳn nửa trái hoặc nửa phải |
| **Đáp án đơn điệu** | Tìm tốc độ ăn chuối tối thiểu, sức chứa thuyền nhỏ nhất | Nếu một giá trị $x$ khả thi, thì mọi giá trị lớn hơn $x$ chắc chắn cũng khả thi (hoặc ngược lại) |

Ví dụ trong bài toán *"Koko ăn chuối (LeetCode 875)"*: Tốc độ ăn càng nhanh thì thời gian ăn hết càng ít, càng dễ hoàn thành trước hạn chót. Ở đây mảng các nải chuối hoàn toàn không cần sắp xếp; tính đơn điệu nằm ở mối quan hệ giữa **Tốc độ ăn** và **Khả năng hoàn thành nhiệm vụ**.

Khi đối diện với một bài toán mới, hãy tự hỏi 3 câu:
1. Đề bài có đang yêu cầu tìm một Vị trí, Điểm ranh giới, hoặc Giá trị khả thi Nhỏ nhất / Lớn nhất hay không?
2. Nếu ta "đoán mò" một đáp án $x$, liệu ta có thể viết một hàm kiểm tra `check(x)` chạy trong thời gian $O(n)$ để xác định $x$ có thỏa mãn hay không?
3. Khi $x$ tăng lên (hoặc giảm đi), tính khả thi của `check(x)` có biến thiên đơn điệu (từ `false` chuyển hẳn sang `true`, hoặc ngược lại) hay không?

Nếu cả 3 câu trả lời đều là **CÓ**, bạn chắc chắn có thể giải bài toán bằng Tìm kiếm nhị phân!

---

## 1. Template Tìm kiếm nhị phân cơ bản (Khoảng đóng `[left, right]`)

Áp dụng khi tìm kiếm một giá trị đích `target` cụ thể trong mảng tăng dần không chứa phần tử trùng lặp:

```java
int binarySearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1; // Khoảng tìm kiếm là [left, right]

    while (left <= right) { // Khi left > right thì khoảng tìm kiếm rỗng -> dừng
        // Tránh tràn số nguyên thay vì dùng (left + right) / 2
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) {
            return mid; // Tìm thấy mục tiêu
        } else if (nums[mid] < target) {
            left = mid + 1; // target nằm ở nửa phải [mid + 1, right]
        } else {
            right = mid - 1; // target nằm ở nửa trái [left, mid - 1]
        }
    }
    return -1; // Không tìm thấy
}
```

> **Bản chất**: Khoảng tìm kiếm là **Khoảng đóng hai đầu `[left, right]`**. Mỗi vị trí trong khoảng đều còn cơ hội là đáp án. Mỗi lần loại trừ ta loại bỏ hoàn toàn `mid`, do đó cập nhật thành `mid + 1` hoặc `mid - 1`. Vòng lặp kết thúc khi khoảng rỗng (`left > right`).

---

## 2. Template Tìm Biên trái (Left Bound / `lowerBound`)

Dùng để tìm **vị trí đầu tiên có giá trị $\ge target$** (phù hợp khi mảng có các phần tử trùng lặp hoặc cần tìm vị trí chèn):

```java
int lowerBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length; // Khoảng tìm kiếm là nửa mở [left, right)

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] >= target) {
            right = mid; // mid vẫn có thể là biên trái hợp lệ, không được trừ 1
        } else {
            left = mid + 1; // Chắc chắn không phải, bỏ hẳn nửa trái
        }
    }
    return left; // Khi vòng lặp dừng, left == right chính là biên trái
}
```

- **Quy ước khoảng**: Sử dụng nửa mở `[left, right)`. `right` khởi tạo bằng `nums.length`.
- Khi `nums[mid] >= target`, vị trí `mid` này rất có thể chính là vị trí xuất hiện đầu tiên của `target` (hoặc một số lớn hơn đầu tiên), vì vậy ta **thu hẹp bờ phải về `right = mid` chứ không được trừ 1**.
- Khi vòng lặp kết thúc, `left == right`. Nếu `left == nums.length`, điều đó có nghĩa mọi phần tử trong mảng đều nhỏ hơn `target`.

---

## 3. Template Tìm Biên phải (Right Bound / `upperBound`)

Dùng để tìm **vị trí cuối cùng có giá trị $\le target$**:

Cách thông minh và ít bị nhầm lẫn nhất là: **Chuyển bài toán tìm biên phải về bài toán tìm biên trái!**
- Vị trí cuối cùng $\le target$ chính là: **(Vị trí đầu tiên $> target$) $- 1$**.

```java
int upperBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left - 1; // Vị trí đầu tiên > target trừ đi 1
}
```

---

## 4. Tìm kiếm nhị phân trên Không gian đáp án (Binary Search on Answer)

Đây là dạng toán xuất hiện dày đặc nhất trong các bài LeetCode Medium/Hard.

### Bài toán tiêu biểu: Koko ăn chuối (LeetCode 875)
> Koko có $n$ đống chuối, mỗi đống có `piles[i]` quả chuối, và có tổng cộng $h$ giờ để ăn hết. Mỗi giờ Koko chọn một đống và ăn tối đa $k$ quả chuối từ đống đó. Hãy tìm tốc độ ăn tối thiểu $k$ để ăn hết tất cả chuối trong vòng $h$ giờ.

```java
public int minEatingSpeed(int[] piles, int h) {
    int left = 1; // Tốc độ ăn tối thiểu là 1 quả/giờ
    int right = 0;
    for (int pile : piles) {
        right = Math.max(right, pile); // Tốc độ tối đa không cần vượt quá đống lớn nhất
    }

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canFinish(piles, h, mid)) {
            right = mid; // Tốc độ mid khả thi, thử tìm tốc độ nhỏ hơn nữa
        } else {
            left = mid + 1; // Tốc độ mid không đủ nhanh, bắt buộc phải tăng tốc
        }
    }
    return left; // left == right chính là tốc độ tối thiểu
}

// Hàm kiểm tra với tốc độ speed thì có ăn hết trong h giờ không
private boolean canFinish(int[] piles, int h, int speed) {
    long hours = 0;
    for (int pile : piles) {
        // Công thức làm tròn lên: (pile + speed - 1) / speed
        hours += (pile + speed - 1) / speed;
    }
    return hours <= h;
}
```

---

## Bảng so sánh 3 loại Tìm kiếm nhị phân

| Dạng bài | Mục tiêu | Điều kiện vòng lặp | Cập nhật khi thỏa mãn | Giá trị trả về |
| :--- | :--- | :--- | :--- | :--- |
| **Cơ bản** | Tìm chính xác chỉ số của `target` | `left <= right` | `return mid` ngay lập tức | Chỉ số hoặc `-1` nếu không có |
| **Biên trái** | Tìm vị trí đầu tiên $\ge target$ | `left < right` | `right = mid` | `left` (trong khoảng $[0, n]$) |
| **Đáp án nhị phân** | Tìm giá trị nghiệm nhỏ nhất khả thi | `left < right` | `right = mid` (nếu `check` đúng) | `left` |

---

## Các lỗi sai kinh điển cần tránh

1. **Tràn số nguyên khi tính `mid`**: Tuyệt đối tránh dùng `(left + right) / 2` vì khi `left + right > Integer.MAX_VALUE` sẽ bị tràn thành số âm. Luôn viết: `left + (right - left) / 2`.
2. **Không phân biệt khoảng đóng và nửa mở**: Nếu dùng `while (left <= right)` thì cập nhật `right = mid - 1`. Nếu dùng `while (left < right)` thì cập nhật `right = mid`. Tránh trộn lẫn hai trường phái.
3. **Bỏ sót trường hợp `target` không tồn tại**: Hàm tìm biên trái luôn trả về một chỉ số `left`. Nếu muốn kiểm tra xem phần tử đó có thực sự bằng `target` không, bạn phải kiểm tra: `left < nums.length && nums[left] == target`.
4. **Tràn số trong hàm `check()`**: Khi cộng dồn thời gian hoặc dung lượng trong hàm `check()`, hãy dùng kiểu `long` thay vì `int`.

## Đề xuất bài tập luyện tập

- [LeetCode 704. Binary Search](https://leetcode.com/problems/binary-search/)
- [LeetCode 35. Search Insert Position](https://leetcode.com/problems/search-insert-position/)
- [LeetCode 34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
- [LeetCode 875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)
- [LeetCode 1011. Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/)

<!-- @include: @article-footer.snippet.md -->
