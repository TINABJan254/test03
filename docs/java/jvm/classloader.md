---
title: 类加载器详解（重点）
description: Java类加载器详解：深入剖析ClassLoader类加载机制、双亲委派模型原理、启动类加载器/平台类加载器/应用类加载器、自定义类加载器实现、打破双亲委派场景。
category: Java
tag:
  - JVM
head:
  - - meta
    - name: keywords
      content: 类加载器,ClassLoader,双亲委派模型,类加载过程,自定义类加载器,打破双亲委派
---

## Nhắc lại quá trình Class Loading

Trước khi bắt đầu giới thiệu về ClassLoader và Parents Delegation Model, hãy cùng nhìn lại ngắn gọn quá trình Class Loading.

- Quá trình Class Loading: **Loading -> Linking -> Initialization**.
- Quá trình Linking lại có thể chia thành 3 bước: **Verification -> Preparation -> Resolution**.

![Quá trình Class Loading](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-procedure.png)

Loading là bước đầu tiên trong quá trình Class Loading, chủ yếu hoàn thành 3 công việc dưới đây:

1. Thông qua FQCN (Fully Qualified Class Name - tên đầy đủ của class) để lấy luồng byte nhị phân định nghĩa Class này.
2. Chuyển đổi cấu trúc lưu trữ tĩnh đại diện bởi luồng byte thành cấu trúc dữ liệu runtime trong Method Area.
3. Tạo một đối tượng `Class` đại diện cho Class đó trong bộ nhớ làm lối vào truy cập các dữ liệu này trong Method Area.

## ClassLoader (Bộ nạp lớp)

### Giới thiệu về ClassLoader

ClassLoader đã xuất hiện từ JDK 1.0, ban đầu chỉ để đáp ứng nhu cầu của Java Applet (đã bị loại bỏ). Sau này, nó dần trở thành một thành phần quan trọng trong chương trình Java, mang lại khả năng cho phép các class Java có thể được nạp động vào JVM và thực thi.

Theo phần giới thiệu trong tài liệu API chính thức:

> A class loader is an object that is responsible for loading classes. The class ClassLoader is an abstract class. Given the binary name of a class, a class loader should attempt to locate or generate data that constitutes a definition for the class. A typical strategy is to transform the name into a file name and then read a "class file" of that name from a file system.
>
> Every Class object contains a reference to the ClassLoader that defined it.
>
> Class objects for array classes are not created by class loaders, but are created automatically as required by the Java runtime. The class loader for an array class, as returned by Class.getClassLoader() is the same as the class loader for its element type; if the element type is a primitive type, then the array class has no class loader.

Dịch ra có ý nghĩa đại thể là:

> ClassLoader là một đối tượng chịu trách nhiệm nạp các class. `ClassLoader` là một abstract class. Khi được cung cấp tên nhị phân của một class, ClassLoader nên thử định vị hoặc tạo ra dữ liệu cấu thành định nghĩa cho class đó. Chiến lược điển hình là chuyển đổi tên thành tên file và sau đó đọc một "class file" có tên đó từ hệ thống file.
>
> Mỗi đối tượng Class phi mảng hoặc interface đều có một reference trỏ tới `ClassLoader` đã định nghĩa nó. Các mảng class không được tạo thông qua `ClassLoader`, mà do JVM tự động tạo khi cần; ClassLoader của mảng kiểu reference đồng nhất với kiểu thành phần của nó, mảng kiểu nguyên thủy thì không có ClassLoader, `getClassLoader()` trả về `null`.

Từ phần giới thiệu trên có thể thấy:

- ClassLoader là một đối tượng chịu trách nhiệm nạp class, dùng để thực hiện bước Loading trong quá trình Class Loading.
- Mỗi class phi mảng hoặc interface đều ghi lại `ClassLoader` định nghĩa nó.
- Mảng class không được tạo thông qua `ClassLoader` (mảng class không có luồng byte nhị phân tương ứng), mà do JVM tạo trực tiếp; mảng kiểu reference kế thừa ClassLoader định nghĩa của kiểu thành phần, mảng kiểu nguyên thủy không có ClassLoader.

