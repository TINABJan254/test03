---
title: 类加载过程详解
description: 拆解 JVM 类加载的各阶段与关键细节，理解验证、准备、解析与初始化的具体行为。
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: 类加载,加载,验证,准备,解析,初始化,clinit,常量池
---

## Vòng đời của Class

Từ khi Class được nạp vào bộ nhớ Virtual Machine cho đến khi gỡ bỏ khỏi bộ nhớ, toàn bộ vòng đời của nó có thể tóm tắt đơn giản qua 7 giai đoạn: Loading (Nạp), Verification (Xác minh), Preparation (Chuẩn bị), Resolution (Phân tích), Initialization (Khởi tạo), Using (Sử dụng) và Unloading (Gỡ bỏ). Trong đó, Verification, Preparation và Resolution có thể gọi chung là Linking (Liên kết).

Thứ tự của 7 giai đoạn này như hình dưới:

![Vòng đời hoàn chỉnh của một Class](https://oss.javaguide.cn/github/javaguide/java/jvm/lifecycle-of-a-class.png)

## Quá trình Class Loading

**File Class cần được nạp vào Virtual Machine mới có thể chạy và sử dụng, vậy Virtual Machine nạp các file Class này như thế nào?**

Hệ thống nạp file kiểu Class chủ yếu qua 3 bước: **Loading -> Linking -> Initialization**. Quá trình Linking lại có thể chia thành 3 bước: **Verification -> Preparation -> Resolution**.

![Quá trình Class Loading](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-procedure.png)

Chi tiết xem tại [Java Virtual Machine Specification - 5.3. Creation and Loading](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3 "Java Virtual Machine Specification - 5.3. Creation and Loading").

### Loading (Nạp)

Bước đầu tiên của quá trình Class Loading, chủ yếu hoàn thành 3 công việc dưới đây:

1. Thông qua FQCN (tên đầy đủ của Class) để lấy luồng byte nhị phân định nghĩa Class này.
2. Chuyển đổi cấu trúc lưu trữ tĩnh đại diện bởi luồng byte thành cấu trúc dữ liệu runtime trong Method Area.
3. Tạo một đối tượng `Class` đại diện cho Class đó trong bộ nhớ làm lối vào truy cập các dữ liệu này trong Method Area.

Ba điểm trên trong quy chuẩn Virtual Machine không quá cụ thể, do đó rất linh hoạt. Ví dụ: "Thông qua FQCN để lấy luồng byte nhị phân định nghĩa Class này" không chỉ rõ cụ thể lấy từ đâu (`ZIP`, `JAR`, `EAR`, `WAR`, mạng, tạo động lúc runtime qua kỹ thuật Dynamic Proxy, tạo từ file khác như `JSP`...), hay lấy như thế nào.

Bước Loading này chủ yếu do **ClassLoader (Bộ nạp lớp)** mà chúng ta nói ở sau hoàn thành. Có rất nhiều loại ClassLoader, khi chúng ta muốn nạp một class, cụ thể là ClassLoader nào nạp do **Parents Delegation Model** quyết định (tuy nhiên, chúng ta cũng có thể phá vỡ Parents Delegation Model).

> ClassLoader, Parents Delegation Model cũng là các điểm kiến thức rất quan trọng, phần nội dung này được giới thiệu chi tiết trong bài viết [Giải thích chi tiết ClassLoader](https://javaguide.cn/java/jvm/classloader.html "Giải thích chi tiết ClassLoader"). Khi đọc bài viết này, mọi người chỉ cần biết có thứ như vậy là được.

Mỗi class phi mảng hoặc interface đều được tạo bởi một ClassLoader nào đó. Mảng class không được tạo thông qua `ClassLoader`, mà do JVM tự động tạo khi cần; ClassLoader định nghĩa của mảng kiểu reference đồng nhất với ClassLoader định nghĩa của kiểu thành phần của nó, còn mảng kiểu nguyên thủy khi gọi `getClassLoader()` sẽ trả về `null`.

Giai đoạn nạp của một class phi mảng (thao tác lấy luồng byte nhị phân của class) có tính kiểm soát rất mạnh. Thông thường có thể kế thừa `ClassLoader` và override `findClass()` để kiểm soát phương thức lấy luồng byte, đồng thời giữ lại quy trình Parents Delegation do `loadClass()` triển khai; chỉ khi thực sự muốn thay đổi quy tắc ủy quyền mới cần override `loadClass()`.

Giai đoạn Loading và một số hành động của giai đoạn Linking (như một phần hành động kiểm tra định dạng file bytecode) là đan xen tiến hành, giai đoạn Loading chưa kết thúc thì giai đoạn Linking có thể đã bắt đầu rồi.

### Verification (Xác minh)

**Verification là bước đầu tiên của giai đoạn Linking, mục đích của giai đoạn này là đảm bảo thông tin chứa trong luồng byte của file Class phù hợp với tất cả các yêu cầu ràng buộc của 《Quy chuẩn Java Virtual Machine》, đảm bảo các thông tin này sau khi chạy như code sẽ không gây hại cho an toàn của chính Virtual Machine.**

Bước trong giai đoạn Verification tiêu tốn tương đối nhiều tài nguyên trong toàn bộ quá trình Class Loading, nhưng rất cần thiết, có thể phòng ngừa hiệu quả việc thực thi code độc hại. Bất kỳ lúc nào, an toàn chương trình luôn ở vị trí hàng đầu.

HotSpot từng cung cấp `-Xverify:none` và `-noverify` để tắt phần lớn kiểm tra xác minh class, nhưng điều này làm suy yếu kiểm tra an toàn bytecode, không nên làm phương pháp tối ưu hóa chung trong môi trường production. Hai option này đã bị deprecated từ JDK 13, các JDK hiện đại còn có thể bỏ qua hoặc loại bỏ chúng.

Giai đoạn Verification chủ yếu do 4 giai đoạn kiểm tra cấu thành:

1. Verification định dạng file (Kiểm tra định dạng file Class)
2. Verification metadata (Kiểm tra ngữ nghĩa bytecode)
3. Verification bytecode (Kiểm tra ngữ nghĩa chương trình)
4. Verification symbolic reference (Kiểm tra tính đúng đắn của Class)

![Sơ đồ giai đoạn Verification](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-process-verification.png)

Giai đoạn Verification định dạng file được tiến hành dựa trên luồng byte nhị phân của Class đó, mục đích chủ yếu là đảm bảo luồng byte đầu vào có thể parse đúng và lưu trữ trong Method Area, về định dạng phù hợp với yêu cầu mô tả thông tin của một type Java. Trừ giai đoạn này ra, 3 giai đoạn verification còn lại đều được tiến hành dựa trên cấu trúc lưu trữ của Method Area, chứ không trực tiếp đọc, thao tác luồng byte nữa.

> Method Area thuộc về một vùng logic trong Runtime Data Area của JVM, là vùng bộ nhớ dùng chung cho các thread. Khi Virtual Machine muốn sử dụng một class, nó cần đọc và parse file Class lấy thông tin liên quan, rồi lưu thông tin vào Method Area. Method Area sẽ lưu trữ các dữ liệu đã được Virtual Machine nạp như **thông tin Class, thông tin field, thông tin phương thức, hằng số, biến static, bộ đệm code biên dịch JIT v.v.**
>
> Về giới thiệu chi tiết Method Area, khuyến nghị đọc bài viết [Giải thích chi tiết vùng bộ nhớ Java](https://javaguide.cn/java/jvm/memory-area.html "Giải thích chi tiết vùng bộ nhớ Java").

Verification symbolic reference xảy ra trong giai đoạn Resolution của quá trình Class Loading, cụ thể là khi JVM chuyển đổi symbolic reference thành direct reference (giai đoạn Resolution sẽ giới thiệu symbolic reference và direct reference).

Mục đích chính của Verification symbolic reference là đảm bảo giai đoạn Resolution có thể thực thi bình thường, nếu không thể thông qua Verification symbolic reference, JVM sẽ ném exception, ví dụ:

- `java.lang.IllegalAccessError`: Khi class thử truy cập hoặc sửa đổi field mà nó không có quyền truy cập, hoặc gọi phương thức mà nó không có quyền truy cập, sẽ ném exception này.
- `java.lang.NoSuchFieldError`: Khi class thử truy cập hoặc sửa đổi một field đối tượng chỉ định, mà đối tượng đó không còn chứa field đó nữa, sẽ ném exception này.
- `java.lang.NoSuchMethodError`: Khi class thử truy cập một phương thức chỉ định, mà phương thức đó không tồn tại, sẽ ném exception này.
- ……

### Preparation (Chuẩn bị)

**Giai đoạn Preparation là giai đoạn chính thức cấp phát bộ nhớ cho biến Class và thiết lập giá trị ban đầu cho biến Class**, các bộ nhớ này đều sẽ được cấp phát trong Method Area. Đối với giai đoạn này cần lưu ý vài điểm dưới đây:

1. Việc cấp phát bộ nhớ lúc này chỉ bao gồm biến Class (Class Variables, tức biến static, biến được modifier bằng keyword `static`, chỉ liên quan đến Class, do đó được gọi là biến Class), chứ không bao gồm biến instance. Biến instance sẽ cùng với đối tượng được cấp phát trên Java Heap khi đối tượng được khởi tạo instance.
2. Về mặt khái niệm, bộ nhớ mà biến Class sử dụng đều nên được cấp phát trong **Method Area**. Tuy nhiên có một điểm cần lưu ý: Trước JDK 7, khi HotSpot sử dụng Permanent Generation để triển khai Method Area, cách triển khai hoàn toàn phù hợp với khái niệm logic này. Còn từ JDK 7 trở đi, HotSpot đã chuyển String Constant Pool, biến static v.v. vốn đặt ở Permanent Generation sang Heap, lúc này biến Class sẽ cùng với đối tượng Class được lưu trữ trong Java Heap. Bài viết liên quan: [《Sâu sắc hiểu về Java Virtual Machine (Phiên bản 3)》Sửa lỗi#75](https://github.com/fenixsoft/jvm_book/issues/75 "《Sâu sắc hiểu về Java Virtual Machine (Phiên bản 3)》Sửa lỗi#75")
3. Giá trị ban đầu được thiết lập ở đây thông thường là giá trị 0 mặc định của kiểu dữ liệu (như 0, 0L, null, false v.v.). Ví dụ định nghĩa `public static int value=111`, `value` ở giai đoạn Preparation thông thường nhận 0 trước, sang giai đoạn Initialization mới gán thành 111. Trường hợp đặc biệt là khi field mang thuộc tính `ConstantValue`, giai đoạn Preparation sẽ gán trực tiếp giá trị chỉ định bởi thuộc tính đó; `public static final int value=111` hằng số compile-time như vậy là ví dụ điển hình, nhưng không phải tất cả các field `static final` đều có thuộc tính `ConstantValue`.

**Giá trị 0 mặc định của các kiểu dữ liệu nguyên thủy**: (Hình trích từ 《Sâu sắc hiểu về Java Virtual Machine》 phiên bản 3 mục 7.3.3)

![Giá trị 0 mặc định của kiểu dữ liệu nguyên thủy](https://oss.javaguide.cn/github/javaguide/java/%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E9%9B%B6%E5%80%BC.png)

### Resolution (Phân tích)

**Giai đoạn Resolution là quá trình Virtual Machine từ symbolic reference trong Constant Pool runtime xác định động giá trị cụ thể.** Quy chuẩn hiện tại liên quan đến các symbolic reference như Class hoặc Interface, field, phương thức, phương thức interface, method type, method handle, dynamic call site cũng như dynamic constant.

Giải thích về symbolic reference và direct reference tại mục 7.3.4 phiên bản 3 cuốn 《Sâu sắc hiểu về Java Virtual Machine》 như sau:

![Symbolic reference và direct reference](https://oss.javaguide.cn/github/javaguide/java/jvm/symbol-reference-and-direct-reference.png)

Lấy một ví dụ: Khi chương trình gọi phương thức, Virtual Machine cần dựa trên method symbolic reference để xác định phương thức thực sự cần gọi. Các Virtual Machine như HotSpot có thể sử dụng bảng phương thức, địa chỉ lối vào hoặc cấu trúc bên trong khác để tăng tốc lời gọi, nhưng các cách biểu diễn này thuộc về chi tiết triển khai, chứ không phải offset bảng phương thức quy định thống nhất trong 《Quy chuẩn Java Virtual Machine》.

Tóm lại, giai đoạn Resolution là quá trình Virtual Machine dựa trên symbolic reference trong Constant Pool runtime để xác định các mục tiêu cụ thể như class, field, phương thức hoặc dynamic call site; biểu diễn bên trong của direct reference không nhất thiết là con trỏ bộ nhớ hay offset cố định.

### Initialization (Khởi tạo)

**Giai đoạn Initialization sẽ thực thi phương thức khởi tạo `<clinit>()` của class hoặc interface (nếu compiler tạo phương thức đó), đây là bước cuối cùng của quá trình Class Loading.**

> Giải thích: Compiler dựa trên biểu thức khởi tạo field static và khối khởi tạo static để tạo `<clinit>()`; nếu không có code khởi tạo class cần thực thi, trong file Class có thể không tồn tại phương thức đó.

JVM sẽ đồng bộ quá trình khởi tạo class hoặc interface, đảm bảo tại một thời điểm chỉ có duy nhất một thread thực thi phương thức khởi tạo của nó; điều này không có nghĩa là bản thân phương thức `<clinit>()` mang khóa Java. Các thread khác có thể bị block khi chờ đợi class đó hoàn thành khởi tạo.

Đối với giai đoạn Initialization, quy chuẩn liệt kê các kịch bản sử dụng chủ động chính bao gồm:

1. Khi gặp 4 lệnh bytecode `new`, `getstatic`, `putstatic` hoặc `invokestatic`:
   - `new`: Tạo một đối tượng instance của class.
   - `getstatic`, `putstatic`: Đọc hoặc thiết lập một field static của type (trừ field static được `final` modifier, đã đưa kết quả vào Constant Pool ở thời điểm compile).
   - `invokestatic`: Gọi phương thức static của class.
2. Khi sử dụng các phương thức của package `java.lang.reflect` tiến hành gọi reflection đối với class như `Class.forName("...")`, `newInstance()` v.v. Nếu class chưa khởi tạo, cần kích hoạt khởi tạo nó.
3. Khởi tạo một class, nếu class cha của nó chưa khởi tạo, thì kích hoạt khởi tạo class cha đó trước.
4. Khi Virtual Machine khởi động, người dùng cần định nghĩa một main class cần thực thi (class chứa phương thức `main`), Virtual Machine sẽ khởi tạo class này trước.
5. Lần đầu gọi `MethodHandle` có kết quả parse là `REF_getStatic`, `REF_putStatic`, `REF_invokeStatic` hoặc `REF_newInvokeSpecial`, cần khởi tạo class hoặc interface khai báo mục tiêu đó.
6. **"Bổ sung, từ [issue745](https://github.com/Snailclimb/JavaGuide/issues/745 "issue745")"** Khi một interface định nghĩa phương thức default mới bổ sung trong JDK8 (phương thức interface được modifier bằng keyword default), nếu có class triển khai của interface này xảy ra khởi tạo, thì interface đó phải được khởi tạo trước nó.

## Class Unloading (Gỡ bỏ lớp)

> Phần nội dung gỡ bỏ này đến từ [issue#662](https://github.com/Snailclimb/JavaGuide/issues/662 "issue#662") do **[guang19](https://github.com/guang19 "guang19")** bổ sung hoàn thiện.

Class unloading là quá trình JVM thu hồi biểu diễn Method Area của một class hoặc interface và các tài nguyên liên quan của nó. Theo 《Quy chuẩn ngôn ngữ Java》, class hoặc interface chỉ khi ClassLoader định nghĩa nó có thể bị Garbage Collection thì mới có thể bị gỡ bỏ; class hoặc interface do Bootstrap ClassLoader định nghĩa không thể bị gỡ bỏ. Trong các ứng dụng HotSpot phổ biến, điều này thông thường xảy ra trên Custom ClassLoader có thể thu hồi và các class do nó định nghĩa.

Trong HotSpot khi phán đoán một class nào đó có khả năng gỡ bỏ hay không, thông thường có thể hiểu từ 3 điều kiện dưới đây:

1. Tất cả các instance object của class đó đã bị GC, nghĩa là Heap không tồn tại instance object của class đó.
2. Class đó không được reference ở bất kỳ nơi nào khác
3. Instance ClassLoader của class đó đã bị GC

Thỏa mãn các điều kiện này chỉ thể hiện class đó đủ điều kiện bị gỡ bỏ, chứ không đảm bảo JVM sẽ lập tức hoặc nhất định gỡ bỏ nó. Class do Bootstrap ClassLoader, Platform ClassLoader và Application ClassLoader tích hợp sẵn trong JDK tạo ra sẽ sống lâu dài cùng JVM; class do Custom ClassLoader tạo ra thì có thể bị gỡ bỏ.

Thông thường, Bootstrap ClassLoader, Platform ClassLoader và App ClassLoader tích hợp sẵn trong JDK sẽ sống lâu dài cùng JVM; instance Custom ClassLoader thì có thể trở thành không thể tới được, do đó class do nó định nghĩa có thể đủ điều kiện gỡ bỏ. Từ JDK 9 trở đi, Extension ClassLoader cũ đã được thay thế bởi Platform ClassLoader.

**Tham khảo**

- 《Sâu sắc hiểu về Java Virtual Machine》
- 《Thực chiến Java Virtual Machine》
- Chapter 5. Loading, Linking, and Initializing - Java Virtual Machine Specification: <https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4>

<!-- @include: @article-footer.snippet.md -->
