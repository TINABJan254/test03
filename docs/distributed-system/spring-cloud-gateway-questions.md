---
title: Spring Cloud Gateway 面试题总结：路由、Predicate、Filter、限流熔断与工作原理
category: 分布式
description: Spring Cloud Gateway 高频面试题总结，覆盖核心概念、路由匹配、Predicate、GatewayFilter、GlobalFilter、限流熔断、负载均衡、跨域处理和常见生产问题。
tag:
  - API 网关
  - Spring Cloud
head:
  - - meta
    - name: keywords
      content: Spring Cloud Gateway,Spring Cloud Gateway 面试题,API 网关,Predicate,GatewayFilter,GlobalFilter,网关限流,网关熔断,微服务网关
---

> Bài viết này được tái cấu trúc và hoàn thiện từ bài viết [6000 字 | 16 图 | 深入理解 Spring Cloud Gateway 的原理 - 悟空聊架构](https://mp.weixin.qq.com/s/XjFYsP1IUqNzWqXZdJn-Aw).

Bài viết này chỉ triển khai các điểm phỏng vấn tần suất cao của Spring Cloud Gateway. Nếu bạn vẫn chưa hiểu rõ tại sao API Gateway lại tồn tại, mối quan hệ giữa Gateway và RPC, nên chọn Zuul / Gateway / Kong / APISIX như thế nào, đề xuất bạn xem trước bài viết [Giải thích chi tiết API Gateway](./api-gateway.md).

## Spring Cloud Gateway là gì?

Spring Cloud Gateway thuộc loại gateway trong hệ sinh thái Spring Cloud, mục tiêu ra đời của nó chủ yếu là để thay thế **Zuul 1.x**. Zuul 1.x dựa trên kiến trúc Blocking I/O của Servlet, trong kịch bản concurrency cao hiệu năng bị hạn chế. Còn Zuul 2.x tuy áp dụng kiến trúc Non-blocking của Netty, nhưng Spring Cloud chính thức vẫn chưa tích hợp chính thức Zuul 2.x. Spring Cloud Gateway bắt đầu còn sớm hơn cả Zuul 2.x.

Để nâng cao hiệu năng của gateway, Spring Cloud Gateway dựa trên Spring WebFlux. Spring WebFlux sử dụng thư viện Reactor để thực hiện mô hình lập trình phản ứng (reactive programming model), tầng dưới dựa trên Netty để thực hiện I/O đồng bộ phi bất đồng bộ (synchronous non-blocking I/O).

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/springcloud-gateway-%20demo.png)

Spring Cloud Gateway không chỉ cung cấp phương thức routing thống nhất, mà còn dựa trên chuỗi Filter (Filter Chain) cung cấp các tính năng cơ bản của gateway, ví dụ: bảo mật, giám sát / metrics, giới hạn lưu lượng (rate limiting).

Sự khác biệt giữa Spring Cloud Gateway và Zuul 2.x không lớn, cũng là thông qua filter để xử lý request. Tuy nhiên, hiện tại khuyến khích sử dụng Spring Cloud Gateway hơn là Zuul, hệ sinh thái Spring Cloud hỗ trợ nó thân thiện hơn.

- Địa chỉ GitHub: <https://github.com/spring-cloud/spring-cloud-gateway>
- Trang chủ chính thức: <https://spring.io/projects/spring-cloud-gateway>

## Quy trình hoạt động của Spring Cloud Gateway?

Quy trình hoạt động của Spring Cloud Gateway như hình dưới đây:

![Quy trình hoạt động của Spring Cloud Gateway](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-workflow.png)

Đây là một hình trong blog chính thức của Spring, địa chỉ bài gốc: <https://spring.io/blog/2022/08/26/creating-a-custom-spring-cloud-gateway-filter>.

Phân tích quy trình cụ thể:

1. **Phán đoán routing**: Sau khi request của client đến gateway, trước tiên đi qua Gateway Handler Mapping để xử lý, ở đây sẽ thực hiện phán đoán đoán nhận (Predicate), xem thỏa mãn quy tắc routing nào, routing này ánh xạ tới một service phía backend nào đó.
2. **Lọc request**: Sau đó request đến Gateway Web Handler, ở đây có rất nhiều filter, cấu thành chuỗi filter (Filter Chain), những filter này có thể đánh chặn và sửa đổi request, ví dụ thêm request header, validation parameter,..., giống như lọc nước thải. Sau đó chuyển tiếp request đến service backend thực tế. Những filter này về mặt logic có thể gọi là Pre-Filters, Pre có thể hiểu là "trước khi...".
3. **Xử lý dịch vụ**: Service backend sẽ tiến hành xử lý request.
4. **Lọc response**: Sau khi backend xử lý xong kết quả, trả về cho filter của Gateway một lần nữa làm xử lý, về mặt logic có thể gọi là Post-Filters, Post có thể hiểu là "sau khi...".
5. **Trả về response**: Response sau khi qua xử lý lọc, được trả về cho client.

