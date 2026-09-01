---
title: Giải thích chi tiết Generics & Wildcards trong Java
description: Phân tích toàn diện Generics và Wildcards trong Java: thấu hiểu cơ chế Xóa kiểu (Type Erasure), cách dùng Upper Bounded / Lower Bounded Wildcards, ứng dụng nguyên tắc PECS, làm chủ các kỹ thuật cốt lõi trong lập trình Generics.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Java Generics,Wildcard,Type Erasure,Ranh giới Generics,nguyên tắc PECS,Generic Method,Upper Lower Bounded Wildcards,Generic Interface
---

## Generics (Kiểu chung)

### Generics là gì? Có tác dụng gì?

**Java Generics (Kiểu chung)** là một tính năng mới được đưa vào từ JDK 5. Sử dụng tham số Generics giúp tăng tính đọc hiểu và tính ổn định của code. **Nếu không có giải thích đặc biệt, các hành vi dưới đây lấy Java 8 làm chuẩn.**

Trình biên dịch có thể kiểm tra tham số Generics, và thông qua tham số Generics có thể chỉ định kiểu đối tượng truyền vào. Ví dụ dòng code `ArrayList<Person> persons = new ArrayList<Person>()` chỉ rõ `ArrayList` này chỉ có thể nhận đối tượng thuộc kiểu `Person`, nếu truyền vào kiểu khác sẽ báo lỗi (từ JDK 7 trở đi có thể viết `new ArrayList<>()`, do trình biên dịch tự suy luận tham số kiểu).

```java
ArrayList<E> extends AbstractList<E>
```

Đồng thời, `List` nguyên thủy trả về kiểu `Object`, cần ép kiểu thủ công mới dùng được, sau khi dùng Generics thì trình biên dịch tự động chuyển đổi.

### Generics có những cách sử dụng nào?

Generics thông thường có 3 cách sử dụng: **Generic Class**, **Generic Interface**, **Generic Method**.

**1. Generic Class**:

```java
// T ở đây có thể viết tùy ý thành ký hiệu bất kỳ, phổ biến như T, E, K, V,...
// Khi khởi tạo Generic Class bắt buộc phải chỉ định kiểu cụ thể cho T
public class Generic<T> {

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey() {
        return key;
    }
}
```

Cách khởi tạo Generic Class:

```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
// Từ JDK 7 trở đi có thể viết: new Generic<>(123456)
```

**2. Generic Interface**:

```java
public interface Generator<T> {
    public T method();
}
```

Implement Generic Interface nhưng không chỉ định kiểu:

```java
class GeneratorImpl<T> implements Generator<T> {
    @Override
    public T method() {
        return null;
    }
}
```

Implement Generic Interface và chỉ định kiểu cụ thể:

```java
class GeneratorImpl implements Generator<String> {
    @Override
    public String method() {
        return "hello";
    }
}
```

**3. Generic Method**:

```java
   public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```

Cách dùng:

```java
// Tạo các mảng thuộc kiểu khác nhau: Integer, Double và Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray );
printArray( stringArray );
```

### Trong dự án hay dùng Generics ở đâu?

- Kết quả trả về chung tùy chỉnh `CommonResult<T>` thông qua tham số `T` có thể chỉ định động kiểu dữ liệu của kết quả theo kiểu trả về cụ thể
- Định nghĩa lớp xử lý `ExcelUtil<T>` dùng để chỉ định động kiểu dữ liệu xuất ra `Excel`
- Xây dựng các lớp utility tập hợp (tham khảo các phương thức `sort`, `binarySearch` trong `Collections`).
- ……

### Cơ chế Xóa kiểu (Type Erasure) là gì? Tại sao phải xóa kiểu?

**Java Generics được thực thi thông qua Type Erasure (Xóa kiểu): Các Generics instance lúc runtime không giữ lại tham số kiểu thực tế cụ thể, nhưng file `.class` vẫn có thể giữ lại thông tin khai báo Generics trong các attribute như `Signature`, và có thể đọc được thông qua Reflection API.**

