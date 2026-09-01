---
title: ArrayBlockingQueue 源码分析
description: ArrayBlockingQueue源码深度解析：详解有界阻塞队列实现、生产者消费者模式应用、ReentrantLock+Condition并发控制、线程池工作队列机制。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: ArrayBlockingQueue源码,阻塞队列,有界队列,生产者消费者模式,ReentrantLock,Condition,线程池工作队列
---

## Giới thiệu về Hàng đợi chặn (BlockingQueue)

### Lịch sử của Blocking Queue

Lịch sử của Java Blocking Queue có thể truy nguyên từ phiên bản JDK1.5, thời điểm đó nền tảng Java đã bổ sung `java.util.concurrent`, tức package JUC mà chúng ta thường gọi, trong đó chứa các công cụ điều khiển luồng concurrent, concurrent container, class nguyên tử (atomic class),... Trong đó tự nhiên cũng bao gồm blocking queue mà chúng ta thảo luận trong bài viết này.

Để giải quyết vấn đề chia sẻ dữ liệu giữa nhiều thread trong kịch bản concurrency cao, phiên bản JDK1.5 đã xuất hiện `ArrayBlockingQueue` và `LinkedBlockingQueue`, chúng là các concurrent container được triển khai theo mô hình Producer-Consumer. Trong đó, `ArrayBlockingQueue` là hàng đợi có giới hạn (bounded queue), tức là sau khi phần tử thêm vào đạt đến giới hạn trên, việc thêm tiếp sẽ bị chặn (block) hoặc ném ngoại lệ. Còn `LinkedBlockingQueue` là hàng đợi cấu thành từ danh sách liên kết, chính vì đặc tính của danh sách liên kết nên `LinkedBlockingQueue` khi thêm phần tử sẽ không có nhiều ràng buộc như `ArrayBlockingQueue`, do đó `LinkedBlockingQueue` việc thiết lập hàng đợi có giới hạn hay không là tùy chọn (lưu ý không có giới hạn ở đây không phải là có thể thêm số lượng phần tử tùy ý, mà là kích thước hàng đợi mặc định là `Integer.MAX_VALUE`, gần như vô hạn).

`SynchronousQueue` và `DelayQueue` cũng được đưa vào trong JDK 1.5, JDK 1.7 bổ sung thêm interface `TransferQueue` hỗ trợ thao tác truyền dẫn (transfer).

### Tư tưởng của Blocking Queue

Blocking Queue chính là mô hình Producer-Consumer điển hình, nó có thể làm được các điểm sau:

1. Khi dữ liệu của blocking queue rỗng, tất cả các consumer thread đều sẽ bị block, chờ hàng đợi không rỗng (not-empty).
2. Khi producer điền dữ liệu vào hàng đợi, hàng đợi sẽ thông báo cho consumer biết hàng đợi không rỗng, consumer lúc này có thể đi vào tiêu thụ (consume).
3. Khi blocking queue bị đầy do consumer tiêu thụ quá chậm hoặc producer đưa phần tử vào quá nhanh dẫn đến hàng đợi không thể chứa thêm phần tử mới, producer sẽ bị block, chờ hàng đợi không đầy (not-full) mới tiếp tục đưa phần tử vào.
4. Khi consumer tiêu thụ một phần tử từ hàng đợi, hàng đợi sẽ thông báo cho producer biết hàng đợi không đầy, producer có thể tiếp tục điền dữ liệu.

Tổng kết lại: Blocking Queue chính là dựa trên 2 điều kiện non-empty (không rỗng) và non-full (không đầy) để thực hiện tương tác giữa producer và consumer. Dù các quy trình tương tác và cơ chế chờ - thông báo này triển khai rất phức tạp, nhưng may mắn dưới bàn tay nhào nặn của Doug Lea, các chi tiết của blocking queue đã được đóng gói che giấu, chúng ta chỉ cần gọi các API như `put`, `take`, `offer`, `poll`,... là có thể thực hiện sản xuất và tiêu thụ giữa nhiều thread.

Điều này cũng khiến blocking queue được ứng dụng rộng rãi trong lập trình multithread, ví dụ phổ biến nhất chính là thread pool của chúng ta, từ source code chúng ta có thể thấy khi core thread không kịp xử lý task, các task này đều sẽ được vứt vào `workQueue`.

```java
public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) {// ...}
```

## Các phương thức thường dùng và test của ArrayBlockingQueue

Sau khi tìm hiểu đơn giản về lịch sử của blocking queue, chúng ta bắt đầu tập trung thảo luận concurrent container sẽ giới thiệu trong bài viết này — `ArrayBlockingQueue`. Để hiểu sâu hơn về `ArrayBlockingQueue` sau này, chúng ta hãy tìm hiểu cách sử dụng `ArrayBlockingQueue` dựa trên vài ví dụ dưới đây.

Xem ví dụ thứ nhất trước, ở đây chúng ta dùng 2 thread lần lượt mô phỏng producer và consumer, producer sau khi sản xuất sẽ dùng phương thức `put` sản xuất 10 phần tử cho consumer tiêu thụ, khi phần tử hàng đợi đạt đến giới hạn trên là 5 do chúng ta thiết lập, phương thức `put` sẽ bị block.
Tương tự consumer cũng sẽ thông qua phương thức `take` để tiêu thụ phần tử, khi hàng đợi rỗng, phương thức `take` sẽ block consumer thread. Ở đây tác giả sử dụng `CountDownLatch` để kiểm soát consumer kết thúc nhằm đảm bảo consumer có thể thoát kịp thời sau khi tiêu thụ xong 10 phần tử, producer ở đây chỉ sản xuất 10 phần tử. Khi consumer tiêu thụ xong 10 phần tử, bấm `CountDownLatch`, tất cả các thread sẽ dừng.

