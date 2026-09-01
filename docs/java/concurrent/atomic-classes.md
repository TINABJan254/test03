---
title: Atomic 原子类总结
description: Java原子类详解：全面总结JUC包Atomic原子类体系、AtomicInteger/AtomicLong/AtomicReference等常用类、基于CAS的线程安全实现、使用场景与性能优势。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: Atomic原子类,AtomicInteger,AtomicLong,AtomicReference,CAS原子操作,JUC并发包,原子类使用
---

## Giới thiệu về các lớp nguyên tử Atomic

`Atomic` dịch sang tiếng Việt có nghĩa là "nguyên tử". Trong hóa học, nguyên tử là đơn vị nhỏ nhất cấu tạo nên chất, không thể chia nhỏ trong các phản ứng hóa học. Trong lập trình, `Atomic` chỉ một thao tác có tính nguyên tử, tức là thao tác đó không thể chia nhỏ, không thể bị gián đoạn. Ngay cả khi có nhiều luồng đồng thời thực thi, thao tác đó hoặc là thực thi hoàn thành toàn bộ, hoặc là không thực thi, không bị các luồng khác nhìn thấy trạng thái hoàn thành một phần.

Các lớp Atomic nói một cách đơn giản chính là các lớp có đặc tính thao tác nguyên tử.

Các lớp nguyên tử `Atomic` trong gói `java.util.concurrent.atomic` cung cấp một phương thức an toàn luồng để thao tác trên một biến đơn lẻ.

Các lớp `Atomic` phụ thuộc vào khóa lạc quan CAS (Compare-And-Swap) để đảm bảo tính nguyên tử cho các phương thức của nó, mà không cần sử dụng cơ chế khóa truyền thống (như khối `synchronized` hay `ReentrantLock`).

Trong bài viết này chúng ta chỉ giới thiệu khái niệm về các lớp Atomic nguyên tử, về nguyên lý triển khai cụ thể bạn có thể đọc bài viết tôi viết tại đây: [Giải thích chi tiết CAS](./cas.md).

