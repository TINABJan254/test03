---
title: "Tổng hợp bài toán phỏng vấn Thuật toán tham lam (Greedy): Khoảng tham lam, Jump Game và Phương pháp chứng minh"
description: "Tổng hợp bài toán phỏng vấn thuật toán tham lam, giải thích nhận diện dạng bài toán tham lam, tham lam sắp xếp, tham lam khoảng, trò chơi nhảy, tư duy chứng minh tham lam và các bài toán LeetCode tần suất cao."
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Thuật toán tham lam,Greedy,Greedy template,Tham lam khoảng,Tham lam sắp xếp,Trò chơi nhảy,Jump Game,Chứng minh tham lam,LeetCode Greedy,Bài toán phỏng vấn thuật toán
---

Code của thuật toán tham lam (Greedy Algorithm) thường không dài, nhưng điểm khó nằm ở việc chứng minh tại sao lựa chọn cục bộ hiện tại lại không làm ảnh hưởng đến tính tối ưu toàn cục. Trong phỏng vấn, nếu chỉ viết code mà không giải thích chiến lược tham lam, bạn sẽ rất dễ bị người phỏng vấn hỏi dồn đến mức bế tắc.

Bạn có thể ghi nhớ một nguyên tắc phán đoán: Nếu bài toán có thể thông qua việc sắp xếp hoặc duy trì một biên tối ưu hiện tại, ở mỗi bước đưa ra một lựa chọn cục bộ và lựa chọn này không phá vỡ nghiệm tối ưu về sau, thì có thể thử tiếp cận theo hướng tham lam.

## Trọng tâm khảo sát trong phỏng vấn

- Tìm ra chiến lược tham lam chuẩn xác.
- Sử dụng phương pháp tráo đổi (exchange argument), phản chứng hoặc biên trực giác để chứng minh tính hợp lý của chiến lược.
- Xử lý điều kiện duyệt sau khi sắp xếp dữ liệu.
- Phân biệt rõ ràng giữa Greedy và Quy hoạch động (Dynamic Programming).

## Tư duy giải bài toán Greedy như thế nào?

Điều kỵ nhất của bài toán tham lam là "chọn theo cảm tính". Trước khi viết code, ít nhất bạn phải nói rõ được 2 điều:

1. Mỗi bước tham lam cái gì? Ví dụ: Thời gian kết thúc sớm nhất, vị trí hiện tại có thể nhảy xa nhất, hoặc lợi nhuận hiện tại là số dương.
2. Tại sao lựa chọn này sẽ không làm kết quả phía sau bị tệ đi?

Chứng minh không nhất thiết phải dùng toán học quá hình thức, nhưng phải giải thích được sự đánh đổi (trade-off). Ví dụ trong bài toán xếp lịch khoảng thời gian (Interval Scheduling), ta chọn khoảng có thời gian kết thúc sớm nhất vì nó để lại không gian lựa chọn lớn nhất cho các khoảng phía sau; nếu chọn một khoảng kết thúc muộn hơn thì số lượng khoảng chọn được chắc chắn không thể nhiều hơn.

## Các dạng bài thường gặp

| Dạng bài | Chiến lược tham lam | Bài toán tiêu biểu |
| ---------- | ------------------------------ | ---------------------------------- |
| Bài toán phân phối (Allocation) | Ưu tiên thỏa mãn đối tượng dễ thỏa mãn nhất | Chia bánh quy (Assign Cookies) |
| Mua bán cổ phiếu | Cộng dồn tất cả các khoản lợi nhuận dương | Thời điểm mua bán cổ phiếu tốt nhất II |
| Bài toán nhảy (Jump Game) | Duy trì vị trí xa nhất hiện tại có thể đạt tới | Jump Game |
| Bài toán khoảng (Intervals) | Sắp xếp theo đầu mút phải hoặc đầu mút trái | Non-overlapping Intervals, Dùng ít tên nhất bắn nổ bóng bay |
| Tái cấu trúc chuỗi | Duy trì số lần còn lại hoặc phạm vi bao phủ xa nhất | Phân chia khoảng ký tự (Partition Labels) |

Greedy rất thường xuyên xuất hiện cùng với việc sắp xếp (Sort), bởi vì sắp xếp giúp cho "lựa chọn tối ưu hiện tại" trở nên rõ ràng. Bài toán khoảng thường sắp xếp theo điểm đầu hoặc điểm cuối; bài toán phân phối thường sắp xếp cả nhu cầu lẫn tài nguyên rồi dùng Two Pointers để ghép cặp.

