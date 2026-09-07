---
title: Kế hoạch vượt qua phỏng vấn Java Backend phiên bản mới nhất 2026 (Bao quát hệ thống kiến thức Backend tổng quát)
description: Kế hoạch ôn tập phỏng vấn Java Backend, cung cấp phiên bản nén 4 tuần và phiên bản tiêu chuẩn 8 tuần, bao quát Dự án & CV, Java Core, MySQL, Redis, Spring, Cơ sở máy tính, Hệ thống phân tán, High Availability và JVM, đồng thời cung cấp kết quả đầu ra và phương pháp tự kiểm tra ở từng giai đoạn.
category: Chuẩn bị phỏng vấn
icon: mdi:star-outline
head:
  - - meta
    - name: keywords
      content: Phỏng vấn Java Backend, Kế hoạch chuẩn bị phỏng vấn, Hướng dẫn phỏng vấn, Câu hỏi cốt lõi, Tuyển dụng sinh viên, Tuyển dụng có kinh nghiệm, Kinh nghiệm dự án, Phỏng vấn Java
---

Đọc toàn bộ các câu hỏi phỏng vấn trong JavaGuide từ đầu đến cuối mới chỉ hoàn thành bước đọc tài liệu. Người phỏng vấn thường bắt đầu hỏi từ CV và dự án, sau đó tiếp tục đào sâu dựa trên Java, MySQL, Redis, Spring hoặc Message Queue được đề cập trong đó. Nếu chỉ ôn tập theo mục lục tài liệu, rất dễ rơi vào tình trạng đọc rất nhiều bài viết nhưng khi đến lượt mình trả lời thì vẫn không biết bắt đầu từ đâu.

Kế hoạch này đặt dự án và CV lên hàng đầu, kiến thức kỹ thuật sẽ được triển khai theo CV và vị trí ứng tuyển mục tiêu. Kế hoạch được chia thành phiên bản nén 4 tuần và phiên bản tiêu chuẩn 8 tuần. Khi không đủ thời gian, hãy lược bỏ các nội dung mở rộng không liên quan đến vị trí ứng tuyển, đừng biến mỗi chuyên đề thành việc cưỡi ngựa xem hoa.

## Chọn bản 4 tuần hay 8 tuần trước?

| Giai đoạn | Phiên bản nén 4 tuần | Phiên bản tiêu chuẩn 8 tuần |
| --- | --- | --- |
| Kiểm tra chuẩn bị ban đầu | Ngày 1～2 | Ngày 1～2 |
| Dự án và CV | Thời gian còn lại của Tuần 1 | Tuần 1 |
| Java, MySQL, Redis | Tuần 2 | Tuần 2～4 |
| Spring và Thiết kế hệ thống | Nửa đầu Tuần 3 | Tuần 5 |
| Cơ sở máy tính và Thuật toán | Đan xen mỗi ngày, chọn lọc theo vị trí | Tuần 6 ôn tập tập trung, thuật toán luyện tập hàng ngày |
| Hệ thống phân tán, JVM và Xử lý sự cố Online | Nửa sau Tuần 3 đến Tuần 4, chọn theo CV | Tuần 7 |
| Mock Interview và Rà soát lỗ hổng | 2 ngày cuối | Tuần 8 |

Bản 4 tuần phù hợp với những người đã học qua các kiến thức chính, hiện tại cần ôn tập tập trung; nếu lần đầu tiên học có hệ thống kiến thức Java Backend, 8 tuần cũng chỉ là một điểm khởi đầu. Nếu mỗi ngày chỉ có thể dành ra chưa đến 2 tiếng ổn định, hãy ưu tiên chọn bản 8 tuần. Nếu đã bắt đầu nộp CV hoặc phỏng vấn sắp đến gần, có thể đi theo bản 4 tuần, nhưng phạm vi ôn tập phải thu hẹp theo CV: CV không viết Kafka, mô tả công việc (JD) cũng không có yêu cầu liên quan, thì không cần tốn hai ba ngày vào chi tiết triển khai của Message Queue.

Thuật toán đừng để dồn đến phút chót mới nhồi nhét. Vị trí có yêu cầu thi viết (coding test) hoặc bài tập code, hãy duy trì luyện tập từ tuần đầu tiên; vị trí không thi thuật toán, hãy dành thời gian này cho dự án, cơ sở dữ liệu và các câu hỏi tình huống (scenario-based questions).