```java
class Class<T> {
  ...
  private final ClassLoader classLoader;
  @CallerSensitive
  public ClassLoader getClassLoader() {
     //...
  }
  ...
}
```

Nói một cách đơn giản, **tác dụng chủ yếu của ClassLoader là nạp động bytecode của các class Java (file `.class`) vào trong JVM (tạo một đối tượng `Class` đại diện cho class đó trong bộ nhớ).** Bytecode có thể thu được từ chương trình nguồn Java (file `.java`) biên dịch qua `javac`, cũng có thể tạo động thông qua công cụ hoặc tải về qua mạng.

Thực ra ngoài nạp class ra, ClassLoader còn có thể nạp các tài nguyên mà ứng dụng Java cần như văn bản, hình ảnh, file cấu hình, video v.v. Bài viết này chỉ thảo luận về chức năng cốt lõi của nó: nạp class.

### Quy tắc nạp của ClassLoader

Khi JVM khởi động, nó không nạp tất cả các class một lúc, mà nạp động theo nhu cầu. Nói cách khác, hầu hết các class khi thực sự dùng tới mới được nạp, làm như vậy sẽ thân thiện hơn với bộ nhớ.

Khi nạp class, `ClassLoader#loadClass` trước tiên sẽ thông qua `findLoadedClass` để phán đoán xem JVM đã ghi lại ClassLoader hiện tại làm nạp phát động cho class tương ứng với tên nhị phân đó chưa, nếu trúng sẽ trả về trực tiếp, nếu không mới tiếp tục ủy quyền hoặc tìm kiếm. Một ClassLoader không thể định nghĩa lặp lại các class có cùng tên nhị phân.

```java
// Các field và method dưới đây trích từ triển khai ClassLoader trong JDK 8, chỉ dùng để minh họa cách ghi lại của phiên bản đó
public abstract class ClassLoader {
  ...
  private final ClassLoader parent;
  // Class do ClassLoader này nạp.
  private final Vector<Class<?>> classes = new Vector<>();
  // Do VM gọi, dùng ClassLoader này ghi lại mỗi Class đã nạp.
  void addClass(Class<?> c) {
        classes.addElement(c);
   }
  ...
}
```

### Tóm tắt ClassLoader

Trong JDK 8, ba ClassLoader quan trọng thường gặp như sau:

1. **`BootstrapClassLoader` (Startup ClassLoader / Bộ nạp lớp khởi động)**: ClassLoader tích hợp sẵn của Virtual Machine ở tầng đỉnh, trong Java API thường biểu diễn là `null`, và không có ClassLoader cha. Trong HotSpot JDK 8, nó chủ yếu nạp các thư viện class cốt lõi lúc runtime (như `rt.jar`) cũng như các class trong đường dẫn chỉ định bởi `-Xbootclasspath`.
2. **`ExtensionClassLoader` (Extension ClassLoader / Bộ nạp lớp mở rộng)**: Chủ yếu chịu trách nhiệm nạp các file jar và class trong thư mục `%JRE_HOME%/lib/ext` cũng như tất cả các class dưới đường dẫn chỉ định bởi biến hệ thống `java.ext.dirs`.
3. **`AppClassLoader` (Application ClassLoader / Bộ nạp lớp ứng dụng)**: ClassLoader hướng tới người dùng chúng ta, chịu trách nhiệm nạp tất cả các file jar và class dưới classpath của ứng dụng hiện tại.

> 🌈 Mở rộng thêm:
>
> - **`rt.jar`**: rt đại diện cho "RunTime", `rt.jar` là thư viện class cơ sở Java, chứa tất cả các class file của các class nhìn thấy trong Java doc. Nói cách khác, các thư viện tích hợp thường dùng `java.xxx.*` của chúng ta đều nằm bên trong, ví dụ `java.util.*`, `java.io.*`, `java.nio.*`, `java.lang.*`, `java.sql.*`, `java.math.*`.
> - Java 9 giới thiệu Module System sau đó không còn sử dụng `rt.jar` và cơ chế thư mục mở rộng nữa, Extension ClassLoader được thay thế bởi Platform ClassLoader. Bootstrap, Platform và App ClassLoader lần lượt định nghĩa các module runtime khác nhau; không thể khái quát đơn giản là trừ `java.base` ra tất cả các module còn lại đều do Platform ClassLoader nạp.

