---
title: JVM垃圾回收详解（重点）
description: JVM垃圾回收详解：全面讲解GC算法（标记清除、复制、标记整理）、分代回收机制、常用垃圾回收器（Serial、Parallel、CMS、G1、ZGC）、GC调优实践。
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: JVM垃圾回收,GC算法,垃圾回收器,分代回收,标记清除,复制算法,G1 GC,ZGC,GC调优
---

> Nếu không có giải thích đặc biệt, bài viết đều nhắm tới HotSpot Virtual Machine.
>
> Bài viết này được tóm tắt và bổ sung dựa trên cuốn 《Sâu sắc hiểu về Java Virtual Machine: Các tính năng nâng cao của JVM và thực tiễn tốt nhất》.
>
> Các câu hỏi phỏng vấn thường gặp:
>
> - Làm thế nào để phán đoán đối tượng đã chết (2 phương pháp).
> - Giới thiệu ngắn gọn về Strong Reference, Soft Reference, Weak Reference, Phantom Reference (Sự khác biệt giữa Phantom Reference với Soft/Weak Reference, lợi ích khi dùng Soft Reference).
> - Làm thế nào để phán đoán một hằng số là hằng số bị phế bỏ.
> - Làm thế nào để phán đoán một Class là Class vô dụng.
> - Garbage Collection có những thuật toán nào, đặc điểm từng loại?
> - Tại sao HotSpot lại chia thành Young Generation và Old Generation?
> - Các Garbage Collector thường gặp là gì?
> - Giới thiệu về collector CMS, G1.
> - Minor GC và Full GC khác nhau như thế nào?

## Lời nói đầu

Khi cần định vị và khắc phục các vấn đề memory overflow, khi Garbage Collection trở thành nút thắt cổ chai để hệ thống đạt mức độ concurrency cao hơn, chúng ta cần thực thi giám sát và điều chỉnh cần thiết đối với các kỹ thuật "tự động hóa" này.

## Cấu trúc cơ bản của không gian Heap

Quản lý bộ nhớ tự động của Java chủ yếu hướng tới việc thu hồi bộ nhớ Object và cấp phát bộ nhớ Object. Đồng thời, chức năng cốt lõi nhất của quản lý bộ nhớ tự động Java là việc cấp phát và thu hồi Object trong bộ nhớ **Heap**.

Java Heap là vùng chủ yếu được quản lý bởi Garbage Collector, do đó còn được gọi là **GC Heap (Garbage Collected Heap)**.

Từ góc độ thu gom rác, vì hiện tại các collector về cơ bản đều áp dụng thuật toán phân thế Garbage Collection (Generational Garbage Collection), nên Java Heap được chia thành nhiều vùng khác nhau, nhờ đó chúng ta có thể dựa vào đặc điểm của từng vùng để chọn thuật toán Garbage Collection phù hợp.

Trong HotSpot JDK 7 và các phiên bản trước đó, GC thường được giới thiệu theo 3 phần dưới đây, trong đó Permanent Generation là triển khai của Method Area, không thuộc về Java Heap:

1. Young Generation (Bộ nhớ thế hệ trẻ)
2. Old Generation (Thế hệ già)
3. Permanent Generation (Thế hệ vĩnh cửu)

Vùng Eden, 2 vùng Survivor S0 và S1 được hiển thị ở hình dưới đều thuộc về Young Generation, tầng giữa thuộc về Old Generation, tầng dưới cùng thuộc về Permanent Generation.

