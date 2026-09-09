---
title: "Tổng hợp bài toán phỏng vấn Quy hoạch động (Dynamic Programming): Chuyển trạng thái, Knapsack, Subsequence và Java Template"
description: "Tổng hợp bài toán phỏng vấn quy hoạch động, giải thích định nghĩa trạng thái, chuyển trạng thái, khởi tạo, thứ tự duyệt, 0-1 Knapsack, Complete Knapsack, Subsequence, Interval DP và các bài toán LeetCode tần suất cao."
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Quy hoạch động,DP,Dynamic Programming,Chuyển trạng thái,Bài toán cái túi,Knapsack,0-1 Knapsack,Complete Knapsack,Subsequence,Interval DP,Java DP,LeetCode DP
---

Độ khó của Quy hoạch động (Dynamic Programming - DP) không phải vì code nhất định dài, mà vì một khi định nghĩa trạng thái bị sai thì toàn bộ phương trình chuyển trạng thái, giá trị khởi tạo và thứ tự duyệt phía sau đều sẽ sai theo.

Trong phỏng vấn, đừng vừa vào đã học vẹt template. Trước tiên hãy tự hỏi bản thân hai câu hỏi: Vấn đề này có thể chia thành các bài toán con (subproblems) không? Đáp án hiện tại có phụ thuộc vào các đáp án đã tính toán phía trước không? Nếu cả hai câu hỏi đều đúng, lúc đó mới cân nhắc áp dụng DP.

## Trọng tâm khảo sát trong phỏng vấn

- Trình bày rõ ràng ý nghĩa của `dp[i]` hoặc `dp[i][j]`.
- Viết được phương trình chuyển trạng thái (state transition equation).
- Xử lý chính xác giá trị khởi tạo và thứ tự duyệt.
- Phán đoán xem có thể nén không gian (space optimization) được không.
- Phân biệt các dạng bài phổ biến: Knapsack (Cái túi), Subsequence (Dãy con), Interval (Khoảng/Đoạn).

## Khi nào nên cân nhắc Quy hoạch động?

Không phải cứ thấy bài toán tìm "giá trị lớn nhất/nhỏ nhất" là áp dụng DP. Dấu hiệu phán đoán đáng tin cậy hơn là nhìn vào hai điều kiện:

1. Bài toán có thể chia thành các bài toán con cùng loại có quy mô nhỏ hơn (Optimal Substructure).
2. Các bài toán con có bị tính toán lặp đi lặp lại nhiều lần không (Overlapping Subproblems).

Ví dụ với dãy số Fibonacci: `f(n)` phụ thuộc vào `f(n - 1)` và `f(n - 2)`, mà `f(n - 2)` sẽ bị tính toán lặp lại nhiều lần trong cây đệ quy. Lưu trữ lại các kết quả trung gian này chính là bản chất của DP.

Trong phỏng vấn, bạn có thể bắt đầu trình bày từ đệ quy vét cạn (brute-force recursion), sau đó chỉ ra chỗ nào bị tính toán trùng lặp, và cuối cùng cải tiến đệ quy thành tìm kiếm có nhớ (Memoization) hoặc quy hoạch động dạng bảng (Tabulation). Quá trình này giúp người phỏng vấn tin tưởng rằng bạn thực sự hiểu rõ bản chất vấn đề hơn là việc chỉ học thuộc lòng mảng `dp`.

## Phương pháp 5 bước giải bài toán DP

1. Định nghĩa trạng thái: `dp[i]` rốt cuộc đại diện cho điều gì.
2. Viết phương trình chuyển trạng thái: Trạng thái hiện tại được suy ra từ những trạng thái nào trước đó.
3. Khởi tạo giá trị ban đầu: Khi chưa có trạng thái phía trước thì đáp án cơ sở là gì.
4. Xác định thứ tự duyệt: Tính toán trạng thái nào trước, trạng thái nào sau.
5. Kiểm tra mẫu thử (Dry run): Dùng một test case nhỏ để chạy tay qua mảng `dp`.

