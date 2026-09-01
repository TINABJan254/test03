---
title: Giải thích chi tiết Proxy Pattern trong Java
description: Giải thích chi tiết nguyên lý và cách triển khai Proxy Pattern trong Java: so sánh sự khác biệt giữa Static Proxy và Dynamic Proxy, phân tích chuyên sâu cơ chế JDK Dynamic Proxy và CGLIB Proxy, hiểu rõ việc triển khai tính năng cắt ngang (cross-cutting concerns) trong AOP.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Java Proxy Pattern,Static Proxy,Dynamic Proxy,JDK Dynamic Proxy,CGLIB Proxy,AOP,Design Pattern,triển khai Proxy
---

## 1. Proxy Pattern (Mẫu thiết kế Đại lý)

Proxy Pattern là một thiết kế tương đối dễ hiểu. Nói một cách đơn giản là **chúng ta sử dụng một đối tượng đại lý (Proxy Object) để thay thế việc truy cập trực tiếp vào đối tượng thực tế (Real Object), nhờ đó có thể cung cấp thêm các thao tác tính năng bổ sung mà không cần sửa đổi đối tượng mục tiêu ban đầu, giúp mở rộng tính năng của đối tượng mục tiêu.**

**Tác dụng chính của Proxy Pattern là mở rộng tính năng của đối tượng mục tiêu, ví dụ như bạn có thể thêm một số thao tác tự định nghĩa vào trước hoặc sau khi một phương thức nào đó của đối tượng mục tiêu được thực thi.**

Ví dụ: Cô dâu nhờ người dì của mình đứng ra xử lý các câu hỏi từ chú rể, những câu hỏi mà cô dâu nhận được đều đã được người dì lọc và xử lý qua. Người dì ở đây có thể xem như proxy object đại diện cho bạn, hành vi (phương thức) đại lý là nhận và trả lời các câu hỏi của chú rể.

