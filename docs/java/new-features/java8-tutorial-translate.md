---
title: 《Java8 指南》中文翻译
description: 翻译与整理 Java 8 教程，涵盖 Lambda、方法引用、接口默认方法、Stream 等新特性与示例代码。
category: Java
tag:
  - Java新特性
head:
  - - meta
    - name: keywords
      content: Java 8,指南,Lambda,方法引用,默认方法,Stream API,函数式接口,Date/Time API
---

# Bản dịch tiếng Việt 《Hướng dẫn Java 8》

JDK 8 được phát hành vào ngày 18 tháng 3 năm 2014, đây là một phiên bản LTS (Long Term Support - Hỗ trợ dài hạn), là một trong những phiên bản quan trọng nhất trong lịch sử Java. Cho đến nay, có tổng cộng 5 phiên bản hỗ trợ dài hạn là JDK 8, JDK 11, JDK 17, JDK 21 và JDK 25.

JDK 8 đã giới thiệu nhiều tính năng mới quan trọng. Bài viết này sẽ chọn lọc một số tính năng mới tương đối quan trọng để giới thiệu chi tiết:

- Biểu thức Lambda (Lambda expression)
- Tham chiếu phương thức (Method reference)
- Phương thức mặc định trong Interface (Default methods)
- Stream API
- Functional interface (Interface chức năng)
- Lớp Optional
- Date/Time API
- Tăng cường Annotation (Annotation enhancements)

Hình dưới đây thống kê số lượng tính năng mới và thời gian cập nhật mang lại bởi từng phiên bản từ JDK 8 đến JDK 24:

