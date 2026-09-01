---
title: Tổng hợp kiến thức cơ bản Java IO
description: Tổng hợp toàn diện kiến thức cơ bản Java IO: giải thích chi tiết sự khác biệt giữa byte stream và character stream, InputStream/OutputStream byte stream, Reader/Writer character stream, tối ưu bằng buffered stream, các thao tác đọc/ghi file.
category: Java
tag:
  - Java IO
  - Java基础
head:
  - - meta
    - name: keywords
      content: Java IO,字节流,字符流,InputStream,OutputStream,Reader,Writer,文件操作,缓冲流
---

## Giới thiệu về IO Stream

IO là viết tắt của `Input/Output` (Đầu vào và Đầu ra). Quá trình dữ liệu đi vào bộ nhớ máy tính gọi là đầu vào (Input), ngược lại, quá trình xuất ra bộ nhớ ngoài (ví dụ: cơ sở dữ liệu, file, máy chủ từ xa) gọi là đầu ra (Output). Quá trình truyền dữ liệu tương tự như dòng nước chảy, vì vậy được gọi là IO Stream. Trong Java, IO Stream được chia thành input stream và output stream, và dựa trên cách xử lý dữ liệu lại được phân thành byte stream và character stream.

Hơn 40 class trong Java IO Stream đều được kế thừa từ 4 class trừu tượng cơ sở sau:

- `InputStream`/`Reader`: Class cơ sở của tất cả input stream, cái trước là byte input stream, cái sau là character input stream.
- `OutputStream`/`Writer`: Class cơ sở của tất cả output stream, cái trước là byte output stream, cái sau là character output stream.

## Byte Stream

### InputStream (Byte Input Stream)

`InputStream` được dùng để đọc dữ liệu (thông tin dạng byte) từ nguồn (thường là file) vào bộ nhớ, class trừu tượng `java.io.InputStream` là class cha của tất cả byte input stream.

Các phương thức thường dùng của `InputStream`:

- `read()`: Trả về byte tiếp theo trong input stream. Giá trị trả về nằm trong khoảng 0 đến 255. Nếu không đọc được byte nào, trả về `-1`, biểu thị kết thúc file.
- `read(byte b[])`: Đọc một số byte từ input stream và lưu vào mảng `b`. Nếu độ dài mảng `b` bằng 0 thì không đọc. Nếu không có byte nào để đọc, trả về `-1`. Nếu có byte để đọc, số byte đọc tối đa bằng `b.length`, trả về số byte đã đọc. Phương thức này tương đương `read(b, 0, b.length)`.
- `read(byte b[], int off, int len)`: Mở rộng phương thức `read(byte b[])` với thêm tham số `off` (offset) và `len` (số byte tối đa cần đọc).
- `skip(long n)`: Bỏ qua n byte trong input stream, trả về số byte thực tế đã bỏ qua.
- `available()`: Trả về ước tính số byte có thể đọc (hoặc bỏ qua) mà không bị chặn, không dùng để xác định tổng độ dài của input stream.
- `close()`: Đóng input stream và giải phóng tài nguyên hệ thống liên quan.

Từ Java 9 trở đi, `InputStream` bổ sung thêm nhiều phương thức tiện ích:

- `readAllBytes()`: Đọc tất cả byte trong input stream, trả về mảng byte.
- `readNBytes(byte[] b, int off, int len)`: Cố gắng đọc tối đa `len` byte, trả về khi đọc đủ độ dài hoặc gặp cuối stream; trong quá trình đọc có thể bị chặn hoặc ném exception.
- `transferTo(OutputStream out)`: Chuyển tất cả byte từ một input stream sang một output stream.

`FileInputStream` là một byte input stream thường dùng, có thể chỉ định đường dẫn file trực tiếp, có thể đọc từng byte đơn lẻ hoặc đọc vào mảng byte.

Ví dụ code `FileInputStream`:

```java
try (InputStream fis = new FileInputStream("input.txt")) {
    System.out.println("Number of remaining bytes:"
            + fis.available());
    int content;
    long skip = fis.skip(2);
    System.out.println("The actual number of bytes skipped:" + skip);
    System.out.print("The content read from file:");
    while ((content = fis.read()) != -1) {
        System.out.print((char) content);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

Nội dung file `input.txt`:

![](https://oss.javaguide.cn/github/javaguide/java/image-20220419155214614.png)

Kết quả đầu ra:

```plain
Number of remaining bytes:11
The actual number of bytes skipped:2
The content read from file:JavaGuide
```

Tuy nhiên, thông thường chúng ta không dùng `FileInputStream` một mình mà thường kết hợp với `BufferedInputStream` (byte buffered input stream, sẽ đề cập ở phần sau).

Đoạn code như dưới đây khá phổ biến trong các dự án, chúng ta dùng `readAllBytes()` để đọc tất cả byte từ input stream và gán trực tiếp cho một `String` object.

```java
// Tạo mới một đối tượng BufferedInputStream
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("input.txt"));
// Đọc nội dung file và sao chép vào đối tượng String
String result = new String(bufferedInputStream.readAllBytes());
System.out.println(result);
```

`DataInputStream` được dùng để đọc dữ liệu với kiểu cụ thể, không thể dùng một mình mà phải kết hợp với stream khác, ví dụ `FileInputStream`.

```java
FileInputStream fileInputStream = new FileInputStream("input.txt");
//phải truyền fileInputStream làm tham số constructor mới dùng được
DataInputStream dataInputStream = new DataInputStream(fileInputStream);
//có thể đọc bất kỳ kiểu dữ liệu cụ thể nào
dataInputStream.readBoolean();
dataInputStream.readInt();
dataInputStream.readUTF();
```

`ObjectInputStream` được dùng để đọc Java object từ input stream (deserialization), `ObjectOutputStream` được dùng để ghi object vào output stream (serialization).

```java
ObjectInputStream input = new ObjectInputStream(new FileInputStream("object.data"));
MyClass object = (MyClass) input.readObject();
input.close();
```

Ngoài ra, class được dùng để serialization và deserialization phải implement interface `Serializable`, nếu có thuộc tính không muốn serialize, dùng modifier `transient`.

### OutputStream (Byte Output Stream)

`OutputStream` được dùng để ghi dữ liệu (thông tin dạng byte) vào đích đến (thường là file), class trừu tượng `java.io.OutputStream` là class cha của tất cả byte output stream.

Các phương thức thường dùng của `OutputStream`:

- `write(int b)`: Ghi byte cụ thể vào output stream.
- `write(byte b[])`: Ghi mảng `b` vào output stream, tương đương `write(b, 0, b.length)`.
- `write(byte[] b, int off, int len)`: Mở rộng phương thức `write(byte b[])` với thêm tham số `off` (offset) và `len` (số byte tối đa cần đọc).
- `flush()`: Flush output stream này và ghi tất cả byte đầu ra đã được buffer.
- `close()`: Đóng output stream và giải phóng tài nguyên hệ thống liên quan.

`FileOutputStream` là byte output stream thường dùng nhất, có thể chỉ định đường dẫn file trực tiếp, có thể xuất từng byte đơn lẻ hoặc xuất mảng byte được chỉ định.

Ví dụ code `FileOutputStream`:

```java
try (FileOutputStream output = new FileOutputStream("output.txt")) {
    byte[] array = "JavaGuide".getBytes();
    output.write(array);
} catch (IOException e) {
    e.printStackTrace();
}
```

Kết quả chạy:

![](https://oss.javaguide.cn/github/javaguide/java/image-20220419155514392.png)

Tương tự `FileInputStream`, `FileOutputStream` thường cũng được dùng kết hợp với `BufferedOutputStream` (byte buffered output stream, sẽ đề cập ở phần sau).

```java
FileOutputStream fileOutputStream = new FileOutputStream("output.txt");
BufferedOutputStream bos = new BufferedOutputStream(fileOutputStream);
```

**`DataOutputStream`** được dùng để ghi dữ liệu với kiểu cụ thể, không thể dùng một mình mà phải kết hợp với stream khác, ví dụ `FileOutputStream`.

```java
// output stream
FileOutputStream fileOutputStream = new FileOutputStream("out.txt");
DataOutputStream dataOutputStream = new DataOutputStream(fileOutputStream);
// xuất bất kỳ kiểu dữ liệu nào
dataOutputStream.writeBoolean(true);
dataOutputStream.writeByte(1);
```

`ObjectInputStream` được dùng để đọc Java object từ input stream (deserialization), `ObjectOutputStream` ghi object vào output stream (serialization).

```java
ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream("file.txt"));
Person person = new Person("Guide哥", "JavaGuide作者");
output.writeObject(person);
```

## Character Stream

Dù là đọc/ghi file hay gửi/nhận qua mạng, đơn vị lưu trữ thông tin nhỏ nhất đều là byte. **Vậy tại sao thao tác I/O Stream lại được chia thành byte stream và character stream?**

Cá nhân tôi nghĩ có hai lý do chính:

- Character stream được JVM chuyển đổi từ byte, quá trình này khá tốn thời gian.
- Nếu chúng ta không biết kiểu encoding thì rất dễ gặp lỗi ký tự rác (mojibake).

Vấn đề ký tự rác rất dễ tái hiện, chúng ta chỉ cần đổi nội dung file `input.txt` trong ví dụ code `FileInputStream` ở trên sang tiếng Trung, không cần thay đổi code.

![](https://oss.javaguide.cn/github/javaguide/java/image-20220419154632551.png)

Kết quả đầu ra:

```java
Number of remaining bytes:9
The actual number of bytes skipped:2
The content read from file:§å®¶å¥½
```

Có thể thấy rõ ràng nội dung đọc ra đã bị lỗi ký tự rác.

Do đó, I/O Stream cung cấp một interface để thao tác trực tiếp với ký tự, thuận tiện cho các thao tác stream với ký tự. Nếu là file âm thanh, hình ảnh và các file media khác thì dùng byte stream tốt hơn, còn nếu liên quan đến ký tự thì dùng character stream tốt hơn.

`Reader` và `Writer` được dùng để thao tác với ký tự; khi chuyển đổi giữa byte stream và character stream, cần chỉ định encoding thông qua `Charset`. Các class chuyển đổi không chỉ định encoding rõ ràng sẽ dùng charset mặc định của JVM.

Unicode bản thân chỉ là một bộ ký tự, nó gán một số định danh duy nhất cho mỗi ký tự, nhưng không quy định cách lưu trữ cụ thể. UTF-8, UTF-16, UTF-32 đều là các phương thức encoding của Unicode, chúng dùng số lượng byte khác nhau để biểu diễn ký tự Unicode. Ví dụ, UTF-8: chữ tiếng Anh chiếm 1 byte, chữ Hán chiếm 3 byte.

### Reader (Character Input Stream)

`Reader` được dùng để đọc dữ liệu (thông tin ký tự) từ nguồn (thường là file) vào bộ nhớ, class trừu tượng `java.io.Reader` là class cha của tất cả character input stream.

`Reader` dùng để đọc văn bản, `InputStream` dùng để đọc byte thô.

Các phương thức thường dùng của `Reader`:

- `read()`: Đọc một ký tự từ input stream.
- `read(char[] cbuf)`: Đọc một số ký tự từ input stream và lưu vào mảng ký tự `cbuf`, tương đương `read(cbuf, 0, cbuf.length)`.
- `read(char[] cbuf, int off, int len)`: Mở rộng phương thức `read(char[] cbuf)` với thêm tham số `off` (offset) và `len` (số ký tự tối đa cần đọc).
- `skip(long n)`: Bỏ qua n ký tự trong input stream, trả về số ký tự thực tế đã bỏ qua.
- `close()`: Đóng input stream và giải phóng tài nguyên hệ thống liên quan.

`InputStreamReader` là cầu nối chuyển đổi byte stream thành character stream, class con của nó `FileReader` là lớp đóng gói dựa trên cơ sở đó, có thể thao tác trực tiếp với file ký tự.

```java
// Cầu nối chuyển đổi byte stream thành character stream
public class InputStreamReader extends Reader {
}
// Dùng để đọc file ký tự
public class FileReader extends InputStreamReader {
}
```

Ví dụ code `FileReader`:

```java
try (FileReader fileReader = new FileReader("input.txt");) {
    int content;
    long skip = fileReader.skip(3);
    System.out.println("The actual number of characters skipped:" + skip);
    System.out.print("The content read from file:");
    while ((content = fileReader.read()) != -1) {
        System.out.print((char) content);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

Nội dung file `input.txt`:

![](https://oss.javaguide.cn/github/javaguide/java/image-20220419154632551.png)

Kết quả đầu ra:

```plain
The actual number of characters skipped:3
The content read from file:我是Guide。
```

### Writer (Character Output Stream)

`Writer` được dùng để ghi dữ liệu (thông tin ký tự) vào đích đến (thường là file), class trừu tượng `java.io.Writer` là class cha của tất cả character output stream.

Các phương thức thường dùng của `Writer`:

- `write(int c)`: Ghi một ký tự đơn lẻ.
- `write(char[] cbuf)`: Ghi mảng ký tự `cbuf`, tương đương `write(cbuf, 0, cbuf.length)`.
- `write(char[] cbuf, int off, int len)`: Mở rộng phương thức `write(char[] cbuf)` với thêm tham số `off` (offset) và `len` (số ký tự tối đa cần đọc).
- `write(String str)`: Ghi chuỗi, tương đương `write(str, 0, str.length())`.
- `write(String str, int off, int len)`: Mở rộng phương thức `write(String str)` với thêm tham số `off` (offset) và `len` (số ký tự tối đa cần đọc).
- `append(CharSequence csq)`: Gắn chuỗi ký tự được chỉ định vào `Writer` được chỉ định và trả về `Writer` object đó.
- `append(char c)`: Gắn ký tự được chỉ định vào `Writer` được chỉ định và trả về `Writer` object đó.
- `flush()`: Flush output stream này và ghi tất cả ký tự đầu ra đã được buffer.
- `close()`: Đóng output stream và giải phóng tài nguyên hệ thống liên quan.

`OutputStreamWriter` là cầu nối chuyển đổi character stream thành byte stream, class con của nó `FileWriter` là lớp đóng gói dựa trên cơ sở đó, có thể ghi ký tự trực tiếp vào file.

```java
// Cầu nối chuyển đổi character stream thành byte stream
public class OutputStreamWriter extends Writer {
}
// Dùng để ghi ký tự vào file
public class FileWriter extends OutputStreamWriter {
}
```

Ví dụ code `FileWriter`:

```java
try (Writer output = new FileWriter("output.txt")) {
    output.write("你好，我是Guide。");
} catch (IOException e) {
    e.printStackTrace();
}
```

Kết quả đầu ra:

![](https://oss.javaguide.cn/github/javaguide/java/image-20220419155802288.png)

## Byte Buffered Stream

Thao tác IO rất tốn hiệu năng, buffered stream nạp dữ liệu vào vùng buffer, đọc/ghi nhiều byte cùng một lúc, từ đó tránh được các thao tác IO thường xuyên và tăng hiệu quả truyền dữ liệu.

Byte buffered stream ở đây áp dụng Decorator pattern để tăng cường chức năng của các object con của `InputStream` và `OutputStream`.

Ví dụ, chúng ta có thể tăng cường chức năng của `FileInputStream` thông qua `BufferedInputStream` (byte buffered input stream).

```java
// Tạo mới một đối tượng BufferedInputStream
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("input.txt"));
```

Sự khác biệt về hiệu năng giữa byte stream và byte buffered stream thể hiện rõ nhất khi cả hai đều gọi `write(int b)` và `read()` - hai phương thức mỗi lần chỉ đọc/ghi một byte. Vì byte buffered stream có vùng buffer nội bộ (mảng byte), byte buffered stream sẽ lưu trữ trước các byte đã đọc vào buffer, giảm đáng kể số lần IO, tăng hiệu quả đọc.

Tôi sử dụng phương thức `write(int b)` và `read()`, so sánh thời gian sao chép file PDF dung lượng `524.9 mb` bằng byte stream và byte buffered stream như sau:

```plain
Thời gian sao chép file PDF bằng buffered stream: 15428 mili giây
Thời gian sao chép file PDF bằng byte stream thông thường: 2555062 mili giây
```

Thời gian chênh lệch rất lớn, buffered stream tốn thời gian bằng 1/165 so với byte stream.

Code kiểm tra như sau:

```java
@Test
void copy_pdf_to_another_pdf_buffer_stream() {
    // Ghi lại thời gian bắt đầu
    long start = System.currentTimeMillis();
    try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("深入理解计算机操作系统.pdf"));
         BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("深入理解计算机操作系统-副本.pdf"))) {
        int content;
        while ((content = bis.read()) != -1) {
            bos.write(content);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    // Ghi lại thời gian kết thúc
    long end = System.currentTimeMillis();
    System.out.println("使用缓冲流复制PDF文件总耗时:" + (end - start) + " 毫秒");
}

@Test
void copy_pdf_to_another_pdf_stream() {
    // Ghi lại thời gian bắt đầu
    long start = System.currentTimeMillis();
    try (FileInputStream fis = new FileInputStream("深入理解计算机操作系统.pdf");
         FileOutputStream fos = new FileOutputStream("深入理解计算机操作系统-副本.pdf")) {
        int content;
        while ((content = fis.read()) != -1) {
            fos.write(content);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    // Ghi lại thời gian kết thúc
    long end = System.currentTimeMillis();
    System.out.println("使用普通流复制PDF文件总耗时:" + (end - start) + " 毫秒");
}
```

Nếu gọi `read(byte b[])` và `write(byte b[], int off, int len)` - hai phương thức ghi/đọc một mảng byte, miễn là kích thước mảng byte phù hợp, chênh lệch hiệu năng giữa hai loại thực ra không lớn, cơ bản có thể bỏ qua.

Lần này chúng ta dùng phương thức `read(byte b[])` và `write(byte b[], int off, int len)`, so sánh thời gian sao chép file PDF dung lượng 524.9 mb bằng byte stream và byte buffered stream như sau:

```plain
Thời gian sao chép file PDF bằng buffered stream: 695 mili giây
Thời gian sao chép file PDF bằng byte stream thông thường: 989 mili giây
```

Thời gian chênh lệch không nhiều, buffered stream chỉ tốt hơn một chút.

Code kiểm tra như sau:

```java
@Test
void copy_pdf_to_another_pdf_with_byte_array_buffer_stream() {
    // Ghi lại thời gian bắt đầu
    long start = System.currentTimeMillis();
    try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("深入理解计算机操作系统.pdf"));
         BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("深入理解计算机操作系统-副本.pdf"))) {
        int len;
        byte[] bytes = new byte[4 * 1024];
        while ((len = bis.read(bytes)) != -1) {
            bos.write(bytes, 0, len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    // Ghi lại thời gian kết thúc
    long end = System.currentTimeMillis();
    System.out.println("使用缓冲流复制PDF文件总耗时:" + (end - start) + " 毫秒");
}

@Test
void copy_pdf_to_another_pdf_with_byte_array_stream() {
    // Ghi lại thời gian bắt đầu
    long start = System.currentTimeMillis();
    try (FileInputStream fis = new FileInputStream("深入理解计算机操作系统.pdf");
         FileOutputStream fos = new FileOutputStream("深入理解计算机操作系统-副本.pdf")) {
        int len;
        byte[] bytes = new byte[4 * 1024];
        while ((len = fis.read(bytes)) != -1) {
            fos.write(bytes, 0, len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    // Ghi lại thời gian kết thúc
    long end = System.currentTimeMillis();
    System.out.println("使用普通流复制PDF文件总耗时:" + (end - start) + " 毫秒");
}
```

### BufferedInputStream (Byte Buffered Input Stream)

`BufferedInputStream` đọc dữ liệu (thông tin dạng byte) từ nguồn (thường là file) vào bộ nhớ không phải từng byte một, mà sẽ lưu trữ trước các byte đã đọc vào vùng cache, và đọc từng byte từ buffer nội bộ. Điều này giảm đáng kể số lần IO, tăng hiệu quả đọc.

`BufferedInputStream` duy trì nội bộ một vùng buffer, vùng buffer này thực chất là một mảng byte, có thể xác nhận điều này thông qua đọc source code của `BufferedInputStream`.

```java
public
class BufferedInputStream extends FilterInputStream {
    // Mảng buffer nội bộ
    protected volatile byte buf[];
    // Kích thước mặc định của buffer
    private static int DEFAULT_BUFFER_SIZE = 8192;
    // Dùng kích thước buffer mặc định
    public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }
    // Tùy chỉnh kích thước buffer
    public BufferedInputStream(InputStream in, int size) {
        super(in);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
}
```

Kích thước buffer mặc định là **8192** byte, tất nhiên bạn cũng có thể chỉ định kích thước buffer thông qua constructor `BufferedInputStream(InputStream in, int size)`.

### BufferedOutputStream (Byte Buffered Output Stream)

`BufferedOutputStream` ghi dữ liệu (thông tin dạng byte) vào đích đến (thường là file) không phải từng byte một, mà sẽ lưu trữ trước các byte cần ghi vào vùng cache, và ghi từng byte từ buffer nội bộ. Điều này giảm đáng kể số lần IO, tăng hiệu quả.

```java
try (BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("output.txt"))) {
    byte[] array = "JavaGuide".getBytes();
    bos.write(array);
} catch (IOException e) {
    e.printStackTrace();
}
```

Tương tự `BufferedInputStream`, `BufferedOutputStream` cũng duy trì nội bộ một vùng buffer, và kích thước của buffer này cũng là **8192** byte.

## Character Buffered Stream

`BufferedReader` (character buffered input stream) và `BufferedWriter` (character buffered output stream) tương tự `BufferedInputStream` (byte buffered input stream) và `BufferedOutputStream` (byte buffered output stream), nhưng cái trước duy trì nội bộ một character buffer, dùng để thao tác với thông tin ký tự.

## Print Stream

Đoạn code dưới đây mọi người hay sử dụng phải không?

```java
System.out.print("Hello！");
System.out.println("Hello！");
```

`System.out` thực ra là để lấy một object `PrintStream`, phương thức `print` thực tế gọi phương thức `write` của object `PrintStream`.

`PrintStream` là byte print stream, tương ứng với nó là `PrintWriter` (character print stream). `PrintStream` là class con của `OutputStream`, `PrintWriter` là class con của `Writer`.

```java
public class PrintStream extends FilterOutputStream
    implements Appendable, Closeable {
}
public class PrintWriter extends Writer {
}
```

## Random Access Stream

Random access stream được giới thiệu ở đây là `RandomAccessFile` - hỗ trợ nhảy tùy ý đến bất kỳ vị trí nào trong file để đọc/ghi.

Constructor của `RandomAccessFile` như sau, chúng ta có thể chỉ định `mode` (chế độ đọc/ghi).

```java
// Tham số openAndDelete mặc định là false, biểu thị mở file và file này sẽ không bị xóa
public RandomAccessFile(File file, String mode)
    throws FileNotFoundException {
    this(file, mode, false);
}
// Private method
private RandomAccessFile(File file, String mode, boolean openAndDelete)  throws FileNotFoundException{
  // Bỏ qua phần lớn code
}
```

Chế độ đọc/ghi chủ yếu có 4 loại:

- `r`: Chế độ chỉ đọc.
- `rw`: Chế độ đọc/ghi.
- `rws`: So với `rw`, `rws` đồng bộ cập nhật các thay đổi đối với "nội dung file" hoặc "metadata" ra thiết bị lưu trữ ngoài.
- `rwd`: So với `rw`, `rwd` đồng bộ cập nhật các thay đổi đối với "nội dung file" ra thiết bị lưu trữ ngoài.

Nội dung file là dữ liệu thực tế được lưu trong file, còn metadata là thông tin mô tả thuộc tính của file như kích thước file, thời gian tạo và thời gian sửa đổi.

`RandomAccessFile` có một file pointer biểu thị vị trí của byte tiếp theo sẽ được ghi hoặc đọc. Chúng ta có thể đặt offset của file pointer thông qua phương thức `seek(long pos)` của `RandomAccessFile` (cách đầu file `pos` byte). Để lấy vị trí hiện tại của file pointer, dùng phương thức `getFilePointer()`.

Ví dụ code `RandomAccessFile`:

```java
RandomAccessFile randomAccessFile = new RandomAccessFile(new File("input.txt"), "rw");
System.out.println("读取之前的偏移量：" + randomAccessFile.getFilePointer() + ",当前读取到的字符" + (char) randomAccessFile.read() + "，读取之后的偏移量：" + randomAccessFile.getFilePointer());
// Con trỏ hiện tại ở offset 6
randomAccessFile.seek(6);
System.out.println("读取之前的偏移量：" + randomAccessFile.getFilePointer() + ",当前读取到的字符" + (char) randomAccessFile.read() + "，读取之后的偏移量：" + randomAccessFile.getFilePointer());
// Bắt đầu ghi byte từ vị trí offset 7 trở đi
randomAccessFile.write(new byte[]{'H', 'I', 'J', 'K'});
// Con trỏ hiện tại ở offset 0, quay về vị trí ban đầu
randomAccessFile.seek(0);
System.out.println("读取之前的偏移量：" + randomAccessFile.getFilePointer() + ",当前读取到的字符" + (char) randomAccessFile.read() + "，读取之后的偏移量：" + randomAccessFile.getFilePointer());
```

Nội dung file `input.txt`:

![](https://oss.javaguide.cn/github/javaguide/java/image-20220421162050158.png)

Kết quả đầu ra:

```plain
读取之前的偏移量：0,当前读取到的字符A，读取之后的偏移量：1
读取之前的偏移量：6,当前读取到的字符G，读取之后的偏移量：7
读取之前的偏移量：0,当前读取到的字符A，读取之后的偏移量：1
```

Nội dung file `input.txt` trở thành `ABCDEFGHIJK`.

Phương thức `write` của `RandomAccessFile` khi ghi object vào vị trí đã có dữ liệu sẽ ghi đè lên dữ liệu đó.

```java
RandomAccessFile randomAccessFile = new RandomAccessFile(new File("input.txt"), "rw");
randomAccessFile.write(new byte[]{'H', 'I', 'J', 'K'});
```

Giả sử trước khi chạy đoạn chương trình trên, nội dung file `input.txt` là `ABCD`, sau khi chạy sẽ trở thành `HIJK`.

Một ứng dụng phổ biến của `RandomAccessFile` là thực hiện **tải file tiếp tục từ điểm dừng** (resumable download) cho file lớn. Tải file tiếp tục từ điểm dừng là gì? Đơn giản mà nói, là khi tải file bị tạm dừng hoặc thất bại giữa chừng (ví dụ do sự cố mạng), không cần tải lại từ đầu mà chỉ cần tải những phần chưa tải thành công. Tải theo từng phần (chia file thành nhiều phần nhỏ trước) là nền tảng của tải file tiếp tục từ điểm dừng.

`RandomAccessFile` có thể giúp chúng ta ghép các phần file, code ví dụ như sau:

![](https://oss.javaguide.cn/github/javaguide/java/io/20210609164749122.png)

Tôi đã giới thiệu chi tiết vấn đề tải file lớn trong [《Hướng dẫn phỏng vấn Java》](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html).

![](https://oss.javaguide.cn/github/javaguide/java/image-20220428104115362.png)

Cách triển khai của `RandomAccessFile` phụ thuộc vào `FileDescriptor` (file descriptor) và `FileChannel` (memory-mapped file).

<!-- @include: @article-footer.snippet.md -->
