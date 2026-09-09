---
title: Tổng hợp câu hỏi phỏng vấn Cây tiền tố Trie: Nguyên lý, Khớp tiền tố và Triển khai Java
description: Tổng hợp câu hỏi phỏng vấn về Cây tiền tố Trie (Dictionary Tree), giải thích cấu trúc node, các thao tác chèn, tìm kiếm từ hoàn chỉnh, khớp tiền tố (startsWith), phân tích độ phức tạp và ứng dụng trong gợi ý tìm kiếm, bộ lọc từ nhạy cảm.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Trie, Cây tiền tố, Prefix Tree, Cây từ điển, Khớp tiền tố, Thuật toán chuỗi, Gợi ý tìm kiếm, Lọc từ nhạy cảm, Java Trie, LeetCode Trie
---

**Trie (Cây tiền tố / Prefix Tree hay Cây từ điển)** là cấu trúc dữ liệu chuyên biệt để xử lý các bài toán khớp tiền tố trên tập hợp chuỗi ký tự khổng lồ. Các tính năng như: Gợi ý tìm kiếm (Search Autocomplete / Search Suggestions), tra cứu từ điển, lọc từ nhạy cảm (Sensitive Word Filtering), định tuyến URL và khớp tiền tố địa chỉ IP đều sử dụng nền tảng của Trie.

Ý tưởng cốt lõi của Trie rất trực quan: **Tách chuỗi thành từng ký tự và chia sẻ chung các tiền tố giống nhau**. Ví dụ: các từ `app`, `apple`, `apply` sẽ dùng chung đoạn đường dẫn `a -> p -> p`.

Nội dung chính:
1. Trie là gì?
2. Tại sao Trie lại vượt trội trong bài toán khớp tiền tố?
3. Thiết kế cấu trúc Node của Trie như thế nào?
4. Cách viết các hàm `insert`, `search` và `startsWith` trong Java?
5. Khi nào nên chọn Trie, khi nào nên chọn Bảng băm (Hash Table)?

![Sơ đồ cấu trúc cây tiền tố Trie](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/trie.png)

## 1. Trie là gì?

Trie là cấu trúc dữ liệu dạng cây chuyên dùng cho tập hợp chuỗi ký tự. Khác với Cây tìm kiếm nhị phân (BST), các node trong Trie không tổ chức theo quan hệ lớn nhỏ mà tổ chức theo **đường đi của các ký tự**:

- **Node Gốc (Root)**: Không đại diện cho ký tự nào, đóng vai trò là cửa ngõ lối vào của tất cả các chuỗi.
- Mỗi bước đi xuống một tầng tương ứng với việc ghép thêm một ký tự tiếp theo trong chuỗi.
- Toàn bộ các ký tự trên đường đi từ Gốc đến một Node bất kỳ tạo thành một **Tiền tố (Prefix)**.
- Một cờ boolean **`isWord` (hoặc `isEnd`)** được đặt tại mỗi Node để đánh dấu xem chuỗi hình thành từ gốc tới node này có phải là một **Từ hoàn chỉnh** hay không.

Ví dụ: Khi chèn `app`, `apple`, `apply`:
- Cả 3 từ dùng chung nhánh `a -> p -> p`.
- Node `p` thứ hai được đánh dấu `isWord = true` (biểu thị `app` là một từ hoàn chỉnh).
- Từ node `p` này, ta tiếp tục rẽ nhánh sang `l -> e` (cho `apple`) và `l -> y` (cho `apply`).

Nếu không có cờ `isWord`, ta sẽ không thể phân biệt được `app` chỉ là một tiền tố trung gian hay bản thân nó cũng là một từ hoàn chỉnh có trong từ điển.

---

## 2. Tại sao Trie lại vượt trội trong bài toán khớp tiền tố?

Bảng băm (Hash Table) rất giỏi kiểm tra xem một từ hoàn chỉnh có tồn tại hay không (ví dụ kiểm tra `apple` có trong bảng không chỉ mất $O(1)$). Nhưng nếu bài toán đổi thành: *"Hãy tìm tất cả các từ bắt đầu bằng tiền tố `app`"*, Bảng băm sẽ gặp bế tắc và buộc phải quét toàn bộ hàng triệu Key.

