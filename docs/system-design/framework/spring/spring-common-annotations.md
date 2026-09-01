---
title: Spring&SpringMVC&SpringBoot常用注解总结
description: Spring和SpringBoot常用注解大全，涵盖@Autowired、@Component、@RequestMapping等核心注解的用法详解。
category: 框架
tag:
  - SpringBoot
  - Spring
head:
  - - meta
    - name: keywords
      content: Spring注解,Spring Boot注解,@SpringBootApplication,@Autowired,@RequestMapping,@Configuration,@Component,常用注解
---

Không hề nói quá khi nói rằng các annotation thường gặp trong Spring/SpringBoot được giới thiệu trong bài viết này về cơ bản đã bao phủ hầu hết các kịch bản thường gặp mà bạn gặp phải trong công việc. Đối với mỗi annotation, bài viết này đều cung cấp cách dùng cụ thể, sau khi nắm vững những nội dung này, việc sử dụng Spring Boot để phát triển dự án về cơ bản không còn vấn đề gì lớn nữa!

**Tại sao lại viết bài viết này?**

Gần đây thấy trên mạng có một bài viết về các annotation thường gặp trong Spring Boot được chia sẻ rộng rãi, nhưng nội dung bài viết tồn tại một số điểm gây hiểu lầm, có thể không quá thân thiện với các lập trình viên chưa có nhiều kinh nghiệm sử dụng thực tế. Thế là tôi đã dành vài ngày thời gian để tổng hợp bài viết này, hy vọng có thể giúp mọi người hiểu và sử dụng các annotation trong Spring tốt hơn.

**Vì năng lực và sức lực cá nhân có hạn, nếu có bất kỳ sai sót hoặc thiếu sót nào, hoan nghênh chỉ ra! Rất cảm ơn!**

## Annotation cơ bản trong Spring Boot

`@SpringBootApplication` là annotation cốt lõi của ứng dụng Spring Boot, thường dùng để đánh dấu class khởi động chính (main startup class).

Ví dụ:

```java
@SpringBootApplication
public class SpringSecurityJwtGuideApplication {
      public static void main(java.lang.String[] args) {
        SpringApplication.run(SpringSecurityJwtGuideApplication.class, args);
    }
}
```

Chúng ta có thể xem `@SpringBootApplication` như là sự kết hợp của ba annotation dưới đây:

- **`@EnableAutoConfiguration`**: Bật cơ chế tự động cấu hình (auto-configuration) của Spring Boot.
- **`@ComponentScan`**: Quét các class có annotation `@Component`, `@Service`, `@Repository`, `@Controller`,...
- **`@Configuration`**: Cho phép đăng ký thêm các Spring Bean hoặc import các class configuration khác.

Mã nguồn như sau:

```java
package org.springframework.boot.autoconfigure;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
   ......
}

package org.springframework.boot;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```

## Spring Bean

### Dependency Injection (DI)

`@Autowired` dùng để tự động inject dependency (tức là các Spring Bean khác). Nó có thể đánh dấu trên constructor, field, method Setter hoặc method configuration, Spring container sẽ tự động tìm kiếm Bean có kiểu tương ứng và inject nó vào.

```java
@Service
public class UserServiceImpl implements UserService {
    // ...
}

@RestController
public class UserController {
    // Field injection
    @Autowired
    private UserService userService;
    // ...
}
```

Khi tồn tại nhiều Bean cùng kiểu, `@Autowired` mặc định inject theo kiểu có thể gây ra sự mơ hồ. Lúc này, có thể kết hợp với `@Qualifier`, thông qua việc chỉ định tên của Bean để lựa chọn chính xác instance cần inject.

```java
@Repository("userRepositoryA")
public class UserRepositoryA implements UserRepository { /* ... */ }

@Repository("userRepositoryB")
public class UserRepositoryB implements UserRepository { /* ... */ }

@Service
public class UserService {
    @Autowired
    @Qualifier("userRepositoryA") // Chỉ định inject Bean có tên "userRepositoryA"
    private UserRepository userRepository;
    // ...
}
```

`@Primary` cũng là để giải quyết vấn đề inject khi cùng một kiểu tồn tại nhiều instance Bean. Khi định nghĩa Bean (ví dụ dùng `@Bean` hoặc annotation trên class), thêm annotation `@Primary`, thể hiện Bean đó là đối tượng inject **được ưu tiên**. Khi tiến hành inject bằng `@Autowired`, nếu không sử dụng `@Qualifier` để chỉ định tên, Spring sẽ ưu tiên chọn Bean có gắn `@Primary`.

```java
@Primary // Đặt UserRepositoryA làm đối tượng inject ưu tiên
@Repository("userRepositoryA")
public class UserRepositoryA implements UserRepository { /* ... */ }

@Repository("userRepositoryB")
public class UserRepositoryB implements UserRepository { /* ... */ }

@Service
public class UserService {
    @Autowired // Sẽ tự động inject UserRepositoryA, vì nó là @Primary
    private UserRepository userRepository;
    // ...
}
```

`@Resource(name="beanName")` là annotation do chuẩn JSR-250 định nghĩa, cũng dùng cho Dependency Injection. Nó mặc định tìm kiếm Bean theo **tên (by Name)** để inject, còn `@Autowired` mặc định theo **kiểu (by Type)** . Nếu không chỉ định thuộc tính `name`, nó sẽ thử tìm kiếm theo tên field hoặc tên method, nếu không tìm thấy thì lùi về tìm kiếm theo kiểu (tương tự `@Autowired`).

`@Resource` chỉ có thể đánh dấu trên field và method Setter, không hỗ trợ constructor injection.

```java
@Service
public class UserService {
    @Resource(name = "userRepositoryA")
    private UserRepository userRepository;
    // ...
}
```

### Scope của Bean

