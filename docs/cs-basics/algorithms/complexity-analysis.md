---
title: Cẩm nang phỏng vấn Độ phức tạp thời gian và không gian: Big-O, Đệ quy và các ngộ nhận thường gặp
description: Cẩm nang hướng dẫn toàn diện về phân tích Độ phức tạp thời gian và không gian trong phỏng vấn thuật toán: Ký hiệu Big-O, độ phức tạp vòng lặp, độ phức tạp hàm đệ quy, ước lượng quy mô dữ liệu đầu vào và các cạm bẫy dễ mất điểm.
category: Cơ sở máy tính
tag:
  - Thuật toán
  - Phỏng vấn
head:
  - - meta
    - name: keywords
      content: Độ phức tạp thời gian, Độ phức tạp không gian, Big O, Time Complexity, Space Complexity, Độ phức tạp đệ quy, Đánh giá thuật toán, LeetCode Complexity, Phỏng vấn thuật toán
---

Phân tích độ phức tạp (Complexity Analysis) là cửa ải đầu tiên trong mọi buổi phỏng vấn thuật toán. Người phỏng vấn có thể không yêu cầu bạn phải viết ra những chứng minh toán học khắt khe, nhưng họ luôn kỳ vọng bạn có thể giải thích rành mạch: Đoạn code này chạy qua bao nhiêu vòng lặp, tiêu tốn thêm bao nhiêu bộ nhớ phụ, và khi quy mô dữ liệu đầu vào tăng vọt lên thì hiệu năng hệ thống sẽ biến chuyển ra sao.

Trước hết, hãy làm rõ một ranh giới quan trọng: **Phân tích độ phức tạp nhìn vào xu hướng tăng trưởng khi quy mô đầu vào ($n$) tiến tới vô cùng lớn, chứ không phản ánh thời gian chạy vật lý tuyệt đối (tính bằng mili-giây)**.
Một thuật toán $O(n)$ chưa chắc đã luôn chạy nhanh hơn thuật toán $O(n \log n)$ trên thực tế, bởi vì hệ số hằng số (Constant Factor), quy mô dữ liệu nhỏ, tỷ lệ trúng CPU Cache và chi tiết hiện thực đều tác động trực tiếp tới thời gian chạy. Tuy nhiên, trong phỏng vấn, bạn chỉ cần dùng ký hiệu **Big-O** để nói rõ cấp bậc độ lớn của sự tăng trưởng, kèm theo một câu phân tích về các giới hạn thực tế là đã hoàn toàn đạt chuẩn điểm tối đa.

---

## Trọng tâm đánh giá trong phỏng vấn

- Có khả năng nhìn vào vòng lặp, đệ quy, và các thao tác cấu trúc dữ liệu để xác định chính xác độ phức tạp thời gian.
- Phân biệt rõ ràng giữa **Bộ nhớ phụ trợ sử dụng thêm (Extra Space)** và không gian của chính dữ liệu đầu vào.
- Trình bày mạch lạc độ phức tạp tốt nhất (Best-case), xấu nhất (Worst-case) và trung bình (Average-case) cho từng thuật toán cụ thể.
- Khi gặp mã nguồn đệ quy, biết cách dùng Cây đệ quy (Recursion Tree) hoặc quy mô bài toán con để suy luận.
- Không bao giờ mặc định ngây thơ rằng các thao tác trên `HashMap`, sắp xếp, hay Heap lúc nào cũng là $O(1)$.

---

## Cách trình bày độ phức tạp khi phỏng vấn

Khi trả lời về độ phức tạp, bạn **tuyệt đối không nên chỉ đưa ra một kết luận cộc lốc** (như "Bài này là $O(n)$"). Cách diễn đạt thuyết phục nhất là trình bày theo cấu trúc: **"Code đã thực hiện những thao tác gì $\rightarrow$ Vì vậy độ phức tạp tương ứng là bao nhiêu"**.

Ví dụ với bài toán kinh điển Two Sum (LeetCode 1):

```text
"Thuật toán duyệt qua mảng một lần, mỗi phần tử thực hiện một lần tra cứu và một lần chèn vào HashMap.
Thao tác trên bảng băm trung bình đạt O(1), do đó tổng độ phức tạp thời gian là O(n).
Về bộ nhớ, thuật toán sử dụng thêm một HashMap để lưu trữ ánh xạ từ giá trị phần tử sang chỉ số mảng;
trong trường hợp xấu nhất bảng băm sẽ lưu trữ n phần tử, do đó độ phức tạp không gian là O(n)."
```

Cách diễn đạt này vững chắc hơn nhiều so với việc chỉ nói "thời gian $O(n)$, không gian $O(n)$", vì nó làm nổi bật toàn bộ quá trình tư duy lập luận của bạn. Nếu người phỏng vấn muốn hỏi sâu thêm về trường hợp xấu nhất của bảng băm khi bị xung đột dữ liệu, bạn cũng có sẵn không gian để đối đáp tự tin.

---

## Các cấp bậc độ phức tạp thường gặp

