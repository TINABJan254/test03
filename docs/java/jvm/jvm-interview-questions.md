---
title: 2026 最新 JVM 面试题总结：内存区域、类加载、垃圾回收与线上排查
category: Java
description: 2026 最新 JVM 面试题总结，覆盖运行时内存区域、对象创建、类文件与类加载、垃圾回收算法与收集器、JVM 参数、JDK 诊断工具、OOM、频繁 Full GC 和 CPU 飙高等高频考点。
tag:
  - Java
  - JVM
  - 面试题
head:
  - - meta
    - name: keywords
      content: JVM面试题,Java内存区域,对象创建,类加载,双亲委派,垃圾回收,GC面试题,G1,ZGC,JVM参数,OOM,Full GC,CPU飙高,JVM调优
---

Phỏng vấn JVM rất ít khi dừng lại ở "Heap và Stack khác nhau như thế nào". Sau khi trả lời xong về vùng bộ nhớ, người phỏng vấn thông thường sẽ tiếp tục hỏi đối tượng được cấp phát ra sao, những đối tượng nào có thể thu hồi, một lần GC tại sao lại bị tạm dừng (pause), cũng như khi online xuất hiện OOM, Full GC thường xuyên hoặc CPU tăng vọt rà soát như thế nào.

Bài viết này là cổng ôn tập phỏng vấn cho chuyên đề JVM của JavaGuide, các câu hỏi được sắp xếp theo 5 phần: Bộ nhớ & Đối tượng, File Class & Class Loading, Garbage Collection, Tham số & Công cụ chẩn đoán, Rà soát sự cố Online. Đáp án hoàn chỉnh của từng câu hỏi nằm trong các bài viết chuyên đề tương ứng.

