---
title: CompletableFuture 详解
description: CompletableFuture异步编程详解：全面讲解CompletableFuture核心API、异步任务编排、thenCompose/thenCombine组合、allOf/anyOf聚合、线程池配置与最佳实践。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: CompletableFuture,异步编程,异步编排,Future,thenCompose,thenCombine,allOf,并行任务
---

Trong dự án thực tế, một API có thể cần đồng thời lấy nhiều loại dữ liệu khác nhau, sau đó mới tổng hợp trả về, kịch bản này tương đối phổ biến. Lấy một ví dụ: User gửi request lấy thông tin đơn hàng, có thể cần đồng thời lấy thông tin người dùng, chi tiết sản phẩm, thông tin vận chuyển, gợi ý sản phẩm...

Nếu thực thi nối tiếp (lần lượt thực thi từng nhiệm vụ theo thứ tự), tốc độ phản hồi của API sẽ rất chậm. Cân nhắc thấy các nhiệm vụ này phần lớn đều **không có mối quan hệ thứ tự trước sau**, có thể **thực thi song song**, ví dụ như khi gọi lấy chi tiết sản phẩm, có thể đồng thời gọi lấy thông tin vận chuyển. Thông qua phương thức thực thi song song nhiều nhiệm vụ, tốc độ phản hồi của API sẽ được tối ưu hóa rõ rệt.

