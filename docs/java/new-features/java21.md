---
title: Java 21（JDK 21）新特性：虚拟线程、分代 ZGC 与有序集合
description: Java 21（JDK 21）新特性详解，涵盖虚拟线程、分代 ZGC、Sequenced Collections、记录模式、switch 模式匹配及 LTS 支持周期，并说明已撤回的字符串模板预览功能。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 21,JDK 21,JDK21新特性,Java21新特性,LTS,虚拟线程,Sequenced Collections,分代 ZGC,记录模式,switch 模式匹配,字符串模板,外部函数与内存 API
---

Java 21 (JDK 21) được phát hành chính thức vào ngày 19 tháng 9 năm 2023, là phiên bản Hỗ trợ dài hạn (LTS) được Oracle xác nhận.

Theo lộ trình hỗ trợ Java SE được Oracle cập nhật vào tháng 4 năm 2026, Premier Support của Oracle JDK 21 kéo dài tới tháng 9 năm 2028, Extended Support kéo dài tới tháng 9 năm 2031. Chu kỳ cập nhật miễn phí và hỗ trợ thương mại của các bản phân phối JDK khác nhau có thể khác nhau.

JDK 21 có tổng cộng 15 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 430: String Templates (Mẫu chuỗi)](https://openjdk.org/jeps/430) (Xem trước)
- [JEP 431: Sequenced Collections (Tập hợp có thứ tự)](https://openjdk.org/jeps/431)
- [JEP 439: Generational ZGC (ZGC phân đại)](https://openjdk.org/jeps/439)
- [JEP 440: Record Patterns (Mẫu Record)](https://openjdk.org/jeps/440)
- [JEP 441: Pattern Matching for switch (Khớp mẫu switch)](https://openjdk.org/jeps/441)
- [JEP 442: Foreign Function & Memory API (Hàm ngoại và Memory API)](https://openjdk.org/jeps/442) (Xem trước lần 3)
- [JEP 443: Unnamed Patterns and Variables (Mẫu và biến không đặt tên)](https://openjdk.org/jeps/443) (Xem trước)
- [JEP 444: Virtual Threads (Luồng ảo)](https://openjdk.org/jeps/444)
- [JEP 445: Unnamed Classes and Instance Main Methods (Class không đặt tên và phương thức main thể hiện)](https://openjdk.org/jeps/445) (Xem trước)

Nếu chủ yếu chú ý đến việc nâng cấp môi trường production, bạn có thể xem trước Sequenced Collections, Generational ZGC và Virtual Thread. String Templates chỉ được cung cấp triển khai xem trước trong JDK 21, 22, đề xuất sau đó đã bị rút lại, JDK hiện tại không còn cung cấp bộ API và cú pháp này nữa.

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 24:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 430: String Templates (Mẫu chuỗi, Xem trước)

String Templates (Mẫu chuỗi) là chức năng xem trước trong JDK 21. Chức năng này được xem trước lần 2 trong JDK 22, sau đó đã bị rút lại, do đó JDK hiện tại không còn cung cấp bộ API và cú pháp này nữa.

String Templates cung cấp một cách đơn giản hơn, trực quan hơn để xây dựng chuỗi động. Cú pháp xem trước của JDK 21 sử dụng `\{biểu thức}` làm biểu thức nhúng, và do bộ xử lý mẫu (template processor) xử lý mẫu. Biểu thức hỗ trợ biến cục bộ, trường static hoặc non-static, gọi phương thức và kết quả tính toán,...

Trên thực tế, String Templates (Mẫu chuỗi) đều tồn tại trong hầu hết các ngôn ngữ lập trình:

```typescript
"Greetings {{ name }}!";  //Angular
`Greetings ${ name }!`;    //Typescript
$"Greetings { name }!"    //Visual basic
f"Greetings { name }!"    //Python
```

Java trước khi chưa có String Templates, chúng ta thông thường sử dụng nối chuỗi hoặc phương thức định dạng để xây dựng chuỗi:

```java
//concatenation
message = "Greetings " + name + "!";

//String.format()
message = String.format("Greetings %s!", name);  //concatenation

//MessageFormat
message = new MessageFormat("Greetings {0}!").format(name);

//StringBuilder
message = new StringBuilder().append("Greetings ").append(name).append("!").toString();
```

Các phương thức này nhiều hay ít đều tồn tại một số nhược điểm, ví dụ khó đọc, rườm rà, phức tạp.

Java sử dụng String Templates để thực hiện nối chuỗi, có thể nhúng trực tiếp biểu thức trong chuỗi mà không cần thực hiện xử lý thêm:

```java
String message = STR."Greetings \{name}!";
```

Trong biểu thức mẫu ở trên:

- STR là bộ xử lý mẫu (template processor).
- `\{name}` là biểu thức, khi runtime, các biểu thức này sẽ được thay thế bởi giá trị biến tương ứng.

Java hiện tại hỗ trợ 3 loại bộ xử lý mẫu:

- STR: Tự động thực thi chèn chuỗi (string interpolation), tức là thay thế từng biểu thức nhúng trong mẫu bằng giá trị của nó (chuyển đổi thành chuỗi).
- FMT: Tương tự như STR, nhưng nó còn có thể nhận ký hiệu mô tả định dạng (format specifier), các ký hiệu mô tả định dạng này xuất hiện bên trái của biểu thức nhúng, dùng để kiểm soát kiểu dáng đầu ra.
- RAW: Không tự động xử lý mẫu chuỗi giống như bộ xử lý mẫu STR và FMT, mà trả về một đối tượng `StringTemplate`, đối tượng này chứa thông tin văn bản và biểu thức trong mẫu.

```java
String name = "Lokesh";

//STR
String message = STR."Greetings \{name}.";

//FMT
String message = FMT."Greetings %-12s\{name}.";

//RAW
StringTemplate st = RAW."Greetings \{name}.";
String message = STR.process(st);
```

Ngoài 3 bộ xử lý mẫu có sẵn của JDK, bạn còn có thể triển khai interface `StringTemplate.Processor` để tạo bộ xử lý mẫu của riêng mình, chỉ cần kế thừa interface `StringTemplate.Processor`, sau đó triển khai phương thức `process` là được.

Chúng ta có thể sử dụng biến cục bộ, trường static/non-static thậm chí cả phương thức làm biểu thức nhúng:

```java
//variable
message = STR."Greetings \{name}!";

//method
message = STR."Greetings \{getName()}!";

//field
message = STR."Greetings \{this.name}!";
```

Còn có thể thực thi tính toán trong biểu thức và in kết quả:

```java
int x = 10, y = 20;
String s = STR."\{x} + \{y} = \{x + y}";  //"10 + 20 = 30"
```

Để nâng cao tính dễ đọc, chúng ta có thể chia biểu thức nhúng thành nhiều dòng:

```java
String time = STR."The current time is \{
    //sample comment - current time in HH:mm:ss
    DateTimeFormatter
      .ofPattern("HH:mm:ss")
      .format(LocalTime.now())
  }.";
```

## JEP 431: Sequenced Collections (Tập hợp có thứ tự)

JDK 21 giới thiệu một nhóm các interface tập hợp mới: **Sequenced Collections (Tập hợp có thứ tự)**. Loại tập hợp này có thứ tự duyệt xác định (encounter order), và cung cấp các phương thức truy cập phần tử đầu cuối của tập hợp cũng như lấy view đảo ngược (reversed view).

Sequenced Collections bao gồm 3 interface dưới đây:

- [`SequencedCollection`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/SequencedCollection.html)
- [`SequencedSet`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/SequencedSet.html)
- [`SequencedMap`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/SequencedMap.html)

Interface `SequencedCollection` kế thừa interface `Collection`, cung cấp các phương thức truy cập, thêm hoặc xóa phần tử ở hai đầu tập hợp cũng như lấy view đảo ngược của tập hợp.

```java
interface SequencedCollection<E> extends Collection<E> {

  // New Method

  SequencedCollection<E> reversed();

  // Promoted methods from Deque<E>

  void addFirst(E);
  void addLast(E);

  E getFirst();
  E getLast();

  E removeFirst();
  E removeLast();
}
```

Các interface `List` và `Deque` kế thừa interface `SequencedCollection`.

Ở đây lấy `ArrayList` làm ví dụ, minh họa hiệu quả sử dụng thực tế:

```java
ArrayList<Integer> arrayList = new ArrayList<>();

arrayList.add(1);   // List contains: [1]

arrayList.addFirst(0);  // List contains: [0, 1]
arrayList.addLast(2);   // List contains: [0, 1, 2]

Integer firstElement = arrayList.getFirst();  // 0
Integer lastElement = arrayList.getLast();  // 2

List<Integer> reversed = arrayList.reversed();
System.out.println(reversed); // Prints [2, 1, 0]
```

Interface `SequencedSet` kế thừa trực tiếp interface `SequencedCollection` và ghi đè phương thức `reversed()`.

```java
interface SequencedSet<E> extends SequencedCollection<E>, Set<E> {

    SequencedSet<E> reversed();
}
```

Interface `SortedSet` kế thừa interface `SequencedSet`, `LinkedHashSet` triển khai interface `SequencedSet`.

Ở đây lấy `LinkedHashSet` làm ví dụ, minh họa hiệu quả sử dụng thực tế:

```java
LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>(List.of(1, 2, 3));

Integer firstElement = linkedHashSet.getFirst();   // 1
Integer lastElement = linkedHashSet.getLast();    // 3

linkedHashSet.addFirst(0);  //List contains: [0, 1, 2, 3]
linkedHashSet.addLast(4);   //List contains: [0, 1, 2, 3, 4]

System.out.println(linkedHashSet.reversed());   //Prints [4, 3, 2, 1, 0]
```

Interface `SequencedMap` kế thừa interface `Map`, cung cấp các phương thức truy cập, thêm hoặc xóa cặp key-value ở hai đầu tập hợp, lấy `SequencedSet` chứa key, `SequencedCollection` chứa value, `SequencedSet` chứa entry (cặp key-value) cũng như lấy view đảo ngược của tập hợp.

```java
interface SequencedMap<K,V> extends Map<K,V> {

  // New Methods

  SequencedMap<K,V> reversed();

  SequencedSet<K> sequencedKeySet();
  SequencedCollection<V> sequencedValues();
  SequencedSet<Entry<K,V>> sequencedEntrySet();

  V putFirst(K, V);
  V putLast(K, V);


  // Promoted Methods from NavigableMap<K, V>

  Entry<K, V> firstEntry();
  Entry<K, V> lastEntry();

  Entry<K, V> pollFirstEntry();
  Entry<K, V> pollLastEntry();
}
```

Interface `SortedMap` kế thừa interface `SequencedMap`, `LinkedHashMap` triển khai interface `SequencedMap`.

Ở đây lấy `LinkedHashMap` làm ví dụ, minh họa hiệu quả sử dụng thực tế:

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>();

map.put(1, "One");
map.put(2, "Two");
map.put(3, "Three");

map.firstEntry();   //1=One
map.lastEntry();    //3=Three

System.out.println(map);  //{1=One, 2=Two, 3=Three}

Map.Entry<Integer, String> first = map.pollFirstEntry();   //1=One
Map.Entry<Integer, String> last = map.pollLastEntry();    //3=Three

System.out.println(map);  //{2=Two}

map.putFirst(1, "One");     //{1=One, 2=Two}
map.putLast(3, "Three");    //{1=One, 2=Two, 3=Three}

System.out.println(map);  //{1=One, 2=Two, 3=Three}
System.out.println(map.reversed());   //{3=Three, 2=Two, 1=One}
```

## JEP 439: Generational ZGC (ZGC phân đại)

Trong JDK 21 đã mở rộng chức năng cho ZGC, bổ sung tính năng phân đại GC. Tuy nhiên, mặc định là tắt, cần phải mở thông qua cấu hình:

```bash
// Bật Generational ZGC
java -XX:+UseZGC -XX:+ZGenerational ...
```

Trong các phiên bản tương lai, phía chính thức sẽ thiết lập ZGenerational làm giá trị mặc định, tức mặc định bật phân đại GC của ZGC. Ở phiên bản muộn hơn nữa, ZGC không phân đại sẽ bị loại bỏ.

> In a future release we intend to make Generational ZGC the default, at which point -XX:-ZGenerational will select non-generational ZGC. In an even later release we intend to remove non-generational ZGC, at which point the ZGenerational option will become obsolete.

ZGC phân đại trong khi duy trì mục tiêu tạm dừng thấp của ZGC, chủ yếu thông qua việc thu hồi các đối tượng trẻ thường xuyên hơn để giảm rủi ro tạm dừng phân bổ, giảm bộ nhớ heap cần thiết và nâng cao thông lượng.

## JEP 440: Record Patterns (Mẫu Record)

Record Patterns thực hiện xem trước lần đầu tiên trong Java 19, do [JEP 405](https://openjdk.org/jeps/405) đề xuất. Trong JDK 20 là xem trước lần thứ hai, do [JEP 432](https://openjdk.org/jeps/432) đề xuất. Cuối cùng, Record Patterns đã chuyển thành chính thức một cách thuận lợi trong JDK 21.

Bài [Tổng quan tính năng mới Java 20](./java20.md) đã giới thiệu chi tiết về Record Patterns, ở đây không lặp lại nữa.

## JEP 441: Pattern Matching for switch (Khớp mẫu cho switch)

Tăng cường biểu thức và câu lệnh switch trong Java, cho phép sử dụng pattern trong thẻ case. Khi pattern khớp, thực thi mã nguồn tương ứng với thẻ case.

Trong mã nguồn dưới đây, biểu thức switch sử dụng pattern kiểu để tiến hành khớp.

```java
static String formatterPatternSwitch(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> obj.toString();
    };
}
```

## JEP 442: Foreign Function & Memory API (Hàm ngoại và Memory API, Xem trước lần 3)

Các chương trình Java có thể thông qua API này để tương tác với mã và dữ liệu bên ngoài Java runtime. Thông qua việc gọi hàm ngoại (tức mã ngoài JVM) hiệu quả và truy cập bộ nhớ ngoài (tức bộ nhớ không do JVM quản lý) an toàn, API này giúp chương trình Java có thể gọi thư viện native và xử lý dữ liệu native, chứ không nguy hiểm và dễ vỡ như JNI.

Foreign Function & Memory API thực hiện ươm tạo lần 1 trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Thực hiện ươm tạo lần 2 trong Java 18, do [JEP 419](https://openjdk.org/jeps/419) đề xuất. Xem trước lần 1 trong Java 19, do [JEP 424](https://openjdk.org/jeps/424) đề xuất. Xem trước lần 2 trong JDK 20, do [JEP 434](https://openjdk.org/jeps/434) đề xuất. Trong JDK 21 là xem trước lần 3, do [JEP 442](https://openjdk.org/jeps/442) đề xuất.

Trong bài [Tổng quan tính năng mới Java 19](./java19.md), tôi đã giới thiệu chi tiết về Foreign Function & Memory API, ở đây không giới thiệu thêm nữa.

## JEP 443: Unnamed Patterns and Variables (Mẫu và biến không đặt tên, Xem trước)

Unnamed Patterns and Variables giúp chúng ta có thể sử dụng dấu gạch dưới `_` để biểu thị các biến không đặt tên cũng như các component không sử dụng khi khớp mẫu, nhằm nâng cao tính dễ đọc và tính dễ bảo trì của mã nguồn.

Kịch bản điển hình của biến không đặt tên là câu lệnh `try-with-resources`, biến ngoại lệ trong mệnh đề `catch` và vòng lặp `for`. Khi biến không cần sử dụng thì có thể dùng dấu gạch dưới `_` thay thế, nhờ đó đánh dấu rõ ràng biến chưa được sử dụng.

```java
try (var _ = ScopedContext.acquire()) {
  // No use of acquired resource
}
try { ... }
catch (Exception _) { ... }
catch (Throwable _) { ... }

for (int i = 0, _ = runOnce(); i < arr.length; i++) {
  ...
}
```

Unnamed pattern (Mẫu không đặt tên) là một pattern vô điều kiện, không ràng buộc bất kỳ giá trị nào. Biến mẫu không đặt tên xuất hiện trong pattern kiểu.

```java
if (r instanceof ColoredPoint(_, Color c)) { ... c ... }

switch (b) {
    case Box(RedBall _), Box(BlueBall _) -> processBox(b);
    case Box(GreenBall _)                -> stopProcessing();
    case Box(_)                          -> pickAnotherBox();
}
```

## JEP 444: Virtual Threads (Luồng ảo)

Virtual Thread là một cập nhật vô cùng quan trọng, nhất định phải coi trọng!

Virtual Thread thực hiện xem trước lần đầu tiên trong Java 19, do [JEP 425](https://openjdk.org/jeps/425) đề xuất. Trong JDK 20 là xem trước lần thứ hai. Cuối cùng, Virtual Thread đã chính thức trở thành tính năng chuẩn trong JDK 21.

Bài [Tổng quan tính năng mới Java 20](./java20.md) đã giới thiệu chi tiết về Virtual Thread, ở đây không lặp lại nữa.

## JEP 445: Unnamed Classes and Instance Main Methods (Class không đặt tên và phương thức main thể hiện, Xem trước)

Tính năng này chủ yếu đơn giản hóa việc khai báo phương thức `main`. Đối với người mới bắt đầu học Java, việc khai báo phương thức `main` này giới thiệu quá nhiều khái niệm cú pháp Java, không có lợi cho người mới bắt đầu tiếp cận nhanh chóng.

Định nghĩa một phương thức `main` khi chưa sử dụng tính năng này:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

Định nghĩa một phương thức `main` sau khi sử dụng tính năng mới này:

```java
class HelloWorld {
    void main() {
        System.out.println("Hello, World!");
    }
}
```

Rút gọn thêm một bước (Class không đặt tên cho phép chúng ta không định nghĩa tên class):

```java
void main() {
   System.out.println("Hello, World!");
}
```

## Tham khảo

- Java 21 String Templates: <https://howtodoinjava.com/java/java-string-templates/>
- Java 21 Sequenced Collections: <https://howtodoinjava.com/java/sequenced-collections/>
