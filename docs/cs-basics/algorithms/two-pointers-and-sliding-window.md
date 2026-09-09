---
title: "Tổng hợp bài toán phỏng vấn Two Pointers và Sliding Window: Template tần suất cao cho Mảng, LinkedList, Chuỗi"
description: "Tổng hợp bài toán phỏng vấn Two Pointers và Sliding Window, giải thích Left-Right Pointers, Fast-Slow Pointers, Read-Write Pointers, Fixed Window, Dynamic Window, Java template và các bài toán LeetCode tần suất cao."
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Two Pointers,Sliding Window,Fast-Slow Pointers,Left-Right Pointers,Read-Write Pointers,Fixed Window,Dynamic Window,Thuật toán mảng,Thuật toán LinkedList,Thuật toán chuỗi,LeetCode
---

Two Pointers (Hai con trỏ) và Sliding Window (Cửa sổ trượt) thường được ôn tập cùng nhau, nhưng bản chất vấn đề chúng giải quyết không hoàn toàn giống nhau. Two Pointers giống một chiến lược di chuyển hơn, trong khi Sliding Window nhấn mạnh vào việc duy trì trạng thái bên trong một khoảng liên tục (continuous interval).

Một quy tắc phán đoán thực tế: Nếu bài toán quan tâm đến mối quan hệ giữa hai vị trí, hãy nghĩ đến Two Pointers trước; nếu bài toán quan tâm đến mảng con liên tục (contiguous subarray) hoặc chuỗi con liên tục (substring), và bên trong cửa sổ có điều kiện cần duy trì, hãy nghĩ đến Sliding Window trước.

## Trọng tâm khảo sát trong phỏng vấn

- Phân biệt được Left-Right Pointers (con trỏ trái phải), Fast-Slow Pointers (con trỏ nhanh chậm), Read-Write Pointers (con trỏ đọc ghi).
- Duy trì được bộ đếm (count), tổng (sum), giá trị lớn nhất (max) hoặc trạng thái khớp (matching) trong Sliding Window.
- Giải thích được tại sao các con trỏ chỉ di chuyển theo một hướng thì độ phức tạp thời gian là `O(n)`.
- Xử lý tốt mảng rỗng, phần tử đơn lẻ, phần tử trùng lặp và các biên thu hẹp cửa sổ.

## Điểm khác biệt cốt lõi giữa hai phương pháp là gì?

Two Pointers là một cách viết mang tính khái quát rộng hơn: chỉ cần dùng hai con trỏ phối hợp tịnh tiến thì đều có thể gọi là Two Pointers. Sliding Window cụ thể hơn, nó duy trì một khoảng liên tục `[left, right]`, bên trong cửa sổ thường chứa một tập hợp trạng thái như: số lần xuất hiện của ký tự, tổng các phần tử, giá trị lớn nhất, số lượng điều kiện đã thỏa mãn.

| Đặc điểm bài toán | Thường ưu tiên sử dụng |
| ----------------------------------- | ---------- |
| Tìm hai số trong mảng đã sắp xếp | Left-Right Pointers |
| LinkedList: tìm chu trình (cycle), tìm trung điểm, tìm node thứ K từ dưới lên | Fast-Slow Pointers |
| Xóa hoặc ghi đè phần tử tại chỗ (In-place) | Read-Write Pointers |
| Dài nhất, ngắn nhất, đếm số lượng của mảng con/chuỗi con liên tục | Sliding Window |

Trong phỏng vấn, nêu rõ ý nghĩa của các con trỏ trước khi viết code sẽ giúp bạn tự tin và chắc chắn hơn rất nhiều. Ví dụ: "`left` biểu thị biên trái của cửa sổ, `right` biểu thị ký tự đang thử thêm vào cửa sổ", khi thu hẹp cửa sổ về sau sẽ không bị rối loạn logic.

## Left-Right Pointers (Con trỏ trái phải)

Left-Right Pointers thường dùng cho mảng đã sắp xếp hoặc các bài toán thu hẹp dần từ hai đầu:

```java
int[] twoSumSorted(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            return new int[] {left, right};
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    return new int[] {-1, -1};
}
```

Nếu mảng chưa sắp xếp, thông thường cần sắp xếp trước rồi mới dùng Left-Right Pointers. Sau khi sort, hãy nhớ độ phức tạp sẽ trở thành `O(n log n)`.

Nguyên nhân Left-Right Pointers hoạt động hiệu quả là vì sau mỗi lần so sánh có thể loại bỏ được một phần không gian nghiệm. Lấy bài toán Two Sum trên mảng đã sắp xếp làm ví dụ:

- Tổng hiện tại quá nhỏ: cho thấy phần tử tại con trỏ trái quá nhỏ, nếu dịch con trỏ phải sang trái thì tổng chỉ càng nhỏ hơn, do đó chỉ có thể dịch con trỏ trái sang phải (`left++`).
- Tổng hiện tại quá lớn: cho thấy phần tử tại con trỏ phải quá lớn, nếu dịch con trỏ trái sang phải thì tổng chỉ càng lớn hơn, do đó chỉ có thể dịch con trỏ phải sang trái (`right--`).

Bài toán Three Sum (Tổng 3 số) cũng dựa trên tư duy tương tự: cố định trước một số, sau đó tìm Two Sum trên khoảng còn lại. Điểm khó nằm ở việc loại bỏ trùng lặp (deduplication): số cố định cần loại bỏ trùng lặp, sau khi hai con trỏ trái phải tìm được đáp án cũng phải bỏ qua các giá trị trùng lặp tiếp theo.

## Fast-Slow Pointers (Con trỏ nhanh chậm)

Fast-Slow Pointers thường dùng cho LinkedList:

```java
boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            return true;
        }
    }
    return false;
}
```

Điểm mấu chốt của các bài toán LinkedList không nằm ở độ dài code, mà ở việc ý nghĩa của con trỏ phải nhất quán và điều kiện biên ổn định. Thứ tự điều kiện `fast != null && fast.next != null` tuyệt đối không được đảo ngược.

Fast-Slow Pointers thường có hai dạng chênh lệch tốc độ:

- `fast` mỗi lần đi 2 bước, `slow` mỗi lần đi 1 bước: dùng để phát hiện chu trình (cycle detection) và tìm trung điểm của LinkedList.
- Một con trỏ đi trước `k` bước, con trỏ còn lại sau đó mới cùng xuất phát: dùng để tìm node thứ `k` tính từ cuối LinkedList lên.

Khi tìm node thứ `k` tính từ cuối lên, hai con trỏ duy trì khoảng cách đúng bằng `k` nodes. Khi con trỏ phía trước đi đến cuối danh sách, con trỏ phía sau sẽ dừng đúng tại vị trí mục tiêu. Khi xóa node thứ `N` tính từ cuối lên, người ta thường dùng Dummy Head (node giả đầu danh sách) để tránh phải xử lý trường hợp ngoại lệ khi xóa chính node đầu.

## Read-Write Pointers (Con trỏ đọc ghi)

Read-Write Pointers thường dùng để sửa đổi mảng tại chỗ (In-place):

```java
int removeDuplicates(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }
    int write = 1;
    for (int read = 1; read < nums.length; read++) {
        if (nums[read] != nums[read - 1]) {
            nums[write] = nums[read];
            write++;
        }
    }
    return write;
}
```

`read` chịu trách nhiệm duyệt qua mảng gốc, `write` trỏ vào vị trí tiếp theo có thể ghi dữ liệu. Trong phỏng vấn, tốt nhất nên nêu rõ ý nghĩa của hai biến này ngay từ đầu.

Cốt lõi của Read-Write Pointers là "duyệt qua toàn bộ mảng, chỉ ghi đè những nội dung cần giữ lại lên phần đầu". Dạng bài này thường yêu cầu thao tác in-place và trả về độ dài mới thay vì cấp phát mảng mới.

Khi xác định thời điểm ghi dữ liệu, hãy tự hỏi: Phần tử mà `read` đang trỏ tới có nên được giữ lại không? Nếu nên giữ lại, ghi vào vị trí `write` và tăng `write++`; nếu không nên giữ lại, chỉ tăng con trỏ `read`.

