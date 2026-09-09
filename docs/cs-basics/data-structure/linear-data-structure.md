---
title: Chi tiết Cấu trúc dữ liệu tuyến tính (Mảng, Danh sách liên kết, Ngăn xếp, Hàng đợi)
description: Tổng hợp toàn diện đặc tính và thao tác của Mảng, Danh sách liên kết, Ngăn xếp, Hàng đợi, phân tích độ phức tạp thuật toán và ứng dụng thực tế trong kỹ thuật phần mềm.
category: Cơ sở máy tính
tag:
  - Cấu trúc dữ liệu
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Mảng, Danh sách liên kết, Ngăn xếp, Hàng đợi, Deque, Phân tích độ phức tạp, Array, LinkedList, Stack, Queue, Truy cập ngẫu nhiên, Chèn xóa
---

# Cấu trúc dữ liệu tuyến tính (Linear Data Structure)

## 1. Mảng (Array)

**Mảng (Array)** là một cấu trúc dữ liệu cơ bản và phổ biến nhất. Nó bao gồm một tập hợp các phần tử (element) có **cùng kiểu dữ liệu**, được lưu trữ trong một **khối bộ nhớ liên tục**.

Nhờ tính liên tục trong bộ nhớ, chúng ta có thể trực tiếp sử dụng chỉ số (index) của phần tử để tính toán chính xác địa chỉ lưu trữ vật lý của nó trong RAM theo công thức:

$$\text{Địa chỉ}(i) = \text{Địa chỉ cơ sở} + i \times \text{Kích thước phần tử}$$

Đặc điểm nổi bật nhất của mảng là: **Cung cấp khả năng truy cập ngẫu nhiên (Random Access) với thời gian $O(1)$** và có dung lượng cố định khi khởi tạo.

```java
Giả sử mảng có độ dài là n:
Truy cập (Access): O(1) // Truy cập phần tử tại vị trí chỉ số xác định
Chèn (Insertion):  O(n) // Trường hợp xấu nhất chèn vào đầu mảng và phải dịch chuyển toàn bộ n phần tử phía sau
Xóa (Deletion):    O(n) // Trường hợp xấu nhất xóa phần tử đầu tiên và phải dịch chuyển toàn bộ các phần tử còn lại
```

![Cấu trúc Mảng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/array.png)

## 2. Danh sách liên kết (LinkedList)

### 2.1. Giới thiệu Danh sách liên kết

**Danh sách liên kết (LinkedList)** mặc dù là một danh sách tuyến tính, nhưng nó **không lưu trữ các phần tử theo thứ tự liên tục trong bộ nhớ vật lý**.

Mỗi phần tử trong danh sách liên kết được gọi là một **Node (Nút)**. Mỗi Node bao gồm 2 phần: **Dữ liệu (Data)** và **Con trỏ liên kết (Pointer / Reference)** trỏ đến địa chỉ của Node kế tiếp (hoặc Node liền trước).

Thao tác chèn và xóa trong danh sách liên kết có độ phức tạp là $O(1)$ khi đã biết trước vị trí của Node cần thao tác (chỉ cần thay đổi liên kết con trỏ). Tuy nhiên, khi tìm kiếm một giá trị hoặc truy cập phần tử tại một vị trí bất kỳ, độ phức tạp là $O(n)$ do phải duyệt tuần tự từ đầu danh sách.

Sử dụng danh sách liên kết giúp khắc phục nhược điểm cần khai báo trước kích thước cố định của mảng, tận dụng linh hoạt các vùng nhớ phân mảnh trong RAM và hỗ trợ mở rộng kích thước động. Tuy nhiên, danh sách liên kết sẽ tốn thêm bộ nhớ để lưu trữ các con trỏ, và không hỗ trợ truy cập ngẫu nhiên nhanh chóng như mảng.

### 2.2. Phân loại Danh sách liên kết

**Các loại danh sách liên kết phổ biến:**

1. Danh sách liên kết đơn (Singly LinkedList)
2. Danh sách liên kết đôi (Doubly LinkedList)
3. Danh sách liên kết vòng đơn (Circular LinkedList)
4. Danh sách liên kết vòng đôi (Doubly Circular LinkedList)

```java
Giả sử danh sách liên kết có n phần tử:
Truy cập (Access): O(n) // Phải duyệt tuần tự để đến vị trí cần tìm
Chèn / Xóa:        O(1) // Khi đã biết con trỏ tại vị trí cần thao tác
```

#### 2.2.1. Danh sách liên kết đơn (Singly LinkedList)

