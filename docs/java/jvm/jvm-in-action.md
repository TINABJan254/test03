---
title: Java 后端线上问题排查：CPU、内存、GC、线程池、数据库与 Redis
description: Java 后端线上故障排查指南，覆盖告警确认、止血留证、CPU 飙高、OOM、频繁 GC、线程池与连接池耗尽、慢 SQL、Redis 阻塞、消息积压和故障复盘。
category: Java
tag:
  - JVM
  - 线上排查
  - 性能调优
  - Java面试
sitemap:
  changefreq: monthly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java线上问题排查,JVM故障排查,CPU飙高,OOM排查,Full GC,线程池耗尽,连接池耗尽,慢SQL,Redis故障,MQ积压
---

Cảnh báo vừa cất lên, việc rà soát rất dễ bị đường cong nổi bật nhất trên trang dashboard giám sát kéo đi. Thấy CPU cao thì lập tức bắt thread stack, thấy Full GC thì lập tức tra xét Heap, vừa kết thúc deploy là chuẩn bị rollback. Nhưng những hiện tượng này thường truyền dẫn lẫn nhau: SQL chậm sẽ chiếm giữ database connection, việc chờ đợi connection pool lại kéo đơ business thread, cuối cùng có thể đồng thời thấy latency, tỷ lệ lỗi và CPU tăng cao. Nếu chỉ nhìn một hạng mục trong đó, rất khó xác định nguyên nhân gốc rễ (root cause).

Khi sự cố vẫn đang lan rộng, trước tiên xử lý theo kịch bản dự phòng để kiểm soát ảnh hưởng. Nếu instance vẫn còn dư tải, sau khi xác nhận việc thu thập chứng cứ không tiếp tục làm sập service, mới tiến hành lưu lại log, Trace, thread stack và thông tin bộ nhớ. Restart thường có thể tạm thời khôi phục, nhưng cũng xóa sạch hiện trường; vì muốn bắt Heap Dump mà liên tục để instance bất thường gánh chịu traffic, cũng có thể kéo vấn đề trở nên nghiêm trọng hơn. Tình hình hiện trường quyết định nên ngắt traffic, rollback trước, hay lấy chứng cứ trước.

Dưới đây chỉ thảo luận các vấn đề thường gặp ở phía ứng dụng Java. Nếu chứng cứ đã trỏ tới container scheduling, OS kernel, thiết bị mạng hoặc cloud platform, cần chuyển sang dashboard giám sát và tài liệu hướng dẫn thao tác của platform tương ứng để tiếp tục rà soát, ở đây không đi sâu.

::: warning Lưu ý thao tác trên môi trường Production
Trước khi thực thi các lệnh chẩn đoán trên môi trường production, trước tiên hãy xem instance có còn gánh traffic hay không, đĩa cứng còn bao nhiêu dung lượng rảnh, cũng như chi phí bản thân lệnh đó. Bắt thread stack liên tục cần kiểm soát tần suất; Heap Dump, histogram class và GC chủ động có thể mang lại chi phí CPU, I/O hoặc pause rất rõ rệt. Instance đã gần như không thể sử dụng được thì nên ngắt traffic hoặc giảm thiểu thiệt hại trước, đừng vì muốn giữ hiện trường mà tiếp tục gánh gượng traffic nghiệp vụ.
:::

## Nhận được cảnh báo thì trước tiên cần xác nhận điều gì?

Nhận được cảnh báo trước tiên xem phía người dùng có bất thường hay không. Một instance đơn lẻ CPU cao nhưng request vẫn được các instance khác gánh vác bình thường, hoàn toàn khác cấp độ với việc tỷ lệ lỗi của toàn bộ service tăng vọt. Còn phải loại trừ các trường hợp báo khống giám sát, độ trễ thu thập chỉ số, cũng như việc downstream timeout làm thread bị tắc nghẽn ở service hiện tại.

Hãy kéo các đường cong lưu lượng request, tỷ lệ lỗi, P95/P99, CPU, bộ nhớ, GC, thread pool và connection pool trong khoảng mười mấy phút trước và sau cảnh báo lại xem cùng lúc, rồi đối chiếu với các mốc thời gian deploy, thay đổi cấu hình, scheduled task và traffic tăng vọt. Cảnh báo của database, Redis, message queue và external interface cũng cần kéo vào. Đi dọc theo timeline để xác nhận những interface và user nào bị ảnh hưởng, bất thường bắt đầu từ khi nào, chỉ số nào biến đổi sớm nhất.

Thời gian phản hồi trung bình (Average Response Time) bình thường cũng không đại diện cho các request đều bình thường. Một số ít request siêu chậm có thể nằm đúng trên các flow then chốt như login, checkout hoặc payment, lúc này P95/P99, tỷ lệ timeout và tỷ lệ thành công nghiệp vụ mới có giá trị tham khảo hơn.

