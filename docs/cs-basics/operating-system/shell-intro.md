---
title: Tổng hợp kiến thức Lập trình Shell Script cơ bản
description: Cẩm nang hướng dẫn toàn diện về lập trình Shell Script: Cú pháp biến, mảng, toán tử, cấu trúc điều kiện if/case, vòng lặp for/while, định nghĩa hàm, các biến đặc biệt và các thủ thuật tự động hóa triển khai trong thực tế.
category: Cơ sở máy tính
tag:
  - Hệ điều hành
  - Linux
  - Shell
head:
  - - meta
    - name: keywords
      content: Shell, Bash, Shell Script, Lập trình Shell, Biến Shell, Vòng lặp, Câu lệnh điều kiện, Hàm, Tự động hóa Linux
---

Shell Script là công cụ mạnh mẽ và không thể thiếu đối với bất kỳ lập trình viên Backend hay DevOps nào để tự động hóa các tác vụ triển khai (Deployment), sao lưu dữ liệu (Backup), giám sát sức khỏe dịch vụ và xử lý log hệ thống.

Bài viết này tổng hợp toàn diện các kiến thức nền tảng của lập trình Shell (Bash).

---

## 1. File Shell Script đầu tiên: Hello World

Tạo file `hello.sh`:

```bash
#!/bin/bash
# Dòng Shebang (#!) ở đầu chỉ định trình thông dịch Bash sẽ thực thi script này

echo "Hello, JavaGuide!"
```

Cấp quyền thực thi và chạy:
```bash
chmod +x hello.sh
./hello.sh
```

---

## 2. Biến trong Shell (Shell Variables)

### Khai báo và sử dụng biến
- **Quy tắc quan trọng**: **KHÔNG ĐƯỢC CÓ KHOẢNG TRẮNG** quanh dấu bằng `=`.
- Sử dụng `$variable_name` hoặc `${variable_name}` để lấy giá trị.

```bash
name="JavaGuide"
echo "Welcome to ${name}!"

# Biến chỉ đọc (Read-only)
readonly APP_ENV="production"

# Xóa biến
unset name
```

### Các biến đặc biệt trong Shell (Rất hay dùng trong Script):
- `$0`: Tên của file script đang chạy.
- `$1, $2, $3, ...`: Các tham số dòng lệnh truyền vào script (ví dụ `./deploy.sh dev v1.0`).
- `$#`: Tổng số lượng tham số truyền vào.
- `$*` hoặc `$@`: Toàn bộ các tham số truyền vào dưới dạng chuỗi.
- `$?`: **Mã trạng thái trả về của lệnh vừa thực thi trước đó** (`0` = Thành công, khác `0` = Thất bại).
- `$$`: Mã PID của tiến trình script hiện tại.

---

## 3. Cấu trúc rẽ nhánh điều kiện: `if - else` và `case`

### Cú pháp `if - elif - else`:

```bash
#!/bin/bash

read -p "Nhập điểm số (0-100): " score

if [ $score -ge 90 ]; then
    echo "Xếp loại: Xuất sắc"
elif [ $score -ge 70 ]; then
    echo "Xếp loại: Khá"
elif [ $score -ge 50 ]; then
    echo "Xếp loại: Trung bình"
else
    echo "Xếp loại: Yếu"
fi
```

### Các toán tử so sánh quan trọng:
- **So sánh số nguyên**:
  - `-eq` (Equal: Bằng)
  - `-ne` (Not Equal: Khác)
  - `-gt` (Greater Than: Lớn hơn)
  - `-ge` (Greater or Equal: Lớn hơn hoặc bằng)
  - `-lt` (Less Than: Nhỏ hơn)
  - `-le` (Less or Equal: Nhỏ hơn hoặc bằng)
- **Kiểm tra tệp tin**:
  - `-f file`: Kiểm tra file có tồn tại và là file thông thường không.
  - `-d dir`: Kiểm tra thư mục có tồn tại không.
  - `-s file`: Kiểm tra file có tồn tại và dung lượng $> 0$ byte không.
- **So sánh chuỗi**:
  - `[ "$str1" = "$str2" ]`: Hai chuỗi bằng nhau.
  - `[ -z "$str" ]`: Chuỗi rỗng (độ dài bằng 0).
  - `[ -n "$str" ]`: Chuỗi không rỗng.

---

## 4. Vòng lặp: `for` và `while`

### Vòng lặp `for`:

```bash
# Duyệt qua danh sách phần tử
for server in web01 web02 db01; do
    echo "Đang kiểm tra server: ${server}"
done

# Vòng lặp theo phong cách C
for ((i = 1; i <= 5; i++)); do
    echo "Lần lặp thứ: $i"
done
```

### Vòng lặp `while`:

```bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done
```

---

## 5. Định nghĩa Hàm trong Shell (Functions)

```bash
#!/bin/bash

# Định nghĩa hàm
check_service() {
    local service_name=$1
    echo "Đang kiểm tra dịch vụ: ${service_name}..."
    if systemctl is-active --quiet "${service_name}"; then
        echo "${service_name} đang chạy bình thường."
        return 0
    else
        echo "${service_name} đang dừng!"
        return 1
    fi
}

# Gọi hàm
check_service "nginx"
status=$?
echo "Trạng thái trả về: ${status}"
```

---

## 6. Script thực chiến: Tự động khởi động lại ứng dụng Java

```bash
#!/bin/bash

APP_NAME="myapp.jar"
APP_PORT=8080

# Tìm PID cũ của ứng dụng
PID=$(pgrep -f "${APP_NAME}")

if [ -n "$PID" ]; then
    echo "Đang dừng tiến trình ${APP_NAME} (PID: ${PID})..."
    kill -15 "$PID"
    sleep 5
    # Nếu chưa dừng thì ép buộc kill
    if pgrep -f "${APP_NAME}" > /dev/null; then
        kill -9 "$PID"
    fi
    echo "Đã dừng thành công."
fi

echo "Đang khởi động ${APP_NAME}..."
nohup java -Xms512m -Xmx1024m -jar "${APP_NAME}" > app.log 2>&1 &

sleep 3
NEW_PID=$(pgrep -f "${APP_NAME}")
if [ -n "$NEW_PID" ]; then
    echo "${APP_NAME} đã khởi động thành công với PID: ${NEW_PID}!"
else
    echo "Khởi động thất bại, vui lòng kiểm tra file app.log!"
fi
```

<!-- @include: @article-footer.snippet.md -->
