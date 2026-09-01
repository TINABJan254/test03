---
title: SpringBoot 自动装配原理详解
description: SpringBoot自动装配原理深度解析，详解@EnableAutoConfiguration、SpringFactories加载机制及条件注解工作原理。
category: 框架
tag:
  - SpringBoot
head:
  - - meta
    - name: keywords
      content: Spring Boot自动装配,AutoConfiguration,EnableAutoConfiguration,SpringFactories,条件注解,Starter,Spring Boot原理
---

> Tác giả: [Miki-byte-1024](https://github.com/Miki-byte-1024) & [Snailclimb](https://github.com/Snailclimb)

Mỗi lần hỏi đến Spring Boot, người phỏng vấn rất thích hỏi câu hỏi này: "Hãy trình bày về nguyên lý tự động cấu hình (auto-assembly / auto-configuration) của SpringBoot?".

Tôi nghĩ chúng ta có thể trả lời từ các khía cạnh dưới đây:

1. SpringBoot tự động cấu hình là gì?
2. SpringBoot thực hiện tự động cấu hình như thế nào? Làm thế nào để đạt được việc load theo nhu cầu (on-demand loading)?
3. Làm thế nào để thực hiện một Starter?

Do giới hạn độ dài bài viết, bài viết này không đi quá sâu, các bạn cũng có thể sử dụng trực tiếp phương thức debug để xem mã nguồn phần tự động cấu hình của SpringBoot.

## Lời nói đầu

Những bạn từng sử dụng Spring, chắc chắn đều có nỗi sợ bị chi phối bởi cấu hình XML. Cho dù Spring về sau đã giới thiệu cấu hình dựa trên annotation, khi chúng ta bật một số tính năng của Spring hoặc đưa vào dependency của bên thứ ba, chúng ta vẫn cần sử dụng XML hoặc Java để tiến hành cấu hình tường minh.

Lấy một ví dụ. Khi chưa có Spring Boot, chúng ta viết một dịch vụ RESTful Web, còn đầu tiên cần tiến hành cấu hình như sau.

```java
@Configuration
public class RESTConfiguration
{
    @Bean
    public View jsonTemplate() {
        MappingJackson2JsonView view = new MappingJackson2JsonView();
        view.setPrettyPrint(true);
        return view;
    }

    @Bean
    public ViewResolver viewResolver() {
        return new BeanNameViewResolver();
    }
}
```

`spring-servlet.xml`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context/ http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/mvc/ http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.howtodoinjava.demo" />
    <mvc:annotation-driven />

    <!-- JSON Support -->
    <bean name="viewResolver" class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
    <bean name="jsonTemplate" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>

</beans>
```

Tuy nhiên, với dự án Spring Boot, chúng ta chỉ cần thêm các dependency liên quan, không cần cấu hình, thông qua khởi động method `main` dưới đây là được.

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

Hơn nữa, chúng ta thông qua file cấu hình toàn cục `application.properties` hoặc `application.yml` của Spring Boot là có thể tiến hành thiết lập cho dự án ví dụ như thay đổi port, cấu hình thuộc tính JPA,...

**Tại sao Spring Boot sử dụng lại sảng khoái như vậy?** Điều này nhờ vào tự động cấu hình của nó. **Tự động cấu hình có thể nói là cốt lõi của Spring Boot, vậy rốt cuộc tự động cấu hình là gì?**

## SpringBoot tự động cấu hình là gì?

Bây giờ khi nhắc tới tự động cấu hình (auto-assembly / auto-configuration), chúng ta thường liên tưởng tới Spring Boot. Tuy nhiên, thực ra Spring Framework đã sớm thực hiện tính năng này rồi. Spring Boot chỉ dựa trên nền tảng đó, thông qua phương thức SPI, tiến hành tối ưu hóa thêm một bước.

> Trong phiên bản Spring Boot 2.6 và sớm hơn, các class auto-configuration chủ yếu đăng ký thông qua `META-INF/spring.factories` trong file jar bên ngoài. Spring Boot 2.7 đã giới thiệu `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`, đồng thời tương thích với cách đăng ký cũ; Spring Boot 3.0 đã xóa bỏ hỗ trợ đăng ký class auto-configuration thông qua key `EnableAutoConfiguration` trong `spring.factories`, nhưng các mục đích sử dụng khác của `spring.factories` không bị ảnh hưởng.

Trong trường hợp không có Spring Boot, nếu chúng ta cần đưa vào dependency bên thứ ba, cần phải cấu hình thủ công, rất phiền phức. Tuy nhiên, trong Spring Boot, chúng ta trực tiếp đưa vào một starter là được. Ví dụ bạn muốn sử dụng redis trong dự án, trực tiếp đưa starter tương ứng vào dự án là được.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

Sau khi đưa starter vào, chúng ta thông qua một lượng nhỏ annotation và một số cấu hình đơn giản là có thể sử dụng các tính năng do component bên thứ ba cung cấp rồi.

Theo tôi thấy, tự động cấu hình có thể hiểu đơn giản là: **Thông qua annotation hoặc một số cấu hình đơn giản là có thể với sự giúp đỡ của Spring Boot thực hiện một khối tính năng nào đó.**

## SpringBoot thực hiện tự động cấu hình như thế nào?

Trước tiên chúng ta xem annotation cốt lõi của SpringBoot: `SpringBootApplication`.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
<1.>@SpringBootConfiguration
<2.>@ComponentScan
<3.>@EnableAutoConfiguration
public @interface SpringBootApplication {

}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration // Thực chất nó cũng là một class configuration
public @interface SpringBootConfiguration {
}
```

Có thể xem đại khái `@SpringBootApplication` như là tập hợp của các annotation `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`. Theo trang chủ chính thức của SpringBoot, tác dụng của 3 annotation này lần lượt là:

- `@EnableAutoConfiguration`: Bật cơ chế tự động cấu hình của SpringBoot
- `@Configuration`: Cho phép trong context đăng ký thêm bean hoặc import các class configuration khác
- `@ComponentScan`: Quét các bean được gắn annotation `@Component` (`@Service`, `@Controller`), annotation mặc định sẽ quét tất cả các class dưới package chứa class khởi động, có thể tùy chỉnh không quét một số bean nào đó. Như hình dưới đây, trong container sẽ loại trừ `TypeExcludeFilter` và `AutoConfigurationExcludeFilter`.

![](https://oss.javaguide.cn/p3-juejin/bcc73490afbe4c6ba62acde6a94ffdfd~tplv-k3u1fbpfcp-watermark.png)

`@EnableAutoConfiguration` là annotation quan trọng thực hiện tự động cấu hình, chúng ta bắt đầu từ annotation này.

### @EnableAutoConfiguration: Annotation cốt lõi thực hiện tự động cấu hình

`EnableAutoConfiguration` chỉ là một annotation đơn giản, việc thực hiện tính năng cốt lõi của tự động cấu hình thực tế là thông qua class `AutoConfigurationImportSelector`.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage // Tác dụng: Đăng ký tất cả các component dưới package main vào container
@Import({AutoConfigurationImportSelector.class}) // Load class auto-configuration xxxAutoconfiguration
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

Bây giờ chúng ta tập trung phân tích xem class `AutoConfigurationImportSelector` rốt cuộc làm những gì?

### AutoConfigurationImportSelector: Load các class auto-configuration

Dưới đây lấy đoạn mã nguồn trích xuất của Spring Boot 2.1.x làm ví dụ phân tích `AutoConfigurationImportSelector`. Code này bỏ qua một phần triển khai không ảnh hưởng đến quy trình, không thể dùng làm class độc lập để biên dịch trực tiếp. Các class auto-configuration ứng viên của phiên bản Spring Boot 2.7 trở lên chủ yếu được đọc từ file `AutoConfiguration.imports`, cấu trúc mã nguồn cụ thể có khác biệt so với code phiên bản cũ dưới đây.

Hệ thống kế thừa của class `AutoConfigurationImportSelector` như sau:

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

}

public interface DeferredImportSelector extends ImportSelector {

}

public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);
}
```

Có thể thấy, class `AutoConfigurationImportSelector` implement interface `ImportSelector`, cũng tức là implement method `selectImports` trong interface này, method này chủ yếu dùng để **lấy tất cả tên class đầy đủ (fully qualified class name) của các class thỏa mãn điều kiện, những class này cần được load vào IoC container**.

```java
private static final String[] NO_IMPORTS = new String[0];

