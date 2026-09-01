---
title: Java 26 新特性概览
description: 概览 JDK 26 的关键新特性与预览改动，关注 HTTP/3、GC 性能优化、AOT 缓存与语言/平台增强。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 26,JDK26,HTTP/3,G1 GC,AOT 缓存,延迟常量,结构化并发,向量 API,模式匹配
---

JDK 26 được phát hành vào ngày 17 tháng 3 năm 2026, đây là một phiên bản không phải LTS (không phải Hỗ trợ dài hạn). Phiên bản Hỗ trợ dài hạn trước đó là **JDK 25**, phiên bản Hỗ trợ dài hạn tiếp theo dự kiến là **JDK 29**.

JDK 26 có tổng cộng 10 tính năng mới, bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- [JEP 517: HTTP/3 for the HTTP Client API (Giới thiệu hỗ trợ HTTP/3 cho HTTP Client API)](https://openjdk.org/jeps/517)
- [JEP 522: G1 GC: Improve Throughput by Reducing Synchronization (Tối ưu hóa thông lượng G1 GC)](https://openjdk.org/jeps/522)
- [JEP 516: Ahead-of-Time Object Caching with Any GC (Bộ đệm đối tượng AOT hỗ trợ GC bất kỳ)](https://openjdk.org/jeps/516)
- [JEP 500: Prepare to Make Final Mean Final (Chuẩn bị để final thực sự bất biến)](https://openjdk.org/jeps/500)
- [JEP 526: Lazy Constants (Hằng số lười, Xem trước lần 2)](https://openjdk.org/jeps/526)
- [JEP 525: Structured Concurrency (Đồng thời cấu trúc, Xem trước lần 6)](https://openjdk.org/jeps/525)
- [JEP 530: Primitive Types in Patterns, instanceof, and switch (Khớp mẫu hỗ trợ kiểu nguyên thủy, Xem trước lần 4)](https://openjdk.org/jeps/530)
- [JEP 524: PEM Encodings of Cryptographic Objects (Mã hóa PEM đối tượng mã hóa, Xem trước lần 2)](https://openjdk.org/jeps/524)
- [JEP 529: Vector API (Vector API, Lần ươm tạo thứ 11)](https://openjdk.org/jeps/529)
- [JEP 504: Remove the Applet API (Loại bỏ Applet API)](https://openjdk.org/jeps/504)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 25:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

## JEP 517: Giới thiệu hỗ trợ HTTP/3 cho HTTP Client API

JDK 26 chính thức bổ sung hỗ trợ **HTTP/3** cho API `java.net.http.HttpClient`, đây là một cập nhật quan trọng được mong đợi từ lâu.

**Ưu điểm của HTTP/3**:

- **Dựa trên giao thức QUIC**: HTTP/2 được triển khai dựa trên giao thức TCP, HTTP/3 bổ sung giao thức QUIC (Quick UDP Internet Connections) để thực hiện truyền tải đáng tin cậy, cung cấp độ an toàn tương đương TLS/SSL, có độ trễ kết nối và truyền tải thấp hơn. Bạn có thể coi QUIC là phiên bản nâng cấp của UDP, bổ sung nhiều chức năng dựa trên nó như mã hóa, truyền lại v.v.
- **Loại bỏ tắc nghẽn đầu hàng (Head-of-Line blocking)**: HTTP/2 tái sử dụng một kết nối TCP cho nhiều request, một khi xảy ra mất gói tin, sẽ làm tắc nghẽn tất cả các HTTP request. Do đặc tính của giao thức QUIC, HTTP/3 giải quyết vấn đề Head-of-Line blocking (viết tắt: HOL blocking) ở một mức độ nhất định, một kết nối thiết lập nhiều luồng dữ liệu khác nhau, khi một luồng dữ liệu xảy ra mất gói tin, các luồng dữ liệu khác không bị ảnh hưởng.
- **Thiết lập kết nối nhanh hơn**: HTTP/2 cần trải qua quá trình bắt tay 3 bước TCP kinh điển (do kết nối an toàn HTTPS thiết lập còn cần bắt tay TLS, tổng cộng cần khoảng 3 RTT). Do đặc tính của giao thức QUIC (TLS 1.3, TLS 1.3 ngoài hỗ trợ bắt tay 1 RTT, còn hỗ trợ bắt tay 0 RTT), việc thiết lập kết nối chỉ cần 0-RTT hoặc 1-RTT. Điều này có nghĩa là QUIC trong trường hợp tốt nhất không cần bất kỳ thời gian khứ hồi bổ sung nào cũng có thể thiết lập kết nối mới.
- **Trải nghiệm thiết bị di động tốt hơn**: HTTP/3 hỗ trợ chuyển đổi kết nối dựa trên QUIC. QUIC sử dụng ID kết nối có độ dài thay đổi do endpoint lựa chọn để định danh và định tuyến kết nối, ID kết nối còn có thể thay đổi trong thời gian kết nối; sau các xử lý như xác thực đường truyền, thay đổi địa chỉ mạng (như chuyển từ Wi-Fi sang dữ liệu di động) không cần phải thiết lập lại toàn bộ kết nối giống như thay đổi 4 bộ thành phần TCP truyền thống.

Giới thiệu chi tiết có thể đọc bài viết này: [Tóm tắt câu hỏi phỏng vấn mạng máy tính thường gặp (Phần trên)](https://javaguide.cn/cs-basics/network/other-network-questions.html) (Mô hình phân tầng mạng, tóm tắt các giao thức mạng thường gặp, HTTP, WebSocket, DNS v.v.)

**Cách sử dụng**:

Giao thức mặc định của `HttpClient` trong JDK 26 vẫn là HTTP/2. Để sử dụng HTTP/3, cần chỉ định rõ `HTTP_3` trên `HttpClient` hoặc từng `HttpRequest` đơn lẻ:

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_3)
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com"))
    .build();

// 优先尝试 HTTP/3；服务器不支持时默认回退到 HTTP/2 或 HTTP/1.1
HttpResponse<String> response = client.send(request,
    HttpResponse.BodyHandlers.ofString());

System.out.println(response.body());
```

Nếu cần chỉ định rõ ràng việc sử dụng HTTP/3, có thể thiết lập thông qua phương thức `version()`:

```java
// 所有请求默认优先使用 HTTP/3
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_3)  // 明确指定 HTTP/3
    .build();

// 设置单个HttpRequest对象的首选协议版本
HttpRequest request = HttpRequest.newBuilder(URI.create("https://javaguide.cn/"))
                         .version(HttpClient.Version.HTTP_3)
                         .GET().build();
```

## JEP 522: Tối ưu hóa thông lượng G1 GC

**Từ JDK 9 trở đi, G1 garbage collector đã trở thành garbage collector mặc định.** Nó tìm kiếm sự cân bằng giữa độ trễ và thông lượng. Tuy nhiên, sự cân bằng này đôi khi ảnh hưởng đến hiệu năng của ứng dụng. So với Parallel GC hướng tới thông lượng, G1 làm việc đồng thời với các thread ứng dụng nhiều hơn để giảm thời gian tạm dừng GC. Nhưng điều này có nghĩa là các thread ứng dụng phải chia sẻ CPU và phối hợp với các thread GC, sự đồng bộ hóa này sẽ làm giảm thông lượng và tăng độ trễ.

JEP 522 giới thiệu cơ chế **bảng thẻ kép (Card Table)**:

1. **Bảng thẻ thứ nhất**: Khi barrier ghi của thread ứng dụng cập nhật bảng thẻ này **không cần bất kỳ sự đồng bộ nào**, làm cho mã barrier ghi đơn giản hơn, nhanh hơn.
2. **Bảng thẻ thứ hai**: Thread optimizer xử lý song song bảng thẻ ban đầu rỗng này trong background.

Khi G1 phát hiện quét bảng thẻ thứ nhất có thể vượt quá mục tiêu thời gian tạm dừng, nó sẽ trao đổi nguyên tử hai bảng thẻ này. Thread ứng dụng tiếp tục cập nhật bảng thứ hai ban đầu rỗng, còn thread optimizer thì xử lý bảng thứ nhất ban đầu đầy, không cần đồng bộ thêm.

**Hiệu quả nâng cao hiệu năng**:

- Trong các ứng dụng **thường xuyên sửa đổi các trường tham chiếu đối tượng**, thông lượng tăng **5-15%**
- Ngay cả trong các ứng dụng không thường xuyên sửa đổi trường tham chiếu, do barrier ghi được đơn giản hóa (trên x64 giảm từ khoảng 50 lệnh xuống chỉ còn 12 lệnh), thông lượng cũng có thể tăng lên tới **5%**
- Thời gian tạm dừng GC cũng có **giảm nhẹ**

**Chi phí bộ nhớ**:

Bảng thẻ thứ hai có dung lượng giống bảng thẻ thứ nhất, mỗi bảng thẻ cần 0.2% dung lượng Java heap, tức mỗi 1GB bộ nhớ heap dùng thêm khoảng 2MB bộ nhớ native.

## JEP 516: Bộ đệm đối tượng AOT hỗ trợ GC bất kỳ

Đây là cột mốc quan trọng của **Project Leyden**, giúp bộ đệm đối tượng Ahead-of-Time (AOT) có thể phối hợp sử dụng với **garbage collector bất kỳ**.

Trước đây trong JDK 24, AOT Class Data Sharing được giới thiệu (JEP 483) chỉ hỗ trợ G1 garbage collector, không thể phối hợp sử dụng với các GC khác như ZGC. Điều này là do các tham chiếu đối tượng được lưu trong bộ đệm AOT sử dụng địa chỉ bộ nhớ vật lý, trong khi bố cục bộ nhớ và chiến lược di chuyển đối tượng của các GC khác nhau là khác nhau.

JEP 516 đã thay đổi phương thức lưu trữ tham chiếu đối tượng từ **địa chỉ bộ nhớ vật lý** thành **index logic**:

- Sử dụng định dạng stream độc lập với GC để lưu bộ đệm
- Bộ đệm có thể được tải và phân tích bởi GC bất kỳ khi runtime
- JVM khi tải sẽ chuyển đổi index logic thành địa chỉ bộ nhớ thực tế

**Lợi ích hiệu năng**:

- **Tối ưu hóa thời gian khởi động**: Giảm đáng kể thời gian khởi động lạnh của ứng dụng Java
- **Hỗ trợ ZGC**: ZGC độ trễ thấp giờ đây cũng có thể hưởng lợi tăng tốc khởi động do bộ đệm AOT mang lại
- **Thân thiện với Cloud Native**: Đặc biệt có giá trị đối với các kịch bản nhạy cảm về thời gian khởi động như microservices và serverless functions

## JEP 500: Chuẩn bị để final thực sự bất biến

Tính năng này mở đường cho nguyên tắc ưu tiên tính toàn vẹn của Java, chuẩn bị làm cho các trường `final` thực sự trở nên bất biến.

Từ JDK 5 trở đi, để hỗ trợ các kịch bản như serialization, Reflection API cho phép sửa đổi một phần trường `final` thông qua **reflection sâu**:

```java
import java.lang.reflect.Field;

class Example {
    private final String name;

    Example(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

// 通过反射修改 final 字段
Example example = new Example("Original");
Field field = Example.class.getDeclaredField("name");
field.setAccessible(true);
field.set(example, "Modified");  // JDK 26 默认会成功，同时发出非法 final 字段修改警告
System.out.println(example.getName());  // 输出 "Modified"
```

Khả năng này mặc dù được một số framework (như các thư viện serialization, dependency injection framework, công cụ testing) sử dụng, nhưng làm phá hỏng đảm bảo tính bất biến của `final`, cũng cản trở tối ưu hóa của bộ biên dịch.

Trong JDK 26, khi sửa đổi trường `final` thông qua reflection sâu, JVM sẽ **phát ra cảnh báo**. Đây là sự chuẩn bị cho việc mặc định cấm các thao tác kiểu này trong các phiên bản tương lai.

Đối với các kịch bản thực sự cần sửa đổi trường `final`, JDK 26 cung cấp cơ chế lựa chọn hiển thị, cho phép nhà phát triển tiếp tục sử dụng khả năng này trong thời gian quá độ, đồng thời chuẩn bị sẵn sàng cho chế độ nghiêm ngặt trong tương lai.

## JEP 526: Hằng số lười（Lazy Constants, Xem trước lần 2）

Tính năng này xem trước lần đầu trong JDK 25 do [JEP 502: Stable Values (Xem trước)](https://openjdk.org/jeps/502) đề xuất; JEP 526 của JDK 26 đổi tên API thành Lazy Constants và tiến hành xem trước lần thứ 2.

Các trường `static final` truyền thống thông thường sẽ khởi tạo ngay khi class được khởi tạo, điều này sẽ:

- Tăng thời gian khởi động.
- Nếu hằng số đó chưa từng được sử dụng, thì lãng phí bộ nhớ.
- Cần các pattern khởi tạo lười (lazy initialization) phức tạp (như double-checked locking, Holder class pattern v.v.).

JEP 526 giới thiệu `LazyConstant<T>`, một đối tượng nắm giữ dữ liệu bất biến, JVM coi nó như hằng số thực sự, để có được hiệu năng tương tự như khai báo trường `final`.

```java
// 传统方式：类初始化时立即初始化
static final ExpensiveObject TRADITIONAL = new ExpensiveObject();

// 新方式：首次访问时才初始化
static final LazyConstant<ExpensiveObject> LAZY =
    LazyConstant.of(() -> new ExpensiveObject());

// 使用时
ExpensiveObject obj = LAZY.get();  // 此时才初始化
```

**Ưu điểm**:

- **Khởi tạo theo nhu cầu**: Chỉ khởi tạo khi truy cập lần đầu, nâng cao hiệu năng khởi động.
- **Thread safe**: Đảm bảo an toàn đa luồng dựng sẵn, không cần đồng bộ thủ công.
- **Tối ưu hóa JVM**: JVM có thể tối ưu hóa hằng số lười giống như đối xử với trường `final`.
- **Đơn giản hóa mã nguồn**: Loại bỏ các pattern khởi tạo lười phức tạp như double-checked locking.

## JEP 525: Structured Concurrency（Đồng thời cấu trúc, Xem trước lần 6）

JDK 19 đã giới thiệu Structured Concurrency dưới dạng Incubator API. Trong JDK 26, API này đang ở giai đoạn xem trước lần thứ 6, mục đích là đơn giản hóa lập trình đa luồng, không phải để thay thế `java.util.concurrent`.

Structured Concurrency coi nhiều tác vụ chạy trong các thread khác nhau là một đơn vị công việc đơn lẻ, từ đó đơn giản hóa việc xử lý lỗi, nâng cao độ tin cậy và tăng cường khả năng quan sát. Nghĩa là, Structured Concurrency giữ lại tính dễ đọc, tính dễ bảo trì và tính quan sát của mã đơn luồng.

API cơ bản của Structured Concurrency là `StructuredTaskScope`, nó hỗ trợ chia nhỏ tác vụ thành nhiều tác vụ con đồng thời, thực thi trong các thread của chính chúng, và các tác vụ con bắt buộc phải hoàn thành trước khi tác vụ chính/cha tiếp tục hoặc các tác vụ con bị hủy cùng với sự thất bại của tác vụ chính/cha.

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

**Thay đổi mới trong Java 26**:

- **Tăng cường Joiner**: Interface `Joiner` bổ sung phương thức `onTimeout()`, cho phép trả về kết quả cụ thể khi xảy ra timeout.
- **Tối ưu hóa kiểu trả về**: `allSuccessfulOrThrow()` hiện trả về trực tiếp danh sách kết quả (`List`), chứ không phải stream tác vụ con như trước.
- **Đơn giản hóa API**: Đổi tên đơn giản hóa `anySuccessfulResultOrThrow()` thành `anySuccessfulOrThrow()`.

## JEP 530: Khớp mẫu hỗ trợ kiểu nguyên thủy（Xem trước lần 4）

Lần xem trước đầu tiên của tính năng này do [JEP 455](https://openjdk.org/jeps/455 "JEP 455") (JDK 23) đề xuất.

Khớp mẫu có thể xử lý tất cả các kiểu dữ liệu nguyên thủy (`int`, `double`, `boolean`,...) trong các câu lệnh `switch` và `instanceof`.

```java
static void test(Object obj) {
    if (obj instanceof int i) {
        System.out.println("这是一个int类型: " + i);
    }
}
```

JDK 26 đã tăng cường thêm đối với tính năng này:

- Loại bỏ nhiều hạn chế liên quan đến kiểu nguyên thủy, giúp khớp mẫu, `instanceof` và `switch` trở nên thống nhất hơn và có sức diễn đạt tốt hơn.
- Tăng cường định nghĩa về tính chính xác vô điều kiện.
- Áp dụng kiểm tra tính áp đảo nghiêm ngặt hơn trong cấu trúc `switch`, giúp bộ biên dịch có thể nhận diện và giảm thiểu rủi ro lỗi lập trình rộng rãi hơn.

Như vậy có thể giống như xử lý kiểu đối tượng, tiến hành khớp kiểu và chuyển đổi an toàn hơn, ngắn gọn hơn đối với kiểu nguyên thủy, loại bỏ thêm mã boilerplate trong Java.

## JEP 524: Mã hóa PEM đối tượng mã hóa（Xem trước lần 2）

Lần xem trước đầu tiên của tính năng này do [JEP 518](https://openjdk.org/jeps/518) (JDK 25) đề xuất.

PEM (Privacy-Enhanced Mail) là một định dạng văn bản được sử dụng rộng rãi, dùng để lưu trữ và truyền tải các đối tượng mã hóa, như chứng chỉ, private key và public key. JEP 524 cung cấp một API mới, dùng để mã hóa đối tượng mã hóa thành định dạng PEM, cũng như giải mã từ định dạng PEM trở lại đối tượng mã hóa.

```java
// 将密钥编码为 PEM 格式
KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
kpg.initialize(2048);
KeyPair keyPair = kpg.generateKeyPair();

// 编码为 PEM
String pemEncoded = PEMEncoder.of().encodeToString(keyPair.getPrivate());

// 从 PEM 解码
PrivateKey decodedKey = PEMDecoder.of().decode(pemEncoded, PrivateKey.class);
```

API này làm giảm rủi ro lỗi, đơn giản hóa các yêu cầu tuân thủ, và thông qua đơn giản hóa thiết lập mã hóa và tích hợp cho nhu cầu doanh nghiệp, đám mây và quản lý, tăng cường tính di trú và khả năng tương tác của các ứng dụng Java an toàn.

## JEP 529: Vector API（Vector API, Lần ươm tạo thứ 11）

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

Mặc dù vẫn đang trong quá trình ươm tạo, nhưng lặp lại lần thứ 11 đủ để chứng minh tầm quan trọng của nó. Nó giúp Java trong các lĩnh vực nhạy cảm về hiệu năng như tính toán khoa học, machine learning, suy luận AI, xử lý dữ liệu lớn, có thể viết ra mã nguồn có hiệu năng tiệm cận thậm chí sánh ngang với các ngôn ngữ native như C++.

## JEP 504: Loại bỏ Applet API

Applet API bị đánh dấu loại bỏ trong JDK 9, bị đánh dấu chuẩn bị loại bỏ trong JDK 17. Trong JDK 26, Applet API cuối cùng đã bị **bỏ hoàn toàn**. Thật hả hê lòng người!

Điều này có nghĩa là:

- Class `java.applet.Applet` và các class liên quan đến nó đã bị xóa.
- Giảm thể tích cài đặt và dung lượng mã nguồn của JDK.
- Nâng cao hiệu năng, tính ổn định và tính an toàn của ứng dụng.

Công nghệ Applet từ lâu đã lỗi thời, phát triển Web hiện đại đã chuyển hoàn toàn sang các công nghệ khác. Loại bỏ API kế thừa này là bước đi tất yếu để hiện đại hóa nền tảng Java.

## Tóm tắt

JDK 26 mặc dù là một phiên bản không phải LTS, nhưng chứa một số tính năng quan trọng đáng chú ý:

| Thể loại | Tính năng |
| --- | --- |
| **Mạng** | Hỗ trợ HTTP/3 |
| **Hiệu năng** | Tối ưu hóa thông lượng G1 GC, Bộ đệm AOT hỗ trợ GC bất kỳ |
| **Ngôn ngữ** | Khớp mẫu hỗ trợ kiểu nguyên thủy (Xem trước lần 4), Hằng số lười (Xem trước lần 2) |
| **Đồng thời** | Đồng thời cấu trúc (Xem trước lần 6), Vector API (Lần ươm tạo thứ 11) |
| **An toàn** | Làm cho final thực sự bất biến, Hỗ trợ mã hóa PEM |
| **Dọn dẹp** | Loại bỏ Applet API |

Oracle sẽ cung cấp cập nhật cho đến tháng 9 năm 2026, khi đó sẽ được thay thế bởi Oracle JDK 27.