| Độ phức tạp | Kịch bản thường gặp | Ghi chú phỏng vấn |
| :--- | :--- | :--- |
| **$O(1)$** | Truy cập mảng theo chỉ số, thao tác đỉnh Stack, tra cứu trung bình trên HashMap | Bảng băm có thể bị suy thoái trong trường hợp xấu nhất |
| **$O(\log n)$** | Tìm kiếm nhị phân, Sift-Up/Sift-Down trong Heap, truy vấn trên Cây cân bằng | Mỗi bước chia đôi hoặc thu hẹp quy mô bài toán theo tỷ lệ |
| **$O(n)$** | Duyệt đơn mảng, danh sách liên kết, chuỗi ký tự | Cần kiểm tra xem có thực sự chỉ duyệt qua 1 lần hay không |
| **$O(n \log n)$** | Quick Sort trung bình, Merge Sort, Heap Sort | Cấp bậc phổ biến nhất của các thuật toán sắp xếp so sánh |
| **$O(n^2)$** | Vòng lặp lồng nhau 2 lớp, duyệt tất cả các cặp đôi | Trong phỏng vấn luôn phải cảnh giác xem có tối ưu xuống được không |
| **$O(2^n)$** | Liệt kê tập con, một số bài toán nhánh cây quay lui | Không gian tìm kiếm tăng theo hàm mũ |
| **$O(n!)$** | Liệt kê tất cả hoán vị, bài toán người du lịch (TSP) vét cạn | Chỉ có thể chạy được với quy mô dữ liệu đầu vào cực nhỏ |

### Ước lượng độ phức tạp dựa trên quy mô dữ liệu đầu vào ($n$)

Trong các bài toán LeetCode hoặc kỳ thi thuật toán, giới hạn thời gian chạy thường là **1 giây** (tương đương khoảng $10^7$ đến $10^8$ phép tính cơ bản). Giới hạn của $n$ trong đề bài chính là gợi ý trực tiếp cho giải thuật được chấp nhận:

| Giới hạn của $n$ | Độ phức tạp kỳ vọng của giải thuật |
| :--- | :--- |
| **$n \le 20$** | Thuật toán hàm mũ $O(2^n)$, Quay lui, Quy hoạch động trạng thái nén (Bitmask DP) |
| **$n \le 100$** | $O(n^3)$ đôi khi có thể chấp nhận được |
| **$n \le 1000$** | $O(n^2)$ thường là mức phổ biến |
| **$n \le 10^5$** | Bắt buộc phải đạt **$O(n \log n)$** hoặc **$O(n)$** |
| **$n \ge 10^6$** | Bắt buộc phải là **$O(n)$** hoặc **$O(\log n)$** |

Bảng ước lượng này giúp bạn định hướng ngay từ đầu xem thuật toán Brute-force có bị dính lỗi TLE (Time Limit Exceeded) hay không.

---

## Cách xác định độ phức tạp của vòng lặp

Với một vòng lặp đơn giản:
```java
for (int i = 0; i < n; i++) {
    // Thao tác O(1)
}
```
Đoạn mã này thực thi $n$ lần, do đó đạt **$O(n)$**.

Với vòng lặp lồng nhau, **không được nhìn một cách máy móc số tầng lồng nhau mà phải tính tổng số lần thực thi thực tế**:
```java
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        // Thao tác O(1)
    }
}
```
Số lần chạy ở vòng trong là: $n + (n - 1) + \dots + 1 = \frac{n(n + 1)}{2} \approx \frac{1}{2}n^2$, do đó độ phức tạp vẫn là **$O(n^2)$**.

Nếu biến lặp nhân đôi sau mỗi bước:
```java
for (int i = 1; i < n; i *= 2) {
    // Thao tác O(1)
}
```
Số bước lặp thỏa mãn $2^k < n \Rightarrow k < \log_2 n$, do đó đạt **$O(\log n)$**.

### Cạm bẫy dễ nhìn nhầm: Kỹ thuật Hai con trỏ (Two Pointers)
```java
while (left < n && right < n) {
    if (needMoveRight()) {
        right++;
    } else {
        left++;
    }
}
```
Mặc dù bên trong `while` có các rẽ nhánh phức tạp, nhưng cả hai con trỏ `left` và `right` đều chỉ di chuyển **đơn điệu tiến về phía trước** và mỗi con trỏ di chuyển tối đa $n$ lần. Tổng số bước đi tối đa chỉ là $2n$, vì vậy độ phức tạp tổng thể là **$O(n)$**, hoàn toàn không phải $O(n^2)$!

---

## Cách xác định độ phức tạp của Thuật toán Đệ quy

Khi phân tích giải thuật đệ quy, hãy đặt ra 2 câu hỏi cốt lõi:
1. **Mỗi tầng đệ quy rẽ nhánh thành bao nhiêu bài toán con?**
2. **Tại mỗi tầng, ngoài việc gọi hàm đệ quy thì cần làm thêm bao nhiêu công việc phụ trợ?**

