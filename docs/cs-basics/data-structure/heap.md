---
title: Chi tiết Heap (Max-Heap, Min-Heap, Hàng đợi ưu tiên PriorityQueue)
description: Phân tích toàn diện cấu trúc Heap: Phân biệt Max-Heap/Min-Heap, cơ chế Sift-Up/Sift-Down, thuật toán Heap Sort và giải quyết các bài toán Top-K, Trung vị luồng dữ liệu trong Java.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Heap, Đống, Max-Heap, Min-Heap, PriorityQueue, Hàng đợi ưu tiên, Heapify, Sift-Up, Sift-Down, Heap Sort, Top-K, Trung vị luồng dữ liệu
---

## 1. Heap là gì?

**Heap (Đống)** là một cấu trúc dữ liệu dạng cây thỏa mãn tính chất sau:

> **Tính chất Heap**: Với mọi node trong cây, giá trị của node cha luôn **lớn hơn hoặc bằng** (hoặc **nhỏ hơn hoặc bằng**) giá trị của tất cả các node con trong cây con của nó.

> *Ẩn dụ trực quan*: Có thể hình dung Heap (Max-Heap) như một tổ chức làm việc công bằng: người nào có năng lực cao nhất sẽ ngồi ở vị trí cao nhất (đỉnh Heap), và bất kỳ cấp trên nào cũng luôn giỏi hơn hoặc bằng cấp dưới trực tiếp của mình.

> [!NOTE]
> **Lưu ý quan trọng**:
> - Rất nhiều tài liệu nói rằng "Heap bắt buộc phải là cây nhị phân hoàn chỉnh", nhưng điều này không hoàn toàn chính xác. Về mặt định nghĩa toán học, **Heap không bắt buộc phải là cây nhị phân hoàn chỉnh**. Chúng ta thường dùng hình thức cây nhị phân hoàn chỉnh để biểu diễn **Binary Heap** nhằm mục đích lưu trữ bằng mảng tiện lợi nhất. Trên thực tế, các biến thể nổi tiếng như *Fibonacci Heap* hay *Binomial Heap* hoàn toàn không phải là cây nhị phân hoàn chỉnh.
> - Trong cuốn sách kinh điển *Introduction to Algorithms (CLRS)*: *"Binary Heap là một mảng dữ liệu có thể được xem như một cây nhị phân gần như hoàn chỉnh."*

![Ví dụ minh họa cấu trúc Heap](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-1.png)

Trong hình trên:
- Cây 1 là **Max-Heap** (Mọi node cha đều $\ge$ node con).
- Cây 2 là **Min-Heap** (Mọi node cha đều $\le$ node con).
- Cây 3 **không phải là Heap** (Gốc 1 nhỏ hơn con 2 và 15, nhưng 15 lại lớn hơn 3).

---

## 2. Ứng dụng và Điểm mạnh của Heap

Heap được sử dụng khi chúng ta **liên tục cần truy xuất giá trị lớn nhất (hoặc nhỏ nhất)** từ một tập dữ liệu biến động (có liên tục các thao tác thêm mới hoặc xóa bỏ phần tử cực trị).

So sánh với Mảng có thứ tự (Sorted Array):
- Khởi tạo mảng có thứ tự mất $O(n \log n)$. Lấy giá trị lớn nhất mất $O(1)$. Nhưng mỗi khi **chèn hoặc xóa một phần tử**, ta mất tới **$O(n)$** để dịch chuyển các phần tử trong mảng.
- **Heap vượt trội hơn hẳn khi thao tác Chèn và Xóa**: Do chỉ cần di chuyển dọc theo chiều cao của cây nhị phân, thao tác chèn và xóa trên Heap chỉ tiêu tốn **$O(\log n)$**!

| Thao tác | Heap (Binary Heap) | Sorted Array (Mảng có thứ tự) | Unsorted Array (Mảng chưa sắp xếp) |
| :--- | :--- | :--- | :--- |
| **Xem phần tử cực trị** | $O(1)$ | $O(1)$ | $O(n)$ |
| **Chèn phần tử mới** | $O(\log n)$ | $O(n)$ | $O(1)$ |
| **Xóa phần tử cực trị** | $O(\log n)$ | $O(n)$ | $O(n)$ |
| **Xây dựng từ mảng (Build Heap)** | **$O(n)$** (Floyd) | $O(n \log n)$ | $O(1)$ |

---

## 3. Phân loại Heap

- **Max-Heap (Đống cực đại)**: Giá trị của mọi node cha đều $\ge$ giá trị các node con. Đỉnh Heap luôn là phần tử **lớn nhất** trong toàn bộ tập dữ liệu.
- **Min-Heap (Đống cực tiểu)**: Giá trị của mọi node cha đều $\le$ giá trị các node con. Đỉnh Heap luôn là phần tử **nhỏ nhất** trong toàn bộ tập dữ liệu.

![Minh họa Max-Heap và Min-Heap](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-2.png)

---

## 4. Cách lưu trữ Heap trong Mảng

Nhờ cấu trúc cây nhị phân hoàn chỉnh, Binary Heap được lưu trữ trực tiếp trong một **Mảng một chiều (Array)** mà không tốn thêm bất kỳ con trỏ nào.

![Lưu trữ Heap trong Mảng một chiều](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-storage.png)

