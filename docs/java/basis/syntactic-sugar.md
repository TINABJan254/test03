---
title: Giải thích chi tiết Syntactic Sugar trong Java
description: Phân tích sâu nguyên lý Cú pháp đường (Syntactic Sugar) trong Java: giải thích chi tiết cơ chế thực hiện lúc biên dịch của Autoboxing/Unboxing, Type Erasure, Enhanced for, Varargs, Enum, Lambda,... tránh các bẫy khi sử dụng.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Java syntactic sugar,cú pháp đường,autoboxing unboxing,type erasure,vòng lặp for nâng cao,varargs,enum,inner class,Lambda expression,nguyên lý cú pháp đường
---

> Tác giả: Hollis
>
> Nguồn bài gốc: <https://mp.weixin.qq.com/s/o4XdEMq1DL-nBS-f8Za5Aw>

Syntactic Sugar (Cú pháp đường) là một điểm kiến thức thường được hỏi trong các buổi phỏng vấn Java của các công ty lớn.

Bài viết này đứng từ góc độ nguyên lý biên dịch Java, đi sâu vào bytecode và file `.class`, bóc tách từng lớp để hiểu rõ nguyên lý và cách dùng của Syntactic Sugar trong Java, giúp mọi người vừa biết cách sử dụng Syntactic Sugar vừa hiểu được nguyên lý đằng sau chúng.

## Syntactic Sugar là gì?

**Syntactic Sugar (Cú pháp đường / Đường cú pháp)** còn gọi là đường bọc đường, là một thuật ngữ do nhà khoa học máy tính người Anh Peter J. Landin phát minh, chỉ việc thêm một cú pháp nào đó vào ngôn ngữ máy tính, cú pháp này không ảnh hưởng đến tính năng của ngôn ngữ, nhưng giúp lập trình viên sử dụng tiện lợi hơn. Nói một cách ngắn gọn, Syntactic Sugar giúp chương trình ngắn gọn hơn, có tính đọc hiểu cao hơn.

