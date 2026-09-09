---
title: Tổng hợp câu hỏi phỏng vấn Union-Find: Nén đường đi, Tính liên thông và Java Template
description: Tổng hợp câu hỏi phỏng vấn về cấu trúc Union-Find (Disjoint Set Union - DSU), phân tích các thao tác find, union, kỹ thuật Nén đường đi (Path Compression), Hợp nhất theo kích thước (Union by Size), kiểm tra liên thông, phát hiện chu trình đồ thị và các bài toán LeetCode tần suất cao.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Union-Find, Disjoint Set Union, DSU, Nén đường đi, Path Compression, Union by Size, Tính liên thông, Thuật toán đồ thị, Phát hiện chu trình, Java Union-Find
---

**Union-Find (hay Disjoint Set Union - DSU / Tập hợp các phần tử không giao nhau)** là cấu trúc dữ liệu chuyên biệt để giải quyết các bài toán về **Phân nhóm (Grouping)** và **Tính liên thông (Connectivity)**.

Các bài toán điển hình:
- Hai phần tử bất kỳ có thuộc về cùng một nhóm/mạng lưới hay không?
- Sau khi kết nối các đỉnh thì đồ thị còn lại bao nhiêu Thành phần liên thông (Connected Components)?
- Thêm một cạnh mới vào đồ thị có tạo thành chu trình khép kín (Cycle Detection) hay không?

Điểm nổi bật của Union-Find là: Mã nguồn rất ngắn gọn nhưng nếu áp dụng **Nén đường đi (Path Compression)** và **Hợp nhất theo kích thước (Union by Size)** thì tốc độ thực thi đạt mức **gần như hằng số ($O(\alpha(n))$)**.

![Sơ đồ cấu trúc cây cha con trong Union-Find](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/union-find.png)

## 1. Union-Find là gì?

Union-Find quản lý một tập hợp gồm nhiều nhóm/tập con không giao nhau. Cấu trúc này tối ưu cho 2 thao tác nghiệp vụ:
1. **Truy vấn (`find`)**: Xác định phần tử đại diện (Node Gốc / Representative) của nhóm chứa phần tử $x$, từ đó kiểm tra 2 phần tử có cùng một nhóm hay không.
2. **Hợp nhất (`union`)**: Gộp hai nhóm chứa phần tử $a$ và phần tử $b$ lại thành một nhóm duy nhất.

Union-Find không quan tâm tới cấu trúc hình học chi tiết bên trong nhóm hay đường đi cụ thể giữa 2 đỉnh (đó là việc của BFS/DFS), mà chỉ tập trung duy nhất vào câu hỏi: *"Hai phần tử này đã được kết nối vào cùng một mạng lưới liên thông hay chưa?"*.

---

## 2. Cách Union-Find biểu diễn tập hợp bằng Mảng

Union-Find sử dụng một mảng **`parent`** để mô phỏng một **Rừng các cây (Forest of Trees)**:
- `parent[x]` lưu trữ chỉ số của Node cha trực tiếp của phần tử `x`.
- Nếu `parent[x] == x`, điều đó có nghĩa `x` chính là **Node Gốc (Root / Đại diện nhóm)** của tập hợp đó.

Khởi tạo ban đầu: Mỗi phần tử là một nhóm độc lập gồm chính nó, do đó `parent[i] = i`.

```text
parent[0] = 0
parent[1] = 1
parent[2] = 2
...
```

Khi thực hiện `union(0, 1)`, ta tìm gốc của `0` (là `0`) và gốc của `1` (là `1`), sau đó gán `parent[1] = 0`. Lúc này `0` trở thành gốc chung của cả `0` và `1`.

---

## 3. Hai kỹ thuật tối ưu hóa cốt lõi: Nén đường đi & Hợp nhất theo kích thước

Nếu chỉ cài đặt đơn giản (Quick Union), các cây có thể bị suy thoái thành đường thẳng dài với chiều cao $O(n)$, khiến thao tác `find` bị chậm. Chúng ta sử dụng 2 kỹ thuật tối ưu hóa sau:

### 1. Nén đường đi (Path Compression)
Trong quá trình thực hiện `find(x)`, sau khi tìm thấy Node Gốc thực sự, ta **gán trực tiếp con trỏ `parent[x]` trỏ thẳng về Node Gốc đó**. Tất cả các node trên đường đi tìm kiếm đều được làm phẳng xuống độ sâu 1.

```java
int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]); // Nén đường đi đệ quy
    }
    return parent[x];
}
```

### 2. Hợp nhất theo kích thước (Union by Size / Union by Rank)
Khi gộp 2 cây, ta luôn gắn **cây có số lượng node ít hơn (size nhỏ hơn) vào bên dưới gốc của cây có size lớn hơn**, giúp khống chế độ sâu của cây không bị tăng nhanh.

---

## 4. Template mã nguồn chuẩn Java

```java
class UnionFind {
    private final int[] parent; // Mảng lưu node cha
    private final int[] size;   // Mảng lưu kích thước của từng cây
    private int count;          // Số lượng thành phần liên thông còn lại

    public UnionFind(int n) {
        this.parent = new int[n];
        this.size = new int[n];
        this.count = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i; // Khởi tạo mỗi node là gốc của chính nó
            size[i] = 1;   // Ban đầu mỗi nhóm có kích thước 1
        }
    }

    // Tìm gốc đại diện kèm Nén đường đi
    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    // Hợp nhất 2 tập hợp chứa a và b
    public boolean union(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);
        if (rootA == rootB) {
            return false; // Đã thuộc cùng một nhóm, không cần hợp nhất
        }

        // Hợp nhất cây nhỏ vào cây lớn
        if (size[rootA] < size[rootB]) {
            parent[rootA] = rootB;
            size[rootB] += size[rootA];
        } else {
            parent[rootB] = rootA;
            size[rootA] += size[rootB];
        }
        count--; // Số lượng nhóm giảm đi 1
        return true;
    }

    // Kiểm tra xem a và b có liên thông không
    public boolean connected(int a, int b) {
        return find(a) == find(b);
    }

    // Trả về số lượng thành phần liên thông hiện tại
    public int count() {
        return count;
    }
}
```

---

## 5. Phân tích Độ phức tạp

Khi kết hợp cả **Nén đường đi** và **Hợp nhất theo kích thước**:
- Độ phức tạp thời gian cho mỗi thao tác `find` / `union`: **$O(\alpha(n))$**, trong đó $\alpha(n)$ là hàm ngược Ackermann. Với mọi giá trị $n$ trong vũ trụ thực tế ($n < 10^{80}$), $\alpha(n) \le 4$, tức là tốc độ thực thi đạt mức **hằng số thời gian $O(1)$**!
- Độ phức tạp không gian: **$O(n)$** (mảng `parent` và `size`).

---

## Ứng dụng thực tế và Các bài toán kinh điển

| Bài toán | Cách xử lý bằng Union-Find |
| :--- | :--- |
| **Kiểm tra đồ thị vô hướng có chu trình** | Duyệt qua từng cạnh `(u, v)`. Nếu `find(u) == find(v)` thì việc thêm cạnh này sẽ tạo thành chu trình khép kín! |
| **Thuật toán Kruskal tìm Cây khung nhỏ nhất (MST)** | Sắp xếp các cạnh theo trọng số tăng dần; lần lượt thêm cạnh vào cây khung nếu 2 đỉnh chưa liên thông (`union`). |
| **Bài toán Số lượng đảo / Số tỉnh thành (Provinces)** | Khởi tạo $N$ phần tử, duyệt qua các liên kết và gọi `union`, kết quả cuối cùng là `uf.count()`. |
| **Kiểm tra tính nhất quán của hệ phương trình đẳng thức** | Gộp các biến có dấu bằng `a == b` vào cùng nhóm; sau đó duyệt lại các điều kiện `a != b`, nếu cùng nhóm thì mâu thuẫn. |

## Đề xuất bài tập luyện tập

- [LeetCode 547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/)
- [LeetCode 684. Redundant Connection](https://leetcode.com/problems/redundant-connection/)
- [LeetCode 990. Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/)
- [LeetCode 1319. Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/)
- [LeetCode 200. Number of Islands](https://leetcode.com/problems/number-of-islands/)

<!-- @include: @article-footer.snippet.md -->
