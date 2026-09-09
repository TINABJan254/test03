---
title: Hệ thống kiến thức Cấu trúc dữ liệu: Mảng, Danh sách liên kết, Bảng băm, Cây, Đồ thị, Heap & Phỏng vấn
description: Lộ trình ôn tập phỏng vấn Cấu trúc dữ liệu, bao gồm Mảng, Danh sách liên kết, Ngăn xếp, Hàng đợi, Bảng băm, Cây, Đồ thị, Heap, Trie, Union-Find, Skip List, Red-Black Tree, Bloom Filter, LRU Cache, phân tích độ phức tạp và ứng dụng thực tế trong Java Backend.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
  - Phỏng vấn
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Cấu trúc dữ liệu, Câu hỏi phỏng vấn Cấu trúc dữ liệu, Lộ trình ôn tập Cấu trúc dữ liệu, Array, LinkedList, Stack, Queue, Hash Table, HashMap, Tree, Graph, Heap, Trie, Union-Find, Skip List, Red-Black Tree, Bloom Filter, LRU, Java Collection, Redis, MySQL Index, Phỏng vấn Backend
---

Tài liệu **Hệ thống kiến thức Cấu trúc dữ liệu** này được tổ chức theo định hướng phỏng vấn và các kịch bản thực tế trong Java Backend: Trước hết hiểu cách dữ liệu được lưu trữ, tiếp theo xem xét độ phức tạp của các thao tác phổ biến, và cuối cùng liên kết cấu trúc dữ liệu với các bài toán kỹ thuật như Java Collections, MySQL Index, Redis, Caching và Message Queue.

Trong các buổi phỏng vấn, nhà tuyển dụng hiếm khi chỉ hỏi đơn thuần "Mảng là gì". Câu hỏi thường gặp hơn là các câu hỏi đào sâu: Tại sao mảng và danh sách liên kết lại có sự khác biệt (một bên truy vấn nhanh, một bên chèn/xóa linh hoạt)? Tại sao `HashMap` cần phải mở rộng dung lượng (resize / rehash)? Tại sao B+ Tree lại phù hợp làm chỉ mục (Index) cơ sở dữ liệu? Tại sao Bloom Filter lại có thể xảy ra nhận định sai (False Positive)? Đằng sau những câu hỏi này đều kiểm tra một điều duy nhất: Bạn có thực sự hiểu sự đánh đổi (trade-offs) và độ phức tạp khi lựa chọn cấu trúc dữ liệu cho từng bài toán thực tế hay không.

Khi chuẩn bị kiến thức về cấu trúc dữ liệu, bạn không nên chỉ học thuộc lòng định nghĩa. Cách học hiệu quả hơn là bóc tách từng cấu trúc theo 4 câu hỏi: Lưu trữ như thế nào? Truy vấn như thế nào? Sửa đổi như thế nào? Phù hợp với kịch bản nào? Khi bạn trả lời rõ ràng được 4 câu hỏi này, việc luyện tập các bài toán giải thuật tương ứng sẽ đạt hiệu suất cao hơn rất nhiều.

## Phù hợp với ai

- Các bạn đang củng cố nền tảng Cấu trúc dữ liệu, chuẩn bị cho kỳ tuyển dụng đại học (Campus Recruitment) hoặc phỏng vấn kỹ sư Backend.
- Những độc giả khi giải bài thuật toán thường xuyên gặp khó khăn ở các cấu trúc như mảng, danh sách liên kết, cây, đồ thị, heap.
- Các kỹ sư muốn liên kết cấu trúc dữ liệu với Java Collections, Redis, MySQL, và các hệ thống Cache.
- Những lập trình viên đã nắm khái niệm nhưng khi trả lời phỏng vấn vẫn còn dừng lại ở mức định nghĩa lý thuyết đơn thuần.

## Phỏng vấn Cấu trúc dữ liệu hỏi những gì?

| Khía cạnh đánh giá | Cách hỏi thường gặp | Trọng tâm ôn tập |
| :--- | :--- | :--- |
| **Phương thức lưu trữ** | Lưu trữ tuần tự (Sequential) và Lưu trữ liên kết (Linked) khác nhau như thế nào? | Tính liên tục của bộ nhớ, con trỏ, tính thân thiện với CPU Cache |
| **Độ phức tạp thao tác** | Tại sao truy vấn mảng là $O(1)$, còn danh sách liên kết là $O(n)$? | Độ phức tạp khi tìm kiếm, chèn, xóa, duyệt |
| **So sánh cấu trúc** | Khi nào chọn Red-Black Tree, khi nào chọn AVL Tree? B-Tree và B+ Tree khác nhau ra sao? | Bảng so sánh + Kịch bản áp dụng |
| **Liên hệ kỹ thuật** | `HashMap`, `TreeMap`, `PriorityQueue`, Redis ZSet sử dụng cấu trúc dữ liệu gì? | Ứng dụng thực tế trong Java / Cơ sở dữ liệu / Bộ nhớ đệm |
| **Áp dụng giải thuật** | Duyệt cây, tìm kiếm trên đồ thị, Top-K, LRU cài đặt như thế nào? | Ôn tập kết hợp với các Template thuật toán |

