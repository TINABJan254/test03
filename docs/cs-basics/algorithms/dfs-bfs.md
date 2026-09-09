---
title: "Tổng hợp bài toán phỏng vấn DFS và BFS: Cây, Đồ thị, Tìm kiếm trên ma trận và Template đường đi ngắn nhất"
description: "Tổng hợp bài toán phỏng vấn DFS và BFS, giải thích Depth-First Search, Breadth-First Search, duyệt cây, duyệt đồ thị, tìm kiếm trên ma trận, Level-order Traversal, đường đi ngắn nhất và Java template."
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: DFS,BFS,Depth-First Search,Breadth-First Search,Duyệt cây,Duyệt đồ thị,Tìm kiếm trên ma trận,Level-order Traversal,Đường đi ngắn nhất,Java DFS,Java BFS,LeetCode
---

DFS (Depth-First Search - Tìm kiếm theo chiều sâu) và BFS (Breadth-First Search - Tìm kiếm theo chiều rộng) là nền tảng của các bài toán về Cây (Tree), Đồ thị (Graph) và Ma trận (Matrix). Trong phỏng vấn, người phỏng vấn sẽ không chỉ hỏi đơn thuần "DFS là gì", mà phổ biến hơn là đưa ra bài toán đếm số đảo (Number of Islands), phụ thuộc khóa học (Course Schedule), số bước di chuyển ngắn nhất hoặc duyệt cây nhị phân theo tầng (Binary Tree Level Order Traversal), yêu cầu bạn chọn phương pháp tìm kiếm phù hợp và xử lý chuẩn xác các điều kiện biên.

Một quy tắc phán đoán đơn giản: Khi cần đi một mạch đến tận cùng, vét cạn/liệt kê tất cả đường đi hoặc xử lý thành phần liên thông (connected component), hãy ưu tiên nghĩ đến DFS; khi cần duyệt theo từng tầng, tìm số bước ngắn nhất/khoảng cách ngắn nhất, hãy ưu tiên nghĩ đến BFS.

## Trọng tâm khảo sát trong phỏng vấn

- Viết thành thạo đệ quy DFS và hàng đợi (Queue) BFS.
- Nêu rõ độ phức tạp thuật toán khi tìm kiếm trên cây và đồ thị.
- Xử lý mảng hoặc tập hợp `visited` để tránh duyệt lặp và vòng lặp vô tận (infinite loop).
- Phân biệt rõ giữa "duyệt qua tất cả các đỉnh" và "tìm số bước ngắn nhất".
- Chuyển đổi thành thạo các bài toán ma trận lưới thành bài toán tìm kiếm trên đồ thị.

## Khi nào nên chọn DFS, khi nào chọn BFS?

Cả DFS và BFS đều có thể duyệt qua các node, nhưng ưu thế tự nhiên của chúng khác nhau:

| Mục tiêu | Thường dùng | Nguyên nhân |
| ---------------- | ----------------- | ---------------------------- |
| Duyệt qua tất cả các node | DFS hoặc BFS đều được | Miễn là không truy cập lặp lại |
| Tìm diện tích thành phần liên thông | DFS thuận tiện hơn | Đệ quy mở rộng liên tục, code ngắn gọn |
| Tìm số bước ngắn nhất trong đồ thị vô hướng không trọng số | BFS | Lan tỏa theo từng tầng, lần đầu tiên chạm tới đích chắc chắn là ngắn nhất |
| Liệt kê tất cả các đường đi | DFS | Đường đi được lưu trữ tự nhiên trong ngăn xếp đệ quy (call stack) |
| Duyệt cây nhị phân theo tầng | BFS | Hàng đợi xử lý chính xác theo từng tầng |

Nếu bài toán xuất hiện cụm từ "ít bước nhất", "đường đi ngắn nhất", "lan tỏa ra toàn bộ các vị trí", hãy nghĩ đến BFS trước. Nếu bài toán xuất hiện cụm từ "tất cả các phương án", "có tồn tại đường đi hay không", "kích thước thành phần liên thông", hãy nghĩ đến DFS trước.

## Template DFS

Cách viết DFS phổ biến trên ma trận:

```java
void dfs(char[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length) {
        return;
    }
    if (grid[i][j] != '1') {
        return;
    }
    grid[i][j] = '2';
    dfs(grid, i + 1, j);
    dfs(grid, i - 1, j);
    dfs(grid, i, j + 1);
    dfs(grid, i, j - 1);
}
```

Ở đây, ta trực tiếp đổi các ô đất liền đã duyệt thành `'2'`, tương đương với việc dùng chính mảng đầu vào để đánh dấu đã truy cập. Nếu đề bài không cho phép sửa đổi mảng đầu vào, hãy tạo riêng một mảng `boolean[][] visited`.

Hàm đệ quy DFS cần định nghĩa rõ ràng ý nghĩa trước khi viết. Đoạn code trên có thể diễn giải là: Xuất phát từ ô `(i, j)`, đánh dấu toàn bộ các ô đất liền liên thông với nó.

