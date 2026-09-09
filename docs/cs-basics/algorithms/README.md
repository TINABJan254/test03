---
title: Chuyên đề Thuật toán: Lộ trình luyện đề phỏng vấn, Template cốt lõi và bài tập LeetCode tần suất cao
description: Lộ trình ôn tập phỏng vấn Thuật toán, bao gồm Phân tích độ phức tạp, Tìm kiếm nhị phân, Hai con trỏ, Cửa sổ trượt, DFS/BFS, Quay lui, Quy hoạch động, Tham lam, Top-K, Chuỗi ký tự, Danh sách liên kết, Sắp xếp và bài tập LeetCode tần suất cao.
category: Cơ sở máy tính
tag:
  - Thuật toán
  - LeetCode
  - Phỏng vấn
sidebar: false
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Thuật toán, Câu hỏi phỏng vấn Thuật toán, LeetCode, Lộ trình luyện đề, Binary Search, Two Pointers, Sliding Window, DFS, BFS, Backtracking, Dynamic Programming, Greedy, Top-K, Thuật toán sắp xếp, Thuật toán chuỗi, Thuật toán LinkedList, Phỏng vấn Backend
---

Tài liệu **Chuyên đề Thuật toán** này không sắp xếp kiến thức theo kiểu liệt kê lý thuyết giáo trình một cách máy móc, mà được tổng hợp dựa trên **lộ trình luyện đề phỏng vấn thực tế**: Bắt đầu từ việc làm rõ bản chất phân tích độ phức tạp (Complexity Analysis), tiếp theo là làm chủ các mẫu giải thuật (Templates) tần suất cao nhất như Tìm kiếm nhị phân (Binary Search), Hai con trỏ (Two Pointers), Cửa sổ trượt (Sliding Window), DFS/BFS, Quay lui (Backtracking), Quy hoạch động (Dynamic Programming), Tham lam (Greedy), Top-K; và cuối cùng là tổng kết, củng cố qua các chuyên đề Chuỗi ký tự, Danh sách liên kết, 10 Thuật toán sắp xếp kinh điển và danh sách đề thi LeetCode chọn lọc.

Khi ôn luyện thuật toán, nhiều bạn rất dễ rơi vào một trạng thái: Đã giải qua khá nhiều bài, nhưng chỉ cần người phỏng vấn thay đổi nhẹ một điều kiện đề bài là lập tức bị tắc nghẽn. Nguyên nhân thường không phải do bạn làm chưa đủ số lượng bài, mà là do bạn chưa gom các bài toán về đúng **Khuôn mẫu tư duy (Algorithm Template)**. Điều thực sự hữu ích khi phỏng vấn là: Vừa nhìn thấy đề bài là có thể nhận diện ngay nó thuộc dạng bài toán nào, nhanh chóng viết ra một phiên bản code hoạt động chuẩn xác, sau đó giải thích rành mạch về độ phức tạp và các trường hợp biên (Edge cases).

## Phù hợp với ai

- Các bạn đang chuẩn bị cho kỳ tuyển dụng đại học (Campus Recruitment) hoặc phỏng vấn kỹ sư Backend, mong muốn luyện đề LeetCode bài bản theo từng dạng bài.
- Những độc giả đã giải qua nhiều bài tập nhưng khi nhìn lại vẫn chưa tự tin trả lời rõ ràng câu hỏi: *"Tại sao bài này lại dùng cách tiếp cận đó?"*.
- Các kỹ sư có nền tảng Cấu trúc dữ liệu khá tốt nhưng còn thiếu kinh nghiệm về Template thuật toán và xử lý các trường hợp góc/biên.
- Những lập trình viên chỉ có từ 7 đến 30 ngày trước buổi phỏng vấn và cần lấy lại phản xạ giải thuật trong thời gian ngắn nhất.

## Phỏng vấn Thuật toán đánh giá điều gì?

Trong các buổi phỏng vấn kỹ thuật, người phỏng vấn thường không chỉ xem bạn có bấm nộp code đạt AC (Accepted) được hay không, mà quan trọng hơn là đánh giá **4 năng lực cốt lõi**:

| Khía cạnh đánh giá | Biểu hiện cụ thể trong phỏng vấn | Việc cần làm khi ôn tập |
| :--- | :--- | :--- |
| **Nhận diện dạng bài** | Bài này là Binary Search, Sliding Window, Backtracking hay Dynamic Programming? | Luyện đề theo từng dạng bài, không làm đề ngẫu nhiên |
| **Độ ổn định của mã nguồn** | Các điểm biên, con trỏ null, chỉ số mảng (Index), điều kiện dừng vòng lặp có tin cậy không? | Mỗi Template luôn chuẩn bị sẵn 2 đến 3 ca kiểm thử biên (Edge cases) |
| **Trình bày độ phức tạp** | Có giải thích rành mạch được Độ phức tạp thời gian và Không gian bộ nhớ không? | Sau mỗi bài giải xong luôn tự phân tích và ghi lại Big-O |
| **Khả năng chuyển giao** | Khi điều kiện bài toán thay đổi, có biết cách biến tấu Template không? | Mỗi dạng bài làm tối thiểu bài cơ bản và các bài biến thể |

