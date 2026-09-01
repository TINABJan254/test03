---
title: Java集合使用注意事项总结
description: Java集合使用注意事项总结：基于阿里巴巴开发手册梳理集合判空、Arrays.asList陷阱、subList问题、并发容器选择等最佳实践，避免常见错误。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: Java集合最佳实践,集合判空,Arrays.asList,subList,并发容器,集合使用注意事项,性能优化
---

Bài viết này tôi dựa trên 《Sổ tay phát triển Java Alibaba》 để tổng kết các lưu ý thường gặp cũng như nguyên lý cụ thể khi sử dụng Collection.

Rất khuyến nghị các bạn đọc kỹ vài lần để tránh gặp phải những lỗi cơ bản này khi tự mình viết code.

## Kiểm tra rỗng cho Collection

Mô tả trong 《Sổ tay phát triển Java Alibaba》 như sau:

> **Kiểm tra xem tất cả phần tử bên trong collection có rỗng hay không, sử dụng phương thức `isEmpty()`, chứ không dùng cách `size() == 0`.**

Điều này là do phương thức `isEmpty()` có tính đọc hiểu tốt hơn và có độ phức tạp thời gian là `O(1)`.

Tuyệt đại đa số các collection chúng ta sử dụng có độ phức tạp thời gian của phương thức `size()` cũng là `O(1)`, tuy nhiên cũng có nhiều trường hợp độ phức tạp không phải `O(1)`, ví dụ như `ConcurrentLinkedQueue` trong package `java.util.concurrent`. Phương thức `isEmpty()` của `ConcurrentLinkedQueue` kiểm tra thông qua phương thức `first()`, trong đó phương thức `first()` trả về node đầu tiên trong queue có giá trị khác `null` (giá trị node là `null` do logic xóa mềm/logical delete được dùng trong iterator).

```java
public boolean isEmpty() { return first() == null; }

Node<E> first() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            boolean hasItem = (p.item != null);
            if (hasItem || (q = p.next) == null) {  // Giá trị node hiện tại không rỗng hoặc đã đến cuối hàng đợi
                updateHead(h, p);  // Đặt head thành p
                return hasItem ? p : null;
            }
            else if (p == q) continue restartFromHead;
            else p = q;  // p = p.next
        }
    }
}
```

Do khi chèn và xóa phần tử đều sẽ thực thi phương thức `updateHead(h, p)`, nên độ phức tạp thời gian thực thi của phương thức này có thể xấp xỉ là `O(1)`. Còn phương thức `size()` cần phải duyệt toàn bộ danh sách liên kết, độ phức tạp thời gian là `O(n)`.

```java
public int size() {
    int count = 0;
    for (Node<E> p = first(); p != null; p = succ(p))
        if (p.item != null)
            if (++count == Integer.MAX_VALUE)
                break;
    return count;
}
```

Ngoài ra, trong `ConcurrentHashMap` 1.7, độ phức tạp thời gian của phương thức `size()` và `isEmpty()` cũng khác nhau. `ConcurrentHashMap` 1.7 lưu trữ số lượng phần tử trong mỗi `Segment`, phương thức `size()` cần thống kê số lượng của từng `Segment`, còn `isEmpty()` chỉ cần tìm thấy `Segment` đầu tiên không rỗng là được. Nhưng trong `ConcurrentHashMap` 1.8 cả phương thức `size()` và `isEmpty()` đều cần gọi phương thức `sumCount()` để tổng hợp đếm số lượng từ `baseCount` và `CounterCell[]`. Dưới đây là source code của phương thức `sumCount()`:

```java
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null)
        for (int i = 0; i < as.length; ++i)
            if ((a = as[i]) != null)
                sum += a.value;
    return sum;
}
```

Trong môi trường concurrency, `ConcurrentHashMap` 1.8 sử dụng `baseCount` và `CounterCell[]` để phân tán tranh chấp khi cập nhật đếm số lượng, chứ không lưu trữ số lượng node trong mỗi `Node`. Trong `ConcurrentHashMap` 1.7, số lượng phần tử được lưu trữ trong từng `Segment`, phương thức `size()` cần thống kê số lượng của từng `Segment`, còn `isEmpty()` chỉ cần tìm `Segment` đầu tiên không rỗng.

## Chuyển Collection thành Map

Mô tả trong 《Sổ tay phát triển Java Alibaba》 như sau:

> **Khi sử dụng phương thức `toMap()` của class `java.util.stream.Collectors` để chuyển thành collection `Map`, nhất định phải chú ý khi value là null sẽ ném ra ngoại lệ NPE.**

