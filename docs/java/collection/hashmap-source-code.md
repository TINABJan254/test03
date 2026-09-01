---
title: HashMap 源码分析
description: HashMap源码深度剖析：详解JDK1.7/1.8结构差异、hash扰动函数、0.75负载因子、扩容rehash机制、链表转红黑树阈值等HashMap核心原理。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: HashMap源码,哈希表,红黑树,链表,扰动函数,负载因子,HashMap扩容,哈希冲突,JDK1.8优化
---

<!-- @include: @article-header.snippet.md -->

> Cảm ơn [changfubai](https://github.com/changfubai) đã đóng góp cải thiện bài viết này!

## Giới thiệu về HashMap

`HashMap` chủ yếu dùng để lưu trữ các cặp key-value, nó được triển khai dựa trên interface `Map` của bảng băm, là một trong những Java collection phổ biến nhất, không thread-safe.

`HashMap` có thể lưu trữ key và value là null, nhưng null làm key chỉ có thể có một, null làm value có thể có nhiều.

Trước JDK1.8, `HashMap` được cấu thành từ mảng + danh sách liên kết, mảng là thành phần chính của `HashMap`, còn danh sách liên kết chủ yếu tồn tại để giải quyết xung đột hash (phương pháp "chaining" giải quyết xung đột). Từ JDK1.8 trở đi, việc giải quyết xung đột hash trong `HashMap` đã có sự thay đổi lớn: khi độ dài danh sách liên kết lớn hơn hoặc bằng ngưỡng (mặc định là 8) (trước khi chuyển danh sách liên kết thành cây đỏ đen sẽ kiểm tra, nếu độ dài mảng hiện tại nhỏ hơn 64 thì sẽ chọn mở rộng mảng trước chứ không chuyển thành cây đỏ đen), danh sách liên kết sẽ được chuyển đổi thành cây đỏ đen để giảm thời gian tìm kiếm.

Kích thước khởi tạo mặc định của `HashMap` là 16. Sau đó mỗi lần mở rộng, dung lượng tăng gấp 2 lần. Đồng thời, `HashMap` luôn sử dụng lũy thừa của 2 làm kích thước bảng băm.

## Phân tích cấu trúc dữ liệu bên dưới

### Trước JDK1.8

Trước JDK1.8, bên dưới `HashMap` là sự kết hợp giữa **mảng và danh sách liên kết**, còn gọi là **băm chuỗi (chaining hash)**.

`HashMap` thông qua `hashCode` của key trải qua xử lý của hàm nhiễu (perturbation function) thu được giá trị hash, sau đó thông qua `(n - 1) & hash` để xác định vị trí lưu trữ của phần tử hiện tại (n ở đây là chiều dài mảng). Nếu vị trí hiện tại đã có phần tử, sẽ kiểm tra xem giá trị hash và key của phần tử đó có giống với phần tử sắp lưu vào hay không. Nếu giống nhau sẽ ghi đè trực tiếp, nếu không giống sẽ giải quyết xung đột bằng phương pháp chaining.

Khái niệm gọi là hàm nhiễu chính là phương thức `hash` của `HashMap`. Sử dụng phương thức `hash` (hàm nhiễu) là để phòng ngừa một số phương thức `hashCode()` triển khai kém chất lượng, nói cách khác sau khi sử dụng hàm nhiễu có thể giảm thiểu va chạm.

**Source code phương thức hash của HashMap trong JDK 1.8:**

Phương thức hash trong JDK 1.8 đơn giản hơn so me với phương thức hash trong JDK 1.7, nhưng nguyên lý không đổi.

```java
    static final int hash(Object key) {
      int h;
      // key.hashCode(): Trả về giá trị băm (hashCode)
      // ^: Phép XOR theo bit
      // >>>: Dịch phải không dấu, bỏ qua bit dấu, các vị trí trống đều bù bằng 0
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

So sánh với source code phương thức hash của HashMap trong JDK 1.7.

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

So với phương thức hash trong JDK 1.8, hiệu năng phương thức hash trong JDK 1.7 sẽ kém hơn một chút vì dù sao cũng trải qua 4 lần nhiễu bit.

Khái niệm gọi là **"Phương pháp chaining (拉链法)"** chính là: kết hợp danh sách liên kết và mảng. Tức là tạo ra một mảng các danh sách liên kết, mỗi ô trong mảng là một danh sách liên kết. Nếu gặp xung đột hash thì chỉ cần thêm giá trị xung đột vào danh sách liên kết là được.

![Cấu trúc bên trong của HashMap trước JDK1.8](https://oss.javaguide.cn/github/javaguide/java/collection/jdk1.7_hashmap.png)

### Từ JDK1.8 trở đi

So với các phiên bản trước, từ JDK1.8 trở đi việc giải quyết xung đột hash có sự thay đổi lớn.

Khi độ dài danh sách liên kết lớn hơn ngưỡng (mặc định là 8), trước tiên sẽ gọi phương thức `treeifyBin()`. Phương thức này sẽ dựa vào mảng `HashMap` để quyết định có chuyển thành cây đỏ đen hay không. Chỉ khi độ dài mảng lớn hơn hoặc bằng 64 mới thực hiện thao tác chuyển thành cây đỏ đen để giảm thời gian tìm kiếm. Ngược lại, chỉ thực thi phương thức `resize()` để mở rộng mảng. Source code liên quan ở đây không dán ra, chỉ cần chú ý phương thức `treeifyBin()` là được!

![Cấu trúc bên trong của HashMap từ JDK1.8 trở đi](https://oss.javaguide.cn/github/javaguide/java/collection/jdk1.8_hashmap.png)

**Thuộc tính của class:**

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // Serial version UID
    private static final long serialVersionUID = 362498820763181265L;
    // Dung lượng ban đầu mặc định là 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // Dung lượng tối đa
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // Hệ số tải (load factor) mặc định
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // Khi số node trên bucket lớn hơn hoặc bằng giá trị này sẽ chuyển thành cây đỏ đen
    static final int TREEIFY_THRESHOLD = 8;
    // Khi số node trên bucket nhỏ hơn hoặc bằng giá trị này cây sẽ chuyển thành danh sách liên kết
    static final int UNTREEIFY_THRESHOLD = 6;
    // Kích thước table tối thiểu để cấu trúc trong bucket chuyển đổi thành cây đỏ đen
    static final int MIN_TREEIFY_CAPACITY = 64;
    // Mảng lưu trữ phần tử, luôn là bội số lũy thừa của 2
    transient Node<k,v>[] table;
    // View collection chứa tất cả các cặp key-value trong ánh xạ
    transient Set<map.entry<k,v>> entrySet;
    // Số lượng phần tử lưu trữ, chú ý số này không bằng chiều dài mảng.
    transient int size;
    // Bộ đếm mỗi lần mở rộng dung lượng và thay đổi cấu trúc map
    transient int modCount;
    // Ngưỡng threshold (dung lượng * load factor), khi kích thước thực tế vượt quá ngưỡng sẽ mở rộng dung lượng
    int threshold;
    // Hệ số tải (load factor)
    final float loadFactor;
}
```

- **loadFactor Hệ số tải**

  Hệ số tải (load factor) kiểm soát độ dày mỏng của dữ liệu lưu trong mảng, `loadFactor` càng tiến gần đến 1 thì dữ liệu (entry) lưu trong mảng càng nhiều, càng dày đặc, tức là làm tăng độ dài danh sách liên kết; `loadFactor` càng nhỏ (tiến gần đến 0) thì dữ liệu lưu trong mảng càng ít, càng thưa thớt.

  **`loadFactor` quá lớn dẫn đến hiệu năng tìm kiếm phần tử thấp, quá nhỏ dẫn đến tỷ lệ sử dụng mảng thấp, dữ liệu lưu trữ bị rải rác. Giá trị mặc định `0.75f` của `loadFactor` là một ngưỡng hợp lý do nhà phát triển đưa ra**.

  Dung lượng mặc định được cho là 16, hệ số tải là 0.75. Trong quá trình sử dụng Map liên tục lưu dữ liệu vào, khi số lượng vượt quá 16 * 0.75 = 12 thì cần phải mở rộng dung lượng 16 hiện tại, mà quá trình mở rộng mảng cần tạo mảng mới và di chuyển node,... nên rất tốn hiệu năng.

- **threshold**

  **threshold = capacity * loadFactor**, khi `Size > threshold`, cần phải cân nhắc việc mở rộng mảng, tức là đây chính là tiêu chuẩn đo lường mảng có cần mở rộng hay không.

**Source code class Node:**

```java
// Kế thừa từ Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;// Giá trị hash, dùng để so sánh với giá trị hash của phần tử khác khi lưu phần tử vào hashmap
       final K key;// Key
       V value;// Value
       // Trỏ đến node tiếp theo
       Node<K,V> next;
       Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // Override phương thức hashCode()
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        // Override phương thức equals()
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
}
```

**Source code class TreeNode:**

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // Cha
        TreeNode<K,V> left;    // Trái
        TreeNode<K,V> right;   // Phải
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;           // Kiểm tra màu
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
        // Trả về node gốc
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
       }
