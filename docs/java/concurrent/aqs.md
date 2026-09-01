---
title: AQS 详解
description: AQS抽象队列同步器深度解析：详解AQS核心原理、CLH队列结构、独占锁与共享锁实现、ReentrantLock/Semaphore等同步器应用、线程阻塞唤醒机制。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: AQS,AbstractQueuedSynchronizer,队列同步器,独占锁,共享锁,CLH队列,ReentrantLock实现原理
---

<!-- markdownlint-disable MD024 -->

## Giới thiệu AQS

Tên đầy đủ của AQS là `AbstractQueuedSynchronizer`, dịch ra có nghĩa là Bộ đồng bộ hóa hàng đợi trừu tượng. Lớp này nằm trong gói `java.util.concurrent.locks`.

![](https://oss.javaguide.cn/github/javaguide/AQS.png)

AQS chính là một lớp trừu tượng, chủ yếu dùng để xây dựng Lock và Synchronizer (Bộ đồng bộ hóa).

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
}
```

AQS cung cấp một số triển khai tính năng chung cho việc xây dựng Lock và Synchronizer. Do đó, sử dụng AQS có thể đơn giản và hiệu quả xây dựng ra lượng lớn bộ đồng bộ hóa áp dụng rộng rãi, như `ReentrantLock`, `Semaphore`, `ReentrantReadWriteLock`... `SynchronousQueue` mặc dù cũng sử dụng hàng đợi chờ và CAS để triển khai ghép cặp luồng, nhưng không dựa trên AQS.

## Nguyên lý AQS

> Giải thích: Phân tích mã nguồn và trạng thái node của AQS dưới đây chủ yếu dựa trên JDK 8. Triển khai bên trong của AQS về sau liên tục tiến hóa: Trong JDK 11 vẫn có thể thấy các field hoặc method liên quan như `waitStatus`, `addWaiter()`, `acquireQueued()` đề cập trong bài, còn các field node và triển khai vào hàng đợi, chờ đợi trong JDK 17 và phiên bản hiện tại đã có sự thay đổi khá lớn. Ý tưởng tổng thể xây dựng bộ đồng bộ hóa dựa trên trạng thái đồng bộ, hàng đợi chờ và template method là không thay đổi.

Khi được hỏi kiến thức concurrency trong phỏng vấn, đa số đều sẽ được hỏi "Xin hãy nói về hiểu biết của bạn đối với nguyên lý AQS". Dưới đây đưa ra một ví dụ cho mọi người tham khảo, phỏng vấn không phải là học vẹt thuộc lòng đề bài, mọi người nhất định phải thêm vào tư tưởng của bản thân, cho dù không thêm được tư tưởng của bản thân cũng phải đảm bảo bản thân có thể giảng ra một cách bình dân thông hiểu chứ không phải đọc thuộc lòng.

### Hiểu nhanh về AQS

Trước khi thực sự giải thích mã nguồn AQS, cần có một sự nhận thức ở tầng tổng thể đối với AQS. Ở đây trước tiên sẽ thông qua vài câu hỏi, từ tầng tổng thể nhận thức AQS, hiểu AQS nằm ở tầng nào trong toàn bộ Java concurrency, về sau trong quá trình học tập mã nguồn AQS mới có thể hiểu rõ hơn mối quan hệ giữa Synchronizer và AQS.

#### Tác dụng của AQS là gì?

AQS giải quyết vấn đề độ phức tạp của nhà phát triển khi triển khai bộ đồng bộ hóa. Nó cung cấp một framework chung, dùng để triển khai các loại bộ đồng bộ hóa khác nhau, ví dụ **ReentrantLock** (Khóa reentrant), **Semaphore** (Tín hiệu số) và **CountDownLatch** (Bộ đếm đếm ngược). Thông qua việc đóng gói cơ chế đồng bộ luồng bên dưới, AQS giấu đi logic quản lý luồng phức tạp, làm cho nhà phát triển chỉ cần tập trung vào logic đồng bộ cụ thể.

Nói đơn giản, AQS là một lớp trừu tượng, cung cấp **framework thực thi** chung cho các bộ đồng bộ hóa. Nó định nghĩa **quy trình chung về lấy và giải phóng tài nguyên**, còn logic lấy tài nguyên cụ thể thì do bộ đồng bộ hóa cụ thể triển khai thông qua việc override các template method. Do đó, có thể xem AQS như **"đế" nền tảng** của bộ đồng bộ hóa, còn bộ đồng bộ hóa chính là **"ứng dụng" cụ thể** được triển khai dựa trên AQS.

#### Tại sao AQS lại sử dụng biến thể của hàng đợi CLH lock?

CLH lock là một triển khai tối ưu hóa dựa trên **Spinlock (Khóa tự xoay)**.

Nói trước về vấn đề tồn tại của tự xoay lock: Tự xoay lock thông qua việc luồng không ngừng thực thi thao tác `compareAndSet` (gọi tắt là `CAS`) đối với một biến atomic để thử lấy lock. Trong kịch bản concurrency cao, nhiều luồng sẽ đồng thời tranh chấp cùng một biến atomic, dễ gây ra thao tác `CAS` của một luồng nào đó thất bại thời gian dài, từ đó dẫn đến **vấn đề "bỏ đói" (Starvation)** (một số luồng có thể không bao giờ lấy được lock).

CLH lock thông qua việc đưa vào một hàng đợi để tổ chức các luồng tranh chấp concurrency, tiến hành cải tiến đối với tự xoay lock:

- Mỗi luồng sẽ đóng vai trò một node gia nhập vào hàng đợi, và thông qua tự xoay giám sát trạng thái của node luồng phía trước, chứ không trực tiếp tranh chấp biến dùng chung.
- Các luồng xếp hàng theo thứ tự, đảm bảo tính công bằng, từ đó tránh được vấn đề "bỏ đói".

AQS (AbstractQueuedSynchronizer) trên cơ sở CLH lock tối ưu hóa thêm một bước nữa, hình thành **biến thể hàng đợi CLH** bên trong của nó. Các điểm cải tiến chính có hai phương diện dưới đây:

1. **Tự xoay + Bị chặn (Spin + Block)**: CLH lock sử dụng phương thức tự xoay thuần túy để chờ giải phóng lock, nhưng lượng lớn thao tác tự xoay sẽ chiếm dụng quá nhiều tài nguyên CPU. AQS đưa vào cơ chế hỗn hợp **Tự xoay + Bị chặn**:
   - Nếu luồng lấy lock thất bại, trước tiên sẽ tự xoay ngắn để thử lấy lock;
   - Nếu vẫn thất bại, luồng sẽ đi vào trạng thái bị chặn (blocked), chờ được đánh thức, từ đó giảm lãng phí CPU.
2. **Hàng đợi một chiều chuyển thành hàng đợi hai chiều**: CLH lock sử dụng hàng đợi một chiều, node chỉ biết trạng thái node tiền nhiệm, mà khi một node nào đó giải phóng lock, cần thông qua hàng đợi đánh thức node tiếp theo. AQS chuyển hàng đợi thành **hàng đợi hai chiều**, bổ sung thêm con trỏ `next`, làm cho node không chỉ biết node tiền nhiệm, mà còn có thể trực tiếp đánh thức node kế nhiệm, từ đó đơn giản hóa thao tác hàng đợi, nâng cao hiệu suất đánh thức.

#### Tại sao hiệu năng của AQS lại khá tốt?

Lý do vì bên trong AQS sử dụng lượng lớn thao tác `CAS`.

Bên trong AQS thông qua hàng đợi để lưu trữ các node luồng chờ đợi. Do hàng đợi là tài nguyên dùng chung, trong kịch bản đa luồng, cần đảm bảo việc truy cập đồng bộ hàng đợi.

Bên trong AQS thông qua thao tác `CAS` để kiểm soát việc truy cập đồng bộ hàng đợi, thao tác `CAS` chủ yếu dùng để kiểm soát an toàn concurrency của hai thao tác `khởi tạo hàng đợi` và `node luồng vào hàng đợi`. Mặc dù tận dụng `CAS` kiểm soát an toàn concurrency có thể đảm bảo hiệu năng tương đối tốt, nhưng đồng thời sẽ mang lại **độ phức tạp lập trình** tương đối cao.

#### Tại sao Node node trong AQS lại cần các trạng thái khác nhau?

Trạng thái `waitStatus` trong AQS tương tự như **State Machine (Máy trạng thái)**, thông qua các trạng thái khác nhau để thể hiện ý nghĩa khác nhau của Node node, và dựa theo các thao tác khác nhau, để kiểm soát sự chuyển đổi giữa các trạng thái.

- Trạng thái `0`: Sau khi node mới gia nhập hàng đợi, trạng thái ban đầu là `0`.
- Trạng thái `SIGNAL`: Khi có node mới gia nhập hàng đợi, lúc này trạng thái node tiền nhiệm của node mới sẽ từ `0` cập nhật thành `SIGNAL`, thể hiện sau khi node tiền nhiệm giải phóng lock, cần tiến hành thao tác đánh thức đối với node mới. Nếu đánh thức node kế nhiệm của node có trạng thái `SIGNAL`, sẽ cập nhật trạng thái `SIGNAL` thành `0`. Tức là thông qua việc xóa trạng thái `SIGNAL`, thể hiện đã thực thi thao tác đánh thức rồi.
- Trạng thái `CANCELLED`: Nếu một node trong hàng đợi chờ lấy lock, vì nguyên nhân nào đó bị thất bại, trạng thái node đó sẽ biến thành `CANCELLED`, thể hiện hủy lấy lock, node ở trạng thái này là bất thường, không thể bị đánh thức, cũng không thể đánh thức node kế nhiệm.

### Tư tưởng cốt lõi AQS

Tư tưởng cốt lõi của AQS là, nếu tài nguyên dùng chung được yêu cầu rảnh rỗi, thì thiết lập luồng yêu cầu tài nguyên hiện tại thành luồng làm việc có hiệu lực, và thiết lập tài nguyên dùng chung thành trạng thái bị khóa. Nếu tài nguyên dùng chung được yêu cầu bị chiếm dụng, vậy thì cần một bộ cơ chế luồng bị chặn chờ đợi cũng như phân bổ lock khi được đánh thức, cơ chế này AQS dựa trên **CLH lock** (Craig, Landin, and Hagersten locks) để tối ưu hóa nâng cấp triển khai.

**CLH lock** tiến hành cải tiến đối với tự xoay lock, là tự xoay lock dựa trên danh sách liên kết đơn. Trong kịch bản đa luồng, sẽ tổ chức các luồng yêu cầu lấy lock thành một hàng đợi một chiều, mỗi luồng chờ đợi sẽ thông qua tự xoay truy cập trạng thái của node luồng phía trước, sau khi node phía trước giải phóng lock, node hiện tại mới có thể lấy lock. Cấu trúc hàng đợi của **CLH lock** như hình dưới đây.

![CLH 锁的队列结构](https://oss.javaguide.cn/github/javaguide/open-source-project/clh-lock-queue-structure.png)

**Hàng đợi chờ** được sử dụng trong AQS là biến thể của hàng đợi CLH lock (tiếp theo gọi tắt là hàng đợi biến thể CLH).

Hàng đợi biến thể CLH của AQS là một hàng đợi hai chiều, các luồng tạm thời chưa lấy được lock sẽ được thêm vào hàng đợi này, điểm khác biệt giữa hàng đợi biến thể CLH và hàng đợi CLH lock ban đầu chủ yếu có hai điểm:

- Từ **Tự xoay** tối ưu thành **Tự xoay + Bị chặn**: Hiệu năng thao tác tự xoay rất cao, nhưng lượng lớn thao tác tự xoay khá chiếm dụng tài nguyên CPU, do đó trong hàng đợi biến thể CLH trước tiên sẽ thông qua tự xoay thử lấy lock, nếu thất bại mới tiến hành bị chặn chờ đợi.
- Từ **Hàng đợi một chiều** tối ưu thành **Hàng đợi hai chiều**: Trong hàng đợi biến thể CLH, sẽ tiến hành thao tác bị chặn đối với luồng chờ đợi, khi luồng phía trước hàng đợi giải phóng lock, cần tiến hành đánh thức đối với luồng phía sau, do đó bổ sung con trỏ `next`, trở thành hàng đợi hai chiều.

AQS đóng gói mỗi luồng yêu cầu tài nguyên dùng chung thành một node (Node) của hàng đợi biến thể CLH để triển khai việc phân bổ lock. Trong hàng đợi biến thể CLH, một node đại diện cho một luồng, nó bảo tồn tham chiếu luồng (thread), trạng thái node hiện tại trong hàng đợi (waitStatus), node tiền nhiệm (prev), node kế nhiệm (next).

Cấu trúc hàng đợi biến thể CLH trong AQS như hình dưới đây:

![CLH 变体队列结构](https://oss.javaguide.cn/github/javaguide/java/concurrent/clh-queue-structure-bianti.png)

Về giải thích chi tiết cấu trúc dữ liệu cốt lõi AQS - CLH lock, cực kỳ khuyến nghị đọc bài viết [Java AQS 核心数据结构-CLH 锁 - Qunar 技术沙龙](https://mp.weixin.qq.com/s/jEx-4XhNGOFdCo4Nou5tqg).

Sơ đồ nguyên lý cốt lõi của AQS (`AbstractQueuedSynchronizer`):

![CLH 变体队列](https://oss.javaguide.cn/github/javaguide/java/concurrent/clh-queue-state.png)

AQS sử dụng **biến thành viên int `state` biểu thị trạng thái đồng bộ**, thông qua **hàng đợi chờ/xếp hàng luồng FIFO** tích hợp sẵn để hoàn thành công việc xếp hàng của các luồng lấy tài nguyên.

Biến `state` được sửa đổi bằng `volatile`, dùng để hiển thị tình hình lấy tài nguyên găng (critical resource) hiện tại. Ở đây tác dụng của `volatile` không chỉ là đảm bảo tính nhìn thấy, điều quan trọng hơn là thông qua quy tắc happens-before (thao tác ghi biến volatile xảy ra trước thao tác đọc tiếp theo) để ngăn ngừa trình biên dịch và bộ xử lý tiến hành sắp xếp lại lệnh, từ đó đảm bảo tính đúng đắn ngữ nghĩa của lock.

```java
// Biến dùng chung, sử dụng volatile sửa đổi, đảm bảo tính nhìn thấy của luồng và ngăn ngừa sắp xếp lại lệnh
private volatile int state;
```

Ngoài ra, thông tin trạng thái `state` có thể thông qua `getState()`, `setState()` và `compareAndSetState()` kiểu `protected` để thao tác. Hơn nữa, mấy phương thức này đều do `final` sửa đổi, trong lớp con không thể bị override.

```java
// Trả về giá trị hiện tại của trạng thái đồng bộ
protected final int getState() {
     return state;
}
 // Thiết lập giá trị của trạng thái đồng bộ
