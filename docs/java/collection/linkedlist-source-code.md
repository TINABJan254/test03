---
title: LinkedList 源码分析
description: LinkedList源码深度解析：剖析双向链表结构、Deque接口实现、头尾插入删除O(1)时间复杂度、与ArrayList性能对比及适用场景。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: LinkedList源码,双向链表,Deque接口,LinkedList与ArrayList区别,插入删除性能,链表实现
---

<!-- @include: @article-header.snippet.md -->

## Giới thiệu về LinkedList

`LinkedList` là một collection class được triển khai dựa trên danh sách liên kết đôi (doubly linked list), thường được mang ra so sánh với `ArrayList`. Chi tiết so sánh giữa `LinkedList` và `ArrayList`, bài viết [Tổng kết câu hỏi phỏng vấn Java Collection thường gặp (Phần 1)](./java-collection-questions-01.md) của chúng tôi đã giới thiệu rất chi tiết.

![Danh sách liên kết đôi](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-linkedlist.png)

Tuy nhiên, trong dự án chúng ta thường không sử dụng `LinkedList`. Các kịch bản cần dùng `LinkedList` hầu như đều có thể thay thế bằng `ArrayList`, và hiệu năng thường sẽ tốt hơn! Ngay cả tác giả của `LinkedList` là Josh Bloch cũng từng nói rằng bản thân chưa bao giờ sử dụng `LinkedList`.

![](https://oss.javaguide.cn/github/javaguide/redisimage-20220412110853807.png)

Ngoài ra, đừng nghĩ theo bản năng rằng `LinkedList` là danh sách liên kết thì sẽ thích hợp nhất cho các kịch bản thêm/xóa phần tử. Như tôi đã nói ở trên, `LinkedList` chỉ có độ phức tạp thời gian xấp xỉ O(1) khi chèn hoặc xóa phần tử ở đầu/cuối, các trường hợp thêm/xóa phần tử khác độ phức tạp thời gian trung bình đều là O(n).

### Độ phức tạp thời gian khi chèn và xóa phần tử trong LinkedList?

- Chèn/Xóa ở đầu: Chỉ cần sửa đổi con trỏ của node đầu là có thể hoàn thành thao tác chèn/xóa, do đó độ phức tạp thời gian là O(1).
- Chèn/Xóa ở cuối: Chỉ cần sửa đổi con trỏ của node cuối là có thể hoàn thành thao tác chèn/xóa, do đó độ phức tạp thời gian là O(1).
- Chèn/Xóa tại vị trí chỉ định: Cần di chuyển đến vị trí chỉ định trước, sau đó mới sửa con trỏ của node chỉ định để hoàn thành việc chèn/xóa. Tuy nhiên do có con trỏ đầu và cuối, có thể xuất phát từ con trỏ gần hơn, nên trung bình cần duyệt n/4 phần tử, độ phức tạp thời gian là O(n).

### Tại sao LinkedList không thể triển khai interface RandomAccess?

`RandomAccess` là một marker interface (interface đánh dấu), được dùng để biểu thị class triển khai interface này hỗ trợ truy cập ngẫu nhiên (tức là có thể truy cập phần tử nhanh chóng qua index). Do cấu trúc dữ liệu bên dưới của `LinkedList` là danh sách liên kết, địa chỉ bộ nhớ không liên tục, chỉ có thể định vị thông qua con trỏ, không hỗ trợ truy cập ngẫu nhiên nhanh chóng, do đó không thể triển khai interface `RandomAccess`.

## Phân tích source code của LinkedList

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code cốt lõi bên dưới của `LinkedList`.

Khai báo class `LinkedList` như sau:

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
  //...
}
```

`LinkedList` kế thừa `AbstractSequentialList`, còn `AbstractSequentialList` lại kế thừa từ `AbstractList`.

Đã đọc qua source code của `ArrayList` thì chúng ta biết, `ArrayList` cũng kế thừa `AbstractList`, do đó `LinkedList` sẽ có đa số các phương thức tương tự như `ArrayList`.

`LinkedList` triển khai các interface sau:

- `List`: Biểu thị nó là một danh sách, hỗ trợ các thao tác như thêm, xóa, tìm kiếm,... và có thể truy cập thông qua index.
- `Deque`: Kế thừa từ interface `Queue`, sở hữu đặc tính của hàng đợi hai đầu (double-ended queue), hỗ trợ chèn và xóa phần tử ở cả hai đầu, thuận tiện triển khai các cấu trúc dữ liệu như stack và queue. Cần chú ý, `Deque` phát âm là "deck" [dɛk], từ này hầu hết mọi người đều đọc sai.
- `Cloneable`: Biểu thị nó có khả năng copy, có thể thực hiện thao tác deep copy hoặc shallow copy.
- `Serializable`: Biểu thị nó có thể thực hiện thao tác serialization, tức là có thể chuyển đổi đối tượng thành byte stream để lưu trữ lâu dài hoặc truyền qua mạng, rất tiện lợi.

![Sơ đồ class LinkedList](https://oss.javaguide.cn/github/javaguide/java/collection/linkedlist--class-diagram.png)

Các phần tử trong `LinkedList` được định nghĩa thông qua `Node`:

```java
private static class Node<E> {
    E item;// Giá trị node
    Node<E> next; // Node tiếp theo trỏ tới (node kế tiếp)
    Node<E> prev; // Node phía trước trỏ tới (node phía trước)

