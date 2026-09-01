---
title: Giải thích chi tiết cơ chế Reflection trong Java
description: Giải thích chi tiết nguyên lý và ứng dụng của cơ chế Reflection trong Java: nắm vững các API cốt lõi như Class, Method, Field, hiểu rõ ứng dụng của Reflection trong các framework như Spring, MyBatis, học cách thực hiện Dynamic Proxy.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Java Reflection,cơ chế phản xạ,lớp Class,phương thức Method,trường Field,dynamic proxy,nguyên lý framework,thao tác runtime
---

## Reflection là gì?

Nếu mọi người từng nghiên cứu nguyên lý bên dưới của các framework hoặc từng tự mình viết framework, chắc chắn không hề xa lạ với khái niệm Reflection (Phản xạ).

Reflection sở dĩ được gọi là linh hồn của framework, chủ yếu vì nó trao cho chúng ta khả năng phân tích class cũng như thực thi các phương thức trong class tại thời điểm runtime.

Thông qua Reflection có thể lấy được các thông tin như field, method, constructor,... của class, và tiến hành gọi hoặc đọc ghi trong điều kiện kiểm soát truy cập và ranh giới module cho phép. Phạm vi của các API khác nhau cũng khác nhau, ví dụ `getMethods()` trả về các phương thức public có thể truy cập, còn `getDeclaredMethods()` trả về các phương thức do class hiện tại khai báo nhưng không bao gồm phương thức kế thừa.

## Bạn hiểu gì về các kịch bản ứng dụng của Reflection?

Hầu hết thời gian chúng ta đều viết code nghiệp vụ, rất ít khi tiếp xúc với các kịch bản trực tiếp sử dụng cơ chế Reflection.

Tuy nhiên, điều đó không có nghĩa là Reflection không có ích. Ngược lại, chính nhờ có Reflection mà bạn mới có thể sử dụng các framework một cách dễ dàng như vậy. Các framework như Spring/Spring Boot, MyBatis,... đều sử dụng rộng rãi cơ chế Reflection.

**Các framework này cũng sử dụng rộng rãi Dynamic Proxy, mà việc thực hiện Dynamic Proxy lại phụ thuộc vào Reflection.**

Ví dụ dưới đây là code minh họa thực hiện Dynamic Proxy thông qua JDK, trong đó có sử dụng lớp Reflection `Method` để gọi phương thức chỉ định.

```java
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * Target object thực tế trong proxy class
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }


    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after method " + method.getName());
        return result;
    }
}
```

Ngoài ra, một công cụ lợi hại trong Java là **Annotation** khi thực thi cũng cần dùng đến Reflection.

Tại sao khi bạn sử dụng Spring, chỉ cần một annotation `@Component` là đã khai báo một lớp thành Spring Bean? Tại sao bạn thông qua một annotation `@Value` là đã đọc được giá trị trong file cấu hình? Rốt cuộc nó hoạt động như thế nào?

Tất cả những điều này là do bạn có thể dựa vào Reflection để phân tích class, sau đó lấy ra các annotation trên class/attribute/method/method parameter. Sau khi lấy được annotation, bạn có thể thực hiện các xử lý tiếp theo.

## Nhận xét về ưu nhược điểm của cơ chế Reflection

**Ưu điểm**: Giúp code của chúng ta linh hoạt hơn, tạo thuận lợi cho việc cung cấp các tính năng out-of-the-box cho nhiều framework khác nhau.

**Nhược điểm**: Trao cho chúng ta khả năng phân tích và thao tác class lúc runtime, điều này đồng thời cũng làm tăng các vấn đề an toàn bảo mật. Ví dụ có thể bỏ qua việc kiểm tra an toàn kiểu của tham số Generics (kiểm tra an toàn tham số Generics diễn ra lúc biên dịch). Ngoài ra, hiệu năng của Reflection cũng kém hơn một chút, tuy nhiên đối với framework thì ảnh hưởng thực tế không quá lớn. Đọc thêm: [Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow)

## Thực hành Reflection

