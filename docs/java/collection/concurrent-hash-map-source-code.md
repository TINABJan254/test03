---
title: ConcurrentHashMap 源码分析
description: ConcurrentHashMap源码深入解析：对比JDK1.7分段锁Segment与JDK1.8 CAS+Synchronized实现，理解高并发Map的线程安全机制与性能优化。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: ConcurrentHashMap源码,线程安全Map,分段锁Segment,CAS操作,并发容器,JDK7与JDK8区别
---

> Bài viết này đóng góp từ 末读代码: <https://mp.weixin.qq.com/s/AHWzboztt53ZfFZmsSnMSw>, JavaGuide đã tiến hành cải thiện và tối ưu hóa quy mô lớn trên bản gốc.

Bài viết trước đã giới thiệu source code `HashMap` nhận được phản hồi tốt và nhiều bạn học đã đưa ra quan điểm của mình. Lần này chúng ta tiếp tục với `ConcurrentHashMap`, với vai trò là một `HashMap` thread-safe, tần suất sử dụng của nó cũng rất cao. Vậy cấu trúc lưu trữ và nguyên lý triển khai của nó như thế nào?

## 1. ConcurrentHashMap 1.7

### 1. Cấu trúc lưu trữ

![Cấu trúc lưu trữ của ConcurrentHashMap Java 7](https://oss.javaguide.cn/github/javaguide/java/collection/java7_concurrenthashmap.png)

Cấu trúc lưu trữ của `ConcurrentHashMap` trong Java 7 như hình trên. `ConcurrentHashMap` được hợp thành bởi nhiều `Segment`, mà mỗi `Segment` lại là một cấu trúc tương tự `HashMap`, do đó bên trong mỗi `HashMap` có thể tiến hành mở rộng dung lượng (resize). Tuy nhiên số lượng `Segment` một khi **khởi tạo thì không thể thay đổi**, số lượng `Segment` mặc định là 16, do đó mặc định tối đa có thể có 16 segment đồng thời thực hiện thao tác cập nhật.

### 2. Khởi tạo

Thông qua constructor không tham số của `ConcurrentHashMap` để khám phá quy trình khởi tạo của `ConcurrentHashMap`.

```java
    /**
     * Creates a new, empty map with a default initial capacity (16),
     * load factor (0.75) and concurrencyLevel (16).
     */
    public ConcurrentHashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
    }
```

Constructor không tham số gọi constructor có tham số, truyền vào giá trị mặc định của 3 tham số.

```java
    /**
     * Dung lượng khởi tạo mặc định
     */
    static final int DEFAULT_INITIAL_CAPACITY = 16;

    /**
     * Hệ số tải (load factor) mặc định
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * Mức độ并发 (concurrency level) mặc định
     */
    static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

Tiếp theo cùng xem logic triển khai bên trong của constructor có tham số này.

```java
@SuppressWarnings("unchecked")
public ConcurrentHashMap(int initialCapacity,float loadFactor, int concurrencyLevel) {
    // Kiểm tra tham số
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    // Kiểm tra kích thước concurrency level, nếu lớn hơn 1<<16, đặt lại thành 65536
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    // Lũy thừa mấy của 2
    int sshift = 0;
    int ssize = 1;
    // Vòng lặp này có thể tìm giá trị lũy thừa của 2 gần nhất lớn hơn hoặc bằng concurrencyLevel
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // Ghi lại segment shift (độ lệch segment)
    this.segmentShift = 32 - sshift;
    // Ghi lại segment mask (mặt nạ segment)
    this.segmentMask = ssize - 1;
    // Thiết lập dung lượng
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // c = Dung lượng / ssize, mặc định 16 / 16 = 1, ở đây tính toán dung lượng tương tự HashMap trong mỗi Segment
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    // Dung lượng tương tự HashMap trong Segment ít nhất là 2 hoặc bội số của 2
    while (cap < c)
        cap <<= 1;
    // create segments and segments[0]
    // Tạo mảng Segment, thiết lập segments[0]
    Segment<K,V> s0 = new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

Tổng kết lại logic khởi tạo của ConcurrentHashMap trong Java 7:

1. Kiểm tra tham số cần thiết.
2. Kiểm tra kích thước `concurrencyLevel`, nếu lớn hơn giá trị tối đa thì đặt lại thành giá trị tối đa. Giá trị mặc định của constructor không tham số là **16**.
3. Tìm giá trị **lũy thừa của 2** gần nhất lớn hơn hoặc bằng `concurrencyLevel`, làm chiều dài mảng `segments`, **mặc định là 16**.
4. Ghi lại độ lệch `segmentShift`, giá trị này là **32 - sshift**, sẽ dùng khi tính toán vị trí lúc Put sau này, mặc định là 28.
5. Ghi lại `segmentMask`, mặc định là `ssize - 1 = 16 - 1 = 15`.
6. **Khởi tạo `segments[0]`**, **kích thước mặc định là 2**, **hệ số tải 0.75**, **ngưỡng mở rộng dung lượng là 2*0.75=1.5**, chèn giá trị thứ hai mới tiến hành mở rộng dung lượng.

### 3. put

Tiếp tục xem source code phương thức put dựa theo các tham số khởi tạo ở trên.

```java
/**
 * Maps the specified key to the specified value in this table.
 * Neither the key nor the value can be null.
 *
 * <p> The value can be retrieved by calling the <tt>get</tt> method
 * with a key that is equal to the original key.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>
 * @throws NullPointerException if the specified key or value is null
 */
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    // Giá trị hash dịch phải không dấu 28 bit (thu được lúc khởi tạo), sau đó thực hiện phép AND với segmentMask=15
    // Thực chất là lấy 4 bit cao thực hiện phép AND với segmentMask (1111)
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        // Nếu Segment tìm thấy là null, khởi tạo
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}

/**
 * Returns the segment for the given index, creating it and
 * recording in segment table (via CAS) if not already present.
 *
 * @param k the index
 * @return the segment
 */
@SuppressWarnings("unchecked")
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    // Kiểm tra Segment tại vị trí u có null hay không
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        Segment<K,V> proto = ss[0]; // use segment 0 as prototype
        // Lấy độ dài khởi tạo của HashEntry<K,V> trong segment 0
        int cap = proto.table.length;
        // Lấy load factor mở rộng dung lượng trong bảng hash của segment 0, tất cả loadFactor của các segment là như nhau
        float lf = proto.loadFactor;
        // Tính toán ngưỡng mở rộng dung lượng
        int threshold = (int)(cap * lf);
        // Tạo một mảng HashEntry có dung lượng cap
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) { // recheck
            // Kiểm tra lại Segment tại vị trí u có null hay không, vì lúc này có thể thread khác đã thao tác
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // Tự xoay (spin) kiểm tra Segment tại vị trí u có null hay không
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                // Sử dụng CAS để gán giá trị, chỉ thành công 1 lần
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```

Source code trên đã phân tích quy trình xử lý của `ConcurrentHashMap` khi put một dữ liệu, dưới đây tổng hợp lại quy trình cụ thể.

1. Tính toán vị trí key cần put, lấy `Segment` tại vị trí chỉ định.

2. Nếu `Segment` tại vị trí chỉ定 là null thì khởi tạo `Segment` này.

   **Quy trình khởi tạo Segment:**

   1. Kiểm tra `Segment` tại vị trí đã tính toán có phải null hay không.
   2. Nếu null tiếp tục khởi tạo, sử dụng dung lượng và load factor của `Segment[0]` để tạo mảng `HashEntry`.
   3. Kiểm tra lại `Segment` tại vị trí chỉ định đã tính toán có phải null hay không.
   4. Sử dụng mảng `HashEntry` đã tạo để khởi tạo Segment này.
   5. Spin kiểm tra `Segment` tại vị trí chỉ định đã tính toán có phải null hay không, sử dụng CAS để gán `Segment` vào vị trí này.

3. `Segment.put` chèn giá trị key, value.

Ở trên đã tìm hiểu thao tác lấy segment `Segment` và khởi tạo segment `Segment`. Phương thức put của `Segment` ở dòng cuối chưa xem qua, tiếp tục phân tích.

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // Lấy ReentrantLock exclusive lock, nếu không lấy được, dùng scanAndLockForPut để lấy.
    HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        // Tính toán vị trí dữ liệu cần put
        int index = (tab.length - 1) & hash;
        // CAS lấy giá trị tại tọa độ index
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                // Kiểm tra key đã tồn tại chưa, nếu tồn tại, duyệt danh sách liên kết tìm vị trí, tìm thấy thì thay thế value
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                // first có giá trị chứng tỏ vị trí index đã có giá trị, có xung đột, dùng phương pháp chèn ở đầu danh sách liên kết.
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                // Dung lượng lớn hơn ngưỡng mở rộng, nhỏ hơn dung lượng tối đa, thực hiện mở rộng dung lượng
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    // Gán node cho vị trí index, node có thể là một phần tử, cũng có thể là head của một danh sách liên kết
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}
```

Do `Segment` kế thừa `ReentrantLock`, nên bên trong `Segment` có thể lấy lock rất tiện lợi, quy trình put đã sử dụng tính năng này.

1. `tryLock()` để lấy lock, nếu không lấy được dùng **`scanAndLockForPut`** để tiếp tục lấy.

2. Tính toán vị trí index dữ liệu put cần đưa vào, sau đó lấy `HashEntry` tại vị trí này.

3. Duyệt put phần tử mới, tại sao phải duyệt? Vì `HashEntry` lấy ở đây có thể là một phần tử rỗng, cũng có thể danh sách liên kết đã tồn tại, nên phải đối xử khác nhau.

   Nếu **`HashEntry` tại vị trí này không tồn tại**:

   1. Nếu dung lượng hiện tại lớn hơn ngưỡng mở rộng, nhỏ hơn dung lượng tối đa, **tiến hành mở rộng dung lượng**.
   2. Trực tiếp chèn theo phương pháp chèn ở đầu (head insertion).

   Nếu **`HashEntry` tại vị trí này đã tồn tại**:

   1. Kiểm tra key và giá trị hash của phần tử hiện tại trong danh sách liên kết có trùng khớp với key và giá trị hash cần put hay không. Trùng khớp thì thay thế giá trị
   2. Không trùng khớp, lấy node tiếp theo của danh sách liên kết, cho đến khi phát hiện giống nhau thì thay thế giá trị, hoặc duyệt hết danh sách liên kết mà không có giá trị giống nhau.
      1. Nếu dung lượng hiện tại lớn hơn ngưỡng mở rộng, nhỏ hơn dung lượng tối đa, **tiến hành mở rộng dung lượng**.
      2. Trực tiếp chèn vào đầu danh sách liên kết.

4. Nếu vị trí cần chèn trước đó đã tồn tại, sau khi thay thế trả về giá trị cũ, ngược lại trả về null.

Thao tác `scanAndLockForPut` ở bước thứ 1 ở đây chưa giới thiệu, thao tác phương thức này thực hiện là liên tục spin `tryLock()` để lấy lock. Khi số lần spin lớn hơn số lần chỉ định, sử dụng `lock()` để nhận lock theo cơ chế block. Trong quá trình spin tiện thể lấy `HashEntry` tại vị trí hash.

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
    // Spin lấy lock
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            // Sau khi spin đạt số lần chỉ định, block chờ cho đến khi lấy được lock
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}

```

### 4. Mở rộng dung lượng rehash

Mở rộng dung lượng của `ConcurrentHashMap` chỉ mở rộng gấp đôi ban đầu. Dữ liệu trong mảng cũ di chuyển sang mảng mới, vị trí hoặc là giữ nguyên, hoặc biến thành `index + oldSize`, node trong tham số sau khi mở rộng dung lượng sẽ sử dụng phương pháp **chèn ở đầu (head insertion)** của danh sách liên kết để chèn vào vị trí chỉ định.

```java
private void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;
    // Dung lượng cũ
    int oldCapacity = oldTable.length;
    // Dung lượng mới, mở rộng 2 lần
    int newCapacity = oldCapacity << 1;
    // Ngưỡng mở rộng dung lượng mới
    threshold = (int)(newCapacity * loadFactor);
    // Tạo mảng mới
    HashEntry<K,V>[] newTable = (HashEntry<K,V>[]) new HashEntry[newCapacity];
    // Mask mới, mặc định 2 mở rộng thành 4, -1 là 3, nhị phân là 11.
    int sizeMask = newCapacity - 1;
    for (int i = 0; i < oldCapacity ; i++) {
        // Duyệt mảng cũ
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            // Tính toán vị trí mới, vị trí mới chỉ có thể là giữ nguyên hoặc vị trí cũ + dung lượng cũ.
            int idx = e.hash & sizeMask;
            if (next == null)   //  Single node on list
                // Nếu vị trí hiện tại chưa phải danh sách liên kết, chỉ là một phần tử, gán trực tiếp
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                // Nếu đã là danh sách liên kết
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                // Vị trí mới chỉ có thể là giữ nguyên hoặc vị trí cũ + dung lượng cũ.
                // Sau khi kết thúc duyệt, vị trí các phần tử đằng sau lastRun đều giống nhau
                for (HashEntry<K,V> last = next; last != null; last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // Vị trí các phần tử đằng sau lastRun đều giống nhau, trực tiếp lấy làm danh sách liên kết gán vào vị trí mới.
                newTable[lastIdx] = lastRun;
                // Clone remaining nodes
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    // Duyệt các phần tử còn lại, chèn vào đầu vị trí k chỉ định.
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // Chèn node mới theo phương pháp chèn vào đầu
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

Một số bạn có thể thắc mắc về 2 vòng lặp for cuối cùng: ở đây vòng for thứ nhất là để tìm một node như vậy, tất cả các node next đằng sau node này đều có vị trí mới giống nhau. Sau đó đưa phần này làm một danh sách liên kết gán vào vị trí mới. Vòng for thứ hai là để đưa các phần tử còn lại chèn vào danh sách liên kết tại vị trí chỉ định bằng phương pháp chèn ở đầu.

Vòng `for` thứ hai bên trong sử dụng `new HashEntry<K,V>(h, p.key, v, n)` tạo một `HashEntry` mới thay vì dùng lại đối tượng trước đó, là vì nếu dùng lại đối tượng trước đó sẽ dẫn đến thread đang duyệt (như đang thực thi phương thức `get`) không thể duyệt tiếp do con trỏ bị sửa đổi. Đúng như comment đã nói:

> Các node bị thay thế sẽ được garbage collection thu gom ngay khi chúng không còn được tham chiếu bởi bất kỳ thread đọc nào đang duyệt bảng đồng thời.

Tại sao cần dùng thêm một vòng `for` để tìm `lastRun`, thực chất là để giảm số lần tạo đối tượng, đúng như comment đã nói:

> Về mặt thống kê, ở ngưỡng mặc định, khi dung lượng bảng tăng gấp đôi, chỉ có khoảng 1/6 số node cần phải clone.

### 5. get

Đến đây rất đơn giản rồi, phương thức get chỉ cần 2 bước:

1. Tính toán lấy vị trí lưu trữ của key.
2. Duyệt vị trí chỉ định để tìm giá trị value có key trùng khớp.

```java
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // Tính toán vị trí lưu trữ của key
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
              e != null; e = e.next) {
            // Nếu là danh sách liên kết, duyệt tìm value có key trùng khớp.
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

## 2. ConcurrentHashMap 1.8

Nhìn chung, `ConcurrentHashMap` trong Java8 so với Java7 có sự thay đổi khá lớn,

### 1. Cấu trúc lưu trữ

![Cấu trúc lưu trữ của ConcurrentHashMap Java 8 (Hình ảnh từ javadoop)](https://oss.javaguide.cn/github/javaguide/java/collection/java8_concurrenthashmap.png)

Có thể phát hiện ConcurrentHashMap của Java8 so với Java7 có sự thay đổi tương đối lớn, không còn là **mảng Segment + mảng HashEntry + danh sách liên kết** như trước, mà là **mảng Node + danh sách liên kết / cây đỏ đen**. Khi danh sách liên kết xung đột đạt đến độ dài nhất định, danh sách liên kết sẽ chuyển đổi thành cây đỏ đen.

### 2. Khởi tạo initTable

```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        // Nếu sizeCtl < 0, chứng tỏ thread khác thực thi CAS thành công, đang tiến hành khởi tạo.
        if ((sc = sizeCtl) < 0)
            // Nhường quyền sử dụng CPU
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

Từ source code có thể thấy sự khởi tạo của `ConcurrentHashMap` được hoàn thành thông qua **spin và CAS**. Trong đó cần lưu ý biến `sizeCtl` (viết tắt của sizeControl), giá trị của nó quyết định trạng thái khởi tạo hiện tại.

1. -1: Cho biết đang khởi tạo, các thread khác cần spin chờ đợi.
2. -N: Cho biết table đang tiến hành mở rộng dung lượng, 16 bit cao biểu thị stamp đánh dấu mở rộng, 16 bit thấp trừ 1 là số thread đang tiến hành mở rộng dung lượng.
3. 0: Biểu thị table chưa khởi tạo, khi khởi tạo sử dụng dung lượng mặc định.
4. >0: Biểu thị ngưỡng mở rộng dung lượng của table, nếu table đã khởi tạo.

### 3. put

Trực tiếp duyệt qua source code put.

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key và value không được null
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        // f = phần tử vị trí mục tiêu
        Node<K,V> f; int n, i, fh;// fh phía sau lưu giá trị hash phần tử vị trí mục tiêu
        if (tab == null || (n = tab.length) == 0)
            // Bucket mảng rỗng, khởi tạo bucket mảng (spin + CAS)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // Trong bucket rỗng, CAS đưa vào, không lock, thành công thì break nhảy ra luôn
            if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))
                break;  // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // Sử dụng synchronized khóa để thêm node
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // Chứng tỏ là danh sách liên kết
                    if (fh >= 0) {
                        binCount = 1;
                        // Vòng lặp thêm mới hoặc ghi đè node
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        // Cây đỏ đen
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

1. Dựa vào key để tính toán hashcode.
2. Kiểm tra xem có cần tiến hành khởi tạo hay không.
3. Nếu Node định vị từ key hiện tại là null, biểu thị vị trí hiện tại có thể ghi dữ liệu, sử dụng CAS thử ghi; thất bại thì vào lại vòng lặp và kiểm tra trạng thái mới nhất.
4. Nếu vị trí hiện tại có `hashcode == MOVED == -1`, thì cần tiến hành mở rộng dung lượng.
5. Nếu đều không thỏa mãn, sử dụng synchronized lock để ghi dữ liệu.
6. Nếu số lượng lớn hơn `TREEIFY_THRESHOLD` thì phải thực thi phương thức treeifyBin, trong `treeifyBin` đầu tiên sẽ kiểm tra độ dài mảng hiện tại ≥64 mới chuyển danh sách liên kết thành cây đỏ đen.

### 4. get

Quy trình get tương đối đơn giản, trực tiếp xem qua source code.

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // Vị trí hash chứa key
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // Nếu phần tử vị trí chỉ định tồn tại, giá trị hash node đầu giống nhau
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                // Giá trị hash key bằng nhau, giá trị key giống nhau, trả về trực tiếp element value
                return e.val;
        }
        else if (eh < 0)
            // Giá trị hash node đầu nhỏ hơn 0, chứng tỏ đang mở rộng dung lượng hoặc là cây đỏ đen, dùng find tìm kiếm
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            // Là danh sách liên kết, duyệt tìm kiếm
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

Tổng kết lại quá trình get:

1. Tính toán vị trí theo giá trị hash.
2. Tìm đến vị trí chỉ định, nếu node đầu chính là phần tử cần tìm thì trả về value của nó.
3. Nếu giá trị hash của node đầu nhỏ hơn 0, chứng tỏ đang mở rộng dung lượng hoặc là cây đỏ đen, thực hiện tìm kiếm.
4. Nếu là danh sách liên kết, duyệt tìm kiếm.

### 5. Đếm size

Phương thức `size()` của `ConcurrentHashMap` dùng để lấy tổng số phần tử trong Map hiện tại, nhưng trong kịch bản concurrency cao, làm thế nào để thống kê số lượng phần tử một cách chính xác và hiệu quả là một bài toán kỹ thuật khó. Java 8 áp dụng một cơ chế đếm phân đoạn (segmented counting) vô cùng tinh xảo để giải quyết vấn đề này.

#### 5.1 Tại sao cần đếm分段 (đếm phân đoạn)

Trong môi trường concurrent, nếu nhiều thread đồng thời thực thi thao tác `put`, tất cả chúng đều cần cập nhật tổng số phần tử. Nếu sử dụng một biến counter chia sẻ, sẽ dẫn đến tranh chấp gay gắt — tất cả các thread đều tranh giành quyền sửa đổi cùng một biến, điều này ảnh hưởng nghiêm trọng đến hiệu năng.

Để giải quyết vấn đề này, `ConcurrentHashMap` đã áp dụng tư tưởng thiết kế **phân tán hot-spot**: không sử dụng một counter duy nhất mà phân tán việc đếm vào nhiều biến khác nhau. Giống như ngân hàng không chỉ mở 1 cửa giao dịch mà mở nhiều cửa để phân luồng khách hàng, từ đó giảm thiểu xung đột rất nhiều.

#### 5.2 Thiết kế của baseCount và counterCells

`ConcurrentHashMap` duy trì 2 field then chốt liên quan đến đếm ở bên trong:

- **baseCount**: Counter cơ sở, trong trường hợp không có tranh chấp, trực tiếp cập nhật biến này thông qua CAS. Có thể hiểu đây là "counter chính".
- **counterCells**: Mảng các counter. Khi nhiều thread tranh chấp `baseCount` thất bại, sẽ thử phân tán lượng tăng của counter sang các vị trí khác nhau trong mảng `counterCells`.
  - Mỗi thread dựa trên giá trị **Probe** của bản thân (có thể hiểu là một loại hashcode do thread ID tạo ra) để ánh xạ vào một slot nào đó trong mảng, ưu tiên cộng dồn vào "ô nghiêng về mình" đó.
  - **Lưu ý**: Ô này không phải mang ý nghĩa nghiêm ngặt là "private của thread", khi xung đột hash, nhiều thread vẫn có thể ánh xạ vào cùng một slot để cập nhật đồng thời.

**Lấy một ví dụ**: Giả sử có 10 thread đồng thời thêm phần tử vào Map. Thread đầu tiên thông qua CAS cập nhật thành công `baseCount`, nhưng 9 thread sau khi cập nhật `baseCount` phát hiện có tranh chấp, sẽ chuyển sang tìm một vị trí trong mảng `counterCells` để cộng dồn. 9 thread này có thể phân tán sang các vị trí khác nhau trong mảng (ví dụ thread 2 ở `counterCells[1]`, thread 3 ở `counterCells[2]`), từ đó phân tán tranh chấp từ 1 điểm ra nhiều điểm.

#### 5.3 Cập nhật counter thế nào khi put phần tử

Ở cuối phương thức `putVal`, chúng ta có thể thấy việc gọi phương thức `addCount(1L, binCount)`, phương thức này dùng để cập nhật counter phần tử.

Logic thực thi của `addCount` có thể khái quát lại như sau:

1. **Ưu tiên thử cập nhật baseCount**

   - Nếu hiện tại chưa bật `counterCells` (`counterCells == null`), thread trước tiên sẽ thử cập nhật trực tiếp `baseCount` qua CAS.
   - Nếu CAS thành công, chứng tỏ tranh chấp không gay gắt, trực tiếp return là được.

2. **Khi xuất hiện tranh chấp, chuyển sang counterCells**

   - Nếu CAS cập nhật `baseCount` thất bại (chứng tỏ có thread khác đang tranh chấp), hoặc `counterCells` đã tồn tại (chứng tỏ hệ thống trước đó đã từng gặp tranh chấp), thread sẽ thử cập nhật trong `counterCells`:
     - Dựa trên giá trị probe của bản thân để ánh xạ vào một slot;
     - Thực hiện một lần CAS cộng dồn trên `CounterCell` tương ứng với slot đó.
   - Nếu slot này null hoặc CAS vẫn xung đột, sẽ đi vào một path "nặng" hơn là `fullAddCount`, bên trong chịu trách nhiệm khởi tạo slot, chọn lại slot,...

3. **Khởi tạo động và mở rộng counterCells**
   - Khi phát hiện tranh chấp tương đối gay gắt (ví dụ: CAS của một cell nào đó cũng thất bại thường xuyên), `fullAddCount` sẽ dưới sự bảo vệ của spin lock nhẹ `cellsBusy`:
     - Nếu `counterCells` chưa khởi tạo thì khởi tạo mảng nhỏ (ví dụ độ dài 2);
     - Nếu đã tồn tại và độ dài chưa đạt giới hạn trên (thường không vượt quá số core CPU), sẽ mở rộng gấp 2 lần, tăng thêm slot đếm để phân tán thread thêm nữa.

Thiết kế này đảm bảo: Khi concurrency thấp chỉ sử dụng `baseCount` đơn giản, path rất ngắn; khi concurrency cao tự động chuyển sang đếm phân đoạn, thông qua `counterCells` và cơ chế mở rộng dung lượng để làm loãng tranh chấp, vừa đảm bảo hiệu năng vừa đảm bảo tính chính xác.

#### 5.4 sumCount tính tổng số phần tử như thế nào

Khi chúng ta gọi phương thức `size()`, cuối cùng sẽ gọi phương thức `sumCount()` để tính tổng số phần tử. Logic của `sumCount()` rất đơn giản và trực tiếp:

1. Đọc giá trị của `baseCount` làm giá trị cơ sở.
2. Duyệt mảng `counterCells`, cộng dồn giá trị đếm của tất cả các vị trí non-null vào giá trị cơ sở.
3. Trả về kết quả cộng dồn.

**Lưu ý:**

- **Tính nhất quán yếu (Weak consistency)**: `sumCount()` toàn bộ quá trình **không lock**. Trong thời gian tính toán nếu có thread khác chèn dữ liệu, kết quả trả về chỉ là một **giá trị xấp xỉ**. Tuy nhiên trong kịch bản concurrency cao, việc theo đuổi "tổng số chính xác tuyệt đối trong khoảnh khắc" chi phí quá lớn và không có ý nghĩa, giá trị xấp xỉ thường đã đủ dùng.
- **Tràn số nguyên (Integer overflow)**: Phương thức `size()` trả về kiểu `int`. Nếu số lượng phần tử vượt quá `Integer.MAX_VALUE`, nó chỉ trả về `Integer.MAX_VALUE`. Phương thức **`mappingCount()`** mới bổ sung trong Java 8 trả về kiểu `long`, thích hợp biểu thị giá trị đếm lớn hơn, nhưng trong thời gian cập nhật đồng thời giá trị trả về vẫn là ước tính.

## 3. Tổng kết

Trong Java 7, `ConcurrentHashMap` sử dụng lock phân đoạn (Segment lock), nghĩa là trên mỗi Segment tại một thời điểm chỉ có 1 thread có thể thao tác, mỗi `Segment` đều là một cấu trúc tương tự mảng `HashMap`, nó có thể resize, xung đột của nó sẽ chuyển thành danh sách liên kết. Tuy nhiên số lượng `Segment` một khi đã khởi tạo thì không thể thay đổi.

Trong Java 8, `ConcurrentHashMap` sử dụng cơ chế `Synchronized` lock cộng với CAS. Cấu trúc cũng tiến hóa từ **mảng `Segment` + mảng `HashEntry` + danh sách liên kết** của Java 7 thành **mảng Node + danh sách liên kết / cây đỏ đen**, Node là một cấu trúc tương tự `HashEntry`. Xung đột của nó khi đạt đến kích thước nhất định `TREEIFY_THRESHOLD = 8` sẽ chuyển thành cây đỏ đen, khi xung đột nhỏ hơn số lượng nhất định `UNTREEIFY_THRESHOLD = 6` sẽ thoái hóa về danh sách liên kết.

Một số bạn có thể nghi ngờ về hiệu năng của `Synchronized`, thực ra `Synchronized` lock kể từ khi đưa vào chiến lược nâng cấp lock (lock escalation) thì hiệu năng không còn là vấn đề nữa, các bạn có hứng thú có thể tự tìm hiểu về **nâng cấp lock (lock escalation)** của `Synchronized`.

<!-- @include: @article-footer.snippet.md -->
