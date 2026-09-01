---
title: Giải thích chi tiết cơ chế Java SPI
description: Phân tích toàn diện nguyên lý và ứng dụng của cơ chế Java SPI: hiểu rõ cơ chế Service Discovery ServiceLoader, ứng dụng của SPI trong JDBC/Dubbo/Spring, so sánh với API và các best practice.
category: Java
tag:
  - Java cơ bản
head:
  - - meta
    - name: keywords
      content: Java SPI,cơ chế SPI,ServiceLoader,service discovery,pluginization,nạp driver JDBC,mở rộng Dubbo,ứng dụng SPI
---

> Bài viết này được đóng góp bởi [Kingshion](https://github.com/jjx0708). Chào mừng thêm nhiều bạn cùng tham gia vào công việc bảo trì JavaGuide, đây là một việc vô cùng có ý nghĩa. Thông tin chi tiết xin xem: [Hướng dẫn đóng góp JavaGuide](https://javaguide.cn/javaguide/contribution-guideline.html).

Thiết kế hướng đối tượng khuyến khích lập trình dựa trên Interface thay vì Implementation cụ thể giữa các module để giảm độ gián tiếp (coupling) giữa các module, tuân thủ nguyên tắc Đảo ngược phụ thuộc (Dependency Inversion Principle), và hỗ trợ nguyên tắc Đóng/Mở (Open/Closed Principle - mở cho mở rộng, đóng cho sửa đổi). Tuy nhiên, phụ thuộc trực tiếp vào bản thực thi cụ thể sẽ khiến việc thay thế implementation phải sửa đổi code, vi phạm nguyên tắc Đóng/Mở. Để giải quyết vấn đề này, SPI ra đời, cung cấp một cơ chế Service Discovery cho phép chỉ định động bản thực thi cụ thể bên ngoài chương trình. Điều này tương tự với tư tưởng Inversion of Control (IoC), chuyển giao quyền kiểm soát lắp ráp component ra bên ngoài chương trình.

Cơ chế SPI cũng giải quyết hạn chế do mô hình [Parent Delegation Model (Ủy quyền kép)](https://javaguide.cn/java/jvm/classloader.html) mang lại trong hệ thống class loading của Java. Mặc dù Parent Delegation Model đảm bảo tính an toàn và nhất quán của thư viện cốt lõi, nhưng cũng hạn chế thư viện cốt lõi hoặc thư viện mở rộng nạp các class trên classpath của ứng dụng (thường do bên thứ ba implement). SPI cho phép thư viện cốt lõi hoặc mở rộng định nghĩa Service Interface, các nhà phát triển bên thứ ba cung cấp và triển khai implementation, cơ chế nạp dịch vụ SPI sẽ tự động phát hiện và nạp các implementation này lúc runtime. Ví dụ, JDBC từ phiên bản 4.0 trở đi tận dụng SPI để tự động phát hiện và nạp driver database, nhà phát triển chỉ cần đặt file JAR driver dưới classpath là được, không cần dùng `Class.forName()` để nạp class driver một cách thủ công.

## Giới thiệu SPI

### SPI là gì?

SPI là viết tắt của Service Provider Interface, nghĩa trên mặt chữ là: "Interface của nhà cung cấp dịch vụ", theo cách hiểu của tôi là: Một interface chuyên cung cấp cho các nhà cung cấp dịch vụ hoặc các nhà phát triển mở rộng tính năng của framework sử dụng.

SPI tách biệt Service Interface và các Service Implementation cụ thể, giúp giảm phụ thuộc giữa bên gọi dịch vụ và bên thực thi dịch vụ, có thể nâng cao tính mở rộng và tính bảo trì của chương trình. Việc sửa đổi hoặc thay thế bản thực thi dịch vụ không cần phải sửa đổi code của bên gọi.

Rất nhiều framework đều sử dụng cơ chế SPI của Java, ví dụ: Spring framework, driver nạp database, interface logging, cũng như bản thực thi mở rộng của Dubbo,...

<img src="https://oss.javaguide.cn/github/javaguide/java/basis/spi/22e1830e0b0e4115a882751f6c417857tplv-k3u1fbpfcp-zoom-1.jpeg" style="zoom:50%;" />

### Sự khác biệt giữa SPI và API là gì?

**Vậy SPI và API có điểm gì khác nhau?**

Nói đến SPI thì không thể không nhắc tới API (Application Programming Interface), về nghĩa rộng chúng đều thuộc về interface, và rất dễ bị nhầm lẫn. Dưới đây dùng một hình vẽ để giải thích trước:

![SPI VS API](https://oss.javaguide.cn/github/javaguide/java/basis/spi-vs-api.png)

Thông thường giữa các module giao tiếp với nhau qua interface, do đó chúng ta đưa vào một "interface" nằm giữa bên gọi dịch vụ và bên thực thi dịch vụ (còn gọi là bên cung cấp dịch vụ).

- Khi bên thực thi cung cấp cả interface và implementation, chúng ta có thể gọi interface của bên thực thi để sở hữu khả năng mà bên thực thi cung cấp cho chúng ta, đó chính là **API**. Trong trường hợp này, interface và implementation đều nằm trong package của bên thực thi. Bên gọi thông qua interface gọi tính năng của bên thực thi mà không cần quan tâm đến chi tiết thực thi cụ thể.
- Khi interface nằm ở phía bên gọi, đó chính là **SPI**. Do bên gọi interface quy định quy tắc interface, sau đó các hãng sản xuất khác nhau dựa trên quy tắc này để implement interface đó, từ đó cung cấp dịch vụ.

Lấy một ví dụ bình dị dễ hiểu: Công ty H là một công ty công nghệ, mới thiết kế một loại chip mới và bây giờ cần sản xuất hàng loạt, mà trên thị trường có vài công ty sản xuất chip, lúc này chỉ cần công ty H quy định xong tiêu chuẩn sản xuất chip này (định nghĩa xong tiêu chuẩn interface), thì các công ty chip hợp tác này (bên cung cấp dịch vụ) sẽ bàn giao chip mang đặc trưng riêng của mình theo tiêu chuẩn (cung cấp bản thực thi theo các phương án khác nhau, nhưng kết quả đưa ra là như nhau).

## Demo thực hành

SLF4J (Simple Logging Facade for Java) là một Logging Facade (Interface) của Java, bản thực thi cụ thể của nó có vài loại như Logback, Log4j, Log4j2,... và còn có thể chuyển đổi, khi chuyển đổi bản thực thi logging cụ thể chúng ta không cần sửa code dự án, chỉ cần sửa đổi một số pom dependency trong Maven dependency là được.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/image-20220723213306039-165858318917813.png)

Đây chính là nhờ cơ chế SPI thực hiện, tiếp theo chúng ta hãy thực hiện một phiên bản đơn giản của logging framework.

### Service Provider Interface

Tạo mới dự án Java `service-provider-interface` có cấu trúc thư mục như sau: (Lưu ý tạo trực tiếp Java project là được, không cần tạo Maven project, Maven project sẽ liên quan tới một số cấu hình biên dịch, nếu có Nexus private thì deploy trực tiếp sẽ tiện hơn, nhưng nếu không có thì trong quá trình làm có thể gặp một số vấn đề kỳ lạ.)

```plain
│  service-provider-interface.iml
│
├─.idea
│  │  .gitignore
│  │  misc.xml
│  │  modules.xml
│  └─ workspace.xml
│
└─src
    └─edu
        └─jiangxuan
            └─up
                └─spi
                        Logger.java
                        LoggerService.java
                        Main.class
```

Tạo mới interface `Logger`, đây chính là SPI - Service Provider Interface, các service provider sau này sẽ phải implement interface này.

```java
package edu.jiangxuan.up.spi;

public interface Logger {
    void info(String msg);
    void debug(String msg);
}
```

Tiếp theo là lớp `LoggerService`, lớp này chủ yếu cung cấp tính năng đặc thù cho người sử dụng dịch vụ (bên gọi). Lớp này cũng là điểm mấu chốt để thực hiện cơ chế Java SPI, nếu có thắc mắc bạn có thể tiếp tục xem phần sau trước.

```java
package edu.jiangxuan.up.spi;

import java.util.ArrayList;
import java.util.List;
import java.util.ServiceLoader;

public class LoggerService {
    private static final LoggerService SERVICE = new LoggerService();

    private final Logger logger;

    private final List<Logger> loggerList;

    private LoggerService() {
        ServiceLoader<Logger> loader = ServiceLoader.load(Logger.class);
        List<Logger> list = new ArrayList<>();
        for (Logger log : loader) {
            list.add(log);
        }
        // loggerList là tất cả ServiceProvider
        loggerList = list;
        if (!list.isEmpty()) {
            // Logger chỉ lấy một cái
            logger = list.get(0);
        } else {
            logger = null;
        }
    }

    public static LoggerService getService() {
        return SERVICE;
    }

    public void info(String msg) {
        if (logger == null) {
            System.out.println("Trong info không tìm thấy Logger service provider");
        } else {
            logger.info(msg);
        }
    }

    public void debug(String msg) {
        if (loggerList.isEmpty()) {
            System.out.println("Trong debug không tìm thấy Logger service provider");
        }
        loggerList.forEach(log -> log.debug(msg));
    }
}
```

Tạo mới lớp `Main` (người sử dụng dịch vụ, bên gọi), chạy chương trình xem kết quả.

```java
package org.spi.service;

public class Main {
    public static void main(String[] args) {
        LoggerService service = LoggerService.getService();

        service.info("Hello SPI");
        service.debug("Hello SPI");
    }
}
```

Kết quả chương trình:

> Trong info không tìm thấy Logger service provider
> Trong debug không tìm thấy Logger service provider

Lúc này chúng ta mới chỉ có interface rỗng, chưa cung cấp bất kỳ implementation nào cho interface `Logger`, do đó kết quả in ra không theo kỳ vọng.

Bạn có thể dùng lệnh hoặc dùng trực tiếp IDEA đóng gói toàn bộ chương trình thành file jar.

### Service Provider

Tiếp theo tạo một project mới dùng để implement interface `Logger`.

Tạo dự án mới `service-provider` có cấu trúc thư mục như sau:

```plain
│  service-provider.iml
│
├─.idea
│  │  .gitignore
│  │  misc.xml
│  │  modules.xml
│  └─ workspace.xml
│
├─lib
│      service-provider-interface.jar
|
└─src
    ├─edu
    │  └─jiangxuan
    │      └─up
    │          └─spi
    │              └─service
    │                      Logback.java
    │
    └─META-INF
        └─services
                edu.jiangxuan.up.spi.Logger

```

Tạo mới lớp `Logback`

```java
package edu.jiangxuan.up.spi.service;

import edu.jiangxuan.up.spi.Logger;

public class Logback implements Logger {
    @Override
    public void info(String s) {
        System.out.println("Logback info in log: " + s);
    }

    @Override
    public void debug(String s) {
        System.out.println("Logback debug in log: " + s);
    }
}
```

Import file jar `service-provider-interface` vào dự án.

Tạo mới thư mục lib, sau đó copy file jar sang, rồi thêm vào dự án.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/523d5e25198444d3b112baf68ce49daetplv-k3u1fbpfcp-watermark.png)

Tiếp tục click OK.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/f4ba0aa71e9b4d509b9159892a220850tplv-k3u1fbpfcp-watermark.png)

