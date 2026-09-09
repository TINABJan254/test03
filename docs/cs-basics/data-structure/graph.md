---
title: Chi tiết Cấu trúc Đồ thị (Biểu diễn, BFS, DFS và Đường đi ngắn nhất)
description: Giới thiệu toàn diện các khái niệm cơ bản và phương thức biểu diễn Đồ thị (Ma trận kề, Danh sách kề), kết hợp với các thuật toán duyệt đồ thị cốt lõi (DFS, BFS, Đường đi ngắn nhất) và ứng dụng thực tế.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Đồ thị, Graph, Danh sách kề, Ma trận kề, DFS, BFS, Bậc của đỉnh, Đồ thị có hướng, Đồ thị vô hướng, Tính liên thông
---

# Đồ thị (Graph)

Đồ thị là một cấu trúc dữ liệu phi tuyến tính phức tạp và linh hoạt bậc nhất trong khoa học máy tính.

So sánh với các cấu trúc dữ liệu đã học:
- **Cấu trúc dữ liệu tuyến tính**: Các phần tử có quan hệ tuyến tính 1-1 duy nhất (mỗi phần tử trừ đầu và cuối chỉ có 1 phần tử đứng trước và 1 phần tử đứng sau).
- **Cấu trúc Cây**: Các phần tử có quan hệ phân cấp 1-N rõ ràng (mỗi node con chỉ có duy nhất 1 node cha).
- **Cấu trúc Đồ thị**: Quan hệ giữa các phần tử là **N-N tùy ý** (bất kỳ đỉnh nào cũng có thể kết nối với bất kỳ đỉnh nào khác).

**Đồ thị là gì?**  
Đơn giản nhất, Đồ thị là một tập hợp gồm **Tập hợp các Đỉnh (Vertices)** hữu hạn, không rỗng và **Tập hợp các Cạnh (Edges)** nối giữa các đỉnh đó. Đồ thị thường được ký hiệu là: **$G(V, E)$**, trong đó:
- $V$ (Vertices) biểu diễn tập hợp các đỉnh.
- $E$ (Edges) biểu diễn tập hợp các cạnh.

![Đồ thị có hướng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/directed-graph.png)

Cấu trúc đồ thị xuất hiện khắp nơi trong thế giới thực: Mạng lưới bạn bè trên mạng xã hội (Facebook, LinkedIn), bản đồ giao thông đường bộ (Google Maps), mạng lưới Internet (Router/Switch), hệ thống gợi ý sản phẩm, đồ thị phụ thuộc giữa các package trong Maven/Gradle.

---

## Các khái niệm cơ bản trong Đồ thị

### 1. Đỉnh (Vertex / Node)
Mỗi phần tử dữ liệu trong đồ thị được gọi là một Đỉnh. Đồ thị luôn có ít nhất 1 đỉnh. Trong mạng xã hội, mỗi tài khoản người dùng đại diện cho một Đỉnh.

### 2. Cạnh (Edge)
Mối quan hệ kết nối giữa hai đỉnh được biểu diễn bằng một Cạnh. Nếu hai người dùng là bạn bè của nhau, giữa hai đỉnh của họ tồn tại một Cạnh.

### 3. Bậc của đỉnh (Degree)
Bậc biểu thị số lượng cạnh gắn liền với đỉnh đó (tương ứng với số lượng bạn bè của một người dùng).
- Trong **Đồ thị có hướng (Directed Graph)**, bậc được chia thành:
  - **Bán bậc ra (Out-degree)**: Số lượng cạnh đi ra từ đỉnh đó.
  - **Bán bậc vào (In-degree)**: Số lượng cạnh đi vào đỉnh đó.

### 4. Đồ thị vô hướng (Undirected Graph) và Đồ thị có hướng (Directed Graph)
- **Đồ thị vô hướng**: Mối quan hệ giữa hai đỉnh mang tính chất 2 chiều bình đẳng (ví dụ: Quan hệ bạn bè 2 chiều trên Facebook, quan hệ bạn cùng lớp). Cạnh nối không có mũi tên định hướng.
- **Đồ thị có hướng**: Mối quan hệ mang tính 1 chiều (ví dụ: Quan hệ Follow trên Twitter/TikTok, quan hệ cha - con, chuyển khoản ngân hàng). Cạnh nối có mũi tên chỉ rõ hướng từ đỉnh nguồn tới đỉnh đích.