```java
public class ProducerConsumerExample {

    public static void main(String[] args) throws InterruptedException {

        // Tạo một ArrayBlockingQueue kích thước là 5
        ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        // Tạo producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    // Thêm phần tử vào hàng đợi, nếu hàng đợi đã đầy thì block chờ
                    queue.put(i);
                    System.out.println("Producer thêm phần tử: " + i);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });

        CountDownLatch countDownLatch = new CountDownLatch(1);

        // Tạo consumer thread
        Thread consumer = new Thread(() -> {
            try {
                int count = 0;
                while (true) {

                    // Lấy phần tử từ hàng đợi, nếu hàng đợi rỗng thì block chờ
                    int element = queue.take();
                    System.out.println("Consumer lấy phần tử: " + element);
                    ++count;
                    if (count == 10) {
                        break;
                    }
                }

                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });

        // Khởi chạy thread
        producer.start();
        consumer.start();

        // Chờ thread kết thúc
        producer.join();
        consumer.join();

        countDownLatch.await();

        producer.interrupt();
        consumer.interrupt();
    }

}
```

Kết quả output của code như sau, có thể thấy chỉ sau khi producer đưa phần tử vào hàng đợi thì consumer mới có thể tiêu thụ, điều này cũng đồng nghĩa với việc khi trong hàng đợi không có dữ liệu thì consumer sẽ bị block, chờ hàng đợi không rỗng mới tiếp tục tiêu thụ.

```cpp
Producer thêm phần tử: 1
Producer thêm phần tử: 2
Consumer lấy phần tử: 1
Consumer lấy phần tử: 2
Producer thêm phần tử: 3
Consumer lấy phần tử: 3
Producer thêm phần tử: 4
Producer thêm phần tử: 5
Consumer lấy phần tử: 4
Producer thêm phần tử: 6
Consumer lấy phần tử: 5
Producer thêm phần tử: 7
Producer thêm phần tử: 8
Producer thêm phần tử: 9
Producer thêm phần tử: 10
Consumer lấy phần tử: 6
Consumer lấy phần tử: 7
Consumer lấy phần tử: 8
Consumer lấy phần tử: 9
Consumer lấy phần tử: 10
```

Sau khi tìm hiểu 2 phương thức lưu và lấy gây block là `put` và `take`, chúng ta cùng xem các phương thức vào/ra hàng đợi không gây block trong blocking queue là `offer` và `poll`.

Như hiển thị bên dưới, chúng ta thiết lập một blocking queue kích thước là 3, chúng ta sẽ thử dùng phương thức `offer` đặt 4 phần tử vào hàng đợi, sau đó thử dùng `poll` lấy 4 lần từ hàng đợi.

```cpp
public class OfferPollExample {

    public static void main(String[] args) {
        // Tạo một ArrayBlockingQueue kích thước là 3
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(3);

        // Thêm phần tử vào hàng đợi
        System.out.println(queue.offer("A"));
        System.out.println(queue.offer("B"));
        System.out.println(queue.offer("C"));

        // Thử thêm phần tử vào hàng đợi, nhưng hàng đợi đã đầy, trả về false
        System.out.println(queue.offer("D"));

        // Lấy phần tử từ hàng đợi
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());

        // Thử lấy phần tử từ hàng đợi, nhưng hàng đợi đã rỗng, trả về null
        System.out.println(queue.poll());
    }

}
```

Kết quả output cuối cùng của code như sau, có thể thấy do kích thước hàng đợi là 3, kết quả 3 lần đặt vào hàng đợi đầu tiên của chúng ta là true, lần đặt thứ 4 do hàng đợi đã đầy nên kết quả trả về false. Đây cũng là lý do tại sao phương thức `poll` tiếp theo của chúng ta chỉ lấy được giá trị của 3 phần tử.

```cpp
true
true
true
false
A
B
C
null
```

Sau khi tìm hiểu lưu/lấy gây block và không gây block, chúng ta cùng xem một thao tác khá đặc biệt của blocking queue. Trong một số kịch bản, chúng ta muốn có thể chuyển kết quả của blocking queue sang list một lần duy nhất để tiến hành thao tác hàng loạt, chúng ta có thể sử dụng phương thức `drainTo` của blocking queue. Phương thức này sẽ chuyển tất cả phần tử trong hàng đợi sang list một lần duy nhất. Nếu trong hàng đợi có phần tử và chuyển thành công vào list, `drainTo` sẽ trả về số lượng phần tử được chuyển sang list lần này, ngược lại nếu hàng đợi rỗng, `drainTo` trực tiếp trả về 0.

```java
public class DrainToExample {

    public static void main(String[] args) {
        // Tạo một ArrayBlockingQueue kích thước là 5
        ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        // Thêm phần tử vào hàng đợi
        queue.add(1);
        queue.add(2);
        queue.add(3);
        queue.add(4);
        queue.add(5);

        // Tạo một List để lưu trữ các phần tử lấy ra từ hàng đợi
        List<Integer> list = new ArrayList<>();

        // Lấy tất cả phần tử từ hàng đợi và thêm vào List
        queue.drainTo(list);

        // In các phần tử trong List
        System.out.println(list);
    }

}
```

Kết quả output của code như sau

```cpp
[1, 2, 3, 4, 5]
```

## Phân tích source code của ArrayBlockingQueue

Từ đây chúng ta đã có ấn tượng cơ bản về việc sử dụng blocking queue, tiếp theo chúng ta có thể tìm hiểu sâu hơn về cơ chế hoạt động của `ArrayBlockingQueue`.

### Thiết kế tổng thể

Trước khi tìm hiểu chi tiết cụ thể của `ArrayBlockingQueue`, chúng ta hãy xem sơ đồ class của `ArrayBlockingQueue`.

