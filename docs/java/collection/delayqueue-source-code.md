---
title: DelayQueue 源码分析
description: DelayQueue源码深度解析：详解延迟队列实现原理、Delayed接口使用、延时任务调度、订单超时取消等应用场景、基于PriorityQueue的线程安全设计。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: DelayQueue源码,延迟队列,Delayed接口,延时任务,定时任务,订单超时,PriorityQueue实现
---

## Giới thiệu về DelayQueue

`DelayQueue` là hàng đợi trì hoãn (delay queue) do package JUC (`java.util.concurrent`) cung cấp cho chúng ta, dùng để thực hiện các delay task ví dụ đơn hàng sau 15 phút chưa thanh toán sẽ tự động hủy. Nó là một loại `BlockingQueue`, bên dưới là một unbounded queue dựa trên `PriorityQueue`, thread-safe. Về `PriorityQueue` có thể tham khảo bài viết này của tác giả: [Phân tích source code PriorityQueue](./priorityqueue-source-code.md).

![Các class implementation của BlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/blocking-queue-hierarchy.png)

Các phần tử lưu trữ trong `DelayQueue` bắt buộc phải triển khai interface `Delayed`, và cần override phương thức `getDelay()` (tính toán xem đã hết hạn chưa).

```java
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

Mặc định, `DelayQueue` sẽ sắp xếp task theo thứ tự thời gian hết hạn tăng dần. Chỉ khi phần tử đã hết hạn (giá trị trả về của phương thức `getDelay()` nhỏ hơn hoặc bằng 0) mới có thể lấy ra khỏi hàng đợi.

`DelayQueue` lần đầu tiên được đưa vào trong Java 5, là một unbounded blocking queue thread-safe.

## Ví dụ kịch bản sử dụng thường gặp của DelayQueue

Ở đây chúng ta muốn task trở thành trạng thái có thể lấy ra theo đúng độ trễ (delay) dự kiến, ví dụ submit 3 task, thiết lập delay lần lượt là 1s, 2s, 3s, cho dù thêm xáo trộn thứ tự, task có delay hết hạn sớm nhất cũng sẽ trở thành trạng thái có thể lấy ra sớm nhất.

![Delay Task](https://oss.javaguide.cn/github/javaguide/java/collection/delayed-task.png)

Về điểm này chúng ta có thể sử dụng `DelayQueue` để thực hiện, do đó trước tiên chúng ta cần triển khai `Delayed` cho `DelayedTask`, triển khai phương thức `getDelay` cũng như so sánh độ ưu tiên `compareTo`.

```java
/**
 * Delay task
 */
public class DelayedTask implements Delayed {
    /**
     * Thời gian hết hạn của task
     */
    private long executeTime;
    /**
     * Task
     */
    private Runnable task;

    public DelayedTask(long delay, Runnable task) {
        this.executeTime = System.currentTimeMillis() + delay;
        this.task = task;
    }

