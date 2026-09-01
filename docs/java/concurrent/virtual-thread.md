---
title: 虚拟线程常见问题总结
description: Java 21 虚拟线程详解：梳理 Virtual Threads 的定位、调度原理、与平台线程的区别、适用场景、创建方式、性能边界、Spring Boot 接入方式和实践注意事项。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: Java虚拟线程,Virtual Threads,Project Loom,Java 21新特性,轻量级线程,协程,虚拟线程原理
---

<!-- @include: @article-header.snippet.md -->

Một Web request đi vào, code phải query database, gọi API từ xa, đọc ghi file. Theo cách viết đồng bộ truyền thống, request này sẽ chiếm giữ một Platform Thread (luồng nền tảng), cho dù phần lớn thời gian đều đang chờ I/O.

ThreadPool có thể giảm bớt chi phí tạo luồng, nhưng không thể thay đổi một thực tế: Số lượng Platform Thread vẫn chịu sự giới hạn bởi số lượng luồng hệ điều hành, bộ nhớ và chi phí điều phối (scheduling cost). Khi concurrent request tiếp tục tăng lên, các luồng trong ThreadPool sẽ bị các nhiệm vụ xếp hàng chiếm đầy, throughput (năng suất) nhanh chóng bị nghẽn.

Virtual Thread (Luồng ảo) ra đời chính là vì bài toán này. Nó cho phép chúng ta tiếp tục sử dụng code đồng bộ bị chặn đơn giản, đồng thời làm cho các nhiệm vụ chờ I/O không còn chiếm giữ thời gian dài những Platform Thread đắt đỏ nữa.

## Virtual Thread là gì?

Virtual Thread (Luồng ảo) là một loại luồng mỏng nhẹ (lightweight thread) được chính thức đưa vào từ Java 21, cũng là một triển khai của `java.lang.Thread`. Nó do JDK quản lý và điều phối, chứ không liên kết trực tiếp một-đối-một với một luồng hệ điều hành nào đó.

Platform Thread (Luồng nền tảng) thường là lớp bọc mỏng (thin wrapper) đối với luồng hệ điều hành. Một Platform Thread khi chạy sẽ chiếm dụng một luồng hệ điều hành trong suốt vòng đời của nó. Virtual Thread thì khác: Khi nó chạy code Java cần gắn (mount) vào một Platform Thread nào đó; khi nó thực thi thao tác bị chặn có thể treo (suspendable blocking operation), JDK có thể tháo (unmount) nó ra khỏi Platform Thread, để Platform Thread đó đi thực thi Virtual Thread khác.

Do đó, số lượng Virtual Thread có thể lớn hơn rất nhiều số lượng Platform Thread. Tài liệu chính thức dùng Virtual Memory (Bộ nhớ ảo) để ví von: Hệ điều hành ánh xạ lượng lớn địa chỉ ảo vào bộ nhớ vật lý hữu hạn, còn Java runtime ánh xạ lượng lớn Virtual Thread vào số lượng Platform Thread ít hơn.

Một vài điểm mấu chốt của Virtual Thread:

- Virtual Thread vẫn là `Thread`, hỗ trợ `ThreadLocal`, ngắt (interrupt), stack ngoại lệ, debugging và JFR (Java Flight Recorder) observation.
- Virtual Thread phù hợp với lượng lớn nhiệm vụ chờ đợi bị chặn, như gọi HTTP, query database, truy cập message queue, file hoặc network I/O.
- Virtual Thread không phải là đơn vị thực thi CPU nhanh hơn, không làm cho một đoạn code thuần tính toán chạy nhanh hơn.
- Virtual Thread rất rẻ, thường nên "mỗi nhiệm vụ một Virtual Thread", chứ không phải tái sử dụng theo dạng pool (ThreadPool) như Platform Thread.

## Virtual Thread và Platform Thread có mối quan hệ gì?

Trong Java, Virtual Thread, Platform Thread và OS Thread (luồng hệ điều hành) có mối quan hệ đại thể như thế này:

![虚拟线程、平台线程和系统内核线程的关系](https://oss.javaguide.cn/github/javaguide/java/new-features/virtual-threads-platform-threads-kernel-threads-relationship.png)

Trong các hệ điều hành phổ biến như Windows, Linux, Platform Thread của HotSpot JVM thường áp dụng mô hình luồng một-đối-một, tức là một Platform Thread tương ứng với một OS Thread. Sau khi đưa vào Virtual Thread, JDK ở trên Platform Thread lại thêm một tầng điều phối:

- Virtual Thread là vật mang của nhiệm vụ, code nghiệp vụ nhìn thấy `Thread.currentThread()` trả về chính là bản thân Virtual Thread.
- Platform Thread là vật chứa của Virtual Thread (Carrier Thread), chịu trách nhiệm thực sự thực thi code Java trong Virtual Thread.
- Hệ điều hành vẫn chỉ điều phối Platform Thread, không biết sự tồn tại của Virtual Thread.

Một Virtual Thread khi bắt đầu thực thi, sẽ được JDK scheduler gắn (mount) vào một Platform Thread nào đó. Thực thi đến các điểm bị chặn hỗ trợ treo như I/O bị chặn, `BlockingQueue.take()`, `Future.get()`..., Virtual Thread có thể tháo (unmount), Platform Thread được giải phóng ra để tiếp tục thực thi Virtual Thread khác. Chờ sau khi thao tác bị chặn sẵn sàng, Virtual Thread lại được submit trở lại scheduler, gắn vào một Platform Thread nào đó để tiếp tục thực thi.

Quá trình gắn và tháo này là trong suốt đối với code nghiệp vụ. Code bạn viết vẫn là code đồng bộ thông thường:

```java
String body = httpClient.send(request, BodyHandlers.ofString()).body();
Result result = repository.query(body);
return service.handle(result);
```

Nếu bên trong các lời gọi này xảy ra bị chặn, Virtual Thread có thể tự treo mình; nếu đổi thành Platform Thread, luồng này sẽ liên tục chiếm giữ luồng hệ điều hành tương ứng.

## Project Loom và Virtual Thread có mối quan hệ gì?

Project Loom là project cải tiến mô hình concurrency trong OpenJDK, Virtual Thread là một trong những thành quả quan trọng nhất của Loom. Virtual Thread lần lượt được preview trong JDK 19, JDK 20, cuối cùng thông qua [JEP 444](https://openjdk.org/jeps/444) được chính thức công nhận trong JDK 21.

Loom không chỉ thêm một API luồng mỏng nhẹ. Nó còn thúc đẩy việc điều chỉnh các năng lực đi kèm của JDK như I/O bị chặn, debugging, JFR, thread dump..., làm cho phong cách lập trình thread-per-request truyền thống trong kịch bản I/O concurrency cao một lần nữa có thể mở rộng (scalable).

Đây cũng là điểm khác biệt quan trọng giữa Virtual Thread và "thuật toán Coroutine" thông thường: Virtual Thread được đưa vào mô hình luồng của Java platform. Debugger, Profiler, JFR, thread dump đều có thể hiểu nó lấy đơn vị là luồng, chứ không phải tách chuỗi gọi nghiệp vụ thành một đống các giai đoạn callback.

## Virtual Thread giải quyết vấn đề gì?

Rất nhiều server application tự nhiên phù hợp với mô hình "một request một luồng". Lợi ích của nó rất rõ ràng: Code thực thi theo thứ tự, ngoại lệ có thể ném ra theo call stack, debugger có thể đi vào từng bước một, thread dump cũng có thể thấy request bị kẹt ở đâu.

Vấn đề nằm ở chỗ Platform Thread quá đắt.

Giả sử một API thời gian tiêu tốn trung bình 50ms, hệ thống muốn đạt 2000 QPS, tính toán thô theo Little's Law, cần đồng thời xử lý khoảng 100 request. Nếu API thời gian tiêu tốn trung bình biến thành 500ms, cùng 2000 QPS sẽ cần khoảng 1000 concurrent request. Mỗi request đều chiếm một Platform Thread, số lượng luồng rất dễ trở thành nút thắt trước cả các tài nguyên như CPU, băng thông mạng, kết quả nối database.

Lập trình bất đồng bộ (Async programming), Reactive programming có thể giải phóng luồng khỏi việc chờ I/O, nhưng cái giá phải trả cũng rất rõ ràng: Chuỗi lời gọi bị tách thành callback, chuỗi `CompletableFuture` hoặc pipeline响应式, xử lý ngoại lệ, debugging, flame graph và context luồng đều sẽ trở nên phức tạp.

Virtual Thread cố gắng giữ lại tính dễ đọc của code đồng bộ, đồng thời giảm chi phí chiếm dụng Platform Thread khi bị chặn chờ đợi. Cái nó nâng cao chủ yếu là năng lực throughput và năng lực gánh concurrency, chứ không phải tốc độ thực thi của một request đơn lẻ.

## Virtual Thread phù hợp với kịch bản nào?

Virtual Thread phù hợp nhất với các loại nhiệm vụ dưới đây:

- Số lượng nhiệm vụ concurrent rất nhiều, thường ở mức hàng ngàn hàng vạn.
- Nhiệm vụ phần lớn thời gian đang chờ I/O, như database, Redis, HTTP/RPC, message queue, file và đọc ghi mạng.
- Code hiện tại chủ yếu là mô hình đồng bộ bị chặn, không muốn vì tính mở rộng mà đổi thành chuỗi bất đồng bộ phức tạp.
- Hy vọng giữ lại call stack truyền thống, thuận tiện cho debugging, phân tích test tải và troubleshooting online.

Các kịch bản điển hình bao gồm:

- Trong API Spring MVC / Servlet gọi database và external HTTP service.
- Task background gọi theo batch API bên thứ ba.
- Gateway hoặc aggregation service gọi đồng thời nhiều downstream service.
- Trong logic tiêu thụ message chứa việc ghi database bị chặn hoặc gọi từ xa.

Virtual Thread không phù hợp để làm cho task CPU-intensive "chạy nhanh hơn". Nếu task chủ yếu là tính hash, nén ảnh, sắp xếp mảng lớn, chạy rule engine phức tạp, sau khi số lượng luồng vượt quá số core CPU, throughput thường sẽ không tiếp tục tăng lên. Công việc CPU-intensive vẫn nên tập trung vào thuật toán, cấu trúc dữ liệu, xử lý batch, parallel stream, ThreadPool tính toán chuyên dụng hoặc tối ưu hóa bản địa.

## Tạo Virtual Thread như thế nào?

Trong JDK 21 có 4 cách tạo thường gặp.

### Sử dụng `Thread.startVirtualThread()`

Phù hợp để khởi động một Virtual Thread rất đơn giản:

```java
public class VirtualThreadDemo {
  public static void main(String[] args) throws InterruptedException {
    Thread thread = Thread.startVirtualThread(() -> {
      System.out.println(Thread.currentThread());
    });

    thread.join();
  }
}
```

Cần lưu ý là, Virtual Thread là daemon thread. Nếu phương thức `main` không chờ nó kết thúc, JVM có thể trực tiếp thoát, dẫn đến task chưa kịp thực thi xong.

### Sử dụng `Thread.ofVirtual()`

`Thread.ofVirtual()` trả về một `Thread.Builder.OfVirtual`, có thể thiết lập tên luồng, cũng có thể chọn tạo xong khởi động ngay hoặc chưa khởi động:

```java
public class VirtualThreadDemo {
  public static void main(String[] args) throws InterruptedException {
    Thread unstarted = Thread.ofVirtual()
        .name("order-query")
        .unstarted(() -> System.out.println("query order"));

    unstarted.start();
    unstarted.join();

    Thread started = Thread.ofVirtual()
        .name("payment-query")
        .start(() -> System.out.println("query payment"));

    started.join();
  }
}
```

### Sử dụng `ThreadFactory`

Nếu bạn hy vọng thống nhất đặt tên luồng, hoặc giao thread factory cho framework sử dụng, có thể thông qua `ThreadFactory` tạo Virtual Thread:

```java
import java.util.concurrent.ThreadFactory;

public class VirtualThreadDemo {
  public static void main(String[] args) throws InterruptedException {
    ThreadFactory factory = Thread.ofVirtual()
        .name("worker-", 0)
        .factory();

    Thread thread = factory.newThread(() -> {
      System.out.println(Thread.currentThread().getName());
    });

    thread.start();
    thread.join();
  }
}
```

### Sử dụng `Executors.newVirtualThreadPerTaskExecutor()`

Trong phát triển nghiệp vụ cách này là thường gặp nhất. Nó sẽ tạo một Virtual Thread mới cho mỗi task được submit:

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class VirtualThreadDemo {
  public static void main(String[] args) throws Exception {
    try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
      Future<String> future = executor.submit(() -> {
        return "hello virtual thread";
      });

      System.out.println(future.get());
    }
  }
}
```

`ExecutorService` ở đây không phải là ThreadPool theo nghĩa truyền thống. Nó sẽ không bảo trì một nhóm Virtual Thread cố định để tái sử dụng, mà là mỗi task một Virtual Thread mới. Khi `try-with-resources` kết thúc sẽ gọi `close()`, chờ các task đã submit hoàn thành.

## Virtual Thread có nên đưa vào pool không?

Không đưa Virtual Thread vào pool (do not pool virtual threads).

Mục tiêu chính của ThreadPool là tái sử dụng các Platform Thread đắt đỏ, và sẵn tiện giới hạn concurrency. Bản thân Virtual Thread không phải tài nguyên hiếm, việc đưa chúng vào pool thường không có ý nghĩa, mà còn làm cho mô hình "mỗi task một luồng" quay trở lại tư duy cũ.

Nếu mục tiêu thực sự của bạn là giới hạn lượng concurrency truy cập một tài nguyên nào đó, nên giới hạn tài nguyên, chứ không phải giới hạn số lượng Virtual Thread. Ví dụ một hệ thống cũ nào đó tối đa chỉ chịu được 20 concurrent request, có thể dùng `Semaphore` kiểm soát concurrency:

```java
import java.util.concurrent.Semaphore;

public class OldServiceClient {
  private static final Semaphore LIMIT = new Semaphore(20);

  public String call() throws InterruptedException {
    LIMIT.acquire();
    try {
      return doCall();
    } finally {
      LIMIT.release();
    }
  }

  private String doCall() {
    return "ok";
  }
}
```

Nếu nút thắt là kết nối database, vậy thì điều chỉnh kích thước connection pool; nếu nút thắt là Rate Limiting của downstream, vậy thì làm Rate Limiting, Circuit Breaking (ngắt mạch) và Retry Backoff. Virtual Thread có thể làm cho việc chờ đợi trở nên rẻ hơn, nhưng không thể làm cho kết nối database, dung lượng downstream, CPU và bộ nhớ trở thành vô hạn.

## So sánh hiệu năng giữa Virtual Thread và Platform Thread

Đưa ra kết luận trước: Virtual Thread không phải là "luồng chạy nhanh hơn", mà là "luồng có thể tạo rất nhiều, chi phí bị chặn thấp hơn". Nó thường có thể nâng cao throughput của I/O-intensive service, nhưng sẽ không làm giảm thời gian tiêu tốn bản thân của một lần query database hay một lần gọi HTTP.

Ví dụ dưới đây giả lập 10,000 nhiệm vụ bị chặn 1 giây:

```java
import java.time.Duration;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.stream.IntStream;

public class VirtualThreadCompareDemo {
  public static void main(String[] args) {
    long start = System.currentTimeMillis();

    try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
      IntStream.range(0, 10_000).forEach(i -> {
        executor.submit(() -> {
          Thread.sleep(Duration.ofSeconds(1));
          return i;
        });
      });
    }

    System.out.println("cost: " + (System.currentTimeMillis() - start) + "ms");
  }
}
```

Nếu đổi nó thành `Executors.newFixedThreadPool(200)`, cùng một thời điểm tối đa chỉ có 200 task đang thực thi, 10,000 task sẽ bị xử lý theo từng đợt. Ước tính thô theo mỗi đợt 1 giây, tổng thời gian tiêu tốn gần 50 giây. Phiên bản Virtual Thread có thể làm cho 10,000 task này gần như đồng thời đi vào trạng thái chờ, Platform Thread trong thời gian chờ đợi được giải phóng ra, tổng thời gian tiêu tốn gần hơn với thời gian chờ của một task đơn lẻ.

Ví dụ này chỉ thể hiện Virtual Thread thân thiện với "bị chặn chờ đợi", không phải là benchmark test nghiêm ngặt. Service thực tế phải xem các yếu tố như database connection pool, HTTP client connection pool, downstream rate limiting, GC, phân bổ đối tượng, tranh chấp khóa, container CPU quota...

## Nguyên lý bên dưới của Virtual Thread là gì?

Có thể chia quá trình thực thi của Virtual Thread thành 3 việc: Scheduling (Điều phối), Mounting/Unmounting (Gắn/Tháo), Stack Management (Quản lý stack).

### Scheduling (Điều phối)

Platform Thread phụ thuộc vào điều phối của hệ điều hành. Virtual Thread do scheduler của bản thân JDK điều phối, rồi mới do Platform Thread gánh thực thi. Trong JEP 444 thuyết minh, Virtual Thread scheduler là một work-stealing `ForkJoinPool` áp dụng FIFO mode, nó và common pool mà parallel stream sử dụng không phải là cùng một pool.

Trong trường hợp mặc định, độ song song của scheduler có liên quan đến số lượng processor khả dụng, có thể điều chỉnh thông qua các system property dưới đây:

- `jdk.virtualThreadScheduler.parallelism`: Độ song song mục tiêu của scheduler.
- `jdk.virtualThreadScheduler.maxPoolSize`: Ngưỡng trên Platform Thread có thể mở rộng của scheduler.

Hầu hết các hệ thống nghiệp vụ không cần sửa hai tham số này. Ưu tiên rà soát connection pool, Rate Limiting, lock và điểm bị chặn, thường hiệu quả hơn việc điều chỉnh tham số scheduler.

### Mounting và Unmounting (Gắn và Tháo)

Virtual Thread khi thực thi code Java, sẽ gắn vào một Platform Thread nào đó. Khi gặp thao tác bị chặn hỗ trợ treo, Virtual Thread có thể lưu trạng thái thực thi hiện tại và tháo ra, Platform Thread tiếp tục phục vụ Virtual Thread khác.

Các thao tác bị chặn thường gặp của JDK đã được làm tương thích cho Virtual Thread. Ví dụ network I/O, `BlockingQueue`, `Future.get()`... khi bị chặn trong Virtual Thread thường sẽ không chiếm giữ thời gian dài Platform Thread bên dưới.

Không phải tất cả các thao tác bị chặn đều có thể tháo. Trong JDK 21 đến JDK 23, Virtual Thread bị chặn trong khối code hoặc phương thức `synchronized`, sẽ xuất hiện Pinning, tức là bị cố định trên carrier thread. [JEP 491](https://openjdk.org/jeps/491) của JDK 24 đã cải tiến điểm này, làm cho Virtual Thread khi bị chặn trong `synchronized` cũng có thể giải phóng Platform Thread bên dưới, loại bỏ tuyệt đại đa số kịch bản Pinning do `synchronized` mang lại. Khi gọi native method hoặc code liên quan đến Foreign Function & Memory API, vẫn phải chú ý rủi ro Pinning còn lại.

### Stack Management (Quản lý stack)

Platform Thread thường sử dụng OS thread stack kích thước cố định. Stack của Virtual Thread tồn tại dưới dạng đối tượng khối stack trong Java Heap, có thể tăng trưởng và thu hẹp cùng với quá trình thực thi. Đây cũng là một trong những nguyên nhân quan trọng làm cho Virtual Thread có thể được tạo ra với lượng lớn.

Tuy nhiên, điều này không đại diện cho việc Virtual Thread không có chi phí bộ nhớ. Mỗi Virtual Thread vẫn là đối tượng, cũng có chi phí bộ nhớ như khối stack, biến cục bộ, `ThreadLocal`... Hàng triệu Virtual Thread không phải là miễn phí, chỉ là thực tế hơn nhiều so với hàng triệu Platform Thread.

## Pinning là gì?

Pinning có thể hiểu là "Virtual Thread tạm thời không cách nào tháo ra khỏi carrier thread". Virtual Thread sau khi bị cố định trên một Platform Thread nào đó, nó trong thời gian bị chặn sẽ kéo theo việc chiếm giữ luồng hệ điều hành bên dưới, tính mở rộng cũng sẽ trở nên kém đi theo.

Trong JDK 21 đến JDK 23, kịch bản Pinning điển hình nhất là: Virtual Thread thực thi I/O bị chặn trong khối code hoặc phương thức `synchronized`.

```java
public synchronized String load() throws IOException {
  return remoteClient.get("/config"); // Trong JDK 21-23, khi bị chặn tại đây có thể cố định carrier thread
}
```

`synchronized` ngắn nhỏ, thuần thao tác bộ nhớ thì vấn đề không lớn. Điểm thực sự cần chú ý là trên path tần suất cao nắm giữ lock thực thi I/O chậm, ví dụ nắm giữ object lock khi query database, gọi API từ xa, đọc file lớn.

Nếu bạn sử dụng JDK 21 đến JDK 23, có thể cân nhắc:

- Tránh thực thi I/O chậm bên trong `synchronized`.
- Sử dụng `ReentrantLock` cho kịch bản lock bị chặn tần suất cao, và dùng `try/finally` giải phóng lock.
- Dùng JFR quan sát sự kiện `jdk.VirtualThreadPinned`.
- Tạm thời sử dụng `-Djdk.tracePinnedThreads=full` để định vị call stack bị cố định.

Nếu bạn sử dụng JDK 24 hoặc phiên bản cao hơn, vấn đề Pinning chính do `synchronized` dẫn đến đã được giải quyết bởi JEP 491. Việc lựa chọn `synchronized` hay `java.util.concurrent.locks`, có thể quay trở lại ngữ nghĩa code, tính dễ bảo trì và bản thân năng lực lock để phán đoán.

## Có những lưu ý nào khi sử dụng Virtual Thread?

### Đừng xem Virtual Thread là công cụ tăng tốc CPU

Cái mà Virtual Thread nâng cao là năng lực gánh concurrency của các task dạng chờ đợi. Task CPU-intensive cuối cùng vẫn phải cướp time slice của CPU, số lượng Virtual Thread dù có nhiều hơn nữa cũng không thể phá vỡ giới hạn vật lý của số core CPU.

### Đừng dùng tư duy ThreadPool để giới hạn Virtual Thread

Đừng tạo Virtual ThreadPool số lượng cố định. Khi cần giới hạn concurrency, dùng `Semaphore`, connection pool, Rate Limiter hoặc dung lượng hàng đợi để giới hạn tài nguyên cụ thể.

### Cẩn thận `ThreadLocal` cache đối tượng lớn

Phiên bản chính thức Java 21 đảm bảo Virtual Thread hỗ trợ `ThreadLocal`, điều này có lợi cho việc tương thích với code cũ và framework. Nhưng đừng dùng `ThreadLocal` để cache đối tượng lớn cho mỗi Virtual Thread.

Trước đây trong ThreadPool, một `ThreadLocal<SimpleDateFormat>` có thể chỉ tương ứng với vài chục hoặc vài trăm Platform Thread. Sau khi migrate sang Virtual Thread, nếu mỗi task một Virtual Thread, cùng cách viết đó có thể biến thành mỗi task tạo một bản sao đối tượng cache, áp lực bộ nhớ và phân bổ sẽ bị phóng đại lên.

Nếu chỉ là truyền request context, user ID, Trace ID, nhìn chung vấn đề không lớn. Nếu là cache database connection, mảng lớn, formatter phức tạp, client object, thì phải đánh giá lại. JDK 25 thông qua JEP 506 đã đưa Scoped Values thành chính thức, nó phù hợp hơn cho việc truyền context bất biến giữa lượng lớn Virtual Thread.

### Virtual Thread không làm biến mất vấn đề an toàn luồng

Virtual Thread làm cho việc tạo luồng rẻ hơn, cũng có nghĩa là bạn dễ dàng đồng thời chạy lượng lớn task concurrent hơn. Tranh chấp dữ liệu trước đây do ThreadPool tương đối nhỏ nên chưa bộc lộ, sau khi chuyển sang Virtual Thread có thể dễ xuất hiện hơn.

Cần tiếp tục tuân thủ các quy tắc cơ bản của lập trình concurrency: Trạng thái có thể thay đổi dùng chung phải cài khóa hoặc cách ly, database connection, session object, client non-thread-safe không được để nhiều Virtual Thread đồng thời dùng hỗn loạn.

### Chú ý connection pool và dung lượng downstream

Rất nhiều service sau khi migrate sang Virtual Thread, nút thắt đầu tiên không còn là ThreadPool nghiệp vụ, mà là database connection pool, HTTP connection pool, Redis connection count hoặc Rate Limiting của downstream.

Đây không phải là vấn đề của Virtual Thread. Virtual Thread chỉ là làm cho nhiều task hơn có cơ hội đồng thời tiến hành, tài nguyên dùng chung thực sự vẫn phải quản lý theo dung lượng. Khi test tải khuyến nghị đồng thời quan sát:

- QPS ứng dụng, thời gian phản hồi và error rate.
- Active connection, waiting queue và timeout count của database connection pool.
- HTTP client connection pool và downstream 429/5xx.
- CPU, heap memory, GC, tỷ lệ phân bổ đối tượng.
- Sự kiện Virtual Thread và lock contention trong JFR.

### Đừng dùng lẫn quá nhiều async model

Virtual Thread phù hợp nhất với code đồng bộ bị chặn. Các hệ thống đã dùng Reactive/WebFlux/Netty viết thành async toàn chuỗi liên kết, chưa chắc đã nhận được lợi ích rõ rệt chỉ vì bật Virtual Thread.

Rắc rối hơn là dùng lẫn model: Bên ngoài Virtual Thread, bên trong lại sử dụng lượng lớn async callback và ThreadPool, khi troubleshooting có thể đồng thời đối mặt với vài bộ context: Virtual Thread, event loop, ThreadPool nghiệp vụ, connection pool. Khi migrate tốt nhất trước tiên chọn chuỗi liên kết đồng bộ bị chặn để thử nghiệm, chứ không phải thay thế một click toàn hệ thống.

## Spring Boot bật Virtual Thread như thế nào?

Spring Boot 3.2 bắt đầu cung cấp công tắc tương đối trực tiếp. Khi sử dụng Java 21 hoặc phiên bản cao hơn, có thể bật trong file cấu hình:

```properties
spring.threads.virtual.enabled=true
```

Tài liệu chính thức Spring Boot còn đề cập vài điểm thực tiễn:

- Sau khi bật Virtual Thread, các thuộc tính cấu hình kích thước ThreadPool truyền thống không còn có hiệu lực theo cách ban đầu nữa, vì điều phối Virtual Thread phụ thuộc vào Platform ThreadPool trong phạm vi JVM.
- Virtual Thread là daemon thread. Nếu ứng dụng phụ thuộc vào các task background như `@Scheduled` để giữ cho JVM sống, khuyến nghị thiết lập `spring.main.keep-alive=true`.
- Phía chính thức Spring Boot hiện tại khuyến nghị Java 24 hoặc phiên bản cao hơn để có được trải nghiệm Virtual Thread tốt hơn, chủ yếu liên quan đến việc cải tiến Pinning.

Một cấu hình đơn giản như sau:

```yaml
spring:
  threads:
    virtual:
      enabled: true
  main:
    keep-alive: true