Tiếp theo là có thể import một số class và method trong file jar vào dự án, giống như import các package utility class của JDK.

Implement interface `Logger`, tạo mới thư mục `META-INF/services` dưới thư mục `src`, sau đó tạo mới file `edu.jiangxuan.up.spi.Logger` (tên lớp đầy đủ của SPI), nội dung bên trong file là: `edu.jiangxuan.up.spi.service.Logback` (tên lớp đầy đủ của Logback, tức package name + class name của lớp implementation của SPI).

**Đây là chuẩn đã được quy ước của ServiceLoader trong cơ chế JDK SPI.**

Ở đây giải thích qua một chút: Khi gọi `ServiceLoader.load()` sẽ tạo ra Service Loader. Khi duyệt `ServiceLoader`, loader sẽ định vị và khởi tạo provider theo nhu cầu; khi gọi `stream()` thu được luồng `ServiceLoader.Provider`, có thể kiểm tra kiểu provider qua `type()` trước, chỉ khi gọi `Provider.get()` mới lấy instance service provider tương ứng. `ServiceLoader` sẽ cache lại các provider đã được nạp. Đối với provider trên classpath, file cấu hình nằm tại `META-INF/services`; đối với named module từ Java 9 trở đi, còn có thể khai báo quan hệ dịch vụ qua `uses` và `provides ... with ...` trong module descriptor.