## Chọn như thế nào giữa cầm máu (giảm thiệt hại) và giữ chứng cứ?

Khi sự cố vẫn đang lan rộng, trước tiên xử lý cầm máu theo kịch bản dự phòng. Các biện pháp thường gặp bao gồm rollback phiên bản gần nhất, loại bỏ instance bất thường, tắt scheduled task có vấn đề, rate limit (hạn chế lưu lượng), circuit breaker (ngắt mạch), downgrade (hạ cấp) các chức năng phi cốt lõi cũng như scale-out (mở rộng dung lượng). Mỗi biện pháp đều có side-effect: Scale-out có thể tiếp tục làm sập database, retry sẽ phóng đại traffic downstream, rollback cũng chưa chắc tương thích với dữ liệu đã thay đổi.

Điều kiện cho phép, trước khi restart hoặc rollback hãy giữ lại những thông tin này:

- Ảnh chụp màn hình giám sát hoặc khoảng thời gian trước và sau khi xảy ra cảnh báo.
- Application log, access log, GC log và lịch sử thay đổi.
- 1 đến 3 bản thread stack cách nhau vài giây, chứ không phải chỉ giữ 1 bản.
- Lời gọi chậm (slow call), lời gọi lỗi và thời gian tiêu tốn upstream/downstream trong Trace.
- Trạng thái CPU process, RSS, Heap, số lượng thread, file descriptor và network connection.
- Slow query database, lock wait, trạng thái connection pool, slow command Redis và tích đọng tiêu thụ MQ.

Instance đã được ngắt traffic thì cũng đừng lập tức giả định rằng nó có thể an toàn làm Heap Dump. Dump heap lớn cần đủ dung lượng đĩa cứng và I/O, một số lệnh còn có thể kích hoạt tạm dừng khá dài. Cách làm vững chắc hơn là cấu hình trước tham số tự động dump khi OOM khi khởi động, và ghi file dump vào thư mục có dung lượng và quyền hạn phù hợp.

```bash
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/with/enough/space
```

## Làm thế nào để nhanh chóng phán đoán vấn đề nằm ở tầng nào?

Tạm thời chưa có manh mối call chain, trước tiên chọn lối vào rà soát dựa theo hiện tượng. Quan hệ tương ứng trong bảng không thể thay thế cho việc xác minh sau đó.

| Hiện tượng                     | Ưu tiên kiểm tra                                         |
| ------------------------- | -------------------------------------------------------- |
| CPU cao, interface đồng thời chậm | Hot thread, vòng lặp vô tận, serialization, lock contention, GC thường xuyên |
| CPU không cao, Load rất cao | I/O đĩa cứng, uninterruptible sleep, network storage, CPU quota container |
| RSS tăng liên tục, Java Heap ổn định | Direct memory, thread stack, Metaspace, JNI, thư viện bản địa và memory mapping |
| Old Gen sau khi thu hồi vẫn tăng liên tục | Đối tượng vòng đời dài, cache không giới hạn, collection nắm giữ, Listener hoặc ThreadLocal |
| Số thread và độ dài queue tăng | Downstream chậm, lock wait, xử lý task chậm, cách ly thread pool không đủ |
| Chờ connection pool database tăng | Slow SQL, transaction dài, rò rỉ connection, dung lượng database không đủ |
| Redis RT tăng             | Slow command, Big Key, Hot Key, chập chờn mạng, chờ connection pool |
| MQ Lag tăng liên tục       | Tiêu thụ chậm, phân chia partition không đều, retry message bất thường, dung lượng downstream không đủ |

Vấn đề cũng có thể lan truyền qua các tầng. Một slow SQL chiếm giữ connection database, business thread sau đó bị block ở connection pool, queue của thread pool bắt đầu tích đọng, upstream timeout xong tiến hành retry, cuối cùng thấy được là CPU, latency và tỷ lệ lỗi đồng thời tăng cao. Khi rà soát phải dựa theo timeline xác nhận chỉ số nào biến đổi đầu tiên.

## Rà soát CPU tăng vọt như thế nào?

Trước tiên xác nhận CPU cao xuất phát từ process nào, sau đó tìm thread chiếm CPU bên trong process. Dưới đây là cách rà soát thường gặp trên Linux:

```bash
# Tìm Java process
jps -l

# Xem tình hình sử dụng CPU của từng thread bên trong process
top -H -p <pid>

# Cũng có thể xem sử dụng CPU trong một khoảng thời gian theo thread
pidstat -t -p <pid> 1
```

