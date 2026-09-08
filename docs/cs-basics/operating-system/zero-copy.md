---
title: Chi tiết Zero-Copy: mmap, sendfile và splice
description: Phân tích toàn diện kỹ thuật Zero-Copy trong Hệ điều hành: So sánh quy trình đọc ghi I/O truyền thống với mmap, sendfile, splice, cơ chế DMA Gather Copy và các ứng dụng kinh điển trong Java NIO (FileChannel.transferTo), Kafka, RocketMQ, Netty.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
  - Zero-Copy
  - Hiệu năng cao
head:
  - - meta
    - name: keywords
      content: Zero-Copy, Không sao chép, mmap, sendfile, splice, DMA, Page Cache, Java NIO, FileChannel, Kafka, Netty, RocketMQ
---

Tại sao **Apache Kafka** có thể đạt thông lượng hàng triệu tin nhắn mỗi giây mà không làm nghẽn CPU? Tại sao **Netty** lại là nền tảng mạng hiệu năng cao số 1 trong hệ sinh thái Java?

Bí quyết nằm ở công nghệ **Zero-Copy (Không sao chép dữ liệu)** của hệ điều hành.

Bài viết này sẽ làm rõ Zero-Copy thực chất đã tiết kiệm những thao tác nào.

---

## 1. Quy trình truyền file truyền thống (Traditional File Transfer)

Hãy xem xét kịch bản kinh điển: Đọc một file từ ổ đĩa và gửi qua Socket mạng ra Internet (ví dụ Web Server phục vụ file tĩnh hoặc Kafka đọc log gửi cho Consumer):

```c
read(file_fd, buf, len);
write(socket_fd, buf, len);
```

![Quy trình truyền file truyền thống trải qua 4 lần chuyển Context Switch và 4 lần Copy dữ liệu](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/traditional-io-copy-process.png)

Để hoàn thành 2 dòng code trên, hệ điều hành phải thực hiện:
- **4 lần chuyển đổi Context Switch** (giữa User Mode và Kernel Mode).
- **4 lần sao chép dữ liệu (Data Copies)**:
  1. *Copy 1 (DMA)*: Đọc dữ liệu từ Ổ đĩa vào Page Cache trong Kernel.
  2. *Copy 2 (CPU)*: Sao chép từ Page Cache Kernel sang User Buffer trong ứng dụng.
  3. *Copy 3 (CPU)*: Sao chép từ User Buffer sang Socket Buffer trong Kernel.
  4. *Copy 4 (DMA)*: Sao chép từ Socket Buffer sang Card mạng (NIC) để phát ra cáp mạng.

> **Điểm nghẽn**: Dữ liệu chỉ đơn thuần là chuyển từ đĩa ra mạng mà phải đi vòng qua User Space, tốn 2 lần CPU Copy và 4 lần Mode Switch lãng phí tài nguyên!

---

## 2. Giải pháp 1: `mmap` + `write` (Memory-Mapped File)

Thay vì dùng `read()`, ứng dụng dùng hàm `mmap()` để ánh xạ trực tiếp vùng Page Cache của file trong Kernel vào không gian địa chỉ ảo của tiến trình:

```c
buf = mmap(NULL, len, PROT_READ, MAP_SHARED, file_fd, 0);
write(socket_fd, buf, len);
```

- **Kết quả**:
  - Giảm xuống còn **3 lần sao chép dữ liệu** (tiết kiệm được bước copy dữ liệu sang User Buffer).
  - Vẫn tốn **4 lần Context Switch**.
- **Ứng dụng thực tế**: Được **RocketMQ** sử dụng cho các file commitlog dung lượng nhỏ (< 1GB) vì ứng dụng có thể trực tiếp can thiệp và sửa đổi dữ liệu trong bộ nhớ trước khi gửi.

---

## 3. Giải pháp 2: `sendfile` (Zero-Copy thực sự)

Trong Linux 2.1, System Call **`sendfile()`** ra đời, cho phép truyền dữ liệu trực tiếp giữa 2 File Descriptor hoàn toàn bên trong Kernel:

```c
sendfile(socket_fd, file_fd, NULL, len);
```

![sendfile truyền dữ liệu trực tiếp trong Kernel](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/sendfile-zero-copy-process.png)

- **Kết quả**:
  - Chỉ tốn **2 lần Context Switch** (chỉ gọi 1 hàm `sendfile()`).
  - Dữ liệu không đi qua User Space.

### Bước tiến vượt bậc: `sendfile` kết hợp DMA Scatter-Gather Copy (Linux 2.4+)
Nếu Card mạng phần cứng hỗ trợ tính năng **Scatter-Gather**:
- CPU không cần sao chép dữ liệu từ Page Cache sang Socket Buffer nữa!
- Kernel chỉ cần chuyển các con trỏ mô tả (File Descriptor & Độ dài) vào Socket Buffer.
- Card mạng sẽ sử dụng bộ điều khiển **DMA Engine đọc trực tiếp từ Page Cache của Kernel** để phát ra mạng!

**Kết quả đạt được Zero-Copy lý tưởng**:
- **2 lần Context Switch**.
- **0 lần CPU Copy** (chỉ có 2 lần DMA Copy tự động bằng phần cứng, CPU hoàn toàn rảnh rỗi 100%!).

---

## 4. Giải pháp 3: `splice` (Truyền dữ liệu qua Pipe)

System Call `splice()` (từ Linux 2.6.17) cho phép di chuyển dữ liệu giữa 2 File Descriptor (trong đó ít nhất một bên là Pipe) hoàn toàn trong Kernel thông qua cơ chế chia sẻ con trỏ buffer, không cần copy dữ liệu và hỗ trợ cả các luồng dữ liệu tùy ý.

---

## 5. Ứng dụng thực tế trong hệ sinh thái Java

1. **Java NIO `FileChannel.transferTo()` / `transferFrom()`**:
   - Bên dưới gọi trực tiếp System Call `sendfile()` của Linux kernel.
2. **Apache Kafka**:
   - Khi Consumer đọc tin nhắn, Kafka sử dụng `FileChannel.transferTo()` để đọc dữ liệu từ tệp tin Log trên đĩa và bắn thẳng ra Socket mạng gửi cho Consumer mà không tốn một chu kỳ CPU nào cho việc sao chép dữ liệu.
3. **Netty**:
   - Hỗ trợ Zero-Copy ở cả cấp độ Hệ điều hành (`FileRegion` / `DefaultFileRegion` dùng `sendfile`) và cấp độ User-space JVM (`ByteBuf` dạng `CompositeByteBuf`, `slice()`, `wrap()` chia sẻ bộ nhớ đệm mà không tạo mảng byte mới).

<!-- @include: @article-footer.snippet.md -->
