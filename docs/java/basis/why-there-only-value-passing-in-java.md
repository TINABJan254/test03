---
title: Giải thích chi tiết Truyền tham trị (Pass by Value) trong Java
description: Giải thích chi tiết tại sao Java chỉ có truyền tham trị (Pass by Value): phân tích chuyên sâu cơ chế truyền tham số trong Java qua các ví dụ, làm rõ những hiểu lầm phổ biến giữa truyền tham trị và truyền tham chiếu, hiểu bản chất sự khác biệt giữa tham số hình thức và tham số thực tế.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Java truyền tham trị,truyền tham chiếu,truyền tham số,tham số hình thức tham số thực tế,tham chiếu đối tượng,gọi phương thức,cơ chế truyền tham số Java
---

Trước khi bắt đầu, chúng ta cùng hiểu rõ hai khái niệm dưới đây:

- Tham số hình thức (Parameters) & Tham số thực tế (Arguments)
- Truyền tham trị (Pass by Value) & Truyền tham chiếu (Pass by Reference)

## Tham số hình thức & Tham số thực tế

Định nghĩa của phương thức có thể sử dụng **tham số** (phương thức có tham số), tham số trong ngôn ngữ lập trình được chia thành:

- **Tham số thực tế (Arguments / Thực tham)**: Tham số được dùng để truyền cho hàm/phương thức, bắt buộc phải có giá trị xác định.
- **Tham số hình thức (Parameters / Hình tham)**: Dùng để định nghĩa hàm/phương thức, nhận tham số thực tế, không cần có giá trị xác định trước.

```java
String hello = "Hello!";
// hello là tham số thực tế
sayHello(hello);
// str là tham số hình thức
void sayHello(String str) {
    System.out.println(str);
}
```

## Truyền tham trị & Truyền tham chiếu

Cách thức ngôn ngữ lập trình truyền tham số thực tế cho phương thức (hoặc hàm) được chia thành 2 loại:

- **Truyền tham trị (Pass by Value)**: Phương thức nhận bản sao (copy) của giá trị tham số thực tế, sẽ tạo ra một bản sao.
- **Truyền tham chiếu (Pass by Reference)**: Phương thức nhận trực tiếp địa chỉ của tham số thực tế chứ không phải giá trị bên trong tham số thực tế, đây chính là con trỏ. Lúc này tham số hình thức chính là tham số thực tế, bất kỳ sửa đổi nào trên tham số hình thức đều phản ánh vào tham số thực tế, bao gồm cả việc gán lại giá trị.

Nhiều ngôn ngữ lập trình (như C++, Pascal) cung cấp cả hai phương thức truyền tham số, tuy nhiên trong Java chỉ có truyền tham trị.

## Tại sao Java chỉ có Truyền tham trị?

**Tại sao nói Java chỉ có truyền tham trị?** Không cần dài dòng, tôi sẽ chứng minh cho mọi người thông qua 3 ví dụ.

### Case 1: Truyền tham số kiểu nguyên thủy (Primitive Type)

Code:

```java
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;
    swap(num1, num2);
    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    System.out.println("a = " + a);
    System.out.println("b = " + b);
}
```

Kết quả:

```plain
a = 20
b = 10
num1 = 10
num2 = 20
```

Phân tích:

Trong phương thức `swap()`, giá trị của `a` và `b` tráo đổi cho nhau nhưng không hề ảnh hưởng đến `num1` và `num2`. Bởi vì giá trị của `a` và `b` chỉ được sao chép từ `num1` và `num2`. Nghĩa là `a` và `b` tương đương với bản sao của `num1` và `num2`, nội dung bản sao dù sửa đổi thế nào cũng không ảnh hưởng đến bản gốc.