![](https://oss.javaguide.cn/github/javaguide/high-performance/serial-to-parallel.png)

Đối với các nhiệm vụ tồn tại mối quan hệ thứ tự gọi trước sau, có thể tiến hành sắp xếp phối hợp nhiệm vụ.

![](https://oss.javaguide.cn/github/javaguide/high-performance/serial-to-parallel2.png)

1. Sau khi lấy được thông tin người dùng, mới có thể gọi API chi tiết sản phẩm và thông tin vận chuyển.
2. Sau khi lấy thành công chi tiết sản phẩm và thông tin vận chuyển, mới có thể gọi API gợi ý sản phẩm.

Các kịch bản có thể cần dùng đến sắp xếp phối hợp nhiệm vụ bất đồng bộ đa luồng (ở đây chỉ lấy ví dụ, dữ liệu không nhất thiết phải trả về 1 lần, có thể sẽ tách API):

1. Trang chủ: Ví dụ trang chủ của cộng đồng kỹ thuật có thể cần đồng thời lấy danh sách gợi ý bài viết, banner quảng cáo, bảng xếp hạng bài viết, chủ đề hot...
2. Trang chi tiết: Ví dụ trang chi tiết bài viết của cộng đồng kỹ thuật có thể cần đồng thời lấy thông tin tác giả, chi tiết bài viết, bình luận bài viết...
3. Module thống kê: Ví dụ module thống kê backend của cộng đồng kỹ thuật có thể cần đồng thời lấy tổng hợp số fan, tổng hợp dữ liệu bài viết (lượt đọc, lượt bình luận, lượt bookmark)...

Đối với chương trình Java, `CompletableFuture` mới được đưa vào từ Java 8 có thể giúp chúng ta làm công việc sắp xếp phối hợp nhiều nhiệm vụ, tính năng vô cùng mạnh mẽ.

Bài viết này là nhập môn đơn giản về `CompletableFuture`, đưa mọi người xem các API thường dùng của `CompletableFuture`.

## Giới thiệu Future

Interface `Future` là ứng dụng điển hình của tư tưởng bất đồng bộ (async), chủ yếu dùng trong một số kịch bản cần thực thi nhiệm vụ tốn thời gian, tránh cho chương trình đứng yên chờ đợi nhiệm vụ tốn thời gian thực thi xong, làm hiệu suất thực thi quá thấp. Cụ thể là: Khi chúng ta thực thi một nhiệm vụ tốn thời gian nào đó, có thể giao nhiệm vụ tốn thời gian này cho một luồng con đi thực thi bất đồng bộ, đồng thời chúng ta có thể làm việc khác, không cần ngây ngốc chờ nhiệm vụ tốn thời gian thực thi xong. Sau khi việc của chúng ta làm xong, chúng ta lại thông qua `Future` để lấy kết quả thực thi của nhiệm vụ tốn thời gian. Như vậy, hiệu suất thực thi của chương trình được nâng cao rõ rệt.

Đây thực ra chính là **Future pattern** kinh điển trong đa luồng, bạn có thể coi nó là một design pattern, tư tưởng cốt lõi là gọi bất đồng bộ, chủ yếu dùng trong lĩnh vực đa luồng, chứ không phải riêng ngôn ngữ Java.

Trong Java, `Future` là một interface generic, nằm trong gói `java.util.concurrent`. Nó có 5 phương thức trừu tượng kinh điển, chủ yếu bao gồm 4 loại chức năng dưới đây; Từ JDK 19 trở đi lại bổ sung thêm 3 phương thức truy vấn mặc định `resultNow()`, `exceptionNow()` và `state()`.

- Hủy nhiệm vụ;
- Phán đoán nhiệm vụ có bị hủy không;
- Phán đoán nhiệm vụ đã thực thi xong chưa;
- Lấy kết quả thực thi nhiệm vụ.

```java
// V đại diện cho kiểu giá trị trả về của nhiệm vụ do Future thực thi
public interface Future<V> {
    // Hủy thực thi nhiệm vụ
    // Hủy thành công trả về true, ngược lại trả về false
    boolean cancel(boolean mayInterruptIfRunning);
    // Phán đoán nhiệm vụ có bị hủy không
    boolean isCancelled();
    // Phán đoán nhiệm vụ đã thực thi xong chưa
    boolean isDone();
    // Lấy kết quả thực thi nhiệm vụ
    V get() throws InterruptedException, ExecutionException;
    // Trong thời gian chỉ định nếu không trả về kết quả tính toán sẽ ném ra ngoại lệ TimeoutException
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

Hiểu đơn giản chính là: Tôi có một nhiệm vụ, submit cho `Future` xử lý. Trong thời gian nhiệm vụ thực thi, bản thân tôi có thể đi làm bất kỳ việc gì mình muốn. Hơn nữa, trong thời gian này tôi còn có thể hủy nhiệm vụ cũng như lấy trạng thái thực thi của nhiệm vụ. Sau một khoảng thời gian, tôi có thể trực tiếp lấy ra kết quả thực thi nhiệm vụ từ `Future`.

## Giới thiệu CompletableFuture

`Future` trong quá trình sử dụng thực tế tồn tại một số hạn chế, ví dụ không hỗ trợ phối hợp kết hợp các nhiệm vụ bất đồng bộ, phương thức `get()` lấy kết quả tính toán là lời gọi bị chặn.

Java 8 mới đưa vào lớp `CompletableFuture` có thể giải quyết các khuyết điểm này của `Future`. `CompletableFuture` ngoài việc cung cấp các tính năng `Future` dễ dùng và mạnh mẽ hơn, còn cung cấp các năng lực như lập trình hàm (functional programming), phối hợp tổ hợp nhiệm vụ bất đồng bộ (có thể chuỗi nhiều nhiệm vụ bất đồng bộ lại với nhau, tạo thành một chuỗi lời gọi hoàn chỉnh).

Dưới đây chúng ta hãy xem đơn giản định nghĩa lớp `CompletableFuture`.

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
}
```

Có thể thấy, `CompletableFuture` đồng thời triển khai interface `Future` và `CompletionStage`.

![](https://oss.javaguide.cn/github/javaguide/java/concurrent/completablefuture-class-diagram.jpg)

`CompletableFuture` ngoài việc cung cấp các tính năng `Future` dễ dùng và mạnh mẽ hơn, còn cung cấp năng lực lập trình hàm.

![](https://oss.javaguide.cn/javaguide/image-20210902092441434.png)

Interface `Future` có 5 phương thức:

- `boolean cancel(boolean mayInterruptIfRunning)`: Thử hủy thực thi nhiệm vụ.
- `boolean isCancelled()`: Phán đoán nhiệm vụ có bị hủy không.
- `boolean isDone()`: Phán đoán nhiệm vụ đã thực thi xong chưa.
- `get()`: Chờ nhiệm vụ thực thi hoàn thành và lấy kết quả tính toán.
- `get(long timeout, TimeUnit unit)`: Có thêm một thời gian timeout.

Interface `CompletionStage` mô tả một giai đoạn của tính toán bất đồng bộ. Nhiều tính toán có thể chia thành nhiều giai đoạn hoặc bước, lúc này có thể thông qua nó để kết hợp tất cả các bước lại, hình thành quy trình tính toán bất đồng bộ.

Các phương thức trong interface `CompletionStage` tương đối nhiều, năng lực hàm của `CompletableFuture` chính là do interface này ban cho. Từ các tham số phương thức của interface này bạn có thể phát hiện nó sử dụng lượng lớn lập trình hàm được đưa vào từ Java 8.

![](https://oss.javaguide.cn/javaguide/image-20210902093026059.png)

Do phương thức rất nhiều, nên ở đây không thể giải thích từng cái một, dưới đây tôi sẽ giới thiệu cách sử dụng của hầu hết các phương thức thường gặp.

## Các thao tác thường gặp của CompletableFuture

### Tạo CompletableFuture

Các phương thức tạo đối tượng `CompletableFuture` thường gặp như sau:

1. Thông qua từ khóa new.
2. Dựa trên các static factory method đi kèm của `CompletableFuture`: `runAsync()`, `supplyAsync()`.

#### Từ khóa new

Thông qua từ khóa new tạo đối tượng `CompletableFuture`, cách sử dụng này có thể coi là dùng `CompletableFuture` như `Future`.

Tôi trong project mã nguồn mở [guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) của tôi chính là dùng cách này để tạo đối tượng `CompletableFuture`.

Dưới đây chúng ta hãy xem một case đơn giản.

Chúng ta tạo một `CompletableFuture` có kiểu giá trị kết quả là `RpcResponse<Object>`, bạn có thể coi `resultFuture` như vật chứa kết quả tính toán bất đồng bộ.

```java
CompletableFuture<RpcResponse<Object>> resultFuture = new CompletableFuture<>();
```

Giả sử ở một thời điểm nào đó trong tương lai, chúng ta nhận được kết quả cuối cùng. Lúc này, chúng ta có thể gọi phương thức `complete()` truyền kết quả vào cho nó, điều này thể hiện `resultFuture` đã được hoàn thành.

```java
// Phương thức complete() chỉ có thể gọi 1 lần, các lần gọi sau đó sẽ bị bỏ qua.
resultFuture.complete(rpcResponse);
```

Bạn có thể thông qua phương thức `isDone()` để kiểm tra xem đã hoàn thành chưa.

```java
public boolean isDone() {
    return result != null;
}
```

Lấy kết quả tính toán bất đồng bộ cũng rất đơn giản, trực tiếp gọi phương thức `get()` là được. Luồng gọi phương thức `get()` sẽ bị chặn cho đến khi `CompletableFuture` hoàn thành tính toán.

```java
rpcResponse = completableFuture.get();
```

Nếu bạn đã biết trước kết quả tính toán, có thể sử dụng static method `completedFuture()` để tạo `CompletableFuture`.

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("hello!");
assertEquals("hello!", future.get());
```

Phương thức `completedFuture()` bên dưới gọi phương thức new có tham số, chỉ là phương thức này không bộc lộ ra ngoài.

```java
public static <U> CompletableFuture<U> completedFuture(U value) {
    return new CompletableFuture<U>((value == null) ? NIL : value);
}
```

#### Static Factory Method

Hai phương thức này có thể giúp chúng ta đóng gói logic tính toán.

```java
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
// Sử dụng ThreadPool tùy chỉnh (Khuyến nghị)
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
static CompletableFuture<Void> runAsync(Runnable runnable);
// Sử dụng ThreadPool tùy chỉnh (Khuyến nghị)
static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);
```

Tham số phương thức `runAsync()` nhận vào là `Runnable`, đây là một functional interface, không cho phép giá trị trả về. Khi bạn cần thao tác bất đồng bộ mà không quan tâm kết quả trả về thì có thể dùng phương thức `runAsync()`.

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

Tham số phương thức `supplyAsync()` nhận vào là `Supplier<U>`, đây cũng là một functional interface, `U` là kiểu giá trị kết quả trả về.

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

Khi bạn cần thao tác bất đồng bộ và quan tâm kết quả trả về, có thể dùng phương thức `supplyAsync()`.

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> System.out.println("hello!"));
future.get();// Output "hello!"
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "hello!");
assertEquals("hello!", future2.get());
```

### Xử lý kết quả tính toán bất đồng bộ

Khi chúng ta lấy được kết quả tính toán bất đồng bộ, còn có thể tiến hành xử lý tiếp đối với nó, các phương thức khá thường dùng gồm những phương thức dưới đây:

- `thenApply()`
- `thenAccept()`
- `thenRun()`
- `whenComplete()`

Phương thức `thenApply()` nhận vào một thể hiện `Function`, dùng nó để xử lý kết quả.

```java
// Callback non-async: Có thể do luồng hoàn thành giai đoạn trước thực thi; nếu giai đoạn đã hoàn thành, cũng có thể do luồng gọi hiện tại thực thi
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}

// Sử dụng ThreadPool ForkJoinPool mặc định (Không khuyến nghị)
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(defaultExecutor(), fn);
}
// Sử dụng ThreadPool tùy chỉnh (Khuyến nghị)
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn, Executor executor) {
    return uniApplyStage(screenExecutor(executor), fn);
}
```

Ví dụ sử dụng phương thức `thenApply()` như sau:

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!");
assertEquals("hello!world!", future.get());
// Lần gọi này sẽ bị bỏ qua.
future.thenApply(s -> s + "nice!");
assertEquals("hello!world!", future.get());
```

Bạn còn có thể tiến hành **gọi dạng chuỗi (fluent/chain call)**:

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!").thenApply(s -> s + "nice!");
assertEquals("hello!world!nice!", future.get());
```

**Nếu bạn không cần lấy kết quả trả về từ hàm callback, có thể sử dụng `thenAccept()` hoặc `thenRun()`. Điểm khác biệt giữa hai phương thức này nằm ở chỗ `thenRun()` không thể truy cập kết quả tính toán bất đồng bộ.**

Tham số của phương thức `thenAccept()` là `Consumer<? super T>`.

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
    return uniAcceptStage(null, action);
}

public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
    return uniAcceptStage(defaultExecutor(), action);
}

public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,
                                               Executor executor) {
    return uniAcceptStage(screenExecutor(executor), action);
}
```

Đúng như tên gọi, `Consumer` thuộc về interface dạng tiêu thụ, nó có thể nhận vào 1 đối tượng input rồi tiến hành "tiêu thụ".

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

Tham số của phương thức `thenRun()` là `Runnable`.

```java
public CompletableFuture<Void> thenRun(Runnable action) {
    return uniRunStage(null, action);
}

public CompletableFuture<Void> thenRunAsync(Runnable action) {
    return uniRunStage(defaultExecutor(), action);
}

public CompletableFuture<Void> thenRunAsync(Runnable action,
                                            Executor executor) {
    return uniRunStage(screenExecutor(executor), action);
}
```

Ví dụ sử dụng `thenAccept()` và `thenRun()` như sau:

```java
CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!").thenApply(s -> s + "nice!").thenAccept(System.out::println);//hello!world!nice!

CompletableFuture.completedFuture("hello!")
        .thenApply(s -> s + "world!").thenApply(s -> s + "nice!").thenRun(() -> System.out.println("hello!"));//hello!
```

Tham số của phương thức `whenComplete()` là `BiConsumer<? super T, ? super Throwable>`.

```java
public CompletableFuture<T> whenComplete(
    BiConsumer<? super T, ? super Throwable> action) {
    return uniWhenCompleteStage(null, action);
}


public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action) {
    return uniWhenCompleteStage(defaultExecutor(), action);
}
// Sử dụng ThreadPool tùy chỉnh (Khuyến nghị)
public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action, Executor executor) {
    return uniWhenCompleteStage(screenExecutor(executor), action);
}
```

Tương đối với `Consumer`, `BiConsumer` có thể nhận 2 đối tượng input rồi tiến hành "tiêu thụ".

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);

    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```

