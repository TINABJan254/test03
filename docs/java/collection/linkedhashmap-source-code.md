---
title: LinkedHashMap 源码分析
description: LinkedHashMap源码深度剖析：详解LinkedHashMap维护双向链表实现插入/访问有序、LRU缓存实现、与HashMap区别及遍历效率优化。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: LinkedHashMap源码,插入顺序,访问顺序,LRU缓存,双向链表,有序Map,LinkedHashMap实现原理
---

## Giới thiệu về LinkedHashMap

`LinkedHashMap` là một collection class do Java cung cấp, kế thừa từ `HashMap`, và trên cơ sở `HashMap` nó duy trì thêm một danh sách liên kết đôi (doubly linked list), giúp sở hữu các đặc tính sau:

1. Hỗ trợ duyệt (iterate) theo đúng thứ tự chèn (insertion order).
2. Hỗ trợ sắp xếp theo thứ tự truy cập phần tử (access order), thích hợp dùng để đóng gói công cụ LRU cache.
3. Vì bên trong sử dụng danh sách liên kết đôi để duy trì các node, nên hiệu năng duyệt tỉ lệ thuận với số lượng phần tử, so với `HashMap` (hiệu năng duyệt tỉ lệ thuận với capacity) thì hiệu năng lặp (iteration) cao hơn rất nhiều.

Cấu trúc logic của `LinkedHashMap` như hình dưới đây, nó dựa trên cơ sở `HashMap` duy trì thêm một danh sách liên kết đôi giữa các node, giúp các node, danh sách liên kết, cây đỏ đen vốn phân tán rải rác trên các bucket khác nhau được liên kết có thứ tự với nhau.

![Cấu trúc logic của LinkedHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/linkhashmap-structure-overview.png)

## Ví dụ sử dụng LinkedHashMap

### Duyệt theo thứ tự chèn

Như hiển thị bên dưới, chúng ta thêm các phần tử vào `LinkedHashMap` theo thứ tự rồi tiến hành duyệt.

```java
HashMap < String, String > map = new LinkedHashMap < > ();
map.put("a", "2");
map.put("g", "3");
map.put("r", "1");
map.put("e", "23");

for (Map.Entry < String, String > entry: map.entrySet()) {
    System.out.println(entry.getKey() + ":" + entry.getValue());
}
```

Output:

```java
a:2
g:3
r:1
e:23
```

Có thể thấy, thứ tự lặp của `LinkedHashMap` nhất quán với thứ tự chèn, điểm này `HashMap` không có được.

### Duyệt theo thứ tự truy cập

`LinkedHashMap` định nghĩa chế độ sắp xếp `accessOrder` (kiểu boolean, mặc định là false), thứ tự truy cập là true, thứ tự chèn là false.