public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // <1>. Phán đoán công tắc tự động cấu hình có bật không
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
          //<2>. Lấy tất cả các bean cần cấu hình
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }
```

Ở đây chúng ta cần tập trung chú ý vào method `getAutoConfigurationEntry()`, method này chủ yếu chịu trách nhiệm load các class auto-configuration.

Chuỗi gọi của method này như sau:

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/3c1200712655443ca4b38500d615bb70~tplv-k3u1fbpfcp-watermark.png)

Bây giờ chúng ta kết hợp với mã nguồn của `getAutoConfigurationEntry()` để phân tích chi tiết:

```java
private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
        //<1>.
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            //<2>.
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //<3>.
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //<4>.
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, autoConfigurationMetadata);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

**Bước 1**:

Phán đoán công tắc tự động cấu hình có bật hay không. Mặc định `spring.boot.enableautoconfiguration=true`, có thể thiết lập trong `application.properties` hoặc `application.yml`

![](https://oss.javaguide.cn/p3-juejin/77aa6a3727ea4392870f5cccd09844ab~tplv-k3u1fbpfcp-watermark.png)

**Bước 2**:

Dùng để lấy `exclude` và `excludeName` trong annotation `EnableAutoConfiguration`.

![](https://oss.javaguide.cn/p3-juejin/3d6ec93bbda1453aa08c52b49516c05a~tplv-k3u1fbpfcp-zoom-1.png)

**Bước 3**

Trong mã nguồn Spring Boot 2.1.x được sử dụng trong bài viết này, khi lấy tất cả các class configuration cần tự động cấu hình sẽ đọc `META-INF/spring.factories`:

```plain
spring-boot/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories
```

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/58c51920efea4757aa1ec29c6d5f9e36~tplv-k3u1fbpfcp-watermark.png)

Từ hình dưới đây có thể thấy nội dung cấu hình của file này đều được chúng ta đọc ra. Tác dụng của `XXXAutoConfiguration` chính là load component theo nhu cầu.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/94d6e1a060ac41db97043e1758789026~tplv-k3u1fbpfcp-watermark.png)

