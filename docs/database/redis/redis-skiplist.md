---
title: Redis为什么用跳表实现有序集合
description: 深入讲解Redis有序集合Zset为何选择跳表而非红黑树、B+树实现，详解跳表的数据结构原理、时间复杂度分析和Redis源码实现。
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: Redis跳表,SkipList,有序集合,Zset,跳表原理,平衡树对比,Redis数据结构
---

## Mở đầu

Trong các buổi phỏng vấn kỹ thuật về Redis, có một câu hỏi rất phổ biến và thú vị: *"Tại sao Sorted Set trong Redis lại dùng Skiplist (Bảng nhảy) mà không dùng Cây cân bằng (AVL Tree), Cây đỏ đen (Red-Black Tree) hay Cây B+ (B+ Tree)?"*.

Bài viết này sẽ đi sâu giải thích cấu trúc dữ liệu Skiplist và câu trả lời toàn diện cho câu hỏi trên.

## Ứng dụng của Skiplist trong Redis ZSet

Redis Sorted Set (ZSet) là một tập hợp các phần tử duy nhất và được sắp xếp có thứ tự theo trọng số `score`.

Khi số lượng phần tử < 128 và độ dài mỗi phần tử < 64 bytes, ZSet được lưu bằng **ZipList (Danh sách nén)** để tiết kiệm bộ nhớ RAM.

Khi vượt ngưỡng trên, ZSet sẽ chuyển sang sử dụng **Skiplist** (thực tế là kết hợp giữa `dict` và `skiplist` để đạt hiệu năng tra cứu $O(1)$ cho phần tử đơn lẻ và $O(\log N)$ cho tra cứu thứ tự).

```bash
zset-max-ziplist-value 64
zset-max-ziplist-entries 128
```

## Tự viết cấu trúc Skiplist

Skiplist có thể hiểu là một danh sách liên kết có thứ tự (Ordered Linked List) nhưng được xây dựng thêm **nhiều tầng chỉ mục (Multi-level Index)** ở phía trên. Nhờ các tầng chỉ mục này, độ phức tạp của các thao tác Thêm, Xóa, Tìm kiếm giảm từ $O(N)$ xuống còn **$O(\log N)$**.

