---
title: Java IO 模型详解
description: Java IO模型详解：深入剖析BIO阻塞IO、NIO非阻塞IO、AIO异步IO三种模型、多路复用机制、Reactor/Proactor模式、同步异步阻塞非阻塞概念辨析。
category: Java
tag:
  - Java IO
  - Java基础
head:
  - - meta
    - name: keywords
      content: Java IO模型,BIO,NIO,AIO,阻塞IO,非阻塞IO,多路复用,Reactor模式,Proactor模式
---

Mảng IO model quả thực rất khó hiểu, đòi hỏi khá nhiều kiến thức máy tính tầng dưới (low-level). Viết bài viết này mất khá nhiều thời gian, rất hy vọng có thể truyền tải những gì mình biết đến mọi người! Hy vọng các bạn thu hoạch được điều gì đó! Để viết bài này, mình còn phải giở lại cuốn "UNIX Network Programming", khó thật đấy!

_Năng lực cá nhân có hạn. Nếu bài viết có bất kỳ điểm nào cần bổ sung/hoàn thiện/sửa đổi, hoan nghênh bạn đóng góp ý kiến ở phần bình luận để cùng tiến bộ!_

## Lời nói đầu

I/O luôn là một điểm kiến thức khó hiểu đối với nhiều bạn, trong bài viết này mình sẽ giải thích I/O theo cách mình hiểu cho các bạn nghe, hy vọng có thể giúp ích cho bạn.

## I/O

### I/O là gì?