Không chỉ `META-INF/spring.factories` dưới dependency này được đọc, mà tài nguyên cùng tên trong các file jar khác thuộc classpath cũng sẽ được `SpringFactoriesLoader` hợp nhất đọc ra. Cần lưu ý, Starter thường chỉ dùng để gom nhóm các jar dependency, code auto-configuration và file đăng ký có thể đặt trong module autoconfigure độc lập, cũng có thể gộp cùng Starter, chứ không phải mỗi Starter đều bắt buộc phải chứa `spring.factories`.

Do đó, bạn có thể thấy rõ ràng, Spring Boot Starter của kết nối cơ sở dữ liệu druid đã tạo file `META-INF/spring.factories`.

Nếu muốn viết auto-configuration cho Spring Boot 2.6 và sớm hơn, cần sử dụng phương thức đăng ký này; auto-configuration hướng tới Spring Boot 3.x nên đổi sang dùng `AutoConfiguration.imports`.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/68fa66aeee474b0385f94d23bcfe1745~tplv-k3u1fbpfcp-watermark.png)

**Bước 4**:

Đến đây có thể người phỏng vấn sẽ hỏi bạn: "Nhiều cấu hình trong `spring.factories` như vậy, mỗi lần khởi động đều phải load hết sao?".

Rất rõ ràng, đây là điều không thực tế. Chúng ta debug về sau sẽ phát hiện, giá trị của `configurations` đã nhỏ đi rồi.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/267f8231ae2e48d982154140af6437b0~tplv-k3u1fbpfcp-watermark.png)