Ngoài ba loại ClassLoader này ra, người dùng còn có thể thêm Custom ClassLoader để mở rộng, nhằm đáp ứng các nhu cầu đặc biệt của mình. Ví dụ, chúng ta có thể mã hóa bytecode của class Java (file `.class`), khi nạp lại dùng Custom ClassLoader để giải mã nó.

![Sơ đồ quan hệ phân cấp ClassLoader](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loader-parents-delegation-model.png)

Bootstrap ClassLoader tích hợp sẵn trong Virtual Machine, thường biểu diễn là `null` trong Java API. Platform ClassLoader, App ClassLoader cũng như Custom ClassLoader thông thường đều là instance của `ClassLoader`. Như vậy người dùng có thể tự định nghĩa ClassLoader, để ứng dụng tự quyết định cách thức lấy các class cần thiết.

Mỗi `ClassLoader` có thể thông qua `getParent()` để lấy ClassLoader cha của nó, nếu `ClassLoader` lấy được là `null`, thì ClassLoader cha của ClassLoader đó là `BootstrapClassLoader`.

```java
public abstract class ClassLoader {
  ...
  // ClassLoader cha
  private final ClassLoader parent;
  @CallerSensitive
  public final ClassLoader getParent() {
     //...
  }
  ...
}
```

**Tại sao Class do Bootstrap ClassLoader định nghĩa khi gọi `getClassLoader()` lại nhận được `null`?** Vì Bootstrap ClassLoader là ClassLoader tích hợp sẵn của JVM, Java API quy định thông thường dùng `null` để biểu thị nó; ngôn ngữ triển khai cụ thể không phải là một phần của quy chuẩn Java.

Dưới đây chúng ta xem một ví dụ nhỏ về việc lấy `ClassLoader`:

```java
public class PrintClassLoaderTree {

    public static void main(String[] args) {

        ClassLoader classLoader = PrintClassLoaderTree.class.getClassLoader();

        StringBuilder split = new StringBuilder("|--");
        boolean needContinue = true;
        while (needContinue){
            System.out.println(split.toString() + classLoader);
            if(classLoader == null){
                needContinue = false;
            }else{
                classLoader = classLoader.getParent();
                split.insert(0, "\t");
            }
        }
    }

}
```

Kết quả đầu ra (JDK 8):

```plain
|--sun.misc.Launcher$AppClassLoader@18b4aac2
    |--sun.misc.Launcher$ExtClassLoader@53bd815b
        |--null
```

Từ kết quả đầu ra có thể thấy:

- `ClassLoader` của class Java `PrintClassLoaderTree` do chúng ta viết là `AppClassLoader`;
- ClassLoader cha của `AppClassLoader` là `ExtClassLoader`;
- ClassLoader cha của `ExtClassLoader` là `Bootstrap ClassLoader`, do đó kết quả đầu ra là null.

### Custom ClassLoader (Bộ nạp lớp tự định nghĩa)

Chúng ta ở phần trước cũng đã nói, ngoài `BootstrapClassLoader` ra các ClassLoader khác đều do Java triển khai và tất cả đều kế thừa từ `java.lang.ClassLoader`. Nếu chúng ta muốn tự định nghĩa ClassLoader của riêng mình, rõ ràng cần kế thừa abstract class `ClassLoader`.

Class `ClassLoader` có hai phương thức then chốt:

- `protected Class loadClass(String name, boolean resolve)`: Nạp class với tên nhị phân chỉ định, triển khai Parents Delegation Model. `name` là tên nhị phân của class, nếu `resolve` là true, khi nạp sẽ gọi phương thức `resolveClass(Class<?> c)` để parse class đó.
- `protected Class findClass(String name)`: Tìm kiếm class dựa trên tên nhị phân của class, triển khai mặc định trực tiếp ném ra `ClassNotFoundException`.

Tài liệu API chính thức có viết:

> Subclasses of `ClassLoader` are encouraged to override `findClass(String name)`, rather than this method.
>
> Khuyến nghị các class con của `ClassLoader` nên override phương thức `findClass(String name)` thay vì phương thức `loadClass(String name, boolean resolve)`.

Nếu chúng ta không muốn phá vỡ Parents Delegation Model, chỉ cần override phương thức `findClass()` trong class `ClassLoader` là được, các class không thể nạp bởi ClassLoader cha cuối cùng sẽ được nạp thông qua phương thức này. Tuy nhiên, nếu muốn phá vỡ Parents Delegation Model thì cần override phương thức `loadClass()`.

## Parents Delegation Model (Mô hình ủy quyền cha)

### Giới thiệu Parents Delegation Model

Có rất nhiều loại ClassLoader, khi chúng ta muốn nạp một class thì cụ thể là ClassLoader nào nạp? Việc này cần phải nhắc tới Parents Delegation Model.

Theo phần giới thiệu trên trang chủ chính thức:

> The ClassLoader class uses a delegation model to search for classes and resources. Each instance of ClassLoader has an associated parent class loader. When requested to find a class or resource, a ClassLoader instance will delegate the search for the class or resource to its parent class loader before attempting to find the class or resource itself. The virtual machine's built-in class loader, called the "bootstrap class loader", does not itself have a parent but may serve as the parent of a ClassLoader instance.

Dịch ra có ý nghĩa đại thể là:

> Class `ClassLoader` sử dụng mô hình ủy quyền để tìm kiếm các class và tài nguyên. Mỗi instance `ClassLoader` đều có một ClassLoader cha liên quan. Khi có yêu cầu tìm kiếm class hoặc tài nguyên, instance `ClassLoader` sẽ trước khi tự mình tìm kiếm class hoặc tài nguyên, ủy quyền nhiệm vụ tìm kiếm class hoặc tài nguyên cho ClassLoader cha của nó.
> ClassLoader tích hợp sẵn trong Virtual Machine được gọi là "bootstrap class loader" bản thân không có ClassLoader cha, nhưng có thể đóng vai trò làm ClassLoader cha cho một instance `ClassLoader`.

Từ phần giới thiệu trên có thể thấy:

- Class `ClassLoader` sử dụng mô hình ủy quyền để tìm kiếm class và tài nguyên.
- Parents Delegation Model yêu cầu ngoài Startup ClassLoader ở tầng đỉnh ra, các ClassLoader còn lại đều nên có ClassLoader cha của mình.
- Instance `ClassLoader` sẽ trước khi tự mình thử tìm kiếm class hoặc tài nguyên, ủy quyền nhiệm vụ tìm kiếm class hoặc tài nguyên cho ClassLoader cha của nó.

Mối quan hệ phân cấp giữa các loại ClassLoader thể hiện ở hình dưới được gọi là "**Parents Delegation Model (Mô hình ủy quyền cha)**" của ClassLoader.

![Sơ đồ quan hệ phân cấp ClassLoader](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loader-parents-delegation-model.png)

Lưu ý ⚠️: Parents Delegation Model không phải là một ràng buộc mang tính bắt buộc, chỉ là một phương thức được JDK khuyến nghị. Nếu chúng ta vì một số nhu cầu đặc biệt muốn phá vỡ Parents Delegation Model thì cũng hoàn toàn có thể, phần sau sẽ giới thiệu phương pháp cụ thể.

Thực ra từ "Parents" dễ gây hiểu nhầm cho người khác, chúng ta thường hiểu parents là bố mẹ, parents ở đây thể hiện nhiều hơn về "thế hệ cha anh" mà thôi, chứ không phải thực sự có một `MotherClassLoader` và một `FatherClassLoader`. Cá nhân thấy dịch thành Single Delegation Model thì chuẩn hơn, tuy nhiên trong nước đã dịch thành Parents Delegation Model và lưu hành rộng rãi, tuân theo cách gọi này cũng không sao, không bị hiểu nhầm là được.

Ngoài ra, quan hệ cha con giữa các ClassLoader thường không phải triển khai bằng quan hệ kế thừa (inheritance), mà thông thường sử dụng quan hệ kết hợp (composition) để tái sử dụng code của ClassLoader cha.

