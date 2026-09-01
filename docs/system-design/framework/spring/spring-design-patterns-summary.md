---
title: Spring 中的设计模式详解
description: Spring框架设计模式详解，涵盖工厂模式、代理模式、单例模式、模板方法等在Spring源码中的应用实践。
category: 框架
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: Spring设计模式,工厂模式,代理模式,模板方法,单例,策略模式,适配器模式,Spring源码
---

"Những design pattern nào được sử dụng trong JDK? Những design pattern nào được sử dụng trong Spring?" hai câu hỏi này khá thường gặp trong phỏng vấn.

Tôi đã tìm kiếm trên mạng về việc giải thích các design pattern trong Spring thì hầu như đều rập khuôn, hơn nữa đa số đều đã có từ lâu. Do đó, tôi đã dành vài ngày tự mình tổng hợp lại.

Vì năng lực cá nhân có hạn, nếu trong bài viết có bất kỳ sai sót nào các bạn đều có thể chỉ ra. Ngoài ra, độ dài bài viết có hạn, đối với các thiết kế pattern cũng như việc đọc hiểu một số mã nguồn tôi chỉ lướt qua, mục đích chính của bài viết này là điểm lại các design pattern trong Spring.

## Inversion of Control (IoC) và Dependency Injection (DI)

**IoC (Inversion of Control, Điều khiển đảo ngược)** là một khái niệm vô cùng vô cùng quan trọng trong Spring, nó không phải là một kỹ thuật gì, mà là một tư tưởng thiết kế nhằm giảm độ gắn kết (decoupling). Mục đích chính của IoC là nhờ vào "bên thứ ba" (IoC container trong Spring) để thực hiện việc giảm độ gắn kết giữa các đối tượng có quan hệ phụ thuộc (IoC container quản lý đối tượng, bạn chỉ việc sử dụng là được), từ đó làm giảm độ gắn kết giữa các đoạn code.

**IoC là một nguyên tắc, chứ không phải là một pattern, các pattern dưới đây (nhưng không giới hạn ở) thực hiện nguyên tắc IoC.**