Để thực hiện duyệt theo thứ tự truy cập, chúng ta có thể sử dụng constructor của `LinkedHashMap` truyền vào thuộc tính `accessOrder`, và thiết lập `accessOrder` thành true, biểu thị nó sở hữu tính có thứ tự khi truy cập.

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.put(4, "four");
map.put(5, "five");
// Truy cập phần tử 2, phần tử này sẽ bị di chuyển đến cuối danh sách liên kết
map.get(2);
// Truy cập phần tử 3, phần tử này sẽ bị di chuyển đến cuối danh sách liên kết
map.get(3);
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " : " + entry.getValue());
}
```

Output:

```java
1 : one
4 : four
5 : five
2 : two
3 : three
```

Có thể thấy, thứ tự lặp của `LinkedHashMap` nhất quán với thứ tự truy cập.

### LRU Cache

Từ phần trước chúng ta hiểu rằng thông qua `LinkedHashMap`, chúng ta có thể đóng gói một phiên bản LRU (**L**east **R**ecently **U**sed, gần đây ít sử dụng nhất) cache đơn giản, đảm bảo khi các phần tử lưu trữ vượt quá dung lượng container, phần tử ít được truy cập nhất gần đây sẽ bị xóa bỏ.

![](https://oss.javaguide.cn/github/javaguide/java/collection/lru-cache.png)

Ý tưởng triển khai cụ thể như sau:

- Kế thừa `LinkedHashMap`;
- Trong constructor chỉ định `accessOrder` là true, như vậy khi truy cập phần tử sẽ chuyển phần tử đó đến cuối danh sách liên kết, phần tử đầu danh sách liên kết chính là phần tử ít được truy cập nhất gần đây;
- Override phương thức `removeEldestEntry`, phương thức này sẽ trả về giá trị boolean, thông báo cho `LinkedHashMap` có cần xóa phần tử đầu danh sách liên kết hay không (dung lượng cache có hạn).

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    /**
     * Kiểm tra size vượt quá dung lượng thì trả về true, thông báo cho LinkedHashMap xóa item cache cũ nhất (tức phần tử đầu tiên của danh sách liên kết)
     */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

Code test như sau, tác giả khởi tạo dung lượng cache là 3, sau đó lần lượt thêm 4 phần tử theo thứ tự.

```java
LRUCache<Integer, String> cache = new LRUCache<>(3);
cache.put(1, "one");
cache.put(2, "two");
cache.put(3, "three");
cache.put(4, "four");
cache.put(5, "five");
for (int i = 1; i <= 5; i++) {
    System.out.println(cache.get(i));
}
```

Output:

```java
null
null
three
four
five
```

Từ kết quả output, do dung lượng cache là 3, nên khi thêm phần tử thứ 4, phần tử thứ 1 sẽ bị xóa. Khi thêm phần tử thứ 5, phần tử thứ 2 sẽ bị xóa.

## Phân tích source code của LinkedHashMap

### Thiết kế của Node

Trước khi thảo luận chính thức về `LinkedHashMap`, chúng ta hãy bàn về thiết kế node `Entry` của `LinkedHashMap`. Chúng ta đều biết node chuyển thành danh sách liên kết do xung đột trên bucket của `HashMap` sẽ chuyển danh sách liên kết thành cây đỏ đen khi thỏa mãn 2 điều kiện sau:

1. Số lượng node trên danh sách liên kết đạt ngưỡng treeify là 8 (thay vì 7).
2. Dung lượng bucket đạt dung lượng treeify tối thiểu tức `MIN_TREEIFY_CAPACITY`.

> **🐛 Sửa lỗi (Xem: [issue#2147](https://github.com/Snailclimb/JavaGuide/issues/2147))**:
>
> Số node trên danh sách liên kết đạt ngưỡng treeify là 8 chứ không phải 7. Vì điều kiện trong source code duyệt từ phần tử khởi tạo của danh sách liên kết, index bắt đầu từ 0, nên điều kiện kiểm tra đặt là 8-1=7, thực chất là khi lặp đến phần tử cuối mới kiểm tra toàn bộ độ dài danh sách liên kết lớn hơn hoặc bằng 8 mới thực hiện thao tác treeify.
>
> ![](https://oss.javaguide.cn/github/javaguide/java/jvm/LinkedHashMap-putval-TREEIFY.png)

Còn `LinkedHashMap` là dựa trên cơ sở `HashMap` thiết lập một danh sách liên kết đôi cho mỗi node trên bucket, điều này khiến node cây chuyển thành cây đỏ đen cũng cần sở hữu đặc tính node danh sách liên kết đôi, tức là mỗi node cây đều cần sở hữu 2 tham chiếu lưu trữ địa chỉ node phía trước (prev) và node kế tiếp (next), do đó việc thiết kế class node cây `TreeNode` là một vấn đề khá hóc chuẩn.

Về điểm này chúng ta hãy nhìn vào sơ đồ class node giữa cả hai, có thể thấy:

1. Inner class node `Entry` của `LinkedHashMap` dựa trên cơ sở `HashMap`, bổ sung con trỏ `before` và `after` giúp node sở hữu đặc tính của danh sách liên kết đôi.
2. Node cây `TreeNode` của `HashMap` kế thừa `Entry` của `LinkedHashMap` (vốn sở hữu đặc tính danh sách liên kết đôi).

![Mối quan hệ giữa LinkedHashMap và HashMap](https://oss.javaguide.cn/github/javaguide/java/collection/map-hashmap-linkedhashmap.png)

Nhiều độc giả lúc này sẽ thắc mắc: Tại sao node cây `TreeNode` của `HashMap` lại phải thông qua `LinkedHashMap` để lấy đặc tính danh sách liên kết đôi? Tại sao không trực tiếp triển khai con trỏ prev và next trên `Node`?

Trả lời câu hỏi thứ nhất trước: Chúng ta đều biết `LinkedHashMap` dựa trên `HashMap` bổ sung con trỏ hai chiều cho node để thực hiện đặc tính danh sách liên kết đôi, nên khi danh sách liên kết bên trong `LinkedHashMap` chuyển thành cây đỏ đen, node tương ứng sẽ chuyển thành node cây `TreeNode`. Để đảm bảo khi sử dụng `LinkedHashMap` node cây sở hữu đặc tính danh sách liên kết đôi, nên node cây `TreeNode` cần kế thừa `Entry` của `LinkedHashMap`.

Nói tiếp về câu hỏi thứ hai: Chúng ta trực tiếp triển khai con trỏ prev và next trên node `Node` của `HashMap`, rồi `TreeNode` trực tiếp kế thừa `Node` để lấy đặc tính danh sách liên kết đôi tại sao lại không được? Thực ra làm vậy cũng được. Chỉ có điều cách làm này sẽ khiến class node `Node` lưu trữ cặp key-value khi sử dụng `HashMap` bị thừa 2 tham chiếu không cần thiết, tiêu tốn dung lượng bộ nhớ không cần thiết.

Do đó, để đảm bảo class node `Node` bên dưới `HashMap` không có tham chiếu dư thừa, đồng thời lại đảm bảo class node `Entry` của `LinkedHashMap` sở hữu tham chiếu lưu trữ danh sách liên kết, nhà thiết kế đã để node `Entry` của `LinkedHashMap` kế thừa Node và bổ sung tham chiếu `before`, `after` lưu trữ node phía trước/kế tiếp, để node nào cần dùng đặc tính danh sách liên kết sẽ triển khai logic cần thiết. Sau đó node cây `TreeNode` thông qua việc kế thừa `Entry` để lấy 2 con trỏ `before`, `after`.

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

Nhưng làm vậy chẳng phải cũng làm cho `TreeNode` khi sử dụng `HashMap` bị thừa 2 tham chiếu không cần thiết sao? Chẳng phải đây cũng là một sự lãng phí không gian sao?

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
  // Lược bỏ

}
```