    /**
     * Xem task hiện tại còn bao lâu nữa hết hạn
     * @param unit
     * @return
     */
    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(executeTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    /**
     * Delay queue cần xếp vào hàng đợi theo thứ tự thời gian hết hạn tăng dần, nên chúng ta cần triển khai compareTo để so sánh thời gian hết hạn
     * @param o
     * @return
     */
    @Override
    public int compareTo(Delayed o) {
        return Long.compare(this.executeTime, ((DelayedTask) o).executeTime);
    }

    public void execute() {
        task.run();
    }
}
```

Sau khi hoàn thành đóng gói task, việc sử dụng rất đơn giản, thiết lập bao lâu nữa hết hạn rồi submit task vào delay queue là được.

```java
// Tạo delay queue, và thêm task
DelayQueue < DelayedTask > delayQueue = new DelayQueue < > ();

// Lần lượt thêm các task hết hạn sau 1s, 2s, 3s
delayQueue.add(new DelayedTask(2000, () -> System.out.println("Task 2")));
delayQueue.add(new DelayedTask(1000, () -> System.out.println("Task 1")));
delayQueue.add(new DelayedTask(3000, () -> System.out.println("Task 3")));

// Lấy task và thực thi
while (!delayQueue.isEmpty()) {
  // Block lấy task hết hạn sớm nhất
  DelayedTask task = delayQueue.take();
  if (task != null) {
    task.execute();
  }
}
```

Từ kết quả output có thể thấy, cho dù tác giả chèn task 2s hết hạn trước, task Task 1 hết hạn 1s vẫn được ưu tiên thực thi trước.

```java
Task 1
Task 2
Task 3
```

## Phân tích source code của DelayQueue

Ở đây lấy JDK1.8 làm ví dụ để phân tích source code cốt lõi bên dưới của `DelayQueue`.

Khai báo class `DelayQueue` như sau:

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E> implements BlockingQueue<E>
{
  //...
}
```

`DelayQueue` kế thừa class `AbstractQueue`, triển khai interface `BlockingQueue`.

![Sơ đồ class DelayQueue](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-class-diagram.png)

### Field thành viên cốt lõi

4 field thành viên cốt lõi của `DelayQueue` như sau:

```java
// Reentrant lock, mấu chốt thực hiện thread-safe
private final transient ReentrantLock lock = new ReentrantLock();
// Collection lưu trữ dữ liệu bên dưới của delay queue, đảm bảo phần tử sắp xếp theo thứ tự thời gian hết hạn tăng dần
private final PriorityQueue<E> q = new PriorityQueue<E>();

// Trỏ đến thread chuẩn bị thực thi task có độ ưu tiên cao nhất
private Thread leader = null;
// Thực hiện tương tác chờ và đánh thức giữa nhiều thread
private final Condition available = lock.newCondition();
```

- `lock`: Chúng ta đều biết lưu/lấy trong `DelayQueue` là thread-safe, nên để đảm bảo thread-safe khi lưu/lấy phần tử, chúng ta cần lock khi lưu/lấy, mà `DelayQueue` dựa trên exclusive lock `ReentrantLock` để đảm bảo thread-safe của thao tác lưu/lấy.
- `q`: Delay queue yêu cầu phần tử sắp xếp theo thứ tự thời gian hết hạn tăng dần, nên khi thêm phần tử nhất định cần sắp xếp độ ưu tiên, do đó việc lưu/lấy phần tử bên dưới của `DelayQueue` đều được quản lý thông qua field member priority queue `PriorityQueue` `q` này.
- `leader`: Task trong delay queue chỉ khi hết hạn mới thực thi, đối với các task chưa hết hạn thì chỉ có chờ đợi. Để đảm bảo task có độ ưu tiên cao nhất sau khi hết hạn có thể được thực thi ngay lập tức, nhà thiết kế dùng `leader` để quản lý delay task, chỉ thread mà `leader` trỏ đến mới sở hữu quyền hạn chờ có thời hạn cho task hết hạn để thực thi, còn những thread có độ ưu tiên thấp hơn chỉ có thể chờ vô hạn, cho đến khi `leader` thread thực thi xong delay task trong tay rồi đánh thức nó.
- `available`: Tương tác chờ đánh thức được đề cập khi nói về `leader` thread ở trên được thực hiện thông qua `available`. Giả sử thread 1 thử lấy task từ `DelayQueue` rỗng, `available` sẽ đưa nó vào queue chờ. Cho đến khi có một thread thêm một delay task rồi thông qua phương thức `signal` của `available` đánh thức nó.

### Constructor

So với các concurrent container khác, constructor của delay queue tương đối đơn giản, nó chỉ có 2 constructor, vì tất cả field member khi load class đều đã khởi tạo xong rồi, nên constructor mặc định không làm gì cả. Còn một constructor truyền vào đối tượng `Collection`, nó sẽ gọi phương thức `addAll()` để lưu các phần tử collection vào priority queue `q`.

```java
public DelayQueue() {}

public DelayQueue(Collection<? extends E> c) {
    this.addAll(c);
}
```

### Thêm phần tử

Các phương thức thêm phần tử của `DelayQueue` dù là `add`, `put` hay `offer`, bản chất đều là gọi `offer`, do đó để hiểu logic thêm của delay queue chúng ta chỉ cần đọc phương thức `offer` là được.

Logic tổng thể của phương thức `offer`:

1. Thử lấy `lock`.
2. Nếu khóa lock thành công, gọi phương thức `offer` của `q` để đặt phần tử vào priority queue.
3. Gọi phương thức `peek` xem phần tử ở đầu queue hiện tại có phải phần tử vừa vào queue lần này hay không, nếu phải chứng tỏ phần tử hiện tại là task sắp hết hạn (tức phần tử có độ ưu tiên cao nhất), thế là gán `leader` thành null, thông báo cho các thread bị block do gọi các phương thức như `take` khi queue rỗng đến tranh giành phần tử.
4. Các bước trên thực hiện xong, giải phóng `lock`.
5. Trả về true.

Source code như sau:

```java
public boolean offer(E e) {
    // Thử lấy lock
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // Nếu khóa lock thành công, gọi phương thức offer của q để đặt phần tử vào priority queue
        q.offer(e);
        // Gọi phương thức peek xem phần tử đầu queue hiện tại có phải phần tử vừa vào queue lần này hay không, nếu phải chứng tỏ phần tử này là task sắp hết hạn (tức phần tử có độ ưu tiên cao nhất)
        if (q.peek() == e) {
            // Gán leader thành null, thông báo cho các thread bị block do gọi phương thức lấy phần tử đến tranh giành task này
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        // Các bước trên thực hiện xong, giải phóng lock
        lock.unlock();
    }
}
```

### Lấy phần tử

Cách lấy phần tử trong `DelayQueue` chia thành block (tắc nghẽn) và non-block (không tắc nghẽn). Trước tiên xem phương thức lấy phần tử dạng block có logic tương đối phức tạp là `take`. Để giúp độc giả có thể hiểu trực quan hơn toàn bộ quy trình lấy phần tử dạng block, tác giả lấy ví dụ 3 thread concurrent lấy phần tử để trình bày quy trình làm việc của `take`.

> 想要理解下面的内容，需要用到 AQS 相关的知识，推荐阅读下面这两篇文章：
>
> - [图文讲解 AQS ，一起看看 AQS 的源码……(图文较长)](https://xie.infoq.cn/article/5a3cc0b709012d40cb9f41986)
> - [AQS 都看完了，Condition 原理可不能少！](https://xie.infoq.cn/article/0223d5e5f19726b36b084b10d)

1. Đầu tiên, 3 thread sẽ thử lấy reentrant lock `lock`. Giả sử chúng ta có 3 thread lần lượt là t1, t2, t3, sau đó t1 lấy được lock, còn t2, t3 không giành được lock, do đó đưa 2 thread này vào wait queue.

![](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-take-0.png)

2. Tiếp sau đó t1 bắt đầu tiến hành logic lấy phần tử.

3. Thread t1 đầu tiên sẽ xem phần tử ở đầu queue của `DelayQueue` có null hay không.

4. Nếu phần tử null, chứng tỏ queue hiện tại không có bất kỳ phần tử nào, do đó t1 sẽ bị block lưu vào queue `conditionWaiter`.

![](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-take-1.png)

Lưu ý, sau khi gọi `await` t1 sẽ giải phóng lock `lock`. Giả sử `DelayQueue` liên tục rỗng, vậy t2, t3 cũng sẽ thực thi logic tương tự như t1 và đi vào queue `conditionWaiter`.

![](https://oss.javaguide.cn/github/javaguide/java/collection/delayqueue-take-2.png)

Nếu phần tử không null, kiểm tra xem task hiện tại đã hết hạn chưa. Nếu phần tử đã hết hạn thì trả về trực tiếp. Nếu phần tử chưa hết hạn, kiểm tra xem `leader` thread hiện tại (tham chiếu thread duy nhất trong `DelayQueue` có thể chờ và lấy phần tử) có null hay không. Nếu không null, chứng tỏ `leader` hiện tại đang chờ thực thi một phần tử có độ ưu tiên còn cao hơn phần tử hiện tại đến hạn, do đó thread t1 hiện tại chỉ có thể gọi `await` đi vào chờ vô hạn, chờ đến khi `leader` lấy được phần tử thì đánh thức. Ngược lại, nếu `leader` thread null, gán thread hiện tại làm leader và đi vào chờ có thời hạn, sau khi hết hạn lấy phần tử ra và trả về.

Đến đây logic lấy phần tử theo cơ chế block đã hoàn thành, source code như sau:

```java
public E take() throws InterruptedException {
    // Thử lấy reentrant lock, gán state của AQS bên dưới thành 1, và thiết lập thành exclusive lock
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            // Xem phần tử đầu tiên của hàng đợi
            E first = q.peek();
            // Nếu null, đưa thread hiện tại vào wait queue của ConditionObject, và gán state của AQS bên dưới thành 0, biểu thị giải phóng lock và vào chờ vô hạn
            if (first == null)
                available.await();
            else {
                // Nếu phần tử không null, xem phần tử hiện tại còn bao lâu hết hạn
                long delay = first.getDelay(NANOSECONDS);
                // Nếu nhỏ hơn hoặc bằng 0 chứng tỏ đã hết hạn trả về trực tiếp
                if (delay <= 0)
                    return q.poll();
                // Nếu lớn hơn 0 chứng tỏ task chưa hết hạn, đầu tiên cần giải phóng tham chiếu đến phần tử này
                first = null; // don't retain ref while waiting
                // Kiểm tra leader có null không, nếu không null chứng tỏ đang có thread làm leader và chờ 1 task hết hạn, thread hiện tại vào chờ vô hạn
                if (leader != null)
                    available.await();
                else {
                    // Ngược lại cho thread của chúng ta làm leader
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        // Và vào chờ có thời hạn
                        available.awaitNanos(delay);
                    } finally {
                        // Khi chờ task hết hạn, giải phóng tham chiếu leader, vào lần lặp tiếp theo return task ra ngoài
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        // Logic kết thúc: Khi leader là null và trong queue có task, đánh thức thread đang chờ lấy phần tử.
        if (leader == null && q.peek() != null)
            available.signal();
        // Giải phóng lock
        lock.unlock();
    }
}
```

Chúng ta cùng xem phương thức lấy phần tử non-blocking `poll`, logic tương đối đơn giản, các bước tổng thể như sau:

1. Thử lấy reentrant lock.
2. Xem phần tử đầu tiên của queue, kiểm tra phần tử có null hay không.
3. Nếu phần tử null, hoặc phần tử chưa hết hạn thì trả về null trực tiếp.
4. Nếu phần tử không null và đã hết hạn, gọi trực tiếp `poll` trả về ra ngoài.
5. Giải phóng reentrant lock `lock`.

Source code như sau:

```java
public E poll() {
    // Thử lấy reentrant lock
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // Xem phần tử đầu tiên của hàng đợi, kiểm tra phần tử có null không
        E first = q.peek();

        // Nếu phần tử null hoặc phần tử chưa hết hạn thì trả về null trực tiếp
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
            // Nếu phần tử không null và đã hết hạn, gọi trực tiếp poll trả về ra ngoài
            return q.poll();
    } finally {
        // Giải phóng reentrant lock lock
        lock.unlock();
    }
}
```

### Xem phần tử

Khi lấy phần tử ở trên đều sẽ gọi đến phương thức `peek`. `peek` đúng như tên gọi chỉ nghía xem phần tử trong queue, các bước của nó gồm 4 bước:

1. Khóa lock.
2. Gọi phương thức `peek` của priority queue `q` để xem phần tử vị trí index 0.
3. Giải phóng lock.
4. Trả phần tử ra ngoài.

```java
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return q.peek();
    } finally {
        lock.unlock();
    }
}
```

## Câu hỏi phỏng vấn thường gặp về DelayQueue

### Nguyên lý triển khai của DelayQueue là gì?

`DelayQueue` bên dưới sử dụng priority queue `PriorityQueue` để lưu trữ phần tử, mà `PriorityQueue` áp dụng tư tưởng min-heap nhị phân đảm bảo phần tử giá trị nhỏ xếp ở vị trí đầu tiên, điều này giúp `DelayQueue` quản lý độ ưu tiên của delay task trở nên vô cùng thuận tiện. Đồng thời `DelayQueue` để đảm bảo thread-safe còn dùng đến reentrant lock `ReentrantLock`, đảm bảo trong một đơn vị thời gian chỉ có 1 thread có thể thao tác trên delay queue. Cuối cùng, để thực hiện hiệu quả tương tác chờ và đánh thức giữa nhiều thread, `DelayQueue` còn dùng đến `Condition`, thông qua phương thức `await` và `signal` của `Condition` để hoàn thành việc chờ và đánh thức giữa nhiều thread.

### Việc triển khai DelayQueue có thread-safe không?

Sự triển khai của `DelayQueue` là thread-safe, nó thông qua `ReentrantLock` thực hiện truy cập tương hỗ (mutual exclusion) và `Condition` thực hiện thao tác chờ và đánh thức giữa các thread, có thể đảm bảo tính an toàn và độ tin cậy trong môi trường multithread.

### Kịch bản sử dụng của DelayQueue gồm những gì?

`DelayQueue` thường được dùng để thực hiện lập lịch định thời task (timer task scheduling) và xóa cache hết hạn,... Trong lập lịch định thời task, cần đóng gói task cần thực thi thành đối tượng delay task, và thêm nó vào `DelayQueue`, phần tử đầu queue khi độ trễ hết hạn có thể được lấy ra; việc task khi nào thực tế được thực thi còn phụ thuộc vào việc điều phối của consumer thread. Đối với kịch bản cache hết hạn, sau khi dữ liệu được cache vào bộ nhớ, chúng ta có thể đóng gói key của cache thành một delay delete task, và thêm nó vào `DelayQueue`, khi dữ liệu hết hạn, lấy được key của task này và gỡ bỏ key đó khỏi bộ nhớ.

### Tác dụng của interface Delayed trong DelayQueue là gì?

Interface `Delayed` định nghĩa thời gian delay còn lại (`getDelay`) của phần tử và quy tắc so sánh giữa các phần tử (interface này kế thừa interface `Comparable`). Nếu muốn phần tử có thể lưu vào `DelayQueue`, bắt buộc phải triển khai phương thức `getDelay()` và `compareTo()` của interface `Delayed`, nếu không `DelayQueue` không thể biết được thời gian còn lại của task hiện于 và việc so sánh độ ưu tiên của task.

### Sự khác biệt giữa DelayQueue và Timer/TimerTask là gì?

`DelayQueue` và `Timer/TimerTask` đều có thể dùng để thực hiện lập lịch định thời task, nhưng cách thức triển khai của chúng khác nhau. `DelayQueue` dựa trên priority queue để triển khai, chỉ chịu trách nhiệm lưu trữ các phần tử delay và cho phép lấy ra khi hết hạn; còn `Timer/TimerTask` do một background thread đơn duy nhất thực thi các task theo thứ tự, nếu một task nào đó thực thi quá lâu sẽ ảnh hưởng đến việc thực thi của các task khác. Cả hai đều hỗ trợ thêm task trong thời gian chạy, nhưng `DelayQueue` còn có thể gỡ bỏ trực tiếp các phần tử delay trong đó.

## Tài liệu tham khảo

- 《Hiểu sâu Lập trình Concurrency cao: Kỹ thuật cốt lõi JDK》:
- Một hơi nói ra 6 cách triển khai delay queue trong Java (người phỏng vấn cũng phải thán phục): <https://www.jb51.net/article/186192.htm>
- Minh họa source code DelayQueue (Java 8) — Điều kỳ diệu của delay queue: <https://blog.csdn.net/every__day/article/details/113810985>
<!-- @include: @article-footer.snippet.md -->