Trong `top -H` thấy được là thread ID dạng thập phân (decimal), còn `nid` trong thread stack thông thường sử dụng dạng thập lục phân (hexadecimal). Trước tiên chuyển đổi, sau đó vào thread stack tìm thread tương ứng:

```bash
printf '%x\n' <tid>
jcmd <pid> Thread.print > /tmp/thread-dump.txt
```

Thread stack cần bắt liên tiếp vài bản. Một thread chỉ ở trong 1 bản stack thực thi JSON serialization thì có thể chỉ là tình cờ sample trúng; nhiều bản stack đều dừng ở cùng một đoạn code thì mới đáng để tiếp tục kiểm tra. Các nguyên nhân thường gặp bao gồm:

- Vòng lặp hoặc đệ quy không thể thoát.
- Các task tính toán như serialization đối tượng lớn, regex backtracking, mã hóa/giải mã.
- Cấp phát lượng lớn đối tượng dẫn đến GC thường xuyên.
- Nhiều thread tranh chấp cùng một lock, đi kèm context switching (chuyển đổi ngữ cảnh).
- Traffic tăng vọt hoặc retry phóng đại, CPU chỉ đang làm nhiều việc hơn một cách bình thường.

CPU đã gần như chạm trần thì không khởi động đồng thời nhiều task chẩn đoán chi phí cao. Trước tiên ngắt một phần traffic hoặc chọn một instance bất thường để lấy chứng cứ, tránh việc thao tác chẩn đoán tiếp tục tranh giành tài nguyên.

## CPU không cao nhưng Load rất cao thì làm sao?

Linux Load Average thống kê các task đang chạy và ở trạng thái uninterruptible sleep. Khi đĩa cứng, network storage hoặc một số kernel I/O wait nghiêm trọng, tỷ lệ sử dụng CPU có thể không cao nhưng Load vẫn tiếp tục tăng.

```bash
vmstat 1
iostat -xz 1
pidstat -d -p <pid> 1
```

Tập trung quan sát run queue, I/O wait và blocked task trong `vmstat`, cũng như mức độ sử dụng thiết bị, thời gian chờ và queue trong `iostat`. `iostat`, `pidstat` thông thường do package sysstat cung cấp, môi trường production có sử dụng được hay không cần xác nhận trước.

## Interface rất chậm nhưng CPU không cao rà soát như thế nào?

Loại vấn đề này thường xảy ra ở việc chờ đợi: Đợi connection database, đợi phản hồi downstream, đợi lock, đợi task trong thread pool hoặc đợi I/O đĩa cứng.

Trước tiên từ Trace tìm ra đoạn tiêu tốn thời gian dài nhất. Nếu không có Trace, có thể kết hợp thời gian tiêu tốn request trong access log, slow query database, chỉ số connection pool phía client và thread stack để thu hẹp phạm vi. Các trạng thái thường gặp trong thread stack bao gồm:

- Lượng lớn thread dừng ở `getConnection()` của database connection pool: Kiểm tra connection pool wait, active connection, slow SQL và transaction dài.
- Lượng lớn thread dừng ở HTTP client đọc response: Kiểm tra downstream latency, cấu hình timeout và số lần retry.
- Lượng lớn thread ở trạng thái `BLOCKED`: Kiểm tra lock holder và các thao tác chậm bên trong critical section.
- Business thread rảnh rỗi nhưng task queue tích đọng: Kiểm tra consumer thread, phương thức submit task và trạng thái executor.

Tổng thời gian tiêu tốn của một slow request phải đối ứng được với các giai đoạn. Nếu Trace chỉ ghi lại thời gian tiêu tốn phương thức nghiệp vụ, mà connection pool wait, thread pool queuing và thời gian DNS/TLS client không được ghi lại, ở giữa sẽ xuất hiện một khoảng trống không giải thích được, cần bổ sung giám sát hoặc埋点 (metrics instrumentation) tương ứng.

## Bộ nhớ tăng liên tục và OOM rà soát như thế nào?

Trước tiên phân biệt rõ bộ nhớ tăng là Java Heap, process RSS hay tổng bộ nhớ container. `-Xmx` chỉ giới hạn Java Heap, process còn sử dụng Metaspace, Code Cache, thread stack, direct memory, cấu trúc dữ liệu bản thân GC và bộ nhớ thư viện bản địa.

Trước tiên xem dữ liệu cả 2 phía JVM và hệ thống:

```bash
jcmd <pid> GC.heap_info
jcmd <pid> VM.flags
jcmd <pid> VM.native_memory summary
```

`VM.native_memory` phụ thuộc vào Native Memory Tracking (NMT), cần bật khi JVM khởi động, ví dụ `-XX:NativeMemoryTracking=summary`. Nó mang lại chi phí bổ sung, không thể bật bù tạm thời sau khi sự cố xảy ra.