## Mức độ nào mới tính là "đã nắm vững"?

"Đã từng đọc qua" và "trả lời được khi phỏng vấn" cách nhau rất xa. Cùng một câu hỏi ít nhất phải trải qua ba tầng dưới đây:

| Tầng cấp | Phương thức tự kiểm tra |
| --- | --- |
| Trả lời được | Không nhìn tài liệu, dùng 30～60 giây nói ra kết luận và từ khóa chính |
| Trả lời được câu hỏi đào sâu | Tiếp tục giải thích nguyên lý hoạt động, điều kiện áp dụng, các trường hợp lỗi thường gặp và giải pháp thay thế |
| Áp dụng được vào dự án | Nêu rõ trong dự án có dùng hay không, tại sao chọn như vậy, từng gặp giới hạn gì và cách kiểm chứng kết quả |

Sổ ghi chép ôn tập không cần làm quá phức tạp, chỉ cần giữ lại 4 cột: "Câu hỏi, Link tài liệu, Tầng cấp hiện tại, Điểm chưa trả lời tốt". Nội dung đọc xong trong ngày ít nhất phải trả lời không nhìn tài liệu một lần; nếu không trả lời được thì tra lại bài gốc, đừng dùng việc đọc đi đọc lại để thay thế cho việc chủ động gợi nhớ.

## Giai đoạn 0: Xác định phạm vi trước tiên

Dùng 1～2 ngày để hoàn thành 3 việc: Xác định vị trí mục tiêu, Kiểm tra CV, Làm một bài tự đánh giá năng lực ban đầu.

Trước tiên hãy tìm một vài bản mô tả công việc (JD) dự định ứng tuyển, ghi lại các kỹ năng xuất hiện lặp đi lặp lại, sau đó đối chiếu từng mục với CV. Phạm vi ôn tập chủ yếu đến từ hai nguồn: Yêu cầu công việc ghi rõ điều gì, và CV chủ động viết điều gì. Trên CV xuất hiện "Thành thạo Redis", "Phụ trách module đơn hàng", "Sử dụng Kafka xử lý tác vụ bất đồng bộ", thì phía sau phải có các câu hỏi tương ứng và chi tiết dự án để đỡ được.

Giai đoạn này ít nhất phải để lại 4 tài liệu:

- Một bản CV định dạng PDF sẵn sàng để ứng tuyển.
- Dàn ý tự giới thiệu bản thân từ 30～60 giây.
- Một bản phác thảo chi tiết cho mỗi dự án.
- Một danh sách các câu hỏi cần ôn tập được sắp xếp theo thứ tự ưu tiên.

Phương pháp chuẩn bị có thể tham khảo [Làm thế nào để chuẩn bị phỏng vấn Java hiệu quả?](./teach-you-how-to-prepare-for-the-interview-hand-in-hand.md) và [Tổng hợp trọng tâm phỏng vấn Java Backend](./key-points-of-interview.md).

Khi CV chưa được chốt bản cuối, hãy xem trước [Hướng dẫn viết CV cho lập trình viên](./resume-guide.md); đừng vừa ôn tập vừa liên tục thêm công nghệ mới vào CV, nếu không phạm vi ôn tập sẽ không ngừng mở rộng.

## Giai đoạn 1: Đào sâu Dự án và CV

Dự án thường là cánh cửa dẫn tới các câu hỏi kỹ thuật đào sâu. Nếu dự án không trình bày rõ ràng, học thuộc bao nhiêu nguyên lý component cũng rất khó liên hệ câu trả lời về trải nghiệm thực tế của chính mình.

Chuẩn bị các nội dung sau cho từng dự án trọng điểm:

| Nội dung | Câu hỏi cần trả lời |
| --- | --- |
| Bối cảnh nghiệp vụ | Dự án phục vụ ai, giải quyết vấn đề gì, luồng xử lý cốt lõi là gì |
| Trách nhiệm cá nhân | Những API, bảng dữ liệu, tác vụ hoặc module nào do mình phụ trách, tham gia ở mức độ nào |
| Luồng xử lý request | Một request đi qua những service, cache, database và message queue nào |
| Lựa chọn công nghệ (Tech Selection) | Tại sao sử dụng giải pháp hiện tại, đã so sánh với những gì, phải trả giá bằng điều gì |
| Khó khăn hoặc Sự cố | Hiện tượng là gì, định vị, khắc phục và kiểm chứng kết quả như thế nào |
| Chỉ số dự án | Dữ liệu đến từ production hay môi trường test, tiêu chuẩn thống kê và điều kiện đối chứng là gì |
| Phạm vi trách nhiệm | Những phần nào do đồng nghiệp hoặc team khác phụ trách |

