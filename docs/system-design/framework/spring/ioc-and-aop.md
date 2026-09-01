---
title: IoC & AOP详解（快速搞懂）
description: Spring IoC与AOP核心原理详解，深入讲解控制反转、依赖注入、切面编程及动态代理的实现机制。
category: 框架
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: IoC,DI,AOP,Spring IoC容器,依赖注入,切面编程,动态代理,Spring原理
---

Bài viết này sẽ triển khai giải thích về IoC & AOP từ các câu hỏi dưới đây:

- IoC là gì?
- IoC giải quyết vấn đề gì?
- Sự khác biệt giữa IoC và DI?
- AOP là gì?
- AOP giải quyết vấn đề gì?
- Các kịch bản ứng dụng của AOP là gì?
- Tại sao AOP gọi là Lập trình hướng khía cạnh?
- Các phương thức thực hiện AOP là gì?

Đầu tiên xin tuyên bố: IoC & AOP không phải do Spring đưa ra, trước Spring chúng thực ra đã tồn tại rồi, có điều lúc đó thiên về lý thuyết hơn. Spring đã thực hiện rất tốt hai tư tưởng này ở cấp độ kỹ thuật.

## IoC (Inversion of Control)

### IoC là gì?

IoC (Inversion of Control) tức Điều khiển đảo ngược / Đảo ngược điều khiển. Nó là một tư tưởng chứ không phải một triển khai kỹ thuật. Nó mô tả vấn đề tạo và quản lý object trong lĩnh vực phát triển Java.

Ví dụ: Hiện tại class A phụ thuộc vào class B

- **Phương thức phát triển truyền thống**: Thường là trong class A thủ công dùng từ khóa new để new một object B ra
- **Phương thức phát triển sử dụng tư tưởng IoC**: Không thông qua từ khóa new để tạo object, mà thông qua IoC container (Spring Framework) để giúp chúng ta khởi tạo object. Chúng ta cần object nào, trực tiếp lấy từ IoC container ra là được.

Tương quan so sánh giữa hai phương thức phát triển trên: Chúng ta "mất đi một quyền hạn" (quyền hạn tạo, quản lý object), từ đó cũng nhận được một lợi ích (không cần phải suy nghĩ về một loạt các việc như tạo, quản lý object nữa)

**Tại sao gọi là Điều khiển đảo ngược?**

- **Điều khiển**: Chỉ quyền hạn tạo (khởi tạo, quản lý) object
- **Đảo ngược**: Quyền điều khiển được giao cho môi trường bên ngoài (IoC container)

![Minh họa IoC](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration.png)

### IoC giải quyết vấn đề gì?

Tư tưởng của IoC chính là hai bên không phụ thuộc lẫn nhau, mà do container bên thứ ba quản lý các tài nguyên liên quan. Như vậy có lợi ích gì?

1. Độ phụ thuộc hay mức độ phụ thuộc giữa các object giảm xuống;
2. Tài nguyên trở nên dễ quản lý hơn; ví dụ như bạn dùng Spring container cung cấp thì rất dễ dàng có thể thực hiện một Singleton.

Ví dụ: Hiện có một thao tác đối với User, sử dụng cấu trúc hai tầng Service và Dao để phát triển

Trong trường hợp không sử dụng tư tưởng IoC, nếu tầng Service muốn sử dụng triển khai cụ thể của tầng Dao, cần thông qua từ khóa new trong `UserServiceImpl` để new thủ công class triển khai cụ thể `UserDaoImpl` của `IUserDao` (không thể new trực tiếp class interface).

Rất hoàn hảo, phương thức này cũng có thể thực hiện được, nhưng chúng ta hãy tưởng tượng kịch bản sau:

Trong quá trình phát triển đột nhiên nhận được một yêu cầu mới, phát triển một class triển khai cụ thể khác cho interface `IUserDao`. Vì tầng Service phụ thuộc vào triển khai cụ thể của `IUserDao`, nên chúng ta cần sửa object new trong `UserServiceImpl`. Nếu chỉ có một class tham chiếu đến triển khai cụ thể của `IUserDao`, có thể cảm thấy không sao, sửa lại cũng không tốn nhiều sức, nhưng nếu có rất nhiều nơi đều tham chiếu đến triển khai cụ thể của `IUserDao`, một khi cần thay đổi phương thức triển khai của `IUserDao`, việc sửa đổi đó sẽ vô cùng đau đầu.