![Sơ đồ class ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/arrayblockingqueue-class-diagram.png)

Từ hình vẽ chúng ta có thể thấy, `ArrayBlockingQueue` triển khai interface `BlockingQueue`, không khó để đoán ra sau khi triển khai interface `BlockingQueue`, `ArrayBlockingQueue` sẽ sở hữu những hành vi thao tác phổ biến của blocking queue.

Đồng thời, `ArrayBlockingQueue` còn kế thừa abstract class `AbstractQueue` (abstract class này kế thừa `AbstractCollection` và `Queue`), từ đặc tính và ngữ nghĩa của abstract class chúng ta cũng có thể đoán ra mối quan hệ kế thừa này giúp `ArrayBlockingQueue` sở hữu các thao tác thường gặp của queue.

Do đó liệu chúng ta có thể đưa ra kết luận: Thông qua kế thừa `AbstractQueue` thu được tất cả các template thao tác của queue, triển khai khung tổng thể của thao tác vào và ra hàng đợi. Sau đó `ArrayBlockingQueue` thông qua triển khai `BlockingQueue` để lấy được các thao tác phổ biến của blocking queue và triển khai các thao tác đó, lấp đầy các chi tiết trong template method của `AbstractQueue`, từ đó `ArrayBlockingQueue` trở thành một blocking queue hoàn chỉnh.

Để kiểm chứng điều này, chúng ta đi vào source code để khám phá. Đầu tiên chúng ta xem `AbstractQueue`, từ mối quan hệ kế thừa class chúng ta có thể rút ra một cách đại thể là nó thông qua `AbstractCollection` thu được các phương thức thao tác phổ biến của collection, sau đó thông qua interface `Queue` thu được các đặc tính của queue.

```java
public abstract class AbstractQueue<E>
    extends AbstractCollection<E>
    implements Queue<E> {
       //...
}
```

Đối với thao tác trên collection chẳng qua là thêm xóa sửa tìm, nên chúng ta hãy bắt đầu từ phương thức thêm, từ source code chúng ta có thể thấy nó triển khai phương thức `add` của `AbstractCollection`, logic bên trong như sau:

1. Gọi phương thức `offer` có được từ việc kế thừa interface `Queue`, nếu `offer` thành công thì trả về `true`.
2. Nếu `offer` thất bại, tức biểu thị phần tử hiện tại đưa vào hàng đợi thất bại thì ném ngoại lệ trực tiếp.

```java
public boolean add(E e) {
  if (offer(e))
      return true;
  else
      throw new IllegalStateException("Queue full");
}
```

Mà trong `AbstractQueue` không hề có sự triển khai cho `offer` của `Queue`, rõ ràng mục đích của việc làm này là định nghĩa sẵn logic cốt lõi của `add`, giao chi tiết của `offer` cho subclass của nó (chính là `ArrayBlockingQueue` của chúng ta) triển khai.

Đến đây, việc phân tích abstract class `AbstractQueue` của chúng ta đã xong, chúng ta tiếp tục xem một interface quan trọng khác được triển khai trong `ArrayBlockingQueue` là `BlockingQueue`.

Sau khi click mở `BlockingQueue`, chúng ta có thể thấy interface này cũng kế thừa interface `Queue`, điều này có nghĩa là nó cũng sở hữu tất cả các hành vi mà queue có. Đồng thời, nó còn định nghĩa các phương thức mà bản thân cần triển khai.

```java
public interface BlockingQueue<E> extends Queue<E> {

     // Phần tử vào hàng đợi thành công trả về true, ngược lại ném ngoại lệ IllegalStateException
    boolean add(E e);

     // Phần tử vào hàng đợi thành công trả về true, ngược lại trả về false
    boolean offer(E e);

     // Phần tử vào hàng đợi thành công thì trả về trực tiếp, nếu hàng đợi đã đầy không thể vào thì block thread, vì trong thời gian block có thể bị ngắt (interrupt) nên ở đây signature ném InterruptedException
    void put(E e) throws InterruptedException;

   // Giống phương thức trên, chỉ có điều khi hàng đợi đầy chỉ block thời lượng timeout đơn vị unit, nếu trong thời gian chờ không vào hàng đợi thành công thì trả về false trực tiếp.
    boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;

    // Lấy một phần tử ở đầu hàng đợi, nếu hàng đợi rỗng thì block chờ, vì nguyên nhân block thread nên phương thức này có thể bị ngắt, do đó signature định nghĩa InterruptedException
    E take() throws InterruptedException;

      // Lấy phần tử đầu hàng đợi và trả về, nếu hàng đợi hiện tại rỗng thì block chờ thời lượng timeout đơn vị unit, nếu trong khoảng thời gian này không có phần tử thì trả về null trực tiếp.
    E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

      // Lấy số lượng phần tử còn lại của hàng đợi
    int remainingCapacity();

     // Xóa đối tượng chỉ định, nếu thành công trả về true, ngược lại trả về false.
    boolean remove(Object o);

    // Kiểm tra hàng đợi có chứa phần tử chỉ định hay không
    public boolean contains(Object o);

     // Chuyển toàn bộ phần tử trong hàng đợi vào collection chỉ định
    int drainTo(Collection<? super E> c);

    // Chuyển maxElements phần tử vào collection
    int drainTo(Collection<? super E> c, int maxElements);
}
```

Sau khi hiểu các thao tác thường gặp của `BlockingQueue`, chúng ta biết được `ArrayBlockingQueue` thông qua việc triển khai các phương thức của `BlockingQueue` và override lại, lấp đầy vào các phương thức của `AbstractQueue`, từ đó chúng ta biết được phương thức `offer` trong phương thức `add` của `AbstractQueue` ở trên được triển khai ở đâu rồi.