Đối với vấn đề này, trích dẫn một đoạn comment của tác giả, các tác giả cho rằng với thuật toán `hashCode` tốt, xác suất `HashMap` chuyển thành cây đỏ đen là không lớn. Cho dù chuyển thành cây đỏ đen biến thành node cây, cũng có thể do xóa hoặc resize làm cho `TreeNode` biến lại thành `Node`, nên xác suất sử dụng `TreeNode` không quá lớn, sự lãng phí một chút tài nguyên bộ nhớ này là có thể chấp nhận được.

```bash
Because TreeNodes are about twice the size of regular nodes, we
use them only when bins contain enough nodes to warrant use
(see TREEIFY_THRESHOLD). And when they become too small (due to
removal or resizing) they are converted back to plain bins.  In
usages with well-distributed user hashCodes, tree bins are
rarely used.  Ideally, under random hashCodes, the frequency of
nodes in bins follows a Poisson distribution
```

### Constructor

Constructor của `LinkedHashMap` có 4 implementation cũng tương đối đơn giản, gọi trực tiếp constructor của class cha (tức `HashMap`) để hoàn thành khởi tạo.

```java
public LinkedHashMap() {
    super();
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity,
    float loadFactor,
    boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

Như chúng ta đã đề cập ở trên, mặc định `accessOrder` là false, nếu chúng ta muốn `LinkedHashMap` thực hiện sắp xếp cặp key-value theo thứ tự truy cập (tức đưa phần tử chưa truy cập gần đây xếp ở đầu danh sách liên kết, phần tử truy cập gần nhất chuyển đến cuối danh sách liên kết), cần gọi constructor thứ 4 để thiết lập `accessOrder` thành true.

### Phương thức get

Phương thức `get` là phương thức duy nhất được override trong các thao tác thêm xóa sửa tìm kiếm của `LinkedHashMap`. Trong trường hợp `accessOrder` là true, sau khi truy vấn phần tử hoàn tất, nó sẽ di chuyển phần tử vừa truy cập đến cuối danh sách liên kết.

```java
public V get(Object key) {
     Node < K, V > e;
     // Lấy cặp key-value của key, nếu null trả về trực tiếp
     if ((e = getNode(hash(key), key)) == null)
         return null;
     // Nếu accessOrder là true, gọi afterNodeAccess để chuyển phần tử hiện tại đến cuối danh sách liên kết
     if (accessOrder)
         afterNodeAccess(e);
     // Trả về value của cặp key-value
     return e.value;
 }
