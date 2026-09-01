---
title: Java 常见并发容器总结
description: Java并发容器全面总结：详解ConcurrentHashMap/CopyOnWriteArrayList/BlockingQueue等JUC线程安全容器特性、适用场景与性能对比。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: Java并发容器,ConcurrentHashMap,CopyOnWriteArrayList,BlockingQueue,ConcurrentLinkedQueue,线程安全容器
---

Hầu hết các container do JDK cung cấp nằm trong gói `java.util.concurrent`.

- **`ConcurrentHashMap`** : `HashMap` an toàn luồng
- **`CopyOnWriteArrayList`** : `List` an toàn luồng, trong kịch bản đọc nhiều ghi ít có hiệu năng rất tốt, tốt hơn nhiều so với `Vector`.
- **`ConcurrentLinkedQueue`** : Hàng đợi concurrency hiệu quả cao, triển khai bằng danh sách liên kết. Có thể xem như một `LinkedList` an toàn luồng, đây là một hàng đợi non-blocking.
- **`BlockingQueue`** : Đây là một interface, bên trong JDK thông qua danh sách liên kết, mảng... để triển khai interface này. Đại diện cho hàng đợi chặn, rất phù hợp dùng làm kênh chia sẻ dữ liệu.
- **`ConcurrentSkipListMap`** : Triển khai của SkipList (danh sách nhảy). Đây là một Map, sử dụng cấu trúc dữ liệu SkipList để tìm kiếm nhanh.

## ConcurrentHashMap

Chúng ta đều biết, `HashMap` là không an toàn luồng, nếu sử dụng trong kịch bản concurrency, một cách giải quyết phổ biến là thông qua phương thức `Collections.synchronizedMap()` để bọc `HashMap`, làm cho nó trở thành an toàn luồng. Tuy nhiên, cách làm này thông qua một khóa toàn cục để đồng bộ hóa việc truy cập concurrency giữa các luồng khác nhau, dẫn đến nút thắt hiệu năng nghiêm trọng, đặc biệt trong kịch bản concurrency cao.

Để giải quyết vấn đề này, `ConcurrentHashMap` ra đời, với vai trò là phiên bản an toàn luồng của `HashMap`, nó cung cấp năng lực xử lý concurrency hiệu quả hơn.

Trong JDK 1.7, `ConcurrentHashMap` tiến hành phân đoạn mảng bucket (`Segment`, khóa phân đoạn), mỗi một ổ khóa chỉ khóa một phần dữ liệu trong container (dưới đây có sơ đồ minh họa), nhiều luồng truy cập dữ liệu ở các đoạn dữ liệu khác nhau trong container sẽ không tồn tại tranh chấp khóa, nâng cao tỷ lệ truy cập concurrency.

