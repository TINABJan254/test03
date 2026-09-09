---
title: Tổng hợp câu hỏi phỏng vấn Skip List: Chỉ mục đa cấp, Truy vấn khoảng và Redis ZSet
description: Tổng hợp câu hỏi phỏng vấn về Danh sách nhảy (Skip List), giải thích nguyên lý chỉ mục đa cấp, các thao tác tìm kiếm, chèn, xóa, phân tích độ phức tạp, so sánh với Cây đỏ đen và ứng dụng làm cấu trúc lõi trong Redis ZSet.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
  - Redis
head:
  - - meta
    - name: keywords
      content: Skip List, SkipList, Redis ZSet, Sorted Set, Tập hợp có thứ tự, Truy vấn khoảng, Range Query, Chỉ mục đa cấp, Cây đỏ đen, Câu hỏi phỏng vấn Redis, Cấu trúc dữ liệu
---

**Skip List (Danh sách nhảy)** có thể hiểu đơn giản là một **"Danh sách liên kết có thứ tự được trang bị hệ thống Chỉ mục đa cấp (Multi-level Index)"**.

Danh sách liên kết có thứ tự thông thường khi tìm kiếm phải quét tuần tự từng phần tử từ đầu đến cuối với độ phức tạp $O(n)$. Skip List bổ sung thêm nhiều tầng chỉ mục thưa hơn ở phía trên danh sách gốc, cho phép quá trình tìm kiếm có thể "nhảy" cóc qua một nhóm lớn các node ở tầng cao, sau đó hạ dần xuống các tầng thấp hơn để định vị chính xác vị trí mục tiêu.

Cấu trúc tập hợp có thứ tự **ZSet** nổi tiếng của **Redis** sử dụng sự kết hợp giữa **Skip List** và **Bảng băm (Hash Table)** làm cấu trúc lưu trữ bên dưới, vì vậy Skip List là chủ đề xuất hiện với tần suất rất cao trong các buổi phỏng vấn Java Backend và Redis.

Nội dung chính:
1. Skip List là gì?
2. Tại sao Skip List có thể hạ độ phức tạp tìm kiếm từ $O(n)$ xuống trung bình $O(\log n)$?
3. Cách thực hiện Tìm kiếm, Chèn và Xóa trên Skip List?
4. So sánh Skip List với Cây đỏ đen (Red-Black Tree)?
5. Tại sao Redis ZSet lại lựa chọn Skip List?

![Skip List xây dựng chỉ mục đa cấp trên danh sách liên kết có thứ tự](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/skip-list.png)

## 1. Skip List là gì?

Skip List là cấu trúc dữ liệu dựa trên danh sách liên kết có thứ tự.
- Tầng đáy cùng (Level 0) là một danh sách liên kết có thứ tự hoàn chỉnh chứa 100% tất cả các phần tử của tập dữ liệu.
- Phía trên Level 0, Skip List xây dựng thêm các tầng chỉ mục (Level 1, Level 2, ... Level $k$) với mật độ thưa dần.

Có thể hình dung Skip List như mục lục của một cuốn sách dày:
- **Tầng đáy cùng (Level 0)**: Giống như từng trang sách chi tiết, đầy đủ dữ liệu nhất nhưng lật từ trang đầu tới trang cuối thì rất chậm.
- **Các tầng chỉ mục phía trên**: Giống như Mục lục các Chương và Mục lớn, chứa ít thông tin hơn nhưng giúp người đọc "nhảy" ngay tới trang sách gần mục tiêu nhất.
- **Khi tìm kiếm**: Bắt đầu nhảy sang phải từ tầng chỉ mục cao nhất; khi gặp node lớn hơn giá trị cần tìm thì hạ xuống 1 tầng, và tiếp tục lặp lại quá trình này cho tới tầng đáy.

Điểm khác biệt cốt lõi: Danh sách liên kết thường mỗi bước chỉ tiến lên được 1 node; Skip List ở tầng cao có thể nhảy cóc qua hàng chục, hàng trăm node cùng lúc.

---

## 2. Cấu trúc Node và Chiều cao ngẫu nhiên

Node của danh sách liên kết thông thường chỉ có duy nhất 1 con trỏ `next`. Node của Skip List chứa một **mảng các con trỏ tiến (Forward Pointers)**:

```java
class SkipListNode {
    int value;
    SkipListNode[] forward; // forward[i] trỏ tới node tiếp theo ở tầng thứ i
}
```

Nếu một node xuất hiện ở Tầng 3, nó sẽ sở hữu các con trỏ ở cả Tầng 0, 1, 2 và 3. Trong Redis ZSet, node Skip List còn lưu thêm `score`, `member`, con trỏ lùi `backward`, và `span` (khoảng cách giữa 2 node) để hỗ trợ truy vấn thứ hạng (Rank).

### Chiều cao của một Node được quyết định như thế nào?

Skip List **không sử dụng các phép quay hay đổi màu phức tạp** như Cây đỏ đen để duy trì cân bằng, mà nó sử dụng **Cơ chế xác suất ngẫu nhiên (Randomized Level Generation)**.

