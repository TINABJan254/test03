---
title: Tóm tắt các từ khóa trong Java
description: Tổng hợp hệ thống các từ khóa thường dùng trong Java: giải thích chi tiết cách dùng và sự khác biệt giữa các từ khóa final, static, this, super, volatile, transient, synchronized,... giúp nhà phát triển Java làm chủ cú pháp cốt lõi.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: từ khóa Java,từ khóa final,từ khóa static,từ khóa this,từ khóa super,volatile,transient,synchronized
---

# Tóm tắt các từ khóa final, static, this, super

## Từ khóa final

**Từ khóa final có nghĩa là cuối cùng, không thể sửa đổi, ghét nhất sự thay đổi, dùng để修饰 class, method và variable, có các đặc điểm sau:**

1. Class được修饰 bởi final không thể bị kế thừa, tất cả các phương thức thành viên trong final class đều được chỉ định ẩn ngầm là final method;

2. Method được修饰 bởi final không thể bị override (ghi đè);

3. Variable được修饰 bởi final chỉ có thể được gán giá trị một lần. Nếu là biến kiểu dữ liệu nguyên thủy, giá trị của nó sau khi khởi tạo không thể thay đổi; nếu là biến kiểu tham chiếu, sau khi khởi tạo không thể trỏ tới đối tượng khác, nhưng bản thân đối tượng được tham chiếu vẫn có thể thay đổi. Chỉ biến final thỏa mãn điều kiện "hằng số biến" của JLS mới là hằng số ở thời điểm biên dịch (compile-time constant).

Giải thích: Có 2 lý do sử dụng phương thức final:

1. Khóa phương thức lại, tránh bất kỳ lớp kế thừa nào sửa đổi ý nghĩa của nó;
2. Hiệu suất. Trong các phiên bản thực thi Java thời kỳ đầu, phương thức final sẽ được chuyển thành cuộc gọi inline (nội hàm). Tuy nhiên nếu phương thức quá lớn, có thể không thấy bất kỳ sự cải thiện hiệu năng nào từ cuộc gọi inline mang lại (các phiên bản Java hiện tại không còn cần dùng phương thức final cho các tối ưu hóa này nữa).

## Từ khóa static

**Từ khóa static chủ yếu có 4 kịch bản sử dụng sau:**

1. **修饰 biến thành viên và phương thức thành viên:** Thành viên được static修饰 thuộc về class, không thuộc về một đối tượng đơn lẻ nào của class đó, được tất cả các đối tượng trong class dùng chung, có thể và khuyến nghị gọi thông qua tên class. Vị trí lưu trữ cụ thể của biến static thuộc về chi tiết thực thi của JVM; lấy HotSpot từ JDK 8 trở đi làm ví dụ, class metadata nằm trong Metaspace của bộ nhớ local, còn biến static của class nằm trong Java Heap. Cú pháp gọi: `TênClass.tênBiếnStatic`, `TênClass.tênPhươngThứcStatic()`
2. **Khối code static (Static Block):** Khối code static được định nghĩa trong class nhưng nằm ngoài các method, khối code static được thực thi trước khối code non-static (Khối code static —> Khối code non-static —> Constructor). Class đó dù tạo bao nhiêu đối tượng thì khối code static cũng chỉ thực thi 1 lần.
3. **Static Inner Class (Lớp nội bộ tĩnh - static nếu修饰 class thì chỉ修饰 inner class):** Giữa Static Inner Class và Non-static Inner Class tồn tại một điểm khác biệt lớn nhất: Non-static Inner Class sau khi biên dịch xong sẽ lưu trữ ẩn ngầm một tham chiếu trỏ đến Outer Class tạo ra nó, còn Static Inner Class thì không. Không có tham chiếu này đồng nghĩa với việc: 1. Việc tạo nó không cần phụ thuộc vào việc tạo Outer Class. 2. Nó không thể sử dụng bất kỳ biến thành viên và phương thức non-static nào của Outer Class.
4. **Import static (Dùng để import tài nguyên static trong class, tính năng mới từ 1.5):** Cú pháp là: `import static`. Hai từ khóa này đi liền nhau có thể chỉ định import tài nguyên static cụ thể trong một class nào đó, và không cần dùng tên class để gọi thành viên static trong class, có thể dùng trực tiếp biến thành viên static và phương thức thành viên static trong class.

## Từ khóa this

Từ khóa this được dùng để tham chiếu đến instance hiện tại của class. Ví dụ:

```java
class Manager {
    Employees[] employees;
    void manageEmployees() {
        int totalEmp = this.employees.length;
        System.out.println("Total employees: " + totalEmp);
        this.report();
    }
    void report() { }
}
```

