---
title: 从ReentrantLock的实现看AQS的原理及应用
description: ReentrantLock与AQS原理深度解析：详解ReentrantLock可重入锁实现、公平锁与非公平锁区别、基于AQS的加锁解锁流程、与synchronized性能对比。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: ReentrantLock,AQS,公平锁,非公平锁,可重入锁,lock unlock,ReentrantLock原理,synchronized对比
---

> Bài viết này được trích từ: <https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html>
>
> Tác giả: Đội ngũ kỹ thuật Meituan

Hầu hết các lớp đồng bộ trong Java (Semaphore, ReentrantLock...) đều được triển khai dựa trên AbstractQueuedSynchronizer (gọi tắt là AQS). AQS là một framework đơn giản cung cấp tính năng quản lý nguyên tử trạng thái đồng bộ, làm bị chặn và đánh thức luồng cũng như mô hình hàng đợi.

Bài viết này sẽ đi từ tầng ứng dụng dần dần đi sâu vào tầng nguyên lý, và thông qua các đặc tính cơ bản của ReentrantLock cũng như sự liên kết giữa ReentrantLock và AQS, để giải thích chi tiết các kiến thức liên quan đến Exclusive lock của AQS, đồng thời áp dụng mode Hỏi-Đáp để giúp mọi người hiểu AQS. Do giới hạn độ dài, bài viết này chủ yếu trình bày logic của Exclusive lock trong AQS và Sync Queue, không giảng phần bao gồm Shared lock và Condition Queue (Trọng tâm của bài viết này là phân tích nguyên lý AQS, chỉ giới thiệu đơn giản ReentrantLock, các bạn quan tâm có thể đọc thêm mã nguồn ReentrantLock).

> Phân tích mã nguồn bài viết dựa trên JDK 8. Triển khai bên trong AQS về sau liên tục tiến hóa: Trong JDK 11 vẫn có thể thấy các field và method chính đề cập trong bài, các field node trong JDK 17 và phiên bản hiện tại cùng triển khai vào hàng đợi, chờ đợi đã có sự thay đổi khá lớn. Tư tưởng cốt lõi về trạng thái đồng bộ, hàng đợi chờ cũng như lấy/giải phóng tài nguyên vẫn có thể làm cơ sở để hiểu.

## 1 ReentrantLock

### 1.1 Khái quát đặc tính ReentrantLock

ReentrantLock có nghĩa là Khóa reentrant (khóa có thể vào lại), chỉ việc một luồng có khả năng lặp lại cài khóa đối với một tài nguyên găng. Để giúp mọi người hiểu tốt hơn đặc tính của ReentrantLock, trước tiên chúng ta so sánh ReentrantLock với Synchronized thường dùng, các đặc tính của nó như sau (phần màu xanh lục là điểm bóc tách chính của bài viết này):

