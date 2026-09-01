---
title: Tổng hợp câu hỏi phỏng vấn Java cơ bản (Phần 3)
description: Tổng hợp các câu hỏi phỏng vấn tính năng nâng cao trong Java: giải thích chuyên sâu cơ chế xử lý ngoại lệ, nguyên lý Generics, ứng dụng Reflection, Annotation, cơ chế SPI, Serialization, mô hình IO Stream (BIO/NIO/AIO), Syntactic Sugar và các kiến thức cốt lõi khác.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Ngoại lệ Java,Generics,Reflection,Annotation,SPI,Serialization,IO Stream,Syntactic Sugar,try-with-resources,BIO NIO AIO,câu hỏi phỏng vấn Java
---

## Ngoại lệ (Exception)

**Tổng quan sơ đồ phân cấp lớp Ngoại lệ trong Java**:

![Sơ đồ phân cấp lớp Ngoại lệ trong Java](https://oss.javaguide.cn/github/javaguide/java/basis/types-of-exceptions-in-java.png)

### Exception và Error có gì khác biệt?

Trong Java, tất cả các ngoại lệ đều có chung một tổ tiên là lớp `Throwable` trong package `java.lang`. Lớp `Throwable` có 2 lớp con quan trọng:

- **`Exception`**: Ngoại lệ mà bản thân chương trình có thể xử lý được, có thể bắt thông qua `catch`. `Exception` lại chia thành Checked Exception (Ngoại lệ bị kiểm tra, bắt buộc phải xử lý) và Unchecked Exception (Ngoại lệ không bị kiểm tra, có thể không cần xử lý).
- **`Error`**: `Error` thuộc về lỗi mà chương trình không thể xử lý được, không khuyến nghị bắt bằng `catch`. Ví dụ lỗi vận hành JVM (`VirtualMachineError`), lỗi thiếu bộ nhớ JVM (`OutOfMemoryError`), lỗi định nghĩa class (`NoClassDefFoundError`),... Khi những ngoại lệ này xảy ra, JVM thông thường sẽ chọn kết thúc Thread.

### Phân biệt ClassNotFoundException và NoClassDefFoundError

- `ClassNotFoundException` là một Exception, xảy ra khi dùng Reflection hoặc nạp class động mà không tìm thấy class, là điều có thể dự đoán và có thể catch để xử lý.
- `NoClassDefFoundError` là một Error, biểu thị JVM hoặc ClassLoader cố gắng nạp định nghĩa class nhưng không tìm thấy. Ngoài việc thiếu file JAR lúc runtime, nó còn có thể do khởi tạo class thất bại sau đó lại tiếp tục dùng class đó gây ra. Nó thường kết thúc Thread hiện tại, nhưng không đồng nghĩa với việc toàn bộ JVM bắt buộc bị dừng.

### ⭐️ Phân biệt Checked Exception và Unchecked Exception

**Checked Exception** (Ngoại lệ bị kiểm tra): Trong quá trình biên dịch mã nguồn Java, nếu Checked Exception không được xử lý bằng `catch` hoặc từ khóa `throws` thì mã nguồn sẽ không thể vượt qua vòng biên dịch.

Ví dụ đoạn code thao tác I/O dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/basis/checked-exception.png)

Ngoại trừ `RuntimeException` và các lớp con của nó, các lớp `Exception` khác và lớp con của chúng đều thuộc Checked Exception. Các Checked Exception phổ biến gồm: các ngoại lệ liên quan đến I/O, `ClassNotFoundException`, `SQLException`,...

**Unchecked Exception** (Ngoại lệ không bị kiểm tra): Trong quá trình biên dịch mã nguồn Java, ngay cả khi chúng ta không xử lý Unchecked Exception thì mã nguồn vẫn vượt qua vòng biên dịch bình thường.

`RuntimeException` và các lớp con của nó thuộc Unchecked Exception; theo phân loại của JLS thì `Error` và các lớp con của nó cũng thuộc Unchecked Exception. Các `RuntimeException` phổ biến gồm:

- `NullPointerException` (Lỗi con trỏ null)
- `IllegalArgumentException` (Lỗi tham số không hợp lệ, ví dụ kiểu tham số truyền vào phương thức bị sai)
- `NumberFormatException` (Lỗi định dạng khi chuyển chuỗi thành số, là lớp con của `IllegalArgumentException`)
- `ArrayIndexOutOfBoundsException` (Lỗi truy cập vượt quá chỉ mục mảng)
- `ClassCastException` (Lỗi ép kiểu)
- `ArithmeticException` (Lỗi phép toán số học, như chia cho 0)
- `SecurityException` (Lỗi an toàn bảo mật, như không đủ quyền)
- `UnsupportedOperationException` (Lỗi thao tác không được hỗ trợ, như tạo trùng user)
- ……

