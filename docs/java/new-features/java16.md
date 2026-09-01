---
title: Java 16 新特性概览
description: 介绍 JDK 16 的语言与平台更新，包含记录类与其他 JEP 改动。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 16,JDK16,记录类改进,新 API,JEP,性能
---

Java 16 chính thức phát hành vào ngày 16 tháng 3 năm 2021, là phiên bản không phải Hỗ trợ dài hạn (LTS).

JDK 16 có tổng cộng 17 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 338: Vector API (Incubator) (Vector API, Lần ươm tạo thứ 1)](https://openjdk.java.net/jeps/338)
- [JEP 376: ZGC: Concurrent Thread-Stack Processing (ZGC Xử lý luồng stack đồng thời)](https://openjdk.java.net/jeps/376)
- [JEP 387: Elastic Metaspace (Metaspace đàn hồi)](https://openjdk.java.net/jeps/387)
- [JEP 390: Warnings for Value-Based Classes (Cảnh báo cho các lớp dựa trên giá trị)](https://openjdk.java.net/jeps/390)
- [JEP 394: Pattern Matching for instanceof (Khớp mẫu cho instanceof, Chính thức)](https://openjdk.java.net/jeps/394)
- [JEP 395: Records (Lớp record, Chính thức)](https://openjdk.java.net/jeps/395)
- [JEP 396: Strongly Encapsulate JDK Internals by Default (Mặc định đóng gói mạnh các thành phần nội bộ JDK)](https://openjdk.java.net/jeps/396)
- [JEP 397: Sealed Classes (Second Preview) (Sealed class, Xem trước lần 2)](https://openjdk.java.net/jeps/397)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![Số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Đọc thêm: [Tài liệu OpenJDK Java 16](https://openjdk.java.net/projects/jdk/16/).

## JEP 338: Vector API (Vector API, Lần ươm tạo thứ 1)

Vector API ban đầu do [JEP 338](https://openjdk.java.net/jeps/338) đề xuất, và được tích hợp vào Java 16 dưới dạng [Incubator API](http://openjdk.java.net/jeps/11). Vòng ươm tạo thứ hai do [JEP 414](https://openjdk.java.net/jeps/414) đề xuất và tích hợp vào Java 17, vòng ươm tạo thứ ba do [JEP 417](https://openjdk.java.net/jeps/417) đề xuất và tích hợp vào Java 18, vòng thứ tư do [JEP 426](https://openjdk.java.net/jeps/426) đề xuất và tích hợp vào Java 19.

API ươm tạo này cung cấp một vòng lặp ban đầu của API để biểu thị các tính toán vector, các tính toán này khi runtime được biên dịch một cách đáng tin cậy thành các lệnh phần cứng vector tối ưu trên kiến trúc CPU được hỗ trợ, từ đó đạt được hiệu năng vượt trội hơn so với các tính toán vô hướng (scalar) tương đương, tận dụng tối đa công nghệ Single Instruction Multiple Data (SIMD) (một loại lệnh có sẵn trên hầu hết các CPU hiện đại). Mặc dù HotSpot hỗ trợ tự động vector hóa, tuy nhiên tập hợp thao tác vô hướng có thể chuyển đổi bị hạn chế và dễ chịu ảnh hưởng bởi việc thay đổi mã nguồn. API này sẽ giúp các nhà phát triển dễ dàng sử dụng Java để viết các thuật toán vector hiệu năng cao có thể di trú.

Trong bài [Tổng quan tính năng mới của Java 18](./java18.md), tôi đã giới thiệu chi tiết về Vector API, ở đây sẽ không giới thiệu thêm nữa.

## JEP 347: Enable C++ 14 Language Features (Bật các tính năng ngôn ngữ C++ 14)

Java 16 cho phép sử dụng các tính năng ngôn ngữ C++ 14 trong mã nguồn C++ của JDK, và cung cấp hướng dẫn cụ thể về việc những tính năng nào có thể sử dụng trong mã HotSpot.

Trong Java 15, các tính năng ngôn ngữ được sử dụng bởi mã C++ trong JDK bị giới hạn ở tiêu chuẩn ngôn ngữ C++98/03. Nó yêu cầu cập nhật phiên bản chấp nhận tối thiểu của các trình biên dịch trên các nền tảng khác nhau.

## JEP 376: ZGC: Concurrent Thread-Stack Processing (ZGC Xử lý luồng stack đồng thời)

Java 16 chuyển việc xử lý luồng stack của ZGC từ Safepoint sang một giai đoạn đồng thời (concurrent phase), ngay cả trên các heap lớn cũng cho phép tạm dừng GC Safepoint trong vòng vài millisecond. Việc loại bỏ nguồn độ trễ cuối cùng trong Bộ thu gom rác ZGC có thể nâng cao đáng kể hiệu năng và hiệu suất của ứng dụng.

## JEP 387: Elastic Metaspace (Metaspace đàn hồi)

Kể từ khi giới thiệu Metaspace, theo phản hồi, Metaspace thường xuyên chiếm dụng quá nhiều bộ nhớ ngoài heap (off-heap), dẫn đến lãng phí bộ nhớ. Tính năng Metaspace đàn hồi này có thể trả bộ nhớ lớp metadata HotSpot chưa sử dụng (tức Metaspace) về cho hệ điều hành nhanh hơn, từ đó giảm không gian chiếm dụng của Metaspace.

Hơn nữa, đề xuất này cũng đơn giản hóa mã nguồn Metaspace để giảm chi phí bảo trì.

## JEP 390: Warnings for Value-Based Classes (Cảnh báo cho các lớp dựa trên giá trị)

> Phần giới thiệu dưới đây trích từ: [Thực hành | Phân tích tính năng cú pháp mới Java 16](https://xie.infoq.cn/article/8304c894c4e38318d38ceb116).

Ngay từ phiên bản Java 9, các nhà thiết kế Java đã thực hiện một lần nâng cấp cho annotation `@Deprecated`, bổ sung thêm 2 phần tử mới là `since` và `forRemoval`. Trong đó, phần tử `since` dùng để chỉ định phiên bản mà API được đánh dấu annotation `@Deprecated` bị loại bỏ, còn `forRemoval` làm rõ hơn ngữ nghĩa khi API được đánh dấu annotation `@Deprecated`, nếu `forRemoval=true`, nghĩa là API đó chắc chắn sẽ bị xóa trong các phiên bản tương lai, nhà phát triển nên sử dụng API mới để thay thế, không còn dễ gây mơ hồ nữa (Trước Java 9, API đánh dấu annotation `@Deprecated` có nhiều khả năng về mặt ngữ nghĩa, ví dụ: có rủi ro sử dụng, có thể tồn tại lỗi tương thích trong tương lai, có thể bị xóa trong phiên bản tương lai, cũng như nên sử dụng giải pháp thay thế tốt hơn,...).

Quan sát kỹ các lớp đóng gói (wrapper class) của kiểu nguyên thủy (ví dụ: `java.lang.Integer`, `java.lang.Double`), không khó để phát hiện ra rằng các constructor của chúng đều đã được đánh dấu annotation `@Deprecated(since="9", forRemoval = true)`, điều này có nghĩa là các constructor này dự kiến sẽ bị xóa trong tương lai, không nên tiếp tục sử dụng cách lập trình như `new Integer(10)` trong chương trình (khuyên dùng `Integer a = 10` hoặc phương thức `Integer.valueOf()`), nếu tiếp tục sử dụng, lúc biên dịch sẽ tạo ra cảnh báo `'Integer(int)' is deprecated and marked for removal`. Hơn nữa, các kiểu đóng gói này đã cùng với `java.util.Optional` và `java.time.LocalDateTime` được chỉ định là các lớp dựa trên giá trị (value-based class), không nhầm lẫn với value type trong Project Valhalla.

Thứ hai, trong khối đồng bộ `synchronized` nếu sử dụng instance của lớp dựa trên giá trị, trình biên dịch của JDK 16 sẽ tạo ra cảnh báo. HotSpot còn có thể ghi lại hoặc ngăn chặn loại đồng bộ này lúc runtime thông qua tham số chẩn đoán. Ngay cả khi không bật chẩn đoán runtime, cũng không nên dùng instance của lớp dựa trên giá trị làm khóa (lock), ví dụ:

```java
private Integer count = 0;

public void inc() {
    synchronized (count) {
        count++;
    }
}
```

`Integer` là đối tượng bất biến, `count++` sẽ trải qua mở hộp (unboxing), cộng một và đóng hộp lại (re-boxing), và thay đổi tham chiếu trường thành một instance `Integer` khác. Do đó, các luồng khác nhau có thể khóa các đối tượng khác nhau, không thể cung cấp sự loại trừ tương hỗ ổn định cho thao tác phức hợp này. Nếu cần tự tăng đồng thời, nên sử dụng đối tượng lock cố định hoặc `AtomicInteger`.

## JEP 392: Packaging Tool (Công cụ đóng gói, Chính thức)

Trong Java 14, JEP 343 đã giới thiệu công cụ đóng gói với câu lệnh là `jpackage`. Trong Java 15, tiếp tục ươm tạo, và đến Java 16 cuối cùng đã trở thành chức năng chính thức.

Công cụ đóng gói này cho phép đóng gói các ứng dụng Java tự chứa (self-contained). Nó hỗ trợ định dạng đóng gói native, mang lại trải nghiệm cài đặt tự nhiên cho người dùng cuối, các định dạng này bao gồm msi và exe trên Windows, pkg và dmg trên macOS, cũng như deb và rpm trên Linux. Nó cũng cho phép chỉ định các tham số lúc khởi chạy khi đóng gói, và có thể gọi trực tiếp từ dòng lệnh, hoặc gọi lập trình thông qua ToolProvider API. Lưu ý module jpackage đổi tên từ jdk.incubator.jpackage thành jdk.jpackage. Điều này sẽ cải thiện trải nghiệm của người dùng cuối khi cài đặt ứng dụng, và đơn giản hóa việc triển khai mô hình "App Store".

Về việc sử dụng thực tế công cụ đóng gói này, có thể xem video [Playing with Java 16 jpackage](https://www.youtube.com/watch?v=KahYIVzRIkQ).

## JEP 393: Foreign Memory Access API (API truy cập bộ nhớ ngoài, Lần ươm tạo thứ 3)

Giới thiệu Foreign Memory Access API để cho phép các chương trình Java truy cập bộ nhớ ngoài ngoài Java heap một cách an toàn và hiệu quả.

Java 14 ([JEP 370](https://openjdk.org/jeps/370)) ươm tạo Foreign Memory Access API lần 1, Java 15 thực hiện ươm tạo lần 2 ([JEP 383](https://openjdk.org/jeps/383)), và Java 16 thực hiện ươm tạo lần 3.

Mục đích giới thiệu Foreign Memory Access API như sau:

- Dùng chung (Generic): Một API đơn lẻ có thể thao tác trên nhiều loại bộ nhớ ngoài (như native memory, persistent memory, heap memory,...).
- An toàn: Bất kể thao tác loại bộ nhớ nào, API cũng không nên làm hỏng tính an toàn của JVM.
- Kiểm soát: Có thể tự do lựa chọn cách giải phóng bộ nhớ (rõ ràng, ẩn danh,...).
- Khả dụng: Đối với các chương trình cần truy cập bộ nhớ ngoài, API này nên cung cấp một tập hợp các giải pháp dễ dùng đủ để thay thế `sun.misc.Unsafe`.

## JEP 394: Pattern Matching for instanceof (Khớp mẫu cho instanceof, Chính thức)

| Phiên bản JDK | Loại cập nhật      | JEP                                     | Nội dung cập nhật                        |
| ------------- | ------------------ | --------------------------------------- | ---------------------------------------- |
| Java SE 14    | Preview            | [JEP 305](https://openjdk.org/jeps/305) | Lần đầu giới thiệu khớp mẫu instanceof.  |
| Java SE 15    | Second Preview     | [JEP 375](https://openjdk.org/jeps/375) | Không thay đổi so với bản trước, tiếp tục thu thập phản hồi. |
| Java SE 16    | Permanent Release  | [JEP 394](https://openjdk.org/jeps/394) | Biến mẫu không còn ẩn định nghĩa là final nữa. |

Từ Java 16 trở đi, bạn có thể sửa đổi giá trị biến trong `instanceof`.

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

## JEP 395: Records (Lớp record, Chính thức)

Lịch sử thay đổi loại record:

| Phiên bản JDK | Loại cập nhật      | JEP                                          | Nội dung cập nhật                                                         |
| ------------- | ------------------ | -------------------------------------------- | ------------------------------------------------------------------------- |
| Java SE 14    | Preview            | [JEP 359](https://openjdk.java.net/jeps/359) | Giới thiệu từ khóa `record`, `record` cung cấp cú pháp gọn gàng để định nghĩa dữ liệu bất biến trong class. |
| Java SE 15    | Second Preview     | [JEP 384](https://openjdk.org/jeps/384)      | Hỗ trợ sử dụng `record` trong phương thức cục bộ và interface.           |
| Java SE 16    | Permanent Release  | [JEP 395](https://openjdk.org/jeps/395)      | Non-static inner class có thể định nghĩa các thành viên static không phải hằng số. |

Từ Java SE 16 trở đi, non-static inner class có thể định nghĩa các thành viên static không phải hằng số.

```java
public class Outer {
  class Inner {
    static int age;
  }
}
```

> Trước JDK 16, nếu viết đoạn mã như trên, IDE sẽ gợi ý trường static age không thể khai báo static trong inner type không phải static, trừ khi nó được khởi tạo bằng một biểu thức hằng số. (The field age cannot be declared static in a non-static inner type, unless initialized with a constant expression)

## JEP 396: Strongly Encapsulate JDK Internals by Default (Mặc định đóng gói mạnh các thành phần nội bộ JDK)

Tính năng này mặc định đóng gói mạnh tất cả các thành phần nội bộ của JDK, trừ các API nội bộ then chốt (ví dụ `sun.misc.Unsafe`). Theo mặc định, mã nguồn truy cập API nội bộ JDK được biên dịch thành công ở các phiên bản trước có thể không còn hoạt động. Khuyến khích các nhà phát triển chuyển dịch từ việc sử dụng các thành phần nội bộ sang các phương thức API tiêu chuẩn, nhờ đó họ và người dùng của họ đều có thể nâng cấp mượt mà lên các phiên bản Java tương lai. Việc đóng gói mạnh được kiểm soát bởi tùy chọn launcher –illegal-access của JDK 9, đến JDK 15 mặc định chuyển thành warning, từ JDK 16 trở đi mặc định là deny. (Hiện tại) vẫn có thể sử dụng tùy chọn dòng lệnh đơn lẻ để nới lỏng việc đóng gói đối với tất cả các package, trong tương lai chỉ khi sử dụng –add-opens mở các package cụ thể mới được.

## JEP 397: Sealed Classes (Sealed class, Xem trước lần 2)

Sealed class được [JEP 360](https://openjdk.java.net/jeps/360) đề xuất xem trước và tích hợp vào Java 15. Trong JDK 16, Sealed class đã được cải tiến ( kiểm tra tham chiếu nghiêm ngặt hơn và quan hệ kế thừa của Sealed class), được [JEP 397](https://openjdk.java.net/jeps/397) đề xuất xem trước lại.

Trong bài [Tổng quan tính năng mới Java 14 & 15](./java14-15.md), tôi đã giới thiệu chi tiết về Sealed class, ở đây không giới thiệu thêm nữa.

## Các tối ưu hóa và cải tiến khác

- **JEP 380: Unix-Domain Socket Channels**: Unix-domain socket luôn là một tính năng của hầu hết các nền tảng Unix, hiện tại trên Windows 10 và Windows Server 2019 cũng đã cung cấp hỗ trợ. Tính năng này bổ sung hỗ trợ Unix-domain (AF_UNIX) socket cho socket channel và server socket channel API trong package java.nio.channels. Nó mở rộng cơ chế channel kế thừa để hỗ trợ Unix-domain socket channel và server socket channel. Unix-domain socket dùng cho giao tiếp giữa các tiến trình (IPC) trên cùng một host. Chúng về cơ bản tương tự TCP/IP, điểm khác biệt là socket được định danh thông qua đường dẫn file system chứ không phải địa chỉ IP và port. Đối với IPC nội bộ, Unix-domain socket an toàn và hiệu quả hơn kết nối loopback TCP/IP
- **JEP 389: Foreign Linker API (Ươm tạo):** API ươm tạo này cung cấp tính năng truy cập mã native bằng Java thuần túy, kiểu tĩnh, API này sẽ đơn giản hóa đáng kể quá trình liên kết các thư viện native vốn phức tạp và dễ phát sinh lỗi. Java 1.1 đã hỗ trợ gọi phương thức native thông qua Java Native Interface (JNI), nhưng không dễ dùng. Các nhà phát triển Java nên có khả năng liên kết các thư viện native cụ thể cho các tác vụ cụ thể. Nó cũng cung cấp hỗ trợ hàm ngoại (foreign function) mà không cần bất kỳ mã dán JNI trung gian nào.
- **JEP 357: Chuyển dịch từ Mercurial sang Git**: Trước đây, mã nguồn OpenJDK được quản lý bằng công cụ quản lý phiên bản Mercurial, hiện tại đã chuyển dịch sang Git.
- **JEP 369: Chuyển dịch sang GitHub**: Thống nhất với sự thay đổi của JEP 357 chuyển dịch từ Mercurial sang Git, sau khi chuyển dịch quản lý phiên bản sang Git, đã lựa chọn host Git repository của cộng đồng OpenJDK trên GitHub. Tuy nhiên chỉ thực hiện chuyển dịch đối với JDK 11 và các phiên bản JDK cao hơn.
- **JEP 386: Porting sang Alpine Linux**: Alpine Linux là một bản phân phối Linux độc lập, phi thương mại, nó rất nhỏ, một container chỉ cần không quá 8MB không gian, cài đặt tối thiểu vào đĩa chỉ cần khoảng 130MB không gian lưu trữ, và rất đơn giản, đồng thời kiêm cả tính an toàn. Đề xuất này đã port JDK sang Alpine Linux, do Alpine Linux là bản phân phối Linux siêu nhẹ dựa trên musl lib, nên các bản phân phối Linux sử dụng musl lib trên kiến trúc x64 và AArch64 khác cũng áp dụng được.
- **JEP 388: Porting sang Windows/AArch64**: Trọng tâm của các JEP này không phải là bản thân công việc porting, mà là tích hợp chúng vào JDK mainline repository; JEP 386 port JDK sang Alpine Linux và các bản phân phối khác dùng musl làm C library chính trên x64. Ngoài ra, JEP 388 port JDK sang Windows AArch64 (ARM64).

## Tài liệu tham khảo

- [Java Language Changes](https://docs.oracle.com/en/java/javase/16/language/java-language-changes.html)
- [Consolidated JDK 16 Release Notes](https://www.oracle.com/java/technologies/javase/16all-relnotes.html)
- [Java 16 chính thức phát hành, phân tích từng tính năng mới](https://www.infoq.cn/article/IAkwhx7i9V7G8zLVEd4L)
- [Thực hành | Phân tích tính năng cú pháp mới Java 16](https://xie.infoq.cn/article/8304c894c4e38318d38ceb116)

<!-- @include: @article-footer.snippet.md -->
