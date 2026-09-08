---
title: Chi tiết Deadlock: Bốn điều kiện cần, Điều tra Deadlock trong Java và Xử lý Deadlock Database
description: Phân tích toàn diện về Deadlock (Bế tắc): Bản chất đồ thị tài nguyên, 4 điều kiện cần của Coffman, các chiến lược phòng ngừa/tránh/phát hiện deadlock, cách dò tìm Deadlock trong Java (jstack, ThreadMXBean) và xử lý Deadlock trong Cơ sở dữ liệu (MySQL InnoDB).
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Deadlock
  - Java Concurrency
  - MySQL
head:
  - - meta
    - name: keywords
      content: Deadlock, Bế tắc, 4 điều kiện cần, Coffman, Đồ thị tài nguyên, Thuật toán Banker, Java Deadlock, jstack, MySQL Deadlock, Deadlock Detection, Retry
---

<!-- @include: @article-header.snippet.md -->

**Deadlock (Bế tắc)** là hiện tượng hai hoặc nhiều tiến trình (hoặc luồng) bị treo vĩnh viễn do mỗi bên đều đang nắm giữ tài nguyên mà bên kia cần, đồng thời đang chờ bên kia giải phóng tài nguyên của họ.

```text
Luồng 1: Giữ Khóa A  ──(Đang chờ)──>  Cần Khóa B
Luồng 2: Giữ Khóa B  ──(Đang chờ)──>  Cần Khóa A
```

Cả hai luồng sẽ chờ đợi nhau mãi mãi nếu không có sự can thiệp từ bên ngoài!

Bài viết này sẽ làm rõ toàn bộ kiến trúc về Deadlock từ lý thuyết hệ điều hành đến thực chiến Java và Cơ sở dữ liệu.

---

## 1. Bốn điều kiện cần để xảy ra Deadlock (Coffman Conditions)

Deadlock **chỉ xảy ra khi đồng thời thỏa mãn cả 4 điều kiện sau**:

1. **Loại trừ tương hỗ (Mutual Exclusion)**: Tài nguyên chỉ có thể được sử dụng bởi một tiến trình tại một thời điểm (không thể chia sẻ).
2. **Giữ và chờ (Hold and Wait)**: Một tiến trình đang nắm giữ ít nhất một tài nguyên và đang tiếp tục yêu cầu thêm tài nguyên mới đang bị tiến trình khác chiếm giữ.
3. **Không chiếm đoạt (No Preemption)**: Tài nguyên đã cấp cho tiến trình thì không thể bị ép buộc thu hồi giữa chừng, chỉ có thể do tiến trình tự nguyện giải phóng.
4. **Chờ đợi vòng tròn (Circular Wait)**: Tồn tại một chuỗi vòng tròn các tiến trình $\{P_0, P_1, \dots, P_n\}$, trong đó $P_0$ chờ tài nguyên của $P_1$, $P_1$ chờ $P_2$, ..., và $P_n$ lại chờ tài nguyên của $P_0$.

> **Nguyên tắc phá vỡ Deadlock**: Chỉ cần phá vỡ **bất kỳ 1 trong 4 điều kiện trên**, Deadlock sẽ không bao giờ xảy ra!

---

## 2. Các chiến lược xử lý Deadlock trong Hệ điều hành

1. **Bỏ qua (Ignorance - Thuật toán đà điểu Ostrich Algorithm)**: Coi như deadlock không bao giờ xảy ra (hầu hết hệ điều hành như Linux/Windows áp dụng cho các tiến trình user thông thường vì chi phí phòng ngừa liên tục quá đắt đỏ so với tần suất xảy ra).
2. **Phòng ngừa (Deadlock Prevention)**: Thiết kế hệ thống sao cho vi phạm 1 trong 4 điều kiện Coffman:
   - Phá vỡ *Giữ và chờ*: Bắt buộc tiến trình phải yêu cầu toàn bộ tài nguyên cùng một lúc trước khi chạy.
   - Phá vỡ *Chờ đợi vòng tròn*: **Đánh số thứ tự toàn cục cho các tài nguyên**, quy định mọi tiến trình bắt buộc phải xin cấp khóa theo đúng thứ tự tăng dần của ID tài nguyên (đây là giải pháp phổ biến nhất trong thực tế!).