Ví dụ sử dụng `whenComplete()` như sau:

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello!")
        .whenComplete((res, ex) -> {
            // res đại diện kết quả trả về
            // ex có kiểu Throwable, đại diện ngoại lệ ném ra
            System.out.println(res);
            // Ở đây không ném ngoại lệ nên là null
            assertNull(ex);
        });
assertEquals("hello!", future.get());
```

### Xử lý ngoại lệ

Bạn có thể thông qua phương thức `handle()` để xử lý trường hợp có thể ném ra ngoại lệ trong quá trình thực thi nhiệm vụ.

```java
public <U> CompletableFuture<U> handle(
    BiFunction<? super T, Throwable, ? extends U> fn) {
    return uniHandleStage(null, fn);
}

public <U> CompletableFuture<U> handleAsync(
    BiFunction<? super T, Throwable, ? extends U> fn) {
    return uniHandleStage(defaultExecutor(), fn);
}

public <U> CompletableFuture<U> handleAsync(
    BiFunction<? super T, Throwable, ? extends U> fn, Executor executor) {
    return uniHandleStage(screenExecutor(executor), fn);
}
```

Mã nguồn ví dụ như sau:

```java
CompletableFuture<String> future
        = CompletableFuture.supplyAsync(() -> {
    if (true) {
        throw new RuntimeException("Computation error!");
    }
    return "hello!";
}).handle((res, ex) -> {
    // res đại diện kết quả trả về
    // ex có kiểu Throwable, đại diện ngoại lệ ném ra
    return res != null ? res : "world!";
});
assertEquals("world!", future.get());
```

Bạn còn có thể thông qua phương thức `exceptionally()` để xử lý trường hợp ngoại lệ.

```java
CompletableFuture<String> future
        = CompletableFuture.supplyAsync(() -> {
    if (true) {
        throw new RuntimeException("Computation error!");
    }
    return "hello!";
}).exceptionally(ex -> {
    System.out.println(ex.toString());// CompletionException
    return "world!";
});
assertEquals("world!", future.get());
```

Nếu bạn muốn kết quả của `CompletableFuture` chính là ngoại lệ, có thể sử dụng phương thức `completeExceptionally()` để gán giá trị cho nó.

```java
CompletableFuture<String> completableFuture = new CompletableFuture<>();
// ...
completableFuture.completeExceptionally(
  new RuntimeException("Calculation failed!"));
