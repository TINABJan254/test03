---
title: 3种常用的缓存读写策略详解
description: 深入对比 Cache Aside、Read/Write Through、Write Behind 三种缓存读写策略，附详细时序图、一致性问题分析及生产级解决方案，Redis 实战必备！
category: 数据库
tag:
  - Redis
head:
  - - meta
    - name: keywords
      content: 缓存读写策略,Cache Aside,Read Through,Write Through,Write Behind,Write Back,缓存一致性,缓存失效,旁路缓存,读写穿透,异步缓存写入,Redis缓存策略,缓存更新策略
---

Không có chiến lược nào là tốt nhất tuyệt đối, chỉ có chiến lược phù hợp nhất với kịch bản nghiệp vụ cụ thể. Dưới đây là phân tích chi tiết 3 chiến lược đọc ghi Cache phổ biến.

### Cache Aside Pattern (Mô hình Side-car Cache / 旁路缓存模式)

Đây là mô hình **phổ biến và kinh điển nhất** trong phát triển hàng ngày, gần như là chuẩn thực tế cho ứng dụng Internet, đặc biệt phù hợp kịch bản **Đọc nhiều Ghi ít (Read-heavy)**.

Mô hình này gọi là "Aside" (bên lề/bên cạnh) vì **thao tác ghi của ứng dụng hoàn toàn bỏ qua Cache, trực tiếp thao tác trên Database**. Ứng dụng đóng vai trò "chỉ huy" điều phối cả Cache và DB.

Các bước đọc ghi dữ liệu:

**Thao tác Ghi (Write):**

1. Ứng dụng **cập nhật Database trước**.
2. Sau đó **trực tiếp xóa Cache (Delete Cache)** chứa dữ liệu tương ứng.

![](https://oss.javaguide.cn/github/javaguide/database/redis/cache-aside-write.png)

**Thao tác Đọc (Read):**

1. Ứng dụng đọc dữ liệu từ Cache trước.
2. Nếu trúng (Hit) thì trả về trực tiếp.
3. Nếu không trúng (Miss) thì đọc từ DB. Đọc thành công sẽ **ghi ngược dữ liệu vào Cache** rồi trả về.

![](https://oss.javaguide.cn/github/javaguide/database/redis/cache-aside-read.png)

Các câu hỏi phỏng vấn hay hỏi sâu:

**1. Tại sao thao tác ghi lại là "Cập nhật DB trước, Xóa Cache sau"? Thứ tự có đảo lại được không?**

- **Trả lời**: Tuyệt đối không! Nếu "Xóa Cache trước, Cập nhật DB sau", dưới môi trường đồng thời cao (Concurrency) sẽ dẫn tới mất nhất quán dữ liệu:
  1. Request A: Xóa dữ liệu trong Cache.
  2. Request B: Đọc thấy Cache rỗng, ra DB đọc **giá trị CŨ** rồi chuẩn bị ghi vào Cache.
  3. Request A: Ghi **giá trị MỚI** vào DB.
  4. Request B: Ghi **giá trị CŨ** vừa đọc vào Cache.
  -> Kết quả: DB lưu giá trị MỚI nhưng Cache lưu giá trị CŨ.

**2. Thao tác "Cập nhật DB trước, Xóa Cache sau" có an toàn tuyệt đối không?**

- **Trả lời**: Không tuyệt đối 100%! Vẫn có xác suất gây mất nhất quán dữ liệu (Request A đọc Miss DB lấy giá trị CŨ -> Request B update DB thành công và xóa Cache -> Request A mới nạp giá trị CŨ vào Cache). Tuy nhiên xác suất cực kỳ nhỏ vì thời gian đọc DB và fill-back Cache ngắn hơn rất nhiều thời gian thực thi ghi DB.

**3. Tại sao lại là "Xóa Cache" mà không phải "Cập nhật Cache"?**

- **Chi phí tính toán**: Thao tác ghi nhiều khi chỉ sửa 1 vài field, việc query lại toàn bộ object để Cập nhật Cache rất tốn tài nguyên. Xóa là thao tác siêu nhẹ.
- **Tư tưởng Lazy Load**: Chỉ khi dữ liệu thực sự cần đọc ở lần sau mới kích hoạt nạp vào Cache.
- **An toàn đồng thời**: Cập nhật Cache dưới môi trường concurrency dễ vướng lỗi thứ tự ghi dẫn tới dữ liệu bẩn.

**Hạn chế của Cache Aside Pattern:**

- **Lần đầu tiên request dữ liệu chắc chắn không có trong Cache**: Khắc phục bằng Pre-heat Cache (Khởi động nạp trước dữ liệu nóng).
- **Thao tác ghi quá thường xuyên làm Cache liên tục bị xóa -> Tỷ lệ Hit thấp**: Khắc phục bằng cách gán khóa phân tán (Distributed Lock) khi cập nhật cả DB và Cache, hoặc đặt TTL ngắn.

### Read/Write Through Pattern (Chiến lược Đọc/Ghi xuyên qua)

Ở mô hình này, ứng dụng coi **Cache là bộ lưu trữ duy nhất và chính yếu**. Tất cả request đọc ghi đều đánh trực tiếp vào Cache, bản thân Cache Service chịu trách nhiệm đồng bộ dữ liệu với DB.

Minh họa thao tác Ghi (Write Through):
- Kiểm tra Cache, nếu chưa có thì update DB trực tiếp.
- Nếu có trong Cache thì update Cache trước, sau đó Cache Service tự update DB. Cả 2 thành công mới trả về client.

![](https://oss.javaguide.cn/github/javaguide/database/redis/write-through.png)

Minh họa thao tác Đọc (Read Through):
- Ứng dụng đọc dữ liệu từ Cache.
- Nếu Miss, **bản thân Cache Service** chịu trách nhiệm nạp dữ liệu từ DB, nạp xong ghi vào Cache rồi trả về ứng dụng.

![](https://oss.javaguide.cn/github/javaguide/database/redis/read-through.png)

Về bản chất, Read-Through chính là việc đóng gói logic "Cache Miss -> Đọc DB -> Fill-back Cache" đẩy xuống bên trong Cache Service, hoàn toàn trong suốt (transparent) với ứng dụng client.

### Write Behind Pattern (Ghi bất đồng bộ / Write-Back)

Write Behind tương tự Read/Write Through ở chỗ Cache Service chịu trách nhiệm đọc ghi giữa Cache và DB.

Điểm khác biệt lớn: **Read/Write Through là đồng bộ update Cache và DB, còn Write Behind chỉ update Cache rồi trả về ngay lập tức, sau đó dùng công việc bất đồng bộ gộp theo lô (batch) để update DB.**

Thao tác Ghi (Write Behind):
1. Ứng dụng ghi dữ liệu vào Cache rồi **lập tức trả về**.
2. Cache Service đưa thao tác ghi này vào một Queue.
3. Một thread/task bất đồng bộ lấy các thao tác ghi trong Queue ra để **ghi theo batch** xuống DB.

- Tốc độ ghi siêu nhanh.
- Nhược điểm: Nguy cơ mất dữ liệu nếu Cache sập trước khi kịp ghi xuống DB.
- Ứng dụng thực tế: InnoDB Buffer Pool của MySQL (sửa trên RAM Buffer Pool trước rồi thread ngầm mới flush xuống đĩa), Page Cache của hệ điều hành, tính năng đếm lượt View/Like bài viết với traffic cực cao.

<!-- @include: @article-footer.snippet.md -->