![Các pattern ioc](https://oss.javaguide.cn/github/javaguide/ioc-patterns.png)

**Spring IoC container giống như một nhà máy vậy, khi chúng ta cần tạo một đối tượng, chỉ cần cấu hình xong file cấu hình/annotation là được, hoàn toàn không cần suy nghĩ đối tượng được tạo ra như thế nào.** IoC container chịu trách nhiệm tạo đối tượng, kết nối các đối tượng lại với nhau, cấu hình các đối tượng này, và xử lý toàn bộ vòng đời của các đối tượng này từ lúc tạo ra cho đến khi chúng bị hủy hoàn toàn.

Trong dự án thực tế nếu một Service class có hàng trăm thậm chí hàng ngàn class làm nền tảng bên dưới của nó, chúng ta cần khởi tạo Service này, bạn có thể mỗi lần đều phải hiểu rõ constructor của tất cả các class bên dưới của Service này, điều này có thể phát điên. Nếu tận dụng IoC, bạn chỉ cần cấu hình xong, rồi ở nơi cần dùng tham chiếu đến là được, điều này làm tăng đáng kể tính bảo trì của dự án và làm giảm độ khó phát triển.

> Về mức độ hiểu biết đối với Spring IoC, khuyến khích xem qua câu trả lời này trên Zhihu: <https://www.zhihu.com/question/23277575/answer/169698662> , rất tuyệt vời.

**Điều khiển đảo ngược hiểu thế nào?** Lấy một ví dụ: "Đối tượng a phụ thuộc vào đối tượng b, khi đối tượng a cần sử dụng đối tượng b thì bắt buộc phải tự mình đi tạo. Tuy nhiên khi hệ thống đưa vào IOC container, giữa đối tượng a và đối tượng b liền mất đi sự liên hệ trực tiếp. Lúc này, khi đối tượng a cần sử dụng đối tượng b, chúng ta có thể chỉ định IOC container đi tạo một đối tượng b inject vào đối tượng a". Quá trình đối tượng a có được đối tượng phụ thuộc b, từ hành vi chủ động chuyển thành hành vi bị động, quyền điều khiển bị đảo ngược, đây chính là nguồn gốc tên gọi của Điều khiển đảo ngược.

**DI (Dependency Injection, Tiêm phụ thuộc / Bơm phụ thuộc) là một design pattern thực hiện Điều khiển đảo ngược, Dependency Injection chính là truyền instance variable vào trong một object.**

## Factory Design Pattern

Spring sử dụng Factory Pattern có thể thông qua `BeanFactory` hoặc `ApplicationContext` để tạo object bean.

**So sánh cả hai:**

- `BeanFactory`: Cung cấp năng lực nền tảng của Spring IoC container. Khi sử dụng trực tiếp `BeanFactory` nền tảng, container thường không chủ động khởi tạo trước tất cả các singleton Bean, mà khi lần đầu tiên request Bean mới tạo.
- `ApplicationContext`: Mở rộng từ `BeanFactory`, thêm vào các năng lực như phát hành event, đa ngôn ngữ (internationalization), load tài nguyên,... và mặc định ở giai đoạn khởi động container sẽ khởi tạo trước các singleton Bean không lazy loading; `prototype` Bean và Bean được đánh dấu lazy sẽ không vì thế mà bị tạo toàn bộ cùng một lúc.

Ba class triển khai thường gặp của `ApplicationContext`:

1. `ClassPathXmlApplicationContext`: Xem file context như tài nguyên classpath.
2. `FileSystemXmlApplicationContext`: Nạp thông tin định nghĩa context từ file XML trong file system.
3. `XmlWebApplicationContext`: Nạp thông tin định nghĩa context từ file XML trong Web system.

Ví dụ:

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class App {
  public static void main(String[] args) {
    ApplicationContext context = new FileSystemXmlApplicationContext(
        "C:/work/IOC Containers/springframework.applicationcontext/src/main/resources/bean-factory-config.xml");

    HelloApplicationContext obj = (HelloApplicationContext) context.getBean("helloApplicationContext");
    obj.getMsg();
  }
}
```

## Singleton Design Pattern

Trong hệ thống của chúng ta, có một số object thực ra chúng ta chỉ cần một cái, ví dụ như: ThreadPool, cache, dialog, registry, object log, object đóng vai trò máy in, card màn hình và các thiết bị driver khác. Trên thực tế, loại object này chỉ có thể có một instance, nếu tạo ra nhiều instance có thể dẫn đến phát sinh một số vấn đề, ví dụ: hành vi chương trình bất thường, sử dụng tài nguyên quá mức, hoặc kết quả không nhất quán.

**Lợi ích của việc sử dụng Singleton Pattern**:

- Đối với các object sử dụng thường xuyên, có thể tiết kiệm thời gian tiêu tốn cho việc tạo object, điều này đối với những object hạng nặng mà nói, là một khoản chi phí hệ thống rất đáng kể;
- Vì số lần thao tác new giảm đi, do đó tần suất sử dụng memory hệ thống cũng sẽ giảm xuống, điều này sẽ giảm bớt áp lực GC, rút ngắn thời gian dừng (pause time) của GC.

**Scope mặc định của bean trong Spring chính là singleton (đơn vị đơn lẻ).** Ngoài scope singleton, bean trong Spring còn có các loại scope dưới đây:

- **prototype**: Mỗi lần lấy đều sẽ tạo một instance bean mới. Nghĩa là, gọi `getBean()` hai lần liên tiếp sẽ nhận được hai instance Bean khác nhau.
- **request** (chỉ dùng cho ứng dụng Web): Mỗi một request HTTP đều tạo ra một bean mới (request bean), bean này chỉ có hiệu lực trong request HTTP hiện tại.
- **session** (chỉ dùng cho ứng dụng Web): Mỗi một request HTTP đến từ session mới đều tạo ra một bean mới (session bean), bean này chỉ có hiệu lực trong session HTTP hiện tại.
- **application** (chỉ dùng cho ứng dụng Web): Mỗi `ServletContext` tương ứng với một instance Bean, bean này chỉ có hiệu lực trong vòng đời ứng dụng Web hiện tại. Phiên bản Spring cũ còn cung cấp scope `globalSession` độc lập cho ứng dụng Portlet, nó không thuộc về danh sách scope tiêu chuẩn hiện tại.
- **websocket** (chỉ dùng cho ứng dụng Web): Mỗi phiên WebSocket tạo ra một bean mới.

Spring thực hiện Singleton Pattern thông qua phương thức đặc thù của registry singleton bằng `ConcurrentHashMap`.

Mã nguồn cốt lõi Spring thực hiện singleton như sau:

```java
// Thực hiện registry singleton thông qua ConcurrentHashMap (thread-safe)
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // Kiểm tra trong cache có tồn tại instance hay không
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                //... Đã bỏ qua rất nhiều code
                try {
                    singletonObject = singletonFactory.getObject();
                }
                //... Đã bỏ qua rất nhiều code
                // Nếu đối tượng instance không tồn tại, chúng ta đăng ký vào registry singleton.
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    // Thêm đối tượng vào registry singleton
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```

**Singleton Bean có tồn tại vấn đề thread-safe không?**

Hầu hết thời gian chúng ta không sử dụng đa luồng trong dự án, nên rất ít người chú ý đến vấn đề này. Singleton Bean tồn tại vấn đề thread-safe, chủ yếu là vì khi nhiều thread thao tác trên cùng một object thì có sự tranh chấp tài nguyên.

Có hai cách giải quyết thường gặp:

1. Trong Bean cố gắng tránh định nghĩa các biến thành viên có thể thay đổi.
2. Trong class định nghĩa một biến thành viên `ThreadLocal`, lưu trữ biến thành viên có thể thay đổi cần thiết vào trong `ThreadLocal` (một cách khuyên dùng).

Tuy nhiên, phần lớn các Bean thực tế đều không có state (không có biến instance) (như Dao, Service), trong trường hợp này, Bean là thread-safe.

## Proxy Design Pattern

### Ứng dụng của Proxy Pattern trong AOP

**AOP (Aspect-Oriented Programming, Lập trình hướng khía cạnh)** có thể đóng gói những logic hoặc trách nhiệm không liên quan đến nghiệp vụ nhưng lại được các module nghiệp vụ cùng gọi (như xử lý transaction, quản lý log, kiểm soát phân quyền,...), giúp giảm code trùng lặp trong hệ thống, giảm độ gắn kết giữa các module, và có lợi cho khả năng mở rộng cũng như bảo trì trong tương lai.

Spring AOP dựa trên Dynamic Proxy. Nếu object cần proxy implement một interface nào đó, Spring AOP sẽ sử dụng **JDK Proxy** để tạo object proxy; còn đối với object không implement interface, sẽ không thể sử dụng JDK Proxy để proxy nữa, lúc này Spring AOP sẽ sử dụng **Cglib** để sinh ra một class con của object được proxy làm proxy, như hình dưới đây:

![Quy trình Spring AOP](https://oss.javaguide.cn/github/javaguide/SpringAOPProcess.jpg)

Tất nhiên, bạn cũng có thể sử dụng AspectJ, Spring AOP đã tích hợp AspectJ, AspectJ được coi là AOP framework hoàn chỉnh nhất trong hệ sinh thái Java.

Sau khi sử dụng AOP, chúng ta có thể trừu tượng hóa một số tính năng dùng chung ra, ở nơi cần dùng trực tiếp sử dụng là được, như vậy rút gọn đáng kể lượng code. Khi chúng ta cần thêm tính năng mới cũng tiện lợi hơn, như vậy cũng nâng cao tính mở rộng của hệ thống. Tính năng log, quản lý transaction,... trong rất nhiều kịch bản đều sử dụng AOP.

### Sự khác biệt giữa Spring AOP và AspectJ AOP là gì?

**Spring AOP thuộc về tăng cường lúc runtime, còn AspectJ là tăng cường lúc compile.** Spring AOP dựa trên Proxying, còn AspectJ dựa trên Bytecode Manipulation.

Spring AOP đã tích hợp AspectJ, AspectJ được coi là AOP framework hoàn chỉnh nhất trong hệ sinh thái Java. AspectJ so với Spring AOP thì tính năng mạnh mẽ hơn, nhưng Spring AOP tương đối mà nói thì đơn giản hơn.

Nếu aspect của chúng ta tương đối ít, thì hiệu năng của cả hai không khác biệt nhiều. Tuy nhiên, khi có quá nhiều aspect, tốt nhất nên chọn AspectJ, nó nhanh hơn Spring AOP rất nhiều.

## Template Method Pattern

Template Method Pattern là một behavioral design pattern, nó định nghĩa khung xương của một thuật toán trong một thao tác, và trì hoãn một số bước sang class con. Template Method cho phép các class con không làm thay đổi cấu trúc của một thuật toán mà vẫn có thể định nghĩa lại cách thức thực hiện các bước cụ thể nào đó của thuật toán đó.

```java
public abstract class Template {
    // Đây là template method của chúng ta
    public final void TemplateMethod(){
        PrimitiveOperation1();
        PrimitiveOperation2();
        PrimitiveOperation3();
    }

    protected void  PrimitiveOperation1(){
        // Class hiện tại thực hiện
    }

    // Method do class con thực hiện
    protected abstract void PrimitiveOperation2();
    protected abstract void PrimitiveOperation3();

}
public class TemplateImpl extends Template {

    @Override
    public void PrimitiveOperation2() {
        // Class hiện tại thực hiện
    }

    @Override
    public void PrimitiveOperation3() {
        // Class hiện tại thực hiện
    }
}

```

Các class thao tác cơ sở dữ liệu kết thúc bằng Template trong Spring như `JdbcTemplate`, `HibernateTemplate`,... đều sử dụng Template Pattern. Trong trường hợp thông thường, chúng ta đều sử dụng kế thừa để thực hiện Template Pattern, nhưng Spring không sử dụng cách này, mà sử dụng Callback Pattern phối hợp với Template Method Pattern, vừa đạt được hiệu quả tái sử dụng code, vừa tăng thêm tính linh hoạt.

## Observer Pattern

Observer Pattern là một behavioral pattern của object. Nó thể hiện một mối quan hệ phụ thuộc giữa object với object, khi một object thay đổi, tất cả các object phụ thuộc vào object này cũng sẽ đưa ra phản ứng. Mô hình xử lý event (event-driven) trong Spring là một ứng dụng rất kinh điển của Observer Pattern. Mô hình xử lý event trong Spring rất hữu ích, trong nhiều kịch bản đều có thể giảm độ gắn kết cho code của chúng ta. Ví dụ mỗi lần chúng ta thêm sản phẩm đều cần cập nhật lại index sản phẩm, lúc này có thể tận dụng Observer Pattern để giải quyết vấn đề này.

### Ba vai trò trong Mô hình Xử lý Event của Spring

#### Vai trò Event

`ApplicationEvent` (dưới package `org.springframework.context`) đóng vai trò là Event, đây là một abstract class kế thừa `java.util.EventObject` và implement interface `java.io.Serializable`.

Trong Spring mặc định tồn tại các event dưới đây, chúng đều là sự triển khai của `ApplicationContextEvent` (kế thừa từ `ApplicationContextEvent`):

- `ContextStartedEvent`: Event kích hoạt sau khi `ApplicationContext` khởi động;
- `ContextStoppedEvent`: Event kích hoạt sau khi `ApplicationContext` dừng;
- `ContextRefreshedEvent`: Event kích hoạt sau khi `ApplicationContext` khởi tạo hoặc refresh xong;
- `ContextClosedEvent`: Event kích hoạt sau khi `ApplicationContext` đóng.

![Class con của ApplicationEvent](https://oss.javaguide.cn/github/javaguide/ApplicationEvent-Subclass.png)

#### Vai trò Event Listener

`ApplicationListener` đóng vai trò là Event Listener, nó là một interface, bên trong chỉ định nghĩa một method `onApplicationEvent()` để xử lý `ApplicationEvent`. Mã nguồn interface `ApplicationListener` như sau, có thể thấy định nghĩa interface cho thấy event trong interface chỉ cần implement `ApplicationEvent` là được. Do đó, trong Spring chúng ta chỉ cần implement method `onApplicationEvent()` của interface `ApplicationListener` là hoàn thành lắng nghe event.

```java
package org.springframework.context;
import java.util.EventListener;
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E var1);
}
```

#### Vai trò Event Publisher

`ApplicationEventPublisher` đóng vai trò là Event Publisher, nó cũng là một interface.

```java
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        this.publishEvent((Object)event);
    }

    void publishEvent(Object var1);
}