Mỗi dự án chuẩn bị 2 phiên bản: 30 giây và 3 phút. Bản 30 giây nói về nghiệp vụ, trách nhiệm và 1 điểm trọng tâm; bản 3 phút bổ sung thêm luồng xử lý cốt lõi, lựa chọn công nghệ và một vấn đề có thể tiếp tục bị hỏi sâu. Không học thuộc từng chữ, chỉ cần nhớ trình tự và các từ khóa chính. Cách viết cụ thể xem tại [Trình bày dự án Backend trong phỏng vấn như thế nào?](./backend-project-interview-guide.md).

Dựa theo từng công nghệ trong dự án để tiếp tục liệt kê câu hỏi. Ví dụ sử dụng Redis cache thông tin sản phẩm, ít nhất phải chuẩn bị thiết kế Key, chiến lược hết hạn (expiration), cache miss, tính nhất quán dữ liệu (data consistency) và cách xử lý khi Redis không khả dụng; nếu viết có dùng Thread Pool, phải giải thích được loại tác vụ, các tham số cốt lõi, hàng đợi, rejection policy cũng như khả năng chịu tải của downstream.

Kết quả dự án có thể lượng hóa, nhưng số liệu phải có nguồn gốc rõ ràng. Khi không có chỉ số production, có thể đo kiểm bổ sung trong môi trường test, và ghi chú rõ cấu hình máy, lượng dữ liệu, mô hình concurrency và thời gian test. Đừng bịa đặt số QPS production cho các dự án luyện tập, cũng đừng viết module mình chỉ xem qua thành do mình phụ trách.

Nếu không có kinh nghiệm thực tập hoặc dự án chính thức vẫn có thể chuẩn bị được. Dự án làm theo khóa học, dự án open source phát triển mở rộng, đồ án môn học và dự án thi đấu đều có thể viết, trọng tâm là bản thân đã thực hiện những thay đổi gì: Thêm tính năng, điều chỉnh cấu trúc bảng, bổ sung unit test, fix bug hoặc so sánh các giải pháp khác nhau. Có thể đọc tiếp [Hướng dẫn kinh nghiệm dự án](./project-experience-guide.md), [Chưa có kinh nghiệm thực tập thì làm sao?](./internship-experience.md) và [Dự án thực chiến Java mã nguồn mở chất lượng](../open-source-project/practical-project.md).

Sau khi hoàn thành giai đoạn này, hãy chọn ngẫu nhiên một dự án và trả lời 4 câu hỏi dưới đây mà không nhìn tài liệu:

1. Dự án này giải quyết vấn đề gì, bạn phụ trách phần nào?
2. Một request cốt lõi luân chuyển như thế nào?
3. Lựa chọn kỹ thuật nào đáng để giải thích nhất, tại sao?
4. Đã từng gặp vấn đề gì, kết luận được chứng minh bằng bằng chứng nào?

## Giai đoạn 2: Java, MySQL và Redis

Ba phần này có phạm vi phủ sóng rất rộng, không nên phân bổ thời gian cào bằng. Hãy làm một lượt rút câu hỏi ngẫu nhiên trước, chuyên đề nào chỉ nói được định nghĩa thì bù thời gian vào đó; nội dung đã có thể kết hợp với dự án để trả lời thì chỉ cần review lại.

### Java cơ bản, Collections và Concurrency

Xem trước 3 nhóm bài viết về Java cơ bản, Collections và Concurrency:

- [Câu hỏi phỏng vấn thường gặp về Java cơ bản (Phần 1)](../java/basis/java-basic-questions-01.md), [(Phần 2)](../java/basis/java-basic-questions-02.md), [(Phần 3)](../java/basis/java-basic-questions-03.md)
- [Câu hỏi phỏng vấn thường gặp về Java Collections (Phần 1)](../java/collection/java-collection-questions-01.md), [(Phần 2)](../java/collection/java-collection-questions-02.md)
- [Câu hỏi phỏng vấn thường gặp về Java Concurrency (Phần 1)](../java/concurrent/java-concurrent-questions-01.md), [(Phần 2)](../java/concurrent/java-concurrent-questions-02.md), [(Phần 3)](../java/concurrent/java-concurrent-questions-03.md)