Khi chèn một node mới, hàm xác suất (giống như tung đồng xu):
- Node luôn xuất hiện ở Tầng 0.
- Tung được mặt ngửa (xác suất $p = 1/2$ hoặc $p = 1/4$), node được nâng lên Tầng 1.
- Tiếp tục tung được mặt ngửa, nâng lên Tầng 2, cho đến khi tung được mặt sấp hoặc chạm mức tầng tối đa (`MAX_LEVEL`).

Kết quả là: Càng lên tầng cao số lượng node càng giảm theo cấp số nhân (mỗi tầng giảm khoảng 50%). Về mặt xác suất thống kê, chiều cao của Skip List luôn được duy trì ở mức **$O(\log n)$**.

---

## 3. Các thao tác trên Skip List

### 1. Tìm kiếm (Search)
Bắt đầu từ Node Đầu (Head) ở tầng cao nhất:
1. So sánh với node kế tiếp ở cùng tầng: Nếu giá trị kế tiếp nhỏ hơn `target`, nhảy sang phải (`current = current.forward[i]`).
2. Nếu giá trị kế tiếp lớn hơn `target` hoặc là `null`, ta hạ xuống tầng thấp hơn (`i--`).
3. Lặp lại cho đến Tầng 0, kiểm tra xem node kế tiếp có đúng bằng `target` hay không.

### 2. Chèn (Insert) và Xóa (Delete)
- Khi chèn hoặc xóa, ta dùng một mảng `update[]` để ghi nhận lại node đứng trước vị trí cần chèn/xóa ở **tất cả các tầng**.
- Khi chèn: Tạo node mới với số tầng ngẫu nhiên, sau đó nối các con trỏ `forward` tại từng tầng.
- Khi xóa: Nối con trỏ của các node trong `update[]` bỏ qua node cần xóa.

---

## 4. Phân tích Độ phức tạp

| Thao tác | Độ phức tạp trung bình | Trường hợp xấu nhất | Giải thích |
| :--- | :--- | :--- | :--- |
| **Tìm kiếm** | **$O(\log n)$** | $O(n)$ | Nhảy cóc qua các node nhờ chỉ mục đa cấp |
| **Chèn phần tử** | **$O(\log n)$** | $O(n)$ | Tìm vị trí chèn $O(\log n)$ + Cập nhật con trỏ |
| **Xóa phần tử** | **$O(\log n)$** | $O(n)$ | Tìm vị trí xóa $O(\log n)$ + Cập nhật con trỏ |
| **Truy vấn khoảng (Range Query)** | **$O(\log n + k)$** | $O(n)$ | Dùng $O(\log n)$ tìm điểm bắt đầu, sau đó duyệt tuần tự $k$ phần tử ở Tầng 0 |

---

## 5. So sánh: Skip List vs Cây đỏ đen (Red-Black Tree)

| Tiêu chí | Skip List | Cây đỏ đen (Red-Black Tree) |
| :--- | :--- | :--- |
| **Cơ chế cân bằng** | Xác suất ngẫu nhiên (Tung đồng xu) | Cân bằng cấu trúc (Phép quay và Đổi màu) |
| **Độ phức tạp cài đặt** | **Đơn giản, dễ hiểu, dễ code** | Rất phức tạp (nhiều trường hợp quay cây) |
| **Truy vấn khoảng (Range Query)** | **Cực kỳ tiện lợi** (chỉ cần duyệt thẳng danh sách liên kết đáy) | Phải duyệt In-order phức tạp hơn |
| **Hỗ trợ đồng thời (Concurrency)** | Khóa từng đoạn trên danh sách liên kết dễ dàng | Khóa toàn cây khi quay rất khó khăn |
| **Ứng dụng tiêu biểu** | **Redis ZSet**, LevelDB, RocksDB | Java `TreeMap`, `TreeSet`, Linux Kernel CFS |

---

## 6. Tại sao Redis ZSet lại sử dụng Skip List?

Trong Redis, cấu trúc `ZSet` hỗ trợ các lệnh:
- `ZSCORE key member`: Tìm điểm số theo member -> Sử dụng **Bảng băm (Hash Table)** để đạt tốc độ $O(1)$.
- `ZRANGEBYSCORE key min max`: Truy vấn danh sách phần tử trong khoảng điểm từ `min` đến `max` -> Sử dụng **Skip List** để đạt tốc độ $O(\log n + k)$.
- `ZRANK key member`: Lấy thứ hạng của một phần tử -> Skip List lưu thêm trường `span` giúp tính rank trong $O(\log n)$.

> **Tại sao Redis chọn Skip List thay vì Red-Black Tree?** (Giải thích chính thức từ Antirez - tác giả của Redis):
> 1. **Hiệu năng truy vấn khoảng vượt trội**: Skip List chỉ cần tìm node đầu tiên của khoảng, sau đó duyệt thẳng theo con trỏ `forward[0]` là lấy được toàn bộ dữ liệu.
> 2. **Cài đặt và bảo trì đơn giản**: Thuật toán Skip List dễ debug và mở rộng hơn nhiều so với Red-Black Tree.
> 3. **Tiết kiệm bộ nhớ hơn**: Với xác suất $p = 1/4$, trung bình mỗi node chỉ tốn $1.33$ con trỏ, ít hơn so với 3 con trỏ + bit màu của cây nhị phân.

<!-- @include: @article-footer.snippet.md -->
