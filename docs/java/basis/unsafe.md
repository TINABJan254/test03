---
title: Giải thích chi tiết Magic Class Unsafe trong Java
description: Phân tích sâu class ma thuật Unsafe trong Java: giải thích khả năng thao tác trực tiếp bộ nhớ, thao tác nguyên tử CAS, khởi tạo đối tượng,... hiểu nguyên lý triển khai bên dưới của các công cụ conccurency JUC và các rủi ro khi sử dụng.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Lớp Unsafe,thao tác bộ nhớ,thao tác nguyên tử CAS,bộ nhớ ngoài Heap,bộ nhớ trực tiếp,sun.misc.Unsafe,triển khai JUC bên dưới
---

> Bài viết được tổng hợp và hoàn thiện từ hai bài viết xuất sắc dưới đây:
>
> - [Java Magic Class: Phân tích ứng dụng Unsafe - Meituan Tech Team - 2019](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)
> - [Giải thích chi tiết Unsafe - Thanh bảo kiếm 2 lưỡi của Java - 2021](https://xie.infoq.cn/article/8b6ed4195e475bfb32dacc5cb)

<!-- markdownlint-disable MD024 -->

Các bạn từng đọc mã nguồn JUC chắc chắn sẽ phát hiện ra nhiều utility class concurrency đều gọi một lớp tên là `Unsafe`.

Vậy lớp này chủ yếu dùng để làm gì? Có các kịch bản ứng dụng nào? Bài viết này sẽ đưa bạn tìm hiểu rõ ràng!

## Giới thiệu Unsafe

`Unsafe` là một class nằm trong package `sun.misc`, chủ yếu cung cấp một số phương thức dùng để thực thi các thao tác cấp thấp, không an toàn (low-level, unsafe), như truy cập trực tiếp tài nguyên bộ nhớ hệ thống, tự chủ quản lý tài nguyên bộ nhớ,... Các phương thức này đóng vai trò rất lớn trong việc nâng cao hiệu năng chạy của Java và tăng cường khả năng thao tác tài nguyên bên dưới của ngôn ngữ Java. Tuy nhiên do `Unsafe` trao cho ngôn ngữ Java khả năng thao tác không gian bộ nhớ tương tự con trỏ trong C, điều này vô hình trung cũng làm tăng rủi ro xảy ra các sự cố liên quan đến con trỏ trong chương trình. Việc lạm dụng hoặc sử dụng không đúng cách `Unsafe` trong chương trình sẽ làm tăng xác suất phát sinh lỗi, khiến một ngôn ngữ an toàn như Java trở nên "không còn an toàn" nữa, do đó việc sử dụng `Unsafe` nhất định phải vô cùng cẩn trọng.

Ngoài ra, việc thực thi các chức năng mà `Unsafe` cung cấp phụ thuộc vào Native Method (Phương thức bản địa). Bạn có thể xem Native Method như những phương thức được viết bằng các ngôn ngữ lập trình khác trong Java. Native Method được修饰 bởi từ khóa **`native`**, trong code Java chỉ khai báo header của method, còn bản thực thi cụ thể được giao cho **Native Code** (mã bản địa).

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717115231125.png)

**Tại sao lại sử dụng Native Method?**

1. Cần dùng tới các đặc tính phụ thuộc hệ điều hành mà Java không có sẵn, Java khi thực hiện đa nền tảng đồng thời muốn kiểm soát bên dưới thì cần nhờ các ngôn ngữ khác phát huy tác dụng.
2. Đối với các chức năng có sẵn đã được hoàn thành bằng ngôn ngữ khác, có thể dùng Java gọi trực tiếp.
3. Khi chương trình nhạy cảm với thời gian hoặc có yêu cầu hiệu năng cực cao, cần thiết phải dùng ngôn ngữ cấp thấp hơn, ví dụ C/C++ hoặc thậm chí Assembly.

Nhiều utility class concurrency trong package JUC khi thực hiện cơ chế concurrency đều gọi Native Method, thông qua chúng phá vỡ ranh giới Java runtime để tiếp xúc với một số tính năng bên dưới hệ điều hành. Đối với cùng một Native Method, các hệ điều hành khác nhau có thể thực thi theo các cách khác nhau, nhưng đối với người dùng thì hoàn toàn trong suốt, cuối cùng đều thu được cùng một kết quả.

## Tạo Unsafe

Một phần mã nguồn của `sun.misc.Unsafe` như sau:

```java
public final class Unsafe {
  // Singleton object
  private static final Unsafe theUnsafe;
  ......
  private Unsafe() {
  }
  @CallerSensitive
  public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    // Chỉ hợp lệ khi được nạp bởi BootstrapClassLoader
    if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
      throw new SecurityException("Unsafe");
    } else {
      return theUnsafe;
    }
  }
}
```

Lớp `Unsafe` được triển khai theo dạng Singleton, cung cấp phương thức static `getUnsafe` để lấy instance `Unsafe`. Phương thức này trông có vẻ như dùng để lấy `Unsafe` instance. Tuy nhiên khi chúng ta gọi trực tiếp phương thức static này sẽ throw ngoại lệ `SecurityException`:

```bash
Exception in thread "main" java.lang.SecurityException: Unsafe
 at sun.misc.Unsafe.getUnsafe(Unsafe.java:90)
 at com.cn.test.GetUnsafeTest.main(GetUnsafeTest.java:12)
```

**Tại sao phương thức `public static` lại không thể gọi trực tiếp?**

Đó là vì trong phương thức `getUnsafe` có kiểm tra `classLoader` của bên gọi, đánh giá xem class hiện tại có phải do `Bootstrap ClassLoader` nạp hay không, nếu không phải sẽ throw `SecurityException`. Nghĩa là chỉ các class do Bootstrap ClassLoader nạp mới có thể gọi phương thức trong lớp Unsafe nhằm ngăn chặn các phương thức này bị gọi trong mã nguồn không đáng tin cậy.

**Tại sao phải hạn chế sử dụng lớp Unsafe một cách cẩn trọng như vậy?**

