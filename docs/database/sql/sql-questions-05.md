---
title: SQL常见面试题总结（5）
description: SQL常见面试题总结第五篇，详解NULL空值处理技巧，包括IFNULL、COALESCE函数，以及使用CASE WHEN进行条件统计和完成率计算。
category: 数据库
tag:
  - 数据库基础
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL面试题,NULL空值处理,IFNULL,COALESCE,CASE WHEN,条件统计,完成率计算
---

> Các câu hỏi từ: [Niuke Tiba - Thử thách SQL nâng cao](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

Các câu hỏi thuộc mức Độ khó trung bình - cao hoặc Khó có thể căn cứ vào tình hình thực tế và nhu cầu phỏng vấn của bản thân để quyết định có nên bỏ qua hay không.

## Xử lý giá trị NULL

### Thống kê số bài thi chưa hoàn thành和tỉ lệ chưa hoàn thành của các đề thi có trạng thái chưa hoàn thành

**Mô tả**:

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số), dữ liệu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | 2021-05-02 10:30:01 | 81     |
| 3   | 1001 | 9001    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |

Hãy thống kê số bài chưa hoàn thành `incomplete_cnt` và tỉ lệ chưa hoàn thành `incomplete_rate` của các đề thi có trạng thái chưa hoàn thành. Kết quả mẫu như sau:

| exam_id | incomplete_cnt | complete_rate |
| ------- | -------------- | ------------- |
| 9001    | 1              | 0.333         |

Giải thích: Đề thi 9001 có 3 bản ghi lịch sử làm bài, trong đó 2 lần hoàn thành, 1 lần chưa hoàn thành, do đó số bài chưa hoàn thành là 1, tỉ lệ chưa hoàn thành là 0.333 (làm tròn 3 chữ số thập phân).

**Tư duy giải đề**:

Bài này chỉ cần chú ý một bên là có giới hạn điều kiện, một bên là không có giới hạn điều kiện; Hoặc là truy vấn từng điều kiện riêng rồi gom lại; Hoặc là trực tiếp thực hiện kiểm tra điều kiện bên trong SELECT.

**Đáp án**:

Cách viết 1:

```sql
SELECT
    exam_id,
    (COUNT(*) - COUNT(submit_time)) AS incomplete_cnt,
    ROUND((COUNT(*) - COUNT(submit_time)) / COUNT(*), 3) AS incomplete_rate
FROM
    exam_record
GROUP BY
    exam_id
HAVING
    (COUNT(*) - COUNT(submit_time)) > 0;
```

Tận dụng `COUNT(*)` thống kê tổng số bản ghi trong nhóm, `COUNT(submit_time)` chỉ thống kê số bản ghi mà field `submit_time` không phải NULL (tức số bài đã hoàn thành). Lấy tổng trừ đi số đã hoàn thành ra số bài chưa hoàn thành.

Cách viết 2:

```sql
SELECT
    exam_id,
    COUNT(CASE WHEN submit_time IS NULL THEN 1 END) AS incomplete_cnt,
    ROUND(COUNT(CASE WHEN submit_time IS NULL THEN 1 END) / COUNT(*), 3) AS incomplete_rate
FROM
    exam_record
GROUP BY
    exam_id
HAVING
    COUNT(CASE WHEN submit_time IS NULL THEN 1 END) > 0;
```

Sử dụng biểu thức `CASE`, khi thỏa mãn điều kiện trả về một giá trị không `NULL` (ví dụ 1), ngược lại trả về `NULL`. Sau đó dùng hàm `COUNT` để thống kê số lượng giá trị không `NULL`.

Cách viết 3:

```sql
SELECT
    exam_id,
    SUM(submit_time IS NULL) AS incomplete_cnt,
    ROUND(SUM(submit_time IS NULL) / COUNT(*), 3) AS incomplete_rate
FROM
    exam_record
GROUP BY
    exam_id
HAVING
    incomplete_cnt > 0;
```

Tận dụng hàm `SUM` để tính tổng một biểu thức. Khi `submit_time` là `NULL`, biểu thức `(submit_time IS NULL)` có giá trị 1 (TRUE), ngược lại là 0 (FALSE). Cộng các số 1 và 0 này lại sẽ được số lượng chưa hoàn thành.

### Thời gian làm bài trung bình和điểm trung bình đối với đề thi độ khó cao của người dùng cấp 0

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký), dữ liệu như sau:

| id  | uid  | nick_name | achievement | level | job  | register_time       |
| --- | ---- | --------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号 | 10          | 0     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号 | 2100        | 6     | 算法 | 2020-01-01 10:00:00 |

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành), dữ liệu như sau:

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL  | easy       | 60       | 2020-01-01 10:00:00 |
| 3   | 9004    | 算法 | medium     | 80       | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số), dữ liệu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 4   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 5   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |

Hãy output thời gian làm bài trung bình và điểm trung bình cho tất cả các đề thi độ khó cao của mỗi người dùng cấp 0, bài thi chưa hoàn thành mặc định xử lý bằng thời lượng thi tối đa của đề thi và 0 điểm. Kết quả mẫu như sau:

| uid  | avg_score | avg_time_took |
| ---- | --------- | ------------- |
| 1001 | 33        | 36.7          |

Giải thích: Người dùng cấp 0 có 1001, đề thi độ khó cao có 9001. 1001 làm đề 9001 có 3 bản ghi, thời gian làm bài lần lượt là 20 phút, chưa hoàn thành (thời lượng đề thi 60 phút), 30 phút (chưa đến 31 phút), điểm số lần lượt là 80 điểm, chưa hoàn thành (tính 0 điểm), 20 điểm. Do đó thời gian làm bài trung bình của anh ấy là 110/3 = 36.7 (làm tròn 1 chữ số thập phân), điểm trung bình là 33 điểm (lấy số nguyên).