```

Method `publishEvent()` của interface `ApplicationEventPublisher` được triển khai trong class `AbstractApplicationContext`, đọc phần triển khai method này, bạn sẽ phát hiện thực chất event được phát sóng ra ngoài thông qua `ApplicationEventMulticaster`. Nội dung cụ thể quá nhiều, không phân tích ở đây, sau này có thể sẽ viết riêng một bài viết đề cập đến.

### Tóm tắt Quy trình Event của Spring

1. Định nghĩa một Event: Thực hiện một class kế thừa từ `ApplicationEvent`, và viết constructor tương ứng;
2. Định nghĩa một Event Listener: Implement interface `ApplicationListener`, override method `onApplicationEvent()`;
3. Sử dụng Event Publisher để phát tin nhắn: Có thể thông qua method `publishEvent()` của `ApplicationEventPublisher` để phát tin nhắn.

Ví dụ:

```java
// Định nghĩa một event, kế thừa từ ApplicationEvent và viết constructor tương ứng
public class DemoEvent extends ApplicationEvent{
    private static final long serialVersionUID = 1L;

    private String message;

    public DemoEvent(Object source,String message){
        super(source);
        this.message = message;
    }

    public String getMessage() {
         return message;
          }


// Định nghĩa một event listener, implement interface ApplicationListener, override method onApplicationEvent();
@Component
public class DemoListener implements ApplicationListener<DemoEvent>{

