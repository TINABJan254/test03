---
title: SQL常见面试题总结（4）
description: SQL常见面试题总结第四篇，详解MySQL 8.0窗口函数ROW_NUMBER、RANK、DENSE_RANK、NTILE、LAG、LEAD等的用法和应用场景。
category: 数据库
tag:
  - 数据库基础
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL面试题,窗口函数,ROW_NUMBER,RANK,DENSE_RANK,NTILE,LAG,LEAD,MySQL 8.0
---

> Các câu hỏi từ: [Niuke Tiba - Thử thách SQL nâng cao](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

Các câu hỏi thuộc mức Độ khó trung bình - cao hoặc Khó có thể căn cứ vào tình hình thực tế và nhu cầu phỏng vấn của bản thân để quyết định có nên bỏ qua hay không.

## Hàm Window chuyên dụng (Specialized Window Functions)

Phiên bản MySQL 8.0 đã hỗ trợ Hàm Window, dưới đây là các Hàm Window phổ biến trong MySQL và cách dùng của chúng:

1. `ROW_NUMBER()`: Phân bổ một giá trị số nguyên duy nhất cho từng dòng trong tập kết quả truy vấn.

```sql
SELECT col1, col2, ROW_NUMBER() OVER (ORDER BY col1) AS row_num
FROM table;
```

2. `RANK()`: Tính thứ hạng (ranking) của từng dòng trong kết quả đã sắp xếp (nếu trùng điểm sẽ nhảy số thứ hạng).

```sql
SELECT col1, col2, RANK() OVER (ORDER BY col1 DESC) AS ranking
FROM table;
```

3. `DENSE_RANK()`: Tính thứ hạng của từng dòng trong kết quả đã sắp xếp (nếu trùng điểm sẽ giữ nguyên thứ hạng liền kề, không nhảy số).

```sql
SELECT col1, col2, DENSE_RANK() OVER (ORDER BY col1 DESC) AS ranking
FROM table;
```

4. `NTILE(n)`: Chia kết quả thành n bucket tương đối đều nhau, và gán một số ID cho mỗi bucket.

```sql
SELECT col1, col2, NTILE(4) OVER (ORDER BY col1) AS bucket
FROM table;
```

5. `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()`: Các hàm Aggregation này cũng có thể kết hợp sử dụng với Hàm Window, tính tổng, trung bình, số lượng, giá trị nhỏ nhất và lớn nhất của column chỉ định trong window.

```sql
SELECT col1, col2, SUM(col1) OVER () AS sum_col
FROM table;
```

6. `LEAD()` và `LAG()`: Hàm LEAD dùng để lấy giá trị dòng phía sau dòng hiện tại theo một offset nào đó, còn hàm LAG dùng để lấy giá trị dòng phía trước dòng hiện tại theo offset.

```sql
SELECT col1, col2, LEAD(col1, 1) OVER (ORDER BY col1) AS next_col1,
                 LAG(col1, 1) OVER (ORDER BY col1) AS prev_col1
FROM table;
```

7. `FIRST_VALUE()` và `LAST_VALUE()`: Hàm FIRST_VALUE dùng để lấy giá trị đầu tiên của column chỉ định trong window, hàm LAST_VALUE dùng để lấy giá trị cuối cùng của column chỉ định trong window.

```sql
SELECT col1, col2, FIRST_VALUE(col2) OVER (PARTITION BY col1 ORDER BY col2) AS first_val,
                 LAST_VALUE(col2) OVER (PARTITION BY col1 ORDER BY col2) AS last_val
FROM table;
```

Hàm Window thường cần phối hợp sử dụng với mệnh đề OVER để định nghĩa kích thước window, quy tắc sắp xếp (ORDER BY) và cách gom nhóm (PARTITION BY).

### Top 3 điểm cao nhất của mỗi thể loại đề thi

**Mô tả**:

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2021-09-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, score 得分):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 78     |
| 2   | 1002 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 3   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 4   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 86     |
| 5   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:51 | 89     |
| 6   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |
| 7   | 1005 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85     |
| 8   | 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 84     |
| 9   | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 10  | 1003 | 9002    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |

Tìm top 3 điểm thi của mỗi thể loại đề thi, nếu điểm cao nhất của 2 người giống nhau thì chọn người có điểm thấp nhất lớn hơn, nếu vẫn giống nhau thì chọn người có uid lớn hơn. Kết quả mẫu như sau:

| tid  | uid  | ranking |
| ---- | ---- | ------- |
| SQL  | 1003 | 1       |
| SQL  | 1004 | 2       |
| SQL  | 1002 | 3       |
| 算法 | 1005 | 1       |
| 算法 | 1006 | 2       |
| 算法 | 1003 | 3       |

**Giải thích**: Tag đề thi có lịch sử điểm làm bài gồm SQL và Thuật toán (算法). Người dùng làm bài thi SQL gồm 1001, 1002, 1003, 1004 có điểm số, điểm cao nhất lần lượt là 81, 81, 89, 85, điểm thấp nhất lần lượt là 78, 81, 86, 40. Do đó xếp hạng theo điểm cao nhất trước rồi theo điểm thấp nhất lấy top 3 là 1003, 1004, 1002.

**Đáp án**:

```sql
SELECT tag,
       UID,
       ranking
FROM
  (SELECT b.tag AS tag,
          a.uid AS UID,
          ROW_NUMBER() OVER (PARTITION BY b.tag
                             ORDER BY b.tag,
                                      max(a.score) DESC,
                                      min(a.score) DESC,
                                      a.uid DESC) AS ranking
   FROM exam_record a
   LEFT JOIN examination_info b ON a.exam_id = b.exam_id
   GROUP BY b.tag,
            a.uid) t
WHERE ranking <= 3
```

### Đề thi có chênh lệch thời gian giữa người nhanh thứ 2 và chậm thứ 2 lớn hơn một nửa thời lượng thi (Khá khó)

**Mô tả**:

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++  | hard       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2021-09-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:51:01 | 78     |
| 2   | 1001 | 9002    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 3   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 4   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:59:01 | 86     |
| 5   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:51 | 89     |
| 6   | 1004 | 9002    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |
| 7   | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85     |
| 8   | 1006 | 9001    | 2021-09-07 10:02:01 | 2021-09-07 10:21:01 | 84     |
| 9   | 1003 | 9001    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 10  | 1003 | 9002    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |
| 11  | 1005 | 9001    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |
| 12  | 1003 | 9003    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |

Tìm thông tin các đề thi có chênh lệch thời gian làm bài giữa người làm nhanh thứ 2 và làm chậm thứ 2 lớn hơn một nửa thời lượng đề thi, sắp xếp giảm dần theo ID đề thi. Kết quả mẫu như sau:

| exam_id | duration | release_time        |
| ------- | -------- | ------------------- |
| 9001    | 60       | 2021-09-01 06:00:00 |

**Giải thích**: Đề thi 9001 có thời gian làm bài là 50 phút, 58 phút, 30 phút 1 giây, 19 phút, 10 phút. Chênh lệch thời gian giữa người nhanh thứ 2 và chậm thứ 2 là 50 phút - 19 phút = 31 phút, thời lượng đề thi là 60 phút, do đó thỏa mãn điều kiện lớn hơn một nửa thời lượng đề thi, output ID đề thi, thời lượng, thời gian phát hành.

**Tư duy giải đề:**

Bước 1, tìm thứ hạng theo chiều xuôi và chiều ngược thời gian hoàn thành của mỗi đề thi, tức là bảng a;

Bước 2, thực hiện INNER JOIN với bảng thông tin đề thi b, gom nhóm theo ID đề thi, dùng `HAVING` để lọc ra dữ liệu thứ hạng là 2, đổi giây thành phút để so sánh, cuối cùng sắp xếp giảm dần theo ID đề thi là xong.

**Đáp án**:

```sql
SELECT a.exam_id,
       b.duration,
       b.release_time
FROM
  (SELECT exam_id,
          row_number() OVER (PARTITION BY exam_id
                             ORDER BY timestampdiff(SECOND, start_time, submit_time) DESC) rn1,
          row_number() OVER (PARTITION BY exam_id
                             ORDER BY timestampdiff(SECOND, start_time, submit_time) ASC) rn2,
                                              timestampdiff(SECOND, start_time, submit_time) timex
   FROM exam_record
   WHERE score IS NOT NULL ) a
INNER JOIN examination_info b ON a.exam_id = b.exam_id
GROUP BY a.exam_id
HAVING (max(IF (rn1 = 2, a.timex, 0))- max(IF (rn2 = 2, a.timex, 0)))/ 60 > b.duration / 2
ORDER BY a.exam_id DESC
```

### Window thời gian tối đa giữa 2 lần làm bài liên tiếp (Khá khó)

**Mô tả**

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:02 | 84    |
| 2   | 1006 | 9001    | 2021-09-01 12:11:01 | 2021-09-01 12:31:01 | 89    |
| 3   | 1006 | 9002    | 2021-09-06 10:01:01 | 2021-09-06 10:21:01 | 81    |
| 4   | 1005 | 9002    | 2021-09-05 10:01:01 | 2021-09-05 10:21:01 | 81    |
| 5   | 1005 | 9001    | 2021-09-05 10:31:01 | 2021-09-05 10:51:01 | 81    |

Trong số những người từng làm bài thi ít nhất 2 ngày trong năm 2021, hãy tính window thời gian tối đa `days_window` của 2 lần làm bài thi liên tiếp trong năm đó, và theo quy luật lịch sử của năm đó thì trong `days_window` ngày anh ấy trung bình làm được bao nhiêu bộ đề thi, sắp xếp giảm dần theo window thời gian tối đa và số bộ đề làm được trung bình. Kết quả mẫu như sau:

| uid  | days_window | avg_exam_cnt |
| ---- | ----------- | ------------ |
| 1006 | 6           | 2.57         |

**Giải thích**: Người dùng 1006 lần lượt làm bài thi 3 lần vào các ngày 01/09/2021, 06/09/2021, 07/09/2021, window thời gian tối đa 2 lần làm bài liên tiếp là 6 ngày (ngày 1 đến ngày 6). Trong 7 ngày từ ngày 1 đến ngày 7 anh ấy làm tổng cộng 3 đề thi, trung bình mỗi ngày 3/7=0.428571 đề, vậy trong 6 ngày trung bình sẽ làm 0.428571 * 6 = 2.57 đề thi (giữ lại 2 chữ số thập phân); Người dùng 1005 làm 2 đề thi vào ngày 05/09/2021, nhưng chỉ có lịch sử làm bài trong 1 ngày duy nhất nên bị lọc đi.