```java
public abstract class ClassLoader {
  ...
  // Kết hợp (Composition)
  private final ClassLoader parent;
  protected ClassLoader(ClassLoader parent) {
       this(checkCreateClassLoader(), parent);
  }
  ...
}
```

Trong lập trình hướng đối tượng, có một nguyên tắc thiết kế rất kinh điển: **Composition tốt hơn Inheritance, dùng nhiều Composition ít dùng Inheritance.**

### Quy trình thực thi của Parents Delegation Model

Logic chủ yếu của Parents Delegation Model tập trung trong `loadClass()` của `java.lang.ClassLoader`. Dưới đây hiển thị đoạn trích triển khai liên quan trong JDK 8:

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // Trước tiên, kiểm tra xem class này đã được load chưa
        Class c = findLoadedClass(name);
        if (c == null) {
            // Nếu c là null, chứng tỏ class này chưa từng được load
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // Khi ClassLoader của class cha không rỗng, nạp class đó qua loadClass của class cha
                    c = parent.loadClass(name, false);
                } else {
                    // Khi ClassLoader của class cha rỗng, gọi Bootstrap ClassLoader để nạp class đó
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassLoader của class cha không rỗng không tìm thấy class tương ứng, ném exception
            }

            if (c == null) {
                // Khi ClassLoader cha không thể nạp, gọi phương thức findClass để nạp class đó
                // Người dùng có thể override phương thức này để tự định nghĩa ClassLoader
                long t1 = System.nanoTime();
                c = findClass(name);

                // Dùng để thống kê thông tin liên quan ClassLoader
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            // Thực hiện thao tác link với Class
            resolveClass(c);
        }
        return c;
    }
}
```

Mỗi khi một ClassLoader nhận được yêu cầu nạp, nó trước tiên sẽ chuyển tiếp yêu cầu tới ClassLoader cha. Trong trường hợp ClassLoader cha không tìm thấy class được yêu cầu, ClassLoader đó mới thử tự mình nạp.

Kết hợp với source code ở trên, tóm tắt ngắn gọn quy trình thực thi của Parents Delegation Model:

- Khi nạp class, hệ thống trước tiên sẽ phán đoán xem class hiện tại đã từng được nạp chưa. Class đã được nạp sẽ trực tiếp trả về, nếu chưa mới thử nạp (mỗi ClassLoader cha đều đi qua quy trình này).
- ClassLoader khi tiến hành nạp class, trước tiên nó không tự mình thử nạp class này, mà ủy quyền yêu cầu này cho ClassLoader cha hoàn thành (gọi phương thức `loadClass()` của ClassLoader cha để nạp class). Như vậy, tất cả các yêu cầu cuối cùng đều truyền tới BootstrapClassLoader ở tầng đỉnh.
- Chỉ khi ClassLoader cha phản hồi bản thân không thể hoàn thành yêu cầu nạp này (trong phạm vi tìm kiếm của nó không tìm thấy class cần thiết), ClassLoader con mới thử tự mình nạp (gọi phương thức `findClass()` của chính mình để nạp class).
- Nếu ClassLoader con cũng không thể nạp class này, nó sẽ ném ra một exception `ClassNotFoundException`.

🌈 Mở rộng thêm:

**Quy tắc cụ thể của JVM để xác định hai class Java có giống nhau hay không**: JVM không chỉ xem FQCN của class có giống nhau hay không, mà còn xem ClassLoader nạp class này có giống nhau hay không. Chỉ khi cả hai đều giống nhau, mới coi hai class là giống nhau. Cho dù hai class xuất xứ từ cùng một file `Class`, được nạp bởi cùng một Virtual Machine, chỉ cần ClassLoader nạp chúng khác nhau, thì hai class đó nhất định không giống nhau.

### Lợi ích của Parents Delegation Model

Parents Delegation Model là một phần cấu thành quan trọng của cơ chế Class Loading trong Java. Ưu tiên ClassLoader cha giúp code trong cùng chuỗi ủy quyền tái sử dụng type mà ClassLoader cha đã định nghĩa, và giảm rủi ro code ứng dụng giả mạo platform API; nó không thể ngăn cản các ClassLoader độc lập với nhau định nghĩa các class trùng tên.

Căn cứ để JVM phân biệt runtime type bao gồm tên nhị phân của class/interface và ClassLoader định nghĩa nó. Parents Delegation giúp platform class ưu tiên được định nghĩa bởi ClassLoader tích hợp tương ứng, nhưng không phải tất cả core hoặc platform API đều được nạp bởi Bootstrap ClassLoader: JDK 9 trở đi còn có Platform ClassLoader chịu trách nhiệm định nghĩa platform class.

Ví dụ, JVM sẽ ưu tiên giao yêu cầu nạp của các core class như `java.lang.Object` cho `BootstrapClassLoader` xử lý; nhưng thực tế `ClassLoader#preDefineClass` còn kiểm tra tên class ở giai đoạn định nghĩa, bất kỳ tên class nào bắt đầu bằng `java.` đều sẽ bị từ chối, do đó không thể giả mạo core class thông qua Custom ClassLoader.

