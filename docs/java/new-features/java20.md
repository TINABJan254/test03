---
title: Java 20 新特性概览
description: 总结 JDK 20 的语言与并发改动，延续虚拟线程与模式匹配相关增强。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 20,JDK20,记录模式预览,虚拟线程改进,语言增强,JEP
---

JDK 20 được phát hành vào ngày 21 tháng 3 năm 2023, là phiên bản không phải Hỗ trợ dài hạn.

Phiên bản tiếp theo của nó là phiên bản LTS JDK 21 phát hành vào tháng 9 năm 2023.

JDK 20 có tổng cộng 7 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 429: Scoped Values (Giá trị phạm vi)](https://openjdk.org/jeps/429) (Lần ươm tạo thứ 1)
- [JEP 432: Record Patterns (Mẫu Record)](https://openjdk.org/jeps/432) (Xem trước lần 2)
- [JEP 433: Pattern Matching for switch (Khớp mẫu switch)](https://openjdk.org/jeps/433) (Xem trước lần 4)
- [JEP 434: Foreign Function & Memory API (Hàm ngoại và Memory API)](https://openjdk.org/jeps/434) (Xem trước lần 2)
- [JEP 436: Virtual Threads (Luồng ảo)](https://openjdk.org/jeps/436) (Xem trước lần 2)
- [JEP 437: Structured Concurrency (Đồng thời cấu trúc)](https://openjdk.org/jeps/437) (Lần ươm tạo thứ 2)
- [JEP 438: Vector API (Vector API)](https://openjdk.org/jeps/438) (Lần ươm tạo thứ 5)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 429: Scoped Values (Giá trị phạm vi, Lần ươm tạo thứ 1)

Scoped Values (Giá trị phạm vi) có thể chia sẻ dữ liệu bất biến trong thread và giữa các thread với nhau, tốt hơn biến thread local (ThreadLocal), đặc biệt là khi sử dụng số lượng lớn virtual thread.

```java
final static ScopedValue<...> V = ScopedValue.newInstance();

// In some method
ScopedValue.where(V, <value>)
           .run(() -> { ... V.get() ... call methods ... });

// In a method called directly or indirectly from the lambda expression
... V.get() ...
```

Scoped Values cho phép chia sẻ dữ liệu an toàn và hiệu quả giữa các component trong các chương trình lớn mà không cần viện đến tham số phương thức.

Về giới thiệu chi tiết của Scoped Values, khuyên đọc bài viết [Giải đáp câu hỏi thường gặp về Scoped Values](https://www.happycoders.eu/java/scoped-values/).

## JEP 432: Record Patterns (Mẫu Record, Xem trước lần 2)

Record Patterns (Mẫu Record) có thể giải cấu trúc (destructure) giá trị của record, tức là trích xuất dữ liệu từ lớp record (Record Class) một cách tiện lợi hơn. Hơn nữa, còn có thể lồng Record Patterns và Type Patterns kết hợp sử dụng, để thực hiện hình thức điều hướng và xử lý dữ liệu mang tính khai báo, có thể gộp và mạnh mẽ.

Record Patterns không thể sử dụng đơn lẻ, mà phải sử dụng cùng với khớp mẫu instanceof hoặc switch.

Trước tiên lấy instanceof làm ví dụ minh họa đơn giản.

Định nghĩa đơn giản một lớp record:

```java
record Shape(String type, long unit){}
```

Trước khi có Record Patterns:

```java
Shape circle = new Shape("Circle", 10);
if (circle instanceof Shape shape) {

  System.out.println("Area of " + shape.type() + " is : " + Math.PI * Math.pow(shape.unit(), 2));
}
```

Sau khi có Record Patterns:

```java
Shape circle = new Shape("Circle", 10);
if (circle instanceof Shape(String type, long unit)) {
  System.out.println("Area of " + type + " is : " + Math.PI * Math.pow(unit, 2));
}
```

Lại xem việc kết hợp sử dụng giữa Record Patterns và switch.

Định nghĩa một số class:

```java
interface Shape {}
record Circle(double radius) implements Shape { }
record Square(double side) implements Shape { }
record Rectangle(double length, double width) implements Shape { }
```

Trước khi có Record Patterns:

```java
Shape shape = new Circle(10);
switch (shape) {
    case Circle c:
        System.out.println("The shape is Circle with area: " + Math.PI * c.radius() * c.radius());
        break;

    case Square s:
        System.out.println("The shape is Square with area: " + s.side() * s.side());
        break;

    case Rectangle r:
        System.out.println("The shape is Rectangle with area: " + r.length() * r.width());
        break;

    default:
        System.out.println("Unknown Shape");
        break;
}
```

Sau khi có Record Patterns:

```java
Shape shape = new Circle(10);
switch(shape) {

  case Circle(double radius):
    System.out.println("The shape is Circle with area: " + Math.PI * radius * radius);
    break;

  case Square(double side):
    System.out.println("The shape is Square with area: " + side * side);
    break;

  case Rectangle(double length, double width):
    System.out.println("The shape is Rectangle with area: " + length * width);
    break;

  default:
    System.out.println("Unknown Shape");
    break;
}
```

Record Patterns có thể tránh việc chuyển đổi kiểu không cần thiết, giúp mã nguồn gọn gàng và dễ đọc hơn. Bản thân Record Patterns không loại bỏ tất cả rủi ro `null` hoặc `NullPointerException`: `null` không khớp với Record Patterns, các tham chiếu component của record vẫn có thể là `null`.

Record Patterns thực hiện xem trước lần đầu tiên trong Java 19, do [JEP 405](https://openjdk.org/jeps/405) đề xuất. Trong JDK 20 là xem trước lần thứ hai, do [JEP 432](https://openjdk.org/jeps/432) đề xuất. Cải tiến lần này bao gồm:

- Thêm hỗ trợ suy luận tham số kiểu Record Patterns chung,
- Thêm hỗ trợ Record Patterns xuất hiện trong tiêu đề của câu lệnh `for` nâng cao
- Bỏ hỗ trợ đối với Record Patterns được đặt tên (named record patterns).

**Lưu ý**: Đừng nhầm lẫn Record Patterns với lớp record (Record class) chính thức được giới thiệu trong [JDK 16](./java16.md).

## JEP 433: Pattern Matching for switch (Khớp mẫu switch, Xem trước lần 4)

Đúng như `instanceof`, `switch` cũng nối tiếp bổ sung chức năng tự động chuyển đổi khớp kiểu.

Ví dụ mã nguồn `instanceof`:

```java
// Old code
if (o instanceof String) {
    String s = (String)o;
    ... use s ...
}

// New code
if (o instanceof String s) {
    ... use s ...
}
```

Ví dụ mã nguồn `switch`:

```java
// Old code
static String formatter(Object o) {
    String formatted = "unknown";
    if (o instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (o instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (o instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (o instanceof String s) {
        formatted = String.format("String %s", s);
    }
    return formatted;
}

// New code
static String formatterPatternSwitch(Object o) {
    return switch (o) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> o.toString();
    };
}
```

Khớp mẫu `switch` đã lần lượt được xem trước trong Java 17, Java 18, Java 19, Java 20 là xem trước lần thứ 4. Mỗi lần xem trước về cơ bản đều có một vài cải tiến nhỏ, ở đây sẽ không đi sâu nữa.

## JEP 434: Foreign Function & Memory API (Hàm ngoại và Memory API, Xem trước lần 2)

Các chương trình Java có thể thông qua API này để tương tác với mã và dữ liệu bên ngoài Java runtime. Thông qua việc gọi hàm ngoại (tức mã ngoài JVM) hiệu quả và truy cập bộ nhớ ngoài (tức bộ nhớ không do JVM quản lý) an toàn, API này giúp chương trình Java có thể gọi thư viện native và xử lý dữ liệu native, chứ không nguy hiểm và dễ vỡ như JNI.

Foreign Function & Memory API thực hiện ươm tạo lần 1 trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Thực hiện ươm tạo lần 2 trong Java 18, do [JEP 419](https://openjdk.org/jeps/419) đề xuất. Xem trước lần 1 trong Java 19, do [JEP 424](https://openjdk.org/jeps/424) đề xuất.

Trong JDK 20 là xem trước lần thứ 2, do [JEP 434](https://openjdk.org/jeps/434) đề xuất, cải tiến lần này bao gồm:

- Thống nhất trừu tượng `MemorySegment` và `MemoryAddress`
- Tăng cường cấp bậc cấu trúc `MemoryLayout`
- Tách `MemorySession` thành `Arena` và `SegmentScope`, để thúc đẩy việc chia sẻ segment qua các ranh giới bảo trì.

Trong bài [Tổng quan tính năng mới Java 19](./java19.md), tôi đã giới thiệu chi tiết về Foreign Function & Memory API, ở đây không giới thiệu thêm nữa.

## JEP 436: Virtual Threads (Luồng ảo, Xem trước lần 2)

Virtual Thread (Luồng ảo) là thread nhẹ do JDK triển khai chứ không phải OS, do JDK điều phối. Nhiều virtual thread chia sẻ cùng một thread của hệ điều hành, số lượng virtual thread có thể lớn hơn nhiều so với số lượng thread của hệ điều hành.

Trước khi giới thiệu virtual thread, package `java.lang.Thread` đã hỗ trợ cái gọi là platform thread (thread nền tảng), tức là thread chúng ta vẫn luôn sử dụng trước khi chưa có virtual thread. Bộ điều phối JVM quản lý virtual thread thông qua platform thread (carrier thread - luồng gánh), một platform thread có thể thực thi các virtual thread khác nhau ở các thời điểm khác nhau (nhiều virtual thread gắn trên một platform thread), khi virtual thread bị block hoặc chờ đợi, platform thread có thể chuyển sang thực thi một virtual thread khác.

Mối quan hệ giữa Virtual Thread, Platform Thread và Kernel Thread hệ điều hành như hình dưới đây (Nguồn hình: [How to Use Java 19 Virtual Threads](https://medium.com/javarevisited/how-to-use-java-19-virtual-threads-c16a32bad5f7)):

![Mối quan hệ giữa Virtual Thread, Platform Thread và Kernel Thread](https://oss.javaguide.cn/github/javaguide/java/new-features/virtual-threads-platform-threads-kernel-threads-relationship.png)

Nói thêm một chút về mối quan hệ tương ứng giữa platform thread và kernel thread hệ điều hành: Trong các hệ điều hành chủ đạo như Windows và Linux, thread Java áp dụng mô hình thread một - một (1:1), tức là một platform thread tương ứng với một kernel thread hệ điều hành. Hệ thống Solaris là một ngoại lệ, HotSpot VM trên Solaris hỗ trợ nhiều - nhiều và một - một. Chi tiết có thể tham khảo câu trả lời của R Đại: [Mô hình thread trong JVM có phải cấp người dùng không?](https://www.zhihu.com/question/23096638/answer/29617153).

So với platform thread, virtual thread rất rẻ và nhẹ, sử dụng xong có thể hủy ngay, do đó chúng không cần phải được tái sử dụng hay gom thành pool (thread pool), mỗi tác vụ có thể có virtual thread riêng của mình để chạy. Việc tạm dừng và phục hồi virtual thread thường không cần tạo hoặc chuyển đổi một thread của hệ điều hành cho mỗi tác vụ, từ đó giảm chi phí tài nguyên thread và điều phối do số lượng lớn tác vụ bị block mang lại.

Virtual thread đã được chứng minh là rất hữu ích trong nhiều ngôn ngữ đa luồng khác, ví dụ Goroutine trong Go, Process trong Erlang.

Zhihu có một thảo luận về Virtual Thread trong Java 19, ai quan tâm có thể xem: <https://www.zhihu.com/question/536743167>.

Giải thích chi tiết và nguyên lý của Java Virtual Thread có thể xem các bài viết dưới đây:

- [Nhập môn siêu đơn giản về Virtual Thread](https://javaguide.cn/java/concurrent/virtual-thread.html)
- [Java 19 chính thức GA! Xem cách Virtual Thread nâng cao đáng kể thông lượng hệ thống](https://mp.weixin.qq.com/s/yyApBXxpXxVwttr01Hld6Q)
- [Virtual Thread - Soi mã nguồn VirtualThread](https://www.cnblogs.com/throwable/p/16758997.html)

Virtual Thread thực hiện xem trước lần đầu tiên trong Java 19, do [JEP 425](https://openjdk.org/jeps/425) đề xuất. Trong JDK 20 là xem trước lần thứ hai, có thực hiện một số thay đổi nhỏ, ở đây không đi sâu nữa.

Cuối cùng, chúng ta hãy xem 4 cách tạo virtual thread:

```java
// 1、通过 Thread.ofVirtual() 创建
Runnable fn = () -> {
  // your code here
};

Thread thread = Thread.ofVirtual()
                      .start(fn);

// 2、通过 Thread.startVirtualThread() 创建
Thread thread = Thread.startVirtualThread(() -> {
  // your code here
});

// 3、通过 Executors.newVirtualThreadPerTaskExecutor() 创建
var executorService = Executors.newVirtualThreadPerTaskExecutor();

executorService.submit(() -> {
  // your code here
});

class CustomThread implements Runnable {
  @Override
  public void run() {
    System.out.println("CustomThread run");
  }
}

//4、通过 ThreadFactory 创建
CustomThread customThread = new CustomThread();
// 获取线程工厂类
ThreadFactory factory = Thread.ofVirtual().factory();
// 创建虚拟线程
Thread thread = factory.newThread(customThread);
// 启动线程
thread.start();
```

Thông qua 4 cách tạo virtual thread được liệt kê ở trên có thể thấy, để giảm rào cản đối với virtual thread, phía chính thức đã cố gắng hết sức tái sử dụng lớp `Thread` ban đầu, nhờ đó có thể chuyển đổi mượt mà sang việc sử dụng virtual thread.

## JEP 437: Structured Concurrency (Lập trình đồng thời cấu trúc, Lần ươm tạo thứ 2)

Java 19 giới thiệu Structured Concurrency, một phương pháp lập trình đa luồng, mục đích là thông qua Structured Concurrency API để đơn giản hóa lập trình đa luồng, không phải để thay thế `java.util.concurrent`, hiện đang ở giai đoạn ươm tạo.

Structured Concurrency coi nhiều tác vụ chạy trong các thread khác nhau là một đơn vị công việc đơn lẻ, từ đó đơn giản hóa việc xử lý lỗi, nâng cao độ tin cậy và tăng cường khả năng quan sát. Nghĩa là, Structured Concurrency giữ lại tính dễ đọc, tính dễ bảo trì và tính quan sát của mã đơn luồng.

API cơ bản của Structured Concurrency là [`StructuredTaskScope`](https://download.java.net/java/early_access/loom/docs/api/jdk.incubator.concurrent/jdk/incubator/concurrent/StructuredTaskScope.html). `StructuredTaskScope` hỗ trợ chia nhỏ tác vụ thành nhiều tác vụ con đồng thời, thực thi trong các thread của chính chúng, và các tác vụ con bắt buộc phải hoàn thành trước khi tác vụ chính tiếp tục.

Cách dùng cơ bản của `StructuredTaskScope` như sau:

```java
    try (var scope = new StructuredTaskScope<Object>()) {
        // 使用fork方法派生线程来执行子任务
        Future<Integer> future1 = scope.fork(task1);
        Future<String> future2 = scope.fork(task2);
        // 等待线程完成
        scope.join();
        // 结果的处理可能包括处理或重新抛出异常
        ... process results/exceptions ...
    } // close
```

Structured Concurrency rất phù hợp với Virtual Thread, Virtual Thread là thread nhẹ do JDK triển khai. Nhiều Virtual Thread chia sẻ cùng một thread hệ điều hành, từ đó cho phép số lượng cực kỳ lớn Virtual Thread.

Thay đổi duy nhất đối với Structured Concurrency trong JDK 20 là cập nhật hỗ trợ cho các thread được tạo trong phạm vi tác vụ `StructuredTaskScope` kế thừa Scoped Values, điều này đơn giản hóa việc chia sẻ dữ liệu bất biến giữa các thread, chi tiết xem [JEP 429](https://openjdk.org/jeps/429).

## JEP 438: Vector API (Vector API, Lần ươm tạo thứ 5)

Tính toán vector được cấu thành bởi một chuỗi các thao tác trên vector. Vector API dùng để biểu thị tính toán vector, tính toán này khi runtime có thể biên dịch một cách đáng tin cậy thành các lệnh vector tối ưu trên kiến trúc CPU được hỗ trợ, từ đó thực hiện hiệu năng vượt trội hơn so với tính toán vô hướng (scalar) tương đương.

Mục tiêu của Vector API là cung cấp cho người dùng tính toán vector biểu thị phạm vi rộng rãi, ngắn gọn dễ dùng và độc lập với nền tảng.

Vector API ban đầu do [JEP 338](https://openjdk.java.net/jeps/338) đề xuất, và được tích hợp vào Java 16 dưới dạng Incubator API. Vòng ươm tạo thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và tích hợp vào Java 17, vòng ươm tạo thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và tích hợp vào Java 18, vòng thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và tích hợp vào Java 19.

Lần ươm tạo này của Java 20 về cơ bản không thay đổi Vector API, chỉ thực hiện một số sửa lỗi và tăng cường hiệu năng, chi tiết xem [JEP 438](https://openjdk.org/jeps/438).

<!-- @include: @article-footer.snippet.md -->