Do đó sẽ đưa ra một số yêu cầu quy phạm: Tên file bắt buộc phải là tên lớp đầy đủ của interface, nội dung bên trong bắt buộc phải là tên lớp đầy đủ của lớp implementation, lớp implementation có thể có nhiều cái, trực tiếp xuống dòng là được, khi có nhiều lớp implementation sẽ được lặp nạp từng cái một.

Tiếp theo cũng đóng gói project `service-provider` thành file jar, file jar này chính là bản thực thi của phía cung cấp dịch vụ. Thông thường chúng ta import maven pom dependency cũng tương tự như vậy, chỉ là hiện tại chúng ta chưa publish file jar này lên maven public repository, nên ở những nơi cần dùng chỉ có thể thêm thủ công vào dự án.

### Trình diễn kết quả

Để trình diễn kết quả trực quan hơn, ở đây tôi tạo thêm một project chuyên dùng để test: `java-spi-test`

Sau đó import file jar interface của `Logger` trước, rồi import tiếp file jar implementation class cụ thể.

![](https://oss.javaguide.cn/github/javaguide/java/basis/spi/image-20220723215812708-165858469599214.png)

Tạo mới phương thức Main để test:

```java
package edu.jiangxuan.up.service;

import edu.jiangxuan.up.spi.LoggerService;

public class TestJavaSPI {
    public static void main(String[] args) {
        LoggerService loggerService = LoggerService.getService();
        loggerService.info("Xin chào");
        loggerService.debug("Test cơ chế Java SPI");
    }
}
```

Kết quả chạy như sau:

> Logback info in log: Xin chào
> Logback debug in log: Test cơ chế Java SPI

Cho thấy lớp implementation trong file jar import vào đã có hiệu lực.

Nếu chúng ta không import file jar của lớp implementation cụ thể, thì kết quả chạy chương trình lúc này sẽ là:

> Trong info không tìm thấy Logger service provider
> Trong debug không tìm thấy Logger service provider

Thông qua việc sử dụng cơ chế SPI, có thể thấy độ phụ thuộc giữa dịch vụ (`LoggerService`) và phía cung cấp dịch vụ là rất thấp. Nếu chúng ta muốn đổi sang một bản thực thi khác, thì thực tế chỉ cần sửa đổi bản thực thi cụ thể cho interface `Logger` trong project `service-provider` là được, chỉ cần đổi một file jar khác là xong, cũng có thể trong một project có nhiều bản thực thi, đây chẳng phải là nguyên lý của SLF4J sao?

Nếu một ngày nào đó yêu cầu thay đổi, lúc này cần xuất log ra Message Queue, hoặc làm một số thao tác khác, lúc này hoàn toàn không cần sửa đổi bản thực thi của Logback, chỉ cần thêm một service implementation (service-provider) mới, có thể thông qua việc thêm implementation trong project này hoặc từ bên ngoài đưa vào file jar service implementation mới. Chúng ta có thể chọn một Service Implementation (service-provider) cụ thể trong dịch vụ (LoggerService) để hoàn thành thao tác mình cần.

Vậy tiếp theo chúng ta sẽ nói cụ thể về nguyên lý trọng tâm của Java SPI — **ServiceLoader**.

## ServiceLoader

### Bản thực thi cụ thể của ServiceLoader

> Dưới đây hiển thị bản thực thi `ServiceLoader` của JDK 8. Phiên bản Java 9 trở đi đã bổ sung thêm Module Layer, `stream()`, `findFirst()` và provider method,... cấu trúc mã nguồn hiện tại đã khác.

Muốn sử dụng cơ chế SPI của Java cần phải phụ thuộc vào `ServiceLoader` để thực hiện, tiếp theo chúng ta cùng xem `ServiceLoader` cụ thể làm như thế nào:

`ServiceLoader` là một utility class do JDK cung cấp, nằm trong package `java.util`.

```plain
A facility to load implementations of a service.
```

Đây là chú thích chính thức của JDK: **Một công cụ dùng để nạp các bản thực thi của dịch vụ.**

Nhìn tiếp xuống dưới, chúng ta thấy lớp này là loại `final`, nên không thể kế thừa sửa đổi, đồng thời nó implement interface `Iterable`. Sở dĩ implement iterator là để thuận tiện cho chúng ta về sau có thể thu được bản thực thi dịch vụ tương ứng thông qua cách lặp (iteration).

```java
public final class ServiceLoader<S> implements Iterable<S>{ xxx...}
```

Có thể thấy một định nghĩa hằng số quen thuộc:

`private static final String PREFIX = "META-INF/services/";`

Dưới đây là phương thức `load`: Có thể thấy phương thức `load` hỗ trợ 2 overload tham số đầu vào:

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}