protected final void setState(int newState) {
     state = newState;
}
// Tự tử (thao tác CAS) thiết lập giá trị trạng thái đồng bộ thành giá trị cho trước update nếu trạng thái đồng bộ hiện tại bằng expect (giá trị kỳ vọng)
protected final boolean compareAndSetState(int expect, int update) {
      return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

Lấy khóa loại trừ reentrant `ReentrantLock` làm ví dụ, bên trong nó bảo trì một biến `state`, dùng để biểu thị trạng thái bị chiếm dụng của lock. Giá trị ban đầu của `state` là 0, biểu thị lock đang ở trạng thái chưa khóa. Khi Luồng A gọi phương thức `lock()`, sẽ thử thông qua phương thức `tryAcquire()` độc chiếm lock đó, và làm cho giá trị của `state` cộng thêm 1. Nếu thành công, vậy Luồng A lấy được lock. Nếu thất bại, vậy Luồng A sẽ được thêm vào một hàng đợi chờ (hàng đợi biến thể CLH), cho đến khi luồng khác giải phóng lock đó. Giả sử Luồng A lấy lock thành công, trước khi giải phóng lock, bản thân Luồng A có thể lặp lại lấy lock này (`state` sẽ lũy kế). Đây chính là sự thể hiện của tính reentrant: Một luồng có thể nhiều lần lấy cùng một lock mà không bị chặn. Tuy nhiên, điều này cũng có nghĩa là, một luồng bắt buộc phải giải phóng số lần bằng với số lần lấy lock, mới có thể làm cho giá trị của `state` trở về 0, tức là làm cho lock khôi phục về trạng thái chưa khóa. Chỉ có như vậy, các luồng chờ đợi khác mới có cơ hội lấy lock đó.

Quá trình Luồng A thử lấy lock như hình dưới đây (Nguồn hình [Từ triển khai ReentrantLock xem nguyên lý và ứng dụng AQS - Đội ngũ kỹ thuật Meituan](./reentrantlock.md)):

![AQS 独占模式获取锁](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-exclusive-mode-acquire-lock.png)

Lại lấy bộ đếm đếm ngược `CountDownLatch` làm ví dụ, có thể khởi tạo `state` thành N, biểu thị cần chờ N lần gọi `countDown()`. N biểu thị số sự kiện hoặc số lần đếm, không yêu cầu nhất quán với số luồng; cùng một luồng có thể gọi nhiều lần `countDown()`, cũng có thể do nhiều luồng lần lượt gọi. Khi `state` biến thành 0, AQS sẽ đánh thức các luồng bị chặn do gọi `await()` trong hàng đợi chờ, các luồng này về sau có thể tiếp tục thực thi.

### Ý nghĩa trạng thái waitStatus của Node node

Trạng thái `waitStatus` trong AQS tương tự như **State Machine (Máy trạng thái)**, thông qua các trạng thái khác nhau để thể hiện ý nghĩa khác nhau của Node node, và dựa theo các thao tác khác nhau, để kiểm soát sự chuyển đổi giữa các trạng thái.

| Trạng thái Node | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `CANCELLED` | 1 | Biểu thị luồng đã hủy lấy lock. Luồng khi chờ lấy tài nguyên bị ngắt, chờ tài nguyên timeout sẽ cập nhật thành trạng thái này. |
| `SIGNAL` | -1 | Biểu thị node kế nhiệm cần node hiện tại đánh thức. Sau khi node luồng hiện tại giải phóng lock, cần tiến hành đánh thức đối với node kế nhiệm. |
| `CONDITION` | -2 | Biểu thị node đang chờ Condition. Khi luồng khác gọi phương thức `signal()` của Condition, node sẽ từ hàng đợi chờ chuyển sang hàng đợi đồng bộ để chờ lấy tài nguyên. |
| `PROPAGATE` | -3 | Dùng cho mode shared (chia sẻ). Trong mode shared, có thể xuất hiện trường hợp luồng trong hàng đợi không thể bị đánh thức, do đó đưa vào trạng thái `PROPAGATE` để giải quyết vấn đề này. |
| | 0 | Trạng thái ban đầu của node mới gia nhập hàng đợi. |

Trong mã nguồn của AQS, thường xuyên sử dụng `> 0`, `< 0` để tiến hành phán đoán đối với `waitStatus`.

Nếu `waitStatus > 0`, thể hiện trạng thái node đã hủy chờ lấy tài nguyên.

Nếu `waitStatus < 0`, thể hiện trạng thái node đang ở trạng thái bình thường, tức là chưa hủy chờ đợi.

Trong đó trạng thái `SIGNAL` là quan trọng nhất, sự chuyển đổi trạng thái node và thao tác tương ứng như sau:

| Chuyển đổi trạng thái | Thao tác tương ứng |
| --- | --- |
| `0` | Khi node mới vào hàng đợi, trạng thái ban đầu là `0`. |
| `0 -> SIGNAL` | Khi node mới vào hàng đợi, trạng thái node tiền nhiệm của nó sẽ từ `0` cập nhật thành `SIGNAL`. Trạng thái `SIGNAL` thể hiện node tiếp theo của node đó cần được đánh thức. |
| `SIGNAL -> 0` | Khi đánh thức node kế nhiệm, cần xóa trạng thái của node hiện tại. Thường xảy ra ở node `head`, ví dụ trạng thái node `head` từ `SIGNAL` cập nhật thành `0`, thể hiện đã đánh thức đối với node kế nhiệm của node `head` rồi. |
| `0 -> PROPAGATE` | Bên trong AQS đưa vào trạng thái `PROPAGATE`, để giải quyết trường hợp trong kịch bản concurrency có thể gây ra node luồng không thể bị đánh thức. (Sẽ giảng trong phần phân tích mã nguồn lấy tài nguyên mode shared AQS) |

### Bộ đồng bộ hóa tự định nghĩa

Dựa trên AQS có thể triển khai bộ đồng bộ hóa tự định nghĩa, AQS cung cấp 5 template method (Template Method Pattern). Nếu cần tự định nghĩa bộ đồng bộ hóa phương thức chung là như thế này (một ứng dụng rất kinh điển của Template Method Pattern):

1. Bộ đồng bộ hóa tự định nghĩa kế thừa `AbstractQueuedSynchronizer`.
2. Override các template method do AQS bộc lộ ra.

**AQS sử dụng Template Method Pattern, khi tự định nghĩa bộ đồng bộ hóa cần override các hook method do AQS cung cấp dưới đây:**

```java
// Phương thức exclusive. Thử lấy tài nguyên, thành công trả về true, thất bại trả về false.
protected boolean tryAcquire(int)
// Phương thức exclusive. Thử giải phóng tài nguyên, thành công trả về true, thất bại trả về false.
protected boolean tryRelease(int)
// Phương thức shared. Thử lấy tài nguyên. Số âm biểu thị thất bại; 0 biểu thị thành công nhưng không còn tài nguyên khả dụng; số dương biểu thị thành công và còn tài nguyên.
protected int tryAcquireShared(int)
// Phương thức shared. Thử giải phóng tài nguyên, thành công trả về true, thất bại trả về false.
protected boolean tryReleaseShared(int)
// Luồng này có đang độc chiếm tài nguyên không. Chỉ khi dùng đến condition mới cần triển khai nó.
protected boolean isHeldExclusively()
```

**Hook method là gì?** Hook method là một phương thức được khai báo trong lớp trừu tượng, thường sử dụng từ khóa `protected` sửa đổi, nó có thể là phương thức rỗng (do lớp con triển khai), cũng có thể là phương thức có triển khai mặc định. Template Design Pattern thông qua hook method kiểm soát việc triển khai các bước cố định.

Do giới hạn độ dài, ở đây không giới thiệu chi tiết Template Method Pattern nữa, bạn nào chưa rõ có thể xem bài viết này: [Template Method Pattern cải tạo bằng Java 8 thực sự là YYDS!](https://mp.weixin.qq.com/s/zpScSCktFpnSWHWIQem2jg).

Ngoài các hook method đề cập ở trên, các phương thức khác trong lớp AQS đều là `final`, nên không thể bị lớp khác override.

### Cách thức chia sẻ tài nguyên AQS

AQS định nghĩa hai cách thức chia sẻ tài nguyên: `Exclusive` (Độc chiếm, chỉ có một luồng có thể thực thi, như `ReentrantLock`) và `Share` (Chia sẻ, nhiều luồng có thể đồng thời thực thi, như `Semaphore`/`CountDownLatch`).

Nói chung, cách thức chia sẻ của bộ đồng bộ hóa tự định nghĩa hoặc là Exclusive, hoặc là Share, họ cũng chỉ cần triển khai một trong hai cặp `tryAcquire-tryRelease`, `tryAcquireShared-tryReleaseShared` là được. Nhưng AQS cũng hỗ trợ bộ đồng bộ hóa tự định nghĩa đồng thời triển khai cả hai cách thức Exclusive và Share, như `ReentrantReadWriteLock`.

### So sánh sâu giữa Mode Exclusive và Mode Shared

Phía trên đã giới thiệu tóm tắt hai cách thức chia sẻ tài nguyên của AQS, dưới đây từ nhiều chiều tiến hành so sánh hệ thống giữa Mode Exclusive và Mode Shared, giúp hiểu sâu hơn về sự khác biệt giữa hai cái.

#### So sánh đặc tính

| Chiều so sánh | Mode Exclusive (Exclusive) | Mode Shared (Share) |
| --- | --- | --- |
| **Độ concurrency** | Tại cùng một thời điểm chỉ có 1 luồng lấy được tài nguyên | Tại cùng một thời điểm có thể có nhiều luồng đồng thời lấy được tài nguyên |
| **Cổng vào lấy tài nguyên** | `acquire(int arg)` | `acquireShared(int arg)` |
| **Cổng vào giải phóng tài nguyên** | `release(int arg)` | `releaseShared(int arg)` |
| **Template method cần override** | `tryAcquire(int)` / `tryRelease(int)` | `tryAcquireShared(int)` / `tryReleaseShared(int)` |
| **Giá trị trả về của tryXxx** | `boolean`, `true` biểu thị lấy/giải phóng thành công | `int` (khi lấy), số âm biểu thị thất bại, 0 biểu thị thành công nhưng không còn tài nguyên, số dương biểu thị thành công và còn tài nguyên; `boolean` (khi giải phóng) |
| **Đánh thức node kế nhiệm** | Khi giải phóng tài nguyên đánh thức một node kế nhiệm | Sau khi lấy tài nguyên thành công, nếu còn tài nguyên dư, tiếp tục đánh thức các node phía sau (đánh thức lan truyền) |
| **Nhãn kiểu Node** | `Node.EXCLUSIVE` (`null`) | `Node.SHARED` (Một instance `Node` static) |
| **Triển khai điển hình** | `ReentrantLock`, Lock Ghi của `ReentrantReadWriteLock` | `Semaphore`, `CountDownLatch`, Lock Đọc của `ReentrantReadWriteLock` |

#### Ngữ nghĩa của `state` trong các bộ đồng bộ hóa khác nhau

`state` trong AQS là một biến trạng thái đồng bộ chung, các bộ đồng bộ hóa khác nhau ban cho nó ý nghĩa khác nhau:

| Bộ đồng bộ hóa | Mode | Ngữ nghĩa của `state` |
| --- | --- | --- |
| `ReentrantLock` | Exclusive | Biểu thị số lần reentrant của lock. `state == 0` biểu thị lock rảnh rỗi; `state > 0` biểu thị lock đang được nắm giữ, giá trị là số lần reentrant |
| `ReentrantReadWriteLock` | Exclusive + Shared | 16 bit cao biểu thị số lượng nắm giữ Lock Đọc (Shared), 16 bit thấp biểu thị số lần reentrant của Lock Ghi (Exclusive) |
| `Semaphore` | Shared | Biểu thị số lượng giấy phép (permit) khả dụng. Mỗi lần `acquire()` giảm, `release()` tăng |
| `CountDownLatch` | Shared | Biểu thị số đếm cần chờ. Mỗi lần `countDown()` giảm 1, đến 0 thì đánh thức tất cả các luồng chờ |

Dưới đây thông qua một ví dụ code để cảm nhận trực quan sự khác biệt khi sử dụng giữa Mode Exclusive và Mode Shared:

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.ReentrantLock;

public class ExclusiveVsSharedDemo {
    public static void main(String[] args) {
        // Mode Exclusive: Tại cùng một thời điểm chỉ có 1 luồng có thể vào vùng găng (critical section)
        ReentrantLock lock = new ReentrantLock();

        // Mode Shared: Tại cùng một thời điểm tối đa 3 luồng có thể vào vùng găng
        Semaphore semaphore = new Semaphore(3);

        // Ví dụ Mode Exclusive
        Runnable exclusiveTask = () -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName()
                        + " lấy được Exclusive lock, đang thực thi...");
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                lock.unlock();
            }
        };

        // Ví dụ Mode Shared
        Runnable sharedTask = () -> {
            boolean acquired = false;
            try {
                semaphore.acquire();
                acquired = true;
                System.out.println(Thread.currentThread().getName()
                        + " lấy được permit, đang thực thi...");
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                if (acquired) {
                    semaphore.release();
                }
            }
        };

        System.out.println("=== Mode Exclusive (ReentrantLock) ===");
        for (int i = 0; i < 5; i++) {
            new Thread(exclusiveTask, "ExclusiveThread-" + i).start();
        }

        try { Thread.sleep(3000); } catch (InterruptedException e) { }

        System.out.println("\n=== Mode Shared (Semaphore) ===");
        for (int i = 0; i < 5; i++) {
            new Thread(sharedTask, "SharedThread-" + i).start();
        }
    }
}
```

Chạy code trên có thể quan sát thấy: Dưới Mode Exclusive tại cùng một thời điểm chỉ có một luồng thực thi, nhưng `ReentrantLock` không công bằng mặc định không đảm bảo các luồng lấy được lock nghiêm ngặt theo thứ tự khởi động hoặc chờ đợi; dưới Mode Shared tối đa có 3 luồng đồng thời thực thi.

### Phân tích mã nguồn lấy tài nguyên AQS (Mode Exclusive)

Phương thức cổng vào lấy tài nguyên theo Mode Exclusive trong AQS là `acquire()`, như sau:

```java
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