Các tính năng mà `Unsafe` cung cấp quá sâu bên dưới (như truy cập trực tiếp tài nguyên bộ nhớ hệ thống, tự quản lý bộ nhớ,...), nguy cơ an toàn bảo mật cũng rất lớn, nếu dùng không đúng cách rất dễ xảy ra sự cố nghiêm trọng.

**Nếu muốn sử dụng lớp `Unsafe` này thì làm thế nào để lấy instance của nó?**

Ở đây giới thiệu 2 phương án khả thi.

1. Sử dụng Reflection lấy đối tượng Singleton `theUnsafe` đã được khởi tạo sẵn trong lớp Unsafe.

```java
private static Unsafe reflectGetUnsafe() {
    try {
      Field field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      return (Unsafe) field.get(null);
    } catch (Exception e) {
      log.error(e.getMessage(), e);
      return null;
    }
}
```

2. Xuất phát từ điều kiện hạn chế của phương thức `getUnsafe`, thông qua lệnh dòng lệnh Java `-Xbootclasspath/a` nối đường dẫn file jar chứa class A (class gọi phương thức Unsafe) vào đường dẫn bootstrap mặc định, khiến A được nạp bởi Bootstrap ClassLoader, từ đó có thể lấy instance Unsafe an toàn qua `Unsafe.getUnsafe`.

```bash
java -Xbootclasspath/a: ${path}   // Trong đó path là đường dẫn file jar chứa class gọi phương thức Unsafe
```

## Chức năng của Unsafe

Tóm tắt lại, các chức năng mà lớp `Unsafe` thực hiện có thể chia thành 8 nhóm:

1. Thao tác bộ nhớ
2. Thao tác rào cản bộ nhớ (Memory Barrier)
3. Thao tác đối tượng (Object)
4. Thao tác mảng (Array)
5. Thao tác CAS
6. Lập lịch Thread (Thread Scheduling)
7. Thao tác Class
8. Thông tin hệ thống

### Thao tác bộ nhớ

#### Giới thiệu

Nếu bạn là lập trình viên từng viết C hoặc C++, chắc chắn không xa lạ gì với thao tác bộ nhớ, còn trong Java thì không cho phép trực tiếp thao tác trên bộ nhớ, việc cấp phát và thu gom bộ nhớ đối tượng đều do JVM tự thực hiện. Tuy nhiên trong `Unsafe`, các interface dưới đây cung cấp khả năng thao tác trực tiếp trên bộ nhớ:

```java
// Cấp phát vùng nhớ local mới
public native long allocateMemory(long bytes);
// Điều chỉnh lại kích thước vùng nhớ
public native long reallocateMemory(long address, long bytes);
// Đặt vùng nhớ thành giá trị chỉ định
public native void setMemory(Object o, long offset, long bytes, byte value);
// Copy bộ nhớ
public native void copyMemory(Object srcBase, long srcOffset,Object destBase, long destOffset,long bytes);
// Giải phóng bộ nhớ
public native void freeMemory(long address);
```

Sử dụng đoạn code dưới đây để test:

```java
private void memoryTest() {
    int size = 4;
    // 1. Cấp phát bộ nhớ ban đầu
    long oldAddr = unsafe.allocateMemory(size);
    System.out.println("Initial address: " + oldAddr);

    // 2. Ghi dữ liệu vào bộ nhớ ban đầu
    unsafe.putInt(oldAddr, 16843009); // Ghi 0x01010101
    System.out.println("Value at oldAddr: " + unsafe.getInt(oldAddr));

    // 3. Cấp phát lại bộ nhớ (mở rộng)
    long newAddr = unsafe.reallocateMemory(oldAddr, size * 2);
    System.out.println("New address: " + newAddr);

    // 4. reallocateMemory đã copy dữ liệu từ oldAddr sang newAddr
    // nên 4 byte đầu của newAddr phải giống nội dung oldAddr
    System.out.println("Value at newAddr (first 4 bytes): " + unsafe.getInt(newAddr));

    // Quan trọng: Sau đó mọi thao tác đều dựa trên newAddr, oldAddr đã vô hiệu!
    try {
        // 5. Ghi dữ liệu mới vào nửa sau của khối nhớ mới
        unsafe.putInt(newAddr + size, 33686018); // Ghi 0x02020202

        // 6. Đọc toàn bộ giá trị long 8 byte
        System.out.println("Value at newAddr (full 8 bytes): " + unsafe.getLong(newAddr));

    } finally {
        // 7. Chỉ giải phóng địa chỉ bộ nhớ hiệu lực cuối cùng
        unsafe.freeMemory(newAddr);
        // Nếu thử freeMemory(oldAddr) sẽ dẫn tới lỗi double free nghiêm trọng!
    }
}
```

Xem kết quả in ra trước:

```plain
Initial address: 140467048086752
Value at oldAddr: 16843009
New address: 140467048086752
Value at newAddr (first 4 bytes): 16843009
Value at newAddr (full 8 bytes): 144680345659310337
```

Hành vi của `reallocateMemory` tương tự hàm realloc trong C, nó sẽ thử mở rộng hoặc thu nhỏ khối nhớ mà không di chuyển dữ liệu. Hành vi của nó chủ yếu có 2 trường hợp:

1. **Mở rộng tại chỗ (In-place expansion)**: Nếu phía sau khối nhớ hiện tại có đủ không gian trống liên tục, `reallocateMemory` sẽ mở rộng bộ nhớ trực tiếp trên địa chỉ gốc và trả về địa chỉ ban đầu.
2. **Mở rộng di dời (Out-of-place expansion)**: Nếu phía sau khối nhớ hiện tại không đủ không gian, nó sẽ tìm một vùng nhớ mới đủ lớn, copy dữ liệu cũ sang, giải phóng địa chỉ bộ nhớ cũ và trả về địa chỉ mới.

**Kết hợp với kết quả chạy lần này, chúng ta có thể phân tích như sau:**

**Bước 1: Cấp phát ban đầu và ghi dữ liệu**