## Khung trả lời phỏng vấn chuẩn

Khi trả lời các câu hỏi về cấu trúc dữ liệu, bạn không nên chỉ dừng lại ở việc trả lời "nó là gì". Cách diễn đạt mạch lạc và vững chắc nhất trong phỏng vấn là triển khai theo mạch tư duy sau:

```text
Định nghĩa -> Phương thức lưu trữ -> Độ phức tạp thao tác thường gặp -> Ưu nhược điểm -> Kịch bản áp dụng -> Ứng dụng trong Java / Redis / MySQL
```

Lấy ví dụ về Bảng băm (Hash Table), một câu trả lời hoàn chỉnh có thể tổ chức như sau:

1. Bảng băm ánh xạ Key thành chỉ số mảng (Index) thông qua Hàm băm (Hash Function).
2. Thao tác tìm kiếm, chèn, xóa trung bình đạt $O(1)$, nhưng khi xảy ra xung đột nghiêm trọng sẽ bị suy thoái.
3. Xung đột băm có thể xử lý bằng phương pháp Separate Chaining (Phương pháp nối chuỗi/danh sách liên kết) hoặc Open Addressing (Địa chỉ mở).
4. Trong Java, `HashMap` sử dụng Mảng + Danh sách liên kết + Cây đỏ đen (Red-Black Tree), và cơ chế Rehash được dùng để kiểm soát Hệ số tải (Load Factor).
5. Phù hợp cho các kịch bản tìm kiếm nhanh, đếm tần suất, loại bỏ trùng lặp, đánh chỉ mục cache, nhưng sẽ tiêu tốn thêm không gian bộ nhớ phụ.

Cách trả lời này sẽ thuyết phục người phỏng vấn hơn nhiều so với việc chỉ nói "truy vấn bảng băm là $O(1)$", bởi vì nó bao quát đầy đủ từ nguyên lý, độ phức tạp cho đến thực tế ứng dụng trong dự án.

## Lộ trình đọc gợi ý

1. [Chi tiết Cấu trúc dữ liệu tuyến tính](./linear-data-structure.md): Nắm vững mảng, danh sách liên kết, ngăn xếp, hàng đợi, hiểu rõ lưu trữ tuần tự và lưu trữ liên kết.
2. [Tổng hợp câu hỏi phỏng vấn Bảng băm](./hash-table.md): Hiểu hàm băm, xung đột băm, mở rộng dung lượng và liên hệ chặt chẽ với `HashMap`.
3. [Chi tiết Cấu trúc Cây](./tree.md): Nắm vững Cây nhị phân, Cây tìm kiếm nhị phân (BST), AVL Tree, B-Tree, B+ Tree và mối liên hệ với MySQL Index.
4. [Chi tiết Heap](./heap.md): Hiểu Hàng đợi ưu tiên (Priority Queue), bài toán Top-K, Heap Sort và `PriorityQueue` trong Java.
5. [Chi tiết Đồ thị](./graph.md): Hiểu cách biểu diễn đồ thị, DFS, BFS, Sắp xếp tô-pô (Topological Sort) và các thuật toán đường đi ngắn nhất.
6. [Tổng hợp câu hỏi phỏng vấn Cây tiền tố Trie](./trie.md), [Tổng hợp câu hỏi phỏng vấn Union-Find](./union-find.md): Bổ sung kiến thức xử lý tập hợp chuỗi và bài toán tính liên thông.
7. [Tổng hợp câu hỏi phỏng vấn Skip List](./skip-list.md), [Chi tiết Cây đỏ đen](./red-black-tree.md), [Chi tiết Bloom Filter](./bloom-filter.md), [Tổng hợp câu hỏi phỏng vấn LRU Cache](./lru-cache.md): Ôn tập chuyên sâu phục vụ Java Collections, Redis, Caching và Database.

## Danh mục bài viết cốt lõi