// ...
completableFuture.get(); // ExecutionException
```

### Tổ hợp CompletableFuture

Bạn có thể sử dụng `thenCompose()` để liên kết hai đối tượng `CompletableFuture` theo thứ tự, triển khai chuỗi nhiệm vụ bất đồng bộ. Tác dụng của nó là lấy kết quả trả về của nhiệm vụ trước làm tham số đầu vào cho nhiệm vụ tiếp theo, từ đó hình thành mối quan hệ phụ thuộc.

```java
public <U> CompletableFuture<U> thenCompose(
    Function<? super T, ? extends CompletionStage<U>> fn) {
    return uniComposeStage(null, fn);
}

public <U> CompletableFuture<U> thenComposeAsync(
    Function<? super T, ? extends CompletionStage<U>> fn) {
    return uniComposeStage(defaultExecutor(), fn);
}

public <U> CompletableFuture<U> thenComposeAsync(
    Function<? super T, ? extends CompletionStage<U>> fn,
    Executor executor) {
    return uniComposeStage(screenExecutor(executor), fn);
}
```

Ví dụ sử dụng phương thức `thenCompose()` như sau:

```java
CompletableFuture<String> future
        = CompletableFuture.supplyAsync(() -> "hello!")
        .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "world!"));
assertEquals("hello!world!", future.get());
```

Trong phát triển thực tế, phương thức này vẫn rất hữu ích. Ví dụ, task1 và task2 đều thực thi bất đồng bộ, nhưng task1 bắt buộc phải thực thi xong mới có thể bắt đầu thực thi task2 (task2 phụ thuộc vào kết quả thực thi của task1).

Tương tự với phương thức `thenCompose()` còn có phương thức `thenCombine()`, nó cũng có thể tổ hợp hai đối tượng `CompletableFuture`.

```java
CompletableFuture<String> completableFuture
        = CompletableFuture.supplyAsync(() -> "hello!")
        .thenCombine(CompletableFuture.supplyAsync(
                () -> "world!"), (s1, s2) -> s1 + s2)
        .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "nice!"));