- `unsafe.allocateMemory(size)` cấp phát 4 byte bộ nhớ off-heap, địa chỉ là `140467048086752`.
- `unsafe.putInt(oldAddr, 16843009)` ghi giá trị int `16843009` vào địa chỉ đó, biểu diễn Hex là `0x01010101`. `getInt` đọc đúng, chứng minh ghi thành công.

**Bước 2: Mở rộng bộ nhớ tại chỗ**

- `long newAddr = unsafe.reallocateMemory(oldAddr, size * 2)` thử mở rộng khối nhớ lên 8 byte.
- Quan sát New address in ra: `140467048086752`, chúng ta thấy `newAddr` và `oldAddr` **hoàn toàn giống nhau**.
- Điều này cho thấy thao tác đã kích hoạt "Mở rộng tại chỗ". Hệ thống tìm thấy đủ không gian phía sau địa chỉ gốc `140467048086752`, trực tiếp mở rộng khối nhớ lên 8 byte. Trong quá trình này, địa chỉ cũ `oldAddr` vẫn còn hiệu lực và chính là `newAddr`, dữ liệu không hề bị di chuyển.

**Bước 3: Xác minh dữ liệu và ghi dữ liệu mới**

- `unsafe.getInt(newAddr)` đọc lại 4 byte đầu, kết quả vẫn là `16843009`, xác minh dữ liệu gốc nguyên vẹn.
- `unsafe.putInt(newAddr + size, 33686018)` ghi giá trị int mới `33686018` (Hex là `0x02020202`) vào 4 byte sau được mở rộng (offset là 4).

**Bước 4: Đọc toàn bộ dữ liệu**

- `unsafe.getLong(newAddr)` đọc một giá trị long (8 byte) từ địa chỉ bắt đầu. Lúc này 8 byte trong bộ nhớ là sự ghép nối giữa `0x01010101` (địa chỉ thấp) và `0x02020202` (địa chỉ cao).
- Trên máy Little-Endian, 8 byte này trong bộ nhớ được giải thích thành số Hex `0x0202020201010101`.
- Số Hex này chuyển sang số thập phân đúng bằng `144680345659310337`. Điều này giải thích hoàn hảo kết quả đầu ra.

**Bước 5: Giải phóng bộ nhớ an toàn**

- Trong khối `finally`, `unsafe.freeMemory(newAddr)` giải phóng an toàn toàn bộ khối nhớ 8 byte.
- Do lần này là mở rộng tại chỗ (`oldAddr == newAddr`), nên nếu viết thừa câu lệnh `freeMemory(oldAddr)` sẽ dẫn tới lỗi nghiêm trọng giải phóng 2 lần (double free).

#### Ứng dụng điển hình

`DirectByteBuffer` là một class quan trọng trong Java dùng để triển khai bộ nhớ off-heap (bộ nhớ ngoài Heap), thường dùng làm buffer pool trong truyền thông, ví dụ ứng dụng rộng rãi trong các NIO framework như Netty, MINA. Việc tạo, sử dụng, hủy bộ nhớ off-heap của `DirectByteBuffer` đều do các API bộ nhớ off-heap của Unsafe thực hiện.

**Tại sao nên sử dụng bộ nhớ ngoài Heap (Off-heap Memory)?**

- Cải thiện tình trạng tạm dừng do Garbage Collection (GC pause). Vì bộ nhớ off-heap do trực tiếp hệ điều hành quản lý chứ không phải JVM, nên khi dùng bộ nhớ off-heap chúng ta có thể duy trì quy mô bộ nhớ trong Heap nhỏ hơn. Từ đó giảm ảnh hưởng của GC pause đối với ứng dụng.
- Nâng cao hiệu năng thao tác I/O của chương trình. Thông thường trong giao tiếp I/O sẽ tồn tại thao tác copy dữ liệu từ bộ nhớ in-heap sang bộ nhớ off-heap, đối với dữ liệu tạm thời cần copy dữ liệu thường xuyên giữa các bộ nhớ và có vòng đời ngắn, khuyến nghị nên lưu vào bộ nhớ off-heap.

Constructor của `DirectByteBuffer` bên dưới: Khi tạo `DirectByteBuffer`, thông qua `Unsafe.allocateMemory` cấp phát bộ nhớ, `Unsafe.setMemory` khởi tạo bộ nhớ, sau đó dựng đối tượng `Cleaner` dùng để theo dõi việc thu gom rác của đối tượng `DirectByteBuffer`, nhằm đạt được mục đích khi `DirectByteBuffer` bị GC thu gom thì bộ nhớ off-heap đã cấp phát cũng được giải phóng cùng.