```java
public boolean add(E e) {
  // offer của AbstractQueue đến từ phương thức offer ở tầng dưới ArrayBlockingQueue triển khai từ BlockingQueue và override
  if (offer(e))
      return true;
  else
      throw new IllegalStateException("Queue full");
}
```

### Khởi tạo

Trước khi tìm hiểu chi tiết về `ArrayBlockingQueue`, chúng ta hãy xem constructor của nó để hiểu quy trình khởi tạo. Từ source code chúng ta có thể thấy `ArrayBlockingQueue` có 3 constructor, mà constructor cốt lõi nhất chính là constructor dưới đây.

```java
// capacity biểu thị dung lượng khởi tạo hàng đợi, fair biểu thị tính fair (công bằng) của lock
public ArrayBlockingQueue(int capacity, boolean fair) {
  // Nếu kích thước hàng đợi thiết lập nhỏ hơn 0 thì ném trực tiếp IllegalArgumentException
  if (capacity <= 0)
      throw new IllegalArgumentException();
  // Khởi tạo một mảng dùng để lưu trữ các phần tử hàng đợi
  this.items = new Object[capacity];
  // Tạo lock để điều khiển luồng blocking queue
  lock = new ReentrantLock(fair);
  // Dùng lock để tạo 2 condition điều khiển sản xuất và tiêu thụ của hàng đợi
  notEmpty = lock.newCondition();
  notFull =  lock.newCondition();
}
```

Trong constructor này có 2 member variable khá cốt lõi là `notEmpty` (không rỗng) và `notFull` (không đầy), cần chúng ta đặc biệt lưu ý, chúng chính là mấu chốt để thực hiện việc sản xuất và tiêu thụ có thứ tự của producer và consumer. Điểm này tác giả sẽ giải thích chi tiết trong phần phân tích source code tiếp theo, ở đây chúng ta chỉ cần hiểu sơ bộ về cấu trúc của blocking queue là được.

2 constructor còn lại đều dựa trên constructor ở trên. Trong trường hợp mặc định, chúng ta sẽ sử dụng constructor dưới đây, constructor này đồng nghĩa với việc `ArrayBlockingQueue` sử dụng unfair lock (khoá không công bằng), tức là các producer hoặc consumer thread sau khi nhận được thông báo, việc tranh giành lock là ngẫu nhiên.

```java
 public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }
```

Còn một constructor ít khi dùng, sau khi khởi tạo dung lượng và tính unfair của lock, nó còn cung cấp một tham số `Collection`, từ source code không khó để nhận ra constructor này đưa các phần tử của collection truyền từ bên ngoài vào trực tiếp trong blocking queue khi khởi tạo.

```java
public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
  // Khởi tạo dung lượng và tính công bằng của lock
  this(capacity, fair);

  final ReentrantLock lock = this.lock;
  // Khóa lock và đưa các phần tử trong c vào mảng bên dưới của ArrayBlockingQueue
  lock.lock();
  try {
      int i = 0;
      try {
                // Duyệt và thêm phần tử vào mảng
          for (E e : c) {
              checkNotNull(e);
              items[i++] = e;
          }
      } catch (ArrayIndexOutOfBoundsException ex) {
          throw new IllegalArgumentException();
      }
      // Ghi lại dung lượng hàng đợi hiện tại
      count = i;
                      // Cập nhật vị trí lần tiếp theo put hoặc offer hoặc dùng phương thức add để thêm vào mảng bên dưới hàng đợi
      putIndex = (i == capacity) ? 0 : i;
  } finally {
      // Sau khi hoàn thành duyệt thì giải phóng lock
      lock.unlock();
  }
}
```

### Thao tác lấy và thêm phần tử theo cơ chế block

Thao tác lấy và thêm phần tử theo cơ chế block trong `ArrayBlockingQueue` tương ứng chính là mô hình Producer-Consumer, tuy rằng nó cũng hỗ trợ lấy và thêm phần tử không gây block (ví dụ phương thức `poll()` và `offer(E e)` sẽ giới thiệu ở phần sau), nhưng thông thường sẽ không sử dụng.

Các phương thức lấy và thêm phần tử theo cơ chế block của `ArrayBlockingQueue` là:

- `put(E e)`: Chèn phần tử vào hàng đợi, nếu hàng đợi đã đầy thì phương thức này sẽ bị block liên tục cho đến khi hàng đợi có không gian khả dụng hoặc thread bị interrupt.
- `take()`: Lấy và gỡ bỏ phần tử ở đầu hàng đợi, nếu hàng đợi rỗng thì phương thức này sẽ bị block liên tục cho đến khi hàng đợi không rỗng hoặc thread bị interrupt.

Mấu chốt triển khai 2 phương thức này nằm ở 2 đối tượng điều kiện `notEmpty` (không rỗng) và `notFull` (không đầy), điều này chúng ta đã đề cập trong constructor ở trên.

Tiếp theo tác giả thông qua 2 bức hình để mọi người hiểu được 2 điều kiện này được vận dụng như thế nào trong blocking queue.

![Điều kiện notEmpty của ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-notEmpty-take.png)

Giả sử trong code của chúng ta consumer khởi chạy trước, khi nó phát hiện trong hàng đợi không có dữ liệu, thì điều kiện not-empty sẽ treo (suspend) thread này lại, tức là treo khi chờ điều kiện không rỗng. Sau đó quyền thực thi của CPU đến producer, producer phát hiện trong hàng đợi có thể đặt dữ liệu, liền đặt dữ liệu vào, thông báo lúc này điều kiện không rỗng, lúc này consumer sẽ được đánh thức đi vào hàng đợi sử dụng các phương thức như `take` để lấy giá trị.