```

Từ source code có thể thấy, các bước thực thi của `get` rất đơn giản:

1. Gọi `getNode` của class cha (tức `HashMap`) để lấy cặp key-value, nếu null thì trả về trực tiếp.
2. Kiểm tra `accessOrder` có phải true hay không, nếu true chứng tỏ cần đảm bảo tính có thứ tự khi truy cập danh sách liên kết của `LinkedHashMap`, thực hiện bước 3.
3. Gọi `afterNodeAccess` được `LinkedHashMap` override để thêm phần tử hiện tại vào cuối danh sách liên kết.

Điểm mấu chốt nằm ở sự triển khai phương thức `afterNodeAccess`, phương thức này chịu trách nhiệm chuyển phần tử đến cuối danh sách liên kết.

```java
void afterNodeAccess(Node < K, V > e) { // move node to last
    LinkedHashMap.Entry < K, V > last;
    // Nếu accessOrder và node hiện tại không phải node cuối của danh sách liên kết
    if (accessOrder && (last = tail) != e) {

        // Lấy node hiện tại, cùng với node phía trước và node kế tiếp
        LinkedHashMap.Entry < K, V > p =
            (LinkedHashMap.Entry < K, V > ) e, b = p.before, a = p.after;

        // Cho con trỏ node kế tiếp của node hiện tại trỏ thành null, khiến nó ngắt liên kết với node kế tiếp
        p.after = null;

        // Nếu node phía trước null, chứng tỏ node hiện tại là node đầu danh sách liên kết, nên thiết lập node kế tiếp làm node đầu
        if (b == null)
            head = a;
        else
            // Nếu node phía trước không null, cho node phía trước trỏ đến node kế tiếp
            b.after = a;

        // Nếu node kế tiếp không null, cho node kế tiếp trỏ đến node phía trước
        if (a != null)
            a.before = b;
        else
            // Nếu node kế tiếp null, chứng tỏ node hiện tại ở cuối danh sách liên kết, cho last trỏ trực tiếp đến node phía trước
            last = b;

        // Nếu last null, chứng tỏ danh sách liên kết hiện tại chỉ có 1 node p, cho head trỏ đến p
        if (last == null)
            head = p;
        else {
            // Ngược lại cho con trỏ prev của p trỏ đến node cuối, rồi cho con trỏ prev của node cuối trỏ đến p
            p.before = last;
            last.after = p;
        }
        // tail trỏ đến p, từ đó di chuyển node p đến cuối danh sách liên kết
        tail = p;

        ++modCount;
    }
}
```

Từ source code có thể thấy, phương thức `afterNodeAccess` hoàn thành các thao tác sau:

1. Nếu `accessOrder` là true và tail danh sách liên kết không phải node p hiện tại, chúng ta cần chuyển node hiện tại đến cuối danh sách liên kết.
2. Lấy node p hiện tại, cùng với node phía trước b và node kế tiếp a của nó.
3. Thiết lập con trỏ after của node p hiện tại thành null, khiến nó ngắt liên kết với node kế tiếp.
4. Thử cho node phía trước b trỏ đến node kế tiếp a, nếu node phía trước null chứng tỏ node p hiện tại là node đầu danh sách liên kết, do đó gán trực tiếp node kế tiếp a thành head, sau đó chúng ta nối p vào cuối a.
5. Thử cho node kế tiếp a trỏ đến node phía trước b.
6. Các thao tác trên giúp node phía trước và node kế tiếp hoàn thành liên kết và cô lập node p hiện tại, bước này nối node p hiện tại vào cuối danh sách liên kết, nếu tail danh sách liên kết rỗng chứng tỏ danh sách liên kết hiện tại chỉ có 1 node p, nên trực tiếp cho head trỏ đến p là được.
7. Thao tác trên đã chuyển p đến cuối danh sách liên kết thành công, cuối cùng cho con trỏ tail (con trỏ trỏ đến cuối danh sách liên kết) trỏ đến p là được.

Có thể kết hợp hình này để hiểu, trình bày phần tử có key là 13 được di chuyển đến cuối danh sách liên kết.

![Di chuyển phần tử 13 trong LinkedHashMap đến cuối danh sách liên kết](https://oss.javaguide.cn/github/javaguide/java/collection/linkedhashmap-get.png)

### newNode — Chèn node mới vào cuối danh sách liên kết

Phần trên đã giới thiệu `afterNodeAccess` di chuyển **node đã tồn tại** đến cuối danh sách liên kết như thế nào, vậy **node mới chèn vào** được thêm vào danh sách liên kết như thế nào?

Câu trả lời nằm ở chỗ `LinkedHashMap` đã override phương thức `newNode` của `HashMap`. Khi `HashMap` chèn cặp key-value mới, sẽ gọi `newNode` để tạo đối tượng node, `LinkedHashMap` trong phương thức override không chỉ tạo node `Entry` mà còn gọi thêm `linkNodeLast` để nối nó vào cuối danh sách liên kết đôi:

```java
// newNode của HashMap là implementation thông thường
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}