I/O (**I**nput/**O**utput) tức là **Đầu vào / Đầu ra**.

**Trước tiên chúng ta hãy giải thích I/O từ góc độ kiến trúc máy tính.**

Theo kiến trúc Von Neumann, kiến trúc máy tính chia thành 5 phần lớn: Bộ toán học và logic (ALU), Bộ điều khiển (CU), Bộ nhớ (Memory), Thiết bị đầu vào (Input), Thiết bị đầu ra (Output).

![Kiến trúc Von Neumann](https://oss.javaguide.cn/github/javaguide/java/io/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9pcy1jbG91ZC5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70.jpeg)

Thiết bị đầu vào (như bàn phím) và thiết bị đầu ra (như màn hình) đều thuộc về thiết bị ngoại vi. Card mạng, ổ cứng thì vừa có thể thuộc về thiết bị đầu vào, vừa thuộc về thiết bị đầu ra.

Thiết bị đầu vào nhập dữ liệu vào máy tính, thiết bị đầu ra nhận dữ liệu máy tính xuất ra.

**Từ góc nhìn kiến trúc máy tính, I/O mô tả quá trình giao tiếp giữa hệ thống máy tính và các thiết bị ngoại vi.**

**Chúng ta lại tiếp tục giải thích I/O từ góc độ ứng dụng.**

Theo kiến thức hệ điều hành đã học ở đại học: Để đảm bảo tính ổn định và an toàn của hệ điều hành, không gian địa chỉ của một tiến trình được chia thành **User space (Không gian người dùng)** và **Kernel space (Không gian nhân)**.

Các ứng dụng chúng ta chạy thường ngày đều chạy ở User space, chỉ có Kernel space mới có thể thực hiện các thao tác liên quan đến tài nguyên ở cấp độ system mode, ví dụ như quản lý file, giao tiếp tiến trình, quản lý bộ nhớ, v.v. Nói cách khác, chúng ta muốn thực hiện thao tác IO thì nhất định phải dựa vào năng lực của Kernel space.

Hơn nữa, chương trình ở User space không thể truy cập trực tiếp Kernel space.

Khi muốn thực hiện thao tác IO, do không có quyền thực hiện các thao tác này, chương trình chỉ có thể phát động system call (tương tác hệ thống) nhờ hệ điều hành hoàn thành giúp.

Do đó, nếu tiến trình người dùng muốn thực hiện thao tác IO, bắt buộc phải thông qua **system call** để truy cập gián tiếp vào Kernel space.

Trong quá trình phát triển hàng ngày, chúng ta tiếp xúc nhiều nhất với **Disk IO (đọc ghi file)** và **Network IO (yêu cầu và phản hồi mạng)**.

**Từ góc nhìn ứng dụng, ứng dụng của chúng ta phát động IO call (system call) tới kernel của hệ điều hành, kernel chịu trách nhiệm thực hiện thao tác IO cụ thể. Nói cách khác, ứng dụng của chúng ta thực chất chỉ phát động lời gọi thao tác IO, còn việc thực thi IO cụ thể là do kernel của hệ điều hành hoàn thành.**

Lấy thao tác đọc truyền thống làm ví dụ, ứng dụng sau khi phát động I/O call thường sẽ trải qua hai bước:

1. Kernel chờ thiết bị I/O chuẩn bị xong dữ liệu
2. Kernel sao chép dữ liệu từ Kernel space sang User space.

### Có những IO model phổ biến nào?

Trong hệ thống UNIX, IO model có tổng cộng 5 loại: **Synchronous Blocking I/O (I/O đồng bộ nghẽn)**, **Synchronous Non-blocking I/O (I/O đồng bộ không nghẽn)**, **I/O Multiplexing (Đa luồng I/O)**, **Signal-driven I/O (I/O điều khiển theo tín hiệu)** và **Asynchronous I/O (I/O bất đồng bộ)**.

Đây cũng là 5 loại IO model mà chúng ta thường hay nhắc đến.

## 3 loại IO model phổ biến trong Java

### BIO (Blocking I/O)

**BIO thuộc về mô hình Synchronous Blocking IO**.

Trong mô hình Synchronous Blocking IO, sau khi ứng dụng phát động lời gọi `read`, thread sẽ bị block (nghẽn) liên tục cho đến khi kernel sao chép dữ liệu sang User space.

![Nguồn hình: 《Giải thích sâu về Tomcat & Jetty》](https://oss.javaguide.cn/p3-juejin/6a9e704af49b4380bb686f0c96d33b81~tplv-k3u1fbpfcp-watermark.png)

Trong trường hợp số lượng kết nối client không cao, mô hình này không có vấn đề gì. Tuy nhiên, khi đối mặt với hàng trăm ngàn hoặc hàng triệu kết nối, mô hình BIO truyền thống hoàn toàn bất lực. Do đó, chúng ta cần một IO model xử lý hiệu quả hơn để ứng phó với lượng truy cập đồng thời (concurrency) cao hơn.

### NIO (New I/O)

NIO trong Java được giới thiệu từ Java 1.4, tương ứng với package `java.nio`, cung cấp các trừu tượng như `Channel`, `Selector`, `Buffer`. Chữ N trong NIO đại diện cho New, NIO đồng thời cung cấp cả channel blocking và non-blocking; trong đó, channel có thể lựa chọn (`SelectableChannel`) có thể cấu hình sang chế độ non-blocking. Nó hỗ trợ các phương thức thao tác I/O hướng buffer, dựa trên channel.

Trong Java NIO, phương thức lập trình mạng dựa trên non-blocking `SelectableChannel` và `Selector` tương ứng với **I/O Multiplexing model**. Tuy nhiên, không thể đánh đồng toàn bộ package `java.nio` với I/O Multiplexing, nó còn bao gồm các API như File I/O, Blocking Channel, v.v.

Đi theo luồng tư duy của mình xuống dưới xem sao, tin rằng bạn sẽ tìm thấy câu trả lời!

Trước tiên chúng ta hãy xem **Synchronous Non-blocking IO model**.

![Nguồn hình: 《Giải thích sâu về Tomcat & Jetty》](https://oss.javaguide.cn/p3-juejin/bb174e22dbe04bb79fe3fc126aed0c61~tplv-k3u1fbpfcp-watermark.png)

Trong mô hình Synchronous Non-blocking IO, ứng dụng sẽ liên tục phát động lời gọi `read`, trong khoảng thời gian chờ dữ liệu từ Kernel space sao chép sang User space, thread vẫn bị nghẽn, cho đến khi kernel sao chép xong dữ liệu sang User space.

So với mô hình Synchronous Blocking IO, mô hình Synchronous Non-blocking IO quả thực đã có cải tiến lớn. Thông qua thao tác polling (thăm dò), tránh được việc bị block liên tục.

> Synchronous Non-blocking IO phát động một lời gọi `read`, nếu dữ liệu chưa chuẩn bị xong, lúc này ứng dụng có thể không bị block chờ đợi mà chuyển sang làm một số tác vụ tính toán nhỏ, sau đó rất nhanh quay lại tiếp tục phát động lời gọi `read`, tức là polling. Quá trình polling này không phải phát động liên tục không ngừng mà có khoảng nghỉ, việc tận dụng khoảng nghỉ này chính là điểm giúp Synchronous Non-blocking IO hiệu quả hơn Synchronous Blocking IO.

Tuy nhiên, IO model này cũng tồn tại vấn đề: **Quá trình ứng dụng liên tục thực hiện system call I/O để polling xem dữ liệu đã sẵn sàng chưa là rất tiêu tốn tài nguyên CPU.**

Lúc này, **I/O Multiplexing model** xuất hiện.

![](https://oss.javaguide.cn/github/javaguide/java/io/88ff862764024c3b8567367df11df6ab~tplv-k3u1fbpfcp-watermark.png)

Trong mô hình I/O Multiplexing, thread trước tiên phát động lời gọi `select`, hỏi kernel xem dữ liệu đã sẵn sàng chưa, đợi sau khi kernel chuẩn bị dữ liệu xong, user thread mới phát động lời gọi `read`. Quá trình gọi `read` (dữ liệu từ Kernel space -> User space) vẫn bị nghẽn (blocking).

> Hiện tại các system call hỗ trợ IO Multiplexing gồm có `select`, `epoll`, v.v. System call `select` hiện được hỗ trợ trên hầu hết tất cả các hệ điều hành.
>
> - **Lời gọi `select`**: System call do kernel cung cấp, nó hỗ trợ truy vấn trạng thái sẵn sàng của nhiều system call cùng lúc. Hầu như mọi hệ điều hành đều hỗ trợ.
> - **Lời gọi `epoll`**: Linux 2.6 kernel, thuộc phiên bản tăng cường của lời gọi `select`, tối ưu hóa hiệu suất thực thi IO.

**Mô hình IO Multiplexing thông qua việc giảm các system call vô ích, đã giảm thiểu sự tiêu tốn tài nguyên CPU.**

NIO trong Java có một khái niệm rất quan trọng là **`Selector` (Bộ chọn)**, cũng có thể gọi là **Multiplexer (Bộ đa luồng)**. Thông qua nó, chỉ cần một thread là có thể quản lý nhiều kết nối client. Khi dữ liệu của client đến nơi, mới phục vụ cho client đó.

![Mối quan hệ giữa Buffer, Channel và Selector](https://oss.javaguide.cn/github/javaguide/java/nio/channel-buffer-selector.png)

### AIO (Asynchronous I/O)

AIO thường chỉ API Asynchronous Channel được NIO.2 giới thiệu trong Java 7. NIO.2 ngoài Asynchronous Channel ra, còn bao gồm File System API mới.

Các thao tác Asynchronous Channel sẽ trả về trực tiếp, phía gọi có thể lấy kết quả thông qua `Future`, hoặc truyền vào `CompletionHandler` để thực hiện callback sau khi thao tác hoàn thành.

![](https://oss.javaguide.cn/github/javaguide/java/io/3077e72a1af049559e81d18205b56fd7~tplv-k3u1fbpfcp-watermark.png)

Hiện tại ứng dụng của AIO vẫn chưa quá rộng rãi. Netty trước đây cũng từng thử dùng AIO, nhưng sau đó đã từ bỏ. Đó là vì sau khi Netty sử dụng AIO, hiệu năng trên hệ thống Linux không cải thiện được bao nhiêu.

Cuối cùng, dùng một hình ảnh để tóm tắt ngắn gọn về BIO, NIO, AIO trong Java.

![So sánh BIO, NIO và AIO](https://oss.javaguide.cn/github/javaguide/java/nio/bio-aio-nio.png)

## Tham khảo

- 《Giải thích sâu về Tomcat & Jetty》
- Làm thế nào để hoàn thành một lần IO: <https://llc687.top/126.html>
- Lập trình viên nên hiểu IO như thế này: [https://www.jianshu.com/p/fa7bdc4f3de7](https://www.jianshu.com/p/fa7bdc4f3de7)
- 10 phút đọc hiểu nguyên lý tầng dưới của Java NIO: <https://www.cnblogs.com/crazymakercircle/p/10225159.html>
- Hiểu biết về IO Model | Phần lý thuyết: <https://www.cnblogs.com/sheng-jie/p/how-much-you-know-about-io-models.html>
- 《UNIX Network Programming Tập 1: Socket Networking API》 Mục 6.2 IO Model

<!-- @include: @article-footer.snippet.md -->