Trong ví dụ trên, từ khóa this được dùng ở 2 nơi:

- `this.employees.length`: Truy cập biến của instance hiện tại của class Manager.
- `this.report()`: Gọi phương thức của instance hiện tại của class Manager.

Từ khóa này là tùy chọn, nghĩa là nếu ví dụ trên không dùng từ khóa này thì hành vi vẫn tương tự. Tuy nhiên, sử dụng từ khóa này có thể giúp code dễ đọc và dễ hiểu hơn.

## Từ khóa super

Từ khóa super được dùng để truy cập biến và phương thức của lớp bố (superclass) từ lớp con (subclass). Ví dụ:

```java
public class Super {
    protected int number;
    protected void showNumber() {
        System.out.println("number = " + number);
    }
}
public class Sub extends Super {
    void bar() {
        super.number = 10;
        super.showNumber();
    }
}
```

Trong ví dụ trên, lớp Sub truy cập biến thành viên number của lớp bố và gọi phương thức `showNumber()` của lớp bố Super.

**Các vấn đề cần lưu ý khi sử dụng this và super:**

- Trong constructor khi dùng `super()` gọi các constructor khác trong lớp bố, câu lệnh đó bắt buộc phải nằm ở dòng đầu tiên của constructor, nếu không trình biên dịch sẽ báo lỗi. Ngoài ra, this khi gọi các constructor khác trong cùng class cũng phải đặt ở dòng đầu tiên.
- this, super không thể dùng trong phương thức static.

**Giải thích đơn giản:**

Thành viên được修饰 bởi static thuộc về class, trong ngữ cảnh static không có instance hiện tại, do đó không thể dùng `this`. `super` cũng không phải là một tham chiếu độc lập trỏ đến "đối tượng lớp bố", mà là dạng cú pháp bị hạn chế dùng để truy cập thành viên lớp bố hoặc gọi constructor lớp bố, do đó cũng không thể dùng trong ngữ cảnh static.

## Tài liệu tham khảo

- <https://www.codejava.net/java-core/the-java-language/java-keywords>
- <https://blog.csdn.net/u013393958/article/details/79881037>

# Giải thích chi tiết từ khóa static

## Từ khóa static chủ yếu có 4 kịch bản sử dụng sau

1. 修饰 biến thành viên và phương thức thành viên
2. Khối code static
3. 修饰 class (chỉ có thể修饰 inner class)
4. Import static (dùng để import tài nguyên static trong class, tính năng mới từ 1.5)

### 修饰 biến thành viên và phương thức thành viên (Thường dùng)

Thành viên được修饰 bởi static thuộc về class, không thuộc về một đối tượng đơn lẻ nào của class đó, được tất cả các đối tượng trong class dùng chung, có thể và khuyến nghị gọi thông qua tên class. Vị trí lưu trữ cụ thể của biến static thuộc về chi tiết thực thi của JVM.

Method Area cũng giống như Java Heap, là vùng dữ liệu runtime được dùng chung bởi các Thread. JVM Specification quy định nó lưu trữ thông tin cấu trúc của mỗi class, ví dụ Runtime Constant Pool, dữ liệu field và method, cũng như code của method và constructor. Bố cục lưu trữ cụ thể do bản thực thi JVM quyết định.

Trong HotSpot JDK 7 và các phiên bản cũ hơn, Method Area chủ yếu được thực hiện bởi Permanent Generation (PermGen), tuy nhiên Method Area và PermGen không hoàn toàn tương đương. JDK 8 đã loại bỏ PermGen: Metadata của class được chuyển sang lưu ở Metaspace trong bộ nhớ local, còn String Constant Pool và class static variable,... nằm trong Java Heap.

Cú pháp gọi:

- `TênClass.tênBiếnStatic`
- `TênClass.tênPhươngThứcStatic()`

Nếu biến hoặc phương thức bị修饰 là private thì đại diện thuộc tính hoặc phương thức đó chỉ có thể được truy cập bên trong class chứ không thể truy cập bên ngoài class.

Phương thức test:

```java
public class StaticBean {
    String name;
    // Biến static
    static int age;
    public StaticBean(String name) {
        this.name = name;
    }
    // Phương thức static
    static void sayHello() {
        System.out.println("Hello i am java");
    }
    @Override
    public String toString() {
        return "StaticBean{"+
                "name=" + name + ",age=" + age +
                "}";
    }
}
```

```java
public class StaticDemo {
    public static void main(String[] args) {
        StaticBean staticBean = new StaticBean("1");
        StaticBean staticBean2 = new StaticBean("2");
        StaticBean staticBean3 = new StaticBean("3");
        StaticBean staticBean4 = new StaticBean("4");
        StaticBean.age = 33;
        System.out.println(staticBean + " " + staticBean2 + " " + staticBean3 + " " + staticBean4);
        // StaticBean{name=1,age=33} StaticBean{name=2,age=33} StaticBean{name=3,age=33} StaticBean{name=4,age=33}
        StaticBean.sayHello();// Hello i am java
    }
}
```

