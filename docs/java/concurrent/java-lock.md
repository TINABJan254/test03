---
title: Java 锁详解：互斥锁、读写锁、自旋锁与 synchronized 锁优化
description: Java 锁机制系统梳理：从互斥锁、读写锁、自旋锁到 synchronized、ReentrantLock、AQS、StampedLock，讲清锁分类、实现原理、版本差异与选型建议。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: Java锁,互斥锁,读写锁,自旋锁,synchronized,ReentrantLock,AQS,StampedLock,CAS,锁升级,锁优化,Java并发
---

Khi học Java Concurrency, các tên gọi liên quan đến khóa rất dễ khiến mọi người nhầm lẫn: Mutex Lock, Read-Write Lock, Spin Lock, Pessimistic Lock, Optimistic Lock, CAS, AQS, `synchronized`, `ReentrantLock`, `StampedLock`, Biased Lock, Lightweight Lock, Heavyweight Lock.

Các tên gọi này không nằm trong cùng một chiều phân loại.

Có những tên nói về "ai có thể vào vùng tới hạn", như Mutex Lock và Read-Write Lock; có những tên nói về "khi không lấy được khóa thì chờ như thế nào", như Spin Lock và Blocking Lock; có những tên nói về "cài khóa trước khi sửa dữ liệu dùng chung, hay kiểm tra khi submit", như Pessimistic Lock và Optimistic Lock; lại có những tên nói về việc HotSpot tối ưu hóa `synchronized` như thế nào dưới các mức độ tranh chấp khác nhau.

Bài viết này trước hết dựng lên hệ tọa độ cho các loại khóa, sau đó xem xét các công cụ khóa thường dùng trong Java được triển khai thực tế ra sao. Về chi tiết của Pessimistic Lock, Optimistic Lock và CAS, trên website đã có 2 bài viết giới thiệu chi tiết: [Giải thích chi tiết Khóa lạc quan và Khóa bi quan](./optimistic-lock-and-pessimistic-lock.md), [Giải thích chi tiết CAS](./cas.md). Bài viết này chỉ giữ lại ngữ cảnh cần thiết, trọng tâm đặt vào "chuỗi hệ thống khóa được kết nối như thế nào".

PS: Bài viết này chủ yếu lấy HotSpot / OpenJDK làm bối cảnh. Tính tương hỗ monitor và ngữ nghĩa bộ nhớ của `synchronized` thuộc cấp độ quy chuẩn Java/JVM; Object Header, Mark Word, Lightweight Lock, Lock Inflation thuộc về tối ưu hóa triển khai HotSpot, Java language specification không hứa hẹn quy trình cố định. Biased Lock từ JDK 15 trở đi mặc định bị vô hiệu hóa và deprecated các tham số liên quan, từ JDK 18 trở đi các tham số liên quan đã bị obsoleted; kết luận về Virtual Thread và `synchronized` pinning cũng cần phân biệt giữa JDK 21~23 và JDK 24+.

Trước tiên dùng một bảng để bóc tách các chiều phân loại:

| Chiều | Tên điển hình | Câu hỏi trả lời |
| --- | --- | --- |
| Phương thức tương hỗ vùng tới hạn | Khóa tương hỗ (Mutex Lock), Khóa đọc ghi (Read-Write Lock) | Ai có thể vào vùng tới hạn |
| Chiến lược chờ đợi | Spin Lock (Khóa tự xoay), Blocking Lock (Khóa chặn) | Khi không lấy được khóa thì chờ như thế nào |
| Tư tưởng kiểm soát concurrency | Khóa bi quan (Pessimistic Lock), Khóa lạc quan (Optimistic Lock) | Khóa trước rồi sửa, hay kiểm tra khi submit |
| Cơ chế cập nhật nguyên tử | CAS, Lớp Atomic | Làm thế nào để cập nhật biến đơn lẻ mà không bị chặn |
| Tối ưu hóa triển khai JVM | Lightweight Lock, Heavyweight Lock, Nâng cấp/phồng khóa (Lock Inflation) | HotSpot giảm chi phí `synchronized` như thế nào |
| Công cụ khóa trong Java | `synchronized`, `ReentrantLock`, `StampedLock` | Dùng cụ thể cái gì trong code |