`@Scope("scopeName")` định nghĩa scope của Spring Bean, tức là vòng đời và phạm vi hiển thị của instance Bean. Các scope thường dùng bao gồm:

- **singleton** : Trong IoC container chỉ có duy nhất một instance bean. Bean trong Spring mặc định đều là singleton, đây là ứng dụng của Singleton Design Pattern.
- **prototype** : Mỗi lần lấy đều sẽ tạo một instance bean mới. Nghĩa là, gọi `getBean()` hai lần liên tiếp sẽ nhận được hai instance Bean khác nhau.
- **request** （chỉ dùng cho ứng dụng Web）: Mỗi một request HTTP đều tạo ra một bean mới (request bean), bean này chỉ có hiệu lực trong request HTTP hiện tại.
- **session** （chỉ dùng cho ứng dụng Web） : Mỗi một request HTTP đến từ session mới đều tạo ra một bean mới (session bean), bean này chỉ có hiệu lực trong session HTTP hiện tại.
- **application/global-session** （chỉ dùng cho ứng dụng Web）：Mỗi ứng dụng Web khi khởi động sẽ tạo một Bean (application bean), bean này chỉ có hiệu lực trong thời gian chạy của ứng dụng hiện tại.
- **websocket** （chỉ dùng cho ứng dụng Web）：Mỗi phiên WebSocket tạo ra một bean mới.

```java
@Component
// Mỗi lần lấy đều sẽ tạo instance PrototypeBean mới
@Scope("prototype")
public class PrototypeBean {
    // ...
}
```

### Đăng ký Bean

Spring container cần biết những class nào cần được quản lý thành Bean. Ngoài việc sử dụng method `@Bean` để khai báo tường minh (thường trong class `@Configuration`), cách thường gặp hơn là sử dụng các annotation Stereotype để đánh dấu class, và phối hợp với cơ chế quét component (Component Scanning), để Spring tự động phát hiện và đăng ký các class này làm Bean. Những Bean này sau đó có thể được inject vào các component khác thông qua `@Autowired`,...

Dưới đây là một số annotation đăng ký Bean thường gặp:

- `@Component`：Annotation dùng chung, có thể đánh dấu bất kỳ class nào thành component của `Spring`. Nếu một Bean không biết thuộc về tầng nào, có thể sử dụng annotation `@Component` để đánh dấu.
- `@Repository` : Tương ứng với tầng lưu trữ (persistence layer) tức tầng Dao, chủ yếu dùng cho các thao tác liên quan đến cơ sở dữ liệu.
- `@Service` : Tương ứng với tầng dịch vụ (service layer), chủ yếu liên quan đến một số logic phức tạp, cần dùng đến tầng Dao.
- `@Controller` : Tương ứng với tầng điều khiển Spring MVC, chủ yếu dùng để nhận request của người dùng và gọi tầng Service để trả về dữ liệu cho trang phía frontend.
- `@RestController`：Một annotation kết hợp, tương đương với `@Controller` + `@ResponseBody`. Nó chuyên dùng để xây dựng controller của dịch vụ RESTful Web. Class được đánh dấu `@RestController`, tất cả các handler method của nó có giá trị trả về đều sẽ tự động được tuần tự hóa (thường là JSON) và ghi vào HTTP response body, chứ không được giải mã thành tên view.

`@Controller` vs `@RestController`：

- `@Controller`：Chủ yếu dùng cho ứng dụng Spring MVC truyền thống, giá trị trả về của method thường là tên view logic, cần bộ giải mã view phối hợp để render trang. Nếu cần trả về dữ liệu (như JSON), thì cần thêm annotation `@ResponseBody` trên method.
- `@RestController`：Chuyên dùng cho thiết kế RESTful API trả về dữ liệu. Sau khi sử dụng annotation này trên class, giá trị trả về của tất cả các method mặc định sẽ được xem là nội dung response body (tương đương với việc mỗi method đều ẩn chứa `@ResponseBody`), thường dùng để trả về dữ liệu JSON hoặc XML. Trong các ứng dụng tách biệt frontend và backend hiện đại, `@RestController` là lựa chọn được sử dụng phổ biến hơn.

Về so sánh giữa `@RestController` và `@Controller`, vui lòng xem bài viết này: [@RestController vs @Controller](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485544&idx=1&sn=3cc95b88979e28fe3bfe539eb421c6d8&chksm=cea247a3f9d5ceb5e324ff4b8697adc3e828ecf71a3468445e70221cce768d1e722085359907&token=1725092312&lang=zh_CN#rd)。

## Cấu hình

### Khai báo class configuration

`@Configuration` chủ yếu dùng để khai báo một class là class configuration của Spring. Mặc dù cũng có thể dùng annotation `@Component` để thay thế, nhưng `@Configuration` có thể thể hiện rõ ràng hơn mục đích của class đó (định nghĩa Bean), ngữ nghĩa rõ ràng hơn, cũng tiện cho Spring tiến hành xử lý đặc thù (ví dụ thông qua CGLIB proxy đảm bảo hành vi singleton của method `@Bean`).

```java
@Configuration
public class AppConfig {

    // Annotation @Bean dùng để khai báo một Bean trong class configuration
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

    // Class configuration có thể chứa một hoặc nhiều method @Bean.
}
```

### Đọc thông tin cấu hình

Trong phát triển ứng dụng, chúng ta thường xuyên cần quản lý một số thông tin cấu hình, ví dụ chi tiết kết nối cơ sở dữ liệu, secret key hoặc địa chỉ của dịch vụ bên thứ ba (như Aliyun OSS, dịch vụ SMS, xác thực WeChat,...). Thông thường, các thông tin này sẽ **được tập trung lưu trữ trong file cấu hình** (như `application.yml` hoặc `application.properties`), tiện cho việc quản lý và chỉnh sửa.

Spring cung cấp nhiều cách tiện lợi để đọc các thông tin cấu hình này. Giả sử chúng ta có file `application.yml` như sau:

```yaml
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！

my-profile:
  name: Guide哥
  email: koushuangbwcx@163.com

library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```

Dưới đây xin giới thiệu một số cách thường dùng để đọc cấu hình:

1, `@Value("${property.key}")` Inject giá trị thuộc tính đơn lẻ trong file cấu hình (như `application.properties` hoặc `application.yml`). Nó còn hỗ trợ Spring Expression Language (SpEL), có thể thực hiện logic inject phức tạp hơn.

```java
@Value("${wuhan2020}")
String wuhan2020;
```

2, `@ConfigurationProperties` có thể đọc thông tin cấu hình và bind với Bean, được sử dụng nhiều hơn.

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  // Bỏ qua getter/setter
  ......
}
```

Bạn có thể inject nó vào class để sử dụng giống như sử dụng một Spring Bean thông thường.

```java
@Service
public class LibraryService {