**Tư duy giải đề**: Bài này dùng `IF` để kiểm tra là tiện nhất, vì liên quan đến kiểm tra giá trị NULL. Tất nhiên `CASE WHEN` cũng được, tương tự nhau. Điểm khó bài này nằm ở xử lý giá trị NULL, còn các điều kiện truy vấn khác tin rằng không làm khó được các bạn.

**Đáp án**:

```sql
SELECT UID,
       round(avg(new_socre)) AS avg_score,
       round(avg(time_diff), 1) AS avg_time_took
FROM
  (SELECT er.uid,
          IF (er.submit_time IS NOT NULL, TIMESTAMPDIFF(MINUTE, start_time, submit_time), ef.duration) AS time_diff,
          IF (er.submit_time IS NOT NULL,er.score,0) AS new_socre
   FROM exam_record er
   LEFT JOIN user_info uf ON er.uid = uf.uid
   LEFT JOIN examination_info ef ON er.exam_id = ef.exam_id
   WHERE uf.LEVEL = 0 AND ef.difficulty = 'hard' ) t
GROUP BY UID
ORDER BY UID
```

## Câu lệnh điều kiện nâng cao

### Lọc người dùng giới hạn biệt danh, điểm thành tựu, ngày hoạt động (Khá khó)

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name   | achievement | level | job  | register_time       |
| --- | ---- | ----------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号   | 1000        | 2     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号   | 1200        | 3     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 进击的 3 号 | 2200        | 5     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号   | 2500        | 6     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 5 号   | 3000        | 7     | C++  | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 4   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 11  | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 81     |
| 12  | 1002 | 9002    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 13  | 1002 | 9002    | 2020-02-02 12:11:01 | 2020-02-02 12:31:01 | 83     |
| 7   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 16  | 1002 | 9001    | 2021-09-06 12:01:01 | 2021-09-06 12:21:01 | 80     |
| 17  | 1002 | 9001    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 18  | 1002 | 9001    | 2021-09-07 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 89     |
| 10  | 1004 | 9002    | 2021-08-06 12:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-02-01 11:01:01 | 2021-02-01 11:31:01 | 84     |
| 15  | 1006 | 9001    | 2021-02-01 11:01:01 | 2021-02-01 11:31:01 | 84     |