| Bài viết | Trọng tâm | Liên hệ thường gặp |
| :--- | :--- | :--- |
| [Chi tiết Cấu trúc dữ liệu tuyến tính](./linear-data-structure.md) | Mảng, Danh sách liên kết, Ngăn xếp, Hàng đợi | `ArrayList`, `LinkedList`, Message Queue |
| [Tổng hợp câu hỏi phỏng vấn Bảng băm](./hash-table.md) | Hàm băm, Xung đột băm, Mở rộng dung lượng | `HashMap`, Cache, Khử trùng lặp dữ liệu |
| [Chi tiết Cấu trúc Cây](./tree.md) | Cây nhị phân, BST, AVL, B-Tree, B+ Tree | MySQL Index, Expression Tree |
| [Chi tiết Đồ thị](./graph.md) | Danh sách kề, Ma trận kề, DFS, BFS | Mối quan hệ phụ thuộc, Routing, Recommendation |
| [Chi tiết Heap](./heap.md) | Max-Heap, Min-Heap, Heap Sort | `PriorityQueue`, Top-K, DelayQueue |
| [Chi tiết Cây đỏ đen](./red-black-tree.md) | Cân bằng gần đúng, Phép quay, Đổi màu | `TreeMap`, `HashMap` Treeify |
| [Chi tiết Bloom Filter](./bloom-filter.md) | Mảng bit, Hàm băm, Xác suất nhận định sai | Chống Cache Penetration, Khử trùng, Blacklist |
| [Tổng hợp câu hỏi phỏng vấn Skip List](./skip-list.md) | Chỉ mục đa cấp, Truy vấn khoảng (Range Query) | Redis ZSet |
| [Tổng hợp câu hỏi phỏng vấn LRU Cache](./lru-cache.md) | Bảng băm + Danh sách liên kết đôi | Local Cache, Page Replacement |

## Bảng tra cứu nhanh lựa chọn cấu trúc dữ liệu

Rất nhiều câu hỏi phỏng vấn thực chất là đang hỏi: "Trong kịch bản này tại sao lại chọn cấu trúc này mà không phải cấu trúc khác?". Bảng dưới đây rất hữu ích để ôn tập nhanh trước buổi phỏng vấn:

| Kịch bản bài toán | Cấu trúc ưu tiên cân nhắc | Điểm đánh đổi (Trade-offs) |
| :--- | :--- | :--- |
| **Truy cập ngẫu nhiên thường xuyên theo Index** | Array, `ArrayList` | Truy vấn cực nhanh ($O(1)$), nhưng chi phí chèn/xóa phần tử ở giữa cao ($O(n)$) |
| **Chèn và xóa thường xuyên ở 2 đầu** | Deque (Hàng đợi hai đầu), LinkedList | Thao tác con trỏ linh hoạt ($O(1)$), nhưng truy cập ngẫu nhiên chậm ($O(n)$) |
| **Phán đoán nhanh phần tử có tồn tại hay không** | Hash Table, Bloom Filter | Hash Table chính xác tuyệt đối nhưng tốn RAM; Bloom Filter siêu tiết kiệm RAM nhưng có thể có sai số (False Positive) |
| **Duy trì tập hợp có thứ tự & Truy vấn khoảng** | Red-Black Tree, Skip List, B+ Tree | Red-Black Tree phù hợp tập có thứ tự trong RAM; Skip List phù hợp truy vấn khoảng và dễ cài đặt; B+ Tree tối ưu cho I/O đĩa |
| **Xử lý Giá trị lớn nhất / Nhỏ nhất / Top-K** | Heap, Priority Queue | Chỉ quan tâm đến phần tử cực trị cục bộ ($O(1)$ lấy đỉnh, $O(\log n)$ điều chỉnh), không phù hợp để duyệt toàn bộ theo thứ tự |
| **Kiểm tra tính liên thông và phân nhóm** | Union-Find (DSU) | Hợp nhất và truy vấn cực nhanh ($O(\alpha(n))$), nhưng không hỗ trợ xóa quan hệ kết nối |
| **Khớp tiền tố chuỗi ký tự, gợi ý tìm kiếm** | Trie | Tốc độ tìm kiếm phụ thuộc vào độ dài chuỗi ký tự, nhưng số lượng node có thể tăng cao |
| **Thuật toán đào thải bộ nhớ đệm (Cache Eviction)** | LRU, LFU | LRU xét theo thời điểm truy cập gần nhất, LFU xét theo tần suất truy cập |

## Lộ trình ôn tập 7 ngày

