---
title: SQL常见面试题总结（3）
description: SQL常见面试题总结第三篇，深入讲解聚合函数COUNT、SUM、AVG、MAX、MIN的使用，以及GROUP BY分组、HAVING过滤、截断平均值计算等进阶技巧。
category: 数据库
tag:
  - 数据库基础
  - SQL
head:
  - - meta
    - name: keywords
      content: SQL面试题,聚合函数,COUNT,SUM,AVG,MAX,MIN,GROUP BY,HAVING,截断平均值
---

> Các câu hỏi từ: [Niuke Tiba - Thử thách SQL nâng cao](https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=240)

Các câu hỏi thuộc mức Độ khó trung bình - cao hoặc Khó có thể căn cứ vào tình hình thực tế và nhu cầu phỏng vấn của bản thân để quyết định có nên bỏ qua hay không.

## Hàm Aggregation

### Giá trị trung bình cắt xén của điểm số đề thi SQL độ khó cao (Khá khó)

**Mô tả**: Bạn vận hành của Niuke muốn xem tình hình điểm số các đề thi độ khó cao thuộc thể loại SQL của mọi người.

Hãy giúp bạn ấy tính giá trị trung bình cắt xén (giá trị trung bình sau khi loại bỏ một giá trị lớn nhất và một giá trị nhỏ nhất) của điểm số tất cả người dùng hoàn thành đề thi độ khó cao thuộc thể loại SQL từ bảng dữ liệu `exam_record`.

Dữ liệu mẫu: `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành)

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | 算法 | medium     | 80       | 2020-08-02 10:00:00 |

Dữ liệu mẫu: `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số)

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | 2021-05-02 10:30:01 | 81     |
| 3   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:31:01 | 84     |
| 4   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 5   | 1001 | 9001    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 8   | 1002 | 9001    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 9   | 1003 | 9001    | 2021-09-07 12:01:01 | 2021-09-07 10:31:01 | 50     |
| 10  | 1004 | 9001    | 2021-09-06 10:01:01 | (NULL)              | (NULL) |

Kết quả truy vấn của bạn dựa trên đầu vào như sau:

| tag | difficulty | clip_avg_score |
| --- | ---------- | -------------- |
| SQL | hard       | 81.7           |

Từ bảng `examination_info` có thể thấy, đề thi 9001 là đề thi SQL độ khó cao, điểm số làm bài của đề thi này có [80,81,84,90,50], sau khi loại bỏ điểm cao nhất và điểm thấp nhất còn lại [80,81,84], điểm trung bình là 81.6666667, sau khi làm tròn lấy 1 chữ số thập phân là 81.7.

**Mô tả đầu vào:**

Dữ liệu đầu vào có ít nhất 3 điểm số hợp lệ.

**Tư duy 1:** Để tìm được đề thi SQL độ khó cao, chắc chắn cần JOIN với bảng examination_info, sau đó tìm các môn học độ khó cao. Từ examination_info ta biết exam_id của đề thi SQL độ khó cao là 9001, vậy tí nữa sẽ dùng `exam_id = 9001` làm điều kiện truy vấn;

Đầu tiên tìm kỳ thi số 9001: `select * from exam_record where exam_id = 9001`

Sau đó tìm điểm cao nhất: `select max(score) 最高分 from exam_record where exam_id = 9001`

Tiếp theo tìm điểm thấp nhất: `select min(score) 最低分 from exam_record where exam_id = 9001`

Trong tập kết quả điểm số truy vấn ra, loại bỏ điểm cao nhất và điểm thấp nhất, cách nghĩ trực quan nhất là dùng `NOT IN` hoặc `NOT EXISTS` đều được, ở đây dùng `NOT IN`.

Đầu tiên viết phần thân chính: `select tag, difficulty, round(avg(score), 1) clip_avg_score from examination_info info INNER JOIN exam_record record`

**Tip nhỏ**: Hàm `ROUND()` của MySQL, `ROUND(X)` trả về số nguyên gần nhất với tham số X; `ROUND(X, D)` trả về X với giá trị được giữ lại D chữ số sau dấu phẩy, cách làm tròn ở chữ số thứ D là làm tròn bốn bỏ súng tăng (round half up).

Sau đó ghép các câu lệnh "mảnh ghép" ở trên lại với nhau. Chú ý trong `NOT IN`, hai Subquery dùng `UNION ALL` để liên kết, dùng UNION tập hợp kết quả của MAX và MIN vào một nơi để tạo thành hiệu ứng 1 cột nhiều dòng.

**Đáp án 1:**

```sql
SELECT tag, difficulty, ROUND(AVG(score), 1) clip_avg_score
	FROM examination_info info  INNER JOIN exam_record record
		WHERE info.exam_id = record.exam_id
			AND  record.exam_id = 9001
				AND record.score NOT IN(
					SELECT MAX(score)
						FROM exam_record
							WHERE exam_id = 9001
								UNION ALL
					SELECT MIN(score)
						FROM exam_record
							WHERE exam_id = 9001
				)
```

Đây là cách giải trực quan nhất và dễ nghĩ đến nhất, nhưng vẫn còn có thể cải tiến, đây coi như là lách luật qua bài. Thực ra tuân thủ nghiêm ngặt yêu cầu đề bài thì nên viết như thế này:

```sql
SELECT tag,
       difficulty,
       ROUND(AVG(score), 1) clip_avg_score
FROM examination_info info
INNER JOIN exam_record record
WHERE info.exam_id = record.exam_id
  AND record.exam_id =
    (SELECT examination_info.exam_id
     FROM examination_info
     WHERE tag = 'SQL'
       AND difficulty = 'hard' )
  AND record.score NOT IN
    (SELECT MAX(score)
     FROM exam_record
     WHERE exam_id =
         (SELECT examination_info.exam_id
          FROM examination_info
          WHERE tag = 'SQL'
            AND difficulty = 'hard' )
     UNION ALL SELECT MIN(score)
     FROM exam_record
     WHERE exam_id =
         (SELECT examination_info.exam_id
          FROM examination_info
          WHERE tag = 'SQL'
            AND difficulty = 'hard' ) )
```

Tuy nhiên bạn sẽ phát hiện ra các câu lệnh lặp lại rất nhiều, do đó có thể lợi dụng mệnh đề `WITH` để trích xuất phần dùng chung.

**Giới thiệu mệnh đề `WITH`**:

Mệnh đề `WITH`, còn gọi là Common Table Expression (CTE - Biểu thức bảng chung), là cách định nghĩa bảng tạm trong truy vấn SQL. Nó cho phép chúng ta tạo một tập kết quả tạm thời có đặt tên trong truy vấn và có thể tham chiếu đến tập kết quả đó ngay trong cùng một câu truy vấn.

Cách dùng cơ bản:

```sql
WITH cte_name (column1, column2, ..., columnN) AS (
    -- Thân truy vấn
    SELECT ...
    FROM ...
    WHERE ...
)
-- Truy vấn chính
SELECT ...
FROM cte_name
WHERE ...
```

Mệnh đề `WITH` bao gồm các phần sau:

- `cte_name`: Đặt tên cho bảng tạm, có thể tham chiếu trong truy vấn chính.
- `(column1, column2, ..., columnN)`: Tùy chọn, chỉ định tên column của bảng tạm.
- `AS`: Bắt buộc, biểu thị bắt đầu định nghĩa bảng tạm.
- `Thân truy vấn CTE`: Câu lệnh truy vấn thực tế, dùng để định nghĩa dữ liệu trong bảng tạm.

Một trong những công dụng chính của mệnh đề `WITH` là tăng cường tính dễ đọc và tính dễ bảo trì của truy vấn, đặc biệt là khi liên quan đến nhiều Subquery lồng nhau hoặc cần tái sử dụng cùng một logic truy vấn. Bằng cách đưa các logic này vào một bảng tạm có tên, chúng ta có thể tổ chức truy vấn rõ ràng hơn và loại bỏ code trùng lặp.

Ngoài ra, mệnh đề `WITH` còn có thể thực hiện truy vấn đệ quy (Recursive Query) trong các truy vấn phức tạp. Truy vấn đệ quy cho phép chúng ta thực thi nhiều vòng lặp trên cùng một bảng trong một câu truy vấn duy nhất, từng bước xây dựng tập kết quả. Điều này rất hữu ích trong các kịch bản xử lý dữ liệu cấu trúc phân cấp, cấu trúc tổ chức và cấu trúc cây.

**Chi tiết nhỏ**: Phiên bản MySQL 5.7 trở về trước không hỗ trợ sử dụng Alias trực tiếp trong mệnh đề `WITH`.