![Điều kiện notFull của ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-notFull-put.png)

Trong quá trình thực thi tiếp theo, tốc độ sản xuất của producer lớn hơn nhiều so với tốc độ tiêu thụ của consumer, do đó producer sau khi nhồi đầy hàng đợi lại thử đưa dữ liệu vào hàng đợi, phát hiện hàng đợi đã đầy, thế là blocking queue treo thread hiện tại lại, chờ không đầy. Sau đó consumer cầm quyền thực thi CPU tiến hành tiêu thụ, thế là hàng đợi có thể đặt dữ liệu mới, phát ra một thông báo không đầy, lúc này producer đang bị treo sẽ chờ khi có quyền thực thi CPU đến sẽ thử lại việc đưa dữ liệu vào hàng đợi.

Sau khi hiểu đơn giản quy trình tương tác dựa trên 2 điều kiện của blocking queue, chúng ta hãy xem source code của phương thức put và take.

```java
public void put(E e) throws InterruptedException {
    // Đảm bảo phần tử chèn vào không null
    checkNotNull(e);
    // Khóa lock
    final ReentrantLock lock = this.lock;
    // Ở đây sử dụng phương thức lockInterruptibly() thay vì lock() là để có thể đáp ứng thao tác interrupt, nếu trong quá trình chờ lấy lock bị ngắt thì phương thức này sẽ ném ngoại lệ InterruptedException.
    lock.lockInterruptibly();
    try {
            // Nếu count bằng độ dài mảng chứng tỏ hàng đợi đã đầy, thread hiện tại sẽ bị treo đưa vào AQS queue, chờ khi hàng đợi không đầy mới chèn (điều kiện non-full).
       // Trong thời gian chờ, lock sẽ được giải phóng, các thread khác có thể tiếp tục thao tác trên hàng đợi.
        while (count == items.length)
            notFull.await();
           // Nếu hàng đợi có thể chứa phần tử thì gọi enqueue để đưa phần tử vào hàng đợi
        enqueue(e);
    } finally {
        // Giải phóng lock
        lock.unlock();
    }
}
```

Bên trong phương thức `put` gọi phương thức `enqueue` để thực hiện đưa phần tử vào hàng đợi, chúng ta tiếp tục đi sâu xem chi tiết triển khai phương thức `enqueue`:

```java
private void enqueue(E x) {
   // Lấy mảng bên dưới của hàng đợi
    final Object[] items = this.items;
    // Đặt giá trị vị trí putIndex thành x truyền vào
    items[putIndex] = x;
    // Cập nhật putIndex, nếu putIndex bằng độ dài mảng thì cập nhật thành 0
    if (++putIndex == items.length)
        putIndex = 0;
    // Độ dài hàng đợi +1
    count++;
    // Thông báo hàng đợi không rỗng, những thread bị block do lấy phần tử có thể tiếp tục làm việc
    notEmpty.signal();
}
```

Từ source code có thể thấy logic thao tác vào hàng đợi chính là nối thêm một phần tử mới trong mảng, các bước thực thi tổng thể là:

1. Lấy mảng `items` bên dưới của `ArrayBlockingQueue`.
2. Đặt phần tử vào vị trí `putIndex`.
3. Cập nhật `putIndex` sang vị trí tiếp theo, nếu `putIndex` bằng độ dài hàng đợi, chứng tỏ `putIndex` đã đến cuối mảng, lần chèn tiếp theo cần bắt đầu từ 0 (`ArrayBlockingQueue` sử dụng tư tưởng circular queue / hàng đợi vòng, tức tái sử dụng tuần hoàn một mảng từ đầu đến cuối).
4. Cập nhật giá trị `count`, biểu thị độ dài hàng đợi hiện tại +1.
5. Gọi `notEmpty.signal()` thông báo hàng đợi không rỗng, consumer có thể lấy giá trị từ hàng đợi.

Từ đây chúng ta đã hiểu quy trình của phương thức `put`, để hiểu đầy đủ hơn thiết kế về mô hình Producer-Consumer của `ArrayBlockingQueue`, chúng ta tiếp tục xem phương thức `take` để lấy phần tử hàng đợi theo cơ chế block.

```java
public E take() throws InterruptedException {
       // Lấy lock
     final ReentrantLock lock = this.lock;
     lock.lockInterruptibly();
     try {
             // Nếu số lượng phần tử trong hàng đợi bằng 0, thread hiện tại sẽ ngắt và đưa vào AQS queue, chờ hàng đợi không rỗng mới lấy và gỡ bỏ phần tử (điều kiện non-empty)
         while (count == 0)
             notEmpty.await();
            // Nếu hàng đợi không rỗng thì gọi dequeue để lấy phần tử
         return dequeue();
     } finally {
          // Giải phóng lock
         lock.unlock();
     }
}
```

Đã hiểu phương thức `put` thì xem phương thức `take` rất đơn giản, logic cốt lõi của nó và phương thức `put` vừa vặn trái ngược nhau. Ví dụ phương thức `put` khi hàng đợi đầy thì chờ khi hàng đợi không đầy mới chèn phần tử (điều kiện non-full), còn phương thức `take` chờ khi hàng đợi không rỗng mới lấy và gỡ bỏ phần tử (điều kiện non-empty).

Bên trong phương thức `take` gọi phương thức `dequeue` để thực hiện ra khỏi hàng đợi, logic cốt lõi của nó và phương thức `enqueue` cũng trái ngược nhau.