NMT chủ yếu thống kê bộ nhớ bản địa do bản thân JVM/HotSpot quản lý, không thể bao phủ toàn bộ cấp phát bộ nhớ của JNI hoặc thư viện bản địa bên thứ ba. RSS tăng liên tục mà NMT không có biến đổi tương ứng thì còn phải nhờ vào hệ điều hành và công cụ phân tích bộ nhớ bản địa để tiếp tục rà soát.

Thông tin OOM thường gặp tương ứng với các hướng khác nhau:

| Thông tin OOM                    | Hướng rà soát thường gặp                               |
| -------------------------------- | ------------------------------------------------------ |
| `Java heap space`                | Đối tượng lớn, đối tượng bị nắm giữ, collection không giới hạn, cache, 1 query trả về quá nhiều dữ liệu |
| `GC overhead limit exceeded`     | Heap gần cạn kiệt, GC dành lượng lớn thời gian nhưng thu hồi rất ít |
| `Metaspace`                      | Tạo class động, rò rỉ ClassLoader, giới hạn Metaspace quá nhỏ |
| `Direct buffer memory`           | NIO direct memory, Netty Buffer, giải phóng chậm hoặc giới hạn không hợp lý |
| `unable to create native thread` | Số thread quá nhiều, giới hạn process, bộ nhớ container không đủ, stack đơn thread quá lớn |

Heap Dump có thể sử dụng các công cụ như MAT, VisualVM để phân tích. Trước tiên xem đối tượng chiếm dụng lớn nhất, Dominator Tree, chuỗi reference tới GC Roots và ClassLoader khả nghi, đừng chỉ dựa vào số lượng đối tượng mà đưa ra kết luận.

Khi cần dump thủ công, có thể sử dụng:

```bash
jcmd <pid> GC.heap_dump filename=/path/with/enough/space/heap.hprof
```

Lệnh này có thể gây ra ảnh hưởng rõ rệt đối với ứng dụng. Trước khi thực thi nên đánh giá dung lượng Heap, dung lượng đĩa rảnh, I/O và rủi ro tạm dừng. Instance production vẫn đang gánh traffic thì ưu tiên thao tác trên instance bất thường sau khi đã ngắt traffic. `jmap -dump:live` có thể kích hoạt Full GC, không thích hợp coi là lệnh không rủi ro để thực thi trực tiếp.

## Làm thế nào để phán đoán là rò rỉ bộ nhớ (Memory Leak) hay tăng trưởng bình thường?

Quan sát mức chiếm dụng Heap sau một lần Full GC hoàn chỉnh. Nếu traffic nghiệp vụ và quy mô dữ liệu ổn định, vùng Old Gen sau nhiều lần thu hồi hoàn chỉnh vẫn tiếp tục tăng lên, mới cần trọng điểm nghi ngờ đối tượng không thể giải phóng. Warm-up cache, class loading và traffic tăng trưởng cũng làm Heap tăng lên từng bước sau khi khởi động, chúng không nhất thiết là rò rỉ.

Khi phân tích còn phải xem tại sao đối tượng lại bị nắm giữ. Một `Map` rất lớn chỉ chứng tỏ nó chiếm bộ nhớ; tiếp tục đi dọc theo chuỗi reference tìm ra cache không giới hạn, static collection, Listener, ThreadLocal hoặc vòng đời session bị sai, mới có thể xác nhận vị trí sửa lỗi.

## Young GC thường xuyên rà soát như thế nào?

Young GC thường xuyên thông thường thể hiện tốc độ cấp phát đối tượng nhanh hoặc dung lượng Young Generation không đủ. Trước tiên xem tốc độ cấp phát, lượng sống sót sau mỗi lần thu hồi và thời gian tạm dừng, sau đó mới phán đoán xem có cần sửa code hay điều chỉnh tham số hay không.

```bash
jstat -gcutil <pid> 1000 10
jcmd <pid> GC.heap_info
```

Các nguyên nhân thường gặp bao gồm một lần query load lượng lớn đối tượng, interface trả về kết quả siêu lớn, log hoặc serialization tạo lượng lớn đối tượng tạm thời, batch processing không kiểm soát batch size, cũng như cấu hình Young Gen không khớp với tải.

Điều chỉnh Young Gen chỉ có thể thay đổi nhịp điệu GC, không thể sửa code query không giới hạn và cấp phát cao. Trước tiên thông qua GC log, JFR, allocation sampling hoặc load test xác nhận đối tượng sinh ra từ đâu. Sau khi sửa tham số phải so sánh Throughput, P95/P99, số lần GC và tổng thời gian tạm dừng dưới cùng điều kiện traffic và dữ liệu.

