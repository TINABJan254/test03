---
title: Java 25 新特性概览
description: 概览 JDK 25 的关键新特性与预览改动，关注并发、GC 与语言/平台增强。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 25,JDK25,LTS,作用域值,紧凑对象头,分代 Shenandoah,模块导入,结构化并发
---

JDK 25 được phát hành vào ngày 16 tháng 9 năm 2025, đây là một phiên bản vô cùng quan trọng, mang tính cột mốc.

JDK 25 được Oracle và hầu hết các nhà phân phối JDK xếp vào loại LTS (Hỗ trợ dài hạn). Các phiên bản LTS hiện tại của Oracle bao gồm JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

JDK 25 có tổng cộng 18 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 506: Scoped Values (Giá trị phạm vi)](https://openjdk.org/jeps/506)
- [JEP 512: Compact Source Files and Instance Main Methods (File nguồn nhỏ gọn và phương thức main thể hiện)](https://openjdk.org/jeps/512)
- [JEP 519: Compact Object Headers (Header đối tượng nhỏ gọn)](https://openjdk.org/jeps/519)
- [JEP 521: Generational Shenandoah (Shenandoah GC phân đại)](https://openjdk.org/jeps/521)
- [JEP 507: Primitive Types in Patterns, instanceof, and switch (Khớp mẫu hỗ trợ kiểu nguyên thủy, Xem trước lần 3)](https://openjdk.org/jeps/507)
- [JEP 505: Structured Concurrency (Đồng thời cấu trúc, Xem trước lần 5)](https://openjdk.org/jeps/505)
- [JEP 511: Module Import Declarations (Khai báo import module)](https://openjdk.org/jeps/511)
- [JEP 513: Flexible Constructor Bodies (Thân constructor linh hoạt)](https://openjdk.org/jeps/513)
- [JEP 508: Vector API (Vector API, Lần ươm tạo thứ 10)](https://openjdk.org/jeps/508)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JDK 25

### JEP 506: Giá trị phạm vi (Scoped Values)

Scoped Values (Giá trị phạm vi) có thể chia sẻ dữ liệu bất biến trong thread và giữa các thread với nhau, tốt hơn biến thread local `ThreadLocal`, đặc biệt là khi sử dụng số lượng lớn virtual thread.

```java
final static ScopedValue<...> V = ScopedValue.newInstance();

// In some method
ScopedValue.where(V, <value>)
           .run(() -> { ... V.get() ... call methods ... });

// In a method called directly or indirectly from the lambda expression
... V.get() ...
```

Scoped Values cung cấp truyền dữ liệu bất biến một chiều từ caller sang callee. Các thread con được tạo thông qua Structured Concurrency có thể kế thừa binding trong thread cha, hơn nữa việc kế thừa này không cần sao chép binding, từ đó giảm chi phí thời gian và không gian khi số lượng lớn thread chia sẻ context.

### JEP 512: File nguồn nhỏ gọn và phương thức main thể hiện

Lần xem trước đầu tiên của tính năng này do [JEP 445](https://openjdk.org/jeps/445 "JEP 445") (JDK 21) đề xuất, sau đó qua các cải tiến và hoàn thiện của JDK 22, JDK 23 và JDK 24, cuối cùng đã thuận lợi chuyển thành chính thức trong JDK 25.

Cải tiến này đơn giản hóa cực kỳ các bước viết chương trình Java đơn giản, cho phép viết class và phương thức main trong cùng một file không có `public class` cấp cao nhất, và cho phép phương thức `main` trở thành một phương thức thể hiện non-static.

```java
class HelloWorld {
    void main() {
        System.out.println("Hello, World!");
    }
}
```

Đơn giản hóa thêm một bước:

```java
void main() {
    System.out.println("Hello, World!");
}
```

Đây là một bước tiến lớn nhằm hạ thấp rào cản học Java và nâng cao hiệu quả viết các chương trình nhỏ, script. Người mới bắt đầu không còn cần phải hiểu chuỗi khai báo phức tạp dài ngoẵng `public static void main(String[] args)`. Đối với việc xác thực nguyên mẫu nhanh và viết script, điều này cũng giúp Java trở thành một lựa chọn hấp dẫn hơn.

### JEP 519: Header đối tượng nhỏ gọn (Compact Object Headers)

Tính năng này được giới thiệu dưới dạng tính năng thử nghiệm trong [JEP 450](https://openjdk.org/jeps/450 "JEP 450") (JDK 24), JDK 25 chuyển thành tính năng chính thức.

Thông qua tối ưu hóa cấu trúc nội bộ của object header, trong HotSpot virtual machine kiến trúc 64-bit, kích thước object header được thu nhỏ từ 96-128 bit (12-16 byte) ban đầu xuống còn 64 bit (8 byte), cuối cùng đạt được hiệu quả giảm chiếm dụng bộ nhớ heap, nâng cao mật độ triển khai, tăng cường tính cục bộ của dữ liệu.

Header đối tượng nhỏ gọn không trở thành bố cục object header mặc định của JVM, cần phải bật thông qua cấu hình hiển thị:

- JDK 24 cần bật thông qua kết hợp tham số dòng lệnh:
  `$ java -XX:+UnlockExperimentalVMOptions -XX:+UseCompactObjectHeaders ...`;
- Từ JDK 25 trở đi chỉ cần `-XX:+UseCompactObjectHeaders` là có thể bật.

### JEP 521: Shenandoah GC phân đại (Generational Shenandoah GC)

Shenandoah GC được giới thiệu dưới dạng tính năng thử nghiệm trong JDK 12, và chuyển thành tính năng sản phẩm chính thức trong JDK 15. Nó mặc định tắt, có thể bật thông qua `-XX:+UseShenandoahGC`.

Shenandoah là bộ thu gom rác tạm dừng thấp do Red Hat chủ đạo phát triển, mục tiêu là khi heap tương đối lớn vẫn cố gắng duy trì thời gian tạm dừng ngắn và tương đối độc lập với kích thước heap.

Shenandoah truyền thống tiến hành đánh dấu và dọn dẹp đồng thời trên toàn bộ heap, mặc dù thời gian tạm dừng cực kỳ ngắn, nhưng hiệu quả khi xử lý đối tượng thế hệ trẻ không bằng GC phân đại. Sau khi giới thiệu phân đại, Shenandoah có thể thu hồi thường xuyên hơn, hiệu quả hơn số lượng lớn đối tượng "sinh nhanh chết nhanh" trong thế hệ trẻ, giúp nó trong khi duy trì thời gian tạm dừng cực thấp, đồng thời sở hữu thông lượng cao hơn và chi phí CPU thấp hơn.

Shenandoah GC cần bật thông qua câu lệnh:

- JDK 24 cần bật thông qua kết hợp tham số dòng lệnh: `-XX:+UseShenandoahGC -XX:+UnlockExperimentalVMOptions -XX:ShenandoahGCMode=generational`
- Từ JDK 25 trở đi chỉ cần `-XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational` là có thể bật.

### JEP 507: Khớp mẫu hỗ trợ kiểu nguyên thủy (Xem trước lần 3)

Lần xem trước đầu tiên của tính năng này do [JEP 455](https://openjdk.org/jeps/455 "JEP 455") (JDK 23) đề xuất.

Khớp mẫu có thể xử lý tất cả các kiểu dữ liệu nguyên thủy (`int`, `double`, `boolean`,...) trong các câu lệnh `switch` và `instanceof`.

```java
static void test(Object obj) {
    if (obj instanceof int i) {
        System.out.println("这是一个int类型: " + i);
    }
}
```

Như vậy có thể giống như xử lý kiểu đối tượng, tiến hành khớp kiểu và chuyển đổi an toàn hơn, ngắn gọn hơn đối với kiểu nguyên thủy, loại bỏ thêm mã boilerplate trong Java.

### JEP 505: Structured Concurrency (Đồng thời cấu trúc, Xem trước lần 5)

JDK 19 đã giới thiệu Structured Concurrency dưới dạng Incubator API. Trong JDK 25, API này đang ở giai đoạn xem trước lần thứ 5, mục đích là đơn giản hóa lập trình đa luồng, không phải để thay thế `java.util.concurrent`.

Structured Concurrency coi nhiều tác vụ chạy trong các thread khác nhau là một đơn vị công việc đơn lẻ, từ đó đơn giản hóa việc xử lý lỗi, nâng cao độ tin cậy và tăng cường khả năng quan sát. Nghĩa là, Structured Concurrency giữ lại tính dễ đọc, tính dễ bảo trì và tính quan sát của mã đơn luồng.

API cơ bản của Structured Concurrency là `StructuredTaskScope`, nó hỗ trợ chia nhỏ tác vụ thành nhiều tác vụ con đồng thời, thực thi trong các thread của chính chúng, và các tác vụ con bắt buộc phải hoàn thành trước khi tác vụ chính tiếp tục.

Cách dùng cơ bản của `StructuredTaskScope` như sau:

```java
    try (var scope = StructuredTaskScope.open()) {
        // 使用fork方法派生线程来执行子任务
        Subtask<Integer> subtask1 = scope.fork(task1);
        Subtask<String> subtask2 = scope.fork(task2);
        // 等待线程完成
        scope.join();
        // 结果的处理可能包括处理或重新抛出异常
        ... process results/exceptions ...
    } // close
```

Structured Concurrency rất phù hợp với Virtual Thread, Virtual Thread là thread nhẹ do JDK triển khai. Nhiều Virtual Thread chia sẻ cùng một thread hệ điều hành, từ đó cho phép số lượng cực kỳ lớn Virtual Thread.

### JEP 511: Khai báo import module (Module Import Declarations)

Lần xem trước đầu tiên của tính năng này do [JEP 476](https://openjdk.org/jeps/476 "JEP 476") (JDK 23) đề xuất, sau đó được hoàn thiện trong [JEP 494](https://openjdk.org/jeps/494 "JEP 494") (JDK 24), JDK 25 thuận lợi chuyển thành chính thức.

Khai báo import module cho phép import ngắn gọn tất cả các package được xuất của toàn bộ module trong mã Java, mà không cần khai báo import từng package một. Tính năng này đơn giản hóa việc tái sử dụng thư viện module hóa, đặc biệt là khi sử dụng nhiều module, tránh được lượng lớn khai báo import package, giúp nhà phát triển truy cập các thư viện bên thứ ba và class cơ bản của Java tiện lợi hơn.

Tính năng này đặc biệt có ích cho người mới bắt đầu và phát triển nguyên mẫu, vì nó không yêu cầu nhà phát triển phải module hóa mã nguồn của chính mình, đồng thời giữ lại tính tương thích với phương thức import truyền thống, nâng cao hiệu suất phát triển và tính dễ đọc của mã nguồn.

```java
// 导入整个 java.base 模块，开发者可以直接访问 List、Map、Stream 等类，而无需每次手动导入相关包
import module java.base;

public class Example {
    public static void main(String[] args) {
        String[] fruits = { "apple", "berry", "citrus" };
        Map<String, String> fruitMap = Stream.of(fruits)
            .collect(Collectors.toMap(
                s -> s.toUpperCase().substring(0, 1),
                Function.identity()));

        System.out.println(fruitMap);
    }
}
```

### JEP 513: Thân constructor linh hoạt (Flexible Constructor Bodies)

Lần xem trước đầu tiên của tính năng này do [JEP 447](https://openjdk.org/jeps/447 "JEP 447") (JDK 22) đề xuất, sau đó trải qua xem trước trong [JEP 482 ](https://openjdk.org/jeps/482 "JEP 482 ") (JDK 23) và [JEP 492](https://openjdk.org/jeps/492 "JEP 492") (JDK 24), JDK 25 thuận lợi chuyển thành chính thức.

Java yêu cầu trong constructor, việc gọi `super(...)` hoặc `this(...)` bắt buộc phải xuất hiện làm câu lệnh đầu tiên. Điều này có nghĩa là chúng ta không thể trực tiếp khởi tạo các trường trong constructor của class con trước khi gọi constructor của class cha.

Flexible Constructor Bodies giải quyết vấn đề này, nó cho phép trong thân constructor, viết các câu lệnh trước khi gọi `super(..)` hoặc `this(..)`, các câu lệnh này có thể khởi tạo trường, nhưng không thể tham chiếu đến thể hiện đang được khởi tạo. Điều này ngăn ngừa việc các trường của class con chưa được khởi tạo đúng cách khi gọi phương thức class con trong constructor class cha, tăng cường độ tin cậy của việc khởi tạo class.

Tính năng này giải quyết hạn chế cú pháp Java trước đây đối với việc tổ chức mã constructor, cho phép nhà phát triển diễn đạt hành vi của constructor tự nhiên hơn, tự do hơn, ví dụ tiến hành kiểm tra hợp lệ tham số, chuẩn bị và chia sẻ trực tiếp trong constructor mà không cần phụ thuộc vào phương thức phụ trợ hay constructor, nâng cao tính dễ đọc và tính dễ bảo trì của mã nguồn.

```java
class Person {
    private final String name;
    private int age;

    public Person(String name, int age) {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative.");
        }
        this.name = name; // 在调用父类构造函数之前初始化字段
        this.age = age;
        // ... 其他初始化代码
    }
}

class Employee extends Person {
    private final int employeeId;

    public Employee(String name, int age, int employeeId) {
        this.employeeId = employeeId; // 在调用父类构造函数之前初始化字段
        super(name, age); // 调用父类构造函数
        // ... 其他初始化代码
    }
}
```

### JEP 508: Vector API (Vector API, Lần ươm tạo thứ 10)

Tính toán vector được cấu thành bởi một chuỗi các thao tác trên vector. Vector API dùng để biểu thị tính toán vector, tính toán này khi runtime có thể biên dịch một cách đáng tin cậy thành các lệnh vector tối ưu trên kiến trúc CPU được hỗ trợ, từ đó thực hiện hiệu năng vượt trội hơn so với tính toán vô hướng (scalar) tương đương.

Mục tiêu của Vector API là cung cấp cho người dùng tính toán vector biểu thị phạm vi rộng rãi, ngắn gọn dễ dùng và độc lập với nền tảng.

Đây là tính toán vô hướng đơn giản trên các phần tử mảng:

```java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}
```

Đây là tính toán vector tương đương sử dụng Vector API:

```java
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_PREFERRED;

void vectorComputation(float[] a, float[] b, float[] c) {
    int i = 0;
    int upperBound = SPECIES.loopBound(a.length);
    for (; i < upperBound; i += SPECIES.length()) {
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i);
        var vb = FloatVector.fromArray(SPECIES, b, i);
        var vc = va.mul(va)
                   .add(vb.mul(vb))
                   .neg();
        vc.intoArray(c, i);
    }
    for (; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}
```

Mặc dù vẫn đang trong quá trình ươm tạo, nhưng lặp lại lần thứ 10 đủ để chứng minh tầm quan trọng của nó. Nó giúp Java trong các lĩnh vực nhạy cảm về hiệu năng như tính toán khoa học, machine learning, xử lý dữ liệu lớn, có thể viết ra mã nguồn có hiệu năng tiệm cận thậm chí sánh ngang với các ngôn ngữ native như C++. Đây là mấu chốt để Java duy trì tính cạnh tranh trong lĩnh vực tính toán hiệu năng cao.