Phần cơ bản phải giải thích được các khái niệm và hành vi code thường gặp; Collections tập trung vào việc lựa chọn cấu trúc, cơ chế mở rộng dung lượng (resize/grow), thread-safety và các lỗi dùng sai phổ biến; Concurrency cần xâu chuỗi được trạng thái Thread, Lock, JMM, ThreadLocal, Thread Pool và tác vụ bất đồng bộ (Async). Nếu CV có liên quan đến lập trình đa luồng, hãy xem sâu hơn [JMM](../java/concurrent/jmm.md), [Chi tiết Thread Pool](../java/concurrent/java-thread-pool-summary.md), [ThreadLocal](../java/concurrent/threadlocal.md), [AQS](../java/concurrent/aqs.md) và [CompletableFuture](../java/concurrent/completablefuture-intro.md).

### MySQL

[Tổng hợp câu hỏi phỏng vấn MySQL thường gặp](../database/mysql/mysql-questions-01.md) thích hợp làm mạch chính. Khi đọc đến Index, Transaction và Lock, hãy chuyển sang các bài viết chuyên đề:

- [Chi tiết về MySQL Index](../database/mysql/mysql-index.md)
- [Ba loại Log lớn trong MySQL (binlog, redo log, undo log)](../database/mysql/mysql-logs.md)
- [Các mức độ cô lập Transaction](../database/mysql/transaction-isolation-level.md)
- [Cách InnoDB triển khai MVCC](../database/mysql/innodb-implementation-of-mvcc.md)
- [SQL được thực thi như thế nào trong MySQL](../database/mysql/how-sql-executed-in-mysql.md)
- [Phân tích Execution Plan trong MySQL (EXPLAIN)](../database/mysql/mysql-query-execution-plan.md)

Câu hỏi về Index đừng chỉ dừng lại ở Leftmost Prefix Rule và Index Invalidation. Hãy lấy một câu SQL trong dự án, giải thích điều kiện truy vấn, phân bổ dữ liệu, execution plan, số dòng được quét (rows scanned) và cuối cùng đã sửa đổi tối ưu như thế nào. Câu hỏi về Transaction cũng phải liên hệ được với code: Tại sao phạm vi transaction quá lớn, những lời gọi nào không nên đặt trong transaction, Spring transaction bị mất tác dụng (invalid) trong những trường hợp nào.

### Redis

Đọc trước [Câu hỏi phỏng vấn Redis thường gặp (Phần 1)](../database/redis/redis-questions-01.md) và [(Phần 2)](../database/redis/redis-questions-02.md), sau đó chọn chuyên đề dựa theo dự án:

- Cấu trúc dữ liệu: [5 kiểu dữ liệu cơ bản](../database/redis/redis-data-structures-01.md), [3 kiểu dữ liệu đặc biệt](../database/redis/redis-data-structures-02.md), [Skip List](../database/redis/redis-skiplist.md)
- Vấn đề Cache: [Cơ bản về Cache](../database/redis/cache-basics.md), [Chiến lược đọc ghi Cache phổ biến](../database/redis/3-commonly-used-cache-read-and-write-strategies.md)
- Vận hành và Lưu trữ: [Cơ chế Persistence](../database/redis/redis-persistence.md), [Phân mảnh bộ nhớ](../database/redis/redis-memory-fragmentation.md), [Nguyên nhân Blocking phổ biến](../database/redis/redis-common-blocking-problems-summary.md)
- Ứng dụng nghiệp vụ: [Delayed Task với Redis](../database/redis/redis-delayed-task.md), [Dùng Redis Stream làm Message Queue](../database/redis/redis-stream-mq.md)

Khi chuẩn bị Redis, đừng chỉ học vẹt các kiểu dữ liệu. Hãy chọn một luồng xử lý cache thực tế trong dự án và thử trình bày hoàn chỉnh: Request đọc cache như thế nào, khi miss thì truy vấn dữ liệu từ đâu, sau khi lấy được thì ghi lại cache ra sao, cache hết hạn sau bao lâu, khi Redis gặp sự cố thì nghiệp vụ phòng thủ (fallback) như thế nào.