// LinkedHashMap override newNode, gọi thêm linkNodeLast
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<>(hash, key, value, e);
    linkNodeLast(p);  // Mấu chốt: Nối node mới vào cuối danh sách liên kết
    return p;
}
```

Sự triển khai phương thức `linkNodeLast` như sau:

```java
// Nối node vào cuối danh sách liên kết đôi
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;  // tail trỏ đến node mới
    if (last == null)
        head = p;  // Danh sách liên kết rỗng, head cũng trỏ đến node mới
    else {
        p.before = last;  // Node phía trước của node mới trỏ đến node tail ban đầu
        last.after = p;   // Node kế tiếp của node tail ban đầu trỏ đến node mới
    }
}
```

**Đây chính là cơ chế cốt lõi để LinkedHashMap thực hiện tính có thứ tự khi chèn**: Mỗi lần chèn node mới, thông qua override `newNode` và gọi `linkNodeLast`, node mới sẽ được nối vào cuối danh sách liên kết đôi. Như vậy khi duyệt sẽ bắt đầu từ head node `head` men theo con trỏ `after` để duyệt, từ đó có thể lấy tất cả phần tử theo thứ tự chèn.

Tương tự, `LinkedHashMap` cũng override phương thức `newTreeNode`, đảm bảo node cây khi chèn vào cũng được nối vào cuối danh sách liên kết:

```java
TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
    TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
    linkNodeLast(p);
    return p;
}
```

### Thao tác hậu xử lý phương thức remove — afterNodeRemoval

`LinkedHashMap` không override phương thức `remove` mà trực tiếp kế thừa phương thức `remove` của `HashMap`. Để đảm bảo node trong danh sách liên kết đôi cũng đồng thời bị gỡ bỏ sau khi cặp key-value bị gỡ bỏ, `LinkedHashMap` đã override phương thức rỗng `afterNodeRemoval` của `HashMap`.

```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                                boolean matchValue, boolean movable) {
        // Lược bỏ
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                // removeNode của HashMap sau khi hoàn thành gỡ bỏ phần tử sẽ gọi afterNodeRemoval để thực hiện hậu xử lý gỡ bỏ
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
// Implementation rỗng
void afterNodeRemoval(Node<K,V> p) { }
```

Chúng ta có thể thấy phương thức `removeNode` được gọi bên trong phương thức `remove` kế thừa từ `HashMap` sau khi gỡ bỏ node khỏi bucket, sẽ gọi `afterNodeRemoval`.

```java
void afterNodeRemoval(Node<K,V> e) { // unlink

    // Lấy node p hiện tại, cùng với node phía trước b và node kế tiếp a của e
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    // Gán cả con trỏ before và after của p thành null, khiến nó ngắt liên kết với node phía trước và node kế tiếp
        p.before = p.after = null;

    // Nếu node phía trước null, chứng tỏ node p hiện tại là node đầu danh sách liên kết, cho con trỏ head trỏ đến node kế tiếp a là được
        if (b == null)
            head = a;
        else
        // Nếu node phía trước b không null, cho b trỏ trực tiếp đến node kế tiếp a
            b.after = a;

    // Nếu node kế tiếp null, chứng tỏ node p hiện tại ở cuối danh sách liên kết, nên cho con trỏ tail trỏ trực tiếp đến node phía trước b là được
        if (a == null)
            tail = b;
        else
        // Ngược lại con trỏ before của node kế tiếp trỏ trực tiếp đến node phía trước
            a.before = b;
    }
```

Từ source code có thể thấy, toàn bộ thao tác của phương thức `afterNodeRemoval` là ngắt liên kết của node p hiện tại với node phía trước và node kế tiếp, chờ GC thu gom, các bước tổng thể bao gồm:

1. Lấy node p hiện tại, cùng với node phía trước b và node kế tiếp a của p.
2. Ngắt liên kết của node p hiện tại với node phía trước và node kế tiếp.
3. Thử cho node phía trước b trỏ đến node kế tiếp a, nếu b null chứng tỏ node p hiện tại ở đầu danh sách liên kết, chúng ta gán trực tiếp head trỏ đến node kế tiếp a là được.
4. Thử cho node kế tiếp a trỏ đến node phía trước b, nếu a null chứng tỏ node p hiện tại ở cuối danh sách liên kết, do đó trực tiếp cho con trỏ tail trỏ đến node phía trước b là được.

Có thể kết hợp hình này để hiểu, trình bày phần tử có key là 13 được xóa, tức là gỡ bỏ phần tử này khỏi danh sách liên kết.

![Xóa phần tử 13 trong LinkedHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/linkedhashmap-remove.png)

### Thao tác hậu xử lý phương thức put — afterNodeInsertion

Tương tự, `LinkedHashMap` không triển khai phương thức chèn mà trực tiếp kế thừa tất cả phương thức chèn của `HashMap` cho người dùng sử dụng, nhưng để duy trì tính có thứ tự khi truy cập danh sách liên kết đôi, nó làm 2 việc:

1. Override `afterNodeAccess` (đã đề cập ở trên), nếu key được chèn vào đã tồn tại trong `map`, vì thao tác chèn của `LinkedHashMap` sẽ nối node mới vào cuối danh sách liên kết, nên đối với key đã tồn tại sẽ gọi `afterNodeAccess` để đưa nó đến cuối danh sách liên kết.
2. Override phương thức `afterNodeInsertion` của `HashMap`, khi `removeEldestEntry` trả về true sẽ gỡ bỏ node đầu của danh sách liên kết.

Điểm này chúng ta có thể thấy trong phương thức cốt lõi của thao tác chèn `putVal` trong `HashMap`.

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
          // Lược bỏ
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                 // Nếu key hiện tại đã tồn tại trong map, gọi afterNodeAccess
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
         // Gọi phương thức hậu xử lý chèn, phương thức này được LinkedHashMap override
        afterNodeInsertion(evict);
        return null;
    }
```

Source code các bước trên đã giải thích ở trước rồi, nên ở đây chúng ta tập trung tìm hiểu quy trình làm việc của `afterNodeInsertion`, giả sử chúng ta override `removeEldestEntry`, khi `size` của danh sách liên kết vượt quá `capacity` thì trả về true.

```java
/**
 * Kiểm tra size vượt quá dung lượng thì trả về true, thông báo cho LinkedHashMap xóa item cache cũ nhất (tức phần tử đầu tiên của danh sách liên kết)
 */
protected boolean removeEldestEntry(Map.Entry < K, V > eldest) {
    return size() > capacity;
}
```

Lấy hình dưới đây làm ví dụ, giả sử tác giả cuối cùng chèn thêm một node 19 chưa tồn tại, giả sử `capacity` là 4, nên `removeEldestEntry` trả về true, chúng ta phải gỡ bỏ node đầu danh sách liên kết.

![Chèn phần tử mới 19 vào LinkedHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/linkedhashmap-after-insert-1.png)

Các bước gỡ bỏ rất đơn giản, kiểm tra node đầu danh sách liên kết có tồn tại không, nếu có thì ngắt mối quan hệ giữa node đầu và node kế tiếp, và cho con trỏ node đầu trỏ đến node tiếp theo, do đó con trỏ head trỏ đến 12, node 10 trở thành đối tượng rỗng không có bất kỳ tham chiếu nào trỏ tới, chờ GC.

![Chèn phần tử mới 19 vào LinkedHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/linkedhashmap-after-insert-2.png)

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        // Nếu evict là true và phần tử đầu queue không null cũng như removeEldestEntry trả về true, chứng tỏ chúng ta cần gỡ bỏ phần tử cũ nhất (tức phần tử ở đầu danh sách liên kết).
        if (evict && (first = head) != null && removeEldestEntry(first)) {
          // Lấy key của cặp key-value ở đầu danh sách liên kết
            K key = first.key;
            // Gọi removeNode để gỡ bỏ phần tử khỏi bucket của HashMap, và ngắt khỏi danh sách liên kết đôi của LinkedHashMap, chờ GC thu gom
            removeNode(hash(key), key, null, false, true);
        }
    }
