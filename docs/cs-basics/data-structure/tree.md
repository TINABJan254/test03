---
title: Chi tiết Cấu trúc Cây (Cây nhị phân, AVL, Cây đỏ đen, B/B+ Tree)
description: Hướng dẫn có hệ thống về các khái niệm cốt lõi của Cây và Cây nhị phân, các phương pháp duyệt cây, tính toán Chiều cao/Độ sâu và tư duy thuật toán nền tảng.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Cây, Cây nhị phân, Cây tìm kiếm nhị phân, BST, Cây cân bằng, AVL Tree, Duyệt cây, Tiền thứ tự, Trung thứ tự, Hậu thứ tự, Tầng thứ tự, Pre-order, In-order, Post-order, Level-order, Chiều cao, Độ sâu
---

Cây (Tree) là một cấu trúc dữ liệu phân cấp phi tuyến tính mô phỏng lại hình ảnh cái cây trong tự nhiên (nhưng lộn ngược, với gốc ở trên cùng). Bất kỳ một cây không rỗng nào cũng chỉ có duy nhất một **Node Gốc (Root Node)**.

Một cây hợp lệ có các đặc điểm sau:
1. Giữa hai node bất kỳ trong cây chỉ tồn tại **duy nhất một đường đi** kết nối chúng.
2. Nếu cây có $n$ node thì nó luôn có chính xác $n - 1$ cạnh nối (Edges).
3. Cây là đồ thị liên thông không chứa chu trình khép kín (Acyclic).

Dưới đây là một ví dụ về cây, cụ thể là một **Cây nhị phân (Binary Tree)**:

![Cây nhị phân](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/%E4%BA%8C%E5%8F%89%E6%A0%91-2.png)

Các khái niệm cơ bản trong cây dựa theo hình minh họa trên:

- **Node (Nút)**: Mỗi phần tử dữ liệu trong cây.
- **Root Node (Nút gốc)**: Node ở tầng cao nhất, không có node cha. Trong hình trên, node `A` là Root.
- **Parent Node (Nút cha)**: Node có nhánh trỏ tới node con. Node `B` là cha của `D` và `E`.
- **Child Node (Nút con)**: Node con của một node cha. `D` và `E` là con của `B`.
- **Sibling Nodes (Nút anh em)**: Các node có cùng một node cha chung. `D` và `E` là anh em.
- **Leaf Node (Nút lá)**: Node không có bất kỳ node con nào (bậc bằng 0). `D`, `F`, `H`, `I` là các Leaf node.
- **Height (Chiều cao của node)**: Số cạnh trên đường đi dài nhất từ node đó xuống một node lá.
- **Depth (Độ sâu của node)**: Số cạnh trên đường đi từ node Gốc đến node đó.
- **Level (Tầng của node)**: Bằng $\text{Độ sâu} + 1$.
- **Chiều cao của cây**: Chính là Chiều cao của node Gốc.

---

## Phân loại Cây nhị phân (Binary Tree)

**Cây nhị phân (Binary Tree)** là cấu trúc cây mà mỗi node có **tối đa 2 nhánh con** (không có node nào có bậc lớn hơn 2).

Hai nhánh con của cây nhị phân được phân biệt rõ ràng thành **Cây con bên trái (Left Subtree)** và **Cây con bên phải (Right Subtree)**. Vị trí trái - phải có thứ tự cố định, không được đảo lộn tùy tiện.

Tại tầng thứ $i$ (với quy ước tầng gốc là 1), cây nhị phân có tối đa $2^{i-1}$ node. Một cây nhị phân có độ sâu $k$ (với độ sâu gốc là 0) có tối đa $2^{k+1} - 1$ node (trường hợp Cây nhị phân đầy đủ), và có ít nhất $k + 1$ node (trường hợp suy thoái thành đường thẳng liên kết).

### 1. Cây nhị phân đầy đủ (Full Binary Tree)

Cây nhị phân mà mọi tầng đều có số lượng node tối đa. Nếu cây có $k$ tầng, tổng số node đạt chính xác $2^k - 1$.

