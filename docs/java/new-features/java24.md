---
title: Java 24 新特性概览
description: 总结 JDK 24 的新特性与改动，便于跟踪 Java 演进。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 24,JDK24,JEP 更新,语言特性,GC 改进,平台增强
---

JDK 24 được phát hành vào tháng 3 năm 2025, đây là một phiên bản không phải LTS (Hỗ trợ dài hạn). Phiên bản tiếp theo của nó là phiên bản LTS **JDK 25** phát hành vào tháng 9 năm 2025.

JDK 24 có tổng cộng 24 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 478: Key Derivation Function API (API Hàm phái sinh khóa, Xem trước)](https://openjdk.org/jeps/478)
- [JEP 483: Early Class-File Loading & Linking (Tải và liên kết class file sớm)](https://openjdk.org/jeps/483)
- [JEP 484: Class File API (Class File API)](https://openjdk.org/jeps/484)
- [JEP 485: Stream Gatherers (Trình thu thập stream)](https://openjdk.org/jeps/485)
- [JEP 486: Disable the Security Manager (Vô hiệu hóa vĩnh viễn Security Manager)](https://openjdk.org/jeps/486)
- [JEP 487: Scoped Values (Giá trị phạm vi, Xem trước lần 4)](https://openjdk.org/jeps/487)
- [JEP 495: Simplified Source Files and Instance Main Methods (File nguồn và phương thức main thể hiện đơn giản hóa, Xem trước lần 4)](https://openjdk.org/jeps/495)
- [JEP 497: Quantum-Resistant Digital Signature Algorithm (ML-DSA) (Thuật toán chữ ký số kháng lượng tử)](https://openjdk.org/jeps/497)
- [JEP 499: Structured Concurrency (Đồng thời cấu trúc, Xem trước lần 4)](https://openjdk.org/jeps/499)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 478: Key Derivation Function API（API Hàm phái sinh khóa，Xem trước）

API Hàm phái sinh khóa cung cấp một tập hợp các interface chuẩn dùng để phái sinh thêm các khóa từ khóa ban đầu và các dữ liệu khác. Nó có thể sinh ra nhiều khóa khác nhau cho các mục đích mã hóa khác nhau (như mã hóa, xác thực,...), tránh việc trực tiếp tái sử dụng cùng một khóa. API này đang ở giai đoạn xem trước trong JDK 24.

Thông qua API này, các nhà phát triển có thể sử dụng các thuật toán phái sinh khóa như HKDF do các provider an toàn triển khai:

```java
// 创建一个 KDF 对象，使用 HKDF-SHA256 算法
KDF hkdf = KDF.getInstance("HKDF-SHA256");

// 创建 Extract 和 Expand 参数规范
AlgorithmParameterSpec params =
    HKDFParameterSpec.ofExtract()
                     .addIKM(initialKeyMaterial) // 设置初始密钥材料
                     .addSalt(salt)             // 设置盐值
                     .thenExpand(info, 32);     // 设置扩展信息和目标长度

// 派生一个 32 字节的 AES 密钥
SecretKey key = hkdf.deriveKey("AES", params);

// 可以使用相同的 KDF 对象进行其他密钥派生操作
```

## JEP 483: Early Class-File Loading & Linking（Tải và liên kết class file sớm）

Trong JVM truyền thống, ứng dụng cần tải và liên kết class động ở mỗi lần khởi động. Tính năng này thông qua việc lưu đệm các class đã được tải và liên kết, giảm bớt các công việc lặp lại trong các lần khởi động sau. Trong Spring PetClinic benchmark do JEP 483 cung cấp, sau khi sử dụng bộ đệm AOT, thời gian khởi động rút ngắn tối đa khoảng 42%; lợi ích thực tế phụ thuộc vào ứng dụng và môi trường chạy.

Tối ưu hóa này không yêu cầu sửa đổi ứng dụng, thư viện hay mã framework, nhưng cần ghi lại lượt chạy huấn luyện và tạo bộ đệm AOT trước, sau đó tải bộ đệm đó khi khởi động chính thức. Các tham số liên quan bao gồm `-XX:AOTMode=record`, `-XX:AOTConfiguration`, `-XX:AOTMode=create` và `-XX:AOTCache`.

## JEP 484: Class File API（Class File API）

Class File API thực hiện xem trước lần đầu tiên trong JDK 22 ([JEP 457](https://openjdk.org/jeps/457)), thực hiện xem trước lần 2 và hoàn thiện hơn trong JDK 23 ([JEP 466](https://openjdk.org/jeps/466)). Cuối cùng, tính năng này đã chính thức trở thành tính năng chuẩn trong JDK 24.

Mục tiêu của Class File API là cung cấp một tập hợp API chuẩn hóa, dùng để phân tích, sinh ra và chuyển đổi các class file Java, thay thế cho sự phụ thuộc trước đây vào các thư viện bên thứ ba (như ASM) khi xử lý class file.

```java
// 创建一个 ClassFile 对象，这是操作类文件的入口。
ClassFile cf = ClassFile.of();
// 解析字节数组为 ClassModel
ClassModel classModel = cf.parse(bytes);

// 构建新的类文件，移除以 "debug" 开头的所有方法
byte[] newBytes = cf.build(classModel.thisClass().asSymbol(),
        classBuilder -> {
            // 遍历所有类元素
            for (ClassElement ce : classModel) {
                // 判断是否为方法 且 方法名以 "debug" 开头
                if (!(ce instanceof MethodModel mm
                        && mm.methodName().stringValue().startsWith("debug"))) {
                    // 添加到新的类文件中
                    classBuilder.with(ce);
                }
            }
        });
```

## JEP 485: Stream Gatherers（Trình thu thập stream）

`Stream::gather(Gatherer)` là một tính năng mới mạnh mẽ, nó cho phép các nhà phát triển định nghĩa các thao tác trung gian tùy chỉnh, từ đó thực hiện chuyển đổi dữ liệu phức tạp và linh hoạt hơn. Interface `Gatherer` là cốt lõi của tính năng này, nó định nghĩa cách thu thập các phần tử từ stream, duy trì trạng thái trung gian, và tạo ra kết quả trong quá trình xử lý.

Khác với các thao tác dựng sẵn hiện có như `filter`, `map` hay `distinct`, `Stream::gather` giúp các nhà phát triển có thể thực hiện những tác vụ khó hoàn thành bằng các thao tác Stream chuẩn. Ví dụ, có thể sử dụng `Stream::gather` để thực hiện cửa sổ trượt (sliding window), loại bỏ trùng lặp theo quy tắc tùy chỉnh, hoặc chuyển đổi và gom tụ trạng thái phức tạp hơn. Sự linh hoạt này mở rộng đáng kể phạm vi ứng dụng của Stream API, giúp nhà phát triển đối phó với các kịch bản xử lý dữ liệu phức tạp hơn.

Triển khai logic loại bỏ trùng lặp dựa trên độ dài chuỗi bằng `Stream::gather(Gatherer)`:

```java
var result = Stream.of("foo", "bar", "baz", "quux")
                   .gather(Gatherer.ofSequential(
                       HashSet::new, // 初始化状态为 HashSet,用于保存已经遇到过的字符串长度
                       (set, str, downstream) -> {
                           if (set.add(str.length())) {
                               return downstream.push(str);
                           }
                           return true; // 继续处理流
                       }
                   ))
                   .toList();// 转换为列表

// 输出结果 ==> [foo, quux]
```

## JEP 486: Disable the Security Manager（Vô hiệu hóa vĩnh viễn Security Manager）

JDK 24 không còn cho phép bật `Security Manager`, ngay cả khi thông qua câu lệnh `java -Djava.security.manager` cũng không thể bật, đây là bước đi then chốt để loại bỏ dần chức năng này. Mặc dù `Security Manager` từng là một công cụ quan trọng trong Java để hạn chế quyền hạn của mã nguồn (như truy cập hệ thống file hoặc mạng, đọc hoặc ghi các file nhạy cảm, thực thi câu lệnh hệ thống), nhưng do tính phức tạp cao, tỷ lệ sử dụng thấp và chi phí bảo trì lớn, cộng đồng Java đã quyết định cuối cùng sẽ xóa bỏ nó.

## JEP 487: Scoped Values（Giá trị phạm vi, Xem trước lần 4）

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

## JEP 491: Virtual Threads Synchronization Without Pinning（Đồng bộ luồng ảo mà không cố định platform thread）

Tối ưu hóa cơ chế hoạt động của virtual thread với `synchronized`. Virtual thread khi bị block trong phương thức hoặc khối mã `synchronized` thường có thể giải phóng thread hệ điều hành (platform thread) mà nó chiếm dụng, tránh việc chiếm dụng platform thread trong thời gian dài, từ đó nâng cao khả năng đồng thời của ứng dụng. Cơ chế này tránh được tình trạng "cố định (Pinning)" - tức là virtual thread chiếm dụng platform thread thời gian dài, ngăn cản nó phục vụ cho các virtual thread khác.

Mã Java hiện có sử dụng `synchronized` không cần sửa đổi vẫn có thể hưởng lợi từ khả năng mở rộng của virtual thread. Ví dụ, một ứng dụng thiên về I/O (I/O intensive), nếu sử dụng platform thread truyền thống, có thể vì thread bị block mà dẫn đến khả năng đồng thời bị giảm. Còn khi sử dụng virtual thread, ngay cả khi xảy ra block trong khối `synchronized`, cũng sẽ không cố định platform thread, từ đó cho phép platform thread tiếp tục phục vụ các virtual thread khác, nâng cao hiệu năng đồng thời tổng thể.

## JEP 493: Linking Run-Time Images Without JMOD Files（Liên kết runtime image mà không cần file JMOD）

Theo mặc định, JDK chứa đồng thời runtime image (các module cần thiết khi runtime) và các file JMOD. Tính năng này giúp công cụ jlink không cần sử dụng file JMOD của JDK cũng có thể tạo runtime image tùy chỉnh, giảm thể tích cài đặt của JDK (khoảng 25%).

Giải thích:

- Jlink là công cụ dòng lệnh mới phát hành cùng Java 9. Nó cho phép các nhà phát triển tạo JRE nhẹ, tùy chỉnh của riêng họ cho các ứng dụng Java dựa trên module.
- File JMOD là file lưu trữ module hóa, có thể chứa class file, thư viện native, file cấu hình, header file, thông báo pháp lý,...

## JEP 495: Simplified Source Files and Instance Main Methods（File nguồn và phương thức main thể hiện đơn giản hóa, Xem trước lần 4）

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

Đơn giản hóa thêm một bước (Class không đặt tên cho phép chúng ta bỏ qua tên class):

```java
void main() {
   System.out.println("Hello, World!");
}
```

## JEP 497: Quantum-Resistant Digital Signature Algorithm (ML-DSA)（Thuật toán chữ ký số kháng lượng tử）

JDK 24 giới thiệu hỗ trợ thuật toán chữ ký số dựa trên ma trận module kháng lượng tử (Module-Lattice-Based Digital Signature Algorithm, **ML-DSA**), chuẩn bị cho việc chống lại các mối đe dọa có thể mang lại bởi các máy tính lượng tử trong tương lai.

ML-DSA là thuật toán kháng lượng tử được Viện Tiêu chuẩn và Công nghệ Quốc gia Hoa Kỳ (NIST) chuẩn hóa trong FIPS 204, dùng cho chữ ký số và xác thực danh tính.

## JEP 498: Warnings When Using `sun.misc.Unsafe` Memory Access Methods (Cảnh báo khi sử dụng các phương thức truy cập bộ nhớ `sun.misc.Unsafe`)

JDK 23 ([JEP 471](https://openjdk.org/jeps/471)) đã đề xuất loại bỏ các phương thức truy cập bộ nhớ trong `sun.misc.Unsafe`, các phương thức này sẽ bị xóa trong các phiên bản tương lai. Trong JDK 24, khi lần đầu tiên gọi bất kỳ phương thức truy cập bộ nhớ nào của `sun.misc.Unsafe`, runtime sẽ phát ra cảnh báo.

Các phương thức không an toàn này đã có giải pháp thay thế an toàn và hiệu quả:

- `java.lang.invoke.VarHandle`: Được giới thiệu trong JDK 9 (JEP 193), cung cấp một phương pháp thao tác bộ nhớ heap an toàn và hiệu quả, bao gồm các trường của đối tượng, các trường static của class và các phần tử mảng.
- `java.lang.foreign.MemorySegment`: Được giới thiệu trong JDK 22 (JEP 454), cung cấp một phương pháp truy cập bộ nhớ ngoài heap an toàn và hiệu quả, đôi khi phối hợp hoạt động với `VarHandle`.

`MemorySegment` là một trong những kiểu cốt lõi của Foreign Function & Memory API, dùng để truy cập an toàn bộ nhớ trong heap hoặc ngoài heap; `VarHandle` là API chuẩn độc lập, có thể dùng để truy cập trường, phần tử mảng hoặc truy cập dữ liệu theo bố cục bộ nhớ. Foreign Function & Memory API đã chuyển thành tính năng chính thức trong JDK 22.

```java
import java.lang.foreign.*;
import java.lang.invoke.VarHandle;

// 管理堆外整数数组的类
class OffHeapIntBuffer {

    // 用于访问整数元素的VarHandle
    private static final VarHandle ELEM_VH = ValueLayout.JAVA_INT.arrayElementVarHandle();

    // 内存管理器
    private final Arena arena;

    // 堆外内存段
    private final MemorySegment buffer;

    // 构造函数，分配指定数量的整数空间
    public OffHeapIntBuffer(long size) {
        this.arena  = Arena.ofShared();
        this.buffer = arena.allocate(ValueLayout.JAVA_INT, size);
    }

    // 释放内存
    public void deallocate() {
        arena.close();
    }

    // 以volatile方式设置指定索引的值
    public void setVolatile(long index, int value) {
        ELEM_VH.setVolatile(buffer, 0L, index, value);
    }

    // 初始化指定范围的元素为0
    public void initialize(long start, long n) {
        buffer.asSlice(ValueLayout.JAVA_INT.byteSize() * start,
                       ValueLayout.JAVA_INT.byteSize() * n)
              .fill((byte) 0);
    }

    // 将指定范围的元素复制到新数组
    public int[] copyToNewArray(long start, int n) {
        return buffer.asSlice(ValueLayout.JAVA_INT.byteSize() * start,
                              ValueLayout.JAVA_INT.byteSize() * n)
                     .toArray(ValueLayout.JAVA_INT);
    }
}
```

## JEP 499: Structured Concurrency（Đồng thời cấu trúc, Xem trước lần 4）

JDK 19 đã giới thiệu Structured Concurrency dưới dạng Incubator API. Trong JDK 24, API này đang ở giai đoạn xem trước lần thứ 4, mục đích là đơn giản hóa lập trình đa luồng, không phải để thay thế `java.util.concurrent`.

Structured Concurrency coi nhiều tác vụ chạy trong các thread khác nhau là một đơn vị công việc đơn lẻ, từ đó đơn giản hóa việc xử lý lỗi, nâng cao độ tin cậy và tăng cường khả năng quan sát. Nghĩa là, Structured Concurrency giữ lại tính dễ đọc, tính dễ bảo trì và tính quan sát của mã đơn luồng.

API cơ bản của Structured Concurrency là `StructuredTaskScope`, nó hỗ trợ chia nhỏ tác vụ thành nhiều tác vụ con đồng thời, thực thi trong các thread của chính chúng, và các tác vụ con bắt buộc phải hoàn thành trước khi tác vụ chính tiếp tục.

Cách dùng cơ bản của `StructuredTaskScope` như sau:

```java
    try (var scope = new StructuredTaskScope<Object>()) {
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