![](https://oss.javaguide.cn/github/javaguide/java/basis/java-value-passing-01.png)

Thông qua ví dụ trên, chúng ta đã biết một phương thức không thể sửa đổi một tham số kiểu dữ liệu nguyên thủy, còn tham chiếu đối tượng khi làm tham số thì lại khác, mời xem Case 2.

### Case 2: Truyền tham số kiểu tham chiếu 1

Code:

```java
  public static void main(String[] args) {
      int[] arr = { 1, 2, 3, 4, 5 };
      System.out.println(arr[0]);
      change(arr);
      System.out.println(arr[0]);
  }

  public static void change(int[] array) {
      // Đổi phần tử đầu tiên của mảng thành 0
      array[0] = 0;
  }
```

Kết quả:

```plain
1
0
```

Phân tích:

![](https://oss.javaguide.cn/github/javaguide/java/basis/java-value-passing-02.png)

Xem xong ví dụ này chắc hẳn nhiều người nghĩ Java áp dụng truyền tham chiếu cho tham số kiểu tham chiếu.

Thực tế không phải vậy, ở đây thứ được truyền đi vẫn là giá trị, nhưng giá trị này là bản sao của tham chiếu đối tượng.

Nghĩa là tham số của phương thức `change` sao chép giá trị tham chiếu được lưu trong `arr`, do đó tham số hình thức và `arr` cùng trỏ đến một đối tượng mảng. Điều này giải thích tại sao việc sửa đổi nội dung mảng thông qua tham số hình thức lại được bên gọi quan sát thấy.

Để phản bác mạnh mẽ hơn việc Java không dùng truyền tham chiếu cho tham số kiểu tham chiếu, chúng ta hãy xem tiếp Case 3 bên dưới!

### Case 3: Truyền tham số kiểu tham chiếu 2

```java
public class Person {
    private String name;
   // Bỏ qua constructor, Getter & Setter
}

public static void main(String[] args) {
    Person xiaoZhang = new Person("Tiểu Trương");
    Person xiaoLi = new Person("Tiểu Lý");
    swap(xiaoZhang, xiaoLi);
    System.out.println("xiaoZhang:" + xiaoZhang.getName());
    System.out.println("xiaoLi:" + xiaoLi.getName());
}

public static void swap(Person person1, Person person2) {
    Person temp = person1;
    person1 = person2;
    person2 = temp;
    System.out.println("person1:" + person1.getName());
    System.out.println("person2:" + person2.getName());
}
```

Kết quả:

```plain
person1:Tiểu Lý
person2:Tiểu Trương
xiaoZhang:Tiểu Trương
xiaoLi:Tiểu Lý
```

Phân tích:

Chuyện gì xảy ra vậy??? Hai tham số hình thức kiểu tham chiếu tráo đổi cho nhau lại không hề ảnh hưởng đến tham số thực tế!

Tham số `person1` và `person2` của phương thức `swap` chỉ sao chép giá trị tham chiếu lưu trong tham số thực tế `xiaoZhang` và `xiaoLi`. Do đó việc hoán đổi `person1` và `person2` chỉ là hoán đổi bản sao tham chiếu mà mỗi bên tự lưu giữ, hoàn toàn không làm thay đổi giá trị của hai biến `xiaoZhang` và `xiaoLi` ở phía bên gọi.

![](https://oss.javaguide.cn/github/javaguide/java/basis/java-value-passing-03.png)

## Truyền tham chiếu thực sự trông như thế nào?

Xem tới đây, tin rằng bạn đã biết trong Java chỉ có truyền tham trị, không có truyền tham chiếu.
Tuy nhiên, truyền tham chiếu rốt cuộc trông như thế nào? Dưới đây lấy ví dụ code `C++` để bạn thấy bộ mặt thực sự của truyền tham chiếu.

```C++
#include <iostream>

void incr(int& num)
{
    std::cout << "incr before: " << num << "\n";
    num++;
    std::cout << "incr after: " << num << "\n";
}

int main()
{
    int age = 10;
    std::cout << "invoke before: " << age << "\n";
    incr(age);
    std::cout << "invoke after: " << age << "\n";
}
```

Kết quả:

```plain
invoke before: 10
incr before: 10
incr after: 11
invoke after: 11
```

Phân tích: Có thể thấy việc sửa đổi tham số hình thức trong hàm `incr` có thể ảnh hưởng trực tiếp đến giá trị của tham số thực tế. Lưu ý: Kiểu dữ liệu của tham số hình thức `incr` ở đây dùng `int&` mới là truyền tham chiếu, nếu dùng `int` thì vẫn là truyền tham trị nhé!

## Tại sao Java không đưa vào truyền tham chiếu?

Truyền tham chiếu trông có vẻ rất hay, có thể sửa đổi trực tiếp giá trị của tham số thực tế ngay trong phương thức, nhưng tại sao Java lại không đưa truyền tham chiếu vào?

**Lưu ý: Dưới đây là quan điểm cá nhân, không phải phát biểu chính thức từ Java:**

1. Xét về mặt an toàn, các thao tác trên giá trị bên trong phương thức đối với người gọi đều là chưa biết (định nghĩa phương thức thành interface, bên gọi không quan tâm bản thực thi cụ thể). Bạn thử tưởng tượng nếu cầm thẻ ngân hàng đi rút tiền, rút 100 mà bị trừ 200 thì có đáng sợ không.
2. Cha đẻ của Java - James Gosling ngay từ đầu thiết kế đã nhìn thấy nhiều nhược điểm của C, C++ nên mới muốn thiết kế một ngôn ngữ mới là Java. Khi ông thiết kế Java đã tuân thủ nguyên tắc đơn giản dễ dùng, loại bỏ nhiều "tính năng" mà lập trình viên chỉ cần bất cẩn một chút là gây ra sự cố, bản thân ngôn ngữ ít đi những thứ phức tạp thì thứ lập trình viên phải học cũng ít đi.

## Tóm tắt

Cách thức Java truyền tham số thực tế cho phương thức (hoặc hàm) là **Truyền tham trị (Pass by Value)**:

- Nếu tham số là kiểu nguyên thủy: Rất đơn giản, truyền đi là bản sao giá trị literal của kiểu nguyên thủy, sẽ tạo ra bản sao.
- Nếu tham số là kiểu tham chiếu: Truyền đi là bản sao của giá trị tham chiếu. Ban đầu tham số hình thức và tham số thực tế cùng trỏ đến một đối tượng, nhưng việc gán lại giá trị cho tham số hình thức sẽ không làm thay đổi biến tham số thực tế.

## Tài liệu tham khảo

- 《Java Core Technology Volume 1》 Kiến thức cơ bản bản 10 chương 4 mục 4.5
- [Java rốt cuộc là truyền tham trị hay truyền tham chiếu? - Trả lời của Hollis trên Zhihu](https://www.zhihu.com/question/31203609/answer/576030121)
- [Oracle Java Tutorials - Passing Information to a Method or a Constructor](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html)
- [Interview with James Gosling, Father of Java](https://mappingthejourney.com/single-post/2017/06/29/episode-3-interview-with-james-gosling-father-of-java/)

<!-- @include: @article-footer.snippet.md -->