Tóm lại: Request của client trước tiên thông qua quy tắc khớp để tìm routing thích hợp, là có thể ánh xạ đến service cụ thể. Sau đó request qua xử lý của filter được chuyển tiếp cho service cụ thể, service sau khi xử lý, lại qua xử lý của filter lần nữa, cuối cùng trả về cho client.

## Đoán nhận (Predicate) trong Spring Cloud Gateway là gì?

Đoán nhận (Predicate) từ này nghe có vẻ tương đối trừu tượng, nó có thể hiểu là thực hiện một lần phán đoán đối với điều kiện của request: Kết quả là true thì khớp với routing hiện tại, kết quả là false thì tiếp tục khớp với các routing khác.

Trong Gateway, nếu request do client gửi thỏa mãn điều kiện của predicate, thì ánh xạ đến router chỉ định, là có thể chuyển tiếp đến service chỉ định để tiến hành xử lý.

Ví dụ cấu hình predicate như dưới đây, cấu hình hai quy tắc routing, có một cấu hình predicate `predicates`, khi url trong request chứa `api/thirdparty`, thì khớp tới routing đầu tiên `route_thirdparty`.

![Ví dụ cấu hình Predicate](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-predicate-example.png)

Các quy tắc route predicate thường gặp như hình dưới đây:

![Quy tắc Route Predicate trong Spring Cloud Gateway](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-predicate-rules.png)

## Mối quan hệ giữa Route và Predicate trong Spring Cloud Gateway là gì?

Mối quan hệ tương ứng giữa Route (Routing) và Predicate (Đoán nhận) như sau:

![Mối quan hệ tương ứng giữa Route và Predicate](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-predicate-route.png)

- **Một - Nhiều**: Một quy tắc routing có thể chứa nhiều predicate. Như trong hình trên routing Route1 cấu hình ba predicate Predicate.
- **Thỏa mãn đồng thời**: Nếu một quy tắc routing có nhiều predicate, thì cần phải thỏa mãn đồng thời mới có thể khớp. Như trong hình trên routing Route2 cấu hình hai predicate, request do client gửi bắt buộc phải thỏa mãn đồng thời hai predicate này mới khớp với routing Route2.
- **Khớp thành công đầu tiên**: Nếu một request có thể khớp với nhiều routing, thì ánh xạ routing khớp thành công đầu tiên. Như hình trên hiển thị, request do client gửi thỏa mãn predicate của Route3 và Route4, nhưng cấu hình của Route3 đứng trước trong file cấu hình, nên sẽ chỉ khớp với Route3.

## Spring Cloud Gateway thực hiện Dynamic Routing như thế nào?

Khi sử dụng Spring Cloud Gateway, phương án do tài liệu chính thức cung cấp luôn dựa trên cách cấu hình bằng file configuration hoặc code.

Spring Cloud Gateway với tư cách là đầu vào của microservices, cần cố gắng tránh khởi động lại, mà hiện tại thay đổi cấu hình cần khởi động lại dịch vụ không đáp ứng được nhu cầu nghiệp vụ refresh động, thay đổi theo thời gian thực trong quá trình sản xuất thực tế, do đó chúng ta cần cấu hình động gateway lúc Spring Cloud Gateway đang chạy.

Có rất nhiều cách thực hiện dynamic routing, trong đó một cách được khuyên dùng là dựa trên registry Nacos để làm. Spring Cloud Gateway có thể lấy metadata của service từ registry (ví dụ service name, path,...), sau đó dựa theo các thông tin này tự động sinh ra quy tắc routing. Như vậy, khi bạn thêm, xóa hoặc update service instance, gateway sẽ tự động cảm nhận và điều chỉnh tương ứng quy tắc routing, không cần bảo trì thủ công cấu hình routing.

Thực ra những bước phức tạp này không cần chúng ta thực hiện thủ công, thông qua Nacos Server và Spring Cloud Alibaba Nacos Config là có thể thực hiện thay đổi động cấu hình, địa chỉ tài liệu chính thức: <https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config> .

## Các Filter trong Spring Cloud Gateway gồm những gì?

Filter (Bộ lọc) dựa theo request và response có thể chia thành hai loại:

- **Loại Pre**: Trước khi request được chuyển tiếp tới microservice, tiến hành đánh chặn và sửa đổi request, ví dụ validation parameter, validation phân quyền, giám sát lưu lượng, xuất log cũng như chuyển đổi giao thức,...
- **Loại Post**: Sau khi microservice xử lý xong request, trả response về cho gateway, gateway có thể tiến hành xử lý một lần nữa, ví dụ sửa đổi nội dung response hoặc response header, xuất log, giám sát lưu lượng,...

Một cách phân loại khác là phân chia dựa theo phạm vi tác dụng của Filter:

- **GatewayFilter**: Filter cục bộ, filter áp dụng trên một routing đơn lẻ hoặc một nhóm routing. Đánh dấu màu đỏ thể hiện filter được sử dụng phổ biến hơn.
- **GlobalFilter**: Filter toàn cục, filter áp dụng trên tất cả các routing.

### Filter cục bộ (GatewayFilter)

Các filter cục bộ thường gặp như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-gatewayfilters.png)

Cụ thể dùng như thế nào? Ở đây có một ví dụ, nếu URL khớp thành công, thì loại bỏ "api" trong URL.

```yaml
filters: # Bộ lọc
  - RewritePath=/api/(?<segment>.*),/$\{segment} # Thay thế "api" chứa trong path chuyển hướng thành rỗng
```

Tất nhiên chúng ta cũng có thể tùy chỉnh filter, bài viết này không đi sâu.

### Filter toàn cục (GlobalFilter)

Các filter toàn cục thường gặp như hình dưới đây:

![](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/api-gateway/spring-cloud-gateway-globalfilters.png)

Cách dùng thường gặp nhất của filter toàn cục là tiến hành load balancing (cân bằng tải). Cấu hình như dưới đây:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: route_member # Quy tắc routing của microservice bên thứ ba
          uri: lb://passjava-member # Load balancing, chuyển tiếp request tới service passjava-member đăng ký ở registry
          predicates: # Đoán nhận
            - Path=/api/member/** # Nếu path request phía frontend chứa api/member, thì áp dụng quy tắc routing này
          filters: # Bộ lọc
            - RewritePath=/api/(?<segment>.*),/$\{segment} # Thay thế api chứa trong path chuyển hướng thành rỗng
```

Ở đây có hằng số `lb`, sử dụng filter toàn cục `LoadBalancerClientFilter`, khi khớp tới routing này, sẽ chuyển tiếp request tới service passjava-member, và hỗ trợ chuyển tiếp load balancing, cũng tức là trước tiên giải mã passjava-member thành host và port của microservice thực tế, sau đó mới chuyển tiếp cho microservice thực tế.

## Spring Cloud Gateway có hỗ trợ Rate Limiting (Giới hạn lưu lượng) không?

Spring Cloud Gateway đi kèm sẵn filter giới hạn lưu lượng, interface tương ứng là `RateLimiter`, interface `RateLimiter` chỉ có một class triển khai `RedisRateLimiter` (giới hạn lưu lượng dựa trên Redis + Lua), tính năng giới hạn lưu lượng cung cấp tương đối đơn sơ và không dễ dùng.

Từ phiên bản Sentinel 1.6.0 trở đi, Sentinel đã đưa vào module thích ứng của Spring Cloud Gateway, có thể cung cấp giới hạn lưu lượng ở hai chiều tài nguyên: chiều route và chiều API tùy chỉnh. Nghĩa là, Spring Cloud Gateway có thể kết hợp với Sentinel để thực hiện kiểm soát lưu lượng gateway mạnh mẽ hơn.

## Custom Global Exception Handling trong Spring Cloud Gateway như thế nào?

Trong dự án SpringBoot, chúng ta bắt exception toàn cục chỉ cần cấu hình `@RestControllerAdvice` và `@ExceptionHandler` trong dự án là được. Tuy nhiên, cách này không áp dụng dưới Spring Cloud Gateway.

Spring Cloud Gateway cung cấp nhiều phương thức xử lý toàn cục, một cách thường dùng hơn là implement `ErrorWebExceptionHandler` và override method `handle` trong đó.

```java
@Order(-1)
@Component
@RequiredArgsConstructor
public class GlobalErrorWebExceptionHandler implements ErrorWebExceptionHandler {
    private final ObjectMapper objectMapper;

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
    // ...
    }
}
```

## Tham khảo

- Tài liệu chính thức Spring Cloud Gateway: <https://cloud.spring.io/spring-cloud-gateway/reference/html/>
- Creating a custom Spring Cloud Gateway Filter: <https://spring.io/blog/2022/08/26/creating-a-custom-spring-cloud-gateway-filter>
- Xử lý Exception toàn cục: <https://zhuanlan.zhihu.com/p/347028665>

<!-- @include: @article-footer.snippet.md -->
