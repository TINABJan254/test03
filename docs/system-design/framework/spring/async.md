---
title: Async 注解原理分析
description: Spring @Async异步注解原理详解，涵盖异步任务配置、线程池设置、@EnableAsync机制及常见使用问题。
category: 框架
tag:
  - Spring
head:
  - - meta
    - name: keywords
      content: Spring异步,@Async,EnableAsync,线程池,TaskExecutor,异步任务,Spring注解,方法异步
---

Annotation `@Async` được cung cấp bởi Spring Framework, class hoặc method được đánh dấu bởi annotation này sẽ thực thi trong **thread bất đồng bộ (async thread)**. Điều này có nghĩa là khi method được gọi, bên gọi (caller) sẽ không chờ method đó thực thi hoàn thành, mà có thể tiếp tục thực thi đoạn code tiếp theo.

Việc sử dụng annotation `@Async` rất đơn giản, cần hai bước:

1. Thêm annotation `@EnableAsync` trên main class khởi động, bật async task.
2. Thêm annotation `@Async` trên method hoặc class cần thực thi bất đồng bộ.

```java
@SpringBootApplication
// Bật async task
@EnableAsync
public class YourApplication {

    public static void main(String[] args) {
        SpringApplication.run(YourApplication.class, args);
    }
}

// Service class bất đồng bộ
@Service
public class MyService {

    // Khuyên dùng ThreadPool tùy chỉnh, ở đây chỉ minh họa cách dùng cơ bản
    @Async
    public CompletableFuture<String> doSomethingAsync() {

        // Ở đây sẽ có một số thao tác tốn thời gian nghiệp vụ
        // ...
        // Sử dụng CompletableFuture có thể xử lý kết quả của async task tiện lợi hơn, tránh làm blocking thread chính
        return CompletableFuture.completedFuture("Async Task Completed");
    }

}
```

Tiếp theo, chúng ta cùng xem nguyên lý bên dưới của `@Async`.

## Phân tích nguyên lý @Async

`@Async` có thể thực thi task bất đồng bộ, về bản chất là sử dụng **Dynamic Proxy** để thực hiện. Thông qua post processor `BeanPostProcessor` trong Spring tạo dynamic proxy cho class sử dụng annotation `@Async`, sau đó cuộc gọi method có annotation `@Async` sẽ bị dynamic proxy chặn lại, trong interceptor đóng gói việc thực thi method thành async task và submit cho ThreadPool xử lý.

Tiếp theo, chúng ta cùng phân tích chi tiết.

### Bật Async

Trước khi sử dụng `@Async`, cần thêm `@EnableAsync` trên class khởi động để bật async, annotation `@EnableAsync` như sau:

```java
// Bỏ qua các annotation khác ...
@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync { /* ... */ }
```

Trên annotation `@EnableAsync` thông qua annotation `@Import` đưa vào `AsyncConfigurationSelector`, do đó Spring sẽ load class được đưa vào thông qua annotation `@Import`.

Class `AsyncConfigurationSelector` implement interface `ImportSelector`, do đó trong class này sẽ override method `selectImports()` để tùy chỉnh logic load Bean, như sau:

```java
public class AsyncConfigurationSelector extends AdviceModeImportSelector<EnableAsync> {
	@Override
	@Nullable
	public String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
	   // Advice weaving dựa trên Spring AOP proxy, cụ thể có thể dùng JDK Dynamic Proxy hoặc CGLIB
			case PROXY:
				return new String[] {ProxyAsyncConfiguration.class.getName()};
	   // Advice weaving dựa trên AspectJ
			case ASPECTJ:
				return new String[] {ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME};
			default:
				return null;
		}
	}
}
```

Trong method `selectImports()`, sẽ dựa theo loại advice khác nhau để chọn load các class khác nhau, trong đó `adviceMode` mặc định là `PROXY`.

Ở đây lấy advice dựa trên Spring AOP proxy làm ví dụ, lúc này sẽ load class `ProxyAsyncConfiguration`, như sau:

```java
@Configuration
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class ProxyAsyncConfiguration extends AbstractAsyncConfiguration {
	@Bean(name = TaskManagementConfigUtils.ASYNC_ANNOTATION_PROCESSOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public AsyncAnnotationBeanPostProcessor asyncAdvisor() {
		 // ...
  // Load post processor
		AsyncAnnotationBeanPostProcessor bpp = new AsyncAnnotationBeanPostProcessor();

  // ...
		return bpp;
	}
}
```

