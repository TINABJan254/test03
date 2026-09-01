---
title: CAS 详解
description: CAS比较并交换深度解析：详解CAS原子操作原理、Unsafe类实现、ABA问题及解决方案、自旋锁机制、与悲观锁性能对比。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: CAS,Compare-And-Swap,原子操作,ABA问题,自旋锁,乐观锁,Unsafe,CAS原理
---

Phần giới thiệu về Khóa lạc quan và Khóa bi quan cũng như các cách triển khai phổ biến của Khóa lạc quan, bạn có thể đọc bài viết tôi viết tại đây: [Giải thích chi tiết Khóa lạc quan và Khóa bi quan](https://javaguide.cn/java/concurrent/optimistic-lock-and-pessimistic-lock.html).

Bài viết này chủ yếu giới thiệu: Cách triển khai CAS trong Java cũng như một số vấn đề tồn tại của CAS.

## CAS trong Java được triển khai như thế nào?

Trong Java, một lớp then chốt để triển khai thao tác CAS (Compare-And-Swap, So sánh và Trao đổi) là `Unsafe`.

Lớp `Unsafe` nằm trong gói `sun.misc`, là một lớp cung cấp các thao tác cấp thấp, không an toàn. Do tính năng mạnh mẽ và nguy cơ tiềm ẩn của nó, nó thường được dùng trong nội bộ JVM hoặc một số thư viện cần hiệu năng cực cao và truy cập cấp thấp, chứ không khuyến nghị các nhà phát triển thông thường dùng trong ứng dụng. Về bài giới thiệu chi tiết lớp `Unsafe`, bạn có thể đọc bài viết này: 📌[Giải thích chi tiết lớp ma thuật Unsafe trong Java](https://javaguide.cn/java/basis/unsafe.html).

Lớp `Unsafe` dưới gói `sun.misc` cung cấp các phương thức `compareAndSwapObject`, `compareAndSwapInt`, `compareAndSwapLong` để triển khai thao tác CAS cho các kiểu `Object`, `int`, `long`:

```java
/**
 * Cập nhật giá trị field của đối tượng theo cách nguyên tử.
 *
 * @param o        Đối tượng cần thao tác
 * @param offset   Offset bộ nhớ của field đối tượng
 * @param expected Giá trị cũ kỳ vọng
 * @param x        Giá trị mới cần đặt
 * @return Nếu giá trị được cập nhật thành công trả về true; ngược lại trả về false
 */
boolean compareAndSwapObject(Object o, long offset, Object expected, Object x);

/**
 * Cập nhật giá trị field kiểu int của đối tượng theo cách nguyên tử.
 */
boolean compareAndSwapInt(Object o, long offset, int expected, int x);

/**
 * Cập nhật giá trị field kiểu long của đối tượng theo cách nguyên tử.
 */
boolean compareAndSwapLong(Object o, long offset, long expected, long x);
```

Các phương thức CAS do `Unsafe` cung cấp trong JDK 8 là các phương thức `native`. Code Java thông qua chúng để thể hiện ngữ nghĩa so sánh và trao đổi nguyên tử, HotSpot thường nhận diện các lời gọi liên quan là hàm nội bộ JVM (intrinsic), rồi ánh xạ thành lệnh nguyên tử mà bộ xử lý mục tiêu hỗ trợ hoặc cách triển khai tương đương. Cách triển khai cụ thể phụ thuộc vào kiến trúc JVM và CPU, nhưng không thể tóm tắt đơn giản là "gọi C++ inline assembly qua JNI".

Gói `java.util.concurrent.atomic` cung cấp một số lớp dùng cho các thao tác nguyên tử.

![JUC原子类概览](https://oss.javaguide.cn/github/javaguide/java/JUC%E5%8E%9F%E5%AD%90%E7%B1%BB%E6%A6%82%E8%A7%88.png)

Về phần giới thiệu và cách sử dụng các lớp Atomic nguyên tử này, bạn có thể đọc bài viết: [Tổng kết các lớp Atomic nguyên tử](https://javaguide.cn/java/concurrent/atomic-classes.html).

Các lớp Atomic phụ thuộc vào khóa lạc quan CAS để đảm bảo tính nguyên tử cho các phương thức của nó, mà không cần sử dụng cơ chế khóa truyền thống (như khối `synchronized` hay `ReentrantLock`).

`AtomicInteger` là một trong những lớp nguyên tử của Java, chủ yếu dùng để thao tác nguyên tử trên biến kiểu `int`. Cách triển khai trong JDK 8 dưới đây tận dụng các phương thức thao tác nguyên tử cấp thấp do `Unsafe` cung cấp; trong các JDK mới hơn, API liên quan và chi tiết triển khai bên trong có thể khác, code ứng dụng cũng có thể dùng `VarHandle` chuẩn để thể hiện ngữ nghĩa truy cập nguyên tử.

Dưới đây, chúng ta hãy đọc mã nguồn cốt lõi của `AtomicInteger` (JDK1.8) để giải thích cách Java dùng phương thức của lớp `Unsafe` triển khai thao tác nguyên tử.

Mã nguồn cốt lõi của `AtomicInteger` như sau:

```java
// Lấy thể hiện Unsafe
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        // Lấy offset bộ nhớ của field "value" trong lớp AtomicInteger
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
// Đảm bảo tính nhìn thấy của field "value"
private volatile int value;

// Nếu giá trị hiện tại bằng giá trị kỳ vọng, thì đặt giá trị thành newValue một cách nguyên tử
// Dùng phương thức Unsafe#compareAndSwapInt để thực hiện thao tác CAS
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

// Cộng delta vào giá trị hiện tại một cách nguyên tử và trả về giá trị cũ
public final int getAndAdd(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta);
}

// Cộng 1 vào giá trị hiện tại một cách nguyên tử và trả về giá trị trước khi cộng (giá trị cũ)
// Dùng phương thức Unsafe#getAndAddInt để thực hiện thao tác CAS.
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

// Trừ 1 vào giá trị hiện tại một cách nguyên tử và trả về giá trị trước khi trừ (giá trị cũ)
public final int getAndDecrement() {
    return unsafe.getAndAddInt(this, valueOffset, -1);
}
```

Mã nguồn `Unsafe#getAndAddInt`:

```java
// Lấy và tăng giá trị số nguyên một cách nguyên tử
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        // Lấy giá trị số nguyên tại offset bộ nhớ của đối tượng o theo kiểu volatile
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));
    // Trả về giá trị cũ
    return v;
}
```

Có thể thấy, `getAndAddInt` sử dụng vòng lặp `do-while`: khi thao tác `compareAndSwapInt` thất bại, nó sẽ liên tục thử lại cho đến khi thành công. Nghĩa là phương thức `getAndAddInt` thông qua phương thức `compareAndSwapInt` để thử cập nhật giá trị của `value`, nếu cập nhật thất bại (giá trị hiện tại trong thời gian đó bị luồng khác sửa đổi), nó sẽ lấy lại giá trị hiện tại và thử cập nhật lại lần nữa, cho đến khi thao tác thành công.

Do thao tác CAS có thể thất bại vì xung đột concurrency, nên thường được phối hợp với vòng lặp `while`, sau khi thất bại sẽ liên tục thử lại cho đến khi thao tác thành công. Đây chính là **cơ chế tự xoay (Spin Lock)**.

## Thuật toán CAS có những vấn đề gì?

Vấn đề ABA là vấn đề thường gặp nhất của thuật toán CAS.

### Vấn đề ABA

Nếu một biến V lần đọc đầu tiên là giá trị A, và khi chuẩn bị gán giá trị kiểm tra lại thấy nó vẫn là A, vậy chúng ta có thể khẳng định giá trị của nó chưa từng bị luồng khác sửa đổi không? Rõ ràng là không thể, bởi vì trong khoảng thời gian đó giá trị của nó có thể đã bị sửa thành giá trị khác, sau đó lại sửa về A, khi đó thao tác CAS sẽ hiểu nhầm là nó chưa từng bị sửa đổi. Vấn đề này được gọi là **vấn đề "ABA"** của thao tác CAS.

Hướng giải quyết vấn đề ABA là bổ sung thêm **số phiên bản hoặc nhãn thời gian** phía trước biến. Lớp `AtomicStampedReference` từ JDK 1.5 trở đi dùng để giải quyết vấn đề ABA, trong đó phương thức `compareAndSet()` đầu tiên kiểm tra xem tham chiếu hiện tại có bằng tham chiếu kỳ vọng hay không, và stamp hiện tại có bằng stamp kỳ vọng hay không, nếu tất cả đều bằng nhau thì sẽ cập nhật giá trị của tham chiếu và stamp đó thành giá trị cập nhật đã cho một cách nguyên tử.

```java
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
          newStamp == current.stamp) ||
         casPair(current, Pair.of(newReference, newStamp)));
}
```

### Vòng lặp thời gian dài gây chi phí lớn

CAS thường dùng thao tác tự xoay (spin) để thử lại, tức là chưa thành công thì liên tục lặp cho đến khi thành công. Nếu thời gian dài không thành công, sẽ mang lại chi phí thực thi rất lớn cho CPU.

Nếu JVM hỗ trợ lệnh `pause` do bộ xử lý cung cấp, hiệu quả của thao tác tự xoay sẽ được nâng cao. Lệnh `pause` có 2 tác dụng quan trọng:

1. **Trì hoãn thực thi lệnh trong pipeline**: Lệnh `pause` có thể trì hoãn việc thực thi lệnh, từ đó giảm tiêu thụ tài nguyên CPU. Thời gian trì hoãn cụ thể phụ thuộc vào phiên bản triển khai của bộ xử lý, trên một số bộ xử lý, thời gian trì hoãn có thể bằng 0.
2. **Tránh xung đột thứ tự bộ nhớ**: Khi thoát khỏi vòng lặp, lệnh `pause` có thể tránh việc pipeline CPU bị xóa sạch (flush) do xung đột thứ tự bộ nhớ, từ đó nâng cao hiệu suất thực thi của CPU.

### Chỉ có thể đảm bảo thao tác nguyên tử cho một biến dùng chung

Thao tác CAS chỉ có hiệu quả đối với một biến dùng chung đơn lẻ. Khi cần thao tác trên nhiều biến dùng chung, CAS tỏ ra bất lực. Tuy nhiên, bắt đầu từ JDK 1.5, Java cung cấp lớp `AtomicReference`, giúp chúng ta có thể đảm bảo tính nguyên tử giữa các đối tượng tham chiếu. Bằng cách đóng gói nhiều biến vào trong một đối tượng, chúng ta có thể dùng `AtomicReference` để thực hiện thao tác CAS.

Ngoài cách dùng `AtomicReference`, cũng có thể tận dụng việc cài khóa (locking) để đảm bảo.

## Tóm tắt

Trong Java, các API như các lớp nguyên tử, `VarHandle` có thể thể hiện thao tác CAS; JVM sẽ dựa trên platform mục tiêu để triển khai nó thành lệnh nguyên tử được bộ xử lý hỗ trợ hoặc cơ chế tương đương. Triển khai cụ thể phụ thuộc vào kiến trúc JVM và CPU, chứ không bị giới hạn bởi Java specification là JNI hay một kiểu viết assembly nào đó.

CAS mặc dù có đặc tính không khóa (lock-free) hiệu năng cao, nhưng cũng cần lưu ý các vấn đề như ABA, vòng lặp thời gian dài gây chi phí lớn.