Trong `acquire()`, luồng sẽ thử lấy tài nguyên dùng chung trước; nếu lấy thất bại, sẽ đóng gói luồng thành Node node thêm vào hàng đợi chờ của AQS; sau khi gia nhập hàng đợi, sẽ cho luồng trong hàng đợi chờ thử lấy tài nguyên, và sẽ tiến hành thao tác bị chặn đối với luồng. Lần lượt tương ứng với ba phương thức sau:

- `tryAcquire()`: Thử lấy lock (template method), `AQS` không cung cấp triển khai cụ thể, do lớp con triển khai.
- `addWaiter()`: Nếu lấy lock thất bại, sẽ đóng gói luồng hiện tại thành Node node thêm vào hàng đợi biến thể CLH của AQS để chờ lấy lock.
- `acquireQueued()`: Tiến hành bị chặn đối với luồng, và gọi phương thức `tryAcquire()` cho luồng trong hàng đợi thử lấy lock.

#### Phân tích `tryAcquire()`

Template method `tryAcquire()` tương ứng trong AQS như sau:

```java
// AQS
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

Phương thức `tryAcquire()` là template method do AQS cung cấp, không cung cấp triển khai mặc định.

Do đó, khi phân tích phương thức `tryAcquire()` ở đây, lấy Non-fair Lock (Exclusive lock) của `ReentrantLock` làm ví dụ để phân tích, `tryAcquire()` triển khai bên trong `ReentrantLock` sẽ gọi đến `nonfairTryAcquire()` dưới đây:

```java
// ReentrantLock
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    // 1. Lấy trạng thái state trong AQS
    int c = getState();
    // 2. Nếu state bằng 0, chứng tỏ lock chưa bị luồng khác chiếm dụng
    if (c == 0) {
        // 2.1. Thông qua CAS tiến hành cập nhật đối với state
        if (compareAndSetState(0, acquires)) {
            // 2.2. Nếu CAS cập nhật thành công, thiết lập chủ sở hữu của lock thành luồng hiện tại
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 3. Nếu luồng hiện tại giống với luồng sở hữu lock, chứng tỏ phát sinh "Lock Reentrancy"
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        // 3.1. Cộng thêm 1 vào số lần reentrant của lock
        setState(nextc);
        return true;
    }
    // 4. Nếu lock bị luồng khác chiếm dụng, thì trả về false, biểu thị lấy lock thất bại
    return false;
}
```

Bên trong phương thức `nonfairTryAcquire()`, chủ yếu thông qua hai thao tác cốt lõi để hoàn thành việc lấy tài nguyên:

- Thông qua `CAS` cập nhật biến `state`. `state == 0` biểu thị tài nguyên chưa bị chiếm dụng. `state > 0` biểu thị tài nguyên bị chiếm dụng, lúc này `state` biểu thị số lần reentrant.
- Thông qua `setExclusiveOwnerThread()` thiết lập luồng nắm giữ tài nguyên.

Nếu luồng cập nhật biến `state` thành công, thể hiện đã lấy được tài nguyên, do đó thiết lập luồng nắm giữ tài nguyên thành luồng hiện tại là được.

#### Phân tích `addWaiter()`

Sau khi thử lấy tài nguyên thất bại thông qua phương thức `tryAcquire()`, sẽ gọi phương thức `addWaiter()` để đóng gói luồng hiện tại thành Node node gia nhập vào hàng đợi bên trong `AQS`. Code `addWaiter()` như sau:

```java
// AQS
private Node addWaiter(Node mode) {
    // 1. Đóng gói luồng hiện tại thành Node node.
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    // 2. Nếu pred != null, chứng tỏ tail node đã được khởi tạo, trực tiếp đưa Node node vào hàng đợi là được.
    if (pred != null) {
        node.prev = pred;
        // 2.1. Thông qua CAS kiểm soát an toàn concurrency.
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 3. Khởi tạo hàng đợi, và đưa Node node mới tạo gia nhập hàng đợi.
    enq(node);
    return node;
}
```

**An toàn concurrency khi node vào hàng đợi:**

Trong phương thức `addWaiter()`, cần thực thi thao tác **vào hàng đợi** của Node node. Do ở trong môi trường đa luồng, do đó cần thông qua thao tác `CAS` đảm bảo an toàn concurrency.

Thông qua thao tác `CAS` để cập nhật con trỏ `tail` trỏ tới Node node mới vào hàng đợi, `CAS` có thể đảm bảo chỉ có một luồng sửa đổi thành công con trỏ `tail`, lấy cái này để đảm bảo an toàn concurrency khi Node node vào hàng đợi.

**Khởi tạo hàng đợi bên trong AQS:**

Khi thực thi `addWaiter()`, nếu phát hiện `pred == null`, tức con trỏ `tail` bằng null, chứng tỏ hàng đợi chưa được khởi tạo, cần gọi phương thức `enq()` để khởi tạo hàng đợi, và đưa `Node` node gia nhập vào hàng đợi sau khi khởi tạo, code như sau:

```java
// AQS
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) {
            // 1. Thông qua thao tác CAS đảm bảo an toàn concurrency khi khởi tạo hàng đợi
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            // 2. Tương tự thao tác node vào hàng đợi trong phương thức addWaiter()
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

Trong phương thức `enq()` khởi tạo hàng đợi, trong quá trình khởi tạo, cũng cần thông qua `CAS` để đảm bảo an toàn concurrency.

Khởi tạo hàng đợi tổng cộng bao gồm hai bước: Khởi tạo node `head`, `tail` trỏ tới node `head`.

**Hàng đợi sau khi khởi tạo như hình dưới đây:**

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/clh-queue-structure-init.png)

#### Phân tích `acquireQueued()`

Để tiện đọc, ở đây dán lại code lấy tài nguyên của `acquire()` trong `AQS`:

```java
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

Trong phương thức `acquire()`, sau khi đưa `Node` node gia nhập hàng đợi thông qua phương thức `addWaiter()`, sẽ gọi phương thức `acquireQueued()`. Code như sau:

```java
// AQS: Làm cho node trong hàng đợi thử lấy lock, và tiến hành bị chặn đối với luồng.
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 1. Thử lấy lock.
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 2. Phán đoán luồng có thể bị chặn không, nếu có thể, thì làm bị chặn luồng hiện tại.
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 3. Nếu lấy lock thất bại, sẽ hủy lấy lock, cập nhật trạng thái node thành CANCELLED.
        if (failed)
            cancelAcquire(node);
    }
}
```

Trong phương thức `acquireQueued()`, chủ yếu làm hai việc:

- **Thử lấy tài nguyên:** Sau khi luồng hiện tại gia nhập hàng đợi, nếu phát hiện node tiền nhiệm là node `head`, chứng tỏ luồng hiện tại là node chờ đầu tiên trong hàng đợi, thế là gọi `tryAcquire()` thử lấy tài nguyên.
- **Bị chặn luồng hiện tại**: Nếu thử lấy tài nguyên thất bại, thì cần làm bị chặn luồng hiện tại, chờ được đánh thức sau đó lấy tài nguyên.

**1. Thử lấy tài nguyên**

Trong phương thức `acquireQueued()`, thử lấy tài nguyên tổng cộng có 2 bước:

- `p == head`: Thể hiện node tiền nhiệm của node hiện tại là node `head`. Lúc này node hiện tại là node chờ đầu tiên trong hàng đợi AQS.
- `tryAcquire(arg) == true`: Thể hiện luồng hiện tại thử lấy tài nguyên thành công.

Sau khi lấy tài nguyên thành công, thì cần **xóa node luồng hiện tại khỏi hàng đợi chờ**. Thao tác xóa là: Thiết lập node luồng đang chờ hiện tại thành node `head` (node `head` là node ảo, không tham gia xếp hàng lấy tài nguyên).

**2. Bị chặn luồng hiện tại**

Trong `AQS`, việc đánh thức node hiện tại cần phụ thuộc vào node phía trước. Nếu node phía trước hủy lấy lock, trạng thái của nó sẽ biến thành `CANCELLED`, node trạng thái `CANCELLED` không lấy được lock, cũng không thể thực thi thao tác mở khóa để đánh thức node hiện tại. Do đó trước khi làm bị chặn luồng hiện tại, cần bỏ qua các node trạng thái `CANCELLED`.

Thông qua phương thức `shouldParkAfterFailedAcquire()` để phán đoán node luồng hiện tại có thể bị chặn không, như sau:

```java
// AQS: Phán đoán node luồng hiện tại có thể bị chặn không.
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    // 1. Trạng thái node tiền nhiệm bình thường, trực tiếp trả về true là được.
    if (ws == Node.SIGNAL)
        return true;
    // 2. ws > 0 biểu thị trạng thái node tiền nhiệm bất thường, tức là trạng thái CANCELLED, cần bỏ qua node trạng thái bất thường.
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 3. Nếu trạng thái node tiền nhiệm không phải SIGNAL, cũng không phải CANCELLED, thì thiết lập trạng thái thành SIGNAL.
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

Logic phán đoán trong phương thức `shouldParkAfterFailedAcquire()`:

- Nếu phát hiện trạng thái của node tiền nhiệm là `SIGNAL`, thì có thể làm bị chặn luồng hiện tại.
- Nếu phát hiện trạng thái của node tiền nhiệm là `CANCELLED`, thì cần bỏ qua node trạng thái `CANCELLED`.
- Nếu phát hiện trạng thái của node tiền nhiệm không phải `SIGNAL` và `CANCELLED`, thể hiện trạng thái của node tiền nhiệm đang ở trạng thái chờ tài nguyên bình thường, do đó thiết lập trạng thái của node tiền nhiệm thành `SIGNAL`, thể hiện node tiền nhiệm đó cần tiến hành đánh thức đối với node phía sau.

Khi phán đoán luồng hiện tại có thể bị chặn, thông qua gọi phương thức `parkAndCheckInterrupt()` để làm bị chặn luồng hiện tại. Bên trong sử dụng `LockSupport` để triển khai bị chặn. `LockSupport` bên dưới dựa trên lớp `Unsafe` để làm bị chặn luồng, code như sau:

```java
// AQS
private final boolean parkAndCheckInterrupt() {
    // 1. Luồng bị chặn tại đây
    LockSupport.park(this);
    // 2. Sau khi luồng được đánh thức, trả về trạng thái ngắt của luồng
    return Thread.interrupted();
}
```

**Tại sao sau khi luồng được đánh thức, lại phải trả về trạng thái ngắt của luồng?**

Trong phương thức `parkAndCheckInterrupt()`, khi thực thi xong `LockSupport.park(this)`, luồng sẽ bị chặn, code như sau:

```java
// AQS
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    // Sau khi luồng được đánh thức, cần trả về trạng thái ngắt của luồng
    return Thread.interrupted();
}
```

Khi luồng được đánh thức, cần thực thi `Thread.interrupted()` để trả về trạng thái ngắt của luồng, tại sao lại như vậy?

Điều này có quan hệ với cơ chế phối hợp ngắt của luồng, sau khi luồng được đánh thức, không chắc chắn là được đánh thức do bị ngắt hay do `LockSupport.unpark()` đánh thức, do đó cần thông qua trạng thái ngắt của luồng để phán đoán.

**Trong phương thức `acquire()`, tại sao cần gọi `selfInterrupt()`?**

Code phương thức `acquire()` như sau:

```java
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

Trong phương thức `acquire()`, khi điều kiện của câu lệnh `if` trả về `true`, sẽ gọi `selfInterrupt()`, phương thức đó sẽ ngắt luồng hiện tại, tại sao cần ngắt luồng hiện tại?

Khi phán đoán `if` thành `true`, cần `tryAcquire()` trả về `false`, và `acquireQueued()` trả về `true`.

Trong đó phương thức `acquireQueued()` trả về là **trạng thái ngắt** của luồng sau khi được đánh thức, thực thi thông qua `Thread.interrupted()` để trả về. Phương thức đó trong khi trả về trạng thái ngắt, sẽ xóa trạng thái ngắt của luồng.

Do đó nếu phán đoán `if` là `true`, thể hiện trạng thái ngắt của luồng là `true`, nhưng sau khi gọi `Thread.interrupted()`, trạng thái ngắt của luồng bị xóa thành `false`, do đó cần thực thi lại `selfInterrupt()` để thiết lập lại trạng thái ngắt của luồng.

### Phân tích mã nguồn giải phóng tài nguyên AQS (Mode Exclusive)

Phương thức cổng vào giải phóng tài nguyên theo Mode Exclusive trong AQS là `release()`, code như sau:

```java
// AQS
public final boolean release(int arg) {
    // 1. Thử giải phóng lock
    if (tryRelease(arg)) {
        Node h = head;
        // 2. Đánh thức node kế nhiệm
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

Trong phương thức `release()`, chủ yếu làm hai việc: Thử giải phóng lock và đánh thức node kế nhiệm. Phương thức tương ứng như sau:

**1. Thử giải phóng lock**

Thông qua phương thức `tryRelease()` thử giải phóng lock, phương thức này là template method, do bộ đồng bộ hóa tự định nghĩa triển khai, do đó ở đây vẫn lấy `ReentrantLock` làm ví dụ để giải thích.

Phương thức `tryRelease()` triển khai trong `ReentrantLock` như sau:

```java
// ReentrantLock
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    // 1. Phán đoán luồng nắm giữ lock có phải là luồng hiện tại không
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 2. Nếu state bằng 0, thì thể hiện luồng hiện tại đã không còn số lần reentrant. Do đó cập nhật free thành true, thể hiện luồng này sẽ giải phóng lock.
    if (c == 0) {
        free = true;
        // 3. Cập nhật luồng nắm giữ tài nguyên thành null
        setExclusiveOwnerThread(null);
    }
    // 4. Cập nhật giá trị state
    setState(c);
    return free;
}
```

Trong phương thức `tryRelease()`, trước tiên sẽ tính giá trị `state` sau khi giải phóng lock, phán đoán giá trị `state` có bằng 0 không.

- Nếu `state == 0`, thể hiện luồng này không còn số lần reentrant nữa, cập nhật `free = true`, và sửa luồng nắm giữ tài nguyên thành null, thể hiện luồng này giải phóng hoàn toàn ổ khóa này.
- Nếu `state != 0`, thể hiện luồng này vẫn còn số lần reentrant, do đó không cập nhật giá trị `free`, giá trị `free` là `false` thể hiện luồng này chưa giải phóng hoàn toàn ổ khóa này.

Sau đó cập nhật giá trị `state`, và trả về giá trị `free`, giá trị `free` thể hiện luồng có giải phóng hoàn toàn lock không.

**2. Đánh thức node kế nhiệm**

Nếu `tryRelease()` trả về `true`, thể hiện luồng đã không còn số lần reentrant nữa, lock đã được giải phóng hoàn toàn, do đó cần đánh thức node kế nhiệm.

Trước khi đánh thức node kế nhiệm, cần phán đoán xem có thể đánh thức node kế nhiệm không, điều kiện phán đoán là: `h != null && h.waitStatus != 0`. Ở đây giải thích một chút tại sao lại phán đoán như vậy:

- `h == null`: Thể hiện node `head` vẫn chưa được khởi tạo, tức là hàng đợi trong AQS chưa được khởi tạo, do đó không thể đánh thức node luồng trong hàng đợi.
- `h != null && h.waitStatus == 0`: Thể hiện node đầu vừa mới khởi tạo xong (trạng thái khởi tạo của node là 0), luồng node kế nhiệm vẫn chưa vào hàng đợi thành công, do đó không cần tiến hành đánh thức đối với node phía sau. (Khi node kế nhiệm vào hàng đợi xong, sẽ sửa trạng thái của node tiền nhiệm thành `SIGNAL`, thể hiện cần tiến hành đánh thức đối với node kế nhiệm)
- `h != null && h.waitStatus != 0`: Trong đó `waitStatus` có khả năng lớn hơn 0, cũng có khả năng nhỏ hơn 0. Trong đó `> 0` thể hiện node đã hủy chờ lấy tài nguyên, `< 0` thể hiện node đang ở trạng thái chờ bình thường.

Tiếp theo đi vào phương thức `unparkSuccessor()` xem làm sao đánh thức node kế nhiệm:

```java
// AQS: Tham số đầu vào node ở đây là node đầu của hàng đợi (node đầu ảo)
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    // 1. Xóa trạng thái của node đầu, chuẩn bị cho việc đánh thức sau đó.
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    // 2. Nếu node kế nhiệm bất thường, thì cần duyệt ngược từ tail về trước, tìm node trạng thái bình thường để đánh thức.
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 3. Đánh thức node kế nhiệm
        LockSupport.unpark(s.thread);
}
```

Trong `unparkSuccessor()`, nếu trạng thái của node đầu `< 0` (trong trường hợp bình thường, chỉ cần có node kế nhiệm, trạng thái node đầu nên là `SIGNAL`, tức -1), biểu thị cần tiến hành đánh thức đối với node kế nhiệm, do đó ở đây xóa trước nhãn trạng thái của node đầu, sửa trạng thái thành 0, biểu thị đã thực thi thao tác đánh thức đối với node phía sau rồi.

Nếu `s == null` hoặc `s.waitStatus > 0`, thể hiện node kế nhiệm bất thường, lúc này không thể đánh thức node bất thường, mà phải tìm node trạng thái bình thường để đánh thức.

Do đó cần duyệt ngược từ con trỏ `tail` về trước, để tìm node đầu tiên trạng thái bình thường (`waitStatus <= 0`) tiến hành đánh thức.

**Tại sao phải duyệt ngược từ con trỏ `tail` về trước, mà không phải duyệt từ con trỏ `head` về sau, tìm node trạng thái bình thường?**

Hướng duyệt có quan hệ với **thao tác node vào hàng đợi**. Phương thức vào hàng đợi như sau:

```java
// AQS: Phương thức node vào hàng đợi
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        // 1. Sửa con trỏ prev trước.
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            // 2. Sau đó mới sửa con trỏ next.
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

Trong phương thức `addWaiter()`, `node` node vào hàng đợi cần sửa hai con trỏ `node.prev` và `pred.next`, nhưng hai thao tác này không phải là **thao tác atomic**, đã sửa con trỏ `node.prev` trước, về sau mới sửa con trỏ `pred.next`.

Trong trường hợp cực đoan, có thể xuất hiện trạng thái node tiếp theo của node `head` là `CANCELLED`, lúc này node mới vào hàng đợi chỉ mới cập nhật con trỏ `node.prev`, chưa cập nhật con trỏ `pred.next`, như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-addWaiter.png)

Như vậy nếu duyệt từ con trỏ `head` về sau, không thể tìm thấy node mới vào hàng đợi, do đó cần duyệt từ con trỏ `tail` về trước để tìm node mới vào hàng đợi.

### Minh họa nguyên lý hoạt động AQS (Mode Exclusive)

Đến đây, mã nguồn lấy tài nguyên, giải phóng tài nguyên theo Mode Exclusive trong AQS đã giảng xong. Để có nhận thức rõ ràng hơn về nguyên lý hoạt động của AQS, sự thay đổi trạng thái node, tiếp theo sẽ thông qua cách vẽ hình để hiểu toàn bộ nguyên lý hoạt động của AQS.

Do AQS là công cụ đồng bộ bên dưới, các phương thức lấy và giải phóng tài nguyên không cung cấp triển khai cụ thể, do đó ở đây dựa trên `ReentrantLock` để vẽ hình giải thích.

Giả sử tổng cộng có 3 luồng thử lấy lock, các luồng lần lượt là `T1`, `T2` và `T3`.

Lúc này, giả sử luồng `T1` lấy được lock trước, luồng `T2` xếp hàng chờ lấy lock. Trước khi luồng `T2` đi vào hàng đợi, cần tiến hành khởi tạo đối với hàng đợi bên trong AQS. Node `head` sau khi khởi tạo có trạng thái là `0`. Hàng đợi bên trong AQS sau khi khởi tạo như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process.png)

Lúc này, luồng `T2` thử lấy lock. Do luồng `T1` nắm giữ lock, do đó luồng `T2` sẽ đi vào hàng đợi chờ lấy lock. Đồng thời sẽ cập nhật trạng thái của node tiền nhiệm (node `head`) từ `0` thành `SIGNAL`, thể hiện cần tiến hành đánh thức đối với node kế nhiệm của node `head`. Lúc này, hàng đợi bên trong AQS như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-2.png)

Lúc này, luồng `T3` thử lấy lock. Do luồng `T1` nắm giữ lock, do đó luồng `T3` sẽ đi vào hàng đợi chờ lấy lock. Đồng thời sẽ cập nhật trạng thái của node tiền nhiệm (node luồng `T2`) từ `0` thành `SIGNAL`, thể hiện node luồng `T2` cần tiến hành đánh thức đối với node kế nhiệm. Lúc này, hàng đợi bên trong AQS như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-3.png)

Lúc này, giả sử luồng `T1` giải phóng lock, sẽ đánh thức node kế nhiệm `T2`. Luồng `T2` sau khi được đánh thức lấy được lock, và sẽ thoát khỏi hàng đợi chờ.

Ở đây node luồng `T2` thoát khỏi hàng đợi chờ không phải trực tiếp xóa khỏi hàng đợi, mà làm cho node luồng `T2` trở thành node `head` mới, lấy cái này để thoát khỏi việc chờ lấy tài nguyên. Lúc này hàng đợi bên trong AQS như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-4.png)

Lúc này, giả sử luồng `T2` giải phóng lock, sẽ đánh thức node kế nhiệm `T3`. Luồng `T3` sau khi lấy được lock, cũng tương tự thoát khỏi hàng đợi chờ, tức biến node luồng `T3` thành node `head` để thoát khỏi việc chờ lấy tài nguyên. Lúc này hàng đợi bên trong AQS như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/aqs-acquire-and-release-process-5.png)

### Phân tích mã nguồn lấy tài nguyên AQS (Mode Shared)

Phương thức cổng vào lấy tài nguyên theo Mode Shared trong AQS là `acquireShared()`, như sau:

```java
// AQS
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

Trong phương thức `acquireShared()`, trước tiên sẽ thử lấy lock chia sẻ, nếu lấy thất bại, thì đưa luồng hiện tại gia nhập hàng đợi bị chặn, chờ được đánh thức sau đó thử lấy lock chia sẻ, lần lượt tương ứng với hai phương thức sau: `tryAcquireShared()` và `doAcquireShared()`.

Trong đó phương thức `tryAcquireShared()` là template method do AQS cung cấp, do bộ đồng bộ hóa triển khai logic cụ thể. Do đó ở đây lấy `Semaphore` làm ví dụ, để phân tích dưới Mode Shared làm sao lấy tài nguyên.

#### Phân tích `tryAcquireShared()`

Trong `Semaphore` triển khai Fair lock và Non-fair lock, tiếp theo lấy Non-fair lock làm ví dụ để phân tích mã nguồn `tryAcquireShared()`.

Phương thức `tryAcquireShared()` được override trong `Semaphore` sẽ gọi phương thức `nonfairTryAcquireShared()` dưới đây:

```java
// Semaphore override template method của AQS
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

// Semaphore
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        // 1. Lấy số lượng tài nguyên khả dụng.
        int available = getState();
        // 2. Tính số lượng tài nguyên còn lại.
        int remaining = available - acquires;
        // 3. Nếu số lượng tài nguyên còn lại < 0, thì chứng tỏ không đủ tài nguyên, trả về trực tiếp; nếu CAS cập nhật state thành công, thì chứng tỏ luồng hiện tại lấy được tài nguyên dùng chung, trả về trực tiếp.
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

Dưới Mode Shared, giá trị `state` trong AQS biểu thị số lượng tài nguyên dùng chung.

Trong phương thức `nonfairTryAcquireShared()`, sẽ ở trong vòng lặp vô hạn không ngừng thử lấy tài nguyên, nếu "Số tài nguyên còn lại không đủ" hoặc "Luồng hiện tại lấy tài nguyên thành công", thì thoát vòng lặp vô hạn. Phương thức trả về **Số lượng tài nguyên còn lại**, dựa theo giá trị trả về khác nhau, chia làm 3 trường hợp:

- **Số lượng tài nguyên còn lại > 0**: Biểu thị lấy tài nguyên thành công, và các luồng phía sau cũng có thể lấy tài nguyên thành công.
- **Số lượng tài nguyên còn lại = 0**: Biểu thị lấy tài nguyên thành công, nhưng các luồng phía sau không thể lấy tài nguyên thành công.
- **Số lượng tài nguyên còn lại < 0**: Biểu thị lấy tài nguyên thất bại.

#### Phân tích `doAcquireShared()`

Để tiện đọc, ở đây dán lại phương thức cổng vào lấy tài nguyên `acquireShared()`:

```java
// AQS
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

Trong phương thức `acquireShared()`, trước tiên sẽ thông qua `tryAcquireShared()` thử lấy tài nguyên.

Nếu phát hiện giá trị trả về của phương thức `< 0`, tức số tài nguyên còn lại nhỏ hơn 0, thì thể hiện luồng hiện tại lấy tài nguyên thất bại. Do đó sẽ đi vào phương thức `doAcquireShared()`, đưa luồng hiện tại gia nhập hàng đợi AQS để chờ đợi. Như sau:

```java
// AQS
private void doAcquireShared(int arg) {
    // 1. Đưa luồng hiện tại gia nhập hàng đợi để chờ.
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                // 2. Nếu luồng hiện tại là node đầu tiên của hàng đợi chờ, thì thử lấy tài nguyên.
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 3. Chuyển node luồng hiện tại ra khỏi hàng đợi chờ, và đánh thức các node luồng phía sau.
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 3. Nếu lấy tài nguyên thất bại, sẽ hủy lấy tài nguyên, cập nhật trạng thái node thành CANCELLED.
        if (failed)
            cancelAcquire(node);
    }
}
```

Do luồng hiện tại đã thử lấy tài nguyên thất bại rồi, do đó trong phương thức `doAcquireShared()`, cần đóng gói luồng hiện tại thành Node node, gia nhập hàng đợi để chờ đợi.

Điểm khác biệt lớn nhất giữa việc lấy tài nguyên theo **Mode Shared** và **Mode Exclusive** nằm ở chỗ: Dưới Mode Shared, số lượng tài nguyên có thể lớn hơn 1, tức có thể nhiều luồng đồng thời nắm giữ tài nguyên.

Do đó dưới Mode Shared, khi luồng được đánh thức, lấy được tài nguyên, nếu phát hiện vẫn còn tài nguyên dư, sẽ thử đánh thức luồng phía sau đi thử lấy tài nguyên. Phương thức `setHeadAndPropagate()` tương ứng như sau:

```java
// AQS
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head;
    // 1. Chuyển node luồng hiện tại ra khỏi hàng đợi chờ.
    setHead(node);
    // 2. Đánh thức node chờ phía sau.
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

Trong phương thức `setHeadAndPropagate()`, việc đánh thức node phía sau cần thỏa mãn điều kiện nhất định, chủ yếu cần thỏa mãn 2 điều kiện:

- `propagate > 0`: `propagate` đại diện số lượng tài nguyên còn lại sau khi lấy tài nguyên, nếu `> 0`, thì có thể đánh thức luồng phía sau đi lấy tài nguyên.
- `h.waitStatus < 0`: Node `h` ở đây là node `head` trước khi thực thi `setHead()`. Khi phán đoán `head.waitStatus` sử dụng `< 0`, chủ yếu để xác định trạng thái node `head` là `SIGNAL` hoặc `PROPAGATE`. Nếu node `head` là `SIGNAL`, thì có thể đánh thức node phía sau; nếu trạng thái node `head` là `PROPAGATE`, cũng có thể đánh thức node phía sau (đây là để giải quyết vấn đề xuất hiện trong kịch bản concurrency, phía sau sẽ giảng chi tiết).

Phán đoán `if` về **đánh thức node chờ phía sau** trong code hơi phức tạp một chút, ở đây giải thích tại sao lại viết như vậy:

```java
if (propagate > 0 || h == null || h.waitStatus < 0 ||
    (h = head) == null || h.waitStatus < 0)
```

- `h == null || h.waitStatus < 0`: `h == null` dùng để phòng ngừa ngoại lệ NullPointer. Trong trường hợp bình thường h sẽ không là `null`, vì trước khi thực thi đến đây, node hiện tại đã gia nhập hàng đợi rồi, hàng đợi không thể chưa được khởi tạo.

  `h.waitStatus < 0` chủ yếu phán đoán trạng thái node `head` có phải là `SIGNAL` hoặc `PROPAGATE` không, trực tiếp sử dụng `< 0` để phán đoán tiện lợi hơn.

- `(h = head) == null || h.waitStatus < 0`: Nếu đến đây chứng tỏ `h.waitStatus < 0` phán đoán trước đó chứng tỏ có concurrency.

  Đồng thời tồn tại luồng khác đang đánh thức node phía sau, đã sửa giá trị của node `head` từ `SIGNAL` thành `0` rồi. Do đó, ở đây lấy lại node `head` mới, node `head` lấy lần này là node luồng hiện tại được thiết lập thông qua `setHead()`, sau đó lần nữa phán đoán trạng thái `waitStatus`.

Nếu điều kiện `if` phán đoán vượt qua, sẽ đi đến phương thức `doReleaseShared()` đánh thức node chờ phía sau, như sau:

```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        // 1. Trong hàng đợi ít nhất cần 1 node luồng chờ.
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            // 2. Nếu trạng thái node head là SIGNAL, thì có thể đánh thức node kế nhiệm.
            if (ws == Node.SIGNAL) {
                // 2.1 Xóa trạng thái SIGNAL của node head, cập nhật thành 0. Biểu thị đã đánh thức node kế nhiệm của node đó rồi.
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                // 2.2 Đánh thức node kế nhiệm
                unparkSuccessor(h);
            }
            // 3. Nếu trạng thái node head là 0, thì cập nhật thành PROPAGATE. Đây là để giải quyết vấn đề tồn tại trong kịch bản concurrency, tiếp theo sẽ giảng chi tiết.
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)
            break;
    }
}
```

Trong phương thức `doReleaseShared()`, sẽ phán đoán trạng thái `waitStatus` của node `head` để quyết định thao tác tiếp theo, có 2 trường hợp:

- Trạng thái node `head` là `SIGNAL`: Thể hiện node `head` tồn tại node kế nhiệm cần đánh thức, do đó thông qua thao tác `CAS` cập nhật trạng thái `SIGNAL` của node `head` thành `0`. Thông qua xóa trạng thái `SIGNAL` để biểu thị đã tiến hành thao tác đánh thức đối với node kế nhiệm của node `head` rồi.
- Trạng thái node `head` là `0`: Thể hiện tồn tại trường hợp concurrency, cần sửa `0` thành `PROPAGATE` để đảm bảo trong kịch bản concurrency có thể đánh thức luồng bình thường.

#### Tại sao cần trạng thái `PROPAGATE`?

Khi `doReleaseShared()` giải phóng tài nguyên, bước 3 không dễ hiểu lắm, tức nếu phát hiện trạng thái node `head` là `0`, thì cập nhật trạng thái node `head` từ `0` thành `PROPAGATE`.

Trong AQS, `PROPAGATE` của Node node chính là để xử lý vấn đề có thể xuất hiện node luồng không thể bị đánh thức trong kịch bản concurrency. `PROPAGATE` chỉ được dùng 1 lần trong phương thức `doReleaseShared()`.

**Tiếp theo thông qua case study phân tích, tại sao cần trạng thái `PROPAGATE`?**

Dưới Mode Shared, chuỗi lời gọi phương thức luồng lấy và giải phóng tài nguyên như sau:

- Chuỗi lời gọi phương thức luồng lấy tài nguyên là: `acquireShared() -> tryAcquireShared() -> luồng bị chặn chờ đánh thức -> tryAcquireShared() -> setHeadAndPropagate() -> if (Số tài nguyên còn lại > 0) || (head.waitStatus < 0) thì đánh thức node phía sau`.
- Chuỗi lời gọi phương thức luồng giải phóng tài nguyên là: `releaseShared() -> tryReleaseShared() -> doReleaseShared()`.

**Nếu khi giải phóng tài nguyên, không sửa trạng thái node `head` từ `0` thành `PROPAGATE`:**

Giả sử tổng cộng có 4 luồng thử lấy tài nguyên theo Mode Shared, tổng cộng có 2 tài nguyên. Ban đầu luồng `T3` và `T4` lấy được tài nguyên, luồng `T1` và `T2` không lấy được, do đó xếp hàng chờ trong hàng đợi.

- Tại thời điểm 1, luồng `T1` và `T2` ở trong hàng đợi chờ, `T3` và `T4` nắm giữ tài nguyên. Lúc này các node trong hàng đợi chờ và trạng thái tương ứng là (trong ngoặc là trạng thái `waitStatus` của node):

  `head(-1) -> T1(-1) -> T2(0)`.

- Tại thời điểm 2, luồng `T3` giải phóng tài nguyên, thông qua phương thức `doReleaseShared()` cập nhật trạng thái node `head` từ `SIGNAL` thành `0`, và đánh thức luồng `T1`, sau đó luồng `T3` thoát.

  Luồng `T1` sau khi được đánh thức, thông qua `tryAcquireShared()` lấy được tài nguyên, nhưng lúc này chưa kịp thực thi `setHeadAndPropagate()` để thiết lập bản thân thành node `head`. Lúc này trạng thái node trong hàng đợi chờ là:

  `head(0) -> T1(-1) -> T2(0)`.

- Tại thời điểm 3, luồng `T4` giải phóng tài nguyên, do lúc này trạng thái node `head` là `0`, nếu trong `doReleaseShared()` khi `ws == 0` không làm gì cả (tức không có trạng thái `PROPAGATE`), vậy `T4` không thể đánh thức node kế nhiệm của `head`, sau đó luồng `T4` thoát.

- Tại thời điểm 4, luồng `T1` tiếp tục thực thi phương thức `setHeadAndPropagate()`. Lưu `head` cũ trước (`waitStatus == 0`), sau đó thực thi `setHead(T1)` thiết lập bản thân thành node `head`.

  Lúc này `propagate == 0`, `head.waitStatus == 0` cũ, hai điều kiện đầu không thỏa mãn. Nhưng do `setHead()` không reset `waitStatus`, `head` mới (tức node `T1` ban đầu) có `waitStatus` vẫn là `-1` (SIGNAL), nên lần phán đoán thứ hai `(h = head).waitStatus < 0` vẫn thành lập, sẽ gọi `doReleaseShared()` đánh thức `T2`.

  Dưới chuỗi thời gian này, việc phán đoán head lần hai quả thực có thể đỡ được. Vậy `PROPAGATE` giải quyết vấn đề gì?

- Tại thời điểm 5 (chuỗi thời gian cực đoan hơn), cân nhắc việc đan xen thế này: Sau khi luồng `T1` thực thi xong `setHead(T1)`, **trước khi** phán đoán `head.waitStatus` mới, `doReleaseShared()` của luồng `T4` vừa vặn thực thi `unparkSuccessor()` đánh thức `T2`, `T2` nhanh chóng lấy được tài nguyên và thực thi `setHead(T2)` thiết lập bản thân làm `head` mới. Do `T2` ban đầu là node đuôi hàng đợi, `waitStatus` của nó là `0`. Khi `T1` khôi phục thực thi đọc `head` mới, đọc được node `T2` (`waitStatus == 0`), lúc này phán đoán lần hai cũng không thể vượt qua rồi.

  Dưới chuỗi thời gian concurrency cực đoan này, `propagate == 0`, `head.waitStatus == 0` cũ, `head.waitStatus == 0` mới, cả ba điều kiện đều không thỏa mãn, `T1` sẽ không gọi `doReleaseShared()`. Nếu lúc này trong hàng đợi còn node chờ phía sau, sẽ dẫn đến tín hiệu đánh thức bị mất.

Bảng thời điểm tương ứng như sau:

| Thời điểm | Luồng T1 | Luồng T2 | Luồng T3 | Luồng T4 | Hàng đợi chờ |
| --- | --- | --- | --- | --- | --- |
| Thời điểm 1 | Hàng đợi chờ | Hàng đợi chờ | Nắm giữ tài nguyên | Nắm giữ tài nguyên | `head(-1) -> T1(-1) -> T2(0)` |
| Thời điểm 2 | (Thực thi) Sau khi được đánh thức, lấy tài nguyên, nhưng chưa kịp thiết lập bản thân thành node `head` | Hàng đợi chờ | (Thực thi) Giải phóng tài nguyên | Nắm giữ tài nguyên | `head(0) -> T1(-1) -> T2(0)` |
| Thời điểm 3 | | Hàng đợi chờ | Đã thoát | (Thực thi) Giải phóng tài nguyên. Nhưng trạng thái node `head` là `0`, không thể đánh thức node kế nhiệm | `head(0) -> T1(-1) -> T2(0)` |
| Thời điểm 4 | (Thực thi) `setHead(T1)` hoàn thành, chưa phán đoán `head.waitStatus` mới | Hàng đợi chờ | Đã thoát | Đã thoát | `head(-1, Node luồng T1) -> T2(0)` |
| Thời điểm 5 | | (Thực thi) Được T4 đánh thức, lấy tài nguyên, thực thi `setHead(T2)` trở thành `head` mới | Đã thoát | (Thực thi) `doReleaseShared()` đánh thức T2 | `head(0, Node luồng T2)` |
| Thời điểm 6 | (Thực thi) Đọc `head` mới là node T2, `waitStatus == 0`, phán đoán lần hai thất bại, không đánh thức node phía sau | Đã lấy tài nguyên | Đã thoát | Đã thoát | `head(0, Node luồng T2)` |

**Nếu khi luồng giải phóng tài nguyên, sửa trạng thái node `head` từ `0` thành `PROPAGATE`, thì có thể giải quyết vấn đề concurrency xuất hiện ở trên, như sau:**

Mẫu chốt của `PROPAGATE` nằm ở chỗ: Nó sửa đổi trạng thái của **`head` cũ**, mà tham chiếu `head` cũ trong phương thức `setHeadAndPropagate()` vừa vào đã được lưu vào biến cục bộ `h` rồi, không bị ảnh hưởng bởi việc sửa đổi concurrency sau đó.

- Tại thời điểm 1~2, giống kịch bản trên:

  Thời điểm 1: `head(-1) -> T1(-1) -> T2(0)`.

  Thời điểm 2: `T3` giải phóng tài nguyên, trạng thái `head` biến thành `0` và đánh thức `T1`.

- Tại thời điểm 3, luồng `T4` giải phóng tài nguyên, do lúc này trạng thái node `head` là `0`, `doReleaseShared()` sẽ cập nhật trạng thái node `head` từ `0` thành `PROPAGATE(-3)`, sau đó luồng `T4` thoát. Lúc này trạng thái node trong hàng đợi chờ là:

  `head(PROPAGATE) -> T1(-1) -> T2(0)`.

- Tại thời điểm 4, luồng `T1` tiếp tục thực thi phương thức `setHeadAndPropagate()`. Lưu `head` cũ vào biến cục bộ `h` trước, lúc này `h.waitStatus == PROPAGATE(-3)`. Sau đó thực thi `setHead(T1)` thiết lập bản thân thành node `head`.

- Tại thời điểm 5, cho dù xảy ra sự đan xen cực đoan giống trước đó (`T4` đánh thức `T2`, `T2` trở thành `head` mới), khi `T1` phán đoán:

  - `propagate > 0` → `0 > 0` → false
  - `h == null` → false
  - `h.waitStatus < 0` → `PROPAGATE(-3) < 0` → **true**!

  Do tham chiếu `h` của `head` cũ vừa vào phương thức đã được lưu lại, không chịu ảnh hưởng của `setHead()` và thao tác concurrency sau đó, nên trạng thái `PROPAGATE` đảm bảo `h.waitStatus < 0` chắc chắn vượt qua. Do đó luồng `T1` sẽ gọi `doReleaseShared()` trong phương thức `setHeadAndPropagate()` để đánh thức node phía sau.

Có trạng thái `PROPAGATE` rồi, là có thể tránh được vấn đề mất tín hiệu đánh thức dưới chuỗi thời gian concurrency cực đoan. Bảng thời điểm tương ứng như sau:

| Thời điểm | Luồng T1 | Luồng T2 | Luồng T3 | Luồng T4 | Hàng đợi chờ |
| --- | --- | --- | --- | --- | --- |
| Thời điểm 1 | Hàng đợi chờ | Hàng đợi chờ | Nắm giữ tài nguyên | Nắm giữ tài nguyên | `head(-1) -> T1(-1) -> T2(0)` |
| Thời điểm 2 | (Thực thi) Sau khi được đánh thức, lấy tài nguyên, nhưng chưa kịp thiết lập bản thân thành node `head` | Hàng đợi chờ | (Thực thi) Giải phóng tài nguyên | Nắm giữ tài nguyên | `head(0) -> T1(-1) -> T2(0)` |
| Thời điểm 3 | Chưa tiếp tục thực thi xuống dưới | Hàng đợi chờ | Đã thoát | (Thực thi) Giải phóng tài nguyên. Lúc này sẽ cập nhật trạng thái node `head` từ `0` thành `PROPAGATE` | `head(PROPAGATE) -> T1(-1) -> T2(0)` |
| Thời điểm 4 | (Thực thi) Lưu `head` cũ (`waitStatus == PROPAGATE`), thực thi `setHead(T1)` | Hàng đợi chờ | Đã thoát | Đã thoát | `head(-1, Node luồng T1) -> T2(0)` |
| Thời điểm 5 | (Thực thi) Phán đoán `h.waitStatus < 0` (`PROPAGATE(-3) < 0`) cũ thành lập, gọi `doReleaseShared()` đánh thức node phía sau | Hàng đợi chờ | Đã thoát | Đã thoát | `head(0, Node luồng T1) -> T2(0)` |
| Thời điểm 6 | Đã thoát | (Thực thi) Luồng `T2` sau khi được đánh thức, lấy được tài nguyên, và thiết lập bản thân thành node `head` | Đã thoát | Đã thoát | `head(0, Node luồng T2)` |

Tóm tắt đơn giản: Trạng thái `PROPAGATE` và việc phán đoán head lần hai trong `setHeadAndPropagate()` là **bảo hiểm kép** của cùng một bug fix trong JDK 7 ([JDK-6801020](https://bugs.openjdk.org/browse/JDK-6801020)). `PROPAGATE` thông qua sửa đổi trạng thái `head` cũ để cung cấp sự bảo đảm đáng tin cậy hơn, vì tham chiếu `head` cũ vừa vào phương thức đã được lưu vào biến cục bộ, không bị thao tác `setHead()` concurrency thay thế.

### Phân tích mã nguồn giải phóng tài nguyên AQS (Mode Shared)

Phương thức cổng vào giải phóng tài nguyên theo Mode Shared trong AQS là `releaseShared()`, code như sau:

```java
// AQS
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