**Danh sách liên kết đơn** chỉ có một chiều duy nhất. Mỗi Node chỉ chứa một con trỏ `next` trỏ tới Node tiếp theo phía sau. Vì vậy, các Node trong danh sách phân tán rải rác trong bộ nhớ. Node đầu tiên được gọi là `head` (Nút đầu), qua đó ta có thể duyệt toàn bộ danh sách. Con trỏ `next` của Node cuối cùng (`tail`) trỏ tới `null`.

![Danh sách liên kết đơn](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/single-linkedlist.png)

#### 2.2.2. Danh sách liên kết vòng (Circular LinkedList)

**Danh sách liên kết vòng** thực chất là một biến thể của danh sách liên kết đơn. Điểm khác biệt duy nhất là con trỏ `next` của Node cuối cùng không trỏ tới `null` mà quay ngược lại trỏ vào Node `head` tạo thành một vòng tròn khép kín.

![Danh sách liên kết vòng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/circular-linkedlist.png)

#### 2.2.3. Danh sách liên kết đôi (Doubly LinkedList)

**Danh sách liên kết đôi** mỗi Node chứa hai con trỏ: con trỏ `prev` trỏ tới Node liền trước và con trỏ `next` trỏ tới Node liền sau. Nhờ vậy, ta có thể duyệt danh sách theo cả hai chiều tiến và lùi một cách thuận tiện.

![Danh sách liên kết đôi](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-linkedlist.png)

#### 2.2.4. Danh sách liên kết vòng đôi (Doubly Circular LinkedList)

**Danh sách liên kết vòng đôi** kết hợp đặc tính của cả hai loại trên: con trỏ `next` của Node cuối cùng trỏ về `head`, và con trỏ `prev` của `head` trỏ tới Node cuối cùng.

![Danh sách liên kết vòng đôi](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-circular-linkedlist.png)

### 2.3. Kịch bản áp dụng

- Nếu bài toán đòi hỏi **truy cập ngẫu nhiên thường xuyên theo chỉ số**, mảng là lựa chọn bắt buộc; danh sách liên kết không đáp ứng được.
- Nếu số lượng phần tử **không xác định trước** và liên tục có nhu cầu **thêm, bớt phần tử**, danh sách liên kết sẽ phù hợp hơn.
- Nếu số lượng phần tử đã cố định hoặc ít khi thay đổi, sử dụng mảng sẽ tiết kiệm bộ nhớ và đạt hiệu năng CPU Cache cao hơn.

### 2.4. So sánh: Mảng (Array) vs Danh sách liên kết (LinkedList)

- Mảng hỗ trợ truy cập ngẫu nhiên $O(1)$, danh sách liên kết chỉ hỗ trợ truy cập tuần tự $O(n)$.
- Mảng sử dụng khối bộ nhớ liên tục nên rất **thân thiện với cơ chế CPU Cache L1/L2 (Spatial Locality)**; danh sách liên kết lưu trữ phân mảnh nên tỷ lệ Cache Miss cao hơn.
- Kích thước của mảng cố định; khi mảng động (`ArrayList`) bị đầy, hệ thống phải cấp phát một vùng nhớ mới lớn hơn và sao chép toàn bộ phần tử cũ sang, thao tác này tiêu tốn thời gian. Danh sách liên kết mở rộng tự nhiên từng Node một mà không cần di chuyển dữ liệu cũ.

---

## 3. Ngăn xếp (Stack)

### 3.1. Giới thiệu Ngăn xếp

**Ngăn xếp (Stack)** là một tập hợp dữ liệu tuyến tính chỉ cho phép thêm phần tử (`push`) và lấy phần tử (`pop`) tại một đầu duy nhất, gọi là **Đỉnh ngăn xếp (Top)**. Do đó, ngăn xếp hoạt động theo nguyên lý **Vào sau ra trước (LIFO - Last In, First Out)**.

Ngăn xếp có thể được hiện thực bằng mảng một chiều (**Sequential Stack - Ngăn xếp tuần tự**) hoặc bằng danh sách liên kết (**Linked Stack - Ngăn xếp liên kết**).

```java
Giả sử ngăn xếp có n phần tử:
Truy cập (Access): O(n) // Trường hợp xấu nhất để tìm một phần tử ở đáy
Chèn / Xóa:        O(1) // Thao tác push/pop tại đỉnh ngăn xếp luôn là hằng số thời gian
```

![Cấu trúc Vào sau ra trước của Ngăn xếp](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/stack.png)

### 3.2. Kịch bản ứng dụng phổ biến của Ngăn xếp