![](https://oss.javaguide.cn/github/javaguide/java/new-features/jdk8~jdk24.png)

Cùng với việc Java 8 ngày càng trở nên phổ biến, nhiều người đề cập rằng trong các buổi phỏng vấn các điểm kiến thức về Java 8 cũng được hỏi rất thường xuyên. Đáp ứng yêu cầu và nhu cầu của mọi người, tôi định làm một tổng kết về phần kiến thức này. Ban đầu tôi định tự tổng hợp, sau đó thấy trên GitHub có một repository liên quan tại địa chỉ:
[https://github.com/winterbe/java8-tutorial](https://github.com/winterbe/java8-tutorial). Repository này bằng tiếng Anh, tôi đã dịch nó sang và thêm vào cũng như sửa đổi một số nội dung, dưới đây là nội dung chính.

---

Chào mừng bạn đọc phần giới thiệu của tôi về Java 8. Hướng dẫn này sẽ từng bước hướng dẫn bạn hoàn thành tất cả các tính năng ngôn ngữ mới. Trên cơ sở các ví dụ mã nguồn ngắn gọn, bạn sẽ học cách sử dụng các phương thức mặc định trong interface, biểu thức lambda, tham chiếu phương thức và annotation có thể lặp lại (repeatable annotation). Ở phần cuối của bài viết này, bạn sẽ quen thuộc với những thay đổi API mới nhất như stream, functional interface (Functional Interfaces), các mở rộng của lớp Map và Date API mới. Không có các đoạn văn bản dài dòng nhàm chán, chỉ có một tập hợp các đoạn mã kèm ghi chú.

## Phương thức mặc định của Interface (Default Methods for Interfaces)

Java 8 giúp chúng ta có thể bổ sung triển khai phương thức không trừu tượng vào interface thông qua việc sử dụng từ khóa `default`. Tính năng này còn được gọi là [phương thức mở rộng ảo (virtual extension methods)](http://stackoverflow.com/a/24102730).

Ví dụ đầu tiên:

```java
interface Formula{

    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }

}
```

Interface Formula ngoài phương thức trừu tượng tính toán công thức `calculate` còn định nghĩa phương thức mặc định `sqrt`. Lớp triển khai interface đó chỉ cần triển khai phương thức trừu tượng `calculate`. Phương thức mặc định `sqrt` có thể trực tiếp sử dụng. Tất nhiên bạn cũng có thể trực tiếp tạo đối tượng thông qua interface, sau đó triển khai phương thức mặc định trong interface là được, chúng ta qua mã nguồn để minh họa cách này.

```java
public class Main {

  public static void main(String[] args) {
    // Truy cập interface thông qua anonymous inner class
    Formula formula = new Formula() {
        @Override
        public double calculate(int a) {
            return sqrt(a * 100);
        }
    };

    System.out.println(formula.calculate(100));     // 100.0
    System.out.println(formula.sqrt(16));           // 4.0

  }

}
```

formula được triển khai như một đối tượng ẩn danh. Đoạn mã này rất dễ hiểu, 6 dòng mã triển khai việc tính toán `sqrt(a * 100)`. Trong phần tiếp theo, chúng ta sẽ thấy trong Java 8 có một cách tốt hơn và tiện lợi hơn để triển khai đối tượng phương thức đơn.

**Ghi chú của người dịch:** Dù là abstract class hay interface, đều có thể truy cập thông qua anonymous inner class. Không thể trực tiếp tạo đối tượng thông qua abstract class hoặc interface. Đối với cách truy cập interface qua anonymous inner class ở trên, chúng ta có thể hiểu như sau: một inner class triển khai phương thức trừu tượng trong interface và trả về một đối tượng inner class, sau đó chúng ta để tham chiếu của interface trỏ đến đối tượng này.

## Biểu thức Lambda (Lambda expressions)

Đầu tiên hãy xem trong phiên bản Java cũ sắp xếp các chuỗi như thế nào:

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

Chỉ cần truyền vào phương thức static `Collections.sort` một đối tượng List cùng một comparator để sắp xếp theo thứ tự chỉ định. Cách làm thông thường là tạo một đối tượng comparator ẩn danh rồi truyền nó vào phương thức `sort`.

Trong Java 8 bạn không cần thiết phải sử dụng cách đối tượng ẩn danh truyền thống này nữa, Java 8 cung cấp cú pháp gọn gàng hơn là biểu thức lambda:

```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

Có thể thấy, mã nguồn trở nên ngắn hơn và có tính đọc cao hơn, nhưng trên thực tế còn có thể viết ngắn hơn nữa:

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

Đối với thân hàm chỉ có một dòng mã, bạn có thể bỏ dấu ngoặc nhọn {} cũng như từ khóa return, nhưng bạn còn có thể viết ngắn hơn nữa:

```java
names.sort((a, b) -> b.compareTo(a));
```

Bản thân lớp List đã có một phương thức `sort`. Và trình biên dịch Java có thể tự động suy luận ra kiểu tham số, nên bạn không cần phải viết lại kiểu một lần nữa. Tiếp theo chúng ta xem biểu thức lambda còn có cách dùng nào khác.

## Functional Interface (Interface Chức Năng)

**Ghi chú của người dịch:** Bản gốc giải thích phần này chưa rõ ràng lắm, nên đã sửa đổi!

Các nhà thiết kế ngôn ngữ Java đã dành nhiều tâm huyết để suy nghĩ làm thế nào giúp các hàm hiện có hỗ trợ tốt cho Lambda. Phương pháp cuối cùng được áp dụng là: bổ sung khái niệm Functional Interface. **“Functional Interface” là interface chỉ chứa duy nhất một phương thức trừu tượng, nhưng có thể có nhiều phương thức không trừu tượng (tức là phương thức mặc định đã đề cập ở trên).** Những interface như vậy có thể làm kiểu đích (target type) cho biểu thức lambda. `java.lang.Runnable` và `java.util.concurrent.Callable` là hai ví dụ điển hình nhất của functional interface. Java 8 bổ sung thêm một annotation đặc biệt là `@FunctionalInterface`, nhưng annotation này thường không bắt buộc. Chỉ cần interface đáp ứng định nghĩa của functional interface, trình biên dịch Java có thể coi nó làm target type của biểu thức lambda. Thông thường nên khai báo annotation `@FunctionalInterface` trên interface, như vậy khi trình biên dịch phát hiện interface được đánh dấu không đáp ứng yêu cầu của functional interface sẽ báo lỗi, như hình bên dưới.

![Annotation @FunctionalInterface](https://oss.javaguide.cn/github/javaguide/java/@FunctionalInterface.png)

Ví dụ:

```java
@FunctionalInterface
public interface Converter<F, T> {
  T convert(F from);
}
```

```java
    // TODO 将数字字符串转换为整数类型
    Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
    Integer converted = converter.convert("123");
    System.out.println(converted.getClass()); //class java.lang.Integer
```

**Ghi chú của người dịch:** Hầu hết các functional interface đều không cần chúng ta tự viết, Java 8 đã triển khai sẵn cho chúng ta, các interface này đều nằm trong package java.util.function.

## Tham Chiếu Phương Thức và Constructor (Method and Constructor References)

Mã nguồn trong phần trước còn có thể biểu diễn thông qua tham chiếu phương thức static:

```java
    Converter<String, Integer> converter = Integer::valueOf;
    Integer converted = converter.convert("123");
    System.out.println(converted.getClass());   //class java.lang.Integer
```

Java 8 cho phép bạn truyền tham chiếu phương thức hoặc constructor thông qua từ khóa `::`. Ví dụ trên hiển thị cách tham chiếu phương thức static. Nhưng chúng ta cũng có thể tham chiếu phương thức của đối tượng:

```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```

Tiếp theo hãy xem constructor được tham chiếu thông qua từ khóa `::` như thế nào, đầu tiên chúng ta định nghĩa một lớp đơn giản chứa nhiều constructor:

```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

Tiếp theo chúng ta chỉ định một interface factory đối tượng dùng để tạo đối tượng Person:

```java
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

Ở đây chúng ta sử dụng tham chiếu constructor để liên kết chúng lại với nhau, thay vì triển khai thủ công một factory hoàn chỉnh:

```java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```

Chúng ta chỉ cần sử dụng `Person::new` để lấy tham chiếu constructor của lớp Person, trình biên dịch Java sẽ tự động dựa vào kiểu tham số của phương thức `PersonFactory.create` để chọn constructor phù hợp.

## Scope của Biểu thức Lambda (Lambda Scopes)

### Truy cập biến cục bộ

Chúng ta có thể truy cập trực tiếp biến cục bộ bên ngoài trong biểu thức lambda:

```java
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

Nhưng khác với đối tượng ẩn danh, biến num ở đây không cần phải khai báo là final, đoạn mã này vẫn chính xác:

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

Tuy nhiên num ở đây bắt buộc không được sửa đổi bởi mã phía sau (tức là mang ngữ nghĩa final ẩn), ví dụ đoạn mã dưới đây sẽ không thể biên dịch:

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;//在lambda表达式中试图修改num同样是不允许的。
```

### Truy cập trường và biến static

So với biến cục bộ, trong biểu thức lambda chúng ta có quyền truy cập đọc và ghi đối với các trường thể hiện (instance fields) và biến static. Hành vi này hoàn toàn giống với đối tượng ẩn danh.

```java
class Lambda4 {
    static int outerStaticNum;
    int outerNum;

    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
            outerNum = 23;
            return String.valueOf(from);
        };

        Converter<Integer, String> stringConverter2 = (from) -> {
            outerStaticNum = 72;
            return String.valueOf(from);
        };
    }
}
```

### Truy cập phương thức mặc định của interface

Còn nhớ ví dụ formula trong phần đầu tiên không? Interface `Formula` định nghĩa một phương thức mặc định `sqrt`, có thể truy cập phương thức đó từ mỗi instance formula chứa đối tượng ẩn danh. Điều này không áp dụng cho biểu thức lambda.

Không thể truy cập phương thức mặc định từ biểu thức lambda, do đó đoạn mã dưới đây không thể biên dịch:

```java
Formula formula = (a) -> sqrt(a * 100);
```

## Built-in Functional Interfaces (Interface Chức Năng Dựng Sẵn)

JDK 1.8 API chứa nhiều functional interface dựng sẵn. Một số interface trong số đó khá phổ biến trong các phiên bản Java cũ như `Comparator` hoặc `Runnable`, những interface này đều đã được thêm annotation `@FunctionalInterface` để có thể dùng trên biểu thức lambda.

Nhưng Java 8 API cũng cung cấp rất nhiều functional interface hoàn toàn mới giúp công việc lập trình của bạn tiện lợi hơn, có những interface đến từ thư viện [Google Guava](https://code.google.com/p/guava-libraries/), ngay cả khi bạn rất quen thuộc với chúng, vẫn cần thiết phải xem chúng được mở rộng để sử dụng trên lambda như thế nào.

### Predicate

Interface Predicate là interface **kiểu đoán nhận (assertion)** chỉ có một tham số và trả về giá trị kiểu boolean. Interface này chứa nhiều phương thức mặc định để kết hợp các Predicate thành các logic phức tạp khác (như AND, OR, NOT):

**Ghi chú của người dịch:** Mã nguồn của Predicate interface như sau:

```java
package java.util.function;
import java.util.Objects;

@FunctionalInterface
public interface Predicate<T> {

    // 该方法是接受一个传入类型,返回一个布尔值.此方法应用于判断.
    boolean test(T t);

    //and方法与关系型运算符"&&"相似，两边都成立才返回true
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    // 与关系运算符"!"相似，对判断进行取反
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    //or方法与关系型运算符"||"相似，两边只要有一个成立就返回true
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
   // 该方法接收一个Object对象,返回一个Predicate类型.此方法用于判断第一个test的方法与第二个test方法相同(equal).
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
```

Ví dụ:

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

### Function

Interface Function nhận một tham số và tạo ra kết quả. Phương thức mặc định có thể dùng để chuỗi hóa (chain) nhiều hàm lại với nhau (compose, andThen):

**Ghi chú của người dịch:** Mã nguồn của Function interface như sau:

```java

package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {

    //将Function对象应用到输入的参数上，然后返回计算结果。
    R apply(T t);
    //将两个Function整合，并返回一个能够执行两个Function对象功能的Function对象。
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    //
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);
backToString.apply("123");     // "123"
```

### Supplier

Interface Supplier tạo ra kết quả của kiểu Generics được cho. Khác với interface Function, interface Supplier không nhận tham số.

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

### Consumer

Interface Consumer biểu thị thao tác cần thực hiện đối với một tham số đầu vào đơn lẻ.

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

### Comparator

Comparator là interface kinh điển trong Java cũ, Java 8 đã bổ sung thêm nhiều phương thức mặc định trên interface này:

```java
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```

## Optional

Optional không phải là functional interface, mà là một container được sử dụng để biểu thị rõ ràng "có thể không có giá trị". Sử dụng đúng cách có thể giảm bớt một phần kiểm tra `null` thủ công, nhưng không thể đảm bảo chương trình không còn xuất hiện `NullPointerException`. Đây là một khái niệm quan trọng ở phần tiếp theo, hãy cùng tìm hiểu nhanh về nguyên lý hoạt động của Optional.

Optional là một container đơn giản, nó hoặc chứa một giá trị khác `null`, hoặc rỗng. Trong kịch bản giá trị trả về thích hợp để biểu thị "kết quả có thể không tồn tại", có thể trả về Optional thay vì dùng `null` để biểu thị không có kết quả.

Ghi chú của người dịch: Tác dụng của từng phương thức trong ví dụ đã được thêm vào.

```java
//of()：为非null的值创建一个Optional
Optional<String> optional = Optional.of("bam");
// isPresent()：如果值存在返回true，否则返回false
optional.isPresent();           // true
//get()：如果Optional有值则将其返回，否则抛出NoSuchElementException
optional.get();                 // "bam"
//orElse()：如果有值则将其返回，否则返回指定的其它值
optional.orElse("fallback");    // "bam"
//ifPresent()：如果Optional实例有值则为其调用consumer，否则不做处理
optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

Khuyên đọc: [[Java8] Làm thế nào để sử dụng Optional đúng cách](https://blog.kaaass.net/archives/764)

## Streams (Luồng)

`java.util.stream.Stream` biểu thị chuỗi các thao tác có thể áp dụng trên một nhóm phần tử và thực thi lần lượt. Thao tác Stream được chia thành thao tác trung gian (intermediate operation) hoặc thao tác kết thúc (terminal operation). Thao tác kết thúc trả về kết quả tính toán của một kiểu cụ thể, còn thao tác trung gian trả về chính Stream đó, nhờ vậy bạn có thể chuỗi hóa (chain) nhiều thao tác lại với nhau. Stream có thể được tạo từ nhiều nguồn dữ liệu như Collection, Array, hàm sinh (generator function); bản thân Map không có phương thức `stream()`, nhưng có thể tạo stream thông qua các view key, value hoặc entry của nó. Thao tác của Stream có thể thực thi nối tiếp (sequential) hoặc song song (parallel).

Đầu tiên hãy xem Stream được sử dụng như thế nào, trước tiên tạo dữ liệu List cần dùng cho mã nguồn ví dụ:

```java
List<String> stringList = new ArrayList<>();
stringList.add("ddd2");
stringList.add("aaa2");
stringList.add("bbb1");
stringList.add("aaa1");
stringList.add("bbb3");
stringList.add("ccc");
stringList.add("bbb2");
stringList.add("ddd1");
```

Java 8 đã mở rộng các lớp tập hợp, có thể thông qua Collection.stream() hoặc Collection.parallelStream() để tạo một Stream. Mấy phần tiếp theo sẽ giải thích chi tiết các thao tác Stream thường dùng:

### Filter (Lọc)

Lọc thông qua một predicate interface để lọc và chỉ giữ lại các phần tử đáp ứng điều kiện, thao tác này thuộc **thao tác trung gian**, nên chúng ta có thể áp dụng các thao tác Stream khác (như forEach) trên kết quả sau khi lọc. forEach cần một hàm để thực thi lần lượt trên các phần tử sau khi lọc. forEach là một thao tác kết thúc, nên chúng ta không thể thực thi các thao tác Stream khác sau forEach.

```java
        // 测试 Filter(过滤)
        stringList
                .stream()
                .filter((s) -> s.startsWith("a"))
                .forEach(System.out::println);//aaa2 aaa1
```

forEach được thiết kế cho Lambda, giữ được phong cách gọn gàng nhất. Hơn nữa bản thân biểu thức Lambda có thể tái sử dụng, rất tiện lợi.

### Sorted (Sắp xếp)

Sắp xếp là một **thao tác trung gian**, trả về Stream đã được sắp xếp. **Nếu bạn không chỉ định một Comparator tùy chỉnh thì sẽ sử dụng sắp xếp mặc định.**

```java
        // 测试 Sort (排序)
        stringList
                .stream()
                .sorted()
                .filter((s) -> s.startsWith("a"))
                .forEach(System.out::println);// aaa1 aaa2
```

Cần lưu ý rằng, sắp xếp chỉ tạo ra một Stream đã được sắp xếp, chứ không ảnh hưởng đến nguồn dữ liệu ban đầu, sau khi sắp xếp thì dữ liệu ban đầu stringList sẽ không bị sửa đổi:

```java
    System.out.println(stringList);// ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
```

### Map (Ánh xạ)

Thao tác trung gian map sẽ chuyển đổi các phần tử thành các đối tượng khác lần lượt dựa trên Function interface được chỉ định.

Ví dụ dưới đây hiển thị việc chuyển đổi chuỗi thành chuỗi chữ hoa. Bạn cũng có thể thông qua map để chuyển đối tượng thành các kiểu khác, kiểu Stream mà map trả về được quyết định bởi giá trị trả về của hàm mà bạn truyền vào map.

```java
        // 测试 Map 操作
        stringList
                .stream()
                .map(String::toUpperCase)
                .sorted((a, b) -> b.compareTo(a))
                .forEach(System.out::println);// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "BBB1", "AAA2", "AAA1"
```

### Match (Khớp / Khớp lệnh)

Stream cung cấp nhiều thao tác khớp, cho phép kiểm tra xem Predicate được chỉ định có khớp với toàn bộ Stream hay không. Tất cả các thao tác khớp đều là **thao tác kết thúc**, và trả về một giá trị kiểu boolean.

```java
        // 测试 Match (匹配)操作
        boolean anyStartsWithA =
                stringList
                        .stream()
                        .anyMatch((s) -> s.startsWith("a"));
        System.out.println(anyStartsWithA);      // true

        boolean allStartsWithA =
                stringList
                        .stream()
                        .allMatch((s) -> s.startsWith("a"));

        System.out.println(allStartsWithA);      // false

        boolean noneStartsWithZ =
                stringList
                        .stream()
                        .noneMatch((s) -> s.startsWith("z"));

        System.out.println(noneStartsWithZ);      // true
```

### Count (Đếm)

Đếm là một **thao tác kết thúc**, trả về số lượng phần tử trong Stream, **kiểu giá trị trả về là long**.

```java
      //测试 Count (计数)操作
        long startsWithB =
                stringList
                        .stream()
                        .filter((s) -> s.startsWith("b"))
                        .count();
        System.out.println(startsWithB);    // 3
```

### Reduce (Gom tụ / Quy ước)

Đây là một **thao tác kết thúc**, cho phép thông qua hàm được chỉ định để gom tụ nhiều phần tử trong stream thành một phần tử, kết quả sau khi gom tụ được biểu thị thông qua Optional interface:

```java
        //测试 Reduce (规约)操作
        Optional<String> reduced =
                stringList
                        .stream()
                        .sorted()
                        .reduce((s1, s2) -> s1 + "#" + s2);

        reduced.ifPresent(System.out::println);//aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2
```

**Ghi chú của người dịch:** Tác dụng chính của phương thức này là kết hợp các phần tử Stream lại với nhau. Nó cung cấp một giá trị khởi tạo (seed), sau đó dựa vào quy tắc tính toán (BinaryOperator) để kết hợp với phần tử thứ nhất, thứ hai, thứ n của Stream phía trước. Theo nghĩa này, nối chuỗi, sum, min, max, average của giá trị số đều là các trường hợp đặc biệt của reduce. Ví dụ sum của Stream tương đương với `Integer sum = integers.reduce(0, (a, b) -> a+b);` Cũng có trường hợp không có giá trị khởi tạo, lúc này sẽ kết hợp hai phần tử đầu tiên của Stream lại với nhau, trả về là Optional.

```java
// 字符串连接，concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat);
// 求最小值，minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min);
// 求和，sumValue = 10, 有起始值
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// 求和，sumValue = 10, 无起始值
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// 过滤，字符串连接，concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F").
 filter(x -> x.compareTo("Z") > 0).
 reduce("", String::concat);
```

Mã nguồn trên ví dụ như reduce() trong ví dụ đầu tiên, tham số thứ nhất (ký tự trống) là giá trị khởi tạo, tham số thứ hai (String::concat) là BinaryOperator. Loại reduce() có giá trị khởi tạo này đều trả về đối tượng cụ thể. Còn đối với reduce() không có giá trị khởi tạo ở ví dụ thứ tư, do có thể không có đủ phần tử, nên trả về là Optional, xin lưu ý sự khác biệt này. Xem thêm nội dung tại: [IBM: Giải thích chi tiết Streams API trong Java 8](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)

## Parallel Streams (Luồng Song Song)

Phía trước đã đề cập Stream có hai loại nối tiếp và song song, thao tác trên Stream nối tiếp được hoàn thành lần lượt trong một thread, còn Stream song song được thực thi đồng thời trên nhiều thread.

Ví dụ dưới đây hiển thị cách nâng cao hiệu năng thông qua Stream song song:

Đầu tiên chúng ta tạo một danh sách lớn không có phần tử trùng lặp:

```java
int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
```

Chúng ta lần lượt sắp xếp nó theo hai cách nối tiếp và song song, cuối cùng xem so sánh thời gian sử dụng.

### Sequential Sort (Sắp xếp nối tiếp)

```java
//串行排序
long t0 = System.nanoTime();
long count = Arrays.stream(list.stream().sorted().toArray()).count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("sequential sort took: %d ms", millis));
```

```plain
1000000
sequential sort took: 709 ms//Thời gian sắp xếp nối tiếp sử dụng
```

### Parallel Sort (Sắp xếp song song)

```java
//并行排序
long t0 = System.nanoTime();

long count = Arrays.stream(list.parallelStream().sorted().toArray()).count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("parallel sort took: %d ms", millis));

```

```java
1000000
parallel sort took: 475 ms//Thời gian sắp xếp song song sử dụng
```

Hai đoạn mã trên hầu như giống hệt nhau, nhưng phiên bản song song nhanh hơn khoảng 50%, thay đổi duy nhất cần làm là đổi `stream()` thành `parallelStream()`.

## Maps

Như đã đề cập trước đó, kiểu Map không hỗ trợ streams, tuy nhiên Map cung cấp một số phương thức mới hữu ích để xử lý các tác vụ hàng ngày. Bản thân interface Map không có phương thức `stream()` khả dụng, nhưng bạn có thể tạo luồng chuyên dụng trên key, value hoặc thông qua `map.keySet().stream()`, `map.values().stream()` và `map.entrySet().stream()`.

Ngoài ra, Maps hỗ trợ nhiều phương thức mới và hữu ích để thực thi các tác vụ phổ biến.

```java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val));//val0 val1 val2 val3 val4 val5 val6 val7 val8 val9
```

`putIfAbsent` giúp chúng ta tránh việc viết thêm mã kiểm tra null; `forEach` nhận một consumer để thao tác trên từng phần tử trong map.

Ví dụ này hiển thị cách sử dụng hàm để tính toán mã trên map:

```java
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3);             // val33

map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9);     // false

map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23);    // true

map.computeIfAbsent(3, num -> "bam");
map.get(3);             // val33
```

Tiếp theo hiển thị cách xóa một mục trong Map mà cả key và value đều khớp:

```java
map.remove(3, "val3");
map.get(3);             // val33
map.remove(3, "val33");
map.get(3);             // null
```

Một phương thức hữu ích khác:

```java
map.getOrDefault(42, "not found");  // not found
```

Việc gộp (merge) các phần tử của Map cũng trở nên rất dễ dàng:

```java
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9
map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9concat
```

Những gì Merge làm là nếu key chưa tồn tại thì chèn vào, nếu không thì thực hiện thao tác gộp đối với giá trị tương ứng với key ban đầu và chèn lại vào map.

## Date API (API Ngày Tháng)

Java 8 chứa một API ngày và giờ hoàn toàn mới dưới package `java.time`. Date API mới tương tự như thư viện Joda-Time, nhưng chúng không giống nhau. Ví dụ dưới đây bao hàm phần quan trọng nhất của API mới này. Người dịch đã tham khảo các sách liên quan để sửa đổi hầu hết phần nội dung này.

**Ghi chú của người dịch (Tóm tắt):**

- Lớp Clock cung cấp phương thức truy cập ngày và giờ hiện tại, Clock nhạy cảm với múi giờ, có thể dùng để lấy số millisecond hiện tại. Một thời điểm cụ thể cũng có thể biểu thị bằng lớp `Instant`, lớp `Instant` cũng có thể dùng để tạo đối tượng `java.util.Date` của phiên bản cũ.

- Trong API mới múi giờ được biểu thị bằng ZoneId. Múi giờ có thể dễ dàng lấy được bằng phương thức static `of`. Class trừu tượng `ZoneId` (trong package `java.time`) biểu thị một định danh khu vực. Nó có một phương thức static tên là `getAvailableZoneIds`, trả về tất cả định danh khu vực.

- Trong JDK 1.8 bổ sung thêm các class như LocalDate và LocalDateTime để giải quyết phương thức xử lý ngày tháng, đồng thời giới thiệu một class mới DateTimeFormatter để giải quyết vấn đề định dạng ngày tháng. Có thể dùng Instant thay cho Date, LocalDateTime thay cho Calendar, DateTimeFormatter thay cho SimpleDateFormat.

### Clock

Lớp Clock cung cấp phương thức truy cập ngày và giờ hiện tại, Clock nhạy cảm với múi giờ, có thể dùng để lấy số millisecond hiện tại. Một thời điểm cụ thể cũng có thể biểu thị bằng lớp `Instant`, lớp `Instant` cũng có thể dùng để tạo đối tượng `java.util.Date` của phiên bản cũ.

```java
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();
System.out.println(millis);//1552379579043
Instant instant = clock.instant();
System.out.println(instant);
Date legacyDate = Date.from(instant); //2019-03-12T08:46:42.588Z
System.out.println(legacyDate);//Tue Mar 12 16:32:59 CST 2019
```

### Timezones (Múi giờ)

Trong API mới múi giờ được biểu thị bằng ZoneId. Múi giờ có thể dễ dàng lấy được bằng phương thức static `of`. Class trừu tượng `ZoneId` (trong package `java.time`) biểu thị một định danh khu vực. Nó có một phương thức static tên là `getAvailableZoneIds`, trả về tất cả định danh khu vực.

```java
//输出所有区域标识符
System.out.println(ZoneId.getAvailableZoneIds());

ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules());// ZoneRules[currentStandardOffset=+01:00]
System.out.println(zone2.getRules());// ZoneRules[currentStandardOffset=-03:00]
```

### LocalTime (Giờ địa phương)

LocalTime định nghĩa một thời gian không có thông tin múi giờ, ví dụ 10 giờ tối hoặc 17:30:15. Ví dụ dưới đây sử dụng múi giờ được tạo trong mã nguồn phía trước để tạo hai thời gian địa phương. Sau đó so sánh thời gian và tính khoảng cách thời gian giữa hai thời gian theo đơn vị giờ và phút:

```java
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);
System.out.println(now1.isBefore(now2));  // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween);       // -3
System.out.println(minutesBetween);     // -239
```

LocalTime cung cấp nhiều phương thức factory để đơn giản hóa việc tạo đối tượng, bao gồm cả việc parse chuỗi thời gian.

```java
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);       // 23:59:59
DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);   // 13:37
```

### LocalDate (Ngày địa phương)

LocalDate biểu thị một ngày xác định, ví dụ 2014-03-11. Giá trị đối tượng này là bất biến (immutable), cách dùng về cơ bản giống với LocalTime. Ví dụ dưới đây hiển thị cách cộng trừ ngày/tháng/năm cho đối tượng Date. Ngoài ra cần lưu ý là các đối tượng này là bất biến, thao tác trả về luôn là một instance mới.

```java
LocalDate today = LocalDate.now();//获取现在的日期
System.out.println("今天的日期: "+today);//2019-03-12
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
System.out.println("明天的日期: "+tomorrow);//2019-03-13
LocalDate yesterday = tomorrow.minusDays(2);
System.out.println("昨天的日期: "+yesterday);//2019-03-11
LocalDate independenceDay = LocalDate.of(2019, Month.MARCH, 12);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println("今天是周几:"+dayOfWeek);//TUESDAY
```

Parse một kiểu LocalDate từ chuỗi cũng đơn giản như parse LocalTime, dưới đây là ví dụ sử dụng `DateTimeFormatter` để parse chuỗi:

```java
    String str1 = "2014==04==12 01时06分09秒";
        // 根据需要解析的日期、时间字符串定义解析所用的格式器
        DateTimeFormatter fomatter1 = DateTimeFormatter
                .ofPattern("yyyy==MM==dd HH时mm分ss秒");

        LocalDateTime dt1 = LocalDateTime.parse(str1, fomatter1);
        System.out.println(dt1); // 输出 2014-04-12T01:06:09

        String str2 = "2014$$$四月$$$13 20小时";
        DateTimeFormatter fomatter2 = DateTimeFormatter
                .ofPattern("yyy$$$MMM$$$dd HH小时");
        LocalDateTime dt2 = LocalDateTime.parse(str2, fomatter2);
        System.out.println(dt2); // 输出 2014-04-13T20:00

```

Lại xem một ví dụ sử dụng `DateTimeFormatter` để định dạng ngày tháng:

```java
LocalDateTime rightNow=LocalDateTime.now();
String date=DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(rightNow);
System.out.println(date);//2019-03-12T16:26:48.29
DateTimeFormatter formatter=DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
System.out.println(formatter.format(rightNow));//2019-03-12 16:26:48
```

**🐛 Sửa lỗi (Xem: [issue#1157](https://github.com/Snailclimb/JavaGuide/issues/1157))**: Khi sử dụng `YYYY` để hiển thị năm, sẽ hiển thị năm của tuần chứa thời gian hiện tại, vào tuần chuyển giao năm (跨年周) sẽ có vấn đề. Trong trường hợp thông thường đều sử dụng `yyyy` để hiển thị chính xác năm.

Ví dụ hiển thị lỗi ngày tháng do chuyển giao năm:

```java
LocalDateTime rightNow = LocalDateTime.of(2020, 12, 31, 12, 0, 0);
String date= DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(rightNow);
// 2020-12-31T12:00:00
System.out.println(date);
DateTimeFormatter formatterOfYYYY = DateTimeFormatter.ofPattern("YYYY-MM-dd HH:mm:ss");
// 2021-12-31 12:00:00
System.out.println(formatterOfYYYY.format(rightNow));

DateTimeFormatter formatterOfYyyy = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
// 2020-12-31 12:00:00
System.out.println(formatterOfYyyy.format(rightNow));
```

Từ hình bên dưới có thể thấy rõ hơn lỗi cụ thể, và IDEA đã gợi ý thông minh nên ưu tiên sử dụng `yyyy` thay vì `YYYY`.

![](https://oss.javaguide.cn/github/javaguide/java/new-features/2021042717491413.png)

### LocalDateTime (Ngày giờ địa phương)

LocalDateTime biểu thị đồng thời cả giờ và ngày, tương đương với việc gộp nội dung hai phần trước vào một đối tượng. LocalDateTime giống như LocalTime và LocalDate, đều là bất biến. LocalDateTime cung cấp một số phương thức có thể truy cập các trường cụ thể.

```java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek);      // WEDNESDAY

Month month = sylvester.getMonth();
System.out.println(month);          // DECEMBER

long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);    // 1439
```

Chỉ cần gắn thêm thông tin múi giờ, là có thể chuyển đổi nó thành một đối tượng mốc thời gian Instant, đối tượng mốc thời gian Instant có thể dễ dàng chuyển đổi thành `java.util.Date` kiểu cũ.

```java
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();

Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014
```

Định dạng LocalDateTime cũng giống như định dạng giờ và ngày, ngoài việc sử dụng định dạng được định nghĩa sẵn, chúng ta cũng có thể tự định nghĩa định dạng:

```java
DateTimeFormatter formatter =
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");
LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = formatter.format(parsed);
System.out.println(string);     // Nov 03, 2014 - 07:13
```

Khác với java.text.NumberFormat, DateTimeFormatter phiên bản mới là bất biến, nên nó an toàn với luồng (thread-safe).
Thông tin chi tiết về định dạng ngày giờ ở [đây](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html).

## Annotations (Ghi chú)

Trong Java 8 đã hỗ trợ đa annotation (multiple annotations), trước tiên hãy xem ví dụ để hiểu ý nghĩa là gì.
Đầu tiên định nghĩa một lớp wrapper annotation Hints dùng để chứa một nhóm các annotation Hint cụ thể:

```java
@Retention(RetentionPolicy.RUNTIME)
@interface Hints {
    Hint[] value();
}
@Repeatable(Hints.class)
@interface Hint {
    String value();
}
```

Java 8 cho phép chúng ta sử dụng cùng một kiểu annotation nhiều lần, chỉ cần đánh dấu `@Repeatable` cho annotation đó là được.

Ví dụ 1: Sử dụng wrapper class làm container để lưu nhiều annotation (cách cũ)

```java
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}
```

Ví dụ 2: Sử dụng đa annotation (cách mới)

```java
@Hint("hint1")
@Hint("hint2")
class Person {}
```

Trong ví dụ thứ hai, trình biên dịch Java sẽ ẩn định nghĩa giúp bạn annotation @Hints, hiểu được điểm này sẽ giúp ích cho bạn khi dùng Reflection để lấy các thông tin này:

```java
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null
Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);          // 2
```

Ngay cả khi chúng ta không định nghĩa annotation `@Hints` trên class `Person`, chúng ta vẫn có thể thông qua `getAnnotation(Hints.class)` để lấy annotation `@Hints`, cách tiện lợi hơn là sử dụng `getAnnotationsByType` có thể trực tiếp lấy được tất cả các annotation `@Hint`.
Ngoài ra annotation trong Java 8 còn được bổ sung thêm vào hai target mới:

```java
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@interface MyAnnotation {}
```

## Tiếp theo đi đâu từ đây?

Về các tính năng mới của Java 8 thì viết tới đây thôi, chắc chắn còn nhiều tính năng hơn đang chờ khám phá. Trong JDK 1.8 còn rất nhiều thứ hữu ích, ví dụ `Arrays.parallelSort`, `StampedLock` và `CompletableFuture`,...

<!-- @include: @article-footer.snippet.md -->