![JUC原子类概览](https://oss.javaguide.cn/github/javaguide/java/JUC%E5%8E%9F%E5%AD%90%E7%B1%BB%E6%A6%82%E8%A7%88.png)

Dựa theo kiểu dữ liệu thao tác, có thể chia các lớp nguyên tử trong gói JUC thành 4 loại:

**1. Kiểu cơ bản**

Sử dụng cách thức nguyên tử để cập nhật kiểu dữ liệu cơ bản

- `AtomicInteger`: Lớp nguyên tử kiểu số nguyên
- `AtomicLong`: Lớp nguyên tử kiểu số nguyên dài
- `AtomicBoolean`: Lớp nguyên tử kiểu boolean

**2. Kiểu mảng**

Sử dụng cách thức nguyên tử để cập nhật một phần tử nào đó trong mảng

- `AtomicIntegerArray`: Lớp nguyên tử mảng số nguyên
- `AtomicLongArray`: Lớp nguyên tử mảng số nguyên dài
- `AtomicReferenceArray`: Lớp nguyên tử mảng kiểu tham chiếu

**3. Kiểu tham chiếu**

- `AtomicReference`: Lớp nguyên tử kiểu tham chiếu
- `AtomicMarkableReference`: Cập nhật nguyên tử kiểu tham chiếu có kèm cờ đánh dấu. Lớp này liên kết cờ boolean với tham chiếu, có thể phát hiện sự thay đổi giữa hai trạng thái do nghiệp vụ quy định, nhưng cờ 1 bit không thể ghi lại sự thay đổi số phiên bản nhiều lần tùy ý.
- `AtomicStampedReference`: Cập nhật nguyên tử kiểu tham chiếu có kèm số phiên bản. Lớp này liên kết giá trị số nguyên với tham chiếu, có thể dùng để giải quyết việc cập nhật nguyên tử dữ liệu và số phiên bản dữ liệu, có thể giải quyết vấn đề ABA xuất hiện khi dùng CAS thực hiện cập nhật nguyên tử.

So với nó, `AtomicStampedReference` sử dụng số phiên bản kiểu số nguyên, phù hợp hơn để phát hiện xem tham chiếu giữa hai lần đọc có trải qua nhiều lần thay đổi hay không.

**4. Kiểu sửa đổi thuộc tính của đối tượng**

- `AtomicIntegerFieldUpdater`: Bộ cập nhật field kiểu số nguyên nguyên tử
- `AtomicLongFieldUpdater`: Bộ cập nhật field kiểu số nguyên dài nguyên tử
- `AtomicReferenceFieldUpdater`: Bộ cập nhật field kiểu tham chiếu nguyên tử

## Các lớp nguyên tử kiểu cơ bản

Sử dụng cách thức nguyên tử để cập nhật kiểu dữ liệu cơ bản

- `AtomicInteger`: Lớp nguyên tử kiểu số nguyên
- `AtomicLong`: Lớp nguyên tử kiểu số nguyên dài
- `AtomicBoolean`: Lớp nguyên tử kiểu boolean

Các phương thức do ba lớp trên cung cấp hầu như giống nhau, nên ở đây chúng ta lấy `AtomicInteger` làm ví dụ để giới thiệu.

**Các phương thức thường dùng của lớp `AtomicInteger`**:

```java
public final int get() // Lấy giá trị hiện tại
public final int getAndSet(int newValue)// Lấy giá trị hiện tại, và đặt giá trị mới
public final int getAndIncrement()// Lấy giá trị hiện tại, và tự tăng
public final int getAndDecrement() // Lấy giá trị hiện tại, và tự giảm
public final int getAndAdd(int delta) // Lấy giá trị hiện tại, và cộng thêm giá trị kỳ vọng
boolean compareAndSet(int expect, int update) // Nếu giá trị nhập vào bằng giá trị kỳ vọng, sẽ đặt giá trị đó thành giá trị nhập vào (update) theo cách nguyên tử
public final void lazySet(int newValue)// Cuối cùng đặt thành newValue, lazySet cung cấp một ngữ nghĩa yếu hơn phương thức set, có thể dẫn đến các luồng khác trong khoảng thời gian ngắn sau đó vẫn đọc được giá trị cũ, nhưng có thể hiệu quả hơn.
```

**Ví dụ sử dụng lớp `AtomicInteger`**:

```java
// Khởi tạo đối tượng AtomicInteger, giá trị ban đầu là 0
AtomicInteger atomicInt = new AtomicInteger(0);

// Dùng phương thức getAndSet lấy giá trị hiện tại, và đặt giá trị mới là 3
int tempValue = atomicInt.getAndSet(3);
System.out.println("tempValue: " + tempValue + "; atomicInt: " + atomicInt);

// Dùng phương thức getAndIncrement lấy giá trị hiện tại, và tự tăng 1
tempValue = atomicInt.getAndIncrement();
System.out.println("tempValue: " + tempValue + "; atomicInt: " + atomicInt);

// Dùng phương thức getAndAdd lấy giá trị hiện tại, và tăng thêm giá trị chỉ định 5
tempValue = atomicInt.getAndAdd(5);
System.out.println("tempValue: " + tempValue + "; atomicInt: " + atomicInt);

// Dùng phương thức compareAndSet để cập nhật điều kiện nguyên tử, giá trị kỳ vọng là 9, giá trị cập nhật là 10
boolean updateSuccess = atomicInt.compareAndSet(9, 10);
System.out.println("Update Success: " + updateSuccess + "; atomicInt: " + atomicInt);

// Lấy giá trị hiện tại
int currentValue = atomicInt.get();
System.out.println("Current value: " + currentValue);

// Dùng phương thức lazySet đặt giá trị mới là 15
atomicInt.lazySet(15);
System.out.println("After lazySet, atomicInt: " + atomicInt);
```

Output:

```java
tempValue: 0; atomicInt: 3
tempValue: 3; atomicInt: 4
tempValue: 4; atomicInt: 9
Update Success: true; atomicInt: 10
Current value: 10
After lazySet, atomicInt: 15
```

## Các lớp nguyên tử kiểu mảng

Sử dụng cách thức nguyên tử để cập nhật một phần tử nào đó trong mảng

- `AtomicIntegerArray`: Lớp nguyên tử mảng số nguyên
- `AtomicLongArray`: Lớp nguyên tử mảng số nguyên dài
- `AtomicReferenceArray`: Lớp nguyên tử mảng kiểu tham chiếu

Các phương thức do ba lớp trên cung cấp hầu như giống nhau, nên ở đây chúng ta lấy `AtomicIntegerArray` làm ví dụ để giới thiệu.

**Các phương thức thường dùng của lớp `AtomicIntegerArray`**:

```java
public final int get(int i) // Lấy giá trị phần tử tại vị trí index=i
public final int getAndSet(int i, int newValue)// Trả về giá trị hiện tại tại vị trí index=i, và đặt nó thành giá trị mới: newValue
public final int getAndIncrement(int i)// Lấy giá trị phần tử tại vị trí index=i, và cho phần tử tại vị trí đó tự tăng
public final int getAndDecrement(int i) // Lấy giá trị phần tử tại vị trí index=i, và cho phần tử tại vị trí đó tự giảm
public final int getAndAdd(int i, int delta) // Lấy giá trị phần tử tại vị trí index=i, và cộng thêm giá trị kỳ vọng
boolean compareAndSet(int i, int expect, int update) // Nếu giá trị nhập vào bằng giá trị kỳ vọng, sẽ đặt giá trị phần tử tại vị trí index=i thành giá trị nhập vào (update) theo cách nguyên tử
public final void lazySet(int i, int newValue)// Cuối cùng đặt phần tử tại vị trí index=i thành newValue, sau khi dùng lazySet thiết lập có thể dẫn đến các luồng khác trong khoảng thời gian ngắn sau đó vẫn đọc được giá trị cũ.
```

**Ví dụ sử dụng lớp `AtomicIntegerArray`**:

```java
int[] nums = {1, 2, 3, 4, 5, 6};
// Tạo AtomicIntegerArray
AtomicIntegerArray atomicArray = new AtomicIntegerArray(nums);

// In giá trị ban đầu trong AtomicIntegerArray
System.out.println("Initial values in AtomicIntegerArray:");
for (int j = 0; j < nums.length; j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}

// Dùng phương thức getAndSet đặt giá trị tại index 0 thành 2, và trả về giá trị cũ
int tempValue = atomicArray.getAndSet(0, 2);
System.out.println("\nAfter getAndSet(0, 2):");
System.out.println("Returned value: " + tempValue);
for (int j = 0; j < atomicArray.length(); j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}

// Dùng phương thức getAndIncrement cộng 1 vào giá trị tại index 0, và trả về giá trị cũ
tempValue = atomicArray.getAndIncrement(0);
System.out.println("\nAfter getAndIncrement(0):");
System.out.println("Returned value: " + tempValue);
for (int j = 0; j < atomicArray.length(); j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}

// Dùng phương thức getAndAdd tăng giá trị tại index 0 thêm 5, và trả về giá trị cũ
tempValue = atomicArray.getAndAdd(0, 5);
System.out.println("\nAfter getAndAdd(0, 5):");
System.out.println("Returned value: " + tempValue);
for (int j = 0; j < atomicArray.length(); j++) {
    System.out.print("Index " + j + ": " + atomicArray.get(j) + " ");
}
```

Output:

```plain
Initial values in AtomicIntegerArray:
Index 0: 1 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
After getAndSet(0, 2):
Returned value: 1
Index 0: 2 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
After getAndIncrement(0):
Returned value: 2
Index 0: 3 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
After getAndAdd(0, 5):
Returned value: 3
Index 0: 8 Index 1: 2 Index 2: 3 Index 3: 4 Index 4: 5 Index 5: 6
```

## Các lớp nguyên tử kiểu tham chiếu

Các lớp nguyên tử kiểu cơ bản chỉ có thể cập nhật một biến, nếu cần cập nhật nguyên tử nhiều biến, cần sử dụng Các lớp nguyên tử kiểu tham chiếu.

- `AtomicReference`: Lớp nguyên tử kiểu tham chiếu
- `AtomicStampedReference`: Cập nhật nguyên tử kiểu tham chiếu có kèm số phiên bản. Lớp này liên kết giá trị số nguyên với tham chiếu, có thể dùng để giải quyết việc cập nhật nguyên tử dữ liệu và số phiên bản dữ liệu, có thể giải quyết vấn đề ABA xuất hiện khi dùng CAS thực hiện cập nhật nguyên tử.
- `AtomicMarkableReference`: Cập nhật nguyên tử kiểu tham chiếu có kèm cờ đánh dấu. Lớp này liên kết cờ boolean với tham chiếu.

Các phương thức do ba lớp trên cung cấp hầu như giống nhau, nên ở đây chúng ta lấy `AtomicReference` làm ví dụ để giới thiệu.

**Ví dụ sử dụng lớp `AtomicReference`**:

```java
// Person 类
class Person {
    private String name;
    private int age;
    //省略getter/setter和toString
}


// Tạo đối tượng AtomicReference và đặt giá trị ban đầu
AtomicReference<Person> ar = new AtomicReference<>(new Person("SnailClimb", 22));

// In giá trị ban đầu
System.out.println("Initial Person: " + ar.get().toString());

// Cập nhật giá trị
Person updatePerson = new Person("Daisy", 20);
ar.compareAndSet(ar.get(), updatePerson);

// In giá trị sau khi cập nhật
System.out.println("Updated Person: " + ar.get().toString());

// Thử cập nhật lại
Person anotherUpdatePerson = new Person("John", 30);
boolean isUpdated = ar.compareAndSet(updatePerson, anotherUpdatePerson);

// In việc cập nhật có thành công không và giá trị cuối cùng
System.out.println("Second Update Success: " + isUpdated);
System.out.println("Final Person: " + ar.get().toString());
```

Output:

```plain
Initial Person: Person{name='SnailClimb', age=22}
Updated Person: Person{name='Daisy', age=20}
Second Update Success: true
Final Person: Person{name='John', age=30}
```

**Ví dụ sử dụng lớp `AtomicStampedReference`**:

```java
// Tạo đối tượng AtomicStampedReference, giá trị ban đầu là "SnailClimb", số phiên bản ban đầu là 1
AtomicStampedReference<String> asr = new AtomicStampedReference<>("SnailClimb", 1);

// In giá trị ban đầu và số phiên bản
int[] initialStamp = new int[1];
String initialRef = asr.get(initialStamp);
System.out.println("Initial Reference: " + initialRef + ", Initial Stamp: " + initialStamp[0]);

// Cập nhật giá trị và số phiên bản
int oldStamp = initialStamp[0];
String oldRef = initialRef;
String newRef = "Daisy";
int newStamp = oldStamp + 1;

boolean isUpdated = asr.compareAndSet(oldRef, newRef, oldStamp, newStamp);
System.out.println("Update Success: " + isUpdated);

// In giá trị và số phiên bản sau cập nhật
int[] updatedStamp = new int[1];
String updatedRef = asr.get(updatedStamp);
System.out.println("Updated Reference: " + updatedRef + ", Updated Stamp: " + updatedStamp[0]);

// Thử cập nhật bằng số phiên bản sai
boolean isUpdatedWithWrongStamp = asr.compareAndSet(newRef, "John", oldStamp, newStamp + 1);
System.out.println("Update with Wrong Stamp Success: " + isUpdatedWithWrongStamp);

// In giá trị và số phiên bản cuối cùng
int[] finalStamp = new int[1];
String finalRef = asr.get(finalStamp);
System.out.println("Final Reference: " + finalRef + ", Final Stamp: " + finalStamp[0]);
```

Output:

```plain
Initial Reference: SnailClimb, Initial Stamp: 1
Update Success: true
Updated Reference: Daisy, Updated Stamp: 2
Update with Wrong Stamp Success: false
Final Reference: Daisy, Final Stamp: 2
```

**Ví dụ sử dụng lớp `AtomicMarkableReference`**:

```java
// Tạo đối tượng AtomicMarkableReference, giá trị ban đầu là "SnailClimb", cờ ban đầu là false
AtomicMarkableReference<String> amr = new AtomicMarkableReference<>("SnailClimb", false);

// In giá trị ban đầu và cờ
boolean[] initialMark = new boolean[1];
String initialRef = amr.get(initialMark);
System.out.println("Initial Reference: " + initialRef + ", Initial Mark: " + initialMark[0]);

// Cập nhật giá trị và cờ
String oldRef = initialRef;
String newRef = "Daisy";
boolean oldMark = initialMark[0];
boolean newMark = true;

boolean isUpdated = amr.compareAndSet(oldRef, newRef, oldMark, newMark);
System.out.println("Update Success: " + isUpdated);

// In giá trị và cờ sau khi cập nhật
boolean[] updatedMark = new boolean[1];
String updatedRef = amr.get(updatedMark);
System.out.println("Updated Reference: " + updatedRef + ", Updated Mark: " + updatedMark[0]);

// Thử cập nhật bằng cờ sai
boolean isUpdatedWithWrongMark = amr.compareAndSet(newRef, "John", oldMark, !newMark);
System.out.println("Update with Wrong Mark Success: " + isUpdatedWithWrongMark);

// In giá trị và cờ cuối cùng
boolean[] finalMark = new boolean[1];
String finalRef = amr.get(finalMark);
System.out.println("Final Reference: " + finalRef + ", Final Mark: " + finalMark[0]);
```

Output:

```plain
Initial Reference: SnailClimb, Initial Mark: false
Update Success: true
Updated Reference: Daisy, Updated Mark: true
Update with Wrong Mark Success: false
Final Reference: Daisy, Final Mark: true
```

## Các lớp nguyên tử kiểu sửa đổi thuộc tính của đối tượng

Nếu cần cập nhật nguyên tử một field nào đó trong một lớp nào đó, cần dùng đến Các lớp nguyên tử kiểu sửa đổi thuộc tính của đối tượng.

- `AtomicIntegerFieldUpdater`: Bộ cập nhật field kiểu số nguyên nguyên tử
- `AtomicLongFieldUpdater`: Bộ cập nhật field kiểu số nguyên dài nguyên tử
- `AtomicReferenceFieldUpdater`: Bộ cập nhật field kiểu tham chiếu nguyên tử

Muốn cập nhật thuộc tính của đối tượng một cách nguyên tử cần hai bước. Bước một, do các lớp nguyên tử kiểu sửa đổi thuộc tính đối tượng đều là lớp trừu tượng, nên mỗi lần sử dụng bắt buộc phải dùng phương thức static `newUpdater()` để tạo một bộ cập nhật, và cần thiết lập lớp và thuộc tính muốn cập nhật. Bước hai, field mục tiêu bắt buộc phải dùng `volatile` tu sửa, và khớp với kiểu của bộ cập nhật: lần lượt là `int`, `long` hoặc kiểu tham chiếu; đồng thời không thể là field `static` hoặc `final`.

Các phương thức do ba lớp trên cung cấp hầu như giống nhau, nên ở đây chúng ta lấy `AtomicIntegerFieldUpdater` làm ví dụ để giới thiệu.

**Ví dụ sử dụng lớp `AtomicIntegerFieldUpdater`**:

```java
// Person 类
class Person {
    private String name;
    // Để dùng AtomicIntegerFieldUpdater, field bắt buộc phải là volatile int
    volatile int age;
    //省略getter/setter和toString
}

// Tạo đối tượng AtomicIntegerFieldUpdater
AtomicIntegerFieldUpdater<Person> ageUpdater = AtomicIntegerFieldUpdater.newUpdater(Person.class, "age");

// Tạo đối tượng Person
Person person = new Person("SnailClimb", 22);

// In giá trị ban đầu
System.out.println("Initial Person: " + person);

// Cập nhật field age
ageUpdater.incrementAndGet(person); // Tự tăng
System.out.println("After Increment: " + person);

ageUpdater.addAndGet(person, 5); // Cộng 5
System.out.println("After Adding 5: " + person);

ageUpdater.compareAndSet(person, 28, 30); // Nếu giá trị hiện tại là 28, thì đặt thành 30
System.out.println("After Compare and Set (28 to 30): " + person);

// Thử dùng giá trị so sánh sai để cập nhật
boolean isUpdated = ageUpdater.compareAndSet(person, 28, 35); // Lần này sẽ thất bại
System.out.println("Compare and Set (28 to 35) Success: " + isUpdated);
System.out.println("Final Person: " + person);
```

Output:

```plain
Initial Person: Name: SnailClimb, Age: 22
After Increment: Name: SnailClimb, Age: 23
After Adding 5: Name: SnailClimb, Age: 28
After Compare and Set (28 to 30): Name: SnailClimb, Age: 30
Compare and Set (28 to 35) Success: false
Final Person: Name: SnailClimb, Age: 30
```

## Tham khảo

- 《Nghệ thuật lập trình Java Concurrency》

<!-- @include: @article-footer.snippet.md -->