> **Nguyên tắc ghi nhớ then chốt:**  
> **Xây dựng Template vững chắc theo từng dạng bài $\rightarrow$ Dùng các bài toán đại diện để rèn luyện khả năng chuyển giao.**

---

## Lộ trình đọc gợi ý

1. [Cẩm nang phỏng vấn Độ phức tạp thời gian & không gian](./complexity-analysis.md): Làm rõ Big-O, độ phức tạp đệ quy và các ngộ nhận thường gặp.
2. [Tổng hợp câu hỏi phỏng vấn Tìm kiếm nhị phân (Binary Search)](./binary-search.md): Luyện Binary Search cơ bản, biên trái, biên phải và tìm kiếm nhị phân trên không gian đáp án.
3. [Tổng hợp câu hỏi phỏng vấn Hai con trỏ & Cửa sổ trượt](./two-pointers-and-sliding-window.md): Giải quyết các dạng bài tần suất cực cao trên Mảng, Chuỗi ký tự và Danh sách liên kết.
4. [Tổng hợp câu hỏi phỏng vấn DFS & BFS](./dfs-bfs.md): Nắm vững duyệt cây, duyệt đồ thị, tìm kiếm trên ma trận và duyệt theo tầng (Level-order).
5. [Tổng hợp câu hỏi phỏng vấn Thuật toán quay lui (Backtracking)](./backtracking.md): Tập trung xử lý các bài toán Tổ hợp, Hoán vị, Tập con và bàn cờ (N-Queens).
6. [Tổng hợp câu hỏi phỏng vấn Quy hoạch động (Dynamic Programming)](./dynamic-programming.md): Tiếp cận từ định nghĩa Trạng thái (State) và Phương trình chuyển trạng thái, không học vẹt lời giải.
7. [Tổng hợp câu hỏi phỏng vấn Thuật toán tham lam (Greedy)](./greedy.md) và [Tổng hợp câu hỏi phỏng vấn Bài toán Top-K](./top-k.md): Bổ sung kỹ thuật Tham lam kết hợp sắp xếp, Heap, Phân vùng Quickselect và Đếm thùng.
8. [Các bài toán thuật toán Chuỗi ký tự thường gặp](./string-algorithm-problems.md), [Các bài toán thuật toán Danh sách liên kết thường gặp](./linkedlist-algorithm-problems.md), [Tổng hợp 10 thuật toán sắp xếp kinh điển](./10-classical-sorting-algorithms.md): Ôn tập chuyên đề tổng lực trước ngày phỏng vấn.

---

## Các Template giải thuật cốt lõi

| Mẫu giải thuật (Template) | Dấu hiệu nhận biết trong đề bài | Bài viết trọng tâm |
| :--- | :--- | :--- |
| **Tìm kiếm nhị phân (Binary Search)** | Mảng có thứ tự, tính đơn điệu, tìm giá trị khả thi nhỏ nhất / lớn nhất | [Tìm kiếm nhị phân](./binary-search.md) |
| **Hai con trỏ (Two Pointers)** | Sửa đổi tại chỗ (In-place), hai đầu co hẹp, con trỏ nhanh chậm, định vị node | [Hai con trỏ & Cửa sổ trượt](./two-pointers-and-sliding-window.md) |
| **Cửa sổ trượt (Sliding Window)** | Mảng con liên tục, chuỗi con liên tục, cửa sổ dài nhất / ngắn nhất | [Hai con trỏ & Cửa sổ trượt](./two-pointers-and-sliding-window.md) |
| **DFS / BFS** | Duyệt cây, duyệt đồ thị, vết loang ma trận, số bước ngắn nhất theo tầng | [DFS & BFS](./dfs-bfs.md) |
| **Quay lui (Backtracking)** | Liệt kê tất cả phương án, chọn đường đi, tổ hợp, hoán vị, ràng buộc bàn cờ | [Quay lui](./backtracking.md) |
| **Quy hoạch động (DP)** | Tìm giá trị tối ưu (Max/Min), đếm số cách, bài toán có thể tới được không, dãy con, cái túi | [Quy hoạch động](./dynamic-programming.md) |
| **Thuật toán tham lam (Greedy)** | Mỗi bước chọn phương án tối ưu cục bộ, thường đi kèm với thao tác sắp xếp | [Tham lam](./greedy.md) |
| **Bài toán Top-K** | Phần tử lớn thứ K, K phần tử xuất hiện nhiều nhất, luồng dữ liệu, mức ưu tiên | [Bài toán Top-K](./top-k.md) |

---

## Lộ trình luyện nhanh 7 ngày

Khi thời gian phỏng vấn đã cận kề, không nên bắt đầu bằng các bài Hard quá hóc búa. Mục tiêu của lộ trình 7 ngày là khôi phục phản xạ viết code Template và kiểm soát tốt các ca biên:

| Ngày | Trọng tâm ôn tập | Hành động gợi ý |
| :--- | :--- | :--- |
| **Ngày 1** | Độ phức tạp + Thuật toán sắp xếp | Ôn tập Big-O, Quick Sort, Merge Sort, Heap Sort và tính ổn định (Stability) |
| **Ngày 2** | Nhị phân + Hai con trỏ | Viết template Biên trái/Biên phải, Two Sum, Three Sum, Xóa phần tử trùng lặp |
| **Ngày 3** | Cửa sổ trượt + Chuỗi ký tự | Viết Chuỗi con không lặp dài nhất, Chuỗi con phủ tối thiểu, Chuỗi đối xứng (Palindrome) |
| **Ngày 4** | Danh sách liên kết | Viết Đảo ngược LinkedList, Phát hiện chu trình (Floyd), Xóa node thứ N từ cuối lên |
| **Ngày 5** | Cây & BFS | Viết duyệt Tiền/Trung/Hậu thứ tự, Duyệt theo tầng (Level-order), Tổ tiên chung gần nhất (LCA) |
| **Ngày 6** | Quay lui + Quy hoạch động | Viết Tập con (Subsets), Tổ hợp (Combinations), Đổi tiền xu (Coin Change), Dãy con tăng dài nhất (LIS) |
| **Ngày 7** | Top-K + Ôn tập tổng hợp | Viết Phần tử lớn thứ K, K phần tử tần suất cao nhất, tổng kết lỗi sai và các trường hợp biên |

---

## Lộ trình bài bản 30 ngày

Với lộ trình 30 ngày, bạn không cần chạy theo số lượng bài giải mỗi ngày. Nhịp độ hiệu quả và bền bỉ nhất là: Mỗi ngày từ **1 đến 3 bài đại diện**, sau khi giải xong viết ngắn gọn 5 dòng đúc kết phản tư (Reflection).

| Giai đoạn | Thời gian | Mục tiêu cụ thể |
| :--- | :--- | :--- |
| **Giai đoạn 1** | Ngày 1 đến 5 | Độ phức tạp, Mảng, Danh sách liên kết, Ngăn xếp, Hàng đợi; đảm bảo tự tay viết code mượt mà |
| **Giai đoạn 2** | Ngày 6 đến 12 | Nhị phân, Hai con trỏ, Cửa sổ trượt, Chuỗi ký tự; tập trung xử lý chắc các điều kiện biên |
| **Giai đoạn 3** | Ngày 13 đến 18 | Cây, Đồ thị, DFS/BFS, Union-Find; xây dựng khung tư duy tìm kiếm bài bản |
| **Giai đoạn 4** | Ngày 19 đến 24 | Quay lui, Quy hoạch động, Tham lam; trọng tâm luyện định nghĩa trạng thái và tỉa nhánh (Pruning) |
| **Giai đoạn 5** | Ngày 25 đến 30 | Top-K, Thuật toán sắp xếp, bài tổng hợp và ôn lại bài sai; luyện giải thích lưu loát khi phỏng vấn |

---

## Tự kiểm tra câu hỏi tần suất cao

- Tại sao khi phân tích độ phức tạp thời gian ta chỉ quan tâm tới bậc cao nhất? Độ phức tạp giải thuật đệ quy tính như thế nào?
- Trong Tìm kiếm nhị phân, khi nào dùng vòng lặp `left < right` và khi nào dùng `left <= right`?
- Kỹ thuật Hai con trỏ và Cửa sổ trượt khác nhau như thế nào?
- DFS và BFS lần lượt phù hợp với những bài toán nào? Khi nào bắt buộc phải dùng mảng `visited`?
- Thuật toán Quay lui và DFS có mối quan hệ gì? Điều kiện tỉa nhánh (Pruning) nên đặt ở đâu?
- Tại sao Quy hoạch động lại là nỗi ám ảnh của nhiều người? Làm sao để xác định định nghĩa trạng thái và thứ tự duyệt bảng DP?
- Tại sao thuật toán Tham lam luôn cần chứng minh tính đúng đắn? Trong phỏng vấn cần giải thích đến mức nào là đủ?
- Đối với bài toán Top-K: Khi nào chọn Heap, khi nào chọn Phân vùng Quickselect, khi nào chọn Đếm thùng?
- Các khái niệm: Tính ổn định (Stable Sort), Sắp xếp tại chỗ (In-place), Độ phức tạp tốt nhất / xấu nhất của các thuật toán sắp xếp là gì?

## Chuyên đề liên quan

- [Hệ thống kiến thức Cơ sở máy tính](../)
- [Chuyên đề Cấu trúc dữ liệu](../data-structure/)
- [Gợi ý các bài tập LeetCode kinh điển theo Cấu trúc dữ liệu](./common-data-structures-leetcode-recommendations.md)
- [Tổng hợp tư duy các thuật toán kinh điển](./classical-algorithm-problems-recommendations.md)
- [Chi tiết Java Collections](../../java/collection/java-collection-questions-01.md)
- [Chuẩn bị phỏng vấn](../../interview-preparation/)
- [Sách tham khảo Cơ sở máy tính](../../books/cs-basics.md)

<!-- @include: @article-footer.snippet.md -->
