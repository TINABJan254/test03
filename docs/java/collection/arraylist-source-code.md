---
title: ArrayList 源码分析
description: ArrayList源码深度解析：详解ArrayList底层数组结构、1.5倍扩容机制、RandomAccess快速随机访问、序列化实现及与Vector性能对比。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: ArrayList源码,ArrayList扩容机制,动态数组,RandomAccess,ArrayList序列化,ArrayList与Vector区别
---

## Giới thiệu về ArrayList

`ArrayList` bên dưới là hàng đợi mảng (array queue), tương đương với mảng động. So với mảng trong Java, dung lượng của nó có thể tăng trưởng một cách động. Trước khi thêm lượng lớn phần tử, ứng dụng có thể sử dụng thao tác `ensureCapacity` để tăng dung lượng cho instance `ArrayList`. Điều này giúp giảm số lần tái phân bổ (reallocation) theo từng bước tăng dần.

`ArrayList` kế thừa từ `AbstractList`, triển khai các interface `List`, `RandomAccess`, `Cloneable`, `java.io.Serializable`.

```java

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{

  }
```

- `List`: Biểu thị nó là một danh sách, hỗ trợ các thao tác như thêm, xóa, tìm kiếm,... và có thể truy cập thông qua index.
- `RandomAccess`: Đây là một marker interface (interface đánh dấu), biểu thị collection `List` triển khai interface này hỗ trợ **truy cập ngẫu nhiên nhanh**. Trong `ArrayList`, chúng ta có thể thông qua index của phần tử để lấy nhanh đối tượng phần tử, đó chính là truy cập ngẫu nhiên nhanh.
- `Cloneable`: Biểu thị nó hỗ trợ sao chép thông qua phương thức `clone()`, `ArrayList#clone()` trả về bản sao nông (shallow copy).
- `Serializable`: Biểu thị nó có thể thực hiện thao tác serialization, tức là có thể chuyển đổi đối tượng thành byte stream để lưu trữ lâu dài hoặc truyền qua mạng, rất tiện lợi.