Có nhiều bạn sẽ nói: "Vậy tôi bỏ qua Parents Delegation Model là được chứ gì?".

Tuy nhiên, cho dù attacker bỏ qua Parents Delegation Model, Java vẫn sở hữu cơ chế an toàn tầng dưới hơn để bảo vệ thư viện class cốt lõi. Phương thức `preDefineClass` của `ClassLoader` sẽ kiểm tra tên class trước khi định nghĩa class. Bất kỳ tên class nào bắt đầu bằng `"java."` đều sẽ kích hoạt `SecurityException`, ngăn chặn code độc hại định nghĩa hoặc nạp core class giả mạo.

Source code phương thức `ClassLoader#preDefineClass` trong JDK 8 như sau:

```java
private ProtectionDomain preDefineClass(String name,
                                            ProtectionDomain pd)
    {
        // Kiểm tra tên class có hợp lệ không
        if (!checkName(name)) {
            throw new NoClassDefFoundError("IllegalName: " + name);
        }

        // Ngăn chặn định nghĩa class trong package "java.*".
        // Kiểm tra này cực kỳ quan trọng đối với an toàn, vì nó ngăn chặn code độc hại thay thế core class Java.
        // JDK 9 tận dụng Platform ClassLoader tăng cường an toàn cho phương thức preDefineClass
        if ((name != null) && name.startsWith("java.")) {
            throw new SecurityException
                ("Package name forbidden: " +
                 name.substring(0, name.lastIndexOf('.')));
        }

         // Nếu chưa chỉ định ProtectionDomain, sử dụng domain mặc định (defaultDomain).
        if (pd == null) {
            pd = defaultDomain;
        }

        if (name != null) {
            checkCerts(name, pd.getCodeSource());
        }

        return pd;
    }
```

JDK 9 giới thiệu Platform ClassLoader, có thể lấy thông qua `ClassLoader.getPlatformClassLoader()`. Giới hạn của `defineClass` đối với tên package `java.*` vẫn tồn tại, nhưng code triển khai cụ thể khác với JDK 8.

### Phương pháp phá vỡ Parents Delegation Model

~~Để tránh cơ chế Parents Delegation, chúng ta có thể tự mình định nghĩa một ClassLoader, sau đó override `loadClass()` là được.~~