Nếu thời gian tương đối gấp, trước tiên có thể xem [Tóm tắt các câu hỏi phỏng vấn JVM thường gặp](https://interview.javaguide.cn/java/java-jvm.html), đánh dấu các câu hỏi trả lời chưa hoàn chỉnh, rồi quay lại bài viết này và bài viết chuyên đề để bổ sung chi tiết.

## Khi ôn tập JVM nên nắm vững những vấn đề nào trước?

| Module         | Nội dung cần trình bày rõ ràng                    | Hướng hỏi sâu thường gặp                              |
| -------------- | ------------------------------------------------- | ------------------------------------------------------ |
| Bộ nhớ & Đối tượng | Runtime Data Area phân chia thế nào, đối tượng được tạo, lưu trữ và truy cập ra sao | Heap, Stack, Method Area, Direct memory, Header đối tượng, TLAB, OOM |
| File Class & Class Loading | File `.class` đi vào JVM như thế nào, Class hoàn thành khởi tạo ở mốc thời gian nào | Constant Pool, quá trình Class Loading, ClassLoader, Parents Delegation, cách ly Class |
| Garbage Collection | Làm sao phán đoán đối tượng còn sống, bộ nhớ thu hồi ra sao, tạm dừng sinh ra từ đâu | GC Roots, các loại Reference, thuật toán thu gom, CMS, G1, ZGC, Full GC |
| Tham số & Công cụ | Tham số được lựa chọn ra sao theo JDK, Collector và hiện tượng vận hành | `-Xms`, `-Xmx`, GC log, Heap Dump, jstat, jstack, JFR |
| Rà soát Online | Sau khi nhận cảnh báo làm sao giữ chứng cứ, thu hẹp phạm vi và xác minh sửa lỗi | CPU, Load, OOM, rò rỉ bộ nhớ, GC thường xuyên, thread bị block |

Phần lớn các instance đối tượng được cấp phát trong Heap, active thread, field static v.v. ở các vị trí có thể nắm giữ reference tới đối tượng; GC xuất phát từ GC Roots để phán đoán xem đối tượng có thể tới được hay không, sau đó do collector cụ thể hoàn thành thu hồi. Khi ứng dụng xuất hiện bộ nhớ tăng cao hoặc tạm dừng, còn phải kết hợp GC log, thread stack, Heap dump và chỉ số hệ thống để phán đoán nguyên nhân, không thể thấy Full GC là sửa trực tiếp kích thước Heap.

## Vùng bộ nhớ JVM và Đối tượng

Khi trả lời về Runtime Data Area chỉ nói tới tên là chưa đủ. Mỗi vùng do những thread nào dùng chung, lưu trữ những gì, vòng đời ra sao, có thể ném ra exception gì, đều có thể trở thành câu hỏi đào sâu tiếp theo. Method Area là vùng logic được định nghĩa trong quy chuẩn JVM, còn Permanent Generation và Metaspace là triển khai của HotSpot trong các phiên bản khác nhau, khi trả lời đừng nhầm lẫn thành cùng một khái niệm.

Nội dung liên quan: [Giải thích chi tiết vùng bộ nhớ Java](./memory-area.md)

Các câu hỏi phỏng vấn thường gặp:

- Runtime Data Area của JVM bao gồm những phần nào? Những vùng nào là riêng của thread (Thread-private)?
- Program Counter, Virtual Machine Stack và Native Method Stack lần lượt có tác dụng gì?
- Một phương thức Java từ khi gọi đến khi trả về, Stack Frame có những thay đổi gì?
- `StackOverflowError` và `OutOfMemoryError` lần lượt có thể sinh ra như thế nào?
- Java Heap chủ yếu lưu trữ cái gì? Tất cả các đối tượng có nhất định được cấp phát trên Heap không?
- Method Area, Permanent Generation và Metaspace có quan hệ gì? Tại sao phải loại bỏ Permanent Generation?
- Constant Pool runtime và String Constant Pool có điểm gì khác nhau? Vị trí của chúng từng có những thay đổi gì?
- Direct memory có thuộc về Runtime Data Area của JVM không? Tại sao nó cũng có thể ném ra OOM?
- HotSpot tạo một đối tượng phải trải qua những bước nào?
- Bump the Pointer (Con trỏ va chạm) và Free List (Danh sách rảnh) được lựa chọn ra sao? Khi cấp phát đối tượng đồng thời, CAS và TLAB lần lượt giải quyết vấn đề gì?
- Header đối tượng, dữ liệu instance và padding căn lề lần lượt lưu trữ cái gì?
- Truy cập đối tượng bằng Handle (Tay cầm) và Direct Pointer (Con trỏ trực tiếp) có điểm gì khác nhau? HotSpot chủ yếu sử dụng loại nào?

Khi gặp câu hỏi OOM, trước tiên xác nhận bộ nhớ cạn kiệt là Java Heap, Metaspace, Direct memory, hay là bộ nhớ bản địa cần thiết để tạo thread. Chỉ trả lời "điều chỉnh `-Xmx` lớn lên" sẽ bỏ sót các nguyên nhân như collection không giới hạn, rò rỉ ClassLoader, Direct memory chưa giải phóng và số lượng thread mất kiểm soát.

## File Class và Class Loading

Câu hỏi về Class Loading thông thường bắt đầu từ "một Class được nạp như thế nào", sau đó hỏi sâu về mốc thời gian khởi tạo, Parents Delegation cũng như sự cách ly giữa các ClassLoader khác nhau. Ở đây vừa có vòng đời trong quy chuẩn JVM, vừa có triển khai cụ thể của HotSpot và JDK ClassLoader, khi mô tả sự khác biệt giữa các phiên bản phải nói rõ JDK được sử dụng.

Nội dung liên quan:

- [Giải thích chi tiết cấu trúc file Class](./class-file-structure.md)
- [Giải thích chi tiết quá trình Class Loading](./class-loading-process.md)
- [Giải thích chi tiết ClassLoader](./classloader.md)

Các câu hỏi phỏng vấn thường gặp:

- File Class được cấu thành từ những phần nào? Magic Number, phiên bản và Constant Pool lần lượt có tác dụng gì?
- Lệnh bytecode của phương thức lưu ở vị trí nào trong file Class?
- Một Class từ khi được nạp đến khi gỡ bỏ phải trải qua những giai đoạn nào?
- Loading, Verification, Preparation, Resolution và Initialization lần lượt làm những gì?
- Gán giá trị cho biến static ở giai đoạn Preparation và giai đoạn Initialization có điểm gì khác nhau?
- Những trường hợp nào sẽ kích hoạt khởi tạo Class? Truy cập hằng số compile-time có kích hoạt không?
- Phương thức `<clinit>` và constructor `<init>` có điểm gì khác nhau?
- Bootstrap ClassLoader, Platform ClassLoader và App ClassLoader lần lượt nạp những Class nào? Extension ClassLoader trong JDK 8 có điểm gì khác biệt?
- Parents Delegation Model là gì? Nó giúp tránh việc các core class bị nạp lặp lại hoặc thay thế như thế nào?
- Parents Delegation được thực thi ra sao? Khi ClassLoader cha không thể hoàn thành việc nạp sẽ xảy ra điều gì?
- Những kịch bản nào sẽ phá vỡ Parents Delegation? Thread Context ClassLoader giải quyết vấn đề gì?
- Hai Class có FQCN giống nhau thì có nhất định là cùng một Class không?
- Custom ClassLoader thông thường override `findClass()` hay `loadClass()`?

Trả lời Parents Delegation thành một chuỗi tìm kiếm ngược lên cố định vẫn là chưa hoàn chỉnh. Định danh của Class do FQCN của Class và ClassLoader nạp nó cùng xác định; các kịch bản như SPI, cách ly Class của ứng dụng server, hot deployment sẽ sử dụng các phương thức nạp khác nhau, mục đích cũng không hoàn toàn giống nhau.

## Garbage Collection

Câu hỏi về Garbage Collection phải trình bày từ cấp phát đối tượng và phán đoán sống sót tới các collector cụ thể. Giữa Throughput, thời gian tạm dừng (pause time) và mức chiếm dụng bộ nhớ tồn tại sự đánh đổi, khi lựa chọn collector còn phải cân nhắc phiên bản JDK, kích thước Heap và yêu cầu của nghiệp vụ đối với độ trễ, không thể chỉ so sánh tên thuật toán.

Nội dung liên quan: [Giải thích chi tiết JVM Garbage Collection](./jvm-garbage-collection.md)

Các câu hỏi phỏng vấn thường gặp:

- Young Generation và Old Generation thông thường phân chia ra sao? Đối tượng thông thường được cấp phát và thăng tiến như thế nào?
- Đối tượng lớn và đối tượng sống lâu năm sẽ vào Old Generation như thế nào?
- Đảm bảo cấp phát không gian (Handle Promotion / Space Guarantee) là gì?
- Phương pháp đếm reference tại sao khó giải quyết việc reference vòng giữa các đối tượng?
- Phân tích tính khả đạt (Reachability Analysis) phán đoán đối tượng còn sống ra sao? Những đối tượng nào có thể làm GC Roots?
- Strong Reference, Soft Reference, Weak Reference và Phantom Reference khác nhau như thế nào?
- Thuật toán Mark-Sweep, Copying và Mark-Compact mỗi loại có ưu nhược điểm gì?
- Thu gom phân thế tại sao phải chọn các thuật toán khác nhau cho Young Generation và Old Generation?
- Minor GC, Major GC và Full GC khác nhau như thế nào? Tại sao phải kết hợp collector và log để hiểu những khái niệm này?
- Mục tiêu và kịch bản áp dụng của Serial, Parallel, CMS, G1 và ZGC khác nhau như thế nào?
- CMS tại sao lại tạo ra mảnh vụn bộ nhớ (memory fragmentation) và rác lơ lửng (floating garbage)?
- G1 kiểm soát tạm dừng thông qua Region, dự đoán giá trị thu hồi và Mixed GC như thế nào?
- ZGC giảm thời gian tạm dừng dài bằng cách nào? Collector tạm dừng thấp phải trả những cái giá nào?
- Những trường hợp nào có thể kích hoạt Full GC? Full GC thường xuyên nên bắt đầu rà soát từ những dữ liệu nào?

Tên event, phân vùng bộ nhớ và tham số trong GC log sẽ biến đổi theo JDK và collector. Trong phỏng vấn trước tiên có thể nêu rõ môi trường chạy của mình, sau đó mới giải thích nguyên nhân kích hoạt một lần thu hồi, phạm vi thu hồi, giai đoạn tạm dừng và kết quả; thoát khỏi phiên bản mà học thuộc lòng kết luận cố định rất dễ dẫn tới mâu thuẫn khi bị hỏi sâu.

## Tham số JVM và Công cụ chẩn đoán

Câu hỏi tham số khảo sát căn cứ cấu hình. Kích thước Heap, tỷ lệ Young Generation, collector và tham số log phải cùng thảo luận với bộ nhớ triển khai, tốc độ cấp phát đối tượng, mục tiêu độ trễ và phiên bản JDK. Đặc biệt là G1, thông thường không khuyến nghị bê nguyên cấu hình cũ xoay quanh kích thước Young Generation cố định.

Nội dung liên quan:

- [Tóm tắt các tham số thường dùng của JVM](./jvm-parameters-intro.md)
- [Tóm tắt các công cụ giám sát và xử lý sự cố JDK](./jdk-monitoring-and-troubleshooting-tools.md)

Các câu hỏi phỏng vấn thường gặp:

- `-Xms`, `-Xmx`, `-Xmn` và `-Xss` lần lượt kiểm soát cái gì?
- Tại sao ứng dụng phía Server thường thiết lập `-Xms` và `-Xmx` thành cùng một giá trị?
- `-XX:MetaspaceSize` và `-XX:MaxMetaspaceSize` khác nhau như thế nào?
- Lựa chọn Garbage Collector cho ứng dụng ra sao? Trước khi chuyển đổi collector cần thu thập những dữ liệu gì?
- JDK 8 và JDK 9 trở đi cấu hình GC log như thế nào?
- Khi xảy ra OOM làm sao tự động tạo Heap Dump? Tại sao phải xác nhận trước không gian lưu trữ?
- `jps`, `jstat`, `jinfo`, `jmap`, `jstack` và `jcmd` lần lượt thích hợp để tra cứu cái gì?
- Dùng `jstat` quan sát tần suất GC, dung lượng các vùng và tình hình thăng tiến đối tượng như thế nào?
- Khi Thread bị block lâu, deadlock hoặc CPU tăng vọt, Thread dump có thể cung cấp những thông tin gì?
- Heap Dump và Thread dump có điểm gì khác nhau? Lệnh `kill -3` tạo ra loại nào?
- MAT khi phân tích Heap Dump, Dominator Tree và chuỗi reference tới GC Roots lần lượt có tác dụng gì?
- JFR, JMC, VisualVM thích hợp quan sát những thông tin runtime nào? Khi thu thập online cần cân nhắc chi phí gì?

Tạo Heap Dump, histogram class hoặc sampling tần suất cao đều có khả năng làm tăng tải online, file Dump cũng có thể chứa dữ liệu nghiệp vụ. Trước khi thực thi lệnh chẩn đoán cần xác nhận phạm vi ảnh hưởng, dung lượng đĩa và vị trí lưu file; trước khi khôi phục khẩn cấp bằng restart, tối thiểu giữ lại thời gian cảnh báo, chỉ số then chốt, thread stack và thông tin GC cần thiết.

## Rà soát sự cố JVM Online

Rà soát online thông thường không có đáp án "dùng một lệnh định vị trực tiếp". Nhận được cảnh báo trước tiên đối chiếu thời gian, phạm vi ảnh hưởng, thay đổi gần đây và tài nguyên hệ thống, rồi mới quyết định thu thập thread stack, GC log hay heap dump. Nếu service đã ảnh hưởng tới người dùng, việc cầm máu và giữ chứng cứ cần sắp xếp đồng thời, không thể vì muốn bắt đầy đủ chứng cứ mà để sự cố tiếp tục mở rộng.

Nội dung liên quan: [Rà soát sự cố Java Backend Online](./jvm-in-action.md)

Các câu hỏi phỏng vấn thường gặp:

- Khi CPU tăng vọt, làm sao định vị từ process tới thread và stack code cụ thể?
- CPU không cao nhưng Load rất cao, nên kiểm tra những trạng thái thread và chỉ số hệ thống nào?
- Response của interface rất chậm nhưng tỷ lệ sử dụng CPU bình thường, phân biệt lock wait, I/O, connection pool và downstream latency như thế nào?
- Bộ nhớ tăng liên tục, làm sao phán đoán là business cache tăng, cấp phát đối tượng quá nhanh hay rò rỉ bộ nhớ?
- Java process sau khi OOM, làm sao phán đoán vấn đề xảy ra ở Heap, Metaspace, Direct memory hay Native memory?
- Young GC rất thường xuyên, nhưng mỗi lần thu hồi rất nhanh, nên trọng điểm kiểm tra cái gì?
- Full GC thường xuyên, làm sao quan sát tăng trưởng Old Gen, thăng tiến đối tượng và hiệu quả thu hồi?
- Heap Dump nên được tạo vào thời điểm nào? Môi trường production trực tiếp thực thi `jmap -dump` có rủi ro gì?
- Tích đọng queue thread pool có quan hệ gì với tỷ lệ sử dụng CPU, trạng thái thread?
- Sau khi sửa lỗi JVM, nên so sánh những chỉ số nào để xác nhận thay đổi có hiệu quả?
- Trong phỏng vấn trả lời một câu hỏi rà soát sự cố JVM mà mình chưa từng trực tiếp trải qua như thế nào?

Kết quả rà soát phải tạo thành một chuỗi chứng cứ (chain of evidence). Ví dụ khi CPU tăng vọt, trước tiên tìm Java process và thread có CPU cao, chuyển thread ID sang định dạng sử dụng trong Thread dump, rồi đối chiếu nhiều bản thread stack xem có tiếp tục rơi vào cùng một đoạn code hay không. Vấn đề bộ nhớ thì phải kết hợp GC log, histogram đối tượng, Heap Dump và bộ nhớ bản địa, chỉ xem tỷ lệ sử dụng JVM Heap không thể bao phủ tất cả OOM.

## Sắp xếp ôn tập theo thời gian chuẩn bị

| Thời gian còn lại | Khuyến nghị sắp xếp                                                                                                                                     | Mục tiêu ôn tập                                          |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| 1 ~ 2 ngày        | Lướt qua một lượt [Tóm tắt câu hỏi phỏng vấn JVM thường gặp](https://interview.javaguide.cn/java/java-jvm.html), ưu tiên bổ sung vùng bộ nhớ, tạo đối tượng, Class Loading, phán đoán sống sót đối tượng và các Collector chủ đạo | Có thể trả lời câu hỏi cơ bản tần suất cao, biết kết luận khác nhau tương ứng JDK và Collector nào |
| 3 ~ 7 ngày        | Bổ sung tham số JVM, GC log và công cụ chẩn đoán JDK, lần lượt trình bày miệng quá trình rà soát CPU tăng vọt, OOM và Full GC thường xuyên               | Có thể từ hiện tượng cảnh báo chọn chỉ số và công cụ, đồng thời nêu rõ rủi ro thao tác |
| Trên 1 tuần       | Đọc toàn bộ các bài viết chuyên đề JVM, tại bản địa hoặc môi trường test phân tích 1 lần GC log, Thread dump hoặc Heap Dump; kết hợp dự án ghi lại JDK thực tế, Collector, cấu hình Heap và chỉ số vận hành | Có thể kết nối nguyên lý, cấu hình, hiện tượng vận hành và phương pháp xác minh lại với nhau |

Phỏng vấn ứng tuyển kinh nghiệm (Social Recruitment) và các vị trí mid/senior thường tiếp tục hỏi sâu về căn cứ tham số và chứng cứ rà soát. Nếu chưa từng xử lý thực tế sự cố JVM online, có thể trả lời theo quan sát môi trường test và tài liệu chuyên đề, nêu rõ mình sẽ thu thập cái gì trước, thu hẹp phạm vi ra sao; đừng nói các ca học tập thành sự cố production do chính mình trải qua.

<!-- @include: @article-footer.snippet.md -->