```

Từ source code có thể thấy, phương thức `afterNodeInsertion` hoàn thành các thao tác sau:

1. Kiểm tra `evict` (tham số) có phải true hay không, chỉ khi là true mới chứng tỏ có thể cần gỡ bỏ cặp key-value cũ nhất (tức phần tử ở đầu danh sách liên kết). Cụ thể có thực hiện gỡ bỏ hay không còn phải xác định xem danh sách liên kết có null hay không `((first = head) != null)`, cũng như phương thức `removeEldestEntry` có trả về true hay không.
2. Lấy key của phần tử đầu tiên trong danh sách liên kết.
3. Gọi phương thức `removeNode` của `HashMap`, phương thức này chúng ta đã đề cập ở trên, nó sẽ gỡ bỏ node khỏi bucket của `HashMap`, và `LinkedHashMap` còn override phương thức `afterNodeRemoval` trong `removeNode`, nên bước này sẽ thông qua việc gọi `removeNode` gỡ bỏ phần tử khỏi bucket của `HashMap`, đồng thời ngắt liên kết với danh sách liên kết đôi của `LinkedHashMap`, chờ GC thu gom.

## So sánh hiệu năng duyệt giữa LinkedHashMap và HashMap

`LinkedHashMap` duy trì một danh sách liên kết đôi để ghi lại thứ tự chèn dữ liệu, do đó khi iterator lặp duyệt, nó sẽ duyệt theo đường đi của danh sách liên kết đôi. Điểm này so với cách duyệt toàn bộ bucket của `HashMap` thì hiệu quả hơn nhiều.

Điểm này chúng ta có thể chứng minh từ iterator của cả hai. Trước tiên hãy nhìn iterator của `HashMap`, có thể thấy khi `HashMap` lặp các cặp key-value sẽ dùng phương thức `nextNode`, phương thức này trả về phần tử tiếp theo mà next trỏ tới, và sẽ duyệt bucket từ next để tìm phần tử Node non-null trong bucket tiếp theo.

```java
 final class EntryIterator extends HashIterator
 implements Iterator < Map.Entry < K, V >> {
     public final Map.Entry < K,
     V > next() {
         return nextNode();
     }
 }

 // Lấy Node tiếp theo
 final Node < K, V > nextNode() {
     Node < K, V > [] t;
     // Lấy phần tử tiếp theo next
     Node < K, V > e = next;
     if (modCount != expectedModCount)
         throw new ConcurrentModificationException();
     if (e == null)
         throw new NoSuchElementException();
     // Cho next trỏ đến Node non-null tiếp theo trong bucket
     if ((next = (current = e).next) == null && (t = table) != null) {
         do {} while (index < t.length && (next = t[index++]) == null);
     }
     return e;
 }