### Post Processor (Bộ xử lý sau)

Trong class `ProxyAsyncConfiguration`, sẽ thông qua `@Bean` load một post processor `AsyncAnnotationBeanPostProcessor`, post processor này là chìa khóa khiến cho annotation `@Async` có tác dụng.

Nếu trên một class hoặc method nào đó sử dụng annotation `@Async`, processor `AsyncAnnotationBeanPostProcessor` sẽ tạo một dynamic proxy cho class đó.

Method của class đó khi thực thi sẽ bị interceptor của object proxy chặn lại, trong đó method được đánh dấu bởi annotation `@Async` sẽ thực thi bất đồng bộ.

Code `AsyncAnnotationBeanPostProcessor` như sau:

```java
public class AsyncAnnotationBeanPostProcessor extends AbstractBeanFactoryAwareAdvisingPostProcessor {
	@Override
	public void setBeanFactory(BeanFactory beanFactory) {
		super.setBeanFactory(beanFactory);
  // Tạo AsyncAnnotationAdvisor, nó là một Advisor
  // Dùng để chặn các method có annotation @Async và thực thi bất đồng bộ các method này.
		AsyncAnnotationAdvisor advisor = new AsyncAnnotationAdvisor(this.executor, this.exceptionHandler);
  // Nếu thiết lập asyncAnnotationType tùy chỉnh, thì thiết lập nó vào advisor.
  // asyncAnnotationType dùng để chỉ định annotation async tùy chỉnh, ví dụ @MyAsync.
		if (this.asyncAnnotationType != null) {
			advisor.setAsyncAnnotationType(this.asyncAnnotationType);
		}
		advisor.setBeanFactory(beanFactory);
		this.advisor = advisor;
	}
}
```

Parent class của `AsyncAnnotationBeanPostProcessor` implement interface `BeanFactoryAware`, do đó trong class này override method `setBeanFactory()` làm điểm mở rộng (extension point), để load `AsyncAnnotationAdvisor`.

#### Tạo Advisor

`Advisor` là sự trừu tượng hóa của `Spring AOP` đối với `Advice` và `Pointcut`. `Advice` là logic thực thi của advice, `Pointcut` là pointcut thực thi của advice.

Trong post processor `AsyncAnnotationBeanPostProcessor` sẽ tạo `AsyncAnnotationAdvisor`, trong constructor của nó sẽ xây dựng `Advice` và `Pointcut` tương ứng, như sau:

```java
public class AsyncAnnotationAdvisor extends AbstractPointcutAdvisor implements BeanFactoryAware {

    private Advice advice; // Advice thực thi bất đồng bộ
    private Pointcut pointcut; // Pointcut khớp các method annotation @Async

    // Constructor
    public AsyncAnnotationAdvisor(/* Bỏ qua parameter */) {
        // 1. Tạo Advice, chịu trách nhiệm logic thực thi bất đồng bộ
        this.advice = buildAdvice(executor, exceptionHandler);
        // 2. Tạo Pointcut, lựa chọn method mục tiêu cần tăng cường
        this.pointcut = buildPointcut(asyncAnnotationTypes);
    }

    // Tạo Advice
    protected Advice buildAdvice(/* Bỏ qua parameter */) {
        // Tạo interceptor xử lý thực thi bất đồng bộ
        AnnotationAsyncExecutionInterceptor interceptor = new AnnotationAsyncExecutionInterceptor(null);
        // Cấu hình interceptor sử dụng executor và exception handler
        interceptor.configure(executor, exceptionHandler);
        return interceptor;
    }

    // Tạo Pointcut
    protected Pointcut buildPointcut(Set<Class<? extends Annotation>> asyncAnnotationTypes) {
        ComposablePointcut result = null;
        for (Class<? extends Annotation> asyncAnnotationType : asyncAnnotationTypes) {
            // 1. Pointcut cấp class: Nếu trên class có annotation thì khớp
            Pointcut cpc = new AnnotationMatchingPointcut(asyncAnnotationType, true);
            // 2. Pointcut cấp method: Nếu trên method có annotation thì khớp
            Pointcut mpc = new AnnotationMatchingPointcut(null, asyncAnnotationType, true);

            if (result == null) {
                result = new ComposablePointcut(cpc);
            } else {
                // Sử dụng union hợp nhất các pointcut trước đó
                result.union(cpc);
            }
            // Thêm pointcut cấp method vào composable pointcut
            result = result.union(mpc);
        }
        // Trả về composable pointcut, nếu không cung cấp loại annotation thì trả về Pointcut.TRUE
        return (result != null ? result : Pointcut.TRUE);
    }
}
```