```java
DirectByteBuffer(int cap) {                   // package-private

    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        // Cấp phát bộ nhớ và trả về địa chỉ cơ sở
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    // Khởi tạo bộ nhớ
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    // Theo dõi việc thu gom rác của đối tượng DirectByteBuffer để giải phóng bộ nhớ off-heap
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

### Rào cản bộ nhớ (Memory Barrier)

#### Giới thiệu

Trước khi giới thiệu Memory Barrier, cần biết rằng trình biên dịch và CPU sẽ tiến hành Reordering (sắp xếp lại chỉ thị) code dưới điều kiện đảm bảo kết quả đầu ra của chương trình nhất quán nhằm tối ưu hiệu năng. Tuy nhiên việc reordering chỉ thị có thể mang lại kết quả không tốt, dẫn tới bất đồng nhất dữ liệu giữa cache tốc độ cao của CPU và RAM. Memory Barrier chính là cơ chế ngăn chặn reordering chỉ thị ở 2 bên rào cản, tránh việc tối ưu hóa không đúng của trình biên dịch và phần cứng.

Về mặt phần cứng, Memory Barrier là các chỉ thị mà CPU cung cấp để ngăn reordering code, cách thực thi Memory Barrier trên các nền tảng phần cứng khác nhau có thể không giống nhau. Trong Java 8 đã đưa vào 3 hàm Memory Barrier, nó che giấu sự khác biệt bên dưới của hệ điều hành, cho phép định nghĩa trong code và do JVM thống nhất sinh ra chỉ thị Memory Barrier để thực hiện tính năng rào cản bộ nhớ.

`Unsafe` cung cấp 3 phương thức liên quan đến Memory Barrier bên dưới:

```java
// Rào cản bộ nhớ, cấm reordering thao tác load. Thao tác load trước rào cản không thể bị reorder xuống sau rào cản, và ngược lại
public native void loadFence();
// Rào cản bộ nhớ, cấm reordering thao tác store. Thao tác store trước rào cản không thể bị reorder xuống sau rào cản, và ngược lại
public native void storeFence();
// Rào cản bộ nhớ, cấm reordering thao tác load, store
public native void fullFence();
```

Rào cản bộ nhớ ràng buộc việc reordering và ngữ nghĩa tính nhìn thấy (visibility) giữa các truy cập bộ nhớ chỉ định, không đồng nghĩa với "xóa rỗng CPU cache" hay cưỡng chế đọc lại toàn bộ dữ liệu từ RAM. `loadFence()` chỉ cung cấp ràng buộc sắp xếp phía đọc, không thể một mình tạo ra quan hệ `happens-before` trong Java Memory Model cho 2 truy cập field thông thường.

Do đó, không thể dùng "field `boolean` thông thường + `loadFence()`" để thay thế `volatile` nhằm đảm bảo tính nhìn thấy xuyên Thread. Cờ hiệu dùng chung nên dùng `volatile`, Lock, hoặc cơ chế đồng bộ `VarHandle` có pattern truy cập đọc ghi phù hợp; nếu không code vẫn tồn tại data race, Thread đọc không đảm bảo quan sát được thao tác ghi.

#### Ứng dụng điển hình

Trong Java 8 đưa vào một cơ chế Lock mới — `StampedLock`, có thể xem như một phiên bản cải tiến của ReadWriteLock. `StampedLock` cung cấp một bản thực thi Optimistic Read Lock, loại lock đọc lạc quan này tương tự thao tác lock-free, hoàn toàn không chặn Thread ghi lấy Write Lock, từ đó giảm bớt hiện tượng Thread ghi bị "đói" khi đọc nhiều ghi ít. Do Optimistic Read Lock của `StampedLock` không chặn Thread ghi lấy Read Lock, khi biến dùng chung giữa các Thread được load từ RAM vào bộ nhớ working memory của Thread có thể xảy ra vấn đề bất đồng nhất dữ liệu.

Để giải quyết vấn đề này, phương thức `validate` của `StampedLock` sẽ thông qua phương thức `loadFence` của `Unsafe` để thêm một rào cản bộ nhớ `load`.

```java
public boolean validate(long stamp) {
   U.loadFence();
   return (stamp & SBITS) == (state & SBITS);
}
```

### Thao tác đối tượng (Object)

#### Giới thiệu

**Ví dụ**

```java
import sun.misc.Unsafe;
import java.lang.reflect.Field;

public class Main {

    private int value;

    public static void main(String[] args) throws Exception {
        Unsafe unsafe = reflectGetUnsafe();
        assert unsafe != null;
        long offset = unsafe.objectFieldOffset(Main.class.getDeclaredField("value"));
        Main main = new Main();
        System.out.println("value before putInt: " + main.value);
        unsafe.putInt(main, offset, 42);
        System.out.println("value after putInt: " + main.value);
        System.out.println("value after putInt: " + unsafe.getInt(main, offset));
    }