Dưới đây là đáp án sau khi cải tiến:

```sql
WITH t1 AS
  (SELECT record.*,
          info.tag,
          info.difficulty
   FROM exam_record record
   INNER JOIN examination_info info ON record.exam_id = info.exam_id
   WHERE info.tag = "SQL"
     AND info.difficulty = "hard" )
SELECT tag,
       difficulty,
       ROUND(AVG(score), 1)
FROM t1
WHERE score NOT IN
    (SELECT max(score)
     FROM t1
     UNION SELECT min(score)
     FROM t1)
```

**Tư duy 2:**

- Lọc đề thi SQL độ khó cao: `WHERE tag="SQL" AND difficulty="hard"`
- Tính giá trị trung bình cắt xén: `(Tổng - Giá trị lớn nhất - Giá trị nhỏ nhất) / (Tổng số lượng - 2)`:
  - `(sum(score) - max(score) - min(score)) / (count(score) - 2)`
  - Nhược điểm là nếu giá trị lớn nhất và nhỏ nhất có nhiều giá trị trùng nhau, phương pháp này khó lọc chính xác ra được. Tuy nhiên đề bài đã nói -----> **`loại bỏ MỘT giá trị lớn nhất và MỘT giá trị nhỏ nhất rồi tính trung bình`**, do đó ở đây có thể dùng công thức này.

**Đáp án 2:**

```sql
SELECT info.tag,
       info.difficulty,
       ROUND((SUM(record.score)- MIN(record.score)- MAX(record.score)) / (COUNT(record.score)- 2), 1) AS clip_avg_score
FROM examination_info info,
     exam_record record
WHERE info.exam_id = record.exam_id
  AND info.tag = "SQL"
  AND info.difficulty = "hard";
```

### Thống kê số lượt làm bài

Có một bảng lịch sử làm bài thi `exam_record`, hãy thống kê tổng số lượt làm bài `total_pv`, số lượt làm bài đã hoàn thành `complete_pv`, số bài thi đã hoàn thành `complete_exam_cnt`.

Dữ liệu mẫu `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1001 | 9001    | 2021-05-02 10:01:01 | 2021-05-02 10:30:01 | 81     |
| 3   | 1001 | 9001    | 2021-06-02 19:01:01 | 2021-06-02 19:31:01 | 84     |
| 4   | 1001 | 9002    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 5   | 1001 | 9001    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 6   | 1001 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 7   | 1002 | 9002    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 8   | 1002 | 9001    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 9   | 1003 | 9001    | 2021-09-07 12:01:01 | 2021-09-07 10:31:01 | 50     |
| 10  | 1004 | 9001    | 2021-09-06 10:01:01 | (NULL)              | (NULL) |

Kết quả mẫu:

| total_pv | complete_pv | complete_exam_cnt |
| -------- | ----------- | ----------------- |
| 10       | 7           | 2                 |

Giải thích: Biểu thị tính đến hiện tại có 10 lượt làm bài thi, số lượt làm bài đã hoàn thành là 7 lượt (thoát giữa chừng là trạng thái chưa hoàn thành, thời gian nộp bài và điểm số là NULL), các bài thi đã hoàn thành gồm 2 bài là 9001 và 9002.

**Tư duy giải đề**: Bài này vừa nhìn thấy thống kê số lượt, chắc chắn đầu tiên nghĩ ngay đến dùng hàm `COUNT` để giải quyết. Vấn đề là muốn thống kê các bản ghi khác nhau thì viết như thế nào? Sử dụng Subquery có thể giải quyết được bài này (bài này dùng CASE WHEN cũng viết được, cách giải tương tự chỉ khác logic); Đầu tiên trước khi làm bài này, hãy cùng tìm hiểu cách dùng cơ bản của `COUNT`;

Cú pháp cơ bản của hàm COUNT() như sau:

```sql
COUNT(expression)
```

Trong đó, `expression` có thể là tên column, biểu thức, hằng số hoặc ký tự đại diện (wildcard). Dưới đây là một số ví dụ cách dùng thường gặp:

1. Tính số lượng tất cả các dòng trong bảng:

```sql
SELECT COUNT(*) FROM table_name;
```

2. Tính số lượng giá trị không rỗng (khác NULL) của một column cụ thể:

```sql
SELECT COUNT(column_name) FROM table_name;
```

3. Tính số dòng thỏa mãn điều kiện:

```sql
SELECT COUNT(*) FROM table_name WHERE condition;
```

4. Kết hợp với `GROUP BY` để tính số dòng của mỗi nhóm sau khi gom nhóm:

```sql
SELECT column_name, COUNT(*) FROM table_name GROUP BY column_name;
```

5. Tính số lượng kết hợp duy nhất của các column khác nhau:

```sql
SELECT COUNT(DISTINCT column_name1, column_name2) FROM table_name;
```

Khi sử dụng hàm `COUNT()`, nếu không chỉ định tham số nào hoặc sử dụng `COUNT(*)`, nó sẽ tính số lượng của tất cả các dòng. Còn nếu sử dụng tên column, nó chỉ tính số lượng các giá trị không rỗng của column đó.

Ngoài ra, kết quả của hàm `COUNT()` là một giá trị số nguyên. Cho dù kết quả là 0 cũng sẽ không trả về NULL, điểm này cần ghi nhớ.

**Đáp án**:

```sql
SELECT
	count(*) total_pv,
	( SELECT count(*) FROM exam_record WHERE submit_time IS NOT NULL ) complete_pv,
	( SELECT COUNT( DISTINCT exam_id, score IS NOT NULL OR NULL ) FROM exam_record ) complete_exam_cnt
FROM
	exam_record
```

Ở đây nói kỹ một chút về câu `COUNT(DISTINCT exam_id, score IS NOT NULL OR NULL)`: Kiểm tra score có phải NULL hay không, nếu không phải NULL thì là true, nếu là NULL thì trả về null; Chú ý ở đây nếu không thêm `OR NULL`, trong trường hợp không phải NULL nó chỉ trả về false (tức là 0);

Bản thân `COUNT` không thể tính số dòng trên nhiều column, sự tham gia của `DISTINCT` khiến nhiều column trở thành một thể thống nhất để tính số dòng xuất hiện; `COUNT DISTINCT` khi tính toán chỉ trả về các dòng không NULL, điểm này cũng cần lưu ý;

Ngoài ra qua bài này thu hoạch được cú pháp thường dùng khi COUNT kèm điều kiện -----> `COUNT(điều_kiện_cột OR NULL)`

### Điểm thấp nhất trong số các điểm không nhỏ hơn điểm trung bình

**Mô tả**: Hãy tìm điểm số thấp nhất của người dùng có điểm thi đề SQL không nhỏ hơn điểm trung bình của thể loại đề thi đó từ bảng lịch sử làm bài thi.

Dữ liệu mẫu `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2020-01-02 09:01:01 | 2020-01-02 09:21:01 | 80     |
| 2   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 89     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 4   | 1002 | 9003    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1002 | 9001    | 2021-02-02 19:01:01 | 2021-02-02 19:30:01 | 87     |
| 6   | 1002 | 9002    | 2021-05-05 18:01:01 | 2021-05-05 18:59:02 | 90     |
| 7   | 1003 | 9002    | 2021-02-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 9   | 1004 | 9003    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |

`examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành)

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL  | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2020-08-02 10:00:00 |

Dữ liệu đầu ra mẫu:

| min_score_over_avg |
| ------------------ |
| 87                 |

**Giải thích**: Đề thi 9001 và 9002 thuộc thể loại SQL, điểm làm hai bài thi này có [80,89,87,90], điểm trung bình là 86.5, điểm nhỏ nhất không nhỏ hơn điểm trung bình là 87.

**Tư duy giải đề**: Loại bài tập này yêu cầu đưa ra nhìn có vẻ rất "lắt léo", nhưng thực tế chải chuốt kỹ lại một lần, tách điều kiện lớn thành các điều kiện nhỏ, sau khi tách xong từng cái thì ghép tất cả các điều kiện lại. Chỉ cần nhớ: **Nắm thân chính, gỡ nhánh nhỏ**, vấn đề sẽ tự giải quyết được.

Thứ nhất: Tìm điểm số đề thi ==SQL==

Thứ hai: ==Điểm trung bình== của loại đề thi đó

Thứ ba: ==Điểm thấp nhất của người dùng== đối với loại đề thi đó

Sau đó "cầu nối" ở giữa chính là ==không nhỏ hơn==

Sau khi chia nhỏ điều kiện, từng bước hoàn thành:

```sql
-- Tìm điểm số có tag là 'SQL' [80, 89, 87, 90]
-- Sau đó tính điểm trung bình của nhóm này
select  ROUND(AVG(score), 1) from  examination_info info INNER JOIN exam_record record
	where info.exam_id = record.exam_id
	and tag= 'SQL'
