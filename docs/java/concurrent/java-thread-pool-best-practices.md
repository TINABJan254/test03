---
title: Java 线程池最佳实践
description: Java 线程池最佳实践总结：从业务吞吐、任务耗时和下游容量估算线程数与队列长度，讲解队列堆积、拒绝策略、任务超时、异常处理、动态调参、监控和虚拟线程等生产问题。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: 线程池最佳实践,ThreadPoolExecutor参数配置,线程池监控,队列堆积,CallerRunsPolicy,动态线程池,任务超时,虚拟线程
---

Khi nhìn thấy bộ cấu hình `corePoolSize = 8`, `maximumPoolSize = 16`, `queueCapacity = 1000`, chỉ dựa vào máy có 8 CPU core thì vẫn chưa phán đoán được nó có phù hợp hay không. Nén hình ảnh, query database theo batch và thời gian gọi API bên ngoài có cấu thành khác nhau, số luồng cần thiết có thể chênh lệch rất lớn. Nhiệm vụ đi vào nhanh thế nào, thực thi bao lâu, cho phép xếp hàng bao lâu, cũng như database và phía downstream có thể gánh được bao nhiêu concurrency, đều sẽ ảnh hưởng đến cấu hình cuối cùng.

Kiến thức cơ bản về 7 tham số lớn, quy trình thực thi nhiệm vụ, hàng đợi chặn và chiến lược từ chối, có thể xem trước [Giải thích chi tiết ThreadPool Java](./java-thread-pool-summary.md). Môi trường sản xuất còn phải xử lý ước tính tham số, quá tải, timeout nhiệm vụ và giám sát, các vấn đề này phụ thuộc nhiều hơn vào dung lượng nghiệp vụ và dữ liệu vận hành.

## Khai báo ThreadPool đúng cách

ThreadPool nghiệp vụ thường cần chỉ định rõ số luồng, dung lượng hàng đợi, tên luồng và chiến lược từ chối. Sử dụng các shortcut method của `Executors` mặc dù tiện lợi, nhưng một số phương thức sẽ tạo hàng đợi xấp xỉ vô hạn hoặc cho phép số luồng liên tục tăng trưởng, thời điểm cao điểm nghiệp vụ rất dễ tích tụ lượng lớn nhiệm vụ hoặc tạo quá nhiều luồng.

- `newFixedThreadPool()` và `newSingleThreadExecutor()` sử dụng `LinkedBlockingQueue` vô hạn, khi nhiệm vụ liên tục đi vào, hàng đợi có thể không ngừng tăng trưởng.
- `newCachedThreadPool()` sử dụng `SynchronousQueue`, số luồng tối đa là `Integer.MAX_VALUE`, khi tốc độ submit thời gian dài cao hơn tốc độ xử lý có thể tạo lượng lớn luồng.
- `newScheduledThreadPool()` sử dụng `DelayedWorkQueue` vô hạn, tương tự cũng phải cân nhắc vấn đề các nhiệm vụ trì hoãn không ngừng tích tụ.

Do đó, ThreadPool nghiệp vụ thông thường phù hợp hơn nếu trực tiếp sử dụng `ThreadPoolExecutor`, viết giới hạn dung lượng trong file cấu hình:

```java
ThreadFactory threadFactory = new NamingThreadFactory("order-query");

ThreadPoolExecutor executor = new ThreadPoolExecutor(
        8,
        16,
        60L,
        TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(200),
        threadFactory,
        new ThreadPoolExecutor.AbortPolicy()
);
```

Các số `8`, `16` và `200` trong ví dụ chỉ dùng để hiển thị vị trí tham số, không thể dùng trực tiếp cho môi trường sản xuất. Tham số phải kết hợp với dung lượng nghiệp vụ và kết quả test tải để xác định.

ThreadFactory ít nhất nên thiết lập tên có ý nghĩa nghiệp vụ cho luồng:

```java
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

public final class NamingThreadFactory implements ThreadFactory {

    private final AtomicInteger sequence = new AtomicInteger();
    private final String prefix;

    public NamingThreadFactory(String prefix) {
        this.prefix = prefix;
    }

    @Override
    public Thread newThread(Runnable task) {
        Thread thread = new Thread(task);
        thread.setName(prefix + "-" + sequence.incrementAndGet());
        thread.setDaemon(false);
        return thread;
    }
}
```

Tên luồng sẽ xuất hiện trong thread dump, log và dữ liệu giám sát. `pool-1-thread-3` rất khó phán đoán thuộc về nghiệp vụ nào, còn `order-query-3` thì có thể trực tiếp thu hẹp phạm vi troubleshooting.

## Tham số ThreadPool ước tính như thế nào?