### 4 cách lấy đối tượng Class

Nếu chúng ta muốn lấy động các thông tin này, chúng ta cần dựa vào đối tượng Class. Lớp đối tượng Class sẽ báo cho chương trình đang chạy biết thông tin về các phương thức, biến,... của một class. Java cung cấp 4 cách để lấy đối tượng Class:

**1. Sử dụng khi đã biết class cụ thể:**

```java
Class alunbarClass = TargetObject.class;
```

Cách này thích hợp cho kịch bản lúc biên dịch đã biết kiểu cụ thể, bản thân việc lấy class literal sẽ không kích hoạt khởi tạo class.

**2. Thông qua `Class.forName()` truyền vào đường dẫn đầy đủ của class:**

```java
Class alunbarClass1 = Class.forName("cn.javaguide.TargetObject");
```

**3. Thông qua instance của đối tượng `instance.getClass()`:**

```java
TargetObject o = new TargetObject();
Class alunbarClass2 = o.getClass();
```

**4. Thông qua ClassLoader `xxxClassLoader.loadClass()` truyền vào đường dẫn class:**

```java
ClassLoader.getSystemClassLoader().loadClass("cn.javaguide.TargetObject");
```

Việc lấy đối tượng Class thông qua ClassLoader sẽ không tiến hành khởi tạo class, nghĩa là không thực hiện chuỗi các bước bao gồm khởi tạo, các khối code static và đối tượng static sẽ không được thực thi.

### Một số thao tác Reflection cơ bản

1. Tạo một class `TargetObject` để chúng ta thao tác bằng Reflection.

```java
package cn.javaguide;

public class TargetObject {
    private String value;

    public TargetObject() {
        value = "JavaGuide";
    }

    public void publicMethod(String s) {
        System.out.println("I love " + s);
    }

    private void privateMethod() {
        System.out.println("value is " + value);
    }
}
```

2. Sử dụng Reflection thao tác các phương thức và thuộc tính của class này

```java
package cn.javaguide;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException, NoSuchFieldException {
        /**
         * Lấy đối tượng Class của lớp TargetObject và tạo instance cho lớp TargetObject
         */
        Class<?> targetClass = Class.forName("cn.javaguide.TargetObject");
        TargetObject targetObject = (TargetObject) targetClass.getDeclaredConstructor().newInstance();
        /**
         * Lấy tất cả các phương thức được định nghĩa trong lớp TargetObject
         */
        Method[] methods = targetClass.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method.getName());
        }

        /**
         * Lấy phương thức chỉ định và gọi
         */
        Method publicMethod = targetClass.getDeclaredMethod("publicMethod",
                String.class);

        publicMethod.invoke(targetObject, "JavaGuide");

        /**
         * Lấy tham số chỉ định và sửa đổi tham số
         */
        Field field = targetClass.getDeclaredField("value");
        // Để sửa đổi tham số trong class, chúng ta hủy bỏ kiểm tra an toàn
        field.setAccessible(true);
        field.set(targetObject, "JavaGuide");

        /**
         * Gọi phương thức private
         */
        Method privateMethod = targetClass.getDeclaredMethod("privateMethod");
        // Để gọi phương thức private, chúng ta hủy bỏ kiểm tra an toàn
        privateMethod.setAccessible(true);
        privateMethod.invoke(targetObject);
    }
}
```

Nội dung in ra:

```plain
publicMethod
privateMethod
I love JavaGuide
value is JavaGuide
```

**Lưu ý**: Có độc giả đề cập chạy code trên sẽ throw ngoại lệ `ClassNotFoundException`, nguyên nhân cụ thể là do bạn chưa thay đổi tên package trong đoạn code bên dưới thành package chứa `TargetObject` do bạn tạo.
Có thể tham khảo bài viết: <https://www.cnblogs.com/chanshuyi/p/head_first_of_reflection.html>.

```java
Class<?> targetClass = Class.forName("cn.javaguide.TargetObject");
```

<!-- @include: @article-footer.snippet.md -->
