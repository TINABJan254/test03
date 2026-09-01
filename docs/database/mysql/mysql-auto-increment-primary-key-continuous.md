---
title: MySQL自增主键一定是连续的吗？
description: 详解MySQL自增主键不连续的原因，分析唯一键冲突、事务回滚、批量插入等场景下自增值的分配机制，以及InnoDB自增锁模式的配置与影响。
category: 数据库
tag:
  - MySQL
  - 大厂面试
head:
  - - meta
    - name: keywords
      content: MySQL自增主键,AUTO_INCREMENT,主键不连续,事务回滚,批量插入,唯一键冲突,innodb_autoinc_lock_mode
---

> Tác giả: 飞天小牛肉
>
> Bài gốc: <https://mp.weixin.qq.com/s/qci10h9rJx_COZbHV3aygQ>

Ai cũng biết rằng Primary Key tự tăng giúp Clustered Index duy trì thứ tự tăng dần khi chèn dữ liệu, tránh việc tìm kiếm ngẫu nhiên, qua đó nâng cao hiệu quả truy vấn.

Tuy nhiên trên thực tế, Primary Key tự tăng trong MySQL không đảm bảo chắc chắn liên tục tăng dần.

Dưới đây là một ví dụ tạo bảng:

![](https://oss.javaguide.cn/p3-juejin/3e6b80ba50cb425386b80924e3da0d23~tplv-k3u1fbpfcp-zoom-1.png)

## Giá trị tự tăng được lưu trữ ở đâu?

Sử dụng `insert into test_pk values(null, 1, 1)` để chèn một hàng dữ liệu, sau đó thực thi lệnh `show create table` để xem định nghĩa cấu trúc bảng:

![](https://oss.javaguide.cn/p3-juejin/c17e46230bd34150966f0d86b2ad5e91~tplv-k3u1fbpfcp-zoom-1.png)

Cấu trúc bảng trên được lưu trong file cục bộ đuôi `.frm` tại thư mục data của MySQL:

![](https://oss.javaguide.cn/p3-juejin/3ec0514dd7be423d80b9e7f2d52f5902~tplv-k3u1fbpfcp-zoom-1.png)

Từ định nghĩa cấu trúc bảng ở trên có thể thấy xuất hiện `AUTO_INCREMENT=2`, nghĩa là lần chèn dữ liệu tiếp theo nếu cần tự động tạo giá trị tự tăng sẽ tạo ra `id = 2`.

Tuy nhiên cần lưu ý, giá trị tự tăng không được lưu trữ trong file định nghĩa cấu trúc bảng `.frm`. Các Storage Engine khác nhau có chiến lược lưu trữ giá trị tự tăng khác nhau:

1) Engine MyISAM lưu giá trị tự tăng trong file dữ liệu.
2) Engine InnoDB lưu giá trị tự tăng trong RAM (bộ nhớ tạm), không persistence. Lần đầu mở bảng sẽ tìm giá trị lớn nhất `max(id)` rồi lấy `max(id) + 1` làm giá trị tự tăng hiện tại cho bảng.

Ví dụ: Bảng hiện tại có id lớn nhất là 1, `AUTO_INCREMENT=2`. Lúc này xóa hàng `id=1`, `AUTO_INCREMENT` vẫn là 2.

![](https://oss.javaguide.cn/p3-juejin/61b8dc9155624044a86d91c368b20059~tplv-k3u1fbpfcp-zoom-1.png)

Nhưng nếu lập tức khởi động lại MySQL instance, sau khi restart giá trị `AUTO_INCREMENT` của bảng này sẽ biến thành 1. Nghĩa là khởi động lại MySQL có thể làm thay đổi giá trị `AUTO_INCREMENT` của một bảng (trong MySQL 5.x).

![](https://oss.javaguide.cn/p3-juejin/27fdb15375664249a31f88b64e6e5e66~tplv-k3u1fbpfcp-zoom-1.png)

![](https://oss.javaguide.cn/p3-juejin/dee15f93e65d44d384345a03404f3481~tplv-k3u1fbpfcp-zoom-1.png)

**Từ phiên bản MySQL 8.0 trở đi, lịch sử thay đổi giá trị tự tăng được ghi vào `redo log`, cung cấp khả năng persistence cho giá trị tự tăng** (sau khi restart giá trị tự tăng sẽ được khôi phục dựa vào redo log).

Sau khi hiểu giá trị tự tăng lưu ở đâu, chúng ta cùng xem cơ chế sửa đổi giá trị tự tăng và các kịch bản dẫn đến tính không liên tục của giá trị tự tăng.

## Các kịch bản giá trị tự tăng không liên tục

### Kịch bản 1: Thiết lập giá trị khởi tạo và bước nhảy (Step) không bằng 1

Trong MySQL, nếu field `id` được định nghĩa là `AUTO_INCREMENT`, khi chèn một hàng dữ liệu:
- Nếu chỉ định `id` là 0, null hoặc không chỉ định, lấy giá trị `AUTO_INCREMENT` hiện tại điền vào field tự tăng;
- Nếu chỉ định giá trị cụ thể cho `id`, dùng trực tiếp giá trị chỉ định.

Khoảng cách bước nhảy được quyết định bởi hai tham số `auto_increment_offset` (giá trị khởi tạo) và `auto_increment_increment` (bước nhảy). Nếu hai tham số này được cấu hình khác 1 (thường dùng trong kiến trúc phân tán/multi-master để tránh xung đột Primary Key giữa các database node), ID tự tăng được cấp phát sẽ cách nhau theo bước nhảy, dẫn đến dãy ID không liên tục liên tiếp.

### Kịch bản 2: Xung đột Unique Key (Duy nhất)

Giả sử chèn bản ghi `(null, 1, 1)` thành công với `id = 1`, `AUTO_INCREMENT = 2`.

![](https://oss.javaguide.cn/p3-juejin/c22c4f2cea234c7ea496025eb826c3bc~tplv-k3u1fbpfcp-zoom-1.png)

Sau đó thực thi tiếp lệnh chèn `(null, 1, 1)`. Do field `a` là Unique Index nên câu lệnh báo lỗi `Duplicate entry`:

![](https://oss.javaguide.cn/p3-juejin/c0325e31398d4fa6bb1cbe08ef797b7f~tplv-k3u1fbpfcp-zoom-1.png)

Tuy nhiên, bạn sẽ kinh ngạc phát hiện giá trị `AUTO_INCREMENT` vẫn tăng từ 2 lên 3!

Quy trình thực thi câu lệnh INSERT:
1. Executor gọi API InnoDB chuẩn bị chèn bản ghi `(null, 1, 1)`.
2. InnoDB thấy chưa chỉ định `id`, lấy giá trị `AUTO_INCREMENT` hiện tại là 2.
3. Đổi bản ghi thành `(2, 1, 1)`.
4. Đổi `AUTO_INCREMENT` của bảng thành 3.
5. Thực hiện chèn dữ liệu, gặp xung đột Unique Key ở `a=1` báo lỗi `Duplicate key error` và dừng lại.

Thao tác sửa đổi giá trị tự tăng diễn ra **trước khi** thực sự chèn dữ liệu. Do xung đột Unique Key, hàng `id = 2` chèn thất bại nhưng giá trị tự tăng không được lùi lại. Lần chèn tiếp theo sẽ nhận `id = 3`, tạo ra khoảng trống không liên tục.

### Kịch bản 3: Transaction Rollback (Hoàn tác Transaction)

Giả sử bảng có bản ghi `(1, 1, 1)`, `AUTO_INCREMENT = 3`.

![](https://oss.javaguide.cn/p3-juejin/6220fcf7dac54299863e43b6fb97de3e~tplv-k3u1fbpfcp-zoom-1.png)

Mở một Transaction chèn dữ liệu `(null, 2, 2)` (nhận `id = 3`), `AUTO_INCREMENT` tăng lên 4.

![](https://oss.javaguide.cn/p3-juejin/3f02d46437d643c3b3d9f44a004ab269~tplv-k3u1fbpfcp-zoom-1.png)

Sau đó thực hiện `ROLLBACK`:

![](https://oss.javaguide.cn/p3-juejin/faf5ce4a2920469cae697f845be717f5~tplv-k3u1fbpfcp-zoom-1.png)

Bản ghi `(3, 2, 2)` không có trong bảng, nhưng giá trị `AUTO_INCREMENT` vẫn giữ nguyên là 4 chứ không lùi về 3. Lần chèn tiếp theo sẽ nhận `id = 4`.

**Tại sao MySQL không lùi lại giá trị tự tăng khi xung đột Unique Key hoặc Rollback Transaction?**

Lý do chính là để **nâng cao hiệu năng**:

Giả sử Transaction A xin được `id = 1`, Transaction B xin được `id = 2`. `AUTO_INCREMENT` của bảng là 3.
- Transaction B commit thành công.
- Transaction A bị xung đột Unique Key hoặc rollback. Nếu cho phép Transaction A lùi `AUTO_INCREMENT` về 1, trong bảng đã có hàng `id = 2` nhưng `AUTO_INCREMENT` lại là 1.
- Lần chèn tiếp theo xin được `id = 1` hoặc `id = 2` sẽ phát sinh lỗi xung đột Primary Key.

Để khắc phục vấn đề này sẽ phải dùng các phương án tốn rất nhiều chi phí hiệu năng (như kiểm tra trước id tồn tại hay mở rộng thời gian giữ Lock khóa tự tăng cho tới khi Transaction commit). Vì thế InnoDB chấp nhận thiết kế không lùi giá trị tự tăng, chỉ đảm bảo ID tăng dần chứ không đảm bảo liên tục.

### Kịch bản 4: Chèn hàng loạt (Batch Insert)

Đối với các câu lệnh chèn hàng loạt không biết trước số lượng bản ghi (như `INSERT ... SELECT`, `REPLACE ... SELECT`, `LOAD DATA`), MySQL áp dụng chiến lược cấp phát ID tự tăng theo cấp số nhân:
- Lần 1: cấp 1 ID
- Lần 2: cấp 2 ID
- Lần 3: cấp 4 ID
- Lần 4: cấp 8 ID...

Lần thứ 3 cấp 4 ID (`id = 4, 5, 6, 7`), nhưng thực tế chỉ dùng đến `id = 5`. Hai ID 6 và 7 bị bỏ phí, làm cho `AUTO_INCREMENT` tăng lên 8. Bản ghi chèn tiếp theo sẽ nhận `id = 8`.

## Tóm tắt

4 kịch bản khiến Primary Key tự tăng trong MySQL không liên tục:

1. `auto_increment_offset` và `auto_increment_increment` thiết lập khác 1.
2. Xung đột Unique Key.
3. Transaction Rollback.
4. Chèn hàng loạt (như câu lệnh `INSERT ... SELECT`).

<!-- @include: @article-footer.snippet.md -->
