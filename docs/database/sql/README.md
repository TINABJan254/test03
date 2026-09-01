---
title: SQL 专题：语法基础、查询、聚合、连接、子查询与常见面试题
description: SQL 面试与数据库基础学习路线，涵盖 SQL 查询、过滤、排序、聚合、分组、连接、子查询、增删改、约束、事务和常见 SQL 面试题。
category: 数据库
tag:
  - SQL
  - 数据库
  - 后端面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: SQL,SQL面试题,SQL语法,SQL查询,SQL聚合,SQL连接,SQL子查询,数据库基础,后端面试
---

SQL là kỹ năng cơ bản về database không thể bỏ qua trong lập trình backend. Bất kể việc học MySQL Index, Execution Plan hay tối ưu Slow SQL sau này, bạn đều cần phải nắm chắc các ngữ nghĩa cơ bản như Query, Filter, Aggregation, JOIN, Subquery và Data Modification trước.

## Phù hợp với ai

- Các lập trình viên backend đang học về Database cơ bản và cú pháp SQL.
- Các bạn học sinh/sinh viên và ứng viên đang chuẩn bị cho các câu hỏi phỏng vấn về SQL cơ bản, bài tập SQL Query, CRUD Database.
- Các độc giả đã từng viết SQL đơn giản, nhưng chưa thực sự thành thạo về JOIN, GROUP BY, HAVING, Subquery và thứ tự thực thi (execution order).
- Các kỹ sư muốn củng cố kỹ năng cơ bản về SQL trước khi học MySQL Index và tối ưu SQL.

## Trọng tâm học tập

- Nhiệm vụ và thứ tự thực thi của SELECT, WHERE, ORDER BY, LIMIT, GROUP BY, HAVING được hiểu như thế nào?
- INNER JOIN, LEFT JOIN, RIGHT JOIN, UNION, Subquery lần lượt phù hợp với những kịch bản nào?
- Kết hợp sử dụng Aggregation Function, Grouping Statistics và Condition Filtering như thế nào?
- Những biên (boundary) nào dễ bị bỏ qua trong cú pháp INSERT, UPDATE, DELETE?
- Làm thế nào để bóc tách các bài tập SQL trong phỏng vấn từ mối quan hệ giữa các bảng, điều kiện lọc, chiều gom nhóm (aggregation dimension) và sắp xếp phân trang?

## Thứ tự đọc đề xuất

1. [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql-syntax-summary.md): Đầu tiên hãy nắm vững một cách hệ thống các cú pháp cơ bản và thao tác thường gặp trong SQL.
2. [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 1)](./sql-questions-01.md), [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 2)](./sql-questions-02.md): Luyện tập các truy vấn cơ bản, Sort, Aggregation và Function thường gặp.
3. [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 3)](./sql-questions-03.md): Tiếp tục bổ sung tư duy về JOIN, Subquery và Complex Query.
4. [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 4)](./sql-questions-04.md), [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 5)](./sql-questions-05.md): Củng cố khả năng bóc tách bài tập Query thông qua nhiều dạng đề hơn.
5. Sau khi học xong SQL cơ bản, khuyến nghị tiếp tục đọc [Chuyên đề MySQL](../mysql/), kết hợp cú pháp SQL với Index và Execution Plan.

## Các bài viết cốt lõi

- [Tổng hợp kiến thức cơ bản về cú pháp SQL](./sql-syntax-summary.md): Bao gồm các cú pháp cơ bản như Query, Filter, Sort, Aggregation, Grouping, JOIN, Subquery, Insert, Update, Delete và Constraint.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 1)](./sql-questions-01.md): Làm quen với Query, Sort và Filter đơn giản thông qua các bài tập cơ bản.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 2)](./sql-questions-02.md): Tiếp tục luyện tập Function, String Processing, Date Processing và các cú pháp Query thường gặp.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 3)](./sql-questions-03.md): Thích hợp dùng để rèn luyện Multi-table Query, Grouping Statistics và bóc tách Subquery.
- [Tổng hợp câu hỏi phỏng vấn thường gặp về SQL (Phần 4)](./sql-questions-04.md): Bổ sung thêm nhiều câu hỏi phỏng vấn SQL thường gặp và tư duy giải đề.
- [Tổng hợp câu hỏi phỏng vấn thường gặp关于 SQL (Phần 5)](./sql-questions-05.md): Củng cố hơn nữa khả năng ứng dụng tổng hợp các bài tập SQL Query.

## Câu hỏi tần suất cao

- Thứ tự thực thi logic (Logical Execution Order) của câu lệnh SQL Query là gì?
- WHERE và HAVING khác nhau như thế nào?
- INNER JOIN và LEFT JOIN khác nhau như thế nào?
- UNION và UNION ALL khác nhau như thế nào?
- COUNT(*), COUNT(1), COUNT(tên_cột) khác nhau như thế nào?
- Nên lựa chọn Subquery hay JOIN như thế nào?
- Tại sao sau GROUP BY chỉ có thể chọn column gom nhóm hoặc kết quả Aggregation?
- Paging Query (Truy vấn phân trang) có những cú pháp phổ biến nào?
- Tại sao UPDATE và DELETE nhất định phải cẩn trọng kèm theo điều kiện lọc (filter condition)?
- Bài tập SQL nên được bóc tách theo mối quan hệ bảng như thế nào?

## Các chuyên đề liên quan

- [Hệ thống kiến thức Database](../)
- [Chuyên đề MySQL](../mysql/)
- [Hệ thống kiến thức hệ thống hiệu năng cao](../../high-performance/)
- [Tổng hợp các phương pháp tối ưu SQL thường gặp](../../high-performance/sql-optimization.md)

<!-- @include: @article-footer.snippet.md -->