Trong đó phương thức `tryReleaseShared()` là template method do AQS cung cấp, ở đây tương tự lấy `Semaphore` làm ví dụ giải thích, như sau:

```java
// Semaphore
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
}
```

Trong phương thức `tryReleaseShared()` do `Semaphore` triển khai, sẽ ở trong vòng lặp vô hạn không ngừng thử giải phóng tài nguyên, tức thông qua thao tác `CAS` để cập nhật giá trị `state`.

Nếu cập nhật thành công, thì chứng tỏ giải phóng tài nguyên thành công, sẽ đi vào phương thức `doReleaseShared()`.

Phương thức `doReleaseShared()` ở phần lấy tài nguyên (Mode Shared) phần trước đã tiến hành phân tích mã nguồn chi tiết rồi, ở đây không lặp lại nữa.

### Cơ chế hoạt động của Hàng đợi điều kiện (Condition Queue)

Phía trước trong bảng trạng thái `waitStatus` có đề cập qua trạng thái `CONDITION` (giá trị -2), biểu thị node đang chờ trong Condition Queue (Hàng đợi điều kiện). Ở đây giải thích hệ thống cơ chế hoạt động của Condition Queue.

#### Condition là gì?

`Condition` là interface được định nghĩa trong gói `java.util.concurrent.locks`, nó cung cấp cơ chế luồng chờ/thông báo tương tự như `Object.wait()` / `Object.notify()`, nhưng tính năng mạnh mẽ và linh hoạt hơn. `Condition` bắt buộc phải phối hợp sử dụng với `Lock`, giống như `wait/notify` bắt buộc phải phối hợp sử dụng với `synchronized` vậy.