```

Sau đó lại tìm điểm thấp nhất của loại đề thi đó, rồi lấy tập kết quả `[80, 89, 87, 90]` so sánh với điểm trung bình mới ra được đáp án cuối cùng.

**Đáp án**:

```sql
SELECT MIN(score) AS min_score_over_avg
FROM examination_info info
INNER JOIN exam_record record
WHERE info.exam_id = record.exam_id
  AND tag= 'SQL'
  AND score >=
    (SELECT ROUND(AVG(score), 1)
     FROM examination_info info
     INNER JOIN exam_record record
     WHERE info.exam_id = record.exam_id
       AND tag= 'SQL' )
```

## Truy vấn gom nhóm (GROUP BY)

### Số ngày hoạt động trung bình và số người hoạt động hàng tháng (MAU)

**Mô tả**: Lịch sử làm bài thi của người dùng ở khu vực làm bài Niuke được lưu trữ trong bảng `exam_record`, nội dung như sau:

`exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số)

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | 2021-07-02 09:21:01 | 80     |
| 2   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 81     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 4   | 1002 | 9003    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1002 | 9001    | 2021-07-02 19:01:01 | 2021-07-02 19:30:01 | 82     |
| 6   | 1002 | 9002    | 2021-07-05 18:01:01 | 2021-07-05 18:59:02 | 90     |
| 7   | 1003 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 9   | 1004 | 9003    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 10  | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 11  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 12  | 1006 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |
| 13  | 1007 | 9002    | 2020-09-02 12:11:01 | 2020-09-02 12:31:01 | 89     |

Hãy tính số ngày hoạt động trung bình hàng tháng `avg_active_days` và số người hoạt động hàng tháng `mau` của người dùng ở khu vực làm bài thi trong từng tháng năm 2021, kết quả mẫu như sau:

| month  | avg_active_days | mau |
| ------ | --------------- | --- |
| 202107 | 1.50            | 2   |
| 202109 | 1.25            | 4   |

**Giải thích**: Tháng 7 năm 2021 có 2 người hoạt động, tổng cộng hoạt động 3 ngày (1001 hoạt động 1 ngày, 1002 hoạt động 2 ngày), số ngày hoạt động trung bình là 1.5; Tháng 9 năm 2021 có 4 người hoạt động, tổng cộng hoạt động 5 ngày, số ngày hoạt động trung bình là 1.25, kết quả giữ lại 2 chữ số thập phân.

Lưu ý: Hoạt động ở đây chỉ hành vi ==nộp bài==.

**Tư duy giải đề**: Đọc xong đề bài đầu tiên chú ý phần highlight; Thông thường tính số ngày và số người hoạt động hàng tháng nghĩ ngay đến các hàm date liên quan; Bài này chúng ta cũng tiến hành chia nhỏ, làm chi tiết từng vấn đề rồi giải quyết. Đầu tiên tìm số người hoạt động chắc chắn cần dùng `COUNT()`, ở đây có một cái "bẫy", không biết mọi người có chú ý không? Người dùng 1002 làm 2 bài thi khác nhau trong tháng 9, do đó ở đây cần chú ý loại bỏ trùng lặp (distinct), nếu không khi thống kê số người hoạt động sẽ bị sai; Thứ 2 là phải biết định dạng date, như bảng trên, đề bài yêu cầu hiển thị định dạng date `202107`, cần dùng `DATE_FORMAT` để định dạng.

Cách dùng cơ bản:

`DATE_FORMAT(date_value, format)`

- `date_value` Tham số là giá trị date hoặc datetime cần định dạng.
- `format` Tham số là định dạng date hoặc datetime chỉ định.

**Đáp án**:

```sql
SELECT DATE_FORMAT(submit_time, '%Y%m') MONTH,
                                        round(count(DISTINCT UID, DATE_FORMAT(submit_time, '%Y%m%d')) / count(DISTINCT UID), 2) avg_active_days,
                                        COUNT(DISTINCT UID) mau
FROM exam_record
WHERE YEAR (submit_time) = 2021
GROUP BY MONTH
```

Nói thêm một câu ở đây, sử dụng `COUNT(DISTINCT uid, DATE_FORMAT(submit_time, '%Y%m%d'))` có thể thống kê số lượng giá trị kết hợp giữa column `uid` và column `submit_time` sau khi định dạng theo năm, tháng, ngày.

### Tổng số bài làm hàng tháng和số bài làm trung bình hàng ngày

**Mô tả**: Hiện có một bảng lịch sử làm bài tập `practice_record`, nội dung mẫu như sau:

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1003 | 8002        | 2021-08-01 19:38:01 | 80    |

Hãy thống kê tổng số bài làm hàng tháng `month_q_cnt` và số bài làm trung bình hàng ngày `avg_day_q_cnt` của người dùng trong từng tháng năm 2021 (sắp xếp tăng dần theo tháng) cũng như tình hình tổng thể của năm đó, kết quả mẫu như sau:

| submit_month | month_q_cnt | avg_day_q_cnt |
| ------------ | ----------- | ------------- |
| 202108       | 2           | 0.065         |
| 202109       | 3           | 0.100         |
| 2021 汇总    | 5           | 0.161         |

**Giải thích**: Tháng 8 năm 2021 có tổng cộng 2 lượt làm bài, số bài làm trung bình hàng ngày là 2/31=0.065 (giữ lại 3 chữ số thập phân); Tháng 9 năm 2021 có tổng cộng 3 lượt làm bài, số bài làm trung bình hàng ngày là 3/30=0.100; Năm 2021 có tổng cộng 5 lượt làm bài (tổng hợp trung bình năm không có ý nghĩa thực tế, ở đây chúng ta tính theo 31 ngày 5/31=0.161)

> Niuke đã áp dụng phiên bản MySQL mới nhất, nếu kết quả chạy của bạn xuất hiện lỗi: ONLY_FULL_GROUP_BY, có nghĩa là: Đối với thao tác Aggregation GROUP BY, nếu column trong SELECT không xuất hiện trong GROUP BY, câu lệnh SQL đó sẽ không hợp lệ, vì column không nằm trong mệnh đề GROUP BY, tức là column truy vấn ra phải xuất hiện sau GROUP BY nếu không sẽ bị lỗi, hoặc field này xuất hiện bên trong hàm Aggregation.

**Tư duy giải đề:**

Vừa nhìn thấy dữ liệu mẫu phải nghĩ ngay đến các hàm liên quan, ví dụ `submit_month` phải dùng `DATE_FORMAT` để định dạng date. Sau đó truy vấn số lượng làm bài mỗi tháng.

Số lượng làm bài mỗi tháng:

```sql
SELECT MONTH ( submit_time ), COUNT( question_id )
FROM
	practice_record
GROUP BY
	MONTH (submit_time)
```

Tiếp theo ở column thứ 3 cần dùng hàm `DAY(LAST_DAY(date_value))` để tìm số ngày trong tháng của date chỉ định.

Code ví dụ như sau:

```sql
SELECT DAY(LAST_DAY('2023-07-08')) AS days_in_month;
-- Đầu ra: 31

SELECT DAY(LAST_DAY('2023-02-01')) AS days_in_month;
-- Đầu ra: 28 (tháng 2 năm nhuận)

SELECT DAY(LAST_DAY(NOW())) AS days_in_current_month;
-- Đầu ra: 31 (số ngày tháng hiện tại)
```

Sử dụng hàm `LAST_DAY()` để lấy ngày cuối cùng của tháng đó, sau đó dùng hàm `DAY()` trích xuất số ngày của date đó. Như vậy sẽ có được số ngày của tháng chỉ định.

Cần lưu ý rằng hàm `LAST_DAY()` trả về giá trị date, còn hàm `DAY()` dùng để trích xuất phần số ngày trong giá trị date.

Sau khi có phân tích trên, lập tức có thể viết đáp án, bài này phức tạp ở chỗ xử lý date, logic bên trong không khó.

**Đáp án**:

