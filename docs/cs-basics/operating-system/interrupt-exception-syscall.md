---
title: Chi tiết Ngắt, Ngoại lệ và System Call: Từ cửa ngõ Kernel đến Page Fault
description: Tổng hợp câu hỏi phỏng vấn tần suất cao về Ngắt, Ngoại lệ và System Call, lấy hàm read() làm sợi dây liên kết để làm rõ mối quan hệ giữa Ngắt phần cứng, Ngoại lệ đồng bộ, System Call, Tín hiệu (Signal), Ngắt đồng hồ, Page Fault và Chuyển đổi ngữ cảnh luồng.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
  - System Call
head:
  - - meta
    - name: keywords
      content: Ngắt, Interrupt, Ngoại lệ, Exception, System Call, Trap, Signal, User Mode, Kernel Mode, Context Switch, Ngắt đồng hồ, Page Fault, SIGSEGV, read, Phỏng vấn Hệ điều hành
---

System Call (Lời gọi hệ thống) chuyển từ User Mode (Chế độ người dùng) sang Kernel Mode (Chế độ nhân) chỉ là điểm bắt đầu. Lần theo một lệnh gọi `read(fd, buf, count)`, chúng ta sẽ gặp một loạt câu hỏi liên quan mật thiết:

- Lệnh `read()` đi vào Kernel như thế nào?
- Tại sao ngắt đồng hồ (Timer Interrupt) lại có thể buộc một luồng đang chạy phải dừng lại để nhường CPU?
- Tại sao Page Fault đôi khi là hành vi bình thường, nhưng đôi khi lại biến thành lỗi `SIGSEGV` (Segmentation Fault)?
- Khi thực hiện System Call vào Kernel, có bắt buộc phải xảy ra Chuyển đổi ngữ cảnh luồng (Thread Context Switch) không?

Bài viết này sẽ xâu chuỗi toàn bộ các khái niệm trên thông qua một luồng xử lý I/O điển hình.

---

## 1. Các loại sự kiện dẫn vào Kernel (Exceptional Control Flow)

Khi CPU đang thực thi chương trình của người dùng, lệnh tiếp theo được quyết định bởi con trỏ lệnh PC. Bất kỳ sự kiện ngoại vi, lỗi chỉ lệnh hiện tại, hoặc yêu cầu dịch vụ chủ động nào đều làm chuyển hướng luồng điều khiển vào Kernel:

1. **Ngắt (Interrupt / Hardware Interrupt)**: Đến từ phần cứng ngoại vi bên ngoài, **hoàn toàn bất đồng bộ** với lệnh CPU hiện tại (ví dụ: Card mạng nhận được gói tin, Ổ đĩa đọc xong dữ liệu, Timer định kỳ của đồng hồ hệ thống).
2. **Bẫy / Hãm (Trap / System Call)**: Chương trình người dùng **chủ động** thực thi chỉ lệnh đặc biệt (như `syscall` trên x86-64 hoặc `int 0x80`) để xin Kernel phục vụ.
3. **Lỗi / Sự cố (Fault)**: Xảy ra do lệnh hiện tại gặp trục trặc nhưng **có thể được Kernel khắc phục và thử lại** (kinh điển nhất là **Page Fault - Ngoại lệ lỗi trang**; sau khi Kernel nạp trang từ đĩa vào RAM sẽ cho CPU chạy lại đúng chỉ lệnh đó).
4. **Hủy bỏ (Abort)**: Phần cứng phát hiện lỗi nghiêm trọng không thể phục hồi (như lỗi hỏng RAM phần cứng, lỗi nguồn), Kernel buộc phải chấm dứt chương trình.

![Sơ đồ mối quan hệ giữa Ngắt, Ngoại lệ và System Call](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/ecf-kernel-entry-map.webp)

---

## 2. So sánh Ngắt, Ngoại lệ, System Call và Tín hiệu (Signal)

| Khái niệm | Nguồn kích hoạt | Tính chất | Ai xử lý | Kết quả phổ biến |
| --- | --- | --- | --- | --- |
| **Ngắt phần cứng** | Thiết bị ngoại vi hoặc Timer | Bất đồng bộ | Trình xử lý ngắt trong Kernel (ISR) | Đánh thức luồng chờ, kích hoạt lập lịch |
| **Ngoại lệ đồng bộ** | Do chỉ lệnh CPU hiện tại gây ra (chia cho 0, lỗi trang) | Đồng bộ | Trình xử lý ngoại lệ trong Kernel | Khắc phục thử lại hoặc chuyển thành Signal |
| **System Call (Trap)** | Lệnh gọi chủ động `syscall` / `ecall` | Đồng bộ | Cửa ngõ System Call trong Kernel | Trả về kết quả hoặc đưa luồng vào giấc ngủ |
| **Tín hiệu (Signal)** | Kernel hoặc tiến trình khác phát ra | Thường bất đồng bộ | Signal Handler ở User-space | Bỏ qua, ngắt, tạm dừng hoặc chạy handler |

### Điểm khác biệt quan trọng:
- **Chuyển đổi User/Kernel Mode (Mode Switch)**: Chỉ là việc đổi mức đặc quyền của CPU (từ Ring 3 sang Ring 0 trên x86), **luồng thực thi vẫn là luồng cũ** (chỉ chuyển từ User Stack sang Kernel Stack).
- **Chuyển đổi ngữ cảnh luồng (Context Switch)**: Là việc CPU dừng hẳn luồng A để chuyển sang chạy luồng B. Một System Call như `getpid()` chuyển Mode Switch nhưng **không hề gây ra Context Switch**!

---

## 3. Vòng đời xử lý khi một System Call kết thúc

Sau khi Kernel xử lý xong ngắt/ngoại lệ/system call:
- Đối với **Interrupt**: CPU quay lại đúng vị trí lệnh vừa bị ngắt để tiếp tục chạy (hoặc chuyển sang luồng khác nếu bị chiếm quyền).
- Đối với **System Call (Trap)**: CPU quay lại **lệnh kế tiếp** sau lệnh `syscall`.
- Đối với **Fault (Page Fault)**: Sau khi nạp trang thành công, CPU quay lại **thực thi lại chính chỉ lệnh vừa gây ra lỗi**.
- Đối với **Abort**: Chương trình bị chấm dứt vĩnh viễn.

<!-- @include: @article-footer.snippet.md -->
