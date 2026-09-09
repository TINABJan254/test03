---
title: "Tổng hợp bài toán phỏng vấn Top K: Heap, Phân vùng Quick Sort, Bucket Sort và Data Stream"
description: "Tổng hợp bài toán phỏng vấn Top K, giải thích phần tử lớn thứ K, Top K phần tử có tần suất cao nhất, Min-Heap, Max-Heap, phân vùng Quick Sort (Quickselect), Bucket Sort, trung vị luồng dữ liệu (Data Stream Median), PriorityQueue và LeetCode."
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: TopK,Top K,Lớn thứ K,Tần suất cao nhất,Heap,Min-Heap,Max-Heap,Quickselect,Phân vùng Quick Sort,Bucket Sort,PriorityQueue,Data Stream Median,Trung vị luồng dữ liệu,LeetCode
---

Bài toán Top K xuất hiện rất phổ biến trong các buổi phỏng vấn backend, bởi vì nó vừa kiểm tra được tư duy thuật toán, vừa dễ dàng mở rộng sang các tình huống kỹ thuật thực tế: Bảng xếp hạng (Leaderboard), thống kê từ khóa hot, tìm trung vị luồng dữ liệu (Data Stream Median), mã lỗi xuất hiện nhiều nhất trong log... tất cả đều quy về bài toán Top K.

Với dạng bài này, đừng chỉ học thuộc một cách giải duy nhất. Người phỏng vấn thường sẽ hỏi mở rộng: Nếu lượng dữ liệu cực kỳ lớn thì làm thế nào? Nếu là dữ liệu luồng (Data Stream) đến liên tục thì sao? Nếu yêu cầu tìm Top K phần tử có tần suất cao nhất thì làm thế nào? Các điều kiện khác nhau sẽ dẫn đến các phương án tối ưu khác nhau.

## Trọng tâm khảo sát trong phỏng vấn

- Sử dụng thành thạo Heap để giải quyết bài toán tìm phần tử lớn thứ K và Top K phần tử có tần suất cao nhất.
- Nêu rõ lý do lựa chọn Min-Heap hay Max-Heap trong từng trường hợp.
- So sánh được độ phức tạp giữa Heap, Phân vùng Quick Sort (Quickselect) và Bucket Sort.
- Xử lý tốt các tình huống dữ liệu dạng luồng (Data Stream).
- Viết thành thạo bộ so sánh (Comparator) cho `PriorityQueue` trong Java.

## Chọn phương án nào cho bài toán Top K?

Trước tiên hãy xem xét 3 điều kiện:

1. Đề bài chỉ cần đúng phần tử thứ K, hay cần toàn bộ tập hợp K phần tử đầu tiên?
2. Dữ liệu được đưa vào một lần (offline array), hay là luồng dữ liệu đến liên tục theo thời gian (stream)?
3. Kết quả đầu ra có yêu cầu phải sắp xếp theo thứ tự hay không?

Nếu chỉ tìm phần tử lớn thứ K trong một mảng tĩnh một lần, thuật toán phân vùng Quick Sort (Quickselect) có hiệu suất trung bình nhanh hơn; nếu dữ liệu đến liên tục, việc duy trì một Heap kích thước K là tự nhiên nhất; nếu đề bài hỏi Top K phần tử có tần suất cao nhất, cần thống kê tần suất trước rồi mới áp dụng Top K lên tần suất đó.

## So sánh các phương án

| Phương án | Phù hợp cho | Độ phức tạp thời gian | Độ phức tạp không gian |
| -------- | ------------------------ | ------------------ | ------------------- |
| Sắp xếp toàn bộ | Dữ liệu nhỏ, ưu tiên code đơn giản | `O(n log n)` | Phụ thuộc thuật toán sắp xếp |
| Min-Heap (Heap nhỏ) | Tìm K phần tử lớn nhất hoặc lớn thứ K | `O(n log k)` | `O(k)` |
| Quickselect (Phân vùng Quick Sort) | Tìm phần tử lớn thứ K, hiệu suất trung bình cao | Trung bình `O(n)` | `O(1)` đến `O(log n)` |
| Bucket Sort (Đếm theo thùng) | Phạm vi tần suất bị chặn, Top K tần suất | `O(n)` | `O(n)` |
| Dual Heaps (Hai Heap) | Tìm trung vị trong luồng dữ liệu (Data Stream Median) | Mỗi lần thêm `O(log n)` | `O(n)` |

Trong phỏng vấn, bạn có thể giải thích sự đánh đổi như sau:

- Sắp xếp toàn bộ đơn giản nhất, phù hợp khi dữ liệu không quá lớn hoặc không yêu cầu tối ưu độ phức tạp.
- Heap phù hợp khi K nhỏ hơn rất nhiều so với n, không gian bộ nhớ chỉ tốn `O(k)`.
- Quickselect thích hợp để tìm phần tử thứ K một lần, thời gian trung bình là `O(n)`, nhưng trường hợp xấu nhất có thể bị thoái hóa.
- Bucket Sort phù hợp cho bài toán tần suất, đặc biệt khi tần suất tối đa không vượt quá `n`.