Ý nghĩa này quyết định thứ tự thực thi trong code:

1. Vượt quá biên ma trận: `return` ngay lập tức.
2. Ô hiện tại không phải đất liền: `return` ngay lập tức.
3. Đánh dấu ô hiện tại đã duyệt để tránh truy cập lặp lại.
4. Đệ quy tiếp tục duyệt sang 4 hướng: trên, dưới, trái, phải.

Rất nhiều bug trong DFS xuất phát từ việc thực hiện bước 3 quá muộn. Nếu đệ quy duyệt các ô lân cận trước rồi mới đánh dấu ô hiện tại, chương trình có thể bị đệ quy qua lại vô tận giữa hai ô kề nhau.

## Template BFS

BFS rất thích hợp cho việc duyệt theo tầng và tìm số bước ngắn nhất. Template dưới đây quy ước đầu vào là ma trận chữ nhật không rỗng, trong đó `0` biểu thị đường đi được, `1` biểu thị vật cản; hàm trả về số bước ngắn nhất từ điểm xuất phát đến điểm đích, nếu tọa độ vượt biên hoặc không thể đến đích thì trả về `-1`:

```java
int bfs(int[][] grid, int startX, int startY, int targetX, int targetY) {
    if (grid == null || grid.length == 0 || grid[0].length == 0) {
        return -1;
    }
    int rows = grid.length;
    int columns = grid[0].length;
    if (startX < 0 || startX >= rows || startY < 0 || startY >= columns
            || targetX < 0 || targetX >= rows || targetY < 0 || targetY >= columns
            || grid[startX][startY] == 1 || grid[targetX][targetY] == 1) {
        return -1;
    }
    int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    Queue<int[]> queue = new ArrayDeque<>();
    queue.offer(new int[] {startX, startY});
    boolean[][] visited = new boolean[rows][columns];
    visited[startX][startY] = true;
    int step = 0;
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int k = 0; k < size; k++) {
            int[] cur = queue.poll();
            if (cur[0] == targetX && cur[1] == targetY) {
                return step;
            }
            for (int[] dir : dirs) {
                int x = cur[0] + dir[0];
                int y = cur[1] + dir[1];
                if (x < 0 || x >= rows || y < 0 || y >= columns
                        || visited[x][y] || grid[x][y] == 1) {
                    continue;
                }
                visited[x][y] = true;
                queue.offer(new int[] {x, y});
            }
        }
        step++;
    }
    return -1;
}
```

Đoạn code này trả về số tầng hiện tại ngay khi node đích lần đầu tiên được lấy ra khỏi hàng đợi, chứ không đợi đến khi hàng đợi rỗng hoàn toàn. Điều kiện đi được trong từng bài toán cụ thể có thể khác nhau (không nhất thiết là `0` và `1`), cần điều chỉnh theo đúng yêu cầu đề bài.

Điểm mấu chốt của BFS là "xử lý theo từng tầng". Ban đầu trong hàng đợi là các node tầng 0, mỗi vòng lặp lấy ra kích thước hàng đợi hiện tại `size`, chỉ xử lý đúng số node thuộc tầng này; các node mới được mở rộng từ chúng sẽ thuộc về tầng tiếp theo.

Tại sao BFS trên đồ thị không trọng số tìm được đường đi ngắn nhất? Vì chi phí của mỗi cạnh là như nhau. Khi BFS lần đầu tiên chạm tới một node nào đó, chắc chắn nó đã đi qua số cạnh ít nhất. Về sau dù có đường khác đến được node đó thì cũng không thể ngắn hơn, vì vậy ta có thể đánh dấu đã duyệt ngay lập tức.

Dạng bài Multi-source BFS (BFS đa nguồn) cũng rất phổ biến. Ví dụ trong bài "Rotting Oranges" (Cam thối), tất cả các quả cam thối ban đầu cùng đồng thời lan tỏa. Cách làm là đưa tất cả các quả cam thối ban đầu vào hàng đợi trước, sau đó lan tỏa theo từng tầng.

## Sự khác biệt giữa tìm kiếm trên Cây và Đồ thị

Cây không có chu trình (cycle), phần lớn trường hợp không cần mảng `visited`. Đồ thị có thể có chu trình nên bắt buộc phải xem xét việc tránh duyệt lặp.

| Kịch bản | Có thường dùng `visited` không | Giải thích |
| -------------- | ------------------ | ---------------------- |
| Duyệt đệ quy cây nhị phân | Thường không cần | Node con không có cạnh nối ngược về node cha |
| Duyệt đồ thị vô hướng | Bắt buộc cần | Nếu không hai node sẽ liên tục duyệt qua lại lẫn nhau |
| Duyệt đồ thị có hướng | Thường cần | Có thể tồn tại chu trình |
| Tìm kiếm trên ma trận | Bắt buộc cần | Trên dưới trái phải có thể quay lại điểm xuất phát |

## Độ phức tạp