## Dynamic Sliding Window (Cửa sổ trượt có thể thay đổi kích thước)

Lấy bài toán "Chuỗi con dài nhất không chứa ký tự trùng lặp" làm ví dụ:

```java
int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> count = new HashMap<>();
    int left = 0;
    int ans = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        count.put(c, count.getOrDefault(c, 0) + 1);
        while (count.get(c) > 1) {
            char d = s.charAt(left);
            count.put(d, count.get(d) - 1);
            left++;
        }
        ans = Math.max(ans, right - left + 1);
    }
    return ans;
}
```

Trong template này, con trỏ phải chịu trách nhiệm mở rộng cửa sổ, con trỏ trái chịu trách nhiệm thu hẹp cửa sổ khi trạng thái cửa sổ không hợp lệ. Mỗi ký tự tối đa vào cửa sổ một lần và ra khỏi cửa sổ một lần, vì vậy độ phức tạp thời gian là `O(n)`.

Cửa sổ trượt biến thiên thường tuân theo một nhịp điệu cố định:

1. Con trỏ phải thêm phần tử mới vào, cập nhật trạng thái cửa sổ.
2. Khi cửa sổ không thỏa mãn điều kiện đề bài, liên tục di chuyển con trỏ trái và đồng bộ cập nhật trạng thái.
3. Cập nhật kết quả tại vị trí cửa sổ thỏa mãn yêu cầu đề bài.

Thời điểm cập nhật kết quả giữa bài toán tìm dài nhất và tìm ngắn nhất là khác nhau:

- Tìm cửa sổ hợp lệ dài nhất: Thường cập nhật kết quả sau khi cửa sổ đã phục hồi về trạng thái hợp lệ.
- Tìm cửa sổ thỏa mãn điều kiện ngắn nhất: Thường cập nhật kết quả ngay khi cửa sổ vừa thỏa mãn điều kiện, sau đó tiếp tục thu hẹp biên trái để tìm phương án ngắn hơn.

Ví dụ trong bài "Minimum Window Substring", khi cửa sổ đã bao phủ đủ các ký tự mục tiêu, ta cập nhật kết quả trước rồi mới cố gắng thu hẹp cửa sổ; còn trong bài "Longest Substring Without Repeating Characters", khi cửa sổ xuất hiện ký tự trùng lặp, ta phải thu hẹp cho đến khi hợp lệ rồi mới cập nhật kết quả.

## Fixed Sliding Window (Cửa sổ trượt kích thước cố định)

Cửa sổ cố định phù hợp với các bài toán yêu cầu "mảng con/chuỗi con có độ dài k":

```java
int maxSum(int[] nums, int k) {
    int window = 0;
    for (int i = 0; i < k; i++) {
        window += nums[i];
    }
    int ans = window;
    for (int right = k; right < nums.length; right++) {
        window += nums[right];
        window -= nums[right - k];
        ans = Math.max(ans, window);
    }
    return ans;
}
```

Trọng tâm của cửa sổ cố định là: phía bên phải nhận thêm một phần tử thì phía bên trái phải loại bỏ một phần tử tương ứng.

Cửa sổ cố định không cần vòng lặp `while` để thu hẹp, vì độ dài cửa sổ luôn giữ nguyên. Nó giống như một phép thống kê trượt (rolling calculation):

- Phần tử mới đi vào cửa sổ.
- Phần tử cũ rời khỏi cửa sổ bị loại bỏ.
- Cập nhật kết quả của cửa sổ hiện tại.

Nếu bên trong cửa sổ cần duy trì giá trị lớn nhất hoặc nhỏ nhất, biến thông thường là không đủ, thường phải dùng Monotonic Queue (hàng đợi đơn điệu). Ví dụ trong bài "Sliding Window Maximum", hàng đợi lưu trữ các chỉ số (index) có khả năng trở thành giá trị lớn nhất, và phần tử đầu hàng đợi luôn là giá trị lớn nhất của cửa sổ hiện tại.