Các nhiệm vụ CPU-intensive thường bắt đầu từ khoảng số core CPU, nhiệm vụ I/O-intensive có thể cấu hình nhiều luồng hơn. Loại kinh nghiệm này phù hợp để làm phán đoán ban đầu, cấu hình thực tế còn phải trả lời một vài câu hỏi cụ thể hơn: Thời điểm cao điểm mỗi giây sẽ submit bao nhiêu nhiệm vụ? Một nhiệm vụ sẽ chạy bao lâu? Trong đó bao nhiêu thời gian chờ I/O? Phía downstream gánh được bao nhiêu concurrency? Nhiệm vụ cho phép xếp hàng tối đa bao lâu?

Trong một cửa sổ cao điểm tương đối ổn định, giả sử số nhiệm vụ submit mỗi giây là $\lambda$, thời gian thực thi trung bình của nhiệm vụ là $T$ giây, số lượng nhiệm vụ đang thực thi $L$ có thể ước tính trước theo cách dưới đây:

$$
L \approx \lambda \times T
$$

Ví dụ, một API nào đó thời điểm cao điểm mỗi giây sinh ra 400 nhiệm vụ bất đồng bộ, mỗi nhiệm vụ thực thi trung bình 80 ms, nhu cầu concurrency trung bình tương ứng khoảng `400 × 0.08 = 32`. Đây chỉ là điểm bắt đầu ước tính, không thể dựa vào đây trực tiếp đặt số luồng thành 32, nguyên nhân bao gồm:

- Thời gian tiêu tốn trung bình sẽ che lấp các nhiệm vụ chậm, P95, P99 có thể cao hơn nhiều so với giá trị trung bình.
- Khi thời gian tiêu tốn nhiệm vụ tăng lên, nhu cầu concurrency cũng sẽ tăng lên theo.
- CPU, bộ nhớ và chuyển đổi ngữ cảnh sẽ giới hạn số lượng Platform Thread.
- Connection pool database hoặc API downstream có thể chỉ cho phép concurrency thấp hơn.

《Java Concurrency in Practice》 từng đưa ra một công thức ước tính có chứa tỷ lệ sử dụng CPU mục tiêu. Đối với các nhiệm vụ pha trộn giữa tính toán và chờ đợi, có thể dùng nó hỗ trợ xác định giá trị ban đầu:

$$
N_{threads} \approx N_{cpu} \times U_{cpu} \times \left(1 + \frac{W}{C}\right)
$$

Trong đó, $N_{cpu}$ là số core CPU, $U_{cpu}$ là tỷ lệ sử dụng CPU mục tiêu, $W/C$ là tỷ lệ giữa thời gian chờ của nhiệm vụ và thời gian tính toán.

Thời gian chờ càng dài, về mặt lý thuyết có thể sắp xếp càng nhiều luồng, để một phần nhiệm vụ khi chờ I/O, các nhiệm vụ khác tiếp tục dùng CPU. Tuy nhiên, khi việc chờ đợi đến từ việc kiệt quệ connection pool database, Rate Limiting của downstream hoặc tranh chấp khóa, việc tăng luồng sẽ chỉ tạo ra thêm sự chờ đợi. Công thức không thể nhận biết sự khác biệt này.

Sau khi xác định giá trị ban đầu, còn phải dùng test tải gần với traffic thực tế để điều chỉnh từng bước. Khi test tải đồng thời quan sát throughput, P95/P99 latency, CPU, chuyển đổi ngữ cảnh, thời gian chờ hàng đợi và connection pool downstream, không thể chỉ nhìn QPS ứng dụng.

## Hàng đợi hữu hạn nên thiết lập to bao nhiêu?

Hàng đợi chủ yếu dùng để hấp thụ biến động traffic trong thời gian ngắn, không phù hợp bảo tồn lâu dài các nhiệm vụ không kịp xử lý. Hàng đợi quá nhỏ, rung lắc nhẹ đã có thể kích hoạt từ chối; hàng đợi quá lớn, nhiệm vụ mặc dù được tiếp nhận, nhưng có thể trước khi thực sự thực thi đã vượt quá thời hạn timeout nghiệp vụ, còn chiếm dụng lượng lớn heap memory.

Có thể dùng lượng traffic bộc phát trước để ước tính lượng buffer cần thiết. $\lambda_{in}$ biểu thị tốc độ đi vào của nhiệm vụ, $\lambda_{out}$ biểu thị tốc độ hoàn thành, $\Delta t$ biểu thị thời gian kéo dài bộc phát:

$$
Q \approx (\lambda_{in} - \lambda_{out}) \times \Delta t
$$

Ví dụ, ThreadPool mỗi giây hoàn thành khoảng 500 nhiệm vụ, một lần cao điểm traffic nào đó trong 2 giây đẩy tốc độ submit lên 600 nhiệm vụ mỗi giây, vậy thời gian này sẽ dư ra khoảng 200 nhiệm vụ. Con số này chỉ thể hiện nhu cầu buffer ngắn hạn, dung lượng cuối cùng còn bị giới hạn bởi tính thời hiệu nhiệm vụ và bộ nhớ.

