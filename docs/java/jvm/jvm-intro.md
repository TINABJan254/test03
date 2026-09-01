---
title: 大白话带你认识 JVM
description: 用通俗方式介绍 JVM 的基本组成与类加载执行流程，帮助快速入门虚拟机原理。
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: JVM 基础,类加载,方法区,堆栈,程序计数器,运行时数据区
---

> Bài viết đóng góp từ [说出你的愿望吧丷](https://juejin.im/user/5c2400afe51d45451758aa96), địa chỉ bài gốc: <https://juejin.im/post/5e1505d0f265da5d5d744050>.

## Lời nói đầu

Nếu trong bài viết có vấn đề về cách dùng từ hoặc thấu hiểu, hoan nghênh bạn đóng góp ý kiến. Bài viết này nhằm mục đích nêu ra các điểm kiến thức một cách dễ hiểu nhất mà không đi quá sâu.

## I. Giới thiệu cơ bản về JVM

JVM là viết tắt của Java Virtual Machine, nó là một máy tính ảo tưởng, một quy chuẩn. Thông qua việc mô phỏng thực tế các chức năng máy tính trên máy tính thực tế···

Được rồi, bỏ qua những câu từ chuyên môn phức tạp như vậy, bạn chỉ cần biết JVM thực chất tương tự như một chiếc máy tính nhỏ chạy trong môi trường hệ điều hành như Windows hoặc Linux là được. Nó tương tác trực tiếp với hệ điều hành chứ không tương tác trực tiếp với phần cứng, còn hệ điều hành sẽ giúp chúng ta hoàn thành công việc tương tác với phần cứng.

![](https://static001.geekbang.org/infoq/da/da0380a04d9c04facd2add5f6dba06fa.png)

### 1.1 File Java được chạy như thế nào

Ví dụ bây giờ chúng ta viết một file `HelloWorld.java`, bỏ qua mọi thứ thì `HelloWorld.java` có phải tương tự như một file văn bản (text file), chỉ có điều file này viết toàn bằng tiếng Anh và có lề nhất định.

**JVM** của chúng ta không đọc được file văn bản, nên nó cần được **biên dịch (compile)** để trở thành file nhị phân **`HelloWorld.class`** mà JVM có thể đọc được.

#### ① ClassLoader (Bộ nạp lớp)

Nếu **JVM** muốn thực thi file **`.class`** này, chúng ta cần đưa nó vào trong một **ClassLoader**, ClassLoader giống như một người thợ bốc vác, sẽ vận chuyển tất cả các file **`.class`** vào trong JVM.

![](https://static001.geekbang.org/infoq/2f/2f012fde94376f43a25dbe1dd07e0dd8.png)

#### ② Method Area (Vùng phương thức)

**Method Area** dùng để lưu trữ các dữ liệu thông tin dạng metadata, ví dụ như thông tin Class, hằng số, biến static, code sau khi biên dịch··· v.v.

Khi ClassLoader vận chuyển file `.class` tới, trước tiên sẽ thả vào vùng bộ nhớ này.

#### ③ Heap (Bộ nhớ Heap)

**Heap** chủ yếu lưu trữ các instance đối tượng và mảng v.v., nó và Method Area đều thuộc về **vùng bộ nhớ dùng chung giữa các thread (Thread-shared)**. Dùng chung giữa các thread chỉ thể hiện nhiều thread có thể truy cập các vùng này, chứ không đồng nghĩa với bản thân vùng bộ nhớ đó "không an toàn với thread" (thread-unsafe); việc có xảy ra xung đột dữ liệu hay không phụ thuộc vào cách chương trình truy cập dữ liệu trong đó.

#### ④ Stack (Stack của JVM)

**Stack** đây là không gian chạy code của chúng ta. Mỗi phương thức mà chúng ta viết đều sẽ được đưa vào trong **Stack** để chạy.

Chúng ta từng nghe qua hai thuật ngữ Native Method Stack hoặc Java Native Interface (JNI). Phương thức Native được triển khai bởi code bản địa bên ngoài Java, ngôn ngữ triển khai phổ biến là C hoặc C++, triển khai cụ thể phụ thuộc vào Virtual Machine và thư viện bản địa.

#### ⑤ Program Counter (Bộ đếm chương trình)

Program Counter ghi lại địa chỉ lệnh JVM hiện đang thực thi của thread; sau khi thực thi rẽ nhánh, vòng lặp, xử lý exception hoặc chuyển đổi thread, Virtual Machine dựa vào nó để tiếp tục thực thi. Nó cũng giống như Stack thuộc về vùng **riêng cho từng thread (Thread-private)**. Khi thực thi phương thức Native, quy chuẩn không quy định giá trị của nó.

![](https://static001.geekbang.org/infoq/c6/c602f57ea9297f50bbc265f1821d6263.png)

#### Tóm tắt ngắn gọn

1. File Java sau khi biên dịch chuyển thành file bytecode `.class`.
2. File bytecode được vận chuyển vào JVM Virtual Machine thông qua ClassLoader.
3. Trong Runtime Data Area của Virtual Machine, Method Area và Heap là các vùng dùng chung giữa các thread; Virtual Machine Stack, Native Method Stack và Program Counter là các vùng riêng cho từng thread. Việc dùng chung hay không là phạm vi hiển thị của vùng bộ nhớ, không đồng nhất trực tiếp với an toàn hay không an toàn đối với thread.

### 1.2 Ví dụ code đơn giản

Một class Student đơn giản:

![](https://static001.geekbang.org/infoq/12/12f0b239db65b8a95f0ce90e9a580e4d.png)

Một phương thức main:

![](https://static001.geekbang.org/infoq/0c/0c6d94ab88a9f2b923f5fea3f95bc2eb.png)

Các bước thực thi phương thức main như sau:

1. Sau khi biên dịch `App.java` thu được `App.class`, thực thi `App.class`, hệ thống sẽ khởi động một JVM process, tìm một file nhị phân tên là `App.class` từ đường dẫn classpath, load thông tin class của `App` vào Method Area trong Runtime Data Area, quá trình này gọi là Class Loading của `App`.
2. JVM tìm thấy điểm bắt đầu chương trình chính của `App`, thực thi phương thức `main`.
3. Câu lệnh đầu tiên trong `main` này là `Student student = new Student("tellUrDream")`, tức là bảo JVM tạo một đối tượng `Student`, nhưng lúc này trong Method Area chưa có thông tin của class `Student`, do đó JVM lập tức load class `Student`, đặt thông tin của class `Student` vào Method Area.
4. Sau khi load xong class `Student`, JVM cấp phát bộ nhớ cho một instance `Student` mới trong Heap, sau đó gọi constructor để khởi tạo instance `Student`, instance `Student` này nắm giữ reference **trỏ tới thông tin kiểu dữ liệu của class Student trong Method Area**.
5. Khi thực thi `student.sayName();`, JVM dựa vào reference của `student` tìm tới đối tượng `student`, sau đó dựa vào reference mà đối tượng `student` nắm giữ định vị tới bảng phương thức (method table) của thông tin kiểu class `Student` trong Method Area, lấy được địa chỉ bytecode của `sayName()`.
6. Thực thi `sayName()`.

Thực ra cũng không cần bận tâm quá nhiều, chỉ cần biết khi khởi tạo instance đối tượng sẽ tới Method Area tìm thông tin class, sau khi hoàn thành lại tới Stack để chạy phương thức. Tìm phương thức thì tìm trong bảng phương thức.

## II. Giới thiệu về ClassLoader

Trước đó cũng đã đề cập nó chịu trách nhiệm load file `.class`, chúng sẽ có định danh file đặc biệt ở đầu file, load nội dung bytecode của file class vào bộ nhớ, và chuyển đổi các nội dung này thành cấu trúc dữ liệu runtime trong Method Area, đồng thời ClassLoader chỉ chịu trách nhiệm load file class, còn việc có thể chạy hay không thì do Execution Engine quyết định.

### 2.1 Quy trình của ClassLoader

Từ khi Class được load vào bộ nhớ Virtual Machine đến khi giải phóng bộ nhớ có tổng cộng 7 bước: Load (Nạp), Verification (Xác minh), Preparation (Chuẩn bị), Resolution (Phân tích), Initialization (Khởi tạo), Using (Sử dụng), Unloading (Gỡ bỏ). Trong đó **Verification, Preparation, Resolution được gọi chung là Linking (Liên kết)**.

#### 2.1.1 Load (Nạp)

1. Nạp file class vào bộ nhớ
2. Chuyển đổi cấu trúc dữ liệu static thành cấu trúc dữ liệu runtime trong Method Area
3. Tạo một đối tượng `java.lang.Class` đại diện cho Class này trong Heap làm lối vào truy cập dữ liệu

#### 2.1.2 Linking (Liên kết)

1. Verification (Xác minh): Đảm bảo Class được load phù hợp với quy chuẩn JVM và an toàn, đảm bảo các phương thức của Class được kiểm tra khi chạy không gây hại cho Virtual Machine, thực chất là kiểm tra an toàn.
2. Preparation (Chuẩn bị): Cấp phát không gian lưu trữ cho các biến Class (field `static`) và thiết lập giá trị khởi tạo mặc định, ví dụ `static int a = 3` trong giai đoạn chuẩn bị thường nhận giá trị mặc định 0 trước, sang giai đoạn khởi tạo mới gán giá trị 3. Vị trí lưu trữ cụ thể thuộc về chi tiết triển khai Virtual Machine.
3. Resolution (Phân tích): Virtual Machine chuyển đổi các symbolic reference trong Constant Pool runtime thành direct reference. `import` trong source code chỉ là cơ chế parse tên ở thời điểm compile, không phải symbolic reference trong Constant Pool; biểu diễn cụ thể của direct reference cũng không nhất thiết là địa chỉ đối tượng.

#### 2.1.3 Initialization (Khởi tạo)

Khởi tạo thực chất chính là quá trình thực thi phương thức `<clinit>()` của class constructor, và phải đảm bảo phương thức `<clinit>()` của class cha đã thực thi xong trước đó. Phương thức này do compiler thu thập, thực thi tuần tự tất cả việc khởi tạo hiển thị của biến Class (biến thành viên được modifier bằng `static`) và các câu lệnh trong khối code static. Lúc này `static int a` ở giai đoạn chuẩn bị từ giá trị khởi tạo mặc định 0 đã trở thành giá trị khởi tạo hiển thị 3. Do thứ tự thực thi, nếu biến Class ở giai đoạn khởi tạo lại bị thay đổi trong khối code static, sẽ đè lên khởi tạo hiển thị của biến Class, giá trị cuối cùng sẽ là gán giá trị trong khối code static.

> Lưu ý: Phương thức khởi tạo trong file bytecode có 2 loại, `<init>` dùng cho khởi tạo tài nguyên phi static và `<clinit>` dùng cho khởi tạo tài nguyên static, phương thức class constructor `<clinit>()` khác với constructor của class, các phương thức này đều là phương thức đặc biệt trong file bytecode chỉ dành cho JVM nhận biết.

#### 2.1.4 Unloading (Gỡ bỏ)

Class unloading chỉ việc Virtual Machine thu hồi metadata của Class không còn tới được và các tài nguyên liên quan, thông thường yêu cầu ClassLoader định nghĩa Class đó cũng không còn tới được nữa. Nó khác với Garbage Collection của đối tượng thông thường, và việc có gỡ bỏ hay không, khi nào gỡ bỏ do triển khai Virtual Machine và Garbage Collector quyết định.

### 2.2 Thứ tự load của ClassLoader

Lấy HotSpot JDK 8 làm ví dụ, thứ tự phân cấp ClassLoader thường gặp như sau. Trong JDK 9 sau khi mô-đun hóa, Extension ClassLoader được thay thế bởi Platform ClassLoader, `rt.jar` cũng không còn tồn tại:

1. Bootstrap ClassLoader: `rt.jar`
2. Extension ClassLoader: Load các file jar mở rộng
3. App ClassLoader: Các file jar dưới classpath chỉ định
4. Custom ClassLoader: ClassLoader do người dùng tự định nghĩa

### 2.3 Cơ chế Parents Delegation (Ủy quyền cha)

Khi một Class nhận được yêu cầu load, nó sẽ không tự mình thử load trước, mà ủy quyền cho ClassLoader cha hoàn thành, ví dụ bây giờ tôi muốn `new` một `Person`, `Person` này là Class do chúng ta tự định nghĩa, nếu muốn load nó, trước tiên sẽ ủy quyền cho App ClassLoader, chỉ khi các ClassLoader cha đều phản hồi bản thân không thể hoàn thành yêu cầu này (tức các ClassLoader cha đều không tìm thấy Class cần load), ClassLoader con mới tự mình thử load.

Lợi ích của việc làm này là, khi load các Class nằm trong package `rt.jar` bất kể là ClassLoader nào load, cuối cùng đều ủy quyền cho Bootstrap ClassLoader tiến hành load, đảm bảo sử dụng các ClassLoader khác nhau đều thu được cùng một kết quả.

Thực ra đây cũng là một tác dụng cách ly, tránh việc code của chúng ta ảnh hưởng đến code của JDK, ví dụ bây giờ tự mình định nghĩa một `java.lang.String`:

```java
package java.lang;
public class String {
    public static void main(String[] args) {
        System.out.println();
    }
}
```

Khi thử chạy hàm `main` của Class hiện tại, code của chúng ta chắc chắn sẽ báo lỗi. Đó là vì khi load thực chất đã tìm thấy `java.lang.String` trong `rt.jar`, tuy nhiên phát hiện bên trong đó không có phương thức `main`.

## III. Runtime Data Area (Vùng dữ liệu thời gian chạy)

### 3.1 Native Method Stack và Program Counter

Ví dụ bây giờ chúng ta mở source code của class Thread, sẽ thấy phương thức `start0` của nó có keyword `native`, và không có thân phương thức Java. Loại phương thức này do code bản địa bên ngoài Java triển khai, ngôn ngữ triển khai phổ biến là C hoặc C++; Virtual Machine sử dụng Native Method Stack để hỗ trợ thực thi phương thức native.

Program Counter ghi lại địa chỉ lệnh JVM mà thread hiện tại đang thực thi. Nó cũng là Runtime Data Area duy nhất trong 《Quy chuẩn Java Virtual Machine》 không quy định bất kỳ trường hợp `OutOfMemoryError` nào. Bytecode Interpreter thông qua việc thay đổi giá trị Program Counter để chọn lệnh bytecode tiếp theo cần thực thi.

Nếu thực thi phương thức native, quy chuẩn không quy định giá trị của Program Counter.

### 3.2 Method Area

Tác dụng chủ yếu của Method Area là lưu trữ thông tin metadata của Class, hằng số và biến static··· v.v. Khi thông tin nó lưu trữ quá lớn, sẽ báo lỗi khi không thể đáp ứng cấp phát bộ nhớ.

### 3.3 Virtual Machine Stack và Virtual Machine Heap

Một câu tóm tắt là: Stack quản lý việc thực thi, Heap quản lý việc lưu trữ. Virtual Machine Stack chịu trách nhiệm chạy code, còn Virtual Machine Heap chịu trách nhiệm lưu trữ dữ liệu.

#### 3.3.1 Khái niệm Virtual Machine Stack

Nó là mô hình bộ nhớ thực thi phương thức Java. Mỗi lần gọi phương thức sẽ tạo một Stack Frame, Stack Frame chứa thông tin bảng biến cục bộ, Operand Stack, Dynamic Linking và phương thức trả về địa chỉ v.v., và do thread độc chiếm. Bảng biến cục bộ là một phần của Stack Frame.

```java
public class Person{
    int a = 1;

    public void doSomething(){
        int b = 2;
    }
}
```

#### 3.3.2 Exception tồn tại ở Virtual Machine Stack

Nếu độ sâu Stack mà thread yêu cầu lớn hơn độ sâu tối đa của Virtual Machine Stack, sẽ báo **StackOverflowError** (lỗi này thường xuất hiện trong đệ quy). Java Virtual Machine cũng có thể mở rộng động, nhưng cùng với việc mở rộng sẽ liên tục xin cấp phát bộ nhớ, khi không thể xin đủ bộ nhớ sẽ báo lỗi **OutOfMemoryError**.

#### 3.3.3 Vòng đời của Virtual Machine Stack

Đối với Stack, không tồn tại Garbage Collection. Chỉ cần chương trình chạy kết thúc, không gian của Stack tự nhiên sẽ được giải phóng. Vòng đời của Stack đồng nhất với thread chứa nó.

Ở đây bổ sung thêm một câu: Tham số phương thức và các biến kiểu nguyên thủy được định nghĩa trong phương thức, reference đối tượng thường được lưu trong bảng biến cục bộ của Stack Frame hiện tại; instance đối tượng thường được cấp phát trên Heap. Bytecode của phương thức thuộc về metadata của Class, không được cấp phát trong Stack như một "phương thức instance".

#### 3.3.4 Thực thi Virtual Machine Stack

Stack Frame bản thân không phải là phương thức, mà là cấu trúc dữ liệu runtime tương ứng với một lần gọi phương thức, dùng để lưu trữ các thông tin bảng biến cục bộ, Operand Stack, Dynamic Linking v.v. Bytecode và metadata của phương thức không lưu trữ trong Virtual Machine Stack.

Dữ liệu trong Stack đều tồn tại dưới dạng Stack Frame, nó là một tập hợp dữ liệu về phương thức và dữ liệu thời gian chạy. Ví dụ chúng ta thực thi một phương thức a, sẽ tương ứng tạo ra một Stack Frame A1, sau đó A1 sẽ được push vào Stack. Tương tự phương thức b có B1, phương thức c có C1, đợi đến khi thread này thực thi xong, Stack sẽ pop C1 trước, sau đó tới B1, A1. Nó tuân theo nguyên tắc Last In First Out (LIFO).

#### 3.3.5 Tái sử dụng biến cục bộ

Bảng biến cục bộ được dùng để lưu trữ tham số phương thức và biến cục bộ định nghĩa bên trong phương thức. Dung lượng của nó lấy Slot làm đơn vị nhỏ nhất, một Slot có thể lưu trữ dữ liệu kiểu 32-bit trở xuống.

Virtual Machine thông qua chỉ số để định vị Slot trong bảng biến cục bộ. Nếu dung lượng bảng biến cục bộ là n, phạm vi chỉ số có hiệu lực là `[0, n)`; `long` và `double` sẽ chiếm 2 Slot liên tiếp. Tham số phương thức được sắp xếp trong bảng biến cục bộ theo thứ tự quy định. Để tiết kiệm không gian Stack Frame, khi vị trí thực thi vượt quá phạm vi tác dụng của biến cục bộ nào đó, Slot của nó có thể được biến khác tái sử dụng; các reference đối tượng vẫn được coi là có hiệu lực trong bảng biến cục bộ sẽ tham gia quét GC Roots.

#### 3.3.6 Khái niệm Virtual Machine Heap

Lấy Garbage Collector phân thế của HotSpot làm ví dụ, Heap thông thường được chia thành **Young Generation** và **Old Generation**, Young Generation lại có thể chia thành **Eden** và 2 vùng **Survivor**. Trong một lần sao chép, một vùng Survivor làm `from`, vùng còn lại làm `to`. Tỷ lệ mặc định giữa Eden và một vùng Survivor đơn lẻ thông thường là **8:1:1**, nhưng kích thước thực tế có thể chịu ảnh hưởng bởi collector được sử dụng và chiến lược tự điều chỉnh. Permanent Generation là một triển khai Method Area của HotSpot trong JDK 7 trở về trước, không đồng nhất với toàn bộ "bộ nhớ phi Heap".

Trong bộ nhớ Heap chủ yếu lưu trữ đối tượng, Garbage Collector sẽ phán đoán và thu hồi các đối tượng không thể tới được trong đó. Bộ nhớ phi Heap mà HotSpot nhắc tới không chỉ bao gồm triển khai của Method Area, mà còn bao gồm các vùng như Code Cache. Sau khi JDK 8 loại bỏ Permanent Generation, metadata của Class chuyển sang do Metaspace của JVM quản lý; Metaspace sử dụng bộ nhớ bản địa chứ không phải Java Heap. Các tham số liên quan bao gồm:

```plain
MetaspaceSize: Ngưỡng high-watermark ban đầu kích hoạt GC metadata, sau đó JVM tự điều chỉnh động
MaxMetaspaceSize: Giới hạn trên kích thước Metaspace
```

Sau khi loại bỏ Permanent Generation, sẽ không còn xuất hiện `java.lang.OutOfMemoryError: PermGen space` do cạn kiệt Permanent Generation nữa. Nhưng Metaspace vẫn có thể ném ra `OutOfMemoryError: Metaspace` do metadata của Class quá nhiều hoặc đạt giới hạn `MaxMetaspaceSize`.

#### 3.3.7 Giới thiệu về Eden Young Generation

Khi chúng ta `new` một đối tượng, trước tiên sẽ đặt vào một vùng không gian bộ nhớ chia ra từ Eden, nhưng chúng ta biết bộ nhớ Heap là dùng chung giữa các thread, do đó có thể xuất hiện trường hợp hai đối tượng dùng chung một bộ nhớ. Ở đây cách xử lý của JVM là dự kiến xin trước cho mỗi thread một khối bộ nhớ liên tục và quy định vị trí đặt đối tượng, nếu không gian không đủ sẽ xin thêm nhiều khối bộ nhớ nữa. Thao tác này chúng ta gọi là TLAB (Thread-Local Allocation Buffer), nếu thích bạn có thể tìm hiểu thêm.

Khi không gian Eden không đủ để tiếp tục cấp phát đối tượng, thông thường sẽ kích hoạt Minor GC (tức GC xảy ra ở Young Generation), các đối tượng còn sống có thể được sao chép sang vùng Survivor hoặc trực tiếp thăng tiến lên Old Generation. Sau khi sao chép hoàn tất, hai vùng Survivor `from` và `to` sẽ trao đổi vai trò cho nhau. Sau khi tuổi đối tượng đạt ngưỡng thăng tiến sẽ vào Old Generation; trong các Garbage Collector phân thế HotSpot phổ biến, giá trị mặc định của `-XX:MaxTenuringThreshold` thông thường là 15, nhưng tuổi thăng tiến thực tế còn chịu ảnh hưởng bởi phán đoán tuổi động, dung lượng không gian Survivor và chiến lược của collector, không phải tất cả các đối tượng đều cố định trải qua 15 lần Minor GC.

> 🐛 Sửa lỗi: Khi không gian bộ nhớ vùng Eden đầy, mới kích hoạt Minor GC, còn Survivor0 đầy không kích hoạt Minor GC.
>
> **Vậy đối tượng ở vùng Survivor0 khi nào được Garbage Collection?**
>
> Giả sử vùng Survivor0 hiện tại đầy, lúc này lại kích hoạt Minor GC, phát hiện vùng Survivor0 vẫn đầy không chứa thêm được, lúc này sẽ cùng tiến hành phân tích tính khả đạt cho các đối tượng ở S0 và Eden, tìm ra các đối tượng còn sống, sao chép sang S1 và xóa rỗng các đối tượng ở S0 và Eden, như vậy những đối tượng không tới được sẽ bị dọn dẹp, và trao đổi S0 với S1.

Old Generation chủ yếu lưu trữ các đối tượng sống lâu hoặc thăng tiến trực tiếp. Không gian Old Generation không đủ có thể kích hoạt Old Generation GC hoặc Full GC, hành vi cụ thể phụ thuộc vào Garbage Collector; phạm vi tạm dừng của các collector khác nhau cũng không giống nhau, không thể khái quát chung là dừng tất cả các thread ứng dụng trong suốt quá trình.

Hơn nữa khi Old Generation thực thi Full GC mà vẫn không thể tiến hành lưu trữ đối tượng, sẽ tạo ra OOM, lúc này chính là bộ nhớ Heap trong Virtual Machine không đủ, nguyên nhân có thể do kích thước bộ nhớ Heap cấu hình quá nhỏ, điều này có thể điều chỉnh qua tham số `-Xms`, `-Xmx`. Cũng có thể do đối tượng tạo trong code vừa lớn vừa nhiều, và chúng liên tục được reference khiến Garbage Collection thời gian dài không thể thu gom chúng.

![](https://static001.geekbang.org/infoq/39/398255141fde8ba208f6c99f4edaa9fe.png)

Giải thích bổ sung: Về vấn đề tham số `-XX:TargetSurvivorRatio`. Thực ra cũng không nhất thiết phải thỏa mãn `-XX:MaxTenuringThreshold` mới chuyển sang Old Generation. Có thể lấy ví dụ: Như đối tượng tuổi 5 chiếm 30%, tuổi 6 chiếm 36%, tuổi 7 chiếm 34%, sau khi cộng thêm một độ tuổi nào đó (như tuổi 6 trong ví dụ), tổng dung lượng chiếm dụng vượt quá `không gian Survivor * TargetSurvivorRatio`, thì từ độ tuổi đó trở đi và các đối tượng có tuổi lớn hơn sẽ đi vào Old Generation (tức đối tượng tuổi 6 trong ví dụ, chính là tuổi 6 và tuổi 7 thăng tiến lên Old Generation), lúc này không cần chờ đến 15 theo yêu cầu trong `MaxTenuringThreshold`.

#### 3.3.8 Làm thế nào để phán đoán một đối tượng cần bị tiêu hủy

![](https://static001.geekbang.org/infoq/1b/1ba7f3cff6e07c6e9c6765cc4ef74997.png)

Trong hình, 3 vùng Program Counter, Virtual Machine Stack, Native Method Stack sinh ra và mất đi theo thread. Việc cấp phát và thu hồi bộ nhớ là xác định. Cùng với việc kết thúc thread, bộ nhớ tự nhiên được thu hồi, do đó không cần cân nhắc vấn đề Garbage Collection. Còn Java Heap và Method Area thì khác, dùng chung giữa các thread, việc cấp phát và thu hồi bộ nhớ đều mang tính động. Do đó Garbage Collector quan tâm đều là phần bộ nhớ Heap và Method Area.

Trước khi tiến hành thu hồi cần phán đoán những đối tượng nào còn sống, những đối tượng nào đã chết. Dưới đây giới thiệu 2 phương pháp tính toán cơ bản:

1. Đếm reference (Reference Counting): Thêm một bộ đếm reference vào đối tượng, mỗi lần reference đối tượng này bộ đếm tăng 1, reference hết hiệu lực giảm 1, bộ đếm bằng 0 thì không sử dụng lại nữa. Tuy nhiên phương pháp này có một trường hợp là khi xuất hiện reference vòng giữa các đối tượng thì GC không thu hồi được.

2. Phân tích tính khả đạt (Reachability Analysis): Từ một loạt GC Roots xuất phát, tìm kiếm theo quan hệ reference giữa các đối tượng, đường đi mà tìm kiếm đi qua gọi là chuỗi reference (Reference Chain). Quan hệ reference giữa các đối tượng cấu thành nên đồ thị (Graph), chứ không phải cây nhị phân (Binary Tree). Khi một đối tượng và GC Roots không tồn tại bất kỳ chuỗi reference nào, chứng tỏ đối tượng đó không thể tới được. Các ngôn ngữ lập trình thương mại chủ đạo như Java, C# v.v. đều sử dụng hướng tư duy này để phán đoán đối tượng còn sống hay không.

(Tìm hiểu qua là được) Trong ngôn ngữ Java các đối tượng có thể làm GC Roots chia thành các loại sau:

1. Đối tượng được reference trong Virtual Machine Stack (bảng phương thức bản địa trong Stack Frame) (biến cục bộ)
2. Đối tượng được reference bởi biến static trong Method Area (biến static)
3. Đối tượng được reference bởi hằng số trong Method Area
4. Đối tượng được reference bởi JNI trong Native Method Stack (tức phương thức modifier bằng native) (JNI là cách Java Virtual Machine gọi hàm C tương ứng, thông qua hàm JNI cũng có thể tạo Java Object mới. Và JNI đối với local reference hoặc global reference của đối tượng đều sẽ đánh dấu đối tượng chúng trỏ tới là không thể thu hồi)
5. Thread Java đã khởi động và chưa chấm dứt

Ưu điểm của phương pháp này là có thể giải quyết vấn đề reference vòng. Collector cần thu được quan hệ reference đối tượng nhất quán tại một số giai đoạn, thông thường tạo ra tạm dừng Stop-The-World; các collector đồng thời hiện đại có thể thực thi lượng lớn công việc đánh dấu đồng thời với thread ứng dụng, chứ không phải toàn bộ quá trình phân tích tính khả đạt đều phải "dừng tất cả các process".

#### 3.3.9 Làm thế nào để tuyên bố một đối tượng thực sự đã chết

Trước tiên bắt buộc phải nhắc tới một phương thức tên là **`finalize()`**

`finalize()` là một phương thức của lớp `Object`. Phương thức `finalize()` của một đối tượng tối đa chỉ được hệ thống tự động gọi một lần; nếu đối tượng thông qua phương thức này thiết lập lại quan hệ tới được, lần tiếp theo bị phán đoán không thể tới được sẽ không gọi lại nữa.

Bổ sung thêm một câu: Không khuyến khích gọi `finalize()` trong chương trình để tự cứu. Thời gian thực thi của nó không xác định, thậm chí không đảm bảo nhất định sẽ thực thi, và chi phí chạy rất cao, không thể đảm bảo thứ tự gọi của từng đối tượng. `finalize()` bị đánh dấu deprecated trong Java 9, và bị đánh dấu chờ loại bỏ trong Java 18. Khi cần quản lý tài nguyên ngoài Heap, có thể dựa vào kịch bản sử dụng `try-with-resources` hoặc `java.lang.ref.Cleaner`; bản thân `Cleaner` không phải tên gọi chung cho Strong, Soft, Weak, Phantom Reference.

![](https://static001.geekbang.org/infoq/8d/8d7f0381c7d857c7ceb8ae5a5fef0f4a.png)

Đối với đối tượng override phương thức `finalize()` và phương thức đó chưa từng thực thi, quá trình xử lý của nó thông thường được khái quát là hai lần đánh dấu:

1. Nếu đối tượng sau khi phân tích tính khả đạt không phát hiện chuỗi reference kết nối với GC Roots, nó sẽ bị đánh dấu lần đầu và nhận lọc lựa. Đối với đối tượng có cần thiết thực thi phương thức `finalize()`, HotSpot có thể sẽ thêm final reference tương ứng vào hàng chờ xử lý, do thread Finalizer xử lý bất đồng bộ.
2. Nếu đối tượng trong phương thức `finalize()` thiết lập lại liên kết với đối tượng trên chuỗi reference, Garbage Collection sau đó sẽ chuyển nó ra khỏi tập hợp "sắp thu hồi"; nếu không, đối tượng vẫn có thể bị thu hồi. Quá trình này không đại diện cho việc Garbage Collector sẽ đồng bộ chờ phương thức `finalize()` thực thi, cũng không đảm bảo phương thức đó nhất định sẽ được gọi.

Nếu xác định đối tượng đã chết, chúng ta lại thu hồi đống rác này như thế nào?

### 3.4 Thuật toán Garbage Collection

Về phần giới thiệu chi tiết các thuật toán Garbage Collection thường gặp, khuyến nghị đọc bài này: [Giải thích chi tiết JVM Garbage Collection (Trọng tâm)](https://javaguide.cn/java/jvm/jvm-garbage-collection.html).

### 3.5 (Tìm hiểu) Các loại Garbage Collector đa dạng

Garbage Collector trong HotSpot VM và các kịch bản áp dụng:

![](https://static001.geekbang.org/infoq/9f/9ff72176ab0bf58bc43e142f69427379.png)

Trong cấu hình JDK 8 Server HotSpot phổ biến, cặp kết hợp mặc định thông thường là Parallel Scavenge và Parallel Old; giá trị mặc định thực tế vẫn có thể chịu ảnh hưởng bởi triển khai Virtual Machine, chế độ chạy và nền tảng.

JDK 9 đặt G1 làm Garbage Collector mặc định của Server HotSpot. Các collector khác nhau có sự đánh đổi giữa Throughput, độ trễ, dung lượng chiếm dụng bộ nhớ và chi phí CPU, không thể khẳng định thời gian tạm dừng của một collector nào đó nhất định ngắn nhất nếu thoát khỏi tải ứng dụng và cấu hình JVM.

### 3.6 (Tìm hiểu) Các tham số thường dùng của JVM

Tham số của JVM rất nhiều, ở đây chỉ liệt kê một vài tham số tương đối quan trọng, thông qua các công cụ tìm kiếm cũng có thể biết được những thông tin này.

| Tên tham số                 | Ý nghĩa                                    | Giải thích                                                                                                                  |
| --------------------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `-Xms`                      | Kích thước Heap ban đầu                    | Giá trị mặc định do chiến lược tự điều chỉnh của HotSpot tính toán theo môi trường chạy, không nên viết thành tỷ lệ cố định |
| `-Xmx`                      | Kích thước Heap tối đa                     | Giá trị mặc định do chiến lược tự điều chỉnh của HotSpot tính toán theo môi trường chạy                                     |
| `-Xmn`                      | Kích thước Young Generation                | Chỉ áp dụng cho các collector có khái niệm Young Generation cố định; thiết lập hiển thị sẽ hạn chế tự điều chỉnh            |
| `-XX:NewSize`               | Kích thước ban đầu Young Generation        | Chỉ có hiệu lực với collector phân thế hỗ trợ tham số này                                                                   |
| `-XX:MaxNewSize`            | Kích thước tối đa Young Generation         | Chỉ có hiệu lực với collector phân thế hỗ trợ tham số này                                                                   |
| `-XX:PermSize`              | Kích thước ban đầu Permanent Generation    | Chỉ áp dụng cho HotSpot JDK 7 trở về trước, JDK 8 đã loại bỏ Permanent Generation                                           |
| `-XX:MaxPermSize`           | Kích thước tối đa Permanent Generation     | Chỉ áp dụng cho HotSpot JDK 7 trở về trước, JDK 8 đã loại bỏ Permanent Generation                                           |
| `-Xss`                      | Kích thước Stack từng Thread               | Giá trị mặc định phụ thuộc nền tảng và phiên bản JVM, nên kết hợp số lượng thread, độ sâu lời gọi và kết quả test          |
| `-XX:NewRatio`              | Tỷ lệ dung lượng Old Gen / Young Gen       | Ví dụ giá trị là 4, biểu thị Old Gen:Young Gen = 4:1; có hiệu lực hay không phụ thuộc collector                             |
| `-XX:SurvivorRatio`         | Tỷ lệ dung lượng Eden / Survivor đơn lẻ    | Ví dụ giá trị 8, Eden:From:To thường là 8:1:1; chiến lược tự điều chỉnh có thể điều chỉnh kích thước thực tế                |
| `-XX:+DisableExplicitGC`    | Bỏ qua yêu cầu GC phát ra từ `System.gc()` | Không đồng nghĩa đóng GC tự động của JVM, trước khi dùng cần đánh giá các kịch bản như Direct Memory                        |
| `-XX:PretenureSizeThreshold`| Ngưỡng đối tượng lớn đi thẳng vào Old Gen  | Hỗ trợ hay không và hiệu lực như thế nào phụ thuộc Garbage Collector                                                        |
| `-XX:ParallelGCThreads`     | Số GC thread trong giai đoạn STW song song | Giá trị mặc định do JVM tính toán dựa trên số lượng bộ xử lý khả dụng                                                       |
| `-XX:MaxGCPauseMillis`      | Mục tiêu thời gian tạm dừng GC tối đa      | Đây là mục tiêu mềm chứ không phải đảm bảo cứng, hiệu quả cụ thể phụ thuộc Garbage Collector được sử dụng                   |

Thực ra còn một số tham số về in ấn và CMS, ở đây không liệt kê từng cái một nữa.

## IV. Về các khía cạnh tối ưu hóa JVM

Dựa trên các điểm kiến thức JVM vừa đề cập, chúng ta có thể thử tối ưu hóa JVM, chủ yếu là mảng bộ nhớ Heap.

Đối với collector áp dụng bố cục phân thế cố định, Java Heap có thể xem gần đúng là tổng của Young Generation và Old Generation; Permanent Generation hoặc Metaspace không thuộc về Java Heap. Khi tổng kích thước Heap cố định, tăng Young Generation sẽ nén không gian Old Generation, nhưng tỷ lệ cụ thể không có "giá trị tối ưu chính thức" áp dụng cho mọi ứng dụng, cần test dựa trên việc cấp phát, sống sót của đối tượng và collector được sử dụng.

### 4.1 Điều chỉnh bộ nhớ Heap tối đa và tối thiểu

`-Xmx` và `-Xms` lần lượt chỉ định giá trị tối đa và giá trị ban đầu của Java Heap. Khi không thiết lập hiển thị, giá trị mặc định do HotSpot tự điều chỉnh tính toán dựa trên phiên bản JVM, bộ nhớ khả dụng, giới hạn container và môi trường chạy, không nên ước tính theo tỷ lệ cố định bộ nhớ vật lý.

HotSpot có thể điều chỉnh không gian Heap đã committed giữa `-Xms` và `-Xmx`, `MinHeapFreeRatio` và `MaxHeapFreeRatio` là các tham số một số collector dùng để kiểm soát tỷ lệ trống sau GC. Hành vi mở rộng / thu nhỏ thực tế và tỷ lệ mặc định phụ thuộc collector được sử dụng, phiên bản JDK và chiến lược tự điều chỉnh, không thể khái quát chung thành 40% và 70% cố định.

Cấu hình `-Xms` và `-Xmx` cùng một giá trị có thể tránh việc điều chỉnh kích thước Heap committed trong quá trình chạy, nhưng sẽ giữ dung lượng Heap tương đối lớn ngay từ khi khởi động, việc có áp dụng hay không vẫn nên đánh giá kết hợp môi trường triển khai và tải.

Chúng ta thực thi code dưới đây:

```java
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");    //Dung lượng tối đa của hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //Dung lượng rảnh của hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //Tổng dung lượng khả dụng hiện tại
```

Lưu ý: Ở đây thiết lập là kích thước Java Heap, tức là kích thước Young Generation + kích thước Old Generation.

![](https://static001.geekbang.org/infoq/11/114f32ddd295b2e30444f42f6180538c.png)

Thiết lập tham số VM options:

```plain
-Xmx20m -Xms5m -XX:+PrintGCDetails
```

![](https://static001.geekbang.org/infoq/7e/7ea0bf0dec20e44bf95128c571d6ef0e.png)

Chạy lại phương thức main:

![](https://static001.geekbang.org/infoq/c8/c89edbd0a147a791cfabdc37923c6836.png)

Ở đây GC bật ra một Allocation Failure thất bại cấp phát, việc này xảy ra ở PSYoungGen, tức là trong Young Generation.

Lúc này bộ nhớ xin được là 18M, bộ nhớ rảnh là 4.214195251464844M.

Lúc này chúng ta tạo một mảng byte xem thử, thực thi code dưới đây:

```java
byte[] b = new byte[1 * 1024 * 1024];
System.out.println("Cấp phát 1M không gian cho mảng");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  //Dung lượng tối đa hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //Dung lượng rảnh hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");
```

![](https://static001.geekbang.org/infoq/db/dbeb6aea0a90949f7d7fe4746ddb11a3.png)

Lúc này free memory lại giảm xuống, tuy nhiên total memory không thay đổi. Java sẽ cố gắng duy trì giá trị total mem ở kích thước Heap tối thiểu.

```java
byte[] b = new byte[10 * 1024 * 1024];
System.out.println("Cấp phát 10M không gian cho mảng");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  //Dung lượng tối đa hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //Dung lượng rảnh hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //Tổng dung lượng khả dụng hiện tại
```

![](https://static001.geekbang.org/infoq/b6/b6a7c522166dbd425dbb06eb56c9b071.png)

Lúc này chúng ta tạo một dữ liệu byte 10M, lúc này bộ nhớ Heap tối thiểu không chịu nổi. Chúng ta sẽ thấy total memory hiện tại đã trở thành 15M, đây là kết quả của việc đã xin thêm bộ nhớ một lần.

Lúc này chúng ta lại chạy lại code này:

```java
System.gc();
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");    //Dung lượng tối đa hệ thống
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //Dung lượng rảnh hệ thống
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //Tổng dung lượng khả dụng hiện tại
```

![](https://static001.geekbang.org/infoq/8d/8dd6e8fccfd1394b83251c136ee44ceb.png)

Ở đây gọi `System.gc()` chỉ là đưa ra đề xuất Garbage Collection tới JVM, không đảm bảo nhất định thực thi Full GC; lần chạy trong ảnh chụp màn hình đúng là đã kích hoạt thu gom tương ứng và thu nhỏ không gian Heap committed, nhưng dưới các tham số JVM và collector khác nhau kết quả có thể khác nhau.

### 4.2 Điều chỉnh tỷ lệ Young Generation và Old Generation

```plain
-XX:NewRatio --- Tỷ lệ giữa Young Generation (eden + 2 * Survivor) và Old Generation (không bao gồm Permanent Region)

Ví dụ: -XX:NewRatio=4, biểu thị Young Generation:Old Generation = 1:4, tức Young Generation chiếm 1/5 toàn bộ Heap. Trong trường hợp Xms=Xmx và đã thiết lập Xmn, tham số này không cần thiết lập.
```

### 4.3 Điều chỉnh tỷ lệ vùng Survivor và vùng Eden

```plain
-XX:SurvivorRatio --- Thiết lập tỷ lệ giữa 2 vùng Survivor và eden

Ví dụ: 8, biểu thị 2 Survivor:eden = 2:8, tức một Survivor chiếm 1/10 Young Generation
```

### 4.4 Thiết lập kích thước Young Generation và Old Generation

```plain
-XX:NewSize --- Thiết lập kích thước Young Generation
-XX:MaxNewSize --- Thiết lập giá trị tối đa Young Generation
```

Có thể thông qua thiết lập các tham số khác nhau để test các trường hợp khác nhau. Tỷ lệ khởi tạo thường gặp giữa Eden và 2 vùng Survivor là 8:1:1, nhưng đây không phải đáp án tối ưu áp dụng cho mọi ứng dụng và collector; `-Xms` khác `-Xmx` cũng không đồng nghĩa chắc chắn dẫn đến GC nhiều lần.

### 4.5 Tóm tắt ngắn gọn

Nên dựa vào tải thực tế, Garbage Collector được dùng và GC log để điều chỉnh kích thước Young Generation và vùng Survivor, không tồn tại tỷ lệ khuyến nghị cố định thành lập cho mọi ứng dụng.

Khi bị OOM, hãy nhớ Dump Heap ra, đảm bảo có thể định vị rà soát vấn đề hiện trường. Thông qua lệnh dưới đây có thể xuất ra một file `.dump`, file này có thể sử dụng các công cụ như VisualVM để phân tích. VisualVM từ JDK 9 trở đi không còn đi kèm theo Oracle JDK nữa, cần phải cài đặt riêng.

```plain
-Xmx20m -Xms5m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=Đường_dẫn_log_bạn_muốn_xuất_ra
```

Thông thường chúng ta cũng có thể thông qua cách viết script để khi OOM xuất hiện sẽ báo tin cho chúng ta, có thể giải quyết bằng cách gửi email hoặc restart chương trình v.v.

### 4.6 Thiết lập Permanent Region (Chỉ áp dụng cho HotSpot JDK 7 trở về trước)

```plain
-XX:PermSize -XX:MaxPermSize
```

`PermSize` thiết lập kích thước ban đầu của Permanent Generation, `MaxPermSize` thiết lập giới hạn trên của nó; giá trị mặc định cụ thể liên quan đến phiên bản JVM và nền tảng, không thể viết chung thành tỷ lệ cố định bộ nhớ vật lý. JDK 8 trở đi nên chú ý đến các tham số liên quan Metaspace, chứ không phải 2 tham số Permanent Generation này.

Tips: Nếu không gian Heap chưa dùng hết mà cũng ném ra OOM, có khả năng do Permanent Region gây ra. Bộ nhớ Heap chiếm dụng thực tế rất ít, nhưng Permanent Region tràn ra thì vẫn ném ra OOM như thường.

### 4.7 Tối ưu tham số Stack của JVM

#### 4.7.1 Điều chỉnh kích thước không gian Stack từng Thread

Có thể thông qua `-Xss`: Điều chỉnh kích thước không gian Stack của từng thread.

Kích thước mặc định của Thread Stack phụ thuộc phiên bản JVM, nền tảng và chế độ chạy, không phải từ JDK 5 trở đi thống nhất là 1 MB. Trong cùng các điều kiện khác, giảm Thread Stack có thể chừa ra không gian địa chỉ cho nhiều platform thread hơn, nhưng số lượng thread còn chịu sự giới hạn của hệ điều hành, và Stack quá nhỏ sẽ làm tăng rủi ro `StackOverflowError`.

#### 4.7.2 Thiết lập kích thước Thread Stack

```plain
-XX:ThreadStackSize=<size>:
Thiết lập kích thước Thread Stack (0 nghĩa là dùng kích thước mặc định)
```

Các tham số này đều có thể thông qua tự viết chương trình để test đơn giản, ở đây do giới hạn độ dài bài viết nên không cung cấp demo nữa.

### 4.8 (Có thể bỏ qua trực tiếp) Giới thiệu các tham số khác của JVM

Các tham số muôn hình muôn vẻ rất nhiều, sẽ không nhắc tới tất cả từng cái một.

#### 4.8.1 Thiết lập kích thước trang bộ nhớ lớn (Large Page)

```plain
-XX:LargePageSizeInBytes=<size>
```

Tham số này dùng để thiết lập kích thước trang bộ nhớ lớn, có hiệu lực hay không phụ thuộc hệ điều hành, JVM build và cấu hình trang lớn.

#### 4.8.2 Tham số lịch sử `UseFastAccessorMethods`

```plain
-XX:+UseFastAccessorMethods
```

Tham số HotSpot bản cũ này nhắm tới tối ưu hóa reflection accessor, chứ không phải "tối ưu hóa nhanh kiểu nguyên thủy"; JDK hiện đại không còn cung cấp tham số này nữa.

#### 4.8.3 Thiết lập tắt GC thủ công

```plain
-XX:+DisableExplicitGC:
Thiết lập tắt System.gc() (tham số này cần test nghiêm ngặt)
```

#### 4.8.4 Thiết lập tuổi rác tối đa

```plain
-XX:MaxTenuringThreshold
Thiết lập giới hạn trên tuổi thăng tiến của đối tượng. Khi thiết lập thành 0, collector phân thế hỗ trợ tham số này sẽ làm đối tượng sống ở Young Gen không trải qua vùng Survivor mà thăng tiến trực tiếp. Tăng giá trị này có thể làm đối tượng trải qua nhiều lần sao chép hơn ở vùng Survivor, nhưng thăng tiến thực tế còn chịu ảnh hưởng bởi phán đoán tuổi động, dung lượng Survivor và chiến lược của collector.
```

Không phải tất cả Garbage Collector đều sử dụng cơ chế phân thế và thăng tiến tuổi giống nhau, việc có hiệu lực hay không nên lấy tài liệu của collector hiện tại làm chuẩn.

#### 4.8.5 Tham số lịch sử `AggressiveOpts`

```plain
-XX:+AggressiveOpts
```

Đây là tham số trong HotSpot bản cũ dùng để bật tối ưu hóa hiệu năng thử nghiệm, không thể hiểu đơn giản là "đẩy nhanh tốc độ biên dịch", và đã bị loại bỏ trong JDK 12.

#### 4.8.6 Tham số lịch sử `UseBiasedLocking`

```plain
-XX:+UseBiasedLocking
```

Biased Lock trong JDK 15 mặc định tắt và bị đánh dấu deprecated, triển khai liên quan sau đó đã bị gỡ khỏi HotSpot, không nên dùng làm tham số tối ưu hóa chung cho JDK hiện đại.

#### 4.8.7 Vô hiệu hóa Class Unloading

```plain
-Xnoclassgc
```

Tham số này vô hiệu hóa Garbage Collection của Class, chứ không phải Garbage Collection của đối tượng.

#### 4.8.8 Thiết lập thời gian sống của không gian Heap

```plain
-XX:SoftRefLRUPolicyMSPerMB
Thiết lập thời gian sống của SoftReference trong mỗi MB không gian rảnh của Heap, giá trị mặc định là 1s.
```

#### 4.8.9 Thiết lập đối tượng cấp phát trực tiếp ở Old Generation

```plain
-XX:PretenureSizeThreshold
Thiết lập đối tượng vượt quá độ lớn bao nhiêu thì cấp phát trực tiếp ở Old Generation, giá trị mặc định là 0.
```

#### 4.8.10 Thiết lập tỷ lệ TLAB chiếm vùng Eden

```plain
-XX:TLABWasteTargetPercent
Thiết lập phần trăm TLAB chiếm trong vùng Eden, giá trị mặc định là 1%.
```

## Finally

Bài viết thực sự đã trình bày rất dài về chủ đề này, tham khảo tài liệu từ nhiều nguồn, hy vọng có thể giúp ích cho bạn, xin cảm ơn.

<!-- @include: @article-footer.snippet.md -->
