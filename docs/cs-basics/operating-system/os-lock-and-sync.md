---
title: Chi tiết Khóa và Cơ chế đồng bộ trong Hệ điều hành: Mutex, Semaphore, Condition Variable, Spinlock và Futex
description: Tổng hợp câu hỏi phỏng vấn tần suất cao về Khóa và Đồng bộ trong Hệ điều hành, làm rõ Critical Section, Mutex, Spinlock, Semaphore, Condition Variable, Futex, lệnh nguyên tử, Memory Barrier, Đảo ngược độ ưu tiên và sự khác biệt giữa Khóa User-space và Khóa Linux Kernel.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
  - Lập trình đa luồng
head:
  - - meta
    - name: keywords
      content: Khóa hệ điều hành, Cơ chế đồng bộ, Critical Section, Mutex, Spinlock, Semaphore, Condition Variable, Futex, Lệnh nguyên tử, Memory Barrier, Đảo ngược độ ưu tiên, Linux Kernel Lock, Phỏng vấn Hệ điều hành
---

Hai luồng cùng tăng giá trị một biến đếm `count++` đồng thời, một việc tưởng chừng rất nhỏ, nhưng kết quả cuối cùng lại có thể bị thiếu mất một lần tăng.

Nguyên nhân rất đơn giản: `count++` trong mã nguồn là 1 dòng, nhưng khi CPU thực thi phải trải qua 3 bước máy: **Đọc giá trị cũ (Read) -> Tính toán cộng 1 (Compute) -> Ghi kết quả mới vào bộ nhớ (Write-back)**. Khi Luồng A vừa đọc giá trị cũ nhưng chưa kịp ghi lại, Luồng B cũng đọc đúng giá trị cũ đó. Cả hai cùng tính toán rồi ghi đè lên nhau, làm mất một kết quả.

Để giải quyết các vấn đề tương tranh (Race Condition) này, Hệ điều hành cung cấp Khóa (Lock) và một loạt các cơ chế đồng bộ (Synchronization).

Bài viết này tập trung phân tích dưới góc nhìn Hệ điều hành về các khái niệm: **Mutex, Spinlock, Semaphore, Condition Variable, Futex**.

---

## 1. Bảng so sánh tổng quan các cơ chế đồng bộ

| Cơ chế | Vấn đề cần giải quyết | Phương thức chờ đợi | Kịch bản phổ biến |
| --- | --- | --- | --- |
| **Mutex (Khóa loại trừ tương hỗ)** | Loại trừ tương hỗ trong Miền găng (Critical Section) | Ngủ (Sleep/Block) chờ khóa khả dụng | Bảo vệ cấu trúc dữ liệu dùng chung |
| **Spinlock (Khóa xoay / Khóa tự quay)** | Loại trừ tương hỗ trong Critical Section cực ngắn | Bận chờ (Busy-waiting / Vòng lặp xoay tại chỗ trên CPU) | Đoạn mã không được phép ngủ trong Kernel (như Interrupt Handler) |
| **Semaphore (Đèn hiệu)** | Kiểm soát số lượng tài nguyên / Giới hạn số luồng đồng thời | Đợi khi bộ đếm $= 0$ | Quản lý Connection Pool, bộ đệm Producer-Consumer |
| **Condition Variable (Biến điều kiện)** | Chờ một điều kiện trạng thái chia sẻ trở thành `true` | Nguyên tử nhả Mutex và đi ngủ chờ đánh thức | Hàng đợi không rỗng, tác vụ hoàn thành |
| **Futex (Fast Userspace Mutex)** | Nền tảng chặn/đánh thức luồng hiệu năng cao | Fast-path tại User-space, Slow-path tại Kernel | Triển khai `pthread_mutex`, AQS trong Java |
| **Memory Barrier (Hàng rào bộ nhớ)** | Ràng buộc trật tự truy cập bộ nhớ và tính khả kiến (Visibility) | Không làm chặn luồng | Lập trình Lock-free, giao tiếp thanh ghi phần cứng |

---

## 2. Miền găng (Critical Section) bảo vệ điều gì?

**Miền găng (Critical Section)** là đoạn mã truy cập vào biến/tài nguyên dùng chung có thể thay đổi và không được phép để nhiều luồng cùng xen kẽ thực thi tùy ý.

![Miền găng và giao thức truy cập khóa](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/os-lock-critical-section.png)

4 tiêu chí đánh giá một cơ chế khóa/đồng bộ tốt:
1. **Tính đúng đắn (Correctness / Mutual Exclusion)**: Tại một thời điểm chỉ có tối đa 1 luồng được ở trong miền găng; đảm bảo tính khả kiến bộ nhớ giữa các CPU.
2. **Tính tiến triển (Progress)**: Cơ chế khóa không được tự làm kẹt tất cả các luồng khiến hệ thống bị treo cứng.
3. **Tính công bằng (Fairness / Bounded Waiting)**: Tránh việc một luồng bị bỏ đói (Starvation) vô thời hạn.
4. **Hiệu năng (Performance)**: Khi không có tranh chấp (No Contention), chi phí lấy khóa phải cực nhẹ; khi tranh chấp cao, không được đốt CPU vô ích.

---

## 3. Mutex (Khóa loại trừ tương hỗ)