```sql
SELECT DATE_FORMAT(submit_time, '%Y%m') submit_month,
       count(question_id) month_q_cnt,
       ROUND(COUNT(question_id) / DAY (LAST_DAY(submit_time)), 3) avg_day_q_cnt
FROM practice_record
WHERE DATE_FORMAT(submit_time, '%Y') = '2021'
GROUP BY submit_month
UNION ALL
SELECT '2021汇总' AS submit_month,
       count(question_id) month_q_cnt,
       ROUND(COUNT(question_id) / 31, 3) avg_day_q_cnt
FROM practice_record
WHERE DATE_FORMAT(submit_time, '%Y') = '2021'
ORDER BY submit_month
```

Trong đầu ra dữ liệu mẫu vì dòng cuối cùng cần đưa ra dữ liệu tổng hợp (summary), do đó ở đây dùng `UNION ALL` cộng vào tập kết quả; Đừng quên cuối cùng phải sắp xếp!

### Người dùng hợp lệ có số bài thi chưa hoàn thành lớn hơn 1 (Khá khó)

**Mô tả**: Hiện có bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số), dữ liệu mẫu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | 2021-07-02 09:21:01 | 80     |
| 2   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 81     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | (NULL)              | (NULL) |
| 4   | 1002 | 9003    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 5   | 1002 | 9001    | 2021-07-02 19:01:01 | 2021-07-02 19:30:01 | 82     |
| 6   | 1002 | 9002    | 2021-07-05 18:01:01 | 2021-07-05 18:59:02 | 90     |
| 7   | 1003 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 8   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 9   | 1004 | 9003    | 2021-09-06 12:01:01 | (NULL)              | (NULL) |
| 10  | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81     |
| 11  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 12  | 1006 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |
| 13  | 1007 | 9002    | 2020-09-02 12:11:01 | 2020-09-02 12:31:01 | 89     |

Còn có một bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành), dữ liệu mẫu như sau:

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | SQL  | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2020-08-02 10:00:00 |

Hãy thống kê dữ liệu của các người dùng hợp lệ có số bài thi chưa hoàn thành lớn hơn 1 trong năm 2021 (người dùng hợp lệ chỉ người dùng có số bài thi đã hoàn thành ít nhất là 1 và số bài thi chưa hoàn thành nhỏ hơn 5), output ID người dùng, số bài thi chưa hoàn thành, số bài thi đã hoàn thành, tập hợp tag các bài thi đã làm, sắp xếp theo số lượng bài thi chưa hoàn thành từ nhiều đến ít. Kết quả mẫu như sau:

| uid  | incomplete_cnt | complete_cnt | detail                                                                      |
| ---- | -------------- | ------------ | --------------------------------------------------------------------------- |
| 1002 | 2              | 4            | 2021-09-01:算法;2021-07-02:SQL;2021-09-02:SQL;2021-09-05:SQL;2021-07-05:SQL |

**Giải thích**: Trong lịch sử làm bài năm 2021, trừ 1004 ra, tất cả người dùng khác đều thỏa mãn định nghĩa người dùng hợp lệ, nhưng chỉ có 1002 có số bài thi chưa hoàn thành lớn hơn 1, do đó chỉ output 1002, trong detail là tập hợp {Ngày:tag} các bài thi 1002 đã làm, giữa Ngày và Tag dùng dấu **:** để nối, giữa nhiều phần tử dùng dấu **;** để nối.

**Tư duy giải đề:**

Đọc kỹ đề xong phân tích ra: Đầu tiên phải JOIN bảng, vì lát nữa cần output `tag`;

Lọc ra dữ liệu năm 2021:

```sql
SELECT *
FROM exam_record er
LEFT JOIN examination_info ei ON er.exam_id = ei.exam_id
WHERE YEAR (er.start_time)= 2021
```

Gom nhóm theo uid, sau đó tiến hành kiểm tra điều kiện đối với từng người dùng, đề bài yêu cầu `số bài thi hoàn thành ít nhất là 1, số bài thi chưa hoàn thành lớn hơn 1, nhỏ hơn 5`

Vậy lát nữa điều kiện khi viết SQL sẽ là: `chưa_hoàn_thành > 1 AND đã_hoàn_thành >= 1 AND chưa_hoàn_thành < 5`

Vì cuối cùng cần nối chuỗi, đồng thời còn phải kết hợp nối chuỗi, cái này có thể dùng hàm `GROUP_CONCAT`, dưới đây giới thiệu ngắn gọn cách dùng hàm này:

Định dạng cơ bản:

```sql
GROUP_CONCAT([DISTINCT] expr [ORDER BY {unsigned_integer | col_name | expr} [ASC | DESC] [, ...]]             [SEPARATOR sep])
```

- `expr`: Column hoặc biểu thức cần nối.
- `DISTINCT`: Tham số tùy chọn, dùng để loại bỏ trùng lặp. Khi chỉ định `DISTINCT`, giá trị giống nhau chỉ xuất hiện 1 lần.
- `ORDER BY`: Tham số tùy chọn, dùng để sắp xếp các giá trị sau khi nối. Có thể chọn tăng dần (`ASC`) hoặc giảm dần (`DESC`).
- `SEPARATOR sep`: Tham số tùy chọn, dùng để thiết lập dấu phân cách của các giá trị sau khi nối. (Bài này dùng tham số này đặt dấu `;`)

Hàm `GROUP_CONCAT()` thường dùng trong mệnh đề `GROUP BY`, nối các giá trị của một nhóm dòng thành một chuỗi và trả về dưới dạng tổng hợp trong tập kết quả.

**Đáp án**:

```sql
SELECT a.uid,
       SUM(CASE
               WHEN a.submit_time IS NULL THEN 1
           END) AS incomplete_cnt,
       SUM(CASE
               WHEN a.submit_time IS NOT NULL THEN 1
           END) AS complete_cnt,
       GROUP_CONCAT(DISTINCT CONCAT(DATE_FORMAT(a.start_time, '%Y-%m-%d'), ':', b.tag)
                    ORDER BY start_time SEPARATOR ";") AS detail
FROM exam_record a
LEFT JOIN examination_info b ON a.exam_id = b.exam_id
WHERE YEAR (a.start_time)= 2021
GROUP BY a.uid
HAVING incomplete_cnt > 1
AND complete_cnt >= 1
AND incomplete_cnt < 5
ORDER BY incomplete_cnt DESC
```

- `SUM(CASE WHEN a.submit_time IS NULL THEN 1 END)` thống kê số lượng bản ghi chưa hoàn thành của từng người dùng.
- `SUM(CASE WHEN a.submit_time IS NOT NULL THEN 1 END)` thống kê số lượng bản ghi đã hoàn thành của từng người dùng.
- `GROUP_CONCAT(DISTINCT CONCAT(DATE_FORMAT(a.start_time, '%Y-%m-%d'), ':', b.tag) ORDER BY a.start_time SEPARATOR ';')` nối ngày thi và tag của từng người dùng thành một chuỗi với dấu phân cách chỉ định, và sắp xếp theo thời gian bắt đầu thi.

## Subquery lồng nhau (Nested Subquery)

### Thể loại bài thi yêu thích của những người dùng có số bài thi hoàn thành trung bình hàng tháng từ 3 trở lên (Khá khó)

**Mô tả**: Hiện có bảng lịch sử làm bài thi `exam_record` (`uid`: ID người dùng, `exam_id`: ID đề thi, `start_time`: Thời gian bắt đầu làm bài, `submit_time`: Thời gian nộp bài, nếu chưa nộp là NULL, `score`: Điểm số), dữ liệu mẫu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | (NULL)              | (NULL) |
| 2   | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:21:01 | 60     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | 2021-09-02 12:31:01 | 70     |
| 4   | 1002 | 9001    | 2021-09-05 19:01:01 | 2021-09-05 19:40:01 | 81     |
| 5   | 1002 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 6   | 1003 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 7   | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 8   | 1003 | 9001    | 2021-09-08 13:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9002    | 2021-09-08 14:01:01 | (NULL)              | (NULL) |
| 10  | 1003 | 9003    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |
| 11  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 12  | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 13  | 1005 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |

Bảng thông tin đề thi `examination_info` (`exam_id`: ID đề thi, `tag`: Thể loại đề thi, `difficulty`: Độ khó đề thi, `duration`: Thời lượng thi, `release_time`: Thời gian phát hành), dữ liệu mẫu như sau:

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2020-01-01 10:00:00 |
| 2   | 9002    | C++  | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2020-08-02 10:00:00 |

Hãy thống kê các thể loại bài thi yêu thích và số lần làm bài của những người dùng có "số bài thi hoàn thành trung bình hàng tháng" từ 3 trở lên, output giảm dần theo số lần làm bài, kết quả mẫu như sau:

| tag  | tag_cnt |
| ---- | ------- |
| C++  | 4       |
| SQL  | 2       |
| 算法 | 1       |