```java
class Person {
    private String name;
    private String phoneNumber;
     // getters and setters
}

List<Person> bookList = new ArrayList<>();
bookList.add(new Person("jack","18163138123"));
bookList.add(new Person("martin",null));
// Ngoại lệ NullPointerException
bookList.stream().collect(Collectors.toMap(Person::getName, Person::getPhoneNumber));
```

Dưới đây chúng ta giải thích nguyên nhân.

Trước tiên, hãy cùng xem phương thức `toMap()` của class `java.util.stream.Collectors`, có thể thấy bên trong nó gọi phương thức `merge()` của interface `Map`.

```java
public static <T, K, U, M extends Map<K, U>>
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                            Function<? super T, ? extends U> valueMapper,
                            BinaryOperator<U> mergeFunction,
                            Supplier<M> mapSupplier) {
    BiConsumer<M, T> accumulator
            = (map, element) -> map.merge(keyMapper.apply(element),
                                          valueMapper.apply(element), mergeFunction);
    return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
}
```

Phương thức `merge()` của interface `Map` như sau, phương thức này là default implementation trong interface.

> Nếu bạn chưa nắm rõ các tính năng mới của Java 8, hãy xem bài viết này: [《Tổng kết tính năng mới của Java 8》](https://mp.weixin.qq.com/s/ojyl7B6PiHaTWADqmUq2rw).

```java
default V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

Phương thức `merge()` trước tiên sẽ gọi phương thức `Objects.requireNonNull()` để kiểm tra xem value có rỗng hay không.

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

> `Collectors` cũng cung cấp phương thức `toMap()` không cần `mergeFunction`, nhưng lúc này nếu xuất hiện xung đột key sẽ ném ra ngoại lệ `duplicateKeyException`, do đó khuyến nghị mạnh mẽ khi dùng phương thức `toMap()` bắt buộc phải điền `mergeFunction`.

## Duyệt Collection

Mô tả trong 《Sổ tay phát triển Java Alibaba》 như sau:

> **Không thực hiện thao tác `remove/add` phần tử trong vòng lặp foreach. Để remove phần tử xin hãy sử dụng cách `Iterator`, nếu là thao tác đồng thời (concurrency), cần phải khóa (lock) đối tượng `Iterator`.**

Cần lưu ý rằng, việc chỉ lock trên đối tượng `Iterator` không thể ngăn cản các thread khác sửa đổi collection. Lấy đồng bộ wrapper trả về từ `Collections.synchronizedXxx()` làm ví dụ, khi duyệt nên đồng bộ hóa đối tượng collection đã bọc và đảm bảo tất cả các truy cập đều thông qua wrapper đó.

Thông qua decompile bạn sẽ thấy cú pháp foreach bên dưới thực chất vẫn phụ thuộc vào `Iterator`. Tuy nhiên, thao tác `remove/add` lại trực tiếp gọi phương thức của chính collection đó chứ không phải phương thức `remove/add` của `Iterator`.

Điều này dẫn đến `Iterator` một cách vô lý phát hiện phần tử của mình bị `remove/add`, sau đó nó sẽ ném ra một `ConcurrentModificationException` để cảnh báo người dùng đã xảy ra ngoại lệ sửa đổi đồng thời. Đây chính là **cơ chế fail-fast** phát sinh trong trạng thái đơn luồng.

> **Cơ chế fail-fast**: Khi nhiều thread cùng sửa đổi trên collection fail-fast, có thể sẽ ném ra `ConcurrentModificationException`. Ngay cả trong môi trường đơn luồng cũng có thể xảy ra trường hợp này như đã đề cập ở trên.
>
> Đọc thêm: [fail-fast là gì](https://www.cnblogs.com/54chensongxia/p/12470446.html).

Từ Java 8 trở đi, có thể sử dụng phương thức `Collection#removeIf()` để xóa các phần tử thỏa mãn điều kiện cụ thể, ví dụ:

```java
List<Integer> list = new ArrayList<>();
for (int i = 1; i <= 10; ++i) {
    list.add(i);
}
list.removeIf(filter -> filter % 2 == 0); /* Xóa tất cả các số chẵn trong list */
System.out.println(list); /* [1, 3, 5, 7, 9] */
```

Ngoài việc trực tiếp sử dụng `Iterator` để thực hiện thao tác duyệt đã giới thiệu ở trên, bạn còn có thể:

- Sử dụng vòng lặp for thông thường
- Dựa theo kịch bản sử dụng các class collection hỗ trợ lặp snapshot hoặc lặp weak-consistency. Ví dụ iterator của `CopyOnWriteArrayList` dựa trên snapshot, iterator của `ConcurrentHashMap` là weak-consistency.
- ……

## Loại bỏ trùng lặp (De-duplication) trong Collection

Mô tả trong 《Sổ tay phát triển Java Alibaba》 như sau:

> **Tận dụng đặc tính phần tử duy nhất của `Set`, có thể loại bỏ trùng lặp cho một collection một cách nhanh chóng, tránh dùng `contains()` của `List` để duyệt loại bỏ trùng lặp hoặc kiểm tra chứa.**

Ở đây chúng ta lấy `HashSet` và `ArrayList` làm ví dụ minh họa.

```java
// Ví dụ code loại bỏ trùng lặp bằng Set
public static <T> Set<T> removeDuplicateBySet(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new HashSet<>();
    }
    return new HashSet<>(data);
}

// Ví dụ code loại bỏ trùng lặp bằng List
public static <T> List<T> removeDuplicateByList(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new ArrayList<>();

    }
    List<T> result = new ArrayList<>(data.size());
    for (T current : data) {
        if (!result.contains(current)) {
            result.add(current);
        }
    }
    return result;
}

```

Sự khác biệt cốt lõi giữa cả hai nằm ở việc triển khai phương thức `contains()`.

Phương thức `contains()` của `HashSet` bên dưới phụ thuộc vào phương thức `containsKey()` của `HashMap`, độ phức tạp thời gian tiệm cận O(1) (khi không xuất hiện xung đột hash là O(1)).

```java
private transient HashMap<E,Object> map;
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

Chúng ta có N phần tử chèn vào Set, độ phức tạp thời gian tiệm cận là O(n).

Phương thức `contains()` của `ArrayList` được thực hiện bằng cách duyệt qua tất cả phần tử, độ phức tạp thời gian tiệm cận là O(n).

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}

```

## Chuyển Collection thành Mảng

Mô tả trong 《Sổ tay phát triển Java Alibaba》 như sau:

> **Sử dụng phương thức chuyển collection thành mảng, bắt buộc phải dùng `toArray(T[] array)` của collection, truyền vào một mảng rỗng có kiểu hoàn toàn nhất quán và chiều dài bằng 0.**

Tham số của phương thức `toArray(T[] array)` là một mảng generic, nếu trong phương thức `toArray` không truyền vào bất kỳ tham số nào thì kết quả trả về sẽ là mảng kiểu `Object`.

```java
String [] s= new String[]{
    "dog", "lazy", "a", "over", "jumps", "fox", "brown", "quick", "A"
};
List<String> list = Arrays.asList(s);
Collections.reverse(list);
// Nếu không chỉ định kiểu sẽ bị báo lỗi
s=list.toArray(new String[0]);
```

Do sự tối ưu hóa của JVM, `new String[0]` làm tham số cho phương thức `Collection.toArray()` hiện tại sử dụng tốt hơn, `new String[0]` đóng vai trò làm một template (khuôn mẫu), chỉ định kiểu của mảng trả về, số 0 là để tiết kiệm không gian bộ nhớ vì nó chỉ dùng để làm rõ kiểu dữ liệu trả về. Chi tiết xem tại: <https://shipilev.net/blog/2016/arrays-wisdom-ancients/>

## Chuyển Mảng thành Collection

Mô tả trong 《Sổ tay phát triển Java Alibaba》 như sau:

> **Khi sử dụng utility class `Arrays.asList()` để chuyển mảng thành collection, không thể sử dụng các phương thức sửa đổi collection tương ứng của nó, các phương thức `add/remove/clear` của nó sẽ ném ra ngoại lệ `UnsupportedOperationException`.**

Tôi đã từng gặp một sự cố ("bẫy"/pitfall) tương tự trong một dự án trước đây.

`Arrays.asList()` trong phát triển hàng ngày tương đối phổ biến, chúng ta có thể sử dụng nó để chuyển một mảng thành một `List` collection.

```java
String[] myArray = {"Apple", "Banana", "Orange"};
List<String> myList = Arrays.asList(myArray);
// Hai câu lệnh trên tương đương với một câu lệnh dưới đây
List<String> myList = Arrays.asList("Apple","Banana", "Orange");
```

Giải thích của JDK source code về phương thức này:

```java
/**
  * Trả về danh sách có kích thước cố định được hỗ trợ bởi mảng chỉ định. Phương thức này đóng vai trò cầu nối giữa API dựa trên mảng và API dựa trên collection,
  * kết hợp sử dụng với Collection.toArray(). List trả về là serializable và triển khai interface RandomAccess.
  */
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

Dưới đây chúng ta tổng kết các lưu ý khi sử dụng.

**1. `Arrays.asList()` không tự động autobox mảng kiểu nguyên thủy (primitive) và trải rộng thành các phần tử của list.**

```java
int[] myArray = {1, 2, 3};
List myList = Arrays.asList(myArray);
System.out.println(myList.size());//1
System.out.println(myList.get(0));// Giá trị địa chỉ mảng
System.out.println(myList.get(1));// Lỗi: ArrayIndexOutOfBoundsException
int[] array = (int[]) myList.get(0);
System.out.println(array[0]);//1
```

Khi truyền vào một mảng kiểu dữ liệu nguyên thủy, tham số thực sự mà `Arrays.asList()` nhận được không phải là các phần tử trong mảng, mà là chính đối tượng mảng đó! Lúc này phần tử duy nhất của `List` chính là mảng này, điều này giải thích cho đoạn code ở trên.

Chúng ta sử dụng mảng kiểu wrapper class là có thể giải quyết vấn đề này.

```java
Integer[] myArray = {1, 2, 3};
```

**2. Sử dụng các phương thức sửa đổi của collection: `add()`, `remove()`, `clear()` sẽ ném ra exception.**

```java
List myList = Arrays.asList(1, 2, 3);
myList.add(4);// Lỗi runtime: UnsupportedOperationException
myList.remove(1);// Lỗi runtime: UnsupportedOperationException
myList.clear();// Lỗi runtime: UnsupportedOperationException
```

Phương thức `Arrays.asList()` trả về không phải là `java.util.ArrayList`, mà là một inner class của `java.util.Arrays`, inner class này không triển khai các phương thức sửa đổi collection hoặc nói cách khác là chưa override các phương thức đó.

```java
List myList = Arrays.asList(1, 2, 3);
System.out.println(myList.getClass());//class java.util.Arrays$ArrayList
```

Dưới đây là source code đơn giản của `java.util.Arrays$ArrayList`, chúng ta có thể thấy class này đã override những phương thức nào.

```java
  private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
        ...

        @Override
        public E get(int index) {
          ...
        }

        @Override
        public E set(int index, E element) {
          ...
        }

        @Override
        public int indexOf(Object o) {
          ...
        }

        @Override
        public boolean contains(Object o) {
           ...
        }

        @Override
        public void forEach(Consumer<? super E> action) {
          ...
        }

        @Override
        public void replaceAll(UnaryOperator<E> operator) {
          ...
        }

        @Override
        public void sort(Comparator<? super E> c) {
          ...
        }
    }