    // Thứ tự tham số khởi tạo lần lượt là: Node phía trước, giá trị node bản thân, node kế tiếp
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### Khởi tạo

Trong `LinkedList` có một constructor không tham số và một constructor có tham số.

```java
// Tạo một đối tượng danh sách liên kết rỗng
public LinkedList() {
}

// Nhận một dạng collection làm tham số, sẽ tạo một đối tượng danh sách liên kết chứa các phần tử giống collection truyền vào
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### Chèn phần tử

`LinkedList` ngoài việc triển khai các phương thức liên quan của interface `List`, còn triển khai rất nhiều phương thức của interface `Deque`, do đó chúng ta có rất nhiều cách để chèn phần tử.

Ở đây chúng ta lấy phương thức chèn liên quan trong interface `List` làm ví dụ giảng giải source code, tương ứng là phương thức `add()`.

Phương thức `add()` có 2 phiên bản:

- `add(E e)`: Dùng để chèn phần tử vào cuối `LinkedList`, tức là đưa phần tử mới làm phần tử cuối cùng của danh sách liên kết, độ phức tạp thời gian là O(1).
- `add(int index, E element)`: Dùng để chèn phần tử vào vị trí chỉ định. Cách chèn này cần phải di chuyển đến vị trí chỉ định trước, sau đó sửa con trỏ của node chỉ định để hoàn thành việc chèn/xóa, do đó trung bình cần di chuyển n/4 phần tử, độ phức tạp thời gian là O(n).

```java
// Chèn phần tử vào cuối danh sách liên kết
public boolean add(E e) {
    linkLast(e);
    return true;
}

// Chèn phần tử vào vị trí chỉ định trong danh sách liên kết
public void add(int index, E element) {
    // Kiểm tra vượt quá chỉ số (index out of bounds)
    checkPositionIndex(index);

    // Kiểm tra index có phải vị trí cuối danh sách liên kết hay không
    if (index == size)
        // Nếu phải thì gọi trực tiếp phương thức linkLast để chèn node phần tử vào cuối danh sách liên kết
        linkLast(element);
    else
        // Nếu không phải thì gọi phương thức linkBefore để chèn nó vào trước phần tử chỉ định
        linkBefore(element, node(index));
}

// Chèn node phần tử vào cuối danh sách liên kết
void linkLast(E e) {
    // Gán phần tử cuối cùng (truyền tham chiếu) cho node l
    final Node<E> l = last;
    // Tạo node mới, và chỉ định node phía trước là node cuối last của danh sách liên kết, tham chiếu kế tiếp là null
    final Node<E> newNode = new Node<>(l, e, null);
    // Cho tham chiếu last trỏ đến node mới
    last = newNode;
    // Kiểm tra node cuối có null hay không
    // Nếu l là null nghĩa là đây là lần đầu tiên thêm phần tử
    if (l == null)
        // Nếu là lần đầu tiên thêm, gán first thành node mới, lúc này danh sách liên kết chỉ có 1 phần tử
        first = newNode;
    else
        // Nếu không phải lần đầu tiên thêm, gán node mới cho next của l (phần tử cuối cùng trước khi thêm)
        l.next = newNode;
    size++;
    modCount++;
}

// Chèn phần tử vào trước phần tử chỉ định
void linkBefore(E e, Node<E> succ) {
    // assert succ != null; Assert succ không được null
    // Định nghĩa một phần tử node để lưu tham chiếu prev của succ, tức là thông tin node phía trước nó
    final Node<E> pred = succ.prev;
    // Khởi tạo node, chỉ rõ node phía trước và node kế tiếp
    final Node<E> newNode = new Node<>(pred, e, succ);
    // Cho tham chiếu prev (node phía trước) của node succ trỏ đến node mới
    succ.prev = newNode;
    // Kiểm tra node phía trước có null hay không, null biểu thị succ là node đầu tiên
    if (pred == null)
        // Node mới trở thành node đầu tiên
        first = newNode;
    else
        // Tham chiếu next của node phía trước succ trỏ đến node mới
        pred.next = newNode;
    size++;
    modCount++;
}
```

### Lấy phần tử

Các phương thức liên quan đến việc lấy phần tử trong `LinkedList` gồm 3 phương thức:

1. `getFirst()`: Lấy phần tử đầu tiên của danh sách liên kết.
2. `getLast()`: Lấy phần tử cuối cùng của danh sách liên kết.
3. `get(int index)`: Lấy phần tử tại vị trí chỉ định của danh sách liên kết.

```java
// Lấy phần tử đầu tiên của danh sách liên kết
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

// Lấy phần tử cuối cùng của danh sách liên kết
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

// Lấy phần tử tại vị trí chỉ định trong danh sách liên kết
public E get(int index) {
  // Kiểm tra vượt quá index, nếu vượt quá thì ném exception
  checkElementIndex(index);
  // Trả về phần tử tương ứng với index trong danh sách liên kết
  return node(index).item;
}
```

Trọng tâm ở đây nằm ở phương thức `node(int index)`:

```java
// Trả về node non-null tại index chỉ định
Node<E> node(int index) {
    // Assert index không vượt quá giới hạn
    // assert isElementIndex(index);
    // Nếu index nhỏ hơn một nửa size, tìm kiếm từ đầu (tìm kiếm về sau), ngược lại tìm kiếm ngược về trước
    if (index < (size >> 1)) {
        Node<E> x = first;
        // Duyệt, lặp tìm kiếm về sau cho đến khi i == index
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

Các phương thức như `get(int index)` hoặc `remove(int index)` bên trong đều gọi phương thức này để lấy node tương ứng.

Từ source code phương thức này có thể thấy, phương thức xác định nên duyệt từ đầu hay từ cuối danh sách liên kết bằng cách so sánh index với một nửa size của danh sách liên kết. Nếu index nhỏ hơn một nửa size thì duyệt từ đầu danh sách, ngược lại duyệt từ cuối danh sách. Điều này giúp tìm thấy node mục tiêu trong thời gian ngắn hơn, tận dụng tối đa đặc tính của danh sách liên kết đôi để nâng cao hiệu năng.

### Xóa phần tử

Các phương thức liên quan đến xóa phần tử trong `LinkedList` gồm 5 phương thức:

1. `removeFirst()`: Xóa và trả về phần tử đầu tiên của danh sách liên kết.
2. `removeLast()`: Xóa và trả về phần tử cuối cùng của danh sách liên kết.
3. `remove(E e)`: Xóa phần tử chỉ định xuất hiện lần đầu tiên trong danh sách liên kết, nếu không tồn tại phần tử đó thì trả về false.
4. `remove(int index)`: Xóa phần tử tại index chỉ định, và trả về giá trị của phần tử đó.
5. `void clear()`: Gỡ bỏ tất cả phần tử trong danh sách liên kết này.

```java
// Xóa và trả về phần tử đầu tiên của danh sách liên kết
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

// Xóa và trả về phần tử cuối cùng của danh sách liên kết
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

// Xóa phần tử chỉ định xuất hiện lần đầu tiên trong danh sách liên kết, nếu không tồn tại phần tử đó thì trả về false
public boolean remove(Object o) {
    // Nếu phần tử chỉ định là null, duyệt danh sách liên kết tìm phần tử null đầu tiên để xóa
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // Nếu không phải null, duyệt danh sách liên kết tìm node cần xóa
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

// Xóa phần tử tại vị trí chỉ định trong danh sách liên kết
public E remove(int index) {
    // Kiểm tra vượt quá index, nếu vượt quá thì ném exception
    checkElementIndex(index);
    return unlink(node(index));
}
```

Trọng tâm ở đây nằm ở phương thức `unlink(Node<E> x)`:

```java
E unlink(Node<E> x) {
    // Assert x không được null
    // assert x != null;
    // Lấy phần tử của node hiện tại (tức là node chờ xóa)
    final E element = x.item;
    // Lấy node kế tiếp của node hiện tại
    final Node<E> next = x.next;
    // Lấy node phía trước của node hiện tại
    final Node<E> prev = x.prev;

    // Nếu node phía trước null, chứng tỏ node hiện tại là node đầu (head)
    if (prev == null) {
        // Cho head danh sách liên kết trỏ trực tiếp đến node kế tiếp của node hiện tại
        first = next;
    } else { // Nếu node phía trước không null
        // Cho con trỏ next của node phía trước trỏ đến node kế tiếp của node hiện tại
        prev.next = next;
        // Gán con trỏ prev của node hiện tại thành null, thuận tiện cho GC thu gom
        x.prev = null;
    }

    // Nếu node kế tiếp null, chứng tỏ node hiện tại là node cuối (tail)
    if (next == null) {
        // Cho tail danh sách liên kết trỏ trực tiếp đến node phía trước của node hiện tại
        last = prev;
    } else { // Nếu node kế tiếp không null
        // Cho con trỏ prev của node kế tiếp trỏ đến node phía trước của node hiện tại
        next.prev = prev;
        // Gán con trỏ next của node hiện tại thành null, thuận tiện cho GC thu gom
        x.next = null;
    }

    // Gán phần tử node hiện tại thành null, thuận tiện cho GC thu gom
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

Logic của phương thức `unlink()` như sau:

1. Đầu tiên lấy node phía trước (prev) và node kế tiếp (next) của node x chờ xóa;
2. Kiểm tra xem node chờ xóa có phải là node đầu hay node cuối hay không:
   - Nếu x là node đầu, cho first trỏ đến node kế tiếp next của x
   - Nếu x là node cuối, cho last trỏ đến node phía trước prev của x
   - Nếu x không phải node đầu cũng không phải node cuối, thực hiện bước tiếp theo
3. Cho con trỏ next của node phía trước x trỏ đến node kế tiếp next của node chờ xóa, ngắt liên kết giữa x và x.prev;
4. Cho con trỏ prev của node kế tiếp x trỏ đến node phía trước prev của node chờ xóa, ngắt liên kết giữa x và x.next;
5. Gán phần tử của node x chờ xóa thành rỗng, sửa đổi độ dài danh sách liên kết.

Có thể tham khảo hình dưới đây để hiểu (Nguồn hình: [Phân tích source code LinkedList (JDK 1.8)](https://www.tianxiaobo.com/2018/01/31/LinkedList-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-JDK-1-8/)):

![Logic của phương thức unlink](https://oss.javaguide.cn/github/javaguide/java/collection/linkedlist-unlink.jpg)

### Duyệt danh sách liên kết

Khuyến nghị sử dụng vòng lặp `for-each` để duyệt các phần tử trong `LinkedList`, vòng lặp `for-each` cuối cùng sẽ được chuyển đổi thành dạng iterator.

```java
LinkedList<String> list = new LinkedList<>();
list.add("apple");
list.add("banana");
list.add("pear");

for (String fruit : list) {
    System.out.println(fruit);
}
```

Trọng tâm việc duyệt `LinkedList` nằm ở sự triển khai iterator của nó.

```java
// Iterator hai chiều
private class ListItr implements ListIterator<E> {
    // Biểu thị node đã đi qua trong lần gọi next() hoặc previous() gần nhất;
    private Node<E> lastReturned;
    // Biểu thị node tiếp theo cần duyệt;
    private Node<E> next;
    // Biểu thị index của node tiếp theo cần duyệt, tức là index của node kế tiếp của node hiện tại;
    private int nextIndex;
    // Biểu thị giá trị đếm sửa đổi kỳ vọng của lần duyệt hiện tại, dùng để so sánh với modCount của LinkedList, kiểm tra xem danh sách liên kết có bị thread khác sửa đổi hay không.
    private int expectedModCount = modCount;
    …………
}
```

Dưới đây chúng ta giới thiệu chi tiết các phương thức cốt lõi trong iterator `ListItr`.

Trước tiên hãy cùng xem duyệt theo chiều từ đầu đến cuối:

```java
// Kiểm tra xem còn node tiếp theo hay không
public boolean hasNext() {
    // Kiểm tra index của node tiếp theo có nhỏ hơn size danh sách liên kết hay không, nếu có nghĩa là còn phần tử tiếp theo để duyệt
    return nextIndex < size;
}
// Lấy node tiếp theo
public E next() {
    // Kiểm tra xem danh sách liên kết có bị sửa đổi trong quá trình lặp hay không
    checkForComodification();
    // Kiểm tra xem còn node tiếp theo để duyệt hay không, nếu không thì ném ngoại lệ NoSuchElementException
    if (!hasNext())
        throw new NoSuchElementException();
    // Cho lastReturned trỏ đến node hiện tại
    lastReturned = next;
    // Cho next trỏ đến node tiếp theo
    next = next.next;
    nextIndex++;
    return lastReturned.item;
}
```

Tiếp theo hãy xem duyệt theo chiều từ cuối về đầu:

```java
// Kiểm tra xem còn node phía trước hay không
public boolean hasPrevious() {
    return nextIndex > 0;
}

// Lấy node phía trước
public E previous() {
    // Kiểm tra xem danh sách liên kết có bị sửa đổi trong quá trình lặp hay không
    checkForComodification();
    // Nếu không có node phía trước thì ném ngoại lệ
    if (!hasPrevious())
        throw new NoSuchElementException();
    // Cho con trỏ lastReturned và next trỏ đến node phía trước
    lastReturned = next = (next == null) ? last : next.prev;
    nextIndex--;
    return lastReturned.item;
}
```

Nếu cần xóa hoặc chèn phần tử, cũng có thể sử dụng iterator để thao tác.

```java
LinkedList<String> list = new LinkedList<>();
list.add("apple");
list.add(null);
list.add("banana");

// Phương thức removeIf của interface Collection bên dưới vẫn dựa trên iterator
list.removeIf(Objects::isNull);

for (String fruit : list) {
    System.out.println(fruit);
}
```

Phương thức xóa phần tử tương ứng của iterator như sau:

```java
// Xóa phần tử được trả về lần gần nhất khỏi danh sách
public void remove() {
    // Kiểm tra xem danh sách liên kết có bị sửa đổi trong quá trình lặp hay không
    checkForComodification();
    // Nếu node trả về lần trước rỗng thì ném ngoại lệ
    if (lastReturned == null)
        throw new IllegalStateException();

    // Lấy node kế tiếp của node hiện tại
    Node<E> lastNext = lastReturned.next;
    // Xóa node trả về lần trước khỏi danh sách liên kết
    unlink(lastReturned);
    // Sửa con trỏ
    if (next == lastReturned)
        next = lastNext;
    else
        nextIndex--;
    // Gán tham chiếu node trả về lần trước thành null, thuận tiện cho GC thu gom
    lastReturned = null;
    expectedModCount++;
}
```

## Test các phương thức thường dùng của LinkedList

Code:

```java
// Tạo đối tượng LinkedList
LinkedList<String> list = new LinkedList<>();

// Thêm phần tử vào cuối danh sách liên kết
list.add("apple");
list.add("banana");
list.add("pear");
System.out.println("Nội dung danh sách liên kết：" + list);

// Chèn phần tử vào vị trí chỉ định
list.add(1, "orange");
System.out.println("Nội dung danh sách liên kết：" + list);

// Lấy phần tử tại vị trí chỉ định
String fruit = list.get(2);
System.out.println("Phần tử tại index 2：" + fruit);

// Sửa phần tử tại vị trí chỉ định
list.set(3, "grape");
System.out.println("Nội dung danh sách liên kết：" + list);

// Xóa phần tử tại vị trí chỉ định
list.remove(0);
System.out.println("Nội dung danh sách liên kết：" + list);

// Xóa phần tử chỉ định xuất hiện lần đầu tiên
list.remove("banana");
System.out.println("Nội dung danh sách liên kết：" + list);

// Lấy độ dài của danh sách liên kết
int size = list.size();
System.out.println("Độ dài danh sách liên kết：" + size);

// Xóa sạch danh sách liên kết
list.clear();
System.out.println("Danh sách liên kết sau khi xóa sạch：" + list);
```

Output:

```plain
Phần tử tại index 2：banana
Nội dung danh sách liên kết：[apple, orange, banana, grape]
Nội dung danh sách liên kết：[orange, banana, grape]
Nội dung danh sách liên kết：[orange, grape]
Độ dài danh sách liên kết：2
Danh sách liên kết sau khi xóa sạch：[]
```

<!-- @include: @article-footer.snippet.md -->