So sánh với `wait/notify` của `Object`, ưu thế chính của `Condition` nằm ở chỗ:

- **Hỗ trợ nhiều hàng đợi chờ**: Một `Lock` có thể tạo nhiều thể hiện `Condition`, các luồng khác nhau có thể chờ trên các điều kiện khác nhau, triển khai sự phối hợp luồng tinh tế hơn. Còn `synchronized` chỉ có 1 hàng đợi chờ.
- **Hỗ trợ chờ không phản hồi ngắt**: `Condition` cung cấp phương thức `awaitUninterruptibly()`.
- **Hỗ trợ chờ timeout**: `Condition` cung cấp các phương thức `awaitNanos(long)` và `await(long, TimeUnit)`, có thể cài đặt thời hạn chờ.

#### Hai loại hàng đợi trong AQS

Bên trong AQS thực tế bảo trì **hai loại hàng đợi**:

1. **Hàng đợi đồng bộ (Hàng đợi biến thể CLH)**: Chính là hàng đợi hai chiều đã phân tích chi tiết ở trên, dùng để chứa các node luồng lấy tài nguyên thất bại mà phải chờ đợi.
2. **Hàng đợi điều kiện (Condition Queue)**: Là một danh sách liên kết một chiều, dùng để chứa các node luồng do gọi phương thức `Condition.await()` mà phải chờ đợi. Mỗi thể hiện `Condition` bảo trì một hàng đợi điều kiện độc lập.

Các node trong hàng đợi điều kiện sử dụng con trỏ `nextWaiter` của `Node` để liên kết node tiếp theo, tạo thành danh sách liên kết một chiều. Node đầu của hàng đợi điều kiện là `firstWaiter`, node đuôi là `lastWaiter`.

#### Quy trình hoạt động cốt lõi của Condition

Inner class `ConditionObject` của AQS triển khai interface `Condition`, các phương thức cốt lõi của nó là `await()` và `signal()`.

**Quy trình hoạt động của phương thức `await()`:**

1. Đóng gói luồng hiện tại thành node `Node` (`waitStatus` thiết lập thành `CONDITION`), thêm vào đuôi hàng đợi điều kiện.
2. Giải phóng hoàn toàn lock do luồng hiện tại nắm giữ (tức đặt giá trị `state` về 0), và lưu giá trị `state` trước khi giải phóng.
3. Bị chặn luồng hiện tại, chờ được `signal()` đánh thức hoặc bị ngắt.
4. Sau khi được đánh thức, đi vào hàng đợi đồng bộ lần nữa thông qua `acquireQueued()` để tranh chấp lock, và khôi phục giá trị `state` đã lưu trước đó (số lần reentrant).

**Quy trình hoạt động của phương thức `signal()`:**

1. Kiểm tra luồng gọi `signal()` có nắm giữ lock không (không nắm giữ thì ném ra `IllegalMonitorStateException`).
2. Xóa node chờ đầu tiên trong hàng đợi điều kiện khỏi hàng đợi điều kiện.
3. Sửa `waitStatus` của node đó từ `CONDITION` thành `0`, và đưa nó vào đuôi hàng đợi đồng bộ thông qua phương thức `enq()`.
4. Nếu trạng thái node tiền nhiệm trong hàng đợi đồng bộ bất thường (`CANCELLED`) hoặc CAS thiết lập trạng thái node tiền nhiệm thành `SIGNAL` thất bại, thì trực tiếp đánh thức luồng đó.

Phương thức `signalAll()` tương tự `signal()`, điểm khác biệt nằm ở chỗ nó sẽ chuyển **tất cả** các node trong hàng đợi điều kiện sang hàng đợi đồng bộ.

Mã nguồn ví dụ dưới đây hiển thị cách dùng điển hình của `Condition` — Triển khai một hàng đợi chặn hữu hạn đơn giản:

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class SimpleBlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    // Hai hàng đợi điều kiện khác nhau: Lần lượt dùng cho "Hàng đợi chưa đầy" và "Hàng đợi không rỗng"
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public SimpleBlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    /**
     * Thêm phần tử vào hàng đợi, nếu hàng đợi đã đầy thì chờ.
     */
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            // Khi hàng đợi đầy, chờ trên điều kiện notFull
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.offer(item);
            // Sau khi thêm phần tử, thông báo luồng consumer đang chờ trên điều kiện notEmpty
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    /**
     * Lấy phần tử từ hàng đợi, nếu hàng đợi rỗng thì chờ.
     */
    public T take() throws InterruptedException {
        lock.lock();
        try {
            // Khi hàng đợi rỗng, chờ trên điều kiện notEmpty
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            T item = queue.poll();
            // Sau khi lấy phần tử, thông báo luồng producer đang chờ trên điều kiện notFull
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        SimpleBlockingQueue<Integer> blockingQueue = new SimpleBlockingQueue<>(5);

        // Luồng Producer
        Thread producer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    blockingQueue.put(i);
                    System.out.println("Sản xuất: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Producer");

        // Luồng Consumer
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    int item = blockingQueue.take();
                    System.out.println("Tiêu thụ: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer");

        producer.start();
        consumer.start();
    }
}
```

Trong ví dụ trên, `notFull` và `notEmpty` là hai thể hiện `Condition` độc lập, lần lượt bảo trì hàng đợi điều kiện riêng của mình. Producer khi hàng đợi đầy thì chờ trên `notFull`, Consumer khi hàng đợi rỗng thì chờ trên `notEmpty`. Thiết kế tách biệt điều kiện chờ này tránh được việc đánh thức luồng không cần thiết, hiệu quả hơn `synchronized` + `wait/notifyAll`.

#### Phân tích mã nguồn cốt lõi `await()`

```java
// Inner class ConditionObject của AQS
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    // 1. Đóng gói luồng hiện tại thành Node node, gia nhập hàng đợi điều kiện
    Node node = addConditionWaiter();
    // 2. Giải phóng hoàn toàn lock, và lưu giá trị state trước khi giải phóng
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    // 3. Nếu node không ở trong hàng đợi đồng bộ, thì làm bị chặn luồng hiện tại
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // 4. Sau khi được đánh thức, đi vào hàng đợi đồng bộ lần nữa tranh chấp lock
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

Trong phương thức `await()` có hai thao tác mấu chốt:

- `fullyRelease(node)`: Giải phóng hoàn toàn lock (chứ không chỉ giải phóng 1 lần), như vậy cho dù luồng reentrant nhiều lần lock, cũng có thể cho luồng khác lấy được lock trong thời gian chờ đợi. Sau khi được đánh thức sẽ thông qua `acquireQueued(node, savedState)` khôi phục lại số lần reentrant trước đó.
- `isOnSyncQueue(node)`: Phán đoán node có phải đã được chuyển sang hàng đợi đồng bộ hay chưa. Khi luồng khác gọi `signal()`, node sẽ từ hàng đợi điều kiện chuyển sang hàng đợi đồng bộ, lúc này `isOnSyncQueue()` trả về `true`, luồng thoát khỏi vòng lặp `while`, bắt đầu tranh chấp lock.

### Phân tích sự chênh lệch hiệu năng giữa Fair Lock và Non-fair Lock

Trong phần phân tích mã nguồn trước, đã lấy Non-fair lock của `ReentrantLock` làm ví dụ để giải thích việc triển khai `tryAcquire()`. Thực tế `ReentrantLock` đồng thời hỗ trợ hai mode Fair lock và Non-fair lock. Ở đây phân tích sâu sự khác biệt triển khai giữa hai cái và ảnh hưởng của nó đến hiệu năng.

#### Sự khác biệt ở tầng mã nguồn

`ReentrantLock` mặc định sử dụng Non-fair lock, thông qua tham số constructor có thể chuyển sang Fair lock:

```java
// Non-fair lock (Mặc định)
ReentrantLock unfairLock = new ReentrantLock();
// Fair lock
ReentrantLock fairLock = new ReentrantLock(true);
```

Sự khác biệt cốt lõi giữa hai cái nằm ở cách triển khai phương thức `tryAcquire()`. `nonfairTryAcquire()` của Non-fair lock trước đó đã phân tích qua, dưới đây xem triển khai của Fair lock:

```java
// ReentrantLock.FairSync
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // Sự khác biệt mấu chốt: Gọi hasQueuedPredecessors() trước để phán đoán trong hàng đợi đồng bộ có luồng nào chờ lâu hơn không
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
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
    return false;
}
```

**Điểm khác biệt duy nhất** chính là Fair lock trước khi CAS sửa `state` có thêm một phán đoán `hasQueuedPredecessors()`:

```java
// AQS
public final boolean hasQueuedPredecessors() {
    Node t = tail;
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

Phương thức này dùng để phán đoán trước luồng hiện tại có luồng nào khác đang xếp hàng không. Nếu có, thì luồng hiện tại không thể trực tiếp lấy lock, bắt buộc phải xếp hàng chờ, từ đó đảm bảo tính công bằng **FIFO**.

Còn Non-fair lock không có phán đoán này, khi lock vừa vặn được giải phóng, luồng mới đến có thể trực tiếp cướp lấy lock thông qua CAS, cho dù trong hàng đợi đồng bộ đã có luồng khác đang chờ.

#### So sánh sự chênh lệch hiệu năng

| Chiều so sánh | Non-fair Lock (Mặc định) | Fair Lock |
| --- | --- | --- |
| **Throughput (Năng suất)** | Cao hơn. Luồng mới có cơ hội trực tiếp lấy lock, giảm chuyển đổi ngữ cảnh luồng | Thấp hơn. Tất cả các luồng bắt buộc phải xếp hàng, làm tăng chi phí chuyển đổi ngữ cảnh |
| **Bỏ đói luồng (Starvation)** | Có thể xảy ra. Trong trường hợp cực đoan một số luồng thời gian dài không lấy được lock | Thường không dễ xảy ra, nhưng Fair lock không đảm bảo điều phối luồng của hệ điều hành |
| **Chuyển đổi ngữ cảnh** | Ít hơn. Luồng nắm giữ lock sau khi giải phóng lock, luồng mới đến có thể trực tiếp lấy lock, không cần đánh thức luồng trong hàng đợi | Nhiều hơn. Mỗi lần giải phóng lock đều cần đánh thức luồng tiếp theo trong hàng đợi |
| **Kịch bản phù hợp** | Hầu hết các kịch bản (Có yêu cầu tương đối cao về thời gian phản hồi và throughput) | Các kịch bản có yêu cầu nghiêm ngặt về tính công bằng (như phân bổ tài nguyên, điều phối task) |

**Tại sao hiệu năng của Non-fair lock thường tốt hơn?**

Lý do mấu chốt nằm ở chỗ **giảm số lần chuyển đổi ngữ cảnh (context switch) của luồng**. Khi Luồng A nắm giữ lock giải phóng lock:

- **Non-fair lock**: Lúc này nếu vừa vặn có Luồng B đang thử lấy lock (chưa vào hàng đợi đồng bộ), Luồng B có thể trực tiếp lấy được lock thông qua CAS và thực thi ngay lập tức, tiết kiệm được chi phí đánh thức luồng trong hàng đợi. Còn luồng chờ trong hàng đợi sau khi được đánh thức phát hiện lock bị chiếm dụng, sẽ bị chặn lại, mặc dù nhìn qua có vẻ "lãng phí" một lần đánh thức, nhưng tổng thể lại làm giảm số lần chuyển đổi luồng.
- **Fair lock**: Luồng B bắt buộc phải xếp vào đuôi hàng đợi, sau đó đánh thức luồng ở đầu hàng đợi. Từ lúc luồng được đánh thức đến khi thực sự bắt đầu thực thi, tồn tại một khoảng **độ trễ điều phối (scheduling latency)** (trạng thái luồng chuyển từ blocked sang running), trong khoảng độ trễ này lock ở trạng thái rảnh rỗi, làm giảm tỷ lệ tận dụng lock.

Doug Lea trong tài liệu của `ReentrantLock` đã chỉ ra: Chương trình sử dụng Fair lock trong môi trường đa luồng có tổng throughput thường thấp hơn chương trình sử dụng Non-fair lock (tức chậm hơn), do đó `ReentrantLock` mặc định sử dụng Non-fair mode. Nhưng trong kịch bản cần đảm bảo thứ tự xử lý request hoặc tránh bỏ đói luồng (như phân bổ connection pool), Fair lock là lựa chọn tốt hơn.

Dưới đây thông qua ví dụ code để minh họa sự khác biệt hành vi giữa Fair lock và Non-fair lock:

```java
import java.util.concurrent.locks.ReentrantLock;