    // Sử dụng onApplicationEvent để nhận tin nhắn
    @Override
    public void onApplicationEvent(DemoEvent event) {
        String msg = event.getMessage();
        System.out.println("Thông tin nhận được là: "+msg);
    }

}
// Phát event, có thể thông qua method publishEvent() của ApplicationEventPublisher để phát tin nhắn.
@Component
public class DemoPublisher {

    @Autowired
    ApplicationContext applicationContext;

    public void publish(String message){
        // Phát event
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}

```

Khi gọi method `publish()` của `DemoPublisher`, ví dụ `demoPublisher.publish("Xin chào")`, console sẽ in ra: `Thông tin nhận được là: Xin chào`.

## Adapter Pattern

Adapter Pattern (Mô hình thích ứng) chuyển đổi một interface thành một interface khác mà client mong muốn, Adapter Pattern làm cho những class có interface không tương thích có thể làm việc cùng nhau.

### Adapter Pattern trong Spring AOP

Chúng ta biết triển khai của Spring AOP dựa trên Proxy Pattern, nhưng tăng cường hoặc advice của Spring AOP lại sử dụng Adapter Pattern, interface liên quan đến nó là `AdvisorAdapter`.

Các loại Advice thường dùng bao gồm: `BeforeAdvice` (trước cuộc gọi method mục tiêu, advice trước), `AfterAdvice` (sau cuộc gọi method mục tiêu, advice sau), `AfterReturningAdvice` (sau khi method mục tiêu thực thi xong, trước khi return),... Mỗi loại Advice (thông báo) đều có interceptor tương ứng: `MethodBeforeAdviceInterceptor`, `AfterReturningAdviceInterceptor`, `ThrowsAdviceInterceptor`,...

Advice định nghĩa sẵn trong Spring phải thông qua adapter tương ứng, thích ứng thành đối tượng kiểu interface `MethodInterceptor` (method interceptor) (như `MethodBeforeAdviceAdapter` thông qua gọi method `getInterceptor`, thích ứng `MethodBeforeAdvice` thành `MethodBeforeAdviceInterceptor`).

### Adapter Pattern trong Spring MVC

Trong Spring MVC, `DispatcherServlet` dựa theo thông tin request gọi `HandlerMapping`, giải mã `Handler` tương ứng với request. Sau khi giải mã ra `Handler` tương ứng (tức là bộ điều khiển `Controller` mà chúng ta thường nói), bắt đầu do adapter `HandlerAdapter` xử lý. `HandlerAdapter` đóng vai trò interface kỳ vọng, class triển khai adapter cụ thể dùng để tiến hành thích ứng cho class mục tiêu, `Controller` đóng vai trò class cần thích ứng.

**Tại sao lại sử dụng Adapter Pattern trong Spring MVC?**

`Controller` trong Spring MVC có rất nhiều loại, các loại `Controller` khác nhau thông qua các method khác nhau để tiến hành xử lý request. Nếu không tận dụng Adapter Pattern, `DispatcherServlet` trực tiếp lấy `Controller` kiểu tương ứng, cần tự mình đứng ra phán đoán, giống như đoạn code dưới đây:

```java
if(mappedHandler.getHandler() instanceof MultiActionController){
   ((MultiActionController)mappedHandler.getHandler()).xxx
}else if(mappedHandler.getHandler() instanceof XXX){
    ...
}else if(...){
   ...
}
```

Giả sử chúng ta thêm một loại `Controller` nữa thì lại phải thêm một dòng lệnh phán đoán vào đoạn code trên, hình thức này khiến chương trình khó bảo trì, cũng vi phạm nguyên tắc Open-Closed (Mở để mở rộng, Đóng để sửa đổi) trong design pattern.

## Decorator Pattern

Decorator Pattern (Mô hình trang trí) có thể thêm động một số thuộc tính hoặc hành vi phụ cho object. So với việc sử dụng kế thừa, Decorator Pattern linh hoạt hơn. Nói một cách đơn giản chính là khi chúng ta cần sửa đổi tính năng vốn có, nhưng chúng ta lại không muốn trực tiếp đi sửa code vốn có, liền thiết kế một Decorator bọc bên ngoài code vốn có. Thực ra trong JDK có rất nhiều nơi sử dụng Decorator Pattern, ví dụ họ hàng `InputStream`, dưới class `InputStream` có `FileInputStream` (đọc file), `BufferedInputStream` (thêm cache, giúp tốc độ đọc file tăng lên đáng kể),... các class con này đều mở rộng tính năng của `InputStream` mà không cần sửa code của nó.

![Sơ đồ minh họa Decorator Pattern](https://oss.javaguide.cn/github/javaguide/Decorator.jpg)

## Tóm tắt

Những design pattern nào được sử dụng trong Spring Framework?

- **Factory Pattern**: Spring sử dụng Factory Pattern thông qua `BeanFactory`, `ApplicationContext` để tạo object bean.
- **Proxy Pattern**: Thực hiện tính năng Spring AOP.
- **Singleton Pattern**: Các Bean trong Spring mặc định đều là singleton.
- **Template Method Pattern**: Các class thao tác cơ sở dữ liệu kết thúc bằng Template trong Spring như `jdbcTemplate`, `hibernateTemplate`,... đều sử dụng Template Pattern.
- **Observer Pattern**: Mô hình xử lý event trong Spring là một ứng dụng rất kinh điển của Observer Pattern.
- **Adapter Pattern**: Tăng cường hoặc advice của Spring AOP sử dụng Adapter Pattern, trong Spring MVC cũng sử dụng Adapter Pattern để thích ứng `Controller`.
- ……

## Tham khảo

- 《Thiết kế bên trong kỹ thuật Spring (Spring Technical Inner Principles)》
- <https://blog.eduonix.com/java-programming-2/learn-design-patterns-used-spring-framework/>
- <https://www.tutorialsteacher.com/ioc/inversion-of-control>
- <https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/observer.html>
- <https://juejin.im/post/5a8eb261f265da4e9e307230>
- <https://juejin.im/post/5ba28986f265da0abc2b6084>

<!-- @include: @article-footer.snippet.md -->