Bất cứ khi nào dữ liệu cần xử lý theo quy tắc phần tử nào đến sau cùng sẽ được xử lý đầu tiên (**LIFO**), ta đều có thể áp dụng cấu trúc Ngăn xếp.

#### 3.2.1. Tính năng Back / Forward trên trình duyệt Web

Chúng ta chỉ cần sử dụng **2 ngăn xếp (Stack 1 và Stack 2)** là có thể cài đặt hoàn hảo tính năng này:
1. Khi bạn lần lượt truy cập các trang `1 -> 2 -> 3 -> 4`, các trang được lần lượt `push` vào Stack 1.
2. Khi bạn bấm nút **Back** để quay lại trang `2`: Ta lần lượt `pop` trang `4` và `3` khỏi Stack 1 rồi `push` sang Stack 2.
3. Nếu bạn bấm nút **Forward** để tiến lên trang `3`: Ta `pop` trang `3` khỏi Stack 2 và `push` ngược lại vào Stack 1.

![Sử dụng 2 ngăn xếp cài đặt tính năng Back và Forward](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/stack-browser-back-forward.png)

#### 3.2.2. Kiểm tra dấu ngoặc hợp lệ (Valid Parentheses)

> Cho một chuỗi chỉ chứa các ký tự `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, hãy kiểm tra chuỗi đó có hợp lệ hay không.
>
> Chuỗi hợp lệ phải thỏa mãn:
> 1. Dấu ngoặc mở phải được đóng bằng dấu ngoặc đóng cùng loại.
> 2. Các dấu ngoặc mở phải được đóng theo đúng thứ tự.
>
> Ví dụ: `"()"`, `"()[]{}"`, `"{[]}"` là hợp lệ; còn `"(]"`, `"([)]"` là không hợp lệ.

Chúng ta sử dụng `Stack` để giải quyết bài toán kinh điển này:
1. Tạo một bảng băm `Map` lưu trữ quy tắc tương ứng giữa ngoặc đóng và ngoặc mở;
2. Duyệt từng ký tự trong chuỗi: nếu là dấu ngoặc mở thì `push` vào stack; nếu là dấu ngoặc đóng, ta kiểm tra phần tử đỉnh stack có khớp với dấu ngoặc mở tương ứng không, nếu không khớp hoặc stack rỗng thì trả về `false`. Khi duyệt hết chuỗi, nếu stack rỗng thì trả về `true`.

```java
public boolean isValid(String s) {
    // Quy tắc khớp giữa ngoặc đóng và ngoặc mở
    HashMap<Character, Character> mappings = new HashMap<Character, Character>();
    mappings.put(')', '(');
    mappings.put('}', '{');
    mappings.put(']', '[');
    
    Stack<Character> stack = new Stack<Character>();
    char[] chars = s.toCharArray();
    for (int i = 0; i < chars.length; i++) {
        if (mappings.containsKey(chars[i])) {
            char topElement = stack.empty() ? '#' : stack.pop();
            if (topElement != mappings.get(chars[i])) {
                return false;
            }
        } else {
            stack.push(chars[i]);
        }
    }
    return stack.isEmpty();
}
```

#### 3.2.3. Đảo ngược chuỗi (String Reversal)
Đẩy lần lượt từng ký tự của chuỗi vào ngăn xếp rồi `pop` toàn bộ ra ngoài.

#### 3.2.4. Quản lý lời gọi hàm (Call Stack)
Hàm nào được gọi sau cùng sẽ phải hoàn thành trước và trả về kết quả, hoàn toàn khớp với nguyên lý **LIFO**. Ví dụ: Lời gọi hàm đệ quy (Recursion) được quản lý trong bộ nhớ JVM bằng Call Stack, mỗi lần gọi hàm mới sẽ đẩy tham số và địa chỉ trả về (Return Address) vào Stack frame.

#### 3.2.5. Tìm kiếm theo chiều sâu (DFS - Depth First Search)
Trong thuật toán DFS trên đồ thị hoặc cây, ngăn xếp được dùng để lưu trữ đường đi tìm kiếm nhằm phục vụ việc quay lui (Backtracking).

### 3.3. Cài đặt Ngăn xếp bằng Mảng

Dưới đây là mã nguồn cài đặt một Ngăn xếp động hỗ trợ các phương thức cơ bản: `push()`, `pop()`, `peek()`, `isEmpty()`, `size()`.

```java
public class MyStack {
    private int[] storage; // Mảng lưu trữ các phần tử
    private int capacity;  // Sức chứa hiện tại
    private int count;     // Số lượng phần tử hiện có
    private static final int GROW_FACTOR = 2; // Hệ số mở rộng