### 5. Đồ thị không trọng số (Unweighted Graph) và Đồ thị có trọng số (Weighted Graph)
- **Đồ thị không trọng số**: Chúng ta chỉ quan tâm giữa hai đỉnh có kết nối hay không ($1$ hoặc $0$).
- **Đồ thị có trọng số (Network)**: Mỗi cạnh được gán thêm một giá trị số gọi là **Trọng số (Weight)**, biểu thị khoảng cách vật lý (km), chi phí truyền tải (latency, bandwidth) hoặc độ thân thiết giữa hai đỉnh.

![Đồ thị có hướng có trọng số](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/weighted-directed-graph.png)

---

## Phương thức lưu trữ Đồ thị

### 1. Ma trận kề (Adjacency Matrix)

Ma trận kề biểu diễn đồ thị bằng một **Mảng hai chiều kích thước $V \times V$** (trong đó $V$ là số lượng đỉnh):
- Nếu đỉnh $i$ và đỉnh $j$ có cạnh nối mang trọng số $w$, thì `matrix[i][j] = w`.
- Đối với đồ thị vô hướng không trọng số: `matrix[i][j] = 1` nếu có cạnh nối, ngược lại bằng `0`.

![Ma trận kề đồ thị vô hướng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/adjacency-matrix-representation-of-undirected-graph.png)

> **Lưu ý**: Ma trận kề của đồ thị vô hướng luôn là một **Ma trận đối xứng** qua đường chéo chính (`matrix[i][j] == matrix[j][i]`).

![Ma trận kề đồ thị có hướng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/adjacency-matrix-representation-of-directed-graph.png)

- **Ưu điểm**: Cực kỳ trực quan, kiểm tra xem hai đỉnh có nối với nhau hay không chỉ mất thời gian **$O(1)$**.
- **Nhược điểm**: Tốn không gian bộ nhớ **$O(V^2)$**. Nếu đồ thị là đồ thị thưa (Sparse Graph - số cạnh ít hơn rất nhiều so với $V^2$), ma trận kề sẽ gây lãng phí bộ nhớ nghiêm trọng.

### 2. Danh sách kề (Adjacency List)

Để giải quyết nhược điểm lãng phí RAM của ma trận kề, **Danh sách kề** được sử dụng phổ biến nhất:
Mỗi đỉnh $V_i$ sở hữu một danh sách liên kết (hoặc `ArrayList`) chứa toàn bộ các đỉnh kề trực tiếp với nó.

![Danh sách kề đồ thị vô hướng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/adjacency-list-representation-of-undirected-graph.png)

![Danh sách kề đồ thị có hướng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/adjacency-list-representation-of-directed-graph.png)

- **Bộ nhớ tiêu thụ**:
  - Đồ thị vô hướng: Chứa $2E$ phần tử (mỗi cạnh xuất hiện ở cả 2 danh sách kề).
  - Đồ thị có hướng: Chứa đúng $E$ phần tử.
- **Ưu điểm**: Tiết kiệm bộ nhớ tối đa ($O(V + E)$), duyệt qua tất cả các đỉnh lân cận của một đỉnh cực nhanh.

---

## Các thuật toán duyệt Đồ thị (Graph Traversal)

### 1. Tìm kiếm theo chiều rộng (BFS - Breadth-First Search)

BFS duyệt đồ thị lan tỏa ra ngoài theo từng lớp vòng tròn đồng tâm như sóng nước lan trên mặt hồ.

![Minh họa BFS](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/breadth-first-search.png)

**Cài đặt BFS sử dụng cấu trúc Hàng đợi (Queue)**:
1. Đưa đỉnh xuất phát vào Hàng đợi, đánh dấu đỉnh đó đã thăm (`visited[start] = true`).
2. Lặp khi hàng đợi chưa rỗng: Lấy đỉnh `u` ra khỏi hàng đợi; duyệt tất cả các đỉnh láng giềng `v` của `u`. Nếu `v` chưa thăm, đánh dấu `visited[v] = true` và đưa `v` vào hàng đợi.