```

Ngược lại, iterator của `LinkedHashMap` trực tiếp sử dụng con trỏ `after` để nhanh chóng định vị đến node kế tiếp của node hiện tại, ngắn gọn và hiệu quả hơn nhiều.

```java
 final class LinkedEntryIterator extends LinkedHashIterator
 implements Iterator < Map.Entry < K, V >> {
     public final Map.Entry < K,
     V > next() {
         return nextNode();
     }
 }
 // Lấy Node tiếp theo
 final LinkedHashMap.Entry < K, V > nextNode() {
     // Lấy node tiếp theo next
     LinkedHashMap.Entry < K, V > e = next;
     if (modCount != expectedModCount)
         throw new ConcurrentModificationException();
     if (e == null)
         throw new NoSuchElementException();
     // Con trỏ current trỏ đến node hiện tại
     current = e;
     // next dùng trực tiếp con trỏ after của node hiện tại để định vị nhanh đến node tiếp theo
     next = e.after;
     return e;
 }
```

Để kiểm chứng quan điểm của mình, tác giả đã benchmark 2 container này, test thời gian tiêu tốn khi chèn 10 triệu và duyệt 10 triệu dữ liệu, code như sau:

```java
int count = 1000_0000;
Map<Integer, Integer> hashMap = new HashMap<>();
Map<Integer, Integer> linkedHashMap = new LinkedHashMap<>();

long start, end;

start = System.currentTimeMillis();
for (int i = 0; i < count; i++) {
    hashMap.put(ThreadLocalRandom.current().nextInt(1, count), ThreadLocalRandom.current().nextInt(0, count));
}
end = System.currentTimeMillis();
System.out.println("map time putVal: " + (end - start));

start = System.currentTimeMillis();
for (int i = 0; i < count; i++) {
    linkedHashMap.put(ThreadLocalRandom.current().nextInt(1, count), ThreadLocalRandom.current().nextInt(0, count));
}
end = System.currentTimeMillis();
System.out.println("linkedHashMap putVal time: " + (end - start));

start = System.currentTimeMillis();
long num = 0;
for (Integer v : hashMap.values()) {
    num = num + v;
}
end = System.currentTimeMillis();
System.out.println("map get time: " + (end - start));