Bảng lịch sử làm bài tập `practice_record` (`uid` ID người dùng, `question_id` ID bài tập, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1003 | 8002        | 2021-09-01 19:38:01 | 80    |

Hãy tìm thông tin người dùng có biệt danh bắt đầu bằng "牛客" và kết thúc bằng "号", điểm thành tựu trong khoảng 1200~2500, và lần hoạt động gần nhất (làm bài tập hoặc làm đề thi) vào tháng 9 năm 2021.

Kết quả mẫu như sau:

| uid  | nick_name | achievement |
| ---- | --------- | ----------- |
| 1002 | 牛客 2 号 | 1200        |

Giải thích: Người dùng có biệt danh bắt đầu bằng "牛客" kết thúc bằng "号" và điểm thành tựu trong khoảng 1200~2500 gồm 1002, 1004;

1002 lần hoạt động gần nhất khu vực làm bài thi là tháng 9/2021, khu vực bài tập là tháng 9/2021; 1004 lần hoạt động gần nhất khu vực làm bài thi là tháng 8/2021, khu vực bài tập không hoạt động.

Do đó cuối cùng chỉ có 1002 thỏa mãn điều kiện.

**Tư duy giải đề:**

Liệt kê các câu lệnh truy vấn chính dựa theo điều kiện trước:

Biệt danh bắt đầu bằng "牛客" kết thúc bằng "号": `nick_name LIKE "牛客%号"`

Điểm thành tựu trong khoảng 1200~2500: `achievement BETWEEN 1200 AND 2500`

Điều kiện thứ 3 vì giới hạn tháng 9 nên viết trực tiếp là được: `(DATE_FORMAT(record.submit_time, '%Y%m') = 202109 OR DATE_FORMAT(pr.submit_time, '%Y%m') = 202109)`

**Đáp án**:

```sql
SELECT DISTINCT u_info.uid,
                u_info.nick_name,
                u_info.achievement
FROM user_info u_info
LEFT JOIN exam_record record ON record.uid = u_info.uid
LEFT JOIN practice_record pr ON u_info.uid = pr.uid
WHERE u_info.nick_name LIKE "牛客%号"
  AND u_info.achievement BETWEEN 1200
  AND 2500
  AND (date_format(record.submit_time, '%Y%m')= 202109
       OR date_format(pr.submit_time, '%Y%m')= 202109)
GROUP BY u_info.uid
```

### Lọc lịch sử làm bài theo quy tắc biệt danh和quy tắc đề thi (Khá khó)

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name    | achievement | level | job  | register_time       |
| --- | ---- | ------------ | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号    | 1900        | 2     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号    | 1200        | 3     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 ♂ | 2200        | 5     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号    | 2500        | 6     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 555 号  | 2000        | 7     | C++  | 2020-01-01 10:00:00 |
| 6   | 1006 | 666666       | 3000        | 6     | C++  | 2020-01-01 10:00:00 |

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag | difficulty | duration | release_time        |
| --- | ------- | --- | ---------- | -------- | ------------------- |
| 1   | 9001    | C++ | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | c#  | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | SQL | medium     | 70       | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 4   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 5   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 11  | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 81     |
| 16  | 1002 | 9001    | 2021-09-06 12:01:01 | 2021-09-06 12:21:01 | 80     |
| 17  | 1002 | 9001    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 18  | 1002 | 9001    | 2021-09-07 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 12  | 1002 | 9002    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 13  | 1002 | 9002    | 2020-02-02 12:11:01 | 2020-02-02 12:31:01 | 83     |
| 9   | 1003 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 89     |
| 8   | 1003 | 9003    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 10  | 1004 | 9002    | 2021-08-06 12:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-02-01 11:01:01 | 2021-02-01 11:31:01 | 84     |
| 15  | 1006 | 9001    | 2021-02-01 11:01:01 | 2021-09-01 11:31:01 | 84     |

Tìm ID đề thi đã hoàn thành và điểm trung bình cho các thể loại đề thi bắt đầu bằng chữ 'c' (như C, C++, c#, v.v.) của những người dùng có biệt danh dạng "牛客" + chữ số + "号" hoặc gồm toàn chữ số. Sắp xếp tăng dần theo ID người dùng, điểm trung bình. Kết quả mẫu như sau:

| uid  | exam_id | avg_score |
| ---- | ------- | --------- |
| 1002 | 9001    | 81        |
| 1002 | 9002    | 85        |
| 1005 | 9001    | 84        |
| 1006 | 9001    | 84        |

Giải thích: Người dùng có biệt danh thỏa mãn điều kiện gồm 1002, 1004, 1005, 1006;

Đề thi bắt đầu bằng chữ c có 9001, 9002;

Trong lịch sử làm bài thỏa mãn điều kiện trên, 1002 hoàn thành 9001 có điểm 81, 80, điểm trung bình là 81 (80.5 làm tròn tứ xá ngũ nhập ra 81);

1002 hoàn thành 9002 có điểm 90, 82, 83, điểm trung bình là 85;

**Tư duy giải đề:**

Vẫn như cũ, vì đã đưa ra điều kiện thì cứ viết từng điều kiện ra trước.

Tìm người dùng có biệt danh dạng "牛客" + chữ số + "号" hoặc gồm toàn chữ số: Ban đầu tôi viết như thế này: `nick_name LIKE '牛客%号' OR nick_name REGEXP '^[0-9]+$'`, nhưng nếu trong bảng có "牛客 H 号" thì cũng pass.

Do đó ở đây vẫn phải dùng Regex: `nick_name REGEXP '^牛客[0-9]+号$'`

Đối với thể loại đề thi bắt đầu bằng chữ c: `e_info.tag LIKE 'c%'` hoặc `tag REGEXP '^c|^C'`, cách thứ nhất cũng khớp được với chữ C in hoa.

**Đáp án**:

```sql
SELECT UID,
       exam_id,
       ROUND(AVG(score), 0) avg_score
FROM exam_record
WHERE UID IN
    (SELECT UID
     FROM user_info
     WHERE nick_name RLIKE "^牛客[0-9]+号 $"
       OR nick_name RLIKE "^[0-9]+$")
  AND exam_id IN
    (SELECT exam_id
     FROM examination_info
     WHERE tag RLIKE "^[cC]")
  AND score IS NOT NULL
GROUP BY UID,exam_id
ORDER BY UID,avg_score;
```

### Output các trường hợp khác nhau tùy theo bản ghi chỉ định có tồn tại hay không (Khó)

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name   | achievement | level | job  | register_time       |
| --- | ---- | ----------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号   | 19          | 0     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号   | 1200        | 3     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 进击的 3 号 | 22          | 0     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号   | 25          | 0     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 555 号 | 2000        | 7     | C++  | 2020-01-01 10:00:00 |
| 6   | 1006 | 666666      | 3000        | 6     | C++  | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 4   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1001 | 9003    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9004    | 2021-09-03 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 99     |
| 8   | 1002 | 9003    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 9   | 1002 | 9003    | 2020-02-02 12:11:01 | (NULL)              | (NULL) |
| 10  | 1002 | 9002    | 2021-05-05 18:01:01 | (NULL)              | (NULL) |
| 11  | 1002 | 9001    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 12  | 1003 | 9003    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 13  | 1003 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 89     |

Hãy lọc dữ liệu trong bảng, khi có BẤT KỲ người dùng cấp 0 nào có số bài thi chưa hoàn thành lớn hơn 2, output số bài thi chưa hoàn thành và tỉ lệ chưa hoàn thành (làm tròn 3 chữ số thập phân) của từng người dùng cấp 0; Nếu không tồn tại người dùng như vậy, output 2 chỉ số này của tất cả người dùng có lịch sử làm bài. Kết quả sắp xếp tăng dần theo tỉ lệ chưa hoàn thành.

Kết quả mẫu như sau:

| uid  | incomplete_cnt | incomplete_rate |
| ---- | -------------- | --------------- |
| 1004 | 0              | 0.000           |
| 1003 | 1              | 0.500           |
| 1001 | 4              | 0.667           |

Giải thích: Người dùng cấp 0 có 1001, 1003, 1004; Số bài làm và số bài chưa hoàn thành của họ lần lượt là 6:4, 2:1, 0:0;

Tồn tại người dùng cấp 0 1001 có số bài chưa hoàn thành lớn hơn 2, do đó output số bài chưa hoàn thành và tỉ lệ chưa hoàn thành của 3 người dùng này (1004 chưa từng làm bài thi nào, tỉ lệ chưa hoàn thành mặc định điền 0, làm tròn 3 chữ số thập phân là 0.000);

Kết quả sắp xếp tăng dần theo tỉ lệ chưa hoàn thành.

Bổ sung: Nếu 1001 không thỏa mãn "số bài thi chưa hoàn thành lớn hơn 2", thì cần output 2 chỉ số này của 1001, 1002, 1003, vì trong bảng lịch sử làm bài thi chỉ có lịch sử làm bài của 3 người dùng này.

**Tư duy giải đề:**

Đầu tiên viết câu lệnh SQL có thể thỏa mãn điều kiện "người dùng cấp 0 có số bài thi chưa hoàn thành lớn hơn 2":

```sql
SELECT ui.uid UID
FROM user_info ui
LEFT JOIN exam_record er ON ui.uid = er.uid
WHERE ui.uid IN
    (SELECT ui.uid
     FROM user_info ui
     LEFT JOIN exam_record er ON ui.uid = er.uid
     WHERE er.submit_time IS NULL
       AND ui.LEVEL = 0 )
GROUP BY ui.uid
HAVING sum(IF(er.submit_time IS NULL, 1, 0)) > 2
```

Sau đó lần lượt viết câu lệnh truy vấn SQL cho 2 trường hợp:

Trường hợp 1. Truy vấn tỉ lệ chưa hoàn thành bài thi của người dùng cấp 0 theo yêu cầu điều kiện

```sql
SELECT
	tmp1.uid uid,
	sum(
	IF
	( er.submit_time IS NULL AND er.start_time IS NOT NULL, 1, 0 )) incomplete_cnt,
	round(
		sum(
		IF
		( er.submit_time IS NULL AND er.start_time IS NOT NULL, 1, 0 ))/ count( tmp1.uid ),
		3
	) incomplete_rate
FROM
	(
	SELECT DISTINCT
		ui.uid
	FROM
		user_info ui
		LEFT JOIN exam_record er ON ui.uid = er.uid
	WHERE
		er.submit_time IS NULL
		AND ui.LEVEL = 0
	) tmp1
	LEFT JOIN exam_record er ON tmp1.uid = er.uid
GROUP BY
	tmp1.uid
ORDER BY
	incomplete_rate
```

Trường hợp 2. Truy vấn tỉ lệ chưa hoàn thành bài thi của tất cả người dùng có lịch sử làm bài khi không tồn tại yêu cầu điều kiện

```sql
SELECT
	ui.uid uid,
	sum( CASE WHEN er.submit_time IS NULL AND er.start_time IS NOT NULL THEN 1 ELSE 0 END ) incomplete_cnt,
	round(
		sum(
		IF
		( er.submit_time IS NULL AND er.start_time IS NOT NULL, 1, 0 ))/ count( ui.uid ),
		3
	) incomplete_rate
FROM
	user_info ui
	JOIN exam_record er ON ui.uid = er.uid
GROUP BY
	ui.uid
ORDER BY
	incomplete_rate
```

Ghép lại với nhau chính là đáp án

```sql
WITH host_user AS
  (SELECT ui.uid UID
   FROM user_info ui
   LEFT JOIN exam_record er ON ui.uid = er.uid
   WHERE ui.uid IN
       (SELECT ui.uid
        FROM user_info ui
        LEFT JOIN exam_record er ON ui.uid = er.uid
        WHERE er.submit_time IS NULL
          AND ui.LEVEL = 0 )
   GROUP BY ui.uid
   HAVING sum(IF (er.submit_time IS NULL, 1, 0))> 2),
      tt1 AS
  (SELECT tmp1.uid UID,
                   sum(IF (er.submit_time IS NULL
                           AND er.start_time IS NOT NULL, 1, 0)) incomplete_cnt,
                   round(sum(IF (er.submit_time IS NULL
                                 AND er.start_time IS NOT NULL, 1, 0))/ count(tmp1.uid), 3) incomplete_rate
   FROM
     (SELECT DISTINCT ui.uid
      FROM user_info ui
      LEFT JOIN exam_record er ON ui.uid = er.uid
      WHERE er.submit_time IS NULL
        AND ui.LEVEL = 0 ) tmp1
   LEFT JOIN exam_record er ON tmp1.uid = er.uid
   GROUP BY tmp1.uid
   ORDER BY incomplete_rate),
      tt2 AS
  (SELECT ui.uid UID,
                 sum(CASE
                         WHEN er.submit_time IS NULL
                              AND er.start_time IS NOT NULL THEN 1
                         ELSE 0
                     END) incomplete_cnt,
                 round(sum(IF (er.submit_time IS NULL
                               AND er.start_time IS NOT NULL, 1, 0))/ count(ui.uid), 3) incomplete_rate
   FROM user_info ui
   JOIN exam_record er ON ui.uid = er.uid
   GROUP BY ui.uid
   ORDER BY incomplete_rate)
  (SELECT tt1.*
   FROM tt1
   LEFT JOIN
     (SELECT UID
      FROM host_user) t1 ON 1 = 1
   WHERE t1.uid IS NOT NULL )
UNION ALL
  (SELECT tt2.*
   FROM tt2
   LEFT JOIN
     (SELECT UID
      FROM host_user) t2 ON 1 = 1
   WHERE t2.uid IS NULL)
```

Phiên bản V2 (Cải tiến dựa trên phần trên, đáp án ngắn hơn, logic mạnh hơn):

```sql
SELECT
	ui.uid,
	SUM(
	IF
	( start_time IS NOT NULL AND score IS NULL, 1, 0 )) AS incomplete_cnt,# 3. Số bài thi chưa hoàn thành
	ROUND( AVG( IF ( start_time IS NOT NULL AND score IS NULL, 1, 0 )), 3 ) AS incomplete_rate # 4. Tỉ lệ chưa hoàn thành

FROM
	user_info ui
	LEFT JOIN exam_record USING ( uid )
WHERE
CASE

		WHEN (# 1. Khi có bất kỳ một người dùng cấp 0 nào có số bài chưa hoàn thành lớn hơn 2
		SELECT
			MAX( lv0_incom_cnt )
		FROM
			(
			SELECT
				SUM(
				IF
				( score IS NULL, 1, 0 )) AS lv0_incom_cnt
			FROM
				user_info
				JOIN exam_record USING ( uid )
			WHERE
				LEVEL = 0
			GROUP BY
				uid
			) table1
			)> 2 THEN
			uid IN ( # 1.1 Tìm từng người dùng cấp 0
			SELECT uid FROM user_info WHERE LEVEL = 0 ) ELSE uid IN ( # 2. Nếu không tồn tại người dùng như vậy, tìm người dùng có lịch sử làm bài
			SELECT DISTINCT uid FROM exam_record )
		END
		GROUP BY
			ui.uid
	ORDER BY
	incomplete_rate # 5. Kết quả sắp xếp tăng dần theo tỉ lệ chưa hoàn thành
```

### Tỉ lệ biểu hiện điểm số khác nhau của từng cấp độ người dùng (Khá khó)

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name    | achievement | level | job  | register_time       |
| --- | ---- | ------------ | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号    | 19          | 0     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号    | 1200        | 3     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 ♂ | 22          | 0     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号    | 25          | 0     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 555 号  | 2000        | 7     | C++  | 2020-01-01 10:00:00 |
| 6   | 1006 | 666666       | 3000        | 6     | C++  | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi exam_record (uid 用户 ID, exam_id 试卷 ID, start_time 开始作答时间, submit_time 交卷时间, score 得分）：

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | (NULL)              | (NULL) |
| 3   | 1001 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 75     |
| 4   | 1001 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:11:01 | 60     |
| 5   | 1001 | 9003    | 2021-09-02 12:01:01 | 2021-09-02 12:41:01 | 90     |
| 6   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:32:00 | 20     |
| 7   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 8   | 1001 | 9004    | 2021-09-03 12:01:01 | (NULL)              | (NULL) |
| 9   | 1002 | 9001    | 2020-01-01 12:01:01 | 2020-01-01 12:31:01 | 99     |
| 10  | 1002 | 9003    | 2020-02-01 12:01:01 | 2020-02-01 12:31:01 | 82     |
| 11  | 1002 | 9003    | 2020-02-02 12:11:01 | 2020-02-02 12:41:01 | 76     |

Để có được biểu hiện định tính của người dùng khi làm bài thi, chúng ta chia điểm bài thi thành 4 cấp độ điểm số Giỏi - Khá - Trung bình - Yếu (优良中差) theo các mốc phân giới [90, 75, 60] (mốc phân giới tính vào khoảng bên trái), hãy thống kê tỉ lệ từng cấp độ điểm số trong các bài thi đã hoàn thành của người dùng ở các level khác nhau (kết quả làm tròn 3 chữ số thập phân), người dùng chưa từng hoàn thành bài thi không cần output, kết quả sắp xếp giảm dần theo level người dùng, giảm dần theo tỉ lệ.

Kết quả mẫu như sau:

| level | score_grade | ratio |
| ----- | ----------- | ----- |
| 3     | 良          | 0.667 |
| 3     | 优          | 0.333 |
| 0     | 良          | 0.500 |
| 0     | 中          | 0.167 |
| 0     | 优          | 0.167 |
| 0     | 差          | 0.167 |

Giải thích: Người dùng từng hoàn thành bài thi có 1001, 1002; Level người dùng và Cấp độ điểm số tương ứng với các bài thi đã hoàn thành như sau:

| uid  | exam_id | score | level | score_grade |
| ---- | ------- | ----- | ----- | ----------- |
| 1001 | 9001    | 80    | 0     | 良          |
| 1001 | 9002    | 75    | 0     | 良          |
| 1001 | 9002    | 60    | 0     | 中          |
| 1001 | 9003    | 90    | 0     | 优          |
| 1001 | 9001    | 20    | 0     | 差          |
| 1001 | 9002    | 89    | 0     | 良          |
| 1002 | 9001    | 99    | 3     | 优          |
| 1002 | 9003    | 82    | 3     | 良          |
| 1002 | 9003    | 76    | 3     | 良          |

Do đó người dùng cấp 0 (chỉ có 1001) có tỉ lệ các cấp độ điểm số là: Giỏi (优) 1/6, Khá (良) 3/6, Trung bình (中) 1/6, Yếu (差) 1/6; Người dùng cấp 3 (chỉ có 1002) có tỉ lệ các cấp độ điểm số là: Giỏi (优) 1/3, Khá (良) 2/3. Kết quả làm tròn 3 chữ số thập phân.

**Tư duy giải đề:**

Đầu tiên viết điều kiện **"Chia điểm bài thi thành 4 cấp độ Giỏi - Khá - Trung bình - Yếu theo các mốc [90, 75, 60]"**, ở đây có thể dùng `CASE WHEN`

```sql
CASE
		WHEN a.score >= 90 THEN
		'优'
		WHEN a.score < 90 AND a.score >= 75 THEN
		'良'
		WHEN a.score < 75 AND a.score >= 60 THEN
	'中' ELSE '差'
END
```

Điểm mấu chốt của bài này nằm ở đây, những phần còn lại là ghép điều kiện.

**Đáp án**:

```sql
SELECT a.LEVEL,
       a.score_grade,
       ROUND(a.cur_count / b.total_num, 3) AS ratio
FROM
  (SELECT b.LEVEL AS LEVEL,
          (CASE
               WHEN a.score >= 90 THEN '优'
               WHEN a.score < 90
                    AND a.score >= 75 THEN '良'
               WHEN a.score < 75
                    AND a.score >= 60 THEN '中'
               ELSE '差'
           END) AS score_grade,
          count(1) AS cur_count
   FROM exam_record a
   LEFT JOIN user_info b ON a.uid = b.uid
   WHERE a.submit_time IS NOT NULL
   GROUP BY b.LEVEL,
            score_grade) a
LEFT JOIN
  (SELECT b.LEVEL AS LEVEL,
          count(b.LEVEL) AS total_num
   FROM exam_record a
   LEFT JOIN user_info b ON a.uid = b.uid
   WHERE a.submit_time IS NOT NULL
   GROUP BY b.LEVEL) b ON a.LEVEL = b.LEVEL
ORDER BY a.LEVEL DESC,
         ratio DESC
```

## Truy vấn giới hạn số lượng (LIMIT)

### Top 3 người có thời gian đăng ký sớm nhất

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name    | achievement | level | job  | register_time       |
| --- | ---- | ------------ | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号    | 19          | 0     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号    | 1200        | 3     | 算法 | 2020-02-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 ♂ | 22          | 0     | 算法 | 2020-01-02 10:00:00 |
| 4   | 1004 | 牛客 4 号    | 25          | 0     | 算法 | 2020-01-02 11:00:00 |
| 5   | 1005 | 牛客 555 号  | 4000        | 7     | C++  | 2020-01-11 10:00:00 |
| 6   | 1006 | 666666       | 3000        | 6     | C++  | 2020-11-01 10:00:00 |

Hãy tìm 3 người có thời gian đăng ký sớm nhất. Kết quả mẫu như sau:

| uid  | nick_name    | register_time       |
| ---- | ------------ | ------------------- |
| 1001 | 牛客 1       | 2020-01-01 10:00:00 |
| 1003 | 牛客 3 号 ♂ | 2020-01-02 10:00:00 |
| 1004 | 牛客 4 号    | 2020-01-02 11:00:00 |

Giải thích: Sau khi sắp xếp theo thời gian đăng ký lấy top 3, output ID người dùng, biệt danh, thời gian đăng ký.

**Đáp án**:

```sql
SELECT uid, nick_name, register_time
    FROM user_info
    ORDER BY register_time
    LIMIT 3
```

### Trang thứ 3 danh sách những người hoàn thành đề thi ngay trong ngày đăng ký (Khá khó)

**Mô tả**: Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name    | achievement | level | job  | register_time       |
| --- | ---- | ------------ | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1       | 19          | 0     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号    | 1200        | 3     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 ♂ | 22          | 0     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号    | 25          | 0     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 555 号  | 4000        | 7     | 算法 | 2020-01-11 10:00:00 |
| 6   | 1006 | 牛客 6 号    | 25          | 0     | 算法 | 2020-01-02 11:00:00 |
| 7   | 1007 | 牛客 7 号    | 25          | 0     | 算法 | 2020-01-02 11:00:00 |
| 8   | 1008 | 牛客 8 号    | 25          | 0     | 算法 | 2020-01-02 11:00:00 |
| 9   | 1009 | 牛客 9 号    | 25          | 0     | 算法 | 2020-01-02 11:00:00 |
| 10  | 1010 | 牛客 10 号   | 25          | 0     | 算法 | 2020-01-02 11:00:00 |
| 11  | 1011 | 666666       | 3000        | 6     | C++  | 2020-01-02 10:00:00 |

Bảng thông tin đề thi examination_info (exam_id 试卷 ID, tag 试卷类别, difficulty 试卷难度, duration 考试时长, release_time 发布时间）：

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | 算法 | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | 算法 | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | SQL  | medium     | 70       | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:59 | 80    |
| 2   | 1002 | 9003    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 81    |
| 3   | 1002 | 9002    | 2020-01-01 12:11:01 | 2020-01-01 12:31:01 | 83    |
| 4   | 1003 | 9002    | 2020-01-01 19:01:01 | 2020-01-01 19:30:01 | 75    |
| 5   | 1004 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:11:01 | 60    |
| 6   | 1005 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:41:01 | 90    |
| 7   | 1006 | 9001    | 2020-01-02 19:01:01 | 2020-01-02 19:32:00 | 20    |
| 8   | 1007 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:40:01 | 89    |
| 9   | 1008 | 9003    | 2020-01-02 12:01:01 | 2020-01-02 12:20:01 | 99    |
| 10  | 1008 | 9001    | 2020-01-02 12:01:01 | 2020-01-02 12:31:01 | 98    |
| 11  | 1009 | 9002    | 2020-01-02 12:01:01 | 2020-01-02 12:31:01 | 82    |
| 12  | 1010 | 9002    | 2020-01-02 12:11:01 | 2020-01-02 12:41:01 | 76    |
| 13  | 1011 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89    |

![](https://oss.javaguide.cn/github/javaguide/database/sql/D2B491866B85826119EE3474F10D3636.png)

Tìm những người có định hướng nghề nghiệp là Kỹ sư Thuật toán, và hoàn thành đề thi thể loại Thuật toán (算法) ngay trong ngày đăng ký, sắp xếp theo điểm cao nhất của tất cả các kỳ thi đã tham gia. Bảng xếp hạng rất dài, chúng ta áp dụng phân trang (paging), mỗi trang 3 bản ghi, bây giờ cần bạn lấy ra thông tin người dùng ở trang thứ 3 (trang bắt đầu từ 1).

Kết quả mẫu như sau:

| uid  | level | register_time       | max_score |
| ---- | ----- | ------------------- | --------- |
| 1010 | 0     | 2020-01-02 11:00:00 | 76        |
| 1003 | 0     | 2020-01-01 10:00:00 | 75        |
| 1004 | 0     | 2020-01-01 11:00:00 | 60        |

Giải thích: Trừ 1011 ra tất cả người dùng khác đều có định hướng nghề nghiệp là Kỹ sư Thuật toán; Đề thi thể loại Thuật toán gồm 9001 và 9002, 11 người dùng đều hoàn thành đề thi Thuật toán ngay trong ngày đăng ký; Khi tính điểm tối đa tất cả các kỳ thi của họ, chỉ có 1002 và 1008 hoàn thành 2 kỳ thi, những người khác chỉ hoàn thành 1 kỳ thi, 1002 điểm cao nhất trong 2 kỳ thi là 81, 1008 điểm cao nhất là 99.

Bảng xếp hạng theo điểm cao nhất như sau:

| uid  | level | register_time       | max_score |
| ---- | ----- | ------------------- | --------- |
| 1008 | 0     | 2020-01-02 11:00:00 | 99        |
| 1005 | 7     | 2020-01-01 10:00:00 | 90        |
| 1007 | 0     | 2020-01-02 11:00:00 | 89        |
| 1002 | 3     | 2020-01-01 10:00:00 | 83        |
| 1009 | 0     | 2020-01-02 11:00:00 | 82        |
| 1001 | 0     | 2020-01-01 10:00:00 | 80        |
| 1010 | 0     | 2020-01-02 11:00:00 | 76        |
| 1003 | 0     | 2020-01-01 10:00:00 | 75        |
| 1004 | 0     | 2020-01-01 11:00:00 | 60        |
| 1006 | 0     | 2020-01-02 11:00:00 | 20        |

Mỗi trang 3 bản ghi, trang thứ 3 tức là các dòng từ 7 đến 9, trả về các bản ghi của 1010, 1003, 1004 là được.

**Tư duy giải đề:**

1. Mỗi trang 3 bản ghi, tức là cần lấy thông tin của người dùng ở trang 3, cần dùng `LIMIT`

2. Thống kê thông tin và điểm số từng bản ghi của người có định hướng Kỹ sư Thuật toán và hoàn thành bài thi Thuật toán ngay trong ngày đăng ký. Đầu tiên tìm người dùng thỏa mãn điều kiện, sau đó dùng LEFT JOIN để truy vấn thông tin và điểm số.

**Đáp án**:

```sql
SELECT t1.uid,
       LEVEL,
       register_time,
       max(score) AS max_score
FROM exam_record t
JOIN examination_info USING (exam_id)
JOIN user_info t1 ON t.uid = t1.uid
AND date(t.submit_time) = date(t1.register_time)
WHERE job = '算法'
  AND tag = '算法'
GROUP BY t1.uid,
         LEVEL,
         register_time
ORDER BY max_score DESC
LIMIT 6,3
```

## Hàm xử lý chuỗi văn bản (String Functions)

### Sửa chữa các bản ghi bị lệch column

**Mô tả**: Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag            | difficulty | duration | release_time        |
| --- | ------- | -------------- | ---------- | -------- | ------------------- |
| 1   | 9001    | 算法           | hard       | 60       | 2021-01-01 10:00:00 |
| 2   | 9002    | 算法           | hard       | 80       | 2021-01-01 10:00:00 |
| 3   | 9003    | SQL            | medium     | 70       | 2021-01-01 10:00:00 |
| 4   | 9004    | 算法,medium,80 |            | 0        | 2021-01-01 10:00:00 |

Bạn nhập đề thi có một lần lỡ tay nhập đồng thời tag thể loại đề thi, độ khó, thời lượng của một số bản ghi vào field `tag`, hãy giúp tìm ra các bản ghi nhập sai này, và bóc tách output theo đúng kiểu column.

Kết quả mẫu như sau:

| exam_id | tag  | difficulty | duration |
| ------- | ---- | ---------- | -------- |
| 9004    | 算法 | medium     | 80       |

**Tư duy giải đề:**

Đầu tiên cùng tìm hiểu hàm sẽ dùng trong bài này:

Hàm `SUBSTRING_INDEX` dùng để trích xuất phần chuỗi theo dấu phân cách chỉ định. Nó nhận 3 tham số: chuỗi ban đầu, dấu phân cách và số lượng phần cần trả về.

Cú pháp của hàm `SUBSTRING_INDEX` như sau:

```sql
SUBSTRING_INDEX(str, delimiter, count)
```

- `str`: Chuỗi ban đầu cần tách.
- `delimiter`: Chuỗi hoặc ký tự dùng làm dấu phân cách.
- `count`: Chỉ định số lượng phần cần trả về.
  - Nếu `count` > 0, trả về `count` phần đầu tiên tính từ bên trái (ngăn cách bởi delimiter).
  - Nếu `count` < 0, trả về `count` phần đầu tiên tính từ bên phải (ngăn cách bởi delimiter), tức đếm từ phải sang trái.

Dưới đây là một số ví dụ minh họa cách dùng hàm `SUBSTRING_INDEX`:

1. Trích xuất phần đầu tiên trong chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', 1);
   -- Kết quả trả về: 'apple'
   ```

2. Trích xuất phần cuối cùng trong chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', -1);
   -- Kết quả trả về: 'cherry'
   ```

3. Trích xuất 2 phần đầu tiên trong chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', 2);
   -- Kết quả trả về: 'apple,banana'
   ```

4. Trích xuất 2 phần cuối cùng trong chuỗi:

   ```sql
   SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', -2);
   -- Kết quả trả về: 'banana,cherry'
   ```

**Đáp án**:

```sql
SELECT
	exam_id,
	substring_index( tag, ',', 1 ) tag,
	substring_index( substring_index( tag, ',', 2 ), ',',- 1 ) difficulty,
	substring_index( tag, ',',- 1 ) duration
FROM
	examination_info
WHERE
	difficulty = ''
```

### Cắt ngắn xử lý biệt danh quá dài

**Mô tả**: Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name              | achievement | level | job  | register_time       |
| --- | ---- | ---------------------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1                 | 19          | 0     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号              | 1200        | 3     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 ♂           | 22          | 0     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号              | 25          | 0     | 算法 | 2020-01-01 11:00:00 |
| 5   | 1005 | 牛客 5678901234 号     | 4000        | 7     | 算法 | 2020-01-11 10:00:00 |
| 6   | 1006 | 牛客 67890123456789 号 | 25          | 0     | 算法 | 2020-01-02 11:00:00 |

Biệt danh của một số người dùng đặc biệt dài, trong một số giao diện hiển thị sẽ gây rối layout. Do đó cần biến đổi biệt danh đặc biệt dài trước khi output. Hãy output thông tin người dùng có số ký tự lớn hơn 10; đối với người dùng có số ký tự lớn hơn 13, output 10 ký tự đầu tiên rồi thêm 3 dấu chấm: '...'

Kết quả mẫu như sau:

| uid  | nick_name          |
| ---- | ------------------ |
| 1005 | 牛客 5678901234 号 |
| 1006 | 牛客 67890123...   |

Giải thích: Người dùng có số ký tự lớn hơn 10 gồm 1005 và 1006, độ dài lần lượt là 13, 17; Do đó cần cắt ngắn output đối với biệt danh của 1006.

**Tư duy giải đề:**

Bài này liên quan đến tính toán ký tự, muốn tính số ký tự của chuỗi (tức độ dài chuỗi), có thể dùng hàm `LENGTH` hoặc hàm `CHAR_LENGTH`. Sự khác biệt của 2 hàm này nằm ở cách xử lý ký tự đa byte (multi-byte).

1. Hàm `LENGTH`: Trả về số byte của chuỗi chỉ định. Đối với chuỗi chứa ký tự đa byte, mỗi ký tự sẽ được tính theo số byte của nó.

Ví dụ:

```sql
SELECT LENGTH('你好'); -- Kết quả: 6, vì mỗi chữ Hán trong '你好' chiếm 3 bytes (với UTF-8)
```

2. Hàm `CHAR_LENGTH`: Trả về số lượng ký tự của chuỗi chỉ định. Đối的于 chuỗi chứa ký tự đa byte, mỗi ký tự được tính là 1 ký tự.

Ví dụ:

```sql
SELECT CHAR_LENGTH('你好'); -- Kết quả: 2, vì trong '你好' có 2 ký tự (2 chữ Hán)
```

**Đáp án**:

```sql
SELECT
	uid,
CASE

		WHEN CHAR_LENGTH( nick_name ) > 13 THEN
		CONCAT( SUBSTR( nick_name, 1, 10 ), '...' ) ELSE nick_name
	END AS nick_name
FROM
	user_info
WHERE
	CHAR_LENGTH( nick_name ) > 10
GROUP BY
	uid;
```

### Thống kê lọc dữ liệu khi chữ hoa chữ thường bị lẫn lộn (Khá khó)

**Mô tả**:

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | 算法 | hard       | 60       | 2021-01-01 10:00:00 |
| 2   | 9002    | C++  | hard       | 80       | 2021-01-01 10:00:00 |
| 3   | 9003    | C++  | hard       | 80       | 2021-01-01 10:00:00 |
| 4   | 9004    | sql  | medium     | 70       | 2021-01-01 10:00:00 |
| 5   | 9005    | C++  | hard       | 80       | 2021-01-01 10:00:00 |
| 6   | 9006    | C++  | hard       | 80       | 2021-01-01 10:00:00 |
| 7   | 9007    | C++  | hard       | 80       | 2021-01-01 10:00:00 |
| 8   | 9008    | SQL  | medium     | 70       | 2021-01-01 10:00:00 |
| 9   | 9009    | SQL  | medium     | 70       | 2021-01-01 10:00:00 |
| 10  | 9010    | SQL  | medium     | 70       | 2021-01-01 10:00:00 |

Bảng thông tin làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 80     |
| 2   | 1002 | 9003    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 81     |
| 3   | 1002 | 9002    | 2020-02-01 12:11:01 | 2020-02-01 12:31:01 | 83     |
| 4   | 1003 | 9002    | 2020-03-01 19:01:01 | 2020-03-01 19:30:01 | 75     |
| 5   | 1004 | 9002    | 2020-03-01 12:01:01 | 2020-03-01 12:11:01 | 60     |
| 6   | 1005 | 9002    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90     |
| 7   | 1006 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 20     |
| 8   | 1007 | 9003    | 2020-01-02 19:01:01 | 2020-01-02 19:40:01 | 89     |
| 9   | 1008 | 9004    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99     |
| 10  | 1008 | 9001    | 2020-02-02 12:01:01 | 2020-02-02 12:31:01 | 98     |
| 11  | 1009 | 9002    | 2020-02-02 12:01:01 | 2020-01-02 12:43:01 | 81     |
| 12  | 1010 | 9001    | 2020-01-02 12:11:01 | (NULL)              | (NULL) |
| 13  | 1010 | 9001    | 2020-02-02 12:01:01 | 2020-01-02 10:31:01 | 89     |

Tag thể loại đề thi có thể xuất hiện trường hợp chữ hoa chữ thường bị lẫn lộn, hãy lọc ra các tag thể loại có số lượt làm bài ít hơn 3 trước, thống kê số lượt làm bài ban đầu tương ứng sau khi chuyển tag đó thành chữ in hoa.

Nếu sau khi chuyển đổi tag không thay đổi (tức ban đầu đã là in hoa), không output kết quả đó.

Kết quả mẫu như sau:

| tag | answer_cnt |
| --- | ---------- |
| C++ | 6          |

Giải thích: Đề thi từng được làm gồm 9001, 9002, 9003, 9004, tag và số lượt làm bài của chúng như sau:

| exam_id | tag  | answer_cnt |
| ------- | ---- | ---------- |
| 9001    | 算法 | 4          |
| 9002    | C++  | 6          |
| 9003    | c++  | 2          |
| 9004    | sql  | 2          |

Tag có số lượt làm bài ít hơn 3 gồm c++ và sql, nhưng sau khi chuyển thành in hoa chỉ có C++ vốn đã có số lượt làm bài (6 lần), do đó output số lượt làm bài sau khi c++ chuyển thành in hoa là 6.

**Tư duy giải đề:**

Đầu tiên bài này hơi rắc rối một chút, 9004 dựa theo dữ liệu mẫu truy vấn ra chỉ có 1 lần, ở đây hiển thị 2 lần.

Cùng xem các hàm chuyển đổi chữ hoa chữ thường:

1. Hàm `UPPER(s)` hoặc `UCASE(s)` có thể chuyển toàn bộ ký tự chữ cái trong chuỗi s thành chữ in hoa;

2. Hàm `LOWER(s)` hoặc `LCASE(s)` có thể chuyển toàn bộ ký tự chữ cái trong chuỗi s thành chữ in thường.

Điểm khó nằm ở chỗ SELF JOIN cùng một bảng để truy vấn các giá trị khác nhau.

**Đáp án**:

```sql
WITH a AS
  (SELECT tag,
          COUNT(start_time) AS answer_cnt
   FROM exam_record er
   JOIN examination_info ei ON er.exam_id = ei.exam_id
   GROUP BY tag)
SELECT a.tag,
       b.answer_cnt
FROM a
INNER JOIN a AS b ON UPPER(a.tag)= b.tag # a chữ thường, b chữ hoa
AND a.tag != b.tag
WHERE a.answer_cnt < 3;
```

<!-- @include: @article-footer.snippet.md -->