Sau khi ôn tập xong, hãy trộn lẫn câu hỏi của Java, MySQL và Redis để bốc thăm ngẫu nhiên, mỗi loại 5 câu. Mỗi câu trước tiên dùng 1-2 câu để đưa ra kết luận, sau đó tiếp tục đào sâu 2 vòng: "Tại sao?" "Trong dự án sử dụng như thế nào?". Câu nào bị nghẽn thì quay lại bài viết tương ứng để bù đúng điểm kiến thức đó, không cần đọc lại toàn bộ chương.

## Giai đoạn 3: Spring và Thiết kế hệ thống

### Spring, Spring Boot và MyBatis

Trọng tâm chuẩn bị Spring là các tính năng thực sự được sử dụng trong dự án. Xem trước [Câu hỏi phỏng vấn Spring thường gặp](../system-design/framework/spring/spring-knowledge-and-questions-summary.md) và [Câu hỏi phỏng vấn Spring Boot thường gặp](../system-design/framework/spring/springboot-knowledge-and-questions-summary.md), sau đó bổ sung các chuyên đề:

- [IoC và AOP](../system-design/framework/spring/ioc-and-aop.md)
- [Spring Transaction](../system-design/framework/spring/spring-transaction.md)
- [Nguyên lý Auto-configuration trong Spring Boot](../system-design/framework/spring/spring-boot-auto-assembly-principles.md)
- [Các Design Patterns được sử dụng trong Spring](../system-design/framework/spring/spring-design-patterns-summary.md)
- [Câu hỏi phỏng vấn MyBatis thường gặp](../system-design/framework/mybatis/mybatis-interview.md)

Khi tự kiểm tra đừng chỉ giải thích ý nghĩa annotation. Hãy kết hợp với dự án để giải thích Bean được tạo như thế nào, AOP dùng ở đâu, ranh giới transaction được phân chia ra sao, tại sao một transaction nào đó bị mất tác dụng, và MyBatis cuối cùng đã thực thi câu lệnh SQL nào. Nếu dự án không dùng Netty, Reactive Programming hoặc Extension Points phức tạp thì không cần tạm thời nhồi nhét vào CV chỉ để tăng độ phủ.

### Authentication, Authorization và Vấn đề bảo mật thường gặp

Khi CV có liên quan đến Đăng nhập, Phân quyền hoặc Open API, hãy chuẩn bị [Cơ bản về Authentication & Authorization](../system-design/security/basis-of-authority-certification.md), [JWT](../system-design/security/jwt-intro.md), [SSO](../system-design/security/sso-intro.md) và [Thiết kế hệ thống phân quyền](../system-design/security/design-of-authority-system.md). Khi trả lời hãy nói rõ thông tin xác thực lưu ở đâu, quyền hạn kiểm tra ở vị trí nào, Token hết hạn/hủy như thế nào, và API phòng chống vượt quyền (Privilege Escalation) cùng Submit lặp lại (Idempotency) ra sao.

### Thiết kế hệ thống và Câu hỏi tình huống (System Design)

Câu hỏi System Design cần xác nhận yêu cầu và ràng buộc trước khi bắt đầu vẽ các component. Trả lời theo trình tự sau:

1. Làm rõ quy mô người dùng, lượng request, độ trễ (latency), yêu cầu về tính sẵn sàng (availability) và tính nhất quán (consistency).
2. Tìm ra luồng nghiệp vụ cốt lõi, mô hình dữ liệu và các API.
3. Đưa ra giải pháp cơ bản có thể hoạt động được.
4. Dựa vào điểm nghẽn (bottleneck) để bổ sung Cache, Async, Sharding, Rate Limiting hoặc Circuit Breaking/Degradation.
5. Giải thích các kịch bản lỗi, tính nhất quán dữ liệu, giám sát (monitoring) và kiểm chứng dung lượng.

Mới bắt đầu hãy xem [Tổng hợp câu hỏi phỏng vấn Thiết kế hệ thống thường gặp](../system-design/system-design-questions.md), [Câu hỏi phỏng vấn Thiết kế hệ thống hiệu năng cao](../high-performance/high-performance-system-interview-questions.md) và [Câu hỏi phỏng vấn Thiết kế hệ thống độ sẵn sàng cao](../high-availability/high-availability-system-interview-questions.md). Các tình huống hoàn chỉnh như Rút gọn link (Short URL), Flash Sale (Seckill), Xử lý dữ liệu lớn có thể tham khảo [Các câu hỏi tình huống và Thiết kế hệ thống tần suất cao trong phỏng vấn Backend](../zhuanlan/back-end-interview-high-frequency-system-design-and-scenario-questions.md).