## Lộ trình viết code từng bước trong phỏng vấn

Với bài toán Two Pointers và Sliding Window, điều kỵ nhất khi phỏng vấn là ý nghĩa con trỏ bị thay đổi giữa chừng. Bạn nên viết code theo trình tự sau:

1. Xác định dạng bài: Là thu hẹp hai đầu, đuổi bắt nhanh chậm, ghi đè in-place, hay cửa sổ liên tục.
2. Xác định rõ ý nghĩa con trỏ: `left`, `right`, `slow`, `fast`, `write` trỏ vào vị trí nào.
3. Xác định rõ trạng thái cửa sổ: Bên trong cửa sổ duy trì tổng, bộ đếm, giá trị lớn nhất, hay số lượng điều kiện khớp.
4. Xác định điều kiện di chuyển: Khi nào con trỏ phải mở rộng, khi nào con trỏ trái thu hẹp.
5. Xác định thời điểm cập nhật kết quả: Sau khi hợp lệ thì cập nhật dài nhất, khi vừa thỏa mãn điều kiện thì cập nhật ngắn nhất.

Một câu tóm tắt phân biệt bài toán dài nhất và ngắn nhất: **Bài toán tìm dài nhất thường sửa cửa sổ cho hợp lệ rồi mới cập nhật kết quả; bài toán tìm ngắn nhất thường ghi nhận kết quả trước rồi tiếp tục thu hẹp.**

## Phân tích bài toán tiêu biểu: Minimum Window Substring