Với quy ước chỉ số mảng bắt đầu từ **1** (hoặc điều chỉnh tương ứng nếu bắt đầu từ 0):
- Node cha tại vị trí $i$.
- Node con bên trái nằm tại vị trí: **$2i$** (hoặc $2i + 1$ nếu mảng 0-indexed).
- Node con bên phải nằm tại vị trí: **$2i + 1$** (hoặc $2i + 2$ nếu mảng 0-indexed).
- Node cha của node $i$ nằm tại vị trí: **$\lfloor i / 2 \rfloor$** (hoặc $\lfloor (i - 1) / 2 \rfloor$).

---

## 5. Các thao tác cốt lõi trên Heap

### 1. Thao tác Chèn phần tử (Insertion / Sift-Up - Nổi lên)
1. Đặt phần tử mới vào vị trí cuối cùng của mảng (cuối Heap).
2. So sánh phần tử mới với node cha của nó. Nếu vi phạm tính chất Heap (ví dụ lớn hơn cha trong Max-Heap), ta **hoán đổi vị trí của nó với node cha**.
3. Lặp lại quá trình này hướng lên trên cho đến khi thỏa mãn tính chất Heap hoặc chạm tới gốc. Thao tác này gọi là **Sift-Up (Thao tác nổi lên)**, độ phức tạp $O(\log n)$.

![Thao tác chèn và Sift-Up trong Heap](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-insert-2.png)

### 2. Thao tác Xóa đỉnh Heap (Extract-Max / Sift-Down - Chìm xuống)
1. Lấy giá trị tại đỉnh Heap (vị trí chỉ số 1 / 0) ra ngoài.
2. Đưa phần tử cuối cùng của mảng lên thế chỗ cho đỉnh Heap vừa bị xóa.
3. So sánh phần tử này với các node con của nó. Hoán đổi nó với **node con có giá trị lớn hơn** (trong Max-Heap).
4. Lặp lại quá trình hoán đổi đi xuống dưới cho đến khi nó lớn hơn tất cả các con hoặc trở thành node lá. Thao tác này gọi là **Sift-Down (Thao tác chìm xuống / Heapify)**, độ phức tạp $O(\log n)$.

![Thao tác xóa đỉnh và Sift-Down trong Heap](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-delete-top-5.png)

---

## 6. Thuật toán Sắp xếp vun đống (Heap Sort)

Heap Sort bao gồm 2 giai đoạn:
1. **Xây dựng Heap (Build Heap)**: Biến mảng $n$ phần tử thành một Max-Heap. Bằng thuật toán Floyd (Sift-Down từ vị trí $\lfloor n/2 \rfloor$ ngược về 1), thời gian xây dựng chỉ tốn **$O(n)$**.
2. **Sắp xếp (Sorting)**:
   - Lặp $n-1$ lần: Hoán đổi đỉnh Heap (phần tử lớn nhất) với phần tử cuối cùng của Heap, thu hẹp kích thước Heap đi 1, sau đó thực hiện Sift-Down từ đỉnh để khôi phục Max-Heap.
   - Kết quả thu được một mảng sắp xếp tăng dần tại chỗ (**In-place Sort**) với thời gian $O(n \log n)$ và bộ nhớ phụ $O(1)$.

![Quy trình sắp xếp vun đống Heap Sort](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/heap-sort-6.png)

---

## 7. Ứng dụng thực tế và Bài toán Top-K trong Java

Trong Java, `java.util.PriorityQueue` chính là cài đặt chuẩn mực của **Min-Heap (Đống cực tiểu)**.

```java
// Mặc định là Min-Heap (phần tử nhỏ nhất ở đầu)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Tùy biến thành Max-Heap (phần tử lớn nhất ở đầu)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> Integer.compare(b, a));
```

> **Lưu ý**: Tuyệt đối không viết biểu thức so sánh dạng `(b - a)` vì có thể gây ra hiện tượng tràn số nguyên (Integer Overflow) khi gặp giá trị cực đoan (`Integer.MIN_VALUE`). Hãy luôn dùng `Integer.compare(b, a)`.

### Giải quyết bài toán Tìm phần tử lớn thứ K (Kth Largest Element)

Sử dụng một **Min-Heap kích thước K**:
- Duyệt qua từng phần tử trong mảng: Nếu Heap chưa đủ $K$ phần tử thì `offer` vào; nếu Heap đã đủ $K$ phần tử và phần tử hiện tại lớn hơn đỉnh Heap (`num > heap.peek()`), ta `poll` đỉnh cũ và `offer` phần tử mới vào.
- Sau khi duyệt hết mảng, đỉnh Heap chính là **Phần tử lớn thứ K**!
- Độ phức tạp thời gian: **$O(n \log k)$**, độ phức tạp không gian: **$O(k)$**.

```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>(k);
    for (int num : nums) {
        if (heap.size() < k) {
            heap.offer(num);
        } else if (num > heap.peek()) {
            heap.poll();
            heap.offer(num);
        }
    }
    return heap.peek();
}
```

## Đề xuất bài tập luyện tập

- [LeetCode 215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)
- [LeetCode 347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
- [LeetCode 703. Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)
- [LeetCode 295. Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)

<!-- @include: @article-footer.snippet.md -->