![](https://oss.javaguide.cn/github/javaguide/java/basis/unchecked-exception.png)

### Bạn thiên về sử dụng Checked Exception hay Unchecked Exception?

Mặc định sử dụng Unchecked Exception, chỉ dùng Checked Exception khi thật sự cần thiết.

Chúng ta có thể xem Unchecked Exception (như `NullPointerException`) là Bug trong code. Đối với Bug, cách xử lý tốt nhất là để nó lộ ra rồi sửa code, chứ không phải dùng `try-catch` để che giấu nó.

Thông thường, chỉ sử dụng Checked Exception trong một trường hợp: Khi ngoại lệ này là một phần của logic nghiệp vụ và bên gọi bắt buộc phải xử lý nó. Ví dụ: Ngoại lệ không đủ số dư tài khoản. Đây không phải bug mà là một nhánh nghiệp vụ bình thường, tôi cần dùng Checked Exception để bắt buộc người gọi phải xử lý tình huống này, ví dụ nhắc người dùng nạp tiền. Nhờ đó vừa đảm bảo tính toàn vẹn của logic nghiệp vụ quan trọng, vừa giúp code giữ được sự gọn gàng nhất có thể.

### Các phương thức phổ biến của lớp Throwable là gì?

- `String getMessage()`: Trả về thông tin chi tiết khi xảy ra ngoại lệ
- `String toString()`: Trả về mô tả ngắn gọn khi xảy ra ngoại lệ
- `String getLocalizedMessage()`: Trả về thông tin bản địa hóa của đối tượng ngoại lệ. Override phương thức này trong lớp con của `Throwable` để tạo thông tin bản địa hóa. Nếu lớp con không override thì kết quả giống `getMessage()`
- `void printStackTrace()`: In thông tin ngoại lệ được đóng gói trong đối tượng `Throwable` ra console

### Cách sử dụng try-catch-finally?

- Khối `try`: Dùng để bắt ngoại lệ. Phía sau có thể đi kèm 0 hoặc nhiều khối `catch`, nếu không có khối `catch` thì bắt buộc phải có một khối `finally`.
- Khối `catch`: Dùng để xử lý ngoại lệ bắt được ở try.
- Khối `finally`: Dù có bắt hay xử lý được ngoại lệ hay không, các câu lệnh trong khối `finally` đều sẽ được thực thi. Khi gặp câu lệnh `return` trong khối `try` hoặc `catch`, khối câu lệnh `finally` sẽ được thực thi trước khi phương thức trả về.

Code ví dụ:

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
```

Kết quả:

```plain
Try to do something
Catch Exception -> RuntimeException
Finally
```

**Lưu ý: Không nên dùng return trong khối finally!** Khi cả khối try và khối finally đều có câu lệnh return, câu lệnh return trong khối try sẽ bị bỏ qua. Đó là vì giá trị trả về của return trong khối try sẽ được tạm lưu vào một biến cục bộ, khi thực thi đến return trong câu lệnh finally, giá trị của biến cục bộ này sẽ bị thay đổi thành giá trị trả về của return trong finally.

Code ví dụ:

```java
public static void main(String[] args) {
    System.out.println(f(2));
}

public static int f(int value) {
    try {
        return value * value;
    } finally {
        if (value == 2) {
            return 0;
        }
    }
}
```

Kết quả:

```plain
0
```

### Code trong finally có chắc chắn được thực thi không?

Không chắc chắn! Trong một số trường hợp, code trong finally sẽ không được thực thi.

Ví dụ trước khi vào finally mà JVM bị dừng hoạt động thì code trong finally sẽ không được thực thi.

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
    // Dừng máy ảo Java đang chạy
    System.exit(1);
} finally {
    System.out.println("Finally");
}
```

Kết quả:

```plain
Try to do something
Catch Exception -> RuntimeException
```

Ngoài ra, nếu tiến trình JVM bị cưỡng chế dừng, ví dụ gọi `Runtime.halt()`, hệ điều hành kill tiến trình hoặc máy bị mất điện, khối `finally` cũng có thể không kịp thực thi. Các uncaught exception thông thường ngay cả khi làm kết thúc Thread hiện tại thì trước khi Thread kết thúc vẫn sẽ thực thi `finally` theo quy tắc ngôn ngữ.

Issue liên quan: <https://github.com/Snailclimb/JavaGuide/issues/190>.

🧗🏻 Nâng cao: Phân tích nguyên lý thực thi đằng sau cú pháp `try catch finally` dưới góc độ bytecode.

### Cách dùng `try-with-resources` thay thế `try-catch-finally`?

1. **Phạm vi áp dụng (Định nghĩa tài nguyên):** Bất kỳ đối tượng nào implement `java.lang.AutoCloseable` hoặc `java.io.Closeable`.
2. **Thứ tự đóng tài nguyên và thực thi khối finally:** Trong câu lệnh `try-with-resources`, bất kỳ khối catch hoặc finally nào cũng chạy sau khi các tài nguyên đã khai báo được đóng.

Sách 《Effective Java》 chỉ rõ:

> Khi đối mặt với tài nguyên bắt buộc phải đóng, chúng ta luôn nên ưu tiên dùng `try-with-resources` thay vì `try-finally`. Code tạo ra gọn gàng hơn, rõ ràng hơn và ngoại lệ sinh ra cũng có ích hơn cho chúng ta. Câu lệnh `try-with-resources` giúp chúng ta dễ dàng viết code cho tài nguyên bắt buộc phải đóng, điều mà nếu dùng `try-finally` thì gần như rất khó làm được.

Các tài nguyên trong Java như `InputStream`, `OutputStream`, `Scanner`, `PrintWriter`,... đều cần chúng ta gọi phương thức `close()` để đóng thủ công, thông thường chúng ta dùng `try-catch-finally` để thực hiện yêu cầu này như sau:

```java
// Đọc nội dung file text
Scanner scanner = null;
try {
    scanner = new Scanner(new File("D://read.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

Dùng câu lệnh `try-with-resources` từ Java 7 trở đi để cải tiến đoạn code trên:

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

Tất nhiên khi có nhiều tài nguyên cần đóng, việc dùng `try-with-resources` thực thi cũng rất đơn giản, nếu bạn vẫn dùng `try-catch-finally` có thể mang lại rất nhiều vấn đề.

Thông qua phân cách bằng dấu chấm phẩy, có thể khai báo nhiều tài nguyên trong khối `try-with-resources`.

```java
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))) {
    int b;
    while ((b = bin.read()) != -1) {
        bout.write(b);
    }
}
catch (IOException e) {
    e.printStackTrace();
}
```

### ⭐️ Các điểm cần lưu ý khi sử dụng Exception?

- Không định nghĩa ngoại lệ làm biến static, vì như vậy sẽ khiến thông tin stack ngoại lệ bị xáo trộn. Mỗi lần throw ngoại lệ thủ công, chúng ta cần new một đối tượng ngoại lệ mới để throw.
- Thông tin ngoại lệ ném ra bắt buộc phải có ý nghĩa.
- Khuyên nên ném ra ngoại lệ cụ thể hơn, ví dụ lỗi định dạng chuyển chuỗi thành số nên throw `NumberFormatException` thay vì lớp bố `IllegalArgumentException`.
- Tránh ghi log trùng lặp: Nếu tại nơi catch ngoại lệ đã ghi đủ thông tin (bao gồm kiểu ngoại lệ, thông tin lỗi và stack trace), thì khi rethrow ngoại lệ này trong code nghiệp vụ không nên ghi lại thông tin lỗi tương tự nữa. Log trùng lặp sẽ làm phình file log và có thể che giấu nguyên nhân thực sự của sự cố, khiến việc truy vết trở nên khó khăn hơn.
- ……

## Generics

### Generics là gì? Có tác dụng gì?

**Java Generics (Kiểu chung)** là một tính năng mới được đưa vào từ JDK 5. Sử dụng tham số Generics giúp tăng tính đọc hiểu và tính ổn định của code.

Trình biên dịch có thể kiểm tra tham số Generics, và thông qua tham số Generics có thể chỉ định kiểu đối tượng truyền vào. Ví dụ dòng code `ArrayList<Person> persons = new ArrayList<Person>()` chỉ rõ đối tượng `ArrayList` này chỉ có thể nhận đối tượng `Person`, nếu truyền vào đối tượng kiểu khác sẽ báo lỗi biên dịch.

```java
ArrayList<E> extends AbstractList<E>
```

Đồng thời, `List` nguyên thủy trả về kiểu `Object`, cần ép kiểu thủ công mới dùng được, sau khi dùng Generics thì trình biên dịch tự động chuyển đổi.

### Generics có những cách sử dụng nào?

Generics thông thường có 3 cách sử dụng: **Generic Class**, **Generic Interface**, **Generic Method**.

**1. Generic Class**:

```java
// T ở đây có thể viết tùy ý thành ký hiệu bất kỳ, phổ biến như T, E, K, V,...
// Khi khởi tạo Generic Class bắt buộc phải chỉ định kiểu cụ thể cho T
public class Generic<T> {

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey() {
        return key;
    }
}
```

Cách khởi tạo Generic Class:

```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