```

Chúng ta xem lại phương thức `add/remove/clear` của `java.util.AbstractList` sẽ biết tại sao lại ném ra `UnsupportedOperationException`.

```java
public E remove(int index) {
    throw new UnsupportedOperationException();
}
public boolean add(E e) {
    add(size(), e);
    return true;
}
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}

public void clear() {
    removeRange(0, size());
}
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```

**Vậy làm thế nào để chuyển đổi mảng sang `ArrayList` một cách chính xác?**

1. Tự tay triển khai utility class

```java
//JDK1.5+
static <T> List<T> arrayToList(final T[] array) {
  final List<T> l = new ArrayList<T>(array.length);

  for (final T s : array) {
    l.add(s);
  }
  return l;
}


Integer [] myArray = { 1, 2, 3 };
System.out.println(arrayToList(myArray).getClass());//class java.util.ArrayList
```

2. Phương pháp đơn giản nhất

```java
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```

3. Sử dụng Stream của Java 8 (Khuyến nghị)

```java
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
// Kiểu nguyên thủy cũng có thể thực hiện chuyển đổi (dựa vào thao tác đóng hộp boxed)
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

4. Sử dụng Guava

Đối với unmodifiable collection, bạn có thể sử dụng class [`ImmutableList`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java) và các factory method [`of()`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L101) và [`copyOf()`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L225): (Tham số không được null)

```java
List<String> il = ImmutableList.of("string", "elements");  // từ varargs
List<String> il = ImmutableList.copyOf(aStringArray);      // từ mảng
```

Đối với modifiable collection, bạn có thể sử dụng class [`Lists`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java) và factory method [`newArrayList()`](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java#L87):

```java
List<String> l1 = Lists.newArrayList(anotherListOrCollection);    // từ collection
List<String> l2 = Lists.newArrayList(aStringArray);               // từ mảng
List<String> l3 = Lists.newArrayList("or", "string", "elements"); // từ varargs
```

5. Sử dụng Apache Commons Collections

```java
List<String> list = new ArrayList<String>();
CollectionUtils.addAll(list, str);
```

6. Sử dụng phương thức `List.of()` của Java 9

```java
Integer[] array = {1, 2, 3};
List<Integer> list = List.of(array);
```

<!-- @include: @article-footer.snippet.md -->