Mutex đảm bảo nguyên tắc: **Đóng cửa trước khi sửa dữ liệu dùng chung**. Luồng nào lấy được Mutex thì được vào sửa; luồng chưa lấy được thì phải đứng ngoài chờ (hoặc đi ngủ để nhường CPU).

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int count = 0;

void increase(void) {
    pthread_mutex_lock(&mutex);   // Lấy khóa (nếu bận -> ngủ chờ)
    count++;                      // Miền găng an toàn
    pthread_mutex_unlock(&mutex); // Mở khóa (đánh thức luồng chờ)
}
```

Đặc điểm then chốt: **Mutex có ngữ nghĩa sở hữu (Owner Semantics)**: Luồng nào gọi Lock lấy khóa thì bắt buộc chính luồng đó phải gọi Unlock giải phóng khóa.

---

## 4. Spinlock (Khóa xoay / Tự quay)

Khác với Mutex (nếu bận thì luồng đi ngủ và nhường CPU), **Spinlock cho luồng tiếp tục chạy vòng lặp kiểm tra liên tục trên CPU (Busy-waiting)** cho đến khi lấy được khóa.

![So sánh cách chờ đợi giữa Mutex và Spinlock](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/os-lock-mutex-spinlock.png)

### Khi nào nên dùng Spinlock?
- **Miền găng cực ngắn (chỉ vài lệnh máy)**: Chi phí chuyển đổi ngữ cảnh (Context Switch) để luồng đi ngủ rồi thức dậy tốn hàng nghìn chu kỳ CPU. Nếu khóa chỉ được giữ trong vài chu kỳ, việc xoay tại chỗ vài vòng sẽ tiết kiệm chi phí hơn rất nhiều.
- **Trong Kernel không được phép ngủ**: Ví dụ trong các hàm xử lý ngắt cứng (Hardware Interrupt Handler), luồng không được phép ngủ.

### Cảnh báo:
- Miền găng của Spinlock tuyệt đối không được chứa các thao tác I/O (đĩa, mạng) hoặc cấp phát bộ nhớ vì sẽ đốt cháy 100% CPU.
- Ứng dụng phía User-space thông thường **không nên tự viết vòng lặp Spinlock**.

---

## 5. Semaphore (Đèn hiệu)

Semaphore hoạt động như một bộ đếm tài nguyên nguyên tử:
- Khởi tạo $S = N$ (với $N$ là số lượng tài nguyên khả dụng).
- Thao tác `wait()` / `sem_wait()`: Giảm $S$ đi 1. Nếu $S < 0$, luồng đi ngủ vào hàng đợi chờ.
- Thao tác `signal()` / `sem_post()`: Tăng $S$ lên 1. Nếu có luồng đang chờ, đánh thức 1 luồng dậy.

Khác biệt với Mutex: **Semaphore không có khái niệm sở hữu**. Luồng A có thể gọi `wait()` để giảm đếm, và Luồng B hoàn toàn có thể gọi `post()` để tăng đếm (rất thích hợp cho mô hình Producer-Consumer).

---

## 6. Condition Variable (Biến điều kiện)

Biến điều kiện cho phép một luồng đi ngủ để chờ cho đến khi một điều kiện nghiệp vụ cụ thể nào đó trở thành `true` (ví dụ: Hàng đợi có phần tử mới).

Biến điều kiện **luôn luôn phải đi kèm với một Mutex**:
1. Luồng khóa Mutex và kiểm tra điều kiện (trong vòng lặp `while (!condition)`).
2. Nếu điều kiện chưa thỏa mãn, luồng gọi `pthread_cond_wait(&cond, &mutex)`: Hàm này sẽ **nguyên tử nhả Mutex và đưa luồng vào trạng thái ngủ**.
3. Luồng khác sau khi thay đổi điều kiện sẽ gọi `pthread_cond_signal(&cond)` (hoặc `broadcast`) để đánh thức luồng đang ngủ.
4. Luồng thức dậy sẽ tự động lấy lại Mutex và tiếp tục kiểm tra lại điều kiện trong vòng lặp `while`.

---

## 7. Futex (Fast Userspace Mutex - Cốt lõi của khóa hiện đại trong Linux)

Trước khi có Futex, mỗi thao tác Lock/Unlock đều phải gọi System Call chuyển vào Kernel Mode, chi phí rất đắt đỏ ngay cả khi không có tranh chấp.

**Futex giải quyết bài toán này bằng cách chia làm 2 nhánh:**
1. **Fast-path (Không có tranh chấp)**: Thực hiện hoàn toàn tại **User Space** bằng lệnh nguyên tử CAS (Compare-And-Swap) trên biến nguyên trong bộ nhớ RAM, tốc độ chỉ vài nano-giây mà **không tốn một System Call nào**!
2. **Slow-path (Khi có tranh chấp thực sự)**: Nếu CAS thất bại (khóa đang bị chiếm giữ), luồng mới thực hiện System Call gọi xuống Kernel để đưa luồng vào hàng đợi ngủ và chờ đánh thức.

Futex chính là nền tảng cốt lõi bên dưới để hiện thực `pthread_mutex` trong thư viện C và cơ chế `LockSupport.park()/unpark()` của AQS trong Java!

<!-- @include: @article-footer.snippet.md -->