```

## Phân tích source code của HashMap

### Constructor

Trong `HashMap` có 4 constructor, lần lượt như sau:

```java
    // Constructor mặc định.
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
     }

     // Constructor chứa một "Map" khác
     public HashMap(Map<? extends K, ? extends V> m) {
         this.loadFactor = DEFAULT_LOAD_FACTOR;
         putMapEntries(m, false);// Bên dưới sẽ phân tích phương thức này
     }

     // Constructor chỉ định "dung lượng"
     public HashMap(int initialCapacity) {
         this(initialCapacity, DEFAULT_LOAD_FACTOR);
     }

     // Constructor chỉ định "dung lượng" và "hệ số tải (load factor)"
     public HashMap(int initialCapacity, float loadFactor) {
         if (initialCapacity < 0)
             throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
         if (initialCapacity > MAXIMUM_CAPACITY)
             initialCapacity = MAXIMUM_CAPACITY;
         if (loadFactor <= 0 || Float.isNaN(loadFactor))
             throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
         this.loadFactor = loadFactor;
         // Dung lượng ban đầu tạm thời lưu vào threshold, trong resize mới gán lại cho newCap để khởi tạo table
         this.threshold = tableSizeFor(initialCapacity);
     }
```

> Cần đặc biệt lưu ý: `initialCapacity` truyền vào không phải là dung lượng mảng cuối cùng. `HashMap` sẽ gọi `tableSizeFor()` để **làm tròn lên (round up) thành số mũ nhỏ nhất của 2 lớn hơn hoặc bằng giá trị đó**, và tạm thời lưu vào field `threshold`. Mảng `table` thực sự chỉ được khởi tạo thành kích thước này ở lần mở rộng dung lượng đầu tiên (`resize()`).
>
> Ví dụ: `initialCapacity = 9` → `threshold = 16` → chiều dài `table` cuối cùng là 16.

**Phương thức putMapEntries:**

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // Kiểm tra table đã khởi tạo hay chưa
        if (table == null) { // pre-size
            /*
             * Chưa khởi tạo, s là số phần tử thực tế của m, ft=s/loadFactor => s=ft*loadFactor, giống như công thức
             * Ngưỡng=Dung lượng*LoadFactor đã đề cập ở trước, đúng vậy, ft đề cập đến dung lượng tối thiểu cần thiết để thêm s phần tử
             */
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            /*
             * Theo constructor, table chưa khởi tạo, threshold thực chất lưu trữ dung lượng khởi tạo, nếu dung lượng tối thiểu cần để thêm s个元素
             * lớn hơn dung lượng khởi tạo thì mở rộng dung lượng tối thiểu thành lũy thừa của 2 gần nhất làm khởi tạo.
             * Chú ý ở đây không phải là khởi tạo ngưỡng
             */
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // Đã khởi tạo, và số phần tử m lớn hơn ngưỡng, thực hiện xử lý mở rộng dung lượng
        else if (s > threshold)
            resize();
        // Thêm tất cả phần tử trong m vào HashMap, nếu table chưa khởi tạo, trong putVal sẽ gọi resize để khởi tạo hoặc mở rộng
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

### Phương thức put

`HashMap` chỉ cung cấp `put` để thêm phần tử, phương thức `putVal` chỉ là một phương thức được gọi bởi phương thức `put`, không cung cấp cho người dùng sử dụng.

**Phân tích việc thêm phần tử của phương thức putVal như sau:**

1. Nếu vị trí mảng định vị được không có phần tử thì trực tiếp chèn vào.
2. Nếu vị trí mảng định vị được đã có phần tử thì so sánh với key cần chèn, nếu key giống nhau thì ghi đè trực tiếp; nếu key không giống nhau thì kiểm tra xem p có phải là node cây (TreeNode) hay không, nếu phải thì gọi `e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)` để thêm phần tử vào. Nếu không phải thì duyệt danh sách liên kết để chèn vào (chèn vào cuối danh sách liên kết).

![ ](https://oss.javaguide.cn/github/javaguide/database/sql/put.png)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table chưa khởi tạo hoặc độ dài bằng 0, thực hiện mở rộng dung lượng
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash xác định phần tử được đặt ở bucket nào, bucket rỗng, tạo node mới đưa vào bucket (lúc này node này nằm trong mảng)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // Trong bucket đã tồn tại phần tử (xử lý xung đột hash)
    else {
        Node<K,V> e; K k;
        // Đánh giá nhanh key của node đầu tiên table[i] có giống key chèn vào hay không, nếu giống thì dùng value chèn vào để thay thế value cũ e.
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
        // Kiểm tra xem phần tử chèn vào có phải node cây đỏ đen hay không
        else if (p instanceof TreeNode)
            // Đưa vào cây
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // Không phải node cây đỏ đen chứng tỏ là node danh sách liên kết
        else {
            // Chèn node vào cuối danh sách liên kết
            for (int binCount = 0; ; ++binCount) {
                // Đi đến cuối danh sách liên kết
                if ((e = p.next) == null) {
                    // Chèn node mới vào cuối
                    p.next = newNode(hash, key, value, null);
                    // Số lượng node đạt ngưỡng (mặc định là 8), thực thi phương thức treeifyBin
                    // Phương thức này dựa trên mảng HashMap để quyết định có chuyển thành cây đỏ đen hay không.
                    // Chỉ khi độ dài mảng lớn hơn hoặc bằng 64 mới thực hiện thao tác chuyển thành cây đỏ đen để giảm thời gian tìm kiếm. Ngược lại chỉ mở rộng mảng.
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // Thoát khỏi vòng lặp
                    break;
                }
                // Kiểm tra giá trị key của node trong danh sách liên kết có bằng giá trị key của phần tử chèn vào hay không
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // Bằng nhau, thoát khỏi vòng lặp
                    break;
                // Dùng để duyệt danh sách liên kết trong bucket, kết hợp với e = p.next ở trước để duyệt danh sách liên kết
                p = e;
            }
        }
        // Biểu thị trong bucket đã tìm thấy node có key và hash bằng phần tử chèn vào
        if (e != null) {
            // Ghi lại value của e
            V oldValue = e.value;
            // onlyIfAbsent là false hoặc giá trị cũ là null
            if (!onlyIfAbsent || oldValue == null)
                // Dùng giá trị mới thay thế giá trị cũ
                e.value = value;
            // Callback sau khi truy cập
            afterNodeAccess(e);
            // Trả về giá trị cũ
            return oldValue;
        }
    }
    // Sửa đổi cấu trúc
    ++modCount;
    // Kích thước thực tế lớn hơn ngưỡng thì mở rộng dung lượng
    if (++size > threshold)
        resize();
    // Callback sau khi chèn
    afterNodeInsertion(evict);
    return null;
}
```