```

Sau khi bật, không đại diện cho việc tất cả các API đều sẽ chạy nhanh hơn. Cái nó dễ làm cải thiện hơn là các API đồng bộ bị chặn, chờ I/O rõ rệt, concurrency tương đối cao. Nếu API chủ yếu tiêu tốn ở CPU, tranh chấp khóa, bản thân slow SQL hoặc downstream Rate Limiting, Virtual Thread chỉ làm cho vấn đề bộc lộ sớm hơn.

## Troubleshooting vấn đề Virtual Thread như thế nào?

JDK đã bổ sung khá nhiều năng lực quan sát cho Virtual Thread.

### Sử dụng `jcmd` export thread dump

`jstack` truyền thống đối mặt với hàng ngàn hàng vạn Virtual Thread thì không quá phù hợp. JDK cung cấp năng lực thread dump mới:

```bash
jcmd <pid> Thread.dump_to_file -format=json thread-dump.json
```

Cũng có thể export định dạng text:

```bash
jcmd <pid> Thread.dump_to_file -format=text thread-dump.txt
```

Định dạng JSON phù hợp hơn cho công cụ phân tích, đặc biệt là khi số lượng Virtual Thread rất nhiều.

### Sử dụng JFR quan sát sự kiện Virtual Thread

Các sự kiện liên quan đến Virtual Thread trong JFR bao gồm:

- `jdk.VirtualThreadStart`
- `jdk.VirtualThreadEnd`
- `jdk.VirtualThreadPinned`
- `jdk.VirtualThreadSubmitFailed`

Trong đó `jdk.VirtualThreadPinned` rất hữu ích cho việc rà soát Pinning. Sau JDK 24, Pinning liên quan đến `synchronized` đa số đã được giải quyết, nhưng các kịch bản còn lại như native/FFM vẫn có thể quan sát thông qua JFR.

### Bật tạm thời stack trace của Pinning

Trong JDK 21 đến JDK 23, có thể tạm thời sử dụng:

```bash
-Djdk.tracePinnedThreads=full
```

Nó sẽ in call stack khi Virtual Thread bị chặn và bị cố định, phù hợp định vị vấn đề trong môi trường local hoặc test. Sau JEP 491 của JDK 24, kịch bản Pinning chính liên quan đến `synchronized` đã được cải tiến; các ranh giới còn lại như native/FFM vẫn khuyến nghị kết hợp JFR và thread dump để phán đoán.

## Câu hỏi phỏng vấn thường gặp về Virtual Thread

### Sự khác biệt giữa Virtual Thread và Platform Thread là gì?

Platform Thread thường tương ứng một-đối-một với luồng hệ điều hành, chi phí tạo và chuyển đổi ngữ cảnh tương đối cao, số lượng hữu hạn. Virtual Thread do JDK điều phối, có thể ánh xạ lượng lớn Virtual Thread vào số lượng ít Platform Thread. Virtual Thread khi bị chặn chờ I/O, thường có thể tháo ra khỏi carrier thread, để Platform Thread tiếp tục thực thi Virtual Thread khác.

### Tại sao Virtual Thread phù hợp với task I/O-intensive?

Task I/O-intensive phần lớn thời gian đang chờ tài nguyên bên ngoài. Platform Thread khi chờ sẽ chiếm giữ luồng hệ điều hành; Virtual Thread khi chờ có thể tự treo mình và giải phóng carrier thread. Như vậy cùng một số lượng Platform Thread có thể gánh được nhiều task concurrent hơn.

### Virtual Thread có phù hợp với task CPU-intensive không?

Không phù hợp xem nó là công cụ tăng tốc CPU. Task CPU-intensive cần time slice CPU thực sự, sau khi số luồng vượt quá số core chỉ làm tăng thêm tranh chấp điều phối. Virtual Thread có thể cải thiện việc chờ đợi concurrency cao, không làm cho một task tính toán đơn lẻ chạy nhanh hơn.

### Virtual Thread có cần đưa vào pool không?

Không cần, cũng không khuyến nghị. Virtual Thread rẻ, nên tạo theo task. Khi cần giới hạn concurrency, giới hạn tài nguyên cụ thể, như database connection pool, HTTP connection pool, `Semaphore`, Rate Limiter, chứ không phải đưa Virtual Thread vào pool.

### Virtual Thread và Coroutine có giống nhau không?

Chúng đều thuộc về tư tưởng concurrency mỏng nhẹ, nhưng Virtual Thread trong Java là triển khai của `java.lang.Thread`, được đưa vào mô hình luồng vốn có của Java. Code nghiệp vụ không cần viết `async/await`, cũng không cần yield thủ công. So sánh với Go goroutine, Virtual Thread nhấn mạnh hơn việc tương thích với Thread API sẵn có, công cụ debugging và phong cách code bị chặn của Java. Đối với nhà phát triển mà nói, nó giống như "luồng rẻ hơn rất nhiều".

### Sau khi sử dụng Virtual Thread có còn cần Reactive programming không?

Xem kịch bản. Rất nhiều API server-side đồng bộ bị chặn có thể dùng Virtual Thread để có được tính dễ đọc tốt hơn và throughput đủ dùng, không nhất thiết phải viết chuỗi callback phức tạp vì muốn giải phóng luồng. Nhưng Reactive vẫn phù hợp cho xử lý dạng stream, backpressure, event-driven, long connection và các hệ thống đã bất đồng bộ hóa toàn chuỗi liên kết. Virtual Thread không phải là viên đạn bạc thay thế tất cả các async model.

### `synchronized` trong JDK 21 có còn dùng được không?

Dùng được, nhưng phải chú ý ranh giới. Trong JDK 21 đến JDK 23, Virtual Thread thực thi thao tác bị chặn bên trong `synchronized` có thể xuất hiện Pinning. Đồng bộ bộ nhớ ngắn nhỏ thì vấn đề không lớn, trên path tần suất cao đừng nắm giữ `synchronized` lock để thực thi I/O chậm. Sau khi JDK 24 cải tiến thông qua JEP 491, Pinning liên quan đến `synchronized` về cơ bản đã được giải quyết.

## Tài liệu tham khảo

- [JEP 444: Virtual Threads](https://openjdk.org/jeps/444)
- [Oracle Java 21 Documentation: Virtual Threads](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)
- [JEP 491: Synchronize Virtual Threads without Pinning](https://openjdk.org/jeps/491)
- [JEP 506: Scoped Values](https://openjdk.org/jeps/506)
- [Spring Boot Reference Documentation: Virtual threads](https://docs.spring.io/spring-boot/reference/features/spring-application.html#features.spring-application.virtual-threads)
- [Spring Blog: Embracing Virtual Threads](https://spring.io/blog/2022/10/11/embracing-virtual-threads/)
- [Inside Java: Managing Throughput with Virtual Threads](https://inside.java/2024/02/04/sip094/)
- [Quarkus Blog: When Quarkus meets Virtual Threads](https://quarkus.io/blog/virtual-thread-1/)

<!-- @include: @article-footer.snippet.md -->