assertEquals("hello!world!nice!", completableFuture.get());
```

**Vậy `thenCompose()` và `thenCombine()` khác nhau thế nào?**

- `thenCompose()` có thể liên kết hai đối tượng `CompletableFuture`, và lấy kết quả trả về của nhiệm vụ trước làm tham số của nhiệm vụ tiếp theo, giữa chúng tồn tại thứ tự trước sau.
- `thenCombine()` sẽ gộp kết quả của hai giai đoạn sau khi cả hai đều hoàn thành bình thường. Hai giai đoạn này có thể độc lập với nhau, nhưng việc có thực thi song song hay không phụ thuộc vào cách tạo chúng và executor được dùng, bản thân `thenCombine()` không chịu trách nhiệm khởi động nhiệm vụ.

Ngoài `thenCompose()` và `thenCombine()`, còn có một số phương thức tổ hợp `CompletableFuture` khác dùng để triển khai các hiệu quả khác nhau, thỏa mãn các nhu cầu nghiệp vụ khác nhau.

Ví dụ, trong kịch bản task1 và task2 đều hoàn thành bình thường, có thể sử dụng `acceptEither()`, để task3 nhận kết quả của một trong hai nhiệm vụ hoàn thành trước. Cần lưu ý, nó không thể coi là selector "kết quả thành công đầu tiên" đáng tin cậy: Chỉ cần một trong các giai đoạn hoàn thành bất thường, kết quả của giai đoạn trả về phải tuân theo quy tắc ngoại lệ của `CompletionStage` đối với tổ hợp either.

```java
public CompletableFuture<Void> acceptEither(
    CompletionStage<? extends T> other, Consumer<? super T> action) {
    return orAcceptStage(null, other, action);
}