**Giải thích**: Người dùng 1002 và 1005 đều có số bài thi hoàn thành trong tháng 09 năm 2021 là 3, các người dùng khác đều nhỏ hơn 3; Sau đó phân bố tag đề thi mà người dùng 1002 và 1005 từng làm sắp xếp giảm dần theo số lần làm bài lần lượt là C++, SQL, Thuật toán (算法).

**Tư duy giải đề**: Bài này kiểm tra Subquery kết hợp, trọng tâm nằm ở `trung bình tháng >= 3`, nhưng cá nhân tôi cho rằng ở đây diễn đạt chưa rõ ràng, nên nói trực tiếp là tìm tháng 9 thì dễ hiểu hơn nhiều; Ở đây không phải tháng nào cũng phải >= 3 hoặc là tổng số lần/tổng số tháng làm bài. Đừng hiểu nhầm.

Đầu tiên truy vấn người dùng nào có số lượt trả lời bài tập trung bình hàng tháng lớn hơn hoặc bằng 3:

```sql
SELECT UID
FROM exam_record record
GROUP BY UID,
         MONTH (start_time)
HAVING count(submit_time) >= 3
```

Sau khi có bước này rồi mới tiến hành đi sâu hơn, chỉ cần hiểu được bước trên (ý tôi là không bị làm phiền bởi chữ "trung bình tháng" trong đề bài), rồi lồng một Subquery để tìm người dùng nào nằm trong đó, sau đó truy vấn các column cần thiết trong đề bài là được. Nhớ sắp xếp!!

```sql
SELECT tag,
       count(start_time) AS tag_cnt
FROM exam_record record
INNER JOIN examination_info info ON record.exam_id = info.exam_id
WHERE UID IN
    (SELECT UID
     FROM exam_record record
     GROUP BY UID,
              MONTH (start_time)
     HAVING count(submit_time) >= 3)
GROUP BY tag
ORDER BY tag_cnt DESC
```

### Số người làm bài和điểm trung bình trong ngày phát hành đề thi

**Mô tả**: Hiện có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký), dữ liệu mẫu như sau:

| id  | uid  | nick_name | achievement | level | job  | register_time       |
| --- | ---- | --------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号 | 3100        | 7     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号 | 2100        | 6     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 | 1500        | 5     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号 | 1100        | 4     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 5 号 | 1600        | 6     | C++  | 2020-01-01 10:00:00 |
| 6   | 1006 | 牛客 6 号 | 3000        | 6     | C++  | 2020-01-01 10:00:00 |

**Giải nghĩa**: Người dùng 1001 có biệt danh Niuke 1, điểm thành tựu 3100, cấp độ người dùng 7, định hướng nghề nghiệp là Thuật toán (算法), thời gian đăng ký 2020-01-01 10:00:00.

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành) Dữ liệu mẫu như sau:

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++  | easy       | 60       | 2020-02-01 10:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2020-08-02 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số) Dữ liệu mẫu như sau:

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-07-02 09:01:01 | 2021-09-01 09:41:01 | 70     |
| 2   | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:21:01 | 60     |
| 3   | 1002 | 9002    | 2021-09-02 12:01:01 | 2021-09-02 12:31:01 | 70     |
| 4   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 80     |
| 5   | 1002 | 9003    | 2021-08-01 12:01:01 | 2021-08-01 12:21:01 | 60     |
| 6   | 1002 | 9002    | 2021-08-02 12:01:01 | 2021-08-02 12:31:01 | 70     |
| 7   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 85     |
| 8   | 1002 | 9002    | 2021-07-06 12:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9002    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 10  | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 11  | 1003 | 9003    | 2021-09-01 13:01:01 | 2021-09-01 13:41:01 | 70     |
| 12  | 1003 | 9001    | 2021-09-08 14:01:01 | (NULL)              | (NULL) |
| 13  | 1003 | 9002    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 90     |
| 15  | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 16  | 1005 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |

Hãy tính số người làm bài `uv` và điểm trung bình `avg_score` của người dùng từ cấp 5 trở lên trong ngày phát hành của mỗi đề thi thể loại SQL, sắp xếp giảm dần theo số người, nếu trùng số người thì sắp xếp tăng dần theo điểm trung bình, kết quả mẫu như sau:

| exam_id | uv  | avg_score |
| ------- | --- | --------- |
| 9001    | 3   | 81.3      |

Giải thích: Chỉ có một đề thi thể loại SQL với ID là 9001, trong ngày phát hành (2021-09-01) có 1001, 1002, 1003, 1005 từng làm bài, nhưng 1003 là người dùng cấp 5, 3 người còn lại từ cấp 5 trở lên. Điểm số của 3 người này có [70, 80, 85, 90], điểm trung bình là 81.3 (giữ lại 1 chữ số thập phân).

**Tư duy giải đề**: Bài này nhìn có vẻ rất phức tạp, nhưng trước tiên hãy từng bước chia nhỏ điều kiện "bên ngoài", sau đó gộp lại với nhau thì đáp án sẽ ra. Thao tác truy vấn nhiều bảng luôn nhớ: từ ngoài vào trong, bóc tách từng lớp.

Đầu tiên JOIN 3 bảng lại với nhau, đồng thời gán một số điều kiện, ví dụ đề bài yêu cầu người dùng có `level > 5`, vậy có thể truy vấn trước:

```sql
SELECT DISTINCT u_info.uid
FROM examination_info e_info
INNER JOIN exam_record record
INNER JOIN user_info u_info
WHERE e_info.exam_id = record.exam_id
  AND u_info.uid = record.uid
  AND u_info.LEVEL > 5
```

Tiếp theo chú ý yêu cầu đề bài: `Người dùng làm bài vào trong ngày sau khi đề thi thể loại SQL được phát hành`, chú ý chữ ==trong ngày==, lúc này ngay lập tức nghĩ đến việc so sánh thời gian.

So sánh ngày phát hành đề thi với ngày bắt đầu thi: `DATE(e_info.release_time) = DATE(record.start_time)`; Không cần lo lắng vấn đề `submit_time` bị NULL, phần sau trong WHERE sẽ lọc đi.

**Đáp án**:

```sql
SELECT record.exam_id AS exam_id,
       COUNT(DISTINCT u_info.uid) AS uv,
       ROUND(SUM(record.score) / COUNT(u_info.uid), 1) AS avg_score
FROM examination_info e_info
INNER JOIN exam_record record
INNER JOIN user_info u_info
WHERE e_info.exam_id = record.exam_id
  AND u_info.uid = record.uid
  AND DATE (e_info.release_time) = DATE (record.start_time)
  AND submit_time IS NOT NULL
  AND tag = 'SQL'
  AND u_info.LEVEL > 5
GROUP BY record.exam_id
ORDER BY uv DESC,
         avg_score ASC
```

Chú ý phần gom nhóm và sắp xếp cuối cùng! Sắp xếp theo số người trước, nếu trùng nhau thì sắp xếp theo điểm trung bình.

### Phân bố cấp độ người dùng của những người có điểm thi lớn hơn 80

**Mô tả**:

Hiện có bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name | achievement | level | job  | register_time       |
| --- | ---- | --------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号 | 3100        | 7     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号 | 2100        | 6     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 | 1500        | 5     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号 | 1100        | 4     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 5 号 | 1600        | 6     | C++  | 2020-01-01 10:00:00 |
| 6   | 1006 | 牛客 6 号 | 3000        | 6     | C++  | 2020-01-01 10:00:00 |

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++  | easy       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2021-09-01 10:00:00 |

Bảng thông tin làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:41:01 | 79     |
| 2   | 1002 | 9003    | 2021-09-01 12:01:01 | 2021-09-01 12:21:01 | 60     |
| 3   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 4   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 80     |
| 5   | 1002 | 9003    | 2021-08-01 12:01:01 | 2021-08-01 12:21:01 | 60     |
| 6   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 7   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 85     |
| 8   | 1002 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |
| 9   | 1003 | 9002    | 2021-09-07 10:01:01 | 2021-09-07 10:31:01 | 86     |
| 10  | 1003 | 9003    | 2021-09-08 12:01:01 | 2021-09-08 12:11:01 | 40     |
| 11  | 1003 | 9003    | 2021-09-01 13:01:01 | 2021-09-01 13:41:01 | 81     |
| 12  | 1003 | 9001    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |
| 13  | 1003 | 9002    | 2021-09-08 15:01:01 | (NULL)              | (NULL) |
| 14  | 1005 | 9001    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 90     |
| 15  | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 88     |
| 16  | 1005 | 9002    | 2021-09-02 12:11:01 | 2021-09-02 12:31:01 | 89     |