![IoC&Aop-ioc-illustration-dao-service](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao-service.png)

Sử dụng tư tưởng IoC, chúng ta giao quyền điều khiển object (tạo, quản lý) cho IoC container quản lý, khi sử dụng chúng ta trực tiếp "hỏi xin" IoC container là được.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/IoC&Aop-ioc-illustration-dao.png)

### IoC và DI có khác nhau không?

IoC (Inversion of Control: Điều khiển đảo ngược) là một tư tưởng thiết kế hay nói là một loại mô hình (pattern). Tư tưởng thiết kế này chính là **giao quyền điều khiển vốn tự tay tạo object trong chương trình cho bên thứ ba như IoC container.** Đối với Spring Framework mà chúng ta thường dùng, IoC container thực chất là một Map (key, value), trong Map lưu trữ các loại object. Tuy nhiên, IoC cũng có ứng dụng trong các ngôn ngữ khác, không phải chỉ riêng Spring.

Phương thức thực hiện phổ biến nhất và hợp lý nhất của IoC gọi là Dependency Injection (Tiêm phụ thuộc / Bơm phụ thuộc, viết tắt là DI).

Bác Mã (Martin Fowler) trong một bài viết từng đề xuất đổi tên IoC thành DI, đoạn văn gốc như sau, địa chỉ bài gốc: <https://martinfowler.com/articles/injection.html> .

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/martin-fowler-injection.png)

Ý chính của Bác Mã là IoC quá phổ biến và không thể hiện rõ ý nghĩa, nhiều người vì thế mà nhầm lẫn, do đó, sử dụng DI để chỉ ra chính xác mô hình này thì tốt hơn.

## AOP (Aspect Oriented Programming)

Ở đây sẽ không liên quan quá nhiều thuật ngữ chuyên ngành, mục tiêu cốt lõi là làm rõ tư tưởng AOP.

### AOP là gì?

AOP (Aspect Oriented Programming) tức Lập trình hướng khía cạnh, AOP là sự nối tiếp của OOP (Lập trình hướng đối tượng), hai cái bổ sung cho nhau, chứ không đối lập nhau.

Mục tiêu của AOP là tách các mối quan tâm cắt ngang (cross-cutting concerns, như ghi log, quản lý transaction, kiểm soát phân quyền, giới hạn lưu lượng interface, tính idempotence của interface,...) ra khỏi logic nghiệp vụ cốt lõi, thông qua các kỹ thuật như Dynamic Proxy, thao tác bytecode,... để đạt được việc tái sử dụng code và giảm độ gắn kết (decoupling), nâng cao tính bảo trì và tính mở rộng của code. Mục tiêu của OOP là đóng gói logic nghiệp vụ theo các thuộc tính và hành vi của đối tượng, thông qua các khái niệm class, object, inheritance, polymorphism,... để đạt được tính module hóa và phân tầng của code (cũng có thể đạt được việc tái sử dụng code), nâng cao tính đọc hiểu và tính bảo trì của code.

### Tại sao AOP gọi là Lập trình hướng khía cạnh?

Sở dĩ AOP gọi là Lập trình hướng khía cạnh là vì tư tưởng cốt lõi của nó là tách các mối quan tâm cắt ngang ra khỏi logic nghiệp vụ cốt lõi, hình thành nên các **Aspect (Khía cạnh)**.

![Minh họa Lập trình hướng khía cạnh](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/aop-program-execution.jpg)

Ở đây nhân tiện tổng kết các thuật ngữ then chốt của AOP (chưa hiểu cũng không sao, có thể tiếp tục đọc xuống dưới):