Thời gian chờ hàng đợi cũng có thể làm một phán đoán thô: Trong hàng đợi có 500 nhiệm vụ, ThreadPool mỗi giây hoàn thành 500 nhiệm vụ, nhiệm vụ ở cuối hàng đợi còn phải chờ khoảng 1 giây. Khi tổng timeout của API chỉ có 800 ms, lô nhiệm vụ này cho dù vào hàng đợi cũng khó trả về đúng giờ.

Sau khi worker thread đạt `corePoolSize`, nhiệm vụ mới sẽ đi vào hàng đợi trước; chỉ khi hàng đợi đầy, ThreadPool mới tiếp tục tạo luồng, cho đến `maximumPoolSize`. Khi hàng đợi được thiết lập rất lớn, số luồng có thể thời gian dài dừng ở số luồng cốt lõi, số luồng tối đa rất ít có cơ hội có hiệu lực. Dưới hàng đợi vô hạn, `maximumPoolSize` thực tế sẽ không tham gia mở rộng.

Khi xác định dung lượng hàng đợi, ít nhất phải làm 4 việc xác minh:

1. Dùng traffic cao điểm và thời gian kéo dài bộc phát ước tính nhu cầu buffer.
2. Kiểm tra thời gian chờ dự kiến của nhiệm vụ cuối hàng đợi có vượt quá thời hạn nghiệp vụ không.
3. Đo đạc bộ nhớ bị chiếm bởi các nhiệm vụ xếp hàng, đặc biệt là các nhiệm vụ mang theo file, request body hoặc collection lớn.
4. Xác minh xem hạ cấp và cảnh báo có hiệu lực không trong trường hợp hàng đợi đầy, nhiệm vụ bị từ chối.

## Tại sao số luồng phải thiết kế cùng với dung lượng downstream?

Không ít nhiệm vụ I/O cuối cùng sẽ truy cập database, Redis hoặc API bên ngoài. ThreadPool có thể cho nhiều nhiệm vụ đồng thời khởi xướng call, nhưng sẽ không làm tăng năng lực xử lý của downstream.

Giả sử order service deploy 4 instance, mỗi instance cấu hình 40 luồng cho một nhiệm vụ database nào đó, về mặt lý thuyết có thể đồng thời sinh ra 160 request database. Khi database connection pool mỗi instance chỉ có 20 connection, lượng lớn luồng sẽ bị chặn ở vị trí lấy connection; khi bản thân database chỉ có thể chịu đựng ổn định 80 query concurrent, cho dù tiếp tục mở rộng connection pool, cũng có thể chuyển áp lực sang database.

Khi cấu hình phải phân bổ ngân sách concurrency từ toàn bộ chuỗi gọi (calling chain):

- Connection database phải dự trù phần cho Web request, scheduled task và các ThreadPool nghiệp vụ khác, không thể giao toàn bộ cho một ThreadPool.
- Giới hạn per-route và tổng connection của HTTP client connection pool nên khớp với concurrency gọi thực tế.
- Khi downstream có ngưỡng Rate Limiting, tổng concurrency của tất cả instance upstream không thể thời gian dài vượt quá ngưỡng đó.
- Khi một nhiệm vụ truy cập tuần tự hoặc song song nhiều phụ thuộc, phải tính riêng thời gian chiếm dụng của từng phụ thuộc.

Sau khi số luồng tăng lên, nếu active connection, thời gian chờ lấy connection, P99 của downstream và error rate đồng thời tăng lên, việc tiếp tục mở rộng luồng thường không có trợ giúp. Lúc này nên giảm concurrency vô hiệu, xử lý slow SQL, downstream timeout hoặc tài nguyên hot-spot.

## Phải xử lý thế nào sau khi hàng đợi bị tích tụ?

Hàng đợi liên tục tăng trưởng, chứng tỏ tốc độ đi vào của nhiệm vụ đã vượt quá tốc độ hoàn thành. Traffic đột ngột tăng lên sẽ gây tích tụ, bản thân nhiệm vụ trở nên chậm cũng sẽ gây tích tụ, cách xử lý của hai cái là khác nhau.