**2. Generic Interface**:

```java
public interface Generator<T> {
    public T method();
}
```

Implement Generic Interface nhưng không chỉ định kiểu:

```java
class GeneratorImpl<T> implements Generator<T> {
    @Override
    public T method() {
        return null;
    }
}
```

Implement Generic Interface và chỉ định kiểu cụ thể:

```java
class GeneratorImpl implements Generator<String> {
    @Override
    public String method() {
        return "hello";
    }
}
```

**3. Generic Method**:

```java
   public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```

Cách dùng:

```java
// Tạo các mảng thuộc kiểu khác nhau: Integer, Double và Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray );
printArray( stringArray );
```

> Lưu ý: `public static <E> void printArray(E[] inputArray)` là phương thức Generic static. Ngữ cảnh static không có instance hiện tại nên không thể tham chiếu tham số kiểu do class khai báo; điều này không liên quan đến việc "phương thức static được nạp trước". Phương thức static có thể khai báo và sử dụng tham số kiểu `<E>` của riêng nó.

### Trong dự án hay dùng Generics ở đâu?

- Kết quả trả về chung tùy chỉnh `CommonResult<T>` thông qua tham số `T` có thể chỉ định động kiểu dữ liệu của kết quả theo kiểu trả về cụ thể
- Định nghĩa lớp xử lý `ExcelUtil<T>` dùng để chỉ định động kiểu dữ liệu xuất ra `Excel`
- Xây dựng các lớp utility tập hợp (tham khảo các phương thức `sort`, `binarySearch` trong `Collections`).
- ……

## ⭐️ Reflection