![Understanding the Proxy Design Pattern | by Mithun Sasidharan | Medium](https://oss.javaguide.cn/2020-8/1*DjWCgTFm-xqbhbNQVsaWQw.png)

<p style="text-align:right;font-size:13px;color:gray">https://medium.com/@mithunsasidharan/understanding-the-proxy-design-pattern-5e63fe38052a</p>

Proxy Pattern có 2 cách triển khai là Static Proxy và Dynamic Proxy, chúng ta cùng xem cách triển khai Static Proxy trước.

## 2. Static Proxy (Proxy tĩnh)

Trong Static Proxy, việc tăng cường cho từng phương thức của đối tượng mục tiêu đều được làm thủ công (phần sau sẽ minh họa qua code), rất thiếu linh hoạt (ví dụ một khi Interface thêm phương thức mới thì cả đối tượng mục tiêu và đối tượng đại lý đều phải sửa đổi) và phiền phức (cần phải viết riêng một proxy class cho từng target class). Kịch bản ứng dụng thực tế rất rất ít, trong phát triển hàng ngày hầu như không thấy sử dụng Static Proxy.

Ở trên chúng ta nói về Static Proxy dưới góc độ triển khai và ứng dụng, còn dưới góc độ JVM, **Static Proxy ngay ở thời điểm biên dịch đã biến Interface, Implementation Class, Proxy Class thành từng file `.class` thực tế.**

Các bước triển khai Static Proxy:

1. Định nghĩa một Interface và Implementation Class của nó;
2. Tạo một Proxy Class cũng implement Interface này;
3. Inject đối tượng mục tiêu (Target Object) vào Proxy Class, sau đó trong phương thức tương ứng của Proxy Class gọi phương thức tương ứng của Target Class. Như vậy, chúng ta có thể chặn việc truy cập trực tiếp đối tượng mục tiêu thông qua Proxy Class, và có thể làm những việc mình muốn trước/sau khi phương thức mục tiêu thực thi.

Minh họa qua code bên dưới!

**1. Định nghĩa Interface gửi tin nhắn SMS**

```java
public interface SmsService {
    String send(String message);
}
```

**2. Implement Interface gửi tin nhắn SMS**

```java
public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
```

**3. Tạo Proxy Class và cũng implement Interface gửi tin nhắn SMS**

```java
public class SmsProxy implements SmsService {

    private final SmsService smsService;

    public SmsProxy(SmsService smsService) {
        this.smsService = smsService;
    }

    @Override
    public String send(String message) {
        // Trước khi gọi phương thức, chúng ta có thể thêm thao tác của riêng mình
        System.out.println("before method send()");
        smsService.send(message);
        // Sau khi gọi phương thức, chúng ta cũng có thể thêm thao tác của riêng mình
        System.out.println("after method send()");
        return null;
    }
}
```

**4. Sử dụng thực tế**

```java
public class Main {
    public static void main(String[] args) {
        SmsService smsService = new SmsServiceImpl();
        SmsProxy smsProxy = new SmsProxy(smsService);
        smsProxy.send("java");
    }
}
```

Sau khi chạy đoạn code trên, console in ra:

```bash
before method send()
send message:java
after method send()
```

Có thể thấy từ kết quả đầu ra, chúng ta đã thêm được logic vào phương thức `send()` của `SmsServiceImpl`.

## 3. Dynamic Proxy (Proxy động)

So với Static Proxy, Dynamic Proxy linh hoạt hơn rất nhiều. Chúng ta không cần tạo riêng một proxy class cho từng target class, và cũng không bắt buộc phải implement Interface, chúng ta có thể đại lý trực tiếp cho Implementation Class (cơ chế CGLIB Dynamic Proxy).

**Dưới góc độ JVM, Dynamic Proxy sinh ra bytecode của class một cách động lúc runtime và nạp vào JVM.**

Nói đến Dynamic Proxy thì Spring AOP và các RPC Framework là 2 ví dụ không thể không nhắc tới, việc triển khai của chúng đều phụ thuộc vào Dynamic Proxy.

**Dynamic Proxy ít khi được sử dụng trực tiếp trong phát triển hàng ngày, nhưng lại là kỹ thuật gần như bắt buộc phải dùng trong các framework. Học được Dynamic Proxy cũng giúp ích rất nhiều cho việc thấu hiểu và học hỏi nguyên lý của các framework.**

Xét riêng trong Java, có rất nhiều cách triển khai Dynamic Proxy, ví dụ **JDK Dynamic Proxy**, **CGLIB Dynamic Proxy**,...

Dự án [guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) sử dụng JDK Dynamic Proxy, trước tiên chúng ta cùng xem cách dùng JDK Dynamic Proxy.

Ngoài ra, mặc dù [guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) không dùng đến **CGLIB Dynamic Proxy**, ở đây chúng ta vẫn sẽ giới thiệu đơn giản cách dùng cũng như so sánh với **JDK Dynamic Proxy**.

### 3.1. Cơ chế JDK Dynamic Proxy

#### 3.1.1. Giới thiệu

**Trong cơ chế Java Dynamic Proxy, interface `InvocationHandler` và lớp `Proxy` là cốt lõi.**

Phương thức có tần suất sử dụng cao nhất trong lớp `Proxy` là: `newProxyInstance()`, phương thức này chủ yếu dùng để tạo ra một proxy object.

```java
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        ......
    }
```

Phương thức này có 3 tham số:

1. **loader**: ClassLoader dùng để nạp proxy object.
2. **interfaces**: Các Interface mà class được proxy implement;
3. **h**: Đối tượng implement interface `InvocationHandler`;

Để triển khai Dynamic Proxy, bắt buộc phải implement `InvocationHandler` để tự định nghĩa logic xử lý. Khi proxy object của chúng ta gọi một phương thức, cuộc gọi phương thức đó sẽ được chuyển tiếp tới phương thức `invoke` của lớp implement interface `InvocationHandler`.

```java
public interface InvocationHandler {

    /**
     * Khi bạn dùng proxy object gọi phương thức thì thực tế sẽ gọi vào phương thức này
     */
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

Phương thức `invoke()` có 3 tham số bên dưới:

1. **proxy**: Proxy class được tạo động
2. **method**: Tương ứng với phương thức được gọi trên proxy object
3. **args**: Tham số của phương thức `method` hiện tại

Nghĩa là: **Proxy object do bạn tạo qua `newProxyInstance()` của lớp `Proxy` khi gọi phương thức thực chất sẽ gọi vào phương thức `invoke()` của lớp implement interface `InvocationHandler`.** Bạn có thể tự định nghĩa logic xử lý trong phương thức `invoke()`, ví dụ làm gì trước và sau khi phương thức thực thi.

#### 3.1.2. Các bước sử dụng lớp JDK Dynamic Proxy

1. Định nghĩa một Interface và Implementation Class của nó;
2. Tự định nghĩa `InvocationHandler` và override phương thức `invoke`, trong `invoke` chúng ta sẽ gọi phương thức gốc (phương thức của class được proxy) và thêm các logic xử lý tự định nghĩa;
3. Tạo proxy object thông qua phương thức `Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`;

#### 3.1.3. Ví dụ code

Nói lý thuyết có thể hơi trìu tượng và khó hiểu, tôi đưa ra ví dụ để mọi người cảm nhận nhé!

**1. Định nghĩa Interface gửi tin nhắn SMS**

```java
public interface SmsService {
    String send(String message);
}
```

**2. Implement Interface gửi tin nhắn SMS**

```java
public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
```

**3. Định nghĩa một lớp JDK Dynamic Proxy**

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * @author shuang.kou
 * @createTime 2020年05月11日 11:23:00
 */
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * Target object thực tế trong proxy class
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        // Trước khi gọi phương thức, chúng ta có thể thêm thao tác của riêng mình
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        // Sau khi gọi phương thức, chúng ta cũng có thể thêm thao tác của riêng mình
        System.out.println("after method " + method.getName());
        return result;
    }
}
```

Phương thức `invoke()`: Khi Dynamic Proxy object của chúng ta gọi phương thức gốc, cuối cùng thực chất sẽ gọi vào phương thức `invoke()`, sau đó phương thức `invoke()` sẽ đại diện chúng ta gọi phương thức gốc của đối tượng được proxy.

**4. Factory Class để lấy proxy object**

```java
public class JdkProxyFactory {
    public static Object getProxy(Object target) {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(), // ClassLoader của target class
                target.getClass().getInterfaces(),  // Các interface mà proxy cần implement, có thể chỉ định nhiều
                new DebugInvocationHandler(target)   // Custom InvocationHandler tương ứng của proxy object
        );
    }
}
```

`getProxy()`: Chủ yếu lấy proxy object của một class nào đó thông qua phương thức `Proxy.newProxyInstance()`

**5. Sử dụng thực tế**

```java
SmsService smsService = (SmsService) JdkProxyFactory.getProxy(new SmsServiceImpl());
smsService.send("java");
```

Sau khi chạy đoạn code trên, console in ra:

```plain
before method send
send message:java
after method send
```

### 3.2. Cơ chế CGLIB Dynamic Proxy

#### 3.2.1. Giới thiệu

**JDK Dynamic Proxy có một điểm yếu chí mạng là chỉ có thể đại lý cho các class có implement Interface.**

**Để giải quyết vấn đề này, chúng ta có thể dùng cơ chế CGLIB Dynamic Proxy để khắc phục.**

[CGLIB](https://github.com/cglib/cglib) (*Code Generation Library*) là một thư viện sinh bytecode dựa trên [ASM](http://www.baeldung.com/java-asm), cho phép chúng ta sửa đổi và sinh động bytecode lúc runtime. CGLIB thực hiện đại lý thông qua kế thừa. Rất nhiều framework mã nguồn mở nổi tiếng đều sử dụng [CGLIB](https://github.com/cglib/cglib), ví dụ module AOP trong Spring: Nếu đối tượng mục tiêu có implement interface thì mặc định dùng JDK Dynamic Proxy, ngược lại dùng CGLIB Dynamic Proxy.

**Trong cơ chế CGLIB Dynamic Proxy, interface `MethodInterceptor` và lớp `Enhancer` là cốt lõi.**

Bạn cần tự định nghĩa `MethodInterceptor` và override phương thức `intercept`, `intercept` dùng để chặn và tăng cường các phương thức của class được đại lý.

```java
public interface MethodInterceptor
extends Callback{
    // Chặn phương thức trong class được đại lý
    public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args,MethodProxy proxy) throws Throwable;
}
```

1. **obj**: Đối tượng được đại lý (đối tượng cần tăng cường)
2. **method**: Phương thức bị chặn (phương thức cần tăng cường)
3. **args**: Tham số đầu vào của phương thức
4. **proxy**: Dùng để gọi phương thức gốc

Bạn có thể lấy class được đại lý một cách động thông qua lớp `Enhancer`, khi proxy class gọi phương thức thì thực tế sẽ gọi vào phương thức `intercept` trong `MethodInterceptor`.

#### 3.2.2. Các bước sử dụng lớp CGLIB Dynamic Proxy

1. Định nghĩa một class;
2. Tự định nghĩa `MethodInterceptor` và override phương thức `intercept`, `intercept` dùng để chặn và tăng cường phương thức của class được đại lý, tương tự phương thức `invoke` trong JDK Dynamic Proxy;
3. Tạo proxy class thông qua phương thức `create()` của lớp `Enhancer`;

#### 3.2.3. Ví dụ code

Khác với JDK Dynamic Proxy không cần dependency bổ sung, [CGLIB](https://github.com/cglib/cglib) (*Code Generation Library*) thực chất thuộc về một dự án mã nguồn mở, nếu bạn muốn dùng nó thì cần thêm dependency liên quan thủ công.

```xml
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>3.3.0</version>
</dependency>
```

**1. Implement một class gửi tin nhắn SMS qua Aliyun**

```java
package github.javaguide.dynamicProxy.cglibDynamicProxy;

public class AliSmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
```

**2. Tự định nghĩa `MethodInterceptor` (Bộ chặn phương thức)**

```java
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * Custom MethodInterceptor
 */
public class DebugMethodInterceptor implements MethodInterceptor {


    /**
     * @param o           Bản thân proxy object (lưu ý không phải đối tượng gốc, nếu dùng method.invoke(o, args) sẽ gây ra vòng lặp vô hạn)
     * @param method      Phương thức bị chặn (phương thức cần tăng cường)
     * @param args        Tham số đầu vào của phương thức
     * @param methodProxy Cơ chế gọi phương thức hiệu năng cao, tránh chi phí của Reflection
     */
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        // Trước khi gọi phương thức, chúng ta có thể thêm thao tác của riêng mình
        System.out.println("before method " + method.getName());
        Object object = methodProxy.invokeSuper(o, args);
        // Sau khi gọi phương thức, chúng ta cũng có thể thêm thao tác của riêng mình
        System.out.println("after method " + method.getName());
        return object;
    }

}
```

**3. Lấy proxy class**

```java
import net.sf.cglib.proxy.Enhancer;

public class CglibProxyFactory {

    public static Object getProxy(Class<?> clazz) {
        // Tạo lớp tăng cường Dynamic Proxy
        Enhancer enhancer = new Enhancer();
        // Đặt ClassLoader
        enhancer.setClassLoader(clazz.getClassLoader());
        // Đặt class được đại lý (super class)
        enhancer.setSuperclass(clazz);
        // Đặt bộ chặn phương thức
        enhancer.setCallback(new DebugMethodInterceptor());
        // Tạo proxy class
        return enhancer.create();
    }
}
```

**4. Sử dụng thực tế**

```java
AliSmsService aliSmsService = (AliSmsService) CglibProxyFactory.getProxy(AliSmsService.class);
aliSmsService.send("java");
```

Sau khi chạy đoạn code trên, console in ra:

```bash
before method send
send message:java
after method send
```

### 3.3. So sánh JDK Dynamic Proxy và CGLIB Dynamic Proxy

1. JDK Dynamic Proxy là chính thức của Java, nó yêu cầu class được đại lý bắt buộc phải implement Interface. Nguyên lý của nó là sinh động một implementation class của interface làm proxy. CGLIB là của bên thứ ba, nó không yêu cầu interface. Nguyên lý của nó là sinh động một lớp con của class được đại lý làm proxy. Nhưng cũng chính vì dựa trên kế thừa nên nó không thể đại lý cho class `final`, và các phương thức được đại lý cũng không thể là `final` hoặc `private`.
2. Về hiệu suất của cả hai, đại đa số trường hợp JDK Dynamic Proxy đều tốt hơn, cùng với việc nâng cấp phiên bản JDK, ưu thế này ngày càng rõ rệt.

## 4. So sánh Static Proxy và Dynamic Proxy

Sự khác biệt cốt lõi giữa Static Proxy và Dynamic Proxy nằm ở **Thời điểm xác định quan hệ proxy, Tính linh hoạt khi thực thi và Chi phí bảo trì**.

| Tiêu chí so sánh | Static Proxy | Dynamic Proxy |
| --- | --- | --- |
| **Thời điểm xác định quan hệ Proxy** | Thời điểm biên dịch (sau khi biên dịch sinh ra file bytecode `.class` cố định) | Thời điểm runtime (sinh động bytecode của proxy class và nạp vào JVM) |
| **Cách thực thi** | Trước khi biên dịch viết thủ công proxy class, thường qua kết hợp và ủy quyền gọi target object | Không cần viết thủ công proxy class cụ thể, đóng gói logic tăng cường qua `Handler`/`Interceptor` |
| **Phụ thuộc Interface** | Không bắt buộc; Static Proxy dựa trên interface thường để proxy class và target class chung interface | JDK Dynamic Proxy hướng tới Interface, CGLIB proxy lớp con hướng tới implementation class có thể kế thừa |
| **Lượng code & Bảo trì** | Lượng code lớn (càng nhiều target class càng nhiều proxy class), chi phí bảo trì cao; khi interface thêm method, target class và proxy class phải sửa đồng bộ | Lượng code cực ít (logic tăng cường chung có thể tái sử dụng), tính bảo trì tốt; tách biệt với interface, thay đổi interface không ảnh hưởng logic proxy |
| **Ưu thế cốt lõi** | Thực hiện đơn giản, logic trực quan, không phụ thuộc framework ngoài | Tính linh hoạt mạnh, tính tái sử dụng cao, giảm viết code trùng lặp, thích ứng kịch bản phức tạp |
| **Kịch bản ứng dụng điển hình** | Decorator Pattern đơn giản, nhu cầu tăng cường cho ít class cố định | Spring AOP, RPC framework (như Dubbo), ORM framework |

## 5. Tóm tắt

Bài viết này chủ yếu giới thiệu 2 cách thực hiện Proxy Pattern: Static Proxy và Dynamic Proxy. Bao gồm thực hành Static Proxy và Dynamic Proxy, sự khác biệt giữa Static Proxy và Dynamic Proxy, sự khác biệt giữa JDK Dynamic Proxy và CGLIB Dynamic Proxy,...

Tất cả mã nguồn liên quan trong bài viết, bạn có thể tìm thấy tại đây: [https://github.com/Snailclimb/guide-rpc-framework-learning/tree/master/src/main/java/github/javaguide/proxy](https://github.com/Snailclimb/guide-rpc-framework-learning/tree/master/src/main/java/github/javaguide/proxy).

<!-- @include: @article-footer.snippet.md -->