> **Ứng dụng quan trọng nhất của BFS**: Tìm **Đường đi ngắn nhất** (Số bước ít nhất) trên đồ thị không trọng số.

### 2. Tìm kiếm theo chiều sâu (DFS - Depth-First Search)

DFS thực hiện chiến lược "đi một mạch tới tận cùng con đường", khi chạm ngõ cụt thì mới quay lui (Backtrack) về đỉnh liền trước để thử các ngã rẽ khác.

![Minh họa DFS](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/depth-first-search.png)

**Cài đặt DFS sử dụng Ngăn xếp (Stack) hoặc Đệ quy (Recursion)**:
1. Đánh dấu đỉnh hiện tại đã thăm (`visited[u] = true`).
2. Với mỗi đỉnh kề `v` của `u`: Nếu `v` chưa được thăm, đệ quy gọi `DFS(v)`.

---

## Bảng so sánh các cấu trúc và thuật toán đồ thị trong phỏng vấn

| Phương thức | Không gian | Kiểm tra 2 đỉnh kề nhau | Duyệt láng giềng của 1 đỉnh | Phù hợp |
| :--- | :--- | :--- | :--- | :--- |
| **Ma trận kề** | $O(V^2)$ | $O(1)$ | $O(V)$ | Đồ thị dày (Dense Graph), số đỉnh nhỏ |
| **Danh sách kề** | $O(V + E)$ | $O(\text{degree}(u))$ | $O(\text{degree}(u))$ | Đồ thị thưa (Sparse Graph), bài tập giải thuật |

### Các thuật toán kinh điển cần nắm:
- **Tìm đường đi ngắn nhất đồ thị không trọng số**: Dùng **BFS** ($O(V + E)$).
- **Tìm đường đi ngắn nhất đồ thị có trọng số dương**: Dùng **Dijkstra** kết hợp `PriorityQueue` ($O(E \log V)$).
- **Sắp xếp lịch trình / Phụ thuộc tác vụ (DAG)**: Dùng **Sắp xếp tô-pô (Topological Sort)** (Thuật toán Kahn dùng BFS In-degree).
- **Kiểm tra tính liên thông / Phát hiện chu trình đồ thị vô hướng**: Dùng **Union-Find (DSU)** hoặc DFS/BFS.

---

## Template mã nguồn Java biểu diễn Đồ thị và BFS

```java
// Xây dựng Danh sách kề cho đồ thị
List<Integer>[] buildGraph(int n, int[][] edges) {
    List<Integer>[] graph = new ArrayList[n];
    for (int i = 0; i < n; i++) {
        graph[i] = new ArrayList<>();
    }
    for (int[] edge : edges) {
        int from = edge[0];
        int to = edge[1];
        graph[from].add(to);
        // Nếu là đồ thị vô hướng, thêm cạnh ngược lại:
        // graph[to].add(from);
    }
    return graph;
}

// Thuật toán BFS tìm số bước ngắn nhất
int bfs(List<Integer>[] graph, int start, int target) {
    boolean[] visited = new boolean[graph.length];
    Queue<Integer> queue = new ArrayDeque<>();
    queue.offer(start);
    visited[start] = true;
    int step = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int cur = queue.poll();
            if (cur == target) {
                return step;
            }
            for (int next : graph[cur]) {
                if (!visited[next]) {
                    visited[next] = true;
                    queue.offer(next);
                }
            }
        }
        step++;
    }
    return -1; // Không tìm thấy đường đi
}
```

## Đề xuất bài tập luyện tập

- [LeetCode 200. Number of Islands](https://leetcode.com/problems/number-of-islands/)
- [LeetCode 695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/)
- [LeetCode 994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)
- [LeetCode 207. Course Schedule](https://leetcode.com/problems/course-schedule/)
- [LeetCode 547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/)

<!-- @include: @article-footer.snippet.md -->