| Thời gian | Trọng tâm ôn tập | Hành động gợi ý |
| :--- | :--- | :--- |
| **Ngày 1** | Mảng, Danh sách liên kết | Lập bảng độ phức tạp, tự tay code Đảo ngược danh sách liên kết và Xóa node |
| **Ngày 2** | Ngăn xếp, Hàng đợi, Bảng băm | Code bài toán Khớp dấu ngoặc, Cài đặt hàng đợi bằng stack, Two Sum |
| **Ngày 3** | Cây | Code Duyệt cây nhị phân, Tổ tiên chung gần nhất (LCA), ôn tập B+ Tree |
| **Ngày 4** | Heap | Code bài toán Top-K, K phần tử xuất hiện nhiều nhất, hiểu `PriorityQueue` |
| **Ngày 5** | Đồ thị | Code DFS/BFS, Số lượng đảo (Number of Islands), Sắp xếp lịch học (Course Schedule) |
| **Ngày 6** | Red-Black Tree, Skip List, Bloom Filter | Tập trung chuẩn bị các câu hỏi đào sâu trong kịch bản kỹ thuật thực tế |
| **Ngày 7** | LRU & Ôn tập tổng hợp | Tự tay code LRU Cache, tổng hợp lại độ phức tạp và ứng dụng của toàn bộ cấu trúc |

## Lộ trình ôn tập 30 ngày

| Giai đoạn | Thời gian | Mục tiêu |
| :--- | :--- | :--- |
| **Giai đoạn 1** | Ngày 1 đến 6 | Cấu trúc tuyến tính và Bảng băm, giải thích rõ độ phức tạp và liên hệ Java Collections |
| **Giai đoạn 2** | Ngày 7 đến 13 | Cây, Heap, Đồ thị, kết hợp luyện đề DFS/BFS và Top-K |
| **Giai đoạn 3** | Ngày 14 đến 20 | Trie, Union-Find, Skip List, Red-Black Tree, hoàn thiện các cấu trúc nâng cao |
| **Giai đoạn 4** | Ngày 21 đến 25 | Bloom Filter, LRU, bài toán thiết kế hệ thống, kết nối với Redis/MySQL/Cache |
| **Giai đoạn 5** | Ngày 26 đến 30 | Ôn lại các bài làm sai, luyện nói phỏng vấn, chuẩn bị 2 câu hỏi đào sâu cho mỗi cấu trúc |

## Tự kiểm tra các câu hỏi tần suất cao

- Bố cục bộ nhớ của Mảng và Danh sách liên kết khác nhau ra sao? Tại sao Mảng truy cập ngẫu nhiên lại nhanh?
- Trong Java, khi nào nên chọn `ArrayList` và khi nào nên chọn `LinkedList`?
- Ngăn xếp và Hàng đợi lần lượt phù hợp với những kịch bản nào? Monotonic Stack (Ngăn xếp đơn điệu) và Monotonic Queue giải quyết bài toán gì?
- Có những phương pháp xử lý xung đột băm nào? Tại sao `HashMap` lại cần mở rộng dung lượng (Rehash)?
- Sự khác biệt giữa Cây tìm kiếm nhị phân (BST), AVL Tree và Red-Black Tree là gì?
- Tại sao B-Tree và B+ Tree lại đặc biệt phù hợp làm chỉ mục cơ sở dữ liệu?
- Heap và Cây nhị phân thông thường khác nhau như thế nào? Tại sao giải bài toán Top-K thường dùng Heap?
- Khi nào nên chọn Danh sách kề, khi nào nên chọn Ma trận kề để biểu diễn đồ thị? Độ phức tạp của DFS và BFS là bao nhiêu?
- Tại sao Skip List lại phù hợp cho truy vấn khoảng? Tại sao Redis lại chọn dùng Skip List cho ZSet?
- Tại sao Bloom Filter lại có thể nhận định sai? Tại sao thao tác xóa trong Bloom Filter lại rất khó khăn?
- Tại sao thuật toán LRU thường được cài đặt bằng Bảng băm kết hợp với Danh sách liên kết đôi?

## Chuyên đề liên quan

- [Hệ thống kiến thức Cơ sở máy tính](../)
- [Chuyên đề Thuật toán](../algorithms/)
- [Gợi ý các bài tập LeetCode kinh điển về Cấu trúc dữ liệu](../algorithms/common-data-structures-leetcode-recommendations.md)
- [Chi tiết Java Collections](../../java/collection/java-collection-questions-01.md)
- [Chi tiết Chỉ mục MySQL (MySQL Index)](../../database/mysql/mysql-index.md)
- [Tổng hợp câu hỏi phỏng vấn Redis](../../database/redis/redis-questions-01.md)
- [Chuẩn bị phỏng vấn](../../interview-preparation/)

<!-- @include: @article-footer.snippet.md -->