public static <S> ServiceLoader<S> load(Class<S> service,
                                        ClassLoader loader) {
    return new ServiceLoader<>(service, loader);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}

public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}
```

Cơ chế giải quyết việc nạp class bên thứ ba của nó thực chất nằm ở `ClassLoader cl = Thread.currentThread().getContextClassLoader();`, `cl` chính là **Thread Context ClassLoader**. Đây là ClassLoader mà mỗi Thread nắm giữ, thiết kế của JDK cho phép ứng dụng hoặc container (như Web Application Server) thiết lập ClassLoader này để thư viện cốt lõi có thể thông qua nó để nạp các class của ứng dụng.

Thread Context ClassLoader mặc định là Application ClassLoader, chịu trách nhiệm nạp các class trên classpath. Khi thư viện cốt lõi cần nạp class do ứng dụng cung cấp, nó có thể dùng Thread Context ClassLoader để hoàn thành. Như vậy, ngay cả mã thư viện cốt lõi do Bootstrap ClassLoader nạp cũng có thể nạp và sử dụng các class do Application ClassLoader nạp.

Theo thứ tự gọi code, trong phương thức `reload()` được thực hiện thông qua một inner class `LazyIterator`. Tiếp tục xem xuống bên dưới.

Sau khi `ServiceLoader` implement phương thức của interface `Iterable`, nó có khả năng lặp, khi phương thức `iterator` này được gọi, đầu tiên sẽ tìm kiếm trong cache `Provider` của `ServiceLoader`, nếu trong cache không trúng thì tìm kiếm trong `LazyIterator`.

```java