![](https://oss.javaguide.cn/github/javaguide/java/basis/syntactic-sugar/image-20220818175953954.png)

> Thú vị là trong lĩnh vực lập trình, ngoài Syntactic Sugar ra còn có các khái niệm Syntactic Salt (Muối cú pháp) và Syntactic Saccharin (Đường hóa học cú pháp), do thời lượng có hạn nên ở đây không mở rộng thêm.

Hầu như tất cả các ngôn ngữ lập trình chúng ta quen thuộc đều có Syntactic Sugar. Tác giả cho rằng, số lượng Syntactic Sugar nhiều hay ít là một trong những tiêu chuẩn đánh giá một ngôn ngữ có đủ "xịn" hay không. Nhiều người nói Java là một "ngôn ngữ ít đường", thực ra từ Java 7 trở đi trên khía cạnh ngôn ngữ Java liên tục thêm vào các loại đường, chủ yếu được nghiên cứu phát triển dưới dự án "Project Coin". Mặc dù hiện tại vẫn có người cho rằng Java là ít đường, nhưng tương lai sẽ tiếp tục phát triển theo hướng "nhiều đường".

## Trong Java có những Syntactic Sugar phổ biến nào?

Như đã đề cập trước đó, sự tồn tại của Syntactic Sugar chủ yếu để thuận tiện cho nhân viên phát triển sử dụng. Nhưng thực ra, **Java Virtual Machine (JVM) hoàn toàn không hỗ trợ những Syntactic Sugar này. Những Syntactic Sugar này ở giai đoạn biên dịch sẽ được hoàn nguyên về các cấu trúc cú pháp cơ bản đơn giản, quá trình này gọi là Desugaring (Giải cú pháp đường).**

Nói đến biên dịch, chắc chắn mọi người đều biết trong ngôn ngữ Java, lệnh `javac` có thể biên dịch các file nguồn có đuôi `.java` thành bytecode có đuôi `.class` có thể chạy trên JVM. Nếu bạn xem mã nguồn của `com.sun.tools.javac.main.JavaCompiler`, bạn sẽ phát hiện trong `compile()` có một bước chính là gọi `desugar()`, phương thức này chịu trách nhiệm thực thi việc giải cú pháp đường.

Các Syntactic Sugar phổ biến nhất trong Java chủ yếu có Generics, Varargs (tham số biến đổi), Conditional Compilation (biên dịch theo điều kiện), Autoboxing/Unboxing (tự động đóng/mở gói), Inner Class (lớp nội bộ),... Bài viết này chủ yếu phân tích nguyên lý đằng sau các Syntactic Sugar này. Từng bước bóc đi lớp vỏ đường để xem bản chất của nó.

Ở đây chúng ta sẽ dùng tới Decompiler (trình giải biên dịch), bạn có thể tiến hành decompile file Class online thông qua [Decompilers online](http://www.javadecompilers.com/).

### switch hỗ trợ String và Enum

Như đã đề cập ở trước, từ Java 7 trở đi, Syntactic Sugar trong ngôn ngữ Java ngày càng phong phú, trong đó có một điểm khá quan trọng là `switch` trong Java 7 bắt đầu hỗ trợ `String`.

Trước khi bắt đầu xin phổ biến kiến thức trước, `switch` trong Java thời kỳ đầu hỗ trợ `byte`, `short`, `char`, `int` và các kiểu đóng gói tương ứng, không hỗ trợ `boolean`, `long`, `float`, `double`. `char` biểu thị code unit UTF-16 chứ không phải kiểu ASCII. Sau đó, Java bổ sung thêm hỗ trợ cho kiểu tham chiếu như Enum và `String`.

Vậy tiếp theo hãy xem hỗ trợ của `switch` đối với `String`, có đoạn code sau:

```java
public class switchDemoString {
    public static void main(String[] args) {
        String str = "world";
        switch (str) {
        case "hello":
            System.out.println("hello");
            break;
        case "world":
            System.out.println("world");
            break;
        default:
            break;
        }
    }
}
```

Nội dung sau khi decompile như sau:

```java
public class switchDemoString
{
    public switchDemoString()
    {
    }
    public static void main(String args[])
    {
        String str = "world";
        String s;
        switch((s = str).hashCode())
        {
        default:
            break;
        case 99162322:
            if(s.equals("hello"))
                System.out.println("hello");
            break;
        case 113318802:
            if(s.equals("world"))
                System.out.println("world");
            break;
        }
    }
}
```

Từ đoạn code được sinh ra và decompile bởi phiên bản `javac` cụ thể này có thể thấy, **chuỗi trong `switch` đã được chuyển đổi thành phân nhánh `hashCode()` và kiểm tra `equals()`.** Đây là chiến lược thực thi của trình biên dịch chứ không phải dạng bytecode quy định bắt buộc của JLS.

Nhìn kỹ có thể phát hiện, thứ thực hiện `switch` thực chất là giá trị hash, sau đó dùng phương thức `equals` để so sánh kiểm tra an toàn, việc kiểm tra này là bắt buộc vì hash có thể xảy ra xung đột (collision). Do đó hiệu năng của nó không bằng việc dùng Enum trong `switch` hay dùng hằng số số nguyên thuần túy, nhưng cũng không phải quá kém.

### Generics (Kiểu chung)

Chúng ta đều biết nhiều ngôn ngữ đều hỗ trợ Generics, nhưng nhiều người không biết là các trình biên dịch khác nhau có cách xử lý Generics khác nhau, thông thường một trình biên dịch xử lý Generics có 2 cách: `Code specialization` và `Code sharing`. C++ và C# sử dụng cơ chế `Code specialization`, còn Java sử dụng cơ chế `Code sharing`.

> Cách `Code sharing` tạo ra dạng biểu diễn bytecode duy nhất cho mỗi kiểu Generics, và ánh xạ tất cả các instance của kiểu Generics đó lên dạng biểu diễn bytecode duy nhất này. Việc ánh xạ nhiều instance kiểu Generics lên dạng biểu diễn bytecode duy nhất được thực hiện thông qua Type Erasure (Xóa kiểu).

Nghĩa là **đối với JVM, nó hoàn toàn không nhận biết cú pháp như `Map<String, String> map`. Cần phải giải cú pháp đường ở giai đoạn biên dịch thông qua Type Erasure.**

Quá trình chính của Type Erasure như sau: 1. Thay thế tất cả các tham số Generics bằng kiểu ranh giới trái nhất (kiểu cha cao nhất) của nó. 2. Loại bỏ tất cả các tham số kiểu.

Đoạn code sau:

```java
Map<String, String> map = new HashMap<String, String>();
map.put("name", "hollis");
map.put("wechat", "Hollis");
map.put("blog", "www.hollischuang.com");
```

Sau khi giải cú pháp đường sẽ biến thành:

```java
Map map = new HashMap();
map.put("name", "hollis");
map.put("wechat", "Hollis");
map.put("blog", "www.hollischuang.com");
```

Đoạn code sau:

```java
public static <A extends Comparable<A>> A max(Collection<A> xs) {
    Iterator<A> xi = xs.iterator();
    A w = xi.next();
    while (xi.hasNext()) {
        A x = xi.next();
        if (w.compareTo(x) < 0)
            w = x;
    }
    return w;
}
```

Sau khi xóa kiểu sẽ biến thành:

```java
 public static Comparable max(Collection xs){
    Iterator xi = xs.iterator();
    Comparable w = (Comparable)xi.next();
    while(xi.hasNext())
    {
        Comparable x = (Comparable)xi.next();
        if(w.compareTo(x) < 0)
            w = x;
    }
    return w;
}
```

**Trong Virtual Machine không có Generics, chỉ có class thông thường và method thông thường, tất cả tham số kiểu của Generic Class lúc biên dịch đều bị xóa, Generic Class không có đối tượng `Class` độc lập của riêng mình. Ví dụ không hề tồn tại `List<String>.class` hay `List<Integer>.class`, mà chỉ có `List.class`.**

### Autoboxing và Unboxing (Tự động đóng/mở gói)

Autoboxing là việc Java tự động chuyển đổi giá trị kiểu nguyên thủy thành đối tượng tương ứng, ví dụ chuyển biến int thành đối tượng Integer, quá trình này gọi là Boxing (Đóng gói), ngược lại chuyển đối tượng Integer thành giá trị kiểu int gọi là Unboxing (Mở gói). Vì việc đóng gói và mở gói ở đây diễn ra tự động chứ không phải chuyển đổi thủ công nên gọi là Autoboxing và Unboxing. Các kiểu nguyên thủy byte, short, char, int, long, float, double và boolean có các Wrapper Class tương ứng là Byte, Short, Character, Integer, Long, Float, Double, Boolean.

Xem đoạn code Autoboxing trước:

```java
 public static void main(String[] args) {
    int i = 10;
    Integer n = i;
}
```

Đoạn code sau khi decompile:

```java
public static void main(String args[])
{
    int i = 10;
    Integer n = Integer.valueOf(i);
}
```

Xem tiếp đoạn code Unboxing:

```java
public static void main(String[] args) {

    Integer i = 10;
    int n = i;
}
```

Đoạn code sau khi decompile:

```java
public static void main(String args[])
{
    Integer i = Integer.valueOf(10);
    int n = i.intValue();
}
```

Từ nội dung decompile có thể thấy, khi đóng gói nó tự động gọi phương thức `valueOf(int)` của `Integer`. Còn khi mở gói nó tự động gọi phương thức `intValue` của `Integer`.

Do đó, **quá trình đóng gói được thực hiện thông qua việc gọi phương thức valueOf của Wrapper Class, còn quá trình mở gói được thực hiện thông qua việc gọi phương thức xxxValue của Wrapper Class.**

### Varargs (Tham số độ dài biến đổi)

Varargs (`variable arguments`) là một tính năng được đưa vào từ Java 1.5. Nó cho phép một phương thức nhận số lượng giá trị tùy ý làm tham số.

Xem đoạn code Varargs dưới đây, trong đó phương thức `print` nhận tham số biến đổi:

```java
public static void main(String[] args)
    {
        print("Holis", "Công khai:Hollis", "Blog:www.hollischuang.com", "QQ:907607222");
    }

public static void print(String... strs)
{
    for (int i = 0; i < strs.length; i++)
    {
        System.out.println(strs[i]);
    }
}
```

Đoạn code sau khi decompile:

```java
 public static void main(String args[])
{
    print(new String[] {
        "Holis", "\u516C\u4F17\u53F7:Hollis", "\u535A\u5BA2\uFF1Awww.hollischuang.com", "QQ\uFF1A907607222"
    });
}

public static transient void print(String strs[])
{
    for(int i = 0; i < strs.length; i++)
        System.out.println(strs[i]);

}
```

Từ code sau khi decompile có thể thấy, khi tham số biến đổi được sử dụng, đầu tiên nó sẽ tạo một mảng có độ dài chính là số lượng tham số thực tế truyền vào khi gọi phương thức đó, sau đó đặt tất cả các giá trị tham số vào mảng đó, rồi truyền mảng này làm tham số vào phương thức được gọi. (Lưu ý: `transient` chỉ có ý nghĩa khi修饰 biến thành viên, ở đây "修饰 phương thức" là do trong javassist dùng cùng giá trị số để biểu thị `transient` và `vararg`, xem [tại đây](https://github.com/jboss-javassist/javassist/blob/7302b8b0a09f04d344a26ebe57f29f3db43f2a3e/src/main/javassist/bytecode/AccessFlag.java#L32)).

### Enum (Kiểu liệt kê)

Java SE 5 cung cấp một kiểu dữ liệu mới — Enum của Java, từ khóa `enum` có thể tạo một tập hợp hữu hạn các giá trị có tên thành một kiểu dữ liệu mới, và các giá trị có tên này có thể dùng làm các component chương trình thông thường, đây là một tính năng rất hữu ích.

Muốn xem mã nguồn thì đầu tiên phải có một class, vậy kiểu Enum rốt cuộc là class gì? Có phải là `enum` không? Đáp án hiển nhiên không phải, `enum` cũng giống như `class` chỉ là một từ khóa chứ không phải một class. Vậy Enum do class nào duy trì? Chúng ta viết đơn giản một Enum:

```java
public enum t {
    SPRING,SUMMER;
}
```

Sau đó chúng ta decompile xem đoạn code này rốt cuộc được thực thi thế nào, nội dung code decompile như sau:

```java
// Tên nhị phân của enum vẫn là t, nhận dạng Java có phân biệt chữ hoa chữ thường
public final class t extends Enum
{
    private t(String s, int i)
    {
        super(s, i);
    }
    public static t[] values()
    {
        t at[];
        int i;
        t at1[];
        System.arraycopy(at = ENUM$VALUES, 0, at1 = new t[i = at.length], 0, i);
        return at1;
    }

    public static t valueOf(String s)
    {
        return (t) Enum.valueOf(t.class, s);
    }

    public static final t SPRING;
    public static final t SUMMER;
    private static final t ENUM$VALUES[];
    static
    {
        SPRING = new t("SPRING", 0);
        SUMMER = new t("SUMMER", 1);
        ENUM$VALUES = (new t[] {
            SPRING, SUMMER
        });
    }
}
```

Thông qua code sau decompile chúng ta có thể thấy `public final class t extends Enum`, cho thấy class này kế thừa lớp `Enum`; enum hiện tại không chứa thân lớp riêng cho hằng số nên nó ẩn ngầm là `final`.

**Enum class được định nghĩa bằng `enum` sẽ trực tiếp kế thừa `Enum`, do đó không thể kế thừa rõ ràng các class khác, cũng không thể bị class thông thường kế thừa. Enum class không có thân lớp riêng cho hằng số thì ẩn ngầm là `final`; chỉ cần có hằng số enum khai báo thân lớp riêng thì enum class sẽ ẩn ngầm là `sealed`, các thân lớp riêng này tương ứng với các anonymous subclass được cấp phép của nó.**

### Inner Class (Lớp nội bộ)

Inner Class còn gọi là Nested Class (lớp lồng nhau), có thể hiểu Inner Class là một thành viên thông thường của Outer Class.

**Inner Class sở dĩ cũng là Syntactic Sugar vì nó chỉ là một khái niệm ở thời điểm biên dịch, trong `outer.java` định nghĩa một inner class `inner`, một khi biên dịch thành công sẽ sinh ra 2 file `.class` hoàn toàn khác nhau là `outer.class` và `outer$inner.class`. Tuy nhiên, JLS nghiêm cấm nested class có cùng simple name với bất kỳ outer class hoặc interface nào chứa nó.**

```java
public class OuterClass {
    private String userName;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public static void main(String[] args) {

    }

    class InnerClass{
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

Đoạn code trên sau khi biên dịch sẽ sinh ra 2 file class: `OuterClass$InnerClass.class`, `OuterClass.class`. Khi chúng ta thử decompile file `OuterClass.class`, dòng lệnh sẽ in ra nội dung: `Parsing OuterClass.class...Parsing inner class OuterClass$InnerClass.class... Generating OuterClass.jad`. Nó sẽ decompile cả 2 file rồi cùng sinh ra một file `OuterClass.jad`. Nội dung file như sau:

```java
public class OuterClass
{
    class InnerClass
    {
        public String getName()
        {
            return name;
        }
        public void setName(String name)
        {
            this.name = name;
        }
        private String name;
        final OuterClass this$0;

        InnerClass()
        {
            this.this$0 = OuterClass.this;
            super();
        }
    }

    public OuterClass()
    {
    }
    public String getUserName()
    {
        return userName;
    }
    public void setUserName(String userName){
        this.userName = userName;
    }
    public static void main(String args1[])
    {
    }
    private String userName;
}
```

**Tại sao Inner Class lại có thể sử dụng thuộc tính private của Outer Class**:

Chúng ta thêm một phương thức vào InnerClass để in ra thuộc tính userName của Outer Class

```java
// Bỏ qua các thuộc tính khác
public class OuterClass {
    private String userName;
    ......
    class InnerClass{
    ......
        public void printOut(){
            System.out.println("Username from OuterClass:" + userName);
        }
    }
}

// Lúc này, dùng lệnh javap -p decompile OuterClass thu được kết quả:
public class OuterClass {
    private String userName;
    ......
    static String access$000(OuterClass);
}
// Lúc này, kết quả decompile của InnerClass:
class OuterClass$InnerClass {
    final OuterClass this$0;
    ......
    public void printOut();
}
```

Thực tế sau khi biên dịch xong, bên trong instance inner thường sẽ có tham chiếu `this$0` trỏ đến instance outer. Trong các file class do JDK 10 và các phiên bản cũ hơn sinh ra, trình biên dịch thường thông qua phương thức truy cập tổng hợp dạng `access$000` để thực hiện truy cập thành viên private giữa các nested class, do đó phương thức `printOut()` sau khi decompile đại thể như sau. Từ JDK 11 trở đi đưa vào kiểm soát truy cập dựa trên Nest (Nest-based Access Control), các class trong cùng một nest có thể truy cập trực tiếp thành viên private của nhau, thông thường không cần đến loại phương thức truy cập tổng hợp này nữa:

```java
public void printOut() {
    System.out.println("Username from OuterClass:" + OuterClass.access$000(this.this$0));
}
```

Bổ sung:

1. Trong đầu ra `javac` điển hình của JDK 10 và các phiên bản cũ hơn, Anonymous Inner Class, Local Inner Class, Static Inner Class cũng có thể thông qua phương thức truy cập tổng hợp để lấy thuộc tính private; từ JDK 11 trở đi thông thường dùng nest-based access control.
2. Static Inner Class không có tham chiếu `this$0`.
3. Anonymous Inner Class, Local Inner Class sử dụng biến cục bộ bằng cách copy, biến đó sau khi khởi tạo thì không thể sửa đổi. Dưới đây là một ví dụ:

```java
public class OuterClass {
    private String userName;

    public void test(){
        // Ở đây i sau khi khởi tạo thành 1 thì không thể sửa đổi nữa
        int i=1;
        class Inner{
            public void printName(){
                System.out.println(userName);
                System.out.println(i);
            }
        }
    }
}
```

Sau khi decompile:

```java
// Kết quả decompile Inner bằng lệnh javap
// i được copy vào trong inner class và là final
class OuterClass$1Inner {
  final int val$i;
  final OuterClass this$0;
  OuterClass$1Inner();
  public void printName();
}
```

### Conditional Compilation (Biên dịch theo điều kiện)

Thông thường mỗi dòng code trong chương trình đều tham gia biên dịch. Nhưng đôi khi xuất phát từ việc tối ưu code chương trình, hy vọng chỉ biên dịch một phần nội dung trong đó, lúc này cần thêm điều kiện vào chương trình để trình biên dịch chỉ biên dịch phần code thỏa mãn điều kiện, bỏ qua phần code không thỏa mãn điều kiện, đó chính là Conditional Compilation.

Như trong C hoặc C++, có thể thông qua câu lệnh tiền xử lý để thực hiện Conditional Compilation. Thực ra trong Java cũng có thể thực hiện Conditional Compilation. Chúng ta hãy xem một đoạn code trước:

```java
public class ConditionalCompilation {
    public static void main(String[] args) {
        final boolean DEBUG = true;
        if(DEBUG) {
            System.out.println("Hello, DEBUG!");
        }

        final boolean ONLINE = false;

        if(ONLINE){
            System.out.println("Hello, ONLINE!");
        }
    }
}
```

Đoạn code sau khi decompile:

```java
public class ConditionalCompilation
{

    public ConditionalCompilation()
    {
    }

    public static void main(String args[])
    {
        boolean DEBUG = true;
        System.out.println("Hello, DEBUG!");
        boolean ONLINE = false;
    }
}
```

Đầu tiên, chúng ta phát hiện trong code sau khi decompile không có `System.out.println("Hello, ONLINE!");`, đây thực chất chính là Conditional Compilation. Khi `if(ONLINE)` là false, trình biên dịch không hề biên dịch phần code bên trong đó.

Do đó, **Conditional Compilation trong cú pháp Java được thực hiện thông qua câu lệnh if với điều kiện đánh giá là hằng số. Nguyên lý của nó cũng là Syntactic Sugar của ngôn ngữ Java. Căn cứ vào tính đúng sai của điều kiện if, trình biên dịch trực tiếp loại bỏ khối code thuộc phân nhánh false. Conditional Compilation thực hiện theo cách này bắt buộc phải thực hiện trong thân phương thức, chứ không thể thực hiện trên toàn bộ cấu trúc của class Java hay thuộc tính của class, điều này so với Conditional Compilation của C/C++ đúng là có hạn chế hơn. Lúc khởi đầu thiết kế ngôn ngữ Java không đưa tính năng Conditional Compilation vào, dù có hạn chế nhưng có vẫn tốt hơn không.**

### Assertion (Khẳng định - assert)

Trong Java, từ khóa `assert` được đưa vào từ Java SE 1.4, để tránh xung đột gây lỗi với code Java phiên bản cũ có dùng từ khóa `assert`, Java khi thực thi mặc định không bật kiểm tra assertion (lúc này tất cả câu lệnh assertion đều bị bỏ qua!), nếu muốn bật kiểm tra assertion thì cần dùng cờ `-enableassertions` hoặc `-ea` để bật.

Xem đoạn code chứa assertion:

```java
public class AssertTest {
    public static void main(String args[]) {
        int a = 1;
        int b = 1;
        assert a == b;
        System.out.println("Công khai: Hollis");
        assert a != b : "Hollis";
        System.out.println("Blog: www.hollischuang.com");
    }
}
```

Code sau khi decompile như sau:

```java
public class AssertTest {
   public AssertTest()
    {
    }
    public static void main(String args[])
{
    int a = 1;
    int b = 1;
    if(!$assertionsDisabled && a != b)
        throw new AssertionError();
    System.out.println("\u516C\u4F17\u53F7\uFF1AHollis");
    if(!$assertionsDisabled && a == b)
    {
        throw new AssertionError("Hollis");
    } else
    {
        System.out.println("\u535A\u5BA2\uFF1Awww.hollischuang.com");
        return;
    }
}

static final boolean $assertionsDisabled = !com/hollis/suguar/AssertTest.desiredAssertionStatus();

}
```

Rõ ràng code sau khi decompile phức tạp hơn nhiều so với code của chúng ta. Do đó dùng Syntactic Sugar assert giúp chúng ta tiết kiệm rất nhiều code. **Thực ra bản thực thi bên dưới của assertion chính là câu lệnh if, nếu kết quả assertion là true thì không làm gì, chương trình tiếp tục chạy; nếu kết quả assertion là false thì chương trình throw `AssertionError` để ngắt việc chạy chương trình.** Cờ `-enableassertions` sẽ thiết lập giá trị cho field `$assertionsDisabled`.

### Numeric Literal (Hằng số số)

Trong Java 7, hằng số số (Numeric Literal), dù là số nguyên hay số thực, đều cho phép chèn số lượng dấu gạch dưới `_` tùy ý giữa các chữ số. Những dấu gạch dưới này không ảnh hưởng đến giá trị của hằng số, mục đích là để thuận tiện cho việc đọc hiểu.

Ví dụ:

```java
public class Test {
    public static void main(String... args) {
        int i = 10_000;
        System.out.println(i);
    }
}
```

Sau khi decompile:

```java
public class Test
{
  public static void main(String[] args)
  {
    int i = 10000;
    System.out.println(i);
  }
}
```

Trong kết quả decompile không thấy `_`, vì nó chỉ là một phần cú pháp literal trong mã nguồn, không ảnh hưởng đến giá trị số và cũng không được ghi vào file class. **Trình biên dịch bắt buộc phải nhận biết và validate vị trí của `_`, sau đó ghi giá trị của hằng số vào bytecode.**

### for-each (Vòng lặp for nâng cao)

Vòng lặp for nâng cao (`for-each`) tin rằng mọi người không còn xa lạ gì, trong phát triển hàng ngày thường xuyên sử dụng, nó giúp viết ít code hơn nhiều so với vòng lặp for thông thường, vậy đằng sau Syntactic Sugar này được thực hiện thế nào?

```java
public static void main(String... args) {
    String[] strs = {"Hollis", "Công khai: Hollis", "Blog: www.hollischuang.com"};
    for (String s : strs) {
        System.out.println(s);
    }
    List<String> strList = ImmutableList.of("Hollis", "Công khai: Hollis", "Blog: www.hollischuang.com");
    for (String s : strList) {
        System.out.println(s);
    }
}
```

Code sau khi decompile như sau:

```java
public static transient void main(String args[])
{
    String strs[] = {
        "Hollis", "\u516C\u4F17\u53F7\uFF1AHollis", "\u535A\u5BA2\uFF1Awww.hollischuang.com"
    };
    String args1[] = strs;
    int i = args1.length;
    for(int j = 0; j < i; j++)
    {
        String s = args1[j];
        System.out.println(s);
    }

    List strList = ImmutableList.of("Hollis", "\u516C\u4F17\u53F7\uFF1AHollis", "\u535A\u5BA2\uFF1Awww.hollischuang.com");
    String s;
    for(Iterator iterator = strList.iterator(); iterator.hasNext(); System.out.println(s))
        s = (String)iterator.next();

}
```

Code rất đơn giản, **nguyên lý thực hiện của for-each thực chất là sử dụng vòng lặp for thông thường và Iterator.**

### try-with-resources

Trong Java, đối với các tài nguyên đắt đỏ như IO stream thao tác file, kết nối database,... sau khi dùng xong bắt buộc phải đóng kịp thời qua phương thức close, nếu không tài nguyên sẽ luôn ở trạng thái mở, có thể dẫn tới các vấn đề như Memory Leak (rò rỉ bộ nhớ).

Cách đóng tài nguyên phổ biến là giải phóng trong khối `finally`, tức gọi phương thức `close`. Ví dụ chúng ta thường viết code như sau:

```java
public static void main(String[] args) {
    BufferedReader br = null;
    try {
        String line;
        br = new BufferedReader(new FileReader("d:\\hollischuang.xml"));
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    } catch (IOException e) {
        // handle exception
    } finally {
        try {
            if (br != null) {
                br.close();
            }
        } catch (IOException ex) {
            // handle exception
        }
    }
}
```

Từ Java 7 trở đi, JDK cung cấp một cách tốt hơn để đóng tài nguyên, sử dụng câu lệnh `try-with-resources`, viết lại đoạn code trên như sau:

```java
public static void main(String... args) {
    try (BufferedReader br = new BufferedReader(new FileReader("d:\\ hollischuang.xml"))) {
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    } catch (IOException e) {
        // handle exception
    }
}
```

Xem này, đây đúng là tin mừng lớn, mặc dù trước đây tôi thường dùng `IOUtils` để đóng stream chứ không viết nhiều code trong `finally`, nhưng Syntactic Sugar mới này nhìn có vẻ thanh lịch hơn nhiều. Hãy xem đằng sau nó:

```java
public static transient void main(String args[])
    {
        BufferedReader br;
        Throwable throwable;
        br = new BufferedReader(new FileReader("d:\\ hollischuang.xml"));
        throwable = null;
        String line;
        try
        {
            while((line = br.readLine()) != null)
                System.out.println(line);
        }
        catch(Throwable throwable2)
        {
            throwable = throwable2;
            throw throwable2;
        }
        finally
        {
            if(br != null)
                if(throwable != null)
                    try
                    {
                        br.close();
                    }
                    catch(Throwable throwable1)
                    {
                        throwable.addSuppressed(throwable1);
                    }
                else
                    br.close();
        }
    }
}
```

**Thực ra nguyên lý đằng sau rất đơn giản, những thao tác đóng tài nguyên mà chúng ta không làm thì trình biên dịch đã làm giúp chúng ta. Do đó một lần nữa chứng minh tác dụng của Syntactic Sugar là thuận tiện cho lập trình viên sử dụng, nhưng cuối cùng vẫn phải chuyển thành ngôn ngữ mà trình biên dịch nhận biết.**

### Biểu thức Lambda (Lambda Expression)

Về biểu thức Lambda, có người có thể thắc mắc vì trên mạng có người nói nó không phải là Syntactic Sugar. Thực ra tôi muốn đính chính lại cách nói này. **Lambda Expression không phải là Syntactic Sugar của Anonymous Inner Class, nhưng bản thân nó cũng là một Syntactic Sugar. Cách thực hiện của nó phụ thuộc vào một số lambda API bên dưới do JVM cung cấp.**

Xem một biểu thức Lambda đơn giản trước. Duyệt một list:

```java
public static void main(String... args) {
    List<String> strList = ImmutableList.of("Hollis", "Công khai: Hollis", "Blog: www.hollischuang.com");

    strList.forEach( s -> { System.out.println(s); } );
}
```

Tại sao nói nó không phải là Syntactic Sugar của inner class? Ở phần trước nói về inner class chúng ta đã biết inner class sau khi biên dịch sẽ có 2 file class, nhưng class chứa Lambda Expression sau khi biên dịch chỉ có 1 file.

Code sau khi decompile như sau:

```java
public static /* varargs */ void main(String ... args) {
    ImmutableList strList = ImmutableList.of((Object)"Hollis", (Object)"\u516c\u4f17\u53f7\uff1aHollis", (Object)"\u535a\u5ba2\uff1awww.hollischuang.com");
    strList.forEach((Consumer<String>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)V, lambda$main$0(java.lang.String ), (Ljava/lang/String;)V)());
}

private static /* synthetic */ void lambda$main$0(String s) {
    System.out.println(s);
}
```

Có thể thấy trong phương thức `forEach` thực chất gọi phương thức `java.lang.invoke.LambdaMetafactory#metafactory`, tham số thứ 4 `implMethod` của phương thức này chỉ định bản thực thi phương thức. Có thể thấy ở đây thực chất gọi phương thức `lambda$main$0` để xuất kết quả.

Xem tiếp một ví dụ phức tạp hơn một chút, lọc List trước rồi mới xuất kết quả:

```java
public static void main(String... args) {
    List<String> strList = ImmutableList.of("Hollis", "Công khai: Hollis", "Blog: www.hollischuang.com");

    List HollisList = strList.stream().filter(string -> string.contains("Hollis")).collect(Collectors.toList());

    HollisList.forEach( s -> { System.out.println(s); } );
}
```

Code sau khi decompile như sau:

```java
public static /* varargs */ void main(String ... args) {
    ImmutableList strList = ImmutableList.of((Object)"Hollis", (Object)"\u516c\u4f17\u53f7\uff1aHollis", (Object)"\u535a\u5ba2\uff1awww.hollischuang.com");
    List<Object> HollisList = strList.stream().filter((Predicate<String>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)Z, lambda$main$0(java.lang.String ), (Ljava/lang/String;)Z)()).collect(Collectors.toList());
    HollisList.forEach((Consumer<Object>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)V, lambda$main$1(java.lang.Object ), (Ljava/lang/Object;)V)());
}

private static /* synthetic */ void lambda$main$1(Object s) {
    System.out.println(s);
}

private static /* synthetic */ boolean lambda$main$0(String string) {
    return string.contains("Hollis");
}
```

Hai biểu thức Lambda lần lượt gọi 2 phương thức `lambda$main$1` và `lambda$main$0`.

**Do đó, việc thực thi Lambda Expression thực chất phụ thuộc vào một số API bên dưới, ở giai đoạn biên dịch trình biên dịch sẽ giải đường cho Lambda Expression, chuyển thành cách gọi API nội bộ.**

## Các bẫy dễ gặp phải

### Generics

**1. Khi Generics gặp Overloading**

```java
public class GenericTypes {

    public static void method(List<String> list) {
        System.out.println("invoke method(List<String> list)");
    }

    public static void method(List<Integer> list) {
        System.out.println("invoke method(List<Integer> list)");
    }
}
```

Đoạn code trên có 2 hàm overloading, vì kiểu tham số của chúng khác nhau, một cái là `List<String>` cái còn lại là `List<Integer>`, tuy nhiên đoạn code này không biên dịch được. Vì như chúng ta đã nói ở trước, các tham số `List<Integer>` và `List<String>` sau khi biên dịch đều bị xóa kiểu, biến thành cùng một kiểu nguyên thủy (raw type) List, hành động xóa kiểu khiến signature của 2 phương thức này trở nên hoàn toàn giống hệt nhau.

**2. Khi Generics gặp catch**

Tham số kiểu của Generics không thể dùng trong câu lệnh catch xử lý ngoại lệ của Java. Vì xử lý ngoại lệ do JVM tiến hành lúc runtime. Do thông tin kiểu bị xóa, JVM không thể phân biệt 2 kiểu ngoại lệ `MyException<String>` và `MyException<Integer>`.

**3. Khi trong Generics chứa biến static**

```java
public class StaticTest{
    public static void main(String[] args){
        GT<Integer> gti = new GT<Integer>();
        gti.var=1;
        GT<String> gts = new GT<String>();
        gts.var=2;
        System.out.println(gti.var);
    }
}
class GT<T>{
    public static int var=0;
    public void nothing(T x){}
}
```

Kết quả in ra của đoạn code trên là: 2!

Một số bạn có thể nhầm tưởng Generic Class là các class khác nhau, tương ứng với các bytecode khác nhau, thực chất do trải qua Type Erasure nên tất cả các instance Generic Class đều liên kết tới cùng một bản bytecode, biến static của Generic Class được dùng chung. Trong ví dụ trên `GT<Integer>.var` và `GT<String>.var` thực chất là cùng 1 biến.

### Autoboxing và Unboxing

**So sánh bằng đối tượng**

```java
public static void main(String[] args) {
    Integer a = 1000;
    Integer b = 1000;
    Integer c = 100;
    Integer d = 100;
    System.out.println("a == b is " + (a == b));
    System.out.println(("c == d is " + (c == d)));
}
```

Kết quả:

```plain
a == b is false
c == d is true
```

Trong Java 5, trên các thao tác của Integer đã đưa vào một tính năng mới để tiết kiệm bộ nhớ và nâng cao hiệu năng. Các đối tượng Integer thông qua việc sử dụng cùng một tham chiếu đối tượng đã thực hiện việc cache và tái sử dụng.

> Áp dụng cho khoảng giá trị số nguyên từ -128 đến +127.
>
> Chỉ áp dụng cho Autoboxing. Việc tạo đối tượng bằng constructor không áp dụng.

### Vòng lặp for nâng cao

```java
for (Student stu : students) {
    if (stu.getId() == 2)
        students.remove(stu);
}
```

Sẽ throw ngoại lệ `ConcurrentModificationException`.

Ở đây liên quan đến cơ chế **fail-fast (thất bại nhanh)** của Collection. Lấy `ArrayList` làm ví dụ, nội bộ nó duy trì một biến đếm `modCount`, mỗi lần thực hiện sửa đổi cấu trúc Collection (như thêm, xóa) thì biến đếm này tăng lên. Khi tạo `Iterator`, nó sẽ ghi lại `modCount` hiện tại thành `expectedModCount`. Mỗi lần gọi `next()`, `Iterator` đều kiểm tra `modCount` có bằng `expectedModCount` không, nếu không bằng chứng tỏ Collection trong quá trình duyệt đã bị sửa đổi theo cách khác, sẽ throw ngoại lệ `java.util.ConcurrentModificationException`.

Do đó `Iterator` khi đang làm việc thì không cho phép đối tượng được lặp bị thay đổi. Nhưng bạn có thể dùng phương thức `remove()` của chính `Iterator` để xóa đối tượng, phương thức `Iterator.remove()` sau khi xóa phần tử sẽ đồng bộ cập nhật `expectedModCount`, từ đó tránh kích hoạt ngoại lệ đó.

## Tóm tắt

Ở trên đã giới thiệu 12 loại Syntactic Sugar thường dùng trong Java. Cái gọi là Syntactic Sugar chỉ là một cú pháp cung cấp cho nhân viên phát triển thuận tiện khi lập trình mà thôi. Nhưng cú pháp này chỉ có nhân viên phát triển nhận biết. Muốn được thực thi thì cần tiến hành Desugaring (giải đường), tức chuyển thành cú pháp mà JVM nhận biết. Khi chúng ta giải đường cho Syntactic Sugar, bạn sẽ phát hiện các cú pháp tiện lợi chúng ta dùng hàng ngày thực chất đều do các cú pháp khác đơn giản hơn cấu thành.

Có các Syntactic Sugar này, chúng ta trong phát triển hàng ngày có thể nâng cao rất nhiều hiệu suất, tuy nhiên đồng thời cũng nên tránh lạm dụng. Trước khi dùng tốt nhất nên tìm hiểu nguyên lý để tránh rơi vào các bẫy.

<!-- @include: @article-footer.snippet.md -->