```java
private E dequeue() {
  // Lấy mảng bên dưới của blocking queue
  final Object[] items = this.items;
  @SuppressWarnings("unchecked")
  // Lấy phần tử vị trí takeIndex từ hàng đợi
  E x = (E) items[takeIndex];
  // Đặt takeIndex thành null
  items[takeIndex] = null;
  // takeIndex dịch về sau, nếu bằng độ dài mảng thì cập nhật thành 0
  if (++takeIndex == items.length)
      takeIndex = 0;
  // Độ dài hàng đợi trừ 1
  count--;
  if (itrs != null)
      itrs.elementDequeued();
  // Thông báo các thread đang bị ngắt rằng trạng thái hàng đợi hiện tại không đầy, có thể tiếp tục đặt phần tử
  notFull.signal();
  return x;
}
```

Do phương thức `dequeue` (ra khỏi hàng đợi) và phương thức `enqueue` (vào hàng đợi) giới thiệu ở trên các bước đại thể tương tự nhau, nên ở đây không lặp lại nữa.

Để giúp dễ hiểu, tác giả vẽ một bức hình biểu diễn 2 đối tượng điều kiện `notEmpty` (không rỗng) và `notFull` (không đầy) kiểm soát việc lưu và lấy của `ArrayBlockingQueue` như thế nào.

![notEmpty và notFull của ArrayBlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-notEmpty-notFull.png)

- **Consumer**: Khi consumer lấy một phần tử khỏi hàng đợi qua các thao tác `take` hoặc `poll`,... sẽ thông báo hàng đợi không đầy, lúc này những producer đang chờ không đầy sẽ được đánh thức chờ lấy CPU time slice để thực hiện thao tác vào hàng đợi.
- **Producer**: Khi producer đưa phần tử vào hàng đợi, sẽ kích hoạt thông báo hàng đợi không rỗng, lúc này consumer sẽ được đánh thức chờ CPU time slice thử lấy phần tử. Cứ như vậy lặp đi lặp lại, 2 đối tượng điều kiện cấu thành một vòng lặp kín (loop), kiểm soát việc lưu và lấy giữa nhiều thread.

### Thao tác lấy và thêm phần tử theo cơ chế non-blocking

Các phương thức lấy và thêm phần tử theo cơ chế non-blocking của `ArrayBlockingQueue` là:

- `offer(E e)`: Chèn phần tử vào cuối hàng đợi. Nếu hàng đợi đã đầy, phương thức này sẽ trực tiếp trả về false, không chờ đợi và không block thread.
- `poll()`: Lấy và gỡ bỏ phần tử ở đầu hàng đợi, nếu hàng đợi rỗng, phương thức này sẽ trực tiếp trả về null, không chờ đợi và không block thread.
- `add(E e)`: Chèn phần tử vào cuối hàng đợi. Nếu hàng đợi đã đầy sẽ ném ngoại lệ `IllegalStateException`, bên dưới dựa trên phương thức `offer(E e)`.
- `remove()`: Gỡ bỏ phần tử ở đầu hàng đợi, nếu hàng đợi rỗng sẽ ném ngoại lệ `NoSuchElementException`, bên dưới dựa trên `poll()`.
- `peek()`: Lấy nhưng không gỡ bỏ phần tử ở đầu hàng đợi, nếu hàng đợi rỗng, phương thức này sẽ trực tiếp trả về null, không chờ đợi và không block thread.

Trước tiên xem phương thức `offer`, logic tương tự `put`, điểm khác biệt duy nhất là khi vào hàng đợi thất bại thì không block thread hiện tại mà trực tiếp trả về `false`.

```java
public boolean offer(E e) {
        // Đảm bảo phần tử chèn vào không null
        checkNotNull(e);
        // Lấy lock
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
             // Hàng đợi đã đầy trả về false trực tiếp
            if (count == items.length)
                return false;
            else {
                // Ngược lại đưa phần tử vào hàng đợi và trả về true trực tiếp
                enqueue(e);
                return true;
            }
        } finally {
            // Giải phóng lock
            lock.unlock();
        }
    }
```

Phương thức `poll` tương tự, lấy phần tử thất bại cũng trực tiếp trả về null, không block thread lấy phần tử.

```java
public E poll() {
        final ReentrantLock lock = this.lock;
        // Khóa lock
        lock.lock();
        try {
            // Nếu hàng đợi rỗng trả về null trực tiếp, ngược lại ra khỏi hàng đợi và trả về giá trị phần tử
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
```

Phương thức `add` thực ra chính là đóng gói một lớp quanh `offer`, như đoạn code dưới đây, có thể thấy `add` sẽ gọi `offer` không quy định thời gian, nếu vào hàng đợi thất bại thì ném ngoại lệ trực tiếp.

```java
public boolean add(E e) {
        return super.add(e);
    }


public boolean add(E e) {
        // Gọi phương thức offer nếu thất bại ném ngoại lệ trực tiếp
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }
```

Phương thức `remove` tương tự, gọi `poll`, nếu trả về `null` chứng tỏ hàng đợi không có phần tử, ném ngoại lệ trực tiếp.

```java
public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }
```

Logic phương thức `peek()` cũng rất đơn giản, bên trong gọi phương thức `itemAt`.

```java
public E peek() {
        // Khóa lock
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // Khi hàng đợi rỗng trả về null
            return itemAt(takeIndex);
        } finally {
            // Giải phóng lock
            lock.unlock();
        }
    }

// Trả về phần tử tại vị trí chỉ định trong hàng đợi
@SuppressWarnings("unchecked")
final E itemAt(int i) {
    return (E) items[i];
}
```

### Thao tác lấy và thêm phần tử theo cơ chế block trong khoảng thời gian timeout chỉ định

Trên cơ sở lấy và thêm phần tử non-blocking của `offer(E e)` và `poll()`, nhà thiết kế đã cung cấp `offer(E e, long timeout, TimeUnit unit)` và `poll(long timeout, TimeUnit unit)` có thời gian chờ, dùng để thêm và lấy phần tử theo cơ chế block trong khoảng thời gian timeout chỉ định.