## Template Jump Game (Trò chơi nhảy)

```java
boolean canJump(int[] nums) {
    int farthest = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > farthest) {
            return false;
        }
        farthest = Math.max(farthest, i + nums[i]);
    }
    return true;
}
```

`farthest` biểu thị vị trí xa nhất hiện tại có thể chạm tới. Khi duyệt đến vị trí `i`, nếu `i > farthest`, điều đó có nghĩa là vị trí hiện tại hoàn toàn không thể nhảy tới được.

Điểm tham lam của bài này là: Không quan tâm cụ thể từ bước nào nhảy đến `i`, chỉ quan tâm đến phạm vi xa nhất hiện tại có thể bao phủ. Chỉ cần vị trí hiện tại nằm trong phạm vi bao phủ, ta có thể dùng nó để tiếp tục mở rộng phạm vi bao phủ tối đa.

Bài toán "Jump Game II" yêu cầu thêm số bước nhảy ít nhất. Bài này sẽ duy trì hai biên:

- `curEnd`: Vị trí xa nhất mà số bước hiện tại có thể bao phủ tới.
- `farthest`: Vị trí xa nhất có thể tới nếu nhảy thêm một bước nữa từ phạm vi hiện tại.

Khi duyệt đến `curEnd`, tức là phạm vi của số bước hiện tại đã dùng hết, bắt buộc phải nhảy thêm một bước nữa và cập nhật `curEnd = farthest`.

## Template Interval Greedy (Khoảng tham lam)

Lấy bài toán tìm các khoảng không trùng lặp (Non-overlapping Intervals) làm ví dụ: Sắp xếp các khoảng theo chiều tăng dần của đầu mút phải (end point), mỗi lần luôn giữ lại khoảng kết thúc sớm nhất:

```java
int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length == 0) {
        return 0;
    }
    Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));
    int count = 1;
    int end = intervals[0][1];
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] >= end) {
            count++;
            end = intervals[i][1];
        }
    }
    return intervals.length - count;
}
```

Kết thúc càng sớm thì không gian để lại cho các khoảng phía sau càng lớn, đây chính là lựa chọn cốt lõi của dạng bài này.

Điểm dễ sai nhất trong bài toán khoảng là chọn sai trường để sắp xếp. Một vài quy tắc lựa chọn phổ biến:

- Muốn chọn nhiều khoảng không trùng nhau nhất: Sắp xếp tăng dần theo đầu mút phải (`end`).
- Muốn gộp các khoảng lại với nhau: Sắp xếp tăng dần theo đầu mút trái (`start`).
- Dùng ít mũi tên nhất để bắn nổ bóng bay: Sắp xếp tăng dần theo đầu mút phải, cố gắng dùng mũi tên hiện tại bắn xuyên nhiều quả bóng nhất.

Nếu một chiến lược tham lam khó giải thích, hãy thử tìm phản ví dụ (counterexample) bằng các test case nhỏ. Ví dụ "mỗi lần chọn khoảng có độ dài ngắn nhất" nghe có vẻ hợp lý, nhưng không thể đảm bảo chọn được số khoảng không trùng nhau nhiều nhất.

## Phân tích bài toán tiêu biểu: Dùng ít tên nhất bắn nổ bóng bay

[452. Minimum Number of Arrows to Burst Balloons](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/) là bài toán kinh điển của Interval Greedy. Đề bài cho một tập hợp các quả bóng dạng khoảng tọa độ `[start, end]`, một mũi tên bắn tại tọa độ `x` sẽ làm nổ quả bóng nếu `start <= x <= end`. Yêu cầu tìm số mũi tên ít nhất để bắn nổ toàn bộ bóng bay.

Điểm tham lam của bài này là: **Mỗi lần bắn mũi tên vào đúng biên phải xa nhất của quả bóng hiện tại**. Đầu tiên sắp xếp theo đầu mút phải tăng dần, mũi tên đầu tiên đặt tại đầu mút phải của quả bóng thứ nhất. Với các quả bóng tiếp theo, nếu đầu mút trái `<= arrow`, tức là mũi tên này vẫn bắn trúng nó; nếu đầu mút trái `> arrow`, tức là mũi tên hiện tại không thể với tới quả bóng này nữa, bắt buộc phải dùng thêm một mũi tên mới và đặt tại đầu mút phải của quả bóng mới đó.