![](https://p0.meituan.net/travelcube/412d294ff5535bbcddc0d979b2a339e6102264.png)

Dưới đây thông qua pseudo-code, tiến hành so sánh trực quan hơn:

```java
// **************************Cách sử dụng Synchronized**************************
// 1. Dùng cho khối code
synchronized (this) {}
// 2. Dùng cho đối tượng
synchronized (object) {}
// 3. Dùng cho phương thức
public synchronized void test () {}
// 4. Có tính reentrant
for (int i = 0; i < 100; i++) {
  synchronized (this) {}
}
// **************************Cách sử dụng ReentrantLock**************************
public void test () throws Exception {
  // 1. Khởi tạo lựa chọn Fair lock, Non-fair lock
  ReentrantLock lock = new ReentrantLock(true);
  // 2. Có thể dùng cho khối code
  lock.lock();
  try {
    // 3. Hỗ trợ nhiều phương thức cài khóa, linh hoạt hơn; Có đặc tính reentrant
    if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
      try {
        // Logic thực thi sau khi lấy lock lần thứ hai
      } finally {
        // Mỗi lần cài khóa thành công đều phải tương ứng giải phóng một lần
        lock.unlock();
      }
    }
  } finally {
    lock.unlock();
  }
}
```

### 1.2 Mối liên kết giữa ReentrantLock và AQS

Thông qua phần trên chúng ta đã hiểu, ReentrantLock hỗ trợ Fair lock và Non-fair lock (về phân tích nguyên lý Fair lock và Non-fair lock, có thể tham khảo 《[Chuyện về "Lock" Java không thể không nói](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749434&idx=3&sn=5ffa63ad47fe166f2f1a9f604ed10091&chksm=bd12a5778a652c61509d9e718ab086ff27ad8768586ea9b38c3dcf9e017a8e49bcae3df9bcc8&scene=38#wechat_redirect)》), và bên dưới của ReentrantLock chính là do AQS triển khai. Vậy ReentrantLock liên kết với AQS thông qua Fair lock và Non-fair lock như thế nào? Chúng ta tập trung hiểu mối quan hệ giữa chúng và AQS từ quá trình cài khóa của hai cái này (trong quá trình cài khóa sự liên kết với AQS tương đối rõ ràng, quy trình mở khóa sẽ giới thiệu phía sau).

Quy trình cài khóa trong mã nguồn Non-fair lock như sau:

```java
// java.util.concurrent.locks.ReentrantLock#NonfairSync

// Non-fair lock
static final class NonfairSync extends Sync {
  ...
  final void lock() {
    if (compareAndSetState(0, 1))
      setExclusiveOwnerThread(Thread.currentThread());
    else
      acquire(1);
    }
  ...
}
```

Ý nghĩa đoạn code này là:

- Nếu thông qua CAS thiết lập biến State (trạng thái đồng bộ) thành công, tức là lấy lock thành công, thì thiết lập luồng hiện tại thành luồng độc chiếm.
- Nếu thông qua CAS thiết lập biến State (trạng thái đồng bộ) thất bại, tức lấy lock thất bại, thì đi vào phương thức Acquire để xử lý tiếp theo.

Bước thứ nhất rất dễ hiểu, nhưng sau khi bước thứ hai lấy lock thất bại, chiến lược xử lý tiếp theo là như thế nào? Phần này có thể có những suy nghĩ sau:

- Quy trình tiếp theo khi một luồng nào đó lấy lock thất bại là gì? Có hai khả năng sau:

(1) Thiết lập kết quả lấy lock của luồng hiện tại thành thất bại, quy trình lấy lock kết thúc. Thiết kế này sẽ giảm cực lớn độ concurrency của hệ thống, không thỏa mãn nhu cầu thực tế của chúng ta. Nên cần quy trình dưới đây, tức là quy trình xử lý của framework AQS.

(2) Tồn tại một cơ chế xếp hàng chờ đợi nào đó, luồng tiếp tục chờ đợi, vẫn giữ lại khả năng lấy lock, quy trình lấy lock vẫn đang tiếp tục.

- Đối với trường hợp thứ hai của câu hỏi 1, đã nói đến cơ chế xếp hàng chờ đợi, vậy thì nhất định sẽ có một hàng đợi nào đó hình thành, hàng đợi như vậy là cấu trúc dữ liệu gì?
- Luồng nằm trong cơ chế xếp hàng chờ đợi, khi nào thì có cơ hội lấy lock?
- Nếu luồng nằm trong cơ chế xếp hàng chờ đợi liên tục không thể lấy lock, thì cần liên tục chờ đợi sao, hay có chiến lược khác để giải quyết vấn đề này?

Mang theo những câu hỏi về Non-fair lock này, lại xem cách lấy lock trong mã nguồn Fair lock:

```java
// java.util.concurrent.locks.ReentrantLock#FairSync

static final class FairSync extends Sync {
  ...
  final void lock() {
    acquire(1);
  }
  ...
}
```

Nhìn thấy đoạn code này, chúng ta có thể tồn tại nghi vấn này: Hàm Lock thông qua phương thức Acquire để tiến hành cài khóa, nhưng cụ thể cài khóa như thế nào?

Kết hợp quy trình cài khóa của Fair lock và Non-fair lock, mặc dù trên quy trình có điểm khác biệt nhất định, nhưng đều gọi phương thức Acquire, mà phương thức Acquire lại là phương thức cốt lõi trong AQS - lớp cha của FairSync và UnfairSync.

Đối với các câu hỏi đề cập ở trên, thực ra trong mã nguồn lớp ReentrantLock đều không thể trả lời được, mà đáp án của những câu hỏi này, đều nằm trong AbstractQueuedSynchronizer - lớp chứa phương thức Acquire, tức là trọng tâm của bài viết này — AQS. Dưới đây chúng ta sẽ giới thiệu chi tiết về AQS cũng như sự liên kết giữa ReentrantLock và AQS (đáp án các câu hỏi liên quan sẽ được giải đáp trong mục 2.3.5).

## 2 AQS

Đầu tiên, chúng ta thông qua sơ đồ kiến trúc dưới đây để hiểu tổng thể về framework AQS:

![](https://p1.meituan.net/travelcube/82077ccf14127a87b77cefd1ccf562d3253591.png)

- Trong hình trên phần có màu là Method, phần không màu là Attribute.
- Tổng thể mà nói, framework AQS chia thành 5 tầng, từ trên xuống dưới từ nông đến sâu, từ API bộc lộ ra ngoài của AQS đến dữ liệu nền tảng bên dưới.
- Khi có bộ đồng bộ hóa tự định nghĩa kết nối vào, chỉ cần override phần phương thức cần thiết ở tầng thứ nhất là được, không cần quan tâm quy trình triển khai cụ thể bên dưới. Khi bộ đồng bộ hóa tự định nghĩa tiến hành thao tác cài khóa hoặc mở khóa, trước tiên trải qua API tầng thứ nhất đi vào phương thức bên trong AQS, sau đó trải qua tầng thứ hai để lấy lock, tiếp đó đối với quy trình lấy lock thất bại, đi vào hàng đợi chờ ở tầng 3 và tầng 4 để xử lý, mà các cách thức xử lý này đều dựa vào tầng cung cấp dữ liệu nền tảng ở tầng thứ 5.

Dưới đây chúng ta sẽ đi từ tổng thể đến chi tiết, từ quy trình đến phương thức để bóc tách từng cái một về framework AQS, quá trình phân tích chính như sau:

![](https://p1.meituan.net/travelcube/d2f7f7fffdc30d85d17b44266c3ab05323338.png)

### 2.1 Khái quát nguyên lý

Tư tưởng cốt lõi của AQS là, nếu tài nguyên dùng chung được yêu cầu rảnh rỗi, thì thiết lập luồng yêu cầu tài nguyên hiện tại thành luồng làm việc có hiệu lực, thiết lập tài nguyên dùng chung thành trạng thái bị khóa; nếu tài nguyên dùng chung bị chiếm dụng, thì cần một cơ chế bị chặn chờ đánh thức nhất định để đảm bảo việc phân bổ lock. Cơ chế này chủ yếu sử dụng biến thể hàng đợi CLH để triển khai, đưa các luồng tạm thời chưa lấy được lock vào hàng đợi.

CLH: Craig, Landin and Hagersten queue, là danh sách liên kết một chiều, hàng đợi trong AQS là hàng đợi hai chiều ảo biến thể CLH (FIFO), AQS thông qua việc đóng gói mỗi luồng yêu cầu tài nguyên dùng chung thành một node để triển khai việc phân bổ lock.

Sơ đồ nguyên lý chính như sau:

![](https://p0.meituan.net/travelcube/7132e4cef44c26f62835b197b239147b18062.png)

AQS sử dụng một biến thành viên kiểu int Volatile để biểu thị trạng thái đồng bộ, thông qua hàng đợi FIFO tích hợp sẵn để hoàn thành công việc xếp hàng lấy tài nguyên, thông qua CAS hoàn thành việc sửa đổi giá trị State.

#### 2.1.1 Cấu trúc dữ liệu AQS

Trước tiên hãy xem cấu trúc dữ liệu cơ bản nhất trong AQS — Node, Node chính là node trong hàng đợi biến thể CLH ở trên.

![](https://p1.meituan.net/travelcube/960271cf2b5c8a185eed23e98b72c75538637.png)

Giải thích ý nghĩa của một vài phương thức và giá trị thuộc tính:

| Phương thức và giá trị thuộc tính | Ý nghĩa |
| :--- | :--- |
| waitStatus | Trạng thái hiện tại của node trong hàng đợi |
| thread | Biểu thị luồng ở trong node đó |
| prev | Con trỏ tiền nhiệm |
| predecessor | Trả về node tiền nhiệm, nếu không có thì ném npe |
| nextWaiter | Trỏ tới node tiếp theo ở trạng thái CONDITION (do bài viết này không giảng hàng đợi Condition Queue, con trỏ này không giới thiệu nhiều) |
| next | Con trỏ kế nhiệm |

Hai mode lock của luồng:

| Mode | Ý nghĩa |
| :--- | :--- |
| SHARED | Biểu thị luồng đang chờ lock theo mode shared |
| EXCLUSIVE | Biểu thị luồng đang chờ lock theo mode exclusive |

`waitStatus` có các giá trị enum dưới đây:

| Enum | Ý nghĩa |
| :--- | :--- |
| 0 | Giá trị mặc định khi một Node được khởi tạo |
| CANCELLED | Là 1, biểu thị yêu cầu lấy lock của luồng đã bị hủy |
| CONDITION | Là -2, biểu thị node ở trong hàng đợi chờ, luồng của node chờ đánh thức |
| PROPAGATE | Là -3, field này chỉ được dùng khi luồng hiện tại ở trường hợp SHARED |
| SIGNAL | Là -1, biểu thị luồng đã chuẩn bị xong xuôi, chỉ chờ tài nguyên giải phóng |

#### 2.1.2 Trạng thái đồng bộ State

Sau khi hiểu cấu trúc dữ liệu, tiếp theo tìm hiểu trạng thái đồng bộ của AQS — State. Trong AQS bảo trì một field tên là state, có nghĩa là trạng thái đồng bộ, do Volatile sửa đổi, dùng để hiển thị tình hình lấy lock của tài nguyên găng hiện tại.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private volatile int state;
```

Dưới đây cung cấp một vài phương thức truy cập field này:

| Tên phương thức | Mô tả |
| :--- | :--- |
| protected final int getState() | Lấy giá trị của State |
| protected final void setState(int newState) | Thiết lập giá trị của State |
| protected final boolean compareAndSetState(int expect, int update) | Sử dụng cách thức CAS để cập nhật State |

Mấy phương thức này đều do Final sửa đổi, thể hiện trong lớp con không thể override chúng. Chúng ta có thể thông qua sửa đổi trạng thái đồng bộ do field State biểu thị để triển khai Exclusive mode và Shared mode của đa luồng (quá trình cài khóa).

![](https://p0.meituan.net/travelcube/27605d483e8935da683a93be015713f331378.png)

![](https://p0.meituan.net/travelcube/3f1e1a44f5b7d77000ba4f9476189b2e32806.png)

Đối với công cụ đồng bộ tự định nghĩa của chúng ta, cần tự định nghĩa phương thức lấy trạng thái đồng bộ và giải phóng trạng thái, tức là tầng thứ nhất trong sơ đồ kiến trúc AQS: Tầng API.

### 2.2 Sự liên kết giữa các phương thức quan trọng của AQS và ReentrantLock

Từ sơ đồ kiến trúc có thể biết, AQS cung cấp lượng lớn phương thức Protected dùng cho triển khai bộ đồng bộ hóa tự định nghĩa. Các phương thức liên quan triển khai bộ đồng bộ hóa tự định nghĩa cũng chỉ là vì thông qua việc sửa đổi field State để triển khai Exclusive mode hoặc Shared mode của đa luồng. Bộ đồng bộ hóa tự định nghĩa cần triển khai các phương thức dưới đây (các phương thức ReentrantLock cần triển khai như sau, không phải toàn bộ):

| Tên phương thức | Mô tả |
| :--- | :--- |
| protected boolean isHeldExclusively() | Luồng này có đang độc chiếm tài nguyên không. Chỉ khi dùng Condition mới cần triển khai nó. |
| protected boolean tryAcquire(int arg) | Cách thức Exclusive. arg là số lần lấy lock, thử lấy tài nguyên, thành công trả về True, thất bại trả về False. |
| protected boolean tryRelease(int arg) | Cách thức Exclusive. arg là số lần giải phóng lock, thử giải phóng tài nguyên, thành công trả về True, thất bại trả về False. |
| protected int tryAcquireShared(int arg) | Cách thức Shared. arg là số lần lấy lock, thử lấy tài nguyên. Số âm biểu thị thất bại; 0 biểu thị thành công nhưng không còn tài nguyên khả dụng; số dương biểu thị thành công và còn tài nguyên. |
| protected boolean tryReleaseShared(int arg) | Cách thức Shared. arg là số lần giải phóng lock, thử giải phóng tài nguyên, nếu sau khi giải phóng cho phép đánh thức node chờ phía sau thì trả về True, ngược lại trả về False. |

Nói chung, bộ đồng bộ hóa tự định nghĩa hoặc là cách thức Exclusive, hoặc là cách thức Shared, họ cũng chỉ cần triển khai một trong hai cặp tryAcquire-tryRelease, tryAcquireShared-tryReleaseShared là được. AQS cũng hỗ trợ bộ đồng bộ hóa tự định nghĩa đồng thời triển khai cả hai cách thức Exclusive và Shared, như ReentrantReadWriteLock. ReentrantLock là Exclusive lock, nên đã triển khai tryAcquire-tryRelease.

Lấy Non-fair lock làm ví dụ, ở đây chủ yếu trình bày điểm liên kết giữa các phương thức của Non-fair lock và AQS, tác dụng của từng phương thức cốt lõi cụ thể sẽ được trình bày chi tiết ở phần sau của bài viết.

![](https://p1.meituan.net/travelcube/b8b53a70984668bc68653efe9531573e78636.png)

> 🐛 Sửa đổi (Tham khảo: [issue#1761](https://github.com/Snailclimb/JavaGuide/issues/1761)): Một lỗi nhỏ trong hình, (AQS) CAS sửa đổi tài nguyên dùng chung State thành công xong đáng lẽ là lấy lock thành công (Non-fair lock).
>
> Mã nguồn tương ứng như sau:
>
> ```java
> final boolean nonfairTryAcquire(int acquires) {
>          final Thread current = Thread.currentThread();// Lấy luồng hiện tại
>          int c = getState();
>          if (c == 0) {
>              if (compareAndSetState(0, acquires)) {// Cướp lock bằng CAS
>                  setExclusiveOwnerThread(current);// Thiết lập luồng hiện tại thành luồng độc chiếm
>                  return true;// Cướp lock thành công
>              }
>          }
>          else if (current == getExclusiveOwnerThread()) {
>              int nextc = c + acquires;
>              if (nextc < 0) // overflow
>                  throw new Error("Maximum lock count exceeded");
>              setState(nextc);
>              return true;
>          }
>          return false;
>      }
> ```

Để giúp mọi người hiểu quy trình tương tác phương thức giữa ReentrantLock và AQS, lấy Non-fair lock làm ví dụ, chúng ta lấy riêng quy trình tương tác của việc cài khóa và mở khóa ra để nhấn mạnh một chút, nhằm thuận tiện cho việc hiểu các nội dung sau.

![](https://p1.meituan.net/travelcube/7aadb272069d871bdee8bf3a218eed8136919.png)

Cài khóa:

- Thông qua phương thức cài khóa Lock của ReentrantLock để tiến hành thao tác cài khóa.
- Sẽ gọi đến phương thức Lock của inner class Sync, do Sync#lock là phương thức trừu tượng, dựa theo Fair lock và Non-fair lock được chọn khi khởi tạo ReentrantLock, thực thi phương thức Lock của inner class liên quan, về bản chất đều sẽ thực thi phương thức Acquire của AQS.
- Phương thức Acquire của AQS sẽ thực thi phương thức tryAcquire, nhưng do tryAcquire cần bộ đồng bộ hóa tự định nghĩa triển khai, do đó đã thực thi phương thức tryAcquire trong ReentrantLock, do ReentrantLock triển khai phương thức tryAcquire thông qua các inner class Fair lock và Non-fair lock, do đó sẽ dựa theo loại lock khác nhau, thực thi tryAcquire khác nhau.
- tryAcquire là logic lấy lock, sau khi lấy thất bại, sẽ thực thi logic tiếp theo của framework AQS, không liên quan đến bộ đồng bộ hóa tự định nghĩa ReentrantLock.

Mở khóa:

- Thông qua phương thức mở khóa Unlock của ReentrantLock để tiến hành mở khóa.
- Unlock sẽ gọi phương thức Release của inner class Sync, phương thức đó kế thừa từ AQS.
- Trong Release sẽ gọi phương thức tryRelease, tryRelease cần bộ đồng bộ hóa tự định nghĩa triển khai, tryRelease chỉ được triển khai trong Sync của ReentrantLock, do đó có thể thấy, quá trình giải phóng lock không phân biệt có phải là Fair lock hay không.
- Sau khi giải phóng thành công, tất cả việc xử lý do framework AQS hoàn thành, không liên quan đến bộ đồng bộ hóa tự định nghĩa.

Thông qua mô tả ở trên, có thể tổng kết đại khái mối quan hệ ánh xạ của các phương thức cốt lõi tầng API khi ReentrantLock cài khóa và mở khóa.

![](https://p0.meituan.net/travelcube/f30c631c8ebbf820d3e8fcb6eee3c0ef18748.png)

## 3 Thông qua ReentrantLock hiểu AQS

Fair lock và Non-fair lock trong ReentrantLock ở tầng dưới cùng là giống nhau, ở đây lấy Non-fair lock làm ví dụ để phân tích.

Trong Non-fair lock, có một đoạn code như thế này:

```java
// java.util.concurrent.locks.ReentrantLock

static final class NonfairSync extends Sync {
  ...
  final void lock() {
    if (compareAndSetState(0, 1))
      setExclusiveOwnerThread(Thread.currentThread());
    else
      acquire(1);
  }
  ...
}
```

Xem thử Acquire này được viết như thế nào:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final void acquire(int arg) {
  if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

Xem tiếp phương thức tryAcquire:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

protected boolean tryAcquire(int arg) {
  throw new UnsupportedOperationException();
}
```

Có thể thấy, ở đây chỉ là triển khai đơn giản của AQS, phương thức triển khai lấy lock cụ thể do các Fair lock và Non-fair lock riêng biệt triển khai (lấy ReentrantLock làm ví dụ). Nếu phương thức này trả về True, thì chứng tỏ luồng hiện tại lấy lock thành công, không cần thực thi về sau nữa; nếu lấy thất bại, thì cần gia nhập vào hàng đợi chờ. Dưới đây sẽ giải thích chi tiết khi nào và làm sao luồng được gia nhập vào hàng đợi chờ.

### 3.1 Luồng gia nhập hàng đợi chờ

#### 3.1.1 Thời điểm gia nhập hàng đợi

Khi thực thi Acquire(1), sẽ thông qua tryAcquire để lấy lock. Trong trường hợp này, nếu lấy lock thất bại, sẽ gọi addWaiter gia nhập vào hàng đợi chờ.

#### 3.1.2 Gia nhập hàng đợi như thế nào

Sau khi lấy lock thất bại, sẽ thực thi addWaiter(Node.EXCLUSIVE) gia nhập hàng đợi chờ, phương thức triển khai cụ thể như sau:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode);
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail;
  if (pred != null) {
    node.prev = pred;
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node;
    }
  }
  enq(node);
  return node;
}
private final boolean compareAndSetTail(Node expect, Node update) {
  return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
```

Quy trình chính như sau:

- Thông qua luồng hiện tại và lock mode tạo mới một node.
- Con trỏ Pred trỏ tới node đuôi Tail.
- Trỏ con trỏ Prev của Node mới tạo tới Pred.
- Thông qua phương thức compareAndSetTail, hoàn thành việc thiết lập node đuôi. Phương thức này chủ yếu tiến hành so sánh đối với tailOffset và Expect, nếu Node của tailOffset và Node của Expect có địa chỉ giống nhau, vậy thì thiết lập giá trị của Tail thành giá trị của Update.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

static {
  try {
    stateOffset = unsafe.objectFieldOffset(AbstractQueuedSynchronizer.class.getDeclaredField("state"));
    headOffset = unsafe.objectFieldOffset(AbstractQueuedSynchronizer.class.getDeclaredField("head"));
    tailOffset = unsafe.objectFieldOffset(AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
    waitStatusOffset = unsafe.objectFieldOffset(Node.class.getDeclaredField("waitStatus"));
    nextOffset = unsafe.objectFieldOffset(Node.class.getDeclaredField("next"));
  } catch (Exception ex) {
    throw new Error(ex);
  }
}
```

Từ khối code static của AQS có thể thấy, đều là lấy offset của một thuộc tính đối tượng so với đối tượng đó trong bộ nhớ, như vậy chúng ta có thể dựa vào offset này để tìm thấy thuộc tính này trong bộ nhớ đối tượng. tailOffset chỉ offset tương ứng của tail, nên lúc này sẽ đặt Node new ra thành node đuôi của hàng đợi hiện tại. Đồng thời, do là danh sách liên kết hai chiều, cũng cần trỏ node phía trước tới node đuôi.

- Nếu con trỏ Pred là Null (chứng tỏ trong hàng đợi chờ không có phần tử), hoặc con trỏ Pred hiện tại và vị trí mà Tail trỏ tới không giống nhau (chứng tỏ đã bị luồng khác sửa đổi), thì cần xem một chút phương thức Enq.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private Node enq(final Node node) {
  for (;;) {
    Node t = tail;
    if (t == null) { // Must initialize
      if (compareAndSetHead(new Node()))
        tail = head;
    } else {
      node.prev = t;
      if (compareAndSetTail(t, node)) {
        t.next = node;
        return t;
      }
    }
  }
}
```

Nếu chưa được khởi tạo, cần tiến hành khởi tạo ra một node head. Nhưng xin chú ý, node head được khởi tạo không phải là node luồng hiện tại, mà là node đã gọi constructor không tham số. Nếu đã trải qua khởi tạo hoặc concurrency dẫn đến trong hàng đợi có phần tử, thì giống với phương thức trước đó. Thực ra, addWaiter chính là một thao tác thêm node đuôi trên danh sách liên kết hai đầu, điểm cần lưu ý là, node head của danh sách liên kết hai đầu là node head của constructor không tham số.

Tóm lại, khi luồng lấy lock, quá trình đại thể như sau:

1. Khi không có luồng nào lấy được lock, Luồng 1 lấy lock thành công.

2. Luồng 2 xin lấy lock, nhưng lock bị Luồng 1 chiếm giữ.

![img](https://p0.meituan.net/travelcube/e9e385c3c68f62c67c8d62ab0adb613921117.png)

3. Nếu lại có luồng muốn lấy lock, lần lượt xếp hàng phía sau trong hàng đợi là được.

Trở lại đoạn code phía trên, hasQueuedPredecessors là phương thức phán đoán trong hàng đợi chờ có tồn tại node có hiệu lực không khi Fair lock cài khóa. Nếu trả về False, chứng tỏ luồng hiện tại có thể tranh chấp tài nguyên dùng chung; nếu trả về True, chứng tỏ trong hàng đợi tồn tại node có hiệu lực, luồng hiện tại bắt buộc phải gia nhập vào hàng đợi chờ.

```java
// java.util.concurrent.locks.ReentrantLock

public final boolean hasQueuedPredecessors() {
  // The correctness of this depends on head being initialized
  // before tail and on head.next being accurate if the current
  // thread is first in queue.
  Node t = tail; // Read fields in reverse initialization order
  Node h = head;
  Node s;
  return h != t && ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

Nhìn đến đây, chúng ta hiểu một chút h != t && ((s = h.next) == null || s.thread != Thread.currentThread()); Tại sao lại phải phán đoán node tiếp theo của node head? Dữ liệu mà node đầu tiên lưu trữ là gì?

> Trong danh sách liên kết hai chiều, node đầu tiên là node ảo, thực ra không lưu trữ bất kỳ thông tin nào, chỉ là chiếm chỗ. Node có dữ liệu thực sự đầu tiên là bắt đầu từ node thứ hai. Khi h != t: Nếu (s = h.next) == null, hàng đợi chờ đang có luồng tiến hành khởi tạo, nhưng chỉ mới tiến hành đến Tail trỏ tới Head, chưa đưa Head trỏ tới Tail, lúc này trong hàng đợi có phần tử, cần trả về True (phần này xem cụ thể phân tích code phía dưới). Nếu (s = h.next) != null, chứng tỏ lúc này trong hàng đợi có ít nhất một node có hiệu lực. Nếu lúc này s.thread == Thread.currentThread(), chứng tỏ luồng trong node có hiệu lực đầu tiên của hàng đợi chờ giống với luồng hiện tại, vậy luồng hiện tại là có thể lấy tài nguyên; nếu s.thread != Thread.currentThread(), chứng tỏ luồng trong node có hiệu lực đầu tiên của hàng đợi chờ khác với luồng hiện tại, luồng hiện tại bắt buộc phải gia nhập vào hàng đợi chờ.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer#enq

if (t == null) { // Must initialize
  if (compareAndSetHead(new Node()))
    tail = head;
} else {
  node.prev = t;
  if (compareAndSetTail(t, node)) {
    t.next = node;
    return t;
  }
}
```

Node vào hàng đợi không phải là thao tác atomic, nên sẽ xuất hiện head != tail ngắn ngủi, lúc này Tail trỏ tới node cuối cùng, và Tail trỏ tới Head. Nếu Head chưa trỏ tới Tail (xem dòng 5, 6, 7), trong trường hợp này cũng cần đưa luồng liên quan gia nhập hàng đợi. Nên đoạn code này là để giải quyết vấn đề concurrency trong trường hợp cực đoan.

#### 3.1.3 Thời điểm luồng trong hàng đợi chờ ra khỏi hàng đợi

Trở lại mã nguồn ban đầu:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final void acquire(int arg) {
  if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

Phần trên đã giải thích phương thức addWaiter, phương thức này thực ra chính là đưa luồng tương ứng dưới dạng cấu trúc dữ liệu Node gia nhập vào danh sách liên kết hai đầu, trả về là một Node chứa luồng đó. Mà Node này sẽ làm tham số đi vào phương thức acquireQueued. Phương thức acquireQueued có thể tiến hành thao tác "lấy lock" đối với luồng đang xếp hàng.

Tổng thể mà nói, một luồng sau khi lấy lock thất bại sẽ được đặt vào hàng đợi chờ, `acquireQueued` sẽ làm cho nó liên tục chờ đợi và thử lấy lock, cho đến khi lấy thành công. `acquire(int)` là cách thức lấy không thể ngắt: Trong thời gian chờ đợi khi xảy ra ngắt sẽ ghi lại trạng thái ngắt trước, sau khi lấy lock thành công mới khôi phục thông qua `selfInterrupt()`, chứ không vì thế mà hủy lần lấy này.

Dưới đây chúng ta phân tích mã nguồn acquireQueued từ hai hướng "Khi nào ra khỏi hàng đợi?" và "Ra khỏi hàng đợi như thế nào?":

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
  // Đánh dấu xem có lấy tài nguyên thành công không
  boolean failed = true;
  try {
    // Đánh dấu trong quá trình chờ đợi có bị ngắt không
    boolean interrupted = false;
    // Bắt đầu tự xoay, hoặc là lấy lock, hoặc là ngắt
    for (;;) {
      // Lấy node tiền nhiệm của node hiện tại
      final Node p = node.predecessor();
      // Nếu p là node head, chứng tỏ node hiện tại ở đầu hàng đợi dữ liệu thực sự, liền thử lấy lock (đừng quên node head là node ảo)
      if (p == head && tryAcquire(arg)) {
        // Lấy lock thành công, con trỏ head di chuyển đến node hiện tại
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      // Thể hiện p là node head và hiện tại chưa lấy được lock (có thể do Non-fair lock bị cướp mất) hoặc là p không phải node head, lúc này phải phán đoán node hiện tại có cần bị làm bị chặn không (Điều kiện bị làm bị chặn: waitStatus của node tiền nhiệm là -1), phòng ngừa vòng lặp vô hạn lãng phí tài nguyên. Cụ thể hai phương thức phân tích kỹ phía dưới
      if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
        interrupted = true;
    }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}
```

Ghi chú: Phương thức setHead là đặt node hiện tại thành node ảo, nhưng không sửa waitStatus, vì nó là dữ liệu liên tục cần dùng.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void setHead(Node node) {
  head = node;
  node.thread = null;
  node.prev = null;
}

// java.util.concurrent.locks.AbstractQueuedSynchronizer

// Dựa vào node tiền nhiệm phán đoán luồng hiện tại có nên bị làm bị chặn không
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  // Lấy trạng thái node của node tiền nhiệm
  int ws = pred.waitStatus;
  // Chứng tỏ node tiền nhiệm ở trạng thái được đánh thức
  if (ws == Node.SIGNAL)
    return true;
  // Thông qua giá trị enum chúng ta biết waitStatus > 0 là trạng thái hủy
  if (ws > 0) {
    do {
      // Vòng lặp tìm kiếm ngược về trước node hủy, loại node hủy khỏi hàng đợi
      node.prev = pred = pred.prev;
    } while (pred.waitStatus > 0);
    pred.next = node;
  } else {
    // Thiết lập trạng thái chờ của node tiền nhiệm thành SIGNAL
    compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
  }
  return false;
}
```

parkAndCheckInterrupt chủ yếu dùng để treo luồng hiện tại, làm bị chặn call stack, trả về trạng thái ngắt của luồng hiện tại.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

Sơ đồ quy trình các phương thức trên như sau:

![](https://p0.meituan.net/travelcube/c124b76dcbefb9bdc778458064703d1135485.png)

Từ hình trên có thể thấy, điều kiện nhảy khỏi vòng lặp hiện tại là khi "node phía trước là node head, và luồng hiện tại lấy lock thành công". Để phòng ngừa do vòng lặp vô hạn dẫn đến tài nguyên CPU bị lãng phí, chúng ta sẽ phán đoán trạng thái của node phía trước để quyết định có treo luồng hiện tại hay không, quy trình treo cụ thể dùng sơ đồ quy trình biểu thị như sau (quy trình shouldParkAfterFailedAcquire):

![](https://p0.meituan.net/travelcube/9af16e2481ad85f38ca322a225ae737535740.png)

Nghi vấn về việc giải phóng node khỏi hàng đợi đã được xua tan, vậy lại có câu hỏi mới:

- Node hủy trong shouldParkAfterFailedAcquire được sinh ra như thế nào? Khi nào thì đặt waitStatus của một node thành -1?
- Vào thời gian nào thì giải phóng node thông báo đến luồng bị treo?

### 3.2 Sinh ra node trạng thái CANCELLED

Code Finally trong phương thức acquireQueued:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true;
  try {
    ...
    for (;;) {
      final Node p = node.predecessor();
      if (p == head && tryAcquire(arg)) {
        ...
        failed = false;
        ...
      }
      ...
  } finally {
    if (failed)
      cancelAcquire(node);
    }
}
```

Thông qua phương thức cancelAcquire, đánh dấu trạng thái của Node thành CANCELLED. Tiếp theo, chúng ta phân tích từng dòng về nguyên lý phương thức này:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void cancelAcquire(Node node) {
  // Lọc node không có hiệu lực
  if (node == null)
    return;
  // Thiết lập node đó không liên kết với bất kỳ luồng nào, tức node ảo
  node.thread = null;
  Node pred = node.prev;
  // Thông qua node tiền nhiệm, bỏ qua node trạng thái hủy
  while (pred.waitStatus > 0)
    node.prev = pred = pred.prev;
  // Lấy node kế nhiệm của node tiền nhiệm sau khi đã lọc
  Node predNext = pred.next;
  // Đặt trạng thái của node hiện tại thành CANCELLED
  node.waitStatus = Node.CANCELLED;
  // Nếu node hiện tại là node đuôi, thiết lập node không phải trạng thái hủy đầu tiên từ sau ra trước thành node đuôi
  // Nếu cập nhật thất bại, thì đi vào else, nếu cập nhật thành công, thiết lập node kế nhiệm của tail thành null
  if (node == tail && compareAndSetTail(node, pred)) {
    compareAndSetNext(pred, predNext, null);
  } else {
    int ws;
    // Nếu node hiện tại không phải node kế nhiệm của head, 1: Phán đoán node tiền nhiệm của node hiện tại có phải là SIGNAL không, 2: Nếu không phải, thì đặt node tiền nhiệm thành SIGNAL xem có thành công không
    // Nếu 1 và 2 có một cái là true, lại phán đoán luồng của node hiện tại có phải là null không
    // Nếu các điều kiện trên đều thỏa mãn, đưa con trỏ kế nhiệm của node tiền nhiệm của node hiện tại trỏ tới node kế nhiệm của node hiện tại
    if (pred != head && ((ws = pred.waitStatus) == Node.SIGNAL || (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) && pred.thread != null) {
      Node next = node.next;
      if (next != null && next.waitStatus <= 0)
        compareAndSetNext(pred, predNext, next);
    } else {
      // Nếu node hiện tại là node kế nhiệm của head, hoặc các điều kiện trên không thỏa mãn, vậy thì đánh thức node kế nhiệm của node hiện tại
      unparkSuccessor(node);
    }
    node.next = node; // help GC
  }
}
```

Quy trình hiện tại:

- Lấy node tiền nhiệm của node hiện tại, nếu trạng thái node tiền nhiệm là CANCELLED, thì liên tục duyệt về trước, tìm node đầu tiên waitStatus <= 0, liên kết node Pred tìm được với Node hiện tại, đặt Node hiện tại thành CANCELLED.
- Dựa theo vị trí của node hiện tại, cân nhắc ba trường hợp dưới đây:

(1) Node hiện tại là node đuôi.

(2) Node hiện tại là node kế nhiệm của Head.

(3) Node hiện tại không phải node kế nhiệm của Head, cũng không phải node đuôi.

Dựa theo điều thứ hai ở trên, chúng ta phân tích quy trình của từng trường hợp.

Node hiện tại là node đuôi.

![](https://p1.meituan.net/travelcube/b845211ced57561c24f79d56194949e822049.png)

Node hiện提 là node kế nhiệm của Head.

![](https://p1.meituan.net/travelcube/ab89bfec875846e5028a4f8fead32b7117975.png)

Node hiện tại không phải node kế nhiệm của Head, cũng không phải node đuôi.

![](https://p0.meituan.net/travelcube/45d0d9e4a6897eddadc4397cf53d6cd522452.png)

Thông qua quy trình ở trên, chúng ta đã có hiểu biết đại thể đối với việc sinh ra và thay đổi trạng thái của node CANCELLED, nhưng tại sao tất cả sự thay đổi đều tiến hành thao tác đối với con trỏ Next, mà không tiến hành thao tác đối với con trỏ Prev? Trong trường hợp nào thì tiến hành thao tác đối với con trỏ Prev?

> Khi thực thi cancelAcquire, node tiền nhiệm của node hiện tại có thể đã ra khỏi hàng đợi rồi (đã thực thi phương thức shouldParkAfterFailedAcquire trong khối code Try rồi), nếu lúc này sửa con trỏ Prev, có khả năng dẫn đến Prev trỏ tới một Node khác đã xóa khỏi hàng đợi, do đó việc thay đổi con trỏ Prev ở đây là không an toàn. Trong phương thức shouldParkAfterFailedAcquire, sẽ thực thi đoạn code dưới đây, thực ra chính là đang xử lý con trỏ Prev. shouldParkAfterFailedAcquire chỉ được thực thi trong trường hợp lấy lock thất bại, sau khi đi vào phương thức này, chứng tỏ tài nguyên dùng chung đã được lấy, các node trước node hiện tại đều sẽ không xuất hiện sự thay đổi, do đó lúc này thay đổi con trỏ Prev tương đối an toàn.
>
> ```java
> do {
>   node.prev = pred = pred.prev;
> } while (pred.waitStatus > 0);
> ```

### 3.3 Mở khóa như thế nào

Chúng ta đã bóc tách quy trình cơ bản trong quá trình cài khóa, tiếp theo lại phân tích quy trình cơ bản của việc mở khóa. Do ReentrantLock khi mở khóa, không phân biệt Fair lock và Non-fair lock, nên chúng ta trực tiếp xem mã nguồn mở khóa:

```java
// java.util.concurrent.locks.ReentrantLock

public void unlock() {
  sync.release(1);
}
```

Có thể thấy, nơi giải phóng lock về bản chất là do framework hoàn thành.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final boolean release(int arg) {
  if (tryRelease(arg)) {
    Node h = head;
    if (h != null && h.waitStatus != 0)
      unparkSuccessor(h);
    return true;
  }
  return false;
}
```

Trong Sync - lớp cha của Fair lock và Non-fair lock trong ReentrantLock đã định nghĩa cơ chế giải phóng lock của ReentrantLock.

```java
// java.util.concurrent.locks.ReentrantLock.Sync

// Phương thức trả về lock hiện tại có phải chưa được luồng nào nắm giữ
protected final boolean tryRelease(int releases) {
  // Giảm số lần reentrant
  int c = getState() - releases;
  // Luồng hiện tại không phải luồng nắm giữ lock, ném ra ngoại lệ
  if (Thread.currentThread() != getExclusiveOwnerThread())
    throw new IllegalMonitorStateException();
  boolean free = false;
  // Nếu luồng nắm giữ giải phóng toàn bộ, thiết lập tất cả các luồng của lock độc chiếm hiện tại thành null, và cập nhật state
  if (c == 0) {
    free = true;
    setExclusiveOwnerThread(null);
  }
  setState(c);
  return free;
}
```

Chúng ta giải thích mã nguồn dưới đây:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final boolean release(int arg) {
  // Nếu tryRelease tự định nghĩa ở trên trả về true, chứng tỏ lock đó không bị bất kỳ luồng nào nắm giữ
  if (tryRelease(arg)) {
    // Lấy node head
    Node h = head;
    // Node head không rỗng và waitStatus của node head không phải trường hợp node khởi tạo, giải trừ trạng thái treo luồng
    if (h != null && h.waitStatus != 0)
      unparkSuccessor(h);
    return true;
  }
  return false;
}
```

Điều kiện phán đoán ở đây tại sao lại là h != null && h.waitStatus != 0?

> h == null Head chưa khởi tạo. Trong trường hợp ban đầu, head == null, node đầu tiên vào hàng đợi, Head sẽ được khởi tạo một node ảo. Nên nói là, ở đây nếu chưa kịp vào hàng đợi, sẽ xuất hiện trường hợp head == null.
>
> h != null && waitStatus == 0 thể hiện luồng tương ứng của node kế nhiệm vẫn đang chạy, không cần đánh thức.
>
> h != null && waitStatus < 0 thể hiện node kế nhiệm có thể bị làm bị chặn, cần đánh thức.

Xem tiếp phương thức unparkSuccessor:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void unparkSuccessor(Node node) {
  // Lấy waitStatus của node head
  int ws = node.waitStatus;
  if (ws < 0)
    compareAndSetWaitStatus(node, ws, 0);
  // Lấy node tiếp theo của node hiện tại
  Node s = node.next;
  // Nếu node tiếp theo là null hoặc node tiếp theo bị cancelled, thì tìm node không cancelled đầu tiên của hàng đợi
  if (s == null || s.waitStatus > 0) {
    s = null;
    // Liền tìm từ node đuôi về trước, đến đầu hàng đợi, tìm node waitStatus < 0 đầu tiên của hàng đợi.
    for (Node t = tail; t != null && t != node; t = t.prev)
      if (t.waitStatus <= 0)
        s = t;
  }
  // Nếu node tiếp theo của node hiện tại không rỗng, và trạng thái <= 0, liền unpark node hiện tại
  if (s != null)
    LockSupport.unpark(s.thread);
}
```

Tại sao lại phải từ sau ra trước tìm node không Cancelled đầu tiên? Nguyên nhân như sau.

Phương thức addWaiter trước đó:

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private Node addWaiter(Node mode) {
  Node node = new Node(Thread.currentThread(), mode);
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail;
  if (pred != null) {
    node.prev = pred;
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node;
    }
  }
  enq(node);
  return node;
}
```

Chúng ta có thể thấy từ đây, node vào hàng đợi không phải là thao tác atomic, nói cách khác, hai chỗ node.prev = pred; compareAndSetTail(pred, node) có thể xem là thao tác atomic Tail vào hàng đợi, nhưng lúc này pred.next = node; chưa thực thi, nếu lúc này thực thi phương thức unparkSuccessor, thì không có cách nào tìm từ trước ra sau nữa, do đó cần tìm từ sau ra trước. Còn có một điểm nguyên nhân, khi sinh ra node trạng thái CANCELLED, ngắt kết nối trước là con trỏ Next, con trỏ Prev chưa ngắt kết nối, do đó cũng là bắt buộc phải duyệt từ sau ra trước mới có thể duyệt hoàn toàn tất cả các Node.

Tóm lại, nếu là tìm từ trước ra sau, do thao tác non-atomic khi vào hàng đợi và thao tác ngắt con trỏ Next trong quá trình sinh ra node CANCELLED ở trường hợp cực đoan, có thể dẫn đến không thể duyệt tất cả các node. Nên, sau khi đánh thức luồng tương ứng, luồng tương ứng sẽ tiếp tục thực thi xuống dưới. Sau khi tiếp tục thực thi phương thức acquireQueued, việc ngắt được xử lý như thế nào?

### 3.4 Quy trình thực thi sau khi khôi phục từ ngắt

Sau khi đánh thức, sẽ thực thi return Thread.interrupted();, hàm này trả về là trạng thái ngắt của luồng đang thực thi hiện tại, và xóa đi.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this);
  return Thread.interrupted();
}
```

Trở lại code acquireQueued, khi parkAndCheckInterrupt trả về True hoặc False, giá trị của interrupted là khác nhau, nhưng đều sẽ thực thi vòng lặp lần tới. Nếu lúc này lấy lock thành công, sẽ trả về interrupted hiện tại.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true;
  try {
    boolean interrupted = false;
    for (;;) {
      final Node p = node.predecessor();
      if (p == head && tryAcquire(arg)) {
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
        interrupted = true;
      }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}
```

Nếu acquireQueued là True, sẽ thực thi phương thức selfInterrupt.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

static void selfInterrupt() {
  Thread.currentThread().interrupt();
}
```

Phương thức đó thực ra là để ngắt luồng. Nhưng tại sao sau khi lấy được lock lại còn phải ngắt luồng? Phần này thuộc nội dung kiến thức ngắt dạng phối hợp do Java cung cấp, các bạn quan tâm có thể tra cứu thêm. Ở đây giới thiệu đơn giản:

1. Khi luồng ngắt được đánh thức, không biết nguyên nhân được đánh thức, có thể luồng hiện tại bị ngắt trong lúc chờ đợi, cũng có thể được đánh thức sau khi giải phóng lock. Do đó chúng ta thông qua phương thức Thread.interrupted() kiểm tra nhãn ngắt (phương thức đó trả về trạng thái ngắt của luồng hiện tại, và thiết lập nhãn ngắt của luồng hiện tại thành False), và ghi lại, nếu phát hiện luồng đó từng bị ngắt, thì ngắt lại lần nữa.
2. Luồng trong quá trình chờ tài nguyên được đánh thức, sau khi đánh thức vẫn sẽ không ngừng đi thử lấy lock, cho đến khi cướp được lock mới thôi. Nói cách khác, trong toàn bộ quy trình, không phản hồi ngắt, chỉ ghi lại bản ghi ngắt. Cuối cùng cướp được lock trả về rồi, vậy nếu từng bị ngắt, thì cần bổ sung một lần ngắt.

Cách xử lý ở đây chủ yếu vận dụng runWorker trong Worker - đơn vị vận hành cơ bản trong ThreadPool, thông qua Thread.interrupted() tiến hành xử lý phán đoán bổ sung, các bạn quan tâm có thể xem mã nguồn ThreadPoolExecutor.

### 3.5 Tóm tắt nhỏ

Chúng ta đã đưa ra một số câu hỏi trong mục 1.3, hiện tại đến trả lời một chút.

> Q: Quy trình tiếp theo khi một luồng nào đó lấy lock thất bại là gì?
>
> A: Tồn tại một cơ chế xếp hàng chờ đợi nào đó, luồng tiếp tục chờ đợi, vẫn giữ lại khả năng lấy lock, quy trình lấy lock vẫn đang tiếp tục.
>
> Q: Đã nói đến cơ chế xếp hàng chờ đợi, vậy thì nhất định sẽ có một hàng đợi nào đó hình thành, hàng đợi như vậy là cấu trúc dữ liệu gì?
>
> A: Là hàng đợi hai đầu FIFO biến thể CLH.
>
> Q: Luồng nằm trong cơ chế xếp hàng chờ đợi, khi nào thì có cơ hội lấy lock?
>
> A: Có thể xem chi tiết mục 2.3.1.3.
>
> Q: Nếu luồng nằm trong cơ chế xếp hàng chờ đợi liên tục không thể lấy lock, cần liên tục chờ đợi sao? Hay có chiến lược khác để giải quyết vấn đề này?
>
> A: Trạng thái node mà luồng đang ở sẽ biến thành trạng thái hủy, node trạng thái hủy sẽ được giải phóng khỏi hàng đợi, cụ thể xem mục 2.3.2.
>
> Q: Hàm Lock thông qua phương thức Acquire để tiến hành cài khóa, nhưng cụ thể cài khóa như thế nào?
>
> A: Acquire của AQS sẽ gọi phương thức tryAcquire, tryAcquire do các bộ đồng bộ hóa tự định nghĩa triển khai, hoàn thành quá trình cài khóa thông qua tryAcquire.

## 4 Ứng dụng AQS

### 4.1 Ứng dụng reentrant của ReentrantLock

Tính reentrant của ReentrantLock là một trong những ứng dụng rất tốt của AQS, sau khi tìm hiểu xong các kiến thức ở trên, chúng ta rất dễ biết được cách triển khai reentrant của ReentrantLock. Trong ReentrantLock, bất kể là Fair lock hay Non-fair lock, đều có một đoạn logic.

Fair lock:

```java
// java.util.concurrent.locks.ReentrantLock.FairSync#tryAcquire

if (c == 0) {
  if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
    setExclusiveOwnerThread(current);
    return true;
  }
}
else if (current == getExclusiveOwnerThread()) {
  int nextc = c + acquires;
  if (nextc < 0)
    throw new Error("Maximum lock count exceeded");
  setState(nextc);
  return true;
}
```

Non-fair lock:

```java
// java.util.concurrent.locks.ReentrantLock.Sync#nonfairTryAcquire

if (c == 0) {
  if (compareAndSetState(0, acquires)){
    setExclusiveOwnerThread(current);
    return true;
  }
}
else if (current == getExclusiveOwnerThread()) {
  int nextc = c + acquires;
  if (nextc < 0) // overflow
    throw new Error("Maximum lock count exceeded");
  setState(nextc);
  return true;
}
```

Từ hai đoạn trên đều có thể thấy, có một trạng thái đồng bộ State để kiểm soát tình hình reentrant tổng thể. State được Volatile sửa đổi, dùng để đảm bảo tính nhìn thấy và tính có thứ tự nhất định.

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private volatile int state;
```

Tiếp theo xem quá trình chính của field State này:

1. Khi State khởi tạo là 0, biểu thị không có bất kỳ luồng nào nắm giữ lock.
2. Khi có luồng nắm giữ lock đó, giá trị sẽ +1 trên cơ sở ban đầu, cùng một luồng nhiều lần có được lock, sẽ nhiều lần +1, ở đây chính là khái niệm reentrant.
3. Mở khóa cũng là -1 đối với field này, cho đến khi về 0, luồng này mới giải phóng lock.

### 4.2 Kịch bản ứng dụng trong JUC

Ngoài ứng dụng tính reentrant của ReentrantLock ở trên, AQS với vai trò là framework lập trình concurrency, đã cung cấp phương án giải quyết tốt cho rất nhiều công cụ đồng bộ khác. Dưới đây liệt kê một vài công cụ đồng bộ trong JUC, giới thiệu đại thể kịch bản ứng dụng của AQS:

| Công cụ đồng bộ | Sự liên kết giữa công cụ đồng bộ và AQS |
| :--- | :--- |
| ReentrantLock | Sử dụng AQS lưu trữ số lần lock được nắm giữ lặp lại. Khi một luồng lấy lock, ReentrantLock ghi lại nhãn luồng hiện tại nhận được lock, dùng để kiểm tra có lấy lặp lại không, cũng như xử lý trường hợp bất thường khi luồng lỗi thử thao tác mở khóa. |
| Semaphore | Sử dụng trạng thái đồng bộ AQS để lưu trữ đếm hiện tại của tín hiệu số. tryRelease sẽ tăng đếm, acquireShared sẽ giảm đếm. |
| CountDownLatch | Sử dụng trạng thái đồng bộ AQS để biểu thị số đếm. Khi số đếm bằng 0, tất cả các thao tác Acquire (Phương thức await của CountDownLatch) mới có thể đi qua. |
| ReentrantReadWriteLock | Sử dụng 16 bit trong trạng thái đồng bộ AQS lưu trữ số lần Lock Ghi nắm giữ, 16 bit còn lại dùng để lưu trữ số lần Lock Đọc nắm giữ. |
| ThreadPoolExecutor | Worker tận dụng trạng thái đồng bộ AQS triển khai việc thiết lập đối với biến luồng độc chiếm (tryAcquire và tryRelease). |

### 4.3 Công cụ đồng bộ tự định nghĩa

Sau khi hiểu nguyên lý cơ bản của AQS, theo các điểm kiến thức AQS đã nói ở trên, tự mình triển khai một công cụ đồng bộ.

```java
public class LeeLock  {

    private static class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire (int arg) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease (int arg) {
            if (getState() == 0 || getExclusiveOwnerThread() != Thread.currentThread()) {
                throw new IllegalMonitorStateException();
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively () {
            return getState() == 1 && getExclusiveOwnerThread() == Thread.currentThread();
        }
    }

    private final Sync sync = new Sync();

    public void lock () {
        sync.acquire(1);
    }

    public void unlock () {
        sync.release(1);
    }
}
```

Thông qua Lock tự chúng ta định nghĩa để hoàn thành tính năng đồng bộ nhất định.

```java
public class LeeMain {

    static int count = 0;
    static LeeLock leeLock = new LeeLock();

    public static void main (String[] args) throws InterruptedException {

        Runnable runnable = new Runnable() {
            @Override
            public void run () {
                try {
                    leeLock.lock();
                    for (int i = 0; i < 10000; i++) {
                        count++;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    leeLock.unlock();
                }

            }
        };
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(count);
    }
}
```

Kết quả mỗi lần chạy code trên đều sẽ là 20000. Thông qua vài dòng code đơn giản đã có thể triển khai tính năng đồng bộ, đây chính là sự mạnh mẽ của AQS.

## 5 Tóm tắt

Kịch bản chúng ta sử dụng concurrency trong phát triển hàng ngày quá nhiều, nhưng người hiểu nguyên lý framework cơ bản bên trong concurrency lại không nhiều. Do giới hạn độ dài, bài viết này chỉ giới thiệu nguyên lý của khóa reentrant ReentrantLock và nguyên lý AQS, hy vọng có thể trở thành "gạch lót đường" cho mọi người hiểu các bộ đồng bộ hóa như AQS và ReentrantLock.

## Tài liệu tham khảo

- Lea D. The java. util. concurrent synchronizer framework\[J]. Science of Computer Programming, 2005, 58(3): 293-309.
- 《Thực chiến lập trình Java Concurrency》
- [Chuyện về "Lock" Java không thể không nói](https://tech.meituan.com/2018/11/15/java-lock.html)

<!-- @include: @article-footer.snippet.md -->
