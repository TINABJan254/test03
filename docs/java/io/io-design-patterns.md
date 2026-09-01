---
title: Java IO 设计模式总结
description: Java IO设计模式深度解析：详解装饰器模式在BufferedInputStream中应用、适配器模式InputStreamReader实现、模板方法模式InputStream设计，理解Java IO类库架构。
category: Java
tag:
  - Java IO
  - Java基础
head:
  - - meta
    - name: keywords
      content: Java IO设计模式,装饰器模式,适配器模式,模板方法模式,FilterInputStream,IO流设计
---

Bài viết này chúng ta sẽ cùng tìm hiểu ngắn gọn về các Design Pattern được áp dụng trong Java IO.

## Decorator Pattern

**Decorator Pattern (Mẫu trang trí)** có thể mở rộng chức năng của một đối tượng mà không làm thay đổi đối tượng ban đầu.

Decorator Pattern mở rộng chức năng của lớp gốc bằng cách sử dụng composition (kết hợp) thay vì inheritance (kế thừa), điều này thực tế hơn trong một số kịch bản có quan hệ kế thừa tương đối phức tạp (quan hệ kế thừa của các lớp khác nhau trong IO khá phức tạp).

Đối với Byte Stream, `FilterInputStream` (tương ứng với Input Stream) và `FilterOutputStream` (tương ứng với Output Stream) là cốt lõi của Decorator Pattern, lần lượt được dùng để tăng cường chức năng cho các đối tượng lớp con của `InputStream` và `OutputStream`.

Các lớp quen thuộc như `BufferedInputStream` (Byte Buffered Input Stream), `DataInputStream`, v.v. đều là lớp con của `FilterInputStream`; `BufferedOutputStream` (Byte Buffered Output Stream), `DataOutputStream`, v.v. đều là lớp con của `FilterOutputStream`.

Lấy ví dụ, chúng ta có thể tăng cường chức năng của `FileInputStream` thông qua `BufferedInputStream` (Byte Buffered Input Stream).

Hàm khởi tạo (constructor) của `BufferedInputStream` như sau:

```java
public BufferedInputStream(InputStream in) {
    this(in, DEFAULT_BUFFER_SIZE);
}

public BufferedInputStream(InputStream in, int size) {
    super(in);
    if (size <= 0) {
        throw new IllegalArgumentException("Buffer size <= 0");
    }
    buf = new byte[size];
}
```

Có thể thấy, một trong các tham số của constructor `BufferedInputStream` chính là `InputStream`.