### 1. Tìm kiếm nhị phân (Binary Search)
Mỗi lần đệ quy chỉ đi vào duy nhất 1 bài toán con với kích thước giảm đi một nửa:
```java
int binarySearch(int[] nums, int target, int left, int right) {
    if (left > right) return -1;
    int mid = left + (right - left) / 2;
    if (nums[mid] == target) return mid;
    if (nums[mid] < target) return binarySearch(nums, target, mid + 1, right);
    return binarySearch(nums, target, left, mid - 1);
}
```
Độ sâu đệ quy là $\log n$, mỗi tầng chỉ tốn thời gian $O(1)$, vì vậy:
- Độ phức tạp thời gian: **$O(\log n)$**.
- Độ phức tạp không gian (Call Stack): **$O(\log n)$**.

### 2. Sắp xếp trộn (Merge Sort)
Mỗi tầng chia thành 2 bài toán con kích thước $n/2$, công việc gộp (Merge) ở mỗi tầng tốn tổng cộng $O(n)$, tổng số tầng đệ quy là $\log n$:
- Độ phức tạp thời gian: **$O(n \log n)$**.
- Độ phức tạp không gian: **$O(n)$** (mảng phụ trợ).

### 3. Phản ví dụ: Đệ quy Fibonacci ngây thơ
```java
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```
Hàm này không phải là $O(n)$, vì mỗi lần gọi hàm lại phân nhánh thành 2 lời gọi mới, tạo thành một Cây đệ quy nhị phân đầy đủ có độ sâu $n$. Rất nhiều bài toán con bị tính toán lặp đi lặp lại hàng triệu lần, khiến thời gian chạy bùng nổ lên mức **$O(2^n)$**!  
*(Nếu áp dụng Kỹ thuật ghi nhớ Memoization trong Quy hoạch động để mỗi trạng thái chỉ tính đúng 1 lần, thời gian sẽ hạ ngay xuống **$O(n)$**)*.

---

## Độ phức tạp không gian (Space Complexity) tính những gì?

Độ phức tạp không gian đo lường **Lượng bộ nhớ phụ trợ được cấp phát thêm trong quá trình thuật toán vận hành**, bao gồm:
1. **Cấu trúc dữ liệu tự tạo**: Mảng mới, Bảng băm, Hàng đợi, Ngăn xếp.
2. **Ngăn xếp lời gọi hàm đệ quy (Call Stack)**: Chiều sâu tối đa của cây đệ quy chính là lượng bộ nhớ Stack mà chương trình chiếm dụng trong RAM.
3. **Bộ nhớ kết quả đầu ra**: Có tính vào bộ nhớ phụ hay không phụ thuộc vào quy ước của đề bài (trong phỏng vấn, bạn nên chủ động làm rõ: *"Nếu không tính mảng kết quả trả về thì không gian phụ trợ là $O(1)$"*).

Ví dụ: Thuật toán Đảo ngược danh sách liên kết dùng vòng lặp chỉ tốn vài con trỏ tạm, đạt $O(1)$ không gian. Nhưng nếu dùng đệ quy, mặc dù không tạo thêm mảng nào, độ sâu Call Stack đạt $n$, do đó độ phức tạp không gian là $O(n)$!

---

## Các cạm bẫy thường gặp trong phỏng vấn

- **Sắp xếp không miễn phí**: Rất nhiều bạn áp dụng Hai con trỏ trên mảng sau khi gọi `Arrays.sort()`, sau đó kết luận bài toán là $O(n)$. Thực tế thao tác sắp xếp đã tốn ít nhất **$O(n \log n)$**, nên tổng thời gian phải là $O(n \log n)$.
- **`HashMap` không đảm bảo $O(1)$ tuyệt đối**: Khi xảy ra xung đột dữ liệu bất lợi hoặc tấn công băm, thao tác có thể suy thoái.
- **Đệ quy luôn tốn bộ nhớ Stack**: Đừng bao giờ quên tính độ sâu đệ quy vào Space Complexity.
- **Duyệt ma trận 2 chiều $M \times N$**: Độ phức tạp là **$O(m \times n)$**, không được quen tay viết thành $O(n^2)$ nếu $m \ne n$.
- **Hàng đợi trong thuật toán BFS**: Bộ nhớ của Queue trong trường hợp xấu nhất có thể phải chứa toàn bộ một tầng của cây/đồ thị, không bao giờ là hằng số $O(1)$.

## Đề xuất bài tập luyện tập

- [LeetCode 704. Binary Search](https://leetcode.com/problems/binary-search/) (Phân tích $O(\log n)$)
- [LeetCode 912. Sort an Array](https://leetcode.com/problems/sort-an-array/) (Phân tích $O(n \log n)$)
- [LeetCode 206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) (So sánh không gian giữa Vòng lặp $O(1)$ và Đệ quy $O(n)$)
- [LeetCode 200. Number of Islands](https://leetcode.com/problems/number-of-islands/) (Phân tích $O(m \times n)$ cho DFS/BFS)

<!-- @include: @article-footer.snippet.md -->