    private final LibraryProperties libraryProperties;

    @Autowired
    public LibraryService(LibraryProperties libraryProperties) {
        this.libraryProperties = libraryProperties;
    }

    public void printLibraryInfo() {
        System.out.println(libraryProperties);
    }
}
```

### Load file cấu hình được chỉ định

Annotation `@PropertySource` cho phép load file cấu hình tùy chỉnh. Áp dụng cho kịch bản cần lưu trữ độc lập một phần thông tin cấu hình.

```java
@Component
@PropertySource("classpath:website.properties")

class WebSite {
    @Value("${url}")
    private String url;

  // Bỏ qua getter/setter
  ......
}
```

**Lưu ý**: Khi sử dụng `@PropertySource`, đảm bảo đường dẫn file bên ngoài chính xác, và file nằm trong classpath.

Nội dung chi tiết hơn vui lòng xem bài viết này của tôi: [10 分钟搞定 SpringBoot 如何优雅读取配置文件？](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486181&idx=2&sn=10db0ae64ef501f96a5b0dbc4bd78786&chksm=cea2452ef9d5cc384678e456427328600971180a77e40c13936b19369672ca3e342c26e92b50&token=816772476&lang=zh_CN#rd) 。

## MVC

### HTTP Request

**5 loại request thường gặp:**

- **GET**: Request lấy tài nguyên cụ thể từ server. Ví dụ: `GET /users` (lấy tất cả học sinh)
- **POST**: Tạo một tài nguyên mới trên server. Ví dụ: `POST /users` (tạo học sinh)
- **PUT**: Update tài nguyên trên server (client cung cấp toàn bộ tài nguyên sau khi update). Ví dụ: `PUT /users/12` (update học sinh có mã số 12)
- **DELETE**: Xóa tài nguyên cụ thể khỏi server. Ví dụ: `DELETE /users/12` (xóa học sinh có mã số 12)
- **PATCH**: Update tài nguyên trên server (client cung cấp các thuộc tính thay đổi, có thể xem là update một phần), ít khi sử dụng hơn nên ở đây không lấy ví dụ.

#### GET Request

`@GetMapping("users")` tương đương với `@RequestMapping(value="/users",method=RequestMethod.GET)`.

```java
@GetMapping("/users")
public ResponseEntity<List<User>> getAllUsers() {
  return ResponseEntity.ok(userRepository.findAll());
}
```

#### POST Request

`@PostMapping("users")` tương đương với `@RequestMapping(value="/users",method=RequestMethod.POST)`.

`@PostMapping` thường phối hợp với `@RequestBody`, dùng để nhận dữ liệu JSON và ánh xạ thành Java object.

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody UserCreateRequest userCreateRequest) {
  User user = userService.create(userCreateRequest);
  return ResponseEntity.status(HttpStatus.CREATED).body(user);
}
```

#### PUT Request

`@PutMapping("/users/{userId}")` tương đương với `@RequestMapping(value="/users/{userId}",method=RequestMethod.PUT)`.

```java
@PutMapping("/users/{userId}")
public ResponseEntity<User> updateUser(@PathVariable(value = "userId") Long userId,
  @Valid @RequestBody UserUpdateRequest userUpdateRequest) {
  ......
}
```

#### DELETE Request

`@DeleteMapping("/users/{userId}")` tương đương với `@RequestMapping(value="/users/{userId}",method=RequestMethod.DELETE)`

```java
@DeleteMapping("/users/{userId}")
public ResponseEntity deleteUser(@PathVariable(value = "userId") Long userId){
  ......
}
```

#### PATCH Request

Thông thường trong dự án thực tế, chúng ta chỉ khi PUT không đủ dùng mới dùng PATCH request để update dữ liệu.

```java
  @PatchMapping("/profile")
  public ResponseEntity updateStudent(@RequestBody StudentUpdateRequest studentUpdateRequest) {
        studentRepository.updateDetail(studentUpdateRequest);
        return ResponseEntity.ok().build();
    }
```

### Parameter Binding

Khi xử lý HTTP request, Spring MVC cung cấp nhiều annotation dùng để bind request parameter vào method parameter. Dưới đây là các phương thức parameter binding thường gặp:

#### Trích xuất parameter từ đường dẫn URL

`@PathVariable` dùng để trích xuất parameter từ đường dẫn URL. Ví dụ:

```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getTeachersByClass(@PathVariable("klassId") Long klassId) {
    return teacherService.findTeachersByClass(klassId);
}
```

Nếu URL request là `/klasses/123/teachers`, thì `klassId = 123`.

#### Bind query parameter

`@RequestParam` dùng để bind query parameter. Ví dụ:

```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getTeachersByClass(@PathVariable Long klassId,
                                        @RequestParam(value = "type", required = false) String type) {
    return teacherService.findTeachersByClassAndType(klassId, type);
}
```