3. **Tránh Deadlock (Deadlock Avoidance)**: Đánh giá trạng thái an toàn trước mỗi lần cấp phát tài nguyên (sử dụng **Thuật toán Ngân hàng - Banker's Algorithm**).
4. **Phát hiện và Phục hồi (Detection and Recovery)**: Định kỳ quét đồ thị cấp phát tài nguyên để tìm chu trình vòng tròn; khi phát hiện deadlock thì chọn hy sinh (Kill/Rollback) một tiến trình để cứu các tiến trình còn lại.

---

## 3. Điều tra và xử lý Deadlock trong Java

### Code Java minh họa Deadlock kinh điển:

```java
public class DeadLockDemo {
    private static final Object resource1 = new Object();
    private static final Object resource2 = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + " get resource1");
                try { Thread.sleep(1000); } catch (InterruptedException e) {}
                System.out.println(Thread.currentThread() + " waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + " get resource2");
                }
            }
        }, "Thread-1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + " get resource2");
                try { Thread.sleep(1000); } catch (InterruptedException e) {}
                System.out.println(Thread.currentThread() + " waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + " get resource1");
                }
            }
        }, "Thread-2").start();
    }
}
```

### Công cụ điều tra Deadlock trong Java:
1. **Sử dụng lệnh `jstack <pid>`**:
   - `jstack` sẽ tự động phân tích đồ thị Thread Lock và in ra thông báo rõ ràng:
   ```text
   Found one Java-level deadlock:
   =============================
   "Thread-2":
     waiting to lock monitor 0x00007f... (object for class java.lang.Object)
     which is held by "Thread-1"
   "Thread-1":
     waiting to lock monitor 0x00007f... (object for class java.lang.Object)
     which is held by "Thread-2"
   ```
2. **Sử dụng công cụ trực quan**: VisualVM, JConsole hoặc Arthas (`thread -b`).
3. **Phát hiện bằng code trong ứng dụng**: Sử dụng `ThreadMXBean.findDeadlockedThreads()`.

### Cách sửa Deadlock trong Java:
- **Khóa theo thứ tự cố định**: Đảm bảo cả Thread 1 và Thread 2 đều lấy `resource1` trước rồi mới lấy `resource2`.
- Sử dụng `ReentrantLock.tryLock(timeout, unit)`: Đặt thời gian chờ lấy khóa, nếu quá hạn thì chủ động nhả khóa đang giữ để tránh bị kẹt vĩnh viễn.

---

## 4. Xử lý Deadlock trong Cơ sở dữ liệu (MySQL InnoDB)

Trong MySQL InnoDB, Deadlock thường xảy ra khi hai Transaction đồng thời cập nhật nhiều dòng dữ liệu theo thứ tự chéo nhau hoặc do cơ chế khóa khoảng cách (Gap Lock / Next-Key Lock).

### Cơ chế tự động phát hiện Deadlock của MySQL
- InnoDB tích hợp sẵn bộ phát hiện chu trình đồ thị chờ đợi (**Deadlock Detector**).
- Khi phát hiện Deadlock, InnoDB sẽ tự động chọn Transaction có chi phí rollback nhỏ nhất (Transaction sửa đổi ít dòng dữ liệu nhất) để **chủ động Rollback**, trả về mã lỗi:
  ```text
  ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  ```
- Transaction còn lại được tiếp tục thực thi thành công.

### Xem chi tiết Deadlock trong MySQL
Chạy lệnh SQL:
```sql
SHOW ENGINE INNODB STATUS;
```
Trong phần `LATEST DETECTED DEADLOCK`, MySQL sẽ hiển thị chi tiết câu lệnh SQL, bảng dữ liệu, kiểu khóa (Record Lock, Gap Lock) của 2 Transaction gây ra deadlock.

### Giải pháp phòng chống Deadlock Database:
1. Luôn cập nhật các bảng/dòng dữ liệu theo một thứ tự cố định trong toàn bộ dự án.
2. Thiết kế Index phù hợp để câu lệnh SQL tìm đúng dòng cần khóa (tránh việc không có index khiến MySQL phải khóa toàn bộ bảng bằng Gap Lock).
3. Giảm phạm vi Transaction, commit càng sớm càng tốt.
4. Ở tầng ứng dụng Backend (Spring Boot), thiết lập cơ chế **Tự động thử lại Transaction (Retry Policy)** khi bắt được lỗi Deadlock.

<!-- @include: @article-footer.snippet.md -->