## Một ổ khóa rốt cuộc bảo vệ cái gì?

Khóa cần giải quyết là vấn đề vùng tới hạn (critical section). Vùng tới hạn chỉ đoạn code sẽ truy cập trạng thái có thể thay đổi dùng chung, và không thể cho phép nhiều đơn vị thực thi giao thoa thực thi tùy ý.

![临界区保护访问协议示意图：多个线程通过统一加锁入口访问共享状态，绕开锁或更换锁对象都会破坏互斥关系](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/os-lock-critical-section.png)

Ví dụ thao tác tự tăng dưới đây:

```java
count++;
```

Trong mã nguồn chỉ có một dòng, nhưng dòng code này không thể coi là hành động không thể chia nhỏ. Luồng thường phải đọc `count` ra trước, cộng thêm 1, rồi ghi trở lại. Hai luồng đồng thời thực thi, có thể đều đọc được giá trị cũ `0`, tự tính ra `1`, cuối cùng đều ghi `1` trở lại. Cả hai luồng đều thực thi tự tăng, kết quả chỉ cộng thêm 1 lần.

Cách làm của khóa rất trực tiếp: Trước khi vào đoạn code này lấy khóa trước, thực thi xong giải phóng khóa. Chỉ cần tất cả code truy cập cùng một trạng thái dùng chung đều tuân thủ giao ước của cùng một ổ khóa, thì có thể ép các thao tác đọc ghi vốn có thể giao thoa thành một đoạn thực thi loại trừ lẫn nhau ( mutual exclusion).

Ở đây có một câu rất dễ bị bỏ qua: **Khóa thực sự bảo vệ là giao thức truy cập trạng thái đối tượng, bản thân đối tượng sẽ không vì được cài khóa mà tự động an toàn**.

`synchronized (account)` sẽ không kỳ diệu làm cho tất cả các field của `account` đều an toàn. Nếu đoạn code khác bỏ qua ổ khóa này trực tiếp sửa `account.balance`, an toàn luồng vẫn sẽ bị phá hỏng. Khóa học về khóa của MIT 6.005 cũng liên tục nhấn mạnh điểm này: Khóa nên giữ vững bất biến biểu diễn của một trừu tượng dữ liệu nào đó, tiện tay tìm một đối tượng bọc lại không thể đảm bảo bất biến đó luôn được thỏa mãn.

## Khóa tương hỗ: Tại một thời điểm chỉ cho phép một luồng đi vào

Khóa tương hỗ (Mutex Lock) có quy tắc rất đơn giản: Tại một thời điểm, tối đa chỉ có một luồng giữ khóa và đi vào vùng tới hạn.

Trong Java, `synchronized` và `ReentrantLock` đều có thể dùng như Mutex Lock:

```java
class Counter {
    private int count;

    public synchronized void increment() {
        count++;
    }

    public synchronized int get() {
        return count;
    }
}
```

Đổi sang `ReentrantLock`, cách viết sẽ dài dòng hơn một chút, nhưng lấy được nhiều quyền kiểm soát hơn:

```java
class Counter {
    private final ReentrantLock lock = new ReentrantLock();
    private int count;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

`try/finally` không thể bỏ qua. Hành động giải phóng của `synchronized` do JVM làm giúp bạn, khối mã thoát bình thường hay thoát do ngoại lệ đều sẽ giải phóng monitor; `ReentrantLock` là API hiển thị, lấy khóa và mở khóa phải tự phối hợp thành cặp. Tài liệu `ReentrantLock` của Oracle cũng coi "gọi `lock` xong lập tức vào khối `try`" là cách viết khuyến nghị.

Điểm thực sự khó của Mutex Lock không nằm ở cú pháp, mà ở độ mịn của khóa (lock granularity).

Một ổ khóa lớn bọc tất cả thao tác lại là đỡ đau đầu nhất, nhưng độ concurrency thấp; nhiều khóa nhỏ lần lượt bảo vệ các dữ liệu khác nhau, throughput có thể tốt hơn, nhưng thứ tự khóa, deadlock, tính nhất quán trạng thái đều khó quản lý hơn. OSTEP khi giảng về POSIX mutex cũng có nhắc đến sự đánh đổi này: Dữ liệu khác nhau dùng khóa khác nhau có thể tăng concurrency, nhưng programmer bắt buộc phải hiểu rõ mỗi ổ khóa rốt cuộc bảo vệ khối trạng thái nào.

## Khóa đọc ghi: Đọc-Đọc dùng chung, thao tác ghi độc chiếm

Khóa tương hỗ đối với thao tác đọc cũng rất nghiêm ngặt: Chỉ cần một luồng đang đọc, luồng khác cũng không thể vào đọc. Nhưng nhiều đối tượng nghiệp vụ có một đặc điểm: Đọc không làm thay đổi trạng thái, nhiều luồng đọc đồng thời thực thi cũng sẽ không phá hỏng lẫn nhau.

Read-Write Lock chính là chuẩn bị cho kịch bản này.

Nó chia truy cập thành 2 loại:

- Khóa đọc: Khóa dùng chung (Shared Lock), nhiều luồng có thể đồng thời giữ.
- Khóa ghi: Khóa độc chiếm (Exclusive Lock), chỉ có thể một luồng giữ, và khóa ghi tương hỗ loại trừ với khóa đọc.

Quy tắc tương ứng cũng rất dễ nhớ:

- Đọc-Đọc không loại trừ nhau.
- Đọc-Ghi loại trừ nhau.
- Ghi-Ghi loại trừ nhau.

Triển khai điển hình trong Java là `ReentrantReadWriteLock`:

```java
class ProfileCache {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    private final Map<Long, String> cache = new HashMap<>();

    public String get(long userId) {
        readLock.lock();
        try {
            return cache.get(userId);
        } finally {
            readLock.unlock();
        }
    }

    public void put(long userId, String profile) {
        writeLock.lock();
        try {
            cache.put(userId, profile);
        } finally {
            writeLock.unlock();
        }
    }
}
```

Read-Write Lock phù hợp với kịch bản đọc nhiều ghi ít, thao tác đọc đủ ngắn, cấu hình dữ liệu không dễ bị phá hỏng. Nó không phù hợp với tất cả cache, cũng chưa chắc nhanh hơn Mutex Lock. Nếu thao tác ghi rất thường xuyên, luồng đọc và luồng ghi sẽ liên tục cản đường nhau, chi phí bảo trì của Read-Write Lock ngược lại có thể triệt tiêu lợi ích.

Java còn cung cấp `StampedLock`, nó hỗ trợ khóa ghi, khóa đọc bi quan và đọc lạc quan. Đọc lạc quan không thực sự giữ khóa đọc truyền thống; nó sẽ lấy một stamp trước, sau khi đọc xong mới kiểm tra lại xem trong thời gian đó có thao tác ghi xảy ra hay không:

```java
class Point {
    private final StampedLock lock = new StampedLock();
    private double x;
    private double y;

