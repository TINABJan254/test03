---
title: "Đề xuất các bài toán LeetCode kinh điển theo cấu trúc dữ liệu phổ biến"
description: "Phân loại các bài toán LeetCode tần suất cao theo Mảng, LinkedList, Stack, Queue, HashTable, Cây, Đồ thị, Heap, Trie, Union-Find, cung cấp dạng bài, template, giá trị phỏng vấn và trọng tâm ôn tập."
category: Cơ sở máy tính
tag:
  - Thuật toán
  - Cấu trúc dữ liệu
  - LeetCode
head:
  - - meta
    - name: keywords
      content: LeetCode,Cấu trúc dữ liệu,Mảng,LinkedList,Stack,Queue,HashTable,Cây nhị phân,Đồ thị,Heap,Trie,Union-Find,Đề xuất bài tập,Lộ trình luyện tập
---

Khi luyện bài tập cấu trúc dữ liệu, bạn không nên chỉ luyện đơn thuần theo độ khó từ Easy đến Hard. Cách học vững chắc hơn là xây dựng dạng bài theo từng cấu trúc dữ liệu: Mảng chú trọng chỉ số và khoảng, LinkedList chú trọng con trỏ, Stack và Queue chú trọng ràng buộc thứ tự, Cây và Đồ thị chú trọng cách duyệt, Heap chú trọng độ ưu tiên, HashTable chú trọng định vị nhanh.

Danh sách bài tập dưới đây được tinh gọn trong phạm vi các bài tần suất cao trong phỏng vấn và mang tính đại diện cho template. Mỗi loại hãy làm các "Bài bắt buộc làm" trước, sau đó làm các "Bài nâng cao". Sau khi làm xong, hãy ghi lại ít nhất độ phức tạp, các trường hợp biên và bài toán này thuộc về template nào.

## Cách sử dụng danh sách bài tập này

Học cấu trúc dữ liệu không chỉ là ghi nhớ kết luận. Mỗi khi ôn một dạng bài, trước tiên hãy quay lại bài viết về cấu trúc dữ liệu tương ứng để xem lại "Cách lưu trữ, Các thao tác cốt lõi, Độ phức tạp", sau đó mới bắt tay vào viết code. Nhờ đó, khi người phỏng vấn hỏi mở rộng về Java Collections, Redis, MySQL Index hoặc các kịch bản caching thực tế, câu trả lời của bạn sẽ không chỉ dừng lại ở mức giải bài thuật toán.

| Cấu trúc dữ liệu | Nên đọc gì trước | Trọng tâm khi làm bài tập |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------- |
| Mảng, LinkedList, Stack & Queue | [Chi tiết cấu trúc dữ liệu tuyến tính](../data-structure/linear-data-structure.md), [Two Pointers & Sliding Window](./two-pointers-and-sliding-window.md) | Chỉ số, cập nhật con trỏ, thời điểm push/pop |
| HashTable (Bảng băm) | [Tổng hợp bài toán phỏng vấn Bảng băm](../data-structure/hash-table.md) | Thiết kế key, thời điểm đếm, xung đột và mở rộng dung lượng (rehash) |
| Cây và Đồ thị | [Chi tiết cấu trúc Cây](../data-structure/tree.md), [Chi tiết Đồ thị](../data-structure/graph.md), [DFS & BFS](./dfs-bfs.md) | Giá trị trả về đệ quy, đánh dấu truy cập, đếm tầng trong BFS |
| Heap và Top K | [Chi tiết cấu trúc Heap](../data-structure/heap.md), [Tổng hợp bài toán Top K](./top-k.md) | Kích thước Heap, Comparator, tình huống luồng dữ liệu (Data Stream) |
| Trie và Union-Find | [Tổng hợp bài toán phỏng vấn Trie](../data-structure/trie.md), [Tổng hợp bài toán phỏng vấn Union-Find](../data-structure/union-find.md) | Cấu trúc node, cờ kết thúc từ, nén đường đi, phán đoán liên thông |
| LRU Cache | [Tổng hợp bài toán phỏng vấn LRU Cache](../data-structure/lru-cache.md) | HashTable và Doubly LinkedList phối hợp duy trì O(1) như thế nào |

## Mảng (Array)