Sau khi hoàn thành, hãy chọn 2 đề bài để trình bày miệng mà không nhìn vào sơ đồ kiến trúc có sẵn. Lần đầu đưa ra giải pháp cơ bản, sau khi người phỏng vấn tăng lưu lượng, sự cố hoặc yêu cầu về tính nhất quán thì điều chỉnh tiếp, tập trung giải thích tại sao phương án lại thay đổi.

## Giai đoạn 4: Cơ sở máy tính và Thuật toán

Độ sâu ôn tập cơ sở máy tính do vị trí ứng tuyển và quy trình phỏng vấn quyết định. Nếu có vòng thi viết, phỏng vấn thuật toán hoặc live coding, thuật toán cần được luyện tập liên tục từ tuần đầu tiên; nếu vị trí chú trọng hơn vào phát triển nghiệp vụ, vẫn cần đảm bảo trả lời được các câu hỏi về Mạng, Hệ điều hành và Cấu trúc dữ liệu phổ biến.

### Thuật toán và Cấu trúc dữ liệu

Trước tiên dùng [Chuyên đề thuật toán](../cs-basics/algorithms/) để xác định phạm vi, sau đó luyện tập [Tìm kiếm nhị phân (Binary Search)](../cs-basics/algorithms/binary-search.md), [Two Pointers & Sliding Window](../cs-basics/algorithms/two-pointers-and-sliding-window.md), [DFS/BFS](../cs-basics/algorithms/dfs-bfs.md), [Quay lui (Backtracking)](../cs-basics/algorithms/backtracking.md), [Quy hoạch động (Dynamic Programming)](../cs-basics/algorithms/dynamic-programming.md) và [Top K](../cs-basics/algorithms/top-k.md).

Khi cày bài tập hãy lưu lại các bài làm sai và các điều kiện biên, không nên chỉ chạy theo việc nhớ mẫu (template). Ít nhất phải giải thích được Time Complexity, tự tay code được các bài liên quan đến Linked List, Tree Traversal, Binary Search, Hash Table và Heap; nếu CV có ghi cấu trúc dữ liệu nào, còn phải giải thích được tại sao nó phù hợp với ngữ cảnh hiện tại.

### Mạng máy tính và Hệ điều hành

Phần Mạng máy tính hãy xem qua [Câu hỏi phỏng vấn Mạng máy tính thường gặp (Phần 1)](../cs-basics/network/other-network-questions.md) và [(Phần 2)](../cs-basics/network/other-network-questions2.md), sau đó tập trung vào [Quy trình từ lúc nhập URL đến khi trang web hiển thị](../cs-basics/network/the-whole-process-of-accessing-web-pages.md), [HTTP và HTTPS](../cs-basics/network/http-vs-https.md), [TCP 3-way Handshake và 4-way Teardown](../cs-basics/network/tcp-connection-and-disconnection.md) cùng [Cách TCP đảm bảo truyền tải tin cậy](../cs-basics/network/tcp-reliability-guarantee.md).

Hệ điều hành lấy [Câu hỏi phỏng vấn Hệ điều hành thường gặp (Phần 1)](../cs-basics/operating-system/operating-system-basic-questions-01.md) và [(Phần 2)](../cs-basics/operating-system/operating-system-basic-questions-02.md) làm chủ đạo, tập trung kiểm tra Process vs Thread, Virtual Memory, I/O, Deadlock và System Call. Đừng chỉ học vẹt định nghĩa, hãy thử liên hệ chúng với Java Thread, File I/O, Network Request, OOM và Context Switch.

## Giai đoạn 5: Hệ thống phân tán, Hiệu năng cao và Độ sẵn sàng cao

Giai đoạn này đi theo CV và vị trí ứng tuyển. Nếu dự án là ứng dụng monolithic, vị trí tuyển dụng cũng không có yêu cầu về hệ thống phân tán, chỉ cần nắm các câu hỏi phổ biến là đủ; nếu CV có viết Microservices, Message Queue, Distributed Lock hoặc Phân cơ sở dữ liệu & bảng (Database Sharding & Partitioning), chuyên đề tương ứng phải chịu được các câu hỏi đào sâu.