- **Cross-cutting concerns (Mối quan tâm cắt ngang)**: Hành vi chung trong nhiều class hoặc object (như ghi log, quản lý transaction, kiểm soát phân quyền, giới hạn lưu lượng interface, tính idempotence của interface,...).
- **Aspect (Khía cạnh)**: Class đóng gói các mối quan tâm cắt ngang, một aspect là một class. Aspect có thể định nghĩa nhiều advice, dùng để thực hiện tính năng cụ thể.
- **JoinPoint (Điểm nối)**: JoinPoint là một thời điểm cụ thể khi calling method hoặc executing method (như gọi method, ném exception,...).
- **Advice (Thông báo/Tăng cường)**: Advice chính là thao tác mà aspect cần thực hiện tại một JoinPoint nào đó. Advice có 5 loại, lần lượt là Before advice (trước), After advice (sau), AfterReturning advice (trả về), AfterThrowing advice (ngoại lệ) và Around advice (bao quanh). 4 loại advice đầu đều thực thi ở trước/sau method mục tiêu, còn Around advice có thể kiểm soát quá trình thực thi của method mục tiêu.
- **Pointcut (Điểm cắt)**: Một Pointcut là một biểu thức, nó dùng để khớp xem những JoinPoint nào cần được aspect tăng cường. Pointcut có thể được định nghĩa thông qua annotation, regex, phép toán logic,... Ví dụ `execution(* com.xyz.service..*(..))` khớp với các class hoặc interface dưới package `com.xyz.service` và các sub-package của nó.
- **Weaving (Dệt/Kết hợp)**: Weaving là quá trình kết nối aspect và target object lại với nhau, tức là áp dụng advice vào các JoinPoint mà Pointcut khớp được. Các thời điểm weaving thường gặp có hai loại, lần lượt là Compile-Time Weaving (như AspectJ) và Runtime Weaving (như AspectJ, Spring AOP).

### Các loại Advice thường gặp trong AOP là gì?

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/aspectj-advice-types.jpg)

- **Before** (Advice trước): Kích hoạt trước khi method của target object được gọi
- **After** (Advice sau): Kích hoạt sau khi method của target object được gọi
- **AfterReturning** (Advice trả về): Kích hoạt sau khi method của target object gọi xong, sau khi trả về giá trị kết quả
- **AfterThrowing** (Advice ngoại lệ): Kích hoạt sau khi method của target object đang chạy ném ra / kích hoạt exception. AfterReturning và AfterThrowing loại trừ lẫn nhau. Nếu method gọi thành công không có exception thì sẽ có giá trị trả về; nếu method ném ra exception thì sẽ không có giá trị trả về.
- **Around** (Advice bao quanh): Kiểm soát bằng lập trình đối với việc gọi method của target object. Around advice là loại advice có phạm vi thao tác lớn nhất trong tất cả các loại advice, vì nó có thể lấy trực tiếp target object cũng như method sắp thực thi, nên around advice có thể tùy ý thực hiện công việc trước và sau khi gọi method của target object, thậm chí không gọi method của target object

### AOP giải quyết vấn đề gì?

OOP không thể xử lý tốt một số hành vi chung phân tán trong nhiều class hoặc object (như ghi log, quản lý transaction, kiểm soát phân quyền, giới hạn lưu lượng interface, tính idempotence của interface,...), những hành vi này thường được gọi là **Cross-cutting concerns (Mối quan tâm cắt ngang)**. Nếu chúng ta trong mỗi class hoặc object đều lặp lại thực hiện các hành vi này, thì sẽ dẫn đến code bị dư thừa, phức tạp và khó bảo trì.

AOP có thể tách các mối quan tâm cắt ngang (như ghi log, quản lý transaction, kiểm soát phân quyền, giới hạn lưu lượng interface, tính idempotence của interface,...) ra khỏi **Core concerns (Mối quan tâm cốt lõi, tức logic nghiệp vụ cốt lõi)**, đạt được việc phân tách các mối quan tâm.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/crosscut-logic-and-businesslogic-separation%20%20%20%20%20%20.png)

Lấy ghi log làm ví dụ để giới thiệu, giả sử chúng ta cần tiến hành ghi log theo định dạng thống nhất đối với một số method, trước khi chưa sử dụng kỹ thuật AOP, chúng ta cần viết từng đoạn code logic ghi log, toàn là logic trùng lặp.

