---
title: Giải thích chi tiết Serialization trong Java
description: Phân tích sâu cơ chế Serialization và Deserialization trong Java: giải thích chi tiết interface Serializable, từ khóa transient, tác dụng của serialVersionUID, cách lựa chọn serialization protocol và các kịch bản ứng dụng như RPC, cache.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Java serialization,deserialization,interface Serializable,từ khóa transient,serialVersionUID,serialization protocol,lưu trữ đối tượng
---

## Serialization và Deserialization là gì?

Nếu chúng ta cần lưu trữ đối tượng Java (ví dụ lưu đối tượng Java vào file) hoặc truyền đối tượng Java qua mạng, những kịch bản này đều cần dùng tới Serialization.

Nói một cách đơn giản:

- **Serialization (Tuần tự hóa / Mã hóa đối tượng)**: Chuyển đổi cấu trúc dữ liệu hoặc đối tượng thành dạng có thể lưu trữ hoặc truyền đi, thông thường là luồng byte nhị phân (binary byte stream), cũng có thể là định dạng văn bản như JSON, XML.
- **Deserialization (Giải tuần tự hóa / Giải mã đối tượng)**: Quá trình chuyển đổi dữ liệu được sinh ra trong quá trình serialization trở lại thành cấu trúc dữ liệu hoặc đối tượng ban đầu.

Đối với ngôn ngữ lập trình hướng đối tượng như Java, thứ chúng ta serialize đều là đối tượng (Object) tức class đã được khởi tạo (Class), nhưng trong ngôn ngữ bán hướng đối tượng như C++, struct (cấu trúc) định nghĩa kiểu cấu trúc dữ liệu, còn class tương ứng với kiểu đối tượng.

Dưới đây là các kịch bản ứng dụng phổ biến của serialization và deserialization:

- Đối tượng trước khi truyền qua mạng (như khi gọi phương thức từ xa RPC) cần được serialize trước, sau khi nhận được đối tượng đã serialize cần deserialize lại;
- Trước khi lưu đối tượng vào file cần serialize, khi đọc đối tượng từ file ra cần deserialize;
- Trước khi lưu đối tượng vào database (như Redis) cần dùng serialize, khi đọc đối tượng từ cache database ra cần deserialize;
- Khi chuyển đối tượng thành dạng byte biểu diễn cần lưu trữ lâu dài hoặc truyền qua các component, thông thường cần serialization; đối tượng Java thông thường sử dụng trong bộ nhớ JVM thì không cần serialization.

Wikipedia giới thiệu về serialization như sau:

> **Tuần tự hóa** (serialization) trong xử lý dữ liệu của khoa học máy tính là quá trình chuyển đổi cấu trúc dữ liệu hoặc trạng thái đối tượng thành định dạng có thể truy xuất (ví dụ lưu thành file, lưu trong bộ đệm, hoặc gửi qua mạng), để phục vụ cho quá trình khôi phục lại trạng thái ban đầu sau này trong cùng môi trường máy tính hoặc một môi trường máy tính khác. Khi thu lại các byte theo định dạng tuần tự hóa, có thể tận dụng nó để tạo ra bản sao có cùng ngữ nghĩa với đối tượng ban đầu. Với nhiều đối tượng, như các đối tượng phức tạp sử dụng lượng lớn tham chiếu, quá trình tái tạo tuần tự hóa này không dễ dàng. Tuần tự hóa đối tượng trong hướng đối tượng không khái quát các hàm liên quan của đối tượng ban đầu. Quá trình này còn gọi là marshalling. Thao tác ngược lại trích xuất cấu trúc dữ liệu từ một chuỗi byte là deserialization (còn gọi là unmarshalling).

Tóm lại: **Mục đích chính của Serialization là chuyển đối tượng thành dạng phù hợp để truyền qua mạng hoặc lưu trữ lâu dài vào file system, database, cache,...**

