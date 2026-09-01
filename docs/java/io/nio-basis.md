---
title: Java NIO 核心知识总结
description: Java NIO核心知识全面总结：详解Channel通道、Buffer缓冲区、Selector选择器三大核心组件、非阻塞IO实现、零拷贝技术、与传统IO性能对比。
category: Java
tag:
  - Java IO
  - Java基础
head:
  - - meta
    - name: keywords
      content: Java IO,Channel,Buffer,Selector,非阻塞IO,多路复用,零拷贝,NIO核心组件
---

Trước khi học NIO, bạn cần tìm hiểu kiến thức lý thuyết cơ sở về I/O model của máy tính. Nếu chưa nắm rõ, bạn có thể tham khảo bài viết này của mình: [Giải thích chi tiết Java IO Model](https://javaguide.cn/java/io/io-model.html).

## Giới thiệu về NIO

Trong mô hình Java I/O truyền thống (BIO), các thao tác I/O được thực hiện theo cơ chế nghẽn (blocking). Nghĩa là, khi một thread thực thi một thao tác I/O, nó sẽ bị block cho đến khi thao tác hoàn tất. Mô hình blocking này khi xử lý nhiều kết nối đồng thời có thể dẫn đến nghẽn hiệu năng, vì cần tạo một thread cho mỗi kết nối, mà việc tạo và chuyển đổi context giữa các thread đều tiêu tốn chi phí tài nguyên.

Để giải quyết vấn đề này, phiên bản Java 1.4 đã giới thiệu một I/O model mới — **NIO** (New IO, còn gọi là Non-blocking IO). NIO bù đắp cho những hạn chế của Synchronous Blocking I/O, nó cung cấp I/O không nghẽn (non-blocking), hướng buffer, dựa trên channel trong code Java tiêu chuẩn, có thể sử dụng một số lượng nhỏ thread để xử lý nhiều kết nối, giúp cải thiện đáng kể hiệu quả I/O và tính đồng thời (concurrency).

Hình dưới đây là sơ đồ so sánh đơn giản về xử lý yêu cầu client của BIO, NIO và AIO (Về phần giới thiệu AIO, bạn có thể xem bài viết này của mình: [Giải thích chi tiết Java IO Model](https://javaguide.cn/java/io/io-model.html), đây không phải trọng tâm, chỉ cần tìm hiểu qua là được).

![So sánh BIO, NIO và AIO](https://oss.javaguide.cn/github/javaguide/java/nio/bio-aio-nio.png)

⚠️ Cần lưu ý: Sử dụng NIO không nhất thiết đồng nghĩa với hiệu năng cao, ưu thế hiệu năng của nó chủ yếu thể hiện trong môi trường mạng có tính đồng thời cao và độ trễ cao. Khi số lượng kết nối ít, mức độ đồng thời thấp hoặc tốc độ truyền tải mạng nhanh, hiệu năng của NIO không nhất thiết tốt hơn BIO truyền thống.

## Các thành phần cốt lõi của NIO

NIO chủ yếu bao gồm 3 thành phần cốt lõi sau:

- **Buffer (Vùng đệm)**: Đọc ghi dữ liệu trong NIO đều được thao tác thông qua buffer. Thao tác đọc sẽ nạp dữ liệu từ Channel vào Buffer, còn thao tác ghi sẽ ghi dữ liệu từ Buffer vào Channel.
- **Channel (Kênh)**: Channel đại diện cho một kết nối mở tới các thực thể như file, socket, v.v., tùy thuộc vào interface cụ thể mà có thể hỗ trợ đọc, ghi hoặc cả hai cùng lúc.
- **Selector (Bộ chọn)**: Cho phép một thread theo dõi các sự kiện sẵn sàng của nhiều channel có thể chọn (`SelectableChannel`), thực hiện I/O Multiplexing. Selector chịu trách nhiệm báo cáo sự kiện đã sẵn sàng hay chưa, không chịu trách nhiệm phân bổ thread xử lý.

Mối quan hệ giữa ba thành phần được thể hiện ở hình dưới (tạm thời chưa hiểu cũng không sao, phần sau sẽ giới thiệu chi tiết):

![Mối quan hệ giữa Buffer, Channel và Selector](https://oss.javaguide.cn/github/javaguide/java/nio/channel-buffer-selector.png)

Dưới đây sẽ giới thiệu chi tiết về ba thành phần này.

### Buffer (Vùng đệm)

Trong BIO truyền thống, việc đọc ghi dữ liệu là hướng stream, chia thành byte stream và character stream.

Trong thư viện NIO của Java 1.4, tất cả dữ liệu đều được xử lý bằng buffer, đây là một điểm khác biệt quan trọng giữa thư viện mới và BIO trước đó, hơi tương tự như buffered stream trong BIO. Khi NIO đọc dữ liệu, nó đọc trực tiếp vào buffer. Khi ghi dữ liệu, nó ghi vào buffer. Khi sử dụng NIO để đọc ghi dữ liệu, tất cả đều được thao tác thông qua buffer.

Các lớp con của `Buffer` được thể hiện ở hình dưới. Trong đó, được sử dụng phổ biến nhất là `ByteBuffer`, có thể dùng để lưu trữ và thao tác dữ liệu dạng byte.

![Các lớp con của Buffer](https://oss.javaguide.cn/github/javaguide/java/nio/buffer-subclasses.png)

Bạn có thể hiểu Buffer là một chuỗi dữ liệu cùng kiểu, tuyến tính và có giới hạn. Một số Buffer được hỗ trợ bởi mảng, nhưng các triển khai như direct buffer không nhất thiết có mảng bên dưới có thể truy cập được.

Để hiểu rõ hơn về buffer, chúng ta hãy xem qua 4 biến thành viên được định nghĩa trong lớp `Buffer`:

```java
public abstract class Buffer {
    // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
}
```

Ý nghĩa cụ thể của 4 biến thành viên này như sau:

1. Dung lượng (`capacity`): Lượng dữ liệu tối đa mà `Buffer` có thể lưu trữ, được thiết lập khi tạo `Buffer` và không thể thay đổi;
2. Giới hạn (`limit`): Ranh giới dữ liệu có thể đọc/ghi trong `Buffer`. Ở chế độ ghi, `limit` đại diện cho lượng dữ liệu tối đa có thể ghi, thường bằng `capacity` (có thể thiết lập qua phương thức `limit(int newLimit)`); ở chế độ đọc, `limit` bằng với kích thước dữ liệu thực tế đã ghi trong Buffer.
3. Vị trí (`position`): Vị trí (chỉ số) của dữ liệu tiếp theo có thể được đọc/ghi. Khi chuyển từ chế độ ghi sang chế độ đọc (flip), `position` sẽ được đưa về 0, nhờ đó có thể đọc/ghi từ đầu.
4. Đánh dấu (`mark`): `Buffer` cho phép định vị trực tiếp position tới vị trí mark này, đây là một thuộc tính tùy chọn;

Khi `mark` được định nghĩa, các biến trên thỏa mãn mối quan hệ: **0 <= mark <= position <= limit <= capacity**. Khi `mark` chưa được định nghĩa, gọi `reset()` sẽ ném ra `InvalidMarkException`.

Ngoài ra, Buffer có hai chế độ là chế độ đọc và chế độ ghi, lần lượt dùng để đọc dữ liệu từ Buffer hoặc ghi dữ liệu vào Buffer. Buffer sau khi được tạo mặc định ở chế độ ghi, gọi `flip()` có thể chuyển sang chế độ đọc. Nếu muốn chuyển lại về chế độ ghi, có thể gọi phương thức `clear()` hoặc `compact()`.

![Mối quan hệ giữa position, limit và capacity](https://oss.javaguide.cn/github/javaguide/java/nio/JavaNIOBuffer.png)

![Mối quan hệ giữa position, limit và capacity](https://oss.javaguide.cn/github/javaguide/java/nio/NIOBufferClassAttributes.png)

Đối tượng `Buffer` không thể tạo bằng cách gọi constructor `new`, mà chỉ có thể khởi tạo thông qua các phương thức static.

Ở đây lấy `ByteBuffer` làm ví dụ giới thiệu:

```java
// Cấp phát bộ nhớ Heap
public static ByteBuffer allocate(int capacity);
// Cấp phát bộ nhớ trực tiếp (Direct Memory)
public static ByteBuffer allocateDirect(int capacity);
```

Hai phương thức cốt lõi nhất của Buffer:

1. `get`: Đọc dữ liệu trong buffer
2. `put`: Ghi dữ liệu vào buffer

Ngoài hai phương thức trên, các phương thức quan trọng khác:

- `flip`: Chuyển buffer từ chế độ ghi sang chế độ đọc, nó sẽ đặt giá trị của `limit` bằng giá trị `position` hiện tại, và đặt giá trị `position` về 0.
- `clear`: Xóa rỗng buffer, chuyển buffer từ chế độ đọc sang chế độ ghi, đồng thời đặt giá trị `position` về 0 và giá trị `limit` bằng giá trị `capacity`.
- ……

Quá trình thay đổi dữ liệu trong Buffer:

```java
import java.nio.*;

public class CharBufferDemo {
    public static void main(String[] args) {
        // Cấp phát một CharBuffer có capacity là 8
        CharBuffer buffer = CharBuffer.allocate(8);
        System.out.println("Trạng thái ban đầu:");
        printState(buffer);

        // Ghi 3 ký tự vào buffer
        buffer.put('a').put('b').put('c');
        System.out.println("Trạng thái sau khi ghi 3 ký tự:");
        printState(buffer);

        // Gọi flip(), chuẩn bị đọc dữ liệu trong buffer, đặt position về 0, limit về 3
        buffer.flip();
        System.out.println("Trạng thái sau khi gọi flip():");
        printState(buffer);

        // Đọc ký tự
        while (buffer.hasRemaining()) {
            System.out.print(buffer.get());
        }

        // Gọi clear(), xóa rỗng buffer, đặt position về 0, limit về capacity
        buffer.clear();
        System.out.println("Trạng thái sau khi gọi clear():");
        printState(buffer);

    }

    // In capacity, limit, position của buffer
    private static void printState(CharBuffer buffer) {
        System.out.print("capacity: " + buffer.capacity());
        System.out.print(", limit: " + buffer.limit());
        System.out.print(", position: " + buffer.position());
        System.out.println("\n");
    }
}
```

Đầu ra:

```bash
Trạng thái ban đầu:
capacity: 8, limit: 8, position: 0

Trạng thái sau khi ghi 3 ký tự:
capacity: 8, limit: 8, position: 3

Chuẩn bị đọc dữ liệu trong buffer!

Trạng thái sau khi gọi flip():
capacity: 8, limit: 3, position: 0

Dữ liệu đọc được: abc

Trạng thái sau khi gọi clear():
capacity: 8, limit: 8, position: 0
```

Để dễ hiểu hơn, mình đã vẽ một bức hình thể hiện sự thay đổi của `capacity`, `limit` và `position` qua từng giai đoạn.

![Sự thay đổi của capacity, limit và position qua từng giai đoạn](https://oss.javaguide.cn/github/javaguide/java/nio/NIOBufferClassAttributesDataChanges.png)

### Channel (Kênh)

Channel là một kênh kết nối thiết lập với nguồn dữ liệu (như file, network socket, v.v.). Chúng ta có thể tận dụng nó để đọc và ghi dữ liệu, giống như mở một đường ống dẫn nước, cho phép dữ liệu tự do di chuyển trong Channel.

Stream trong BIO là một chiều, chia thành các loại `InputStream` (input stream) và `OutputStream` (output stream), dữ liệu chỉ truyền theo một hướng. Các loại channel khác nhau có thể dùng để đọc, ghi hoặc đồng thời dùng cho cả đọc và ghi. Ví dụ khả năng đọc ghi của `FileChannel` phụ thuộc vào cách mở file, còn `SocketChannel` hỗ trợ cả đọc và ghi.

Channel tương tác với Buffer đã giới thiệu ở trên, khi thực hiện thao tác đọc sẽ nạp dữ liệu từ Channel vào Buffer, còn khi thực hiện thao tác ghi sẽ ghi dữ liệu từ Buffer vào Channel.

![Mối quan hệ giữa Channel và Buffer](https://oss.javaguide.cn/github/javaguide/java/nio/channel-buffer.png)

Ngoài ra, một số Channel (như `SocketChannel`) đồng thời hỗ trợ đọc ghi, có thể ánh xạ trực tiếp hơn năng lực giao tiếp hai chiều của hệ điều hành bên dưới.

Các lớp con của `Channel` được thể hiện ở hình dưới.

![Các lớp con của Channel](https://oss.javaguide.cn/github/javaguide/java/nio/channel-subclasses.png)

Trong đó, phổ biến nhất là các loại channel sau:

- `FileChannel`: Channel truy cập file;
- `SocketChannel`, `ServerSocketChannel`: Channel giao tiếp TCP;
- `DatagramChannel`: Channel giao tiếp UDP;

![Sơ đồ quan hệ kế thừa Channel](https://oss.javaguide.cn/github/javaguide/java/nio/channel-inheritance-relationship.png)

Hai phương thức cốt lõi nhất của Channel:

1. `read`: Đọc dữ liệu và ghi vào Buffer.
2. `write`: Ghi dữ liệu trong Buffer vào Channel.

Ở đây chúng ta lấy `FileChannel` làm ví dụ minh họa cách đọc dữ liệu từ file.

```java
RandomAccessFile reader = new RandomAccessFile("/Users/guide/Documents/test_read.in", "r");
FileChannel channel = reader.getChannel();
ByteBuffer buffer = ByteBuffer.allocate(1024);
channel.read(buffer);
```

### Selector (Bộ chọn)

Selector (Bộ chọn) là một thành phần then chốt trong NIO, cho phép một thread xử lý nhiều Channel. Selector dựa trên mô hình I/O Multiplexing hướng sự kiện (event-driven), nguyên lý hoạt động chính là: Đăng ký các sự kiện của channel thông qua Selector, Selector sẽ liên tục polling các Channel đã đăng ký trên đó. Khi có sự kiện xảy ra, ví dụ: trên một Channel có kết nối TCP mới đi vào, sự kiện đọc hoặc ghi, Channel này sẽ ở trạng thái ready (sẵn sàng) và được Selector polling ra. Selector sẽ thêm các Channel liên quan vào tập hợp ready set. Thông qua `SelectionKey`, có thể lấy tập hợp các Channel đã sẵn sàng, sau đó thực hiện các thao tác I/O tương ứng trên các Channel đó.

![Sơ đồ hoạt động của Selector](https://oss.javaguide.cn/github/javaguide/java/nio/selector-channel-selectionkey.png)

Một multiplexer Selector có thể đồng thời giám sát nhiều Channel. Triển khai bên dưới của Selector do `SelectorProvider` của nền tảng quyết định, ví dụ trên Linux có thể sử dụng `epoll`, nhưng trên các nền tảng khác sẽ sử dụng triển khai tương ứng của riêng mình. Số lượng kết nối có thể tiếp nhận vẫn bị giới hạn bởi các tài nguyên như file descriptor, bộ nhớ và cấu hình hệ thống.

Selector có thể lắng nghe 4 loại sự kiện sau:

1. `SelectionKey.OP_ACCEPT`: Đại diện cho sự kiện channel chấp nhận kết nối, thường dùng cho `ServerSocketChannel`.
2. `SelectionKey.OP_CONNECT`: Đại diện cho sự kiện channel hoàn thành kết nối, thường dùng cho `SocketChannel`.
3. `SelectionKey.OP_READ`: Đại diện cho sự kiện channel đã chuẩn bị xong để đọc, tức là có dữ liệu có thể đọc.
4. `SelectionKey.OP_WRITE`: Đại diện cho sự kiện channel đã chuẩn bị xong để ghi, tức là có thể ghi dữ liệu.

`Selector` là lớp trừu tượng, có thể tạo instance Selector bằng cách gọi phương thức static `open()` của lớp này. Selector có thể đồng thời giám sát tình trạng IO của nhiều `SelectableChannel`, là cốt lõi của non-blocking IO.

Một instance Selector có 3 tập hợp `SelectionKey`:

1. Tập hợp tất cả `SelectionKey`: Đại diện cho các `Channel` đã đăng ký trên Selector đó, tập hợp này có thể trả về qua phương thức `keys()`.
2. Tập hợp `SelectionKey` được chọn: Đại diện cho tất cả các Channel có thể lấy qua phương thức `select()` và cần xử lý IO, tập hợp này có thể trả về qua phương thức `selectedKeys()`.
3. Tập hợp `SelectionKey` bị hủy: Đại diện cho tất cả các `Channel` đã bị hủy quan hệ đăng ký, trong lần thực thi phương thức `select()` tiếp theo, các `SelectionKey` tương ứng với các `Channel` này sẽ bị xóa hoàn toàn, chương trình thường không cần truy cập trực tiếp tập hợp này và cũng không có phương thức truy cập công khai.

Minh họa đơn giản cách duyệt và xử lý tập hợp `SelectionKey` được chọn:

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
while (keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if (key != null) {
        if (key.isAcceptable()) {
            // ServerSocketChannel nhận một kết nối mới
        } else if (key.isConnectable()) {
            // Đại diện một kết nối mới đã được thiết lập
        } else if (key.isReadable()) {
            // Channel có dữ liệu đã chuẩn bị xong, có thể đọc
        } else if (key.isWritable()) {
            // Channel đã sẵn sàng, có thể thử ghi dữ liệu
        }
    }
    keyIterator.remove();
}
```

Selector còn cung cấp một loạt các phương thức liên quan đến `select()`:

- `int select()`: Giám sát tất cả các `Channel` đã đăng ký, khi giữa chúng có thao tác IO cần xử lý, phương thức này sẽ trả về và thêm `SelectionKey` tương ứng vào tập hợp `SelectionKey` được chọn. Giá trị trả về là số lượng key trong ready set được cập nhật trong lần thao tác này.
- `int select(long timeout)`: Thao tác `select()` có thể thiết lập thời gian timeout.
- `int selectNow()`: Thực thi một thao tác `select()` trả về ngay lập tức, so với phương thức `select()` không tham số, phương thức này không làm block thread.
- `Selector wakeup()`: Làm cho một phương thức `select()` chưa trả về lập tức trả về.
- ……

Ví dụ đơn giản sử dụng Selector để triển khai đọc ghi qua mạng:

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NioSelectorExample {

  public static void main(String[] args) {
    try {
      ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
      serverSocketChannel.configureBlocking(false);
      serverSocketChannel.socket().bind(new InetSocketAddress(8080));

      Selector selector = Selector.open();
      // Đăng ký ServerSocketChannel vào Selector và lắng nghe sự kiện OP_ACCEPT
      serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

      while (true) {
        int readyChannels = selector.select();

        if (readyChannels == 0) {
          continue;
        }

        Set<SelectionKey> selectedKeys = selector.selectedKeys();
        Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

        while (keyIterator.hasNext()) {
          SelectionKey key = keyIterator.next();

          if (key.isAcceptable()) {
            // Xử lý sự kiện kết nối
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            SocketChannel client = server.accept();
            client.configureBlocking(false);

            // Đăng ký client channel vào Selector và lắng nghe sự kiện OP_READ
            client.register(selector, SelectionKey.OP_READ);
          } else if (key.isReadable()) {
            // Xử lý sự kiện đọc
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int bytesRead = client.read(buffer);

            if (bytesRead > 0) {
              buffer.flip();
              System.out.println("Dữ liệu nhận được: " + new String(buffer.array(), 0, bytesRead));
              // Lưu dữ liệu chờ gửi và lắng nghe sự kiện OP_WRITE
              key.attach(ByteBuffer.wrap("Hello, Client!".getBytes()));
              key.interestOps(SelectionKey.OP_WRITE);
            } else if (bytesRead < 0) {
              // Client ngắt kết nối
              client.close();
            }
          } else if (key.isWritable()) {
            // Xử lý sự kiện ghi
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = (ByteBuffer) key.attachment();
            client.write(buffer);

            // write không nghẽn có thể chỉ ghi một phần dữ liệu, sau khi ghi xong toàn bộ mới chuyển lại OP_READ
            if (!buffer.hasRemaining()) {
              key.attach(null);
              key.interestOps(SelectionKey.OP_READ);
            }
          }

          keyIterator.remove();
        }
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

Trong ví dụ trên, chúng ta đã tạo một server đơn giản lắng nghe port 8080, sử dụng Selector để xử lý các sự kiện kết nối, đọc và ghi. Khi nhận được dữ liệu từ client, server sẽ đọc dữ liệu và in ra console, sau đó phản hồi lại client "Hello, Client!".

## NIO Zero-Copy (Sao chép 0)

Zero-Copy là một phương pháp thường dùng để nâng cao hiệu năng thao tác IO, các dự án mã nguồn mở hàng đầu như ActiveMQ, Kafka, RocketMQ, QMQ, Netty đều sử dụng Zero-Copy.

Zero-Copy chỉ việc khi máy tính thực thi thao tác IO, CPU không cần sao chép dữ liệu từ vùng lưu trữ này sang vùng lưu trữ khác, nhờ đó có thể giảm việc chuyển đổi context cũng như thời gian sao chép của CPU. Nói cách khác, Zero-Copy chủ yếu giải quyết vấn đề hệ điều hành sao chép dữ liệu thường xuyên khi xử lý thao tác I/O. Các kỹ thuật triển khai Zero-Copy phổ biến gồm có: `mmap+write`, `sendfile` và `sendfile + DMA gather copy`.

Bảng dưới đây hiển thị so sánh giữa các kỹ thuật Zero-Copy:

|                            | Sao chép CPU | Sao chép DMA | System call | Chuyển đổi context |
| -------------------------- | ------------ | ------------ | ----------- | ------------------ |
| Phương pháp truyền thống   | 2            | 2            | read+write  | 4                  |
| mmap+write                 | 1            | 2            | mmap+write  | 4                  |
| sendfile                   | 1            | 2            | sendfile    | 2                  |
| sendfile + DMA gather copy | 0            | 2            | sendfile    | 2                  |

Có thể thấy, dù là phương pháp I/O truyền thống hay sau khi áp dụng Zero-Copy, 2 lần sao chép DMA (Direct Memory Access) đều không thể thiếu. Vì 2 lần DMA này đều phụ thuộc vào phần cứng để hoàn thành. Zero-Copy chủ yếu làm giảm sự sao chép của CPU và việc chuyển đổi context.

Sự hỗ trợ của Java đối với Zero-Copy:

- `MappedByteBuffer` là triển khai memory-mapped file do Java NIO cung cấp, có thể ánh xạ một phần của file vào bộ nhớ. Cơ chế bên dưới phụ thuộc vào hệ điều hành, ví dụ trên Linux thường dựa trên `mmap`.
- `transferTo()/transferFrom()` của `FileChannel` có thể truyền trực tiếp byte giữa các channel, nhiều hệ điều hành có thể tối ưu hóa kiểu truyền tải này, ví dụ trên Linux có thể sử dụng `sendfile`. Triển khai cụ thể phụ thuộc vào JDK và hệ điều hành. Về cách dùng `FileChannel`, bạn có thể xem bài viết này: [Cách dùng kênh file Java NIO FileChannel](https://www.cnblogs.com/robothy/p/14235598.html).

Code minh họa:

```java
private void loadFileIntoMemory(File xmlFile) throws IOException {
  FileInputStream fis = new FileInputStream(xmlFile);
  // Tạo đối tượng FileChannel
  FileChannel fc = fis.getChannel();
  // FileChannel.map() ánh xạ file vào bộ nhớ trực tiếp (Direct Memory) và trả về đối tượng MappedByteBuffer
  MappedByteBuffer mmb = fc.map(FileChannel.MapMode.READ_ONLY, 0, fc.size());
  xmlFileBuffer = new byte[(int)fc.size()];
  mmb.get(xmlFileBuffer);
  fis.close();
}
```

## Tóm tắt

Bài viết này chúng ta đã giới thiệu các điểm kiến thức cốt lõi của NIO, bao gồm các thành phần cốt lõi của NIO và Zero-Copy.

Nếu chúng ta cần sử dụng NIO để xây dựng chương trình mạng, không khuyến khích sử dụng trực tiếp NIO nguyên bản vì lập trình phức tạp và tính năng quá yếu, khuyến khích sử dụng một số framework lập trình mạng trưởng thành dựa trên NIO như Netty. Netty đã thực hiện một số tối ưu hóa và mở rộng trên cơ sở NIO như hỗ trợ nhiều giao thức, hỗ trợ SSL/TLS, v.v.

## Tham khảo

- Phân tích ngắn gọn về Java NIO: <https://tech.meituan.com/2016/11/04/nio.html>
- Người phỏng vấn: Có tìm hiểu về Java NIO không? <https://mp.weixin.qq.com/s/mZobf-U8OSYQfHfYBEB6KA>
- Java NIO: Buffer, Channel và Selector: <https://www.javadoop.com/post/java-nio>

<!-- @include: @article-footer.snippet.md -->