Nếu URL request là `/klasses/123/teachers?type=web`, thì `klassId = 123`, `type = web`.

#### Bind dữ liệu JSON trong request body

`@RequestBody` dùng để đọc phần body của Request request (có thể là POST, PUT, DELETE, GET request) và **Content-Type có định dạng application/json**, sau khi nhận dữ liệu sẽ tự động bind dữ liệu sang Java object. Hệ thống sẽ sử dụng `HttpMessageConverter` hoặc `HttpMessageConverter` tùy chỉnh để chuyển đổi chuỗi json trong body request thành java object.

Tôi dùng một ví dụ đơn giản để minh họa cách sử dụng cơ bản!

Chúng ta có một API đăng ký:

```java
@PostMapping("/sign-up")
public ResponseEntity signUp(@RequestBody @Valid UserRegisterRequest userRegisterRequest) {
  userService.save(userRegisterRequest);
  return ResponseEntity.ok().build();
}
```

Đối tượng `UserRegisterRequest`:

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserRegisterRequest {
    @NotBlank
    private String userName;
    @NotBlank
    private String password;
    @NotBlank
    private String fullName;
}
```

Chúng ta gửi POST request đến API này, và body mang theo dữ liệu JSON:

```json
{ "userName": "coder", "fullName": "shuangkou", "password": "123456" }
```

Như vậy phía backend của chúng ta có thể trực tiếp ánh xạ dữ liệu định dạng json sang class `UserRegisterRequest` của chúng ta.

![](./images/spring-annotations/@RequestBody.png)

**Lưu ý**:

- Một method chỉ có thể có một parameter `@RequestBody`, nhưng có thể có nhiều `@PathVariable` và `@RequestParam`.
- Nếu cần nhận nhiều object phức tạp, khuyên bạn nên gộp thành một object duy nhất.

## Validation dữ liệu

Validation dữ liệu là khâu then chốt đảm bảo tính ổn định và an toàn của hệ thống. Ngay cả khi giao diện người dùng (frontend) đã thực hiện validation dữ liệu, **dịch vụ backend vẫn phải tiến hành validation lần nữa đối với dữ liệu nhận được**. Đó là vì validation phía frontend có thể bị bỏ qua một cách dễ dàng (ví dụ thông qua developer tools chỉnh sửa request hoặc dùng công dụng HTTP như Postman, curl để gọi trực tiếp API), dữ liệu độc hại hoặc sai sót có thể gửi trực tiếp đến backend. Do đó, validation phía backend là tuyến phòng thủ cuối cùng và quan trọng nhất để ngăn chặn dữ liệu bất hợp pháp, duy trì tính nhất quán của dữ liệu, và đảm bảo logic nghiệp vụ thực thi chính xác.

Bean Validation là một bộ quy chuẩn (specification) định nghĩa chuẩn validation parameter cho JavaBean (JSR 303, 349, 380), nó cung cấp một loạt các annotation, có thể trực tiếp sử dụng trên thuộc tính của JavaBean, từ đó thực hiện validation parameter một cách tiện lợi.

- **JSR 303 (Bean Validation 1.0)**: Đặt nền móng, giới thiệu các annotation validation cốt lõi (như `@NotNull`, `@Size`, `@Min`, `@Max`,...), định nghĩa cách thức tiến hành validation thuộc tính của JavaBean thông qua annotation, và hỗ trợ validation object lồng nhau và validator tùy chỉnh.
- **JSR 349 (Bean Validation 1.1)**: Mở rộng dựa trên 1.0, ví dụ giới thiệu hỗ trợ validation cho method parameter và return value, tăng cường xử lý cho validation theo nhóm (Group Validation).
- **JSR 380 (Bean Validation 2.0)**: Đón nhận các tính năng mới của Java 8, và tiến hành một số cải tiến, ví dụ hỗ trợ các kiểu date và time trong package `java.time`, giới thiệu một số annotation validation mới (như `@NotEmpty`, `@NotBlank`,...).

Bản thân Bean Validation chỉ là một bộ **quy chuẩn (interface và annotation)**, chúng ta cần một **framework cụ thể** implement bộ quy chuẩn này để thực thi logic validation. Hiện tại, **Hibernate Validator** là triển khai tham chiếu uy tín nhất và được sử dụng rộng rãi nhất của quy chuẩn Bean Validation.

- Hibernate Validator 4.x implement Bean Validation 1.0 (JSR 303).
- Hibernate Validator 5.x implement Bean Validation 1.1 (JSR 349).
- Hibernate Validator 6.x implement Bean Validation 2.0 (JSR 380), sử dụng package `javax.validation`.
- Hibernate Validator 7.x và 8.x implement Jakarta Bean Validation 3.0, sử dụng package `jakarta.validation`; Hibernate Validator 9.x implement Jakarta Validation 3.1.

Trong dự án Spring Boot sử dụng Bean Validation rất tiện lợi, nhờ vào khả năng tự động cấu hình của Spring Boot. Về việc khai báo dependency, cần lưu ý:

- Trong các phiên bản Spring Boot sớm hơn (thường chỉ trước 2.3.x), dependency `spring-boot-starter-web` mặc định đã bao gồm hibernate-validator. Do đó, chỉ cần khai báo Web Starter là không cần thêm dependency liên quan đến validation nữa.
- Từ phiên bản Spring Boot 2.3.x trở đi, để quản lý dependency tinh chỉnh hơn, các dependency liên quan đến validation đã được chuyển khỏi spring-boot-starter-web. Nếu dự án của bạn sử dụng các phiên bản này hoặc mới hơn, và cần tính năng Bean Validation, bạn cần khai báo tường minh dependency `spring-boot-starter-validation`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

![](https://oss.javaguide.cn/2021/03/c7bacd12-1c1a-4e41-aaaf-4cad840fc073.png)

Dự án không phải SpringBoot cần tự mình khai báo các package dependency liên quan, ở đây không giải thích thêm, cụ thể có thể xem bài viết này của tôi: [如何在 Spring/Spring Boot 中做参数校验？你需要了解的都在这里！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485783&idx=1&sn=a407f3b75efa17c643407daa7fb2acd6&chksm=cea2469cf9d5cf8afbcd0a8a1c9cc4294d6805b8e01bee6f76bb2884c5bc15478e91459def49&token=292197051&lang=zh_CN#rd)。

👉 Cần lưu ý là: Ưu tiên sử dụng các annotation ràng buộc do quy chuẩn Bean Validation/Jakarta Validation cung cấp, chứ không dùng ràng buộc riêng của Hibernate Validator. Spring Boot 2.x thường dùng `javax.validation.constraints`, Spring Boot 3.x trở lên dùng `jakarta.validation.constraints`.

### Một số annotation validation field thường gặp

Quy chuẩn Bean Validation và các bản triển khai (như Hibernate Validator) cung cấp phong phú các annotation, dùng để định nghĩa quy tắc validation dạng khai báo. Dưới đây là một số annotation thường gặp và giải thích:

- `@NotNull`: Kiểm tra phần tử được gắn annotation (bất kỳ kiểu nào) không được là `null`.
- `@NotEmpty`: Kiểm tra phần tử được gắn annotation (như `CharSequence`, `Collection`, `Map`, `Array`) không được là `null` và size/độ dài của nó không được bằng 0. Lưu ý: Đối với String, `@NotEmpty` cho phép chuỗi chứa ký tự khoảng trắng, như `" "`.
- `@NotBlank`: Kiểm tra `CharSequence` được gắn annotation (như `String`) không được là `null`, và độ dài sau khi loại bỏ khoảng trắng đầu cuối phải lớn hơn 0. (Tức là không được là chuỗi khoảng trắng).
- `@Null`: Kiểm tra phần tử được gắn annotation phải là `null`.
- `@AssertTrue` / `@AssertFalse`: Kiểm tra phần tử kiểu `boolean` hoặc `Boolean` được gắn annotation phải là `true` / `false`.
- `@Min(value)` / `@Max(value)`: Kiểm tra giá trị của kiểu số được gắn annotation (hoặc biểu diễn dạng chuỗi của nó) phải lớn hơn hoặc bằng / nhỏ hơn hoặc bằng `value` được chỉ định. Áp dụng cho kiểu số nguyên (`byte`, `short`, `int`, `long`, `BigInteger`,...).
- `@DecimalMin(value)` / `@DecimalMax(value)`: Tính năng tương tự `@Min` / `@Max`, nhưng áp dụng cho kiểu số chứa phần thập phân (`BigDecimal`, `BigInteger`, `CharSequence`, `byte`, `short`, `int`, `long` và wrapper class của chúng). `value` phải là biểu diễn dạng chuỗi của số.
- `@Size(min=, max=)`: Kiểm tra size/độ dài của phần tử được gắn annotation (như `CharSequence`, `Collection`, `Map`, `Array`) phải nằm trong phạm vi `min` và `max` được chỉ định (bao gồm cả biên).
- `@Digits(integer=, fraction=)`: Kiểm tra giá trị của kiểu số được gắn annotation (hoặc biểu diễn dạng chuỗi của nó), số chữ số phần nguyên phải ≤ `integer`, số chữ số phần thập phân phải ≤ `fraction`.
- `@Pattern(regexp=, flags=)`: Kiểm tra `CharSequence` được gắn annotation (như `String`) có khớp với regex (`regexp`) được chỉ định hay không. `flags` có thể chỉ định mode khớp (như không phân biệt chữ hoa chữ thường).
- `@Email`: Kiểm tra `CharSequence` được gắn annotation (như `String`) có phù hợp với định dạng Email hay không (tích hợp sẵn một regex tương đối nới lỏng).
- `@Past` / `@Future`: Kiểm tra kiểu date hoặc time được gắn annotation (`java.util.Date`, `java.util.Calendar`, các kiểu dưới package JSR 310 `java.time`) có nằm trước / nằm sau thời gian hiện tại hay không.
- `@PastOrPresent` / `@FutureOrPresent`: Tương tự `@Past` / `@Future`, nhưng cho phép bằng thời gian hiện tại.
- ……

### Validation Request Body (RequestBody)

Khi Controller method sử dụng annotation `@RequestBody` để nhận request body và bind nó vào một object, có thể thêm annotation `@Valid` trước parameter đó để kích hoạt validation đối với object đó. Nếu validation thất bại, nó sẽ ném ra `MethodArgumentNotValidException`.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    @NotNull(message = "classId không được để trống")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name không được để trống")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "giá trị sex không nằm trong phạm vi lựa chọn")
    @NotNull(message = "sex không được để trống")
    private String sex;

    @Email(message = "định dạng email không đúng")
    @NotNull(message = "email không được để trống")
    private String email;
}


@RestController
@RequestMapping("/api")
public class PersonController {
    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```