public CompletableFuture<Void> acceptEitherAsync(
    CompletionStage<? extends T> other, Consumer<? super T> action) {
    return orAcceptStage(asyncPool, other, action);
}
```

Lấy đơn giản một ví dụ:

```java
CompletableFuture<String> task = CompletableFuture.supplyAsync(() -> {
    System.out.println("Nhiệm vụ 1 bắt đầu thực thi, thời gian hiện tại：" + System.currentTimeMillis());
    try {
        Thread.sleep(500);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Nhiệm vụ 1 thực thi hoàn tất, thời gian hiện tại：" + System.currentTimeMillis());
    return "task1";
});

CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Nhiệm vụ 2 bắt đầu thực thi, thời gian hiện tại：" + System.currentTimeMillis());
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Nhiệm vụ 2 thực thi hoàn tất, thời gian hiện tại：" + System.currentTimeMillis());
    return "task2";
});

task.acceptEitherAsync(task2, (res) -> {
    System.out.println("Nhiệm vụ 3 bắt đầu thực thi, thời gian hiện tại：" + System.currentTimeMillis());
    System.out.println("Kết quả nhiệm vụ trước là：" + res);
});

// Thêm một chút thời gian trì hoãn, đảm bảo nhiệm vụ bất đồng bộ có đủ thời gian hoàn thành
try {
    Thread.sleep(2000);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

Output:

```plain
Nhiệm vụ 1 bắt đầu thực thi, thời gian hiện tại：1695088058520
Nhiệm vụ 2 bắt đầu thực thi, thời gian hiện tại：1695088058521
Nhiệm vụ 1 thực thi hoàn tất, thời gian hiện tại：1695088059023
Nhiệm vụ 3 bắt đầu thực thi, thời gian hiện tại：1695088059023
Kết quả nhiệm vụ trước là：task1
Nhiệm vụ 2 thực thi hoàn tất, thời gian hiện tại：1695088059523
```

Khi cả hai giai đoạn đều hoàn thành bình thường, `acceptEitherAsync()` sẽ dùng kết quả của một trong hai giai đoạn đã hoàn thành để thực thi bất đồng bộ nhiệm vụ 3, thường là kết quả của nhiệm vụ hoàn thành trước. Tuy nhiên, nếu một trong các giai đoạn hoàn thành bất thường, mà giai đoạn kia chưa hoàn thành hoặc hoàn thành bình thường, quy chuẩn không đảm bảo giai đoạn trả về cuối cùng nhất định hoàn thành bình thường hay bất thường. Do đó, không thể dựa vào phương thức này để bỏ qua ngoại lệ xảy ra trước và tiếp tục chờ đợi một kết quả thành công khác.

### Chờ nhiều CompletableFuture hoàn thành

Bạn có thể thông qua static method `allOf()` của `CompletableFuture` để chờ nhiều `CompletableFuture` hoàn thành toàn bộ. `allOf()` chỉ kết hợp trạng thái hoàn thành của các giai đoạn đã có, không chịu trách nhiệm khởi động các nhiệm vụ này; nhiệm vụ có song song hay không phụ thuộc vào cách tạo chúng và executor.

Trong dự án thực tế, chúng ta thường xuyên cần chạy song song nhiều nhiệm vụ không liên quan đến nhau, giữa các nhiệm vụ này không có mối quan hệ phụ thuộc, có thể chạy độc lập với nhau.

Ví dụ chúng ta cần đọc xử lý 6 file, 6 nhiệm vụ này đều là nhiệm vụ không có phụ thuộc thứ tự thực thi, nhưng chúng ta cần tổng hợp kết quả xử lý của các file này lại khi trả về cho user. Trường hợp như thế này chúng ta có thể sử dụng việc chạy song song nhiều `CompletableFuture` để xử lý.

Code ví dụ như sau:

```java
CompletableFuture<Void> task1 =
  CompletableFuture.supplyAsync(()->{
    // Thao tác nghiệp vụ tự định nghĩa
  });
......
CompletableFuture<Void> task6 =
  CompletableFuture.supplyAsync(()->{
    // Thao tác nghiệp vụ tự định nghĩa
  });
......
 CompletableFuture<Void> headerFuture=CompletableFuture.allOf(task1,.....,task6);

  try {
    headerFuture.join();
  } catch (Exception ex) {
    ......
  }
System.out.println("all done. ");
```

Thường được mang ra so sánh với phương thức `allOf()` là phương thức `anyOf()`.

**Phương thức `allOf()` sẽ chờ cho đến khi tất cả các `CompletableFuture` đều chạy hoàn thành rồi mới trả về**

```java
Random rand = new Random();
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        System.out.println("future1 done...");
    }
    return "abc";
});
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        System.out.println("future2 done...");
    }
    return "efg";
});
```

Gọi `join()` có thể làm cho chương trình chờ `future1` và `future2` đều chạy xong mới tiếp tục thực thi.

```java
CompletableFuture<Void> completableFuture = CompletableFuture.allOf(future1, future2);
completableFuture.join();
assertTrue(completableFuture.isDone());
System.out.println("all futures done...");
```

Output:

```plain
future1 done...
future2 done...
all futures done...
```

**Phương thức `anyOf()` sẽ không chờ tất cả các `CompletableFuture` đều chạy hoàn thành mới trả về, chỉ cần có một cái thực thi hoàn thành là được!**

```java
CompletableFuture<Object> f = CompletableFuture.anyOf(future1, future2);
System.out.println(f.get());
```

Kết quả output có thể là:

```plain
future2 done...
efg
```

Cũng có thể là:

```plain
future1 done...
abc
```

## Gợi ý sử dụng CompletableFuture

### Sử dụng ThreadPool tùy chỉnh

Trong các ví dụ code ở trên của chúng ta, vì để tiện lợi nên đều không chọn ThreadPool tùy chỉnh. Trong dự án thực tế, điều này là không nên.

Trong triển khai mặc định của `CompletableFuture`, các phương thức bất đồng bộ không truyền hiển thị `Executor` thường sử dụng `ForkJoinPool.commonPool()` dùng chung toàn cục; lớp con có thể override `defaultExecutor()` để thay đổi executor mặc định của các phương thức bất đồng bộ non-static. Điều này có nghĩa là ứng dụng, nhiều thư viện hoặc framework nếu đều sử dụng triển khai mặc định, các nhiệm vụ bất đồng bộ liên quan thường sẽ dùng chung một ThreadPool.

Mặc dù `ForkJoinPool` hiệu suất rất cao, nhưng khi đồng thời submit lượng lớn nhiệm vụ, có thể dẫn đến tranh chấp tài nguyên và bỏ đói luồng (thread starvation), từ đó ảnh hưởng đến hiệu năng hệ thống.

Để tránh các vấn đề này, khuyến nghị cung cấp ThreadPool tùy chỉnh cho `CompletableFuture`, mang lại các ưu thế sau:

- **Tính cách ly**: Phân bổ ThreadPool độc lập cho các nhiệm vụ khác nhau, tránh tranh giành tài nguyên ThreadPool toàn cục.
- **Kiểm soát tài nguyên**: Điều chỉnh kích thước ThreadPool và loại hàng đợi dựa trên đặc tính nhiệm vụ, tối ưu hóa biểu hiện hiệu năng.
- **Xử lý ngoại lệ**: Thông qua `ThreadFactory` tùy chỉnh để xử lý tốt hơn các trường hợp ngoại lệ trong luồng.

```java
private ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 10,
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>());

CompletableFuture.runAsync(() -> {
     //...
}, executor);
```

### Cố gắng tránh sử dụng get()

Phương thức `get()` của `CompletableFuture` là bị chặn, cố gắng tránh sử dụng. Nếu bắt buộc phải sử dụng, cần thêm thời gian timeout, nếu không có thể dẫn đến luồng chính liên tục chờ đợi, không thể thực thi các nhiệm vụ khác.

```java
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Hello, world!";
    });

    // Lấy giá trị trả về của nhiệm vụ bất đồng bộ, đặt thời gian timeout là 5 giây
    try {
        String result = future.get(5, TimeUnit.SECONDS);
        System.out.println(result);
    } catch (InterruptedException | ExecutionException | TimeoutException e) {
        // Xử lý ngoại lệ
        e.printStackTrace();
    }
}
```

Đoạn code trên khi gọi `get()` đã ném ra ngoại lệ `TimeoutException`. Như vậy chúng ta có thể tiến hành các thao tác tương ứng trong phần xử lý ngoại lệ, ví dụ hủy nhiệm vụ, thử lại nhiệm vụ, ghi log...

### Xử lý ngoại lệ đúng cách

Khi sử dụng `CompletableFuture` nhất định phải dùng cách đúng đắn để xử lý ngoại lệ, tránh việc mất ngoại lệ hoặc xuất hiện vấn đề không thể kiểm soát.

Dưới đây là một số gợi ý:

- `whenComplete` sẽ thực thi callback khi giai đoạn hoàn thành bình thường hoặc bất thường, phù hợp để quan sát kết quả và ghi log ngoại lệ; nó mặc định giữ nguyên kết quả hoặc ngoại lệ của giai đoạn gốc, không dùng để chuyển ngoại lệ thành kết quả bình thường.
- `exceptionally` chỉ thực thi khi giai đoạn hoàn thành bất thường, và dùng giá trị trả về của callback để khôi phục về kết quả bình thường; nếu cần tiếp tục lan truyền ngoại lệ, có thể ném ra ngoại lệ một cách hiển thị trong callback.
- `handle` bất kể giai đoạn hoàn thành bình thường hay bất thường đều sẽ thực thi, và dựa trên kết quả cùng ngoại lệ để sinh ra một kết quả mới.
- `CompletableFuture.allOf` có thể chờ nhiều giai đoạn hoàn thành toàn bộ; chỉ cần một trong số các giai đoạn hoàn thành bất thường, `CompletableFuture` trả về cũng sẽ hoàn thành bất thường, nhưng vẫn cần kiểm tra riêng từng giai đoạn mới lấy được kết quả hoặc ngoại lệ của từng nhiệm vụ.
- ……

### Tổ hợp hợp lý nhiều nhiệm vụ bất đồng bộ

Sử dụng đúng đắn các phương thức `thenCompose()`, `thenCombine()`, `acceptEither()`, `allOf()`, `anyOf()`... để tổ hợp nhiều nhiệm vụ bất đồng bộ, nhằm thỏa mãn nhu cầu của nghiệp vụ thực tế, nâng cao hiệu suất thực thi của chương trình.

Trong sử dụng thực tế, chúng ta còn có thể tận dụng hoặc tham khảo các framework sắp xếp phối hợp nhiệm vụ bất đồng bộ có sẵn, ví dụ như [asyncTool](https://gitee.com/jd-platform-opensource/asyncTool) của JD.

![asyncTool README 文档](https://oss.javaguide.cn/github/javaguide/java/concurrent/asyncTool-readme.png)

## Lời bạt

Bài viết này chỉ giới thiệu đơn giản về khái niệm cốt lõi và một số API khá thường dùng của `CompletableFuture`. Nếu muốn học tập sâu hơn, còn có thể tìm thêm một số sách và blog để xem, ví dụ như vài bài viết dưới đây khá hay:

- [Nguyên lý và thực tiễn CompletableFuture - Bất đồng bộ hóa API phía Meituan Takeout Merchant - Đội ngũ kỹ thuật Meituan](https://tech.meituan.com/2022/05/12/principles-and-practices-of-completablefuture.html): Bài viết này giới thiệu chi tiết việc vận dụng `CompletableFuture` trong dự án thực tế. Tham khảo bài viết này, có thể tối ưu hóa đối với các kịch bản tương tự trong dự án, coi như một điểm sáng nhỏ. Phương thức tối ưu hiệu năng này tương đối đơn giản và hiệu quả cũng rất tốt!
- [Đọc mã nguồn RocketMQ, học ba thần khí lập trình Concurrency - Chia sẻ thực chiến Java của Dũng Ca](https://mp.weixin.qq.com/s/32Ak-WFLynQfpn0Cg0N-0A): Bài viết này giới thiệu việc ứng dụng `CompletableFuture` của RocketMQ. Cụ thể, bắt đầu từ RocketMQ 4.7, RocketMQ đã đưa vào `CompletableFuture` để triển khai xử lý message bất đồng bộ.

Ngoài ra, khuyến nghị các bạn G-Friend có thể xem framework concurrency [asyncTool](https://gitee.com/jd-platform-opensource/asyncTool) của JD, bên trong sử dụng lượng lớn `CompletableFuture`.

<!-- @include: @article-footer.snippet.md -->