![Sơ đồ class ArrayList](https://oss.javaguide.cn/github/javaguide/java/collection/arraylist-class-diagram.png)

### Sự khác biệt giữa ArrayList và Vector? (Chỉ cần tham khảo)

- `ArrayList` là implementation chính của `List`, bên dưới sử dụng `Object[]` để lưu trữ, thích hợp cho việc tìm kiếm thường xuyên, không thread-safe.
- `Vector` là implementation cũ của `List`, bên dưới sử dụng `Object[]` để lưu trữ, thread-safe.

### ArrayList có thể thêm giá trị null không?

Trong `ArrayList` có thể lưu trữ bất kỳ đối tượng kiểu nào, bao gồm cả giá trị `null`. Tuy nhiên, không khuyến khích thêm giá trị `null` vào `ArrayList`, giá trị `null` không có ý nghĩa và làm cho code khó bảo trì, ví dụ nếu quên kiểm tra null sẽ dẫn đến `NullPointerException`.

Ví dụ code:

```java
ArrayList<String> listOfStrings = new ArrayList<>();
listOfStrings.add(null);
listOfStrings.add("java");
System.out.println(listOfStrings);
```

Output:

```plain
[null, java]
```

### Sự khác biệt giữa ArrayList và LinkedList?

- **Có đảm bảo thread-safe hay không:** Cả `ArrayList` và `LinkedList` đều không được đồng bộ hóa (unsynchronized), nghĩa là không đảm bảo thread-safe;
- **Cấu trúc dữ liệu bên dưới:** `ArrayList` sử dụng **mảng `Object`**; `LinkedList` sử dụng **danh sách liên kết đôi** (trước JDK1.6 là danh sách liên kết vòng, JDK1.7 đã hủy bỏ tính chất vòng. Chú ý sự khác biệt giữa danh sách liên kết đôi và danh sách liên kết vòng đôi!)
- **Thao tác chèn và xóa có bị ảnh hưởng bởi vị trí phần tử hay không:**
  - `ArrayList` sử dụng mảng để lưu trữ, do đó độ phức tạp thời gian khi chèn và xóa phần tử chịu ảnh hưởng bởi vị trí phần tử. Ví dụ: khi thực thi phương thức `add(E e)`, `ArrayList` mặc định sẽ thêm phần tử chỉ định vào cuối danh sách này, trường hợp này độ phức tạp thời gian là O(1). Nhưng nếu muốn chèn hoặc xóa phần tử tại vị trí chỉ định `i` (`add(int index, E element)`), độ phức tạp thời gian sẽ là O(n). Bởi vì khi thực hiện các thao tác trên, phần tử thứ `i` và (n-i) phần tử đằng sau phần tử thứ `i` trong collection đều phải thực hiện thao tác dịch chuyển lùi/tiến 1 vị trí.
  - `LinkedList` sử dụng danh sách liên kết để lưu trữ, nên việc chèn hoặc xóa phần tử ở đầu/cuối không bị ảnh hưởng bởi vị trí phần tử (`add(E e)`, `addFirst(E e)`, `addLast(E e)`, `removeFirst()`, `removeLast()`), độ phức tạp thời gian là O(1). Nếu muốn chèn hoặc xóa phần tử tại vị trí chỉ định `i` (`add(int index, E element)`, `remove(Object o)`, `remove(int index)`), độ phức tạp thời gian là O(n), vì cần phải di chuyển đến vị trí chỉ định trước rồi mới chèn/xóa.
- **Có hỗ trợ truy cập ngẫu nhiên nhanh hay không:** `LinkedList` không hỗ trợ truy cập phần tử ngẫu nhiên hiệu quả, còn `ArrayList` (triển khai interface `RandomAccess`) thì hỗ trợ. Truy cập ngẫu nhiên nhanh là việc thông qua index của phần tử để lấy nhanh đối tượng phần tử (tương ứng với phương thức `get(int index)`).
- **Dung lượng bộ nhớ tiêu tốn:** Việc lãng phí không gian của `ArrayList` chủ yếu thể hiện ở việc dự phòng một khoảng dung lượng nhất định ở cuối danh sách list, trong khi chi phí không gian của `LinkedList` thể hiện ở việc mỗi phần tử của nó đều tiêu tốn nhiều không gian hơn `ArrayList` (vì phải lưu trữ con trỏ trỏ đến phần tử kế tiếp, phần tử phía trước và dữ liệu).

## Phân tích source code cốt lõi của ArrayList

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code bên dưới của `ArrayList`.

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Dung lượng ban đầu mặc định
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Mảng rỗng (dùng cho các instance rỗng).
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // Instance mảng rỗng chia sẻ dùng cho các instance rỗng có kích thước mặc định.
    // Chúng ta phân biệt nó với mảng EMPTY_ELEMENTDATA để biết khi thêm phần tử đầu tiên cần tăng dung lượng lên bao nhiêu.
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * Mảng lưu trữ dữ liệu của ArrayList
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * Số lượng phần tử chứa trong ArrayList
     */
    private int size;

    /**
     * Constructor có tham số dung lượng ban đầu (người dùng có thể tự chỉ định kích thước ban đầu của collection khi tạo đối tượng ArrayList)
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            // Nếu tham số truyền vào lớn hơn 0, tạo mảng kích thước initialCapacity
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            // Nếu tham số truyền vào bằng 0, tạo mảng rỗng
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            // Trường hợp khác, ném ngoại lệ
            throw new IllegalArgumentException("Illegal Capacity: " +
                    initialCapacity);
        }
    }

    /**
     * Constructor mặc định không tham số
     * DEFAULTCAPACITY_EMPTY_ELEMENTDATA có kích thước là 0. Khởi tạo mặc định là 10, nghĩa là ban đầu thực chất là mảng rỗng, khi thêm phần tử đầu tiên dung lượng mảng mới trở thành 10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * Khởi tạo một danh sách chứa các phần tử của collection chỉ định, theo thứ tự trả về bởi iterator của collection đó.
     */
    public ArrayList(Collection<? extends E> c) {
        // Chuyển collection chỉ định thành mảng
        elementData = c.toArray();
        // Nếu độ dài mảng elementData khác 0
        if ((size = elementData.length) != 0) {
            // Nếu elementData không phải kiểu Object[] (c.toArray có thể trả về mảng không phải kiểu Object[] nên cần câu lệnh dưới đây để kiểm tra)
            if (elementData.getClass() != Object[].class)
                // Gán nội dung mảng elementData vốn không phải kiểu Object[] sang mảng elementData mới kiểu Object[]
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // Trường hợp khác, thay thế bằng mảng rỗng
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * Sửa đổi dung lượng của instance ArrayList này thành kích thước hiện tại của danh sách. Ứng dụng có thể dùng thao tác này để tối thiểu hóa dung lượng lưu trữ của instance ArrayList.
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
                    ? EMPTY_ELEMENTDATA
                    : Arrays.copyOf(elementData, size);
        }
    }
// Dưới đây là cơ chế mở rộng (resize/扩容) của ArrayList
// Cơ chế mở rộng của ArrayList nâng cao hiệu năng. Nếu mỗi lần chỉ mở rộng 1 phần tử,
// thì việc chèn thường xuyên sẽ dẫn đến copy thường xuyên làm giảm hiệu năng, và cơ chế mở rộng của ArrayList tránh được tình trạng này.

    /**
     * Tăng dung lượng instance ArrayList này nếu cần thiết để đảm bảo nó có thể chứa ít nhất số lượng phần tử chỉ định
     *
     * @param minCapacity Dung lượng tối thiểu cần thiết
     */
    public void ensureCapacity(int minCapacity) {
        // Nếu không phải mảng rỗng mặc định, giá trị minExpand là 0;
        // Nếu là mảng rỗng mặc định, giá trị minExpand là 10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                // Nếu không phải bảng phần tử mặc định, có thể dùng kích thước bất kỳ
                ? 0
                // Nếu là mảng rỗng mặc định, nó nên ở kích thước mặc định
                : DEFAULT_CAPACITY;

        // Nếu dung lượng tối thiểu lớn hơn dung lượng lớn nhất hiện có
        if (minCapacity > minExpand) {
            // Đảm bảo dung lượng đủ dựa trên dung lượng tối thiểu cần thiết
            ensureExplicitCapacity(minCapacity);
        }
    }


    // Tính toán dung lượng cần thiết dựa trên dung lượng tối thiểu cho trước và các phần tử mảng hiện tại.
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // Nếu mảng hiện tại là mảng rỗng (trường hợp khởi tạo), trả về giá trị lớn hơn giữa dung lượng mặc định và dung lượng tối thiểu làm dung lượng cần thiết
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        // Ngược lại trả về trực tiếp dung lượng tối thiểu
        return minCapacity;
    }

    // Đảm bảo dung lượng bên trong đạt đến dung lượng tối thiểu chỉ định.
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    // Kiểm tra xem có cần mở rộng dung lượng hay không
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            // Gọi phương thức grow để thực hiện mở rộng dung lượng, gọi phương thức này biểu thị đã bắt đầu mở rộng dung lượng
            grow(minCapacity);
    }

    /**
     * Kích thước mảng tối đa có thể cấp phát
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Phương thức cốt lõi cho việc mở rộng dung lượng ArrayList.
     */
    private void grow(int minCapacity) {
        // oldCapacity là dung lượng cũ, newCapacity là dung lượng mới
        int oldCapacity = elementData.length;
        // Dịch phải oldCapacity 1 bit, hiệu quả tương đương với oldCapacity / 2,
        // chúng ta biết tốc độ của phép toán bit nhanh hơn nhiều so với phép chia nguyên, kết quả của cả câu lệnh này là cập nhật dung lượng mới thành 1.5 lần dung lượng cũ,
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // Sau đó kiểm tra dung lượng mới có lớn hơn dung lượng tối thiểu cần thiết hay không, nếu vẫn nhỏ hơn thì lấy dung lượng tối thiểu cần thiết làm dung lượng mới của mảng,
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // Lại kiểm tra dung lượng mới có vượt quá dung lượng tối đa mà ArrayList định nghĩa hay không,
        // nếu vượt quá, gọi hugeCapacity() để so sánh minCapacity và MAX_ARRAY_SIZE,
        // nếu minCapacity lớn hơn MAX_ARRAY_SIZE thì dung lượng mới là Integer.MAX_VALUE, ngược lại dung lượng mới là MAX_ARRAY_SIZE.
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    // So sánh minCapacity và MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
                Integer.MAX_VALUE :
                MAX_ARRAY_SIZE;
    }

    /**
     * Trả về số lượng phần tử trong danh sách này.
     */
    public int size() {
        return size;
    }

    /**
     * Trả về true nếu danh sách này không chứa phần tử nào.
     */
    public boolean isEmpty() {
        // Chú ý sự khác biệt giữa = và ==
        return size == 0;
    }

    /**
     * Trả về true nếu danh sách này chứa phần tử chỉ định.
     */
    public boolean contains(Object o) {
        // Phương thức indexOf(): Trả về index xuất hiện lần đầu tiên của phần tử chỉ định trong danh sách này, nếu không chứa thì trả về -1
        return indexOf(o) >= 0;
    }

    /**
     * Trả về index xuất hiện lần đầu tiên của phần tử chỉ định trong danh sách này, nếu không chứa phần tử thì trả về -1
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i] == null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                // So sánh bằng phương thức equals()
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * Trả về index xuất hiện lần cuối cùng của phần tử chỉ định trong danh sách này, nếu không chứa phần tử thì trả về -1.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size - 1; i >= 0; i--)
                if (elementData[i] == null)
                    return i;
        } else {
            for (int i = size - 1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * Trả về bản sao nông (shallow copy) của instance ArrayList này. (Bản thân các phần tử không được sao chép.)
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            // Chức năng của Arrays.copyOf là thực hiện sao chép mảng, trả về mảng sau khi sao chép. Tham số là mảng được sao chép và độ dài sao chép
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // Điều này không nên xảy ra vì chúng ta có thể clone
            throw new InternalError(e);
        }
    }

    /**
     * Trả về mảng chứa tất cả phần tử trong danh sách này theo đúng thứ tự (từ phần tử đầu tiên đến phần tử cuối cùng).
     * Mảng trả về sẽ là "an toàn", vì danh sách này không giữ tham chiếu đến nó.
     * (Nói cách khác, phương thức này bắt buộc phải cấp phát một mảng mới).
     * Do đó, bên gọi có thể tự do sửa đổi cấu trúc mảng trả về.
     * Lưu ý: Nếu phần tử là kiểu tham chiếu (reference type), việc sửa đổi nội dung phần tử sẽ ảnh hưởng đến đối tượng trong danh sách ban đầu.
     * Phương thức này đóng vai trò cầu nối giữa API dựa trên mảng và API dựa trên collection.
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * Trả về mảng chứa tất cả phần tử trong danh sách này theo đúng thứ tự (từ phần tử đầu tiên đến phần tử cuối cùng);
     * Kiểu runtime của mảng trả về là kiểu runtime của mảng chỉ định. Nếu danh sách phù hợp với mảng chỉ định thì trả về mảng đó.
     * Ngược lại, sẽ cấp phát một mảng mới với kiểu runtime của mảng chỉ định và kích thước của danh sách này.
     * Nếu mảng chỉ định vừa vặn chứa danh sách, phần không gian còn lại (tức là số lượng mảng nhiều hơn phần tử danh sách) thì phần tử trong mảng ngay sau khi kết thúc collection sẽ được đặt thành null.
     * (Điều này chỉ giúp bên gọi xác định được độ dài danh sách trong trường hợp biết danh sách không chứa bất kỳ phần tử null nào.)
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Tạo một mảng kiểu runtime mới với nội dung là mảng ArrayList
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        // Gọi phương thức arraycopy() do System cung cấp để thực hiện sao chép giữa các mảng
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * Trả về phần tử tại vị trí chỉ định trong danh sách này.
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * Thay thế phần tử tại vị trí chỉ định trong danh sách này bằng phần tử chỉ định.
     */
    public E set(int index, E element) {
        // Kiểm tra giới hạn index
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        // Trả về phần tử vốn ở vị trí này
        return oldValue;
    }

    /**
     * Thêm phần tử chỉ định vào cuối danh sách này.
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // Ở đây có thể thấy bản chất việc thêm phần tử của ArrayList tương đương với việc gán giá trị cho mảng
        elementData[size++] = e;
        return true;
    }

    /**
     * Chèn phần tử chỉ định vào vị trí chỉ định trong danh sách này.
     * Trước tiên gọi rangeCheckForAdd để kiểm tra giới hạn index; sau đó gọi phương thức ensureCapacityInternal để đảm bảo capacity đủ lớn;
     * Sau đó dịch tất cả thành phần từ index trở đi lùi về sau một vị trí; chèn element vào vị trí index; cuối cùng size cộng 1.
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // Phương thức arraycopy() thực hiện sao chép giữa các mảng nhất định nên xem qua, bên dưới sử dụng arraycopy() để mảng tự copy chính mình
        System.arraycopy(elementData, index, elementData, index + 1,
                size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * Xóa phần tử tại vị trí chỉ định trong danh sách này. Dịch chuyển bất kỳ phần tử kế tiếp nào sang bên trái (giảm index của nó đi 1).
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index + 1, elementData, index,
                    numMoved);
        elementData[--size] = null; // clear to let GC do its work
        // Phần tử bị xóa khỏi danh sách
        return oldValue;
    }

    /**
     * Xóa lần xuất hiện đầu tiên của phần tử chỉ định khỏi danh sách (nếu tồn tại). Nếu danh sách không chứa phần tử đó thì sẽ không thay đổi.
     * Trả về true nếu danh sách này chứa phần tử chỉ định
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Phương thức này là phương thức xóa private, bỏ qua kiểm tra ranh giới và không trả về giá trị bị xóa.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index + 1, elementData, index,
                    numMoved);
        elementData[--size] = null; // Sau khi xóa phần tử, đặt vị trí đó thành null để Garbage Collector (GC) có thể thu gom phần tử đó.
    }

    /**
     * Xóa tất cả phần tử khỏi danh sách.
     */
    public void clear() {
        modCount++;

        // Đặt giá trị tất cả phần tử trong mảng thành null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * Thêm tất cả phần tử trong collection chỉ định vào cuối danh sách này theo thứ tự trả về từ Iterator của collection chỉ định.
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * Chèn tất cả phần tử trong collection chỉ định vào danh sách này, bắt đầu từ vị trí chỉ định.
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                    numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * Xóa tất cả phần tử có index từ fromIndex (bao gồm) đến toIndex khỏi danh sách này.
     * Dịch chuyển bất kỳ phần tử kế tiếp nào sang bên trái (giảm index của nó).
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex - fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * Kiểm tra index cho trước có nằm trong phạm vi hay không.
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * Một phiên bản rangeCheck được add và addAll sử dụng
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * Trả về thông tin chi tiết IndexOutOfBoundsException
     */
    private String outOfBoundsMsg(int index) {
        return "Index: " + index + ", Size: " + size;
    }

    /**
     * Xóa tất cả phần tử chứa trong collection chỉ định khỏi danh sách này.
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        // Trả về true nếu danh sách này bị sửa đổi
        return batchRemove(c, false);
    }

    /**
     * Chỉ giữ lại các phần tử trong danh sách này mà có chứa trong collection chỉ định.
     * Nói cách khác, xóa tất cả phần tử trong danh sách này không có trong collection chỉ định.
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }


    /**
     * Trả về list iterator cho các phần tử trong danh sách (theo thứ tự đúng), bắt đầu từ vị trí chỉ định trong danh sách.
     * Index chỉ định biểu thị phần tử đầu tiên mà lần gọi ban đầu next sẽ trả về. Lần gọi ban đầu previous sẽ trả về phần tử tại index chỉ định trừ 1.
     * List iterator trả về là fail-fast .
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: " + index);
        return new ListItr(index);
    }

    /**
     * Trả về list iterator trong danh sách (theo thứ tự thích hợp).
     * List iterator trả về là fail-fast .
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * Trả về iterator cho các phần tử trong danh sách này theo đúng thứ tự.
     * Iterator trả về là fail-fast .
     */
    public Iterator<E> iterator() {
        return new Itr();
    }
```

## Phân tích cơ chế mở rộng (resize/扩容) của ArrayList

### Bắt đầu từ constructor của ArrayList

ArrayList có 3 cách để khởi tạo, source code constructor như sau (JDK8):

```java
/**
 * Dung lượng ban đầu mặc định
 */
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * Constructor mặc định, sử dụng dung lượng ban đầu 10 để tạo một list rỗng (constructor không tham số)
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * Constructor có tham số dung lượng ban đầu (Người dùng tự chỉ định dung lượng)
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {// Dung lượng ban đầu lớn hơn 0
        // Tạo mảng kích thước initialCapacity
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {// Dung lượng ban đầu bằng 0
        // Tạo mảng rỗng
        this.elementData = EMPTY_ELEMENTDATA;
    } else {// Dung lượng ban đầu nhỏ hơn 0, ném ngoại lệ
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}


/**
 * Khởi tạo list chứa các phần tử của collection chỉ định theo thứ tự trả về bởi iterator của collection đó
 * Nếu collection chỉ định là null, throws NullPointerException.
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

Những bạn tinh ý chắc chắn sẽ phát hiện ra: **Khi tạo `ArrayList` bằng constructor không tham số, thực chất khởi tạo được gán là một mảng rỗng. Chỉ khi thực sự thực hiện thao tác thêm phần tử vào mảng thì mới thực sự phân bổ dung lượng. Tức là khi thêm phần tử đầu tiên vào mảng, dung lượng mảng mới mở rộng thành 10.** Phần dưới đây khi phân tích việc mở rộng dung lượng của `ArrayList` chúng ta sẽ nói đến điểm này!

> Bổ sung: Trong JDK6 khi `new` đối tượng `ArrayList` bằng constructor không tham số, nó trực tiếp tạo mảng `Object[]` `elementData` có độ dài 10.

### Phân tích từng bước cơ chế mở rộng của ArrayList

Ở đây lấy `ArrayList` tạo bởi constructor không tham số làm ví dụ phân tích.

#### Phương thức add

```java
/**
* Thêm phần tử chỉ định vào cuối danh sách này.
*/
public boolean add(E e) {
    // Trước khi thêm phần tử, gọi phương thức ensureCapacityInternal trước
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // Ở đây có thể thấy bản chất việc thêm phần tử của ArrayList tương đương với việc gán giá trị cho mảng
    elementData[size++] = e;
    return true;
}
```

**Lưu ý**: JDK11 đã loại bỏ các phương thức `ensureCapacityInternal()` và `ensureExplicitCapacity()`.

Source code phương thức `ensureCapacityInternal` như sau:

```java
// Tính toán dung lượng cần thiết dựa trên dung lượng tối thiểu cho trước và các phần tử mảng hiện tại.
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // Nếu mảng hiện tại là mảng rỗng (trường hợp khởi tạo), trả về giá trị lớn hơn giữa dung lượng mặc định và dung lượng tối thiểu làm dung lượng cần thiết
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // Ngược lại trả về trực tiếp dung lượng tối thiểu
    return minCapacity;
}

// Đảm bảo dung lượng bên trong đạt đến dung lượng tối thiểu chỉ định.
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

Phương thức `ensureCapacityInternal` rất đơn giản, bên trong trực tiếp gọi phương thức `ensureExplicitCapacity`:

```java
// Kiểm tra xem có cần mở rộng dung lượng hay không
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // Kiểm tra dung lượng mảng hiện tại có đủ để lưu trữ minCapacity phần tử hay không
    if (minCapacity - elementData.length > 0)
        // Gọi phương thức grow để mở rộng dung lượng
        grow(minCapacity);
}
```

Chúng ta hãy cùng phân tích kỹ một chút:

- Khi chúng ta muốn `add` phần tử thứ 1 vào `ArrayList`, `elementData.length` bằng 0 (vì vẫn là một list rỗng), do thực thi phương thức `ensureCapacityInternal()`, nên `minCapacity` lúc này là 10. Lúc này `minCapacity - elementData.length > 0` thỏa mãn, do đó sẽ đi vào phương thức `grow(minCapacity)`.
- Khi `add` phần tử thứ 2, `minCapacity` bằng 2, lúc này `elementData.length` (dung lượng) sau khi thêm phần tử đầu tiên đã được mở rộng thành `10`. Lúc này `minCapacity - elementData.length > 0` không thỏa mãn, nên không đi vào (thực thi) phương thức `grow(minCapacity)`.
- Khi thêm phần tử thứ 3, 4,... đến phần tử thứ 10, vẫn sẽ không thực thi phương thức grow, dung lượng mảng đều là 10.

Cho đến khi thêm phần tử thứ 11, `minCapacity` (bằng 11) lớn hơn `elementData.length` (bằng 10). Mới đi vào phương thức `grow` để mở rộng dung lượng.

#### Phương thức grow

```java
/**
 * Kích thước mảng tối đa có thể cấp phát
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * Phương thức cốt lõi cho việc mở rộng dung lượng ArrayList.
 */
private void grow(int minCapacity) {
    // oldCapacity là dung lượng cũ, newCapacity là dung lượng mới
    int oldCapacity = elementData.length;
    // Dịch phải oldCapacity 1 bit, hiệu quả tương đương với oldCapacity / 2,
    // chúng ta biết tốc độ của phép toán bit nhanh hơn nhiều so với phép chia nguyên, kết quả của cả câu lệnh này là cập nhật dung lượng mới thành 1.5 lần dung lượng cũ,
    int newCapacity = oldCapacity + (oldCapacity >> 1);

    // Sau đó kiểm tra dung lượng mới có lớn hơn dung lượng tối thiểu cần thiết hay không, nếu vẫn nhỏ hơn thì lấy dung lượng tối thiểu cần thiết làm dung lượng mới của mảng,
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;

    // Nếu dung lượng mới lớn hơn MAX_ARRAY_SIZE, đi vào (thực thi) phương thức `hugeCapacity()` để so sánh minCapacity và MAX_ARRAY_SIZE,
    // nếu minCapacity lớn hơn dung lượng tối đa thì dung lượng mới là `Integer.MAX_VALUE`, ngược lại dung lượng mới là MAX_ARRAY_SIZE tức là `Integer.MAX_VALUE - 8`.
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);

    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**`int newCapacity = oldCapacity + (oldCapacity >> 1)`, do đó mỗi lần ArrayList mở rộng dung lượng thì dung lượng mới đều biến thành khoảng 1.5 lần dung lượng cũ (oldCapacity là số chẵn thì đúng 1.5 lần, nếu không thì khoảng 1.5 lần)!** Chẵn lẻ khác nhau, ví dụ: 10+10/2 = 15, 33+33/2 = 49. Nếu là số lẻ thì phần thập phân sẽ bị bỏ qua.

> ">>" (Toán tử dịch bit / shift operator): `>>1` dịch phải 1 bit tương đương chia cho 2, dịch phải n bit tương đương chia cho 2 mũ n. Ở đây `oldCapacity` rõ ràng dịch phải 1 bit nên tương đương `oldCapacity / 2`. Đối với tính toán nhị phân dữ liệu lớn, toán tử dịch bit nhanh hơn nhiều so với toán tử chia thông thường, vì chương trình chỉ đơn giản là di chuyển bit chứ không tính toán, giúp nâng cao hiệu năng và tiết kiệm tài nguyên.

**Chúng ta cùng tìm hiểu kỹ hơn phương thức grow() thông qua ví dụ:**

- Khi `add` phần tử thứ 1, `oldCapacity` bằng 0, sau khi so sánh kiểm tra `if` đầu tiên thỏa mãn, `newCapacity = minCapacity` (bằng 10). Nhưng kiểm tra `if` thứ hai không thỏa mãn, tức là `newCapacity` không lớn hơn `MAX_ARRAY_SIZE`, thì sẽ không đi vào phương thức `hugeCapacity`. Dung lượng mảng là 10, phương thức `add` return true, size tăng lên 1.
- Khi `add` phần tử thứ 11 đi vào phương thức `grow`, `newCapacity` bằng 15, lớn hơn `minCapacity` (bằng 11), kiểm tra `if` đầu tiên không thỏa mãn. Dung lượng mới không lớn hơn size tối đa của mảng, không đi vào phương thức `hugeCapacity`. Dung lượng mảng mở rộng thành 15, phương thức `add` return true, size tăng lên 11.
- Tương tự suy ra...

**Ở đây bổ sung một kiến thức tương đối quan trọng nhưng dễ bị bỏ qua:**

- Thuộc tính `length` trong Java là dành cho mảng, ví dụ khi bạn khai báo một mảng, muốn biết chiều dài mảng đó thì dùng thuộc tính `length`.
- Phương thức `length()` trong Java là dành cho chuỗi (String), nếu muốn xem chiều dài chuỗi đó thì dùng phương thức `length()`.
- Phương thức `size()` trong Java là dành cho generic collection, nếu muốn xem collection này có bao nhiêu phần tử thì gọi phương thức này để xem!

#### Phương thức hugeCapacity()

Từ source code phương thức `grow()` ở trên chúng ta biết: Nếu dung lượng mới lớn hơn `MAX_ARRAY_SIZE`, sẽ đi vào (thực thi) phương thức `hugeCapacity()` để so sánh `minCapacity` và `MAX_ARRAY_SIZE`. Nếu `minCapacity` lớn hơn dung lượng tối đa thì dung lượng mới sẽ là `Integer.MAX_VALUE`, ngược lại kích thước dung lượng mới là `MAX_ARRAY_SIZE` tức là `Integer.MAX_VALUE - 8`.

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    // So sánh minCapacity và MAX_ARRAY_SIZE
    // Nếu minCapacity lớn hơn, lấy Integer.MAX_VALUE làm kích thước mảng mới
    // Nếu MAX_ARRAY_SIZE lớn hơn, lấy MAX_ARRAY_SIZE làm kích thước mảng mới
    // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

### Phương thức `System.arraycopy()` và `Arrays.copyOf()`

Nếu đọc source code, chúng ta sẽ phát hiện `ArrayList` gọi rất nhiều hai phương thức này. Ví dụ: Thao tác mở rộng dung lượng đã nói ở trên cũng như các phương thức `add(int index, E element)`, `toArray()`,... đều sử dụng phương thức này!

#### Phương thức `System.arraycopy()`

Source code:

```java
    // Chúng ta thấy arraycopy là một native method, tiếp theo chúng ta giải thích ý nghĩa cụ thể của từng tham số
    /**
    * Sao chép mảng
    * @param src Mảng nguồn
    * @param srcPos Vị trí bắt đầu trong mảng nguồn
    * @param dest Mảng đích
    * @param destPos Vị trí bắt đầu trong mảng đích
    * @param length Số lượng phần tử mảng cần sao chép
    */
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

Kịch bản sử dụng:

```java
    /**
     * Chèn phần tử chỉ định vào vị trí chỉ định trong danh sách này.
     * Trước tiên gọi rangeCheckForAdd để kiểm tra giới hạn index; sau đó gọi phương thức ensureCapacityInternal để đảm bảo capacity đủ lớn;
     * Sau đó dịch tất cả thành phần từ index trở đi lùi về sau một vị trí; chèn element vào vị trí index; cuối cùng size cộng 1.
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // Phương thức arraycopy() để mảng tự copy chính mình
        // elementData: Mảng nguồn; index: Vị trí bắt đầu trong mảng nguồn; elementData: Mảng đích; index + 1: Vị trí bắt đầu trong mảng đích; size - index: Số lượng phần tử mảng cần sao chép;
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
    }
```

Chúng ta viết một phương thức đơn giản để test:

```java
public class ArraycopyTest {

  public static void main(String[] args) {
    // TODO Auto-generated method stub
    int[] a = new int[10];
    a[0] = 0;
    a[1] = 1;
    a[2] = 2;
    a[3] = 3;
    System.arraycopy(a, 2, a, 3, 3);
    a[2]=99;
    for (int i = 0; i < a.length; i++) {
      System.out.print(a[i] + " ");
    }
  }

}
```

Kết quả:

```plain
0 1 99 2 3 0 0 0 0 0
```

#### Phương thức `Arrays.copyOf()`

Source code:

```java
    public static int[] copyOf(int[] original, int newLength) {
      // Xin cấp phát một mảng mới
        int[] copy = new int[newLength];
  // Gọi System.arraycopy để copy dữ liệu trong mảng nguồn và trả về mảng mới
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

Kịch bản sử dụng:

```java
   /**
     Trả về mảng chứa tất cả phần tử trong danh sách này theo đúng thứ tự (từ phần tử đầu tiên đến phần tử cuối cùng); Kiểu runtime của mảng trả về là kiểu runtime của mảng chỉ định.
     */
    public Object[] toArray() {
    // elementData: Mảng cần sao chép; size: Độ dài cần sao chép
        return Arrays.copyOf(elementData, size);
    }
```

Cá nhân tôi thấy việc sử dụng phương thức `Arrays.copyOf()` chủ yếu là để mở rộng dung lượng mảng ban đầu, code test như sau:

```java
public class ArrayscopyOfTest {

  public static void main(String[] args) {
    int[] a = new int[3];
    a[0] = 0;
    a[1] = 1;
    a[2] = 2;
    int[] b = Arrays.copyOf(a, 10);
    System.out.println("b.length"+b.length);
  }
}
```

Kết quả:

```plain
10
```

#### Mối liên hệ và sự khác biệt giữa cả hai

**Mối liên hệ:**

Xem source code của cả hai có thể thấy bên trong `copyOf()` thực chất gọi phương thức `System.arraycopy()`.

**Sự khác biệt:**

`arraycopy()` cần mảng đích, copy mảng gốc sang mảng bạn tự định nghĩa hoặc chính mảng gốc, và có thể chọn điểm bắt đầu, độ dài copy cũng như vị trí đặt vào mảng mới; còn `copyOf()` là hệ thống tự động tạo một mảng mới bên trong và trả về mảng đó.

### Phương thức `ensureCapacity`

Trong source code `ArrayList` có phương thức `ensureCapacity` không biết mọi người có chú ý hay không, phương thức này bên trong `ArrayList` chưa từng được gọi, nên rõ ràng là cung cấp cho người dùng gọi. Vậy phương thức này có tác dụng gì?

```java
    /**
    Tăng dung lượng của instance ArrayList này nếu cần thiết để đảm bảo nó có thể chứa ít nhất số lượng phần tử được chỉ định bởi tham số minimum capacity.
     *
     * @param   minCapacity   Dung lượng tối thiểu cần thiết
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

```

Về mặt lý thuyết, tốt nhất trước khi thêm lượng lớn phần tử vào `ArrayList` hãy dùng phương thức `ensureCapacity` để giảm số lần tái phân bổ tăng dần.

Chúng ta kiểm tra thực tế hiệu quả phương thức này qua đoạn code dưới đây:

```java
public class EnsureCapacityTest {
  public static void main(String[] args) {
    ArrayList<Object> list = new ArrayList<Object>();
    final int N = 10000000;
    long startTime = System.currentTimeMillis();
    for (int i = 0; i < N; i++) {
      list.add(i);
    }
    long endTime = System.currentTimeMillis();
    System.out.println("Trước khi dùng phương thức ensureCapacity: "+(endTime - startTime));

  }
}
```

Kết quả chạy:

```plain
Trước khi dùng phương thức ensureCapacity: 2158
```

```java
public class EnsureCapacityTest {
    public static void main(String[] args) {
        ArrayList<Object> list = new ArrayList<Object>();
        final int N = 10000000;
        long startTime1 = System.currentTimeMillis();
        list.ensureCapacity(N);
        for (int i = 0; i < N; i++) {
            list.add(i);
        }
        long endTime1 = System.currentTimeMillis();
        System.out.println("Sau khi dùng phương thức ensureCapacity: "+(endTime1 - startTime1));
    }
}
```

Kết quả chạy:

```plain
Sau khi dùng phương thức ensureCapacity: 1773
```

Thông qua kết quả chạy, chúng ta có thể thấy trước khi thêm lượng lớn phần tử vào `ArrayList` việc sử dụng phương thức `ensureCapacity` có thể nâng cao hiệu năng. Tuy nhiên sự chênh lệch hiệu năng này hầu như có thể bỏ qua. Hơn nữa trong dự án thực tế cũng không thể thêm nhiều phần tử như vậy vào `ArrayList`.

<!-- @include: @article-footer.snippet.md -->
