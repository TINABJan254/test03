---
title: Tổng hợp câu hỏi phỏng vấn LRU Cache: Bảng băm, Danh sách liên kết đôi và LinkedHashMap
description: Cẩm nang chuyên sâu về Bộ nhớ đệm LRU (Least Recently Used Cache): Phân tích nguyên lý đào thải bộ nhớ đệm, tự tay lập trình LRU Cache bằng Bảng băm kết hợp Danh sách liên kết đôi, sử dụng Java LinkedHashMap và so sánh với thuật toán LFU.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: LRU Cache, LRU, Least Recently Used, Bộ nhớ đệm, Đào thải Cache, Bảng băm, Danh sách liên kết đôi, LinkedHashMap, LFU, Cache Replacement, LeetCode 146
---

**LRU** là viết tắt của **Least Recently Used (Ít được sử dụng gần đây nhất)**. Đây là thuật toán đào thải bộ nhớ đệm (Cache Eviction Policy) phổ biến và quan trọng bậc nhất trong khoa học máy tính.

Khi dung lượng bộ nhớ đệm bị đầy, thuật toán LRU sẽ **loại bỏ phần tử có thời điểm truy cập xa nhất trong quá khứ** để nhường chỗ cho dữ liệu mới.

Bài toán thiết kế cấu trúc dữ liệu LRU Cache ([LeetCode 146](https://leetcode.com/problems/lru-cache/)) là một trong những câu hỏi phỏng vấn xuất hiện nhiều nhất tại các công ty công nghệ lớn, bởi vì nó kiểm tra hoàn hảo khả năng phối hợp giữa **Bảng băm (Hash Table)** và **Danh sách liên kết đôi (Doubly LinkedList)** để đạt tốc độ thực thi **$O(1)$** cho tất cả các thao tác.

Nội dung chính:
1. LRU Cache là gì?
2. Tại sao cần kết hợp Bảng băm và Danh sách liên kết đôi?
3. Cách tự tay lập trình các phương thức `get(key)` và `put(key, value)` đạt $O(1)$.
4. Triển khai nhanh LRU Cache bằng Java `LinkedHashMap`.
5. So sánh LRU với các chiến lược khác (FIFO, LFU, TTL).

![Cấu trúc LRU Cache kết hợp Bảng băm và Danh sách liên kết đôi](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/lru-cache.png)

## 1. Nguyên lý hoạt động của LRU Cache

Dung lượng bộ nhớ RAM luôn luôn có hạn. Do không thể đoán trước tương lai, thuật toán LRU đưa ra một giả định dựa trên **Tính cục bộ về thời gian (Temporal Locality)**: *Dữ liệu nào vừa mới được truy cập gần đây thì nhiều khả năng sẽ tiếp tục được truy cập trong tương lai gần; ngược lại, dữ liệu lâu rồi không được đụng tới thì khả năng cao sẽ không còn cần thiết nữa.*

Ví dụ với một bộ nhớ đệm LRU có sức chứa `capacity = 2`:
```text
put(1, 1)  // Cache: [1]
put(2, 2)  // Cache: [2, 1] (2 vừa vào nên mới nhất)
get(1)     // Truy cập 1 -> 1 trở thành mới nhất! Cache: [1, 2]
put(3, 3)  // Dung lượng đầy! Đào thải phần tử cũ nhất là 2. Cache: [3, 1]
get(2)     // Trả về -1 (không tìm thấy vì 2 đã bị xóa)
```

> **Lưu ý**: Cả thao tác đọc `get()` và thao tác ghi `put()` đều được tính là một lần "truy cập" làm mới dữ liệu!

---

## 2. Tại sao bắt buộc phải dùng Bảng băm + Danh sách liên kết đôi?

Yêu cầu kỹ thuật: Cả hai thao tác `get(key)` và `put(key, value)` đều bắt buộc phải đạt độ phức tạp thời gian **$O(1)$**.

- **Nếu chỉ dùng Bảng băm (`HashMap`)**: Tra cứu `get(key)` mất $O(1)$, nhưng không thể nào biết được phần tử nào có thời điểm truy cập cũ nhất (không lưu được thứ tự thời gian).
- **Nếu chỉ dùng Danh sách liên kết (`LinkedList`)**: Dễ dàng duy trì thứ tự (đầu danh sách là mới nhất, đuôi danh sách là cũ nhất). Nhưng khi tìm kiếm `get(key)` ta phải duyệt tuần tự từ đầu tới cuối với thời gian $O(n)$!
- **Nếu dùng Danh sách liên kết đơn**: Khi xóa một node ở giữa để đưa lên đầu, ta không thể tìm được node đứng trước nó trong thời gian $O(1)$.

### Sự kết hợp hoàn hảo: Bảng băm + Danh sách liên kết đôi
1. **`HashMap<Integer, Node>`**: Cho phép định vị trực tiếp con trỏ tới Node trong danh sách liên kết chỉ với thời gian $O(1)$.
2. **Danh sách liên kết đôi (Doubly LinkedList)**: Nhờ có cả 2 con trỏ `prev` và `next`, ta có thể dễ dàng tách một node bất kỳ ra khỏi vị trí hiện tại và chèn lên đầu danh sách với thời gian $O(1)$.
3. **Node ảo Đầu (`head`) và Đuôi (`tail`) (Sentinel / Dummy Nodes)**: Giúp loại bỏ hoàn toàn các điều kiện rẽ nhánh kiểm tra `null` phức tạp khi danh sách rỗng hoặc thao tác ở biên.

---

## 3. Tự tay lập trình hoàn chỉnh LRU Cache bằng Java

```java
import java.util.HashMap;
import java.util.Map;

public class LRUCache {
    // Định nghĩa Node của Danh sách liên kết đôi
    private static class Node {
        int key;
        int value;
        Node prev;
        Node next;

        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    private final int capacity;
    private final Map<Integer, Node> map;
    private final Node head; // Node ảo ở đầu (đại diện cho phần tử mới nhất)
    private final Node tail; // Node ảo ở đuôi (đại diện cho phần tử cũ nhất)

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.head = new Node(0, 0);
        this.tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }

    // Lấy giá trị theo Key
    public int get(int key) {
        Node node = map.get(key);
        if (node == null) {
            return -1;
        }
        // Làm mới vị trí: Chuyển node lên đầu danh sách
        moveToHead(node);
        return node.value;
    }

    // Thêm hoặc cập nhật Key-Value
    public void put(int key, int value) {
        Node node = map.get(key);
        if (node != null) {
            node.value = value; // Cập nhật giá trị
            moveToHead(node);   // Chuyển lên đầu
            return;
        }

        // Tạo node mới
        Node newNode = new Node(key, value);
        map.put(key, newNode);
        addToHead(newNode);

        // Nếu vượt quá dung lượng, đào thải phần tử cũ nhất ở đuôi
        if (map.size() > capacity) {
            Node removedNode = removeTail();
            map.remove(removedNode.key); // Xóa khỏi HashMap
        }
    }

    // Tách node khỏi vị trí hiện tại và đưa lên đầu
    private void moveToHead(Node node) {
        removeNode(node);
        addToHead(node);
    }

    // Chèn node vào ngay sau Node head ảo
    private void addToHead(Node node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    // Tách node ra khỏi danh sách liên kết đôi
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    // Xóa node cũ nhất (node đứng ngay trước Node tail ảo)
    private Node removeTail() {
        Node res = tail.prev;
        removeNode(res);
        return res;
    }
}
```

---

## 4. Triển khai nhanh bằng Java `LinkedHashMap`

Trong Java Standard Library, `java.util.LinkedHashMap` đã tích hợp sẵn cơ chế sắp xếp theo thứ tự truy cập (Access Order) và phương thức đào thải `removeEldestEntry`:

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LRUCacheWithLinkedHashMap<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCacheWithLinkedHashMap(int capacity) {
        // accessOrder = true: Sắp xếp theo thứ tự truy cập thay vì thứ tự chèn
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // Tự động xóa phần tử cũ nhất khi số lượng phần tử vượt quá capacity
        return size() > capacity;
    }
}
```

---

## 5. So sánh các chiến lược đào thải bộ nhớ đệm (Cache Eviction)

| Chiến lược | Tiêu chí đào thải | Ưu điểm | Nhược điểm | Kịch bản phù hợp |
| :--- | :--- | :--- | :--- | :--- |
| **FIFO (First In, First Out)** | Dữ liệu vào cache sớm nhất | Cài đặt cực kỳ đơn giản | Bỏ qua mức độ sử dụng thực tế của dữ liệu | Dữ liệu có tính tuần tự một lần |
| **LRU (Least Recently Used)** | Dữ liệu có thời điểm truy cập **xa nhất** | Tận dụng tốt tính cục bộ thời gian | Dễ bị ô nhiễm cache khi có thao tác quét dữ liệu lớn hàng loạt | Bộ nhớ đệm tổng quát (OS Page, Redis, DB Buffer) |
| **LFU (Least Frequently Used)** | Dữ liệu có **tần suất truy cập ít nhất** | Bảo vệ tốt các dữ liệu hot lâu dài | Tốn thêm bộ nhớ lưu bộ đếm, dữ liệu cũ khó bị xóa | Hệ thống có tập dữ liệu hot cố định |

## Đề xuất bài tập luyện tập

- [LeetCode 146. LRU Cache](https://leetcode.com/problems/lru-cache/)
- [LeetCode 460. LFU Cache](https://leetcode.com/problems/lfu-cache/)

<!-- @include: @article-footer.snippet.md -->