```java
public CommonResponse<Object> method1() {
      // Logic nghiệp vụ
      xxService.method1();
      // Bỏ qua logic xử lý nghiệp vụ cụ thể
      // Ghi log
      ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = attributes.getRequest();
      // Bỏ qua logic cụ thể của việc ghi log như: lấy các thông tin, ghi vào database...
      return CommonResponse.success();
}

public CommonResponse<Object> method2() {
      // Logic nghiệp vụ
      xxService.method2();
      // Bỏ qua logic xử lý nghiệp vụ cụ thể
      // Ghi log
      ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = attributes.getRequest();
      // Bỏ qua logic cụ thể của việc ghi log như: lấy các thông tin, ghi vào database...
      return CommonResponse.success();
}

// ...
```

Sau khi sử dụng kỹ thuật AOP, chúng ta có thể đóng gói logic ghi log thành một Aspect, sau đó thông qua Pointcut và Advice để chỉ định trong những method nào cần thực thi thao tác ghi log.

```java

// Annotation Log
@Target({ElementType.PARAMETER,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Log {

    /**
     * Mô tả
     */
    String description() default "";

    /**
     * Loai method INSERT DELETE UPDATE OTHER
     */
    MethodType methodType() default MethodType.OTHER;
}

// Aspect Log
@Component
@Aspect
public class LogAspect {
  // Pointcut, tất cả các method được đánh dấu bởi annotation Log
  @Pointcut("@annotation(cn.javaguide.annotation.Log)")
  public void webLog() {
  }

   /**
   * Around advice
   */
  @Around("webLog()")
  public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
    // Bỏ qua logic xử lý cụ thể
  }

  // Bỏ qua code khác
}
```

Như thế này, chúng ta chỉ cần một dòng annotation là có thể thực hiện ghi log:

```java
@Log(description = "method1",methodType = MethodType.INSERT)
public CommonResponse<Object> method1() {
      // Logic nghiệp vụ
      xxService.method1();
      // Bỏ qua logic xử lý nghiệp vụ cụ thể
      return CommonResponse.success();
}
```

### Các kịch bản ứng dụng của AOP là gì?

- Ghi log: Tùy chỉnh annotation ghi log, tận dụng AOP, chỉ một dòng code là có thể thực hiện ghi log.
- Thống kê hiệu năng: Tận dụng AOP ở trước và sau khi thực thi method mục tiêu để thống kê thời gian thực thi của method, tiện cho việc tối ưu và phân tích.
- Quản lý transaction: Annotation `@Transactional` có thể cho phép Spring thực hiện quản lý transaction giúp chúng ta như rollback thao tác exception, tránh được logic quản lý transaction trùng lặp. Annotation `@Transactional` chính là được thực hiện dựa trên AOP.
- Kiểm soát phân quyền: Tận dụng AOP trước khi thực thi method mục tiêu để phán đoán xem user có đủ quyền hạn cần thiết hay không, nếu có thì thực thi method mục tiêu, nếu không thì không thực thi. Ví dụ, SpringSecurity tận dụng annotation `@PreAuthorize` một dòng code là có thể tùy chỉnh kiểm tra phân quyền.
- Giới hạn lưu lượng interface (Rate Limiting): Tận dụng AOP trước khi thực thi method mục tiêu thông qua thuật toán và triển khai giới hạn lưu lượng cụ thể để xử lý giới hạn request.
- Quản lý cache: Tận dụng AOP trước và sau khi thực thi method mục tiêu để tiến hành đọc và update cache.
- ……

### Các phương thức thực hiện AOP là gì?

Các phương thức thực hiện AOP thường gặp có Dynamic Proxy, thao tác bytecode,...

Spring AOP chính là dựa trên Dynamic Proxy. Nếu object cần proxy implement một interface nào đó, Spring AOP sẽ sử dụng **JDK Proxy** để tạo object proxy, còn đối với object không implement interface, sẽ không thể sử dụng JDK Proxy để proxy nữa, lúc này Spring AOP sẽ sử dụng CGLIB để sinh ra một class con của object được proxy làm proxy, như hình dưới đây:

![Quy trình Spring AOP](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)

**Chiến lược Dynamic Proxy của Spring Boot và Spring có giống nhau không?** Thực ra không giống nhau, rất nhiều người đều hiểu sai.

Trước Spring Boot 2.0, giá trị mặc định của `spring.aop.proxy-target-class` là `false`, khi có interface thường sử dụng **JDK Dynamic Proxy**; nếu class mục tiêu không có interface khả dụng, Spring AOP vẫn sẽ fallback về **CGLIB Dynamic Proxy**, chứ không chỉ vì class mục tiêu không implement interface mà ném ra exception. Code tự động cấu hình AOP trong Spring Boot 1.5.x như sau:

```java
@Configuration
@ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class })
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = false)
 // Class configuration này chỉ khi spring.aop.proxy-target-class=false hoặc không cấu hình tường minh mới có hiệu lực.
 // Nghĩa là, nếu lập trình viên không chọn rõ ràng phương thức proxy, Spring sẽ mặc định load JDK Dynamic Proxy.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = true)
	public static class JdkDynamicAutoProxyConfiguration {

	}

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = true)
 // Class configuration này chỉ khi spring.aop.proxy-target-class=true mới có hiệu lực.
 // Tức là khi lập trình viên chỉ định rõ ràng sử dụng CGLIB Dynamic Proxy thông qua cấu hình thuộc tính, Spring sẽ load class configuration này.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = false)
	public static class CglibAutoProxyConfiguration {

	}

}
```

Từ Spring Boot 2.0 trở đi, nếu người dùng không cấu hình gì cả, mặc định sử dụng **CGLIB Dynamic Proxy**. Nếu cần ép buộc sử dụng JDK Dynamic Proxy, có thể thêm vào file cấu hình: `spring.aop.proxy-target-class=false`. Code tự động cấu hình AOP trong Spring Boot 2.0 như sau:

```java
@Configuration
@ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class,
		AnnotatedElement.class })
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = false)
 // Class configuration này chỉ khi spring.aop.proxy-target-class=false mới có hiệu lực.
 // Tức là khi lập trình viên chỉ định rõ ràng sử dụng JDK Dynamic Proxy thông qua cấu hình thuộc tính, Spring sẽ load class configuration này.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)
	public static class JdkDynamicAutoProxyConfiguration {

	}

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = true)
 // Class configuration này chỉ khi spring.aop.proxy-target-class=true hoặc không cấu hình tường minh mới có hiệu lực.
 // Nghĩa là, nếu lập trình viên không chọn rõ ràng phương thức proxy, Spring sẽ mặc định load CGLIB proxy.
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)
	public static class CglibAutoProxyConfiguration {

	}

}
```

Tất nhiên bạn cũng có thể sử dụng **AspectJ**! Spring AOP đã tích hợp AspectJ, AspectJ được coi là AOP framework hoàn chỉnh nhất trong hệ sinh thái Java.

**Spring AOP thuộc về tăng cường lúc runtime, AspectJ hỗ trợ weaving lúc compile, sau compile cũng như khi load class.** Spring AOP dựa trên Proxying, còn AspectJ dựa trên Bytecode Manipulation.

Spring AOP đã tích hợp AspectJ, AspectJ được coi là AOP framework hoàn chỉnh nhất trong hệ sinh thái Java. AspectJ so với Spring AOP thì tính năng mạnh mẽ hơn, nhưng Spring AOP tương đối mà nói thì đơn giản hơn.

Nếu aspect của chúng ta tương đối ít, thì hiệu năng của cả hai không khác biệt nhiều. Tuy nhiên, khi có quá nhiều aspect, tốt nhất nên chọn AspectJ, nó nhanh hơn Spring AOP rất nhiều.

## Tham khảo

- AOP in Spring Boot, is it a JDK dynamic proxy or a Cglib dynamic proxy?: <https://www.springcloud.io/post/2022-01/springboot-aop/>
- Spring Proxying Mechanisms: <https://docs.spring.io/spring-framework/reference/core/aop/proxying.html>