public Iterator<S> iterator() {
    return new Iterator<S>() {

        Iterator<Map.Entry<String, S>> knownProviders
                = providers.entrySet().iterator();

        public boolean hasNext() {
            if (knownProviders.hasNext())
                return true;
            return lookupIterator.hasNext(); // Gọi LazyIterator
        }

        public S next() {
            if (knownProviders.hasNext())
                return knownProviders.next().getValue();
            return lookupIterator.next(); // Gọi LazyIterator
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

    };
}
```

Khi gọi `LazyIterator`, bản thực thi cụ thể như sau:

```java

public boolean hasNext() {
    if (acc == null) {
        return hasNextService();
    } else {
        PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
            public Boolean run() {
                return hasNextService();
            }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

private boolean hasNextService() {
    if (nextName != null) {
        return true;
    }
    if (configs == null) {
        try {
            // Thông qua PREFIX (META-INF/services/) và class name để lấy file cấu hình tương ứng, thu được implementation class cụ thể
            String fullName = PREFIX + service.getName();
            if (loader == null)
                configs = ClassLoader.getSystemResources(fullName);
            else
                configs = loader.getResources(fullName);
        } catch (IOException x) {
            fail(service, "Error locating configuration files", x);
        }
    }
    while ((pending == null) || !pending.hasNext()) {
        if (!configs.hasMoreElements()) {
            return false;
        }
        pending = parse(service, configs.nextElement());
    }
    nextName = pending.next();
    return true;
}


public S next() {
    if (acc == null) {
        return nextService();
    } else {
        PrivilegedAction<S> action = new PrivilegedAction<S>() {
            public S run() {
                return nextService();
            }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

private S nextService() {
    if (!hasNextService())
        throw new NoSuchElementException();
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service,
                "Provider " + cn + " not found");
    }
    if (!service.isAssignableFrom(c)) {
        fail(service,
                "Provider " + cn + " not a subtype");
    }
    try {
        S p = service.cast(c.newInstance());
        providers.put(cn, p);
        return p;
    } catch (Throwable x) {
        fail(service,
                "Provider " + cn + " could not be instantiated",
                x);
    }
    throw new Error();          // This cannot happen
}
```

Có thể nhiều người xem đoạn này sẽ thấy hơi phức tạp, không sao, ở đây tôi thực hiện một mô hình `ServiceLoader` nhỏ đơn giản, dùng để trình diễn tư tưởng cốt lõi đọc tên implementation class từ file cấu hình trên classpath và tạo instance. Nó chưa bao quát các hành vi đầy đủ như nạp lười (lazy loading), de-duplication, xử lý lỗi, module provider,... của bản thực thi JDK:

### Tự implement một ServiceLoader

Tôi đăng code ra trước:

```java
package edu.jiangxuan.up.service;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.lang.reflect.Constructor;
import java.net.URL;
import java.net.URLConnection;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

public class MyServiceLoader<S> {

    // Template Class của Interface tương ứng
    private final Class<S> service;

    // Các Implementation Class tương ứng có thể có nhiều cái, đóng gói bằng List
    private final List<S> providers = new ArrayList<>();

    // ClassLoader
    private final ClassLoader classLoader;

    // Phương thức bộc lộ ra ngoài sử dụng, thông qua việc gọi phương thức này để bắt đầu quy trình nạp bản thực thi tự định nghĩa.
    public static <S> MyServiceLoader<S> load(Class<S> service) {
        return new MyServiceLoader<>(service);
    }

    // Private constructor
    private MyServiceLoader(Class<S> service) {
        this.service = service;
        this.classLoader = Thread.currentThread().getContextClassLoader();
        doLoad();
    }

    // Phương thức mấu chốt, logic nạp implementation class cụ thể
    private void doLoad() {
        try {
            // Đọc tất cả các file dưới package META-INF/services trong file jar, tên file này chính là tên interface, nội dung bên trong file là đường dẫn + tên lớp đầy đủ của implementation class cụ thể
            Enumeration<URL> urls = classLoader.getResources("META-INF/services/" + service.getName());
            // Duyệt từng file lấy được
            while (urls.hasMoreElements()) {
                // Lấy ra file hiện tại
                URL url = urls.nextElement();
                System.out.println("File = " + url.getPath());
                // Thiết lập kết nối
                URLConnection urlConnection = url.openConnection();
                urlConnection.setUseCaches(false);
                // Lấy stream đầu vào của file
                InputStream inputStream = urlConnection.getInputStream();
                // Lấy buffer từ stream đầu vào
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                // Lấy tên lớp đầy đủ của implementation class từ nội dung file
                String className = bufferedReader.readLine();

                while (className != null) {
                    // Nhận instance của implementation class thông qua Reflection
                    Class<?> clazz = Class.forName(className, false, classLoader);
                    // Nếu interface khai báo và implementation class cụ thể này thuộc cùng kiểu (có thể hiểu như tính đa hình của Java), thì khởi tạo instance
                    if (service.isAssignableFrom(clazz)) {
                        Constructor<? extends S> constructor = (Constructor<? extends S>) clazz.getConstructor();
                        S instance = constructor.newInstance();
                        // Thêm đối tượng instance vừa khởi tạo vào danh sách Provider
                        providers.add(instance);
                    }
                    // Tiếp tục đọc implementation class ở dòng tiếp theo, có thể có nhiều implementation class, chỉ cần xuống dòng là được.
                    className = bufferedReader.readLine();
                }
            }
        } catch (Exception e) {
            System.out.println("Ngoại lệ khi đọc file...");
        }
    }

    // Trả về danh sách implementation class cụ thể tương ứng với spi interface
    public List<S> getProviders() {
        return providers;
    }
}
```

Các thông tin quan trọng cơ bản đã được mô tả qua comment trong code:

Quy trình chính là:

1. Thông qua utility class URL tìm file tương ứng dưới thư mục `/META-INF/services` của file jar,
2. Đọc tên file này để tìm spi interface tương ứng,
3. Thông qua stream `InputStream` đọc ra tên lớp đầy đủ của implementation class cụ thể bên trong file,
4. Dựa trên tên lớp đầy đủ thu được, đánh giá trước xem có cùng kiểu với spi interface không, nếu đúng thì thông qua cơ chế Reflection khởi tạo đối tượng instance tương ứng,
5. Thêm đối tượng instance vừa khởi tạo vào danh sách `Providers`.

## Tóm tắt

Không khó để nhận ra bản thực thi cụ thể của cơ chế SPI cần định vị và tạo service provider một cách động. Provider trên classpath cần được khai báo trong file cấu hình `META-INF/services/`; provider trong named module thì khai báo qua `provides ... with ...` trong module descriptor.

Ngoài ra, cơ chế SPI được ứng dụng trong rất nhiều framework: Nguyên lý cơ bản của Spring framework cũng tương tự. Framework Dubbo cũng cung cấp cơ chế mở rộng SPI tương tự, chỉ là cách thực hiện cụ thể cơ chế SPI trong Dubbo và Spring framework hơi khác một chút so với thứ chúng ta học hôm nay, tuy nhiên nguyên lý tổng thể đều nhất quán, tin rằng mọi người thông qua việc học cơ chế SPI trong JDK có thể suy rộng ra các framework khác, làm sâu sắc thêm việc thấu hiểu các framework nâng cao.

Thông qua cơ chế SPI có thể nâng cao rất nhiều tính linh hoạt khi thiết kế interface, tuy nhiên cơ chế SPI cũng tồn tại một số nhược điểm, ví dụ:

1. `ServiceLoader` sẽ nạp lười (lazily load) các provider; nếu bên gọi muốn chọn bản thực thi mà phải duyệt toàn bộ provider thì vẫn có thể sinh ra chi phí bổ sung;
2. Một instance `ServiceLoader` đơn lẻ không đảm bảo an toàn đa luồng, nếu muốn dùng chung qua các Thread thì cần được bên gọi tiến hành đồng bộ. Các instance `ServiceLoader` khác nhau gọi `load` cùng lúc không vì vậy mà bắt buộc xảy ra xung đột concurrency.

<!-- @include: @article-footer.snippet.md -->