Bởi vì, bước này đã trải qua một lần sàng lọc, tất cả các điều kiện trong `@ConditionalOnXXX` đều thỏa mãn, thì class đó mới có hiệu lực.

```java
@Configuration
// Kiểm tra các class liên quan: RabbitTemplate và Channel có tồn tại hay không
// Tồn tại mới load
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
}
```

Các bạn có hứng thú có thể tìm hiểu chi tiết các conditional annotation do Spring Boot cung cấp:

- `@ConditionalOnBean`: Khi trong container có Bean được chỉ định
- `@ConditionalOnMissingBean`: Khi trong container không có Bean được chỉ định
- `@ConditionalOnSingleCandidate`: Khi Bean chỉ định trong container chỉ có một, hoặc tuy có nhiều nhưng chỉ định Bean ưu tiên
- `@ConditionalOnClass`: Khi dưới classpath có class được chỉ định
- `@ConditionalOnMissingClass`: Khi dưới classpath không có class được chỉ định
- `@ConditionalOnProperty`: Thuộc tính chỉ định có giá trị được chỉ định hay không
- `@ConditionalOnResource`: Classpath có giá trị được chỉ định hay không
- `@ConditionalOnExpression`: Dựa trên biểu thức SpEL làm điều kiện phán đoán
- `@ConditionalOnJava`: Dựa trên phiên bản Java làm điều kiện phán đoán
- `@ConditionalOnJndi`: Trong điều kiện JNDI tồn tại tìm ở vị trí chỉ định
- `@ConditionalOnNotWebApplication`: Trong điều kiện dự án hiện tại không phải là dự án Web
- `@ConditionalOnWebApplication`: Trong điều kiện dự án hiện tại là dự án Web

## Làm thế nào để thực hiện một Starter

Nói suông không bằng làm thật, bây giờ cùng code một starter, thực hiện ThreadPool tùy chỉnh

Bước thứ nhất, tạo project `threadpool-spring-boot-starter`

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/1ff0ebe7844f40289eb60213af72c5a6~tplv-k3u1fbpfcp-watermark.png)

Bước thứ hai, đưa vào các dependency liên quan đến Spring Boot

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/5e14254276604f87b261e5a80a354cc0~tplv-k3u1fbpfcp-watermark.png)

Bước thứ ba, tạo `ThreadPoolAutoConfiguration`

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/1843f1d12c5649fba85fd7b4e4a59e39~tplv-k3u1fbpfcp-watermark.png)

Bước thứ tư, đăng ký class auto-configuration. Đối với Spring Boot 2.6 và sớm hơn, tạo file `META-INF/spring.factories` dưới package resources của project `threadpool-spring-boot-starter`; phiên bản Spring Boot 2.7 trở lên nên dùng `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`, hướng tới Spring Boot 3.x các class auto-configuration thường được đánh dấu bằng `@AutoConfiguration`.

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/97b738321f1542ea8140484d6aaf0728~tplv-k3u1fbpfcp-watermark.png)

Cuối cùng tạo project mới đưa `threadpool-spring-boot-starter` vào

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/edcdd8595a024aba85b6bb20d0e3fed4~tplv-k3u1fbpfcp-watermark.png)

Test thành công!!!

![](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/9a265eea4de742a6bbdbbaa75f437307~tplv-k3u1fbpfcp-watermark.png)

## Tóm tắt

Spring Boot thông qua `@EnableAutoConfiguration` bật tự động cấu hình, và load các class auto-configuration ứng viên đã đăng ký trong classpath. Spring Boot 2.6 và sớm hơn chủ yếu đăng ký thông qua `spring.factories`, Spring Boot 2.7 trở lên sử dụng `AutoConfiguration.imports`. Class auto-configuration sẽ kết hợp với chuỗi annotation `@Conditional` để có hiệu lực theo nhu cầu; tác dụng chính của Starter là gom nhóm các dependency thường dùng, chứ không phải tên package cố định bắt buộc cho auto-configuration có hiệu lực.

<!-- @include: @article-footer.snippet.md -->
