---
title: "Tổng hợp tư duy thuật toán kinh điển (Kèm gợi ý bài tập LeetCode)"
description: "Tổng hợp các tư duy thuật toán tần suất cao như Binary Search, Two Pointers, Sliding Window, DFS/BFS, Backtracking, Dynamic Programming, Greedy, Divide and Conquer, Topological Sort, Union-Find, Bit Manipulation, cung cấp cách nhận diện dạng bài, template, bài toán tiêu biểu và trọng tâm ôn tập."
category: Cơ sở máy tính
tag:
  - Thuật toán
  - LeetCode
  - Phỏng vấn
head:
  - - meta
    - name: keywords
      content: Tư duy thuật toán,Binary Search,Two Pointers,Sliding Window,DFS,BFS,Backtracking,Dynamic Programming,Greedy,Divide and Conquer,Topological Sort,Union-Find,Bit Manipulation,Đề xuất bài tập LeetCode
---

Đừng học vẹt các tư duy thuật toán một cách cô lập. Trong phỏng vấn, những câu hỏi thực tế và hữu ích hơn thường là: Dấu hiệu nào cho thấy tôi nên áp dụng thuật toán này? Điểm nào trong template dễ sai nhất? Nếu người phỏng vấn thay đổi điều kiện, tôi nên bắt đầu điều chỉnh từ biến số hoặc trạng thái nào?

Danh sách bài tập này được tổ chức theo tư duy thuật toán, mỗi thể loại đều cung cấp: "Dấu hiệu nhận biết, Template thông dụng, Bài toán tiêu biểu, Trọng tâm ôn tập". Số lượng bài toán được chọn lọc vừa đủ để đại diện cho template; việc nắm vững và giải thích trôi chảy những bài này sẽ hiệu quả hơn nhiều so với việc cày cuốc cơ học số lượng lớn bài tập.

## Cách sử dụng danh sách bài tập này

Đừng vội vàng làm tuần tự tất cả các bài từ trên xuống dưới ngay từ đầu. Cách chuẩn bị phỏng vấn hiệu quả hơn là: Đọc bài viết template tương ứng trước để chắc chắn bản thân có thể tự viết code phần cốt lõi, sau đó làm các "Bài bắt buộc làm", và cuối cùng dùng các "Bài nâng cao" để kiểm tra các trường hợp biên và biến thể.

| Mục tiêu | Hành động đề xuất |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Xây dựng nhanh các template | Đọc trước các bài template tần suất cao: [Binary Search](./binary-search.md), [Two Pointers & Sliding Window](./two-pointers-and-sliding-window.md), [DFS & BFS](./dfs-bfs.md) |
| Bổ sung phần Tìm kiếm và DP | Tiếp tục đọc [Thuật toán Backtracking](./backtracking.md), [Quy hoạch động (DP)](./dynamic-programming.md), mỗi dạng tự tay viết ít nhất 2 bài cơ bản |
| Rà soát lỗ hổng trước phỏng vấn | Dùng [Thuật toán tham lam (Greedy)](./greedy.md), [Bài toán Top K](./top-k.md), [Union-Find](../data-structure/union-find.md) để củng cố các biến thể phổ biến |
| Đúc kết lại câu trả lời | Với mỗi bài, hãy ghi lại: dấu hiệu nhận diện, ý nghĩa các biến cốt lõi, độ phức tạp, trường hợp biên. Nếu chưa nói rõ được các điểm này thì chứng tỏ bạn chưa thực sự nắm vững bài toán |

## Tìm kiếm nhị phân (Binary Search)