Cốt lõi của `AsyncAnnotationAdvisor` nằm ở việc xây dựng `Advice` và `Pointcut`:

- Xây dựng `Advice`: Sẽ tạo interceptor `AnnotationAsyncExecutionInterceptor`, trong method `invoke()` của interceptor sẽ thực thi logic của advice.
- Xây dựng `Pointcut`: Cấu thành từ `ClassFilter` và `MethodMatcher`, dùng để khớp xem những method nào cần thực thi logic của advice (`Advice`).

#### Logic xử lý sau (Post Processing Logic)

Method `postProcessAfterInitialization()` được implement trong post processor `AsyncAnnotationBeanPostProcessor` nằm trong parent class của nó `AbstractAdvisingBeanPostProcessor`, sau khi `Bean` khởi tạo xong sẽ đi vào method `postProcessAfterInitialization()` để tiến hành xử lý sau.

Trong method xử lý sau, sẽ phán đoán `Bean` có thỏa mãn điều kiện của `Advisor` advice trong post processor hay không, nếu thỏa mãn thì tạo object proxy. Như sau:

```java
// AbstractAdvisingBeanPostProcessor
public Object postProcessAfterInitialization(Object bean, String beanName) {
	if (this.advisor == null || bean instanceof AopInfrastructureBean) {
		return bean;
	}
	if (bean instanceof Advised) {
		Advised advised = (Advised) bean;
		if (!advised.isFrozen() && isEligible(AopUtils.getTargetClass(bean))) {
			if (this.beforeExistingAdvisors) {
				advised.addAdvisor(0, this.advisor);
			}
			else {
				advised.addAdvisor(this.advisor);
			}
			return bean;
		}
	}
 // Phán đoán Bean được cấp có thỏa mãn điều kiện Advisor advice trong post processor hay không, nếu thỏa mãn thì tạo object proxy.
	if (isEligible(bean, beanName)) {
		ProxyFactory proxyFactory = prepareProxyFactory(bean, beanName);
		if (!proxyFactory.isProxyTargetClass()) {
			evaluateProxyInterfaces(bean.getClass(), proxyFactory);
		}
  // Thêm Advisor.
		proxyFactory.addAdvisor(this.advisor);
		customizeProxyFactory(proxyFactory);
  // Trả về object proxy.
		return proxyFactory.getProxy(getProxyClassLoader());
	}
	return bean;
}
```

### Chặn method của annotation @Async

Việc thực thi method mang annotation `@Async` sẽ bị chặn trong `AnnotationAsyncExecutionInterceptor`, trong method `invoke()` sẽ thực thi logic của interceptor. Lúc này sẽ đóng gói method được đánh dấu annotation `@Async` thành async task, giao cho executor thực thi.

Method `invoke()` được định nghĩa trong parent class `AsyncExecutionInterceptor` của `AnnotationAsyncExecutionInterceptor`, như sau:

```java
public class AsyncExecutionInterceptor extends AsyncExecutionAspectSupport implements MethodInterceptor, Ordered {
	@Override
	@Nullable
	public Object invoke(final MethodInvocation invocation) throws Throwable {
		Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);
		Method specificMethod = ClassUtils.getMostSpecificMethod(invocation.getMethod(), targetClass);
		final Method userDeclaredMethod = BridgeMethodResolver.findBridgedMethod(specificMethod);

  // 1. Xác định executor của async task
		AsyncTaskExecutor executor = determineAsyncExecutor(userDeclaredMethod);

  // 2. Đóng gói method sắp thực thi thành Callable async task
		Callable<Object> task = () -> {
			try {
    // 2.1. Thực thi method
				Object result = invocation.proceed();
    // 2.2. Nếu kiểu return value của method là Future, blocking chờ kết quả
				if (result instanceof Future) {
					return ((Future<?>) result).get();
				}
			}
			catch (ExecutionException ex) {
				handleError(ex.getCause(), userDeclaredMethod, invocation.getArguments());
			}
			catch (Throwable ex) {
				handleError(ex, userDeclaredMethod, invocation.getArguments());
			}
			return null;
		};
		// 3. Submit task
		return doSubmit(task, executor, invocation.getMethod().getReturnType());
	}
}
```