Trong khi đó, trên cây Trie, một tiền tố tương ứng tự nhiên với **duy nhất một đường đi trên cây**:
- Ta chỉ cần đi theo đường dẫn `a -> p -> p`.
- Nếu đi được tới node `p`, toàn bộ cây con bên dưới node `p` đó chính là tất cả các từ có tiền tố `app`!
- Thời gian tìm kiếm tiền tố chỉ phụ thuộc vào **Độ dài của tiền tố ($L$)**, hoàn toàn độc lập với việc từ điển có 100 từ hay 10 triệu từ.

---

## 3. Thiết kế Node trong Trie

Một Node của Trie thường chứa 2 thông tin:
1. Mảng liên kết (hoặc Map) chứa các tham chiếu tới các node con.
2. Biến cờ `isWord` đánh dấu kết thúc từ.

```java
// Trường hợp chỉ chứa 26 chữ cái tiếng Anh in thường:
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isWord = false;
}
```

| Cách hiện thực Node | Ưu điểm | Nhược điểm | Kịch bản áp dụng |
| :--- | :--- | :--- | :--- |
| **`TrieNode[26]`** | Tốc độ truy cập cực nhanh ($O(1)$) | Lãng phí bộ nhớ nếu nhiều ô trống | Bảng chữ cái cố định kích thước nhỏ (a-z) |
| **`Map<Character, TrieNode>`** | Tiết kiệm RAM, chỉ cấp phát cho ký tự thực tế | Tốn chi phí boxing và truy cập Map | Hỗ trợ Unicode, tiếng Việt, số, URL |

---

## 4. Cài đặt hoàn chỉnh Trie trong Java

```java
class Trie {
    private final TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    // Thao tác 1: Chèn một từ vào Trie
    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                node.children[index] = new TrieNode();
            }
            node = node.children[index];
        }
        node.isWord = true; // Đánh dấu kết thúc từ
    }

    // Thao tác 2: Tìm kiếm xem một từ hoàn chỉnh có tồn tại không
    public boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isWord;
    }

    // Thao tác 3: Kiểm tra xem có từ nào bắt đầu bằng tiền tố prefix không
    public boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    // Hàm phụ trợ duyệt theo chuỗi ký tự
    private TrieNode find(String text) {
        TrieNode node = root;
        for (char c : text.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                return null;
            }
            node = node.children[index];
        }
        return node;
    }
}
```

---

## 5. Phân tích Độ phức tạp

Gọi $L$ là độ dài của chuỗi ký tự:
- **Chèn một từ (`insert`)**: **$O(L)$**.
- **Tìm kiếm từ hoàn chỉnh (`search`)**: **$O(L)$**.
- **Tìm kiếm tiền tố (`startsWith`)**: **$O(L)$**.
- **Không gian bộ nhớ**: Phụ thuộc vào số lượng node. Trường hợp xấu nhất khi các từ không có tiền tố chung, bộ nhớ xấp xỉ tổng số ký tự của tất cả các từ cộng lại.

---

## So sánh: Trie vs Bảng băm (Hash Table)

| Tiêu chí | Trie | Bảng băm (Hash Table) |
| :--- | :--- | :--- |
| **Tìm từ hoàn chỉnh** | Tốc độ $O(L)$, tốn nhiều RAM hơn | Tốc độ trung bình $O(1)$, tiết kiệm RAM hơn |
| **Tìm kiếm theo tiền tố** | **Cực nhanh và tự nhiên ($O(L)$)** | Không hỗ trợ trực tiếp (phải quét toàn bộ) |
| **Liệt kê các từ theo thứ tự từ điển** | Rất dễ dàng qua duyệt cây | Cần sắp xếp lại toàn bộ |
| **Ứng dụng tối ưu** | Gợi ý tìm kiếm, Auto-complete, Lọc từ nhạy cảm | Tra cứu Key-Value tổng quát |

## Đề xuất bài tập luyện tập

- [LeetCode 208. Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/)
- [LeetCode 211. Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/)
- [LeetCode 212. Word Search II](https://leetcode.com/problems/word-search-ii/)
- [LeetCode 648. Replace Words](https://leetcode.com/problems/replace-words/)

<!-- @include: @article-footer.snippet.md -->