![Java7 ConcurrentHashMap 存储结构](https://oss.javaguide.cn/github/javaguide/java/collection/java7_concurrenthashmap.png)

Đến JDK 1.8, `ConcurrentHashMap` đã hủy bỏ khóa phân đoạn `Segment`, áp dụng `Node + CAS + synchronized` để đảm bảo an toàn concurrency. Cấu trúc dữ liệu tương tự như cấu trúc `HashMap` 1.8, mảng + danh sách liên kết / cây nhị phân đỏ đen. Java 8 khi chiều dài danh sách liên kết vượt quá một ngưỡng nhất định (8) sẽ chuyển đổi danh sách liên kết (độ phức tạp thời gian tìm kiếm O(N)) thành cây đỏ đen (độ phức tạp thời gian tìm kiếm O(log(N))).

Trong Java 8, độ mịn của khóa đối với thao tác cập nhật tinh tế hơn: Khi cần cài khóa, `synchronized` chỉ khóa node đầu tiên của bucket tương ứng, các thao tác cập nhật trên các bucket khác nhau thường có thể tiến hành đồng thời. Thao tác đọc nhìn chung không cần cài khóa, cũng có thể tiến hành đồng thời với thao tác cập nhật.

![Java8 ConcurrentHashMap 存储结构](https://oss.javaguide.cn/github/javaguide/java/collection/java8_concurrenthashmap.png)

Về bài giới thiệu chi tiết `ConcurrentHashMap`, xin xem bài viết tôi viết tại đây: [Phân tích mã nguồn `ConcurrentHashMap`](./../collection/concurrent-hash-map-source-code.md).

## CopyOnWriteArrayList

Trực thuộc JDK 1.5 trước khi đưa vào `CopyOnWriteArrayList`, ngoài `Vector` thời kỳ đầu, cũng có thể thông qua `Collections.synchronizedList()` bọc `List` thông thường để có được năng lực truy cập đồng bộ. Các phương thức thêm xóa sửa truy vấn của `Vector` về cơ bản đều đã thêm `synchronized`, đơn lời gọi phương thức sở hữu tính an toàn luồng, nhưng các thao tác phức hợp vẫn cần đồng bộ hóa bổ sung.

JDK 1.5 đưa vào gói `java.util.concurrent` (JUC), trong đó cung cấp rất nhiều container an toàn luồng và có hiệu năng concurrency tốt, trong đó triển khai `List` an toàn luồng duy nhất chính là `CopyOnWriteArrayList`.

Đối với hầu hết các kịch bản nghiệp vụ, thao tác đọc thường lớn hơn nhiều so với thao tác ghi. Do thao tác đọc không làm sửa đổi dữ liệu ban đầu, do đó, đối với mỗi lần đọc đều tiến hành cài khóa thực chất là một sự lãng phí tài nguyên. So sánh lại, chúng ta nên cho phép nhiều luồng đồng thời truy cập dữ liệu bên trong `List`, dù sao đối với thao tác đọc mà nói là an toàn.

Tư tưởng này rất tương đồng với tư tưởng thiết kế khóa đọc ghi `ReentrantReadWriteLock`, tức là Đọc-Đọc không loại trừ, Đọc-Ghi loại trừ, Ghi-Ghi loại trừ (chỉ có Đọc-Đọc không loại trừ). `CopyOnWriteArrayList` tiến thêm một bước nữa triển khai tư tưởng này. Để phát huy hiệu năng thao tác đọc đến mức tối đa, thao tác đọc trong `CopyOnWriteArrayList` hoàn toàn không cần cài khóa. Lợi hại hơn nữa là, thao tác ghi cũng không làm chặn thao tác đọc, chỉ có Ghi-Ghi mới loại trừ nhau. Nhờ đó, hiệu năng của thao tác đọc có thể được nâng cao với biên độ lớn.

Cốt lõi an toàn luồng của `CopyOnWriteArrayList` nằm ở việc nó áp dụng chiến lược **Sao chép khi ghi (Copy-On-Write)**, từ tên gọi của `CopyOnWriteArrayList` cũng có thể thấy được.

Khi cần sửa đổi (thao tác `add`, `set`, `remove`...) nội dung của `CopyOnWriteArrayList`, sẽ không trực tiếp sửa đổi mảng gốc, mà trước tiên tạo bản sao của mảng bên dưới, tiến hành sửa đổi trên mảng bản sao, sau khi sửa đổi xong lại gán mảng đã sửa đổi trở lại, như vậy có thể đảm bảo thao tác ghi không ảnh hưởng đến thao tác đọc.

Về bài giới thiệu chi tiết `CopyOnWriteArrayList`, xin xem bài viết tôi viết tại đây: [Phân tích mã nguồn `CopyOnWriteArrayList`](./../collection/copyonwritearraylist-source-code.md).

## ConcurrentLinkedQueue

`Queue` an toàn luồng do Java cung cấp có thể chia thành **hàng đợi chặn (blocking queue)** và **hàng đợi non-blocking**, trong đó ví dụ điển hình của hàng đợi chặn là `BlockingQueue`, ví dụ điển hình của hàng đợi non-blocking là `ConcurrentLinkedQueue`, trong ứng dụng thực tế phải lựa chọn hàng đợi chặn hoặc hàng đợi non-blocking dựa theo nhu cầu thực tế. **Hàng đợi chặn có thể triển khai bằng cách cài khóa, hàng đợi non-blocking có thể triển khai bằng thao tác CAS.**

Từ tên gọi có thể thấy, hàng đợi `ConcurrentLinkedQueue` này sử dụng danh sách liên kết làm cấu trúc dữ liệu của nó. `ConcurrentLinkedQueue` nên được coi là hàng đợi có hiệu năng tốt nhất trong môi trường concurrency cao. Lý do nó có thể có hiệu năng rất tốt là vì triển khai phức tạp bên trong của nó.

Code bên trong `ConcurrentLinkedQueue` chúng ta không phân tích ở đây, mọi người chỉ cần biết `ConcurrentLinkedQueue` chủ yếu sử dụng thuật toán non-blocking CAS để triển khai an toàn luồng là được.

`ConcurrentLinkedQueue` phù hợp với kịch bản có yêu cầu tương đối cao về hiệu năng, đồng thời việc đọc ghi hàng đợi tồn tại nhiều luồng đồng thời tiến hành, tức là nếu chi phí cài khóa đối với hàng đợi tương đối cao thì phù hợp dùng `ConcurrentLinkedQueue` không khóa để thay thế.

## BlockingQueue

### Giới thiệu BlockingQueue

Phía trên chúng ta đã đề cập đến `ConcurrentLinkedQueue` với vai trò hàng đợi non-blocking hiệu năng cao. Dưới đây chúng ta sẽ nói về hàng đợi chặn — `BlockingQueue`. Hàng đợi chặn (`BlockingQueue`) được sử dụng rộng rãi trong bài toán "Người sản xuất - Người tiêu thụ" (Producer-Consumer), nguyên nhân là `BlockingQueue` cung cấp các phương thức chèn và xóa có thể bị chặn. Khi container hàng đợi đã đầy, luồng producer sẽ bị chặn cho đến khi hàng đợi hết đầy; khi container hàng đợi rỗng, luồng consumer sẽ bị chặn cho đến khi hàng đợi không rỗng mới thôi.

`BlockingQueue` là một interface, kế thừa từ `Queue`, do đó các lớp triển khai của nó cũng có thể đóng vai trò như lớp triển khai của `Queue` để sử dụng, mà `Queue` lại kế thừa từ interface `Collection`. Dưới đây là các lớp triển khai liên quan của `BlockingQueue`:

![BlockingQueue 的实现类](https://oss.javaguide.cn/github/javaguide/java/51622268.jpg)

Dưới đây chủ yếu giới thiệu 3 lớp triển khai `BlockingQueue` thường gặp: `ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`.

### ArrayBlockingQueue

`ArrayBlockingQueue` là lớp triển khai hàng đợi hữu hạn của interface `BlockingQueue`, bên dưới áp dụng mảng để triển khai.

```java
public class ArrayBlockingQueue<E>
extends AbstractQueue<E>
implements BlockingQueue<E>, Serializable{}
```

`ArrayBlockingQueue` một khi đã tạo ra thì dung lượng không thể thay đổi. Việc kiểm soát concurrency của nó áp dụng khóa reentrant `ReentrantLock`, bất kể là thao tác chèn hay thao tác đọc đều cần lấy được khóa mới có thể tiến hành thao tác. Khi dung lượng hàng đợi đầy, thử đặt phần tử vào hàng đợi sẽ dẫn đến thao tác bị chặn; thử lấy một phần tử từ một hàng đợi rỗng cũng sẽ bị chặn tương tự.

`ArrayBlockingQueue` mặc định không thể đảm bảo tính công bằng truy cập hàng đợi của các luồng chờ đợi. Sau khi bật chiến lược công bằng, hàng đợi khi tồn tại tranh chấp sẽ trao quyền truy cập cho producer hoặc consumer đang chờ theo thứ tự FIFO; điều này mô tả thứ tự chờ đợi trong hàng đợi, không tương đồng với việc hệ điều hành điều phối luồng nghiêm ngặt theo wall-clock time. Dưới mode không công bằng, luồng bị chặn thời gian dài có thể vẫn không lấy được quyền truy cập hàng đợi kịp thời. Chiến lược công bằng thường sẽ giảm throughput, nếu cần có thể áp dụng code như sau:

```java
private static ArrayBlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<Integer>(10,true);
```

### LinkedBlockingQueue

`LinkedBlockingQueue` bên dưới dựa trên **danh sách liên kết một chiều** để triển khai hàng đợi chặn, có thể dùng như hàng đợi vô hạn cũng như hàng đợi hữu hạn, tương tự thỏa mãn đặc tính FIFO, so sánh với `ArrayBlockingQueue` sở hữu throughput cao hơn, để phòng ngừa dung lượng `LinkedBlockingQueue` tăng nhanh làm tiêu tốn lượng lớn bộ nhớ. Thường khi tạo đối tượng `LinkedBlockingQueue`, sẽ chỉ định kích thước của nó, nếu không chỉ định, dung lượng bằng `Integer.MAX_VALUE`.

**Các constructor liên quan:**

```java
    /**
     * Hàng đợi vô hạn theo một ý nghĩa nào đó
     * Creates a {@code LinkedBlockingQueue} with a capacity of
     * {@link Integer#MAX_VALUE}.
     */
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }

    /**
     * Hàng đợi hữu hạn
     * Creates a {@code LinkedBlockingQueue} with the given (fixed) capacity.
     *
     * @param capacity the capacity of this queue
     * @throws IllegalArgumentException if {@code capacity} is not greater
     *         than zero
     */
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }
```

### PriorityBlockingQueue

`PriorityBlockingQueue` là một hàng đợi chặn vô hạn hỗ trợ độ ưu tiên. Mặc định các phần tử áp dụng thứ tự tự nhiên để sắp xếp, cũng có thể thông qua lớp tự định nghĩa triển khai phương thức `compareTo()` để chỉ định quy tắc sắp xếp phần tử, hoặc khi khởi tạo thông qua tham số constructor `Comparator` để chỉ định quy tắc sắp xếp.

Việc kiểm soát concurrency của `PriorityBlockingQueue` áp dụng khóa reentrant `ReentrantLock`, hàng đợi là hàng đợi vô hạn (`ArrayBlockingQueue` là hàng đợi hữu hạn, `LinkedBlockingQueue` cũng có thể thông qua truyền `capacity` trong constructor để chỉ định dung lượng tối đa của hàng đợi, nhưng `PriorityBlockingQueue` chỉ có thể chỉ định kích thước ban đầu của hàng đợi, về sau khi chèn phần tử, **nếu không đủ không gian sẽ tự động mở rộng**).

Nói đơn giản, nó chính là phiên bản an toàn luồng của `PriorityQueue`. Không thể chèn giá trị null, đồng thời các đối tượng chèn vào hàng đợi bắt buộc phải so sánh được kích thước (comparable), nếu không sẽ báo ngoại lệ `ClassCastException`. Phương thức put chèn của nó sẽ không block vì nó là hàng đợi vô hạn (phương thức take sẽ bị chặn khi hàng đợi rỗng).

**Bài viết khuyến nghị:** [《Đọc hiểu hàng đợi concurrency Java BlockingQueue》](https://javadoop.com/post/java-concurrent-queue)

## ConcurrentSkipListMap

> Nội dung phần dưới đây tham khảo chuyên mục Geek Time [《Vẻ đẹp của Cấu trúc dữ liệu và Thuật toán》](https://time.geekbang.org/column/intro/126?code=zl3GYeAsRI4rEJIBNu5B/km7LSZsPDlGWQEpAYw5Vu0=&utm_term=SPoster “《数据结构与算法之美》”) và 《Lập trình Java Concurrency thực chiến》.

Để dẫn ra `ConcurrentSkipListMap`, trước tiên đưa mọi người hiểu đơn giản về SkipList (danh sách nhảy).

Đối với một danh sách liên kết đơn, cho dù danh sách liên kết có thứ tự, nếu chúng ta muốn tìm kiếm một dữ liệu nào đó trong đó, cũng chỉ có thể duyệt từ đầu đến cuối danh sách liên kết, như vậy hiệu suất tự nhiên sẽ rất thấp, SkipList thì khác. SkipList là một cấu trúc dữ liệu có thể dùng để tìm kiếm nhanh, hơi giống với cây cân bằng. Chúng đều có thể tiến hành tìm kiếm nhanh đối với phần tử. Nhưng một điểm khác biệt quan trọng là: Thao tác chèn và xóa đối với cây cân bằng thường rất có thể dẫn đến cây cân bằng tiến hành điều chỉnh toàn cục. Còn thao tác chèn và xóa đối với SkipList chỉ cần tiến hành thao tác trên cục bộ của toàn bộ cấu trúc dữ liệu. Lợi ích mang lại là: Trong trường hợp concurrency cao, bạn sẽ cần một khóa toàn cục để đảm bảo an toàn luồng cho toàn bộ cây cân bằng. Còn đối với SkipList, bạn chỉ cần khóa cục bộ là được. Như vậy, trong môi trường concurrency cao, bạn có thể sở hữu hiệu năng tốt hơn. Còn về hiệu năng truy vấn, độ phức tạp thời gian của SkipList cũng là **O(logn)** nên trong cấu trúc dữ liệu concurrency, JDK sử dụng SkipList để triển khai một Map.

Bản chất của SkipList là đồng thời bảo trì nhiều danh sách liên kết, và danh sách liên kết được phân tầng,

![2级索引跳表](https://oss.javaguide.cn/github/javaguide/java/93666217.jpg)

Danh sách liên kết tầng thấp nhất bảo trì tất cả các phần tử trong SkipList, mỗi một tầng danh sách liên kết phía trên đều là tập con của tầng phía dưới.

Tất cả các phần tử của các danh sách liên kết trong SkipList đều được sắp xếp. Khi tìm kiếm, có thể bắt đầu tìm từ danh sách liên kết tầng top. Một khi phát hiện phần tử được tìm kiếm nhỏ hơn node kế nhiệm của node hiện tại truy cập (hoặc node kế nhiệm rỗng), thì chuyển sang danh sách liên kết tầng tiếp theo tiếp tục tìm. Điều này có nghĩa là trong quá trình tìm kiếm, việc tìm kiếm là dạng nhảy vọt. Như hình trên thể hiện, tìm kiếm phần tử 18 trong SkipList.

![在跳表中查找元素18](https://oss.javaguide.cn/github/javaguide/java/32005738.jpg)

Khi tìm 18 ban đầu cần duyệt 18 lần, hiện tại chỉ cần 7 lần là được. Đối với trường hợp chiều dài danh sách liên kết tương đối lớn, việc dựng index nâng cao hiệu suất tìm kiếm sẽ vô cùng rõ rệt.

Từ trên rất dễ nhận thấy, **SkipList là một thuật toán sử dụng không gian đổi lấy thời gian.**

Sử dụng SkipList triển khai `Map` và sử dụng thuật toán hash triển khai `Map` có một điểm khác biệt nữa là: Hash sẽ không bảo tồn thứ tự của phần tử, còn tất cả các phần tử trong SkipList đều được sắp xếp. Do đó khi tiến hành duyệt trên SkipList, bạn sẽ nhận được một kết quả có thứ tự. Vì vậy, nếu ứng dụng của bạn cần tính có thứ tự, vậy thì SkipList chính là lựa chọn duy nhất của bạn. Lớp triển khai cấu trúc dữ liệu này trong JDK là `ConcurrentSkipListMap`.

## Tham khảo

- 《Lập trình Java Concurrency thực chiến》
- <https://javadoop.com/post/java-concurrent-queue>
- <https://juejin.im/post/5aeebd02518825672f19c546>

<!-- @include: @article-footer.snippet.md -->