Trong method `invoke()`, chủ yếu có 3 bước:

1. Xác định executor thực thi async task.
2. Đóng gói method được đánh dấu annotation `@Async` thành `Callable` async task.
3. Submit task cho executor thực thi.

#### 1. Lấy Executor của Async Task

Trong method `determineAsyncExecutor()`, sẽ lấy executor của async task (tức là **ThreadPool** thực thi async task). Code như sau:

```java
// Xác định executor của async task
protected AsyncTaskExecutor determineAsyncExecutor(Method method) {
 // 1. Lấy từ cache trước.
	AsyncTaskExecutor executor = this.executors.get(method);
	if (executor == null) {
		Executor targetExecutor;
  // 2. Lấy qualifier của executor.
		String qualifier = getExecutorQualifier(method);
		if (StringUtils.hasLength(qualifier)) {
   // 3. Dựa theo qualifier lấy executor tương ứng.
			targetExecutor = findQualifiedExecutor(this.beanFactory, qualifier);
		}
		else {
   // 4. Nếu không có qualifier, thì sử dụng executor mặc định. Tức là ThreadPool mặc định do Spring cung cấp: SimpleAsyncTaskExecutor.
			targetExecutor = this.defaultExecutor.get();
		}
		if (targetExecutor == null) {
			return null;
		}
  // 5. Đóng gói executor thành adapter TaskExecutorAdapter.
  // TaskExecutorAdapter là một lớp trừu tượng do Spring làm cho JDK ThreadPool, vẫn kế thừa từ Executor của JDK ThreadPool. Ở đây không cần quan tâm quá nhiều, chỉ cần biết nó là ThreadPool là được.
		executor = (targetExecutor instanceof AsyncListenableTaskExecutor ?
				(AsyncListenableTaskExecutor) targetExecutor : new TaskExecutorAdapter(targetExecutor));
		this.executors.put(method, executor);
	}
	return executor;
}
```

Trong method `determineAsyncExecutor()` xác định executor (ThreadPool) của async task, chủ yếu thông qua giá trị `value` của annotation `@Async` để lấy qualifier của executor, dựa theo qualifier đi tìm kiếm executor tương ứng trong `BeanFactory` là được.

Nếu trong annotation `@Async` không chỉ định ThreadPool, thì sẽ thông qua `this.defaultExecutor.get()` để lấy ThreadPool mặc định, trong đó `defaultExecutor` được gán giá trị ở method dưới đây:

```java
// AsyncExecutionInterceptor
protected Executor getDefaultExecutor(@Nullable BeanFactory beanFactory) {
 // 1. Thử lấy ThreadPool từ beanFactory.
	Executor defaultExecutor = super.getDefaultExecutor(beanFactory);
 // 2. Nếu trong beanFactory không có, thì tạo ThreadPool SimpleAsyncTaskExecutor.
	return (defaultExecutor != null ? defaultExecutor : new SimpleAsyncTaskExecutor());
}
```

Trong đó `super.getDefaultExecutor()` sẽ thử lấy ThreadPool kiểu `Executor` trong `beanFactory`. Code như sau:

```java
protected Executor getDefaultExecutor(@Nullable BeanFactory beanFactory) {
	if (beanFactory != null) {
		try {
   // 1. Lấy ThreadPool kiểu TaskExecutor từ beanFactory.
			return beanFactory.getBean(TaskExecutor.class);
		}
		catch (NoUniqueBeanDefinitionException ex) {
			try {
				// 2. Nếu có nhiều cái, thì thử lấy ThreadPool Executor có name chỉ định từ beanFactory.
				return beanFactory.getBean(DEFAULT_TASK_EXECUTOR_BEAN_NAME, Executor.class);
			}
			catch (NoSuchBeanDefinitionException ex2) {
				if (logger.isInfoEnabled()) {
					// ...
				}
			}
		}
		catch (NoSuchBeanDefinitionException ex) {
			try {
    // 3. Nếu không có, thì thử lấy ThreadPool Executor có name chỉ định từ beanFactory.
				return beanFactory.getBean(DEFAULT_TASK_EXECUTOR_BEAN_NAME, Executor.class);
			}
			catch (NoSuchBeanDefinitionException ex2) {
				// ...
			}
		}
	}
	return null;
}
```

