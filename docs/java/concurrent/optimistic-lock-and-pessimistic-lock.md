---
title: 乐观锁和悲观锁详解
description: 乐观锁与悲观锁深度对比：详解synchronized/ReentrantLock悲观锁实现、CAS/版本号乐观锁机制、适用场景分析、性能对比与选型建议。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: 乐观锁,悲观锁,synchronized,ReentrantLock,CAS,版本号机制,并发控制,锁优化
---

Nếu ví Khóa bi quan (Pessimistic Lock) và Khóa lạc quan (Optimistic Lock) với đời sống thực tế: Khóa bi quan giống như một người khá bi quan (hoặc có thể nói là luôn lo xa), luôn giả định tình huống xấu nhất để tránh xảy ra sự cố. Khóa lạc quan giống như một người khá lạc quan, luôn giả định tình huống tốt nhất, giải quyết nhanh chóng trước khi sự cố xảy ra.

## Khóa bi quan là gì?

Khóa bi quan luôn giả định trường hợp xấu nhất, cho rằng tài nguyên dùng chung mỗi lần được truy cập thì sẽ xảy ra vấn đề (như dữ liệu dùng chung bị sửa đổi), nên mỗi lần thực hiện thao tác lấy tài nguyên thì đều cài khóa, như vậy các luồng khác muốn lấy tài nguyên này sẽ bị chặn cho đến khi khóa được người sở hữu trước đó giải phóng. Nói cách khác, **tài nguyên dùng chung mỗi lần chỉ cho một luồng sử dụng, các luồng khác bị chặn, dùng xong mới chuyển giao tài nguyên cho luồng khác**.

Các khóa độc chiếm như `synchronized` và `ReentrantLock` trong Java chính là sự triển khai của tư tưởng khóa bi quan.

```java
public void performSynchronisedTask() {
    synchronized (this) {
        // Thao tác cần đồng bộ
    }
}

private Lock lock = new ReentrantLock();
lock.lock();
try {
   // Thao tác cần đồng bộ
} finally {
    lock.unlock();
}
```

Trong kịch bản concurrency cao, việc tranh chấp khóa dữ dội sẽ khiến các luồng bị chặn, số lượng lớn luồng bị chặn sẽ dẫn đến chuyển đổi ngữ cảnh của hệ thống, làm tăng chi phí hiệu năng của hệ thống. Hơn nữa, khóa bi quan còn có thể gặp phải vấn đề deadlock (khi thứ tự lấy khóa của các luồng không hợp lý), ảnh hưởng đến việc vận hành bình thường của mã nguồn.

## Khóa lạc quan là gì?

Khóa lạc quan luôn giả định trường hợp tốt nhất, cho rằng tài nguyên dùng chung mỗi lần truy cập sẽ không gặp vấn đề gì, luồng có thể liên tục thực thi mà không cần cài khóa cũng không cần chờ đợi, chỉ khi submit sửa đổi mới đi xác minh xem tài nguyên tương ứng (tức là dữ liệu) có bị luồng khác sửa đổi hay không (phương pháp cụ thể có thể dùng cơ chế số phiên bản hoặc thuật toán CAS).

