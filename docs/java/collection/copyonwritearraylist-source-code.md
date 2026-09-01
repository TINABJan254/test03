---
title: CopyOnWriteArrayList 源码分析
description: CopyOnWriteArrayList源码深度解析：详解写时复制COW机制、适用读多写少场景、线程安全List实现、快照一致性保证及内存开销权衡。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: CopyOnWriteArrayList源码,写时复制COW,线程安全List,读多写少,并发容器,快照一致性
---

## Giới thiệu về CopyOnWriteArrayList

Trước JDK1.5, nếu muốn sử dụng một `List` an toàn về concurrency, bạn có thể chọn `Vector` hoặc wrapper đồng bộ hóa trả về bởi `Collections.synchronizedList()`. Trong đó `Vector` là một collection cũ kỹ đã bị loại bỏ. `Vector` cơ bản đều thêm `synchronized` vào các phương thức thêm xóa sửa tìm kiếm, cách này tuy có thể đảm bảo đồng bộ hóa, nhưng điều này tương đương với việc khoác một cái khóa lớn lên toàn bộ `Vector`, khiến mỗi phương thức khi thực thi đều phải đi lấy lock, dẫn đến hiệu năng rất thấp.

JDK1.5 đã đưa vào package `java.util.concurrent` (JUC), cung cấp rất nhiều container thread-safe và có hiệu năng concurrency tốt, trong đó implementation `List` thread-safe duy nhất chính là `CopyOnWriteArrayList`. Về bài tổng kết các concurrent container phổ biến trong package `java.util.concurrent`, bạn có thể xem bài viết này của tôi: [Tổng kết các Java Concurrent Container thường gặp](https://javaguide.cn/java/concurrent/java-concurrent-collections.html).

### CopyOnWriteArrayList rốt cuộc có điểm gì lợi hại?

Đối với hầu hết các kịch bản nghiệp vụ, thao tác đọc thường nhiều hơn rất nhiều so với thao tác ghi. Do thao tác đọc không sửa đổi dữ liệu ban đầu, do đó việc lock cho mỗi lần đọc thực chất là một sự lãng phí tài nguyên. So với điều đó, chúng ta nên cho phép nhiều thread đồng thời truy cập dữ liệu bên trong của `List`, dù sao đối với thao tác đọc là an toàn.

Tư tưởng này rất tương đồng với tư tưởng thiết kế read-write lock của `ReentrantReadWriteLock`, tức là đọc-đọc không xung đột (non-mutually exclusive), đọc-ghi xung đột, ghi-ghi xung đột (chỉ có đọc-đọc là không xung đột). `CopyOnWriteArrayList` còn đi xa hơn một bước để thực hiện tư tưởng này. Để phát huy hiệu năng thao tác đọc đến mức tối đa, thao tác đọc trong `CopyOnWriteArrayList` hoàn toàn không cần lock. Đáng kinh ngạc hơn là, thao tác ghi cũng không làm block thao tác đọc, chỉ có ghi-ghi mới xung đột. Nhờ đó, hiệu năng thao tác đọc có thể được nâng cao rất lớn.

Trọng tâm thread-safe của `CopyOnWriteArrayList` nằm ở việc nó áp dụng chiến lược **Copy-On-Write (Viết khi sao chép)**, từ tên gọi `CopyOnWriteArrayList` đã có thể thấy điều đó.

### Tư tưởng của Copy-On-Write là gì?

"Copy-On-Write" trong tên của `CopyOnWriteArrayList` chính là sao chép khi ghi, gọi tắt là COW.

Dưới đây là phần giới thiệu về Copy-On-Write trên Wikipedia, giới thiệu khá hay:

> Copy-on-write (viết tắt là COW) là một chiến lược tối ưu hóa trong lĩnh vực thiết kế chương trình máy tính. Tư tưởng cốt lõi của nó là, nếu có nhiều bên gọi (callers) đồng thời yêu cầu cùng một tài nguyên (như bộ nhớ hoặc nơi lưu trữ dữ liệu trên đĩa), họ sẽ cùng lấy con trỏ giống nhau trỏ đến cùng tài nguyên đó, cho đến khi có một bên gọi cố gắng sửa đổi nội dung tài nguyên, hệ thống mới thực sự sao chép một bản sao riêng (private copy) cho bên gọi đó, còn tài nguyên ban đầu mà các bên gọi khác nhìn thấy vẫn giữ nguyên không đổi. Quá trình này hoàn toàn trong suốt đối với các bên gọi khác. Ưu điểm chính của phương pháp này là nếu bên gọi không sửa đổi tài nguyên đó thì sẽ không có bản sao riêng (private copy) nào được tạo ra, do đó nhiều bên gọi khi chỉ thực hiện thao tác đọc có thể chia sẻ cùng một tài nguyên.

Ở đây lại lấy `CopyOnWriteArrayList` làm ví dụ để giới thiệu: Khi cần sửa đổi (các thao tác `add`, `set`, `remove`,...) nội dung của `CopyOnWriteArrayList`, sẽ không trực tiếp sửa mảng ban đầu, mà trước tiên tạo một bản sao của mảng bên dưới, tiến hành sửa đổi trên mảng bản sao, sau khi sửa xong mới gán mảng đã sửa lại, như vậy có thể đảm bảo thao tác ghi không ảnh hưởng đến thao tác đọc.

Có thể thấy, cơ chế Copy-On-Write rất thích hợp cho kịch bản concurrency đọc nhiều ghi ít, có thể nâng cao rất lớn hiệu năng concurrency của hệ thống.

Tuy nhiên, cơ chế Copy-On-Write không phải là viên đạn bạc (silver bullet), nó vẫn tồn tại một số nhược điểm, dưới đây liệt kê vài điểm:

1. Chiếm dụng bộ nhớ: Mỗi thao tác ghi đều cần copy một bản dữ liệu gốc, sẽ chiếm thêm không gian bộ nhớ, trong trường hợp dung lượng dữ liệu tương đối lớn có thể dẫn đến thiếu hụt tài nguyên bộ nhớ.
2. Chi phí thao tác ghi lớn: Mỗi lần thao tác ghi đều cần copy một bản dữ liệu gốc, sau đó mới tiến hành sửa đổi và thay thế, nên chi phí thao tác ghi tương đối lớn, trong kịch bản ghi chép thường xuyên thì hiệu năng có thể bị ảnh hưởng.
3. Tính nhất quán của snapshot (ảnh chụp nhanh): Iterator sẽ giữ snapshot của mảng bên dưới tại thời điểm tạo, các sửa đổi sau đó sẽ không phản ánh vào iterator này.
4. ...

## Phân tích source code của CopyOnWriteArrayList

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code cốt lõi bên dưới của `CopyOnWriteArrayList`.

Khai báo class `CopyOnWriteArrayList` như sau:

```java
public class CopyOnWriteArrayList<E>
extends Object
implements List<E>, RandomAccess, Cloneable, Serializable
{
  //...
}
```

`CopyOnWriteArrayList` triển khai các interface sau:

- `List`: Biểu thị nó là một danh sách, hỗ trợ các thao tác thêm, xóa, tìm kiếm,... và có thể truy cập thông qua index.
- `RandomAccess`: Đây là một marker interface (interface đánh dấu), biểu thị collection `List` triển khai interface này hỗ trợ **truy cập ngẫu nhiên nhanh**.
- `Cloneable`: Biểu thị nó hỗ trợ sao chép thông qua phương thức `clone()`, `CopyOnWriteArrayList#clone()` trả về bản sao nông (shallow copy).
- `Serializable`: Biểu thị nó có thể thực hiện thao tác serialization, tức là có thể chuyển đổi đối tượng thành byte stream để lưu trữ lâu dài hoặc truyền qua mạng, rất tiện lợi.

![Sơ đồ class CopyOnWriteArrayList](https://oss.javaguide.cn/github/javaguide/java/collection/copyonwritearraylist-class-diagram.png)

### Khởi tạo

Trong `CopyOnWriteArrayList` có một constructor không tham số và 2 constructor có tham số.

```java
// Tạo một CopyOnWriteArrayList rỗng
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}

// Tạo một CopyOnWriteArrayList chứa các phần tử của collection chỉ định theo thứ tự trả về từ iterator của collection
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    if (c.getClass() == CopyOnWriteArrayList.class)
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        elements = c.toArray();
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    setArray(elements);
}

// Tạo một list chứa bản sao của mảng chỉ định
public CopyOnWriteArrayList(E[] toCopyIn) {
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}
```

### Chèn phần tử

Phương thức `add()` của `CopyOnWriteArrayList` có 3 phiên bản:

- `add(E e)`: Chèn phần tử vào cuối `CopyOnWriteArrayList`.
- `add(int index, E element)`: Chèn phần tử vào vị trí chỉ định của `CopyOnWriteArrayList`.
- `addIfAbsent(E e)`: Nếu phần tử chỉ định không tồn tại thì thêm phần tử đó. Nếu thêm thành công phần tử thì trả về true.

Ở đây lấy `add(E e)` làm ví dụ để giới thiệu:

```java
// Chèn phần tử vào cuối CopyOnWriteArrayList
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // Khóa lock
    lock.lock();
    try {
        // Lấy mảng ban đầu
        Object[] elements = getArray();
        // Độ dài mảng ban đầu
        int len = elements.length;
        // Tạo một mảng mới có độ dài +1, và copy các phần tử mảng ban đầu sang mảng mới
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // Đặt phần tử vào cuối mảng mới
        newElements[len] = e;
        // array trỏ đến mảng mới
        setArray(newElements);
        return true;
    } finally {
        // Giải phóng lock
        lock.unlock();
    }
}
```

Từ source code trên có thể thấy:

- Bên trong phương thức `add` sử dụng `ReentrantLock` để khóa, đảm bảo tính đồng bộ, tránh việc nhiều thread đồng thời thực thi thao tác ghi. Field lock được modifier `final` quy định, tham chiếu sau khi khởi tạo không thể trỏ sang đối tượng khác, đồng thời logic giải phóng lock được đặt trong `finally`, có thể đảm bảo lock được giải phóng.
- `CopyOnWriteArrayList` thực hiện thao tác ghi bằng cách copy mảng bên dưới, tức là tạo một mảng mới trước để chứa phần tử mới thêm vào, sau đó thực hiện thao tác ghi trên mảng mới, cuối cùng gán mảng mới cho tham chiếu mảng bên dưới, thay thế mảng cũ. Điều này chứng minh điều chúng ta đã nói ở trước: Trọng tâm thread-safe của `CopyOnWriteArrayList` nằm ở việc nó áp dụng chiến lược **Copy-On-Write (Viết khi sao chép)**.
- Mỗi lần thao tác ghi đều cần thông qua `Arrays.copyOf` để copy mảng bên dưới, độ phức tạp thời gian là O(n), và sẽ chiếm thêm dung lượng bộ nhớ. Do đó, `CopyOnWriteArrayList` áp dụng cho kịch bản đọc nhiều ghi ít, trong trường hợp thao tác ghi không thường xuyên và tài nguyên bộ nhớ dồi dào, có thể nâng cao hiệu năng của hệ thống.
- Trong `CopyOnWriteArrayList` không hề có thao tác mở rộng dung lượng tương tự phương thức `grow()` của `ArrayList`.

> Phương thức `Arrays.copyOf` có độ phức tạp thời gian là O(n), trong đó n biểu thị độ dài mảng cần sao chép. Vì nguyên lý triển khai của phương thức này là tạo một mảng mới trước, sau đó copy dữ liệu trong mảng nguồn sang mảng mới, cuối cùng trả về mảng mới. Phương thức này copy toàn bộ mảng, do đó độ phức tạp thời gian tỉ lệ thuận với độ dài mảng, tức O(n). Đáng chú ý là do bên dưới gọi lệnh copy ở cấp hệ thống, nên trong ứng dụng thực tế phương thức này thể hiện hiệu năng tương đối ưu tú, nhưng cũng cần chú ý kiểm soát dung lượng dữ liệu sao chép, tránh xuất hiện tình trạng chiếm dụng bộ nhớ quá cao.

### Đọc phần tử

Thao tác đọc của `CopyOnWriteArrayList` dựa trên mảng nội bộ `array` không xảy ra sự sửa đổi thực tế, do đó khi đọc không cần kiểm soát đồng bộ hóa hay thao tác lock, có thể đảm bảo an toàn dữ liệu. Dưới cơ chế này, nhiều thread có thể đồng thời đọc các phần tử trong list.

```java
// Mảng bên dưới, chỉ có thể truy cập qua phương thức getArray và setArray
private transient volatile Object[] array;

public E get(int index) {
    return get(getArray(), index);
}

final Object[] getArray() {
    return array;
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

Tuy nhiên, phương thức `get` có tính nhất quán yếu (weak consistency), trong một số trường hợp có thể đọc được giá trị phần tử cũ.

Phương thức `get(int index)` tiến hành theo 2 bước:

1. Thông qua `getArray()` lấy tham chiếu của mảng hiện tại;
2. Lấy trực tiếp phần tử có index trong mảng.

Quá trình này không lock, do đó trong môi trường concurrent có thể xuất hiện trường hợp sau:

1. Thread 1 gọi phương thức `get(int index)` để lấy giá trị, bên trong thông qua phương thức `getArray()` đã lấy được giá trị thuộc tính array;
2. Thread 2 gọi các phương thức sửa đổi như `add`, `set`, `remove` của `CopyOnWriteArrayList`, bên trong thông qua phương thức `setArray` sửa đổi giá trị thuộc tính `array`;
3. Thread 1 vẫn lấy giá trị từ mảng `array` cũ.

### Lấy số lượng phần tử trong list

```java
public int size() {
    return getArray().length;
}
```

Mảng `array` trong `CopyOnWriteArrayList` mỗi lần copy đều vừa vặn chứa đủ tất cả phần tử, không dự phòng không gian như `ArrayList`. Do đó, `CopyOnWriteArrayList` không hề có thuộc tính `size`. Độ dài mảng bên dưới của `CopyOnWriteArrayList` chính là số lượng phần tử, do đó phương thức `size()` chỉ cần trả về độ dài mảng là được.

### Xóa phần tử

Các phương thức liên quan đến xóa phần tử trong `CopyOnWriteArrayList` có 4 phương thức:

1. `remove(int index)`: Gỡ bỏ phần tử tại vị trí chỉ định trong list này. Dịch chuyển bất kỳ phần tử kế tiếp nào sang trái (giảm index của chúng đi 1).
2. `boolean remove(Object o)`: Xóa phần tử chỉ định xuất hiện lần đầu trong list này, nếu không tồn tại phần tử đó thì trả về false.
3. `boolean removeAll(Collection<?> c)`: Xóa tất cả phần tử chứa trong collection chỉ định khỏi list này.
4. `void clear()`: Gỡ bỏ tất cả phần tử trong list này.

Ở đây lấy `remove(int index)` làm ví dụ để giới thiệu:

```java
public E remove(int index) {
    // Lấy reentrant lock
    final ReentrantLock lock = this.lock;
    // Khóa lock
    lock.lock();
    try {
         // Lấy mảng array hiện tại
        Object[] elements = getArray();
        // Lấy độ dài array hiện tại
        int len = elements.length;
        // Lấy phần tử tại index chỉ định (giá trị cũ)
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        // Kiểm tra phần tử bị xóa có phải phần tử cuối cùng hay không
        if (numMoved == 0)
             // Nếu xóa phần tử cuối cùng, copy trực tiếp tất cả phần tử trước phần tử đó sang mảng mới
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            // Copy phân đoạn, copy phần tử trước index và phần tử sau index+1 sang mảng mới
            // Độ dài mảng mới là độ dài mảng cũ - 1
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            // Gán mảng mới cho tham chiếu array
            setArray(newElements);
        }
        return oldValue;
    } finally {
         // Giải phóng lock
        lock.unlock();
    }
}
```

### Kiểm tra phần tử có tồn tại hay không

`CopyOnWriteArrayList` cung cấp 2 phương thức dùng để kiểm tra phần tử chỉ định có trong list hay không:

- `contains(Object o)`: Kiểm tra xem có chứa phần tử chỉ định hay không.
- `containsAll(Collection<?> c)`: Kiểm tra xem có chứa toàn bộ phần tử của collection chỉ định hay không.

```java
// Kiểm tra xem có chứa phần tử chỉ định hay không
public boolean contains(Object o) {
    // Lấy mảng array hiện tại
    Object[] elements = getArray();
    // Gọi indexOf thử tìm phần tử chỉ định, nếu giá trị trả về lớn hơn hoặc bằng 0 thì trả về true, ngược lại trả về false
    return indexOf(o, elements, 0, elements.length) >= 0;
}

// Kiểm tra xem có chứa toàn bộ phần tử của collection chỉ định hay không
public boolean containsAll(Collection<?> c) {
    // Lấy mảng array hiện tại
    Object[] elements = getArray();
    // Lấy độ dài mảng
    int len = elements.length;
    // Duyệt collection chỉ định
    for (Object e : c) {
        // Vòng lặp gọi phương thức indexOf để kiểm tra, chỉ cần có 1 cái không chứa là trả về false luôn
        if (indexOf(e, elements, 0, len) < 0)
            return false;
    }
    // Cuối cùng biểu thị chứa toàn bộ hoặc collection chỉ định là collection rỗng, trả về true
    return true;
}
```

## Test các phương thức thường dùng của CopyOnWriteArrayList

Code:

```java
// Tạo đối tượng CopyOnWriteArrayList
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

// Thêm phần tử vào list
list.add("Java");
list.add("Python");
list.add("C++");
System.out.println("Danh sách ban đầu：" + list);

// Sử dụng phương thức get để lấy phần tử tại vị trí chỉ định
System.out.println("Phần tử thứ hai trong list là：" + list.get(1));

// Sử dụng phương thức remove để xóa phần tử chỉ định
boolean result = list.remove("C++");
System.out.println("Kết quả xóa：" + result);
System.out.println("List sau khi xóa phần tử：" + list);

// Sử dụng phương thức set để cập nhật phần tử tại vị trí chỉ định
list.set(1, "Golang");
System.out.println("List sau khi cập nhật：" + list);

// Sử dụng phương thức add để chèn phần tử vào vị trí chỉ định
list.add(0, "PHP");
System.out.println("List sau khi chèn phần tử：" + list);

// Sử dụng phương thức size để lấy kích thước list
System.out.println("Kích thước list là：" + list.size());

// Sử dụng phương thức removeAll để xóa tất cả phần tử xuất hiện trong collection chỉ định
result = list.removeAll(List.of("Java", "Golang"));
System.out.println("Kết quả xóa hàng loạt：" + result);
System.out.println("List sau khi xóa hàng loạt phần tử：" + list);

// Sử dụng phương thức clear để xóa sạch tất cả phần tử trong list
list.clear();
System.out.println("List sau khi xóa sạch：" + list);
```

Output:

```plain
List sau khi cập nhật：[Java, Golang]
List sau khi chèn phần tử：[PHP, Java, Golang]
Kích thước list là：3
Kết quả xóa hàng loạt：true
List sau khi xóa hàng loạt phần tử：[PHP]
List sau khi xóa sạch：[]
```

<!-- @include: @article-footer.snippet.md -->