Trong `getDefaultExecutor()`, nếu lấy ThreadPool từ `beanFactory` thất bại, thì sẽ tạo ThreadPool `SimpleAsyncTaskExecutor`.

ThreadPool này mỗi lần thực thi async task đều tạo một thread mới để thực thi task, chứ không tái sử dụng thread, dẫn đến chi phí thực thi async task rất lớn. Một khi tại một thời điểm nào đó concurrency của method được đánh dấu annotation `@Async` tăng đột biến, ứng dụng sẽ tạo ra một lượng lớn thread, từ đó ảnh hưởng chất lượng dịch vụ thậm chí làm dịch vụ không thể sử dụng.

Cùng một thời điểm nếu submit 10000 task cho ThreadPool `SimpleAsyncTaskExecutor`, thì ThreadPool đó sẽ tạo 10000 thread, method `execute()` của nó như sau:

```java
// SimpleAsyncTaskExecutor: bên trong execute() sẽ gọi doExecute()
protected void doExecute(Runnable task) {
    // Tạo thread mới
    Thread thread = (this.threadFactory != null ? this.threadFactory.newThread(task) : createThread(task));
    thread.start();
}
```

**Đề xuất: Khi sử dụng `@Async` cần tự mình chỉ định ThreadPool, tránh rủi ro do ThreadPool mặc định của Spring mang lại.**

`value` trong annotation `@Async` chỉ định qualifier của ThreadPool, dựa theo qualifier có thể lấy **ThreadPool tùy chỉnh**. Code lấy qualifier như sau:

```java
// AnnotationAsyncExecutionInterceptor
protected String getExecutorQualifier(Method method) {
	// 1. Lấy annotation Async từ method.
	Async async = AnnotatedElementUtils.findMergedAnnotation(method, Async.class);
 // 2. Nếu trên method không tìm thấy annotation @Async, thì thử lấy annotation @Async từ class chứa method đó.
	if (async == null) {
		async = AnnotatedElementUtils.findMergedAnnotation(method.getDeclaringClass(), Async.class);
	}
 // 3. Nếu tìm thấy annotation @Async, thì lấy giá trị value của annotation và trả về, làm qualifier của ThreadPool.
 //    Nếu giá trị thuộc tính "value" là chuỗi rỗng, thì sử dụng ThreadPool mặc định.
 //    Nếu không tìm thấy annotation @Async, thì trả về null, cũng sử dụng ThreadPool mặc định.
	return (async != null ? async.value() : null);
}
```

#### 2. Đóng gói Method thành Async Task

Sau khi method `invoke()` lấy được executor, sẽ đóng gói method thành async task, code như sau:

```java
// Đóng gói method sắp thực thi thành Callable async task
Callable<Object> task = () -> {
    try {
        // 2.1. Thực thi method bị chặn (method proceed() là method cốt lõi trong AOP, dùng để thực thi method mục tiêu)
        Object result = invocation.proceed();

        // 2.2. Object proxy trả về cho bên gọi là Future bất đồng bộ thực tế, mà method mục tiêu chịu ràng buộc bởi chữ ký phương thức,
        //     sẽ trả về một Future tạm thời trước, do đó ở đây cần giải mã kết quả của Future tạm thời trong worker thread.
        if (result instanceof Future) {
            return ((Future<?>) result).get(); // Blocking chờ kết quả của Future
        }
    }
    catch (ExecutionException ex) {
        // 2.3. Xử lý exception ExecutionException. ExecutionException là exception do method Future.get() ném ra,
        handleError(ex.getCause(), userDeclaredMethod, invocation.getArguments()); // Xử lý exception nguyên bản
    }
    catch (Throwable ex) {
        // 2.4. Xử lý các loại exception khác. Đưa exception, method bị chặn và method parameter làm tham số gọi method handleError() để xử lý.
        handleError(ex, userDeclaredMethod, invocation.getArguments());
    }
    // 2.5. Nếu return value của method không phải kiểu Future, hoặc xảy ra exception, thì trả về null.
    return null;
};
```