## Full GC thường xuyên rà soát như thế nào?

Trước tiên từ GC log xác nhận nguyên nhân kích hoạt, mức chiếm dụng trước sau thu hồi và thời gian tạm dừng. Field log và nguyên nhân kích hoạt của các collector khác nhau không hoàn toàn giống nhau, không thể quy tất cả Full GC về việc "Old Generation đầy".

Khi rà soát trọng điểm xem:

- Vùng Old Gen sau khi thu hồi có giảm xuống rõ rệt hay không.
- Đối tượng lớn và đối tượng thăng tiến có quá nhiều hay không.
- Metaspace có gần đạt tới giới hạn trên hay không.
- Có tồn tại `System.gc()` hiển thị hoặc công cụ chẩn đoán kích hoạt hay không.
- Dung lượng Heap, giới hạn bộ nhớ container và cấu hình collector có khớp nhau hay không.
- Trước GC có xuất hiện traffic tăng vọt, batch task hoặc refresh cache hay không.

Mức chiếm dụng sau thu hồi vẫn rất cao, tiếp tục phân tích việc nắm giữ đối tượng; hiệu quả thu hồi bình thường nhưng rất nhanh lại lấp đầy, trọng điểm kiểm tra tốc độ cấp phát, tốc độ thăng tiến và dung lượng Heap. Trước khi sửa tham số JVM, trước tiên hãy xác nhận hành vi ứng dụng, nếu không việc tăng Heap chỉ hoãn sự cố lần tiếp theo, và có thể làm tăng chi phí tạm dừng hoặc dump.

## Tích đọng queue trong thread pool rà soát như thế nào?

Vấn đề thread pool không thể chỉ xem số lượng active thread. Tối thiểu phải đồng thời giám sát:

- `corePoolSize`, `maximumPoolSize` và số thread hiện tại.
- Số active thread, độ dài queue và dung lượng queue.
- Tốc độ submit task, tốc độ hoàn thành, thời gian chờ và thời gian thực thi.
- Số lần từ chối (rejection) và chiến lược từ chối cụ thể.

Queue tăng liên tục thể hiện tốc độ task đi vào cao hơn tốc độ hoàn thành. Nguyên nhân có thể do traffic bộc phát, cũng có thể do bản thân task trở nên chậm, ví dụ chờ connection database, downstream timeout hoặc lock contention. Trực tiếp tăng số thread sẽ tăng truy cập đồng thời đối với database, Redis và external interface, có thể đẩy vấn đề sang downstream.

Rà soát theo thứ tự dưới đây:

1. Xác nhận những task nghiệp vụ nào sử dụng thread pool này, có task không liên quan dùng chung hay không.
2. So sánh tốc độ submit, tốc độ hoàn thành, thời gian xếp hàng và thời gian thực thi.
3. Trích xuất thread stack của các thread đang chạy task, xác nhận chúng đang tính toán hay đang chờ đợi.
4. Kiểm tra connection pool của database, HTTP và Redis có đồng thời cạn kiệt hay không.
5. Dựa vào tầm quan trọng của task quyết định chiến lược rate limit, downgrade, scale-out, cách ly hoặc drop.

`CallerRunsPolicy` sẽ làm thread submit task tự mình thực thi task, có thể tạm thời làm chậm tốc độ submit, nhưng request thread của Web do đó bị task dài chiếm giữ thì latency của interface cũng sẽ tăng cao. Nó có phù hợp với thread pool hiện tại hay không cần kết hợp caller và loại task để phán đoán.

## Cạn kiệt connection pool database rà soát như thế nào?

Khi connection pool cạn kiệt, trước tiên xem active connection, idle connection, waiting thread và thời gian lấy connection. Business thread chờ đợi connection lượng lớn, thông thường tiếp tục kiểm tra theo 3 hướng:

- Execution SQL chậm, connection thời gian dài không thể trả lại.
- Phạm vi transaction quá lớn, bao gồm remote call, xử lý file hoặc lượng lớn tính toán.
- Code không đóng connection đúng cách ở tất cả các nhánh, xuất hiện rò rỉ connection.

Phía database kiểm tra tiếp session hiện tại, slow query, lock wait, transaction dài và tài nguyên instance. MySQL có thể kết hợp `SHOW PROCESSLIST`, slow query log, Performance Schema, `EXPLAIN` và thông tin transaction lock của InnoDB để định vị. Execution plan đến từ SQL hiện tại, tham số và phân bố dữ liệu, "chạy qua index" trong db test không thể chứng minh môi trường production nhất định giống như vậy.