| Dạng bài | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| -------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ----------------------------- |
| Tìm kiếm nhị phân | [704. Binary Search](https://leetcode.cn/problems/binary-search/) | [34. Find First and Last Position of Element in Sorted Array](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) | Khảo sát điều kiện lặp và biên | `left <= right`, cập nhật biên trái phải |
| Sửa đổi tại chỗ (In-place) | [26. Remove Duplicates from Sorted Array](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/) | [80. Remove Duplicates from Sorted Array II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/) | Khảo sát kỹ năng Two Pointers | Ý nghĩa con trỏ chậm, thời điểm ghi đè |
| Two Pointers | [977. Squares of a Sorted Array](https://leetcode.cn/problems/squares-of-a-sorted-array/) | [15. 3Sum](https://leetcode.cn/problems/3sum/) | Dạng bài mảng tần suất cao | Khử trùng lặp sau sắp xếp, di chuyển con trỏ |
| Tiền tố tổng (Prefix Sum) | [303. Range Sum Query - Immutable](https://leetcode.cn/problems/range-sum-query-immutable/) | [560. Subarray Sum Equals K](https://leetcode.cn/problems/subarray-sum-equals-k/) | Cửa ngõ bài toán mảng con | Ý nghĩa tiền tố tổng, đếm bằng HashTable |

## LinkedList (Danh sách liên kết)

| Dạng bài | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| -------- | ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------- | -------------------------------- |
| Thao tác cơ bản | [707. Design Linked List](https://leetcode.cn/problems/design-linked-list/) | [24. Swap Nodes in Pairs](https://leetcode.cn/problems/swap-nodes-in-pairs/) | Khảo sát thao tác node cơ bản | Dummy Node, thứ tự chèn/xóa node |
| Đảo ngược LinkedList | [206. Reverse Linked List](https://leetcode.cn/problems/reverse-linked-list/) | [92. Reverse Linked List II](https://leetcode.cn/problems/reverse-linked-list-ii/) | Rất hay bắt viết tay | Thứ tự cập nhật `prev`, `cur`, `next` |
| Fast-Slow Pointers | [141. Linked List Cycle](https://leetcode.cn/problems/linked-list-cycle/) | [142. Linked List Cycle II](https://leetcode.cn/problems/linked-list-cycle-ii/) | Thường xuyên hỏi mở rộng | Suy luận điểm gặp nhau và điểm bắt đầu vào chu trình |
| Xóa node | [19. Remove Nth Node From End of List](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/) | [61. Rotate List](https://leetcode.cn/problems/rotate-list/) | Khảo sát xử lý biên | Độ dài danh sách, trường hợp node đầu bị xóa |

## Stack & Queue (Ngăn xếp và Hàng đợi)

| Dạng bài | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| -------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | --------------- | -------------------------- |
| Mô phỏng cấu trúc | [232. Implement Queue using Stacks](https://leetcode.cn/problems/implement-queue-using-stacks/) | [225. Implement Stack using Queues](https://leetcode.cn/problems/implement-stack-using-queues/) | Khảo sát hiểu biết cấu trúc | Vai trò của stack vào và stack ra |
| Khớp dấu ngoặc | [20. Valid Parentheses](https://leetcode.cn/problems/valid-parentheses/) | [394. Decode String](https://leetcode.cn/problems/decode-string/) | Cửa ngõ bài toán chuỗi dùng stack | Khi nào đẩy vào stack, khi nào lấy ra khỏi stack |
| Monotonic Stack (Stack đơn điệu) | [739. Daily Temperatures](https://leetcode.cn/problems/daily-temperatures/) | [84. Largest Rectangle in Histogram](https://leetcode.cn/problems/largest-rectangle-in-histogram/) | Dạng bài trung bình và nâng cao | Stack duy trì tính đơn điệu tăng hay giảm |
| Monotonic Queue (Hàng đợi đơn điệu) | [239. Sliding Window Maximum](https://leetcode.cn/problems/sliding-window-maximum/) | [862. Shortest Subarray with Sum at Least K](https://leetcode.cn/problems/shortest-subarray-with-sum-at-least-k/) | Template thường gặp bài Hard | Hết hạn ở đầu hàng đợi, duy trì đơn điệu ở cuối |

## HashTable (Bảng băm)

| Dạng bài | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| ------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ------------ | ------------------------------ |
| Tìm kiếm nhanh | [1. Two Sum](https://leetcode.cn/problems/two-sum/) | [49. Group Anagrams](https://leetcode.cn/problems/group-anagrams/) | Nhập môn HashTable | Thiết kế key băm |
| Đếm tần suất | [242. Valid Anagram](https://leetcode.cn/problems/valid-anagram/) | [347. Top K Frequent Elements](https://leetcode.cn/problems/top-k-frequent-elements/) | Thống kê tần suất cao | Lựa chọn đếm bằng mảng hay bằng Map |
| Prefix Sum + Hash | [560. Subarray Sum Equals K](https://leetcode.cn/problems/subarray-sum-equals-k/) | [974. Subarray Sums Divisible by K](https://leetcode.cn/problems/subarray-sums-divisible-by-k/) | Thường gặp bài mảng con | Tra cứu trước rồi mới thêm vào Map để tránh cộng nhầm tiền tố hiện tại |
| Cấu trúc Caching | [146. LRU Cache](https://leetcode.cn/problems/lru-cache/) | [460. LFU Cache](https://leetcode.cn/problems/lfu-cache/) | Bài toán tự thiết kế viết tay | Sự phối hợp giữa HashTable và Doubly LinkedList |

## Cây nhị phân (Binary Tree)

| Dạng bài | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | ---------- | ----------------------- |
| Duyệt cây | [144. Binary Tree Preorder Traversal](https://leetcode.cn/problems/binary-tree-preorder-traversal/) | [102. Binary Tree Level Order Traversal](https://leetcode.cn/problems/binary-tree-level-order-traversal/) | Nền tảng bài toán cây | Biên đệ quy, đếm số tầng bằng hàng đợi |
| Bài toán đường đi | [112. Path Sum](https://leetcode.cn/problems/path-sum/) | [124. Binary Tree Maximum Path Sum](https://leetcode.cn/problems/binary-tree-maximum-path-sum/) | DFS tần suất cao | Tách biệt giá trị trả về của hàm và đáp án toàn cục |
| Xây dựng cây | [105. Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | [106. Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/) | Khảo sát khoảng đệ quy | Phân định chính xác chỉ số biên trái và phải |
| Tổ tiên chung gần nhất (LCA) | [236. Lowest Common Ancestor of a Binary Tree](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/) | [235. Lowest Common Ancestor of a Binary Search Tree](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/) | Câu hỏi phỏng vấn kinh điển | Điểm khác biệt giữa giải trên cây thường và trên BST |

## Đồ thị (Graph)

| Dạng bài | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| ------------ | ------------------------------------------------------------------ | ----------------------------------------------------------------------- | ------------ | ---------------------- |
| DFS/BFS trên lưới | [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/) | [695. Max Area of Island](https://leetcode.cn/problems/max-area-of-island/) | Nhập môn tìm kiếm đồ thị | Kiểm tra vượt biên, đánh dấu đã truy cập |
| Sắp xếp tô-pô | [207. Course Schedule](https://leetcode.cn/problems/course-schedule/) | [210. Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/) | Quan hệ phụ thuộc tác vụ | Mảng bán bậc vào, hàng đợi |
| Đường đi ngắn nhất | [994. Rotting Oranges](https://leetcode.cn/problems/rotting-oranges/) | [127. Word Ladder](https://leetcode.cn/problems/word-ladder/) | Ứng dụng BFS theo tầng | Thống kê số bước theo từng tầng |
| Tính liên thông | [547. Number of Provinces](https://leetcode.cn/problems/number-of-provinces/) | [684. Redundant Connection](https://leetcode.cn/problems/redundant-connection/) | Cửa ngõ Union-Find | Template `find` và `union` |

## Heap

| Dạng bài | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| -------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ----------- | ------------------ |
| Phần tử lớn thứ K | [215. Kth Largest Element in an Array](https://leetcode.cn/problems/kth-largest-element-in-an-array/) | [703. Kth Largest Element in a Stream](https://leetcode.cn/problems/kth-largest-element-in-a-stream/) | Top K tần suất cao | Duy trì kích thước Min-Heap cố định bằng K |
| Thống kê tần suất | [347. Top K Frequent Elements](https://leetcode.cn/problems/top-k-frequent-elements/) | [692. Top K Frequent Words](https://leetcode.cn/problems/top-k-frequent-words/) | HashTable + Heap | Cách viết Comparator |
| Hai Heap (Dual Heaps) | [295. Find Median from Data Stream](https://leetcode.cn/problems/find-median-from-data-stream/) | [480. Sliding Window Median](https://leetcode.cn/problems/sliding-window-median/) | Thiết kế nâng cao | Cân bằng kích thước giữa Max-Heap và Min-Heap |

## Trie và Union-Find

| Cấu trúc dữ liệu | Bài bắt buộc làm | Bài nâng cao | Giá trị phỏng vấn | Trọng tâm ôn tập |
| ---------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- | ------------ | ---------------------- |
| Trie | [208. Implement Trie (Prefix Tree)](https://leetcode.cn/problems/implement-trie-prefix-tree/) | [211. Design Add and Search Words Data Structure](https://leetcode.cn/problems/design-add-and-search-words-data-structure/) | Tập hợp chuỗi ký tự | Cấu trúc node, cờ đánh dấu kết thúc từ |
| Trie + DFS | [212. Word Search II](https://leetcode.cn/problems/word-search-ii/) | [648. Replace Words](https://leetcode.cn/problems/replace-words/) | Dạng bài trung cao cấp | Cắt tỉa nhánh theo tiền tố |
| Union-Find | [547. Number of Provinces](https://leetcode.cn/problems/number-of-provinces/) | [1319. Number of Operations to Make Network Connected](https://leetcode.cn/problems/number-of-operations-to-make-network-connected/) | Template tính liên thông | Nén đường đi (Path compression) |
| Union-Find phát hiện chu trình | [684. Redundant Connection](https://leetcode.cn/problems/redundant-connection/) | [990. Satisfiability of Equality Equations](https://leetcode.cn/problems/satisfiability-of-equality-equations/) | Biến thể bài toán đồ thị | Hợp nhất các đẳng thức trước, sau đó kiểm tra xung đột |

## Lối vào lộ trình ôn tập

Bài viết này chỉ giữ lại danh sách các đề xuất bài tập liên quan đến cấu trúc dữ liệu. Lộ trình ôn tập 7 ngày và 30 ngày được duy trì thống nhất tại [Tổng quan ôn tập cấu trúc dữ liệu](../data-structure/README.md), tránh việc phải cập nhật trùng lặp cùng một kế hoạch ở nhiều nơi.

<!-- @include: @article-footer.snippet.md -->