So với `Runnable`, `Callable` có thể trả về kết quả, và ném ra exception.

Đóng gói việc thực thi `invocation.proceed()` (sự thực thi của method gốc) thành `Callable` async task. Ở đây chỉ khi `result` (return value của method) có kiểu là `Future` mới trả về, nếu là các kiểu khác thì trực tiếp trả về `null`.

Do đó method được đánh dấu annotation `@Async` nếu sử dụng return value ngoài kiểu `Future`, thì sẽ không thể lấy được kết quả thực thi của method.

#### 3. Submit Async Task

Sau khi đóng gói method sắp thực thi thành task Callable trong `AsyncExecutionInterceptor#invoke()`, sẽ giao task cho executor thực thi. Dưới đây là trích đoạn mã nguồn `doSubmit()` của Spring Framework 5.3.x, trong đó bao gồm API liên quan đến `ListenableFuture` sau này đã bị deprecated và loại bỏ:

```java
protected Object doSubmit(Callable<Object> task, AsyncTaskExecutor executor, Class<?> returnType) {
    // Dựa theo kiểu return value của method, chọn phương thức thực thi bất đồng bộ khác nhau và trả về kết quả.
    // 1. Nếu kiểu return value của method là CompletableFuture
    if (CompletableFuture.class.isAssignableFrom(returnType)) {
        // Sử dụng method CompletableFuture.supplyAsync() để thực thi bất đồng bộ task.
        return CompletableFuture.supplyAsync(() -> {
            try {
                return task.call();
            }
            catch (Throwable ex) {
                throw new CompletionException(ex); // Đóng gói exception thành CompletionException, để ném ra khi future.get()
            }
        }, executor);
    }
    // 2. Nếu kiểu return value của method là ListenableFuture
    else if (ListenableFuture.class.isAssignableFrom(returnType)) {
        // Ép kiểu AsyncTaskExecutor thành AsyncListenableTaskExecutor,
        // và gọi method submitListenable() để submit task.
        // AsyncListenableTaskExecutor là executor bất đồng bộ chuyên dụng của ListenableFuture,
        // nó có thể trả về một đối tượng ListenableFuture, cho phép thêm function callback để lắng nghe sự hoàn thành của task.
        return ((AsyncListenableTaskExecutor) executor).submitListenable(task);
    }
    // 3. Nếu kiểu return value của method là Future
    else if (Future.class.isAssignableFrom(returnType)) {
        // Trực tiếp gọi method submit() của AsyncTaskExecutor để submit task, và trả về một đối tượng Future.
        return executor.submit(task);
    }
    // 4. Nếu kiểu return value của method là void hoặc các kiểu khác
    else {
        // Trực tiếp gọi method submit() của AsyncTaskExecutor để submit task.
        // Vì kiểu return value của method là void, do đó không cần trả về bất kỳ kết quả nào, trực tiếp trả về null.
        executor.submit(task);
        return null;
    }
}
```

Trong method `doSubmit()`, sẽ dựa theo return value khác nhau của method được đánh dấu annotation `@Async`, để chọn các phương thức submit task khác nhau, cuối cùng task sẽ do executor (ThreadPool) thực thi.

### Tóm tắt

![Tóm tắt nguyên lý Async](./images/async/async.png)

Mấu chốt của việc hiểu nguyên lý `@Async` nằm ở việc hiểu annotation `@EnableAsync`, annotation đó đã bật tính năng async task.

Quy trình chính như hình trên, sẽ thông qua post processor để tạo object proxy, sau đó việc thực thi method `@Async` trong object proxy sẽ đi vào interceptor bên trong `Advice`, sau đó đóng gói method thành async task, và submit cho ThreadPool tiến hành xử lý.

## Đề xuất sử dụng @Async

### ThreadPool tùy chỉnh

Nếu không cấu hình tường minh ThreadPool, ở dưới tầng của `@Async` trước tiên sẽ thử lấy ThreadPool trong `BeanFactory`, nếu không lấy được, thì sẽ tạo một bản thực thi `SimpleAsyncTaskExecutor`. `SimpleAsyncTaskExecutor` về bản chất không được tính là một ThreadPool thực sự, vì đối với mỗi request nó đều sẽ khởi động một thread mới chứ không tái sử dụng thread hiện có, điều này mang lại một số vấn đề tiềm ẩn, ví dụ tiêu tốn tài nguyên quá lớn.