Trong code cần chú ý hai điều kiện biên: Mảng rỗng trả về `0`; comparator sắp xếp không nên viết `a[1] - b[1]` vì có thể bị tràn số (integer overflow) ở các giá trị tọa độ âm lớn, nên dùng `Integer.compare`.

```java
int findMinArrowShots(int[][] points) {
    if (points.length == 0) {
        return 0;
    }
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1]));
    int arrows = 1;
    int arrow = points[0][1];
    for (int i = 1; i < points.length; i++) {
        if (points[i][0] > arrow) {
            arrows++;
            arrow = points[i][1];
        }
    }
    return arrows;
}
```

Nếu test case là `[[10,16],[2,8],[1,6],[7,12]]`, sau khi sắp xếp theo biên phải sẽ thành `[1,6], [2,8], [7,12], [10,16]`. Mũi tên đầu tiên bắn tại `6`, bao phủ được 2 quả bóng đầu tiên; khi gặp `[7,12]`, biên trái của nó đã lớn hơn `6`, bắt buộc thêm mũi tên thứ hai bắn tại `12`, mũi tên này lại bao phủ được `[10,16]`. Kết quả cuối cùng cần 2 mũi tên.

## Phân biệt giữa Greedy và Dynamic Programming

| Tiêu chí so sánh | Greedy (Thuật toán tham lam) | Dynamic Programming (Quy hoạch động) |
| ------------ | ------------------------ | ---------------------- |
| Cách thức quyết định | Quyết định trực tiếp ở bước hiện tại | Phụ thuộc vào nhiều trạng thái trước đó |
| Có nhìn lại lịch sử không | Thông thường không nhìn lại | Cần chuyển tiếp trạng thái |
| Trọng tâm chứng minh | Lựa chọn hiện tại không phá vỡ tối ưu toàn cục | Cấu trúc con tối ưu và bài toán con trùng lặp |
| Bài toán thường gặp | Khoảng, Nhảy, Phân phối | Knapsack, Subsequence, Đường đi trên lưới |

Nếu lựa chọn hiện tại nhìn có vẻ hợp lý nhưng chỉ cần đưa ra một phản ví dụ nhỏ là sai ngay, thì rất có khả năng bài đó phải dùng DP hoặc tìm kiếm (DFS/BFS).

## Các lỗi thường gặp (Pitfalls)

- Bài toán tham lam thường cần sắp xếp trước, nếu sắp xếp sai trường dữ liệu thì kết quả sẽ sai hoàn toàn.
- Với bài toán khoảng, cần chú ý xem đề bài có cho phép các đầu mút bằng nhau hay không (ví dụ `[1,2]` và `[2,3]` có tính là trùng nhau không).
- Trong Jump Game II, thời điểm "tăng số bước nhảy" gắn liền với biên bao phủ hiện tại (`curEnd`).
- Chiến lược tham lam phải giải thích được logic, không thể chỉ nói chung chung là "mỗi lần chọn cái tốt nhất".

## Câu hỏi tự kiểm tra tần suất cao

- Phân biệt giữa Greedy và Quy hoạch động như thế nào?
- Tại sao bài toán khoảng thường sắp xếp theo đầu mút phải?
- Trong Jump Game, tại sao chỉ cần duy trì vị trí xa nhất có thể tới là đủ?
- Trong bài toán tham lam, làm thế nào để dùng phương pháp tráo đổi hoặc phản chứng chứng minh tính đúng đắn của chiến lược?
- Khi các biên của khoảng cho phép bằng nhau, điều kiện so sánh nên viết như thế nào?

## Bài tập rèn luyện đề xuất

- [455. Assign Cookies](https://leetcode.cn/problems/assign-cookies/)
- [122. Best Time to Buy and Sell Stock II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)
- [55. Jump Game](https://leetcode.cn/problems/jump-game/)
- [45. Jump Game II](https://leetcode.cn/problems/jump-game-ii/)
- [435. Non-overlapping Intervals](https://leetcode.cn/problems/non-overlapping-intervals/)
- [763. Partition Labels](https://leetcode.cn/problems/partition-labels/)

<!-- @include: @article-footer.snippet.md -->