    // Constructor mặc định với sức chứa ban đầu là 8
    public MyStack() {
        this.capacity = 8;
        this.storage = new int[8];
        this.count = 0;
    }

    // Constructor với sức chứa tùy chỉnh
    public MyStack(int initialCapacity) {
        if (initialCapacity < 1)
            throw new IllegalArgumentException("Capacity too small.");

        this.capacity = initialCapacity;
        this.storage = new int[initialCapacity];
        this.count = 0;
    }

    // Đẩy phần tử vào đỉnh ngăn xếp
    public void push(int value) {
        if (count == capacity) {
            ensureCapacity();
        }
        storage[count++] = value;
    }

    // Tự động mở rộng dung lượng khi đầy
    private void ensureCapacity() {
        int newCapacity = capacity * GROW_FACTOR;
        storage = Arrays.copyOf(storage, newCapacity);
        capacity = newCapacity;
    }

    // Lấy phần tử đỉnh ngăn xếp ra ngoài và xóa khỏi stack
    public int pop() {
        if (count == 0)
            throw new IllegalArgumentException("Stack is empty.");
        count--;
        return storage[count];
    }

    // Xem giá trị phần tử đỉnh ngăn xếp mà không xóa
    public int peek() {
        if (count == 0) {
            throw new IllegalArgumentException("Stack is empty.");
        } else {
            return storage[count - 1];
        }
    }

    // Kiểm tra ngăn xếp có rỗng không
    public boolean isEmpty() {
        return count == 0;
    }

