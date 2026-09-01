---
title: Spring & Spring Boot 专题：IoC、AOP、事务、自动装配、常用注解与源码
description: Spring 和 Spring Boot 面试学习路线，涵盖 IoC、AOP、Bean 生命周期、事务、自动装配、常用注解、设计模式、@Async、源码和常见面试题。
category: 框架
tag:
  - Spring
  - Spring Boot
  - 后端面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Spring,Spring Boot,Spring面试题,SpringBoot面试题,IoC,AOP,Bean生命周期,Spring事务,Spring自动装配,Spring常用注解,Spring源码,@Async,Java后端面试
---

Spring là một trong những hạ tầng cốt lõi nhất của Java backend. Học Spring không thể chỉ học thuộc lòng các annotation, mà còn phải hiểu IoC, AOP, vòng đời của Bean, transaction, tự động cấu hình, pattern thiết kế và các điểm mở rộng (extension points) phổ biến.

Spring Boot tiếp tục tích hợp cấu hình, quản lý dependency, tự động cấu hình và khả năng giám sát (observability) trong môi trường production, giúp việc phát triển ứng dụng nhanh hơn, nhưng cũng dễ khiến người ta bỏ qua các nguyên lý bên dưới.

Nếu thời gian gấp, bạn có thể xem trước bản ôn tập phỏng vấn siêu tốc [Spring 常见面试题总结](https://interview.javaguide.cn/system-design/spring.html), sau đó quay lại chuyên đề này để bổ sung đầy đủ chi tiết về IoC, AOP, transaction và tự động cấu hình.

## Dành cho ai

- Các nhà phát triển Java backend đang học một cách hệ thống về Spring, Spring MVC, Spring Boot.
- Các bạn học sinh/sinh viên đang chuẩn bị cho các câu hỏi phỏng vấn tần suất cao về Spring, Spring Boot.
- Những người đọc đã từng sử dụng Spring Boot để phát triển dự án, nhưng chưa hiểu đủ sâu về IoC, AOP, transaction và tự động cấu hình.
- Các kỹ sư muốn hiểu hạ tầng kỹ thuật backend từ góc độ nguyên lý khung làm việc (framework).

## Trọng tâm học tập

- Spring IoC giải quyết vấn đề tạo đối tượng và quản lý dependency, AOP giải quyết vấn đề tái sử dụng logic cắt ngang (cross-cutting logic).
- Vòng đời Bean, scope, phụ thuộc vòng (circular dependency) và điểm mở rộng là chìa khóa để hiểu Spring container.
- Spring transaction cần nắm vững hành vi lan truyền (propagation behavior), mức độ cô lập (isolation level), quy tắc rollback và các kịch bản thất bại (không có hiệu lực).
- Cốt lõi của tự động cấu hình (auto-assembly) trong Spring Boot nằm ở cấu hình điều kiện (conditional configuration), binding cấu hình và hệ thống Starter.
- Học annotation không chỉ là học thuộc công dụng, mà còn phải biết năng lực container tương ứng đằng sau nó.
- Mã nguồn Spring và pattern thiết kế thích hợp dùng để đào sâu mức độ hiểu biết, không khuyến khích đào sâu ngay từ đầu vào chi tiết mã nguồn.

## Thứ tự đọc đề xuất

1. [Tổng kết câu hỏi phỏng vấn Spring thường gặp](./spring-knowledge-and-questions-summary.md): Xây dựng danh sách các câu hỏi Spring tần suất cao trước.
2. [Giải thích chi tiết IoC & AOP (Hiểu nhanh)](./ioc-and-aop.md): Hiểu hai khái niệm cơ bản cốt lõi nhất của Spring.
3. [Tổng kết các annotation thường gặp trong Spring & Spring MVC & Spring Boot](./spring-common-annotations.md): Tương quan các annotation thường gặp với năng lực của container.
4. [Giải thích chi tiết Spring Transaction](./spring-transaction.md): Tập trung nắm vững lan truyền transaction, mức cô lập, quy tắc rollback và các kịch bản thất bại.
5. [Giải thích chi tiết nguyên lý tự động cấu hình Spring Boot](./spring-boot-auto-assembly-principles.md): Hiểu tại sao Spring Boot có thể đạt được khả năng dùng được ngay (out of the box).
6. Sau đó tùy theo nhu cầu đọc tiếp [Giải thích chi tiết Pattern thiết kế trong Spring](./spring-design-patterns-summary.md), [Phân tích nguyên lý annotation @Async](./async.md) và [Đọc hiểu mã nguồn cốt lõi Spring Boot](./springboot-source-code.md).

## Bài viết cốt lõi

- [Tổng kết câu hỏi phỏng vấn Spring thường gặp](./spring-knowledge-and-questions-summary.md): Bao phủ các kiến thức cốt lõi của Spring như IoC container, nguyên lý AOP, vòng đời Bean, Dependency Injection,...
- [Tổng kết câu hỏi phỏng vấn Spring Boot thường gặp](./springboot-knowledge-and-questions-summary.md): Bao phủ các kiến thức như nguyên lý tự động cấu hình, cơ chế Starter, load file cấu hình và giám sát Actuator,...
- [Giải thích chi tiết IoC & AOP (Hiểu nhanh)](./ioc-and-aop.md): Giải thích cơ chế thực hiện Inversion of Control, Dependency Injection, Aspect-Oriented Programming và Dynamic Proxy.
- [Tổng kết các annotation thường gặp trong Spring & Spring MVC & Spring Boot](./spring-common-annotations.md): Tổng hợp các annotation thường gặp như `@Autowired`, `@Component`, `@RequestMapping`,...
- [Giải thích chi tiết Spring Transaction](./spring-transaction.md): Bao phủ `@Transactional`, hành vi lan truyền transaction, mức cô lập, kịch bản transaction thất bại và quy tắc rollback.
- [Giải thích chi tiết nguyên lý tự động cấu hình Spring Boot](./spring-boot-auto-assembly-principles.md): Phân tích `@EnableAutoConfiguration`, cơ chế load SpringFactories và conditional annotation.
- [Giải thích chi tiết Pattern thiết kế trong Spring](./spring-design-patterns-summary.md): Hiểu ứng dụng của Factory Pattern, Proxy Pattern, Singleton Pattern, Template Method,... trong Spring.
- [Phân tích nguyên lý annotation @Async](./async.md): Hiểu cấu hình task bất đồng bộ, thiết lập ThreadPool và cơ chế `@EnableAsync`.
- [Đọc hiểu mã nguồn cốt lõi Spring Boot](./springboot-source-code.md): Hiểu quy trình khởi động, cơ chế tự động cấu hình và SpringApplication từ góc độ mã nguồn.

## Câu hỏi tần suất cao

- IoC là gì? DI là gì?
- Mối quan hệ giữa Spring AOP và Dynamic Proxy là gì?
- Vòng đời của Spring Bean diễn ra như thế nào?
- Spring giải quyết phụ thuộc vòng (circular dependency) như thế nào? Những phụ thuộc vòng nào không thể giải quyết được?
- Sự khác biệt giữa `@Autowired` và `@Resource` là gì?
- Các hành vi lan truyền transaction trong Spring gồm những gì?
- Các kịch bản thất bại thường gặp của `@Transactional` là gì?
- Quy trình tự động cấu hình của Spring Boot là gì?
- Tác dụng của Starter là gì? Làm thế nào để tùy chỉnh một Starter?
- Những pattern thiết kế nào được sử dụng trong Spring?
- Tại sao `@Async` đôi khi không có hiệu lực?
- Nên bắt đầu đọc mã nguồn Spring từ những điểm đầu vào (entry point) nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức Thiết kế hệ thống](../../)
- [Chuyên đề nền tảng Thiết kế hệ thống](../../basis/)
- [Tổng kết câu hỏi phỏng vấn Pattern thiết kế thường gặp](../../design-pattern.md)
- [Tổng kết câu hỏi phỏng vấn MyBatis thường gặp](../mybatis/mybatis-interview.md)
- [Hệ thống kiến thức Hệ thống phân tán](../../../distributed-system/)

<!-- @include: @article-footer.snippet.md -->