Về việc lấy ThreadPool cụ thể có thể tham khảo bài viết này: [浅析 Spring 中 Async 注解底层异步线程池原理｜得物技术](https://mp.weixin.qq.com/s/FySv5L0bCdrlb5MoSfQtAA).

Nhất định phải cấu hình tường minh một ThreadPool, khuyên dùng `ThreadPoolTaskExecutor`. Hơn nữa, còn có thể dựa vào tính chất và nhu cầu của task, chỉ định các ThreadPool khác nhau cho các method async khác nhau.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "executor1")
    public Executor executor1() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("AsyncExecutor1-");
        executor.initialize();
        return executor;
    }

    @Bean(name = "executor2")
    public Executor executor2() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(4);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("AsyncExecutor2-");
        executor.initialize();
        return executor;
    }
}
```

Chỉ định Bean name của ThreadPool trong annotation `@Async`:

```java
@Service
public class AsyncService {

    @Async("executor1")
    public void performTask1() {
        // Logic của task 1
        System.out.println("Executing Task1 with Executor1");
    }

    @Async("executor2")
    public void performTask2() {
        // Logic của task 2
        System.out.println("Executing Task2 with Executor2");
    }
}
```

### Tránh annotation @Async bị mất hiệu lực

Annotation `@Async` sẽ bị mất hiệu lực trong các kịch bản dưới đây, cần lưu ý:

**1. Gọi method async trong cùng một class**

Nếu bạn gọi một method có annotation `@Async` bên trong cùng một class, thì method đó sẽ không thực thi bất đồng bộ.

```java
@Service
public class MyService {

    public void myMethod() {
        // Gọi trực tiếp thông qua tham chiếu this, bỏ qua cơ chế proxy của Spring, thực thi bất đồng bộ bị mất hiệu lực
        asyncMethod();
    }

    @Async
    public void asyncMethod() {
        // Logic thực thi bất đồng bộ
    }
}
```

Đó là vì cơ chế async của Spring được thực hiện thông qua **Proxy**, mà cuộc gọi method nội bộ trong cùng một class sẽ bỏ qua cơ chế proxy của Spring, tức là bỏ qua object proxy, gọi trực tiếp thông qua tham chiếu this. Vì không đi qua proxy, tất cả các xử lý liên quan đến proxy (tức submit task cho ThreadPool thực thi bất đồng bộ) đều không xảy ra.

Để tránh vấn đề này, cách làm khuyên dùng hơn là chuyển method async sang một Spring Bean khác.

```java
@Service
public class AsyncService {
    @Async
    public void asyncMethod() {
        // Logic thực thi bất đồng bộ
    }
}

@Service
public class MyService {
    @Autowired
    private AsyncService asyncService;

    public void myMethod() {
        asyncService.asyncMethod();
    }
}
```

**2. Sử dụng từ khóa static để修饰 method async**

Nếu method có annotation `@Async` bị từ khóa `static`修饰, thì method đó sẽ không thực thi bất đồng bộ.

Đó là vì cơ chế async của Spring được thực hiện thông qua proxy, vì static method không thuộc về instance mà thuộc về class và không tham gia kế thừa, cơ chế proxy của Spring (bất kể dựa trên JDK hay CGLIB) đều không thể chặn static method để cung cấp tính năng tăng cường như thực thi bất đồng bộ.

Do giới hạn độ dài bài viết, ở đây không giới thiệu chi tiết thêm, những bạn chưa hiểu về cơ chế proxy có thể xem bài viết [Giải thích chi tiết Proxy Pattern trong Java](https://javaguide.cn/java/basis/proxy.html) do tôi viết.

Nếu bạn cần thực thi bất đồng bộ logic của một static method, có thể cân nhắc thiết kế một method wrapper non-static, method wrapper này sử dụng annotation `@Async`, và bên trong nó gọi static method

```java
@Service
public class AsyncService {

    @Async
    public void asyncWrapper() {
        // Gọi static method
        SClass.staticMethod();
    }
}