## Dùng Min-Heap tìm phần tử lớn thứ K

```java
int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    for (int num : nums) {
        heap.offer(num);
        if (heap.size() > k) {
            heap.poll();
        }
    }
    return heap.peek();
}
```

Bên trong Heap luôn lưu giữ đúng K phần tử lớn nhất hiện tại. Đỉnh Heap (peek) chính là phần tử nhỏ nhất trong số K phần tử lớn này, tức là phần tử lớn thứ K của toàn bộ mảng.

Tại sao lại dùng Min-Heap? Vì mục tiêu là giữ lại K phần tử lớn nhất. Khi một phần tử mới đi vào làm kích thước Heap vượt quá K, ta phải loại bỏ phần tử nhỏ nhất trong số K + 1 phần tử này. Đỉnh của Min-Heap chính là giá trị nhỏ nhất, cho phép loại bỏ cực kỳ tiện lợi với `poll()`.

Nếu bài toán yêu cầu tìm phần tử nhỏ thứ K, tư duy sẽ đảo ngược lại: Duy trì một Max-Heap (Heap lớn) kích thước K, khi kích thước vượt quá K thì loại bỏ phần tử lớn nhất ở đỉnh Heap.

## Phân tích bài toán tiêu biểu: Top K phần tử có tần suất cao nhất

