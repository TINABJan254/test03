---
title: 类文件结构详解
description: 介绍 Java 字节码 Class 文件结构与常量池等核心组成，辅助理解编译产物。
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: Class 文件,常量池,魔数,版本,字段,方法,属性
---

## Nhắc lại về Bytecode

Trong Java, code mà JVM có thể hiểu được gọi là `Bytecode` (tức các file có phần mở rộng `.class`), nó không hướng tới bất kỳ bộ xử lý cụ thể nào, mà chỉ hướng tới Virtual Machine. Ngôn ngữ Java thông qua phương thức bytecode, ở một mức độ nhất định đã giải quyết được vấn đề hiệu năng thực thi thấp của các ngôn ngữ thông dịch truyền thống, đồng thời vẫn giữ được đặc tính có thể di trú (portable) của ngôn ngữ thông dịch. Do đó chương trình Java khi chạy khá hiệu quả, hơn nữa, vì bytecode không nhắm tới một loại máy cụ thể nào, do đó chương trình Java không cần biên dịch lại mà có thể chạy trên máy tính thuộc nhiều hệ điều hành khác nhau.

Các ngôn ngữ như Clojure (một biến thể của ngôn ngữ Lisp), Groovy, Scala, JRuby, Kotlin v.v. đều chạy trên Java Virtual Machine. Hình dưới hiển thị các ngôn ngữ khác nhau được các compiler khác nhau biên dịch thành file `.class` cuối cùng chạy trên Java Virtual Machine. Định dạng nhị phân của file `.class` có thể xem bằng [WinHex](https://www.x-ways.net/winhex/).

![Các ngôn ngữ lập trình chạy trên Java Virtual Machine](https://oss.javaguide.cn/github/javaguide/java/basis/java-virtual-machine-program-language-os.png)

Có thể nói file `.class` là chiếc cầu nối quan trọng giữa các ngôn ngữ khác nhau trên Java Virtual Machine, đồng thời cũng là một lý do rất quan trọng hỗ trợ Java cross-platform.

## Tóm tắt cấu trúc file Class

Theo quy chuẩn Java Virtual Machine, file Class được định nghĩa thông qua `ClassFile`, hơi giống struct trong ngôn ngữ C.

Cấu trúc của `ClassFile` như sau:

```java
ClassFile {
    u4             magic; // Cờ định danh của file Class
    u2             minor_version; // Phiên bản phụ của Class
    u2             major_version; // Phiên bản chính của Class
    u2             constant_pool_count; // Số lượng trong Constant Pool
    cp_info        constant_pool[constant_pool_count-1]; // Constant Pool
    u2             access_flags; // Cờ truy cập của Class
    u2             this_class; // Class hiện tại
    u2             super_class; // Class cha
    u2             interfaces_count; // Số lượng interface
    u2             interfaces[interfaces_count]; // Một class có thể implement nhiều interface
    u2             fields_count; // Số lượng field
    field_info     fields[fields_count]; // Một class có thể có nhiều field
    u2             methods_count; // Số lượng phương thức
    method_info    methods[methods_count]; // Một class có thể có nhiều phương thức
    u2             attributes_count; // Số lượng thuộc tính trong bảng thuộc tính của Class này
    attribute_info attributes[attributes_count]; // Tập hợp bảng thuộc tính
}
```

Thông qua phân tích nội dung `ClassFile`, chúng ta sẽ biết được thành phần của file class.

![Phân tích nội dung ClassFile](https://oss.javaguide.cn/java-guide-blog/16d5ec47609818fc.jpeg)

Bức hình dưới đây xem qua plugin IDEA `jclasslib`, bạn có thể thấy cấu trúc file Class trực quan hơn.

![](https://oss.javaguide.cn/java-guide-blog/image-20210401170711475.png)

Sử dụng `jclasslib` không chỉ có thể xem trực quan file bytecode tương ứng của một class, mà còn xem được thông tin cơ bản của class, Constant Pool, interface, thuộc tính, hàm v.v.

Dưới đây sẽ giới thiệu chi tiết về một số thành phần liên quan trong cấu trúc file Class.

### Magic Number (Số kỳ diệu)

```java
    u4             magic; // Cờ định danh của file Class
```

4 byte đầu tiên của mỗi file Class được gọi là Magic Number (Số kỳ diệu), tác dụng duy nhất của nó là **xác định xem file này có phải là một file Class mà Virtual Machine có thể tiếp nhận hay không**. Quy chuẩn Java quy định Magic Number là một giá trị cố định: 0xCAFEBABE. Nếu file được đọc không bắt đầu bằng Magic Number này, Java Virtual Machine sẽ từ chối nạp nó.

### Phiên bản file Class (Minor & Major Version)

```java
    u2             minor_version; // Phiên bản phụ của Class
    u2             major_version; // Phiên bản chính của Class
```

4 byte tiếp theo sau Magic Number lưu trữ phiên bản của file Class: Byte thứ 5 và 6 là **phiên bản phụ (minor version)**, byte thứ 7 và 8 là **phiên bản chính (major version)**.

Mỗi khi Java phát hành phiên bản lớn (như Java 8, Java 9), phiên bản chính đều tăng thêm 1. Bạn có thể sử dụng lệnh `javap -v` để xem nhanh thông tin phiên bản của file Class.

Java Virtual Machine phiên bản cao có thể thực thi file Class được tạo bởi compiler phiên bản thấp, nhưng Java Virtual Machine phiên bản thấp không thể thực thi file Class được tạo bởi compiler phiên bản cao. Do đó, trong phát triển thực tế chúng ta phải đảm bảo phiên bản JDK phát triển và phiên bản JDK môi trường production giữ nguyên sự nhất quán.

### Constant Pool (Tập hằng số)

```java
    u2             constant_pool_count; // Số lượng trong Constant Pool
    cp_info        constant_pool[constant_pool_count-1]; // Constant Pool
```

Tiếp theo sau phiên bản chính phụ là Constant Pool, số lượng của Constant Pool là `constant_pool_count-1` (**Bộ đếm Constant Pool bắt đầu đếm từ 1, việc để trống mục hằng số thứ 0 có cân nhắc đặc biệt, giá trị chỉ số 0 đại diện cho "không reference tới bất kỳ mục Constant Pool nào"**).

Constant Pool chủ yếu lưu trữ hai loại hằng số lớn: Literal và Symbolic Reference. Literal khá gần với khái niệm hằng số ở cấp độ ngôn ngữ Java, như chuỗi văn bản, giá trị hằng số khai báo `final` v.v. Còn Symbolic Reference thuộc về khái niệm nguyên lý biên dịch. Bao gồm 3 loại hằng số dưới đây:

- FQCN của Class và Interface
- Tên và descriptor của Field
- Tên và descriptor của Phương thức

Mỗi mục hằng số trong Constant Pool đều là một bảng. Quy chuẩn 《Java Virtual Machine Specification》 hiện tại định nghĩa 17 loại bảng Constant Pool, chúng đều có một đặc điểm chung: **Đầu bảng là một `tag` kiểu `u1`, dùng để nhận biết kiểu của hằng số hiện tại.**

| Kiểu                             | Cờ (tag)    | Mô tả                                   |
| :------------------------------: | :---------: | :-------------------------------------: |
|        CONSTANT_utf8_info        |      1      | Chuỗi ký tự mã hóa UTF-8                |
|      CONSTANT_Integer_info       |      3      | Literal kiểu số nguyên                  |
|       CONSTANT_Float_info        |      4      | Literal kiểu số thực                    |
|        CONSTANT_Long_info        |      5      | Literal kiểu số nguyên dài              |
|       CONSTANT_Double_info       |      6      | Literal kiểu số thực độ chính xác kép   |
|       CONSTANT_Class_info        |      7      | Symbolic Reference của Class/Interface  |
|       CONSTANT_String_info       |      8      | Literal kiểu chuỗi                      |
|      CONSTANT_FieldRef_info      |      9      | Symbolic Reference của Field            |
|     CONSTANT_MethodRef_info      |     10      | Symbolic Reference của phương thức Class|
| CONSTANT_InterfaceMethodRef_info |     11      | Symbolic Reference của phương thức Interface|
|    CONSTANT_NameAndType_info     |     12      | Symbolic Reference của Field/Phương thức|
|     CONSTANT_MethodType_info     |     16      | Biểu thị kiểu phương thức               |
|    CONSTANT_MethodHandle_info    |     15      | Biểu thị Method Handle                  |
|      CONSTANT_Dynamic_info       |     17      | Biểu thị hằng số tính toán động         |
|   CONSTANT_InvokeDynamic_info    |     18      | Biểu thị một điểm gọi phương thức động  |
|       CONSTANT_Module_info       |     19      | Biểu thị Module                         |
|      CONSTANT_Package_info       |     20      | Biểu thị Package trong Module           |

File `.class` có thể thông qua lệnh `javap -v TênClass` để xem thông tin trong Constant Pool (`javap -v TênClass -> temp.txt`: Xuất kết quả ra file temp.txt).

### Access Flags (Cờ truy cập)

```java
    u2             access_flags; // Cờ truy cập của Class
```

Sau khi Constant Pool kết thúc, hai byte tiếp theo đại diện cho cờ truy cập, cờ này dùng để nhận biết một số thông tin truy cập ở cấp độ Class hoặc Interface, bao gồm: Class này là class hay interface, có phải kiểu `public` hoặc `abstract` hay không, nếu là class thì có khai báo là `final` hay không v.v.

Cờ truy cập Class và thuộc tính:

![Cờ truy cập Class và thuộc tính](https://oss.javaguide.cn/github/javaguide/java/%E8%AE%BF%E9%97%AE%E6%A0%87%E5%BF%97.png)

Chúng ta định nghĩa một class `Employee`:

```java
package top.snailclimb.bean;
public class Employee {
   ...
}
```

Xem cờ truy cập của class thông qua lệnh `javap -v TênClass`.

![Xem cờ truy cập của class](https://oss.javaguide.cn/github/javaguide/java/%E6%9F%A5%E7%9C%8B%E7%B1%BB%E7%9A%84%E8%AE%BF%E9%97%AE%E6%A0%87%E5%BF%97.png)

### Chỉ số Class hiện tại (This Class), Class cha (Super Class), Tập hợp chỉ số Interface (Interfaces)

```java
    u2             this_class; // Class hiện tại
    u2             super_class; // Class cha
    u2             interfaces_count; // Số lượng interface
    u2             interfaces[interfaces_count]; // Một class có thể implement nhiều interface
```

Quan hệ kế thừa của Class Java do 3 mục chỉ số Class, chỉ số Class cha và tập hợp chỉ số Interface xác định. Chỉ số Class, chỉ số Class cha và tập hợp chỉ số Interface được xếp theo thứ tự sau cờ truy cập.

Chỉ số Class dùng để xác định FQCN của class này, chỉ số Class cha dùng để xác định FQCN của class cha của class này, do ngôn ngữ Java là đơn kế thừa, nên chỉ số Class cha chỉ có 1, trừ `java.lang.Object` ra, tất cả các class Java đều có class cha, do đó trừ `java.lang.Object` ra, chỉ số Class cha của tất cả các class Java đều không bằng 0.

Tập hợp chỉ số Interface dùng để mô tả class này implement những interface nào, các interface được implement này sẽ được sắp xếp theo thứ tự interface sau `implements` (nếu bản thân class này là interface thì là `extends`) từ trái sang phải trong tập hợp chỉ số Interface.

### Tập hợp bảng Field (Fields)

```java
    u2             fields_count; // Số lượng field
    field_info     fields[fields_count]; // Một class có thể có nhiều field
```

Bảng Field (field info) dùng để mô tả các biến khai báo trong interface hoặc class. Field bao gồm biến cấp Class cũng như biến instance, nhưng không bao gồm các biến cục bộ khai báo bên trong phương thức.

**Cấu trúc của field info (Bảng field):**

![Cấu trúc bảng field](https://oss.javaguide.cn/github/javaguide/java/%E5%AD%97%E6%AE%B5%E8%A1%A8%E7%9A%84%E7%BB%93%E6%9E%84.png)

- **access_flags:** Phạm vi tác dụng của field (modifier `public`, `private`, `protected`), là biến instance hay biến Class (modifier `static`), có thể serialize hay không (modifier `transient`), tính khả biến (`final`), tính hiển thị (modifier `volatile`, có bắt buộc đọc ghi từ bộ nhớ chính hay không).
- **name_index:** Reference tới Constant Pool, biểu thị tên của field;
- **descriptor_index:** Reference tới Constant Pool, biểu thị descriptor của field và phương thức;
- **attributes_count:** Một field còn sở hữu một số thuộc tính bổ sung, attributes_count lưu trữ số lượng thuộc tính;
- **attributes[attributes_count]:** Lưu trữ nội dung cụ thể của thuộc tính.

Trong các thông tin trên, từng modifier đều là giá trị boolean, hoặc có modifier nào đó hoặc không có, rất phù hợp sử dụng cờ bit để biểu thị. Còn field tên gì, field được định nghĩa là kiểu dữ liệu nào thì không thể cố định được, chỉ có thể reference hằng số trong Constant Pool để mô tả.

**Các giá trị access_flag của Field:**

![Giá trị access_flag của Field](https://oss.javaguide.cn/github/javaguide/java/jvm/class-file-fields-access_flag.png)

### Tập hợp bảng Phương thức (Methods)

```java
    u2             methods_count; // Số lượng phương thức
    method_info    methods[methods_count]; // Một class có thể có nhiều phương thức
```

methods_count biểu thị số lượng phương thức, còn method_info biểu thị bảng phương thức.

Mô tả phương thức trong định dạng lưu trữ file Class hầu như áp dụng phương thức hoàn toàn nhất quán với mô tả field. Cấu trúc bảng phương thức giống như bảng field, lần lượt bao gồm các mục cờ truy cập, chỉ số tên, chỉ số descriptor, tập hợp bảng thuộc tính.

**Cấu trúc method_info (Bảng phương thức):**

![Cấu trúc bảng phương thức](https://oss.javaguide.cn/github/javaguide/java/%E6%96%B9%E6%B3%95%E8%A1%A8%E7%9A%84%E7%BB%93%E6%9E%84.png)

**Các giá trị access_flag của Bảng phương thức:**

![Giá trị access_flag của Bảng phương thức](https://oss.javaguide.cn/github/javaguide/java/jvm/class-file-methods-access_flag.png)

Lưu ý: Vì modifier `volatile` và `transient` không thể dùng cho phương thức, nên trong cờ truy cập của bảng phương thức không có 2 cờ tương ứng này, nhưng bổ sung các keyword `synchronized`, `native`, `abstract` v.v. modifier cho phương thức, do đó cũng có thêm các cờ tương ứng với các keyword này.

### Tập hợp bảng Thuộc tính (Attributes)

```java
   u2             attributes_count; // Số lượng thuộc tính trong bảng thuộc tính của Class này
   attribute_info attributes[attributes_count]; // Tập hợp bảng thuộc tính
```

Trong file Class, bảng field, bảng phương thức đều có thể mang tập hợp bảng thuộc tính của riêng mình, để dùng mô tả thông tin chuyên biệt cho các kịch bản nào đó. Khác với thứ tự, độ dài và nội dung mà các mục dữ liệu khác trong file Class yêu cầu, giới hạn của tập hợp bảng thuộc tính hơi linh hoạt hơn, không còn yêu cầu các bảng thuộc tính phải có thứ tự nghiêm ngặt, và chỉ cần không trùng với tên thuộc tính đã có, bất kỳ compiler nào triển khai cũng có thể ghi thông tin thuộc tính do mình tự định nghĩa vào bảng thuộc tính, Java Virtual Machine lúc runtime sẽ bỏ qua các thuộc tính mà nó không nhận biết được.

## Tham khảo

- 《Thực chiến Java Virtual Machine》
- Chapter 4. The class File Format - Java Virtual Machine Specification: <https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html>
- Phân tích ví dụ cấu trúc file JAVA CLASS: <https://coolshell.cn/articles/9229.html>
- 《Minh họa nguyên lý Java Virtual Machine》 1.2.2, Giải thích chi tiết Constant Pool trong file Class (Thượng): <https://blog.csdn.net/luanlouis/article/details/39960815>

<!-- @include: @article-footer.snippet.md -->