### Validation Request Parameter (Path Variables và Request Parameters)

Đối với dữ liệu kiểu đơn giản trực tiếp ánh xạ vào method parameter (như path variable `@PathVariable` hoặc request parameter `@RequestParam`), phương thức validation sẽ khác nhau tùy theo phiên bản Spring Framework:

1. **Spring Framework 6.1 trở lên**: Spring MVC tích hợp sẵn hỗ trợ Handler Method Validation. Đặt trực tiếp các annotation ràng buộc như `@Min`, `@Max`, `@Size`, `@Pattern` lên method parameter là được, không thêm `@Validated` trên Controller class, nếu không sẽ chuyển sang dùng validation method dựa trên AOP.
2. **Spring Framework 6.0 trở về trước**: Thường cần thêm `@Validated` do Spring cung cấp trên Controller class, xử lý ràng buộc parameter thông qua hạ tầng validation method.

Dưới đây lấy ví dụ phương thức validation tích hợp sẵn của Spring Framework 6.1 trở lên:

```java
@RestController
@RequestMapping("/api")
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(
            @PathVariable("id")
            @Max(value = 5, message = "ID không được quá 5")
            Integer id
    ) {
        // Spring MVC 6.1+ sẽ ném ra HandlerMethodValidationException trước khi vào method body.
        return ResponseEntity.ok().body(id);
    }

    @GetMapping("/person")
    public ResponseEntity<String> findPersonByName(
            @RequestParam("name")
            @NotBlank(message = "Họ tên không được để trống") // Cũng áp dụng cho @RequestParam
            @Size(max = 10, message = "Độ dài họ tên không được quá 10")
            String name
    ) {
        return ResponseEntity.ok().body("Found person: " + name);
    }
}
```

