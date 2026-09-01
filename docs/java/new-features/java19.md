---
title: Java 19 新特性概览
description: 介绍 JDK 19 的预览特性与并发相关更新，为后续虚拟线程铺垫。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 19,JDK19,虚拟线程预览,结构化并发,外部函数 API,JEP
---

JDK 19 được phát hành chính thức vào ngày 20 tháng 9 năm 2022, là phiên bản không phải Hỗ trợ dài hạn.

JDK 19 có tổng cộng 7 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 424: Foreign Function & Memory API (Hàm ngoại và Memory API)](https://openjdk.org/jeps/424) (Xem trước)
- [JEP 425: Virtual Threads (Luồng ảo)](https://openjdk.org/jeps/425) (Xem trước)
- [JEP 426: Vector API (Vector API)](https://openjdk.java.net/jeps/426) (Lần ươm tạo thứ 4)
- [JEP 428: Structured Concurrency (Đồng thời cấu trúc / Đa luồng cấu trúc)](https://openjdk.org/jeps/428) (Ươm tạo)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 424: Hàm ngoại và Memory API (Xem trước)

Các chương trình Java có thể thông qua API này để tương tác với mã và dữ liệu bên ngoài Java runtime. Thông qua việc gọi hàm ngoại (tức mã ngoài JVM) hiệu quả và truy cập bộ nhớ ngoài (tức bộ nhớ không do JVM quản lý) an toàn, API này giúp chương trình Java có thể gọi thư viện native và xử lý dữ liệu native, chứ không nguy hiểm và dễ vỡ như JNI.

Foreign Function & Memory API thực hiện vòng ươm tạo thứ nhất trong Java 17, do [JEP 412](https://openjdk.java.net/jeps/412) đề xuất. Vòng ươm tạo thứ hai do [JEP 419](https://openjdk.org/jeps/419) đề xuất và tích hợp vào Java 18, bản xem trước do [JEP 424](https://openjdk.org/jeps/424) đề xuất và tích hợp vào Java 19.

Trước khi có Foreign Function & Memory API:

- Java thông qua [`sun.misc.Unsafe`](https://hg.openjdk.java.net/jdk/jdk/file/tip/src/jdk.unsupported/share/classes/sun/misc/Unsafe.java) cung cấp một số phương thức thực thi thao tác cấp thấp, không an toàn (như truy cập trực tiếp tài nguyên bộ nhớ hệ thống, tự quản lý tài nguyên bộ nhớ,...), lớp `Unsafe` giúp ngôn ngữ Java có được khả năng thao tác không gian bộ nhớ tương tự như con trỏ trong ngôn ngữ C, đồng thời cũng làm tăng tính không an toàn của ngôn ngữ Java, việc sử dụng không đúng lớp `Unsafe` sẽ làm cho xác suất chương trình phát sinh lỗi lớn hơn.
- Java 1.1 đã hỗ trợ gọi phương thức native thông qua Java Native Interface (JNI), nhưng không dễ dùng. Triển khai JNI quá phức tạp, các bước phiền phức (các bước cụ thể có thể tham khảo bài viết: [Guide to JNI (Java Native Interface)](https://www.baeldung.com/jni)), không bị kiểm soát bởi cơ chế an toàn ngôn ngữ của JVM, ảnh hưởng đến tính đa nền tảng của ngôn ngữ Java. Hơn nữa, hiệu năng của JNI cũng không tốt, vì việc gọi phương thức JNI không thể được hưởng lợi từ nhiều tối ưu hóa JIT thông thường (như inlining). Mặc dù các framework như [JNA](https://github.com/java-native-access/jna), [JNR](https://github.com/jnr/jnr-ffi) và [JavaCPP](https://github.com/bytedeco/javacpp) đã cải tiến JNI, nhưng kết quả vẫn chưa thực sự lý tưởng.

Giới thiệu Foreign Function & Memory API là để giải quyết một số điểm nhói lòng khi Java truy cập hàm ngoại và bộ nhớ ngoài.

Foreign Function & Memory API (FFM API) định nghĩa các lớp và interface:

- Phân bổ bộ nhớ ngoài: `MemorySegment`, `MemoryAddress` và `SegmentAllocator`
- Thao tác và truy cập bộ nhớ ngoài cấu trúc hóa: `MemoryLayout`, `VarHandle`
- Kiểm soát phân bổ và giải phóng bộ nhớ ngoài: `MemorySession`
- Gọi hàm ngoại: `Linker`, `FunctionDescriptor` và `SymbolLookup`

Dưới đây là ví dụ sử dụng FFM API, đoạn mã này lấy method handle của phương thức `radixsort` trong C library, sau đó sử dụng nó để sắp xếp bốn chuỗi trong mảng Java.

```java
// 1. 在 C 库路径上查找外部函数
Linker linker = Linker.nativeLinker();
SymbolLookup stdlib = linker.defaultLookup();
MethodHandle radixSort = linker.downcallHandle(
                             stdlib.lookup("radixsort"), ...);
// 2. 分配堆上内存以存储四个字符串
String[] javaStrings   = { "mouse", "cat", "dog", "car" };
// 3. 分配堆外内存以存储四个指针
SegmentAllocator allocator = implicitAllocator();
MemorySegment offHeap  = allocator.allocateArray(ValueLayout.ADDRESS, javaStrings.length);
// 4. 将字符串从堆上复制到堆外
for (int i = 0; i < javaStrings.length; i++) {
    // 在堆外分配一个字符串，然后存储指向它的指针
    MemorySegment cString = allocator.allocateUtf8String(javaStrings[i]);
    offHeap.setAtIndex(ValueLayout.ADDRESS, i, cString);
}
// 5. 通过调用外部函数对堆外数据进行排序
radixSort.invoke(offHeap, javaStrings.length, MemoryAddress.NULL, '\0');
// 6. 将（重新排序的）字符串从堆外复制到堆上
for (int i = 0; i < javaStrings.length; i++) {
    MemoryAddress cStringPtr = offHeap.getAtIndex(ValueLayout.ADDRESS, i);
    javaStrings[i] = cStringPtr.getUtf8String(0);
}
assert Arrays.equals(javaStrings, new String[] {"car", "cat", "dog", "mouse"});  // true
```

## JEP 425: Luồng ảo (Virtual Thread, Xem trước)

Virtual Thread (Luồng ảo) là thread nhẹ do JDK triển khai chứ không phải OS, nhiều virtual thread chia sẻ cùng một thread của hệ điều hành, số lượng virtual thread có thể lớn hơn nhiều so với số lượng thread của hệ điều hành.

Virtual thread đã được chứng minh là rất hữu ích trong nhiều ngôn ngữ đa luồng khác, ví dụ Goroutine trong Go, Process trong Erlang.

Virtual thread thường không cần tạo hoặc chuyển đổi một thread của hệ điều hành cho mỗi tác vụ, từ đó giảm chi phí tài nguyên thread và điều phối (scheduling) do số lượng lớn các tác vụ bị block mang lại, đồng thời giảm khối lượng công việc viết, bảo trì và quan sát các ứng dụng đồng thời thông lượng cao.

Zhihu có một thảo luận về Virtual Thread trong Java 19, ai quan tâm có thể xem: <https://www.zhihu.com/question/536743167>.

Giải thích chi tiết và nguyên lý của Java Virtual Thread có thể xem hai bài viết dưới đây:

- [Nguyên lý và phân tích hiệu năng Virtual Thread ｜ Dewu Tech](https://mp.weixin.qq.com/s/vdLXhZdWyxc6K-D3Aj03LA)
- [Java 19 chính thức GA! Xem cách Virtual Thread nâng cao đáng kể thông lượng hệ thống](https://mp.weixin.qq.com/s/yyApBXxpXxVwttr01Hld6Q)
- [Virtual Thread - Soi mã nguồn VirtualThread](https://www.cnblogs.com/throwable/p/16758997.html)

## JEP 426: Vector API (Lần ươm tạo thứ 4)

Vector API ban đầu do [JEP 338](https://openjdk.java.net/jeps/338) đề xuất, và được tích hợp vào Java 16 dưới dạng Incubator API. Vòng ươm tạo thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và tích hợp vào Java 17, vòng ươm tạo thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và tích hợp vào Java 18, vòng thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và tích hợp vào Java 19.

Trong bài [Tổng quan tính năng mới Java 18](./java18.md), tôi đã giới thiệu chi tiết về Vector API, ở đây không giới thiệu thêm nữa.

## JEP 428: Structured Concurrency (Lập trình đồng thời cấu trúc, Ươm tạo)

JDK 19 giới thiệu Structured Concurrency, một phương pháp lập trình đa luồng, mục đích là thông qua Structured Concurrency API để đơn giản hóa lập trình đa luồng, không phải để thay thế `java.util.concurrent`, hiện đang ở giai đoạn ươm tạo.

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

<!-- @include: @article-footer.snippet.md -->
