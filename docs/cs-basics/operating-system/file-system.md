---
title: Chi tiết Hệ thống tệp tin (File System): Inode, VFS, Page Cache và Cơ chế ghi nhật ký (Journaling)
description: Phân tích toàn diện kiến trúc Hệ thống tệp tin trong Linux: File Descriptor, Inode, Dentry, Virtual File System (VFS), Page Cache, Hard Link vs Soft Link, cơ chế fsync và phục hồi an toàn bằng Journaling.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
  - File System
head:
  - - meta
    - name: keywords
      content: File System, Hệ thống tệp tin, Inode, Dentry, File Descriptor, VFS, Page Cache, Hard Link, Soft Link, fsync, Journaling, ext4
---

Khi chúng ta mở một file trong code bằng `open("/var/log/app.log")`:

Hệ điều hành làm thế nào để tìm được vị trí các khối dữ liệu thực tế trên ổ đĩa từ một đường dẫn văn bản? Tại sao xóa một file đang được tiến trình khác mở thì dung lượng đĩa vẫn chưa được giải phóng? Hard Link và Soft Link khác nhau như thế nào? Tại sao cơ chế ghi đệm **Page Cache** giúp tăng tốc độ đọc/ghi file lên hàng nghìn lần nhưng lại tiềm ẩn nguy cơ mất dữ liệu khi mất điện đột ngột?

Bài viết này sẽ giải thích toàn bộ kiến trúc bên dưới của File System trong Linux.

---

## 1. Cấu trúc cốt lõi: Inode, Dentry và Data Block

Trong hệ thống tệp tin Linux (như ext4, XFS):

![Cấu trúc Inode và Dentry trong Linux File System](https://oss.javaguide.cn/github/javaguide/cs-basics/operating-system/inode-dentry-structure.png)

1. **Inode (Index Node - Nút chỉ mục)**:
   - Đại diện cho một **tệp tin hoặc thư mục thực tế**.
   - Lưu trữ toàn bộ **Metadata (Siêu dữ liệu)** của tệp: Dung lượng file, quyền truy cập (`chmod`), User/Group ID, mốc thời gian (atime, mtime, ctime), số lượng liên kết cứng (`link count`), và danh sách con trỏ trỏ tới các khối dữ liệu thực tế (**Data Blocks**) trên đĩa.
   - ⚠️ **Lưu ý**: Inode **KHÔNG lưu tên tệp tin**!
2. **Dentry (Directory Entry - Mục thư mục)**:
   - Lưu trữ mối quan hệ ánh xạ giữa **Tên tệp (Filename)** và **Số hiệu Inode (Inode Number)**.
   - Thư mục trong Linux thực chất là một tệp tin đặc biệt, bên trong chứa danh sách các Dentry (Tên file -> Inode).
3. **Data Block (Khối dữ liệu)**: Khối lưu trữ trên ổ đĩa vật lý chứa nội dung nhị phân thực tế của tệp tin.

---

## 2. Phân biệt Hard Link (Liên kết cứng) và Soft Link (Liên kết mềm / Symbolic Link)

| Tiêu chí | Hard Link (Liên kết cứng) | Soft Link (Symbolic Link / Liên kết mềm) |
| --- | --- | --- |
| **Bản chất** | Một Dentry mới trỏ tới **cùng một số hiệu Inode gốc** | Một tệp tin hoàn toàn mới với **Inode riêng**, Data Block chứa đường dẫn tới tệp gốc |
| **Số hiệu Inode** | **Giống hệt Inode của tệp gốc** | **Có Inode riêng biệt** |
| **Ảnh hưởng khi xóa tệp gốc** | Tệp vẫn tồn tại bình thường, chỉ giảm `link count` đi 1 | Liên kết bị gãy (Dangling / Broken Link), không thể mở được nữa |
| **Phạm vi áp dụng** | Không thể liên kết thư mục, **không thể liên kết xuyên qua các phân vùng đĩa khác nhau** | Có thể liên kết thư mục, **hoạt động xuyên phân vùng đĩa bình thường** |
| **Lệnh tạo** | `ln file_goc hard_link` | `ln -s file_goc soft_link` |

---

## 3. VFS (Virtual File System - Hệ thống tệp tin ảo)

Linux hỗ trợ hàng chục loại File System khác nhau (ext4, XFS, Btrfs, NTFS, FAT32, NFS, procfs, sysfs).

Nhờ có tầng trừu tượng **VFS (Virtual File System)**:
- Tầng ứng dụng chỉ cần sử dụng các hàm chuẩn POSIX thống nhất (`open()`, `read()`, `write()`, `close()`).
- VFS cung cấp một giao diện API trừu tượng gồm 4 đối tượng chính: `superblock_operations`, `inode_operations`, `dentry_operations`, `file_operations`, và tự động chuyển tiếp lời gọi xuống Driver của hệ thống tệp tin tương ứng bên dưới.

---

## 4. Page Cache và Cơ chế đồng bộ dữ liệu (`fsync`)

Khi ứng dụng gọi hàm `write(fd, buf, count)`:
- Mặc định, dữ liệu **chưa hề được ghi ngay xuống ổ đĩa vật lý**!
- Dữ liệu được ghi vào **Page Cache (Bộ đệm trang trong RAM)** của Kernel, trang này được đánh dấu là **Dirty Page (Trang bẩn)**.
- Hàm `write()` lập tức trả về thành công với tốc độ bộ nhớ RAM (cực nhanh).
- Kernel định kỳ có luồng nền `flusher` quét các Dirty Page và ghi dồn dập xuống đĩa (Flushing / Writeback).

### Rủi ro mất dữ liệu và vai trò của `fsync()`:
- Nếu mất nguồn điện đột ngột trước khi Kernel kịp ghi Dirty Page xuống đĩa, dữ liệu trong Page Cache sẽ bị mất hoàn toàn!
- Các hệ quản trị cơ sở dữ liệu (MySQL Redo Log, Redis AOF) bắt buộc phải gọi **`fsync(fd)`** sau khi ghi log quan trọng để ép buộc Kernel xả toàn bộ dữ liệu từ Page Cache xuống đĩa vật lý trước khi trả về cho ứng dụng.

---

## 5. Cơ chế ghi nhật ký (Journaling File System)

Trước đây khi máy tính bị mất điện đột ngột, toàn bộ hệ thống tệp tin bị hỏng cấu trúc và phải chạy lệnh `fsck` quét từng block đĩa hàng tiếng đồng hồ để sửa lỗi.

Các hệ thống tệp tin hiện đại (ext4, XFS) áp dụng cơ chế **Journaling (Ghi nhật ký giao dịch)**:
- Trước khi thực hiện thay đổi cấu trúc Metadata, hệ thống tệp tin ghi một bản tóm tắt giao dịch vào vùng **Journal** chuyên biệt trên đĩa.
- Sau khi Journal ghi thành công, mới tiến hành cập nhật Metadata thực tế.
- Khi mất điện đột ngột khởi động lại: Kernel chỉ cần đọc lại vùng Journal để hoàn tất hoặc rollback các giao dịch đang dở trong vài giây, bảo vệ hệ thống tệp tin không bao giờ bị hỏng cấu trúc.

<!-- @include: @article-footer.snippet.md -->