Trước khi mở rộng connection pool cần xác nhận database còn chịu được nhiều truy cập đồng thời hơn hay không. Số instance ứng dụng nhân với giới hạn trên connection pool của mỗi instance mới là tổng số connection mà database có thể phải đối mặt; sau khi scale-out service, tổng số này cũng sẽ tăng lên theo.

Nội dung liên quan: [Phân tích MySQL Execution Plan](../../database/mysql/mysql-query-execution-plan.md), [Tóm tắt tối ưu hóa SQL](../../high-performance/sql-optimization.md).

## Redis bị chậm rà soát như thế nào?

Khi ứng dụng truy cập Redis bị chậm, trước tiên phân biệt client wait và server execution. Connection pool cạn kiệt, chập chờn mạng, vấn đề DNS đều làm cho một lần gọi Redis bị chậm, mặc dù bản thân command thực thi rất nhanh.

Phía Redis server trọng điểm xem:

- Độ trễ command và slow log.
- CPU, bộ nhớ, lưu lượng mạng và số lượng connection.
- Big Key, Hot Key, xóa expired key cũng như thao tác persistence.
- Độ trễ master-slave replication, trạng thái cluster và failover.

`SLOWLOG GET` ghi lại thời gian thực thi của command tại phía Redis server, không bao gồm xếp hàng và truyền tải mạng. Tổng thời gian tiêu tốn ứng dụng rất cao mà Slow Log bình thường, tiếp tục kiểm tra connection pool wait, mạng và xử lý phía client.

Khi rà soát Big Key có thể sử dụng các công cụ như `redis-cli --bigkeys`, nhưng nó cần scan keyspace, sẽ tăng tải bổ sung. Môi trường production nên chọn khung giờ thấp điểm, slave node hoặc sampling có kiểm soát, và xác nhận trước ảnh hưởng của command đối với phiên bản hiện tại và cluster. Đừng trực tiếp thực thi `KEYS *` trên production.

Hot Key cần kết hợp tần suất truy cập và tải của node để xác nhận. Chỉ nhìn kích thước Key không tìm thấy Small Key có số lần đọc đặc biệt cao; có thể sử dụng proxy layer, thống kê phía client, platform giám sát hoặc khả năng phân tích Hot Key có kiểm soát để lấy chứng cứ.

Nội dung liên quan: [Các câu hỏi phỏng vấn thường gặp về Redis](../../database/redis/redis-questions-02.md).

## Tích đọng Message Queue rà soát như thế nào?

Trước tiên so sánh tốc độ production và tốc độ tiêu thụ, và xác nhận tích đọng tập trung ở Topic, Queue hay Partition nào. Sau đó kiểm tra consumer instance có còn sống không, consumer thread có bị block không, thời gian xử lý đơn bản tin có biến đổi không, cũng như database và interface downstream có bị chậm không.

Các nguyên nhân thường gặp bao gồm:

- Traffic tăng vọt, tốc độ production vượt quá dung lượng thiết kế.
- Một loại message nào đó xử lý chậm, kéo đơ toàn bộ partition hoặc queue.
- Message bất thường liên tục retry, tạo thành retry storm.
- Phân chia partition không đều hoặc consumer xảy ra Rebalance thường xuyên.
- Sau khi tiêu thụ thành công submit offset thất bại, gây ra tiêu thụ lặp lại.
- Database, Redis hoặc external interface downstream trở thành bottleneck (nút thắt cổ chai).

Việc tăng consumer có hiệu quả hay không phụ thuộc vào mô hình Message Queue. Độ song song có hiệu lực trong cùng một consumer group của Kafka bị giới hạn bởi số lượng partition; consumer tiếp tục tăng nhưng không có partition để phân bổ thì sẽ không nâng cao Throughput. Cho dù tăng partition và consumer, cũng phải xác nhận dung lượng downstream, tránh việc sau khi khôi phục tiêu thụ sẽ đánh dồn traffic tích đọng một lúc vào database.

Trong thời gian xử lý tích đọng còn phải đảm bảo tính idempotent, giám sát lỗi tiêu thụ và dead letter. Khi cần bỏ qua message bất thường, nên giữ lại nội dung message và nguyên nhân thất bại để bù đắp sau, chứ không trực tiếp drop.

Nội dung liên quan: [Các vấn đề thường gặp về Message Queue](../../high-performance/message-queue/message-queue.md).

## Interface downstream timeout rà soát như thế nào?

Trước tiên chia thời gian của một lời gọi ra: Connection pool wait, DNS, thiết lập kết nối TCP, bắt tay TLS, gửi request, xử lý phía server và đọc response. Các giai đoạn khác nhau cần cấu hình timeout khác nhau, chỉ thiết lập một tổng timeout sẽ không dễ phán đoán thời gian tiêu tốn vào đâu.

Kiểm tra phía caller trọng điểm xem:

- Connection timeout, read timeout và tổng ngân sách call chain.
- Active connection, waiting queue và tái sử dụng connection của HTTP connection pool.
- Số lần retry, chiến lược backoff cũng như những lỗi nào sẽ kích hoạt retry.
- Trạng thái circuit breaker, kết quả downgrade và tỷ lệ thành công request lần đầu.

Downstream đã trở nên chậm, retry nhiều tầng sẽ nhanh chóng phóng đại traffic. API Gateway, business service và SDK mỗi bên retry 3 lần, trong trường hợp xấu nhất sẽ tạo ra lượng request vượt xa một lời gọi. Trước tiên hạn chế cấp độ retry và tổng ngân sách, sau đó dựa vào interface có idempotent hay không để quyết định có thể retry hay không.

## Dùng một slow interface xâu chuỗi quá trình rà soát

Giả sử interface query order sau một lần release phiên bản thì P99 tăng lên, tỷ lệ lỗi cũng bắt đầu tăng, rà soát theo thứ tự dưới đây. Ví dụ này dùng để minh họa phương pháp, không đại diện cho sự cố production thực tế.

1. Từ giám sát xác nhận vấn đề bắt đầu từ sau release, tập trung ở các instance phiên bản mới và interface query order.
2. Tạm dừng tiếp tục release, ngắt một phần instance bất thường, tỷ lệ lỗi giảm xuống, trước tiên kiểm soát phạm vi ảnh hưởng.
3. So sánh Trace của instance mới và cũ, phát hiện phần lớn thời gian tiêu tốn vào việc lấy database connection và thực thi query.
4. Giám sát connection pool hiển thị waiting thread tăng lên, slow query database xuất hiện cùng một câu SQL.
5. Sử dụng điều kiện query tương ứng với tham số production để xem execution plan, phát hiện điều kiện filter mới thêm làm cho composite index cũ không thể lọc hiệu quả, và đi kèm sorting bổ sung.
6. Tại môi trường test gần với phân bố dữ liệu production sửa đổi query và index, so sánh số dòng scan, P95/P99, CPU và mức chiếm dụng connection.
7. Thay đổi sau khi qua canary (canary release) tiếp tục quan sát slow query, connection pool wait và tỷ lệ lỗi nghiệp vụ, rồi mới từng bước khôi phục traffic.
8. Khi復盘 (post-mortem / họp rút kinh nghiệm) bổ sung performance test cho kịch bản query tương ứng và các mục giám sát sau release.

Trong kịch bản giả định này, tích đọng thread pool và connection pool xuất hiện sau slow SQL; execution plan và bản ghi slow query đã chỉ nguyên nhân gốc rễ về SQL. Nếu bước 4 phát hiện SQL rất nhanh, thời gian chủ yếu tiêu tốn ở ngoài connection pool, thì phải quay lại các giả định khác để tiếp tục xác minh.

## Sau khi sửa lỗi thì xác minh như thế nào?

Sau khi sửa đổi code hoặc cấu hình, còn phải xác nhận chỉ số phía user đã Khôi phục, các biện pháp tạm thời có thể hủy bỏ, và không đưa vào các vấn đề dữ liệu mới.

Khi xác minh so sánh cùng một bộ chỉ số trước sự cố, trong sự cố và sau khi sửa lỗi: Lưu lượng request, P95/P99, tỷ lệ lỗi, tỷ lệ thành công nghiệp vụ, CPU, bộ nhớ, GC, thread pool, connection pool và độ trễ phụ thuộc. Liên quan đến tiêu thụ lặp lại, tồn kho, đơn hàng và trạng thái tiền tệ, còn phải làm đối soát dữ liệu (reconciliation).

Thay đổi về hiệu năng cần so sánh dưới dung lượng dữ liệu, mô hình traffic, cấu hình máy và trạng thái cache tương tự. Chỉ chạy 1 lần load test đơn interface không thể đại diện cho việc toàn bộ business chain đã khôi phục. Cụ thể có thể tham khảo: [Nhập môn Performance Testing và Stress Testing](../../high-availability/performance-test.md).

## Phục盤 (Post-Mortem) sự cố nên ghi lại những gì?

Phục盤 (Post-Mortem) phải để lại các tài liệu có thể trực tiếp sử dụng trong lần sự cố tiếp theo. Document tối thiểu phải lưu lại:

- Timeline sự cố, phạm vi ảnh hưởng và biểu hiện phía người dùng.
- Nguyên nhân trực tiếp, điều kiện kích hoạt cũng như tại sao sự cố lại mở rộng.
- Chứng cứ sử dụng trong quá trình cầm máu, định vị và sửa lỗi.
- Những giám sát, kịch bản dự phòng hoặc test nào chưa phát huy tác dụng.
- Các mục cải tiến sau đó, người chịu trách nhiệm và thời gian hoàn thành.