**Tư duy giải đề:**

Trong lời giải thích trên có gợi ý cần loại bỏ trùng lặp bản ghi làm bài, tuyệt đối đừng bị lừa, không được DISTINCT! DISTINCT sẽ không pass test case. Chú ý giới hạn thời gian là năm 2021;

Đồng thời chú ý chênh lệch thời gian phải +1 ngày; Ngoài ra chú ý ==chưa nộp bài cũng được tính vào==!!!! (Nói chung cảm giác bài này mô tả không rõ ràng, đề bài ra chưa được hay lắm).

**Đáp án**:

```sql
SELECT UID,
       max(datediff(next_time, start_time)) + 1 AS days_window,
       round(count(start_time)/(datediff(max(start_time), min(start_time))+ 1) * (max(datediff(next_time, start_time))+ 1), 2) AS avg_exam_cnt
FROM
  (SELECT UID,
          start_time,
          lead(start_time, 1) OVER (PARTITION BY UID
                                    ORDER BY start_time) AS next_time
   FROM exam_record
   WHERE YEAR (start_time) = '2021' ) a
GROUP BY UID
HAVING count(DISTINCT date(start_time)) > 1
ORDER BY days_window DESC,
         avg_exam_cnt DESC
```

### Tình hình hoàn thành của người dùng có số bài chưa hoàn thành trong 3 tháng gần nhất bằng 0

**Mô tả**:

Bảng lịch sử làm bài thi `exam_record` (`uid`: ID người dùng, `exam_id`: ID đề thi, `start_time`: Thời gian bắt đầu làm bài, `submit_time`: Thời gian nộp bài, nếu rỗng biểu thị chưa hoàn thành, `score`: Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1006 | 9003    | 2021-09-06 10:01:01 | 2021-09-06 10:21:02 | 84     |
| 2   | 1006 | 9001    | 2021-08-02 12:11:01 | 2021-08-02 12:31:01 | 89     |
| 3   | 1006 | 9002    | 2021-06-06 10:01:01 | 2021-06-06 10:21:01 | 81     |
| 4   | 1006 | 9002    | 2021-05-06 10:01:01 | 2021-05-06 10:21:01 | 81     |
| 5   | 1006 | 9001    | 2021-05-01 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9001    | 2021-09-05 10:31:01 | 2021-09-05 10:51:01 | 81     |
| 7   | 1001 | 9003    | 2021-08-01 09:01:01 | 2021-08-01 09:51:11 | 78     |
| 8   | 1001 | 9002    | 2021-07-01 09:01:01 | 2021-07-01 09:31:00 | 81     |
| 9   | 1001 | 9002    | 2021-07-01 12:01:01 | 2021-07-01 12:31:01 | 81     |
| 10  | 1001 | 9002    | 2021-07-01 12:01:01 | (NULL)              | (NULL) |

Tìm số bài thi đã hoàn thành của người dùng mà trong 3 tháng gần nhất có lịch sử làm bài thi không có bài thi nào ở trạng thái chưa hoàn thành, xếp hạng giảm dần theo số bài thi hoàn thành và ID người dùng. Kết quả mẫu như sau:

| uid  | exam_complete_cnt |
| ---- | ----------------- |
| 1006 | 3                 |

**Giải thích**: Người dùng 1006 có 3 tháng gần nhất có lịch sử làm bài là 202109, 202108, 202106, số bài làm là 3, tất cả đều hoàn thành; Người dùng 1001 có 3 tháng gần nhất có lịch sử làm bài là 202109, 202108, 202107, số bài làm là 5, số bài hoàn thành là 4, vì có bài thi chưa hoàn thành nên bị lọc đi.

**Tư duy giải đề:**

1. "Tìm số bài thi đã hoàn thành của người dùng mà trong 3 tháng gần nhất có lịch sử làm bài thi không có bài thi nào ở trạng thái chưa hoàn thành", đầu tiên xem câu này, chắc chắn phải gom nhóm theo người dùng trước.
2. 3 tháng gần nhất, có thể áp dụng DENSE_RANK() sắp xếp giảm dần, ranking <= 3.
3. Thống kê số lượt làm bài.
4. Ghép các điều kiện còn lại.
5. Sắp xếp.

**Đáp án**:

```sql
SELECT UID,
       count(score) exam_complete_cnt
FROM
  (SELECT *, DENSE_RANK() OVER (PARTITION BY UID
                             ORDER BY date_format(start_time, '%Y%m') DESC) dr
   FROM exam_record) t1
WHERE dr <= 3
GROUP BY UID
HAVING count(dr)= count(score)
ORDER BY exam_complete_cnt DESC,
         UID DESC
```

### Tình hình làm bài 3 tháng gần nhất của 50% người dùng có tỉ lệ chưa hoàn thành cao hơn (Khó)

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name    | achievement | level | job  | register_time       |
| --- | ---- | ------------ | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号    | 3200        | 7     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号    | 2500        | 6     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 ♂ | 2200        | 5     | 算法 | 2020-01-01 10:00:00 |

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag    | difficulty | duration | release_time        |
| --- | ------- | ------ | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL    | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL    | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | 算法   | hard       | 80       | 2020-01-01 10:00:00 |
| 4   | 9004    | PYTHON | medium     | 70       | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 90    |
| 15  | 1002 | 9001    | 2020-01-01 18:01:01 | 2020-01-01 18:59:02 | 90    |
| 13  | 1001 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89    |
| 2   | 1002 | 9001    | 2020-01-20 10:01:01 |                     |       |
| 3   | 1002 | 9001    | 2020-02-01 12:11:01 |                     |       |
| 5   | 1001 | 9001    | 2020-03-01 12:01:01 |                     |       |
| 6   | 1002 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90    |
| 4   | 1003 | 9001    | 2020-03-01 19:01:01 |                     |       |
| 7   | 1002 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 90    |
| 14  | 1001 | 9002    | 2020-01-01 12:11:01 |                     |       |
| 8   | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69    |
| 9   | 1001 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99    |
| 10  | 1002 | 9002    | 2020-02-02 12:01:01 |                     |       |
| 11  | 1002 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:43:01 | 81    |
| 12  | 1002 | 9002    | 2020-03-02 12:11:01 |                     |       |
| 17  | 1001 | 9002    | 2020-05-05 18:01:01 |                     |       |
| 16  | 1002 | 9003    | 2020-05-06 12:01:01 |                     |       |

Hãy thống kê số lượng bài làm và số lượng bài hoàn thành hàng tháng trong 3 tháng gần nhất có lịch sử làm bài thi của những người dùng cấp 6 và 7 nằm trong nhóm 50% người dùng có tỉ lệ chưa hoàn thành bài thi SQL cao nhất. Sắp xếp tăng dần theo ID người dùng và tháng.

Kết quả mẫu như sau:

| uid  | start_month | total_cnt | complete_cnt |
| ---- | ----------- | --------- | ------------ |
| 1002 | 202002      | 3         | 1            |
| 1002 | 202003      | 2         | 1            |
| 1002 | 202005      | 2         | 1            |

Giải thích: Số bài chưa hoàn thành, tổng số bài làm, tỉ lệ chưa hoàn thành đối với đề thi SQL của từng người dùng như sau:

| uid  | incomplete_cnt | total_cnt | incomplete_rate |
| ---- | -------------- | --------- | --------------- |
| 1001 | 3              | 7         | 0.4286          |
| 1002 | 4              | 8         | 0.5000          |
| 1003 | 1              | 1         | 1.0000          |

1001, 1002, 1003 lần lượt xếp ở vị trí 1.0, 0.5, 0.0, do đó 50% người dùng cao hơn (vị trí ranking >= 0.5 trong PERCENT_RANK) là 1002, 1003;

1003 không phải người dùng cấp 6 hoặc cấp 7;

3 tháng gần nhất có lịch sử làm bài thi là 202005, 202003, 202002;

Trong 3 tháng này số bài 1002 làm lần lượt là 3, 2, 2, số bài hoàn thành lần lượt là 1, 1, 1.

**Tư duy giải đề:**

Điểm chú ý: Bài này chú ý tính tổng số lần làm bài và số lần hoàn thành của TẤT CẢ bài thi, còn thể loại đề thi SQL chỉ dùng để giới hạn thứ hạng tỉ lệ chưa hoàn thành, người dùng cấp 6, 7 dùng để giới hạn lịch sử làm bài.

Đầu tiên tính thứ hạng tỉ lệ chưa hoàn thành:

```sql
SELECT UID,
       count(submit_time IS NULL
             OR NULL)/ count(start_time) AS num,
       PERCENT_RANK() OVER (
                            ORDER BY count(submit_time IS NULL
                                           OR NULL)/ count(start_time)) AS ranking
FROM exam_record
LEFT JOIN examination_info USING (exam_id)
WHERE tag = 'SQL'
GROUP BY UID
```

Sau đó tính lịch sử làm bài trong 3 tháng gần nhất:

```sql
SELECT UID,
       date_format(start_time, '%Y%m') AS month_d,
       submit_time,
       exam_id,
       dense_rank() OVER (PARTITION BY UID
                          ORDER BY date_format(start_time, '%Y%m') DESC) AS ranking
FROM exam_record
LEFT JOIN user_info USING (UID)
WHERE LEVEL IN (6,7)
```

**Đáp án**:

```sql
SELECT t1.uid,
       t1.month_d,
       count(*) AS total_cnt,
       count(t1.submit_time) AS complete_cnt
FROM-- Đầu tiên tính thứ hạng tỉ lệ chưa hoàn thành

  (SELECT UID,
          count(submit_time IS NULL OR NULL)/ count(start_time) AS num,
          PERCENT_RANK() OVER (
                               ORDER BY count(submit_time IS NULL OR NULL)/ count(start_time)) AS ranking
   FROM exam_record
   LEFT JOIN examination_info USING (exam_id)
   WHERE tag = 'SQL'
   GROUP BY UID) t
INNER JOIN
  (-- Sau đó tính lịch sử làm bài trong 3 tháng gần nhất
  SELECT UID,
         date_format(start_time, '%Y%m') AS month_d,
         submit_time,
         exam_id,
         dense_rank() OVER (PARTITION BY UID
                            ORDER BY date_format(start_time, '%Y%m') DESC) AS ranking
   FROM exam_record
   LEFT JOIN user_info USING (UID)
   WHERE LEVEL IN (6,7) ) t1 USING (UID)
WHERE t1.ranking <= 3 AND t.ranking >= 0.5 -- Sử dụng giới hạn để tìm bản ghi thỏa mãn điều kiện

GROUP BY t1.uid,
         t1.month_d
ORDER BY t1.uid,
         t1.month_d
```

### Tốc độ tăng trưởng số bài thi hoàn thành so với cùng kỳ năm 2020和sự thay đổi thứ hạng (Khó)

**Mô tả**:

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag    | difficulty | duration | release_time        |
| --- | ------- | ------ | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL    | hard       | 60       | 2021-01-01 10:00:00 |
| 2   | 9002    | C++    | hard       | 80       | 2021-01-01 10:00:00 |
| 3   | 9003    | 算法   | hard       | 80       | 2021-01-01 10:00:00 |
| 4   | 9004    | PYTHON | medium     | 70       | 2021-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2020-08-02 10:01:01 | 2020-08-02 10:31:01 | 89    |
| 2   | 1002 | 9001    | 2020-04-01 18:01:01 | 2020-04-01 18:59:02 | 90    |
| 3   | 1001 | 9001    | 2020-04-01 09:01:01 | 2020-04-01 09:21:59 | 80    |
| 5   | 1002 | 9001    | 2021-03-02 19:01:01 | 2021-03-02 19:32:00 | 20    |
| 8   | 1003 | 9001    | 2021-05-02 12:01:01 | 2021-05-02 12:31:01 | 98    |
| 13  | 1003 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89    |
| 9   | 1001 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99    |
| 10  | 1002 | 9002    | 2021-02-02 12:01:01 | 2020-02-02 12:43:01 | 81    |
| 11  | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69    |
| 16  | 1002 | 9002    | 2020-02-02 12:01:01 |                     |       |
| 17  | 1002 | 9002    | 2020-03-02 12:11:01 |                     |       |
| 18  | 1001 | 9002    | 2021-05-05 18:01:01 |                     |       |
| 4   | 1002 | 9003    | 2021-01-20 10:01:01 | 2021-01-20 10:10:01 | 81    |
| 6   | 1001 | 9003    | 2021-04-02 19:01:01 | 2021-04-02 19:40:01 | 89    |
| 15  | 1002 | 9003    | 2021-01-01 18:01:01 | 2021-01-01 18:59:02 | 90    |
| 7   | 1004 | 9004    | 2020-05-02 12:01:01 | 2020-05-02 12:20:01 | 99    |
| 12  | 1001 | 9004    | 2021-09-02 12:11:01 |                     |       |
| 14  | 1002 | 9004    | 2020-01-01 12:11:01 | 2020-01-01 12:31:01 | 83    |

Hãy tính tốc độ tăng trưởng số lần hoàn thành các loại đề thi trong nửa đầu năm 2021 so với cùng kỳ nửa đầu năm 2020 (định dạng phần trăm, làm tròn 1 chữ số thập phân), cũng như sự thay đổi thứ hạng số lần hoàn thành, sắp xếp giảm dần theo tốc độ tăng trưởng và thứ hạng năm 2021.

Kết quả mẫu như sau:

| tag | exam_cnt_20 | exam_cnt_21 | growth_rate | exam_cnt_rank_20 | exam_cnt_rank_21 | rank_delta |
| --- | ----------- | ----------- | ----------- | ---------------- | ---------------- | ---------- |
| SQL | 3           | 2           | -33.3%      | 1                | 2                | 1          |

Giải thích: Nửa đầu năm 2020 có 3 tag có lịch sử hoàn thành bài thi là C++, SQL, PYTHON, số lần làm xong lần lượt là 3, 3, 2, thứ hạng số lần hoàn thành là 1, 1 (đồng hạng), 3;

Nửa đầu năm 2021 có 2 tag có lịch sử hoàn thành bài thi là Thuật toán (算法), SQL, số lần làm xong lần lượt là 3, 2, thứ hạng số lần hoàn thành là 1, 2; Cụ thể như sau:

| tag    | start_year | exam_cnt | exam_cnt_rank |
| ------ | ---------- | -------- | ------------- |
| C++    | 2020       | 3        | 1             |
| SQL    | 2020       | 3        | 1             |
| PYTHON | 2020       | 2        | 3             |
| 算法   | 2021       | 3        | 1             |
| SQL    | 2021       | 2        | 2             |

Do đó tag có thể output kết quả so với cùng kỳ chỉ có SQL, từ 2020 đến 2021 số lần hoàn thành 3 => 2, giảm 33.3% (làm tròn 1 chữ số thập phân); thứ hạng 1 => 2, giảm 1 hạng.

**Tư duy giải đề:**

Điểm khó bài này nằm ở kiểu dữ liệu long integer yêu cầu không tạo ra dấu âm khi trừ unsigned, dùng hàm CAST chuyển kiểu dữ liệu thành SIGNED.

Cũng như công thức tính tốc độ tăng trưởng: `(exam_cnt_21 - exam_cnt_20) / exam_cnt_20`

Sự thay đổi thứ hạng hoàn thành (So sánh năm 2021 với 2020 thứ hạng tăng hay giảm bao nhiêu)

Công thức tính: `exam_cnt_rank_21 - exam_cnt_rank_20`

Trong MySQL, hàm `CAST()` dùng để chuyển đổi kiểu dữ liệu của một biểu thức sang một kiểu dữ liệu khác. Cú pháp cơ bản như sau:

```sql
CAST(expression AS data_type)

-- Chuyển một chuỗi thành số nguyên
SELECT CAST('123' AS INT);
```

Ví dụ không nêu từng cái một nữa, hàm này rất đơn giản

**Đáp án**:

```sql
SELECT
  tag,
  exam_cnt_20,
  exam_cnt_21,
  concat(
    round(
      100 * (exam_cnt_21 - exam_cnt_20) / exam_cnt_20,
      1
    ),
    '%'
  ) AS growth_rate,
  exam_cnt_rank_20,
  exam_cnt_rank_21,
  cast(exam_cnt_rank_21 AS signed) - cast(exam_cnt_rank_20 AS signed) AS rank_delta
FROM
  (
    # Số lần hoàn thành và thứ hạng hoàn thành các loại đề thi nửa đầu năm 2020, 2021
    SELECT
      tag,
      count(
        IF (
          date_format(start_time, '%Y%m%d') BETWEEN '20200101'
          AND '20200630',
          start_time,
          NULL
        )
      ) AS exam_cnt_20,
      count(
        IF (
          substring(start_time, 1, 10) BETWEEN '2021-01-01'
          AND '2021-06-30',
          start_time,
          NULL
        )
      ) AS exam_cnt_21,
      rank() over (
        ORDER BY
          count(
            IF (
              date_format(start_time, '%Y%m%d') BETWEEN '20200101'
              AND '20200630',
              start_time,
              NULL
            )
          ) DESC
      ) AS exam_cnt_rank_20,
      rank() over (
        ORDER BY
          count(
            IF (
              substring(start_time, 1, 10) BETWEEN '2021-01-01'
              AND '2021-06-30',
              start_time,
              NULL
            )
          ) DESC
      ) AS exam_cnt_rank_21
    FROM
      examination_info
      JOIN exam_record USING (exam_id)
    WHERE
      submit_time IS NOT NULL
    GROUP BY
      tag
  ) main
WHERE
  exam_cnt_21 * exam_cnt_20 <> 0
ORDER BY
  growth_rate DESC,
  exam_cnt_rank_21 DESC
```

## Hàm Aggregation Window (Aggregate Window Functions)

### Chuẩn hóa Min-Max cho điểm thi đề thi

**Mô tả**:

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag    | difficulty | duration | release_time        |
| --- | ------- | ------ | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL    | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | C++    | hard       | 80       | 2020-01-01 10:00:00 |
| 3   | 9003    | 算法   | hard       | 80       | 2020-01-01 10:00:00 |
| 4   | 9004    | PYTHON | medium     | 70       | 2020-01-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 6   | 1003 | 9001    | 2020-01-02 12:01:01 | 2020-01-02 12:31:01 | 68     |
| 9   | 1001 | 9001    | 2020-01-02 10:01:01 | 2020-01-02 10:31:01 | 89     |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 90     |
| 12  | 1002 | 9002    | 2021-05-05 18:01:01 | (NULL)              | (NULL) |
| 3   | 1004 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:11:01 | 60     |
| 2   | 1003 | 9002    | 2020-01-01 19:01:01 | 2020-01-01 19:30:01 | 75     |
| 7   | 1001 | 9002    | 2020-01-02 12:01:01 | 2020-01-02 12:43:01 | 81     |
| 10  | 1002 | 9002    | 2020-01-01 12:11:01 | 2020-01-01 12:31:01 | 83     |
| 4   | 1003 | 9002    | 2020-01-01 12:01:01 | 2020-01-01 12:41:01 | 90     |
| 5   | 1002 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:32:00 | 90     |
| 11  | 1002 | 9004    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1001 | 9005    | 2020-01-02 12:11:01 | (NULL)              | (NULL) |

Trong tính toán dữ liệu vật lý và thống kê, có một khái niệm gọi là Chuẩn hóa Min-Max (Min-Max Normalization), còn gọi là Chuẩn hóa độ lệch (Deviation Normalization), là phép biến đổi tuyến tính trên dữ liệu gốc làm cho giá trị kết quả ánh xạ vào khoảng [0 - 1].

Hàm biến đổi là:

![](https://oss.javaguide.cn/github/javaguide/database/sql/29A377601170AB822322431FCDF7EDFE.png)

Hãy thực hiện Chuẩn hóa Min-Max cho điểm số của người dùng làm bài thi độ khó cao trong từng lịch sử làm bài thi, sau đó scale về khoảng [0, 100], và output ID người dùng, ID đề thi, điểm trung bình sau chuẩn hóa; Cuối cùng sắp xếp tăng dần theo ID đề thi, giảm dần theo điểm sau chuẩn hóa. (Lưu ý: Khoảng điểm số mặc định là [0, 100], nếu trong một lịch sử làm bài thi chỉ có duy nhất 1 điểm số thì không cần dùng công thức, điểm số sau khi chuẩn hóa và scale vẫn giữ nguyên là điểm ban đầu).

Kết quả mẫu như sau:

| uid  | exam_id | avg_new_score |
| ---- | ------- | ------------- |
| 1001 | 9001    | 98            |
| 1003 | 9001    | 0             |
| 1002 | 9002    | 88            |
| 1003 | 9002    | 75            |
| 1001 | 9002    | 70            |
| 1004 | 9002    | 0             |

Giải thích: Đề thi độ khó cao gồm 9001, 9002, 9003;

Lịch sử làm bài đề 9001 có 3 bản ghi, điểm số lần lượt là 68, 89, 90, sau khi chuẩn hóa theo công thức cho trước điểm số là: 0, 95, 100. 2 điểm sau đều do người dùng 1001 làm, do đó điểm mới của người dùng 1001 đối với đề thi 9001 là (95 + 100) / 2 ≈ 98 (chỉ lấy phần nguyên), điểm mới của người dùng 1003 đối với đề thi 9001 là 0. Kết quả cuối cùng sắp xếp tăng dần theo ID đề thi, giảm dần theo điểm chuẩn hóa.

**Tư duy giải đề:**

Điểm chú ý:

1. Đối với các đề thi độ khó cao, dựa theo điểm số mỗi loại đề thi, dùng hàm window MAX/MIN(col) OVER() để tìm giá trị lớn nhất và nhỏ nhất trong từng nhóm, sau đó tính theo công thức chuẩn hóa, scale khoảng là [0, 100], tức min_max * 100.
2. Nếu một loại đề thi chỉ có 1 điểm số, không cần dùng công thức chuẩn hóa, vì chỉ có 1 điểm số max_score = min_score, sau khi tính công thức kết quả có thể biến thành 0.
3. Kết quả cuối cùng gom nhóm theo uid, exam_id để tính giá trị trung bình sau chuẩn hóa, score là NULL cần phải lọc đi.

Cuối cùng là xem kỹ công thức trên (Nói thật bài này nhìn rất lắt léo)

**Đáp án**:

```sql
SELECT
  uid,
  exam_id,
  round(sum(min_max) / count(score), 0) AS avg_new_score
FROM
  (
    SELECT
      *,
      IF (
        max_score = min_score,
        score,
        (score - min_score) / (max_score - min_score) * 100
      ) AS min_max
    FROM
      (
        SELECT
          uid,
          a.exam_id,
          score,
          max(score) over (PARTITION BY a.exam_id) AS max_score,
          min(score) over (PARTITION BY a.exam_id) AS min_score
        FROM
          exam_record a
          LEFT JOIN examination_info b USING (exam_id)
        WHERE
          difficulty = 'hard'
      ) t
    WHERE
      score IS NOT NULL
  ) t1
GROUP BY
  uid,
  exam_id
ORDER BY
  exam_id ASC,
  avg_new_score DESC;
```

### Số lượt làm bài hàng tháng của mỗi đề thi和tổng số lượt làm bài tính đến tháng đó

**Mô tả:**

Bảng lịch sử làm bài thi exam_record (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 90     |
| 2   | 1002 | 9001    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 89     |
| 3   | 1002 | 9001    | 2020-02-01 12:11:01 | 2020-02-01 12:31:01 | 83     |
| 4   | 1003 | 9001    | 2020-03-01 19:01:01 | 2020-03-01 19:30:01 | 75     |
| 5   | 1004 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:11:01 | 60     |
| 6   | 1003 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90     |
| 7   | 1002 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 90     |
| 8   | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69     |
| 9   | 1004 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99     |
| 10  | 1003 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:31:01 | 68     |
| 11  | 1001 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:43:01 | 81     |
| 12  | 1001 | 9002    | 2020-03-02 12:11:01 | (NULL)              | (NULL) |

Hãy output số lượt làm bài hàng tháng của mỗi đề thi và tổng số lượt làm bài tính đến tháng đó.
Kết quả mẫu như sau:

| exam_id | start_month | month_cnt | cum_exam_cnt |
| ------- | ----------- | --------- | ------------ |
| 9001    | 202001      | 2         | 2            |
| 9001    | 202002      | 1         | 3            |
| 9001    | 202003      | 3         | 6            |
| 9001    | 202005      | 1         | 7            |
| 9002    | 202001      | 1         | 1            |
| 9002    | 202002      | 3         | 4            |
| 9002    | 202003      | 1         | 5            |

Giải thích: Đề thi 9001 có lịch sử làm bài trong 4 tháng 202001, 202002, 202003, 202005, số lượt làm bài mỗi tháng lần lượt là 2, 1, 3, 1, tổng số lượt làm bài lũy kế tính đến tháng đó lần lượt là 2, 3, 6, 7.

**Tư duy giải đề:**

Bài này chỉ có 2 điểm mấu chốt: thống kê tổng số lượt làm bài tính đến tháng đó, output số lượt làm bài hàng tháng của từng đề thi và tổng số lượt làm bài tính đến tháng đó.

Điểm mấu chốt nằm ở: `SUM(COUNT(*)) OVER (PARTITION BY exam_id ORDER BY DATE_FORMAT(start_time, '%Y%m'))`

**Đáp án**:

```sql
SELECT exam_id,
       date_format(start_time, '%Y%m') AS start_month,
       count(*) AS month_cnt,
       sum(count(*)) OVER (PARTITION BY exam_id
                           ORDER BY date_format(start_time, '%Y%m')) AS cum_exam_cnt
FROM exam_record
GROUP BY exam_id,
         start_month
```

### Tình hình làm bài hàng tháng和tính đến tháng đó (Khá khó)

**Mô tả**: Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-01 09:01:01 | 2020-01-01 09:21:59 | 90     |
| 2   | 1002 | 9001    | 2020-01-20 10:01:01 | 2020-01-20 10:10:01 | 89     |
| 3   | 1002 | 9001    | 2020-02-01 12:11:01 | 2020-02-01 12:31:01 | 83     |
| 4   | 1003 | 9001    | 2020-03-01 19:01:01 | 2020-03-01 19:30:01 | 75     |
| 5   | 1004 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:11:01 | 60     |
| 6   | 1003 | 9001    | 2020-03-01 12:01:01 | 2020-03-01 12:41:01 | 90     |
| 7   | 1002 | 9001    | 2020-05-02 19:01:01 | 2020-05-02 19:32:00 | 90     |
| 8   | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-01-02 19:59:01 | 69     |
| 9   | 1004 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:20:01 | 99     |
| 10  | 1003 | 9002    | 2020-02-02 12:01:01 | 2020-02-02 12:31:01 | 68     |
| 11  | 1001 | 9002    | 2020-01-02 19:01:01 | 2020-02-02 12:43:01 | 81     |
| 12  | 1001 | 9002    | 2020-03-02 12:11:01 | (NULL)              | (NULL) |

Hãy output số người dùng hoạt động hàng tháng (MAU), số người dùng mới, số người dùng mới tối đa trong một tháng tính đến tháng đó, số người dùng tích lũy tính đến tháng đó trong lịch sử làm bài thi hàng tháng kể từ khi có lịch sử làm bài của người dùng. Kết quả sắp xếp tăng dần theo tháng.

Kết quả mẫu như sau:

| start_month | mau | month_add_uv | max_month_add_uv | cum_sum_uv |
| ----------- | --- | ------------ | ---------------- | ---------- |
| 202001      | 2   | 2            | 2                | 2          |
| 202002      | 4   | 2            | 2                | 4          |
| 202003      | 3   | 0            | 2                | 4          |
| 202005      | 1   | 0            | 2                | 4          |

| month  | 1001 | 1002 | 1003 | 1004 |
| ------ | ---- | ---- | ---- | ---- |
| 202001 | 1    | 1    |      |      |
| 202002 | 1    | 1    | 1    | 1    |
| 202003 | 1    |      | 1    | 1    |
| 202005 |      | 1    |      |      |

Từ ma trận trên có thể thấy, tháng 1 năm 2020 có 2 người dùng hoạt động (mau=2), số người dùng mới trong tháng là 2;

Tháng 2 năm 2020 có 4 người dùng hoạt động, số người dùng mới trong tháng là 2, số người dùng mới tối đa trong một tháng tính đến thời điểm đó là 2, số người dùng tích lũy hiện tại là 4.

**Tư duy giải đề:**

Điểm khó:

1. Làm sao tính số người dùng mới hàng tháng

2. Tình hình làm bài tính đến tháng đó

Quy trình tổng thể:

(1) Thống kê tháng đăng nhập (làm bài) đầu tiên của từng người dùng `MIN()`

(2) Thống kê MAU và số người dùng mới hàng tháng: Lấy tháng đăng nhập đầu tiên của từng người trước, sau đó gom nhóm theo tháng đăng nhập đầu tiên để tính tổng số người mới của tháng đó

(3) Thống kê số người dùng mới tối đa trong một tháng tính đến tháng đó, số người dùng tích lũy tính đến tháng đó, cuối cùng output tăng dần theo tháng.

**Đáp án**:

```sql
-- Số người dùng mới tối đa trong một tháng tính đến tháng đó, số người dùng tích lũy tính đến tháng đó, output tăng dần theo tháng
SELECT
	start_month,
	mau,
	month_add_uv,
	max( month_add_uv ) over ( ORDER BY start_month ),
	sum( month_add_uv ) over ( ORDER BY start_month )
FROM
	(
	-- Thống kê MAU và số người dùng mới hàng tháng
	SELECT
		date_format( a.start_time, '%Y%m' ) AS start_month,
		count( DISTINCT a.uid ) AS mau,
		count( DISTINCT b.uid ) AS month_add_uv
	FROM
		exam_record a
		LEFT JOIN (
         -- Thống kê tháng đăng nhập đầu tiên của từng người
		SELECT uid, min( date_format( start_time, '%Y%m' )) AS first_month FROM exam_record GROUP BY uid ) b ON date_format( a.start_time, '%Y%m' ) = b.first_month
	GROUP BY
		start_month
	) main
ORDER BY
	start_month
```

<!-- @include: @article-footer.snippet.md -->