![Cấu trúc bộ nhớ Heap](https://oss.javaguide.cn/github/javaguide/java/jvm/hotspot-heap-structure.png)

**JDK 8 loại bỏ PermGen (Permanent Generation), metadata của Class chuyển sang lưu trữ trong Metaspace (Vùng nguyên dữ liệu) sử dụng bộ nhớ bản địa (Native Memory)**.

Về phần giới thiệu chi tiết hơn về cấu trúc không gian Heap, bạn có thể xem lại bài viết [Giải thích chi tiết vùng bộ nhớ Java](./memory-area.md).

## Nguyên tắc cấp phát và thu hồi bộ nhớ

### Đối tượng ưu tiên cấp phát tại vùng Eden

Trong hầu hết các trường hợp, Object được cấp phát tại vùng Eden thuộc Young Generation. Khi vùng Eden không có đủ không gian để cấp phát, Virtual Machine sẽ kích hoạt một lần Minor GC. Dưới đây chúng ta tiến hành kiểm tra thực tế.

Code kiểm tra:

```java
public class GCTest {
  public static void main(String[] args) {
    byte[] allocation1, allocation2;
    allocation1 = new byte[30900*1024];
  }
}
```

Chạy bằng cách sau:
![](https://oss.javaguide.cn/github/javaguide/java/jvm/25178350.png)

Tham số thêm vào: `-XX:+PrintGCDetails`
![](https://oss.javaguide.cn/github/javaguide/java/jvm/run-with-PrintGCDetails.png)

Kết quả chạy (mô tả chữ màu đỏ bị nhầm lẫn, đúng ra phải là Permanent Generation tương ứng với JDK1.7):

![](https://oss.javaguide.cn/github/javaguide/java/jvm/28954286.jpg)

Từ hình trên chúng ta có thể thấy bộ nhớ vùng Eden hầu như đã được cấp phát hoàn toàn (ngay cả khi chương trình không làm gì, Young Generation cũng sẽ sử dụng hơn 2000k bộ nhớ).

Giả sử chúng ta lại cấp phát bộ nhớ cho `allocation2` thì điều gì sẽ xảy ra?

```java
allocation2 = new byte[900*1024];
```

![](https://oss.javaguide.cn/github/javaguide/java/jvm/28128785.jpg)

Khi cấp phát bộ nhớ cho `allocation2`, bộ nhớ vùng Eden hầu như đã được cấp phát hết.

Khi vùng Eden không có đủ không gian để cấp phát, Virtual Machine sẽ kích hoạt một lần Minor GC. Trong quá trình GC, Virtual Machine lại phát hiện `allocation1` không thể đưa vào không gian Survivor, do đó đành phải thông qua **cơ chế bảo đảm cấp phát (Handle Promotion)** để chuyển trước các Object thuộc Young Generation sang Old Generation, không gian trên Old Generation đủ để chứa `allocation1`, nên không xảy ra Full GC. Sau khi thực thi Minor GC, các Object cấp phát tiếp theo nếu có thể chứa trong Eden thì vẫn sẽ cấp phát bộ nhớ tại Eden. Có thể thực thi code sau để kiểm chứng:

```java
public class GCTest {

  public static void main(String[] args) {
    byte[] allocation1, allocation2,allocation3,allocation4,allocation5;
    allocation1 = new byte[32000*1024];
    allocation2 = new byte[1000*1024];
    allocation3 = new byte[1000*1024];
    allocation4 = new byte[1000*1024];
    allocation5 = new byte[1000*1024];
  }
}
```

### Đối tượng lớn đi trực tiếp vào Old Generation

Đối tượng lớn (Large Object) là đối tượng cần không gian bộ nhớ liên tục dung lượng lớn (ví dụ: chuỗi, mảng).

Hành vi đối tượng lớn đi trực tiếp vào Old Generation do Virtual Machine quyết định động, nó liên quan đến Garbage Collector cụ thể được sử dụng và các tham số liên quan. Đối tượng lớn đi trực tiếp vào Old Generation là một chiến lược tối ưu hóa, nhằm tránh đưa đối tượng lớn vào Young Generation, từ đó giảm tần suất và chi phí Garbage Collection ở Young Generation.

- G1 Garbage Collector coi các đối tượng có kích thước đạt hoặc vượt quá một nửa Region là Humongous Object, và cấp phát trực tiếp vào Humongous Region liên tục thuộc Old Generation. Kích thước Region có thể thiết lập qua `-XX:G1HeapRegionSize`.
- Việc các collector khác có cấp phát đối tượng lớn trực tiếp vào Old Generation hay không và trong điều kiện nào phụ thuộc vào collector cụ thể và phiên bản JDK, không thể khái quát bằng một ngưỡng chung.

### Đối tượng sống lâu sẽ đi vào Old Generation

Vì Virtual Machine áp dụng tư tưởng thu gom phân thế để quản lý bộ nhớ, nên khi thu hồi bộ nhớ phải nhận biết được những đối tượng nào nên đặt ở Young Generation, những đối tượng nào nên đặt ở Old Generation. Để làm được điều này, Virtual Machine gán cho mỗi đối tượng một bộ đếm tuổi (Age).

Trong hầu hết các trường hợp, đối tượng đầu tiên được cấp phát tại vùng Eden. Nếu đối tượng sinh ra ở Eden và sau lần Minor GC đầu tiên vẫn còn sống, đồng thời có thể chứa trong Survivor, nó sẽ được chuyển sang không gian Survivor (S0 hoặc S1), và tuổi của đối tượng được đặt thành 1 (tuổi ban đầu chuyển từ Eden -> Survivor thành 1).

Đối tượng trong Survivor mỗi lần vượt qua một lần Minor GC thì tuổi tăng thêm 1 tuổi, khi tuổi của nó đạt đến ngưỡng thăng tiến (Tenuring Threshold), sẽ được thăng tiến vào Old Generation. `-XX:MaxTenuringThreshold` được dùng để thiết lập ngưỡng tuổi thăng tiến tối đa của đối tượng, giá trị mặc định liên quan đến Garbage Collector. Ví dụ, trong JDK 8 giá trị mặc định của Parallel GC là 15, của CMS là 6; ngưỡng thăng tiến thực tế còn có thể do JVM tự điều chỉnh động.

> Sửa lỗi ([issue552](https://github.com/Snailclimb/JavaGuide/issues/552)): "Khi Hotspot duyệt qua tất cả đối tượng, nó cộng dồn kích thước chiếm dụng theo tuổi từ nhỏ đến lớn, khi dung lượng cộng dồn ở một độ tuổi nào đó vượt quá 50% vùng Survivor (giá trị mặc định là 50%, có thể thiết lập qua `-XX:TargetSurvivorRatio=percent`, xem [issue1199](https://github.com/Snailclimb/JavaGuide/issues/1199)), sẽ lấy giá trị nhỏ hơn giữa tuổi này và MaxTenuringThreshold làm ngưỡng tuổi thăng tiến mới".
>
> Trích dẫn tài liệu chính thức JDK 8: <https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html>.
>
> ![](https://oss.javaguide.cn/java-guide-blog/image-20210523201742303.png)
>
> **Code tính toán tuổi động như sau:**
>
> ```c++
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> //survivor_capacity là kích thước không gian survivor
> size_t desired_survivor_size = (size_t)((((double)survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> //Mảng sizes là kích thước đối tượng theo từng độ tuổi
> total += sizes[age];
> if (total > desired_survivor_size) {
> break;
> }
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> ...
> }
> ```
>
> Bổ sung thêm ([issue672](https://github.com/Snailclimb/JavaGuide/issues/672)): **Về phát biểu tuổi thăng tiến mặc định là 15, nguồn gốc chủ yếu từ cuốn sách 《Sâu sắc hiểu về Java Virtual Machine》.**
> Nếu bạn đọc [các tham số Virtual Machine liên quan](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html) trên trang chủ Oracle, bạn sẽ thấy ở `-XX:MaxTenuringThreshold=threshold` có ghi chú:
>
> **Sets the maximum tenuring threshold for use in adaptive GC sizing. The largest value is 15. The default value is 15 for the parallel (throughput) collector, and 6 for the CMS collector. Tuổi thăng tiến mặc định không phải lúc nào cũng là 15, điều này phân biệt theo Garbage Collector, CMS là 6.**

### Các vùng thực hiện GC chính

Thầy Zhou Zhiming trong cuốn 《Sâu sắc hiểu về Java Virtual Machine》 phiên bản 2 trang 92 có viết:

> ~~_“Old Generation GC (Major GC/Full GC), chỉ GC xảy ra ở Old Generation……”_~~

Phát biểu trên đã được sửa lại trong phiên bản 3 cuốn 《Sâu sắc hiểu về Java Virtual Machine》. Cảm ơn câu trả lời của R đại:

![Trả lời của R đại](https://oss.javaguide.cn/github/javaguide/java/jvm/rf-hotspot-vm-gc.png)

**Tóm tắt:**

Nhắm tới triển khai của HotSpot VM, GC bên trong được phân loại chính xác chỉ gồm 2 loại lớn:

Partial GC (Thu gom một phần):

- Minor GC / Young GC: Chỉ thu gom rác trên Young Generation;
- Major GC / Old GC: Chỉ thu gom rác trên Old Generation. Cần lưu ý Major GC trong một số ngữ cảnh cũng được dùng để chỉ thu gom toàn bộ Heap;
- Mixed GC: Thu gom rác trên toàn bộ Young Generation và một phần Old Generation.

Full GC (Thu gom toàn bộ Heap): Thu gom rác trên toàn bộ Java Heap; việc có đồng thời thu hồi metadata của Class trong Method Area hay không phụ thuộc vào collector, phiên bản JDK và điều kiện unload Class.

### Bảo đảm cấp phát không gian (Space Allocation Guarantee)

Bảo đảm cấp phát không gian là để đảm bảo trước khi xảy ra Minor GC, bản thân Old Generation vẫn còn đủ không gian trống để chứa tất cả các đối tượng ở Young Generation.

Mô tả về bảo đảm cấp phát không gian tại chương 3 cuốn 《Sâu sắc hiểu về Java Virtual Machine》 như sau:

> Trước JDK 6 Update 24, trước khi xảy ra Minor GC, Virtual Machine bắt buộc phải kiểm tra không gian liên tục khả dụng tối đa ở Old Generation xem có lớn hơn tổng không gian của tất cả đối tượng ở Young Generation hay không, nếu điều kiện này thành lập, thì lần Minor GC này có thể đảm bảo là an toàn. Nếu không thành lập, Virtual Machine sẽ xem giá trị thiết lập của tham số `-XX:HandlePromotionFailure` xem có cho phép thất bại bảo đảm (Handle Promotion Failure) hay không; nếu cho phép, sẽ tiếp tục kiểm tra không gian liên tục khả dụng tối đa ở Old Generation xem có lớn hơn kích thước trung bình của các đối tượng đã thăng tiến lên Old Generation qua các lần trước hay không, nếu lớn hơn, sẽ thử tiến hành một lần Minor GC, mặc dù lần Minor GC này có rủi ro; nếu nhỏ hơn, hoặc thiết lập `-XX:HandlePromotionFailure` không cho phép mạo hiểm, thì lúc này phải chuyển sang tiến hành một lần Full GC.
>
> Quy tắc sau JDK 6 Update 24 đổi thành chỉ cần không gian liên tục của Old Generation lớn hơn tổng kích thước đối tượng ở Young Generation hoặc lớn hơn kích thước trung bình của các lần thăng tiến trước đó, thì sẽ tiến hành Minor GC, nếu không sẽ tiến hành Full GC.

## Phương pháp phán đoán đối tượng đã chết

Trong Heap hầu như chứa tất cả các instance của đối tượng, bước đầu tiên trước khi thu gom rác ở Heap là phải phán đoán xem những đối tượng nào đã chết (tức là những đối tượng không thể sử dụng qua bất kỳ đường nào nữa).

### Thuật toán đếm Reference (Reference Counting)

Thêm một bộ đếm reference vào đối tượng:

- Mỗi khi có một nơi reference tới nó, bộ đếm tăng thêm 1;
- Khi reference hết hiệu lực, bộ đếm giảm đi 1;
- Bất kỳ lúc nào đối tượng có bộ đếm bằng 0 thì không thể sử dụng nữa.

**Phương pháp này triển khai đơn giản, nhưng hiện tại các Java Virtual Machine chủ đạo không chọn thuật toán này để quản lý vòng đời đối tượng, một trong những lý do quan trọng là nó không thể tự giải quyết vấn đề reference vòng (circular reference) giữa các đối tượng.**

![Circular Reference giữa các đối tượng](https://oss.javaguide.cn/github/javaguide/java/jvm/object-circular-reference.png)

Cái gọi là vấn đề reference qua lại giữa các đối tượng như đoạn code dưới đây: Ngoài việc đối tượng `objA` và `objB` reference tới nhau ra, giữa hai đối tượng này không còn bất kỳ reference nào khác. Nhưng vì chúng reference lẫn nhau, dẫn đến bộ đếm reference của chúng đều không bằng 0, thế là thuật toán đếm reference không thể thông báo cho GC collector thu hồi chúng.

```java
public class ReferenceCountingGc {
    Object instance = null;
    public static void main(String[] args) {
        ReferenceCountingGc objA = new ReferenceCountingGc();
        ReferenceCountingGc objB = new ReferenceCountingGc();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
    }
}
```

### Thuật toán phân tích tính khả đạt (Reachability Analysis)

Tư tưởng cơ bản của thuật toán này là thông qua một loạt các đối tượng gọi là **"GC Roots"** làm điểm xuất phát, từ các node này bắt đầu tìm kiếm xuống dưới, đường đi mà các node đi qua gọi là chuỗi reference (Reference Chain), khi một đối tượng không có bất kỳ chuỗi reference nào kết nối tới GC Roots, chứng tỏ đối tượng này không còn khả dụng, cần phải thu hồi.

Hình dưới `Object 6 ~ Object 10` mặc dù có quan hệ reference với nhau, nhưng chúng không thể tới được GC Roots, do đó là các đối tượng cần thu hồi.

![Thuật toán phân tích tính khả đạt](https://oss.javaguide.cn/github/javaguide/java/jvm/jvm-gc-roots.png)

**Những đối tượng nào có thể làm GC Roots?**

- Đối tượng được reference trong Java Virtual Machine Stack (bảng biến cục bộ trong Stack Frame)
- Đối tượng được reference trong Native Method Stack (phương thức Native)
- Đối tượng được reference bởi thuộc tính static của Class trong Method Area
- Đối tượng được reference bởi hằng số (Constant) trong Method Area
- Tất cả các đối tượng đang được giữ bởi khóa đồng bộ (Synchronization Lock)
- Đối tượng được reference bởi JNI (Java Native Interface)

**Đối tượng có thể bị thu hồi, có đồng nghĩa với việc nhất định sẽ bị thu hồi không?**

Đối với đối tượng override phương thức `finalize()` và phương thức đó chưa từng được thực thi, HotSpot sau khi xác nhận đối tượng không thể tới được (unreachable), có thể sẽ thêm final reference tương ứng vào hàng chờ xử lý, do thread Finalizer xử lý bất đồng bộ. Nếu đối tượng trong phương thức `finalize()` tái thiết lập liên kết với đối tượng trên chuỗi reference, nó có thể tạm thời thoát khỏi việc thu hồi; nếu không, lần thu gom rác tiếp theo sẽ xác nhận lại trạng thái có thể thu hồi của nó. Quá trình này thường được khái quát là "hai lần đánh dấu", nhưng không đại diện cho việc Garbage Collector sẽ đồng bộ chờ phương thức `finalize()` thực thi, cũng không đảm bảo phương thức đó nhất định sẽ được gọi.

> Phương thức `finalize` trong lớp `Object` luôn bị coi là một thiết kế tồi, trở thành gánh nặng cho ngôn ngữ Java, ảnh hưởng đến an toàn và hiệu năng GC của Java. `Object.finalize()` bị đánh dấu deprecated từ JDK 9, JEP 421 lại đánh dấu cơ chế finalization chờ loại bỏ trong JDK 18. Code mới không nên phụ thuộc vào nó.
>
> Tham khảo:
>
> - [JEP 421: Deprecate Finalization for Removal](https://openjdk.java.net/jeps/421)
> - [Đã đến lúc quên đi phương thức finalize](https://mp.weixin.qq.com/s/LW-paZAMD08DP_3-XCUxmg)

### Tóm tắt các loại Reference

Dù thông qua thuật toán đếm reference để phán đoán số lượng reference đối tượng, hay thông qua phân tích tính khả đạt để phán đoán chuỗi reference đối tượng có tới được hay không, việc xác định sự sống chết của đối tượng đều liên quan đến "reference".

Trước JDK 1.2, định nghĩa reference trong Java rất truyền thống: Nếu dữ liệu kiểu reference lưu trữ giá trị đại diện cho địa chỉ bắt đầu của một khối bộ nhớ khác, thì gọi khối bộ nhớ đó đại diện cho một reference.

Từ JDK 1.2 trở đi, Java đã mở rộng khái niệm reference, chia reference thành 4 loại: Strong Reference, Soft Reference, Weak Reference, Phantom Reference (cường độ reference giảm dần). Strong Reference là việc gán reference thông thường phổ biến trong code, các lớp định nghĩa Soft Reference, Weak Reference, Phantom Reference trong JDK lần lượt là `SoftReference`, `WeakReference`, `PhantomReference`.

![Tóm tắt các loại reference trong Java](https://oss.javaguide.cn/github/javaguide/java/jvm/java-reference-type.png)

**1. Strong Reference (Reference mạnh)**

Strong Reference thực tế là việc gán reference phổ biến trong code chương trình, đây là loại reference được sử dụng phổ biến nhất, code như sau:

```java
String strongReference = new String("abc");
```

Nếu một đối tượng vẫn có thể truy cập qua Strong Reference, nó tương tự như **đồ dùng sinh hoạt thiết yếu**, Garbage Collector sẽ không thu hồi nó. Khi không gian bộ nhớ không đủ, Java Virtual Machine thà ném lỗi `OutOfMemoryError` làm chương trình chấm dứt bất thường, chứ không tùy tiện thu hồi đối tượng có thể tới được qua Strong Reference để giải quyết vấn đề thiếu bộ nhớ.

**2. Soft Reference (Reference mềm)**

Nếu một đối tượng chỉ có Soft Reference, nó tương tự như **đồ dùng sinh hoạt có cũng được không có cũng không sao**. Code Soft Reference như sau:

```java
// --- Ví dụ 1 ---
String str = new String("abc");
SoftReference<String> softReference1 = new SoftReference<>(str);
str = null; // Bỏ Strong Reference

// --- Ví dụ 2 ---
SoftReference<String> softReference2 = new SoftReference<>(new String("def")); // Đối tượng ẩn danh
```

Đối tượng Soft Reference khi áp lực bộ nhớ lớn có thể bị thu hồi, nhưng JVM không đảm bảo chỉ dọn dẹp khi không đủ bộ nhớ. Đảm bảo duy nhất là: Trước khi ném OutOfMemoryError, tất cả các đối tượng chỉ có thể tới được qua Soft Reference nhất định sẽ bị dọn dẹp. Miễn là Garbage Collector chưa thu hồi nó, đối tượng đó có thể được chương trình sử dụng. Soft Reference có thể dùng để triển khai cache nhạy cảm với bộ nhớ.

Soft Reference có thể kết hợp sử dụng với hàng chờ ReferenceQueue. Sau khi Garbage Collector xóa Soft Reference, sẽ đưa Soft Reference đã đăng ký ReferenceQueue vào hàng chờ tương ứng cùng lúc hoặc sau đó. Việc reference vào hàng chờ thể hiện Garbage Collector đã phát hiện sự thay đổi tính khả đạt tương ứng, không được dùng để chứng minh bộ nhớ đối tượng chiếm dụng đã hoàn thành giải phóng vật lý.

**3. Weak Reference (Reference yếu)**

Nếu một đối tượng chỉ có Weak Reference, nó tương tự như **đồ dùng sinh hoạt có cũng được không có cũng không sao**. Code Weak Reference như sau:

```java
// --- Ví dụ 1 ---
String str = new String("abc");
WeakReference<String> weakReference1 = new WeakReference<>(str);
str = null; // Bỏ Strong Reference

// --- Ví dụ 2 ---
WeakReference<String> weakReference2 = new WeakReference<>(new String("abc")); // Đối tượng ẩn danh
```

Điểm khác biệt giữa Weak Reference và Soft Reference là: Đối tượng chỉ có Weak Reference sở hữu vòng đời ngắn hơn. Khi Garbage Collector xác định một đối tượng chỉ có thể tới được qua Weak Reference, nó sẽ xóa nguyên tử Weak Reference trỏ tới đối tượng đó. Tuy nhiên, hành động xóa phải chờ đến khi Garbage Collection xảy ra mới thực thi, do đó không đảm bảo đối tượng sau khi trở thành weak-reachable sẽ lập tức bị thu hồi.

Weak Reference có thể kết hợp sử dụng với hàng chờ ReferenceQueue. Sau khi Garbage Collector xóa Weak Reference, sẽ đưa Weak Reference đã đăng ký ReferenceQueue vào hàng chờ tương ứng cùng lúc hoặc sau đó.

**4. Phantom Reference (Reference ảo)**

"Phantom Reference" đúng như tên gọi, chỉ mang tính hình thức, khác với các loại reference khác, Phantom Reference sẽ không ngăn cản Garbage Collector thu hồi đối tượng mà nó trỏ tới. Code Phantom Reference như sau:

```java
// --- Ví dụ 1 ---
String str = new String("abc");
ReferenceQueue queue = new ReferenceQueue();
// Tạo Phantom Reference, bắt buộc phải liên kết với một ReferenceQueue
PhantomReference phantomReference1 = new PhantomReference(str, queue);
str = null; // Bỏ Strong Reference

// --- Ví dụ 2 ---
PhantomReference phantomReference2 = new PhantomReference(new String("abc"), queue); // Đối tượng ẩn danh
```

**Phantom Reference chủ yếu được dùng để nhận thông báo khi tính khả đạt của đối tượng thay đổi, và phối hợp với ReferenceQueue để sắp xếp công việc dọn dẹp**.

**Điểm khác biệt giữa Phantom Reference với Soft/Weak Reference là:** Phantom Reference thường được sử dụng kết hợp với hàng chờ ReferenceQueue. Sau khi Garbage Collector xác định đối tượng đi vào trạng thái phantom-reachable, sẽ xóa Phantom Reference liên quan, và đưa Phantom Reference đã đăng ký vào hàng chờ cùng lúc hoặc sau đó. `PhantomReference.get()` luôn trả về `null`, chương trình không thể lấy lại đối tượng thông qua Phantom Reference; nó chủ yếu được dùng để sắp xếp công việc dọn dẹp sau khi đối tượng không thể truy cập được nữa.

Đặc biệt lưu ý, Soft Reference chủ yếu dùng để triển khai cache nhạy cảm với bộ nhớ, nhưng nó không đảm bảo ứng dụng không xảy ra `OutOfMemoryError`; memory overflow còn có thể do cạn kiệt tài nguyên bên ngoài Heap v.v. gây ra.

### Làm thế nào để phán đoán một hằng số là hằng số bị phế bỏ?

String Constant Pool chủ yếu thu hồi các hằng số bị phế bỏ. Vậy làm thế nào để phán đoán một hằng số là hằng số bị phế bỏ?

~~**JVM từ JDK 1.7 trở đi đã chuyển Runtime Constant Pool ra khỏi Method Area, mở một vùng trong Java Heap để chứa Runtime Constant Pool.**~~

> **🐛 Sửa lỗi (Xem thêm: [issue747](https://github.com/Snailclimb/JavaGuide/issues/747), [reference](https://blog.csdn.net/q5706503/article/details/84640762))**:
>
> 1. **Trước JDK 1.7, Runtime Constant Pool về mặt logic bao gồm String Constant Pool nằm trong Method Area, lúc này triển khai Method Area của HotSpot Virtual Machine là Permanent Generation**
> 2. **Trong JDK 1.7, String Constant Pool được tách từ Method Area ra đưa vào Heap, ở đây không đề cập đến Runtime Constant Pool, nghĩa là String Constant Pool được tách riêng ra Heap, những thứ còn lại của Runtime Constant Pool vẫn nằm trong Method Area, tức là Permanent Generation trong HotSpot**.
> 3. **Trong JDK 1.8 HotSpot loại bỏ Permanent Generation thay bằng Metaspace, lúc này String Constant Pool vẫn ở Heap, Runtime Constant Pool vẫn ở Method Area, chỉ có điều triển khai Method Area chuyển từ Permanent Generation sang Metaspace**

Giả sử trong String Constant Pool tồn tại chuỗi "abc", nếu hiện tại không có bất kỳ đối tượng String nào reference tới chuỗi hằng số này, thì chứng tỏ hằng số "abc" là hằng số bị phế bỏ, nếu lúc này xảy ra thu hồi bộ nhớ và có cần thiết, "abc" sẽ bị hệ thống dọn dẹp khỏi Constant Pool.

### Làm thế nào để phán đoán một Class là Class vô dụng?

Method Area chủ yếu thu hồi các Class vô dụng, vậy làm sao phán đoán một Class là Class vô dụng?

Việc phán đoán một hằng số có phải là "hằng số bị phế bỏ" tương đối đơn giản, còn điều kiện để phán đoán một Class có phải là "Class vô dụng" thì hà khắc hơn nhiều. Class cần đồng thời thỏa mãn 3 điều kiện dưới đây mới được coi là **"Class vô dụng"**:

- Tất cả các instance của Class đó đã bị thu hồi, tức là trong Java Heap không tồn tại bất kỳ instance nào của Class đó.
- `ClassLoader` load Class đó đã bị thu hồi.
- Đối tượng `java.lang.Class` tương ứng với Class đó không được reference ở bất kỳ đâu, không thể truy cập phương thức của Class đó qua reflection ở bất kỳ đâu.

Virtual Machine có thể tiến hành thu hồi đối với các Class vô dụng thỏa mãn 3 điều kiện trên, ở đây chỉ nói là "có thể", chứ không phải giống đối tượng không sử dụng nữa thì nhất định sẽ bị thu hồi.

## Thuật toán Garbage Collection

### Thuật toán Mark-Sweep (Đánh dấu - Xóa)

Thuật toán Mark-Sweep (Mark-and-Sweep) chia thành hai giai đoạn "Mark (Đánh dấu)" và "Sweep (Xóa)": Trước tiên đánh dấu tất cả các đối tượng không cần thu hồi, sau khi đánh dấu hoàn tất sẽ thu hồi đồng loạt tất cả các đối tượng không được đánh dấu.

Đây là thuật toán thu gom cơ sở nhất, các thuật toán sau này đều được cải tiến dựa trên những hạn chế của nó. Thuật toán Garbage Collection này mang lại hai vấn đề rõ rệt:

1. **Vấn đề hiệu năng**: Cả hai quá trình đánh dấu và xóa hiệu năng đều không cao.
2. **Vấn đề không gian**: Sau khi đánh dấu xóa sẽ tạo ra lượng lớn mảnh vụn bộ nhớ không liên tục.

![Thuật toán Mark-Sweep](https://oss.javaguide.cn/github/javaguide/java/jvm/mark-and-sweep-garbage-collection-algorithm.png)

Về việc cụ thể là đánh dấu đối tượng có thể thu hồi (unreachable object) hay đối tượng không thể thu hồi (reachable object), ý kiến trái chiều rất nhiều, cả hai cách nói thực ra đều không có vấn đề gì, cá nhân mình nghiêng về cách thứ hai hơn.

Nếu hiểu theo cách thứ nhất, toàn bộ quá trình Mark-Sweep đại thể như sau:

1. Khi một đối tượng được tạo ra, gán một bit đánh dấu, giả sử là 0 (false);
2. Trong giai đoạn Mark, chúng ta đặt bit đánh dấu của tất cả các đối tượng tới được (hoặc đối tượng user có thể reference) thành 1 (true);
3. Giai đoạn Sweep sẽ dọn dẹp các đối tượng có bit đánh dấu là 0 (false).

### Thuật toán Copying (Sao chép)

Để giải quyết vấn đề hiệu năng và mảnh vụn bộ nhớ của thuật toán Mark-Sweep, thuật toán Copying xuất hiện. Nó có thể chia bộ nhớ thành hai phần có kích thước bằng nhau, mỗi lần sử dụng một phần. Khi bộ nhớ của phần này dùng hết, sẽ sao chép các đối tượng còn sống sang phần còn lại, sau đó dọn dẹp toàn bộ không gian đã sử dụng một lần. Làm như vậy giúp mỗi lần thu hồi bộ nhớ đều thực hiện trên một nửa khoảng bộ nhớ.

![Thuật toán Copying](https://oss.javaguide.cn/github/javaguide/java/jvm/copying-garbage-collection-algorithm.png)

Mặc dù cải tiến thuật toán Mark-Sweep, nhưng vẫn tồn tại các vấn đề sau:

- **Bộ nhớ khả dụng giảm**: Bộ nhớ khả dụng giảm xuống còn một nửa ban đầu.
- **Không phù hợp với Old Generation**: Nếu số lượng đối tượng sống tương đối lớn, hiệu năng sao chép sẽ trở nên rất kém.

### Thuật toán Mark-Compact (Đánh dấu - Dồn)

Thuật toán Mark-Compact (Mark-and-Compact) là một thuật toán đánh dấu đưa ra dựa trên đặc điểm của Old Generation, quá trình đánh dấu vẫn giống như thuật toán "Mark-Sweep", nhưng các bước sau không phải trực tiếp thu hồi đối tượng thu gom, mà là để tất cả các đối tượng còn sống di chuyển về một phía, sau đó dọn dẹp trực tiếp bộ nhớ bên ngoài ranh giới.

![Thuật toán Mark-Compact](https://oss.javaguide.cn/github/javaguide/java/jvm/mark-and-compact-garbage-collection-algorithm.png)

Vì có thêm bước dồn bộ nhớ (compact), nên hiệu năng cũng không cao, phù hợp với kịch bản Old Generation có tần suất Garbage Collection không quá cao.

### Thuật toán Generational Collection (Thu gom phân thế)

Các Garbage Collector phân thế kinh điển sẽ chia bộ nhớ thành các phần dựa trên chu kỳ sống khác nhau của đối tượng. Thông thường chia Java Heap thành Young Generation và Old Generation, như vậy có thể dựa trên đặc điểm của từng thế hệ để chọn thuật toán Garbage Collection phù hợp. Cần lưu ý, Garbage Collector không phải lúc nào cũng áp dụng thiết kế phân thế, ví dụ ZGC bản đầu là collector phi phân thế.

Ví dụ trong Young Generation, mỗi lần thu gom đều có lượng lớn đối tượng chết đi, nên có thể chọn thuật toán "Copying", chỉ cần bỏ ra chi phí sao chép một lượng nhỏ đối tượng là có thể hoàn thành mỗi lần Garbage Collection. Còn tỷ lệ đối tượng sống ở Old Generation tương đối cao, và không có không gian bổ sung để bảo đảm cấp phát cho nó, do đó chúng ta bắt buộc phải chọn thuật toán "Mark-Sweep" hoặc "Mark-Compact" để Garbage Collection.

**Mở rộng câu hỏi phỏng vấn:** Tại sao HotSpot lại chia thành Young Generation và Old Generation?

Trả lời dựa trên phần giới thiệu về thuật toán Generational Collection ở trên.

## Garbage Collector (Bộ thu gom rác)

**Nếu nói thuật toán thu gom là phương pháp luận của thu hồi bộ nhớ, thì Garbage Collector chính là triển khai cụ thể của thu hồi bộ nhớ.**

Mặc dù chúng ta so sánh các collector với nhau, nhưng không phải để chọn ra một collector tốt nhất. Vì tính đến nay vẫn chưa xuất hiện Garbage Collector tốt nhất, càng không có Garbage Collector vạn năng, **những gì chúng ta có thể làm là dựa trên kịch bản ứng dụng cụ thể để chọn Garbage Collector phù hợp với mình**. Dùng thử suy nghĩ: Nếu tồn tại một collector hoàn hảo áp dụng cho mọi kịch bản trên đời, thì HotSpot Virtual Machine của chúng ta đã không triển khai nhiều Garbage Collector khác nhau đến thế.

Garbage Collector mặc định của Oracle/OpenJDK HotSpot trong môi trường Server VM điển hình (việc lựa chọn thực tế còn chịu ảnh hưởng bởi nền tảng và môi trường chạy, có thể dùng lệnh `java -XX:+PrintCommandLineFlags -version` để xem):

- JDK 8: Parallel Scavenge (Young Generation) + Parallel Old (Old Generation)
- JDK 9 trở đi: G1

### Serial Collector

Serial Collector là Garbage Collector cơ bản nhất và có lịch sử lâu đời nhất. Nhìn vào tên gọi mọi người cũng biết collector này là một collector đơn luồng. Ý nghĩa **"Đơn luồng"** của nó không chỉ đơn thuần là nó chỉ sử dụng một GC thread để hoàn thành công việc thu gom rác, mà điều quan trọng hơn là khi nó tiến hành công việc thu gom rác, bắt buộc phải tạm dừng tất cả các thread làm việc khác (**"Stop The World"**), cho đến khi nó thu gom xong.

**Young Generation áp dụng thuật toán Mark-Copy, Old Generation áp dụng thuật toán Mark-Compact.**

![Serial Collector](https://oss.javaguide.cn/github/javaguide/java/jvm/serial-garbage-collector.png)

Các nhà thiết kế Virtual Machine dĩ nhiên biết trải nghiệm người dùng không tốt do Stop The World mang lại, nên trong thiết kế các Garbage Collector sau này thời gian tạm dừng liên tục được rút ngắn (vẫn còn tạm dừng, quá trình tìm kiếm Garbage Collector xuất sắc nhất vẫn đang tiếp diễn).

Nhưng Serial Collector có điểm nào ưu việt hơn các Garbage Collector khác không? Dĩ nhiên là có, nó **đơn giản và hiệu quả (so với đơn luồng của các collector khác)**. Serial Collector do không có chi phí tương tác giữa các thread, tự nhiên có thể đạt được hiệu quả thu gom đơn luồng rất cao. Serial Collector đối với Virtual Machine chạy ở chế độ Client là một lựa chọn không tồi.

### ParNew Collector

ParNew Collector thực ra chính là phiên bản đa luồng của Serial Collector, ngoài việc sử dụng đa luồng để Garbage Collection ra, các hành vi còn lại (tham số điều khiển, thuật toán thu gom, chiến lược thu hồi v.v.) hoàn toàn giống Serial Collector.

ParNew chỉ chịu trách nhiệm ở Young Generation, áp dụng thuật toán Mark-Copy; nó thường phối hợp sử dụng với CMS Collector chịu trách nhiệm ở Old Generation.

![ParNew Collector](https://oss.javaguide.cn/github/javaguide/java/jvm/parnew-garbage-collector.png)

Trong các phiên bản JDK còn hỗ trợ CMS, ParNew là đối tác ở Young Generation của CMS; CMS đã bị gỡ bỏ trong JDK 14, do đó cặp kết hợp này chỉ áp dụng cho HotSpot phiên bản cũ.

**Bổ sung khái niệm Song song (Parallel) và Đồng thời (Concurrent):**

- **Parallel (Song song)**: Chỉ nhiều GC thread hoạt động song song cùng lúc, nhưng lúc này User thread vẫn ở trạng thái chờ.

- **Concurrent (Đồng thời)**: Chỉ User thread và GC thread cùng thực thi đồng thời (nhưng không nhất thiết phải song song, có thể thực thi xem kẽ), chương trình người dùng tiếp tục chạy, còn Garbage Collector chạy trên một CPU khác.

### Parallel Scavenge Collector

Parallel Scavenge Collector cũng là collector đa luồng sử dụng thuật toán Mark-Copy, nó nhìn qua hầu như giống hệt ParNew. **Vậy nó có điểm gì đặc biệt?**

```bash
-XX:+UseParallelGC

    Sử dụng Parallel Collector (Mặc định kết hợp với Parallel Old trong JDK 8)

-XX:+UseParallelOldGC

    Sử dụng Parallel Collector + Old Generation song song
```

Điểm tập trung của Parallel Scavenge Collector là Throughput (tối ưu hóa việc tận dụng CPU hiệu quả). Điểm tập trung của các Garbage Collector như CMS nghiêng nhiều hơn về thời gian tạm dừng của User thread (nâng cao trải nghiệm người dùng). Cái gọi là Throughput là tỷ lệ giữa thời gian CPU chạy code người dùng và tổng thời gian tiêu tốn của CPU. Parallel Scavenge Collector cung cấp rất nhiều tham số cho người dùng tìm thời gian tạm dừng phù hợp nhất hoặc Throughput lớn nhất, nếu đối với việc vận hành collector chưa hiểu rõ, gặp khó khăn khi tối ưu hóa thủ công, thì việc sử dụng Parallel Scavenge Collector phối hợp chiến lược tự điều chỉnh, giao việc tối ưu quản lý bộ nhớ cho Virtual Machine hoàn thành cũng là một lựa chọn không tồi.

**Young Generation áp dụng thuật toán Mark-Copy, Old Generation áp dụng thuật toán Mark-Compact.**

![Sơ đồ chạy Parallel Old Collector](https://oss.javaguide.cn/github/javaguide/java/jvm/parallel-scavenge-garbage-collector.png)

**Đây là Collector mặc định của JDK 1.8**

Sử dụng lệnh `java -XX:+PrintCommandLineFlags -version` để xem:

```bash
-XX:InitialHeapSize=262921408 -XX:MaxHeapSize=4206742528 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```

JDK 1.8 mặc định sử dụng Parallel Scavenge + Parallel Old, nếu chỉ định tham số `-XX:+UseParallelGC`, thì mặc định chỉ định `-XX:+UseParallelOldGC`, có thể dùng `-XX:-UseParallelOldGC` để vô hiệu hóa tính năng đó.

### Serial Old Collector

**Phiên bản Old Generation của Serial Collector**, nó cũng là một collector đơn luồng. Nó chủ yếu có 2 công dụng lớn: Một công dụng là trong phiên bản JDK 1.5 và trước đó kết hợp sử dụng với Parallel Scavenge Collector, công dụng khác là làm phương án dự phòng cho CMS Collector.

![Serial Collector](https://oss.javaguide.cn/github/javaguide/java/jvm/serial-garbage-collector.png)

### Parallel Old Collector

**Phiên bản Old Generation của Parallel Scavenge Collector**. Sử dụng đa luồng và thuật toán "Mark-Compact". Ở những trường hợp coi trọng Throughput và tài nguyên CPU, đều có thể ưu tiên cân nhắc Parallel Scavenge Collector và Parallel Old Collector.

![Sơ đồ chạy Parallel Old Collector](https://oss.javaguide.cn/github/javaguide/java/jvm/parallel-scavenge-garbage-collector.png)

### CMS Collector

**CMS (Concurrent Mark Sweep) Collector là một collector với mục tiêu đạt được thời gian tạm dừng thu hồi ngắn nhất. Nó rất phù hợp sử dụng trên các ứng dụng coi trọng trải nghiệm người dùng.**

**CMS (Concurrent Mark Sweep) Collector là collector đồng thời đúng nghĩa đầu tiên của HotSpot Virtual Machine, lần đầu tiên nó thực hiện việc cho phép GC thread và User thread làm việc đồng thời (về cơ bản).**

Từ hai từ **Mark Sweep** trong tên gọi có thể thấy, CMS Collector là một collector triển khai bằng **thuật toán "Mark-Sweep"**, quá trình vận hành của nó so với vài loại Garbage Collector trước đó thì phức tạp hơn. Toàn bộ quá trình chia làm 4 bước:

- **Initial Mark (Đánh dấu ban đầu):** Tạm dừng ngắn (STW), đánh dấu các đối tượng trực tiếp kết nối với root (đối tượng gốc);
- **Concurrent Mark (Đánh dấu đồng thời):** Đồng thời bật GC và User thread, dùng một cấu trúc closure để ghi lại các đối tượng tới được. Nhưng khi giai đoạn này kết thúc, cấu trúc closure này không thể đảm bảo chứa tất cả các đối tượng tới được hiện tại. Vì User thread có thể liên tục cập nhật miền reference, nên GC thread không thể đảm bảo tính thời gian thực của phân thích tính khả đạt. Do đó trong thuật toán này sẽ theo dõi ghi lại những nơi xảy ra cập nhật reference.
- **Remark (Đánh dấu lại):** Giai đoạn Remark chính là để sửa đổi bản ghi đánh dấu của phần đối tượng bị thay đổi đánh dấu do chương trình người dùng tiếp tục chạy trong giai đoạn Concurrent Mark, thời gian tạm dừng ở giai đoạn này thông thường dài hơn giai đoạn Initial Mark một chút, nhưng ngắn hơn nhiều so với giai đoạn Concurrent Mark.
- **Concurrent Sweep (Xóa đồng thời):** Bật User thread, đồng thời GC thread bắt đầu dọn dẹp các vùng chưa được đánh dấu.

![CMS Collector](https://oss.javaguide.cn/github/javaguide/java/jvm/cms-garbage-collector.png)

Từ tên gọi của nó có thể thấy nó là một Garbage Collector xuất sắc, ưu điểm chính: **Thu gom đồng thời, tạm dừng thấp**. Tuy nhiên nó có 3 nhược điểm rõ rệt dưới đây:

- **Nhạy cảm với tài nguyên CPU;**
- **Không thể xử lý Floating Garbage (rác nổi);**
- **Thuật toán thu hồi nó sử dụng là thuật toán "Mark-Sweep" dẫn đến sau khi thu gom kết thúc sẽ có lượng lớn mảnh vụn không gian được tạo ra.**

**CMS Garbage Collector trong Java 9 đã bị đánh dấu là quá thời (deprecated), và bị loại bỏ trong Java 14.**

### G1 Collector

**G1 (Garbage-First) là một Garbage Collector hướng tới máy chủ, chủ yếu nhắm tới các máy trang bị nhiều CPU và dung lượng bộ nhớ lớn. Với xác suất cực cao thỏa mãn yêu cầu thời gian tạm dừng GC đồng thời sở hữu đặc trưng hiệu năng Throughput cao.**

Được coi là một đặc trưng tiến hóa quan trọng của HotSpot Virtual Machine trong JDK 1.7. Nó sở hữu các đặc điểm sau:

- **Song song và Đồng thời**: G1 có thể tận dụng tối đa ưu thế phần cứng trong môi trường CPU, đa nhân, sử dụng nhiều CPU (CPU hoặc CPU core) để rút ngắn thời gian tạm dừng Stop-The-World. Một số hành động GC của các collector khác vốn cần tạm dừng Java thread thực thi, G1 Collector vẫn có thể cho phép chương trình Java tiếp tục thực thi thông qua phương thức đồng thời.
- **Thu gom phân thế**: Mặc dù G1 có thể không cần các collector khác phối hợp vẫn có thể độc lập quản lý toàn bộ GC Heap, nhưng vẫn giữ lại khái niệm phân thế.
- **Gom nhóm không gian**: Khác với thuật toán "Mark-Sweep" của CMS, G1 nhìn từ tổng thể là collector triển khai dựa trên thuật toán "Mark-Compact"; nhìn từ cục bộ là triển khai dựa trên thuật toán "Mark-Copy".
- **Tạm dừng có thể dự đoán**: Đây là một ưu thế lớn khác của G1 so với CMS, giảm thời gian tạm dừng là điểm tập trung chung của G1 và CMS. G1 sẽ dựa trên mục tiêu thời gian tạm dừng do người dùng thiết lập để xây dựng mô hình dự đoán và chọn tập hợp thu hồi, nhưng mục tiêu này là mục tiêu mềm (soft target), không đảm bảo mỗi lần tạm dừng đều không vượt quá giá trị chỉ định.

Quá trình vận hành của G1 Collector đại thể chia thành các bước sau:

- **Initial Mark**: Tạm dừng ngắn (STW), đánh dấu các đối tượng trực tiếp reference từ GC Roots, tức đánh dấu tất cả các đối tượng còn sống trực tiếp tới được
- **Concurrent Mark**: Chạy đồng thời với ứng dụng, đánh dấu tất cả các đối tượng tới được. Giai đoạn này có thể kéo dài thời gian tương đối dài, phụ thuộc vào kích thước Heap và số lượng đối tượng.
- **Final Mark**: Tạm dừng ngắn (STW), xử lý một lượng nhỏ thay đổi reference còn sót lại chưa xử lý sau khi giai đoạn Concurrent Mark kết thúc.
- **Live Data Counting and Evacuation (Lọc thu hồi)**: Dựa trên kết quả đánh dấu, chọn vùng có giá trị thu hồi cao, sao chép đối tượng còn sống sang vùng mới, thu hồi bộ nhớ vùng cũ. Giai đoạn này chứa một hoặc nhiều lần tạm dừng (STW), cụ thể phụ thuộc vào độ phức tạp của việc thu hồi.

![G1 Collector](https://oss.javaguide.cn/github/javaguide/java/jvm/g1-garbage-collector.png)

**G1 Collector duy trì một danh sách ưu tiên ở background, mỗi lần dựa trên thời gian thu gom cho phép, ưu tiên chọn Region có giá trị thu hồi lớn nhất (đây cũng chính là nguồn gốc tên gọi Garbage-First của nó)**. Cách thức sử dụng Region chia nhỏ không gian bộ nhớ và thu hồi vùng có ưu tiên này đảm bảo G1 Collector trong thời gian hữu hạn có thể đạt hiệu quả thu gom cao nhất có thể (chia nhỏ bộ nhớ).

**Từ JDK 9 trở đi, G1 Garbage Collector trở thành Garbage Collector mặc định.**

### ZGC Collector

Tương tự ParNew và G1, ZGC cũng áp dụng thuật toán Mark-Copy, tuy nhiên ZGC đã thực hiện cải tiến trọng đại đối với thuật toán này.

ZGC có thể kiểm soát thời gian tạm dừng trong vòng vài mili giây, và thời gian tạm dừng không bị ảnh hưởng bởi kích thước bộ nhớ Heap, trường hợp xuất hiện Stop The World sẽ ít hơn rất nhiều, nhưng cái giá phải trả là hy sinh một chút Throughput. ZGC hỗ trợ bộ nhớ Heap tối đa lên tới 16TB.

ZGC được giới thiệu trong Java 11, ở giai đoạn thử nghiệm. Qua nhiều phiên bản lặp lại, không ngừng hoàn thiện và sửa lỗi, ZGC trong Java 15 đã có thể chính thức sử dụng.

Tuy nhiên, Garbage Collector mặc định vẫn là G1. Bạn có thể bật ZGC thông qua tham số sau:

```bash
java -XX:+UseZGC className
```

Java 21 giới thiệu ZGC phân thế (Generational ZGC). Từ Java 23 trở đi chế độ phân thế trở thành chế độ mặc định của ZGC, Java 24 lại loại bỏ chế độ phi phân thế.

Bạn có thể bật Generational ZGC qua tham số sau:

```bash
java -XX:+UseZGC className
```

Trong Java 21, 22 có thể sử dụng thêm `-XX:+ZGenerational` để bật chế độ phân thế; tham số này trong Java 24 đã bị quá thời.

Về phần giới thiệu chi tiết về ZGC Collector khuyến nghị nên xem vài bài viết này:

- [Phân tích ZGC từ góc độ thuật toán GC qua các thời kỳ - Kinh Đông Technology](https://mp.weixin.qq.com/s/ExkB40cq1_Z0ooDzXn7CVw)
- [Khám phá và thực tiễn của Garbage Collector thế hệ mới ZGC - Meituan Technical Team](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)
- [Bát cổ văn cực hạn: Giải thích chi tiết JVM Garbage Collector G1 & ZGC - Alibaba Cloud Developer](https://mp.weixin.qq.com/s/Ywj3XMws0IIK-kiUllN87Q)

## Tham khảo

- 《Sâu sắc hiểu về Java Virtual Machine: Các tính năng nâng cao của JVM và thực tiễn tốt nhất (Phiên bản 2)》
- The Java® Virtual Machine Specification - Java SE 8 Edition: <https://docs.oracle.com/javase/specs/jvms/se8/html/index.html>

<!-- @include: @article-footer.snippet.md -->
