---
title: Một số bài toán thuật toán LinkedList phổ biến
description: Tuyển chọn tư duy và hiện thực các bài toán LinkedList tần suất cao, bao gồm cộng hai số, đảo ngược, phát hiện chu trình, nhấn mạnh xử lý biên và phân tích độ phức tạp.
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Thuật toán LinkedList,Cộng hai số,Đảo ngược LinkedList,Phát hiện chu trình,Hợp nhất LinkedList,Phân tích độ phức tạp
---

<!-- markdownlint-disable MD024 -->

## 1. Cộng hai số (Add Two Numbers)

### Mô tả bài toán

> LeetCode: Cho hai LinkedList không rỗng đại diện cho hai số nguyên không âm. Các chữ số được lưu trữ theo thứ tự đảo ngược (ngược vị số), và mỗi node chỉ lưu trữ một chữ số duy nhất. Hãy cộng hai số này lại và trả về kết quả dưới dạng một LinkedList mới.
>
> Bạn có thể giả định rằng ngoài số 0 ra, cả hai số này đều không bắt đầu bằng chữ số 0.

Ví dụ:

```plain
Đầu vào: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Đầu ra: 7 -> 0 -> 8
Giải thích: 342 + 465 = 807
```

### Phân tích bài toán

Địa chỉ lời giải chi tiết chính thức của LeetCode:

<https://leetcode.cn/problems/add-two-numbers/solution/>

> Khi cần thao tác trên node đầu (head), hãy cân nhắc tạo một node giả (dummy node), sử dụng `dummy.next` để đại diện cho node đầu thực sự. Cách này giúp tránh việc phải xử lý riêng trường hợp biên khi head là null.

Chúng ta sử dụng một biến để theo dõi phần nhớ (carry), và mô phỏng quá trình cộng từng chữ số bắt đầu từ đầu danh sách (chứa chữ số có trọng số thấp nhất).

![Hình 1: Minh họa phương pháp cộng hai số: 342 + 465 = 807, mỗi node chứa một chữ số và lưu trữ theo thứ tự ngược](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/34910956.jpg)

### Lời giải (Solution)

**Chúng ta bắt đầu cộng từ vị trí có trọng số thấp nhất, tức là đầu danh sách l1 và l2. Lưu ý cần tính đến trường hợp có nhớ (carry)!**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
// https://leetcode.cn/problems/add-two-numbers/
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummyHead = new ListNode(0);
        ListNode p = l1, q = l2, curr = dummyHead;
        // carry đại diện cho giá trị nhớ
        int carry = 0;
        while (p != null || q != null) {
            int x = (p != null) ? p.val : 0;
            int y = (q != null) ? q.val : 0;
            int sum = carry + x + y;
            // Tính giá trị nhớ
            carry = sum / 10;
            // Giá trị của node mới là sum % 10
            curr.next = new ListNode(sum % 10);
            curr = curr.next;
            if (p != null) p = p.next;
            if (q != null) q = q.next;
        }
        if (carry > 0) {
            curr.next = new ListNode(carry);
        }
        return dummyHead.next;
    }
}
```

## 2. Đảo ngược LinkedList (Reverse Linked List)

### Mô tả bài toán

> Kiếm Chỉ Offer: Cho một LinkedList, hãy đảo ngược LinkedList đó và xuất ra tất cả các phần tử.

![Đảo ngược LinkedList](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/81431871.jpg)

### Phân tích bài toán

Bài toán thuật toán này, nói một cách dễ hiểu là: Làm thế nào để node đứng sau trỏ ngược lại node đứng trước! Trong đoạn code dưới đây, ta định nghĩa một con trỏ `next`, biến này chủ yếu để tạm lưu node tiếp theo trước khi đảo ngược, ngăn ngừa LinkedList bị "đứt gãy" mất liên kết.

### Lời giải (Solution)

```java
public class ListNode {
  int val;
  ListNode next = null;

  ListNode(int val) {
    this.val = val;
  }
}
```

```java
/**
 * @Description: Đảo ngược LinkedList đơn
 */
public class Solution {

  public ListNode ReverseList(ListNode head) {

    ListNode next = null;
    ListNode pre = null;

    while (head != null) {
      // Lưu lại node kế tiếp chuẩn bị đảo ngược
      next = head.next;
      // Cho node hiện tại trỏ ngược về node phía trước đã đảo ngược (lần đầu sẽ trỏ về null)
      head.next = pre;
      // Cập nhật pre thành node hiện tại
      pre = head;
      // Tiến sang node tiếp theo của danh sách gốc
      head = next;
    }
    return pre;
  }

}
```

Phương thức kiểm thử (main):

```java
  public static void main(String[] args) {

    ListNode a = new ListNode(1);
    ListNode b = new ListNode(2);
    ListNode c = new ListNode(3);
    ListNode d = new ListNode(4);
    ListNode e = new ListNode(5);
    a.next = b;
    b.next = c;
    c.next = d;
    d.next = e;
    new Solution().ReverseList(a);
    while (e != null) {
      System.out.println(e.val);
      e = e.next;
    }
  }