"Dev tăng cường kiểm tra code" rất khó xác minh xem có hoàn thành hay không. Mục cải tiến nên đi vào cơ chế cụ thể, ví dụ thêm cảnh báo cho thời gian chờ connection pool, giới hạn số lượng bản ghi batch query, bổ sung test tính idempotent cho tiêu thụ message, thêm kiểm tra so sánh interface then chốt cho quy trình release.

## Trong phỏng vấn trả lời rà soát sự cố online như thế nào?

Người phỏng vấn hỏi "CPU online tăng vọt xử lý như thế nào", có thể trả lời thứ tự chung trước, sau đó kết hợp ca thực tế mình từng làm:

> Tôi sẽ trước tiên xác nhận phạm vi ảnh hưởng, traffic và các thay đổi gần đây, phán đoán xem có cần ngắt instance, rate limit hay rollback hay không. Điều kiện cho phép sẽ giữ lại giám sát, log và nhiều bản thread stack. CPU thực sự xuất phát từ Java process xong, dùng `top -H` hoặc `pidstat -t` tìm hot thread, chuyển thread ID sang dạng hex rồi đối ứng với `nid` trong thread stack. Sampling liên tục xác nhận thread thời gian dài dừng ở đoạn code nào, rồi kết hợp GC, Trace và traffic phán đoán xem là hot spot tính toán, vòng lặp vô tận, lock contention hay GC thường xuyên. Sau khi sửa lỗi sử dụng kịch bản tương tự để xác minh, và bổ sung giám sát cũng như regression test.

Nếu chưa từng xử lý sự cố production thực tế, có thể trình bày mình đã diễn tập những kịch bản nào trong môi trường test, cũng như đã tham khảo qua những ca sự cố nào. Đừng nói những bài viết đã đọc thành sự cố do chính mình phụ trách.

## Tra cứu nhanh công cụ thường dùng

| Công cụ            | Tác dụng thường gặp            | Lưu ý sử dụng                         |
| ------------------ | ------------------------------ | ------------------------------------- |
| `top`, `pidstat`   | CPU, I/O của process và thread | Trước tiên xác nhận góc nhìn quan sát container và host machine |
| `vmstat`, `iostat` | Run queue, bộ nhớ và đĩa cứng hệ thống | Cần kết hợp xu hướng trong một khoảng thời gian |
| `jcmd`             | Thread stack, thông tin Heap, JFR, histogram class | Một số lệnh chi phí khá cao, đánh giá trước khi thực thi |
| `jstat`            | Biến đổi GC và từng vùng bộ nhớ | Sampling liên tục có ích hơn dữ liệu đơn điểm |
| JFR, JMC           | Phân tích event CPU, allocation, lock, I/O v.v. | Cấu hình thu thập phải kết hợp chi phí online |
| MAT, VisualVM      | Phân tích Heap Dump            | Trọng điểm xem chuỗi reference và GC Roots |
| Arthas             | Chẩn đoán thời gian tiêu tốn phương thức, thread, class và runtime | Phương thức traffic cao thực thi Trace/Watch phải giới hạn phạm vi |
| Trace/Platform giám sát | Call chain request, tỷ lệ lỗi, latency và phụ thuộc | Chú ý tỷ lệ sampling, khử nhạy (masking) và đồng bộ thời gian |

## Khuyến nghị đọc thêm

- [Tóm tắt các công cụ giám sát và xử lý sự cố JDK](./jdk-monitoring-and-troubleshooting-tools.md)
- [Tóm tắt các tham số JVM quan trọng nhất](./jvm-parameters-intro.md)
- [Giải thích chi tiết JVM Garbage Collection](./jvm-garbage-collection.md)
- [Giải thích chi tiết Java Thread Pool](../concurrent/java-thread-pool-summary.md)
- [Giải thích chi tiết Deadlock](../../cs-basics/operating-system/dead-lock.md)
- [Hướng dẫn rà soát sự cố Oracle Java: Rò rỉ bộ nhớ](https://docs.oracle.com/en/java/javase/17/troubleshoot/troubleshooting-memory-leaks.html)
- [Rà soát vấn đề độ trễ Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/latency/)
- [Redis Slow Log](https://redis.io/docs/latest/commands/slowlog/)
- [Phân tích một sự cố OOM online](https://juejin.cn/post/7205141492264976445)
- [Phân tích và giải quyết 9 loại vấn đề CMS GC thường gặp trong Java](https://tech.meituan.com/2020/11/12/java-9-cms-gc.html)

<!-- @include: @article-footer.snippet.md -->