Trong tìm kiếm đồ thị, ta thường dùng `V` biểu thị số đỉnh (vertices) và `E` biểu thị số cạnh (edges). Khi lưu trữ bằng danh sách kề (adjacency list), độ phức tạp thời gian của cả DFS và BFS thông thường là `O(V + E)`, độ phức tạp không gian là `O(V)`.

Với tìm kiếm trên ma trận kích thước `m * n`, mỗi ô tối đa được duyệt một lần, độ phức tạp thời gian là `O(mn)`, không gian đánh dấu đã duyệt hoặc hàng đợi trong trường hợp xấu nhất cũng là `O(mn)`.

## Chuyển bài toán ma trận thành đồ thị như thế nào?

Mỗi ô trong ma trận đều có thể xem như một đỉnh (node) trong đồ thị. 4 hướng trên, dưới, trái, phải chính là các cạnh nối từ đỉnh đó ra ngoài.

Mảng hướng (direction array) thường dùng:

```java
int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
```

Khi duyệt các ô lân cận, ta chỉ cần làm 3 việc:

1. Tính toán tọa độ mới.
2. Kiểm tra xem có vượt ra ngoài biên ma trận không.
3. Kiểm tra xem đã duyệt qua chưa hoặc có thỏa mãn điều kiện đề bài không.

Nếu đề bài cho phép di chuyển theo đường chéo, chỉ cần mở rộng mảng hướng thành 8 hướng. Đừng viết 4 đoạn gọi đệ quy lặp đi lặp lại trong code, dùng mảng hướng sẽ gọn gàng và ít khi bị sót điều kiện.

## Minh họa quy trình và các trường hợp biên

Lấy bài toán đếm số đảo (Number of Islands) làm ví dụ: khi gặp một ô đất liền chưa được duyệt, ta bắt đầu DFS/BFS từ ô đó để đánh dấu toàn bộ hòn đảo.

| Bước | Thao tác | Mục đích |
| ---- | ---------------------- | ---------------------- |
| 1 | Quét qua ma trận, tìm thấy một ô `'1'` | Phát hiện một hòn đảo mới |
| 2 | Tăng số lượng đảo thêm 1 | Ghi nhận một thành phần liên thông |
| 3 | Thực hiện DFS/BFS từ ô hiện tại | Đánh dấu toàn bộ đất liền của hòn đảo này |
| 4 | Tiếp tục quét các ô tiếp theo | Tránh thống kê trùng lặp cùng một hòn đảo |

Với tìm kiếm trên ma trận, bạn nên kiểm tra các trường hợp biên sau:

| Đầu vào | Trọng tâm kiểm tra |
| ------------ | ---------------------------------- |
| Ma trận rỗng | Đã kiểm tra số hàng và số cột chưa |
| Toàn bộ là nước | Kết quả phải bằng 0 |
| Toàn bộ là đất liền | Chỉ được tính thành đúng 1 thành phần liên thông |
| Chỉ tiếp xúc theo đường chéo | Nếu đề bài chỉ cho phép 4 hướng thì không được tính là liên thông |

Lỗi thường gặp khi viết code:

```java
void dfs(char[][] grid, int i, int j) {
    dfs(grid, i + 1, j);
    grid[i][j] = '2'; // Sai: Đánh dấu quá muộn, có thể gây đệ quy qua lại vô tận
}
```

Việc đánh dấu đã duyệt phải hoàn thành trước khi gọi đệ quy mở rộng sang các ô lân cận. Trong đồ thị và ma trận, chỉ cần tồn tại cạnh quay ngược hoặc kề nhau hai chiều, đánh dấu quá muộn sẽ gây duyệt lặp và thậm chí tràn ngăn xếp (Stack Overflow).

## Các lỗi thường gặp (Pitfalls)

- Với BFS, phải đánh dấu đã duyệt ngay khi đưa vào hàng đợi (`offer`), tránh trường hợp một node bị thêm vào hàng đợi nhiều lần.
- DFS có độ sâu đệ quy quá lớn có thể dẫn đến tràn ngăn xếp (Stack Overflow); trong phỏng vấn có thể giải thích phương án chuyển sang dùng ngăn xếp tường minh (explicit stack).
- Bài toán ma trận phải kiểm tra điều kiện vượt biên trước khi truy cập vào phần tử mảng.
- Đồ thị vô hướng phải chú ý vấn đề node con duyệt ngược lại node cha.
- Khi tìm số bước ngắn nhất, việc đếm tầng trong BFS phải gắn chặt với kích thước tầng hiện tại của hàng đợi (`queue.size()`).

## Bài tập rèn luyện đề xuất

- [102. Binary Tree Level Order Traversal](https://leetcode.cn/problems/binary-tree-level-order-traversal/)
- [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/)
- [695. Max Area of Island](https://leetcode.cn/problems/max-area-of-island/)
- [994. Rotting Oranges](https://leetcode.cn/problems/rotting-oranges/)
- [207. Course Schedule](https://leetcode.cn/problems/course-schedule/)

<!-- @include: @article-footer.snippet.md -->
