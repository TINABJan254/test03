---
title: Tổng hợp câu hỏi phỏng vấn Java Concurrent (Phần 1)
description: Câu hỏi phỏng vấn nền tảng về lập trình đồng thời Java: giải thích chi tiết sự khác biệt giữa Thread và tiến trình, các cách tạo đa luồng, trạng thái vòng đời Thread, bốn điều kiện deadlock và cách phòng tránh, khái niệm concurrent và parallel, v.v.
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: Java并发,线程与进程,多线程,死锁,线程生命周期,并发编程,Java面试题,线程创建方式
---

## Thread

### ⭐️ Thread và tiến trình là gì?

#### Tiến trình là gì?

Tiến trình là một lần thực thi của chương trình, là đơn vị cơ bản để hệ thống chạy chương trình, do đó tiến trình là động. Việc hệ thống chạy một chương trình chính là quá trình một tiến trình từ khi được tạo ra, chạy đến khi kết thúc.

Trong Java, khi chúng ta khởi động hàm main thực ra là khởi động một tiến trình JVM, và Thread chứa hàm main chính là một Thread trong tiến trình đó, còn được gọi là Thread chính (main thread).

Như hình dưới đây, trong Windows chúng ta có thể xem Task Manager để thấy rõ các tiến trình đang chạy trên Windows (việc chạy các file `.exe`).