public class FairVsUnfairLockDemo {
    // Lần lượt test Fair lock và Non-fair lock
    private static void testLock(ReentrantLock lock, String lockType) {
        System.out.println("=== " + lockType + " ===");
        Runnable task = () -> {
            for (int i = 0; i < 2; i++) {
                lock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + " lấy được lock");
                } finally {
                    lock.unlock();
                }
            }
        };

        Thread[] threads = new Thread[5];
        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(task, lockType + "-Thread-" + i);
        }
        for (Thread t : threads) {
            t.start();
        }
        for (Thread t : threads) {
            try { t.join(); } catch (InterruptedException e) { }
        }
        System.out.println();
    }

    public static void main(String[] args) {
        // Non-fair lock: Cùng một luồng có thể liên tiếp nhiều lần lấy được lock
        testLock(new ReentrantLock(false), "Non-fair Lock");

        // Fair lock: Khi tồn tại tranh chấp có xu hướng cho luồng chờ lâu hơn lấy lock trước
        testLock(new ReentrantLock(true), "Fair Lock");
    }
}
```

Chạy code trên thường có thể quan sát thấy: Dưới mode Non-fair lock, cùng một luồng dễ liên tiếp nhiều lần lấy được lock hơn (vì sau khi nó giải phóng lock lập tức lại đi tranh chấp, có cơ hội cướp được lock trước khi luồng trong hàng đợi được đánh thức); Fair lock khi tồn tại người chờ có xu hướng phân bổ lock theo thứ tự hàng đợi. Tuy nhiên, tính công bằng không tương đương với việc hệ điều hành điều phối công bằng, nếu luồng khác chưa chạy đến điểm chờ, cùng một luồng vẫn có thể liên tiếp lấy được lock.

## Các lớp công cụ đồng bộ thường gặp

### Semaphore (Tín hiệu số)

#### Giới thiệu

`synchronized` và `ReentrantLock` đều là chỉ cho phép một luồng truy cập một tài nguyên nào đó tại một thời điểm, còn `Semaphore` (Tín hiệu số) có thể dùng để kiểm soát số lượng luồng đồng thời truy cập tài nguyên đặc định.

Cách sử dụng `Semaphore` đơn giản, ở đây chúng ta giả sử có `N (N > 5)` luồng đến lấy tài nguyên dùng chung trong `Semaphore`, đoạn code dưới đây thể hiện tại cùng một thời điểm trong N luồng chỉ có 5 luồng có thể lấy được tài nguyên dùng chung, các luồng khác đều sẽ bị chặn, chỉ có luồng lấy được tài nguyên dùng chung mới có thể thực thi. Chờ đến khi có luồng giải phóng tài nguyên dùng chung, các luồng bị chặn khác mới có thể lấy được.

```java
// Số lượng tài nguyên dùng chung ban đầu
final Semaphore semaphore = new Semaphore(5);
// Lấy 1 permit (giấy phép)
semaphore.acquire();
// Giải phóng 1 permit
semaphore.release();
```

Khi số lượng tài nguyên ban đầu là 1, `Semaphore` thoái hóa thành Lock loại trừ.

`Semaphore` có hai mode:

- **Fair mode:** Khi tồn tại tranh chấp, phương thức `acquire` dạng bị chặn sẽ xếp hàng bên trong chọn luồng theo FIFO; điều này không tương đương với việc sắp xếp nghiêm ngặt theo wall-clock time của lời gọi phương thức. Ngoài ra, `tryAcquire()` không tham số không tuân thủ cài đặt công bằng, vẫn có thể chen hàng thành công;
- **Non-fair mode:** Dạng tranh đoạt (preemptive).

Hai constructor tương ứng của `Semaphore` như sau:

```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

**Hai constructor này đều bắt buộc phải cung cấp số lượng permit, constructor thứ hai có thể chỉ định là Fair mode hay Non-fair mode, mặc định Non-fair mode.**

`Semaphore` thường dùng cho các kịch bản mà tài nguyên có giới hạn rõ ràng về số lượng truy cập như Rate Limiting (giới hạn trong single instance, trong dự án thực tế khuyến nghị sử dụng Redis + Lua để làm Rate Limiting).

#### Nguyên lý

`Semaphore` là một triển khai của Shared lock, nó mặc định dựng giá trị `state` của AQS là `permits`, bạn có thể hiểu giá trị của `permits` là số lượng giấy phép, chỉ có luồng lấy được giấy phép mới có thể thực thi.

Lấy phương thức `acquire` không tham số làm ví dụ, gọi `semaphore.acquire()`, luồng thử lấy giấy phép, nếu `state > 0`, thì thể hiện có thể lấy thành công, nếu `state <= 0`, thì thể hiện số lượng giấy phép không đủ, lấy thất bại.

Nếu có thể lấy thành công (`state > 0`), sẽ thử sử dụng thao tác CAS để sửa giá trị của `state`: `state = state - 1`. Nếu lấy thất bại thì sẽ tạo một Node node gia nhập hàng đợi chờ, treo (suspend) luồng hiện tại.

```java
// Lấy 1 giấy phép
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

// Lấy một hoặc nhiều giấy phép
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}
```

Phương thức `acquireSharedInterruptibly` là triển khai mặc định trong `AbstractQueuedSynchronizer`.

```java
// Lấy giấy phép dưới Mode Shared, lấy thành công thì trả về, thất bại thì gia nhập hàng đợi chờ, treo luồng
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // Thử lấy giấy phép, arg là số lượng giấy phép lấy, khi lấy thất bại, thì tạo một node gia nhập hàng đợi chờ, treo luồng hiện tại.
    if (tryAcquireShared(arg) < 0)
      doAcquireSharedInterruptibly(arg);
}
```

Ở đây lại lấy Non-fair mode (`NonfairSync`) làm ví dụ, xem cách triển khai phương thức `tryAcquireShared`.

```java
// Thử lấy tài nguyên dưới Mode Shared (Tài nguyên trong Semaphore chính là giấy phép):
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

// Lấy giấy phép theo Non-fair Mode Shared
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        // Số lượng giấy phép khả dụng hiện tại
        int available = getState();
        /*
         * Thử lấy giấy phép, khi số lượng giấy phép khả dụng hiện tại nhỏ hơn hoặc bằng 0, trả về số âm, biểu thị lấy thất bại,
         * Khi số lượng giấy phép khả dụng hiện tại lớn hơn 0 mới có thể lấy thành công, CAS thất bại sẽ lặp vòng lấy lại giá trị mới nhất để thử lấy
         */
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

Lấy phương thức `release` không tham số làm ví dụ, gọi `semaphore.release()`, luồng thử giải phóng giấy phép, và sử dụng thao tác CAS để sửa giá trị của `state`: `state = state + 1`. Sau khi giải phóng giấy phép thành công, đồng thời sẽ đánh thức một luồng trong hàng đợi chờ. Luồng được đánh thức sẽ thử lần nữa để sửa giá trị của `state`: `state = state - 1`, nếu `state > 0` thì lấy token thành công, nếu không lại gia nhập hàng đợi chờ, treo luồng.

```java
// Giải phóng một giấy phép
public void release() {
    sync.releaseShared(1);
}

// Giải phóng một hoặc nhiều giấy phép
public void release(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.releaseShared(permits);
}
```

Phương thức `releaseShared` là triển khai mặc định trong `AbstractQueuedSynchronizer`.

```java
// Giải phóng Shared lock
// Nếu tryReleaseShared trả về true, liền đánh thức một hoặc nhiều luồng trong hàng đợi chờ.
public final boolean releaseShared(int arg) {
    // Giải phóng Shared lock
    if (tryReleaseShared(arg)) {
      // Giải phóng node chờ phía sau của node hiện tại
      doReleaseShared();
      return true;
    }
    return false;
}
```

Phương thức `tryReleaseShared` là phương thức được inner class `Sync` của `Semaphore` override, triển khai mặc định trong `AbstractQueuedSynchronizer` chỉ ném ra ngoại lệ `UnsupportedOperationException`.

```java
// Phương thức được override trong inner class Sync
// Thử giải phóng tài nguyên
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        // Giấy phép khả dụng + 1
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
         // CAS sửa giá trị state
        if (compareAndSetState(current, next))
            return true;
    }
}
```

Có thể thấy, mấy phương thức đề cập ở trên bên dưới về cơ bản đều được triển khai thông qua bộ đồng bộ hóa `sync`. `Sync` là inner class của `Semaphore`, kế thừa `AbstractQueuedSynchronizer`, override một số phương thức trong đó. Hơn nữa, Sync tương ứng còn có hai lớp con `NonfairSync` (tương ứng Non-fair mode) và `FairSync` (tương ứng Fair mode).

```java
private static final class Sync extends AbstractQueuedSynchronizer {
  // ...
}
static final class NonfairSync extends Sync {
  // ...
}
static final class FairSync extends Sync {
  // ...
}
```

#### Thực chiến

```java
public class SemaphoreExample {
  // Số lượng request
  private static final int threadCount = 550;

  public static void main(String[] args) throws InterruptedException {
    // Tạo một đối tượng ThreadPool có số lượng luồng cố định (Nếu số luồng ThreadPool ở đây đưa quá ít bạn sẽ phát hiện thực thi rất chậm)
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    // Số lượng giấy phép ban đầu
    final Semaphore semaphore = new Semaphore(20);

    for (int i = 0; i < threadCount; i++) {
      final int threadnum = i;
      threadPool.execute(() -> {// Vận dụng biểu thức Lambda
        try {
          semaphore.acquire();// Lấy một permit, nên số lượng luồng có thể chạy là 20/1=20
          test(threadnum);
          semaphore.release();// Giải phóng một permit
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }

      });
    }
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);// Giả lập thao tác tốn thời gian của request
    System.out.println("threadnum:" + threadnum);
    Thread.sleep(1000);// Giả lập thao tác tốn thời gian của request
  }
}
```

Thực thi phương thức `acquire()` bị chặn, cho đến khi có một giấy phép có thể lấy được rồi lấy đi một giấy phép; Mỗi phương thức `release` tăng thêm một giấy phép, điều này có thể giải phóng một phương thức `acquire()` bị chặn. Tuy nhiên, thực ra không có đối tượng giấy phép thực tế nào cả, `Semaphore` chỉ duy trì một số lượng giấy phép có thể lấy được. `Semaphore` thường xuyên dùng để giới hạn số luồng lấy một loại tài nguyên nào đó.

Đương nhiên một lần cũng có thể lấy và giải phóng nhiều permit, nhưng nhìn chung không có lý do cần làm vậy:

```java
semaphore.acquire(5);// Lấy 5 permit, nên số luồng có thể chạy là 20/5=4
test(threadnum);
semaphore.release(5);// Giải phóng 5 permit
```

Ngoài phương thức `acquire()`, một phương thức thường dùng khác tương ứng với nó là `tryAcquire()`, phương thức này nếu không lấy được permit sẽ trả về false ngay lập tức.

[Nội dung bổ sung issue645](https://github.com/Snailclimb/JavaGuide/issues/645):

> `Semaphore` dựa trên AQS để triển khai, dùng để kiểm soát số lượng luồng truy cập concurrent, nhưng nó có điểm khác biệt với khái niệm Shared lock. Constructor của `Semaphore` sử dụng tham số `permits` để khởi tạo biến `state` của AQS, biến này biểu thị số lượng permit khả dụng. Khi luồng gọi phương thức `acquire()` thử lấy permit, `state` sẽ giảm 1 một cách atomic. Nếu `state` sau khi giảm 1 lớn hơn hoặc bằng 0, thì `acquire()` thành công trả về, luồng có thể tiếp tục thực thi. Nếu `state` sau khi giảm 1 nhỏ hơn 0, biểu thị số lượng luồng truy cập concurrent hiện tại đã đạt giới hạn của `permits`, luồng đó sẽ được đưa vào hàng đợi chờ của AQS và bị chặn, **chứ không phải tự xoay chờ đợi**. Khi luồng khác hoàn thành task và gọi phương thức `release()`, `state` sẽ tăng 1 một cách atomic. Thao tác `release()` sẽ đánh thức một hoặc nhiều luồng bị chặn trong hàng đợi chờ của AQS. Các luồng được đánh thức này sẽ lần nữa thử thao tác `acquire()`, tranh chấp lấy permit khả dụng. Do đó, `Semaphore` thông qua việc kiểm soát số lượng permit để giới hạn số luồng truy cập concurrent, chứ không phải thông qua cơ chế tự xoay và Shared lock.

### CountDownLatch (Bộ đếm đếm ngược)

#### Giới thiệu

`CountDownLatch` cho phép `count` luồng bị chặn tại một nơi, cho đến khi task của tất cả các luồng đều thực thi hoàn tất.

`CountDownLatch` là loại dùng 1 lần, giá trị của counter chỉ có thể khởi tạo 1 lần trong constructor, về sau không có bất kỳ cơ chế nào để cài đặt lại giá trị cho nó nữa, khi `CountDownLatch` sử dụng xong, nó không thể được sử dụng lại nữa.

#### Nguyên lý

`CountDownLatch` là một triển khai của Shared lock, nó mặc định dựng giá trị `state` của AQS là `count`. Điều này chúng ta thông qua constructor của `CountDownLatch` là có thể nhìn ra.

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}

private static final class Sync extends AbstractQueuedSynchronizer {
    Sync(int count) {
        setState(count);
    }
  //...
}
```

Khi luồng gọi `countDown()`, thực ra đã sử dụng `tryReleaseShared` để giảm `state`:

```java
public void countDown() {
    // Sync là inner class của CountDownLatch , kế thừa AbstractQueuedSynchronizer
    sync.releaseShared(1);
}
```