![Cây nhị phân đầy đủ](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/full-binary-tree.png)

### 2. Cây nhị phân hoàn chỉnh (Complete Binary Tree)

Cây nhị phân mà tất cả các tầng từ tầng 1 đến tầng $k-1$ đều chứa đầy đủ các node, và ở tầng cuối cùng (tầng $k$), các node được điền liên tục từ trái qua phải mà không bị khuyết ô nào ở giữa.

![Cây nhị phân hoàn chỉnh](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/complete-binary-tree.png)

**Tính chất cực kỳ quan trọng của Cây nhị phân hoàn chỉnh:**
Khi đánh số các node từ 1 bắt đầu từ gốc:
- Nếu node cha có chỉ số $i$, thì node con bên trái có chỉ số $2i$, và node con bên phải có chỉ số $2i + 1$.
- Node cha của node $i$ có chỉ số là $\lfloor i / 2 \rfloor$.

Tính chất này cho phép ta lưu trữ cây nhị phân hoàn chỉnh trực tiếp vào một **Mảng (Array)** một chiều mà không cần dùng con trỏ, tiết kiệm bộ nhớ tối đa (đây chính là nền tảng của cấu trúc **Heap**).

### 3. AVL Tree (Cây tìm kiếm nhị phân cân bằng tuyệt đối)

**AVL Tree** là một Cây tìm kiếm nhị phân (BST) tự cân bằng với điều kiện:
1. Có thể là cây rỗng.
2. Nếu không rỗng, độ chênh lệch chiều cao giữa cây con trái và cây con phải (Hệ số cân bằng - Balance Factor) của **mọi node** không được vượt quá **1** ($|h_{left} - h_{right}| \le 1$).

![Cây suy thoái thành danh sách liên kết](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/oblique-tree.png)

Nếu không có cơ chế cân bằng, khi ta chèn các phần tử theo thứ tự tăng dần vào một Cây tìm kiếm nhị phân thông thường (BST), cây sẽ bị **suy thoái hoàn toàn thành một danh sách liên kết (Skewed Tree)** với thời gian tìm kiếm tăng vọt lên $O(n)$. Cây AVL sử dụng phép quay (Rotations) để giữ cây luôn cân bằng ở độ cao $O(\log n)$.

![Cây cân bằng AVL](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/balanced-binary-tree.png)

---

## Phương thức lưu trữ Cây nhị phân

### 1. Lưu trữ liên kết (Linked Storage - Dùng con trỏ / Tham chiếu)

Mỗi node là một đối tượng chứa:
- Dữ liệu `data`.
- Tham chiếu node con trái `left`.
- Tham chiếu node con phải `right`.

```java
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;

    public TreeNode(int val) {
        this.val = val;
    }
}
```

![Lưu trữ liên kết cây nhị phân](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/chain-store-binary-tree.png)

### 2. Lưu trữ tuần tự (Sequential Storage - Dùng Mảng)

Sử dụng mảng một chiều, chỉ số mảng đại diện cho quan hệ cha con ($2i$ và $2i+1$). Nếu cây không phải là cây hoàn chỉnh, các vị trí trống sẽ để lại ô nhớ rỗng trong mảng làm giảm hiệu suất sử dụng RAM.

![Lưu trữ tuần tự mảng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/sequential-storage.png)

---

## Các phương pháp duyệt Cây nhị phân (Tree Traversal)

### 1. Tiền thứ tự (Pre-order Traversal: Gốc -> Trái -> Phải)
Duyệt node Gốc trước, sau đó đệ quy duyệt cây con Trái, cuối cùng duyệt cây con Phải.

![Duyệt tiền thứ tự](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/preorder-traversal.png)

```java
public void preOrder(TreeNode root) {
    if (root == null) return;
    System.out.println(root.val); // Xử lý gốc
    preOrder(root.left);          // Trái
    preOrder(root.right);         // Phải
}
```

