---
title: Java集合常见面试题总结(下)
description: Java集合高频面试题：深入分析HashMap底层原理、红黑树转换、哈希冲突解决、ConcurrentHashMap线程安全机制、与Hashtable区别等核心知识点。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: HashMap,ConcurrentHashMap,Hashtable,红黑树,哈希冲突,线程安全,集合面试题
---

<!-- @include: @article-header.snippet.md -->

## Map (Quan trọng)

### ⭐️ Sự khác biệt giữa HashMap và Hashtable

- **Có thread-safe hay không:** `HashMap` không thread-safe, `Hashtable` thì thread-safe vì hầu hết các phương thức bên trong `Hashtable` đều đã được修饰 bằng `synchronized`. (Nếu bạn muốn đảm bảo thread-safe thì hãy sử dụng `ConcurrentHashMap` nhé!);
- **Hiệu năng:** Do vấn đề thread-safe, `HashMap` có hiệu năng cao hơn `Hashtable` một chút. Ngoài ra `Hashtable` về cơ bản đã bị loại bỏ/lạc hậu, không nên sử dụng nó trong code;
- **Hỗ trợ Null key và Null value:** `HashMap` có thể lưu trữ key và value là null, nhưng null làm key thì chỉ có thể có 1, null làm value thì có thể có nhiều; `Hashtable` không cho phép key và value là null, nếu không sẽ ném ra `NullPointerException`.
- **Kích thước dung lượng ban đầu và kích thước mở rộng dung lượng mỗi lần khác nhau:** ① Khi tạo nếu không chỉ định giá trị dung lượng ban đầu, dung lượng ban đầu mặc định của `Hashtable` là 11, sau đó mỗi lần mở rộng, dung lượng sẽ tăng thành 2n+1. Dung lượng ban đầu mặc định của `HashMap` là 16, sau đó mỗi lần mở rộng dung lượng tăng gấp 2 lần. ② Khi tạo nếu đã cung cấp giá trị dung lượng ban đầu, `Hashtable` sẽ sử dụng trực tiếp kích thước bạn cung cấp, còn `HashMap` sẽ mở rộng nó thành lũy thừa của 2 (phương thức `tableSizeFor()` trong `HashMap` đảm bảo điều này, bên dưới có đưa ra source code). Nghĩa là `HashMap` luôn sử dụng lũy thừa của 2 làm kích thước bảng băm, phần sau sẽ giải thích tại sao lại là lũy thừa của 2.
- **Cấu trúc dữ liệu bên dưới:** Từ JDK1.8 trở đi `HashMap` có sự thay đổi lớn khi giải quyết xung đột hash: khi độ dài danh sách liên kết lớn hơn ngưỡng (mặc định là 8), danh sách liên kết sẽ được chuyển đổi thành cây đỏ đen (trước khi chuyển danh sách liên kết thành cây đỏ đen sẽ kiểm tra, nếu độ dài mảng hiện tại nhỏ hơn 64 thì chọn mở rộng mảng trước chứ không chuyển thành cây đỏ đen) để giảm thời gian tìm kiếm (sau đây tôi sẽ kết hợp source code để phân tích quá trình này). `Hashtable` không có cơ chế này.
- **Triển khai hàm hash:** `HashMap` thực hiện xử lý nhiễu trộn (perturbation) bit cao và bit thấp đối với giá trị hash để giảm xung đột, trong khi `Hashtable` sử dụng trực tiếp giá trị `hashCode()` của key.

**Constructor có dung lượng ban đầu trong HashMap:**

```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
     public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

Phương thức dưới đây đảm bảo `HashMap` luôn sử dụng lũy thừa của 2 làm kích thước bảng băm.

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### Sự khác biệt giữa HashMap và HashSet

Nếu bạn đã từng xem source code của `HashSet` thì sẽ biết: `HashSet` bên dưới chính là được triển khai dựa trên `HashMap`. (Source code của `HashSet` rất rất ít, vì ngoài `clone()`, `writeObject()`, `readObject()` là những phương thức mà `HashSet` tự mình bắt buộc phải triển khai, các phương thức khác đều gọi trực tiếp phương thức trong `HashMap`).

| `HashMap` | `HashSet` |
| :---: | :---: |
| Triển khai interface `Map` | Triển khai interface `Set` |
| Lưu trữ cặp key-value | Chỉ lưu trữ đối tượng |
| Gọi `put()` để thêm phần tử vào map | Gọi `add()` để thêm phần tử vào `Set` |
| `HashMap` sử dụng key để tính `hashCode` | `HashSet` sử dụng đối tượng thành viên để tính giá trị `hashCode`, đối với hai đối tượng giá trị `hashCode` có thể giống nhau, do đó phương thức `equals()` được dùng để kiểm tra tính bằng nhau của đối tượng |

### ⭐️ Sự khác biệt giữa HashMap và TreeMap

Cả `TreeMap` và `HashMap` đều kế thừa từ `AbstractMap`, nhưng cần lưu ý rằng `TreeMap` còn triển khai interface `NavigableMap` và interface `SortedMap`.

![Sơ đồ kế thừa TreeMap](https://oss.javaguide.cn/github/javaguide/java/collection/treemap_hierarchy.png)

Việc triển khai interface `NavigableMap` giúp `TreeMap` có khả năng tìm kiếm các phần tử trong collection.

Interface `NavigableMap` cung cấp các phương thức phong phú để khám phá và thao tác trên các cặp key-value:

1. **Tìm kiếm định hướng**: Các phương thức `ceilingEntry()`, `floorEntry()`, `higherEntry()` và `lowerEntry()` có thể dùng để định vị cặp key-value gần nhất lớn hơn hoặc bằng, nhỏ hơn hoặc bằng, lớn hơn hẳn, nhỏ hơn hẳn key cho trước.
2. **Thao tác tập hợp con**: Các phương thức `subMap()`, `headMap()` và `tailMap()` có thể tạo ra view tập hợp con của collection ban đầu một cách hiệu quả mà không cần copy toàn bộ collection.
3. **View đảo ngược**: Phương thức `descendingMap()` trả về một view `NavigableMap` đảo ngược, giúp có thể lặp ngược toàn bộ `TreeMap`.
4. **Thao tác ranh giới**: Các phương thức `firstEntry()`, `lastEntry()`, `pollFirstEntry()` và `pollLastEntry()` có thể dễ dàng truy cập và loại bỏ phần tử.

Các phương thức này đều dựa trên đặc tính của cấu trúc dữ liệu cây đỏ đen để triển khai. Cây đỏ đen duy trì trạng thái cân bằng, từ đó đảm bảo độ phức tạp thời gian của thao tác tìm kiếm là O(log n), điều này biến `TreeMap` thành công cụ mạnh mẽ để xử lý bài toán tìm kiếm trên các tập hợp có thứ tự.

Việc triển khai interface `SortedMap` giúp `TreeMap` có khả năng sắp xếp các phần tử trong collection dựa theo key. Mặc định là sắp xếp tăng dần theo key, tuy nhiên chúng ta cũng có thể chỉ định `Comparator` để sắp xếp. Ví dụ code như sau:

```java
/**
 * @author shuang.kou
 * @createTime 2020年06月15日 17:02:00
 */