Phương thức `releaseShared` là triển khai mặc định trong `AbstractQueuedSynchronizer`.

```java
// Giải phóng Shared lock
// Nếu tryReleaseShared trả về true, liền đánh thức một hoặc nhiều luồng trong hàng đợi chờ.
public final boolean releaseShared(int arg) {
    // Giải phóng Shared lock
    if (tryReleaseShared(arg)) {
      // Giải phóng node chờ phía sau của node hiện tại
      doReleaseShared();
      return true;
    }
    return false;
}
```

Phương thức `tryReleaseShared` là phương thức được inner class `Sync` của `CountDownLatch` override, triển khai mặc định trong `AbstractQueuedSynchronizer` chỉ ném ra ngoại lệ `UnsupportedOperationException`.

```java
// Tiến hành giảm dần đối với state, cho đến khi state biến thành 0;
// Chỉ khi count giảm dần đến 0, countDown mới trả về true
protected boolean tryReleaseShared(int releases) {
    // Tự xoay kiểm tra state có phải là 0 không
    for (;;) {
        int c = getState();
        // Nếu state đã là 0 rồi, trả về false trực tiếp
        if (c == 0)
            return false;
        // Tiến hành giảm dần đối với state
        int nextc = c-1;
        // Thao tác CAS cập nhật giá trị của state
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

Lấy phương thức `await` không tham số làm ví dụ, khi gọi `await()`, nếu `state` không bằng 0, vậy chứng tỏ task vẫn chưa thực thi xong, `await()` sẽ liên tục bị chặn, tức là các câu lệnh sau `await()` sẽ không được thực thi (luồng `main` được thêm vào hàng đợi chờ tức hàng đợi biến thể CLH). Sau đó, `CountDownLatch` sẽ tự xoay CAS phán đoán `state == 0`, nếu `state == 0`, sẽ giải phóng tất cả các luồng chờ đợi, các câu lệnh sau phương thức `await()` được thực thi.

```java
// Chờ đợi (Cũng có thể gọi là cài khóa)
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
// Chờ đợi có thời gian timeout
public boolean await(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

Phương thức `acquireSharedInterruptibly` là triển khai mặc định trong `AbstractQueuedSynchronizer`.

```java
// Thử lấy lock, lấy thành công thì trả về, thất bại thì gia nhập hàng đợi chờ, treo luồng
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // Thử lấy lock, lấy thành công thì trả về
    if (tryAcquireShared(arg) < 0)
      // Lấy thất bại gia nhập hàng đợi chờ, treo luồng
      doAcquireSharedInterruptibly(arg);
}
```

Phương thức `tryAcquireShared` là phương thức được inner class `Sync` của `CountDownLatch` override, tác dụng của nó chính là phán đoán giá trị của `state` có phải là 0 không, nếu phải thì trả về 1, ngược lại trả về -1.

```java
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

#### Thực chiến

**Hai cách dùng điển hình của CountDownLatch**:

1. Một luồng nào đó trước khi bắt đầu chạy chờ n luồng thực thi hoàn tất: Khởi tạo counter của `CountDownLatch` thành n (`new CountDownLatch(n)`), mỗi khi một luồng task thực thi hoàn tất, liền giảm counter đi 1 (`countdownlatch.countDown()`), khi giá trị counter biến thành 0, luồng đang `await()` trên `CountDownLatch` sẽ được đánh thức. Một kịch bản ứng dụng điển hình chính là khi khởi động một service, luồng chính cần chờ nhiều component load xong, sau đó mới tiếp tục thực thi.
2. Triển khai tính song song tối đa khi nhiều luồng bắt đầu thực thi task: Chú ý là tính song song (parallelism), không phải concurrency, nhấn mạnh là nhiều luồng tại một thời điểm nào đó đồng thời bắt đầu thực thi. Tương tự như chạy đua, đưa nhiều luồng đến vạch xuất phát, chờ tiếng súng lệnh vang lên, sau đó đồng thời chạy. Cách làm là khởi tạo một đối tượng `CountDownLatch` dùng chung, khởi tạo counter của nó thành 1 (`new CountDownLatch(1)`), nhiều luồng trước khi bắt đầu thực thi task đầu tiên `countdownlatch.await()`, khi luồng chính gọi `countDown()`, counter biến thành 0, nhiều luồng đồng thời được đánh thức.

**Ví dụ code CountDownLatch**:

```java
public class CountDownLatchExample {
  // Số lượng request
  private static final int THREAD_COUNT = 550;

  public static void main(String[] args) throws InterruptedException {
    // Tạo một đối tượng ThreadPool có số lượng luồng cố định (Nếu số luồng ThreadPool ở đây đưa quá ít bạn sẽ phát hiện thực thi rất chậm)
    // Chỉ dùng cho test, kịch bản thực tế xin gán giá trị tham số ThreadPool thủ công
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    final CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);
    for (int i = 0; i < THREAD_COUNT; i++) {
      final int threadNum = i;
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          e.printStackTrace();
        } finally {
          // Biểu thị một request đã được hoàn thành
          countDownLatch.countDown();
        }

      });
    }
    countDownLatch.await();
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);
    System.out.println("threadNum:" + threadnum);
    Thread.sleep(1000);
  }
}
```

Trong đoạn code trên, chúng ta định nghĩa số lượng request là 550, khi 550 request này được xử lý hoàn thành, mới thực thi `System.out.println("finish");`.

Tương tác đầu tiên với `CountDownLatch` là luồng chính chờ các luồng khác. Luồng chính bắt buộc phải gọi phương thức `CountDownLatch.await()` ngay sau khi khởi động các luồng khác. Như vậy thao tác của luồng chính sẽ bị chặn ở phương thức này, cho đến khi các luồng khác hoàn thành task của mình.

N luồng khác bắt buộc phải tham chiếu đối tượng latch, vì họ cần thông báo đối tượng `CountDownLatch`, họ đã hoàn thành task của mình rồi. Cơ chế thông báo này được hoàn thành thông qua phương thức `CountDownLatch.countDown()`; mỗi khi gọi phương thức này một lần, giá trị count được khởi tạo trong constructor giảm đi 1. Nên khi N luồng đều đã gọi phương thức này, giá trị của count bằng 0, sau đó luồng chính có thể thông qua phương thức `await()`, khôi phục thực thi task của mình.

Nói thêm một câu: Phương thức `await()` của `CountDownLatch` nếu sử dụng không đúng cách rất dễ tạo ra deadlock, ví dụ vòng lặp for trong code trên của chúng ta sửa thành:

```java
for (int i = 0; i < threadCount-1; i++) {
.......
}
```

Như vậy dẫn đến giá trị của `count` không cách nào bằng 0, sau đó sẽ dẫn đến chờ đợi mãi.

### CyclicBarrier (Rào chắn vòng)

#### Giới thiệu

`CyclicBarrier` và `CountDownLatch` rất tương đồng, nó cũng có thể triển khai việc chờ đợi kỹ thuật giữa các luồng, nhưng tính năng của nó phức tạp và mạnh mẽ hơn `CountDownLatch`. Kịch bản ứng dụng chính tương tự `CountDownLatch`.

> Triển khai của `CountDownLatch` dựa trên AQS, còn `CyclicBarrier` dựa trên `ReentrantLock` (`ReentrantLock` cũng thuộc bộ đồng bộ hóa AQS) và `Condition`.

Ý nghĩa trên mặt chữ của `CyclicBarrier` là Barrier (Rào chắn) có thể tái sử dụng theo vòng (Cyclic). Việc nó cần làm là: Cho một nhóm luồng khi đến một barrier (cũng có thể gọi là điểm đồng bộ) thì bị chặn, cho đến khi luồng cuối cùng đến barrier, barrier mới mở cửa, tất cả các luồng bị barrier chặn mới tiếp tục làm việc.

#### Nguyên lý

Bên trong `CyclicBarrier` thông qua một biến `count` làm counter, giá trị ban đầu của `count` là giá trị khởi tạo của thuộc tính `parties`, mỗi khi một luồng đến rào chắn ở đây rồi, thì giảm counter đi 1. Nếu giá trị count bằng 0 rồi, biểu thị đây là luồng cuối cùng của thế hệ này đến rào chắn, thì thử thực thi task chúng ta đưa vào trong constructor.

```java
// Số luồng chặn mỗi lần
private final int parties;
// Bộ đếm
private int count;
```

Dưới đây chúng ta kết hợp mã nguồn để xem đơn giản.

1. Constructor mặc định của `CyclicBarrier` là `CyclicBarrier(int parties)`, tham số của nó biểu thị số lượng luồng rào chắn chặn, mỗi luồng gọi phương thức `await()` nói với `CyclicBarrier` tôi đã đến rào chắn rồi, sau đó luồng hiện tại bị chặn.

```java
public CyclicBarrier(int parties) {
    this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

Trong đó, `parties` đại diện cho số lượng luồng bị chặn, khi số lượng luồng bị chặn đạt đến giá trị này thì mở rào chắn, cho tất cả luồng đi qua.

2. Khi đối tượng `CyclicBarrier` gọi phương thức `await()`, thực tế gọi là phương thức `dowait(false, 0L)`. Phương thức `await()` giống như hành vi dựng lên một rào chắn vậy, chặn các luồng lại, khi số lượng luồng bị chặn đạt đến giá trị của `parties`, rào chắn mới mở ra, luồng mới được đi qua thực thi.

```java
public int await() throws InterruptedException, BrokenBarrierException {
  try {
      return dowait(false, 0L);
  } catch (TimeoutException toe) {
      throw new Error(toe); // cannot happen
  }
}
```

Phân tích mã nguồn phương thức `dowait(false, 0L)` như sau:

```java
    // Khi số lượng luồng hoặc số lượng request đạt count thì phương thức sau await mới được thực thi. Trong ví dụ trên giá trị của count là 5.
    private int count;
    /**
     * Main barrier code, covering the various policies.
     */
    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        // Lock lại
        lock.lock();
        try {
            final Generation g = generation;

            if (g.broken)
                throw new BrokenBarrierException();

            // Nếu luồng bị ngắt, ném ra ngoại lệ
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }
            // count giảm 1
            int index = --count;
            // Khi số lượng count giảm về 0 chứng tỏ luồng cuối cùng đã đến rào chắn rồi, tức là đạt điều kiện có thể thực thi các phương thức sau await
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();
                    ranAction = true;
                    // Reset count thành giá trị khởi tạo của thuộc tính parties
                    // Đánh thức các luồng chờ trước đó
                    // Đợt thực thi tiếp theo bắt đầu
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

                if (g != generation)
                    return index;

                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();
        }
    }
```

#### Thực chiến

Ví dụ 1:

```java
public class CyclicBarrierExample1 {
  // Số lượng request
  private static final int threadCount = 550;
  // Số lượng luồng cần đồng bộ
  private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5);

  public static void main(String[] args) throws InterruptedException {
    // Tạo ThreadPool
    ExecutorService threadPool = Executors.newFixedThreadPool(10);

    for (int i = 0; i < threadCount; i++) {
      final int threadNum = i;
      Thread.sleep(1000);
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } catch (BrokenBarrierException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      });
    }
    threadPool.shutdown();
  }

  public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
    System.out.println("threadnum:" + threadnum + "is ready");
    try {
      /**Chờ 60 giây, đảm bảo luồng con thực thi hoàn toàn kết thúc*/
      cyclicBarrier.await(60, TimeUnit.SECONDS);
    } catch (Exception e) {
      System.out.println("-----CyclicBarrierException------");
    }
    System.out.println("threadnum:" + threadnum + "is finish");
  }

}
```

Kết quả chạy như sau:

```plain
threadnum:0is ready
threadnum:1is ready
threadnum:2is ready
threadnum:3is ready
threadnum:4is ready
threadnum:4is finish
threadnum:0is finish
threadnum:1is finish
threadnum:2is finish
threadnum:3is finish
threadnum:5is ready
threadnum:6is ready
threadnum:7is ready
threadnum:8is ready
threadnum:9is ready
threadnum:9is finish
threadnum:5is finish
threadnum:8is finish
threadnum:7is finish
threadnum:6is finish
......
```

Có thể thấy khi số lượng luồng tức số lượng request đạt 5 cái mà chúng ta định nghĩa, phương thức sau `await()` mới được thực thi.

Ngoài ra, `CyclicBarrier` còn cung cấp một constructor cao cấp hơn `CyclicBarrier(int parties, Runnable barrierAction)`, dùng để khi các luồng đến rào chắn, ưu tiên thực thi `barrierAction`, thuận tiện xử lý các kịch bản nghiệp vụ phức tạp hơn.

Ví dụ 2:

```java
public class CyclicBarrierExample2 {
  // Số lượng request
  private static final int threadCount = 550;
  // Số lượng luồng cần đồng bộ
  private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
    System.out.println("------Khi số lượng luồng đạt đủ, ưu tiên thực thi------");
  });

  public static void main(String[] args) throws InterruptedException {
    // Tạo ThreadPool
    ExecutorService threadPool = Executors.newFixedThreadPool(10);

    for (int i = 0; i < threadCount; i++) {
      final int threadNum = i;
      Thread.sleep(1000);
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } catch (BrokenBarrierException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      });
    }
    threadPool.shutdown();
  }

  public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
    System.out.println("threadnum:" + threadnum + "is ready");
    cyclicBarrier.await();
    System.out.println("threadnum:" + threadnum + "is finish");
  }

}
```

Kết quả chạy như sau:

```plain
threadnum:0is ready
threadnum:1is ready
threadnum:2is ready
threadnum:3is ready
threadnum:4is ready
------Khi số lượng luồng đạt đủ, ưu tiên thực thi------
threadnum:4is finish
threadnum:0is finish
threadnum:2is finish
threadnum:1is finish
threadnum:3is finish
threadnum:5is ready
threadnum:6is ready
threadnum:7is ready
threadnum:8is ready
threadnum:9is ready
------Khi số lượng luồng đạt đủ, ưu tiên thực thi------
threadnum:9is finish
threadnum:5is finish
threadnum:6is finish
threadnum:8is finish
threadnum:7is finish
......
```

## Tham khảo

- Giải thích chi tiết Java Concurrency AQS: <https://www.cnblogs.com/waterystone/p/4920797.html>
- Từ triển khai ReentrantLock xem nguyên lý và ứng dụng AQS: <https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html>

<!-- @include: @article-footer.snippet.md -->