```

Đầu ra:

```plain
5
4
3
2
1
```

## 3. Node thứ k tính từ cuối LinkedList

### Mô tả bài toán

> Kiếm Chỉ Offer: Cho một LinkedList, hãy xuất ra node thứ k tính từ cuối LinkedList lên.

### Phân tích bài toán

> **Node thứ k tính từ cuối LinkedList thực chất chính là node thứ (L - k + 1) tính từ đầu danh sách (với L là độ dài danh sách). Hiểu được điểm này thì bài toán coi như đã giải quyết xong!**

Đầu tiên sử dụng hai node/con trỏ: Cho con trỏ node1 chạy trước k-1 bước; sau đó con trỏ node2 mới bắt đầu chạy cùng. Khi node1 chạy đến node cuối cùng, con trỏ node2 sẽ dừng đúng ở node thứ k tính từ cuối lên, tức là node thứ (L - k + 1) từ đầu.

### Lời giải (Solution)

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/

// Độ phức tạp thời gian O(n), chỉ cần một lần duyệt
public class Solution {
  public ListNode FindKthToTail(ListNode head, int k) {
    // Nếu LinkedList rỗng hoặc k <= 0
    if (head == null || k <= 0) {
      return null;
    }
    // Khai báo hai con trỏ trỏ vào node đầu
    ListNode node1 = head, node2 = head;
    // Ghi nhận số lượng node
    int count = 0;
    // Ghi nhận giá trị k ban đầu để so sánh
    int index = k;
    // Con trỏ node1 chạy trước và đếm số node
    // Khi node1 đã chạy qua k node thì node2 bắt đầu di chuyển
    // Khi node1 đi đến cuối thì node2 chính là node thứ k từ dưới lên
    while (node1 != null) {
      node1 = node1.next;
      count++;
      if (k < 1) {
        node2 = node2.next;
      }
      k--;
    }
    // Nếu tổng số node nhỏ hơn k cần tìm thì trả về null
    if (count < index)
      return null;
    return node2;

  }
}
```

## 4. Xóa node thứ N tính từ cuối LinkedList

> LeetCode: Cho một LinkedList, hãy xóa node thứ n tính từ cuối LinkedList lên và trả về node đầu danh sách (head).

**Ví dụ:**

```plain
Cho LinkedList: 1->2->3->4->5, và n = 2.

Sau khi xóa node thứ 2 tính từ cuối lên, LinkedList trở thành 1->2->3->5.
```

**Mô tả thêm:**

Đảm bảo `n` luôn hợp lệ.

**Nâng cao:**

Bạn có thể thực hiện thao tác này chỉ trong một lần duyệt (One-pass) không?

### Phân tích bài toán

Ta nhận thấy bài toán này có thể đơn giản hóa thành: Xóa node thứ `(L - n + 1)` tính từ đầu LinkedList, trong đó `L` là tổng độ dài danh sách. Chỉ cần tìm được độ dài `L`, bài toán sẽ rất dễ giải quyết.

![Hình 1: Xóa phần tử thứ L - n + 1 trong danh sách](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/94354387.jpg)

### Lời giải (Solution)

**Phương pháp hai lần duyệt (Two-pass):**

Đầu tiên ta thêm một **node giả (dummy node)** làm phụ trợ ở đầu danh sách. Node giả dùng để đơn giản hóa một số trường hợp biên đặc biệt, chẳng hạn khi danh sách chỉ có một node hoặc cần xóa chính node đầu danh sách. Ở lần duyệt thứ nhất, ta tìm độ dài L của danh sách. Sau đó thiết lập con trỏ trỏ vào node giả và di chuyển nó qua `(L - n)` bước. **Ta nối con trỏ next của node thứ (L - n) trực tiếp đến node thứ (L - n + 2), hoàn thành thuật toán.**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
// https://leetcode.cn/problems/remove-nth-node-from-end-of-list/
public class Solution {
  public ListNode removeNthFromEnd(ListNode head, int n) {
    // Node giả giúp đơn giản hóa việc xóa head hoặc danh sách chỉ có 1 node
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    // Đếm độ dài danh sách
    int length = 0;
    ListNode len = head;
    while (len != null) {
      length++;
      len = len.next;
    }
    length = length - n;
    ListNode target = dummy;
    // Tìm node ở vị trí (L - n)
    while (length > 0) {
      target = target.next;
      length--;
    }
    // Nối node (L - n) sang node (L - n + 2)
    target.next = target.next.next;
    return dummy.next;
  }
}
```

**Nâng cao — Phương pháp một lần duyệt (One-pass):**

> Node thứ N tính từ cuối lên chính là node thứ (L - n + 1) tính từ đầu.

Thực ra phương pháp này hoàn toàn đồng nhất với tư duy tìm "node thứ k tính từ cuối LinkedList" ở phần trước. **Ý tưởng cơ bản:** Định nghĩa hai con trỏ `node1`, `node2` cùng trỏ vào `dummy`; `node1` chạy trước, khi `node1` đi được `n + 1` bước thì `node2` mới bắt đầu di chuyển. Khi `node1` chạm tới `null` (hết danh sách), con trỏ `node2` sẽ dừng đúng ở node thứ `(L - n)` tính từ đầu (tức node ngay trước node cần xóa).

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {
  public ListNode removeNthFromEnd(ListNode head, int n) {

    ListNode dummy = new ListNode(0);
    dummy.next = head;
    // Khai báo hai con trỏ cùng trỏ vào dummy
    ListNode node1 = dummy, node2 = dummy;

    // node1 chạy trước n bước, sau đó node2 mới bắt đầu chạy cùng
    // Khi node1 đi đến cuối thì node2 đang ở vị trí ngay trước node cần xóa
    while (node1 != null) {
      node1 = node1.next;
      if (n < 0 && node1 != null) {
        node2 = node2.next;
      }
      n--;
    }

    node2.next = node2.next.next;

    return dummy.next;

  }
}
```