```java
 public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
        // Hàng đợi đã đầy, vào vòng lặp
            while (count == items.length) {
            // Hết thời gian mà hàng đợi vẫn đầy thì trả về false trực tiếp
                if (nanos <= 0)
                    return false;
                 // Block thời gian nanos, chờ không đầy
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

Có thể thấy, phương thức `offer` có thời gian timeout trong trường hợp hàng đợi đã đầy sẽ chờ khoảng thời gian người dùng truyền vào, nếu trong thời gian quy định vẫn không thể đặt phần tử thì trả về `false` trực tiếp.

```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
          // Hàng đợi rỗng, vòng lặp chờ, nếu hết thời gian vẫn rỗng thì trả về null trực tiếp
            while (count == 0) {
                if (nanos <= 0)
                    return null;
                nanos = notEmpty.awaitNanos(nanos);
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```

Tương tự, `poll` có thời gian timeout cũng vậy, hàng đợi rỗng sẽ chờ trong thời gian quy định, nếu hết thời gian vẫn rỗng thì trực tiếp trả về null.

### Kiểm tra phần tử có tồn tại hay không

`ArrayBlockingQueue` cung cấp `contains(Object o)` để kiểm tra phần tử chỉ định có tồn tại trong hàng đợi hay không.

```java
public boolean contains(Object o) {
    // Nếu phần tử mục tiêu null thì trả về false trực tiếp
    if (o == null) return false;
    // Lấy mảng phần tử của hàng đợi hiện tại
    final Object[] items = this.items;
    // Khóa lock
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // Nếu hàng đợi không rỗng
        if (count > 0) {
            final int putIndex = this.putIndex;
            // Duyệt từ đầu hàng đợi
            int i = takeIndex;
            do {
                if (o.equals(items[i]))
                    return true;
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        // Giải phóng lock
        lock.unlock();
    }
}
```

## So sánh các phương thức lấy và thêm phần tử trong ArrayBlockingQueue

Để giúp dễ hiểu `ArrayBlockingQueue`, chúng ta cùng so sánh lại các phương thức lấy và thêm phần tử đã đề cập ở trên.

Thêm phần tử:

| Phương thức | Xử lý khi hàng đợi đầy | Giá trị trả về |
| --- | --- | --- |
| `put(E e)` | Thread bị block cho đến khi bị interrupt hoặc được đánh thức | void |
| `offer(E e)` | Trực tiếp trả về false | boolean |
| `offer(E e, long timeout, TimeUnit unit)` | Block trong thời gian timeout chỉ định, vượt quá thời gian quy định chưa thêm thành công thì trả về false | boolean |
| `add(E e)` | Ném trực tiếp ngoại lệ `IllegalStateException` | boolean |

Lấy/gỡ bỏ phần tử:

| Phương thức | Xử lý khi hàng đợi rỗng | Giá trị trả về |
| --- | --- | --- |
| `take()` | Thread bị block cho đến khi bị interrupt hoặc được đánh thức | E |
| `poll()` | Trả về null | E |
| `poll(long timeout, TimeUnit unit)` | Block trong thời gian timeout chỉ định, vượt quá thời gian quy định vẫn rỗng thì trả về null | E |
| `peek()` | Trả về null | E |
| `remove()` | Ném trực tiếp ngoại lệ `NoSuchElementException` | boolean |

![](https://oss.javaguide.cn/github/javaguide/java/collection/ArrayBlockingQueue-get-add-element-methods.png)

## Câu hỏi phỏng vấn liên quan đến ArrayBlockingQueue

### ArrayBlockingQueue là gì? Đặc điểm của nó là gì?

`ArrayBlockingQueue` là class implementation hàng đợi có giới hạn (bounded queue) của interface `BlockingQueue`, thường dùng để chia sẻ dữ liệu giữa nhiều thread, bên dưới sử dụng mảng để triển khai, từ tên gọi của nó đã có thể thấy được.

Dung lượng của `ArrayBlockingQueue` có hạn, một khi đã tạo thì dung lượng không thể thay đổi.

Để đảm bảo thread-safe, việc kiểm soát concurrency của `ArrayBlockingQueue` áp dụng reentrant lock `ReentrantLock`, bất kể là thao tác chèn hay thao tác đọc đều cần lấy được lock mới có thể thao tác. Đồng thời, nó còn hỗ trợ 2 cơ chế truy cập lock fair và unfair, mặc định là unfair lock.

`ArrayBlockingQueue` dù tên là blocking queue, nhưng cũng hỗ trợ lấy và thêm phần tử non-blocking (ví dụ phương thức `poll()` và `offer(E e)`). Khi hàng đợi đầy, `offer(E e)` sẽ trả về `false`; khi hàng đợi rỗng, `poll()` sẽ trả về `null`.

### ArrayBlockingQueue và LinkedBlockingQueue có sự khác biệt gì?

`ArrayBlockingQueue` và `LinkedBlockingQueue` là 2 implementation blocking queue thường dùng trong package concurrent của Java, chúng đều là thread-safe. Tuy nhiên giữa chúng tồn tại các điểm khác biệt sau:

- Implementation bên dưới: `ArrayBlockingQueue` dựa trên mảng, còn `LinkedBlockingQueue` dựa trên danh sách liên kết.
- Có giới hạn (bounded) hay không: `ArrayBlockingQueue` là bounded queue, bắt buộc phải chỉ định kích thước dung lượng khi tạo. `LinkedBlockingQueue` khi tạo có thể không chỉ định kích thước dung lượng, mặc định là `Integer.MAX_VALUE` (unbounded queue). Nhưng cũng có thể chỉ định kích thước hàng đợi để trở thành bounded queue.
- Lock có phân tách hay không: Lock trong `ArrayBlockingQueue` không được phân tách, tức là sản xuất và tiêu thụ dùng chung 1 lock; Lock trong `LinkedBlockingQueue` được phân tách, tức sản xuất dùng `putLock`, tiêu thụ dùng `takeLock`, từ đó tránh tranh giành lock giữa producer thread và consumer thread.
- Chiếm dụng bộ nhớ: `ArrayBlockingQueue` cần cấp phát bộ nhớ mảng trước, còn `LinkedBlockingQueue` cấp phát bộ nhớ node danh sách liên kết một cách động. Điều này có nghĩa là `ArrayBlockingQueue` khi tạo sẽ chiếm một khoảng không gian bộ nhớ nhất định, và thường bộ nhớ xin cấp phát lớn hơn bộ nhớ thực tế sử dụng, còn `LinkedBlockingQueue` chiếm dụng bộ nhớ dần theo sự tăng lên của phần tử.

### ArrayBlockingQueue và ConcurrentLinkedQueue có sự khác biệt gì?

`ArrayBlockingQueue` và `ConcurrentLinkedQueue` là 2 implementation queue thường dùng trong package concurrent của Java, chúng đều là thread-safe. Tuy nhiên giữa chúng tồn tại những điểm khác biệt sau:

- Implementation bên dưới: `ArrayBlockingQueue` dựa trên mảng, còn `ConcurrentLinkedQueue` dựa trên danh sách liên kết.
- Có giới hạn hay không: `ArrayBlockingQueue` là bounded queue, bắt buộc phải chỉ định kích thước dung lượng khi tạo, còn `ConcurrentLinkedQueue` là unbounded queue, có thể mở rộng dung lượng động.
- Có blocking hay không: `ArrayBlockingQueue` hỗ trợ 2 cách lấy và thêm phần tử blocking và non-blocking (thông thường chỉ dùng cách trước), `ConcurrentLinkedQueue` là unbounded, chỉ hỗ trợ lấy và thêm phần tử theo cơ chế non-blocking.

### Nguyên lý triển khai của ArrayBlockingQueue là gì?

Nguyên lý triển khai của `ArrayBlockingQueue` chủ yếu chia thành các điểm sau (ở đây lấy việc lấy và thêm phần tử theo cơ chế block làm ví dụ giới thiệu):

- `ArrayBlockingQueue` bên trong duy trì một mảng độ dài cố định dùng để lưu trữ phần tử.
- Thông qua việc sử dụng đối tượng lock `ReentrantLock` để đồng bộ hóa các thao tác đọc ghi, tức là thông qua cơ chế lock để thực hiện thread-safe.
- Thông qua `Condition` thực hiện các thao tác chờ (wait) và đánh thức (notify/signal) giữa các thread.

Ở đây giới thiệu chi tiết thêm sự triển khai cụ thể của việc chờ và đánh thức giữa các thread (không cần nhớ phương thức cụ thể, khi phỏng vấn chỉ cần trả lời các ý chính là được):

- Khi hàng đợi đã đầy, producer thread sẽ gọi phương thức `notFull.await()` cho producer chờ đợi, chờ khi hàng đợi không đầy mới chèn (điều kiện non-full).
- Khi hàng đợi rỗng, consumer thread sẽ gọi phương thức `notEmpty.await()` cho consumer chờ đợi, chờ khi hàng đợi không rỗng mới tiêu thụ (điều kiện non-empty).
- Khi có phần tử mới được thêm vào, producer thread sẽ gọi phương thức `notEmpty.signal()` để đánh thức consumer thread đang chờ tiêu thụ.
- Khi trong hàng đợi có phần tử được lấy ra, consumer thread sẽ gọi phương thức `notFull.signal()` để đánh thức producer thread đang chờ chèn phần tử.

Bổ sung về interface Condition:

> `Condition` có từ JDK1.5 trở đi, nó có tính linh hoạt rất tốt, ví dụ có thể thực hiện tính năng thông báo đa luồng (multi-path notification), tức là trong một đối tượng `Lock` có thể tạo nhiều instance `Condition` (tức object monitor). Đối tượng thread có thể đăng ký vào `Condition` chỉ định, từ đó có thể thực hiện thông báo thread một cách có lựa chọn, linh hoạt hơn nhiều trong việc điều phối thread. Khi sử dụng phương thức `notify()/notifyAll()` để thông báo, thread được thông báo do JVM lựa chọn; còn dùng class `ReentrantLock` kết hợp với instance `Condition` có thể thực hiện "thông báo có lựa chọn", tính năng này rất quan trọng và được cung cấp mặc định bởi interface `Condition`. Trong khi keyword `synchronized` tương đương với việc trong toàn bộ đối tượng `Lock` chỉ có một instance `Condition`, tất cả thread đều đăng ký trên 1 instance đó. Nếu thực thi phương thức `notifyAll()`, nó sẽ thông báo cho tất cả thread đang trong trạng thái chờ, điều này gây ra vấn đề lớn về mặt hiệu năng. Còn phương thức `signalAll()` của instance `Condition` chỉ đánh thức tất cả thread đang chờ được đăng ký trong instance `Condition` đó.

## Tài liệu tham khảo

- Tìm hiểu sâu series Java | Chi tiết cách dùng BlockingQueue: <https://juejin.cn/post/6999798721269465102>
- Hiểu đơn giản về BlockingQueue và implementation điển hình ArrayBlockingQueue: <https://zhuanlan.zhihu.com/p/539619957>
- Quét sạch Concurrent Programming: Nguyên lý bên dưới và thực chiến ArrayBlockingQueue: <https://zhuanlan.zhihu.com/p/339662987>
<!-- @include: @article-footer.snippet.md -->