| Hạng mục | Nội dung |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Mảng đã sắp xếp, điều kiện đơn điệu, tìm kiếm biên, tìm giá trị khả thi nhỏ nhất hoặc lớn nhất |
| Template thông dụng | Binary Search cơ bản, biên trái (left boundary), biên phải (right boundary), tìm kiếm nhị phân trên không gian nghiệm |
| Bài bắt buộc làm | [704. Binary Search](https://leetcode.cn/problems/binary-search/), [34. Find First and Last Position of Element in Sorted Array](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) |
| Bài nâng cao | [35. Search Insert Position](https://leetcode.cn/problems/search-insert-position/), [875. Koko Eating Bananas](https://leetcode.cn/problems/koko-eating-bananas/) |
| Trọng tâm ôn tập | Điều kiện vòng lặp, cách tính `mid`, cập nhật biên để không bị rơi vào vòng lặp vô tận |

## Hai con trỏ (Two Pointers)

| Hạng mục | Nội dung |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Mảng đã sắp xếp, sửa đổi tại chỗ (in-place), thu hẹp từ hai đầu vào giữa, con trỏ nhanh chậm đuổi bắt trên LinkedList |
| Template thông dụng | Left-Right Pointers (trái phải), Fast-Slow Pointers (nhanh chậm), Read-Write Pointers (đọc ghi) |
| Bài bắt buộc làm | [26. Remove Duplicates from Sorted Array](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/), [977. Squares of a Sorted Array](https://leetcode.cn/problems/squares-of-a-sorted-array/) |
| Bài nâng cao | [15. 3Sum](https://leetcode.cn/problems/3sum/), [142. Linked List Cycle II](https://leetcode.cn/problems/linked-list-cycle-ii/) |
| Trọng tâm ôn tập | Ý nghĩa con trỏ phải nhất quán, không bỏ sót điều kiện loại bỏ trùng lặp, bài toán LinkedList nên vẽ trước 3 node |

## Cửa sổ trượt (Sliding Window)

| Hạng mục | Nội dung |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Mảng con liên tục, chuỗi con liên tục, dài nhất/ngắn nhất, bên trong cửa sổ thỏa mãn điều kiện nhất định |
| Template thông dụng | Cửa sổ cố định (Fixed Window), cửa sổ biến thiên (Dynamic Window), Map đếm tần suất |
| Bài bắt buộc làm | [3. Longest Substring Without Repeating Characters](https://leetcode.cn/problems/longest-substring-without-repeating-characters/), [209. Minimum Size Subarray Sum](https://leetcode.cn/problems/minimum-size-subarray-sum/) |
| Bài nâng cao | [76. Minimum Window Substring](https://leetcode.cn/problems/minimum-window-substring/), [438. Find All Anagrams in a String](https://leetcode.cn/problems/find-all-anagrams-in-a-string/) |
| Trọng tâm ôn tập | Khi nào mở rộng biên phải, khi nào thu hẹp biên trái, duy trì các biến bên trong cửa sổ như thế nào |

## DFS và BFS

| Hạng mục | Nội dung |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Duyệt cây, duyệt đồ thị, thành phần liên thông trên ma trận lưới, số bước ngắn nhất, duyệt theo tầng (level order) |
| Template thông dụng | Đệ quy DFS, mô phỏng DFS bằng Stack, hàng đợi BFS, BFS theo tầng |
| Bài bắt buộc làm | [102. Binary Tree Level Order Traversal](https://leetcode.cn/problems/binary-tree-level-order-traversal/), [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/) |
| Bài nâng cao | [994. Rotting Oranges](https://leetcode.cn/problems/rotting-oranges/), [127. Word Ladder](https://leetcode.cn/problems/word-ladder/) |
| Trọng tâm ôn tập | Đánh dấu đã truy cập (`visited`), kiểm tra vượt biên, đếm số tầng trong BFS |

## Thuật toán Backtracking (Quay lui)

| Hạng mục | Nội dung |
| -------- | ------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Liệt kê tất cả các phương án, chọn đường đi, tổ hợp, hoán vị, tập con, ràng buộc bàn cờ |
| Template thông dụng | Đường đi `path`, danh sách lựa chọn, tầng đệ quy, thu hồi lựa chọn (backtrack) |
| Bài bắt buộc làm | [77. Combinations](https://leetcode.cn/problems/combinations/), [78. Subsets](https://leetcode.cn/problems/subsets/) |
| Bài nâng cao | [39. Combination Sum](https://leetcode.cn/problems/combination-sum/), [51. N-Queens](https://leetcode.cn/problems/n-queens/) |
| Trọng tâm ôn tập | Ý nghĩa các tham số đệ quy, điều kiện cắt tỉa nhánh đặt trước vòng lặp hay bên trong vòng lặp |

## Quy hoạch động (Dynamic Programming - DP)

| Hạng mục | Nội dung |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Dấu hiệu nhận diện | Tìm giá trị tối ưu, số phương án, kiểm tra khả năng đạt tới, dãy con, bài toán cái túi (Knapsack), hợp nhất khoảng |
| Template thông dụng | DP 1 chiều, DP 2 chiều, mảng cuộn (Rolling Array), Knapsack DP |
| Bài bắt buộc làm | [70. Climbing Stairs](https://leetcode.cn/problems/climbing-stairs/), [322. Coin Change](https://leetcode.cn/problems/coin-change/) |
| Bài nâng cao | [300. Longest Increasing Subsequence](https://leetcode.cn/problems/longest-increasing-subsequence/), [416. Partition Equal Subset Sum](https://leetcode.cn/problems/partition-equal-subset-sum/) |
| Trọng tâm ôn tập | Ý nghĩa của `dp[i]`, khởi tạo giá trị ban đầu, thứ tự duyệt, có thể nén không gian bộ nhớ không |

## Thuật toán tham lam (Greedy)

| Hạng mục | Nội dung |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Ở mỗi bước luôn chọn đối tượng thích hợp nhất hiện tại, thường đi kèm sắp xếp, khoảng, nhảy, mua bán |
| Template thông dụng | Lựa chọn sau khi sắp xếp, duy trì biên xa nhất, hợp nhất/bao phủ khoảng |
| Bài bắt buộc làm | [455. Assign Cookies](https://leetcode.cn/problems/assign-cookies/), [55. Jump Game](https://leetcode.cn/problems/jump-game/) |
| Bài nâng cao | [45. Jump Game II](https://leetcode.cn/problems/jump-game-ii/), [435. Non-overlapping Intervals](https://leetcode.cn/problems/non-overlapping-intervals/) |
| Trọng tâm ôn tập | Tại sao chiến lược tham lam không bị sai, có phản ví dụ nào lật đổ được chiến lược hiện tại không |

## Thuật toán chia để trị (Divide and Conquer)

| Hạng mục | Nội dung |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Bài toán có thể chia thành các bài toán con cùng loại, kết quả các bài toán con có thể gộp lại được |
| Template thông dụng | Chia nhỏ bằng đệ quy, giải quyết bài toán con, hợp nhất kết quả |
| Bài bắt buộc làm | [108. Convert Sorted Array to Binary Search Tree](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/), [148. Sort List](https://leetcode.cn/problems/sort-list/) |
| Bài nâng cao | [23. Merge k Sorted Lists](https://leetcode.cn/problems/merge-k-sorted-lists/), [215. Kth Largest Element in an Array](https://leetcode.cn/problems/kth-largest-element-in-an-array/) |
| Trọng tâm ôn tập | Điều kiện dừng đệ quy, khoảng trái và phải có bị chồng lấn không, độ phức tạp của bước hợp nhất |

## Sắp xếp tô-pô (Topological Sort)

| Hạng mục | Nội dung |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Dấu hiệu nhận diện | Phụ thuộc khóa học, phụ thuộc tác vụ, đồ thị có hướng không chu trình (DAG), kiểm tra khả năng hoàn thành |
| Template thông dụng | Mảng bán bậc vào (In-degree array) + Hàng đợi (Kahn's Algorithm), hoặc DFS với 3 màu đánh dấu |
| Bài bắt buộc làm | [207. Course Schedule](https://leetcode.cn/problems/course-schedule/) |
| Bài nâng cao | [210. Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/), [269. Alien Dictionary](https://leetcode.cn/problems/alien-dictionary/) |
| Trọng tâm ôn tập | Khi nào giảm bậc vào, số lượng phần tử kết quả có bằng tổng số node của đồ thị không |

## Tập hợp rời rạc / Cấu trúc dữ liệu hợp nhất (Union-Find / DSU)

| Hạng mục | Nội dung |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Dấu hiệu nhận diện | Tính liên thông, phân nhóm, mạng xã hội bạn bè, cạnh dư thừa (redundant connection), quan hệ đẳng thức |
| Template thông dụng | `find`, `union`, nén đường đi (path compression), hợp nhất theo kích thước/hạng |
| Bài bắt buộc làm | [547. Number of Provinces](https://leetcode.cn/problems/number-of-provinces/) |
| Bài nâng cao | [684. Redundant Connection](https://leetcode.cn/problems/redundant-connection/), [990. Satisfiability of Equality Equations](https://leetcode.cn/problems/satisfiability-of-equality-equations/) |
| Trọng tâm ôn tập | `find` có áp dụng nén đường đi không, khi nào thì phát hiện xung đột |

## Thao tác bit (Bit Manipulation)

| Hạng mục | Nội dung |
| -------- | ------------------------------------------------------------------------------------------------|
| Dấu hiệu nhận diện | Kiểm tra chẵn lẻ, kiểm tra lũy thừa của 2, phần tử chỉ xuất hiện một lần, nén trạng thái (bitmask) |
| Template thông dụng | XOR, phép AND xóa bit 1 thấp nhất (`n & (n - 1)`), duyệt bitmask |
| Bài bắt buộc làm | [136. Single Number](https://leetcode.cn/problems/single-number/), [231. Power of Two](https://leetcode.cn/problems/power-of-two/) |
| Bài nâng cao | [191. Number of 1 Bits](https://leetcode.cn/problems/number-of-1-bits/), [78. Subsets](https://leetcode.cn/problems/subsets/) |
| Trọng tâm ôn tập | Tính chất phép XOR, ý nghĩa của `n & (n - 1)`, biểu diễn số âm dưới dạng bù 2 |

## Lối vào lộ trình ôn tập

Bài viết này chỉ giữ lại danh sách các dạng bài và bài tập đề xuất kinh điển. Lộ trình ôn cấp tốc 7 ngày và lộ trình hệ thống 30 ngày được duy trì thống nhất tại [Tổng quan ôn tập phỏng vấn thuật toán](./README.md). Nếu sau này điều chỉnh nhịp độ ôn tập thì chỉ cần cập nhật trang tổng quan, tránh việc các bảng lộ trình ở nhiều bài bị lệch pha nhau.

<!-- @include: @article-footer.snippet.md -->