[76. Minimum Window Substring](https://leetcode.cn/problems/minimum-window-substring/) là bài toán kiểm tra chi tiết kỹ năng Sliding Window toàn diện nhất. Đề bài yêu cầu tìm chuỗi con ngắn nhất trong `s` sao cho bao phủ toàn bộ các ký tự và số lần xuất hiện tương ứng trong `t`.

Mấu chốt của bài này không phải là biết dùng cửa sổ hay không, mà là có thể trình bày rõ ràng hai bộ đếm sau:

- `need`: Mỗi ký tự trong chuỗi mục tiêu `t` cần bao nhiêu lần xuất hiện.
- `window`: Mỗi ký tự trong cửa sổ hiện tại đã xuất hiện bao nhiêu lần.
- `valid`: Có bao nhiêu loại ký tự đã đạt đủ số lần yêu cầu.

Khi `valid == need.size()`, tức là cửa sổ hiện tại đã bao phủ toàn bộ chuỗi `t`, lúc này ta cập nhật kết quả và cố gắng thu hẹp biên trái.

```java
String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    for (char c : t.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }

    int left = 0;
    int valid = 0;
    int start = 0;
    int minLen = Integer.MAX_VALUE;

    for (int right = 0; right < s.length(); right++) {
        char in = s.charAt(right);
        if (need.containsKey(in)) {
            window.put(in, window.getOrDefault(in, 0) + 1);
            if (window.get(in).equals(need.get(in))) {
                valid++;
            }
        }

        while (valid == need.size()) {
            if (right - left + 1 < minLen) {
                start = left;
                minLen = right - left + 1;
            }
            char out = s.charAt(left);
            left++;
            if (need.containsKey(out)) {
                if (window.get(out).equals(need.get(out))) {
                    valid--;
                }
                window.put(out, window.get(out) - 1);
            }
        }
    }

    return minLen == Integer.MAX_VALUE ? "" : s.substring(start, start + minLen);
}
```

Có hai điểm rất dễ viết sai:

- `valid--` phải diễn ra trước khi giảm `window.get(out)`, vì tại thời điểm này cửa sổ vẫn vừa đủ thỏa mãn điều kiện.
- Việc cập nhật kết quả phải đặt bên trong `while (valid == need.size())`, vì chỉ khi cửa sổ hiện tại đã bao phủ đầy đủ `t` thì mới đủ điều kiện tham gia so sánh tìm đáp án ngắn nhất.

## Minh họa quy trình và các trường hợp biên

Lấy bài toán "Chuỗi con dài nhất không chứa ký tự trùng lặp" làm ví dụ, sự thay đổi cửa sổ với chuỗi `abba` như sau:

| Ký tự con trỏ phải | Cửa sổ sau khi thêm | Hợp lệ không | Con trỏ trái di chuyển thế nào | Chiều dài lớn nhất hiện tại |
| ---------- | ---------- | -------- | ------------------------------------- | -------- |
| `a` | `a` | Hợp lệ | Đứng yên | 1 |
| `b` | `ab` | Hợp lệ | Đứng yên | 2 |
| `b` | `abb` | Không hợp lệ | Bỏ `a` vẫn không hợp lệ, tiếp tục bỏ ký tự `b` đầu tiên | 2 |
| `a` | `ba` | Hợp lệ | Đứng yên | 2 |

Với Sliding Window, bạn nên kiểm tra ít nhất các trường hợp biên sau:

| Đầu vào | Trọng tâm kiểm tra |
| -------------------- | ------------------------ |
| Chuỗi rỗng hoặc mảng rỗng | Có trả về 0 trực tiếp không |
| Toàn bộ ký tự giống nhau | Biên trái có thu hẹp liên tục không |
| Không có ký tự nào trùng lặp | Kết quả có cập nhật được đến toàn bộ độ dài chuỗi không |
| Cửa sổ tối ưu nằm ở đầu hoặc cuối chuỗi | Thời điểm cập nhật kết quả có chính xác không |

Lỗi thường gặp khi viết code:

```java
if (count.get(c) > 1) {
    left++; // Sai: chỉ di chuyển 1 lần chưa chắc đã phục hồi được cửa sổ hợp lệ
}
```

Khi thu hẹp cửa sổ biến thiên, thông thường phải dùng vòng lặp `while` cho đến khi cửa sổ thỏa mãn lại điều kiện. Nếu chỉ di chuyển một lần, khi gặp các chuỗi như `abba`, `aaabc` sẽ rất dễ bị sai.

## Các lỗi thường gặp (Pitfalls)

- Với bài toán Two Pointers, cần xác định rõ ý nghĩa của cả hai con trỏ ngay từ đầu, không vừa viết vừa đoán mò.
- Trong Sliding Window, thời điểm cập nhật kết quả phụ thuộc vào câu hỏi yêu cầu dài nhất hay ngắn nhất.
- Khi cửa sổ thu hẹp, các trạng thái đếm, tổng, số điều kiện khớp trong cửa sổ đều phải được đồng bộ cập nhật.
- Với Fast-Slow Pointers trên LinkedList, phải kiểm tra `fast` và `fast.next` trước khi truy cập `fast.next.next`.
- Những bài như Three Sum, sau khi sắp xếp thì logic loại bỏ trùng lặp cần được xử lý riêng biệt và cẩn thận.

## Câu hỏi tự kiểm tra tần suất cao

- Tại sao các bài toán Two Pointers thường có độ phức tạp `O(n)` thay vì `O(n^2)` như hai vòng lặp lồng nhau?
- Tại sao Three Sum cần sắp xếp trước? Việc loại bỏ phần tử trùng lặp diễn ra ở những vị trí nào?
- Khi dùng Fast-Slow Pointers tìm trung điểm LinkedList, với danh sách độ dài chẵn thì con trỏ dừng ở trung điểm trước hay trung điểm sau?
- Khi nào Sliding Window dùng `if` để thu hẹp, khi nào bắt buộc phải dùng `while`?
- Thời điểm cập nhật đáp án giữa bài toán cửa sổ dài nhất và cửa sổ ngắn nhất khác nhau như thế nào?

## Bài tập rèn luyện đề xuất

- [26. Remove Duplicates from Sorted Array](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)
- [15. 3Sum](https://leetcode.cn/problems/3sum/)
- [141. Linked List Cycle](https://leetcode.cn/problems/linked-list-cycle/)
- [3. Longest Substring Without Repeating Characters](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)
- [76. Minimum Window Substring](https://leetcode.cn/problems/minimum-window-substring/)

<!-- @include: @article-footer.snippet.md -->
