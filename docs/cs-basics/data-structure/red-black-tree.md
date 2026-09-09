---
title: Chi tiết Cây đỏ đen (Red-Black Tree: Tính chất, Phép quay và Ứng dụng)
description: Phân tích chuyên sâu về 5 tính chất cốt lõi của Cây đỏ đen (Red-Black Tree), các thao tác đổi màu, phép quay trái/quay phải, cơ chế tự cân bằng và ứng dụng trong Java Collections (TreeMap, HashMap).
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Cây đỏ đen, Red-Black Tree, Tự cân bằng, Phép quay, Phép quay trái, Phép quay phải, Đổi màu, Chiều cao đen, Black Height, TreeMap, HashMap
---

# Cây đỏ đen (Red-Black Tree)

## 1. Giới thiệu Cây đỏ đen

**Cây đỏ đen (Red-Black Tree)** là một dạng Cây tìm kiếm nhị phân tự cân bằng (Self-balancing Binary Search Tree). Cấu trúc này được phát minh vào năm 1972 bởi Rudolf Bayer (với tên gọi ban đầu là *Symmetric Binary B-Tree*), và sau đó được hoàn thiện, đặt tên là "Cây đỏ đen" vào năm 1978 bởi Leo J. Guibas và Robert Sedgewick.

Nhờ đặc tính tự cân bằng, Cây đỏ đen đảm bảo rằng trong trường hợp xấu nhất, các thao tác tìm kiếm, chèn và xóa đều được hoàn thành trong thời gian **$O(\log n)$**, mang lại hiệu năng vận hành vô cùng ổn định.

Trong hệ sinh thái JDK, `TreeMap`, `TreeSet` và cơ chế Treeify của `HashMap` (từ JDK 1.8 trở đi) đều sử dụng Cây đỏ đen bên dưới.

---

## 2. Tại sao chúng ta cần Cây đỏ đen?

Sự ra đời của Cây đỏ đen nhằm giải quyết triệt để khuyết tật chí mạng của **Cây tìm kiếm nhị phân thông thường (BST)**.

Trong BST thông thường, hình dạng của cây phụ thuộc hoàn toàn vào thứ tự chèn dữ liệu:
- Nếu dữ liệu được chèn ngẫu nhiên, cây phân nhánh cân đối và đạt tốc độ $O(\log n)$.
- Nếu dữ liệu được chèn theo thứ tự tăng dần hoặc giảm dần, cây sẽ bị **suy thoái hoàn toàn thành một danh sách liên kết đơn (Skewed Tree)**, khiến độ phức tạp của mọi thao tác tìm kiếm/chèn/xóa tụt dốc từ $O(\log n)$ xuống $O(n)$.

Cây đỏ đen ra đời để loại bỏ khả năng suy thoái này, đảm bảo chiều cao của cây luôn được khống chế ở mức không vượt quá $2 \log_2(n + 1)$.

---

## 3. Năm tính chất cốt lõi của Cây đỏ đen

Một Cây tìm kiếm nhị phân được gọi là Cây đỏ đen khi và chỉ khi nó thỏa mãn đầy đủ **5 quy tắc bất biến** sau:

1. **Mọi Node đều chỉ có một trong hai màu: Đỏ (RED) hoặc Đen (BLACK).**
2. **Node Gốc (Root) luôn luôn có màu ĐEN.**
3. **Mọi Node lá rỗng (NIL / Null leaf) đều được coi là Node ĐEN.**
4. **Nếu một Node có màu ĐỎ thì cả hai Node con của nó bắt buộc phải có màu ĐEN** (Nói cách khác: **Không bao giờ xuất hiện hai Node màu đỏ liền kề nhau** trên cùng một nhánh).
5. **Với mọi Node bất kỳ, mọi đường đi đơn từ Node đó xuống các Node lá NIL hậu duệ đều chứa số lượng Node ĐEN bằng nhau** (Tính chất này gọi là **Chiều cao đen đồng nhất - Black Height**).

Chính sự kết hợp của Tính chất 4 (không có 2 đỏ liên tiếp) và Tính chất 5 (chiều cao đen bằng nhau) đã ép cho đường đi dài nhất từ gốc đến lá không bao giờ dài quá gấp đôi đường đi ngắn nhất, giữ cho cây luôn đạt trạng thái **Cân bằng gần đúng**.

---

## 4. Cấu trúc dữ liệu và Triển khai Node

```java
public class Node {
    public Integer value;
    public Node parent;
    public Node left;
    public Node right;

    // Thuộc tính màu sắc của Cây đỏ đen
    public Color color = Color.RED; // Node mới chèn vào mặc định là màu ĐỎ
}
```

---

## 5. Các thao tác tự cân bằng: Đổi màu và Phép quay

Khi chèn hoặc xóa node, các quy tắc trên có thể bị vi phạm. Cây đỏ đen tự khôi phục lại trạng thái cân bằng thông qua 2 thao tác:

### 1. Đổi màu (Coloring / Recoloring)
Đổi màu các node liên quan (Đỏ $\leftrightarrow$ Đen) để phân phối lại chiều cao đen mà không làm thay đổi cấu trúc hình học của cây.

### 2. Phép quay (Rotations)
Thay đổi cấu trúc liên kết con trỏ giữa các node để giảm bớt chiều cao của nhánh bị lệch:
- **Quay trái (Left Rotation)**: Nâng node con bên phải lên làm cha, hạ node hiện tại xuống làm con bên trái.
- **Quay phải (Right Rotation)**: Nâng node con bên trái lên làm cha, hạ node hiện tại xuống làm con bên phải.

![Cây đỏ đen minh họa phép quay và đổi màu](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/red-black-tree-3.png)

---

## So sánh: AVL Tree vs Cây đỏ đen (Red-Black Tree)

| Tiêu chí so sánh | AVL Tree | Cây đỏ đen (Red-Black Tree) |
| :--- | :--- | :--- |
| **Mức độ cân bằng** | Cân bằng nghiêm ngặt ($|h_L - h_R| \le 1$) | Cân bằng gần đúng ($h_{max} \le 2 h_{min}$) |
| **Hiệu suất tìm kiếm** | Nhanh hơn một chút vì cây phẳng hơn | Rất nhanh ($O(\log n)$) |
| **Chi phí Chèn / Xóa** | Thường xuyên phải thực hiện nhiều phép quay phức tạp | Số lần xoay ít hơn (tối đa 2 lần xoay khi chèn, 3 lần khi xóa) |
| **Ứng dụng tối ưu** | Các hệ thống **Đọc nhiều, Ghi ít** | Các cấu trúc dữ liệu tổng quát trong RAM (**Đọc/Ghi hỗn hợp**) như `TreeMap`, `HashMap` |

<!-- @include: @article-footer.snippet.md -->