Trình biên dịch trong quá trình biên dịch sẽ xóa động Generics `T` thành `Object` hoặc xóa `T extends xxx` thành kiểu giới hạn `xxx`.

Type Erasure giúp code Generics duy trì tính tương thích với các thư viện lớp Java và mã nhị phân trước khi đưa Generics vào. Trình biên dịch sẽ duy trì an toàn kiểu và ngữ nghĩa đa hình thông qua các phép ép kiểu cần thiết và phương thức cầu nối (Bridge Method).

Điều này nói nghe có vẻ hơi trừu tượng, tôi lấy một ví dụ:

```java
List<Integer> list = new ArrayList<>();

list.add(12);
// 1. Lúc biên dịch nếu thêm trực tiếp chuỗi sẽ báo lỗi
list.add("a");
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add", Object.class);
// 2. Lúc runtime thêm thông qua Reflection thì lại được
add.invoke(list, "kl");

System.out.println(list);
```

Xem thêm một ví dụ nữa: Do vấn đề Type Erasure, việc Overloading phương thức bên dưới sẽ báo lỗi biên dịch.

```java
public void print(List<String> list)  { }
public void print(List<Integer> list) { }
```

![Vấn đề của Type Erasure](https://oss.javaguide.cn/github/javaguide/java/basis/generics-runtime-erasure.png)

Nguyên nhân rất đơn giản, sau khi Type Erasure diễn ra, cả `List<String>` và `List<Integer>` sau khi biên dịch đều biến thành `List`.

**Nếu trình biên dịch đã xóa Generics, tại sao vẫn phải dùng Generics? Dùng Object thay thế không được sao?**

Câu hỏi này thực chất đang biến tướng kiểm tra tác dụng của Generics:

- Sử dụng Generics cho phép kiểm tra kiểu dữ liệu ngay ở giai đoạn biên dịch.
- Sử dụng kiểu `Object` cần thêm các phép ép kiểu thủ công, làm giảm tính đọc hiểu của code và tăng xác suất xảy ra lỗi.
- Generics có thể dùng kiểu tự giới hạn như `T extends Comparable`.

### Phương thức cầu nối (Bridge Method) là gì?

Phương thức cầu nối (`Bridge Method`) dùng để đảm bảo tính đa hình khi kế thừa Generic Class.

```java
class Node<T> {
    public T data;
    public Node(T data) { this.data = data; }
    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }

    // Node<T> sau khi bị xóa kiểu sẽ thành setData(Object data), mà lớp con MyNode lại không override phương thức đó, nên trình biên dịch sẽ tự động thêm phương thức cầu nối này để đảm bảo tính đa hình
    public void setData(Object data) {
        setData((Integer) data);
    }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

⚠️**Lưu ý**: Bridge Method do trình biên dịch tự động sinh ra, không phải viết thủ công.

### Generics có những hạn chế nào? Tại sao?

Các hạn chế của Generics thông thường do cơ chế Type Erasure gây ra. Sau khi bị xóa thành `Object` thì không thể đánh giá kiểu dữ liệu lúc runtime.

- Có thể khai báo biến thuộc kiểu `T`, nhưng không thể trực tiếp khởi tạo tham số kiểu qua `new T()`.
- Tham số Generics không thể là kiểu nguyên thủy. Vì kiểu nguyên thủy không phải là lớp con của `Object`, nên dùng kiểu đóng gói (Wrapper Class) tương ứng để thay thế.
- Không thể khởi tạo mảng của tham số Generics. Vì sau khi xóa thành `Object` sẽ không thể đánh giá kiểu.
- Không thể khởi tạo mảng Generics.
- Generics không thể dùng `instanceof` để đánh giá tham số kiểu T lúc runtime; `getClass()` sau khi bị xóa kiểu cũng không thể phân biệt các tham số Generics thực tế khác nhau (như `List<String>` và `List<Integer>` đều thu được `List.class`).
- Không thể implement hai Interface cùng loại với các tham số Generics khác nhau, vì sau khi xóa kiểu, các Bridge Method của nhiều lớp bố sẽ bị xung đột.
- Ngữ cảnh `static` của class không thể tham chiếu tham số kiểu do class đó khai báo, nhưng Generic Method static có thể khai báo và sử dụng tham số kiểu của riêng nó.
- ……

### Các đoạn code sau có thể biên dịch không, tại sao?

```java
public final class Algorithm {
    public static <T> T max(T x, T y) {
        return x > y ? x : y;
    }
}
```

Không thể biên dịch, vì cả x và y đều bị xóa thành kiểu `Object`, mà `Object` không thể dùng toán tử `>` để so sánh.

```java
public class Singleton<T> {

    public static T getInstance() {
        if (instance == null)
            instance = new Singleton<T>();

        return instance;
    }

    private static T instance = null;
}
```

Không thể biên dịch, vì field static và method static của class không thể tham chiếu tham số kiểu `T` do class khai báo. Method static có thể khai báo tham số kiểu của riêng nó, ví dụ `public static <T> T getInstance()`.

## Wildcards (Ký tự đại diện / Ký tự thông chiếu)

### Wildcard là gì? Có tác dụng gì?

Kiểu Generics là cố định, trong một số kịch bản sử dụng lại không được linh hoạt cho lắm, thế là Wildcard ra đời! Wildcard cho phép tham số kiểu biến đổi, dùng để giải quyết vấn đề Generics không thể hiệp biến (covariance).

Ví dụ:

```java
// Giới hạn kiểu phải là lớp con của Person
<? extends Person>
// Giới hạn kiểu phải là lớp bố của Manager
<? super Manager>
```

### Phân biệt Wildcard `?` và Generic Type `T` phổ biến?

- `T` có thể dùng để khai báo biến hoặc hằng số, còn `?` thì không.
- `T` thông thường dùng để khai báo Generic Class hoặc Method, còn Wildcard `?` thông thường dùng trong code gọi Generic Method và tham số hình thức.
- `T` lúc biên dịch sẽ bị xóa thành kiểu giới hạn hoặc `Object`. Wildcard `?` bên trong phương thức sẽ được trình biên dịch "bắt" (capture) thành một kiểu cụ thể nhưng chưa biết, do đó không thể ghi bất kỳ phần tử nào ngoại trừ `null` vào `List<?>`, nhưng có thể phối hợp sử dụng với Generic Method.

### Unbounded Wildcard (Wildcard không giới hạn) là gì?

Unbounded Wildcard có thể nhận bất kỳ dữ liệu kiểu Generics nào, dùng để thực thi các phương thức đơn giản không phụ thuộc vào tham số kiểu cụ thể, có thể bắt kiểu tham số và giao cho Generic Method xử lý.

```java
void testMethod(Person<?> p) {
  // Generic Method tự xử lý
}
```

**`List<?>` và `List` có khác nhau không?** Tất nhiên là có!

- `List<?> list` biểu thị kiểu phần tử của `list` là **một kiểu cụ thể nhưng chưa biết** (tức "tồn tại một kiểu `T` nào đó mà list là `List<T>`"), do đó trình biên dịch không cho phép thêm bất kỳ phần tử nào ngoài `null` vào đó để tránh không an toàn về kiểu.
- `List list` là Raw Type (kiểu nguyên thủy), sẽ bỏ qua một phần kiểm tra kiểu Generics, và không tương đương với `List<Object>`. Việc thêm phần tử vào đó thường sinh ra cảnh báo unchecked, và có thể đẩy lỗi kiểu dữ liệu về muộn hơn tới thời điểm runtime.

```java
List<?> list = new ArrayList<>();
list.add("sss");// Báo lỗi
List list2 = new ArrayList<>();
list2.add("sss");// Thông tin cảnh báo
```

### Upper Bounded Wildcard là gì? Lower Bounded Wildcard là gì?

Khi sử dụng Generics, chúng ta còn có thể giới hạn ranh giới trên và ranh giới dưới cho tham số thực tế truyền vào, như: **Tham số kiểu thực tế chỉ cho phép truyền vào lớp bố của một kiểu nào đó hoặc lớp con của một kiểu nào đó**.

**Upper Bounded Wildcard `extends` (Ranh giới trên)** biểu thị tham số kiểu thực tế phải là kiểu được chỉ định hoặc lớp con của nó.

Ví dụ:

```java
// Giới hạn bắt buộc phải là lớp con của lớp Person
<? extends Person>
```

Ranh giới kiểu có thể đặt nhiều cái, còn có thể giới hạn đối với kiểu `T`.

```java
<T extends T1 & T2>
<T extends XXX>
```

**Lower Bounded Wildcard `super` (Ranh giới dưới)** biểu thị tham số kiểu thực tế phải là kiểu được chỉ định hoặc lớp bố của nó.

Ví dụ:

```java
// Giới hạn bắt buộc phải là lớp bố của lớp Employee
List<? super Employee>
```

**`? extends xxx` và `? super xxx` khác nhau thế nào?**

Phạm vi nhận tham số kiểu thực tế của hai bên khác nhau. Đối với `List<? extends Xxx>`, có thể đọc ra dưới dạng `Xxx`, nhưng ngoại trừ `null` ra thì không thể ghi an toàn; đối với `List<? super Xxx>`, có thể ghi `Xxx` và các lớp con của nó vào, nhưng kết quả đọc ra chỉ có thể xem an toàn dưới dạng `Object`.

**Nguyên tắc PECS (Producer Extends, Consumer Super)**: Khi **lấy** phần tử từ cấu trúc dữ liệu ra thì dùng `extends` (Nhà sản xuất - Producer); khi **ghi** phần tử vào cấu trúc dữ liệu thì dùng `super` (Người tiêu thụ - Consumer). Ví dụ: `List<? extends Number>` chỉ có thể đọc `Number` từ đó ra, không thể ghi vào; `List<? super Integer>` có thể ghi `Integer` và lớp con của nó vào, khi đọc ra thu được `Object`. `Collections.copy(List<? super T> dest, List<? extends T> src)` chính là ví dụ điển hình: đọc từ `src`, ghi vào `dest`.

**`T extends xxx` và `? extends xxx` khác nhau thế nào?**

`T extends xxx` dùng để khai báo tham số kiểu có ranh giới trên, sau khi bị xóa kiểu sẽ thành `xxx`; `? extends xxx` dùng cho tham số Wildcard thực tế trong kiểu được tham số hóa, có thể xuất hiện ở các vị trí như field, biến cục bộ, tham số phương thức và kiểu trả về.

**Sự khác biệt giữa `Class<?>` và `Class`?**

Dùng trực tiếp Class sẽ có cảnh báo về kiểu, dùng `Class<?>` thì không có, vì Class là một Generic Class, nhận Raw Type sẽ sinh ra cảnh báo.

### Các đoạn code sau có thể biên dịch không, tại sao?

```java
class Shape { /* ... */ }
class Circle extends Shape { /* ... */ }
class Rectangle extends Shape { /* ... */ }

class Node<T> { /* ... */ }

Node<Circle> nc = new Node<>();
Node<Shape>  ns = nc;
```

Không thể, vì `Node<Circle>` không phải là lớp con của `Node<Shape>`.

```java
class Shape { /* ... */ }
class Circle extends Shape { /* ... */ }
class Rectangle extends Shape { /* ... */ }

class Node<T> { /* ... */ }
class ChildNode<T> extends Node<T>{

}
ChildNode<Circle> nc = new ChildNode<>();
Node<Circle>  ns = nc;
```

Có thể biên dịch, vì `ChildNode<Circle>` là lớp con của `Node<Circle>`.

```java
public static void print(List<? extends Number> list) {
    for (Number n : list)
        System.out.print(n + " ");
    System.out.println();
}
```

Có thể biên dịch, `List<? extends Number>` có thể lấy phần tử ra ngoài, nhưng không thể gọi `add()` để thêm phần tử.

## Tài liệu tham khảo

- Tài liệu chính thức Java: https://docs.oracle.com/javase/tutorial/java/generics/index.html
- Java cơ bản - Một bài viết hiểu trọn Generics: https://www.cnblogs.com/XiiX/p/14719568.html