![](https://oss.javaguide.cn/github/javaguide/a478c74d-2c48-40ae-9374-87aacf05188c.png)

<p style="text-align:right;font-size:13px;color:gray">https://www.corejavaguru.com/java/serialization/interview-questions-1</p>

**Protocol Serialization tương ứng với tầng nào trong mô hình 4 tầng TCP/IP?**

Chúng ta biết hai bên truyền thông mạng bắt buộc phải áp dụng và tuân thủ cùng một protocol. Mô hình 4 tầng TCP/IP như hình bên dưới, vậy protocol serialization thuộc tầng nào?

1. Tầng ứng dụng (Application Layer)
2. Tầng giao vận (Transport Layer)
3. Tầng mạng (Network Layer)
4. Tầng truy cập mạng (Network Interface Layer)

![Mô hình 4 tầng TCP/IP](https://oss.javaguide.cn/github/javaguide/cs-basics/network/tcp-ip-4-model.png)

Như hình trên, trong mô hình 7 tầng OSI, công việc của tầng Trình diễn (Presentation Layer) chủ yếu là xử lý dữ liệu người dùng từ tầng ứng dụng để chuyển thành luồng nhị phân. Ngược lại là chuyển luồng nhị phân thành dữ liệu người dùng của tầng ứng dụng. Đây chính là tương ứng với serialization và deserialization đúng không?

Vì tầng Ứng dụng, tầng Trình diễn và tầng Phiên trong mô hình 7 tầng OSI tương ứng với tầng Ứng dụng trong mô hình 4 tầng TCP/IP, nên protocol serialization thuộc về một phần của tầng Ứng dụng trong protocol TCP/IP.

## Các Protocol Serialization phổ biến là gì?

Cách serialization tích hợp sẵn của JDK thường không được sử dụng vì hiệu suất serialization thấp và tồn tại vấn đề an toàn bảo mật. Các serialization protocol phổ biến hơn gồm Hessian, Kryo, Protobuf, ProtoStuff, đây đều là các serialization protocol dựa trên nhị phân.

Các dạng như JSON và XML thuộc về phương thức serialization dạng văn bản (text). Mặc dù tính đọc hiểu tốt hơn nhưng hiệu năng kém hơn, thông thường không lựa chọn.

### Cách Serialization tích hợp của JDK

Cách serialization tích hợp của JDK chỉ cần implement interface `java.io.Serializable` là được.

```java
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Builder
@ToString
public class RpcRequest implements Serializable {
    private static final long serialVersionUID = 1905122041950251207L;
    private String requestId;
    private String interfaceName;
    private String methodName;
    private Object[] parameters;
    private Class<?>[] paramTypes;
    private RpcMessageTypeEnum rpcMessageTypeEnum;
}
```

**serialVersionUID có tác dụng gì?**

Mã số serialization `serialVersionUID` đóng vai trò kiểm soát phiên bản (version control). Khi deserialize, nó sẽ kiểm tra xem `serialVersionUID` trong luồng dữ liệu có khớp với `serialVersionUID` của lớp hiện tại hay không; nếu không khớp sẽ throw `InvalidClassException`. Khuyến nghị mạnh mẽ mỗi lớp serializable đều nên chỉ định thủ công `serialVersionUID`. Nếu không khai báo rõ ràng, Java runtime sẽ tự tính toán giá trị mặc định dựa trên cấu trúc của class chứ không phải do trình biên dịch sinh ra field.

**serialVersionUID được修饰 bởi từ khóa static, tại sao vẫn được “serialize”?**

~~Biến được修饰 bởi `static` là biến tĩnh nằm trong vùng nhớ Metaspace/Method Area, bản thân không được serialize. Biến `static` thuộc về class chứ không thuộc đối tượng. Sau khi deserialize, giá trị biến `static` giống như mặc định được gán cho đối tượng, nhìn tưởng như biến `static` được serialize nhưng thực chất chỉ là hiện tượng giả mà thôi.~~

**🐛 Sửa lỗi (Xem: [issue#2174](https://github.com/Snailclimb/JavaGuide/issues/2174))**:

Trong trường hợp thông thường, biến `static` thuộc về class, không thuộc về bất kỳ đối tượng đơn lẻ nào, do đó bản thân chúng không được đưa vào luồng dữ liệu serialization của đối tượng. Serialization lưu trữ trạng thái của đối tượng (tức giá trị các biến instance). Tuy nhiên `serialVersionUID` là một trường hợp đặc biệt, quá trình serialization của `serialVersionUID` được xử lý riêng. Điểm mấu chốt là `serialVersionUID` không được serialize như một phần trạng thái đối tượng, mà được bản thân cơ chế serialization sử dụng làm "dấu vân tay" hoặc "mã phiên bản" đặc biệt.

Khi một đối tượng được serialize, `serialVersionUID` sẽ được ghi vào luồng nhị phân serialization (như lưu lại một mã phiên bản chứ không phải lưu trạng thái của biến `static` đó); khi deserialize, nó cũng sẽ được parse và so sánh tính nhất quán để xác minh tính tương thích phiên bản của đối tượng serialize. Nếu hai bên không khớp, quá trình deserialize sẽ throw `InvalidClassException`, vì điều này thường có nghĩa là định nghĩa của lớp serialize đã bị thay đổi và có thể không còn tương thích nữa.

Giải thích chính thức như sau:

> A serializable class can declare its own serialVersionUID explicitly by declaring a field named `"serialVersionUID"` that must be `static`, `final`, and of type `long`;
>
> Nếu muốn chỉ định rõ `serialVersionUID`, cần khai báo một biến kiểu `long` có tên là `"serialVersionUID"` sử dụng hai từ khóa `static` và `final` trong lớp.

Nghĩa là bản thân `serialVersionUID` (với tư cách là biến static) quả thực không được serialize làm trạng thái đối tượng. Tuy nhiên giá trị của nó được cơ chế Java serialization xử lý đặc biệt — được đọc và ghi vào luồng serialization như một nhận dạng phiên bản, dùng để kiểm tra tính tương thích phiên bản khi deserialize.

**Nếu có một số field không muốn serialize thì làm thế nào?**

Đối với các biến không muốn serialize, sử dụng từ khóa `transient` để修饰.

Tác dụng của từ khóa `transient` là: Ngăn cản các biến trong instance được修饰 bởi từ khóa này serialize; khi đối tượng được deserialize, giá trị của biến được修饰 bởi `transient` sẽ không được lưu trữ và khôi phục.

Một số lưu ý về `transient`:

- `transient` chỉ có thể修饰 biến, không thể修饰 class và method.
- Biến được修饰 bởi `transient` sau khi deserialize sẽ được đặt về giá trị mặc định của kiểu đó. Ví dụ nếu修饰 kiểu `int` thì kết quả sau deserialize là `0`.
- Biến `static` vì không thuộc về bất kỳ đối tượng (Object) nào nên dù có từ khóa `transient` hay không thì cũng đều không được serialize.

**Tại sao không khuyến nghị sử dụng Serialization tích hợp của JDK?**

Chúng ta rất ít hoặc gần như không trực tiếp sử dụng cách serialization tích hợp của JDK, nguyên nhân chính gồm:

- **Không hỗ trợ gọi đa ngôn ngữ**: Không hỗ trợ khi gọi các dịch vụ phát triển bằng ngôn ngữ khác.
- **Hiệu năng kém**: So với các serialization framework khác thì hiệu năng thấp hơn, nguyên nhân chính là mảng byte sau khi serialize có kích thước lớn, làm tăng chi phí truyền tải.
- **Tồn tại vấn đề an toàn bảo mật**: Bản thân serialization và deserialization không có vấn đề. Nhưng khi dữ liệu đầu vào để deserialization có thể bị kiểm soát bởi người dùng, kẻ tấn công có thể dựng đầu vào độc hại để deserialization tạo ra đối tượng ngoài dự kiến, từ đó thực thi mã tùy ý được dựng sẵn. Đọc thêm: [An toàn ứng dụng: Lỗ hổng Java Deserialization - Cryin](https://cryin.github.io/blog/secure-development-java-deserialization-vulnerability/), [Lỗ hổng an toàn Java Deserialization là gì? - Monica](https://www.zhihu.com/question/37562657/answer/1916596031).

### Kryo

Kryo là một công cụ serialization/deserialization hiệu năng cao. Nhờ đặc tính lưu trữ độ dài biến đổi và sử dụng cơ chế sinh bytecode, Kryo có tốc độ chạy rất nhanh và kích thước bytecode nhỏ gọn.

Ngoài ra, Kryo là một bản thực thi serialization rất chín mùi, được sử dụng rộng rãi trong Twitter, Groupon, Yahoo và nhiều dự án mã nguồn mở nổi tiếng (như Hive, Storm).

Dự án [guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) sử dụng Kryo để serialize, đoạn code liên quan như sau:

```java
/**
 * Kryo serialization class, Kryo serialization efficiency is very high, but only compatible with Java language
 *
 * @author shuang.kou
 * @createTime 2020年05月13日 19:29:00
 */
@Slf4j
public class KryoSerializer implements Serializer {

    /**
     * Because Kryo is not thread safe. So, use ThreadLocal to store Kryo objects
     */
    private final ThreadLocal<Kryo> kryoThreadLocal = ThreadLocal.withInitial(() -> {
        Kryo kryo = new Kryo();
        kryo.register(RpcResponse.class);
        kryo.register(RpcRequest.class);
        return kryo;
    });

    @Override
    public byte[] serialize(Object obj) {
        try (ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
             Output output = new Output(byteArrayOutputStream)) {
            Kryo kryo = kryoThreadLocal.get();
            // Object->byte: Serialize đối tượng thành mảng byte
            kryo.writeObject(output, obj);
            kryoThreadLocal.remove();
            return output.toBytes();
        } catch (Exception e) {
            throw new SerializeException("Serialization failed");
        }
    }

    @Override
    public <T> T deserialize(byte[] bytes, Class<T> clazz) {
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
             Input input = new Input(byteArrayInputStream)) {
            Kryo kryo = kryoThreadLocal.get();
            // byte->Object: Deserialize từ mảng byte ra đối tượng
            Object o = kryo.readObject(input, clazz);
            kryoThreadLocal.remove();
            return clazz.cast(o);
        } catch (Exception e) {
            throw new SerializeException("Deserialization failed");
        }
    }

}
```

Địa chỉ GitHub: [https://github.com/EsotericSoftware/kryo](https://github.com/EsotericSoftware/kryo).

### Protobuf

Protobuf đến từ Google, hiệu năng rất xuất sắc, hỗ trợ nhiều ngôn ngữ và là cross-platform. Điểm trừ là cách dùng hơi rườm rà vì bạn phải tự định nghĩa file IDL và sinh ra code serialization tương ứng. Tuy điều này thiếu linh hoạt, nhưng ở một khía cạnh khác khiến Protobuf không có rủi ro lỗ hổng serialization.

> Protobuf bao gồm định nghĩa định dạng serialization, thư viện cho các ngôn ngữ và một trình biên dịch IDL. Thông thường bạn cần định nghĩa file proto, sau đó dùng trình biên dịch IDL biên dịch thành ngôn ngữ bạn cần.

Một file proto đơn giản như sau:

```protobuf
// Phiên bản protobuf
syntax = "proto3";
// Person sẽ được biên dịch thành đối tượng tương ứng của các ngôn ngữ khác nhau, như class trong Java, struct trong Go
message Person {
  // Field kiểu string
  string name = 1;
  // Field kiểu int
  int32 age = 2;
}
```

Địa chỉ GitHub: [https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf).

### ProtoStuff

Do tính dễ dùng của Protobuf hơi kém nên "người anh em" Protostuff ra đời.

Protostuff dựa trên Google Protobuf nhưng cung cấp nhiều tính năng hơn và cách dùng đơn giản hơn. Tuy dễ dùng hơn nhưng không có nghĩa là ProtoStuff có hiệu năng kém hơn.

Địa chỉ GitHub: [https://github.com/protostuff/protostuff](https://github.com/protostuff/protostuff).

### Hessian

Hessian là một protocol RPC nhị phân nhẹ, tự mô tả. Hessian là một bản thực thi serialization khá lâu đời và cũng hỗ trợ đa ngôn ngữ.

![](https://oss.javaguide.cn/github/javaguide/8613ec4c-bde5-47bf-897e-99e0f90b9fa3.png)

Dubbo 2.x mặc định bật phương thức serialization là Hessian2, tuy nhiên Dubbo đã chỉnh sửa Hessian2 một chút, cấu trúc tổng thể vẫn tương tự.

### Tóm tắt

Kryo là phương thức serialization chuyên dành cho ngôn ngữ Java với hiệu năng cực tốt, nếu ứng dụng của bạn chuyên biệt cho Java thì có thể cân nhắc sử dụng. Bài viết trên trang chủ Dubbo cũng khuyến nghị dùng Kryo làm phương thức serialization cho môi trường production. (Địa chỉ bài viết: <https://cn.dubbo.apache.org/zh-cn/docsv2.7/user/serialization/>).

![](https://oss.javaguide.cn/github/javaguide/java/569e541a-22b2-4846-aa07-0ad479f07440-20230814090158124.png)

Các loại như Protobuf, ProtoStuff, Hessian đều là phương thức serialization đa ngôn ngữ, nếu có nhu cầu đa ngôn ngữ thì có thể cân nhắc sử dụng.

Ngoài các phương thức serialization tôi đã giới thiệu ở trên, còn có các loại như Thrift, Avro.

<!-- @include: @article-footer.snippet.md -->