## 5. Hợp nhất hai LinkedList đã sắp xếp (Merge Two Sorted Lists)

### Mô tả bài toán

> Kiếm Chỉ Offer: Cho hai LinkedList đơn tăng dần, hãy hợp nhất hai danh sách này thành một LinkedList mới thỏa mãn thứ tự không giảm.

### Phân tích bài toán

Chúng ta có thể phân tích như sau:

1. Giả sử có hai danh sách A và B;
2. So sánh giá trị head của A (A1) với giá trị head của B (B1), giả sử A1 nhỏ hơn, thì chọn A1 làm head;
3. Tiếp tục so sánh A2 với B1, nếu B1 nhỏ hơn, thì A1 trỏ tới B1;
4. Tiếp tục so sánh A2 với B2...
5. Cứ thế lặp đi lặp lại tuần tự.

Cân nhắc hiện thực bằng đệ quy rất ngắn gọn và trực quan!

### Lời giải (Solution)

**Phiên bản đệ quy:**

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
  public ListNode Merge(ListNode list1, ListNode list2) {
    if (list1 == null) {
      return list2;
    }
    if (list2 == null) {
      return list1;
    }
    if (list1.val <= list2.val) {
      list1.next = Merge(list1.next, list2);
      return list1;
    } else {
      list2.next = Merge(list1, list2.next);
      return list2;
    }
  }
}
```

## Trọng tâm ôn tập phỏng vấn

Code của bài toán LinkedList thường không dài, nhưng thứ tự cập nhật con trỏ rất dễ bị nhầm lẫn. Trước khi phỏng vấn, ít nhất bạn cần nắm vững 4 template kinh điển: Dummy Node (node giả), Đảo ngược LinkedList, Fast-Slow Pointers (con trỏ nhanh chậm), Hợp nhất LinkedList.

| Template | Dạng bài phù hợp | Điểm then chốt |
| ---------- | ---------------------------------- | -------------------------------- |
| Dummy Node (Node giả) | Xóa node, hợp nhất danh sách, head có thể thay đổi | Trả về `dummy.next` |
| Đảo ngược LinkedList | Đảo ngược toàn bộ, đảo ngược theo khoảng, nhóm K phần tử | Lưu `next`, rồi mới cập nhật `cur.next` |
| Fast-Slow Pointers | Phát hiện chu trình, tìm node thứ K từ dưới lên, tìm trung điểm | Luôn kiểm tra `fast != null && fast.next != null` trước |
| Hợp nhất LinkedList | Hợp nhất 2 danh sách, K danh sách có thứ tự | Đệ quy hoặc lặp, lưu ý nối phần dư còn lại vào đuôi |

Template lặp để đảo ngược LinkedList nên ghi nhớ thật chắc:

```java
ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode cur = head;
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}
```

## Minh họa quy trình và các trường hợp biên

Khi đảo ngược LinkedList, mấu chốt là lưu `next` trước rồi mới thay đổi con trỏ `cur.next`. Có thể ghi nhớ nhịp biến đổi con trỏ như sau:

```text
Khởi tạo: prev = null, cur = head

Mỗi vòng lặp:
next = cur.next
cur.next = prev
prev = cur
cur = next

Kết thúc: cur == null, prev trỏ vào head mới
```

Với các bài toán xóa node, hợp nhất danh sách, hãy ưu tiên sử dụng dummy node:

```java
ListNode dummy = new ListNode(0);
dummy.next = head;
// Thao tác thống nhất trên các node sau dummy
return dummy.next;
```

Một số lỗi thường gặp cần lưu ý:

- Khi xóa node thứ N từ cuối lên, dummy node giúp xử lý thống nhất cả trường hợp xóa chính node đầu (head).
- Khi đảo ngược khoảng, cần lưu lại node đứng trước khoảng và node đứng sau khoảng.
- Khi kiểm tra chu trình LinkedList, điều kiện vòng lặp là `fast != null && fast.next != null`.
- Hợp nhất đệ quy code ngắn nhưng nếu danh sách quá dài có nguy cơ gây tràn ngăn xếp đệ quy (Stack Overflow).
- Luôn kiểm tra riêng các trường hợp: Danh sách rỗng, danh sách chỉ có 1 node, xóa node đầu, xóa node cuối.

<!-- @include: @article-footer.snippet.md -->