### Khối code static

Khối code static được định nghĩa trong class nhưng nằm ngoài các method, khối code static được thực thi trước khối code non-static (Khối code static —> Khối code non-static —> Constructor). Class đó dù tạo bao nhiêu đối tượng thì khối code static cũng chỉ thực thi 1 lần.

Cú pháp khối code static là:

```plain
static {
câu lệnh;
}
```

Trong một class có thể có nhiều khối code static, vị trí có thể đặt tùy ý, nó không nằm trong bất kỳ thân phương thức nào. Khối code static thực thi khi class khởi tạo; nếu có nhiều khối code static, JVM sẽ thực thi lần lượt theo thứ tự xuất hiện của chúng trong class, mỗi khối code chỉ được thực thi 1 lần. Việc loading class có thể diễn ra sớm hơn initialization.

![](https://oss.javaguide.cn/github/javaguide/88531075.jpg)

Khối code static đối với biến static định nghĩa đằng sau nó, có thể gán giá trị, nhưng không thể truy cập.

### Static Inner Class

Giữa Static Inner Class và Non-static Inner Class tồn tại một điểm khác biệt lớn nhất, chúng ta biết Non-static Inner Class sau khi biên dịch xong sẽ lưu trữ ẩn ngầm một tham chiếu trỏ đến Outer Class tạo ra nó, còn Static Inner Class thì không. Không có tham chiếu này đồng nghĩa với việc:

1. Việc tạo nó không cần phụ thuộc vào việc tạo Outer Class.
2. Nó không thể sử dụng bất kỳ biến thành viên và phương thức non-static nào của Outer Class.

Example (Static Inner Class triển khai Singleton Pattern)

```java
public class Singleton {
    // Khai báo private để tránh gọi constructor mặc định tạo đối tượng
    private Singleton() {
    }
   // Khai báo private thể hiện static inner class này chỉ được truy cập trong lớp Singleton này
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

Khi gọi `getUniqueInstance()` và chủ động sử dụng `SingletonHolder.INSTANCE` lần đầu tiên, `SingletonHolder` mới được khởi tạo, lúc này khởi tạo `INSTANCE`. JVM có thể nạp `SingletonHolder` sớm hơn, nhưng không vì vậy mà thực thi khởi tạo static của nó, và có thể đảm bảo class này chỉ khởi tạo 1 lần.

Cách này không chỉ có lợi ích của Lazy Initialization, mà còn được JVM hỗ trợ an toàn đa luồng (Thread-safety).

### Import static

Cú pháp là: `import static`

Hai từ khóa này đi liền nhau có thể chỉ định import tài nguyên static cụ thể trong một class nào đó, và không cần dùng tên class để gọi thành viên static trong class, có thể dùng trực tiếp biến thành viên static và phương thức thành viên static trong class.

```java
 // Import tất cả tài nguyên static trong Math, lúc này có thể dùng trực tiếp phương thức static bên trong mà không cần qua tên class để gọi
 // Nếu chỉ muốn import duy nhất 1 phương thức static, chỉ cần đổi * thành tên phương thức tương ứng là được
import static java.lang.Math.*;// Đổi thành import static java.lang.Math.max; là chỉ định import duy nhất phương thức static max
public class Demo {
  public static void main(String[] args) {
    int max = max(1,2);
    System.out.println(max);
  }
}
```

## Nội dung bổ sung

### Phương thức static và phương thức non-static

Phương thức static thuộc về bản thân class, phương thức non-static thuộc về từng đối tượng sinh ra từ class đó. Nếu thao tác mà phương thức của bạn thực thi không phụ thuộc vào các biến và phương thức riêng của class đó, hãy đặt nó thành static (điều này giúp dung lượng chiếm dụng của chương trình nhỏ hơn). Nêu không, nó nên là non-static.

Example

```java
class Foo {
    int i;
    public Foo(int i) {
       this.i = i;
    }
    public static String method1() {
       return "An example string that doesn't depend on i (an instance variable)";
    }
    public int method2() {
       return this.i + 1;  // Depends on i
    }
}
```

Bạn có thể gọi phương thức static như thế này: `Foo.method1()`. Nếu bạn thử dùng cách này để gọi method2 sẽ thất bại. Nhưng làm thế này thì được:

```java
Foo bar = new Foo(1);
bar.method2();
```

Tóm tắt:

- Khi gọi phương thức static ở bên ngoài, có thể dùng cách "TênClass.tênPhươngThức" hoặc dùng cách "TênĐốiTượng.tênPhươngThức". Còn phương thức instance chỉ có cách phía sau. Nghĩa là, gọi phương thức static không cần tạo đối tượng.
- Phương thức static khi truy cập thành viên trong cùng class chỉ cho phép truy cập thành viên static (tức biến thành viên static và phương thức static), không cho phép truy cập biến thành viên instance và phương thức instance; phương thức instance thì không có hạn chế này.

### Khối code static `static{}` và Khối code non-static `{}` (Instance Initialization Block)

Giống nhau: Đều có thể định nghĩa nhiều cái trong class; nhiều khối code static trong cùng class được đưa vào quá trình khởi tạo class theo thứ tự văn bản, nhiều khối code khởi tạo instance được đưa vào quá trình khởi tạo instance theo thứ tự văn bản.

Khác nhau: Khối code static thực thi 1 lần khi class khởi tạo, thời điểm kích hoạt không nhất thiết phải là lần `new` đầu tiên; khối code non-static (instance initialization block) thực thi mỗi khi khởi tạo một instance mới, và cùng với initializer của instance field được đưa vào quá trình khởi tạo instance theo thứ tự văn bản. Chúng chạy sau khi constructor của superclass trả về và trước khi các câu lệnh tiếp theo của constructor thực thi; nếu constructor thông qua `this(...)` ủy quyền cho constructor khác trong cùng class, thì quá trình khởi tạo này do constructor thực sự gọi constructor superclass trong chuỗi ủy quyền hoàn thành. Khối code trần trong phương thức thông thường chỉ là khối code cục bộ (local block), không phải khối code khởi tạo instance.

> **🐛 Sửa lỗi (Xem: [issue #677](https://github.com/Snailclimb/JavaGuide/issues/677))**: Khối code static có thể thực thi khi new đối tượng lần đầu tiên, nhưng không nhất thiết chỉ thực thi khi new lần đầu. Ví dụ khi tạo Class object qua `Class.forName("ClassDemo")` cũng sẽ thực thi, tức new hoặc `Class.forName("ClassDemo")` đều sẽ thực thi khối code static.
> Thông thường, nếu có một số code như các biến hoặc đối tượng thường dùng nhất của dự án bắt buộc phải thực thi khi khởi động dự án thì cần dùng khối code static, loại code này tự động thực thi. Nếu chúng ta muốn thiết kế không cần tạo đối tượng vẫn có thể gọi phương thức trong class, ví dụ các class `Arrays`, `Character`, `String`,... thì cần dùng phương thức static. Sự khác biệt giữa 2 cái là khối code static tự động thực thi còn phương thức static được gọi mới thực thi.

Example:

```java
public class Test {
    public Test() {
        System.out.print("Constructor mặc định! --");
    }
    // Khối code non-static
    {
        System.out.print("Khối code non-static! --");
    }
    // Khối code static
    static {
        System.out.print("Khối code static! --");
    }
    private static void test() {
        System.out.print("Nội dung trong phương thức static! --");
        {
            System.out.print("Khối code trong phương thức static! --");
        }
    }
    public static void main(String[] args) {
        Test test = new Test();
        Test.test();// Khối code static! --Nội dung trong phương thức static! --Khối code trong phương thức static! --
    }
}
```

Đoạn code trên in ra:

```plain
Khối code static! --Khối code non-static! --Constructor mặc định! --Nội dung trong phương thức static! --Khối code trong phương thức static! --
```

Khi chỉ thực thi `Test.test();` in ra:

```plain
Khối code static! --Nội dung trong phương thức static! --Khối code trong phương thức static! --
```

Khi chỉ thực thi `Test test = new Test();` in ra:

```plain
Khối code static! --Khối code non-static! --Constructor mặc định! --
```

Sự khác biệt giữa khối code non-static và constructor là: Khối code non-static tiến hành khởi tạo thống nhất cho tất cả các đối tượng, còn constructor tiến hành khởi tạo cho đối tượng tương ứng, vì constructor có thể có nhiều cái, chạy constructor nào thì sẽ tạo ra đối tượng như thế đó, nhưng dù tạo đối tượng nào thì cũng đều thực thi khối code khởi tạo giống nhau trước. Nghĩa là, trong khối code khởi tạo định nghĩa nội dung khởi tạo mang tính chung của các đối tượng khác nhau.

### Tài liệu tham khảo

- <https://blog.csdn.net/chen13579867831/article/details/78995480>
- <https://www.cnblogs.com/chenssy/p/3388487.html>
- <https://www.cnblogs.com/Qian123/p/5713440.html>

<!-- @include: @article-footer.snippet.md -->