| Hiện tượng | Bằng chứng cần đối chiếu | Hướng xử lý |
| --- | --- | --- |
| Tốc độ submit tăng, thời gian thực thi nhiệm vụ cơ bản không đổi | QPS đầu vào, traffic hoạt động, phía gọi thử lại | Rate limiting đầu vào, mở rộng instance hoặc dùng MQ削峰 (giảm đỉnh) |
| Tốc độ submit ổn định, thời gian thực thi nhiệm vụ tăng | Thread dump, slow SQL, chờ khóa, độ trễ downstream | Xử lý nhiệm vụ chậm và sự cố downstream, không vội tăng luồng |
| Luồng hoạt động đã đầy, CPU thời gian dài tiệm cận giới hạn | Tỷ lệ sử dụng CPU, hàng đợi chạy, chuyển đổi ngữ cảnh | Tối ưu tính toán, tách nhiệm vụ hoặc mở rộng máy |
| Luồng bị chặn lượng lớn chờ connection database | Số active connection pool, số chờ, timeout lấy connection | Giới hạn concurrency, tối ưu SQL, phân bổ lại ngân sách connection |
| Thời gian xếp hàng đã vượt quá thời hạn hiệu lực nhiệm vụ | P95/P99 chờ hàng đợi, timeout nghiệp vụ | Fail-fast, bỏ rơi nhiệm vụ hết hạn hoặc chuyển sang quy trình bù đắp |
| Hàng đợi và từ chối tăng, downstream vẫn còn dung lượng | Chỉ số CPU, connection pool và downstream bình thường | Điều chỉnh số luồng, hàng đợi hoặc số instance sau khi test tải |

Mở rộng máy phù hợp với trường hợp tổng dung lượng service không đủ và nhiệm vụ có thể tách theo chiều ngang (horizontal scale). Mở rộng luồng phù hợp với trường hợp đơn instance vẫn còn dư lượng CPU và downstream, nhưng độ concurrency hiện tại hơi thấp. Khi nhiệm vụ đã mất tính thời hiệu, tiếp tục xếp hàng không có nhiều ý nghĩa; nhiệm vụ bất đồng bộ không thể mất thì nên ghi vào MQ hoặc database trước, do quy trình tiêu thụ có thể thử lại xử lý.