![](https://oss.javaguide.cn/javaguide/database/redis/skiplist/202401222005436.png)

Ví dụ tìm kiếm số 6:
1. Từ tầng chỉ mục 2: Đến node 4 -> nhìn sang node tiếp theo là 8 (> 6) -> tụt xuống tầng chỉ mục 1.
2. Tại tầng chỉ mục 1: Từ node 4 nhìn sang node 6 -> tìm thấy số 6. Chỉ tốn 2 bước thay vì 6 bước trên Linked List gốc.

### Thuật toán chiều cao chỉ mục ngẫu nhiên (Probabilistic Balancing)

Để tránh việc phải tính toán cân bằng cứng nhắc khi chèn phần tử mới, Skiplist áp dụng thuật toán xác suất:
- Chiều cao ban đầu của node = 1.
- Sinh số ngẫu nhiên từ 0 đến 1. Nếu số ngẫu nhiên > 0.5 ($p = 50\%$) thì tăng chiều cao lên 1 tầng.
- Nhờ đó, xác suất tạo tầng 1 là 50%, tầng 2 là 25%, tầng 3 là 12.5%...
- Chiều cao tầng tối đa khuyến nghị là 16 (trong Redis cấu hình tối đa là 32 với macro `ZSKIPLIST_MAXLEVEL`).

![](https://oss.javaguide.cn/javaguide/database/redis/skiplist/202401222005505.png)

### Đặc điểm của Skiplist trong Redis

1. Sử dụng **Linked List kép (Doubly Linked List)** có thêm con trỏ lùi (`backward` pointer) giúp thao tác tìm phần tử phía trước và xóa nút diễn ra vô cùng thuận tiện.
2. `score` có thể trùng lặp. Nếu `score` trùng nhau sẽ sắp xếp theo thứ tự từ điển của chuỗi `ele` (SDS).
3. Chiều cao tầng tối đa là 32 (`ZSKIPLIST_MAXLEVEL`).

## So sánh Skiplist với các cấu trúc dữ liệu khác

### 1. AVL Tree (Cây cân bằng) vs Skiplist

AVL Tree là cây nhị phân tìm kiếm cân bằng tuyệt đối (chênh lệch chiều cao 2 cây con $\le 1$). Thời gian tìm kiếm, thêm, xóa của AVL Tree và Skiplist đều là **$O(\log N)$**.

Tuy nhiên, mỗi lần Insert hoặc Delete trên AVL Tree đều yêu cầu cây giữ cân bằng tuyệt đối. Chỉ cần mất cân bằng là phải thực hiện các phép **xoay cây (Rotation)** phức tạp và tốn CPU.

Skiplist ra đời nhằm khắc phục nhược điểm này bằng cách sử dụng **Cân bằng xác suất (Probabilistic Balancing)** thay vì cân bằng cứng nhắc, giúp thuật toán Insert/Delete đơn giản và nhanh hơn nhiều.

### 2. Red-Black Tree (Cây đỏ đen) vs Skiplist

Red-Black Tree là cây tìm kiếm nhị phân tự cân bằng đen. Thời gian tìm kiếm, thêm, xóa cũng là **$O(\log N)$**.

So với Red-Black Tree:
- **Cài đặt Skiplist đơn giản hơn nhiều** (không cần quản lý việc đổi màu node và xoay cây).
- **Thao tác truy vấn theo khoảng (Range Query `ZRANGE`)**: Skiplist vượt trội hoàn toàn so với Red-Black Tree. Trong Skiplist, sau khi định vị được node đầu tiên, chỉ cần đi tiếp trên Linked List tầng đáy là lấy được toàn bộ khoảng dữ liệu.

### 3. B+ Tree vs Skiplist

B+ Tree là cấu trúc cây nhiều nhánh được tối ưu cho **Cơ sở dữ liệu trên đĩa cứng (MySQL, FileSystem)** nhằm giảm thiểu tối đa số lần I/O đĩa.

Redis là một **In-Memory Database** (mọi dữ liệu nằm trên RAM), nên không chịu ảnh hưởng bởi I/O đĩa. Vì vậy Redis không cần cấu trúc B+ Tree phức tạp. Khi chèn phần tử vào B+ Tree, nếu node bị đầy phải thực hiện tách node (split) và gộp node (merge) rất tốn kém. Skiplist chỉ cần chèn nút ngẫu nhiên theo xác suất, vừa tiết kiệm RAM vừa dễ cài đặt.

### Lý do do chính tác giả Redis (Antirez) đưa ra:

1. **Tiết kiệm bộ nhớ**: Việc điều chỉnh xác suất $p$ giúp Skiplist tốn ít bộ nhớ hơn B-Tree.
2. **Tối ưu cho thao tác `ZRANGE` / `ZREVRANGE`**: Duyệt Skiplist tương tự như duyệt Linked List, có Cache Locality rất tốt.
3. **Đơn giản, dễ cài đặt và debug**: Nhờ sự đơn giản của Skiplist, việc mở rộng thêm tính năng lấy thứ hạng `ZRANK` trong $O(\log N)$ diễn ra rất dễ dàng mà hầu như không phải sửa đổi nhiều code.

## Đọc thêm về Data Structure

- [Tóm tắt câu hỏi phỏng vấn Skip List](../../cs-basics/data-structure/skip-list.md)
- [Chi tiết Cây đỏ đen (Red-Black Tree)](../../cs-basics/data-structure/red-black-tree.md)
- [Tóm tắt câu hỏi phỏng vấn Hash Table](../../cs-basics/data-structure/hash-table.md)

<!-- @include: @article-footer.snippet.md -->
