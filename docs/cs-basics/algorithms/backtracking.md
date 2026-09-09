---
title: "Tổng hợp bài toán phỏng vấn thuật toán Backtracking: Tổ hợp, Hoán vị, Tập con, Cắt tỉa nhánh và Java template"
description: "Tổng hợp bài toán phỏng vấn thuật toán Backtracking, giải thích nhận diện dạng bài toán Backtracking, template tổ hợp, template hoán vị, template tập con, cắt tỉa loại bỏ trùng lặp, phân tích độ phức tạp và các bài toán LeetCode tần suất cao."
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Thuật toán Backtracking,Backtracking template,Tổ hợp,Hoán vị,Tập con,N-Queens,Cắt tỉa nhánh,Java Backtracking,LeetCode Backtracking,Bài toán phỏng vấn thuật toán
---

Đặc điểm của các bài toán Backtracking (Quay lui) rất rõ ràng: Đề bài yêu cầu bạn tìm tất cả các phương án, tất cả các đường đi, tất cả các tổ hợp, hoặc thử nghiệm từng bước trong một tập hợp các lựa chọn. Nó rất giống với DFS, điểm khác biệt là Backtracking nhấn mạnh hơn vào chu trình "Đưa ra lựa chọn -> Đệ quy -> Thu hồi lựa chọn (Backtrack)".

Khi viết Backtracking trong phỏng vấn, điều quan trọng nhất là nêu rõ ý nghĩa của hàm đệ quy trước. Khi ý nghĩa hàm đã rõ ràng, các tham số, điều kiện dừng và bước thu hồi lựa chọn sẽ không bị nhầm lẫn.

## Trọng tâm khảo sát trong phỏng vấn

- Viết thành thạo 3 bộ template: Tổ hợp (Combinations), Hoán vị (Permutations), Tập con (Subsets).
- Giải thích được vai trò của `path`, `startIndex`, `used`.
- Phán đoán chính xác bài toán có cần loại bỏ trùng lặp (deduplication) hay không.
- Thực hiện cắt tỉa nhánh (pruning) cơ bản để tránh tìm kiếm vô ích.
- Nêu rõ độ phức tạp thuật toán gắn liền với quy mô kết quả sinh ra.

## Tư duy giải bài toán Backtracking như thế nào?

Bài toán Backtracking có thể vẽ thành một "Cây lựa chọn" (Decision Tree). Mỗi tầng trên cây đại diện cho một lần lựa chọn, node gốc đại diện cho trạng thái chưa chọn gì, các node lá đại diện cho một phương án hoàn chỉnh.

Trước khi viết code, hãy trả lời 4 câu hỏi:

1. Đường đi (path) là gì? Thường là các phần tử đã được chọn, trong code thường đặt tên là `path`.
2. Danh sách lựa chọn (choices) là gì? Những phần tử nào hiện tại vẫn có thể chọn tiếp.
3. Điều kiện kết thúc là gì? Khi nào thì đưa `path` vào danh sách kết quả (`ans`).
4. Có cần cắt tỉa nhánh không? Những lựa chọn nào chắc chắn không thể tạo ra kết quả hợp lệ.

Bước "Thu hồi lựa chọn" trong template Backtracking không phải là thủ tục hình thức. Vì đối tượng `path` được tái sử dụng trong suốt quá trình đệ quy, sau khi một nhánh duyệt xong, bắt buộc phải hoàn trả hiện trường (undo choice) để nhánh tiếp theo có thể sử dụng.

## Template Tổ hợp (Combinations)

Tổ hợp không quan tâm đến thứ tự, thường dùng `startIndex` để kiểm soát tầng tiếp theo bắt đầu chọn từ vị trí nào:

```java
List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> ans = new ArrayList<>();
    backtrack(1, n, k, new ArrayList<>(), ans);
    return ans;
}

void backtrack(int start, int n, int k, List<Integer> path, List<List<Integer>> ans) {
    if (path.size() == k) {
        ans.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i <= n; i++) {
        path.add(i);
        backtrack(i + 1, n, k, path, ans);
        path.remove(path.size() - 1);
    }
}
```

Bài toán tổ hợp không quan tâm đến thứ tự, vì vậy `[1, 2]` và `[2, 1]` là cùng một đáp án. Vai trò của `start` là đảm bảo các bước tiếp theo chỉ được chọn các số đứng sau vị trí hiện tại, tránh bị trùng lặp.

Nếu muốn chọn `k` số từ tập hợp `1..n`, ta còn có thể cắt tỉa nhánh:

```java
for (int i = start; i <= n - (k - path.size()) + 1; i++) {
    // ...
}
```

Ý nghĩa là: Nếu bắt đầu từ `i`, số lượng phần tử còn lại trong dãy không còn đủ để ghép thành `k` phần tử nữa, thì không cần tiếp tục duyệt vòng lặp `for`.

## Template Hoán vị (Permutations)

Hoán vị quan tâm đến thứ tự, thường dùng mảng đánh dấu `used` để ghi nhận xem phần tử đã được chọn hay chưa:

```java
List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> ans = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), ans);
    return ans;
}

void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> ans) {
    if (path.size() == nums.length) {
        ans.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) {
            continue;
        }
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, ans);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

Bài toán hoán vị quan tâm đến thứ tự, vì vậy mỗi tầng đều có thể chọn từ tất cả các số, chỉ là không được dùng lặp lại cùng một số đã chọn trước đó. `used[i]` biểu thị phần tử `nums[i]` đã nằm trong đường đi hiện tại hay chưa.

Nếu mảng đầu vào chứa các phần tử trùng lặp, việc loại bỏ trùng lặp trong hoán vị sẽ dễ sai hơn tổ hợp. Cách chuẩn là sắp xếp mảng trước, sau đó ở cùng một tầng, bỏ qua trường hợp "phần tử trùng lặp đứng trước chưa được sử dụng":

```java
if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
    continue;
}
```

Câu lệnh này có tác dụng cố định thứ tự lựa chọn của các phần tử trùng lặp trên cùng một tầng, tránh sinh ra các hoán vị giống nhau.

## Template Tập con (Subsets)

Với bài toán tập con, thông thường mỗi node trên cây lựa chọn đều là một đáp án hợp lệ:

```java
List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> ans = new ArrayList<>();
    backtrack(0, nums, new ArrayList<>(), ans);
    return ans;
}

void backtrack(int start, int[] nums, List<Integer> path, List<List<Integer>> ans) {
    ans.add(new ArrayList<>(path));
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);
        backtrack(i + 1, nums, path, ans);
        path.remove(path.size() - 1);
    }
}
```

Bài toán tập con rất giống bài toán tổ hợp, nhưng nó không chỉ thu thập đáp án khi đạt độ dài cố định, mà cứ mỗi khi duyệt đến một node là thu thập kết quả một lần. Bởi vì đường đi với bất kỳ độ dài nào cũng là một tập con hợp lệ.

Nếu đề bài yêu cầu loại bỏ trùng lặp, ví dụ mảng đầu vào là `[1, 2, 2]`, ta vẫn sắp xếp mảng trước, sau đó bỏ qua các phần tử trùng lặp trên cùng một tầng:

```java
if (i > start && nums[i] == nums[i - 1]) {
    continue;
}
```

## Khử trùng lặp như thế nào?

Nếu đầu vào chứa các phần tử trùng lặp, thông thường ta sắp xếp trước, sau đó chọn chiến lược khử trùng lặp tùy theo dạng bài:

- Với dạng bài chọn tịnh tiến theo chỉ số (chẳng hạn như Subsets, Combinations), bỏ qua phần tử trùng lặp trên cùng một tầng, ví dụ: `i > start && nums[i] == nums[i - 1]`.
- Với dạng bài hoán vị (Permutations) mà mỗi tầng đều có thể quét từ đầu mảng, thường phải kết hợp thêm `used[]` để tránh một vị trí bị dùng lại nhiều lần.
- Điều kiện khử trùng lặp cần phân biệt rõ giữa "chọn trùng lặp trên cùng một tầng" và "dùng trùng lặp trên cùng một đường đi". Trường hợp trước sẽ tạo ra đáp án trùng nhau, còn trường hợp sau có thể chính là lựa chọn hợp lệ mà đề bài cho phép.

## Minh họa quy trình và các trường hợp biên

Lấy bài toán tổ hợp với `n = 3, k = 2` làm ví dụ, cây lựa chọn có thể rút gọn như sau:

| Lựa chọn tầng 1 | Khả năng chọn ở tầng 2 | Kết quả sinh ra |
| ---------- | ---------- | ------------------ |
| Chọn 1 | 2, 3 | `[1, 2]`, `[1, 3]` |
| Chọn 2 | 3 | `[2, 3]` |
| Chọn 3 | Không còn | Không đủ 2 số, cắt tỉa nhánh |

Với bài toán Backtracking, bạn nên kiểm tra các trường hợp biên sau:

| Đầu vào | Trọng tâm kiểm tra |
| ------------ | ----------------------- |
| Mảng rỗng | Bài toán Subsets thông thường phải trả về `[[]]` |
| `k = 0` | Bài toán tổ hợp có trả về tổ hợp rỗng hay không |
| Chứa phần tử trùng lặp | Đã sắp xếp và khử trùng lặp trên cùng tầng chưa |
| Chỉ có 1 kết quả duy nhất | Có sao chép `path` chính xác không |

Lỗi viết code rất phổ biến:

```java
ans.add(path); // Sai: path sẽ tiếp tục bị thay đổi trong các đệ quy tiếp theo
```

Bắt buộc phải viết thành:

```java
ans.add(new ArrayList<>(path));
```

Trong Backtracking, đối tượng `path` được tái sử dụng liên tục. Nếu không tạo bản sao (copy) mới, danh sách lưu trong đáp án sẽ bị các bước đệ quy sau đó làm biến đổi toàn bộ.

## Các lỗi thường gặp (Pitfalls)

- Khi đưa vào danh sách đáp án phải tạo bản sao của `path`, không được đưa trực tiếp tham chiếu vào.
- Bài toán tổ hợp dùng `startIndex`, bài toán hoán vị dùng `used`, không được viết lẫn lộn hai cơ chế này.
- Loại bỏ trùng lặp thông thường bắt buộc phải sắp xếp mảng trước.
- Điều kiện cắt tỉa nhánh tuyệt đối không được làm ảnh hưởng đến các đáp án chính xác.
- Độ phức tạp của Backtracking thường cùng bậc với số lượng kết quả sinh ra, đừng tùy tiện kết luận là `O(n)`.

## Bài tập rèn luyện đề xuất

- [77. Combinations](https://leetcode.cn/problems/combinations/)
- [78. Subsets](https://leetcode.cn/problems/subsets/)
- [46. Permutations](https://leetcode.cn/problems/permutations/)
- [39. Combination Sum](https://leetcode.cn/problems/combination-sum/)
- [51. N-Queens](https://leetcode.cn/problems/n-queens/)

<!-- @include: @article-footer.snippet.md -->