**Chúng ta cùng so sánh code phương thức put trong JDK1.7**

**Phân tích phương thức put như sau:**

- ① Nếu vị trí mảng định vị được không có phần tử thì trực tiếp chèn vào.
- ② Nếu vị trí mảng định vị được đã có phần tử, duyệt danh sách liên kết có node đầu là phần tử đó, lần lượt so sánh với key chèn vào, nếu key giống nhau thì ghi đè trực tiếp, khác nhau thì áp dụng phương pháp chèn ở đầu (head insertion) để chèn phần tử.

```java
public V put(K key, V value)
    if (table == EMPTY_TABLE) {
    inflateTable(threshold);
}
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) { // Duyệt trước
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);  // Sau đó chèn
    return null;
}
```

### Phương thức get

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // Phần tử mảng bằng nhau
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // Trong bucket có nhiều hơn một node
        if ((e = first.next) != null) {
            // get trong cây
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // get trong danh sách liên kết
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### Phương thức resize

Khi thực hiện mở rộng dung lượng sẽ duyệt qua các phần tử trong bảng hash, và tận dụng giá trị hash đã có của node cũng như dung lượng cũ để xác định vị trí node trong mảng mới, quá trình này rất tốn thời gian. Khi viết chương trình, hãy cố gắng tránh `resize`. Phương thức `resize` thực chất đã tích hợp việc khởi tạo table và mở rộng table làm một, hành vi bên dưới đều là gán mảng mới cho table.

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // Vượt quá giá trị tối đa thì không mở rộng nữa, chấp nhận va chạm
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // Chưa vượt quá giá trị tối đa thì mở rộng thành 2 lần dung lượng ban đầu
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        // Dung lượng khởi tạo khi tạo đối tượng được đặt trong threshold, lúc này chỉ cần lấy nó làm dung lượng mảng mới
        newCap = oldThr;
    else {
        // signifies using defaults Đối tượng tạo bởi constructor không tham số sẽ tính toán dung lượng và ngưỡng ở đây
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        // Khi tạo đã chỉ định dung lượng khởi tạo hoặc load factor, tiến hành khởi tạo ngưỡng ở đây,
    	// hoặc dung lượng cũ trước khi mở rộng nhỏ hơn 16, tính toán giới hạn resize mới ở đây
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // Di chuyển mỗi bucket sang buckets mới
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    // Chỉ có một node, tính toán trực tiếp vị trí mới của phần tử
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // Tách cây đỏ đen thành 2 cây con, nếu số node của cây con nhỏ hơn hoặc bằng UNTREEIFY_THRESHOLD (mặc định là 6) thì chuyển cây con thành danh sách liên kết.
                    // Nếu số node của cây con lớn hơn UNTREEIFY_THRESHOLD thì giữ nguyên cấu trúc cây của cây con.
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // Index ban đầu
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // Index ban đầu + oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // Đặt index ban đầu vào bucket
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // Đặt index ban đầu + oldCap vào bucket
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## Test các phương thức thường dùng của HashMap

```java
package map;

import java.util.Collection;
import java.util.HashMap;
import java.util.Set;

public class HashMapDemo {

    public static void main(String[] args) {
        HashMap<String, String> map = new HashMap<String, String>();
        // Key không thể trùng lặp, value có thể trùng lặp
        map.put("san", "张三");
        map.put("si", "李四");
        map.put("wu", "王五");
        map.put("wang", "老王");
        map.put("wang", "老王2");// Lão Vương bị ghi đè
        map.put("lao", "老王");
        System.out.println("-------In trực tiếp hashmap:-------");
        System.out.println(map);
        /**
         * Duyệt HashMap
         */
        // 1. Lấy tất cả key trong Map
        System.out.println("-------foreach lấy tất cả key trong Map:------");
        Set<String> keys = map.keySet();
        for (String key : keys) {
            System.out.print(key+"  ");
        }
        System.out.println();// Xuống dòng
        // 2. Lấy tất cả value trong Map
        System.out.println("-------foreach lấy tất cả value trong Map:------");
        Collection<String> values = map.values();
        for (String value : values) {
            System.out.print(value+"  ");
        }
        System.out.println();// Xuống dòng
        // 3. Lấy key đồng thời lấy value tương ứng với key
        System.out.println("-------Lấy key đồng thời lấy value tương ứng với key:-------");
        Set<String> keys2 = map.keySet();
        for (String key : keys2) {
            System.out.print(key + "：" + map.get(key)+"   ");

        }
        /**
         * Nếu vừa muốn duyệt key vừa muốn duyệt value thì khuyến nghị cách này, vì nếu lấy keySet trước rồi mới thực thi map.get(key), bên trong map sẽ thực thi duyệt 2 lần.
         * Một lần là khi lấy keySet, một lần là khi duyệt tất cả key.
         */
        // Khi gọi phương thức put(key,value), đầu tiên key và value được đóng gói vào
        // đối tượng static inner class Entry, thêm đối tượng Entry vào mảng, do đó chúng ta muốn lấy
        // tất cả các cặp key-value trong map, chỉ cần lấy tất cả đối tượng Entry trong mảng, sau đó
        // gọi phương thức getKey() và getValue() trong đối tượng Entry là có thể lấy được cặp key-value
        Set<java.util.Map.Entry<String, String>> entrys = map.entrySet();
        for (java.util.Map.Entry<String, String> entry : entrys) {
            System.out.println(entry.getKey() + "--" + entry.getValue());
        }

        /**
         * các phương thức thường dùng khác của HashMap
         */
        System.out.println("after map.size()："+map.size());
        System.out.println("after map.isEmpty()："+map.isEmpty());
        System.out.println(map.remove("san"));
        System.out.println("after map.remove()："+map);
        System.out.println("after map.get(si)："+map.get("si"));
        System.out.println("after map.containsKey(si)："+map.containsKey("si"));
        System.out.println("after containsValue(李四)："+map.containsValue("李四"));
        System.out.println(map.replace("si", "李四2"));
        System.out.println("after map.replace(si, 李四2):"+map);
    }

}
```

<!-- @include: @article-footer.snippet.md -->