## Xử lý Exception toàn cục

Giới thiệu về việc xử lý exception tầng Controller toàn cục không thể thiếu trong dự án Spring của chúng ta.

**Annotation liên quan:**

1. `@ControllerAdvice`: Annotation định nghĩa class xử lý exception toàn cục
2. `@ExceptionHandler`: Annotation khai báo method xử lý exception

Sử dụng như thế nào? Lấy ví dụ ở phần validation parameter trong mục 5 của chúng ta. Nếu method parameter không đúng thì sẽ ném ra `MethodArgumentNotValidException`, chúng ta sẽ xử lý exception này.

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    /**
     * Xử lý exception request parameter
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex, HttpServletRequest request) {
       ......
    }
}
```

Xem thêm nội dung về xử lý exception trong Spring Boot tại hai bài viết này của tôi:

1. [SpringBoot 处理异常的几种常见姿势](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485568&idx=2&sn=c5ba880fd0c5d82e39531fa42cb036ac&chksm=cea2474bf9d5ce5dcbc6a5f6580198fdce4bc92ef577579183a729cb5d1430e4994720d59b34&token=2133161636&lang=zh_CN#rd)
2. [使用枚举简单封装一个优雅的 Spring Boot 全局异常处理！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486379&idx=2&sn=48c29ae65b3ed874749f0803f0e4d90e&chksm=cea24460f9d5cd769ed53ad7e17c97a7963a89f5350e370be633db0ae8d783c3a3dbd58c70f8&token=1054498516&lang=zh_CN#rd)

## Transaction

Trên method cần bật transaction chỉ cần sử dụng annotation `@Transactional` là được!

```java
@Transactional(rollbackFor = Exception.class)
public void save() {
  ......
}

```

Chúng ta biết Exception chia thành RuntimeException và non-RuntimeException. Trong annotation `@Transactional` nếu không cấu hình thuộc tính `rollbackFor`, thì transaction chỉ khi gặp `RuntimeException` mới rollback, cộng thêm `rollbackFor=Exception.class`, có thể giúp transaction khi gặp non-RuntimeException cũng sẽ rollback.

Annotation `@Transactional` thường có thể tác động lên `class` hoặc `method`.

- **Tác động lên class**: Khi đặt annotation `@Transactional` lên class, thể hiện tất cả các method public của class đó đều được cấu hình thông tin thuộc tính transaction giống nhau.
- **Tác động lên method**: Khi class cấu hình `@Transactional`, method cũng cấu hình `@Transactional`, transaction của method sẽ ghi đè (override) thông tin cấu hình transaction của class.

Xem thêm nội dung về Spring Transaction tại bài viết này của tôi: [Giải thích chi tiết Spring Transaction](./spring-transaction.md) .

## JPA

Spring Data JPA cung cấp một loạt các annotation và tính năng, giúp lập trình viên dễ dàng thực hiện ORM (Object-Relational Mapping).

### Tạo bảng

`@Entity` dùng để khai báo một class là JPA entity class, ánh xạ với bảng trong cơ sở dữ liệu. `@Table` chỉ định tên bảng tương ứng với entity.

```java
@Entity
@Table(name = "role")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;

    // Bỏ qua getter/setter
}
```

### Chiến lược sinh Primary Key

`@Id` khai báo field là primary key. `@GeneratedValue` chỉ định chiến lược sinh primary key.

Jakarta Persistence 3.1 cung cấp 5 loại chiến lược sinh primary key:

- **`GenerationType.TABLE`**: Sinh primary key thông qua bảng cơ sở dữ liệu.
- **`GenerationType.SEQUENCE`**: Sinh primary key thông qua sequence của cơ sở dữ liệu (áp dụng cho các cơ sở dữ liệu như Oracle).
- **`GenerationType.IDENTITY`**: Primary key tự động tăng (áp dụng cho các cơ sở dữ liệu như MySQL).
- **`GenerationType.UUID`**: Sinh RFC 4122 UUID, áp dụng cho primary key kiểu `UUID` hoặc `String`.
- **`GenerationType.AUTO`**: Do JPA tự động chọn chiến lược sinh thích hợp (chiến lược mặc định).

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Khai báo chiến lược sinh primary key tùy chỉnh thông qua `@GenericGenerator`:

```java
@Id
@GeneratedValue(generator = "IdentityIdGenerator")
@GenericGenerator(name = "IdentityIdGenerator", strategy = "identity")
private Long id;
```

Tương đương với:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Dưới đây là đoạn mã nguồn trích xuất từ bản thực thi bên trong Hibernate cũ, hiển thị các chiến lược generator chuỗi mà Hibernate hỗ trợ lúc đó. Nó không thuộc về API tiêu chuẩn của JPA/Jakarta Persistence, và cũng không thể thay thế cho enum `GenerationType` tiêu chuẩn ở trên; dự án mới nên lấy tài liệu chính thức của phiên bản Hibernate đang dùng làm chuẩn.

```java
public class DefaultIdentifierGeneratorFactory
    implements MutableIdentifierGeneratorFactory, Serializable, ServiceRegistryAwareService {

  @SuppressWarnings("deprecation")
  public DefaultIdentifierGeneratorFactory() {
    register( "uuid2", UUIDGenerator.class );
    register( "guid", GUIDGenerator.class );      // can be done with UUIDGenerator + strategy
    register( "uuid", UUIDHexGenerator.class );      // "deprecated" for new use
    register( "uuid.hex", UUIDHexGenerator.class );   // uuid.hex is deprecated
    register( "assigned", Assigned.class );
    register( "identity", IdentityGenerator.class );
    register( "select", SelectGenerator.class );
    register( "sequence", SequenceStyleGenerator.class );
    register( "seqhilo", SequenceHiLoGenerator.class );
    register( "increment", IncrementGenerator.class );
    register( "foreign", ForeignGenerator.class );
    register( "sequence-identity", SequenceIdentityGenerator.class );
    register( "enhanced-sequence", SequenceStyleGenerator.class );
    register( "enhanced-table", TableGenerator.class );
  }

  public void register(String strategy, Class generatorClass) {
    LOG.debugf( "Registering IdentifierGenerator strategy [%s] -> [%s]", strategy, generatorClass.getName() );
    final Class previous = generatorStrategyToClassNameMap.put( strategy, generatorClass );
    if ( previous != null ) {
      LOG.debugf( "    - overriding [%s]", previous.getName() );
    }
  }

}
```

### Ánh xạ Field

`@Column` dùng để chỉ định mối quan hệ ánh xạ giữa entity field và cột trong cơ sở dữ liệu.

- **`name`**: Chỉ định tên cột trong cơ sở dữ liệu.
- **`nullable`**: Chỉ định có cho phép là `null` hay không.
- **`length`**: Thiết lập độ dài của field (chỉ áp dụng cho kiểu `String`).
- **`columnDefinition`**: Chỉ định kiểu cơ sở dữ liệu và giá trị mặc định của field.

```java
@Column(name = "user_name", nullable = false, length = 32)
private String userName;

@Column(columnDefinition = "tinyint(1) default 1")
private Boolean enabled;
```

### Bỏ qua Field

`@Transient` dùng để khai báo field không cần lưu trữ (persistent).

```java
@Entity
public class User {

    @Transient
    private String temporaryField; // Sẽ không ánh xạ vào bảng cơ sở dữ liệu
}
```

Các cách khác làm field không bị lưu trữ:

- **`static`**: Field static sẽ không bị lưu trữ.
- **`final`**: Field final sẽ không bị lưu trữ.
- **`transient`**: Field sử dụng từ khóa `transient` của Java sẽ không bị tuần tự hóa hoặc lưu trữ.

### Lưu trữ Field lớn

`@Lob` dùng để khai báo field lớn (như `CLOB` hoặc `BLOB`).

```java
@Lob
@Column(name = "content", columnDefinition = "LONGTEXT NOT NULL")
private String content;
```

### Ánh xạ Enum Type

`@Enumerated` dùng để ánh xạ Enum type thành field cơ sở dữ liệu.

- **`EnumType.ORDINAL`**: Lưu số thứ tự của enum (mặc định).
- **`EnumType.STRING`**: Lưu tên của enum (khuyên dùng).

```java
public enum Gender {
    MALE,
    FEMALE
}

@Entity
public class User {

    @Enumerated(EnumType.STRING)
    private Gender gender;
}
```

Giá trị lưu trong cơ sở dữ liệu là `MALE` hoặc `FEMALE`.

### Tính năng Audit

Thông qua tính năng Audit của JPA, có thể tự động ghi lại các thông tin như thời gian tạo, thời gian update, người tạo và người update trong entity.

Class nền tảng Audit:

```java
@Data
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate
    @Column(updatable = false)
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;
}
```

Cấu hình tính năng Audit:

```java
@Configuration
@EnableJpaAuditing
public class AuditConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(Authentication::isAuthenticated)
                .map(Authentication::getName);
    }
}
```

Giới thiệu đơn giản một số annotation liên quan ở trên:

1. `@CreatedDate`: Thể hiện field này là field thời gian tạo, khi entity này được insert, sẽ thiết lập giá trị
2. `@CreatedBy`: Thể hiện field này là người tạo, khi entity này được insert, sẽ thiết lập giá trị. `@LastModifiedDate`, `@LastModifiedBy` tương tự.
3. `@EnableJpaAuditing`: Bật tính năng JPA audit.

### Thao tác Update và Delete

`@Modifying` dùng để đánh dấu câu lệnh khai báo bởi `@Query` là thao tác sửa đổi như INSERT, UPDATE, DELETE hoặc DDL. Method delete dẫn xuất (ví dụ `deleteByUserName`) không cần `@Modifying`. Ranh giới transaction vừa có thể khai báo trên Repository method, vừa có thể do unit of work của Service cấp trên quản lý thống nhất.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Modifying
    @Transactional
    @Query("delete from User user where user.userName = :userName")
    int deleteByUserName(@Param("userName") String userName);
}
```

### Quan hệ liên kết

JPA cung cấp 4 loại annotation quan hệ liên kết:

- **`@OneToOne`**: Quan hệ 1 - 1.
- **`@OneToMany`**: Quan hệ 1 - Nhiều.
- **`@ManyToOne`**: Quan hệ Nhiều - 1.
- **`@ManyToMany`**: Quan hệ Nhiều - Nhiều.

```java
@Entity
public class User {

    @OneToOne
    private Profile profile;

    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}
```

## Xử lý dữ liệu JSON

Trong phát triển Web, thường xuyên cần xử lý chuyển đổi giữa Java object và định dạng JSON. Spring thường tích hợp thư viện Jackson để hoàn thành nhiệm vụ này, dưới đây là một số annotation Jackson thường gặp, có thể giúp chúng ta tùy chỉnh quá trình serialization (Java object sang JSON) và deserialization (JSON sang Java object) của JSON.

### Lọc field JSON

Đôi khi chúng ta không muốn một số field của Java object được bao gồm trong JSON sinh ra cuối cùng, hoặc khi chuyển đổi JSON sang Java object không xử lý một số thuộc tính JSON.

`@JsonIgnoreProperties` tác động lên class dùng để lọc bỏ các field cụ thể không trả về hoặc không parse.

```java
// Khi sinh JSON bỏ qua thuộc tính userRoles
// Nếu cho phép thuộc tính không xác định (tức thuộc tính có trong JSON mà không có trong class), có thể thêm ignoreUnknown = true
@JsonIgnoreProperties({"userRoles"})
public class User {
    private String userName;
    private String fullName;
    private String password;
    private List<UserRole> userRoles = new ArrayList<>();
    // getters and setters...
}
```

`@JsonIgnore` tác động ở cấp độ field hoặc method `getter/setter`, dùng để chỉ định bỏ qua thuộc tính cụ thể đó khi serialization hoặc deserialization.

```java
public class User {
    private String userName;
    private String fullName;
    private String password;

    // Khi sinh JSON bỏ qua thuộc tính userRoles
    @JsonIgnore
    private List<UserRole> userRoles = new ArrayList<>();
    // getters and setters...
}
```

`@JsonIgnoreProperties` thích hợp hơn khi định nghĩa class loại trừ rõ ràng nhiều field, hoặc loại trừ field trong kịch bản kế thừa; còn `@JsonIgnore` thì trực tiếp hơn khi dùng để đánh dấu một field cụ thể đơn lẻ.

### Định dạng dữ liệu JSON

`@JsonFormat` dùng để chỉ định định dạng của thuộc tính khi serialization và deserialization. Thường dùng định dạng kiểu date time.

Ví dụ:

```java
// Chỉ định kiểu Date serialize thành chuỗi định dạng ISO 8601, và thiết lập timezone là GMT
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", timezone = "GMT")
private Date date;
```

### Làm phẳng (Flat) JSON Object

Annotation `@JsonUnwrapped` tác động lên field, dùng để khi serialization sẽ "nâng" các thuộc tính của object lồng nhau của nó lên cấp độ của object hiện tại, khi deserialization thực thi thao tác ngược lại. Điều này có thể làm cấu trúc JSON phẳng hơn.

Giả sử có class `Account`, chứa hai object lồng nhau là `Location` và `PersonInfo`.

```java
@Getter
@Setter
@ToString
public class Account {
    private Location location;
    private PersonInfo personInfo;

  @Getter
  @Setter
  @ToString
  public static class Location {
     private String provinceName;
     private String countyName;
  }
  @Getter
  @Setter
  @ToString
  public static class PersonInfo {
    private String userName;
    private String fullName;
  }
}

```

Cấu trúc JSON trước khi làm phẳng:

```json
{
  "location": {
    "provinceName": "湖北",
    "countyName": "武汉"
  },
  "personInfo": {
    "userName": "coder1234",
    "fullName": "shaungkou"
  }
}
```

Sử dụng `@JsonUnwrapped` làm phẳng object:

```java
@Getter
@Setter
@ToString
public class Account {
    @JsonUnwrapped
    private Location location;
    @JsonUnwrapped
    private PersonInfo personInfo;
    ......
}
```

Cấu trúc JSON sau khi làm phẳng:

```json
{
  "provinceName": "湖北",
  "countyName": "武汉",
  "userName": "coder1234",
  "fullName": "shaungkou"
}
```

## Testing

`@ActiveProfiles` thường tác động lên test class, dùng để khai báo file cấu hình Spring có hiệu lực.

```java
// Chỉ định khởi động application context trên RANDOM_PORT, và kích hoạt profile "test"
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
@Slf4j
public abstract class TestBase {
    // Common test setup or abstract methods...
}
```

`@Test` là annotation do framework JUnit (thường là JUnit 5 Jupiter) cung cấp, dùng để đánh dấu một method là test method. Mặc dù không phải là annotation của bản thân Spring, nhưng nó là nền tảng để thực thi unit test và integration test.

Các test method `@Transactional` do Spring TestContext quản lý trong test thread mặc định sẽ rollback sau khi test kết thúc, tránh làm bẩn dữ liệu test. Cần lưu ý, nếu sử dụng `RANDOM_PORT` gửi HTTP request thực sự, server side xử lý chạy trên một thread và transaction khác, sẽ không tự động rollback cùng với transaction của test thread, lúc này cần sử dụng cơ sở dữ liệu cô lập hoặc dọn dẹp dữ liệu tường minh.

`@WithMockUser` là annotation do module Spring Security Test cung cấp, dùng để mô phỏng một user đã được xác thực trong thời gian test. Có thể tiện lợi chỉ định username, password, role (authorities),... từ đó test các endpoint hoặc method được bảo vệ an toàn.

```java
public class MyServiceTest extends TestBase { // Assuming TestBase provides Spring context

    @Test
    @Transactional // Dữ liệu test sẽ rollback
    @WithMockUser(username = "test-user", authorities = { "ROLE_TEACHER", "read" }) // Mô phỏng một user tên "test-user", có role TEACHER và quyền read
    void should_perform_action_requiring_teacher_role() throws Exception {
        // ... Logic test ...
        // Ở đây có thể gọi method service cần quyền "ROLE_TEACHER"
    }
}
```

<!-- @include: @article-footer.snippet.md -->