public class Person {
    private Integer age;

    public Person(Integer age) {
        this.age = age;
    }

    public Integer getAge() {
        return age;
    }


    public static void main(String[] args) {
        TreeMap<Person, String> treeMap = new TreeMap<>(new Comparator<Person>() {
            @Override
            public int compare(Person person1, Person person2) {
                int num = person1.getAge() - person2.getAge();
                return Integer.compare(num, 0);
            }
        });
        treeMap.put(new Person(3), "person1");
        treeMap.put(new Person(18), "person2");
        treeMap.put(new Person(35), "person3");
        treeMap.put(new Person(16), "person4");
        treeMap.entrySet().stream().forEach(personStringEntry -> {
            System.out.println(personStringEntry.getValue());
        });
    }
}
```

Output:

```plain
person1
person4
person2
person3
```

Có thể thấy, các phần tử trong `TreeMap` đã được sắp xếp tăng dần theo trường `age` của `Person`.

Ở trên, chúng ta đã triển khai bằng cách truyền vào một anonymous inner class, bạn có thể thay thế code bằng cách triển khai Lambda expression:

```java
TreeMap<Person, String> treeMap = new TreeMap<>((person1, person2) -> {
  int num = person1.getAge() - person2.getAge();
  return Integer.compare(num, 0);
});
```

**Tóm lại, so với `HashMap`, `TreeMap` chủ yếu có thêm khả năng sắp xếp các phần tử trong collection theo key và khả năng tìm kiếm các phần tử bên trong collection.**

### HashSet kiểm tra trùng lặp như thế nào?

Nội dung dưới đây trích từ cuốn sách nhập môn Java của tôi - 《Head First Java》phiên bản 2:

> Khi bạn thêm một đối tượng vào `HashSet`, `HashSet` trước tiên sẽ tính giá trị `hashCode` của đối tượng để xác định vị trí thêm vào, đồng thời cũng so sánh với giá trị `hashCode` của các đối tượng đã được thêm vào trước đó. Nếu không có `hashCode` trùng khớp, `HashSet` sẽ giả định đối tượng không bị trùng lặp. Nhưng nếu phát hiện đối tượng có cùng giá trị `hashCode`, lúc này phương thức `equals()` sẽ được gọi để kiểm tra xem các đối tượng có cùng `hashCode` đó có thực sự giống nhau hay không. Nếu cả hai giống nhau, `HashSet` sẽ không cho phép thao tác thêm thành công.

Trong JDK1.8, phương thức `add()` của `HashSet` chỉ đơn giản gọi phương thức `put()` của `HashMap`, và kiểm tra giá trị trả về để đảm bảo xem có phần tử trùng lặp hay không. Hãy xem trực tiếp source code trong `HashSet`:

```java
// Returns: true if this set did not already contain the specified element
// Giá trị trả về: Trả về true khi set chưa chứa phần tử add
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}
```

Còn trong phương thức `putVal()` của `HashMap` cũng có thể thấy phần giải thích như sau:

```java
// Returns : previous value, or null if none
// Giá trị trả về: Nếu vị trí chèn chưa có phần tử thì trả về null, ngược lại trả về phần tử trước đó
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
...
}
```

Nghĩa là trong JDK1.8, nếu trong `HashSet` đã tồn tại phần tử giống nhau, `HashMap` bên dưới sẽ giữ lại key ban đầu, `HashSet#add()` trả về `false`; chỉ khi chưa tồn tại thì mới thêm phần tử mới và trả về `true`.

### ⭐️ Triển khai bên dưới của HashMap

#### Trước JDK1.8

Trước JDK1.8, bên dưới `HashMap` là sự kết hợp giữa **mảng và danh sách liên kết**, còn gọi là **băm chuỗi (chaining hash)**. `HashMap` thông qua `hashCode` của key trải qua xử lý của hàm nhiễu (perturbation function) thu được giá trị hash, sau đó thông qua `(n - 1) & hash` để xác định vị trí lưu trữ của phần tử hiện tại (n ở đây là chiều dài mảng). Nếu vị trí hiện tại đã có phần tử, sẽ kiểm tra xem giá trị hash và key của phần tử đó có giống với phần tử sắp lưu vào hay không. Nếu giống nhau sẽ ghi đè trực tiếp, nếu không giống sẽ giải quyết xung đột bằng phương pháp chaining.

Hàm nhiễu (phương thức `hash`) trong `HashMap` được dùng để tối ưu hóa sự phân bố giá trị hash. Thông qua việc xử lý bổ sung đối với `hashCode()` ban đầu, hàm nhiễu có thể giảm thiểu va chạm do việc triển khai `hashCode()` kém chất lượng gây ra, từ đó nâng cao tính phân bố đều của dữ liệu.

**Source code phương thức hash của HashMap trong JDK1.8:**

Phương thức hash trong JDK 1.8 đơn giản hơn so với phương thức hash trong JDK 1.7, nhưng nguyên lý không đổi.

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

#### Từ JDK1.8 trở đi