![Ví dụ minh họa tiến trình - Windows](https://oss.javaguide.cn/github/javaguide/java/%E8%BF%9B%E7%A8%8B%E7%A4%BA%E4%BE%8B%E5%9B%BE%E7%89%87-Windows.png)

#### Thread là gì?

Thread tương tự tiến trình, nhưng Thread là đơn vị thực thi nhỏ hơn tiến trình. Một tiến trình trong quá trình thực thi có thể tạo ra nhiều Thread. Khác với tiến trình, nhiều Thread cùng loại chia sẻ tài nguyên **heap** và **phương thức vùng (method area)** của tiến trình, nhưng mỗi Thread có **bộ đếm chương trình (program counter)**, **stack máy ảo (virtual machine stack)** và **stack phương thức native (native method stack)** riêng. Vì vậy, chi phí để hệ thống tạo một Thread hay chuyển đổi giữa các Thread thấp hơn nhiều so với tiến trình, đó cũng là lý do Thread còn được gọi là tiến trình nhẹ (lightweight process).

Chương trình Java vốn dĩ là chương trình đa luồng, chúng ta có thể dùng JMX để xem một chương trình Java thông thường có những Thread nào, code như sau:

```java
public class MultiThread {
	public static void main(String[] args) {
		// Lấy Java thread management MXBean
	ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
		// Không cần lấy thông tin monitor và synchronizer, chỉ lấy thông tin Thread và thread stack
		ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
		// Duyệt qua thông tin Thread, chỉ in ID và tên Thread
		for (ThreadInfo threadInfo : threadInfos) {
			System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
		}
	}
}
```

Kết quả đầu ra của chương trình trên như sau (nội dung có thể khác nhau, không cần quá chú ý vai trò của từng Thread, chỉ cần biết main thread thực thi hàm main):

```plain
[5] Attach Listener //Thêm sự kiện
[4] Signal Dispatcher // Thread xử lý và phân phối tín hiệu JVM
[3] Finalizer //Thread gọi phương thức finalize của đối tượng
[2] Reference Handler //Thread dọn dẹp reference
[1] main //main thread, điểm vào chương trình
```

Từ nội dung đầu ra trên có thể thấy: **một chương trình Java chạy là main thread và nhiều Thread khác chạy đồng thời**.

### Java Thread và Thread của hệ điều hành khác nhau như thế nào?

Trong JDK thời kỳ đầu từng sử dụng Green Thread để triển khai user-level thread. Sau đó, Thread truyền thống được tạo bằng `new Thread()` trong HotSpot sử dụng Platform Thread, Platform Thread thường ánh xạ theo tỷ lệ 1:1 với Thread của hệ điều hành, do hệ điều hành chịu trách nhiệm lập lịch. Virtual Thread được giới thiệu chính thức trong Java 21 được lập lịch bởi JVM, nhiều Virtual Thread có thể tái sử dụng ít Platform Thread hơn làm carrier, vì vậy không thể coi tất cả Java Thread đều tương đương với Thread của hệ điều hành nữa.

Chúng ta đã đề cập đến user thread và kernel thread, để giúp nhiều bạn đọc chưa hiểu rõ sự khác biệt, đây là phần giới thiệu ngắn:

- User thread: Thread được quản lý và lập lịch bởi chương trình user space, chạy trong user space (dành riêng cho ứng dụng).
- Kernel thread: Thread được quản lý và lập lịch bởi kernel hệ điều hành, chạy trong kernel space (chỉ kernel program mới có thể truy cập).

Tóm tắt ngắn gọn sự khác biệt và đặc điểm của user thread và kernel thread: User-level thread thường được lập lịch bởi runtime trong user space, chi phí tạo và chuyển đổi thấp hơn; khả năng tận dụng đa nhân phụ thuộc vào mô hình ánh xạ giữa user thread và kernel thread, mô hình nhiều-một không thể tận dụng đa nhân song song, mô hình nhiều-nhiều thì có thể. Kernel thread được lập lịch bởi hệ điều hành, chi phí tạo và chuyển đổi thường cao hơn, có thể trực tiếp tận dụng đa nhân.

Tóm gọn mối quan hệ giữa Java Thread và Thread hệ điều hành: **Platform Thread thường ánh xạ đến Thread hệ điều hành, còn Virtual Thread được JVM lập lịch và gắn vào Platform Thread để thực thi**.

Mô hình Thread là cách liên kết giữa user thread và kernel thread, có ba mô hình Thread phổ biến:

1. Một-một (một user thread tương ứng một kernel thread)
2. Nhiều-một (nhiều user thread ánh xạ vào một kernel thread)
3. Nhiều-nhiều (nhiều user thread ánh xạ vào nhiều kernel thread)

![Ba mô hình Thread phổ biến](https://oss.javaguide.cn/github/javaguide/java/concurrent/three-types-of-thread-models.png)

Trong các hệ điều hành phổ biến như Windows và Linux, Platform Thread của HotSpot thường sử dụng mô hình một-một, tức là một Platform Thread tương ứng một Thread hệ điều hành. Virtual Thread không sử dụng ánh xạ một-một này mà được JVM lập lịch trên một tập Platform Thread.

### ⭐️ Hãy mô tả ngắn gọn mối quan hệ, sự khác biệt và ưu nhược điểm của Thread và tiến trình?

Hình dưới đây là vùng bộ nhớ Java, qua đó chúng ta sẽ nhìn từ góc độ JVM để nói về mối quan hệ giữa Thread và tiến trình.

![Vùng dữ liệu runtime Java (sau JDK1.8)](https://oss.javaguide.cn/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.8.png)

Từ hình trên có thể thấy: một tiến trình có thể có nhiều Thread, nhiều Thread chia sẻ tài nguyên **heap** và **phương thức vùng (Method Area - Metaspace sau JDK1.8)** của tiến trình, nhưng mỗi Thread có **bộ đếm chương trình**, **stack máy ảo** và **stack phương thức native** riêng.

**Tổng kết:** Thread là đơn vị thực thi nhỏ hơn được chia từ tiến trình. Điểm khác biệt lớn nhất giữa Thread và tiến trình là về cơ bản các tiến trình độc lập với nhau, còn các Thread thì không nhất thiết, vì các Thread trong cùng một tiến trình rất có thể sẽ ảnh hưởng lẫn nhau. Thread có chi phí thực thi thấp nhưng không có lợi cho quản lý và bảo vệ tài nguyên; tiến trình thì ngược lại.

Dưới đây là nội dung mở rộng cho kiến thức này!

Hãy suy nghĩ về vấn đề này: Tại sao **bộ đếm chương trình**, **stack máy ảo** và **stack phương thức native** là riêng của Thread? Tại sao heap và phương thức vùng được chia sẻ giữa các Thread?

#### Tại sao bộ đếm chương trình lại là riêng của Thread?

Bộ đếm chương trình có hai công dụng chính:

1. Bộ thông dịch bytecode thay đổi bộ đếm chương trình để lần lượt đọc các lệnh, từ đó thực hiện kiểm soát luồng code như: thực thi tuần tự, lựa chọn, vòng lặp, xử lý ngoại lệ.
2. Trong trường hợp đa luồng, bộ đếm chương trình dùng để ghi lại vị trí thực thi của Thread hiện tại, để khi Thread được chuyển trở lại có thể biết Thread đó đã chạy đến đâu.

Cần lưu ý rằng nếu thực thi phương thức native, bộ đếm chương trình ghi địa chỉ undefined, chỉ khi thực thi code Java thì bộ đếm chương trình mới ghi địa chỉ lệnh tiếp theo.

Vì vậy, bộ đếm chương trình là riêng chủ yếu để **sau khi chuyển đổi Thread có thể khôi phục đúng vị trí thực thi**.

#### Tại sao stack máy ảo và stack phương thức native lại là riêng của Thread?

- **Stack máy ảo (Virtual Machine Stack):** Mỗi phương thức Java trước khi thực thi sẽ tạo một stack frame để lưu bảng biến cục bộ, operand stack, tham chiếu constant pool, v.v. Quá trình từ khi gọi phương thức đến khi hoàn thành tương ứng với quá trình một stack frame được đẩy vào và lấy ra khỏi Java Virtual Machine Stack.
- **Stack phương thức native (Native Method Stack):** Vai trò rất giống stack máy ảo, điểm khác biệt là: **stack máy ảo phục vụ cho việc thực thi phương thức Java (tức là bytecode), còn stack phương thức native phục vụ cho các phương thức Native mà máy ảo sử dụng.** Trong HotSpot VM, stack phương thức native và Java Virtual Machine Stack được hợp nhất làm một.

Vì vậy, để **đảm bảo biến cục bộ trong Thread không bị Thread khác truy cập**, stack máy ảo và stack phương thức native là riêng của Thread.

#### Một câu để hiểu heap và phương thức vùng

Heap và phương thức vùng là tài nguyên chia sẻ cho tất cả các Thread, trong đó heap là vùng bộ nhớ lớn nhất trong tiến trình, chủ yếu dùng để lưu trữ các đối tượng mới được tạo (hầu hết các đối tượng được cấp phát bộ nhớ ở đây), phương thức vùng chủ yếu dùng để lưu thông tin lớp đã được nạp, hằng số, biến tĩnh, code được biên dịch bởi JIT compiler, v.v.

### Làm thế nào để tạo Thread?

Nói chung, có nhiều cách để tạo Thread, ví dụ như kế thừa lớp `Thread`, implement interface `Runnable`, implement interface `Callable`, sử dụng Thread Pool, sử dụng lớp `CompletableFuture`, v.v.

Tuy nhiên, những cách này thực ra không thực sự tạo ra Thread. Nói chính xác hơn, đây đều là các phương thức để sử dụng đa luồng trong code Java.

Nói nghiêm ngặt, Java chỉ có một cách duy nhất để tạo Thread, đó là thông qua `new Thread().start()`. Bất kể là cách nào, cuối cùng đều phụ thuộc vào `new Thread().start()`.

### ⭐️ Hãy nói về vòng đời và trạng thái của Thread?

Thread Java trong vòng đời thực thi của nó tại bất kỳ thời điểm cụ thể nào cũng chỉ có thể ở một trong 6 trạng thái khác nhau sau:

- NEW: Trạng thái khởi tạo, Thread được tạo nhưng chưa gọi `start()`.
- RUNNABLE: Trạng thái chạy, Thread đã được gọi `start()` và đang chờ chạy.
- BLOCKED: Trạng thái bị chặn, cần chờ Lock được giải phóng.
- WAITING: Trạng thái chờ, Thread này cần chờ Thread khác thực hiện một số hành động cụ thể (thông báo hoặc ngắt).
- TIMED_WAITING: Trạng thái chờ có thời hạn, có thể tự trở về sau một thời gian chỉ định thay vì chờ vô thời hạn như WAITING.
- TERMINATED: Trạng thái kết thúc, Thread đã chạy xong.

Thread trong vòng đời không cố định ở một trạng thái mà chuyển đổi giữa các trạng thái khác nhau theo quá trình thực thi code.

Sơ đồ chuyển đổi trạng thái Thread Java (nguồn hình: [Sửa lỗi | Ba lỗi về trạng thái Thread trong "The Art of Java Concurrent Programming"](https://mp.weixin.qq.com/s/0UTyrJpRKaKhkhHcQtXAiA)):

![Sơ đồ chuyển đổi trạng thái Thread Java](https://oss.javaguide.cn/github/javaguide/java/concurrent/640.png)

Từ hình trên có thể thấy: sau khi tạo Thread sẽ ở trạng thái **NEW (Mới tạo)**, sau khi gọi phương thức `start()` bắt đầu chạy, Thread lúc này ở trạng thái **READY (Sẵn sàng)**. Thread ở trạng thái sẵn sàng khi nhận được CPU time slice (timeslice) sẽ ở trạng thái **RUNNING (Đang chạy)**.

> Ở tầng hệ điều hành, Thread có trạng thái READY và RUNNING; còn ở tầng JVM, chỉ có thể thấy trạng thái RUNNABLE (nguồn hình: [HowToDoInJava](https://howtodoinJava.com/ "HowToDoInJava"): [Java Thread Life Cycle and Thread States](https://howtodoinJava.com/Java/multi-threading/Java-thread-life-cycle-and-thread-states/ "Java Thread Life Cycle and Thread States")), nên hệ thống Java thường gọi chung hai trạng thái này là trạng thái **RUNNABLE (Đang chạy)**.
>
> **Tại sao JVM không phân biệt hai trạng thái này?** `Thread.State` của Java mô tả trạng thái Thread ở tầng JVM, không nhằm phản ánh toàn bộ trạng thái nội bộ của scheduler hệ điều hành. Chiến lược lập lịch cụ thể, độ dài time slice và liệu có sử dụng round-robin hay không đều do hệ điều hành và cấu hình của nó quyết định.

![RUNNABLE-VS-RUNNING](https://oss.javaguide.cn/github/javaguide/java/RUNNABLE-VS-RUNNING.png)

- Khi Thread thực thi phương thức `wait()`, Thread sẽ vào trạng thái **WAITING (Chờ)**. Thread ở trạng thái chờ cần dựa vào thông báo từ Thread khác để trở lại trạng thái chạy.
- Trạng thái **TIMED_WAITING (Chờ có thời hạn)** tương đương với trạng thái chờ nhưng thêm giới hạn thời gian, ví dụ có thể đặt Thread vào trạng thái TIMED_WAITING bằng phương thức `sleep(long millis)` hoặc `wait(long millis)`. Khi thời gian chờ kết thúc, Thread sẽ trở về trạng thái RUNNABLE.
- Khi Thread vào phương thức/block `synchronized` hoặc sau khi gọi `wait` (được `notify`) muốn vào lại phương thức/block `synchronized`, nhưng Lock bị Thread khác chiếm giữ, lúc này Thread sẽ vào trạng thái **BLOCKED (Bị chặn)**.
- Thread sau khi thực thi xong phương thức `run()` sẽ vào trạng thái **TERMINATED (Kết thúc)**.

### Chuyển đổi ngữ cảnh Thread là gì?

Thread trong quá trình thực thi sẽ có điều kiện và trạng thái chạy riêng (còn gọi là ngữ cảnh), như bộ đếm chương trình, thông tin stack đã đề cập ở trên. Khi xảy ra các tình huống sau, Thread sẽ thoát khỏi trạng thái chiếm dụng CPU.

- Chủ động nhường CPU, ví dụ gọi `sleep()`, `wait()`, v.v.
- Hết time slice, vì hệ điều hành phải ngăn một Thread hoặc tiến trình chiếm dụng CPU quá lâu gây ra các Thread hoặc tiến trình khác bị đói tài nguyên.
- Gọi system interrupt loại blocking, ví dụ yêu cầu IO, Thread bị block.
- Bị kết thúc hoặc dừng chạy

Trong ba trường hợp đầu đều xảy ra chuyển đổi Thread, chuyển đổi Thread đồng nghĩa với việc cần lưu ngữ cảnh của Thread hiện tại, để khi Thread chiếm CPU tiếp theo có thể khôi phục. Và tải ngữ cảnh của Thread sẽ chiếm CPU tiếp theo. Đây gọi là **chuyển đổi ngữ cảnh (context switch)**.

Chuyển đổi ngữ cảnh là chức năng cơ bản của hệ điều hành hiện đại, vì mỗi lần cần lưu và khôi phục thông tin sẽ tiêu tốn tài nguyên CPU, bộ nhớ của hệ thống, có nghĩa là hiệu suất sẽ bị giảm đi ít nhiều, nếu chuyển đổi quá thường xuyên sẽ gây hiệu suất tổng thể thấp.

### So sánh phương thức Thread#sleep() và Object#wait()

**Điểm chung**: Cả hai đều có thể tạm dừng thực thi Thread.

**Sự khác biệt**:

- **`sleep()` không giải phóng Lock, còn `wait()` giải phóng Lock**.
- `wait()` thường được dùng cho giao tiếp/tương tác giữa các Thread, `sleep()` thường dùng để tạm dừng thực thi.
- Sau khi gọi `wait()`, Thread sẽ không tự thức dậy, cần Thread khác gọi `notify()` hoặc `notifyAll()` trên cùng đối tượng. Sau khi `sleep()` thực thi xong, Thread sẽ tự thức dậy, hoặc cũng có thể dùng `wait(long timeout)` để Thread tự thức dậy sau thời gian chờ.
- `sleep()` là phương thức tĩnh native của lớp `Thread`, còn `wait()` là phương thức native của lớp `Object`. Tại sao thiết kế như vậy? Câu hỏi tiếp theo sẽ đề cập.

### Tại sao phương thức wait() không được định nghĩa trong Thread?

`wait()` là để Thread đang giữ object lock thực hiện chờ, sẽ tự động giải phóng object lock mà Thread hiện tại đang nắm giữ. Mỗi `Object` đều có object lock, vì muốn giải phóng object lock của Thread hiện tại và đưa nó vào trạng thái WAITING, tự nhiên phải thao tác trên đối tượng tương ứng (`Object`) chứ không phải Thread hiện tại (`Thread`).

Câu hỏi tương tự: **Tại sao phương thức `sleep()` được định nghĩa trong `Thread`?**

Vì `sleep()` là để Thread hiện tại tạm dừng thực thi, không liên quan đến lớp đối tượng, cũng không cần lấy object lock.

### Có thể trực tiếp gọi phương thức run của lớp Thread không?

Đây là một câu hỏi phỏng vấn đa luồng Java rất kinh điển khác, và thường được hỏi trong phỏng vấn. Rất đơn giản nhưng nhiều người không trả lời được!

Khi new một `Thread`, Thread vào trạng thái mới tạo. Gọi phương thức `start()` sẽ khởi động một Thread và đưa Thread vào trạng thái sẵn sàng, khi được phân bổ time slice có thể bắt đầu chạy. `start()` sẽ thực hiện công việc chuẩn bị tương ứng của Thread, sau đó tự động thực thi nội dung phương thức `run()`, đây mới là làm việc đa luồng thực sự. Nhưng nếu trực tiếp gọi `run()`, nó sẽ coi `run()` là một phương thức thông thường và thực thi trong Thread đang gọi phương thức đó, vì vậy đây không phải làm việc đa luồng.

**Tổng kết: Gọi phương thức `start()` mới khởi động Thread và đưa Thread vào trạng thái sẵn sàng, trực tiếp thực thi phương thức `run()` sẽ không thực thi theo cách đa luồng.**

## Đa luồng

### Sự khác biệt giữa concurrent và parallel

- **Concurrent**: Hai hoặc nhiều hơn công việc thực thi trong cùng một **khoảng thời gian**.
- **Parallel**: Hai hoặc nhiều hơn công việc thực thi tại cùng **một thời điểm**.

Điểm mấu chốt là: có phải thực thi **đồng thời** hay không.

### Sự khác biệt giữa đồng bộ và bất đồng bộ

- **Đồng bộ (Synchronous)**: Sau khi phát ra một lời gọi, trước khi nhận được kết quả, lời gọi đó không thể trả về, phải chờ liên tục.
- **Bất đồng bộ (Asynchronous)**: Lời gọi sau khi được phát đi, không cần chờ kết quả trả về, lời gọi đó trả về ngay lập tức.

### ⭐️ Tại sao phải sử dụng đa luồng?

Nói từ tổng thể trước:

- **Xét từ tầng đáy của máy tính:** Thread có thể được ví như tiến trình nhẹ, là đơn vị nhỏ nhất trong thực thi chương trình, chi phí chuyển đổi và lập lịch giữa các Thread thấp hơn nhiều so với tiến trình. Ngoài ra, kỷ nguyên CPU đa nhân có nghĩa là nhiều Thread có thể chạy đồng thời, điều này giảm chi phí chuyển đổi ngữ cảnh Thread.
- **Xét từ xu hướng phát triển Internet hiện đại:** Các hệ thống hiện nay thường yêu cầu concurrency hàng triệu thậm chí hàng chục triệu, và lập trình concurrent đa luồng chính là nền tảng để phát triển hệ thống high concurrency, tận dụng tốt cơ chế đa luồng có thể nâng cao đáng kể khả năng concurrent tổng thể và hiệu suất của hệ thống.

Đi sâu hơn vào tầng đáy máy tính:

- **Kỷ nguyên đơn nhân:** Trong kỷ nguyên đơn nhân, đa luồng chủ yếu để nâng cao hiệu suất sử dụng CPU và hệ thống IO của một tiến trình. Giả sử chỉ chạy một tiến trình Java, khi chúng ta yêu cầu IO, nếu tiến trình Java chỉ có một Thread, Thread đó bị IO block thì toàn bộ tiến trình bị block. CPU và thiết bị IO chỉ có một cái chạy, có thể nói đơn giản là hiệu suất tổng thể của hệ thống chỉ đạt 50%. Khi sử dụng đa luồng, một Thread bị IO block, các Thread khác vẫn có thể tiếp tục sử dụng CPU, từ đó nâng cao hiệu suất sử dụng tài nguyên hệ thống của tiến trình Java.
- **Kỷ nguyên đa nhân:** Trong kỷ nguyên đa nhân, đa luồng chủ yếu để nâng cao khả năng tận dụng CPU đa nhân của tiến trình. Ví dụ: giả sử chúng ta cần tính toán một tác vụ phức tạp, nếu chỉ dùng một Thread thì dù hệ thống có bao nhiêu nhân CPU cũng chỉ có một nhân được sử dụng. Còn tạo nhiều Thread, các Thread đó có thể được ánh xạ đến nhiều nhân CPU ở tầng đáy để thực thi, trong trường hợp các Thread trong tác vụ không có tranh chấp tài nguyên, hiệu suất thực thi tác vụ sẽ được cải thiện đáng kể, khoảng bằng (thời gian thực thi trên đơn nhân / số nhân CPU).

### ⭐️ CPU đơn nhân có hỗ trợ đa luồng Java không?

CPU đơn nhân có hỗ trợ đa luồng Java. Hệ điều hành phân phối thời gian CPU cho các Thread khác nhau theo cách time-slice round-robin. Mặc dù CPU đơn nhân mỗi lần chỉ thực thi được một tác vụ, nhưng bằng cách chuyển đổi nhanh giữa nhiều Thread, có thể khiến người dùng cảm thấy như nhiều tác vụ đang diễn ra đồng thời.

Đây cũng là dịp để đề cập đến phương thức lập lịch Thread mà Java sử dụng.

Hệ điều hành chủ yếu quản lý thực thi đa luồng thông qua hai phương thức lập lịch Thread:

- **Preemptive Scheduling (Lập lịch ưu tiên):** Hệ điều hành quyết định khi nào tạm dừng Thread đang chạy và chuyển sang thực thi Thread khác. Việc chuyển đổi này thường được kích hoạt bởi ngắt đồng hồ hệ thống (time-slice round-robin) hoặc các sự kiện có độ ưu tiên cao khác (như hoàn thành thao tác I/O). Phương thức này có chi phí chuyển đổi ngữ cảnh nhưng tính công bằng và hiệu suất sử dụng tài nguyên CPU tốt hơn, khó bị block.
- **Cooperative Scheduling (Lập lịch hợp tác):** Sau khi Thread thực thi xong, chủ động thông báo hệ thống chuyển sang Thread khác. Phương thức này có thể giảm chi phí hiệu suất do chuyển đổi ngữ cảnh, nhưng tính công bằng kém hơn, dễ bị block.

Java sử dụng lập lịch Thread ưu tiên (preemptive). Có nghĩa là JVM không chịu trách nhiệm lập lịch Thread mà ủy thác việc lập lịch cho hệ điều hành. Hệ điều hành thường lập lịch thực thi Thread dựa trên độ ưu tiên Thread và time slice, Thread có độ ưu tiên cao thường có nhiều cơ hội nhận được CPU time slice hơn.

### ⭐️ Chạy nhiều Thread trên CPU đơn nhân có luôn hiệu quả hơn không?

Hiệu quả của việc chạy đồng thời nhiều Thread trên CPU đơn nhân phụ thuộc vào loại Thread và tính chất của tác vụ. Nói chung, có hai loại Thread:

1. **CPU-intensive:** Thread CPU-intensive chủ yếu thực hiện tính toán và xử lý logic, cần chiếm dụng nhiều tài nguyên CPU.
2. **IO-intensive:** Thread IO-intensive chủ yếu thực hiện các thao tác đầu vào/đầu ra như đọc/ghi file, giao tiếp mạng, cần chờ phản hồi từ thiết bị IO mà không chiếm dụng nhiều tài nguyên CPU.

Trên CPU đơn nhân, tại một thời điểm chỉ có một Thread chạy, các Thread khác cần chờ phân bổ time slice CPU. Nếu Thread là CPU-intensive thì nhiều Thread chạy đồng thời sẽ dẫn đến chuyển đổi Thread thường xuyên, tăng chi phí hệ thống, giảm hiệu suất. Nếu Thread là IO-intensive thì nhiều Thread chạy đồng thời có thể tận dụng thời gian nhàn rỗi của CPU khi chờ IO, tăng hiệu suất.

Do đó, với CPU đơn nhân, nếu tác vụ là CPU-intensive thì mở nhiều Thread sẽ ảnh hưởng đến hiệu suất; nếu tác vụ là IO-intensive thì mở nhiều Thread sẽ cải thiện hiệu suất. Tất nhiên, "nhiều" ở đây cũng phải ở mức hợp lý, không được vượt quá giới hạn chịu đựng của hệ thống.

### Sử dụng đa luồng có thể mang lại những vấn đề gì?

Mục đích của lập trình concurrent là để nâng cao hiệu suất thực thi chương trình, nhưng lập trình concurrent không phải lúc nào cũng cải thiện tốc độ chạy, và lập trình concurrent có thể gặp nhiều vấn đề như: rò rỉ bộ nhớ, deadlock, thread safety, v.v.

### Làm thế nào để hiểu thread safety và thread unsafe?

Thread safety và thread unsafe là mô tả về việc truy cập vào cùng một dữ liệu trong môi trường đa luồng có đảm bảo tính chính xác và nhất quán hay không.

- Thread safety có nghĩa là trong môi trường đa luồng, cho cùng một dữ liệu, dù có bao nhiêu Thread truy cập đồng thời vẫn đảm bảo tính chính xác và nhất quán của dữ liệu đó.
- Thread unsafe có nghĩa là trong môi trường đa luồng, cho cùng một dữ liệu, nhiều Thread truy cập đồng thời có thể dẫn đến dữ liệu bị lộn xộn, lỗi hoặc mất mát.

## ⭐️ Deadlock

### Deadlock Thread là gì?

Deadlock Thread mô tả tình huống: nhiều Thread bị block đồng thời, một hoặc tất cả đều đang chờ một tài nguyên được giải phóng. Vì Thread bị block vô thời hạn, chương trình không thể kết thúc bình thường.

Như hình dưới đây, Thread A giữ resource 2, Thread B giữ resource 1, cả hai đều muốn xin tài nguyên của nhau, vì vậy hai Thread này sẽ chờ lẫn nhau và rơi vào trạng thái deadlock.

![Sơ đồ tình huống deadlock: Thread A giữ resource1 và chờ resource2, Thread B giữ resource2 và chờ resource1, chuỗi chờ tạo thành vòng kín](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/dead-lock-deadlock-scenario.png)

Dưới đây là ví dụ để minh họa deadlock Thread, code mô phỏng tình huống deadlock trong hình trên (code được lấy từ "Beauty of Concurrent Programming"):

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//Tài nguyên 1
    private static Object resource2 = new Object();//Tài nguyên 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

Output

```plain
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```

Thread A qua `synchronized (resource1)` lấy được monitor lock của `resource1`, sau đó dùng `Thread.sleep(1000);` cho Thread A ngủ 1 giây, mục đích là để Thread B chạy và lấy được monitor lock của resource2. Thread A và Thread B sau khi ngủ xong đều bắt đầu cố gắng xin tài nguyên của nhau, rồi hai Thread này sẽ rơi vào trạng thái chờ lẫn nhau, đây chính là deadlock.

Ví dụ trên thỏa mãn bốn điều kiện cần thiết để xảy ra deadlock:

1. **Điều kiện loại trừ lẫn nhau (Mutual Exclusion):** Tại bất kỳ thời điểm nào, tài nguyên chỉ được chiếm dụng bởi một Thread.
2. **Điều kiện giữ và chờ (Hold and Wait):** Khi Thread bị block do yêu cầu tài nguyên, vẫn giữ những tài nguyên đã lấy được mà không chịu nhả.
3. **Điều kiện không bị chiếm đoạt (No Preemption):** Tài nguyên mà Thread đã lấy không thể bị Thread khác cưỡng bức chiếm đoạt trước khi dùng xong, chỉ khi tự mình dùng xong mới giải phóng tài nguyên.
4. **Điều kiện chờ vòng tròn (Circular Wait):** Một số Thread hình thành mối quan hệ chờ tài nguyên theo vòng đầu-đuôi nối tiếp nhau.

### Làm thế nào để phát hiện deadlock?

- Dùng `jstack <pid>` hoặc `jcmd <pid> Thread.print -l` để xem thông tin thread stack và concurrent lock. Nếu phát hiện deadlock ở tầng Java, output sẽ liệt kê các Thread liên quan và Lock mà chúng đang giữ, đang chờ. `jmap` chủ yếu dùng để xem thông tin heap hoặc tạo heap dump, không phải công cụ chẩn đoán deadlock Thread.
- Dùng các công cụ như VisualVM, JConsole để kiểm tra.

Ở đây lấy JConsole làm ví dụ minh họa.

Đầu tiên, chúng ta cần tìm thư mục bin của JDK, tìm jconsole và double-click để mở.

![jconsole](https://oss.javaguide.cn/github/javaguide/java/concurrent/jdk-home-bin-jconsole.png)

Với người dùng MAC, có thể dùng `/usr/libexec/java_home -V` để xem thư mục cài đặt JDK, sau đó dùng `open . + địa chỉ thư mục` để mở. Ví dụ, đường dẫn một JDK trên máy local của tôi là:

```bash
 open . /Users/guide/Library/Java/JavaVirtualMachines/corretto-1.8.0_252/Contents/Home
```

Sau khi mở jconsole, kết nối với chương trình tương ứng, rồi vào giao diện Thread chọn phát hiện deadlock!

![jconsole phát hiện deadlock](https://oss.javaguide.cn/github/javaguide/java/concurrent/jconsole-check-deadlock.png)

![jconsole phát hiện được deadlock](https://oss.javaguide.cn/github/javaguide/java/concurrent/jconsole-check-deadlock-done.png)

### Làm thế nào để phòng ngừa và tránh deadlock Thread?

**Làm thế nào để phòng ngừa deadlock?** Phá vỡ các điều kiện cần thiết để xảy ra deadlock:

1. **Phá vỡ điều kiện giữ và chờ:** Xin tất cả tài nguyên một lần.
2. **Phá vỡ điều kiện không bị chiếm đoạt:** Khi Thread đang chiếm một phần tài nguyên và tiếp tục xin tài nguyên khác mà không được, có thể chủ động giải phóng tài nguyên đang chiếm.
3. **Phá vỡ điều kiện chờ vòng tròn:** Dùng cách xin tài nguyên theo thứ tự để phòng ngừa. Xin tài nguyên theo một thứ tự nhất định, giải phóng theo thứ tự ngược lại. Phá vỡ điều kiện chờ vòng tròn.

**Làm thế nào để tránh deadlock?**

Tránh deadlock là trong quá trình phân bổ tài nguyên, dựa vào thuật toán (ví dụ Banker's Algorithm) để tính toán đánh giá việc phân bổ tài nguyên, đưa nó vào trạng thái an toàn.

> **Trạng thái an toàn** có nghĩa là hệ thống có thể phân bổ tài nguyên cần thiết cho mỗi Thread theo một thứ tự Thread tiến hành nào đó (P1, P2, P3...Pn), cho đến khi đáp ứng nhu cầu tối đa về tài nguyên của mỗi Thread, để mỗi Thread có thể hoàn thành thuận lợi. Gọi chuỗi `<P1, P2, P3.....Pn>` là chuỗi an toàn.

Nếu sửa code của Thread 2 thành như dưới đây thì sẽ không xảy ra deadlock.

```java
new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 2").start();
```

Kết quả đầu ra:

```plain
Thread[线程 1,5,main]get resource1
Thread[线程 1,5,main]waiting get resource2
Thread[线程 1,5,main]get resource2
Thread[线程 2,5,main]get resource1
Thread[线程 2,5,main]waiting get resource2
Thread[线程 2,5,main]get resource2

Process finished with exit code 0
```

Chúng ta phân tích tại sao code trên tránh được deadlock?

Thread 1 trước tiên lấy được monitor lock của resource1, lúc này Thread 2 không lấy được. Sau đó Thread 1 tiếp tục lấy monitor lock của resource2, có thể lấy được. Sau đó Thread 1 giải phóng việc chiếm dụng monitor lock của resource1 và resource2, Thread 2 có thể lấy được và thực thi. Như vậy đã phá vỡ điều kiện chờ vòng tròn, do đó tránh được deadlock.

<!-- @include: @article-footer.snippet.md -->