Ví dụ code `BufferedInputStream`:

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("input.txt"))) {
    int content;
    long skip = bis.skip(2);
    while ((content = bis.read()) != -1) {
        System.out.print((char) content);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

Lúc này, có thể bạn sẽ thắc mắc: **Tại sao chúng ta không tạo trực tiếp một `BufferedFileInputStream` (Character Buffered File Input Stream)?**

```java
BufferedFileInputStream bfis = new BufferedFileInputStream("input.txt");
```

Nếu số lớp con của `InputStream` là ít, làm như vậy không có vấn đề gì. Tuy nhiên, các lớp con của `InputStream` lại quá nhiều, quan hệ kế thừa cũng quá phức tạp. Nếu chúng ta tùy chỉnh một Buffered Input Stream tương ứng cho mỗi lớp con, chẳng phải sẽ quá rắc rối sao.

Nếu bạn khá quen thuộc với IO Stream, bạn sẽ phát hiện ra `ZipInputStream` và `ZipOutputStream` còn có thể lần lượt tăng cường khả năng cho `BufferedInputStream` và `BufferedOutputStream`.

```java
BufferedInputStream bis = new BufferedInputStream(new FileInputStream(fileName));
ZipInputStream zis = new ZipInputStream(bis);

BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(fileName));
ZipOutputStream zipOut = new ZipOutputStream(bos);
```

`ZipInputStream` và `ZipOutputStream` lần lượt kế thừa từ `InflaterInputStream` và `DeflaterOutputStream`.

```java
public
class InflaterInputStream extends FilterInputStream {
}

public
class DeflaterOutputStream extends FilterOutputStream {
}

```

Đây cũng là một đặc điểm rất quan trọng của Decorator Pattern, đó là có thể lồng ghép nhiều decorator trên lớp gốc.

Để đạt được hiệu ứng này, lớp decorator cần kế thừa cùng một lớp trừu tượng hoặc implement cùng một interface với lớp gốc. Các lớp decorator và lớp gốc liên quan đến IO được giới thiệu ở trên có lớp cha chung là `InputStream` và `OutputStream`.

Đối với Character Stream, `BufferedReader` có thể được dùng để tăng cường chức năng của các lớp con `Reader` (Character Input Stream), `BufferedWriter` có thể được dùng để tăng cường chức năng của các lớp con `Writer` (Character Output Stream).

```java
BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(fileName), "UTF-8"));
```

Ví dụ về ứng dụng của Decorator Pattern trong IO Stream thực sự rất nhiều, không cần phải cố ghi nhớ! Sau khi nắm vững cốt lõi của Decorator Pattern, khi sử dụng bạn sẽ tự nhiên biết những chỗ nào đang áp dụng Decorator Pattern.

## Adapter Pattern

**Adapter Pattern (Mẫu thích ứng)** chủ yếu được sử dụng để phối hợp các lớp có interface không tương thích với nhau, bạn có thể liên tưởng nó với bộ chuyển đổi nguồn (adapter) mà chúng ta thường dùng hàng ngày.

Trong Adapter Pattern, đối tượng hoặc lớp được thích ứng gọi là **Adaptee (Bên được thích ứng)**, đối tượng hoặc lớp tác động lên Adaptee gọi là **Adapter (Bộ thích ứng)**. Adapter được chia thành Object Adapter và Class Adapter. Class Adapter sử dụng quan hệ kế thừa để triển khai, Object Adapter sử dụng quan hệ composition để triển khai.

Interface của Character Stream và Byte Stream trong IO Stream là khác nhau, việc chúng có thể phối hợp làm việc với nhau chính là dựa trên Adapter Pattern, chính xác hơn là Object Adapter. Thông qua Adapter, chúng ta có thể chuyển đổi một đối tượng Byte Stream thành một đối tượng Character Stream, nhờ đó chúng ta có thể trực tiếp đọc hoặc ghi dữ liệu ký tự thông qua đối tượng Byte Stream.

`InputStreamReader` và `OutputStreamWriter` chính là hai Adapter, đồng thời chúng cũng là cầu nối giữa Byte Stream và Character Stream. `InputStreamReader` sử dụng `StreamDecoder` (Bộ giải mã stream) để giải mã byte, **thực hiện chuyển đổi từ Byte Stream sang Character Stream,** `OutputStreamWriter` sử dụng `StreamEncoder` (Bộ mã hóa stream) để mã hóa ký tự, thực hiện chuyển đổi từ Character Stream sang Byte Stream.

Các lớp con của `InputStream` và `OutputStream` là Adaptee, còn `InputStreamReader` và `OutputStreamWriter` là Adapter.

```java
// InputStreamReader là Adapter, FileInputStream là lớp được thích ứng (Adaptee)
InputStreamReader isr = new InputStreamReader(new FileInputStream(fileName), "UTF-8");
// BufferedReader tăng cường chức năng của InputStreamReader (Decorator Pattern)
BufferedReader bufferedReader = new BufferedReader(isr);
```

Trích đoạn source code `java.io.InputStreamReader`:

```java
public class InputStreamReader extends Reader {
    // Đối tượng dùng để giải mã
    private final StreamDecoder sd;
    public InputStreamReader(InputStream in) {
        super(in);
        try {
            // Lấy đối tượng StreamDecoder
            sd = StreamDecoder.forInputStreamReader(in, this, (String)null);
        } catch (UnsupportedEncodingException e) {
            throw new Error(e);
        }
    }
    // Sử dụng đối tượng StreamDecoder để thực hiện công việc đọc cụ thể
    public int read() throws IOException {
        return sd.read();
    }
}
```

Trích đoạn source code `java.io.OutputStreamWriter`:

```java
public class OutputStreamWriter extends Writer {
    // Đối tượng dùng để mã hóa
    private final StreamEncoder se;
    public OutputStreamWriter(OutputStream out) {
        super(out);
        try {
           // Lấy đối tượng StreamEncoder
            se = StreamEncoder.forOutputStreamWriter(out, this, (String)null);
        } catch (UnsupportedEncodingException e) {
            throw new Error(e);
        }
    }
    // Sử dụng đối tượng StreamEncoder để thực hiện công việc ghi cụ thể
    public void write(int c) throws IOException {
        se.write(c);
    }
}
```

**Sự khác biệt giữa Adapter Pattern và Decorator Pattern là gì?**

**Decorator Pattern** tập trung nhiều hơn vào việc tăng cường động chức năng của lớp gốc, lớp decorator cần kế thừa cùng một lớp trừu tượng hoặc implement cùng một interface với lớp gốc. Đồng thời, Decorator Pattern hỗ trợ lồng ghép nhiều decorator trên lớp gốc.

**Adapter Pattern** tập trung nhiều hơn vào việc cho phép các lớp có interface không tương thích có thể làm việc cùng nhau. Khi chúng ta gọi phương thức tương ứng của Adapter, bên trong Adapter sẽ gọi phương thức của lớp Adaptee hoặc các lớp liên quan, quá trình này là trong suốt. Ví dụ, `InputStreamReader` và `OutputStreamWriter` lần lượt chuyển đổi Byte Input Stream, Byte Output Stream thành Character Input Stream, Character Output Stream, và hoàn thành việc mã hóa/giải mã giữa byte và ký tự ở bên trong.

```java
Reader reader = new InputStreamReader(inputStream, StandardCharsets.UTF_8);
Writer writer = new OutputStreamWriter(outputStream, StandardCharsets.UTF_8);
```

Adapter và Adaptee không cần phải kế thừa cùng một lớp trừu tượng hoặc implement cùng một interface.

Ngoài ra, lớp `FutureTask` cũng sử dụng Adapter Pattern, lớp nội bộ `RunnableAdapter` của `Executors` thuộc về Adapter, được dùng để chuyển đổi `Runnable` thành `Callable`.

Constructor của `FutureTask` nhận tham số `Runnable`:

```java
public FutureTask(Runnable runnable, V result) {
    // Gọi phương thức callable của lớp Executors
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;
}
```

Phương thức và Adapter tương ứng trong `Executors`:

```java
// Thực tế gọi constructor của lớp nội bộ RunnableAdapter trong Executors
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
// Adapter
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

## Factory Pattern

Factory Pattern được dùng để tạo đối tượng. Trong NIO sử dụng rất nhiều Factory Pattern, chẳng hạn phương thức `newInputStream` của lớp `Files` được dùng để tạo đối tượng `InputStream` (Static Factory), phương thức `get` của lớp `Paths` tạo đối tượng `Path` (Static Factory), phương thức `getPath` của lớp `ZipFileSystem` (lớp trong package `sun.nio`, thuộc về các triển khai nội bộ liên quan đến `java.nio`) tạo đối tượng `Path` (Simple Factory).

```java
InputStream is = Files.newInputStream(Paths.get(generatorLogoPath))
```

## Observer Pattern

Dịch vụ giám sát thư mục file trong NIO có sử dụng Observer Pattern.

Dịch vụ giám sát thư mục file trong NIO dựa trên interface `WatchService` và interface `Watchable`. `WatchService` thuộc về Observer (bên quan sát), `Watchable` thuộc về Observable (bên bị quan sát).

Interface `Watchable` định nghĩa phương thức `register` dùng để đăng ký đối tượng vào `WatchService` (dịch vụ giám sát) và ràng buộc các sự kiện lắng nghe.

```java
public interface Path
    extends Comparable<Path>, Iterable<Path>, Watchable{
}

public interface Watchable {
    WatchKey register(WatchService watcher,
                      WatchEvent.Kind<?>[] events,
                      WatchEvent.Modifier... modifiers)
        throws IOException;
}
```

`WatchService` được dùng để lắng nghe sự thay đổi của thư mục file, cùng một đối tượng `WatchService` có thể lắng nghe nhiều thư mục file.

```java
// Tạo đối tượng WatchService
WatchService watchService = FileSystems.getDefault().newWatchService();

// Khởi tạo lớp Path của thư mục được giám sát:
Path path = Paths.get("workingDirectory");
// Đăng ký đối tượng path này vào WatchService (dịch vụ giám sát)
WatchKey watchKey = path.register(
watchService, StandardWatchEventKinds...);
```

Tham số thứ hai `events` (các sự kiện cần lắng nghe) của phương thức `register` trong lớp `Path` là tham số độ dài biến đổi (varargs), nghĩa là chúng ta có thể lắng nghe nhiều loại sự kiện cùng một lúc.

```java
WatchKey register(WatchService watcher,
                  WatchEvent.Kind<?>... events)
    throws IOException;
```

Các sự kiện lắng nghe thường dùng có 3 loại:

- `StandardWatchEventKinds.ENTRY_CREATE`: Tạo file.
- `StandardWatchEventKinds.ENTRY_DELETE`: Xóa file.
- `StandardWatchEventKinds.ENTRY_MODIFY`: Sửa đổi file.

Phương thức `register` trả về đối tượng `WatchKey`, thông qua đối tượng `WatchKey` có thể lấy thông tin chi tiết của sự kiện như trong thư mục file là tạo, xóa hay sửa đổi file, tên cụ thể của file được tạo, xóa hoặc sửa đổi là gì.

```java
WatchKey key;
while ((key = watchService.take()) != null) {
    for (WatchEvent<?> event : key.pollEvents()) {
      // Có thể gọi phương thức của đối tượng WatchEvent để làm một số việc như xuất thông tin context cụ thể của sự kiện
    }
    key.reset();
}
```

Triển khai cụ thể của `WatchService` phụ thuộc vào hệ thống file và Provider bên dưới: triển khai có thể sử dụng trực tiếp cơ chế sự kiện file nguyên bản của hệ thống, hoặc có thể hạ cấp xuống kiểm tra định kỳ (polling). Dưới đây là source code đơn giản hóa của triển khai polling.

```java
class PollingWatchService
    extends AbstractWatchService
{
    // Định nghĩa một daemon thread (luồng chạy ngầm) để kiểm tra định kỳ sự thay đổi của file
    private final ScheduledExecutorService scheduledExecutor;

    PollingWatchService() {
        scheduledExecutor = Executors
            .newSingleThreadScheduledExecutor(new ThreadFactory() {
                 @Override
                 public Thread newThread(Runnable r) {
                     Thread t = new Thread(r);
                     t.setDaemon(true);
                     return t;
                 }});
    }

  void enable(Set<? extends WatchEvent.Kind<?>> events, long period) {
    synchronized (this) {
      // Cập nhật sự kiện lắng nghe
      this.events = events;

        // Bật kiểm tra định kỳ (polling)
      Runnable thunk = new Runnable() { public void run() { poll(); }};
      this.poller = scheduledExecutor
        .scheduleAtFixedRate(thunk, period, period, TimeUnit.SECONDS);
    }
  }
}
```

## Tham khảo

- Patterns in Java APIs: <http://cecs.wright.edu/~tkprasad/courses/ceg860/paper/node26.html>
- Decorator Pattern: Phân tích source code thư viện Java IO để học Decorator Pattern: <https://time.geekbang.org/column/article/204845>
- Package sun.nio là gì, có phải code Java không? - RednaxelaFX <https://www.zhihu.com/question/29237781/answer/43653953>

<!-- @include: @article-footer.snippet.md -->