| Nội dung xuất hiện trong CV hoặc JD | Lối vào ôn tập | Ít nhất phải chuẩn bị đến mức độ nào |
| --- | --- | --- |
| Microservices, RPC | [Câu hỏi phỏng vấn Microservices](../distributed-system/microservices-interview-questions.md), [Cơ bản về RPC](../distributed-system/rpc/rpc-intro.md) | Service được chia tách thế nào, lời gọi xử lý timeout và retry ra sao, cô lập sự cố thế nào |
| API Gateway, Configuration Center | [API Gateway](../distributed-system/api-gateway.md), [Distributed Configuration Center](../distributed-system/distributed-configuration-center.md) | Request routing, authentication, rate limiting, push cấu hình và xử lý sự cố |
| Distributed ID, Lock, Transaction | [Distributed ID](../distributed-system/distributed-id.md), [Distributed Lock](../distributed-system/distributed-lock-implementations.md), [Distributed Transaction](../distributed-system/distributed-transaction.md) | Điều kiện chọn giải pháp, rủi ro về tính chính xác, timeout và phục hồi sau lỗi |
| Message Queue | [Câu hỏi phỏng vấn Message Queue](../high-performance/message-queue/message-queue-interview-questions.md) | Gửi lỗi, tiêu thụ lặp lại (duplicate consumption), thứ tự tin nhắn, tích tụ tin nhắn (backlog) và dung lượng downstream |
| High Concurrency & Tối ưu Database | [Thiết kế hệ thống hiệu năng cao](../high-performance/high-performance-system-interview-questions.md), [Tối ưu SQL](../high-performance/sql-optimization.md) | Vị trí điểm nghẽn, giới hạn dung lượng, cái giá phải trả của Cache và Asynchronous |
| Xây dựng tính ổn định | [Thiết kế hệ thống độ sẵn sàng cao](../high-availability/high-availability-system-design.md), [Timeout và Retry](../high-availability/timeout-and-retry.md), [Rate Limiting](../high-availability/limit-request.md), [Idempotency](../high-availability/idempotency.md) | Sự cố lan truyền thế nào, cách cắt lỗ (stop loss), biện pháp tạm thời có tác dụng phụ gì |

Các lý thuyết như CAP, BASE, Consistent Hashing, Raft dùng để giải thích thiết kế cụ thể, không cần học thuộc lòng định nghĩa dài dòng tách rời khỏi dự án. Hãy chọn một giải pháp phân tán trong dự án, trả lời tại sao cần, tại sao lại chọn giải pháp đó, khi thất bại sẽ ra sao và làm thế nào để chứng minh nó thực sự có hiệu quả.

## Giai đoạn 6: JVM và Xử lý sự cố Online

Nếu CV có ghi JVM Tuning, GC Optimization, Điều tra OOM, hoặc vị trí ứng tuyển nhấn mạnh vào việc xử lý sự cố production, giai đoạn này nên được đẩy lên trước sau phần Java Concurrency. Đối với sinh viên mới ra trường còn thiếu kinh nghiệm thực tế, ít nhất phải nắm vững Memory Regions, Object Garbage Collection, Class Loading và tư duy chẩn đoán lỗi thường gặp.

Trước tiên dùng [Tổng hợp câu hỏi phỏng vấn JVM thường gặp](../java/jvm/jvm-interview-questions.md) để liệt kê các câu hỏi cần trả lời, sau đó bổ sung các chuyên đề:

- [Vùng nhớ trong Java (JVM Memory Area)](../java/jvm/memory-area.md)
- [JVM Garbage Collection](../java/jvm/jvm-garbage-collection.md)
- [Quy trình Class Loading](../java/jvm/class-loading-process.md) và [ClassLoader](../java/jvm/classloader.md)
- [Công cụ giám sát và xử lý sự cố JDK (jstat, jmap, jstack, etc.)](../java/jvm/jdk-monitoring-and-troubleshooting-tools.md)
- [Xử lý sự cố Online trong Java Backend](../java/jvm/jvm-in-action.md)

Tự kiểm tra đừng chỉ dừng lại ở mức "Heap chứa Object, Stack chứa biến cục bộ". Hãy tự đặt cho mình một cảnh báo cụ thể, ví dụ CPU tăng vọt, Full GC liên tục hoặc OOM, trình bày rõ cần xác nhận các chỉ số nào trước, lưu lại hiện trường ra sao, dùng công cụ gì để thu hẹp phạm vi, thao tác nào có thể làm sự cố lan rộng, và sau khi sửa xong thì kiểm chứng như thế nào.