Trong Java, các lớp biến nguyên tử dưới gói `java.util.concurrent.atomic` (như `AtomicInteger`, `LongAdder`) chính là cách triển khai khóa lạc quan thông qua **CAS**.
![JUC原子类概览](https://oss.javaguide.cn/github/javaguide/java/JUC%E5%8E%9F%E5%AD%90%E7%B1%BB%E6%A6%82%E8%A7%88-20230814005211968.png)

```java
// LongAdder trong kịch bản concurrency cao sẽ có hiệu năng tốt hơn AtomicInteger và AtomicLong
// Đánh đổi lại là tiêu tốn nhiều bộ nhớ hơn (đổi không gian lấy thời gian)
LongAdder sum = new LongAdder();
sum.increment();
```

Trong kịch bản concurrency cao, so với khóa bi quan, khóa lạc quan không bị chặn luồng do tranh chấp khóa, cũng không gặp vấn đề deadlock, về mặt hiệu năng thường vượt trội hơn. Tuy nhiên, nếu xung đột xảy ra thường xuyên (trường hợp tỷ lệ ghi rất nhiều), sẽ dẫn đến thất bại và thử lại liên tục, điều này cũng ảnh hưởng rất lớn đến hiệu năng, làm CPU tăng cao.

`LongAdder` sẽ phân tán việc cập nhật sang nhiều ô nhớ nội bộ (internal cell) khi tranh chấp dữ dội, từ đó giảm xác suất tất cả luồng tranh giành cùng một giá trị, nhưng việc cập nhật nội bộ vẫn có thể dùng CAS và thử lại, chứ không loại bỏ hoàn toàn tranh chấp. Ngoài ra, `sum()` trả về cũng không phải là snapshot đồng bộ nguyên tử với các cập nhật concurrency.

Về mặt lý thuyết:

- Khóa bi quan thường dùng nhiều cho trường hợp ghi nhiều (kịch bản nhiều thao tác ghi, tranh chấp dữ dội), như vậy có thể tránh thất bại và thử lại liên tục ảnh hưởng đến hiệu năng. Tuy nhiên, các phương án như `LongAdder` thông qua phân tán tranh chấp để giảm xác suất thử lại, khi chỉ cần các ngữ nghĩa đặc định như tích lũy thì cũng có thể cân nhắc, vẫn cần cân nhắc dựa trên kịch bản thực tế.
- Khóa lạc quan thường dùng nhiều cho trường hợp ghi ít (kịch bản đọc nhiều, tranh chấp ít), như vậy có thể tránh việc cài khóa liên tục ảnh hưởng đến hiệu năng. Tuy nhiên, đối tượng chủ yếu của khóa lạc quan là một biến dùng chung đơn lẻ (tham khảo các lớp biến nguyên tử dưới gói `java.util.concurrent.atomic`).

## Làm thế nào để triển khai khóa lạc quan?

Khóa lạc quan thường sử dụng cơ chế số phiên bản hoặc thuật toán CAS để triển khai, thuật toán CAS tương đối phổ biến hơn, cần đặc biệt lưu ý ở đây.

### Cơ chế số phiên bản

Thường là thêm một trường số phiên bản dữ liệu `version` vào bảng dữ liệu, thể hiện số lần dữ liệu đã bị sửa đổi. Khi dữ liệu bị sửa đổi, giá trị `version` sẽ cộng thêm 1. Khi luồng A muốn cập nhật giá trị dữ liệu, trong lúc đọc dữ liệu cũng sẽ đọc giá trị `version`, khi submit cập nhật, nếu giá trị `version` vừa đọc được bằng với giá trị `version` hiện tại trong cơ sở dữ liệu thì mới cập nhật, nếu không sẽ thử lại thao tác cập nhật cho đến khi thành công.

**Lấy một ví dụ đơn giản**: Giả sử trong bảng thông tin tài khoản cơ sở dữ liệu có một trường `version`, giá trị hiện tại là 1; trường số dư tài khoản (`balance`) hiện tại là $100.

1. Thao tác viên A lúc này đọc ra (`version`=1), và trừ đi $50 từ số dư tài khoản ($100-$50).
2. Trong quá trình thao tác viên A làm việc, thao tác viên B cũng đọc thông tin người dùng này (`version`=1), và trừ $20 từ số dư tài khoản ($100-$20).
3. Thao tác viên A hoàn thành công việc sửa đổi, đem số phiên bản dữ liệu (`version`=1) cùng số dư sau khi trừ (`balance`=$50) submit lên cơ sở dữ liệu để cập nhật. Lúc này do số phiên bản submit bằng phiên bản hiện tại của bản ghi cơ sở dữ liệu, dữ liệu được cập nhật, `version` của bản ghi cơ sở dữ liệu được cập nhật thành 2.
4. Thao tác viên B hoàn thành thao tác, cũng định submit số phiên bản (`version`=1) và dữ liệu (`balance`=$80) lên cơ sở dữ liệu, nhưng lúc này đối chiếu phiên bản bản ghi cơ sở dữ liệu phát hiện số phiên bản thao tác viên B submit là 1, còn phiên bản hiện tại của cơ sở dữ liệu đã là 2, không thỏa mãn chiến lược khóa lạc quan "phiên bản submit phải bằng phiên bản hiện tại mới được thực thi cập nhật", do đó submit của thao tác viên B bị từ chối.

Như vậy đã tránh được việc thao tác viên B dùng kết quả sửa đổi dựa trên dữ liệu cũ `version`=1 ghi đè lên kết quả thao tác của thao tác viên A.

### Thuật toán CAS

Tên đầy đủ của CAS là **Compare And Swap (So sánh và Trao đổi)**, được dùng để triển khai khóa lạc quan, được áp dụng rộng rãi trong các framework lớn. Tư tưởng của CAS rất đơn giản, đó là dùng một giá trị kỳ vọng so sánh với giá trị biến cần cập nhật, chỉ khi hai giá trị bằng nhau thì mới tiến hành cập nhật.

CAS là một thao tác nguyên tử, bên dưới phụ thuộc vào một lệnh nguyên tử của CPU.

> **Thao tác nguyên tử** là thao tác nhỏ nhất không thể chia nhỏ, nghĩa là thao tác một khi bắt đầu thì không thể bị gián đoạn cho đến khi hoàn thành.

CAS liên quan đến 3 toán hạng:

- **V**: Giá trị biến cần cập nhật (Var)
- **E**: Giá trị kỳ vọng (Expected)
- **N**: Giá trị mới định ghi vào (New)

Khi và chỉ khi giá trị của V bằng E, CAS mới dùng cách nguyên tử để cập nhật giá trị của V thành giá trị mới N. Nếu không bằng, chứng tỏ đã có luồng khác cập nhật V, luồng hiện tại từ bỏ cập nhật.

**Lấy một ví dụ đơn giản**: Luồng A muốn sửa giá trị biến i thành 6, giá trị ban đầu của i là 1 (V = 1, E = 1, N = 6, giả sử không có vấn đề ABA).

1. i được so sánh với 1, nếu bằng nhau thì chứng tỏ chưa bị luồng khác sửa đổi, có thể đặt thành 6.
2. i được so sánh với 1, nếu không bằng thì chứng tỏ đã bị luồng khác sửa đổi, luồng hiện tại từ bỏ cập nhật, thao tác CAS thất bại.

Khi nhiều luồng đồng thời dùng CAS để thao tác trên một biến, chỉ có một luồng chiến thắng và cập nhật thành công, các luồng còn lại đều thất bại, nhưng các luồng thất bại không bị treo, mà chỉ được thông báo thất bại và được phép thử lại lần nữa, tất nhiên cũng cho phép luồng thất bại từ bỏ thao tác.

Về bài giới thiệu sâu hơn về CAS, có thể đọc bài viết: [Giải thích chi tiết CAS](./cas.md), trong đó có đề cập chi tiết cách triển khai CAS trong Java cũng như một số vấn đề tồn tại của CAS.

## Tóm tắt

Bài viết này đã giới thiệu chi tiết khái niệm về Khóa lạc quan và Khóa bi quan cũng như các cách triển khai phổ biến của Khóa lạc quan:

- Khóa bi quan dựa trên giả định bi quan, cho rằng tài nguyên dùng chung mỗi lần truy cập đều xảy ra xung đột, do đó mỗi lần thao tác đều sẽ cài khóa. Cơ chế khóa này làm cho các luồng khác bị chặn cho đến khi khóa được giải phóng. `synchronized` và `ReentrantLock` trong Java là các cách triển khai điển hình của Khóa bi quan. Mặc dù Khóa bi quan có thể tránh tranh chấp dữ liệu hiệu quả, nhưng trong kịch bản concurrency cao sẽ dẫn đến luồng bị chặn, chuyển đổi ngữ cảnh thường xuyên, từ đó ảnh hưởng đến hiệu năng hệ thống, và còn có thể gây ra vấn đề deadlock.
- Khóa lạc quan dựa trên giả định lạc quan, cho rằng tài nguyên dùng chung mỗi lần truy cập sẽ không xảy ra xung đột, do đó không cần cài khóa, chỉ cần khi submit sửa đổi xác minh xem dữ liệu có bị luồng khác sửa hay không. Các lớp như `AtomicInteger` và `LongAdder` trong Java triển khai Khóa lạc quan thông qua thuật toán CAS (Compare-And-Swap). Khóa lạc quan tránh được vấn đề chặn luồng và deadlock, trong kịch bản đọc nhiều ghi ít có hiệu năng vượt trội. Nhưng trong trường hợp thao tác ghi thường xuyên, có thể dẫn đến lượng lớn thử lại và thất bại, từ đó ảnh hưởng đến hiệu năng.
- Khóa lạc quan chủ yếu được triển khai thông qua cơ chế số phiên bản hoặc thuật toán CAS. Cơ chế số phiên bản đảm bảo tính nhất quán dữ liệu bằng cách so sánh số phiên bản, còn CAS triển khai thao tác nguyên tử thông qua lệnh phần cứng, trực tiếp so sánh và trao đổi giá trị biến.

Khóa bi quan và Khóa lạc quan đều có ưu nhược điểm riêng, áp dụng cho các kịch bản ứng dụng khác nhau. Trong phát triển thực tế, lựa chọn cơ chế khóa phù hợp có thể nâng cao hiệu quả hiệu năng concurrency và tính ổn định của hệ thống.

## Tham khảo

- 《78 bài cốt lõi về Java Concurrency》
- Dễ hiểu Khóa bi quan, Khóa lạc quan, Khóa reentrant, Spin lock, Biased lock, Lightweight/Heavyweight lock, Read-write lock, các loại khóa và triển khai Java của chúng!: <https://zhuanlan.zhihu.com/p/71156910>

<!-- @include: @article-footer.snippet.md -->