Khi trên sản xuất đã xuất hiện tích tụ ThreadPool, có thể tham khảo [Troubleshooting sự cố Java backend online](../jvm/jvm-in-action.md#线程池队列堆积怎么排查), kết hợp tốc độ submit, tốc độ hoàn thành, thread dump và trạng thái downstream để định vị.

## CallerRunsPolicy có thể hình thành backpressure (ngược áp) không?

`CallerRunsPolicy` sẽ để luồng gọi `execute()` thực thi nhiệm vụ bị từ chối. Luồng submit do đó chậm lại, tốc độ submit nhiệm vụ sau đó cũng có thể giảm xuống, nên nó sở hữu khả năng điều tiết phản hồi nhất định.

Hiệu quả phụ thuộc vào người submit là ai. Luồng tiêu thụ message hoặc luồng điều phối batch job bị buộc thực thi nhiệm vụ, tốc độ kéo từ upstream có thể tự nhiên giảm xuống; khi luồng Tomcat request thực thi nhiệm vụ dài, luồng request sẽ bị chiếm giữ, độ trễ API và áp lực ThreadPool request cũng sẽ tăng lên. Khi luồng submit nắm giữ khóa hoặc transaction database, nhiệm vụ thực thi trong luồng gọi còn kéo dài thời gian chiếm dụng khóa và transaction.

Sau khi ThreadPool đóng, `CallerRunsPolicy` sẽ không thực thi nhiệm vụ nữa, mà trực tiếp bỏ rơi. Khi nghiệp vụ không thể chấp nhận mất nhiệm vụ, cần tùy chỉnh xử lý từ chối: Ghi lại số lần từ chối, trả về thất bại rõ ràng, hoặc ghi nhiệm vụ vào bộ nhớ tin cậy. Đừng chỉ đổi một chiến lược từ chối rồi để lại vấn đề độ tin cậy nhiệm vụ cho phía gọi.

## Các nghiệp vụ khác nhau có nên cách ly ThreadPool không?

Dùng chung ThreadPool sẽ làm cho một loại nhiệm vụ chậm chiếm dụng tài nguyên thực thi của các nghiệp vụ khác. Tính thời hiệu, cách xử lý thất bại, tài nguyên phụ thuộc của thanh toán callback, xuất báo cáo và thông báo thông thường là khác nhau, đặt trong cùng một ThreadPool rất khó cấu hình thống nhất số luồng, hàng đợi và chiến lược từ chối.

Cách ly cũng không phải là mỗi API đều tạo một ThreadPool. Thường phân chia theo độ ưu tiên nhiệm vụ, đặc trưng thời gian tiêu tốn và phụ thuộc downstream: Nhiệm vụ gọi cùng một service chậm có thể cách ly riêng; giao dịch cốt lõi và thông báo phi cốt lõi không dùng chung hàng đợi; nhiệm vụ CPU-intensive đừng trộn lẫn với lượng lớn nhiệm vụ I/O bị chặn. Quá nhiều ThreadPool sẽ làm tăng chi phí luồng, hàng đợi, giám sát và cấu hình, các nhiệm vụ cùng loại có thể tái sử dụng một pool.

Bài viết hiện có từng dẫn một sự cố thực tế về việc nhiệm vụ cha con dùng chung ThreadPool (Nguồn: [Một sự cố online do sử dụng ThreadPool không đúng cách](https://heapdump.cn/article/646639)):

![案例代码概览](https://oss.javaguide.cn/github/javaguide/java/concurrent/production-accident-threadpool-sharing-example.png)

Giả sử ThreadPool có $n$ worker thread, đồng thời chạy $n$ nhiệm vụ cha. Mỗi nhiệm vụ cha submit nhiệm vụ con xong, lại chờ đồng bộ nhiệm vụ con kết thúc. Nhiệm vụ con đi vào hàng đợi, nhưng không có luồng rảnh rỗi nào có thể thực thi; nhiệm vụ cha không kết thúc, worker thread cũng không giải phóng, cuối cùng hình thành deadlock do bỏ đói luồng.

![线程池使用不当导致死锁](https://oss.javaguide.cn/github/javaguide/java/concurrent/production-accident-threadpool-sharing-deadlock.png)

Nhiệm vụ cha và nhiệm vụ con mà nó chờ đồng bộ không nên sử dụng ThreadPool hữu hạn này để hình thành việc chờ lặp vòng. Có thể cho nhiệm vụ cha thực thi trực tiếp logic con, đổi thành phối hợp nhiệm vụ không bị chặn, hoặc phân bổ tài nguyên thực thi độc lập cho các nhiệm vụ con thực sự cần cách ly.

## Nhiệm vụ sau khi timeout có tự động dừng không?

`Future.get(timeout, unit)` chỉ giới hạn thời gian luồng gọi chờ kết quả. Khi ném ra `TimeoutException`, nhiệm vụ background có thể vẫn đang chạy. Gọi `cancel(true)` có thể thử ngắt luồng thực thi, nhưng ngắt là cơ chế hợp tác (cooperative mechanism), nhiệm vụ không kiểm سرا trạng thái ngắt, hoặc lời gọi bên dưới không phản hồi ngắt, nhiệm vụ vẫn có thể tiếp tục thực thi.

```java
Future<String> future = executor.submit(this::callRemoteService);

try {
    return future.get(200, TimeUnit.MILLISECONDS);
} catch (TimeoutException e) {
    future.cancel(true);
    throw new IllegalStateException("Gọi timeout", e);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    throw new IllegalStateException("Trong lúc chờ nhiệm vụ bị ngắt", e);
} catch (ExecutionException e) {
    throw new IllegalStateException("Nhiệm vụ thực thi thất bại", e.getCause());
}
```

Đoạn code này vẫn chưa đủ để thay thế timeout mạng. Client HTTP, database và Redis vẫn phải cấu hình connection, read và total call timeout, nếu không luồng có thể bị kẹt mãi ở thao tác cấp thấp không phản hồi ngắt.

Code nhiệm vụ sau khi nhận được `InterruptedException`, thường phải kết thúc công việc hiện tại; nếu không thể kết thúc ở layer hiện tại, nên khôi phục cờ ngắt, để layer phía trên tiếp tục xử lý:

```java
try {
    blockingCall();
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    return;
}
```

`CompletableFuture.orTimeout()` sẽ làm cho `CompletableFuture` sau khi timeout hoàn thành bất thường bằng ngoại lệ, nhưng sẽ không tự động chấm dứt nhiệm vụ bên dưới. Tham số của `CompletableFuture.cancel(true)` trong cách triển khai đó cũng sẽ không kích hoạt ngắt luồng. Khi dùng `CompletableFuture` sắp xếp nhiệm vụ bị chặn, vẫn phải cấu hình phương án timeout và cancel cho lời gọi bên dưới.

## Tại sao ngoại lệ của nhiệm vụ bất đồng bộ lại dễ bị mất?

`Runnable` submit qua `execute()` khi ném ra unhandled exception, worker thread sẽ kết thúc bất thường, có thể do `UncaughtExceptionHandler` của luồng ghi lại. Khi submit qua `submit()`, nhiệm vụ thường được bọc thành `FutureTask`, ngoại lệ được lưu trong `Future` trả về; phía gọi nếu vừa không lưu `Future`, vừa không gọi `get()`, ngoại lệ có thể sẽ không có log nghiệp vụ.

`ThreadPoolExecutor.afterExecute()` cũng có sự khác biệt tương tự. Khi dùng `submit()`, `Throwable` truyền cho `afterExecute()` thường là `null`, cần phán đoán nhiệm vụ có phải là `Future` đã hoàn thành hay không, rồi thông qua `get()` lấy ngoại lệ.

Cách xử lý ngoại lệ tốt nhất nên thống nhất trong project: Phía gọi tiêu thụ `Future`, đầu vào nhiệm vụ chủ động bắt và ghi log ngoại lệ, hoặc mở rộng ThreadPool để xử lý tập trung. Log phải mang theo loại nhiệm vụ, ID nghiệp vụ và Trace ID, không thể chỉ in một đoạn stack trace tách rời nghiệp vụ.

## ThreadPool nên giám sát những chỉ số nào?

Thông tin thống kê đi kèm của `ThreadPoolExecutor` có thể thấy số luồng hiện tại, số luồng active, số luồng tối đa lịch sử, tổng số nhiệm vụ, số nhiệm vụ hoàn thành và chiều dài hàng đợi. Các giá trị này phần lớn là giá trị xấp xỉ, phù hợp cho giám sát và phân tích xu hướng, không phù hợp tham gia vào phán đoán nghiệp vụ yêu cầu tính nhất quán nghiêm ngặt.

Giám sát trên sản xuất ít nhất nên bao phủ:

- **Traffic nhiệm vụ**: Tốc độ submit, tốc độ bắt đầu thực thi và tốc độ hoàn thành.
- **Trạng thái luồng**: Số luồng hiện tại, số luồng active, số luồng tối đa lịch sử và độ active.
- **Trạng thái hàng đợi**: Chiều dài hiện tại, tổng dung lượng, tỷ lệ sử dụng và thời gian chờ hàng đợi.
- **Kết quả nhiệm vụ**: Thời gian thực thi, số thành công, số ngoại lệ, số timeout, số cancel và số bị từ chối.
- **Tài nguyên liên quan**: CPU, heap memory, database connection pool, HTTP connection pool và độ trễ downstream.

Tổng số nhiệm vụ và số nhiệm vụ hoàn thành là giá trị tích lũy, cần tính giá trị chênh lệch trong một khoảng thời gian mới ra được tốc độ (rate). Chiều dài hàng đợi cũng phải kết hợp với xu hướng thay đổi: Hàng đợi đều là 100, một cái đang giảm từ 500 xuống 100, cái khác tăng từ 10 lên 100, rủi ro là không giống nhau.

Thời gian xếp hàng và thời gian thực thi không thể trộn thành một chỉ số. Thời gian xếp hàng tăng, thời gian thực thi ổn định, phần nhiều là dung lượng không đủ hoặc traffic tăng đột ngột; thời gian thực thi tăng trước, sau đó hàng đợi bắt đầu tăng, nên ưu tiên kiểm tra logic nhiệm vụ và downstream. Có thể ghi lại timestamp khi submit nhiệm vụ, khi nhiệm vụ thực sự bắt đầu và kết thúc tính riêng thời gian tiêu tốn của 2 giai đoạn.

Hàng đợi trong một chu kỳ thu thập đột nhiên dài ra, có thể chỉ là đỉnh traffic (spike), sẽ sớm hạ xuống. Cảnh báo ngoài ngưỡng ra, còn nên thêm thời gian kéo dài, và xác nhận xem tích tụ có thể tự tiêu hóa không. Hàng đợi thời gian dài ở mức nước cao, tốc độ hoàn thành liên tục thấp hơn tốc độ submit, chứng tỏ nhiệm vụ cũ ngày càng tích tụ nhiều; thời gian xếp hàng P99 tiến gần thời gian nhiệm vụ có thể chờ đợi, cho dù hàng đợi chưa đầy, cũng nên cảnh báo. Số lần từ chối phải thống kê riêng. Nhiệm vụ cốt lõi xuất hiện từ chối cảnh báo ngay lập tức, nhiệm vụ cho phép bỏ rơi đặt ngưỡng theo tỷ lệ từ chối và thời gian kéo dài.

## Điều chỉnh động tham số có thể giải quyết vấn đề gì?

Thời gian tiêu tốn nhiệm vụ không có thay đổi rõ rệt, CPU, connection pool và downstream vẫn còn dư lượng, chỉ là độ concurrency hiện tại không theo kịp tốc độ submit, lúc này có thể điều chỉnh số luồng. Nếu các luồng đều bị kẹt ở connection database, slow SQL hoặc downstream timeout, điều chỉnh to số luồng sẽ chỉ sinh ra thêm một lô người ngồi chờ.

Một số tham số của `ThreadPoolExecutor` có thể sửa đổi online, tuy nhiên cách thức có hiệu lực không hoàn toàn giống nhau. Trong thời gian chạy có thể sửa `corePoolSize`, `maximumPoolSize`, `keepAliveTime`, chiến lược từ chối và ThreadFactory. Sau khi điều chỉnh lớn số luồng cốt lõi, chỉ cần trong hàng đợi còn nhiệm vụ, ThreadPool sẽ tạo luồng mới theo nhu cầu. Số luồng cốt lõi hoặc số luồng tối đa điều chỉnh nhỏ, sẽ không ngắt các nhiệm vụ đang thực thi; các worker thread dư ra sẽ thoát sau khi rảnh rỗi. ThreadFactory mới thiết lập chỉ ảnh hưởng đến các luồng tạo ra sau đó, không đổi tên các worker thread đã tồn tại.

Khi sửa đổi số luồng cốt lõi và số luồng tối đa cần lưu ý thứ tự:

- Số luồng cốt lõi mới cao hơn số luồng tối đa cũ, điều chỉnh lớn số luồng tối đa trước, rồi mới điều chỉnh số luồng cốt lõi.
- Số luồng tối đa mới thấp hơn số luồng cốt lõi cũ, điều chỉnh nhỏ số luồng cốt lõi trước, rồi mới điều chỉnh số luồng tối đa.

Nếu không, kiểm tra tham số sẽ ném ra `IllegalArgumentException`.

JDK không cung cấp phương thức chung để sửa đổi dung lượng hàng đợi chặn hiện có. Khi cần điều chỉnh hàng đợi, có thể sử dụng hàng đợi dung lượng biến đổi hoặc framework dynamic thread pool đã qua xác minh; tự sửa đổi triển khai `LinkedBlockingQueue` phải đồng thời xử lý tính nhìn thấy concurrency, điều kiện vào hàng đợi và logic đánh thức, không thể chỉ sửa một field capacity.

Đội ngũ kỹ thuật Meituan trong [《Nguyên lý triển khai ThreadPool trong Java và thực tiễn trong nghiệp vụ Meituan》](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html) đã từng giới thiệu tư tưởng triển khai cấu hình động tham số ThreadPool và giám sát cảnh báo. Trong các phương án mã nguồn mở, [Hippo4j](https://github.com/opengoofy/hippo4j) và [Dynamic TP](https://github.com/dromara/dynamic-tp) đều cung cấp năng lực cấu hình động, giám sát và cảnh báo. Trước khi đưa vào vẫn phải đối chiếu loại ThreadPool, config center, phiên bản framework và phương thức hạ cấp sự cố mà project sử dụng.

Trước cao điểm nghiệp vụ điều chỉnh trước tham số, tốt nhất lấy chỉ số lịch sử hoặc kết quả test tải làm căn cứ. Sau khi điều chỉnh, nếu tốc độ hoàn thành không tăng lên, ngược lại xuất hiện CPU tăng, connection pool chờ hoặc error rate downstream tăng, nên rollback tham số, tiếp tục kiểm tra bản thân nhiệm vụ và phụ thuộc.

Khi sửa đổi online nên giữ lại giới hạn trên dưới của tham số, bản ghi thay đổi, phạm vi canary (canary scope) và giá trị rollback. Mỗi lần cố gắng chỉ sửa một biến chính, quan sát tốc độ hoàn thành nhiệm vụ, thời gian xếp hàng, số từ chối và áp lực downstream, rồi mới quyết định có tiếp tục điều chỉnh hay không.

## Còn có những vấn đề nào dễ bị bỏ qua?

### Đừng tạo đi tạo lại ThreadPool

ThreadPool dùng để tái sử dụng luồng, không nên tạo mới trong mỗi request hoặc mỗi lần gọi phương thức. Tạo thường xuyên sẽ làm tăng chi phí khởi động và tiêu hủy luồng, cũng dễ bỏ sót logic đóng. ThreadPool cấp ứng dụng thường giao cho container quản lý thống nhất lifecycle.

`@Async` của Spring, scheduled task, Web container và một số client bên trong cũng sẽ dùng ThreadPool. Project không gọi hiển thị `new ThreadPoolExecutor()`, không đại diện cho việc không tồn tại ThreadPool cần cấu hình và giám sát.

### Đừng để nhiệm vụ dài chiếm đầy ThreadPool dùng chung

Nhiệm vụ bị chặn thời gian dài sẽ chiếm dụng worker thread, các nhiệm vụ ngắn phía sau chỉ có thể xếp hàng. Xuất báo cáo, xử lý file và API bên ngoài chậm phù hợp dùng tài nguyên thực thi độc lập, hoặc đổi thành nhiệm vụ bất đồng bộ và trả về trạng thái nhiệm vụ cho user.

`CompletableFuture` chỉ chịu trách nhiệm sắp xếp nhiệm vụ, không làm cho request mạng dạng chặn biến thành thao tác non-blocking. Phương thức bất đồng bộ không chỉ định hiển thị `Executor` thường dùng ThreadPool công cộng, ThreadPool công cộng sau khi bị chặn, còn có thể ảnh hưởng đến các nhiệm vụ bất đồng bộ khác trong ứng dụng.

### Dọn dẹp Thread Context

ThreadPool sẽ tái sử dụng luồng. Nhiệm vụ trước ghi vào `ThreadLocal` xong không dọn dẹp, nhiệm vụ sau có thể đọc được giá trị cũ trên cùng luồng đó, còn có thể làm cho đối tượng lớn thời gian dài bị worker thread tham chiếu đến.

Truyền và dọn dẹp context nên do wrapper nhiệm vụ thống nhất xử lý, khôi phục hoặc xóa giá trị gốc trong `finally`. Log MDC, thông tin đăng nhập và thông tin tenant đều phải cân nhắc vấn đề này. Kịch bản truyền xuyên luồng cũng có thể dùng [TransmittableThreadLocal](https://github.com/alibaba/transmittable-thread-local), nhưng vẫn phải xác nhận phương thức wrap và thời điểm dọn dẹp.

### Đóng ThreadPool đúng cách

`shutdown()` không tiếp nhận nhiệm vụ mới nữa, sẽ tiếp tục xử lý các nhiệm vụ đã submit; `shutdownNow()` sẽ thử ngắt các nhiệm vụ đang thực thi, và trả về các nhiệm vụ chưa bắt đầu thực thi. Cả hai đều sẽ không chờ ThreadPool chấm dứt hoàn toàn.

```java
executor.shutdown();
try {
    if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
        executor.shutdownNow();
        if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
            System.err.println("ThreadPool không thể thoát bình thường");
        }
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}
```

Nhiệm vụ cần phản hồi ngắt đúng cách, nếu không `shutdownNow()` cũng không thể đảm bảo dừng ngay lập tức. Trong thời gian ứng dụng đóng còn phải quyết định các nhiệm vụ trong hàng đợi có thể mất không; các nhiệm vụ nghiệp vụ không thể mất không nên chỉ tồn tại trong tiến trình bộ nhớ.

## Sau khi dùng Virtual Thread có còn cần ThreadPool truyền thống không?

Virtual Thread phù hợp với các nhiệm vụ chứa lượng lớn chặn chờ đợi. Chi phí tạo và switch của nó thấp hơn Platform Thread, thường áp dụng cách mỗi nhiệm vụ một Virtual Thread, đừng đặt Virtual Thread vào pool kích thước cố định để tái sử dụng.

Khi cần giới hạn concurrency của database hoặc API downstream, tiếp tục sử dụng connection pool, `Semaphore`, Rate Limiter... để ràng buộc tài nguyên cụ thể. Số lượng Virtual Thread rất nhiều, cũng sẽ không làm tăng số lượng connection database, số core CPU hay dung lượng API downstream.

ThreadPool Platform Thread truyền thống vẫn áp dụng cho các nhiệm vụ CPU-intensive, nhiệm vụ cần luồng điều phối cố định, và các component bắt buộc phải kiểm soát hiển thị Platform Thread và hàng đợi công việc. Project có migrate hay không còn phải xem phiên bản JDK, hỗ trợ framework, công cụ giám sát và async model hiện có. Giải thích chi tiết có thể tham khảo [Tổng kết các câu hỏi thường gặp về Virtual Thread](./virtual-thread.md).

## Trả lời cấu hình tham số ThreadPool như thế nào trong phỏng vấn?

Khi trả về tham số ThreadPool, có thể xuất phát từ nhiệm vụ và dung lượng hệ thống: Trước tiên dựa vào tốc độ submit nhiệm vụ cao điểm và thời gian tiêu tốn nhiệm vụ để ước tính nhu cầu concurrency, rồi phân biệt thời gian tính toán và thời gian chờ I/O; số luồng còn bị ràng buộc bởi CPU, database connection pool, HTTP connection pool và Rate Limiting của downstream. Hàng đợi chỉ hấp thụ bộc phát ngắn hạn, dung lượng phải đồng thời thỏa mãn thời hạn chờ nghiệp vụ và giới hạn bộ nhớ. Sau khi xác định tham số ban đầu, thông qua test tải và giám sát online quan sát tốc độ submit, tốc độ hoàn thành, thời gian xếp hàng, thời gian thực thi và số lần từ chối, rồi mới điều chỉnh từng bước.

Khi người phỏng vấn tiếp tục hỏi sâu về hàng đợi bị tích tụ, đừng chỉ trả lời mở rộng ThreadPool. Trước tiên so sánh tốc độ submit nhiệm vụ và tốc độ hoàn thành, rồi xem thời gian thực thi, thread dump, CPU, connection pool và độ trễ downstream. Traffic tăng có thể Rate limiting hoặc mở rộng instance; nhiệm vụ chậm lại phải xử lý SQL, khóa hoặc sự cố downstream; nhiệm vụ đã hết hạn nên fail-fast hoặc chuyển sang bù đắp. Điều chỉnh động tham số chỉ xử lý việc cấu hình dung lượng không phù hợp, không thể thay thế timeout, cách ly, Rate limiting và quản trị downstream.

## Tham khảo

- [Tài liệu API ThreadPoolExecutor](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)
- [Tài liệu API Future](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Future.html)
- [Tài liệu API CompletableFuture](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
- [JEP 444: Virtual Threads](https://openjdk.org/jeps/444)
- [Nguyên lý triển khai ThreadPool Java và thực tiễn trong nghiệp vụ Meituan](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- 《Java Concurrency in Practice》

<!-- @include: @article-footer.snippet.md -->