## Sắp xếp ôn tập trong một tuần như thế nào?

Thời gian mỗi ngày chia thành 3 phần: Một nửa dùng để đọc và hiểu, một phần tư dùng để trả lời không nhìn tài liệu, thời gian còn lại luyện diễn đạt dự án hoặc thuật toán. Ngày hôm đó đọc được bao nhiêu trang không quan trọng, ít nhất phải đọng lại một câu hỏi có thể diễn đạt trôi chảy và một điểm vẫn chưa trả lời tốt.

Mỗi tuần sắp xếp một buổi phỏng vấn thử (Mock Interview) từ 30～60 phút. Nhờ đối phương hỏi bắt đầu từ CV, sau khi đào sâu dự án thì chuyển sang Java, Database và câu hỏi tình huống. Khi không có bạn cùng học, có thể tự ghi âm, hoặc dùng AI để mô phỏng hỏi dồn, nhưng sau khi trả lời xong vẫn phải đối chiếu lại thực tế với bài viết, code hoặc tài liệu chính thức.

Trong quá trình ôn tập, việc liên tục thêm tài liệu mới rất dễ khiến kế hoạch mất kiểm soát. Một chuyên đề chỉ cần giữ lại một tài liệu mạch chính và một số ít bài viết chuyên đề; cùng một câu hỏi đã đọc 3 bản câu trả lời mà vẫn không nói ra được thì nên bắt đầu luyện nói không nhìn tài liệu, chứ không phải tiếp tục lưu lại bản thứ 4.

## 1～2 ngày trước khi phỏng vấn nên làm gì?

Cận kề ngày phỏng vấn không nên mở thêm chuyên đề mới, hãy chốt lại theo CV và các câu làm sai:

| Hạng mục | Cách thực hiện |
| --- | --- |
| Tự giới thiệu | Trình bày thử bản 30～60 giây, xác nhận kinh nghiệm, tech stack và định hướng tìm việc đồng nhất |
| Dự án | Mỗi dự án trọng điểm nói thử bản 30 giây và 3 phút, chỗ nào bị vấp phải lập tức bổ sung tư liệu |
| Tech stack trong CV | Rà soát ngẫu nhiên các công nghệ ghi "thành thạo" hoặc "nắm vững", xác nhận có thể trả lời được nguyên lý, giới hạn và cách dùng trong dự án |
| Lỗi sai tần suất cao | Chỉ review lại các câu làm sai và điểm yếu của chính mình, không cày lại toàn bộ kho đề |
| Code và Thiết bị | Phỏng vấn online cần kiểm tra trước mạng, camera, micro, chia sẻ màn hình và môi trường lập trình |
| Thông tin vị trí | Xem lại bản mô tả công việc (JD) một lần nữa, chuẩn bị các dự án và câu hỏi liên quan nhất đến vị trí |

Nếu sự hồi hộp ảnh hưởng đến phong độ, có thể tham khảo [Phải làm gì khi quá hồi hộp trong phỏng vấn?](./how-to-handle-interview-nerves.md).

## Sau khi kết thúc phỏng vấn nên review rút kinh nghiệm như thế nào?

Sau khi phỏng vấn kết thúc, hãy nhanh chóng ghi lại các câu hỏi, không cần cầu toàn nhớ chính xác 100%. Mỗi câu trả lời chưa tốt ghi lại 5 mục: Đề bài, Lúc đó trả lời thế nào, Còn thiếu cái gì, Căn cứ chính xác ở đâu, Lần sau trả lời như thế nào. Khi câu hỏi đào sâu về dự án bị tắc, còn phải quay lại bản phác thảo dự án để bổ sung trách nhiệm, vị trí code, tiêu chuẩn chỉ số hoặc giới hạn của phương án.

Trước buổi phỏng vấn tiếp theo chỉ xem bản review này và các câu hỏi ưu tiên cao ban đầu. Các nội dung mở rộng qua nhiều buổi liên tiếp không được hỏi đến, CV và JD cũng không xuất hiện, có thể hạ cấp ưu tiên; các câu hỏi xuất hiện lặp lại sẽ đưa vào danh sách chính. Phạm vi ôn tập sẽ dần dần thu hẹp theo các buổi phỏng vấn thực tế.