    // Trả về số lượng phần tử trong ngăn xếp
    public int size() {
        return count;
    }
}
```

---

## 4. Hàng đợi (Queue)

### 4.1. Giới thiệu Hàng đợi

**Hàng đợi (Queue)** là cấu trúc dữ liệu tuyến tính hoạt động theo nguyên lý **Vào trước ra trước (FIFO - First In, First Out)**.

Trong ứng dụng thực tế, hàng đợi có thể cài đặt bằng mảng (**Sequential Queue - Hàng đợi tuần tự**) hoặc bằng danh sách liên kết (**Linked Queue - Hàng đợi liên kết**).

Quy tắc của Hàng đợi:
- Thao tác thêm phần tử (**Enqueue / Đưa vào hàng đợi**) chỉ được thực hiện ở **Đuôi hàng đợi (Rear / Tail)**.
- Thao tác lấy phần tử (**Dequeue / Lấy khỏi hàng đợi**) chỉ được thực hiện ở **Đầu hàng đợi (Front / Head)**.

```java
Giả sử hàng đợi có n phần tử:
Truy cập (Access): O(n) // Trường hợp xấu nhất khi tìm kiếm phần tử ở giữa
Chèn / Xóa:        O(1) // Enqueue ở đuôi và Dequeue ở đầu luôn đạt O(1)
```

![Cấu trúc Hàng đợi FIFO](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/queue.png)

### 4.2. Phân loại Hàng đợi

#### 4.2.1. Hàng đợi đơn (Single Queue)

Hàng đợi thông thường thêm phần tử vào đuôi và lấy ra ở đầu.

**Nhược điểm của Hàng đợi tuần tự cài bằng mảng là hiện tượng "Tràn ảo" (False Overflow):**
Khi thực hiện enqueue và dequeue liên tục, cả 2 con trỏ `front` và `rear` đều dịch chuyển về phía cuối mảng. Khi `rear` chạm đến giới hạn cuối mảng, ta không thể thêm phần tử mới mặc dù các ô nhớ ở đầu mảng (do dequeue tạo ra) vẫn còn trống!

![Hiện tượng Tràn ảo trong Hàng đợi tuần tự](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/seq-queue-false-overflow.png)

#### 4.2.2. Hàng đợi vòng (Circular Queue)

**Hàng đợi vòng** giải quyết triệt để hiện tượng tràn ảo bằng cách nối liền vị trí cuối mảng quay vòng về vị trí đầu tiên (chỉ số 0).

![Hàng đợi vòng](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/circular-queue.png)

Trong hàng đợi vòng, để phân biệt giữa trạng thái **Hàng đợi Rỗng** và **Hàng đợi Đầy**, có 2 giải pháp phổ biến:
1. **Dùng biến cờ (`flag`) hoặc biến đếm `count`**: Khi `front == rear` và `count == 0` là rỗng; khi `front == rear` và `count == capacity` là đầy.
2. **Quy ước để trống 1 vị trí trong mảng**:
   - Điều kiện rỗng: `front == rear`
   - Điều kiện đầy: `(rear + 1) % capacity == front`

#### 4.2.3. Hàng đợi hai đầu (Deque - Double Ended Queue)

**Deque (Double Ended Queue)** cho phép thực hiện chèn và xóa phần tử linh hoạt ở cả hai đầu (Head và Tail). Deque cung cấp các hàm: `addFirst()`, `addLast()`, `removeFirst()`, `removeLast()`.

Trong Java, `Deque` có thể hoạt động vừa như một Hàng đợi FIFO, vừa như một Ngăn xếp LIFO hoàn hảo.

#### 4.2.4. Hàng đợi ưu tiên (Priority Queue)

**Hàng đợi ưu tiên (Priority Queue)** về bản chất không phải cấu trúc tuyến tính đơn thuần mà thường được hiện thực bên dưới bằng cấu trúc **Heap (Đống)**.
- Khi thêm phần tử, phần tử mới được đưa vào Heap và tiến hành điều chỉnh (Sift-Up) với độ phức tạp $O(\log n)$.
- Khi lấy phần tử, phần tử có mức ưu tiên cao nhất (hoặc nhỏ nhất) tại đỉnh Heap được lấy ra và Heap tự cân bằng lại (Sift-Down) với độ phức tạp $O(\log n)$.

---

### 4.3. Kịch bản ứng dụng của Hàng đợi trong thực tế

- **Hàng đợi chặn (BlockingQueue) & Mô hình Producer-Consumer**: Khi hàng đợi rỗng, luồng Consumer sẽ bị chặn (block) chờ dữ liệu; khi hàng đợi đầy, luồng Producer bị chặn chờ chỗ trống.
- **Hàng đợi tác vụ trong ThreadPool (Task Queue)**: Khi toàn bộ luồng trong ThreadPool đang bận, các task mới được đưa vào `BlockingQueue` (ví dụ `LinkedBlockingQueue`, `ArrayBlockingQueue`) để chờ xử lý.
- **Thay thế Stack trong Java**: JDK khuyến nghị sử dụng `Deque` (ví dụ `ArrayDeque`) thay thế cho lớp cổ điển `java.util.Stack` vì `Stack` kế thừa từ `Vector` và bị khóa đồng bộ hóa chậm chạp.
- **Tìm kiếm theo chiều rộng (BFS - Breadth First Search)**: Sử dụng hàng đợi để duyệt qua các đỉnh của đồ thị/cây theo từng lớp lang (Level-order).
- **Message Queue (RabbitMQ, Kafka, RocketMQ)**: Xử lý bất đồng bộ, gọt đỉnh tải (Peak Shaving) và tách rời hệ thống (Decoupling).

---

## Bảng so sánh tổng kết các cấu trúc tuyến tính

| Cấu trúc | Truy vấn | Chèn / Xóa | Kiểu dữ liệu Java tiêu biểu | Dạng bài toán phỏng vấn thường gặp |
| :--- | :--- | :--- | :--- | :--- |
| **Mảng (Array)** | Theo chỉ số $O(1)$ | Vị trí giữa $O(n)$ | Mảng cơ sở, `ArrayList` | Tìm kiếm nhị phân, Hai con trỏ, Mảng tiền tố |
| **Danh sách liên kết (LinkedList)** | $O(n)$ | Khi biết node $O(1)$ | `LinkedList` | Đảo ngược linkedlist, Con trỏ nhanh chậm, Hợp nhất |
| **Ngăn xếp (Stack)** | Đỉnh $O(1)$ | Đỉnh $O(1)$ | `ArrayDeque`, `Deque` | Khớp dấu ngoặc, Monotonic Stack, DFS |
| **Hàng đợi (Queue)** | Đầu $O(1)$ | Đầu/Đuôi $O(1)$ | `ArrayDeque`, `BlockingQueue` | BFS, Producer-Consumer, Hàng đợi xử lý tác vụ |

## Đề xuất bài tập luyện tập

- **Mảng**: [LeetCode 704. Binary Search](https://leetcode.com/problems/binary-search/), [LeetCode 26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)
- **Danh sách liên kết**: [LeetCode 206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/), [LeetCode 19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)
- **Ngăn xếp**: [LeetCode 20. Valid Parentheses](https://leetcode.com/problems/valid-parentheses/), [LeetCode 739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)
- **Hàng đợi**: [LeetCode 102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/), [LeetCode 239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

<!-- @include: @article-footer.snippet.md -->