public class SClass {
    public static void staticMethod() {
        // Thực thi một số thao tác
    }
}
```

**3. Quên bật hỗ trợ async**

Spring Boot trong trường hợp mặc định không bật hỗ trợ async, hãy đảm bảo thêm annotation `@EnableAsync` trên main configuration class `Application` để bật tính năng async.

```java
@SpringBootApplication
@EnableAsync
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**4. Class chứa method annotation `@Async` bắt buộc phải là Spring Bean**

Method gắn annotation `@Async` phải nằm trong Bean do Spring quản lý, chỉ có như vậy, Spring mới có thể áp dụng proxy khi tạo Bean, proxy mới có thể chặn cuộc gọi method và thực hiện logic thực thi bất đồng bộ. Nếu method đó không nằm trong bean do Spring quản lý, Spring sẽ không thể tạo proxy cần thiết, annotation `@Async` sẽ không tạo ra bất kỳ hiệu quả nào.

### Kiểu return value

Khuyên nên định nghĩa kiểu return value của method gắn annotation `@Async` thành `void` và `Future`.

- Nếu không cần lấy kết quả trả về của method async, định nghĩa kiểu return value thành `void`.
- Nếu cần lấy kết quả trả về của method async, định nghĩa kiểu return value thành `Future` (thường dùng `CompletableFuture`). `ListenableFuture` thuộc về API Spring bản cũ, không nên tiếp tục sử dụng trong Spring 6.1 trở lên.

Nếu định nghĩa return value của method gắn annotation `@Async` thành các kiểu khác (như `Object`, `String`,...), thì không thể lấy được return value của method.

Thiết kế này phù hợp với nguyên tắc cơ bản của lập trình bất đồng bộ, tức bên gọi không nên lập tức chờ đợi một kết quả, mà nên có thể lấy được kết quả tại một thời điểm nào đó trong tương lai. Nếu kiểu trả về là `Future`, bên gọi có thể sử dụng đối tượng `Future` trả về này để truy vấn trạng thái task, hủy task, hoặc lấy kết quả khi task hoàn thành.

### Xử lý Exception trong Async Method

Exception ném ra trong async method sẽ không được catch trực tiếp bởi calling thread. Async method trả về `Future` hoặc `CompletableFuture` sẽ bộc lộ exception thông qua Future, có thể sử dụng `get()`, `join()` hoặc phương thức xử lý exception của `CompletableFuture` để xử lý; async method trả về `void` không thể truyền exception cho bên gọi, có thể cấu hình `AsyncUncaughtExceptionHandler` toàn cục.

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer{

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new CustomAsyncExceptionHandler();
    }

}

// Custom exception handler
class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(Throwable ex, Method method, Object... params) {
        // Ghi log hoặc logic xử lý khác
    }
}
```

### Chưa xét đến quản lý Transaction

Method gắn annotation `@Async` khi cần hỗ trợ transaction, bắt buộc phải sử dụng độc lập trên async method đó.

```java
@Service
public class AsyncTransactionalService {

    @Async
    // Propagation.REQUIRES_NEW thể hiện Spring khi thực thi async method sẽ mở một transaction mới không liên quan tới transaction hiện tại
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void asyncTransactionalMethod() {
        // Thao tác ở đây sẽ thực thi trong transaction mới
        // Thực thi một số thao tác cơ sở dữ liệu
    }
}
```

### Chưa chỉ định thứ tự thực thi Async Method

Method gắn annotation `@Async` thực thi là non-blocking, chúng có thể hoàn thành theo thứ tự bất kỳ. Nếu cần xử lý kết quả theo thứ tự cụ thể, bạn có thể thiết lập return value của method thành `Future` hoặc `CompletableFuture`, thông qua object return value để thực hiện việc một method sau khi method khác hoàn thành mới thực thi.

```java
@Async
public CompletableFuture<String> fetchDataAsync() {
    return CompletableFuture.completedFuture("Data");
}

@Async
public CompletableFuture<String> processDataAsync(String data) {
    // Bản thân method đã được @Async điều phối đến executor do Spring quản lý, không submit lại vào commonPool nữa.
    return CompletableFuture.completedFuture("Processed " + data);
}
```

Method `processDataAsync` thực thi sau `fetchDataAsync`:

```java
CompletableFuture<String> dataFuture = asyncService.fetchDataAsync();
dataFuture.thenCompose(data -> asyncService.processDataAsync(data))
          .thenAccept(result -> System.out.println(result));
```

##