    double distanceFromOrigin() {
        long stamp = lock.tryOptimisticRead();
        double currentX = x;
        double currentY = y;
        if (!lock.validate(stamp)) {
            stamp = lock.readLock();
            try {
                currentX = x;
                currentY = y;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return Math.hypot(currentX, currentY);
    }
}
```

Đọc lạc quan của `StampedLock` có ranh giới: Dữ liệu đọc được có thể tạm thời không nhất quán, nên chỉ thích hợp với kịch bản đọc ngắn có thể hoàn thành việc đọc trong biến cục bộ và có thể dùng việc đọc lại sau khi `validate` thất bại để bọc lót. Nó cũng rất khó thay thế trực tiếp `ReentrantReadWriteLock`, đặc biệt cần lưu ý nó không hỗ trợ reentrant.

## Khóa tự xoay: Không bị chặn, xoay tại chỗ chờ một chút

Khi luồng không lấy được khóa, thường có 2 cách chờ đợi:

- Chặn (Blocking): Treo luồng hiện tại, để hệ điều hành đánh thức sau.
- Tự xoay (Spinning): Không treo luồng, lặp kiểm tra trên CPU xem khóa đã khả dụng chưa.

Spin Lock phù hợp với kịch bản vùng tới hạn cực kỳ ngắn. Ví dụ luồng giữ khóa sắp sửa giải phóng khóa ngay, nếu luồng chờ đợi trực tiếp bị chặn, chi phí treo và đánh thức luồng có thể còn cao hơn việc "xoay tại chỗ vài vòng" chờ đợi.

Vấn đề cũng nằm ở đây: Tự xoay sẽ trả giá bằng CPU, nó sẽ liên tục chiếm dụng CPU. Nếu khóa thời gian dài không giải phóng, hoặc luồng chờ đợi rất nhiều, tự xoay sẽ lãng phí thời gian CPU vào việc quay không.

Trong code Java có thể dùng CAS viết ra một ví dụ Spin Lock rất nhỏ:

```java
class SpinLock {
    private final AtomicReference<Thread> owner = new AtomicReference<>();

    public void lock() {
        Thread current = Thread.currentThread();
        while (!owner.compareAndSet(null, current)) {
            Thread.onSpinWait();
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        if (!owner.compareAndSet(current, null)) {
            throw new IllegalMonitorStateException();
        }
    }
}
```

Đoạn code này chỉ dùng để giải thích mối quan hệ "Spin + CAS", không khuyến nghị trực tiếp mang đi làm khóa nghiệp vụ. Khóa thực tế phải xem xét các vấn đề như reentrant, tính công bằng, ngắt, timeout, giải phóng khi ngoại lệ, chỉ số giám sát, hàng đợi chờ... JDK đã đóng gói các độ phức tạp này trong `synchronized`, `ReentrantLock`, bộ đồng bộ AQS và các lớp Atomic.

## Vị trí của Khóa bi quan, Khóa lạc quan và CAS

Khóa bi quan (Pessimistic Lock) và Khóa lạc quan (Optimistic Lock) mô tả hai tư tưởng kiểm soát concurrency, không tương ứng với một lớp Java cố định nào.

Khóa bi quan giả định xung đột rất có thể xảy ra, nên cài khóa tài nguyên trước rồi mới thao tác. `synchronized`, `ReentrantLock`, `SELECT ... FOR UPDATE` trong database đều là các ví dụ thường gặp.

Khóa lạc quan giả định xung đột không thường xuyên, trước tiên không chặn người khác, khi submit sửa đổi mới kiểm tra dữ liệu có bị sửa hay chưa. Trường `version` trong database, CAS trong các lớp Atomic của Java đều thuộc hướng này.

CAS (Compare-And-Swap) có thể hiểu là một phương thức cập nhật nguyên tử được phần cứng hỗ trợ: Chỉ khi giá trị trong bộ nhớ vẫn bằng giá trị cũ kỳ vọng, mới sửa nó thành giá trị mới. Nếu không chứng tỏ đã có người sửa trước, luồng hiện tại có thể chọn thử lại, từ bỏ hoặc đi theo logic hạ cấp.

CAS thường có 3 vấn đề:

- Thử lại khi thất bại sẽ tiêu thụ CPU, tranh chấp càng dữ dội càng biểu hiện rõ.
- Chỉ xử lý tự nhiên được một biến đơn lẻ, tính nhất quán của nhiều biến phải thiết kế thêm.
- Vấn đề ABA: Giá trị từ A biến thành B, rồi lại biến về A, chỉ nhìn giá trị tưởng như chưa từng thay đổi.

ABA có thể dùng số phiên bản, nhãn thời gian hoặc tham chiếu có đánh dấu để giải quyết. Trong Java có các công cụ như `AtomicStampedReference` và `AtomicMarkableReference`, nhưng trong code nghiệp vụ cách làm phổ biến hơn là làm cho bản thân data model mang số phiên bản.

Phần này nếu tiếp tục triển khai sẽ trùng lặp với các bài viết đã có. Muốn xem cách triển khai, ví dụ số phiên bản, xử lý ABA và mã nguồn các lớp Atomic, có thể đọc tiếp:

- [Giải thích chi tiết Khóa lạc quan và Khóa bi quan](./optimistic-lock-and-pessimistic-lock.md)
- [Giải thích chi tiết CAS](./cas.md)
- [Tổng kết các lớp Atomic nguyên tử](./atomic-classes.md)

## Bản chất của synchronized: monitor, bytecode và ngữ nghĩa bộ nhớ

`synchronized` là cơ chế đồng bộ được tích hợp sẵn trong ngôn ngữ Java, có thể tu sửa phương thức instance, phương thức static, cũng có thể bọc khối mã.

```java
class Account {
    private long balance;

    public synchronized void deposit(long amount) {
        balance += amount;
    }

    public long balance() {
        synchronized (this) {
            return balance;
        }
    }
}
```

Biểu hiện của phương thức đồng bộ và khối mã đồng bộ ở cấp độ bytecode không hoàn toàn giống nhau:

- Phương thức đồng bộ phụ thuộc vào cờ truy cập phương thức `ACC_SYNCHRONIZED`.
- Khối mã đồng bộ sẽ sinh ra các lệnh `monitorenter` và `monitorexit`.

Bất kể hình thức biểu hiện ra sao, ngữ nghĩa đều là đi vào monitor, thoát khỏi monitor. Java Language Specification còn quy định mối quan hệ happens-before giữa việc giải phóng khóa và việc lấy khóa tiếp theo: Các thao tác ghi trước khi một luồng giải phóng một khóa nào đó sẽ hiển thị đối với luồng tiếp theo lấy được cùng một khóa đó.

Đây cũng là điểm khác biệt giữa `synchronized` và khái niệm khóa thông thường "chỉ làm loại trừ lẫn nhau". Nó đồng thời cung cấp tính loại trừ lẫn nhau và tính nhìn thấy của bộ nhớ. Chỉ bảo vệ vùng tới hạn mà không thiết lập tính nhìn thấy, luồng khác vẫn có thể đọc được giá trị cũ.

Ngoài ra, `synchronized` là có thể reentrant. Khi một luồng đã nắm giữ monitor của một đối tượng nào đó, có thể đi vào lại đoạn code được bảo vệ bởi cùng một khóa đó, JVM sẽ ghi lại số lần reentrant, khi thoát ra sẽ giảm dần từng lớp.

```java
class ReentrantDemo {
    public synchronized void outer() {
        inner();
    }

    public synchronized void inner() {
        // Cùng một luồng có thể đi vào lại monitor của this
    }
}
```

## Tối ưu hóa khóa synchronized: Đừng học vẹt "Lock Escalation" thành khẩu quyết cố định

Nhiều tài liệu sẽ giảng `synchronized` thành "Không khóa -> Khóa thiên vị (Biased Lock) -> Khóa hạng nhẹ (Lightweight Lock) -> Khóa hạng nặng (Heavyweight Lock)". Manh mối này rất có ích cho việc hiểu tối ưu hóa thời kỳ đầu của HotSpot, nhưng không thể tách rời phiên bản.

Sau JDK 6, HotSpot đã thực hiện lượng lớn tối ưu cho `synchronized`. Biased Lock hướng tới kịch bản "luôn luôn là cùng một luồng đi vào cùng một khóa"; Lightweight Lock hướng tới kịch bản "tranh chấp không dữ dội, nhiều luồng đi vào lệch giờ nhau"; Heavyweight Lock sẽ dùng đến ObjectMonitor, luồng tranh chấp có thể bị chặn và đánh thức.

Sự khác biệt phiên bản cần ghi nhớ riêng:

- JDK 6 đến JDK 14: Biased Lock là một trong những tối ưu phổ biến của HotSpot.
- JDK 15: JEP 374 tắt mặc định Biased Lock, và deprecated các tham số liên quan.
- JDK 18: Các tham số liên quan đến Biased Lock bị obsoleted, truyền vào sẽ bị bỏ qua và đưa ra cảnh báo.
- JDK 21 đến JDK 23: Virtual Thread khi bị chặn trong `synchronized` có thể làm pin (gắn chặt) Platform Thread.
- JDK 24: JEP 491 cải tiến sự phối hợp giữa Virtual Thread và `synchronized`, Virtual Thread bị chặn trên `synchronized` có thể giải phóng Platform Thread bên dưới, giảm bớt vấn đề pinning.

Do đó, khi phỏng vấn hoặc viết bài có thể nói "HotSpot từng thông qua Biased Lock, Lightweight Lock, Heavyweight Lock để giảm chi phí `synchronized`", nhưng đừng nói Biased Lock thành con đường mặc định mà JDK hiện đại chắc chắn sẽ đi.

Trong kỹ thuật kết luận quan trọng hơn là một điều khác: Câu nói "có thể không dùng thì không dùng" năm xưa đã không còn phù hợp với `synchronized` ngày nay. Trong kịch bản tương hỗ thông thường, cú pháp của nó đơn giản, giải phóng ngoại lệ an toàn, tối ưu JIT chín thành. Chỉ khi bạn cần khóa công bằng, lấy khóa có thể ngắt, lấy khóa timeout, nhiều hàng đợi điều kiện, mới chuyển sang `ReentrantLock` một cách tự nhiên hơn.

## Kết nối giữa ReentrantLock, Condition và AQS như thế nào

`ReentrantLock` cung cấp năng lực kiểm soát tinh tế hơn `synchronized`:

- Có thể chọn khóa công bằng (Fair Lock) hoặc khóa không công bằng (Non-fair Lock).
- Có thể dùng `lockInterruptibly()` phản hồi ngắt.
- Có thể dùng `tryLock()` hoặc `tryLock(timeout, unit)` tránh chờ đợi vô hạn.
- Có thể tạo nhiều `Condition`, tách các điều kiện chờ đợi khác nhau ra quản lý.

Một cách viết điển hình như sau:

```java
class BoundedBuffer<E> {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    private final Queue<E> queue = new ArrayDeque<>();
    private final int capacity;

    BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public void put(E item) throws InterruptedException {
        lock.lockInterruptibly();
        try {
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.add(item);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public E take() throws InterruptedException {
        lock.lockInterruptibly();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            E item = queue.remove();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

Vòng lặp `while` ở đây cũng không thể tùy tiện đổi thành `if`. Luồng sau khi được đánh thức, chỉ có thể giải thích là "có cơ hội cạnh tranh lại khóa và kiểm tra điều kiện", không đại diện cho điều kiện chắc chắn thỏa mãn. Đánh thức giả (spurious wakeup), nhiều luồng chờ tranh chấp, điều kiện bị luồng khác tiêu thụ trước, đều yêu cầu sau khi tỉnh dậy phải kiểm tra lại lần nữa.

`ReentrantLock` bên dưới phụ thuộc AQS (AbstractQueuedSynchronizer). AQS có thể hiểu thô trước thành một bộ framework bộ đồng bộ hóa: Dùng một `state` biểu thị trạng thái đồng bộ, dùng hàng đợi FIFO quản lý các luồng chưa tranh được tài nguyên, phối hợp với CAS, `LockSupport.park/unpark` hoàn thành việc xếp hàng, chặn và đánh thức.

Nhiều công cụ concurrency đều xây dựng trên AQS, như `ReentrantLock`, `Semaphore`, `CountDownLatch`, `ReentrantReadWriteLock`. Nếu muốn tiếp tục bóc tách luồng tuyến hàng đợi, `state`, CAS và chặn/đánh thức này, có thể đọc tiếp [Giải thích chi tiết AQS](./aqs.md) và [Từ triển khai ReentrantLock nhìn vào nguyên lý và ứng dụng AQS](./reentrantlock.md).

## Chọn khóa Java như thế nào?

Khi chọn khóa đừng vội so sánh "cái nào nhanh nhất". Biểu hiện của khóa có liên quan đến độ dài vùng tới hạn, cường độ tranh chấp, số lượng luồng, cách xử lý sau khi thất bại. Hãy hỏi rõ vài câu hỏi trước: Đoạn code này bảo vệ trạng thái dùng chung nào? Thời gian giữ khóa khoảng bao lâu? Khi không lấy được khóa có thể chờ không? Sau khi chờ thất bại là trả về, thử lại, hay ném lỗi trực tiếp?

Nếu chỉ bảo vệ một đoạn trạng thái nhỏ trong tiến trình JVM, như update vài field, bảo trì một Map trong bộ nhớ, chuyển đổi trạng thái đối tượng, `synchronized` thường là đủ rồi. Nó viết ngắn, khi thoát khối mã tự động giải phóng khóa, cũng bớt được rủi ro viết tay `unlock()` bị bỏ sót. Cho đến khi code cần lấy khóa timeout, lấy khóa có thể ngắt, khóa công bằng, hoặc cần dùng nhiều `Condition` quản lý các hàng đợi chờ khác nhau, hãy đổi sang `ReentrantLock` sẽ tiện tay hơn.

Khi đọc nhiều ghi ít, có thể xem `ReentrantReadWriteLock`. Trọng tâm ở đây là "ghi ít", chỉ có phương thức đọc thôi là chưa đủ. Nếu thao tác ghi rất thường xuyên, khóa đọc và khóa ghi sẽ liên tục cản đường nhau, việc bảo trì trạng thái đọc ghi cũng có chi phí, cuối cùng chưa chắc đã hời hơn một khóa tương hỗ. Đọc lạc quan của `StampedLock` càng kén kịch bản: Logic đọc phải ngắn, đọc phải trạng thái trung gian cũng không được xảy ra vấn đề lớn, và bắt buộc phải chấp nhận việc đọc lại một lần sau khi kiểm tra thất bại.

Nếu chỉ cập nhật một bộ đếm, cờ trạng thái hoặc tham chiếu, ưu tiên xem các lớp Atomic, `LongAdder`, `LongAccumulator`. Chúng phù hợp với cập nhật nguyên tử rất ngắn, không phù hợp cho cả một quy trình nghiệp vụ vào vòng lặp thử lại CAS. Quy trình nghiệp vụ càng dài, thử lại thất bại càng dễ tiêu thụ CPU vào các vòng lặp vô ích, cũng khó xử lý tác dụng phụ (side effect).

Nếu vấn đề đã vượt qua ranh giới JVM, ví dụ nhiều thể hiện ứng dụng đồng thời sửa cùng một bản ghi database, khóa trong Java sẽ không quản được nữa. Khi xung đột không thường xuyên, có thể dùng số phiên bản làm khóa lạc quan; khi xung đột tương đối thường xuyên, bắt buộc phải sửa đổi nhất quán mạnh, thường phải quay về cơ chế database như khóa dòng database, `SELECT ... FOR UPDATE`, ràng buộc duy nhất (unique constraint). Đi ra xa hơn đến tương hỗ loại trừ xuyên service (cross-service), thì cần các hệ thống bên ngoài như Redis, ZooKeeper, database đứng ra gánh vác, không thể trông chờ vào `synchronized` hay `ReentrantLock`.

Độ mịn của khóa (lock granularity) cũng đừng một mực theo đuổi "nhỏ". Một ổ khóa lớn dễ đảm bảo tính đúng đắn, nhưng throughput có thể bị ảnh hưởng; tách thành nhiều khóa nhỏ, tranh chấp sẽ ít hơn một chút, nhưng thứ tự khóa, deadlock và chi phí troubleshooting đều sẽ tăng lên. Nhiều lúc, dùng một khóa rõ ràng để giữ vững bất biến trước, rồi dựa vào kết quả test tải để tách khóa, sẽ chắc chắn hơn việc vừa vào đã thiết kế một đống khóa độ mịn nhỏ.

## Các cạm bẫy thường gặp

**Đối tượng khóa không ổn định.**

Một số code nhìn thì có vẻ đã cài khóa, thực tế có thể đã khóa vào các đối tượng khác nhau. Nguyên nhân thường gặp là đối tượng khóa bị thay đổi, ví dụ kết quả nối chuỗi, đối tượng autoboxing, field có thể gán lại giá trị. Luồng A vào khóa đối tượng cũ, luồng B vào khóa đối tượng mới, hai bên không ảnh hưởng lẫn nhau, vùng tới hạn đã bị xé lẻ.

```java
private Object lock = new Object();

public void update() {
    synchronized (lock) {
        lock = new Object(); // Luồng sau đó có thể sẽ khóa vào một ổ khóa khác
    }
}
```

Nếu cần đối tượng khóa riêng biệt, thường định nghĩa nó thành `private final`, và không bộc lộ nó ra cho code bên ngoài.

**Khóa đối tượng hiển thị ra bên ngoài.**

`synchronized (this)` và `synchronized (SomeClass.class)` đôi khi không vấn đề gì, nhưng chúng cũng có thể bị code bên ngoài dùng để cài khóa, dẫn đến bạn không thể kiểm soát phạm vi tranh chấp khóa. Code thư viện đặc biệt phải cẩn trọng, thường khuyến nghị đối tượng khóa private final.

```java
private final Object lock = new Object();
```

**Thực hiện thao tác chậm trong khi giữ khóa.**

Khi giữ khóa mà truy cập database, gọi RPC, ghi file lớn, đều sẽ kéo dài thời gian chiếm dụng khóa. Khóa bị chiếm càng lâu, luồng chờ đợi càng nhiều, rủi ro timeout, kiệt quệ ThreadPool và deadlock đều cao lên.

**Thứ tự khóa không nhất quán.**

Hai luồng lần lượt cài khóa theo thứ tự `A -> B` và `B -> A`, rất dễ hình thành deadlock. Khi dùng nhiều khóa đồng thời, phải sắp xếp một thứ tự ổn định toàn cục cho tài nguyên. Bài giới thiệu hoàn chỉnh về deadlock có thể xem [Giải thích chi tiết Deadlock](../../cs-basics/operating-system/dead-lock.md).

**Nhầm lẫn giữa lớp an toàn luồng và thao tác phức hợp.**

Đơn lần `get`, `put` của `ConcurrentHashMap` là an toàn luồng, nhưng "kiểm tra trước không tồn tại rồi mới chèn" là thao tác phức hợp, cần dùng phương thức nguyên tử như `computeIfAbsent`, hoặc đồng bộ hóa bổ sung.

```java
// Không khuyến nghị: Giữa containsKey và put có thể bị luồng khác chèn vào
if (!map.containsKey(key)) {
    map.put(key, createValue());
}

// Khuyến nghị: Giao logic phức hợp cho phương thức nguyên tử của ConcurrentHashMap
map.computeIfAbsent(key, ignored -> createValue());
```

**Chỉ nhìn vào khóa, không nhìn vào bể tài nguyên (resource pool).**

Nhiều trường hợp "kẹt" trên sản xuất không liên quan đến deadlock của khóa Java, các luồng có thể đều đang chờ connection database, connection HTTP, hàng đợi ThreadPool hoặc service bên ngoài trả về. Thấy `WAITING` trong thread stack chỉ thể hiện luồng đang chờ, phán đoán deadlock còn phải tìm được vòng chờ đợi ổn định.

## Tóm tắt

Khóa là tên gọi chung của một nhóm công cụ kiểm soát concurrency, khái niệm này tương đối lớn.

Khóa tương hỗ và Khóa đọc ghi trả lời "Ai có thể vào vùng tới hạn"; Spin lock và Blocking lock trả lời "Không lấy được khóa thì chờ thế nào"; Khóa bi quan và Khóa lạc quan trả lời "Xử lý trước và sau khi xảy ra xung đột thế nào"; CAS và Lớp Atomic giải quyết cập nhật nguyên tử biến đơn lẻ; `synchronized`, `ReentrantLock`, `StampedLock`, AQS là cách Java đưa các tư tưởng này vào trong code.

Khi thực sự viết code, trước tiên hãy tìm ra trạng thái dùng chung và bất biến, rồi quyết định khóa bảo vệ cái gì, độ mịn khóa to hay nhỏ, chờ đợi có thể ngắt không, thất bại có thể thử lại không. Công cụ chỉ là phương tiện, thứ thực sự phải giữ vững là cùng một giao thức đồng bộ: Tất cả các đường truy cập trạng thái dùng chung đều bắt buộc phải tuân thủ nó.

## Tài liệu tham khảo

- [The Java Language Specification, Chapter 17. Threads and Locks](https://docs.oracle.com/javase/specs/jls/se24/html/jls-17.html)
- [The Java Virtual Machine Specification, monitorenter](https://docs.oracle.com/javase/specs/jvms/se24/html/jvms-6.html#jvms-6.5.monitorenter)
- [Oracle Java API: Lock](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html)
- [Oracle Java API: ReentrantLock](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html)
- [Oracle Java API: StampedLock](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/StampedLock.html)
- [OpenJDK JEP 374: Deprecate and Disable Biased Locking](https://openjdk.org/jeps/374)
- [OpenJDK JEP 491: Synchronize Virtual Threads without Pinning](https://openjdk.org/jeps/491)
- [OSTEP 中文版：Locks](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/28.pdf)
- [MIT 6.005: Locks and Synchronization](http://web.mit.edu/6.005/www/fa15/classes/23-locks/)