start = System.currentTimeMillis();
for (Integer v : linkedHashMap.values()) {
    num = num + v;
}
end = System.currentTimeMillis();
System.out.println("linkedHashMap get time: " + (end - start));
System.out.println(num);
```

Từ kết quả output, vì `LinkedHashMap` cần duy trì danh sách liên kết đôi, nên việc chèn phần tử so với `HashMap` tốn thời gian hơn. Tuy nhiên nhờ có mối quan hệ node trước sau rõ ràng của danh sách liên kết đôi, hiệu năng lặp lại hiệu quả hơn nhiều so với `HashMap`. Tuy vậy tổng thể chênh lệch không quá lớn.

```bash
map time putVal: 5880
linkedHashMap putVal time: 7567
map get time: 143
linkedHashMap get time: 67
63208969074998
```

## Câu hỏi phỏng vấn thường gặp về LinkedHashMap

### LinkedHashMap là gì?

`LinkedHashMap` là một subclass của `HashMap` trong Java Collection Framework, nó kế thừa tất cả thuộc tính và phương thức của `HashMap`, đồng thời trên cơ sở `HashMap` nó override các phương thức `afterNodeRemoval`, `afterNodeInsertion`, `afterNodeAccess`. Giúp nó sở hữu đặc tính chèn có thứ tự và truy cập có thứ tự.

### LinkedHashMap duyệt các phần tử theo thứ tự chèn như thế nào?

`LinkedHashMap` duyệt các phần tử theo thứ tự chèn là hành vi mặc định của nó. Bên trong `LinkedHashMap` duy trì một danh sách liên kết đôi, dùng để ghi lại thứ tự chèn phần tử. Do đó, khi sử dụng iterator để lặp các phần tử, thứ tự phần tử giống hệt với thứ tự chúng được chèn vào ban đầu.

### LinkedHashMap duyệt các phần tử theo thứ tự truy cập như thế nào?

`LinkedHashMap` có thể thông qua tham số `accessOrder` trong constructor để chỉ định duyệt các phần tử theo thứ tự truy cập. Khi `accessOrder` là true, mỗi lần truy cập một phần tử, phần tử đó sẽ bị di chuyển đến cuối danh sách liên kết, do đó lần tới truy cập phần tử đó, nó sẽ trở thành phần tử cuối cùng trong danh sách liên kết, từ đó thực hiện duyệt các phần tử theo thứ tự truy cập.

### LinkedHashMap thực hiện LRU cache như thế nào?

Thiết lập `accessOrder` thành true và override phương thức `removeEldestEntry` trả về true khi kích thước danh sách liên kết vượt quá capacity, làm cho mỗi lần truy cập phần tử, phần tử đó sẽ bị di chuyển đến cuối danh sách liên kết. Một khi thao tác chèn làm cho `removeEldestEntry` trả về true, coi như cache đã đầy, `LinkedHashMap` sẽ gỡ bỏ phần tử đầu danh sách liên kết, từ đó chúng ta có thể thực hiện một LRU cache.

### LinkedHashMap và HashMap có sự khác biệt gì?

`LinkedHashMap` và `HashMap` đều là class implementation của interface Map trong Java Collection Framework. Sự khác biệt lớn nhất giữa chúng nằm ở thứ tự lặp các phần tử. Thứ tự lặp các phần tử của `HashMap` là không cố định, còn `LinkedHashMap` cung cấp tính năng lặp các phần tử theo thứ tự chèn hoặc thứ tự truy cập. Ngoài ra, bên trong `LinkedHashMap` duy trì một danh sách liên kết đôi dùng để ghi lại thứ tự chèn hoặc thứ tự truy cập của các phần tử, còn `HashMap` thì không có danh sách liên kết này. Do đó, hiệu năng chèn của `LinkedHashMap` có thể thấp hơn một chút so với `HashMap`, nhưng nó cung cấp nhiều tính năng hơn và hiệu năng lặp hiệu quả hơn so với `HashMap`.

## Tài liệu tham khảo

- Phân tích chi tiết source code LinkedHashMap (JDK1.8): <https://www.imooc.com/article/22931>
- HashMap và LinkedHashMap: <https://www.cnblogs.com/Spground/p/8536148.html>
- Nguồn từ source code LinkedHashMap: <https://leetcode.cn/problems/lru-cache/solution/yuan-yu-linkedhashmapyuan-ma-by-jeromememory/>
<!-- @include: @article-footer.snippet.md -->
