---
title: SQL常见面试题总结（2）
description: SQL常见面试题总结第二篇，详解INSERT、UPDATE、DELETE等DML数据操作语句，包括批量插入、从其他表导入、带更新的插入等实战技巧。
category: 数据库
tag:
  - 数据库基础
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL面试题,INSERT插入,UPDATE更新,DELETE删除,批量插入,REPLACE INTO,数据操作
---

> Các câu hỏi từ: [Niuke Tiba - Thử thách SQL nâng cao](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

## Thao tác Thêm, Xóa, Sửa (DML)

Tổng hợp các cách INSERT bản ghi trong SQL:

- **INSERT thông thường (tất cả field)** : `INSERT INTO table_name VALUES (value1, value2, ...)`
- **INSERT thông thường (chỉ định field)** : `INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...)`
- **INSERT nhiều dòng một lúc** : `INSERT INTO table_name (column1, column2, ...) VALUES (value1_1, value1_2, ...), (value2_1, value2_2, ...), ...`
- **Import từ một bảng khác** : `INSERT INTO table_name SELECT * FROM table_name2 [WHERE key=value]`
- **INSERT kèm cập nhật (Replace)** : `REPLACE INTO table_name VALUES (value1, value2, ...)` (Lưu ý nguyên lý của cách này là khi phát hiện trùng Primary Key hoặc Unique Index Key thì sẽ xóa bản ghi cũ rồi mới INSERT bản ghi mới)

### INSERT bản ghi (Phần 1)

**Mô tả**: Hệ thống Niuke sẽ ghi lại lịch sử làm bài thi của từng người dùng vào bảng `exam_record`, hiện tại chi tiết lịch sử làm bài của 2 người dùng như sau:

- Người dùng 1001 bắt đầu làm đề thi 9001 vào lúc 22:11:12 ngày 01/09/2021, nộp bài sau 50 phút và đạt 90 điểm;
- Người dùng 1002 bắt đầu làm đề thi 9002 vào lúc 07:01:02 ngày 04/09/2021, và thoát khỏi nền tảng sau 10 phút.

Bảng lịch sử làm bài thi `exam_record` đã được tạo sẵn có cấu trúc như bên dưới, hãy dùng một câu lệnh duy nhất để INSERT hai bản ghi này vào bảng.

| Filed       | Type       | Null | Key | Extra          | Default | Comment  |
| ----------- | ---------- | ---- | --- | -------------- | ------- | -------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng  |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng  |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID đề thi  |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm số     |

**Đáp án**:

```sql
// Tồn tại Primary Key tự tăng, không cần gán giá trị thủ công
INSERT INTO exam_record (uid, exam_id, start_time, submit_time, score) VALUES
(1001, 9001, '2021-09-01 22:11:12', '2021-09-01 23:01:12', 90),
(1002, 9002, '2021-09-04 07:01:02', NULL, NULL);
```

### INSERT bản ghi (Phần 2)

**Mô tả**: Hiện có một bảng lịch sử làm bài thi `exam_record`, cấu trúc như bảng dưới, chứa lịch sử làm bài thi của người dùng trong nhiều năm. Do dữ liệu ngày càng nhiều, độ khó bảo trì ngày càng lớn, cần tinh giản nội dung bảng dữ liệu và backup dữ liệu lịch sử.

Bảng `exam_record`:

| Filed       | Type       | Null | Key | Extra          | Default | Comment  |
| ----------- | ---------- | ---- | --- | -------------- | ------- | -------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng  |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng  |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID đề thi  |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm số     |

Chúng ta đã tạo một bảng mới `exam_record_before_2021` dùng để backup lịch sử làm bài thi trước năm 2021, cấu trúc đồng nhất với bảng `exam_record`. Hãy import các bản ghi làm bài thi đã hoàn thành trước năm 2021 vào bảng này.

**Đáp án**:

```sql
INSERT INTO exam_record_before_2021 (uid, exam_id, start_time, submit_time, score)
SELECT uid,exam_id,start_time,submit_time,score
FROM exam_record
WHERE YEAR(submit_time) < 2021;
```

### INSERT bản ghi (Phần 3)

**Mô tả**: Hiện có một bộ đề thi SQL độ khó cao có ID là 9003, thời lượng là 1 tiếng rưỡi. Hãy INSERT thời gian phát hành '2021-01-01 00:00:00' vào bảng thông tin đề thi `examination_info`. Bất kể đề thi có ID đó đã tồn tại hay chưa, đều phải INSERT thành công, hãy thử INSERT nó.

Bảng thông tin đề thi `examination_info`:

| Filed        | Type        | Null | Key | Extra          | Default | Comment      |
| ------------ | ----------- | ---- | --- | -------------- | ------- | ------------ |
| id           | int(11)     | NO   | PRI | auto_increment | (NULL)  | ID tự tăng      |
| exam_id      | int(11)     | NO   | UNI |                | (NULL)  | ID đề thi      |
| tag          | varchar(32) | YES  |     |                | (NULL)  | Tag thể loại     |
| difficulty   | varchar(8)  | YES  |     |                | (NULL)  | Độ khó         |
| duration     | int(11)     | NO   |     |                | (NULL)  | Thời lượng (số phút) |
| release_time | datetime    | YES  |     |                | (NULL)  | Thời gian phát hành     |

**Đáp án**:

```sql
REPLACE INTO examination_info VALUES
 (NULL, 9003, "SQL", "hard", 90, "2021-01-01 00:00:00");
```

### UPDATE bản ghi (Phần 1)

**Mô tả**: Hiện có một bảng thông tin đề thi `examination_info`, cấu trúc bảng như hình dưới:

| Filed        | Type     | Null | Key | Extra          | Default | Comment  |
| ------------ | -------- | ---- | --- | -------------- | ------- | -------- |
| id           | int(11)  | NO   | PRI | auto_increment | (NULL)  | ID tự tăng  |
| exam_id      | int(11)  | NO   | UNI |                | (NULL)  | ID đề thi  |
| tag          | char(32) | YES  |     |                | (NULL)  | Tag thể loại |
| difficulty   | char(8)  | YES  |     |                | (NULL)  | Độ khó     |
| duration     | int(11)  | NO   |     |                | (NULL)  | Thời lượng     |
| release_time | datetime | YES  |     |                | (NULL)  | Thời gian phát hành |

Hãy sửa tất cả field `tag` có giá trị `PYTHON` thành `Python` trong bảng **examination_info**.

**Tư duy giải đề**: Bài này có 2 hướng tư duy giải: dễ nghĩ đến nhất là dùng trực tiếp `UPDATE + WHERE` để chỉ định điều kiện cập nhật; hướng thứ 2 là tìm kiếm và thay thế (replace) dựa theo field cần sửa.

**Đáp án 1**:

```sql
UPDATE examination_info SET tag = 'Python' WHERE tag='PYTHON'
```

**Đáp án 2**:

```sql
UPDATE examination_info
SET tag = REPLACE(tag,'PYTHON','Python')

# REPLACE(target_field, "search_string", "replacement_string")
```

### UPDATE bản ghi (Phần 2)

**Mô tả**: Hiện có một bảng lịch sử làm bài thi exam_record, chứa lịch sử làm bài thi của người dùng trong nhiều năm. Cấu trúc như bảng dưới. Bảng lịch sử làm bài `exam_record`: **`submit_time`** là thời gian hoàn thành (chú ý câu này).

| Filed       | Type       | Null | Key | Extra          | Default | Comment  |
| ----------- | ---------- | ---- | --- | -------------- | ------- | -------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng  |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng  |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID đề thi  |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm số     |

**Yêu cầu đề bài**: Hãy sửa tất cả các bản ghi ==chưa hoàn thành== bắt đầu làm bài ==trước== ngày 01/09/2021 trong bảng `exam_record` thành hoàn thành thụ động, tức là: sửa thời gian hoàn thành thành '2099-01-01 00:00:00' và điểm số thành 0.

**Tư duy giải đề**: Chú ý từ khóa trong đề bài (đã highlight) điều kiện trước `" thời gian xxx "`, lúc này ngay lập tức nghĩ đến việc so sánh thời gian: có thể dùng trực tiếp `xxx_time < "2021-09-01 00:00:00"`, hoặc dùng hàm `DATE()` để so sánh; Điều kiện thứ 2 là `"chưa hoàn thành"`, tức thời gian hoàn thành là NULL, hay chính là thời gian nộp bài trong đề bài ----- `submit_time IS NULL`.

**Đáp án**:

```sql
UPDATE exam_record SET submit_time = '2099-01-01 00:00:00', score = 0 WHERE DATE(start_time) < "2021-09-01" AND submit_time IS null
```

### DELETE bản ghi (Phần 1)

**Mô tả**: Hiện có một bảng lịch sử làm bài thi `exam_record`, chứa lịch sử làm bài thi của người dùng trong nhiều năm, cấu trúc như bảng dưới:

Bảng lịch sử làm bài `exam_record`: **`start_time`** là thời gian bắt đầu làm bài, `submit_time` là nộp bài, tức thời gian kết thúc.

| Filed       | Type       | Null | Key | Extra          | Default | Comment  |
| ----------- | ---------- | ---- | --- | -------------- | ------- | -------- |
| id          | int(11)    | NO   | PRI | auto_increment | (NULL)  | ID tự tăng  |
| uid         | int(11)    | NO   |     |                | (NULL)  | ID người dùng  |
| exam_id     | int(11)    | NO   |     |                | (NULL)  | ID đề thi  |
| start_time  | datetime   | NO   |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm số     |

**Yêu cầu**: Hãy xóa các bản ghi có thời gian làm bài dưới 5 phút và điểm không đạt (điểm đạt là 60 điểm) trong bảng `exam_record`;

**Tư duy giải đề**: Bài này tuy là luyện tập xóa (DELETE), nhưng xem kỹ lại là kiểm tra cách dùng hàm thời gian. Việc so sánh số phút ở đây, các hàm thường dùng có **`TIMEDIFF`** và **`TIMESTAMPDIFF`**, cách dùng của cả hai hơi khác nhau một chút, cái sau linh hoạt hơn, tùy thuộc vào thói quen cá nhân.

1. `TIMEDIFF`: Chênh lệch giữa 2 thời gian

```sql
TIMEDIFF(time1, time2)
```

Cả 2 tham số đều là bắt buộc, đều là một biểu thức time hoặc datetime. Nếu tham số chỉ định không hợp lệ hoặc là NULL, hàm sẽ trả về NULL.

Đối với bài này, có thể dùng bên trong hàm MINUTE(), vì TIMEDIFF tính ra chênh lệch thời gian, bọc một hàm MINUTE() ở ngoài sẽ tính ra số phút.

2. `TIMESTAMPDIFF`: Dùng để tính chênh lệch thời gian giữa 2 ngày/giờ

```sql
TIMESTAMPDIFF(unit,datetime_expr1,datetime_expr2)
# Giải thích tham số
# unit: Đơn vị chênh lệch thời gian trả về khi so sánh date, các giá trị thường dùng như sau:
SECOND：Giây
MINUTE：Phút
HOUR：Giờ
DAY：Ngày
WEEK：Tuần
MONTH：Tháng
QUARTER：Quý
YEAR：Năm
# Hàm TIMESTAMPDIFF trả về kết quả datetime_expr2 - datetime_expr1 (nói một cách dễ hiểu: Cái sau - Cái trước, tức 2-1), trong đó datetime_expr1 và datetime_expr2 có thể là giá trị kiểu DATE hoặc DATETIME (nói một cách dễ hiểu: có thể là "2023-01-01", cũng có thể là "2023-01-01 00:00:00")
```

Bài này cần so sánh số phút, vậy chính là TIMESTAMPDIFF(MINUTE, start_time, submit_time) < 5

**Đáp án**:

```sql
DELETE FROM exam_record WHERE MINUTE (TIMEDIFF(submit_time , start_time)) < 5 AND score < 60
```

```sql
DELETE FROM exam_record WHERE TIMESTAMPDIFF(MINUTE, start_time, submit_time) < 5 AND score < 60
```

### DELETE bản ghi (Phần 2)

**Mô tả**: Hiện có một bảng lịch sử làm bài thi `exam_record`, trong đó chứa lịch sử làm bài thi của người dùng qua nhiều năm, cấu trúc như bảng dưới:

Bảng lịch sử làm bài `exam_record`: `start_time` là thời gian bắt đầu làm bài, `submit_time` là thời gian nộp bài (thời gian kết thúc), nếu chưa hoàn thành thì rỗng (NULL).

| Filed       | Type       | Null | Key | Extra          | Default | Comment  |
| ----------- | ---------- | :--: | --- | -------------- | ------- | -------- |
| id          | int(11)    |  NO  | PRI | auto_increment | (NULL)  | ID tự tăng  |
| uid         | int(11)    |  NO  |     |                | (NULL)  | ID người dùng  |
| exam_id     | int(11)    |  NO  |     |                | (NULL)  | ID đề thi  |
| start_time  | datetime   |  NO  |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm số     |

**Yêu cầu**: Hãy xóa 3 bản ghi có thời gian bắt đầu làm bài sớm nhất trong số các bản ghi chưa hoàn thành bài làm ==hoặc== thời gian làm bài dưới 5 phút trong bảng `exam_record`.

**Tư duy giải đề**: Bài này tương đối đơn giản, nhưng cần chú ý thông tin đề bài đưa ra: thời gian kết thúc nếu chưa hoàn thành thì rỗng (NULL), đây thực chất là một điều kiện.

Một điều kiện nữa là dưới 5 phút, tương tự bài trước, nhưng ở đây là **HOẶC (OR)**, tức chỉ cần thỏa mãn 1 trong 2 điều kiện; Ngoài ra còn kiểm tra một chút về cách dùng ORDER BY và LIMIT.

**Đáp án**:

```sql
DELETE FROM exam_record WHERE submit_time IS null OR TIMESTAMPDIFF(MINUTE, start_time, submit_time) < 5
ORDER BY start_time
LIMIT 3
# Mặc định là ASC, DESC là sắp xếp giảm dần
```

### DELETE bản ghi (Phần 3)

**Mô tả**: Hiện có một bảng lịch sử làm bài thi exam_record, chứa lịch sử làm bài thi của người dùng trong nhiều năm, cấu trúc như bảng dưới:

| Filed       | Type       | Null | Key | Extra          | Default | Comment  |
| ----------- | ---------- | :--: | --- | -------------- | ------- | -------- |
| id          | int(11)    |  NO  | PRI | auto_increment | (NULL)  | ID tự tăng  |
| uid         | int(11)    |  NO  |     |                | (NULL)  | ID người dùng  |
| exam_id     | int(11)    |  NO  |     |                | (NULL)  | ID đề thi  |
| start_time  | datetime   |  NO  |     |                | (NULL)  | Thời gian bắt đầu |
| submit_time | datetime   | YES  |     |                | (NULL)  | Thời gian nộp bài |
| score       | tinyint(4) | YES  |     |                | (NULL)  | Điểm số     |

**Yêu cầu**: Hãy xóa tất cả các bản ghi trong bảng `exam_record`, ==đồng thời reset Primary Key tự tăng==

**Tư duy giải đề**: Bài này kiểm tra sự khác nhau giữa 3 câu lệnh xóa, chú ý phần highlight yêu cầu reset Primary Key;

- `DROP`: Xóa bảng, xóa cấu trúc bảng, không thể đảo ngược
- `TRUNCATE`: Format bảng, không xóa cấu trúc bảng, không thể đảo ngược
- `DELETE`: Xóa dữ liệu, có thể đảo ngược (Rollback trong Transaction)

Ở đây chọn `TRUNCATE` là vì: TRUNCATE chỉ tác động lên bảng; `TRUNCATE` sẽ xóa toàn bộ các dòng trong bảng, nhưng cấu trúc bảng cùng các Constraint, Index, v.v. được giữ nguyên; `TRUNCATE` sẽ reset giá trị tự tăng của bảng; Sử dụng `TRUNCATE` sẽ làm dung lượng bảng và Index chiếm dụng khôi phục về kích thước ban đầu.

Bài này cũng có thể dùng `DELETE` để làm, nhưng sau khi xóa còn cần phải `ALTER` thủ công cấu trúc bảng để đặt giá trị ban đầu cho Primary Key;

Tương tự cũng có thể dùng `DROP` để làm, xóa trực tiếp toàn bộ bảng bao gồm cấu trúc bảng, sau đó tạo lại bảng mới là được.

**Đáp án**:

```sql
TRUNCATE  exam_record;
```

## Thao tác Bảng và Index

### Tạo một bảng mới

**Mô tả**: Hiện có một bảng thông tin người dùng chứa thông tin người dùng đã đăng ký trên nền tảng nhiều năm qua. Cùng với sự lớn mạnh không ngừng của nền tảng Niuke, lượng người dùng tăng trưởng thần tốc, để phục vụ hiệu quả cho những người dùng có độ hoạt động cao, hiện cần tách một phần người dùng ra một bảng mới.

Bảng thông tin người dùng ban đầu:

| Filed         | Type        | Null | Key | Default           | Extra          | Comment  |
| ------------- | ----------- | ---- | --- | ----------------- | -------------- | -------- |
| id            | int(11)     | NO   | PRI | (NULL)            | auto_increment | ID tự tăng  |
| uid           | int(11)     | NO   | UNI | (NULL)            |                | ID người dùng  |
| nick_name     | varchar(64) | YES  |     | (NULL)            |                | Biệt danh     |
| achievement   | int(11)     | YES  |     | 0                 |                | Điểm thành tựu   |
| level         | int(11)     | YES  |     | (NULL)            |                | Cấp độ người dùng |
| job           | varchar(32) | YES  |     | (NULL)            |                | Định hướng nghề nghiệp |
| register_time | datetime    | YES  |     | CURRENT_TIMESTAMP |                | Thời gian đăng ký |

Với tư cách là Data Analyst, hãy **tạo một bảng thông tin người dùng chất lượng cao user_info_vip**, cấu trúc bảng đồng nhất với bảng thông tin người dùng.

Kết quả bạn nên trả về như bảng dưới đây, hãy viết câu lệnh tạo bảng đưa tất cả giới hạn và mô tả trong bảng vào.

| Filed         | Type        | Null | Key | Default           | Extra          | Comment  |
| ------------- | ----------- | ---- | --- | ----------------- | -------------- | -------- |
| id            | int(11)     | NO   | PRI | (NULL)            | auto_increment | ID tự tăng  |
| uid           | int(11)     | NO   | UNI | (NULL)            |                | ID người dùng  |
| nick_name     | varchar(64) | YES  |     | (NULL)            |                | Biệt danh     |
| achievement   | int(11)     | YES  |     | 0                 |                | Điểm thành tựu   |
| level         | int(11)     | YES  |     | (NULL)            |                | Cấp độ người dùng |
| job           | varchar(32) | YES  |     | (NULL)            |                | Định hướng nghề nghiệp |
| register_time | datetime    | YES  |     | CURRENT_TIMESTAMP |                | Thời gian đăng ký |

**Tư duy giải đề**: Nếu bài này đưa ra tên bảng cũ, có thể dùng trực tiếp `CREATE TABLE table_new AS SELECT * FROM table_old;`. Nhưng bài này không đưa ra tên bảng cũ, do đó cần tự mình tạo mới, chú ý giá trị mặc định (DEFAULT) và việc tạo Key là được, tương đối đơn giản. (Lưu ý: Nếu thực thi trên Niuke, chú ý trong COMMENT phải giữ nguyên như COMMENT trong đề bài bao gồm cả chữ hoa/thường, nếu không sẽ không pass, và Character Set cũng cần cài đặt).

Đáp án:

```sql
CREATE TABLE IF NOT EXISTS user_info_vip(
    id INT(11) PRIMARY KEY AUTO_INCREMENT COMMENT'自增ID',
    uid INT(11) UNIQUE NOT NULL COMMENT '用户ID',
    nick_name VARCHAR(64) COMMENT'昵称',
    achievement INT(11) DEFAULT 0 COMMENT '成就值',
    `level` INT(11) COMMENT '用户等级',
    job VARCHAR(32) COMMENT '职业方向',
    register_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间'
)CHARACTER SET UTF8
```

### Sửa đổi bảng (ALTER TABLE)

**Mô tả**: Hiện có một bảng thông tin người dùng `user_info`, chứa thông tin người dùng đã đăng ký trên nền tảng qua nhiều năm.

**Bảng thông tin người dùng `user_info`:**

| Filed         | Type        | Null | Key | Default           | Extra          | Comment  |
| ------------- | ----------- | ---- | --- | ----------------- | -------------- | -------- |
| id            | int(11)     | NO   | PRI | (NULL)            | auto_increment | ID tự tăng  |
| uid           | int(11)     | NO   | UNI | (NULL)            |                | ID người dùng  |
| nick_name     | varchar(64) | YES  |     | (NULL)            |                | Biệt danh     |
| achievement   | int(11)     | YES  |     | 0                 |                | Điểm thành tựu   |
| level         | int(11)     | YES  |     | (NULL)            |                | Cấp độ người dùng |
| job           | varchar(32) | YES  |     | (NULL)            |                | Định hướng nghề nghiệp |
| register_time | datetime    | YES  |     | CURRENT_TIMESTAMP |                | Thời gian đăng ký |

**Yêu cầu**: Trong bảng thông tin người dùng, hãy thêm một column `school` có thể lưu tối đa 15 chữ Hán vào phía sau field `level`; Đồng thời đổi tên column `job` trong bảng thành `profession`, độ dài field `VARCHAR` thành 10; Đặt giá trị mặc định của `achievement` thành 0.

**Tư duy giải đề**: Đầu tiên trước khi làm bài này, cần nắm vững cách dùng cơ bản của câu lệnh ALTER:

- Thêm một column: `ALTER TABLE table_name ADD COLUMN column_name type [FIRST | AFTER field_name];` (FIRST: Thêm vào trước column nào đó, AFTER: Thêm vào sau)
- Sửa kiểu hoặc Constraint của column: `ALTER TABLE table_name MODIFY COLUMN column_name new_type [new_constraint];`
- Đổi tên column: `ALTER TABLE table_name CHANGE COLUMN old_column_name new_column_name type;`
- Xóa column: `ALTER TABLE table_name DROP COLUMN column_name;`
- Đổi tên bảng: `ALTER TABLE table_name RENAME [TO] new_table_name;`
- Chuyển một column thành column đầu tiên: `ALTER TABLE table_name MODIFY COLUMN column_name type FIRST;`

Từ khóa `COLUMN` thực chất có thể bỏ qua không viết, ở đây liệt kê ra dựa theo quy chuẩn.

Khi sửa đổi, nếu có nhiều mục cần sửa, có thể viết chung lại với nhau nhưng chú ý định dạng.

**Đáp án**:

```sql
ALTER TABLE user_info
    ADD school VARCHAR(15) AFTER level,
    CHANGE job profession VARCHAR(10),
    MODIFY achievement INT(11) DEFAULT 0;
```

### Xóa bảng (DROP TABLE)

**Mô tả**: Hiện có một bảng lịch sử làm bài thi `exam_record`, chứa lịch sử làm bài thi của người dùng qua nhiều năm. Thông thường mỗi năm đều sẽ tạo cho bảng `exam_record` một bảng backup `exam_record_{YEAR}`, với `{YEAR}` là năm tương ứng.

Hiện tại dữ liệu ngày càng nhiều, bộ nhớ báo động, hãy xóa tất cả các bảng backup lâu đời (từ năm 2011 đến 2014) (nếu tồn tại).

**Tư duy giải đề**: Bài này rất đơn giản, xóa trực tiếp là được. Nếu ngại phiền phức, có thể dùng dấu phẩy ngăn cách các bảng cần xóa, viết trên một dòng. Ở đây chắc chắn sẽ có bạn hỏi: Nếu muốn xóa rất nhiều bảng thì sao? Yên tâm, nếu muốn xóa rất nhiều bảng, có thể viết script để xóa.

**Đáp án**:

```sql
DROP TABLE IF EXISTS exam_record_2011;
DROP TABLE IF EXISTS exam_record_2012;
DROP TABLE IF EXISTS exam_record_2013;
DROP TABLE IF EXISTS exam_record_2014;
```

### Tạo Index

**Mô tả**: Hiện có một bảng thông tin đề thi `examination_info`, chứa thông tin các loại đề thi. Để truy vấn bảng tiện lợi và nhanh chóng hơn, cần tạo các Index sau trên bảng `examination_info`:

Quy tắc như sau: Tạo Normal Index `idx_duration` trên column `duration`, tạo Unique Index `uniq_idx_exam_id` trên column `exam_id`, tạo Fulltext Index `full_idx_tag` trên column `tag`.

Dựa theo ý đề bài, sẽ trả về kết quả như sau:

| examination_info | 0   | PRIMARY          | 1   | id       | A   | 0   |     |     |     | BTREE    |
| ---------------- | --- | ---------------- | --- | -------- | --- | --- | --- | --- | --- | -------- |
| examination_info | 0   | uniq_idx_exam_id | 1   | exam_id  | A   | 0   |     |     | YES | BTREE    |
| examination_info | 1   | idx_duration     | 1   | duration | A   | 0   |     |     |     | BTREE    |
| examination_info | 1   | full_idx_tag     | 1   | tag      |     | 0   |     |     | YES | FULLTEXT |

Ghi chú: Backend sẽ dùng câu lệnh `SHOW INDEX FROM examination_info` để so sánh kết quả đầu ra

**Tư duy giải đề**: Làm bài này đầu tiên cần hiểu các loại Index phổ biến:

- B-Tree Index: B-Tree Index (hoặc gọi là cây cân bằng) là loại Index phổ biến nhất và là mặc định. Nó áp dụng cho các điều kiện truy vấn khác nhau, có thể nhanh chóng định vị đến dữ liệu thỏa mãn điều kiện. B-Tree Index áp dụng cho các thao tác tìm kiếm thông thường, hỗ trợ Equal Query, Range Query và Sort.
- Unique Index: Unique Index tương tự Normal B-Tree Index, điểm khác biệt là nó yêu cầu giá trị của column được tạo Index phải là duy nhất. Điều này có nghĩa là khi INSERT hoặc UPDATE dữ liệu, MySQL sẽ kiểm tra tính duy nhất của column Index.
- Primary Key Index: Primary Key Index là một loại Unique Index đặc biệt, dùng để định danh duy nhất cho từng dòng dữ liệu trong bảng. Mỗi bảng chỉ có thể có 1 Primary Key Index, giúp nâng cao tốc độ truy cập dữ liệu và tính toàn vẹn của dữ liệu.
- Fulltext Index: Fulltext Index dùng để tìm kiếm toàn văn (full-text search) trong dữ liệu văn bản. Nó hỗ trợ tìm kiếm từ khóa trong text field, chứ không chỉ là Equal hay Range Query đơn giản. Fulltext Index áp dụng cho các kịch bản cần thực hiện full-text search.

```sql
-- Ví dụ:
-- Thêm B-Tree Index:
	CREATE INDEX idx_name ON table_name (field_name);   -- idx_name là tên Index, phía dưới cũng vậy
-- Tạo Unique Index:
	CREATE UNIQUE INDEX idx_name ON table_name (field_name);
-- Tạo một Primary Key Index:
	ALTER TABLE table_name ADD PRIMARY KEY (field_name);
-- Tạo một Fulltext Index:
	ALTER TABLE table_name ADD FULLTEXT INDEX idx_name (field_name);

-- Qua các ví dụ trên, có thể thấy cả CREATE và ALTER đều có thể thêm Index
```

Sau khi có kiến thức nền tảng trên, đáp án bài này đã lộ diện.

**Đáp án**:

```sql
ALTER TABLE examination_info
    ADD INDEX idx_duration(duration),
    ADD UNIQUE INDEX uniq_idx_exam_id(exam_id),
    ADD FULLTEXT INDEX full_idx_tag(tag);
```

### Xóa Index

**Mô tả**: Hãy xóa Unique Index uniq_idx_exam_id và Fulltext Index full_idx_tag trên bảng `examination_info`.

**Tư duy giải đề**: Bài này kiểm tra cú pháp cơ bản để xóa Index:

```sql
-- Sử dụng DROP INDEX để xóa Index
DROP INDEX idx_name ON table_name;

-- Sử dụng ALTER TABLE để xóa Index
ALTER TABLE employees DROP INDEX idx_email;
```

Ở đây cần lưu ý: Trong MySQL, thao tác xóa nhiều Index cùng lúc không được hỗ trợ. Mỗi lần xóa Index chỉ được chỉ định 1 tên Index để xóa.

Đồng thời lệnh **DROP** cần phải thận trọng khi dùng!!!

**Đáp án**:

```sql
DROP INDEX uniq_idx_exam_id ON examination_info;
DROP INDEX full_idx_tag ON examination_info;
```

<!-- @include: @article-footer.snippet.md -->