So với các phiên bản trước, từ JDK1.8 trở đi việc giải quyết xung đột hash có sự thay đổi lớn: khi độ dài danh sách liên kết lớn hơn ngưỡng (mặc định là 8) (trước khi chuyển danh sách liên kết thành cây đỏ đen sẽ kiểm tra, nếu độ dài mảng hiện tại nhỏ hơn 64 thì sẽ chọn mở rộng mảng trước chứ không chuyển thành cây đỏ đen), danh sách liên kết sẽ được chuyển đổi thành cây đỏ đen.

Mục đích của việc làm này là để giảm thời gian tìm kiếm: hiệu năng truy vấn của danh sách liên kết là O(n) (n là độ dài danh sách liên kết), cây đỏ đen là một loại cây nhị phân tìm kiếm tự cân bằng có hiệu năng truy vấn là O(log n). Khi danh sách liên kết ngắn, sự khác biệt hiệu năng giữa O(n) và O(log n) là không đáng kể. Nhưng khi danh sách liên kết trở nên dài, hiệu năng truy vấn sẽ giảm sút rõ rệt.

![Cấu trúc bên trong của HashMap từ JDK1.8 trở đi](https://oss.javaguide.cn/github/javaguide/java/collection/jdk1.8_hashmap.png)

**Tại sao lại ưu tiên mở rộng mảng (扩容) chứ không chuyển trực tiếp thành cây đỏ đen?**

Mở rộng mảng có thể giảm xác suất xảy ra xung đột hash (tức là phân tán lại các phần tử sang mảng mới lớn hơn), điều này trong đa số trường hợp hiệu quả hơn so với việc chuyển trực tiếp sang cây đỏ đen.

Cây đỏ đen cần duy trì tự cân bằng, chi phí bảo trì tương đối cao. Hơn nữa, việc đưa cây đỏ đen vào quá sớm sẽ làm tăng độ phức tạp không cần thiết.

**Tại sao lại chọn ngưỡng là 8 và 64?**

1. Phân phối Poisson chỉ ra rằng xác suất độ dài danh sách liên kết đạt 8 là cực kỳ thấp (nhỏ hơn một phần chục triệu). Trong tuyệt đại đa số trường hợp, độ dài danh sách liên kết đều không vượt quá 8. Ngưỡng được thiết lập là 8 có thể đảm bảo sự cân bằng giữa hiệu năng và hiệu quả bộ nhớ.
2. Ngưỡng chiều dài mảng 64 cũng là giá trị kinh nghiệm đã qua kiểm chứng thực tế. Trong mảng nhỏ, chi phí mở rộng mảng thấp, ưu tiên mở rộng mảng giúp tránh đưa cây đỏ đen vào quá sớm. Khi kích thước mảng đạt đến 64, xác suất xung đột tương đối cao, lúc này ưu thế hiệu năng của cây đỏ đen mới bắt đầu thể hiện.

> `TreeMap`, `TreeSet` và `HashMap` từ JDK1.8 trở đi bên dưới đều sử dụng cây đỏ đen. Cây đỏ đen ra đời nhằm khắc phục nhược điểm của cây nhị phân tìm kiếm (BST), vì cây nhị phân tìm kiếm trong một số trường hợp sẽ thoái hóa thành cấu trúc tuyến tính.

Chúng ta hãy cùng kết hợp source code để phân tích quá trình chuyển đổi từ danh sách liên kết sang cây đỏ đen trong `HashMap`.

**1. Logic kiểm tra chuyển danh sách liên kết thành cây đỏ đen trong phương thức putVal.**

Khi độ dài danh sách liên kết lớn hơn 8, sẽ thực thi logic `treeifyBin` (chuyển đổi thành cây đỏ đen).

```java
// Duyệt danh sách liên kết
for (int binCount = 0; ; ++binCount) {
    // Duyệt đến node cuối cùng của danh sách liên kết
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        // Nếu số lượng phần tử trong danh sách liên kết lớn hơn TREEIFY_THRESHOLD (8)
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            // Chuyển đổi cây đỏ đen (và không trực tiếp chuyển thành cây đỏ đen ngay)
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

**2. Kiểm tra xem có thực sự chuyển thành cây đỏ đen hay không trong phương thức treeifyBin.**

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // Kiểm tra xem độ dài mảng hiện tại có nhỏ hơn 64 hay không
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        // Nếu độ dài mảng hiện tại nhỏ hơn 64 thì chọn mở rộng mảng trước
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // Ngược lại mới chuyển danh sách thành cây đỏ đen

        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

Trước khi chuyển danh sách liên kết thành cây đỏ đen sẽ kiểm tra, nếu độ dài mảng hiện tại nhỏ hơn 64 thì sẽ chọn mở rộng mảng trước chứ không chuyển thành cây đỏ đen.

### ⭐️ Tại sao chiều dài của HashMap lại là lũy thừa của 2?

Để việc lưu trữ và truy xuất trong `HashMap` đạt hiệu quả cao và giảm thiểu va chạm, chúng ta cần đảm bảo dữ liệu được phân bố càng đều càng tốt. Giá trị hash trong Java thường dùng kiểu `int`, phạm vi từ `-2147483648 ~ 2147483647`, tổng cộng khoảng 4 tỷ không gian ánh xạ. Chỉ cần hàm hash ánh xạ tương đối đều và thưa thì ứng dụng thông thường rất khó xảy ra va chạm. Tuy nhiên, vấn đề là một mảng có chiều dài 4 tỷ thì bộ nhớ không thể chứa nổi. Vì vậy, giá trị băm này không thể trực tiếp lấy ra dùng ngay. Trước khi dùng còn phải thực hiện phép chia lấy dư (mod) cho chiều dài mảng, phần dư thu được mới dùng làm vị trí cần lưu trữ, tức là index tương ứng của mảng.

**Thuật toán này nên được thiết kế như thế nào?**

Đầu tiên chúng ta có thể nghĩ đến việc sử dụng phép chia lấy dư `%` để thực hiện. Đối với `hash` không âm, khi `length` là lũy thừa của 2, `hash % length` tương đương với `hash & (length - 1)`. `HashMap` sử dụng cách thứ hai để tính index của mảng.

Ngoài lý do phép toán bit (bitwise) ở trên có hiệu năng cao hơn phép chia lấy dư, tôi nghĩ một lý do quan trọng hơn là: **Chiều dài là lũy thừa của 2 giúp `HashMap` khi mở rộng dung lượng (扩容) phân bố đều hơn**. Ví dụ:

- Khi length = 8, length - 1 = 7 có dạng nhị phân là `0111`
- Khi length = 16, length - 1 = 15 có dạng nhị phân là `1111`

Lúc này các phần tử vốn có trong `HashMap` khi tính vị trí mảng mới `hash & (length - 1)` sẽ phụ thuộc vào bit nhị phân thứ 4 của hash (tính từ phải sang), sẽ xảy ra 2 trường hợp:

1. Bit nhị phân thứ 4 bằng 0, vị trí trong mảng không đổi, nghĩa là vị trí phần tử hiện tại trong mảng mới và mảng cũ là giống nhau.
2. Bit nhị phân thứ 4 bằng 1, vị trí trong mảng thuộc về phần không gian mới mở rộng thêm của mảng mới.

Dưới đây là một ví dụ minh họa:

```plain
Giả sử có một phần tử có giá trị hash là 10101100

Tính vị trí phần tử trong mảng cũ:
hash        = 10101100
length - 1  = 00000111
& -----------------
index       = 00000100  (4)

Tính vị trí phần tử trong mảng mới:
hash        = 10101100
length - 1  = 00001111
& -----------------
index       = 00001100  (12)

Xét bit thứ 4 (tính từ phải sang):
1. Bit cao bằng 0: Vị trí không đổi.
2. Bit cao bằng 1: Di chuyển sang vị trí mới (vị trí index cũ + dung lượng cũ).
```

⚠️ Lưu ý: Kịch bản được liệt kê ở đây xem xét bit nhị phân thứ 4, nói chính xác hơn là xem xét bit cao (tính từ phải sang), ví dụ khi `length = 32`, `length - 1 = 31`, nhị phân là `11111`, ở đây sẽ xem xét bit nhị phân thứ 5.

Nghĩa là sau khi mở rộng dung lượng, trong trường hợp giá trị hash của phần tử mảng cũ tương đối đều (giá trị hash có đều hay không phụ thuộc vào phương thức `hashCode()` của đối tượng và hàm nhiễu đã nói ở trước), các phần tử mảng mới cũng sẽ được phân bổ tương đối đều, trường hợp tốt nhất là một nửa ở phần đầu mảng mới và một nửa ở phần sau mảng mới.

Điều này cũng giúp cơ chế mở rộng dung lượng trở nên đơn giản và hiệu quả. Sau khi mở rộng chỉ cần kiểm tra sự thay đổi bit cao của giá trị hash để quyết định vị trí mới của phần tử: hoặc là vị trí không đổi (bit cao bằng 0), hoặc là di chuyển sang vị trí mới (bit cao bằng 1, vị trí index cũ + dung lượng cũ).

Cuối cùng, tổng kết đơn giản về lý do chiều dài của `HashMap` là lũy thừa của 2:

1. Hiệu năng phép toán bit cao hơn: Phép toán bit (`&`) hiệu quả hơn phép chia lấy dư (`%`). Đối với `hash` không âm, khi chiều dài là lũy thừa của 2, `hash % length` tương đương với `hash & (length - 1)`.
2. Đảm bảo sự phân bố đều của giá trị hash tốt hơn: Sau khi mở rộng, trong trường hợp giá trị hash của mảng cũ tương đối đều, các phần tử mảng mới cũng được phân bổ tương đối đều, tốt nhất là một nửa ở nửa đầu mảng mới, một nửa ở nửa sau mảng mới.
3. Cơ chế mở rộng trở nên đơn giản và hiệu quả: Sau khi mở rộng chỉ cần kiểm tra sự thay đổi của bit cao trong giá trị hash để quyết định vị trí mới của phần tử: hoặc vị trí không đổi (bit cao bằng 0), hoặc di chuyển sang vị trí mới (bit cao bằng 1, vị trí index cũ + dung lượng cũ).

### ⭐️ Vấn đề vòng lặp vô tận do thao tác đa luồng trong HashMap

Trong JDK1.7 trở về trước, thao tác mở rộng mảng (扩容) của `HashMap` trong môi trường đa luồng có thể gặp vấn đề vòng lặp vô tận (dead loop). Điều này xảy ra khi một bucket có nhiều phần tử cần mở rộng dung lượng, nhiều thread đồng thời thao tác trên danh sách liên kết, phương pháp chèn ở đầu (head insertion) có thể dẫn đến việc các node trong danh sách liên kết trỏ sai vị trí, tạo thành danh sách liên kết vòng (circular linked list), từ đó khiến thao tác truy vấn phần tử rơi vào vòng lặp vô tận không thể kết thúc.

Để giải quyết vấn đề này, `HashMap` phiên bản JDK1.8 sử dụng phương pháp chèn ở cuối (tail insertion) thay vì chèn ở đầu để tránh đảo ngược danh sách liên kết, giúp các node được chèn luôn nằm ở cuối danh sách liên kết, tránh được cấu trúc vòng trong danh sách liên kết. Tuy nhiên vẫn không khuyến nghị sử dụng `HashMap` trong môi trường đa luồng, vì trong đa luồng sử dụng `HashMap` vẫn tồn tại vấn đề ghi đè dữ liệu. Trong môi trường concurrency, khuyến nghị sử dụng `ConcurrentHashMap`.

Trong phỏng vấn thông thường giới thiệu như vậy là tương đối đầy đủ, không cần nhớ các chi tiết rườm rà. Nếu muốn tìm hiểu chi tiết vấn đề mở rộng `HashMap` dẫn đến vòng lặp vô tận, bạn có thể xem bài viết này của tác giả Hạo Tử Thúc: [Vòng lặp vô tận của Java HashMap](https://coolshell.cn/articles/9606.html).

### ⭐️ Tại sao HashMap lại không thread-safe?

`HashMap` không thread-safe. Trong môi trường đa luồng, thao tác ghi đồng thời (concurrent write) vào `HashMap` có thể dẫn đến 2 vấn đề chính:

1. **Mất dữ liệu**: Thao tác `put` đồng thời có thể khiến việc ghi dữ liệu của một thread bị một thread khác ghi đè.
2. **Vòng lặp vô tận**: Trong JDK 7 trở về trước, khi mở rộng dung lượng đồng thời, do phương pháp chèn ở đầu có thể khiến danh sách liên kết tạo thành vòng, từ đó dẫn đến vòng lặp vô tận khi thực hiện thao tác `get`, làm CPU tăng vọt lên 100%.

Vấn đề mất dữ liệu này tồn tại ở cả JDK 1.7 và JDK 1.8, ở đây lấy JDK 1.8 làm ví dụ giới thiệu.

Từ JDK 1.8 trở đi, trong `HashMap`, nhiều cặp key-value có thể được phân bổ vào cùng một bucket và lưu trữ dưới dạng danh sách liên kết hoặc cây đỏ đen. Thao tác `put` của nhiều thread vào `HashMap` sẽ dẫn đến không an toàn luồng, cụ thể sẽ có rủi ro bị ghi đè dữ liệu.

Lấy một ví dụ:

- Hai thread 1, 2 đồng thời thực hiện thao tác `put`, và xảy ra xung đột hash (chiều dài index chèn do hàm hash tính ra là giống nhau).
- Các thread khác nhau có thể nhận được cơ hội thực thi CPU trong các time slice khác nhau. Sau khi thread 1 hiện tại thực hiện xong kiểm tra xung đột hash thì bị treo do hết time slice. Thread 2 hoàn thành thao tác chèn trước.
- Sau đó thread 1 nhận được time slice, do trước đó đã thực hiện kiểm tra va chạm hash nên lúc này sẽ trực tiếp thực hiện chèn, điều này khiến dữ liệu thread 2 vừa chèn bị thread 1 ghi đè.

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    // ...
    // Kiểm tra xem có xuất hiện va chạm hash hay không
    // (n - 1) & hash xác định phần tử được đặt trong bucket nào, nếu bucket rỗng, tạo node mới đưa vào bucket (lúc này node này được đặt trong mảng)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // Trong bucket đã tồn tại phần tử (xử lý xung đột hash)
    else {
    // ...
}
```

Còn một trường hợp nữa là hai thread này đồng thời `put` làm cho giá trị của `size` không chính xác:

1. Thread 1 khi thực hiện kiểm tra `if(++size > threshold)`, giả sử lấy được giá trị `size` là 10, bị treo do hết time slice.
2. Thread 2 cũng thực hiện kiểm tra `if(++size > threshold)`, lấy được giá trị `size` cũng là 10, và chèn phần tử vào vị trí bucket đó, rồi cập nhật giá trị `size` thành 11.
3. Sau đó thread 1 nhận được time slice, nó cũng đưa phần tử vào vị trí bucket, và cập nhật giá trị `size` thành 11.
4. Cả thread 1 và 2 đều đã thực hiện 1 thao tác `put`, nhưng giá trị `size` chỉ tăng thêm 1. Lúc này đếm số lượng xảy ra lỗi mất cập nhật (lost update), nhưng không thể chỉ dựa vào kết quả của `size` để đánh giá số lượng phần tử thực tế đã chèn.

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    // ...
    // Kích thước thực tế lớn hơn ngưỡng thì mở rộng dung lượng
    if (++size > threshold)
        resize();
    // Callback sau khi chèn
    afterNodeInsertion(evict);
    return null;
}
```

### Các cách duyệt phổ biến của HashMap?

[7 cách duyệt HashMap và phân tích hiệu năng!](https://mp.weixin.qq.com/s/zQBN3UvJDhRTKP6SzcZFKw)

**🐛 Sửa lỗi (Xem thêm: [issue#1411](https://github.com/Snailclimb/JavaGuide/issues/1411))**:

Bài viết này phân tích hiệu năng của cách duyệt `parallelStream` bị sai, xin nêu kết luận trước: **Khi có blocking thì parallelStream có hiệu năng cao nhất, khi không blocking thì parallelStream có hiệu năng thấp nhất**.

Khi việc lặp không tồn tại blocking, hiệu năng của `parallelStream` là thấp nhất:

```plain
Benchmark               Mode  Cnt     Score      Error  Units
Test.entrySet           avgt    5   288.651 ±   10.536  ns/op
Test.keySet             avgt    5   584.594 ±   21.431  ns/op
Test.lambda             avgt    5   221.791 ±   10.198  ns/op
Test.parallelStream     avgt    5  6919.163 ± 1116.139  ns/op
```

Sau khi thêm code blocking `Thread.sleep(10)`, hiệu năng của `parallelStream` mới là cao nhất:

```plain
Benchmark               Mode  Cnt           Score          Error  Units
Test.entrySet           avgt    5  1554828440.000 ± 23657748.653  ns/op
Test.keySet             avgt    5  1550612500.000 ±  6474562.858  ns/op
Test.lambda             avgt    5  1551065180.000 ± 19164407.426  ns/op
Test.parallelStream     avgt    5   186345456.667 ±  3210435.590  ns/op
```

### ⭐️ Sự khác biệt giữa ConcurrentHashMap và Hashtable

Sự khác biệt giữa `ConcurrentHashMap` và `Hashtable` chủ yếu thể hiện ở phương thức triển khai thread-safe khác nhau.

- **Cấu trúc dữ liệu bên dưới:** `ConcurrentHashMap` trong JDK1.7 bên dưới sử dụng **mảng phân đoạn (Segment) + danh sách liên kết**, trong JDK1.8 cấu trúc dữ liệu giống như `HashMap`: mảng + danh sách liên kết / cây nhị phân đỏ đen. `Hashtable` và `HashMap` trước JDK1.8 có cấu trúc dữ liệu bên dưới tương tự nhau, đều sử dụng dạng **mảng + danh sách liên kết**, mảng là thành phần chính, danh sách liên kết chủ yếu tồn tại để giải quyết xung đột hash;
- **Cách thức triển khai thread-safe (Quan trọng):**
  - Trong JDK1.7, `ConcurrentHashMap` chia phân đoạn mảng bucket (`Segment`, khóa phân đoạn / lock segmentation), mỗi khóa chỉ lock một phần dữ liệu trong container (có hình minh họa ở dưới). Nhiều thread truy cập dữ liệu ở các đoạn dữ liệu khác nhau trong container sẽ không xảy ra tranh chấp lock, giúp nâng cao tỷ lệ truy cập đồng thời (concurrency rate).
  - Đến JDK1.8, `ConcurrentHashMap` đã hủy bỏ khái niệm `Segment`, mà trực tiếp sử dụng cấu trúc dữ liệu mảng Node + danh sách liên kết + cây đỏ đen để triển khai, kiểm soát concurrency bằng `synchronized` và CAS. (Từ JDK1.6 trở đi lock `synchronized` đã được tối ưu hóa rất nhiều). Nhìn tổng thể nó giống như một `HashMap` được tối ưu hóa và thread-safe, mặc dù trong JDK1.8 vẫn có thể thấy cấu trúc dữ liệu `Segment`, nhưng đã tối giản thuộc tính, chỉ dùng để tương thích với các phiên bản cũ;
  - **Hashtable (Dùng chung một lock)**: Sử dụng `synchronized` để đảm bảo thread-safe, hiệu năng rất thấp. Khi một thread truy cập phương thức đồng bộ, các thread khác truy cập phương thức đồng bộ cũng có thể rơi vào trạng thái block hoặc polling. Ví dụ khi một thread dùng `put` để thêm phần tử, thread khác không thể dùng `put` để thêm phần tử và cũng không thể dùng `get`, tranh chấp ngày càng gay gắt dẫn đến hiệu năng ngày càng thấp.

Dưới đây, chúng ta hãy cùng xem hình so sánh cấu trúc dữ liệu bên dưới của cả hai.

**Hashtable** :

![Cấu trúc bên trong của Hashtable](https://oss.javaguide.cn/github/javaguide/java/collection/jdk1.7_hashmap.png)

<p style="text-align:right;font-size:13px;color:gray">https://www.cnblogs.com/chengxiao/p/6842045.html></p>

**JDK1.7 ConcurrentHashMap**：

![Cấu trúc lưu trữ của Java7 ConcurrentHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/java7_concurrenthashmap.png)

`ConcurrentHashMap` được cấu thành từ cấu trúc mảng `Segment` và cấu trúc mảng `HashEntry`.

Mỗi phần tử trong mảng `Segment` chứa một mảng `HashEntry`, mỗi mảng `HashEntry` thuộc về cấu trúc danh sách liên kết.

**JDK1.8 ConcurrentHashMap**：

![Cấu trúc lưu trữ của Java8 ConcurrentHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/java8_concurrenthashmap.png)

JDK1.8 `ConcurrentHashMap` không còn là **mảng Segment + mảng HashEntry + danh sách liên kết**, mà là **mảng Node + danh sách liên kết / cây đỏ đen**. Tuy nhiên, Node chỉ dùng cho trường hợp danh sách liên kết, trường hợp cây đỏ đen cần sử dụng **`TreeNode`**. Khi danh sách liên kết xung đột đạt đến độ dài nhất định, danh sách liên kết sẽ chuyển đổi thành cây đỏ đen.

`TreeNode` lưu trữ các node cây đỏ đen, được bọc bởi `TreeBin`. `TreeBin` thông qua thuộc tính `root` để duy trì node gốc của cây đỏ đen, vì khi cây đỏ đen xoay (rotate), node gốc có thể bị node con ban đầu của nó thay thế, tại thời điểm này nếu có thread khác muốn ghi vào cây đỏ đen thì sẽ xảy ra vấn đề không an toàn luồng, do đó trong `ConcurrentHashMap`, `TreeBin` thông qua thuộc tính `waiter` duy trì thread hiện đang sử dụng cây đỏ đen này để ngăn chặn thread khác đi vào.

```java
static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
        // values for lockState
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock
...
}
```

### ⭐️ Cách thức triển khai thread-safe cụ thể / Triển khai cụ thể bên dưới của ConcurrentHashMap

#### Trước JDK1.8

![Cấu trúc lưu trữ của Java7 ConcurrentHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/java7_concurrenthashmap.png)

Trước tiên chia dữ liệu thành từng đoạn (đoạn này chính là `Segment`) để lưu trữ, sau đó trang bị một khóa (lock) cho mỗi đoạn dữ liệu. Khi một thread chiếm giữ lock để truy cập dữ liệu của một đoạn, dữ liệu ở các đoạn khác vẫn có thể được các thread khác truy cập.

`ConcurrentHashMap` được cấu thành từ cấu trúc mảng `Segment` và cấu trúc mảng `HashEntry`.

`Segment` kế thừa `ReentrantLock`, nên `Segment` là một loại reentrant lock (khóa có thể vào lại), đóng vai trò làm lock. `HashEntry` dùng để lưu trữ dữ liệu cặp key-value.

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

Một `ConcurrentHashMap` chứa một mảng `Segment`, số lượng `Segment` một khi được khởi tạo thì không thể thay đổi. Kích thước mảng `Segment` mặc định là 16, nghĩa là mặc định có thể hỗ trợ đồng thời 16 thread ghi đồng thời.

Cấu trúc của `Segment` tương tự như `HashMap`, là một cấu trúc mảng và danh sách liên kết, một `Segment` chứa một mảng `HashEntry`, mỗi `HashEntry` là một phần tử của cấu trúc danh sách liên kết, mỗi `Segment` bảo vệ các phần tử trong một mảng `HashEntry`. Khi sửa đổi dữ liệu trong mảng `HashEntry`, bắt buộc trước tiên phải lấy được lock của `Segment` tương ứng. Nghĩa là việc ghi đồng thời vào cùng một `Segment` sẽ bị block, còn việc ghi vào các `Segment` khác nhau có thể thực thi đồng thời.

#### Từ JDK1.8 trở đi

![Cấu trúc lưu trữ của Java8 ConcurrentHashMap](https://oss.javaguide.cn/github/javaguide/java/collection/java8_concurrenthashmap.png)

Java 8 hầu như đã viết lại hoàn toàn `ConcurrentHashMap`, số lượng dòng code từ hơn 1000 dòng trong Java 7 đã tăng lên hơn 6000 dòng hiện tại.

`ConcurrentHashMap` đã hủy bỏ lock phân đoạn Segment, áp dụng `Node + CAS + synchronized` để đảm bảo an toàn concurrency. Cấu trúc dữ liệu tương tự cấu trúc `HashMap` 1.8: mảng + danh sách liên kết / cây nhị phân đỏ đen. Java 8 khi độ dài danh sách liên kết vượt quá ngưỡng nhất định (8) sẽ chuyển danh sách liên kết (độ phức tạp thời gian định vị O(N)) thành cây đỏ đen (độ phức tạp thời gian định vị O(log(N))).

Trong Java 8, độ mịn của lock tỉ mỉ hơn (fine-grained), khi cập nhật bucket không rỗng thường dùng `synchronized` để lock node đầu tiên của bucket. Thao tác cập nhật trên các bucket khác nhau thường có thể thực thi song song, thao tác đọc cũng không sử dụng các lock bucket này.

### ⭐️ Triển khai ConcurrentHashMap trong JDK 1.7 và JDK 1.8 khác nhau như thế nào?

- **Cách thức triển khai thread-safe**: JDK 1.7 áp dụng lock phân đoạn `Segment` để đảm bảo an toàn, `Segment` kế thừa từ `ReentrantLock`. JDK1.8 từ bỏ thiết kế lock phân đoạn `Segment`, áp dụng `Node + CAS + synchronized` để đảm bảo thread-safe, độ mịn lock nhỏ hơn, `synchronized` chỉ lock node đầu của danh sách liên kết hiện tại hoặc cây nhị phân đỏ đen.
- **Phương pháp giải quyết va chạm Hash**: JDK 1.7 áp dụng phương pháp chaining, JDK1.8 áp dụng phương pháp chaining kết hợp cây đỏ đen (khi độ dài danh sách liên kết vượt quá ngưỡng nhất định thì chuyển danh sách liên kết thành cây đỏ đen).
- **Mức độ concurrency**: Cập nhật đồng thời trong JDK 1.7 chủ yếu bị giới hạn bởi số lượng `Segment`, mặc định là 16. JDK 1.8 không còn sử dụng số lượng `Segment` cố định, cập nhật trên các bucket khác nhau thường có thể thực thi song song.

### Tại sao key và value trong ConcurrentHashMap không thể là null?

Key và value trong `ConcurrentHashMap` không thể là null chủ yếu là để tránh sự mơ hồ (ambiguity / 二义性). Null là một giá trị đặc biệt, biểu thị không có đối tượng hoặc không có tham chiếu. Nếu bạn dùng null làm key, bạn không thể phân biệt được key này có tồn tại trong `ConcurrentHashMap` hay là hoàn toàn không có key đó. Tương tự, nếu bạn dùng null làm value, bạn không thể phân biệt được value này là thực sự được lưu trữ trong `ConcurrentHashMap` hay là do không tìm thấy key tương ứng mà trả về.

Lấy việc lấy giá trị qua phương thức get làm ví dụ, kết quả trả về là null tồn tại 2 trường hợp:

- Giá trị không nằm trong collection;
- Bản thân giá trị chính là null.

Đây chính là nguồn gốc của sự mơ hồ (ambiguity).

Chi tiết có thể tham khảo [Phân tích source code ConcurrentHashMap](https://javaguide.cn/java/collection/concurrent-hash-map-source-code.html).

Trong môi trường đa luồng, tồn tại trường hợp khi một thread đang thao tác trên `ConcurrentHashMap` thì các thread khác sửa đổi `ConcurrentHashMap` đó, vì vậy không thể thông qua `containsKey(key)` để kiểm tra xem có tồn tại cặp key-value này hay không, do đó cũng không có cách nào giải quyết vấn đề mơ hồ.

Trái ngược với điều đó, `HashMap` có thể lưu trữ key và value là null, nhưng null làm key chỉ có thể có một, null làm value có thể có nhiều. Nếu truyền null làm tham số, nó sẽ trả về giá trị tại vị trí có giá trị hash bằng 0. Trong môi trường đơn luồng, không tồn tại trường hợp một thread thao tác trên `HashMap` mà các thread khác sửa đổi `HashMap` đó, do đó có thể thông qua `containsKey(key)` để kiểm tra xem có tồn tại cặp key-value này hay không để xử lý tương ứng, do đó không tồn tại vấn đề mơ hồ.

Nghĩa là, dưới đa luồng không thể phán đoán chính xác cặp key-value có tồn tại hay không (tồn tại trường hợp thread khác sửa đổi), còn đơn luồng thì có thể (không tồn tại trường hợp thread khác sửa đổi).

Nếu bạn thực sự cần sử dụng null trong `ConcurrentHashMap`, bạn có thể sử dụng một đối tượng rỗng tĩnh đặc biệt để thay thế null.

```java
public static final Object NULL = new Object();
```

Cuối cùng, xin chia sẻ câu trả lời của chính tác giả `ConcurrentHashMap` (Doug Lea) về vấn đề này:

> The main reason that nulls aren't allowed in ConcurrentMaps (ConcurrentHashMaps, ConcurrentSkipListMaps) is that ambiguities that may be just barely tolerable in non-concurrent maps can't be accommodated. The main one is that if `map.get(key)` returns `null`, you can't detect whether the key explicitly maps to `null` vs the key isn't mapped. In a non-concurrent map, you can check this via `map.contains(key)`, but in a concurrent one, the map might have changed between calls.

Sau khi dịch ra, ý chính vẫn là đơn luồng có thể dung thứ sự mơ hồ, còn đa luồng thì không thể dung thứ.

### ⭐️ ConcurrentHashMap có thể đảm bảo tính nguyên tử của các thao tác phức hợp (composite operations) không?

`ConcurrentHashMap` là thread-safe, nghĩa là nó có thể đảm bảo khi nhiều thread đồng thời thực hiện thao tác đọc/ghi trên nó thì sẽ không xảy ra tình trạng dữ liệu không nhất quán, cũng không dẫn đến vấn đề vòng lặp vô tận khi thao tác đa luồng trong `HashMap` phiên bản JDK1.7 trở về trước. Tuy nhiên, điều này không có nghĩa là nó có thể đảm bảo tất cả các thao tác phức hợp đều mang tính atomic (nguyên tử), tuyệt đối không được nhầm lẫn!

Thao tác phức hợp (composite operation) là thao tác được cấu thành từ nhiều thao tác cơ bản (như `put`, `get`, `remove`, `containsKey`,...), ví dụ như trước tiên kiểm tra xem key nào đó có tồn tại `containsKey(key)` hay không, sau đó dựa vào kết quả để thực hiện chèn hoặc cập nhật `put(key, value)`. Thao tác kiểu này trong quá trình thực thi có thể bị các thread khác ngắt quãng, dẫn đến kết quả không đúng như kỳ vọng.

Ví dụ, có hai thread A và B đồng thời thực hiện thao tác phức hợp trên `ConcurrentHashMap` như sau:

```java
// Thread A
if (!map.containsKey(key)) {
map.put(key, value);
}
// Thread B
if (!map.containsKey(key)) {
map.put(key, anotherValue);
}
```

Nếu thứ tự thực thi của thread A và B như thế này:

1. Thread A kiểm tra map không tồn tại key
2. Thread B kiểm tra map không tồn tại key
3. Thread B chèn (key, anotherValue) vào map
4. Thread A chèn (key, value) vào map

Khi đó kết quả cuối cùng là (key, value), chứ không phải (key, anotherValue) như kỳ vọng. Đây chính là vấn đề do tính phi nguyên tử của thao tác phức hợp gây ra.

**Vậy làm thế nào để đảm bảo tính nguyên tử của thao tác phức hợp trong ConcurrentHashMap?**

`ConcurrentHashMap` cung cấp một số phương thức thao tác phức hợp mang tính atomic như `putIfAbsent`, `compute`, `computeIfAbsent`, `computeIfPresent`, `merge`,... Các phương thức này đều có thể nhận một hàm làm tham số, dựa trên key và value cho trước để tính toán một value mới và cập nhật nó vào map.

Code ở trên có thể sửa lại thành:

```java
// Thread A
map.putIfAbsent(key, value);
// Thread B
map.putIfAbsent(key, anotherValue);
```

Hoặc:

```java
// Thread A
map.computeIfAbsent(key, k -> value);
// Thread B
map.computeIfAbsent(key, k -> anotherValue);
```

Nhiều bạn có thể nói, trường hợp này cũng có thể thêm lock đồng bộ mà! Đúng là có thể, nhưng không khuyến nghị sử dụng cơ chế đồng bộ dùng lock, vì nó đi ngược lại mục đích ban đầu khi sử dụng `ConcurrentHashMap`. Khi sử dụng `ConcurrentHashMap`, hãy cố gắng sử dụng các phương thức thao tác phức hợp nguyên tử này để đảm bảo tính atomic.

## Utility class Collections (Không quan trọng)

**Các phương thức thường dùng của utility class Collections**:

- Sắp xếp
- Thao tác tìm kiếm, thay thế
- Kiểm soát đồng bộ (Không khuyến nghị, khi cần kiểu collection thread-safe xin hãy cân nhắc sử dụng concurrent collection trong package JUC)

### Thao tác sắp xếp

```java
void reverse(List list)// Đảo ngược
void shuffle(List list)// Sắp xếp ngẫu nhiên
void sort(List list)// Sắp xếp tăng dần theo thứ tự tự nhiên
void sort(List list, Comparator c)// Sắp xếp tùy chỉnh, do Comparator kiểm soát logic sắp xếp
void swap(List list, int i , int j)// Đổi chỗ phần tử ở hai vị trí index
void rotate(List list, int distance)// Xoay (rotate). Khi distance là số dương, dịch chuyển tổng thể distance phần tử phía sau của list về đằng trước. Khi distance là số âm, dịch chuyển tổng thể distance phần tử phía trước của list về đằng sau
```

### Thao tác tìm kiếm, thay thế

```java
int binarySearch(List list, Object key)// Tìm kiếm nhị phân trên List, trả về index, chú ý List bắt buộc phải có thứ tự
int max(Collection coll)// Dựa trên thứ tự tự nhiên của phần tử, trả về phần tử lớn nhất. Tương tự int min(Collection coll)
int max(Collection coll, Comparator c)// Dựa trên sắp xếp tùy chỉnh, trả về phần tử lớn nhất, quy tắc sắp xếp do class Comparator kiểm soát. Tương tự int min(Collection coll, Comparator c)
void fill(List list, Object obj)// Thay thế tất cả phần tử trong list chỉ định bằng phần tử chỉ định
int frequency(Collection c, Object o)// Thống kê số lần xuất hiện của phần tử
int indexOfSubList(List list, List target)// Thống kê index lần xuất hiện đầu tiên của target trong list, không tìm thấy trả về -1, tương tự int lastIndexOfSubList(List source, list target)
boolean replaceAll(List list, Object oldVal, Object newVal)// Thay thế phần tử cũ bằng phần tử mới
```

### Kiểm soát đồng bộ

`Collections` cung cấp nhiều phương thức `synchronizedXxx()`, phương thức này có thể bọc collection chỉ định thành collection đồng bộ hóa luồng (thread-synchronized collection), từ đó giải quyết vấn đề thread-safe khi nhiều thread truy cập đồng thời vào collection.

Tất cả các truy cập đều phải được thực hiện thông qua wrapper trả về; khi dùng `Iterator`, `Spliterator` hoặc `Stream` để lặp, còn cần phải đồng bộ thủ công trên wrapper đó.

Chúng ta biết rằng `HashSet`, `TreeSet`, `ArrayList`, `LinkedList`, `HashMap`, `TreeMap` đều không thread-safe. `Collections` cung cấp nhiều phương thức static có thể bọc chúng thành các collection đồng bộ luồng.

**Tốt nhất không nên sử dụng các phương thức dưới đây vì hiệu năng rất thấp, khi cần kiểu collection thread-safe xin hãy cân nhắc sử dụng concurrent collection trong package JUC.**

Các phương thức như sau:

```java
synchronizedCollection(Collection<T>  c) // Trả về collection đồng bộ (thread-safe) được hỗ trợ bởi collection chỉ định.
synchronizedList(List<T> list)// Trả về List đồng bộ (thread-safe) được hỗ trợ bởi list chỉ định.
synchronizedMap(Map<K,V> m) // Trả về Map đồng bộ (thread-safe) được hỗ trợ bởi map chỉ định.
synchronizedSet(Set<T> s) // Trả về Set đồng bộ (thread-safe) được hỗ trợ bởi set chỉ định.
```

<!-- @include: @article-footer.snippet.md -->