Về phân tích chi tiết Reflection, vui lòng đọc bài viết [Giải thích chi tiết cơ chế Reflection trong Java](https://javaguide.cn/java/basis/reflection.html).

### Reflection là gì?

Nói một cách đơn giản, Java Reflection (Phản xạ) là một **khả năng động lấy thông tin của class và thao tác trên class hoặc object (phương thức, thuộc tính) tại thời điểm chương trình đang chạy (runtime)**.

Thông thường, code chúng ta viết đã xác định kiểu ở thời điểm biên dịch, gọi phương thức nào, truy cập field nào đều rõ ràng. Nhưng Reflection cho phép chúng ta tại thời điểm **runtime** mới dò tìm một class có những phương thức nào, thuộc tính nào, constructor của nó ra sao, và trong điều kiện kiểm soát truy cập và ranh giới module cho phép có thể khởi tạo đối tượng, gọi phương thức hoặc chỉnh sửa thuộc tính một cách động.

Chính khả năng "nhìn lại bản thân" và thao tác tại thời điểm runtime này khiến Reflection trở thành **nền tảng của nhiều framework và thư viện chung**. Nó giúp code linh hoạt hơn, có thể xử lý các kiểu dữ liệu chưa xác định ở thời điểm biên dịch.

### Reflection có ưu nhược điểm gì?

**Ưu điểm:**

1. **Tính linh hoạt và tính động**: Reflection cho phép chương trình nạp class, tạo object, gọi method và truy cập field một cách động tại thời điểm runtime, thích ứng và mở rộng hành vi của chương trình dựa trên nhu cầu thực tế (như file cấu hình, user input, annotation,...). Nhiều Java framework hiện đại (như Spring, Hibernate, MyBatis) dựa trên đặc tính này để thực hiện Dependency Injection (DI), Aspect-Oriented Programming (AOP), Object-Relational Mapping (ORM), xử lý Annotation,... Có thể nói Reflection là nền tảng không thể thiếu của phát triển framework.
2. **Giảm phụ thuộc (Decoupling) và tính tổng quát**: Thông qua Reflection, có thể viết code tổng quát hơn, tái sử dụng cao hơn và giảm phụ thuộc giữa các module. Ví dụ có thể dùng Reflection thực thi copy object, serialization, Bean tool chung.

**Nhược điểm:**

1. **Chi phí hiệu năng**: Các thao tác Reflection thường chậm hơn so với gọi code trực tiếp. Bởi vì nó liên quan đến phân tích kiểu động, tìm kiếm phương thức và sự hạn chế tối ưu hóa của trình biên dịch JIT. Tuy nhiên đối với hầu hết kịch bản framework, tổn thất hiệu năng này thường chấp nhận được, hoặc bản thân framework sẽ làm một số tối ưu hóa bằng cache.
2. **Vấn đề an toàn bảo mật**: Reflection khi đáp ứng điều kiện kiểm tra truy cập và quan hệ mở module có thể bỏ qua một phần kiểm tra truy cập của ngôn ngữ Java (như truy cập field và method `private`), có thể phá vỡ tính đóng gói. Ngoài ra Reflection còn có thể bỏ qua kiểm tra Generics lúc biên dịch, mang lại rủi ro an toàn kiểu dữ liệu. Ranh giới module từ Java 9 trở đi có thể từ chối các truy cập Reflection sâu này và throw `InaccessibleObjectException`.
3. **Tính dễ đọc và khả năng bảo trì code**: Lạm dụng Reflection sẽ khiến code trở nên phức tạp, khó hiểu và khó debug. Lỗi thường chỉ lộ ra lúc runtime chứ không dễ phát hiện như lỗi lúc biên dịch.

Đọc thêm: [Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow).

### Kịch bản ứng dụng của Reflection?

Khi viết code nghiệp vụ hàng ngày chúng ta có thể ít làm việc trực tiếp với Reflection. Nhưng có thể bạn chưa nhận ra mình đang tận hưởng sự tiện lợi do Reflection mang lại mỗi ngày! **Rất nhiều framework nổi tiếng như Spring/Spring Boot, MyBatis,... bên dưới đều áp dụng rộng rãi cơ chế Reflection**, nhờ đó chúng mới trở nên linh hoạt và mạnh mẽ như vậy.

Dưới đây liệt kê một số kịch bản phổ biến nhất giúp mọi người dễ hiểu.

**1. Dependency Injection và Inversion of Control (IoC)**

Các IoC framework đại diện như Spring/Spring Boot khi khởi động sẽ quét các class có annotation đặc biệt (như `@Component`, `@Service`, `@Repository`, `@Controller`), tận dụng Reflection để khởi tạo đối tượng (Bean), và thông qua Reflection để inject phụ thuộc (như `@Autowired`, constructor injection,...).

**2. Xử lý Annotation**

Annotation bản thân chỉ là một "nhãn dán", phải có người đọc nhãn đó mới biết cần làm gì. Reflection chính là "đầu đọc" đó. Framework thông qua Reflection để kiểm tra class, method, field có annotation đặc biệt không, sau đó dựa vào thông tin annotation để thực thi logic tương ứng. Ví dụ nhìn thấy `@Value` sẽ dùng Reflection đọc nội dung annotation, tìm giá trị tương ứng trong file cấu hình, rồi dùng Reflection gán giá trị đó cho field.

**3. Dynamic Proxy và AOP**

Muốn tự động thêm logic vào trước/sau khi gọi một phương thức (như ghi log, mở transaction, kiểm tra quyền)? AOP (Aspect-Oriented Programming) làm việc này, và Dynamic Proxy là giải pháp phổ biến để thực hiện AOP. Dynamic Proxy tích hợp sẵn của JDK (`Proxy` và `InvocationHandler`) không thể tách rời Reflection. Khi proxy object gọi phương thức của target object bên trong, nó được thực hiện thông qua `Method.invoke` của Reflection.

```java
public class DebugInvocationHandler implements InvocationHandler {
    private final Object target; // Target object thực tế

    public DebugInvocationHandler(Object target) { this.target = target; }

    // proxy: Proxy object, method: Phương thức được gọi, args: Tham số phương thức
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Logic Aspect: Trước khi gọi phương thức " + method.getName());
        // Thông qua Reflection gọi phương thức trùng tên của target object
        Object result = method.invoke(target, args);
        System.out.println("Logic Aspect: Sau khi gọi phương thức " + method.getName());
        return result;
    }
}
```

**4. Object-Relational Mapping (ORM)**

Các framework như MyBatis, Hibernate giúp bạn tự động chuyển từng dòng dữ liệu truy vấn từ database thành các đối tượng Java. Làm sao nó biết cột trong database tương ứng với thuộc tính nào của Java? Vẫn dựa vào Reflection. Nó thông qua Reflection lấy danh sách thuộc tính của lớp Java, sau đó khớp kết quả truy vấn theo tên hoặc cấu hình, rồi dùng Reflection gọi setter hoặc chỉnh sửa trực tiếp giá trị field. Ngược lại khi lưu đối tượng vào database, nó cũng dùng Reflection đọc giá trị thuộc tính để ghép thành câu lệnh SQL.

## Proxy (Ủy quyền / Đại lý)

Về giới thiệu chi tiết Proxy trong Java, vui lòng đọc bài viết [Giải thích chi tiết Proxy Pattern trong Java](https://javaguide.cn/java/basis/proxy.html).

### Thực hiện Dynamic Proxy như thế nào?

Dynamic Proxy (Đại lý động) là một design pattern rất mạnh mẽ, nó cho phép chúng ta **tăng cường tính năng (Enhancement)** cho phương thức của một class hoặc object mà **không cần chỉnh sửa mã nguồn**.

Trong Java, có 2 cách phổ biến nhất để thực hiện Dynamic Proxy: **JDK Dynamic Proxy** và **CGLIB Dynamic Proxy**.

**Cách 1: JDK Dynamic Proxy**

Do Java chính thức cung cấp, yêu cầu cốt lõi là target class bắt buộc phải implement một hoặc nhiều Interface. JDK Dynamic Proxy lúc runtime sẽ tận dụng phương thức `Proxy.newProxyInstance()` để tạo động một instance của proxy class implement các interface đó. Proxy class này được sinh ra trong bộ nhớ, bạn không nhìn thấy file `.java` hay `.class` của nó.

Khi bạn gọi bất kỳ phương thức nào của proxy object, cuộc gọi đó sẽ được chuyển tiếp tới phương thức `invoke` của interface `InvocationHandler` do chúng ta cung cấp. Trong phương thức `invoke`, chúng ta có thể thêm logic tăng cường của mình vào trước hoặc sau khi gọi phương thức gốc (target method).

**Cách 2: CGLIB Dynamic Proxy**

CGLIB là một thư viện sinh code của bên thứ ba. Nguyên lý của nó hoàn toàn khác JDK, nó không yêu cầu target class phải implement interface. Lúc runtime, nó tạo động lớp con của target class để làm proxy class (thông qua kỹ thuật thao tác bytecode ASM). Sau đó, nó override tất cả các phương thức non-`final`, non-`private` và non-`static` trong lớp bố (tức class được proxy).

Khi bạn gọi bất kỳ phương thức nào của proxy object, cuộc gọi đó sẽ bị chặn bởi phương thức `intercept` của interface `MethodInterceptor` trong CGLIB. Giống như `invoke` của `InvocationHandler`, chúng ta có thể thêm logic tăng cường vào trước hoặc sau khi gọi phương thức của lớp bố gốc trong phương thức `intercept`.

### Phân biệt Static Proxy và Dynamic Proxy?

Sự khác biệt cốt lõi giữa Static Proxy và Dynamic Proxy nằm ở **Thời điểm xác định quan hệ proxy, Tính linh hoạt khi thực thi và Chi phí bảo trì**.

| Tiêu chí so sánh | Static Proxy | Dynamic Proxy |
| --- | --- | --- |
| **Thời điểm xác định quan hệ Proxy** | Thời điểm biên dịch (sau khi biên dịch sinh ra file bytecode `.class` cố định) | Thời điểm runtime (sinh động bytecode của proxy class và nạp vào JVM) |
| **Cách thực thi** | Trước khi biên dịch viết thủ công proxy class, thường qua kết hợp và ủy quyền gọi target object | Không cần viết thủ công proxy class cụ thể, đóng gói logic tăng cường qua `Handler`/`Interceptor` |
| **Phụ thuộc Interface** | Không bắt buộc; Static Proxy dựa trên interface thường để proxy class và target class chung interface | JDK Dynamic Proxy hướng tới Interface, CGLIB proxy lớp con hướng tới implementation class có thể kế thừa |
| **Lượng code & Bảo trì** | Lượng code lớn (càng nhiều target class càng nhiều proxy class), chi phí bảo trì cao; khi interface thêm method, target class và proxy class phải sửa đồng bộ | Lượng code cực ít (logic tăng cường chung có thể tái sử dụng), tính bảo trì tốt; tách biệt với interface, thay đổi interface không ảnh hưởng logic proxy |
| **Ưu thế cốt lõi** | Thực hiện đơn giản, logic trực quan, không phụ thuộc framework ngoài | Tính linh hoạt mạnh, tính tái sử dụng cao, giảm viết code trùng lặp, thích ứng kịch bản phức tạp |
| **Kịch bản ứng dụng điển hình** | Decorator Pattern đơn giản, nhu cầu tăng cường cho ít class cố định | Spring AOP, RPC framework (như Dubbo), ORM framework |

### ⭐️ Phân biệt JDK Dynamic Proxy và CGLIB Dynamic Proxy?

1. JDK Dynamic Proxy là chính thức của Java, yêu cầu class được proxy bắt buộc phải implement Interface. Nguyên lý của nó là sinh động một implementation class của interface để làm proxy. CGLIB là của bên thứ ba, không yêu cầu interface. Nguyên lý của nó là sinh động một lớp con của class được proxy để làm proxy. Nhưng cũng vì dựa trên kế thừa nên nó không thể proxy class `final`, và phương thức được proxy cũng không thể là `final` hoặc `private`.
2. Về mặt hiệu năng giữa hai bên, đại đa số trường hợp JDK Dynamic Proxy đều tốt hơn, cùng với sự nâng cấp của các phiên bản JDK, ưu thế này ngày càng rõ rệt.

### ⭐️ Giới thiệu kịch bản ứng dụng thực tế của Dynamic Proxy trong Framework

Kịch bản ứng dụng điển hình nhất của Dynamic Proxy chính là **Spring AOP**.

AOP (Aspect-Oriented Programming: Lập trình hướng khía cạnh) có thể đóng gói những logic hoặc trách nhiệm không liên quan đến nghiệp vụ nhưng lại được các module nghiệp vụ gọi chung (như xử lý transaction, quản lý log, kiểm soát quyền,...), giúp giảm code trùng lặp trong hệ thống, giảm độ phụ thuộc giữa các module, thuận tiện cho tính mở rộng và bảo trì trong tương lai.

Spring AOP chính là dựa trên Dynamic Proxy. Nếu object cần proxy có implement một interface nào đó thì Spring AOP sẽ dùng **JDK Proxy** để tạo proxy object; còn đối với object không implement interface thì không thể dùng JDK Proxy, lúc này Spring AOP sẽ dùng **CGLIB** sinh một lớp con của target object làm proxy, như hình dưới đây:

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

## Annotation (Chú thích / Chú giải)

### Annotation là gì?

`Annotation` (Chú thích) là tính năng mới được đưa vào từ Java 5, có thể xem như một loại comment đặc biệt, chủ yếu dùng để修饰 class, method hoặc variable, cung cấp một số thông tin nhất định cho chương trình sử dụng lúc biên dịch hoặc runtime.

Annotation về bản chất là một interface đặc biệt kế thừa `Annotation`:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {

}

public interface Override extends Annotation {

}
```

JDK cung cấp rất nhiều annotation tích hợp sẵn (như `@Override`, `@Deprecated`), đồng thời chúng ta cũng có thể tự định nghĩa annotation.

### Có những phương pháp parse Annotation nào?

Annotation chỉ có hiệu lực sau khi được parse, có 2 phương pháp parse phổ biến:

- **Quét trực tiếp lúc biên dịch**: Trình biên dịch khi biên dịch code Java sẽ quét các annotation tương ứng và xử lý, ví dụ một method sử dụng annotation `@Override`, trình biên dịch khi biên dịch sẽ kiểm tra xem method hiện tại có thực sự override method tương ứng của lớp bố hay không.
- **Xử lý qua Reflection lúc runtime**: Các annotation đi kèm trong framework (như `@Value`, `@Component` của Spring framework) đều được xử lý thông qua Reflection.

## ⭐️ SPI

Về phân tích chi tiết SPI, vui lòng đọc bài viết [Giải thích chi tiết cơ chế SPI trong Java](https://javaguide.cn/java/basis/spi.html).

### SPI là gì?

SPI viết tắt của Service Provider Interface, nghĩa trên mặt chữ là: "Giao diện của nhà cung cấp dịch vụ", theo cách hiểu của tôi: Đây là một interface chuyên cung cấp cho các bên cung cấp dịch vụ hoặc các nhà phát triển mở rộng tính năng framework sử dụng.

SPI tách biệt interface dịch vụ và bản thực thi dịch vụ cụ thể, giảm phụ thuộc giữa bên gọi dịch vụ và bên thực thi dịch vụ, giúp tăng tính mở rộng và khả năng bảo trì của chương trình. Việc chỉnh sửa hoặc thay thế bản thực thi dịch vụ không cần phải sửa đổi bên gọi.

Rất nhiều framework sử dụng cơ chế SPI của Java, ví dụ: Spring framework, driver nạp database, interface log, và các bản mở rộng của Dubbo,...

<img src="https://oss.javaguide.cn/github/javaguide/java/basis/spi/22e1830e0b0e4115a882751f6c417857tplv-k3u1fbpfcp-zoom-1.jpeg" style="zoom:50%;" />

### SPI và API khác nhau như thế nào?

**Vậy SPI và API khác nhau như thế nào?**

Nói đến SPI thì không thể không nhắc tới API (Application Programming Interface). Theo nghĩa rộng chúng đều thuộc về interface, và rất dễ bị nhầm lẫn. Dưới đây dùng một hình vẽ để giải thích:

![SPI VS API](https://oss.javaguide.cn/github/javaguide/java/basis/spi-vs-api.png)

Thông thường giữa các module giao tiếp với nhau qua interface, do đó chúng ta đưa vào một "interface" giữa bên gọi dịch vụ và bên thực thi dịch vụ (còn gọi là bên cung cấp dịch vụ).

- Khi bên thực thi cung cấp cả interface và bản thực thi, chúng ta có thể gọi interface của bên thực thi để sở hữu năng lực mà bên thực thi cung cấp cho chúng ta, đây chính là **API**. Trong trường hợp này, cả interface và bản thực thi đều nằm trong package của bên thực thi. Bên gọi thông qua interface để gọi tính năng của bên thực thi mà không cần quan tâm chi tiết thực thi cụ thể.
- Khi interface nằm ở phía bên gọi, đây chính là **SPI**. Bên gọi interface xác định quy tắc interface, sau đó các hãng sản xuất khác nhau căn cứ vào quy tắc này để thực thi interface đó, từ đó cung cấp dịch vụ.

Một ví dụ dễ hiểu: Công ty H là một công ty công nghệ, mới thiết kế ra một mẫu chip và giờ cần sản xuất hàng loạt. Trên thị trường có vài công ty sản xuất chip, lúc này chỉ cần công ty H chỉ định rõ tiêu chuẩn sản xuất chip này (định nghĩa xong tiêu chuẩn interface), thì các công ty chip hợp tác (bên cung cấp dịch vụ) sẽ giao chip mang đặc trưng riêng của họ theo đúng tiêu chuẩn (cung cấp các bản thực thi phương án khác nhau, nhưng kết quả đưa ra là giống nhau).

### Ưu nhược điểm của SPI?

Thông qua cơ chế SPI có thể làm tăng đáng kể tính linh hoạt khi thiết kế interface, tuy nhiên cơ chế SPI cũng có một số nhược điểm, ví dụ:

- `ServiceLoader` sẽ định vị và khởi tạo provider theo nhu cầu; nếu bên gọi duyệt qua toàn bộ provider để lựa chọn bản thực thi thì mới kích hoạt việc nạp toàn bộ các provider khả dụng.
- Một instance `ServiceLoader` đơn lẻ không đảm bảo an toàn đa luồng; giữa các instance khác nhau không tồn tại quy tắc "cùng `load` là bắt buộc xung đột".

## ⭐️ Serialization và Deserialization

Về phân tích chi tiết Serialization và Deserialization, vui lòng đọc bài viết [Giải thích chi tiết Java Serialization](https://javaguide.cn/java/basis/serialization.html).

### Serialization là gì? Deserialization là gì?

Nếu chúng ta cần lưu trữ lâu dài đối tượng Java như lưu vào file, hoặc truyền đối tượng Java qua mạng, những kịch bản này đều cần dùng đến Serialization.

Nói một cách đơn giản:

- **Serialization (Tuần tự hóa / Mã hóa đối tượng)**: Chuyển đổi cấu trúc dữ liệu hoặc đối tượng thành dạng có thể lưu trữ hoặc truyền đi, thông thường là luồng byte nhị phân (binary byte stream), cũng có thể là định dạng văn bản như JSON, XML.
- **Deserialization (Giải tuần tự hóa / Giải mã đối tượng)**: Quá trình chuyển đổi dữ liệu được sinh ra trong quá trình serialization trở lại thành cấu trúc dữ liệu hoặc đối tượng ban đầu.

Đối với ngôn ngữ lập trình hướng đối tượng như Java, thứ chúng ta serialize đều là đối tượng (Object) tức class đã được khởi tạo (Class), nhưng trong ngôn ngữ bán hướng đối tượng như C++, struct (cấu trúc) định nghĩa kiểu cấu trúc dữ liệu, còn class tương ứng với kiểu đối tượng.

Dưới đây là các kịch bản ứng dụng phổ biến của serialization và deserialization:

- Đối tượng trước khi truyền qua mạng (như khi gọi phương thức từ xa RPC) cần được serialize trước, sau khi nhận được đối tượng đã serialize cần deserialize lại;
- Trước khi lưu đối tượng vào file cần serialize, khi đọc đối tượng từ file ra cần deserialize;
- Trước khi lưu đối tượng vào database (như Redis) cần dùng serialize, khi đọc đối tượng từ cache database ra cần deserialize;
- Khi chuyển đối tượng thành dạng byte biểu diễn cần lưu trữ lâu dài hoặc truyền qua các component, thông thường cần serialization; đối tượng Java thông thường sử dụng trong bộ nhớ JVM thì không cần serialization.

Wikipedia giới thiệu về serialization như sau:

> **Tuần tự hóa** (serialization) trong xử lý dữ liệu của khoa học máy tính là quá trình chuyển đổi cấu trúc dữ liệu hoặc trạng thái đối tượng thành định dạng có thể truy xuất (ví dụ lưu thành file, lưu trong bộ đệm, hoặc gửi qua mạng), để phục vụ cho quá trình khôi phục lại trạng thái ban đầu sau này trong cùng môi trường máy tính hoặc một môi trường máy tính khác. Khi thu lại các byte theo định dạng tuần tự hóa, có thể tận dụng nó để tạo ra bản sao có cùng ngữ nghĩa với đối tượng ban đầu. Với nhiều đối tượng, như các đối tượng phức tạp sử dụng lượng lớn tham chiếu, quá trình tái tạo tuần tự hóa này không dễ dàng. Tuần tự hóa đối tượng trong hướng đối tượng không khái quát các hàm liên quan của đối tượng ban đầu. Quá trình này còn gọi là marshalling. Thao tác ngược lại trích xuất cấu trúc dữ liệu từ một chuỗi byte là deserialization (còn gọi là unmarshalling).

Tóm lại: **Mục đích chính của Serialization là chuyển đối tượng thành dạng phù hợp để truyền qua mạng hoặc lưu trữ lâu dài vào file system, database, cache,...**

![](https://oss.javaguide.cn/github/javaguide/a478c74d-2c48-40ae-9374-87aacf05188c.png)

**Protocol Serialization tương ứng với tầng nào trong mô hình 4 tầng TCP/IP?**

Chúng ta biết hai bên truyền thông mạng bắt buộc phải áp dụng và tuân thủ cùng một protocol. Mô hình 4 tầng TCP/IP như hình bên dưới, vậy protocol serialization thuộc tầng nào?

1. Tầng ứng dụng (Application Layer)
2. Tầng giao vận (Transport Layer)
3. Tầng mạng (Network Layer)
4. Tầng truy cập mạng (Network Interface Layer)

![Mô hình 4 tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

Như hình trên, trong mô hình 7 tầng OSI, công việc của tầng Trình diễn (Presentation Layer) chủ yếu là xử lý dữ liệu người dùng từ tầng ứng dụng để chuyển thành luồng nhị phân. Ngược lại là chuyển luồng nhị phân thành dữ liệu người dùng của tầng ứng dụng. Đây chính là tương ứng với serialization và deserialization đúng không?

Vì tầng Ứng dụng, tầng Trình diễn và tầng Phiên trong mô hình 7 tầng OSI tương ứng với tầng Ứng dụng trong mô hình 4 tầng TCP/IP, nên protocol serialization thuộc về một phần của tầng Ứng dụng trong protocol TCP/IP.

### Nếu có một số field không muốn serialize thì làm thế nào?

Đối với các biến không muốn serialize, sử dụng từ khóa `transient` để修饰.

Tác dụng của từ khóa `transient` là: Ngăn cản các biến trong instance được修饰 bởi từ khóa này serialize; khi đối tượng được deserialize, giá trị của biến được修饰 bởi `transient` sẽ không được lưu trữ và khôi phục.

Một số lưu ý về `transient`:

- `transient` chỉ có thể修饰 biến, không thể修饰 class và method.
- Biến được修饰 bởi `transient` sau khi deserialize sẽ được đặt về giá trị mặc định của kiểu đó. Ví dụ nếu修饰 kiểu `int` thì kết quả sau deserialize là `0`.
- Biến `static` vì không thuộc về bất kỳ đối tượng (Object) nào nên dù có từ khóa `transient` hay không thì cũng đều không được serialize.

### Các Serialization Protocol phổ biến là gì?

Cách serialization tích hợp sẵn của JDK thường không được sử dụng vì hiệu suất serialization thấp và tồn tại vấn đề an toàn bảo mật. Các serialization protocol phổ biến hơn gồm Hessian, Kryo, Protobuf, ProtoStuff, đây đều là các serialization protocol dựa trên nhị phân.

Các dạng như JSON và XML thuộc về phương thức serialization dạng văn bản (text). Mặc dù tính đọc hiểu tốt hơn nhưng hiệu năng kém hơn, thông thường không lựa chọn.

### Tại sao không khuyến nghị sử dụng Serialization tích hợp của JDK?

Chúng ta rất ít hoặc gần như không trực tiếp sử dụng cách serialization tích hợp của JDK, nguyên nhân chính gồm:

- **Không hỗ trợ gọi đa ngôn ngữ**: Không hỗ trợ khi gọi các dịch vụ phát triển bằng ngôn ngữ khác.
- **Hiệu năng kém**: So với các serialization framework khác thì hiệu năng thấp hơn, nguyên nhân chính là mảng byte sau khi serialize có kích thước lớn, làm tăng chi phí truyền tải.
- **Tồn tại vấn đề an toàn bảo mật**: Bản thân serialization và deserialization không có vấn đề. Nhưng khi dữ liệu đầu vào để deserialization có thể bị kiểm soát bởi người dùng, kẻ tấn công có thể dựng đầu vào độc hại để deserialization tạo ra đối tượng ngoài dự kiến, từ đó thực thi mã tùy ý được dựng sẵn. Đọc thêm: [An toàn ứng dụng: Lỗ hổng Java Deserialization](https://cryin.github.io/blog/secure-development-java-deserialization-vulnerability/).

## I/O

Về phân tích chi tiết I/O, vui lòng đọc các bài viết bên dưới:

- [Tóm tắt kiến thức cơ bản Java IO](https://javaguide.cn/java/io/io-basis.html)
- [Tóm tắt Design Pattern trong Java IO](https://javaguide.cn/java/io/io-design-patterns.html)
- [Giải thích chi tiết mô hình Java IO](https://javaguide.cn/java/io/io-model.html)

### Bạn hiểu gì về Java IO Stream?

IO là `Input/Output`, Đầu vào và Đầu ra. Quá trình dữ liệu nhập vào bộ nhớ máy tính là Đầu vào, ngược lại xuất ra bộ lưu trữ bên ngoài (như database, file, host từ xa) là Đầu ra. Quá trình truyền dữ liệu tương tự như dòng nước nên gọi là IO Stream. IO Stream trong Java chia thành Stream đầu vào và Stream đầu ra, và căn cứ theo cách xử lý dữ liệu lại chia thành Byte Stream và Character Stream.

Hơn 40 lớp của Java IO Stream đều được dẫn xuất từ 4 lớp cơ sở abstract sau:

- `InputStream`/`Reader`: Lớp cơ sở của tất cả Stream đầu vào, trước là Byte Input Stream, sau là Character Input Stream.
- `OutputStream`/`Writer`: Lớp cơ sở của tất cả Stream đầu ra, trước là Byte Output Stream, sau là Character Output Stream.

### Tại sao I/O Stream lại chia thành Byte Stream và Character Stream?

Bản chất câu hỏi muốn hỏi: **Dù là đọc ghi file hay gửi nhận qua mạng, đơn vị lưu trữ nhỏ nhất của thông tin đều là byte, vậy tại sao thao tác I/O Stream lại chia thành thao tác Byte Stream và Character Stream?**

Theo tôi có 2 nguyên nhân chính:

- Character Stream do JVM chuyển đổi từ byte mà ra, quá trình này khá tốn thời gian;
- Nếu chúng ta không biết kiểu encoding thì trong quá trình dùng Byte Stream rất dễ xảy ra lỗi hiển thị ký tự rác (vấn đề bể font/vỡ chữ).

### Các Design Pattern trong Java IO là gì?

Đáp án tham khảo: [Tóm tắt Design Pattern trong Java IO](https://javaguide.cn/java/io/io-design-patterns.html)

### ⭐️ Phân biệt BIO, NIO và AIO?

Đáp án tham khảo: [Giải thích chi tiết mô hình Java IO](https://javaguide.cn/java/io/io-model.html)

## Syntactic Sugar (Kẹo cú pháp)

### Syntactic Sugar là gì?

**Syntactic Sugar (Kẹo cú pháp / Cú pháp cú pháp)** đại diện cho cú pháp đặc biệt mà ngôn ngữ lập trình thiết kế nhằm thuận tiện cho lập trình viên phát triển chương trình, cú pháp này không ảnh hưởng đến tính năng của ngôn ngữ lập trình. Để thực hiện cùng một tính năng, code viết dựa trên Syntactic Sugar thường đơn giản, gọn gàng và dễ đọc hơn.

Ví dụ `for-each` trong Java là một Syntactic Sugar thường dùng, nguyên lý của nó thực chất dựa trên vòng lặp for thông thường và Iterator.

```java
String[] strs = {"JavaGuide", "Công khai: JavaGuide", "Blog: https://javaguide.cn/"};
for (String s : strs) {
    System.out.println(s);
}
```

Tuy nhiên, JVM thực chất không thể nhận diện Syntactic Sugar. Để Syntactic Sugar trong Java được thực thi chính xác, trước tiên cần thông qua trình biên dịch để "Desugar" (giải kẹo), tức là trong giai đoạn biên dịch chương trình sẽ chuyển đổi nó thành cú pháp cơ bản mà JVM hiểu được. Điều này cũng cho thấy từ góc độ khác: Thứ thực sự hỗ trợ Syntactic Sugar trong Java là trình biên dịch Java chứ không phải JVM. Nếu bạn xem mã nguồn của `com.sun.tools.javac.main.JavaCompiler`, bạn sẽ thấy trong `compile()` có một bước gọi `desugar()`, phương thức này chịu trách nhiệm thực thi việc giải Syntactic Sugar.

### Java có những Syntactic Sugar phổ biến nào?

Các Syntactic Sugar phổ biến nhất trong Java gồm Generics, Auto-boxing/unboxing, Varargs, Enum, Inner Class, Enhanced for loop, Cú pháp try-with-resources, Lambda expression,...

Về phân tích chi tiết các Syntactic Sugar này, vui lòng đọc bài viết [Giải thích chi tiết Java Syntactic Sugar](./syntactic-sugar.md).

<!-- @include: @article-footer.snippet.md -->