### 2. Trung thứ tự (In-order Traversal: Trái -> Gốc -> Phải)
Đệ quy duyệt cây con Trái, sau đó duyệt Gốc, cuối cùng duyệt cây con Phải.
> **Đặc tính cốt lõi**: Đối với Cây tìm kiếm nhị phân (BST), duyệt Trung thứ tự sẽ cho ra một **dãy số có thứ tự tăng dần**.

![Duyệt trung thứ tự](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/inorder-traversal.png)

```java
public void inOrder(TreeNode root) {
    if (root == null) return;
    inOrder(root.left);          // Trái
    System.out.println(root.val);// Xử lý gốc
    inOrder(root.right);         // Phải
}
```

### 3. Hậu thứ tự (Post-order Traversal: Trái -> Phải -> Gốc)
Đệ quy duyệt cây con Trái, duyệt cây con Phải, và cuối cùng duyệt node Gốc. Phù hợp cho các thao tác giải phóng bộ nhớ, tính toán chiều cao hoặc xóa cây từ dưới lên.

![Duyệt hậu thứ tự](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/postorder-traversal.png)

```java
public void postOrder(TreeNode root) {
    if (root == null) return;
    postOrder(root.left);         // Trái
    postOrder(root.right);        // Phải
    System.out.println(root.val); // Xử lý gốc
}
```

### 4. Tầng thứ tự (Level-order Traversal / BFS)
Duyệt từng tầng một từ trên xuống dưới, từ trái sang phải bằng cấu trúc Hàng đợi (`Queue`).

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> ans = new ArrayList<>();
    if (root == null) return ans;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        ans.add(level);
    }
    return ans;
}
```

---

## Bảng so sánh các biến thể Cây trong phỏng vấn

| Cấu trúc | Đặc tính cốt lõi | Ứng dụng tiêu biểu |
| :--- | :--- | :--- |
| **Cây nhị phân (Binary Tree)** | Mỗi node có tối đa 2 con | Duyệt cây, Expression Tree, Cấu trúc phân cấp |
| **Cây tìm kiếm nhị phân (BST)** | Nhánh trái < Gốc < Nhánh phải; In-order cho dãy tăng dần | Tra cứu dữ liệu có thứ tự |
| **AVL Tree** | Cân bằng nghiêm ngặt ($|h_L - h_R| \le 1$), tra cứu cực nhanh | Kịch bản đọc nhiều, ít chèn xóa |
| **Red-Black Tree** | Cân bằng gần đúng qua màu Đỏ/Đen, chi phí chèn/xóa thấp hơn AVL | Java `TreeMap`, `TreeSet`, JDK 8 `HashMap` |
| **B-Tree** | Cây tìm kiếm đa phân nhánh cân bằng, mỗi node chứa nhiều Key | Tối ưu hóa I/O đĩa cho File System |
| **B+ Tree** | Dữ liệu chỉ nằm ở Leaf node, các lá liên kết thành danh sách nối vòng | Chỉ mục cơ sở dữ liệu (MySQL InnoDB Index) |

---

## Các bài toán thuật toán kinh điển về Cây

### 1. Kiểm tra Cây tìm kiếm nhị phân hợp lệ (LeetCode 98)
Phải truyền khoảng giá trị hợp lệ `(lower, upper)` xuống từng node thay vì chỉ so sánh với con trực tiếp.

```java
public boolean isValidBST(TreeNode root) {
    return check(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean check(TreeNode node, long lower, long upper) {
    if (node == null) return true;
    if (node.val <= lower || node.val >= upper) return false;
    return check(node.left, lower, node.val) && check(node.right, node.val, upper);
}
```

### 2. Tổ tiên chung gần nhất (LCA - LeetCode 236)
Sử dụng tư duy duyệt Hậu thứ tự để tìm điểm giao nhau của 2 nhánh tìm kiếm:

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root; // p và q nằm ở 2 nhánh khác nhau
    return left != null ? left : right;
}
```

## Đề xuất bài tập luyện tập

- [LeetCode 144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)
- [LeetCode 102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)
- [LeetCode 98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)
- [LeetCode 236. Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)
- [LeetCode 105. Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

<!-- @include: @article-footer.snippet.md -->