Trong đó, quan trọng nhất là bước 1. Một khi ý nghĩa của `dp[i]` bị mơ hồ thì toàn bộ code phía sau sẽ trở thành việc đoán mò và thử sai.

Một định nghĩa trạng thái tốt thường thỏa mãn:

- Bao phủ được câu hỏi mà đề bài yêu cầu.
- Có thể suy diễn ra được từ các trạng thái nhỏ hơn.
- Số chiều càng ít càng tốt, nhưng đừng vì tiết kiệm không gian mà làm ý nghĩa trạng thái bị rối loạn.

## Ví dụ DP một chiều: Bài toán leo thang (Climbing Stairs)

```java
int climbStairs(int n) {
    if (n <= 2) {
        return n;
    }
    int prev2 = 1;
    int prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

Ý nghĩa trạng thái: Có bao nhiêu cách để bước lên bậc thứ `i`. Phương trình chuyển trạng thái: `dp[i] = dp[i - 1] + dp[i - 2]`.

Bài này có thể suy luận trực tiếp từ tư duy đệ quy:

```text
Bước cuối cùng để lên đến bậc thứ i: Hoặc là từ bậc i-1 bước lên 1 bước, hoặc từ bậc i-2 bước lên 2 bước.
```

Vì vậy, `dp[i]` chỉ phụ thuộc vào đúng hai trạng thái liền trước, có thể nén mảng thành hai biến. Tiền đề của việc nén không gian là bạn chắc chắn các trạng thái cũ phía trước sẽ không bao giờ được sử dụng lại nữa.

## Template 0-1 Knapsack (Balo 0-1)

Mỗi đồ vật chỉ được chọn tối đa một lần:

```java
int knapsack01(int[] weights, int[] values, int capacity) {
    int[] dp = new int[capacity + 1];
    for (int i = 0; i < weights.length; i++) {
        for (int j = capacity; j >= weights[i]; j--) {
            dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]);
        }
    }
    return dp[capacity];
}
```

Dung lượng balo (capacity) bắt buộc phải duyệt theo thứ tự ngược (từ lớn về nhỏ), để tránh một đồ vật bị sử dụng nhiều lần trong cùng một lượt.

Duyệt ngược là câu hỏi thường xuyên được hỏi nhất về 0-1 Knapsack. Giả sử duyệt xuôi dung lượng, khi tính `dp[j]` có thể sẽ sử dụng `dp[j - weight]` vừa mới được cập nhật trong chính lượt này, tương đương với việc đồ vật đó được chọn nhiều lần. Khi đó bài toán sẽ biến thành Complete Knapsack (Balo hoàn toàn).

Cách hỏi điển hình của 0-1 Knapsack không nhất thiết phải dùng từ "cái túi/balo". Ví dụ bài toán "Có thể chia mảng thành hai tập con có tổng bằng nhau không?", thực chất có thể chuyển đổi thành: Liệu có thể chọn một số phần tử từ mảng sao cho tổng của chúng bằng đúng một nửa tổng mảng ban đầu.

## Template Complete Knapsack (Balo hoàn toàn)

Mỗi đồ vật có thể được chọn nhiều lần không giới hạn:

```java
int unboundedKnapsack(int[] weights, int[] values, int capacity) {
    int[] dp = new int[capacity + 1];
    for (int i = 0; i < weights.length; i++) {
        for (int j = weights[i]; j <= capacity; j++) {
            dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]);
        }
    }
    return dp[capacity];
}
```

Dung lượng balo duyệt theo thứ tự xuôi, cho phép đồ vật hiện tại được sử dụng lặp lại nhiều lần.

Trong Complete Knapsack, việc duyệt xuôi dung lượng chính là để cho phép đồ vật hiện tại được dùng lại. Ví dụ trong bài toán đổi tiền xu (Coin Change), mỗi loại mệnh giá tiền có thể dùng nhiều lần, khi tính số tiền lớn hơn có thể chuyển tiếp trạng thái dựa trên chính trạng thái đã sử dụng đồng xu này trước đó.

Nếu đề bài hỏi về "Số tổ hợp" (Combinations) hay "Số hoán vị" (Permutations), thứ tự duyệt của hai vòng lặp cũng sẽ thay đổi:

- Số tổ hợp: Thường duyệt đồ vật (tiền xu) ở vòng ngoài, duyệt dung lượng (số tiền) ở vòng trong.
- Số hoán vị: Thường duyệt dung lượng (số tiền) ở vòng ngoài, duyệt đồ vật (tiền xu) ở vòng trong.

Phần này trong phỏng vấn không hẳn sẽ hỏi quá sâu, nhưng là mấu chốt khi giải các bài như Coin Change II.

## Các dạng bài thường gặp

| Dạng bài | Thiết kế trạng thái | Bài toán tiêu biểu |
| --------------- | ------------------------------------------------------ | ------------- |
| Leo thang / Trộm nhà (House Robber) | `dp[i]` biểu thị giá trị tối ưu của `i` vị trí đầu tiên | 70, 198 |
| Knapsack (Balo) | `dp[j]` biểu thị giá trị tối ưu hoặc số phương án khi dung lượng là `j` | 416, 518, 322 |
| Dãy con (Subsequence) | `dp[i]` hoặc `dp[i][j]` biểu thị kết quả kết thúc tại vị trí nào hoặc của hai tiền tố | 300, 1143 |
| Palindrome (Đối xứng) | `dp[i][j]` biểu thị đoạn `[i, j]` có đối xứng hay giá trị tối ưu không | 647, 516 |
| Đường đi trên lưới (Path) | `dp[i][j]` biểu thị kết quả khi đi đến ô `(i, j)` | 62, 64 |

## Nên chọn Memoization (Đệ quy có nhớ) hay Tabulation (Quy hoạch động lặp bảng)?

Cả hai cách viết đều lưu lại đáp án của các bài toán con:

| Cách viết | Đặc điểm | Phù hợp cho |
| ---------- | ---------------------------- | -------------------------- |
| Memoization (Đệ quy có nhớ) | Xuất phát từ trạng thái mục tiêu gọi đệ quy xuống, tính toán theo nhu cầu | Chuyển trạng thái phức tạp, tư duy đệ quy tự nhiên hơn |
| Tabulation (Lặp điền bảng) | Điền bảng từ trạng thái nhỏ dần lên trạng thái lớn | Thứ tự duyệt rõ ràng, dễ dàng nén không gian bộ nhớ |

Nếu ban đầu chưa hình dung rõ thứ tự duyệt bảng, bạn có thể viết Memoization trước. Khi quan hệ trạng thái đã sáng tỏ thì chuyển đổi sang Tabulation. Rất nhiều bài Tree DP (DP trên cây) hay Interval DP (DP trên đoạn) viết bằng Memoization sẽ trực quan và ít bị lỗi biên hơn.

## Lộ trình viết code từng bước trong phỏng vấn

Với bài toán DP, không nên bắt đầu ngay bằng việc viết code. Khi phỏng vấn viết tay hoặc live coding, hãy làm rõ 4 câu sau đây trước:

1. Ý nghĩa mảng `dp` là gì, và đáp án cuối cùng nằm ở vị trí nào.
2. Trạng thái hiện tại phụ thuộc vào những trạng thái cũ nào, tại sao các trạng thái cũ đó chắc chắn đã được tính toán xong.
3. Tại sao giá trị khởi tạo lại được gán như vậy, đặc biệt là `0`, `1`, hoặc dương/âm vô cùng đại diện cho điều gì.
4. Tại sao thứ tự duyệt không vô tình sử dụng các trạng thái chưa được tính hoặc các trạng thái không được phép tái sử dụng.

Nếu không trình bày rõ được 4 câu này, code phần lớn chỉ là viết theo trí nhớ, khi gặp bài biến thể sẽ rất dễ bị rối loạn.

## Phân tích bài toán tiêu biểu: Coin Change (Đổi tiền xu)

[322. Coin Change](https://leetcode.cn/problems/coin-change/) là bài toán Complete Knapsack cực kỳ phổ biến trong các buổi phỏng vấn. Đề bài cho các mệnh giá tiền xu và số tiền mục tiêu `amount`, hỏi cần ít nhất bao nhiêu đồng xu để ghép thành số tiền đó, mỗi loại đồng xu được dùng số lần không giới hạn.

Định nghĩa trạng thái có thể trình bày như sau:

```text
dp[j] biểu thị số lượng đồng xu ít nhất cần dùng để tạo thành số tiền j.
```

Khởi tạo là mấu chốt của bài này: `dp[0] = 0`, vì để tạo thành số tiền 0 thì cần 0 đồng xu; các số tiền còn lại ban đầu được gán một giá trị lớn không thể đạt tới (`amount + 1`), biểu thị tạm thời chưa có cách ghép hợp lệ.

Trong code có dùng `Arrays.fill`, cần import `java.util.Arrays`.

```java
int coinChange(int[] coins, int amount) {
    int max = amount + 1;
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, max);
    dp[0] = 0;

    for (int coin : coins) {
        for (int j = coin; j <= amount; j++) {
            dp[j] = Math.min(dp[j], dp[j - coin] + 1);
        }
    }

    return dp[amount] == max ? -1 : dp[amount];
}
```

Tại sao dung lượng duyệt theo thứ tự xuôi? Vì mỗi đồng xu có thể được dùng nhiều lần. Khi tính `dp[j]`, ta sử dụng `dp[j - coin]`. Nếu `dp[j - coin]` đã được cập nhật bởi chính đồng xu này trong lượt hiện tại, điều đó có nghĩa là đồng xu hiện tại có thể tiếp tục được sử dụng, hoàn toàn khớp với bài toán Complete Knapsack.

Nếu đề bài đổi thành "mỗi đồng xu chỉ được dùng đúng một lần", dung lượng sẽ phải duyệt ngược. Hướng duyệt không phải là vấn đề hình thức, mà là công cụ để kiểm soát xem cùng một vật phẩm có được phép tái sử dụng trong quá trình chuyển trạng thái hay không.

## So sánh các định nghĩa trạng thái dễ nhầm lẫn

Trong DP, nguyên nhân thất bại thường không phải là không biết viết chuyển trạng thái, mà do chọn sai ý nghĩa trạng thái. Những nhóm trạng thái dưới đây nhìn rất giống nhau nhưng cách viết hoàn toàn khác biệt:

| Dạng bài | Ý nghĩa trạng thái | Điểm trọng tâm khi chuyển trạng thái |
| -------------- | ---------------------------------------- | ------------------------------ |
| Longest Increasing Subsequence (LIS) | `dp[i]` biểu thị độ dài LIS kết thúc tại `nums[i]` | Bắt buộc phải chọn `nums[i]`, tìm kiếm ngược về trước giá trị nhỏ hơn |
| House Robber (Trộm nhà) | `dp[i]` biểu thị số tiền lớn nhất trong `i` ngôi nhà đầu | Ngôi nhà thứ `i` chọn trộm hoặc không trộm |
| Longest Common Subsequence (LCS) | `dp[i][j]` biểu thị độ dài LCS của hai tiền tố | So sánh ký tự cuối cùng của hai tiền tố |
| Palindromic Substring | `dp[i][j]` biểu thị đoạn `[i, j]` có phải là chuỗi đối xứng không | Phụ thuộc vào đoạn con bên trong `[i + 1, j - 1]` |

Trong phỏng vấn, bạn nên chủ động nói rõ một câu: Ở đây `dp[i]` là "kết thúc tại vị trí i", chứ không phải là "kết quả tối ưu trong số i phần tử đầu tiên". Câu nói này sẽ giúp bạn tránh được rất nhiều lỗi logic trong các bài toán dãy con.

## Minh họa quy trình và các trường hợp biên

Lấy bài toán leo thang làm ví dụ, với `n = 5`, sự biến đổi trạng thái như sau:

| `i` | `dp[i - 2]` | `dp[i - 1]` | `dp[i]` |
| --- | ----------- | ----------- | ------- |
| 3 | 1 | 2 | 3 |
| 4 | 2 | 3 | 5 |
| 5 | 3 | 5 | 8 |

Điểm cần chú ý ở bảng này không phải là bản thân các con số, mà là trạng thái chỉ phụ thuộc vào đúng hai vị trí liền trước, vì vậy hoàn toàn có thể nén thành 2 biến cục bộ thay vì dùng cả mảng.

Với các bài toán DP, bạn nên kiểm tra các trường hợp biên sau:

| Đầu vào | Trọng tâm kiểm tra |
| ---------------- | ------------------------ |
| `n = 0` hoặc mảng rỗng | Khởi tạo đã bao phủ trường hợp này chưa |
| Chỉ có 1 phần tử | Có bị truy cập mảng vượt biên tại `dp[1]` không |
| Không thể ghép thành mục tiêu | Giá trị ban đầu có thể hiện rõ trạng thái "không thể đạt tới" hay không |
| Đếm số phương án | Khởi tạo và thứ tự duyệt có chuẩn xác không |

Lỗi thường gặp khi viết code:

```java
for (int j = weights[i]; j <= capacity; j++) {
    dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]); // Sai trong 0-1 Knapsack
}
```

Trong 0-1 Knapsack, dung lượng bắt buộc phải duyệt ngược, nếu không trạng thái vừa cập nhật trong lượt này sẽ bị tái sử dụng, tương đương với việc một đồ vật bị chọn nhiều lần.

## Các lỗi thường gặp (Pitfalls)

- Ý nghĩa của `dp` không được thay đổi giữa chừng.
- Khởi tạo không phải lúc nào cũng điền 0 bừa bãi, phải dựa theo đúng định nghĩa trạng thái.
- 0-1 Knapsack duyệt dung lượng ngược, Complete Knapsack duyệt dung lượng xuôi.
- Khởi tạo khi đếm số phương án khác với khởi tạo khi tìm giá trị lớn nhất/nhỏ nhất.
- Bài toán Subsequence thường cần phân biệt rõ giữa "kết thúc tại i" và "trong phạm vi i phần tử đầu tiên".

## Câu hỏi tự kiểm tra tần suất cao

- Tại sao bước đầu tiên của DP nhất định phải là định nghĩa trạng thái?
- Sự khác biệt giữa Memoization và Tabulation là gì? Khi nào viết Memoization trước sẽ an toàn hơn?
- Tại sao trong 0-1 Knapsack, dung lượng balo phải duyệt theo chiều ngược?
- Tại sao trong Complete Knapsack, dung lượng balo lại có thể duyệt theo chiều xuôi?
- Khi `dp[i]` biểu thị "kết thúc tại i" so với biểu thị "trong i phần tử đầu tiên", phương trình chuyển trạng thái khác nhau ra sao?
- Khi tìm số lần ít nhất, giá trị lớn nhất, và số phương án, việc khởi tạo giá trị ban đầu cần chú ý những gì?

## Bài tập rèn luyện đề xuất

- [70. Climbing Stairs](https://leetcode.cn/problems/climbing-stairs/)
- [198. House Robber](https://leetcode.cn/problems/house-robber/)
- [322. Coin Change](https://leetcode.cn/problems/coin-change/)
- [416. Partition Equal Subset Sum](https://leetcode.cn/problems/partition-equal-subset-sum/)
- [300. Longest Increasing Subsequence](https://leetcode.cn/problems/longest-increasing-subsequence/)
- [1143. Longest Common Subsequence](https://leetcode.cn/problems/longest-common-subsequence/)

<!-- @include: @article-footer.snippet.md -->