    private static Unsafe reflectGetUnsafe() {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            return (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

}
```

Kết quả:

```plain
value before putInt: 0
value after putInt: 42
value after putInt: 42
```

**Thuộc tính đối tượng**

Lấy offset bộ nhớ của thuộc tính thành viên đối tượng, cũng như chỉnh sửa giá trị thuộc tính field, chúng ta đã test trong ví dụ trên. Ngoài các phương thức `putInt`, `getInt` ở trên, Unsafe cung cấp đầy đủ các phương thức `put` và `get` cho 8 kiểu dữ liệu cơ bản và `Object`, và tất cả các phương thức `put` đều có thể vượt qua quyền truy cập, chỉnh sửa trực tiếp dữ liệu trong bộ nhớ. Đọc comment trong mã nguồn openJDK phát hiện, việc đọc ghi kiểu dữ liệu cơ bản và `Object` hơi khác nhau một chút, kiểu dữ liệu cơ bản là thao tác trực tiếp giá trị thuộc tính (`value`), còn thao tác `Object` thì dựa trên giá trị tham chiếu (`reference value`). Dưới đây là phương thức đọc ghi `Object`:

```java
// Lấy một tham chiếu đối tượng tại địa chỉ offset chỉ định của đối tượng
public native Object getObject(Object o, long offset);
// Ghi một tham chiếu đối tượng vào địa chỉ offset chỉ định của đối tượng
public native void putObject(Object o, long offset, Object x);
```

Ngoài đọc ghi thông thường thuộc tính đối tượng, `Unsafe` còn cung cấp phương thức **volatile read/write** và **ordered write**. Phạm vi phủ sóng của phương thức volatile read/write giống đọc ghi thông thường, bao gồm tất cả các kiểu cơ bản và `Object`, lấy kiểu `int` làm ví dụ:

```java
// Đọc một giá trị int tại địa chỉ offset chỉ định của đối tượng, hỗ trợ ngữ nghĩa volatile load
public native int getIntVolatile(Object o, long offset);
// Ghi một giá trị int vào địa chỉ offset chỉ định của đối tượng, hỗ trợ ngữ nghĩa volatile store
public native void putIntVolatile(Object o, long offset, int x);
```

So với đọc ghi thông thường, volatile read/write có chi phí cao hơn vì nó cần đảm bảo tính nhìn thấy và tính thứ tự. Khi thực thi thao tác `get`, nó sẽ ép lấy giá trị thuộc tính từ RAM, khi dùng phương thức `put` gán giá trị thuộc tính, nó sẽ ép cập nhật giá trị vào RAM, từ đó đảm bảo các thay đổi này có thể nhìn thấy đối với các Thread khác.

Các phương thức Ordered Write có 3 phương thức sau:

```java
public native void putOrderedObject(Object o, long offset, Object x);
public native void putOrderedInt(Object o, long offset, int x);
public native void putOrderedLong(Object o, long offset, long x);
```

Chi phí của Ordered Write tương đối thấp hơn `volatile`, vì nó chỉ đảm bảo tính thứ tự khi ghi chứ không đảm bảo tính nhìn thấy lập tức, tức giá trị một Thread ghi vào không đảm bảo các Thread khác nhìn thấy ngay lập tức. Để giải quyết sự khác biệt ở đây, cần bổ sung thêm kiến thức về Memory Barrier, trước tiên cần hiểu 2 khái niệm chỉ thị:

- `Load`: Copy dữ liệu từ RAM vào cache của bộ xử lý
- `Store`: Refresh dữ liệu trong cache của bộ xử lý vào RAM

Sự khác biệt giữa Ordered Write và Volatile Write ở chỗ: Loại Memory Barrier thêm vào khi Ordered Write là loại `StoreStore`, còn loại Memory Barrier thêm vào khi Volatile Write là `StoreLoad`, như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144834132.png)

Trong phương thức Ordered Write sử dụng `StoreStore` barrier, barrier này đảm bảo `Store1` refresh dữ liệu ngay vào bộ nhớ, thao tác này diễn ra trước `Store2` và các chỉ thị store phía sau. Còn trong Volatile Write sử dụng `StoreLoad` barrier, barrier này đảm bảo `Store1` refresh dữ liệu ngay vào bộ nhớ, thao tác này diễn ra trước `Load2` và các chỉ thị load phía sau, đồng thời `StoreLoad` barrier sẽ khiến tất cả các chỉ thị truy cập bộ nhớ trước barrier đó (bao gồm store và access) hoàn thành hết mới thực thi các chỉ thị truy cập bộ nhớ phía sau barrier.

Tóm lại, trong 3 loại phương thức ghi trên, về mặt hiệu suất ghi, theo thứ tự `put`, `putOrder`, `putVolatile` hiệu suất giảm dần.

**Khởi tạo đối tượng (Object Instantiation)**

Sử dụng phương thức `allocateInstance` của `Unsafe` cho phép chúng ta dùng cách phi truyền thống để khởi tạo đối tượng, trước tiên định nghĩa một entity class và gán giá trị cho thành viên trong constructor:

```java
@Data
public class A {
    private int b;
    public A() {
        this.b = 1;
    }
}
```

So sánh việc tạo đối tượng theo các cách khác nhau dựa trên Constructor, Reflection và Unsafe:

```java
public void objTest() throws Exception {
    A a1 = new A();
    System.out.println(a1.getB());
    A a2 = A.class.newInstance();
    System.out.println(a2.getB());
    A a3 = (A) unsafe.allocateInstance(A.class);
    System.out.println(a3.getB());
}
```

Kết quả in ra lần lượt là 1, 1, 0, chứng tỏ trong quá trình tạo đối tượng qua `allocateInstance` sẽ không gọi phương thức constructor của class. Khi dùng cách này tạo đối tượng chỉ cần đến đối tượng `Class`, do đó nếu muốn bỏ qua giai đoạn khởi tạo đối tượng hoặc bỏ qua kiểm tra an toàn của constructor thì có thể dùng phương thức này. Trong ví dụ trên, nếu đổi constructor của class A thành `private`, sẽ không thể tạo đối tượng qua constructor và Reflection (có thể tạo sau khi setAccessible cho constructor), nhưng phương thức `allocateInstance` vẫn có hiệu lực.

#### Ứng dụng điển hình

- **Cách khởi tạo đối tượng thông thường**: Các cách tạo đối tượng thông thường chúng ta hay dùng về bản chất đều thông qua cơ chế new. Tuy nhiên cơ chế new có đặc điểm là khi class chỉ cung cấp constructor có tham số và không khai báo constructor không tham số, thì bắt buộc phải dùng constructor có tham số để dựng đối tượng, mà khi dùng constructor có tham số thì bắt buộc phải truyền số lượng tham số tương ứng mới hoàn thành khởi tạo đối tượng.
- **Cách khởi tạo phi truyền thống**: Unsafe cung cấp phương thức `allocateInstance`, chỉ cần thông qua đối tượng Class là có thể tạo instance của class đó mà không cần gọi constructor, code khởi tạo hay kiểm tra an toàn của JVM. Nó bỏ qua kiểm tra modifier, tức ngay cả khi constructor là `private` cũng có thể khởi tạo qua phương thức này, chỉ cần cung cấp Class object là tạo được đối tượng tương ứng. Nhờ đặc tính này, `allocateInstance` có các ứng dụng tương ứng trong `java.lang.invoke`, Objenesis (cung cấp cách tạo đối tượng bỏ qua constructor), Gson (dùng khi deserialize).

### Thao tác mảng (Array)

#### Giới thiệu

Hai phương thức `arrayBaseOffset` và `arrayIndexScale` phối hợp với nhau có thể định vị vị trí của từng phần tử trong mảng trên bộ nhớ.

```java
// Trả về địa chỉ offset của phần tử đầu tiên trong mảng
public native int arrayBaseOffset(Class<?> arrayClass);
// Trả về kích thước chiếm dụng của 1 phần tử trong mảng
public native int arrayIndexScale(Class<?> arrayClass);
```

#### Ứng dụng điển hình

Hai phương thức liên quan đến thao tác dữ liệu này có ứng dụng điển hình trong `AtomicIntegerArray` thuộc package `java.util.concurrent.atomic` (có thể thực hiện thao tác nguyên tử cho từng phần tử trong mảng Integer). Như mã nguồn `AtomicIntegerArray` hình bên dưới, thông qua `arrayBaseOffset` và `arrayIndexScale` của `Unsafe` lần lượt lấy địa chỉ offset `base` của phần tử đầu tiên và hệ số kích thước `scale` của một phần tử. Các thao tác nguyên tử liên quan về sau đều dựa vào 2 giá trị này để định vị phần tử trong mảng, phương thức `getAndAdd` ở hình 2 thông qua `checkedByteOffset` lấy địa chỉ offset của một phần tử mảng, sau đó thông qua CAS thực hiện thao tác nguyên tử.

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144927257.png)

### Thao tác CAS

#### Giới thiệu

Phần này chủ yếu là các phương thức thao tác liên quan đến CAS.

```java
/**
  *  CAS
  * @param o         Đối tượng chứa field cần sửa
  * @param offset    Offset của field trong đối tượng
  * @param expected  Giá trị kỳ vọng
  * @param update    Giá trị cập nhật
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);

public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

**CAS là gì?** CAS là viết tắt của Compare And Swap (So sánh và Trao đổi), là một kỹ thuật thường dùng khi thực thi các thuật toán concurrency. Thao tác CAS chứa 3 operand — vị trí bộ nhớ, giá trị nguyên bản kỳ vọng và giá trị mới. Khi thực thi CAS, nếu giá trị ở vị trí bộ nhớ giống với giá trị kỳ vọng thì cập nhật thành giá trị mới theo cách nguyên tử (atomic), ngược lại không cập nhật. HotSpot sẽ ánh xạ các thao tác liên quan thành nguyên lý nguyên tử mà nền tảng mục tiêu cung cấp; trên x86 thường dùng `cmpxchg`, các kiến trúc CPU khác có thể dùng chỉ thị khác.

#### Ứng dụng điển hình

Trong các utility class concurrency của package JUC sử dụng rất nhiều thao tác CAS, như trong các bài viết giới thiệu `synchronized` và `AQS` trước đây cũng nhiều lần nhắc tới CAS, nó phát huy tác dụng rộng rãi làm Optimistic Lock trong các concurrency tools. Trong lớp `Unsafe` cung cấp các phương thức `compareAndSwapObject`, `compareAndSwapInt`, `compareAndSwapLong` để thực hiện thao tác CAS cho các kiểu `Object`, `int`, `long`. Lấy phương thức `compareAndSwapInt` làm ví dụ:

```java
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);
```

Trong tham số, `o` là đối tượng cần cập nhật, `offset` là offset của int field trong đối tượng `o`, nếu giá trị của field này giống `expected` thì đặt giá trị field thành giá trị mới `x`, và việc cập nhật này không thể bị ngắt (uninterruptible), tức là một thao tác nguyên tử. Dưới đây là một ví dụ sử dụng `compareAndSwapInt`:

```java
private volatile int a;
public static void main(String[] args){
    CasTest casTest=new CasTest();
    new Thread(()->{
        for (int i = 1; i < 5; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
    new Thread(()->{
        for (int i = 5 ; i <10 ; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
}

private void increment(int x){
    while (true){
        try {
            long fieldOffset = unsafe.objectFieldOffset(CasTest.class.getDeclaredField("a"));
            if (unsafe.compareAndSwapInt(this,fieldOffset,x-1,x))
                break;
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
}
```

Chạy code sẽ lần lượt in ra:

```plain
1 2 3 4 5 6 7 8 9
```

Nếu bạn dán đoạn code trên vào IDE để chạy, sẽ thấy không thu được kết quả đầu ra như mục tiêu. Có bạn trên Github đã chỉ ra vấn đề này: [issue#2650](https://github.com/Snailclimb/JavaGuide/issues/2650). Dưới đây là code đã sửa lại:

```java
// Đóng gói thao tác tăng và in vào trong một phương thức có tính nguyên tử mạnh hơn
private void incrementAndPrint(int targetValue) {
    while (true) {
        int currentValue = a; // Đọc giá trị hiện tại của a
        // Nếu giá trị hiện tại đã đạt hoặc vượt quá giá trị mục tiêu, chứng tỏ đã được Thread khác xử lý, bỏ qua
        if (currentValue >= targetValue) {
            return;
        }
        // Thử thao tác CAS: Nếu giá trị hiện tại bằng targetValue - 1, thì đặt thành targetValue một cách nguyên tử
        if (currentValue == targetValue - 1) {
          if (unsafe.compareAndSwapInt(this, fieldOffset, currentValue, targetValue)) {
              // CAS thành công lập tức in ra, đảm bảo in đúng giá trị vừa đặt
              System.out.print(targetValue + " ");
              return;
          }
        }
        // CAS thất bại, đọc lại và thử lại
    }
}
```

Trong ví dụ trên, chúng ta tạo 2 Thread, cả hai đều thử sửa đổi biến dùng chung a. Mỗi Thread khi gọi phương thức `incrementAndPrint(targetValue)`:

1. Đầu tiên đọc giá trị hiện tại `currentValue` của a.
2. Kiểm tra `currentValue` có bằng `targetValue - 1` (giá trị phía trước kỳ vọng) không.
3. Nếu điều kiện thỏa mãn, gọi `unsafe.compareAndSwapInt()` thử cập nhật `a` từ `currentValue` thành `targetValue`.
4. Nếu thao tác CAS thành công (trả về true), in `targetValue` và thoát vòng lặp.
5. Nếu thao tác CAS thất bại, chứng tỏ có Thread khác tranh chấp cùng lúc, lúc này sẽ đọc lại `currentValue` và thử lại cho đến khi thành công mới dừng.

Cơ chế này đảm bảo mỗi số (từ 1 đến 9) chỉ được đặt và in ra thành công đúng 1 lần, và theo đúng thứ tự.

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144939826.png)

Cần lưu ý:

1. **Logic tự xoay (Spinning):** Bản thân phương thức `compareAndSwapInt` chỉ thực thi 1 lần so sánh và đổi, rồi lập tức trả về kết quả. Do đó để đảm bảo thao tác cuối cùng thành công (khi giá trị phù hợp kỳ vọng), chúng ta cần chủ động thực thi logic tự xoay (như vòng lặp `while(true)`), liên tục thử cho đến khi CAS thành công.
2. **Bản thực thi `AtomicInteger`:** Trong JDK, lớp `java.util.concurrent.atomic.AtomicInteger` nội bộ chính là tận dụng thao tác CAS tương tự và logic tự xoay để thực hiện các phương thức nguyên tử `getAndIncrement()`, `compareAndSet()`,... Sử dụng trực tiếp `AtomicInteger` thông thường là cách an toàn và được khuyến nghị hơn vì nó đã đóng gói độ phức tạp bên dưới.
3. **Vấn đề ABA:** Thao tác CAS tồn tại vấn đề ABA (một giá trị từ A biến thành B, rồi biến lại thành A, khi CAS kiểm tra sẽ tưởng giá trị chưa từng đổi). Trong một số kịch bản nếu lịch sử thay đổi giá trị rất quan trọng, có thể cần dùng `AtomicStampedReference` để giải quyết. Nhưng trong kịch bản tăng đơn giản của ví dụ này, vấn đề ABA thông thường không gây ảnh hưởng.
4. **Tiêu thụ CPU:** Tự xoay thời gian dài sẽ tiêu thụ tài nguyên CPU. Trong trường hợp tranh chấp gay gắt hoặc điều kiện lâu không thỏa mãn, có thể cân nhắc thêm chiến lược lùi lại phức tạp hơn (như `Thread.sleep()` hoặc `LockSupport.parkNanos()`) để tối ưu.

### Lập lịch Thread (Thread Scheduling)

#### Giới thiệu

Trong `Unsafe` hiện tại các phương thức liên quan trực tiếp đến lập lịch Thread chủ yếu là `park` và `unpark`. Các phương thức lịch sử như `monitorEnter`, `monitorExit`, `tryMonitorEnter` đã bị xóa bỏ trong JDK 9.

```java
// Hủy block Thread
public native void unpark(Object thread);
// Block Thread
public native void park(boolean isAbsolute, long time);
```

Phương thức `park`, `unpark` có thể thực hiện việc treo (suspend) và phục hồi (resume) Thread, việc treo một Thread được thực hiện qua phương thức `park`, sau khi gọi `park`, Thread sẽ bị block cho đến khi timeout hoặc bị ngắt (interrupt); `unpark` có thể kết thúc một Thread đang treo, giúp nó phục hồi bình thường.

Ba phương thức liên quan đến `monitor` chỉ áp dụng cho việc giới thiệu bản thực thi phiên bản cũ, code JDK hiện tại không thể gọi chúng nữa. Khi cần Monitor đối tượng nên dùng câu lệnh `synchronized` của ngôn ngữ Java hoặc Lock và Synchronizer trong `java.util.concurrent`.

#### Ứng dụng điển hình

Class cốt lõi của framework Java Lock và Synchronizer là `AbstractQueuedSynchronizer` (AQS), chính là thông qua việc gọi `LockSupport.park()` và `LockSupport.unpark()` để thực hiện việc block và đánh thức Thread, mà phương thức `park`, `unpark` của `LockSupport` thực tế là gọi phương thức `park`, `unpark` của `Unsafe`.

```java
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    UNSAFE.park(false, 0L);
    setBlocker(t, null);
}
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

Phương thức `park` của `LockSupport` bên dưới sẽ gọi phương thức `park` của `Unsafe`. `park` có thể trả về do giấy phép khả dụng, Thread khác gọi `unpark`, Thread bị interrupt hoặc trả về không lý do; biến thể có timeout cũng sẽ trả về sau khi hết giờ. Do đó, logic block phụ thuộc điều kiện nên kiểm tra lại điều kiện trong vòng lặp. Ví dụ dưới đây minh họa trường hợp được Thread khác gọi `unpark`:

```java
public static void main(String[] args) {
    Thread mainThread = Thread.currentThread();
    new Thread(()->{
        try {
            TimeUnit.SECONDS.sleep(5);
            System.out.println("subThread try to unpark mainThread");
            unsafe.unpark(mainThread);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    System.out.println("park main mainThread");
    unsafe.park(false,0L);
    System.out.println("unpark mainThread success");
}
```

Chương trình in ra:

```plain
park main mainThread
subThread try to unpark mainThread
unpark mainThread success
```

Luồng chạy của chương trình khá dễ hiểu, Thread con sau khi bắt đầu chạy sẽ ngủ trước để đảm bảo Thread chính có thể gọi phương thức `park` tự block mình, Thread con sau khi ngủ 5 giây sẽ gọi phương thức `unpark` đánh thức Thread chính, giúp Thread chính có thể tiếp tục chạy xuống dưới. Toàn bộ quy trình như hình bên dưới:

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717144950116.png)

### Thao tác Class

#### Giới thiệu

Thao tác của `Unsafe` đối với `Class` chủ yếu bao gồm nạp class và các phương thức thao tác biến static.

**Các phương thức liên quan đến đọc thuộc tính static**

> Giải thích phiên bản: `shouldBeInitialized` và `ensureClassInitialized` đã bị xóa khỏi `sun.misc.Unsafe` trong JDK 22, phương thức thay thế chuẩn là `MethodHandles.Lookup.ensureInitialized` đưa vào từ JDK 15. Code liên quan bên dưới chỉ áp dụng cho các phiên bản JDK cũ hơn.

```java
// Lấy offset của thuộc tính static
public native long staticFieldOffset(Field f);
// Lấy con trỏ đối tượng của thuộc tính static
public native Object staticFieldBase(Field f);
// Kiểm tra class có cần khởi tạo hay không (dùng để kiểm tra trước khi lấy thuộc tính static của class)
public native boolean shouldBeInitialized(Class<?> c);
```

Tạo một class chứa thuộc tính static để test:

```java
@Data
public class User {
    public static String name="Hydra";
    int age;
}
private void staticTest() throws Exception {
    User user=new User();
    // Cũng có thể dùng câu lệnh bên dưới để kích hoạt khởi tạo class
    // 1.
    // unsafe.ensureClassInitialized(User.class);
    // 2.
    // System.out.println(User.name);
    System.out.println(unsafe.shouldBeInitialized(User.class));
    Field sexField = User.class.getDeclaredField("name");
    long fieldOffset = unsafe.staticFieldOffset(sexField);
    Object fieldBase = unsafe.staticFieldBase(sexField);
    Object object = unsafe.getObject(fieldBase, fieldOffset);
    System.out.println(object);
}
```

Kết quả:

```plain
false
Hydra
```

Trong thao tác đối tượng của `Unsafe`, chúng ta đã học cách lấy offset thuộc tính đối tượng qua `objectFieldOffset` và dựa vào đó để đọc ghi giá trị biến, tuy nhiên nó không áp dụng cho thuộc tính static trong class, lúc này cần dùng phương thức `staticFieldOffset`. Trong code ở trên, chỉ có trong quá trình lấy đối tượng `Field` là phụ thuộc `Class`, còn khi lấy thuộc tính biến static thì không còn phụ thuộc `Class` nữa.

Trong code ở trên đầu tiên tạo một đối tượng `User`, đó là vì nếu một class chưa được khởi tạo thì thuộc tính static của nó cũng sẽ không được khởi tạo, thuộc tính field lấy ra cuối cùng sẽ là `null`. Do đó trước khi lấy thuộc tính static cần gọi `shouldBeInitialized` để đánh giá trước khi lấy có cần khởi tạo class này không. Nếu xóa câu lệnh tạo đối tượng User, kết quả sẽ thành:

```plain
true
null
```

**Sử dụng phương thức `defineClass` cho phép chương trình tạo động một class lúc runtime**

> Giải thích phiên bản: `sun.misc.Unsafe.defineClass` đã bị xóa trong JDK 11. JDK 9 trở đi có thể dựa theo nhu cầu kiểm soát truy cập để dùng `MethodHandles.Lookup.defineClass`.

```java
public native Class<?> defineClass(String name, byte[] b, int off, int len, ClassLoader loader,ProtectionDomain protectionDomain);
```

Trong quá trình sử dụng thực tế, chỉ cần truyền mảng byte, chỉ mục byte bắt đầu và độ dài byte đọc vào, mặc định `ClassLoader` và `ProtectionDomain` đến từ instance gọi phương thức này. Ví dụ bên dưới thực hiện tính năng sau khi decompile ra file class:

```java
private static void defineTest() {
    String fileName="F:\\workspace\\unsafe-test\\target\\classes\\com\\cn\\model\\User.class";
    File file = new File(fileName);
    try(FileInputStream fis = new FileInputStream(file)) {
        byte[] content=new byte[(int)file.length()];
        fis.read(content);
        Class clazz = unsafe.defineClass(null, content, 0, content.length, null, null);
        Object o = clazz.getDeclaredConstructor().newInstance();
        Object age = clazz.getMethod("getAge").invoke(o, null);
        System.out.println(age);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

Trong đoạn code lịch sử ở trên, đầu tiên đọc một file `class` và thông qua stream chuyển nó thành mảng byte, sau đó dùng `defineClass` tạo động class và khởi tạo instance. Class được định nghĩa theo cách này vẫn phải trải qua kiểm tra định dạng class file của JVM, xác minh bytecode cũng như các ràng buộc nạp tương ứng, chứ không bỏ qua toàn bộ kiểm tra an toàn.

![](https://oss.javaguide.cn/github/javaguide/java/basis/unsafe/image-20220717145000710.png)

Phiên bản Unsafe cũ còn từng cung cấp phương thức `defineAnonymousClass`:

```java
public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
```

Phương thức này dùng để tạo động anonymous class, nhưng đã bị xóa trong JDK 17. `MethodHandles.Lookup.defineHiddenClass` đưa vào từ JDK 15 là giải pháp thay thế được hỗ trợ. Việc thực thi cụ thể của Lambda thuộc chi tiết thực thi của JDK, phiên bản hiện tại không thể mô tả là phụ thuộc vào `Unsafe.defineAnonymousClass` đã bị xóa nữa.

#### Ứng dụng điển hình

Các phiên bản JDK lịch sử từng dùng `Unsafe.defineAnonymousClass` để hỗ trợ một phần bản thực thi ngôn ngữ động; JDK hiện tại dùng Hidden Class,... không còn cung cấp phương thức Unsafe này nữa.

### Thông tin hệ thống

#### Giới thiệu

Phần này bao gồm 2 phương thức lấy thông tin liên quan đến hệ thống.

```java
// Trả về kích thước con trỏ hệ thống. Giá trị trả về là 4 (hệ thống 32-bit) hoặc 8 (hệ thống 64-bit).
public native int addressSize();
// Kích thước trang bộ nhớ (Page Size), giá trị này là lũy thừa của 2.
public native int pageSize();
```

#### Ứng dụng điển hình

Kịch bản ứng dụng của 2 phương thức này tương đối ít, trong lớp `java.nio.Bits`, khi dùng `pageCount` tính toán số lượng trang bộ nhớ cần thiết, nó đã gọi phương thức `pageSize` để lấy kích thước trang bộ nhớ. Ngoài ra khi dùng phương thức `copySwapMemory` để copy bộ nhớ, nó đã gọi phương thức `addressSize` để kiểm tra trường hợp hệ thống 32-bit.

## Tóm tắt

Trong bài viết này, chúng ta đã giới thiệu các khái niệm cơ bản, nguyên lý hoạt động và một phần API lịch sử của `Unsafe`. Cần lưu ý rằng `sun.misc.Unsafe` thuộc về API nội bộ không được hỗ trợ chính thức, nhiều phương thức đã bị xóa trong các phiên bản JDK khác nhau. JDK 23 đã đánh dấu các phương thức truy cập bộ nhớ của nó là chờ xóa bỏ, JDK 24 trở đi mặc định đưa ra cảnh báo runtime khi gọi lần đầu. Code mới nên ưu tiên dùng API chuẩn: Truy cập field và array trong Heap sử dụng `VarHandle`, truy cập bộ nhớ off-heap sử dụng Foreign Function and Memory API (`MemorySegment`,...), đồng bộ Thread sử dụng `java.util.concurrent`.

<!-- @include: @article-footer.snippet.md -->