Thống kê phân bố cấp độ người dùng của những người làm bài thi thể loại SQL có điểm số lớn hơn 80, sắp xếp giảm dần theo số lượng (đảm bảo số lượng đều khác nhau). Kết quả mẫu như sau:

| level | level_cnt |
| ----- | --------- |
| 6     | 2         |
| 5     | 1         |

Giải thích: 9001 là đề thi thể loại SQL, những người làm bài thi này có điểm lớn hơn 80 gồm 1002, 1003, 1005 tổng cộng 3 người, trong đó cấp 6 có 2 người, cấp 5 có 1 người.

**Tư duy giải đề:** Bài này và bài trước đều dùng chung dữ liệu, chỉ có điều kiện truy vấn thay đổi mà thôi, bài trước đã hiểu rồi thì bài này làm cực kỳ nhanh.

**Đáp án**:

```sql
SELECT u_info.LEVEL AS LEVEL,
       count(u_info.uid) AS level_cnt
FROM examination_info e_info
INNER JOIN exam_record record
INNER JOIN user_info u_info
WHERE e_info.exam_id = record.exam_id
  AND u_info.uid = record.uid
  AND record.score > 80
  AND submit_time IS NOT NULL
  AND tag = 'SQL'
GROUP BY LEVEL
ORDER BY level_cnt DESC
```

## Truy vấn hợp nhất (UNION)

### Số người và số lượt làm bài của từng bài tập和từng đề thi

**Mô tả**:

Bảng lịch sử làm bài thi exam_record (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:41:01 | 81     |
| 2   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 3   | 1002 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 80     |
| 4   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 5   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 85     |
| 6   | 1002 | 9002    | 2021-09-01 12:01:01 | (NULL)              | (NULL) |

Bảng luyện tập bài tập practice_record (`uid` ID người dùng, `question_id` ID bài tập, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1003 | 8001        | 2021-08-02 19:38:01 | 70    |
| 6   | 1003 | 8001        | 2021-08-02 19:48:01 | 90    |
| 7   | 1003 | 8002        | 2021-08-01 19:38:01 | 80    |

Hãy thống kê số người và số lượt làm bài của từng bài tập và từng đề thi, hiển thị giảm dần theo uv & pv của "Đề thi" và "Bài tập", kết quả mẫu như sau:

| tid  | uv  | pv  |
| ---- | --- | --- |
| 9001 | 3   | 3   |
| 9002 | 1   | 3   |
| 8001 | 3   | 5   |
| 8002 | 2   | 2   |

**Giải thích**: “Đề thi” có 3 người luyện tập tổng cộng 3 lần đề thi 9001, 1 người làm bài 3 lần đề 9002; “Bài tập” có 3 người làm 5 lần bài 8001, 2 người làm 2 lần bài 8002.

**Tư duy giải đề**: Điểm khó và điểm dễ sai của bài này là vấn đề sử dụng đồng thời `UNION` và `ORDER BY`.

Có các trường hợp sau: Sử dụng `UNION` và nhiều `ORDER BY` không thêm ngoặc đơn -> Báo lỗi!

`ORDER BY` trong câu lệnh con liên kết bởi `UNION` không có tác dụng;

Chẳng hạn không thêm ngoặc đơn:

```sql
SELECT exam_id AS tid,
       COUNT(DISTINCT UID) AS uv,
       COUNT(UID) AS pv
FROM exam_record
GROUP BY exam_id
ORDER BY uv DESC,
         pv DESC
UNION
SELECT question_id AS tid,
       COUNT(DISTINCT UID) AS uv,
       COUNT(UID) AS pv
FROM practice_record
GROUP BY question_id
ORDER BY uv DESC,
         pv DESC
```

Báo lỗi cú pháp trực tiếp, nếu không có ngoặc đơn chỉ được phép có 1 `ORDER BY`.

Cũng có một trường hợp `ORDER BY` không phát huy tác dụng, nhưng có thể phát huy tác dụng trong sub-query của câu lệnh con, giải pháp ở đây là lồng thêm một lớp query bên ngoài.

**Đáp án**:

```sql
SELECT *
FROM
  (SELECT exam_id AS tid,
          COUNT(DISTINCT exam_record.uid) uv,
          COUNT(*) pv
   FROM exam_record
   GROUP BY exam_id
   ORDER BY uv DESC, pv DESC) t1
UNION
SELECT *
FROM
  (SELECT question_id AS tid,
          COUNT(DISTINCT practice_record.uid) uv,
          COUNT(*) pv
   FROM practice_record
   GROUP BY question_id
   ORDER BY uv DESC, pv DESC) t2;
```

### Những người lần lượt thỏa mãn 2 hoạt động

**Mô tả**: Để thúc đẩy nhiều người dùng học tập và làm bài tiến bộ trên nền tảng Niuke, chúng ta thường phát ưu đãi cho một số người dùng vừa hoạt động tốt vừa có thành tích tốt. Giả sử trước đây chúng ta có 2 đợt hoạt động vận hành: lần lượt phát voucher ưu đãi cho những người mỗi lần làm bài thi đều đạt 85 điểm (activity1), và những người có ít nhất 1 lần chỉ dùng một nửa thời gian đã hoàn thành đề thi độ khó cao và điểm lớn hơn 80 (activity2).

Bây giờ cần bạn lọc ra những người thỏa mãn hai hoạt động này một lần duy nhất để đưa cho bạn vận hành. Hãy viết một câu lệnh SQL thực hiện: Output ID người dùng và số hiệu hoạt động của tất cả những người mà mỗi lần làm bài thi đều đạt 85 điểm trở lên cũng như những người có ít nhất 1 lần dùng một nửa thời gian hoàn thành đề thi độ khó cao và điểm lớn hơn 80 trong năm 2021, sắp xếp theo ID người dùng.

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++  | easy       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2021-09-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score  |
| --- | ---- | ------- | ------------------- | ------------------- | ------ |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 2   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 70     |
| 3   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | **86** |
| 4   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 89     |
| 5   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |

Dữ liệu đầu ra mẫu:

| uid  | activity  |
| ---- | --------- |
| 1001 | activity2 |
| 1003 | activity1 |
| 1004 | activity1 |
| 1004 | activity2 |

**Giải thích**: Người dùng 1001 điểm nhỏ nhất là 81 không thỏa mãn hoạt động 1, nhưng hoàn thành đề thi dài 60 phút trong 29 phút 59 giây đạt 81 điểm, thỏa mãn hoạt động 2; 1003 điểm nhỏ nhất là 86 thỏa mãn hoạt động 1, thời gian hoàn thành đều lớn hơn một nửa thời lượng đề thi, không thỏa mãn hoạt động 2; Người dùng 1004 vừa vặn dùng đúng một nửa thời gian (đúng 30 phút) hoàn thành đề thi đạt 85 điểm, thỏa mãn cả hoạt động 1 và 2.

**Tư duy giải đề**: Bài này liên quan đến phép trừ thời gian, cần dùng hàm `TIMESTAMPDIFF()` để tính chênh lệch số phút giữa hai timestamp.

Dưới đây chúng ta xem cách dùng cơ bản

Ví dụ:

```sql
TIMESTAMPDIFF(MINUTE, start_time, end_time)
```

Tham số đầu tiên của hàm `TIMESTAMPDIFF()` là đơn vị thời gian, ở đây chúng ta chọn `MINUTE` biểu thị trả về chênh lệch phút. Tham số thứ 2 là timestamp sớm hơn, tham số thứ 3 là timestamp muộn hơn. Hàm sẽ trả về chênh lệch phút giữa chúng.

Sau khi nắm được cách dùng hàm này, quay lại xem yêu cầu của `activity1`: tìm điểm lớn hơn 85 là được, vậy chúng ta cứ viết phần này ra trước, logic về sau sẽ rõ ràng hơn nhiều:

```sql
SELECT DISTINCT UID
FROM exam_record
WHERE score >= 85
  AND YEAR (start_time) = '2021'
```

Dựa theo điều kiện 2, tiếp tục viết `người hoàn thành đề thi độ khó cao trong một nửa thời gian và điểm lớn hơn 80`:

```sql
SELECT UID
FROM examination_info info
INNER JOIN exam_record record
WHERE info.exam_id = record.exam_id
  AND (TIMESTAMPDIFF(MINUTE, start_time, submit_time)) < (info.duration / 2)
  AND difficulty = 'hard'
  AND score >= 80
```

Sau đó đem cả 2 `UNION` lại với nhau là xong. (Ở đây đặc biệt chú ý vấn đề ngoặc đơn và vị trí `ORDER BY`, cách dùng cụ thể đã đề cập ở bài trước).

**Đáp án**:

```sql
SELECT DISTINCT UID UID,
                    'activity1' activity
FROM exam_record
WHERE UID not in
    (SELECT UID
     FROM exam_record
     WHERE score<85
       AND YEAR(submit_time) = 2021 )
UNION
SELECT DISTINCT UID UID,
                    'activity2' activity
FROM exam_record e_r
LEFT JOIN examination_info e_i ON e_r.exam_id = e_i.exam_id
WHERE YEAR(submit_time) = 2021
  AND difficulty = 'hard'
  AND TIMESTAMPDIFF(SECOND, start_time, submit_time) <= duration *30
  AND score>80
ORDER BY UID
```

## Truy vấn JOIN (JOIN Query)

### Số bài thi hoàn thành和số bài tập luyện tập của người dùng thỏa mãn điều kiện (Khó)

**Mô tả**:

Bảng thông tin người dùng user_info (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name | achievement | level | job  | register_time       |
| --- | ---- | --------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号 | 3100        | 7     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号 | 2300        | 7     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 | 2500        | 7     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号 | 1200        | 5     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 5 号 | 1600        | 6     | C++  | 2020-01-01 10:00:00 |
| 6   | 1006 | 牛客 6 号 | 2000        | 6     | C++  | 2020-01-01 10:00:00 |

Bảng thông tin đề thi examination_info (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++  | hard       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2021-09-01 10:00:00 |

Bảng lịch sử làm bài thi exam_record (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | exam_id | start_time          | submit_time         | score |
| --- | ---- | ------- | ------------------- | ------------------- | ----- |
| 1   | 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81    |
| 2   | 1002 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:01 | 81    |
| 3   | 1003 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:40:01 | 86    |
| 4   | 1003 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:51 | 89    |
| 5   | 1004 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85    |
| 6   | 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85    |
| 7   | 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 84    |
| 8   | 1006 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 80    |

Bảng lịch sử làm bài tập practice_record (`uid` ID người dùng, `question_id` ID bài tập, `submit_time` Thời gian nộp bài, `score` Điểm số):

| id  | uid  | question_id | submit_time         | score |
| --- | ---- | ----------- | ------------------- | ----- |
| 1   | 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 2   | 1002 | 8001        | 2021-09-02 19:30:01 | 50    |
| 3   | 1002 | 8001        | 2021-09-02 19:20:01 | 70    |
| 4   | 1002 | 8002        | 2021-09-02 19:38:01 | 70    |
| 5   | 1004 | 8001        | 2021-08-02 19:38:01 | 70    |
| 6   | 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 7   | 1001 | 8002        | 2021-08-02 19:38:01 | 70    |
| 8   | 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 9   | 1004 | 8002        | 2021-08-02 19:58:01 | 94    |
| 10  | 1004 | 8003        | 2021-08-02 19:38:01 | 70    |
| 11  | 1004 | 8003        | 2021-08-02 19:48:01 | 90    |
| 12  | 1004 | 8003        | 2021-08-01 19:38:01 | 80    |

Hãy tìm các cao thủ cấp 7 có điểm trung bình bài thi SQL độ khó cao lớn hơn 80, thống kê tổng số lần hoàn thành bài thi và tổng số lần luyện tập bài tập năm 2021 của họ, chỉ giữ lại người dùng có lịch sử hoàn thành bài thi năm 2021. Kết quả sắp xếp tăng dần theo số bài thi hoàn thành, giảm dần theo số bài tập luyện tập.

Dữ liệu đầu ra mẫu như sau:

| uid  | exam_cnt | question_cnt |
| ---- | -------- | ------------ |
| 1001 | 1        | 2            |
| 1003 | 2        | 0            |

Giải thích: Người dùng 1001, 1003, 1004, 1006 thỏa mãn điểm trung bình bài thi SQL độ khó cao lớn hơn 80, nhưng chỉ có 1001, 1003 là cao thủ cấp 7; 1001 hoàn thành 1 lần đề thi 9001, luyện tập 2 lần bài tập; 1003 hoàn thành 2 lần đề thi 9001, 9002, chưa luyện tập bài tập nào (do đó đếm là 0).

**Tư duy giải đề:**

Đầu tiên lọc điều kiện sơ bộ, ví dụ truy vấn trước những người dùng từng làm đề thi SQL độ khó cao:

```sql
SELECT
	record.uid
FROM
	exam_record record
	INNER JOIN examination_info e_info ON record.exam_id = e_info.exam_id
	JOIN user_info u_info ON record.uid = u_info.uid
WHERE
	e_info.tag = 'SQL'
	AND e_info.difficulty = 'hard'
```

Sau đó dựa theo yêu cầu đề bài, tiếp tục lồng các điều kiện vào bên trong;

Nhưng ở đây lại cần lưu ý:

Thứ 1: Không thể đặt điều kiện `YEAR(submit_time) = 2021` xuống cuối cùng, mà phải đặt trong điều kiện `ON`, vì LEFT JOIN tồn tại trường hợp trả về toàn bộ dòng của bảng bên trái, bảng bên phải là NULL, đặt trong mệnh đề `ON` của `JOIN` là để đảm bảo khi kết nối 2 bảng chỉ những bản ghi thỏa mãn điều kiện năm mới được kết nối. Như vậy có thể tránh được việc các bản ghi năm khác bị bao gồm trong kết quả. Tức 1001 từng làm bài thi năm 2021 nhưng chưa từng làm bài tập, nếu đặt điều kiện ở cuối cùng sẽ loại bỏ mất trường hợp này.

Thứ 2: Bắt buộc phải dùng `COUNT(DISTINCT er.exam_id) exam_cnt, COUNT(DISTINCT pr.id) question_cnt`, phải thêm DISTINCT vì LEFT JOIN sinh ra rất nhiều giá trị trùng lặp.

**Đáp án**:

```sql
SELECT er.uid AS UID,
       count(DISTINCT er.exam_id) AS exam_cnt,
       count(DISTINCT pr.id) AS question_cnt
FROM exam_record er
LEFT JOIN practice_record pr ON er.uid = pr.uid
AND YEAR (er.submit_time)= 2021
AND YEAR (pr.submit_time)= 2021
WHERE er.uid IN
    (SELECT er.uid
     FROM exam_record er
     LEFT JOIN examination_info ei ON er.exam_id = ei.exam_id
     LEFT JOIN user_info ui ON er.uid = ui.uid
     WHERE tag = 'SQL'
       AND difficulty = 'hard'
       AND LEVEL = 7
     GROUP BY er.uid
     HAVING avg(score) > 80)
GROUP BY er.uid
ORDER BY exam_cnt,
         question_cnt DESC
```

Có thể những bạn cẩn thận sẽ phát hiện ra: Tại sao rõ ràng đã giới hạn điều kiện `tag = 'SQL' AND difficulty = 'hard'`, nhưng người dùng 1003 vẫn có thể truy vấn ra 2 bản ghi bài thi, trong đó 1 bài thi có `tag` là `C++`? Điều này là do đặc tính của `LEFT JOIN`, cho dù không có dòng khớp ở bảng bên phải, tất cả các bản ghi ở bảng bên trái vẫn sẽ được giữ lại.

### Tình hình hoạt động của từng người dùng cấp 6/7 (Khó)

**Mô tả**:

Bảng thông tin người dùng `user_info` (`uid` ID người dùng, `nick_name` Biệt danh, `achievement` Điểm thành tựu, `level` Cấp độ, `job` Định hướng nghề nghiệp, `register_time` Thời gian đăng ký):

| id  | uid  | nick_name | achievement | level | job  | register_time       |
| --- | ---- | --------- | ----------- | ----- | ---- | ------------------- |
| 1   | 1001 | 牛客 1 号 | 3100        | 7     | 算法 | 2020-01-01 10:00:00 |
| 2   | 1002 | 牛客 2 号 | 2300        | 7     | 算法 | 2020-01-01 10:00:00 |
| 3   | 1003 | 牛客 3 号 | 2500        | 7     | 算法 | 2020-01-01 10:00:00 |
| 4   | 1004 | 牛客 4 号 | 1200        | 5     | 算法 | 2020-01-01 10:00:00 |
| 5   | 1005 | 牛客 5 号 | 1600        | 6     | C++  | 2020-01-01 10:00:00 |
| 6   | 1006 | 牛客 6 号 | 2600        | 7     | C++  | 2020-01-01 10:00:00 |

Bảng thông tin đề thi `examination_info` (`exam_id` ID đề thi, `tag` Thể loại đề thi, `difficulty` Độ khó đề thi, `duration` Thời lượng thi, `release_time` Thời gian phát hành):

| id  | exam_id | tag  | difficulty | duration | release_time        |
| --- | ------- | ---- | ---------- | -------- | ------------------- |
| 1   | 9001    | SQL  | hard       | 60       | 2021-09-01 06:00:00 |
| 2   | 9002    | C++  | easy       | 60       | 2021-09-01 06:00:00 |
| 3   | 9003    | 算法 | medium     | 80       | 2021-09-01 10:00:00 |

Bảng lịch sử làm bài thi `exam_record` (`uid` ID người dùng, `exam_id` ID đề thi, `start_time` Thời gian bắt đầu làm bài, `submit_time` Thời gian nộp bài, `score` Điểm số):

| uid  | exam_id | start_time          | submit_time         | score  |
| ---- | ------- | ------------------- | ------------------- | ------ |
| 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 78     |
| 1001 | 9001    | 2021-09-01 09:01:01 | 2021-09-01 09:31:00 | 81     |
| 1005 | 9001    | 2021-09-01 19:01:01 | 2021-09-01 19:30:01 | 85     |
| 1005 | 9002    | 2021-09-01 12:01:01 | 2021-09-01 12:31:02 | 85     |
| 1006 | 9003    | 2021-09-07 10:01:01 | 2021-09-07 10:21:59 | 84     |
| 1006 | 9001    | 2021-09-07 10:01:01 | 2021-09-07 10:21:01 | 81     |
| 1002 | 9001    | 2020-09-01 13:01:01 | 2020-09-01 13:41:01 | 81     |
| 1005 | 9001    | 2021-09-01 14:01:01 | (NULL)              | (NULL) |

Bảng lịch sử làm bài tập `practice_record` (`uid` ID người dùng, `question_id` ID bài tập, `submit_time` Thời gian nộp bài, `score` Điểm số):

| uid  | question_id | submit_time         | score |
| ---- | ----------- | ------------------- | ----- |
| 1001 | 8001        | 2021-08-02 11:41:01 | 60    |
| 1004 | 8001        | 2021-08-02 19:38:01 | 70    |
| 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 1001 | 8002        | 2021-08-02 19:38:01 | 70    |
| 1004 | 8002        | 2021-08-02 19:48:01 | 90    |
| 1006 | 8002        | 2021-08-04 19:58:01 | 94    |
| 1006 | 8003        | 2021-08-03 19:38:01 | 70    |
| 1006 | 8003        | 2021-08-02 19:48:01 | 90    |
| 1006 | 8003        | 2020-08-01 19:38:01 | 80    |

Hãy thống kê tổng số tháng hoạt động, số ngày hoạt động năm 2021, số ngày hoạt động ở khu vực làm bài thi năm 2021, số ngày hoạt động ở khu vực làm bài tập năm 2021 của từng người dùng cấp 6/7, sắp xếp giảm dần theo tổng số tháng hoạt động và số ngày hoạt động năm 2021. Kết quả mẫu như sau:

| uid  | act_month_total | act_days_2021 | act_days_2021_exam |
| ---- | --------------- | ------------- | ------------------ |
| 1006 | 3               | 4             | 1                  |
| 1001 | 2               | 2             | 1                  |
| 1005 | 1               | 1             | 1                  |
| 1002 | 1               | 0             | 0                  |
| 1003 | 0               | 0             | 0                  |

**Giải thích**: Người dùng cấp 6/7 tổng cộng có 5 người, trong đó 1006 hoạt động trong 3 tháng 202109, 202108, 202008; Ngày hoạt động trong năm 2021 có 4 ngày là 20210907, 20210804, 20210803, 20210802; Năm 2021 ở khu vực làm bài thi 20210907 hoạt động 1 ngày, ở khu vực làm bài tập hoạt động 3 ngày.

**Tư duy giải đề:**

Mấu chốt của bài này nằm ở việc sử dụng `CASE WHEN THEN`, nếu không sẽ phải viết rất nhiều `LEFT JOIN` vì sinh ra nhiều tập kết quả.

Mệnh đề `CASE WHEN THEN` là một biểu thức điều kiện dùng để thực thi các thao tác khác nhau hoặc trả về các kết quả khác nhau tùy theo điều kiện trong SQL.

Cấu trúc cú pháp như sau:

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE result
END
```

Trong cấu trúc này, có thể thêm nhiều mệnh đề `WHEN` tùy theo nhu cầu, mỗi `WHEN` đi kèm một điều kiện (condition) và một kết quả (result). Điều kiện có thể là bất kỳ biểu thức logic nào, nếu thỏa mãn điều kiện sẽ trả về kết quả tương ứng.

Mệnh đề `ELSE` ở cuối là tùy chọn, dùng để chỉ định kết quả trả về mặc định khi tất cả các điều kiện phía trước đều không thỏa mãn. Nếu không cung cấp mệnh đề `ELSE`, mặc định trả về `NULL`.

Ví dụ:

```sql
SELECT score,
    CASE
        WHEN score >= 90 THEN 'Giỏi'
        WHEN score >= 80 THEN 'Khá'
        WHEN score >= 60 THEN 'Trung bình'
        ELSE 'Yếu'
    END AS grade
FROM student_scores;
```

Trong ví dụ trên, dựa vào phạm vi điểm số (score) khác nhau của học sinh, sử dụng câu lệnh CASE WHEN THEN để trả về xếp loại (grade) tương ứng. Nếu điểm >= 90 trả về "Giỏi"; >= 80 trả về "Khá"; >= 60 trả về "Trung bình"; ngược lại trả về "Yếu".

Sau khi đã nắm được cách dùng ở trên, quay lại xem bài này, yêu cầu liệt kê các số ngày hoạt động khác nhau:

```sql
count(distinct act_month) as act_month_total,
count(distinct case when year(act_time)='2021'then act_day end) as act_days_2021,
count(distinct case when year(act_time)='2021' and tag='exam' then act_day end) as act_days_2021_exam,
count(distinct case when year(act_time)='2021' and tag='question'then act_day end) as act_days_2021_question
```

`tag` ở đây được gán nhãn trước để thuận tiện phân biệt khi truy vấn, phân tách giữa làm bài thi và làm bài tập.

Tìm người dùng khu vực làm bài thi:

```sql
SELECT
		uid,
		exam_id AS ans_id,
		start_time AS act_time,
		date_format( start_time, '%Y%m' ) AS act_month,
		date_format( start_time, '%Y%m%d' ) AS act_day,
		'exam' AS tag
	FROM
		exam_record
```

Ngay sau đó là người dùng khu vực làm bài tập:

```sql
SELECT
		uid,
		question_id AS ans_id,
		submit_time AS act_time,
		date_format( submit_time, '%Y%m' ) AS act_month,
		date_format( submit_time, '%Y%m%d' ) AS act_day,
		'question' AS tag
	FROM
		practice_record
```

Cuối cùng đem 2 kết quả `UNION` lại với nhau, đừng quên sắp xếp kết quả (bài này hơi giống tư tưởng Chia để trị).

**Đáp án**:

```sql
SELECT user_info.uid,
       count(DISTINCT act_month) AS act_month_total,
       count(DISTINCT CASE
                          WHEN YEAR (act_time)= '2021' THEN act_day
                      END) AS act_days_2021,
       count(DISTINCT CASE
                          WHEN YEAR (act_time)= '2021'
                               AND tag = 'exam' THEN act_day
                      END) AS act_days_2021_exam,
       count(DISTINCT CASE
                          WHEN YEAR (act_time)= '2021'
                               AND tag = 'question' THEN act_day
                      END) AS act_days_2021_question
FROM
  (SELECT UID,
          exam_id AS ans_id,
          start_time AS act_time,
          date_format(start_time, '%Y%m') AS act_month,
          date_format(start_time, '%Y%m%d') AS act_day,
          'exam' AS tag
   FROM exam_record
   UNION ALL SELECT UID,
                    question_id AS ans_id,
                    submit_time AS act_time,
                    date_format(submit_time, '%Y%m') AS act_month,
                    date_format(submit_time, '%Y%m%d') AS act_day,
                    'question' AS tag
   FROM practice_record) total
RIGHT JOIN user_info ON total.uid = user_info.uid
WHERE user_info.LEVEL IN (6,
                          7)
GROUP BY user_info.uid
ORDER BY act_month_total DESC,
         act_days_2021 DESC
```

<!-- @include: @article-footer.snippet.md -->