[347. Top K Frequent Elements](https://leetcode.cn/problems/top-k-frequent-elements/) là bài toán tần suất phổ biến nhất của Top K. Đề bài cho một mảng số nguyên và một số nguyên `k`, yêu cầu trả về `k` phần tử xuất hiện nhiều nhất, thứ tự kết quả thông thường không quan trọng.

Với bài này, không được sắp xếp trực tiếp trên mảng gốc, vì tiêu chí so sánh là "tần suất xuất hiện" chứ không phải giá trị phần tử. Cách giải ổn định gồm hai bước:

1. Dùng `HashMap` để đếm số lần xuất hiện của từng phần tử.
2. Duy trì một Min-Heap sắp xếp tăng dần theo tần suất, trong Heap chỉ giữ lại đúng `k` phần tử có tần suất cao nhất hiện tại.

Tại sao vẫn dùng Min-Heap? Vì sau khi Heap đầy, mỗi khi thêm phần tử mới khiến kích thước vượt quá `k`, ta sẽ loại bỏ phần tử có tần suất thấp nhất hiện tại. Nhờ đó, sau khi duyệt qua toàn bộ các phần tử phân biệt, những phần tử còn lại trong Heap chắc chắn là Top `k` phần tử có tần suất cao nhất.

```java
int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) {
        freq.put(num, freq.getOrDefault(num, 0) + 1);
    }
    PriorityQueue<int[]> heap = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        heap.offer(new int[] {entry.getKey(), entry.getValue()});
        if (heap.size() > k) {
            heap.poll();
        }
    }
    int[] ans = new int[k];
    for (int i = k - 1; i >= 0; i--) {
        ans[i] = heap.poll()[0];
    }
    return ans;
}
```

Ở đây Heap được sắp xếp tăng dần theo tần suất (`a[1]`), khi kích thước vượt quá K thì loại bỏ phần tử có tần suất nhỏ nhất.

Lấy `nums = [1,1,1,2,2,3]`, `k = 2` làm ví dụ: Bảng tần suất là `{1=3, 2=2, 3=1}`. Heap lần lượt nhận `1` và `2`, khi nhận thêm `3` thì kích thước vượt quá 2, Heap sẽ loại bỏ phần tử `3` có tần suất thấp nhất, cuối cùng giữ lại `1` và `2`.

Nếu `k` bằng đúng số lượng phần tử phân biệt, Heap sẽ giữ lại tất cả các phần tử; nếu người phỏng vấn yêu cầu kết quả xuất ra phải sắp xếp giảm dần theo tần suất, ta cần sắp xếp thêm mảng kết quả cuối cùng.

Nếu người phỏng vấn yêu cầu khi tần suất bằng nhau thì sắp xếp theo thứ tự từ điển hoặc độ lớn phần tử, ta cần viết quy tắc so sánh thứ hai vào trong Comparator. Ví dụ bài toán Top K từ ngữ xuất hiện nhiều nhất thường yêu cầu tần suất cao đứng trước, tần suất bằng nhau thì thứ tự từ điển nhỏ đứng trước.

## Tư duy phân vùng Quick Sort (Quickselect)

Quickselect thích hợp để tìm phần tử lớn thứ K khi không yêu cầu xuất ra K phần tử theo thứ tự sắp xếp. Ý tưởng là mỗi lần chia mảng thành hai nửa dựa vào một phần tử chốt (pivot), căn cứ vào thứ hạng của pivot để quyết định chỉ tiếp tục tìm kiếm ở nửa bên trái hay nửa bên phải. Độ phức tạp thời gian trung bình là `O(n)`, nhưng trường hợp xấu nhất có thể thoái hóa về `O(n^2)`. Trong thực tế, pivot thường được chọn ngẫu nhiên để tránh thoái hóa.

Ưu điểm của Quickselect là không cần cấu trúc Heap, độ phức tạp thời gian trung bình thấp; hạn chế là nó chỉ phù hợp với dữ liệu tĩnh một lần trong bộ nhớ. Nếu là dữ liệu luồng liên tục đổ về hoặc dữ liệu quá lớn không thể nạp hết vào RAM cùng lúc, phương án dùng Heap sẽ khả thi hơn nhiều.

## Tình huống dữ liệu dạng luồng (Data Stream)

Với bài toán Data Stream, không thể cứ mỗi khi có phần tử mới đến lại sắp xếp lại toàn bộ. Cách làm chuẩn là duy trì liên tục một cấu trúc dữ liệu thích hợp:

- Phần tử lớn thứ K trong luồng: Duy trì một Min-Heap kích thước K.
- Trung vị trong luồng dữ liệu (Data Stream Median): Duy trì hai Heap, bên trái là Max-Heap lưu trữ nửa phần tử nhỏ hơn, bên phải là Min-Heap lưu trữ nửa phần tử lớn hơn.
- Trung vị trong cửa sổ trượt (Sliding Window Median): Cần xử lý thêm phần tử hết hạn khỏi cửa sổ, Heap thông thường xóa phần tử bất kỳ không tiện, thường phải áp dụng kỹ thuật xóa lười (lazy deletion) hoặc cấu trúc tập hợp có thứ tự (TreeMap/Multiset).

## Minh họa quy trình và các trường hợp biên

Lấy mảng `[3, 2, 1, 5, 6, 4]` tìm phần tử lớn thứ 2 làm ví dụ, duy trì một Min-Heap kích thước 2. Trong bảng dưới đây, các phần tử trong Heap được hiển thị theo thứ tự tăng dần để tiện quan sát (không đại diện cho thứ tự mảng nội bộ của `PriorityQueue`):

| Phần tử đọc vào | Các phần tử ứng viên trong Heap | Xử lý khi kích thước vượt quá K |
| -------- | ----------- | --------------------- |
| 3 | `[3]` | Chưa vượt quá K |
| 2 | `[2, 3]` | Chưa vượt quá K |
| 1 | `[1, 2, 3]` | Loại bỏ 1, giữ lại `[2, 3]` |
| 5 | `[2, 3, 5]` | Loại bỏ 2, giữ lại `[3, 5]` |
| 6 | `[3, 5, 6]` | Loại bỏ 3, giữ lại `[5, 6]` |
| 4 | `[4, 5, 6]` | Loại bỏ 4, giữ lại `[5, 6]` |

Cuối cùng, đỉnh Heap là `5`, chính là phần tử lớn thứ 2.

Lỗi thường gặp khi viết code:

```java
PriorityQueue<Integer> heap = new PriorityQueue<>((a, b) -> b - a);
```

Bộ so sánh này có thể bị tràn số nguyên (integer overflow) nếu gặp các giá trị số âm và dương cực trị. Cách viết an toàn nhất là:

```java
PriorityQueue<Integer> heap = new PriorityQueue<>((a, b) -> Integer.compare(b, a));
```

## Các lỗi thường gặp (Pitfalls)

- Tìm K phần tử lớn nhất thường dùng Min-Heap, tìm K phần tử nhỏ nhất thường dùng Max-Heap.
- `PriorityQueue` trong Java mặc định là Min-Heap.
- Top K phần tử có tần suất cao nhất phải thống kê tần suất trước, sau đó mới áp dụng Top K lên tần suất.
- Nếu yêu cầu kết quả đầu ra có thứ tự, sau khi dùng Heap hoặc Quickselect vẫn cần sắp xếp thêm.
- Tình huống Data Stream tuyệt đối không được sắp xếp lại toàn bộ dữ liệu sau mỗi lần nhận thêm phần tử.

## Câu hỏi tự kiểm tra tần suất cao

- Tại sao tìm phần tử lớn thứ K thông thường lại duy trì một Min-Heap kích thước K?
- Min-Heap và Max-Heap lần lượt phù hợp cho những tình huống Top K nào?
- Sự khác biệt về độ phức tạp thời gian và không gian giữa phương án dùng Heap và Quickselect là gì?
- Tại sao bài toán Top K phần tử có tần suất cao nhất bắt buộc phải thống kê tần suất trước?
- Tại sao tìm trung vị trong luồng dữ liệu lại thích hợp dùng hai Heap để duy trì?

## Bài tập rèn luyện đề xuất

- [215. Kth Largest Element in an Array](https://leetcode.cn/problems/kth-largest-element-in-an-array/)
- [347. Top K Frequent Elements](https://leetcode.cn/problems/top-k-frequent-elements/)
- [692. Top K Frequent Words](https://leetcode.cn/problems/top-k-frequent-words/)
- [703. Kth Largest Element in a Stream](https://leetcode.cn/problems/kth-largest-element-in-a-stream/)
- [295. Find Median from Data Stream](https://leetcode.cn/problems/find-median-from-data-stream/)

<!-- @include: @article-footer.snippet.md -->