**🐛 Sửa lỗi (Xem thêm: [issue871](https://github.com/Snailclimb/JavaGuide/issues/871))**: Tự định nghĩa ClassLoader thì cần kế thừa `ClassLoader`. Nếu chúng ta không muốn phá vỡ Parents Delegation Model, chỉ cần override phương thức `findClass()` trong class `ClassLoader` là được, class không nạp được bởi ClassLoader cha cuối cùng sẽ được nạp qua phương thức này. Tuy nhiên, nếu muốn phá vỡ Parents Delegation Model thì cần override phương thức `loadClass()`.

Tại sao lại là override phương thức `loadClass()` để phá vỡ Parents Delegation Model? Quy trình thực thi của Parents Delegation Model đã giải thích:

> ClassLoader khi tiến hành nạp class, trước tiên nó không tự mình thử nạp class này, mà ủy quyền yêu cầu này cho ClassLoader cha hoàn thành (gọi phương thức `loadClass()` của ClassLoader cha để nạp class).

Sau khi override phương thức `loadClass()`, chúng ta có thể thay đổi quy trình thực thi của Parents Delegation Model truyền thống. Ví dụ, ClassLoader con có thể trước khi ủy quyền cho ClassLoader cha, tự mình thử nạp class này trước, hoặc sau khi ClassLoader cha trả về, lại thử nạp class này từ nơi khác. Quy tắc cụ thể do chính chúng ta triển khai, tùy chỉnh hóa theo nhu cầu dự án.

Tomcat server mà chúng ta khá quen thuộc để có thể ưu tiên nạp các class dưới thư mục ứng dụng Web, rồi mới nạp các class dưới các thư mục khác, đã tự định nghĩa ClassLoader `WebAppClassLoader` để phá vỡ cơ chế Parents Delegation. Đây cũng là nguyên lý cụ thể để thực hiện cách ly các class giữa các ứng dụng Web dưới Tomcat.

Cấu trúc phân cấp ClassLoader của Tomcat như sau:

![Cấu trúc phân cấp ClassLoader của Tomcat](https://oss.javaguide.cn/github/javaguide/java/jvm/tomcat-class-loader-parents-delegation-model.png)

Tomcat hiện đại mặc định sử dụng phân cấp `Bootstrap -> System -> Common -> WebappX`. Vị trí tìm kiếm của từng ClassLoader như sau:

- Vị trí tìm kiếm của `Common` do `common.loader` trong `$CATALINA_BASE/conf/catalina.properties` cấu hình, mặc định chủ yếu bao gồm `$CATALINA_BASE/lib` và `$CATALINA_HOME/lib`.
- Mỗi ClassLoader `WebappX` chịu trách nhiệm với `/WEB-INF/classes` và `/WEB-INF/lib/*.jar` của ứng dụng Web tương ứng.
- ClassLoader `Server` và `Shared` mặc định chưa được định nghĩa; chỉ khi cấu hình `server.loader` hoặc `shared.loader` mới xuất hiện trong phân cấp phức tạp hơn.

Từ quan hệ ủy quyền trong hình có thể thấy:

- Class hiển thị đối với `Common` loader có thể được dùng chung bởi các thành phần nội bộ Tomcat và tất cả ứng dụng Web.
- Nếu cấu hình hiển thị, `Server` chỉ hiển thị với nội bộ Tomcat, `Shared` thì hiển thị với tất cả ứng dụng Web. Cấu hình mặc định không có 2 ClassLoader này, Web application loader lấy trực tiếp `Common` làm ClassLoader cha.
- Mỗi ứng dụng Web đều sẽ tạo một `WebAppClassLoader` riêng biệt, và trong thread khởi động ứng dụng Web thiết lập Thread Context ClassLoader thành `WebAppClassLoader`. Các instance `WebAppClassLoader` cách ly lẫn nhau, từ đó thực hiện cách ly class giữa các ứng dụng Web.

Đơn thuần dựa vào Custom ClassLoader không cách nào đáp ứng yêu cầu của một số kịch bản, ví dụ, một số trường hợp, ClassLoader tầng cao hơn cần nạp các class mà chỉ ClassLoader tầng thấp hơn mới nạp được.

Ví dụ, trong SPI, interface của SPI (như `java.sql.Driver`) do thư viện Java Core cung cấp, do `BootstrapClassLoader` nạp. Còn triển khai của SPI (như `com.mysql.cj.jdbc.Driver`) do nhà cung cấp bên thứ ba cung cấp, chúng được nạp bởi Application ClassLoader hoặc Custom ClassLoader. Mặc định, một class và các class phụ thuộc của nó do cùng một ClassLoader nạp. Do đó, ClassLoader nạp interface của SPI (`BootstrapClassLoader`) cũng sẽ được dùng để nạp triển khai của SPI. Theo Parents Delegation Model, `BootstrapClassLoader` không thể tìm thấy class triển khai của SPI, vì nó không thể ủy quyền cho ClassLoader con thử nạp.

Ở đây cần lưu ý: Sau JDK 9+ giới thiệu mô-đun hóa, JDBC API được tách vào module `java.sql`, không còn là `BootstrapClassLoader` nạp trực tiếp nữa, mà do `PlatformClassLoader` nạp.

```java
public class ClassLoaderTest {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> clazz = Class.forName("java.sql.Driver");
        ClassLoader loader = clazz.getClassLoader();
        System.out.println("Loader for java.sql.Driver: " + loader);

        // Môi trường .jdks/corretto-1.8.0_442/bin/java là Loader for java.sql.Driver: null

        // Môi trường .jdks/jbr-17.0.12/bin/java là Loader for java.sql.Driver: jdk.internal.loader.ClassLoaders$PlatformClassLoader@30f39991
    }
}
```

Một ví dụ khác, giả sử trong dự án của chúng ta có jar package của Spring, do nó dùng chung giữa các ứng dụng Web nên sẽ do `SharedClassLoader` nạp (Web server là Tomcat). Trong dự án của chúng ta có một số business class dùng tới Spring, như implement interface do Spring cung cấp, dùng annotation do Spring cung cấp. Do đó, ClassLoader nạp Spring (tức `SharedClassLoader`) cũng sẽ được dùng để nạp các business class này. Nhưng business class nằm dưới thư mục ứng dụng Web, không nằm trên đường dẫn nạp của `SharedClassLoader`, nên `SharedClassLoader` không thể tìm thấy business class, cũng không thể nạp chúng.

Làm thế nào để giải quyết vấn đề này? Lúc này cần dùng tới **Thread Context ClassLoader (`ThreadContextClassLoader`)**.

Lấy ví dụ Spring làm minh họa, khi Spring cần nạp business class, nó không dùng ClassLoader của chính mình, mà dùng Context ClassLoader của thread hiện tại. Còn nhớ những gì mình đã nói ở trên không? Mỗi ứng dụng Web đều sẽ tạo một `WebAppClassLoader` riêng biệt, và trong thread khởi động ứng dụng Web thiết lập Thread Context ClassLoader thành `WebAppClassLoader`. Như vậy có thể cho phép ClassLoader tầng cao hơn (`SharedClassLoader`) nhờ vào ClassLoader con (`WebAppClassLoader`) để nạp business class, phá vỡ cơ chế ủy quyền Class Loading của Java, cho phép ứng dụng sử dụng ngược ClassLoader.

Nguyên lý của Thread Context ClassLoader là lưu một ClassLoader trong dữ liệu riêng của thread, ràng buộc với thread, sau đó khi cần thì lấy ra sử dụng. ClassLoader này thông thường do ứng dụng hoặc container (như Tomcat) thiết lập.

`getContextClassLoader()` và `setContextClassLoader(ClassLoader cl)` trong `java.lang.Thread` lần lượt dùng để lấy và thiết lập Thread Context ClassLoader. Nếu không thiết lập qua `setContextClassLoader(ClassLoader cl)`, thread sẽ kế thừa Thread Context ClassLoader của thread cha.

Code của Spring để lấy Thread Context ClassLoader như sau:

```java
cl = Thread.currentThread().getContextClassLoader();
```

Các bạn quan tâm có thể tự mình nghiên cứu sâu nguyên lý Tomcat phá vỡ Parents Delegation Model, tài liệu khuyến nghị: [《Giải thích sâu về Tomcat & Jetty》](http://gk.link/a/10Egr).

## Khuyến nghị đọc thêm

- 《Sâu sắc giải thích Java Virtual Machine》
- Phân tích sâu nguyên lý Java ClassLoader: <https://blog.csdn.net/xyang81/article/details/7292380>
- Java ClassLoader: <http://gityuan.com/2016/01/24/java-classloader/>
- Class Loaders in Java: <https://www.baeldung.com/java-classloaders>
- Class ClassLoader - Tài liệu chính thức Oracle: <https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html>
- Vấn đề nan giải Java ClassLoader không hiểu là già: <https://zhuanlan.zhihu.com/p/51374915>

<!-- @include: @article-footer.snippet.md -->
