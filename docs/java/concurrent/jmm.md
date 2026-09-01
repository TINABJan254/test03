---
title: JMM（Java 内存模型）详解
description: 深入解析Java内存模型JMM：详解CPU缓存模型、指令重排序机制、happens-before原则、内存可见性保证，理解多线程并发编程的底层规范。
category: Java
tag:
  - Java并发
head:
  - - meta
    - name: keywords
      content: JMM,Java内存模型,CPU缓存,指令重排序,happens-before,内存可见性,并发编程模型
---

Đối với Java, bạn có thể xem **JMM (Java Memory Model - Mô hình bộ nhớ Java)** như một tập hợp các quy chuẩn liên quan đến lập trình concurrency được Java định nghĩa. Ngoài việc trừu tượng hóa mối quan hệ giữa các luồng và bộ nhớ chính (Main Memory), nó còn quy định từ mã nguồn Java đến các lệnh CPU có thể thực thi phải tuân thủ những nguyên tắc và quy chuẩn liên quan đến concurrency nào. Mục đích chính của nó là để **đơn giản hóa lập trình đa luồng**, **tăng cường tính di động (portability) của chương trình**.

JMM chủ yếu định nghĩa đối với một biến dùng chung, khi một luồng thực hiện thao tác ghi, **tính nhìn thấy (visibility)** của biến đó đối với các luồng khác sẽ như thế nào.

Muốn hiểu thấu đáo JMM, chúng ta cần bắt đầu từ **Mô hình CPU Cache** và **Sắp xếp lại lệnh (Instruction Reordering)**.

## Bắt đầu từ mô hình CPU Cache

**Tại sao lại cần phải tạo ra CPU High-Speed Cache (Bộ nhớ đệm tốc độ cao)?** Tương tự như việc chúng ta sử dụng Cache (như Redis) khi phát triển hệ thống backend website là để giải quyết vấn đề tốc độ xử lý của chương trình và tốc độ truy cập cơ sở dữ liệu quan hệ thông thường không đồng đều nhau. **CPU Cache là để giải quyết vấn đề tốc độ xử lý của CPU và tốc độ xử lý của bộ nhớ (RAM) không đồng đều nhau.**

Chúng ta thậm chí có thể xem **bộ nhớ (RAM) là Cache tốc độ cao của bộ nhớ ngoài (HDD/SSD)**, khi chương trình chạy chúng ta copy dữ liệu của bộ nhớ ngoài vào bộ nhớ (RAM), do tốc độ xử lý của bộ nhớ cao hơn nhiều so với bộ nhớ ngoài, nên đã nâng cao tốc độ xử lý.

Tóm lại: **CPU Cache lưu tạm dữ liệu bộ nhớ (RAM) dùng để giải quyết vấn đề tốc độ xử lý CPU và bộ nhớ không khớp nhau; bộ nhớ (RAM) lưu tạm dữ liệu ổ cứng dùng để giải quyết vấn đề tốc độ truy cập ổ cứng quá chậm.**

Để hiểu rõ hơn, tôi đã vẽ một sơ đồ mô hình CPU Cache đơn giản như dưới đây.

> **🐛 Sửa đổi (Tham khảo: [issue#1848](https://github.com/Snailclimb/JavaGuide/issues/1848))**: Hoàn thiện những điểm chưa nghiêm ngặt trong sơ đồ mô hình CPU Cache.

![CPU 缓存模型示意图](https://oss.javaguide.cn/github/javaguide/java/concurrent/cpu-cache.png)

CPU Cache hiện đại thường chia thành 3 tầng, lần lượt gọi là L1, L2, L3 Cache. Một số CPU có thể còn có L4 Cache, ở đây không thảo luận, không phổ biến.

**Cách thức hoạt động của CPU Cache:** Trước tiên copy một bản sao dữ liệu vào CPU Cache, khi CPU cần dùng đến thì có thể trực tiếp đọc dữ liệu từ CPU Cache, sau khi tính toán xong, lại ghi dữ liệu tính toán được trở lại Main Memory. Tuy nhiên, như vậy tồn tại **vấn đề tính không nhất quán của bộ nhớ đệm (Cache Inconsistency)**! Ví dụ nếu tôi thực thi một thao tác i++, nếu hai luồng đồng thời thực thi, giả sử hai luồng đều đọc i=1 từ CPU Cache, hai luồng làm xong tính toán i++ rồi ghi trở lại Main Memory thì i=2, mà kết quả đúng đáng lẽ phải là i=3.

**CPU để giải quyết vấn đề tính không nhất quán của bộ nhớ đệm có thể thông qua việc quy định Giao thức nhất quán bộ nhớ đệm (Cache Coherence Protocol, ví dụ [MESI Protocol](https://zh.wikipedia.org/wiki/MESI%E5%8D%8F%E8%AE%AE)) hoặc các biện pháp khác để giải quyết.** Giao thức nhất quán bộ nhớ đệm này chỉ các nguyên tắc và quy chuẩn cần tuân thủ khi CPU Cache tương tác với Main Memory. Trong các CPU khác nhau, giao thức nhất quán bộ nhớ đệm được sử dụng thường cũng sẽ khác nhau.

![缓存一致性协议](https://oss.javaguide.cn/github/javaguide/java/concurrent/cpu-cache-protocol.png)

Tính nhất quán của CPU Cache do bộ xử lý và hệ thống con bộ nhớ của nó phối hợp triển khai. Các kiến trúc bộ xử lý khác nhau còn quy định các loại truy cập bộ nhớ dưới góc nhìn của các bộ xử lý khác có thể xuất hiện theo thứ tự nào, điều này thường được gọi là Hardware Memory Model (Mô hình bộ nhớ phần cứng). JMM nằm ở tầng ngôn ngữ cao hơn, JVM cần ánh xạ các yêu cầu của nó thành các lệnh và barrier (hàng rào) do bộ xử lý cụ thể cung cấp.

## Sắp xếp lại lệnh (Instruction Reordering)

Nói xong mô hình CPU Cache, chúng ta lại xem một khái niệm khá quan trọng khác: **Sắp xếp lại lệnh (Instruction Reordering)**.

Để nâng cao tốc độ/hiệu năng thực thi, máy tính khi thực thi code chương trình sẽ tiến hành sắp xếp lại các lệnh.

**Sắp xếp lại lệnh là gì?** Nói đơn giản chính là hệ thống khi thực thi code không nhất thiết phải thực thi lần lượt theo đúng thứ tự code bạn viết.

Các trường hợp sắp xếp lại lệnh thường gặp gồm 2 loại dưới đây:

- **Sắp xếp lại do tối ưu trình biên dịch (Compiler Optimization Reordering)**: Trình biên dịch (bao gồm JVM, JIT compiler...) trong điều kiện không làm thay đổi ngữ nghĩa chương trình đơn luồng, sắp xếp lại thứ tự thực thi của các câu lệnh.
- **Sắp xếp lại do song song cấp lệnh (Instruction-Level Parallelism Reordering)**: Bộ xử lý hiện đại áp dụng kỹ thuật song song cấp lệnh (ILP) để chồng lấp thực thi nhiều lệnh. Nếu không tồn tại phụ thuộc dữ liệu (data dependency), bộ xử lý có thể thay đổi thứ tự thực thi của các lệnh máy tương ứng với câu lệnh.

Ngoài ra, hệ thống bộ nhớ cũng sẽ có "sắp xếp lại", nhưng lại không phải là sắp xếp lại theo đúng nghĩa đen. Trong JMM nó biểu hiện thành nội dung của main memory và local memory có thể không nhất quán, từ đó dẫn đến chương trình dưới đa luồng thực thi có thể xuất hiện vấn đề.

Mã nguồn Java trải qua biên dịch bytecode, thông dịch thực thi hoặc JIT biên dịch, cuối cùng do JVM thực thi các lệnh máy tương ứng trên platform mục tiêu. Tối ưu trình biên dịch, thực thi lộn xộn của bộ xử lý (out-of-order execution) cũng như hành vi của hệ thống con bộ nhớ đều có thể làm cho thứ tự quan sát được trong đa luồng khác với thứ tự trực quan trong mã nguồn, chúng không phải là một "pipeline sắp xếp lại" cố định và tuyến tính nghiêm ngặt.

**Sắp xếp lại lệnh có thể đảm bảo tính nhất quán ngữ nghĩa nối tiếp (serial semantics), nhưng không có nghĩa vụ đảm bảo ngữ nghĩa giữa các đa luồng cũng nhất quán**, nên trong đa luồng, sắp xếp lại lệnh có thể dẫn đến một số vấn đề.

Đối với sắp xếp lại do tối ưu trình biên dịch và sắp xếp lại lệnh của bộ xử lý (sắp xếp lại song song cấp lệnh và sắp xếp lại hệ thống bộ nhớ đều thuộc về sắp xếp lại lệnh cấp bộ xử lý), cách xử lý vấn đề này không giống nhau.

- Đối với trình biên dịch, thông qua cách cấm các loại sắp xếp lại trình biên dịch đặc định để cấm sắp xếp lại.
- Đối với bộ xử lý, thông qua cách chèn Memory Barrier (Hàng rào bộ nhớ, đôi khi gọi là Memory Fence) để cấm các loại sắp xếp lại bộ xử lý đặc định.

> Memory Barrier (Hàng rào bộ nhớ) dùng để ràng buộc thứ tự và tính nhìn thấy giữa các truy cập bộ nhớ đặc định. Cách triển khai cụ thể của các kiến trúc bộ xử lý khác nhau là không giống nhau, không thể hiểu đơn giản chung chung là "refresh cache ra bộ nhớ vật lý" hay "làm cho cache hoàn toàn vô hiệu"; JVM sẽ dựa trên platform mục tiêu để chọn các lệnh thỏa mãn ngữ nghĩa JMM.

## JMM (Java Memory Model)

### JMM là gì? Tại sao cần JMM?

Java là ngôn ngữ lập trình đầu tiên thử cung cấp mô hình bộ nhớ. Do mô hình bộ nhớ thời kỳ đầu tồn tại một số khuyết điểm (như rất dễ làm yếu năng lực tối ưu của trình biên dịch), bắt đầu từ Java 5, Java bắt đầu sử dụng mô hình bộ nhớ mới [《JSR-133: Java Memory Model and Thread Specification》](http://www.cs.umd.edu/~pugh/java/memoryModel/CommunityReview.pdf).

Nói chung, ngôn ngữ lập trình cũng có thể tái sử dụng trực tiếp mô hình bộ nhớ ở cấp độ hệ điều hành. Tuy nhiên, mô hình bộ nhớ của các hệ điều hành khác nhau lại khác nhau. Nếu tái sử dụng trực tiếp mô hình bộ nhớ cấp hệ điều hành, có thể dẫn đến cùng một bộ code đổi sang hệ điều hành khác lại không thể thực thi được. Ngôn ngữ Java là đa nền tảng (cross-platform), nó cần tự cung cấp một bộ mô hình bộ nhớ để che giấu sự khác biệt của hệ thống.

Đây chỉ là một trong các nguyên nhân tồn tại của JMM. Thực tế, đối với Java mà nói, bạn có thể xem JMM như một tập hợp các quy chuẩn liên quan đến lập trình concurrency do Java định nghĩa, ngoài việc trừu tượng hóa mối quan hệ giữa các luồng và bộ nhớ chính, nó còn quy định từ mã nguồn Java đến các lệnh CPU có thể thực thi phải tuân thủ những nguyên tắc và quy chuẩn liên quan đến concurrency nào, mục đích chính là để đơn giản hóa lập trình đa luồng, tăng cường tính di động của chương trình.

**Tại sao phải tuân thủ các nguyên tắc và quy chuẩn liên quan đến concurrency này?** Đó là vì trong lập trình concurrency, các thiết kế như CPU multi-level cache và sắp xếp lại lệnh có thể dẫn đến việc chương trình chạy xuất hiện một số vấn đề. Ví dụ như việc sắp xếp lại lệnh mà chúng ta đề cập ở trên có thể làm cho chương trình đa luồng thực thi gặp sự cố, vì vậy, JMM đã trừu tượng hóa nguyên tắc happens-before (phía sau sẽ giải thích chi tiết) để giải quyết vấn đề sắp xếp lại lệnh này.

JMM nói trắng ra chính là định nghĩa một số quy chuẩn để giải quyết các vấn đề này, nhà phát triển có thể tận dụng các quy chuẩn này để phát triển chương trình đa luồng thuận tiện hơn. Đối với Java developer mà nói, bạn không cần hiểu nguyên lý bên dưới, trực tiếp sử dụng một số từ khóa và lớp liên quan đến concurrency (như `volatile`, `synchronized`, các loại `Lock`) là có thể phát triển ra chương trình an toàn concurrency.

### JMM trừu tượng hóa mối quan hệ giữa các luồng và bộ nhớ chính như thế nào?

**Mô hình bộ nhớ Java (JMM)** trừu tượng hóa mối quan hệ giữa các luồng và bộ nhớ chính (Main Memory), ví dụ các biến dùng chung giữa các luồng bắt buộc phải lưu trữ trong Main Memory.

Java từ các quy chuẩn thời kỳ đầu đã có mô hình bộ nhớ; Java 5 thông qua JSR-133 đã tiến hành sửa đổi quan trọng đối với nó, làm rõ và tăng cường các ngữ nghĩa như `volatile`, `final` và happens-before. JMM cho phép JVM sử dụng thanh ghi (register), cache cũng như tối ưu trình biên dịch để triển khai việc truy cập biến dùng chung; nếu chương trình không thiết lập mối quan hệ đồng bộ cần thiết, một luồng có thể không nhìn thấy thao tác ghi mới nhất của luồng khác.

Điều này rất giống với mô hình CPU Cache mà chúng ta giảng ở trên.

**Main Memory là gì? Local Memory là gì?**

- **Main Memory (Bộ nhớ chính)**: JMM dùng nó trừu tượng hóa các biến có thể dùng chung giữa các luồng, bao gồm instance field, static field và phần tử mảng. Biến cục bộ phương thức, tham số phương thức và tham số xử lý ngoại lệ sẽ không dùng chung giữa các luồng, do đó không thuộc về biến dùng chung nói ở đây. Main Memory là sự trừu tượng ở cấp độ quy chuẩn, không tương đương với một khối bộ nhớ vật lý nào đó.
- **Local Memory (Bộ nhớ cục bộ / Bộ nhớ làm việc)**: Mỗi luồng đều có một Local Memory riêng tư, Local Memory lưu trữ bản sao của biến dùng chung mà luồng đó đã đọc / ghi. Mỗi luồng chỉ có thể thao tác trên biến trong Local Memory của chính mình, không thể truy cập trực tiếp Local Memory của luồng khác. Nếu giữa các luồng cần giao tiếp, bắt buộc phải thông qua Main Memory để tiến hành. Local Memory là một khái niệm được JMM trừu tượng hóa ra, không thực sự tồn tại, nó bao quát cả cache, write buffer, thanh ghi và các tối ưu phần cứng, trình biên dịch khác.

Sơ đồ trừu tượng của Mô hình bộ nhớ Java như sau:

![JMM(Java 内存模型)](https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm.png)

Nhìn từ hình trên, nếu Luồng 1 và Luồng 2 muốn giao tiếp với nhau, bắt buộc phải trải qua 2 bước dưới đây:

1. Luồng 1 đồng bộ giá trị bản sao biến dùng chung đã sửa đổi trong Local Memory của mình ra Main Memory.
2. Luồng 2 đến Main Memory đọc giá trị biến dùng chung tương ứng.

Nói cách khác, dữ liệu dùng chung giữa các luồng cần tuân thủ quy tắc của JMM để giao tiếp; chỉ khi thông qua `volatile`, lock, khởi động & chấm dứt luồng... thiết lập mối quan hệ happens-before tương ứng, JMM mới cung cấp đảm bảo tính nhìn thấy cho các thao tác ghi liên quan.

Tuy nhiên, trong đa luồng, thao tác trên một biến dùng chung trong Main Memory có khả năng gây ra vấn đề an toàn luồng. Lấy một ví dụ:

1. Luồng 1 và Luồng 2 lần lượt thao tác trên cùng một biến dùng chung, một luồng thực thi sửa đổi, một luồng thực thi đọc.
2. Luồng 2 đọc được giá trị trước khi Luồng 1 sửa đổi hay giá trị sau khi sửa đổi là không chắc chắn, đều có thể xảy ra, vì Luồng 1 và Luồng 2 đều copy biến dùng chung từ Main Memory vào Working Memory của luồng tương ứng trước.

Về giao thức tương tác cụ thể giữa Main Memory và Working Memory, tức là một biến làm sao copy từ Main Memory vào Working Memory, làm sao đồng bộ từ Working Memory ra Main Memory, chi tiết triển khai JMM định nghĩa 8 loại thao tác đồng bộ dưới đây (chỉ cần tìm hiểu, không cần học vẹt):

- **lock (khóa)**: Tác động lên biến trong Main Memory, đánh dấu nó thành biến độc chiếm bởi một luồng.
- **unlock (mở khóa)**: Tác động lên biến trong Main Memory, giải phóng trạng thái khóa của biến, biến được giải phóng trạng thái khóa mới có thể bị luồng khác lock.
- **read (đọc)**: Tác động lên biến trong Main Memory, nó truyền giá trị của một biến từ Main Memory vào Working Memory của luồng, để hành động load sau đó sử dụng.
- **load (nạp)**: Đưa giá trị biến lấy được từ Main Memory qua thao tác read vào bản sao biến trong Working Memory.
- **use (sử dụng)**: Truyền giá trị của một biến trong Working Memory cho execution engine, mỗi khi virtual machine gặp phải một lệnh sử dụng đến biến đều sẽ dùng lệnh này.
- **assign (gán giá trị)**: Tác động lên biến trong Working Memory, nó gán một giá trị nhận được từ execution engine cho biến trong Working Memory, mỗi khi virtual machine gặp phải một lệnh bytecode gán giá trị cho biến thì thực thi thao tác này.
- **store (lưu trữ)**: Tác động lên biến trong Working Memory, nó truyền giá trị của một biến trong Working Memory đến Main Memory, để thao tác write sau đó sử dụng.
- **write (ghi)**: Tác động lên biến trong Main Memory, nó đưa giá trị biến lấy được từ Working Memory qua thao tác store vào biến trong Main Memory.

Ngoài 8 thao tác đồng bộ này, còn quy định các quy tắc đồng bộ dưới đây để đảm bảo việc thực thi chính xác các thao tác đồng bộ này (chỉ cần tìm hiểu, không cần học vẹt):

- Không cho phép một luồng vô lý (không xảy ra bất kỳ thao tác assign nào) đồng bộ dữ liệu từ Working Memory của luồng trở lại Main Memory.
- Một biến mới chỉ có thể "sinh ra" trong Main Memory, không cho phép trong Working Memory trực tiếp sử dụng một biến chưa được khởi tạo (load hoặc assign), nói cách khác là trước khi thực thi thao tác use và store đối với một biến, bắt buộc phải thực thi xong thao tác assign và load trước.
- Một biến tại cùng một thời điểm chỉ cho phép một luồng tiến hành thao tác lock đối với nó, nhưng thao tác lock có thể bị cùng một luồng thực thi lặp lại nhiều lần, sau khi thực thi lock nhiều lần, chỉ khi thực thi thao tác unlock số lần tương ứng, biến mới được mở khóa.
- Nếu tiến hành thao tác lock đối với một biến, sẽ xóa rỗng giá trị của biến này trong Working Memory, trước khi execution engine sử dụng biến này, cần thực thi lại thao tác load hoặc assign để khởi tạo giá trị của biến.
- Nếu một biến trước đó không bị thao tác lock khóa lại, thì không cho phép thực thi thao tác unlock đối với nó, cũng không cho phép đi unlock một biến đang bị luồng khác lock.
- ……

### Vùng bộ nhớ Java (Java Memory Area) và JMM khác nhau thế nào?

Đây là một câu hỏi tương đối phổ biến, nhiều bạn mới học rất dễ nhầm lẫn. **Vùng bộ nhớ Java và Mô hình bộ nhớ Java là hai thứ hoàn toàn khác nhau**:

- Cấu trúc bộ nhớ JVM liên quan đến vùng runtime của Java Virtual Machine, định nghĩa cách JVM phân vùng lưu trữ dữ liệu chương trình khi chạy, ví dụ Heap chủ yếu dùng để chứa các thể hiện đối tượng.
- Mô hình bộ nhớ Java liên quan đến lập trình concurrency của Java, trừu tượng hóa mối quan hệ giữa các luồng và bộ nhớ chính ví dụ các biến dùng chung giữa các luồng bắt buộc phải lưu trữ trong Main Memory, quy định từ mã nguồn Java đến các lệnh CPU có thể thực thi phải tuân thủ những nguyên tắc và quy chuẩn liên quan đến concurrency nào, mục đích chính là để đơn giản hóa lập trình đa luồng, tăng cường tính di động của chương trình.

### Nguyên tắc happens-before là gì?

Khái niệm happens-before này xuất hiện sớm nhất trong luận văn của Leslie Lamport xuất bản năm 1978 [《Time, Clocks and the Ordering of Events in a Distributed System》](https://lamport.azurewebsites.net/pubs/time-clocks.pdf). Trong luận văn này, Leslie Lamport đã đề xuất khái niệm [Logical Clock](https://writings.sh/post/logical-clocks), đây cũng trở thành thuật toán đồng hồ logic đầu tiên. Trong môi trường phân tán, thông qua một chuỗi quy tắc để định nghĩa sự thay đổi của đồng hồ logic, từ đó có thể thông qua đồng hồ logic để phán đoán thứ tự trước sau của các sự kiện trong hệ thống phân tán. **Đồng hồ logic không đo lường bản thân thời gian, chỉ phân biệt thứ tự trước sau xảy ra sự kiện, bản chất của nó chính là định nghĩa một mối quan hệ happens-before.**

Bối cảnh ra đời của khái niệm happens-before đề cập ở trên không phải là trọng tâm, tìm hiểu đơn giản là được.

JSR 133 đưa vào khái niệm happens-before để mô tả tính nhìn thấy của bộ nhớ giữa hai thao tác.

**Tại sao cần nguyên tắc happens-before?** Nguyên tắc happens-before ra đời là vì sự cân bằng giữa programmer và compiler, processor. Programmer truy cầu mô hình bộ nhớ mạnh (Strong Memory Model) dễ hiểu và dễ lập trình, tuân thủ quy tắc đã định để code là được. Compiler và processor truy cầu mô hình bộ nhớ yếu (Weak Memory Model) ít ràng buộc hơn, để chúng tận lực tối ưu hóa hiệu năng, làm hiệu năng đạt tối đa. Tư tưởng thiết kế của nguyên tắc happens-before thực ra rất đơn giản:

- Để ràng buộc đối với compiler và processor ít nhất có thể, chỉ cần không làm thay đổi kết quả thực thi chương trình (chương trình đơn luồng và chương trình đa luồng thực thi đúng đắn), compiler và processor tiến hành tối ưu sắp xếp lại thế nào cũng được.
- Đối với việc sắp xếp lại sẽ làm thay đổi kết quả thực thi chương trình, JMM yêu cầu compiler và processor bắt buộc phải cấm loại sắp xếp lại đó.

Bức hình dưới đây là tôi vẽ lại dựa trên sơ đồ tư tưởng thiết kế JMM trong cuốn sách 《Nghệ thuật lập trình Java Concurrency》.

![ JMM 设计思想](https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm-design-idea.png)

Hiểu được tư tưởng thiết kế của nguyên tắc happens-before, chúng ta lại xem định nghĩa của JSR-133 đối với nguyên tắc happens-before:

- Nếu một thao tác happens-before một thao tác khác, vậy thì kết quả thực thi của thao tác thứ nhất sẽ hiển thị (nhìn thấy được) đối với thao tác thứ hai, và thứ tự thực thi của thao tác thứ nhất xếp trước thao tác thứ hai.
- Giữa hai thao tác tồn tại mối quan hệ happens-before, không có nghĩa là triển khai cụ thể của Java platform bắt buộc phải thực thi theo đúng thứ tự chỉ định bởi mối quan hệ happens-before. Nếu kết quả thực thi sau khi sắp xếp lại nhất quán với kết quả thực thi theo mối quan hệ happens-before, vậy thì JMM cũng cho phép việc sắp xếp lại như vậy.

Chúng ta xem đoạn code dưới đây:

```java
int userNum = getUserNum();   // 1
int teacherNum = getTeacherNum();   // 2
int totalNum = userNum + teacherNum;  // 3
```

- 1 happens-before 2
- 2 happens-before 3
- 1 happens-before 3

Mặc dù 1 happens-before 2, nhưng tiến hành sắp xếp lại 1 và 2 không ảnh hưởng đến kết quả thực thi code, nên JMM cho phép compiler và processor thực thi loại sắp xếp lại này. Nhưng 1 và 2 bắt buộc phải ở trước khi 3 thực thi, tức là 1,2 happens-before 3.

**Ý nghĩa mà nguyên tắc happens-before thể hiện thực ra không phải là một thao tác xảy ra ở phía trước một thao tác khác, mặc dù đứng ở góc độ programmer mà nói thì cũng không sao. Nói chính xác hơn, ý nghĩa nó muốn thể hiện hơn là kết quả của thao tác trước là hiển thị (nhìn thấy được) đối với thao tác sau, bất kể hai thao tác này có ở trong cùng một luồng hay không.**

Lấy một ví dụ: Thao tác 1 happens-before Thao tác 2, cho dù Thao tác 1 và Thao tác 2 không ở trong cùng một luồng, JMM cũng sẽ đảm bảo kết quả của Thao tác 1 hiển thị đối với Thao tác 2.

### Các quy tắc happens-before thường gặp là gì? Nói về hiểu biết của bạn?

Happens-before có nhiều quy tắc, dưới đây liệt kê 5 quy tắc thường dùng nhất:

1. **Quy tắc thứ tự chương trình (Program Order Rule)**: Trong một luồng, theo thứ tự code, thao tác viết ở phía trước happens-before thao tác viết ở phía sau;
2. **Quy tắc khóa monitor (Monitor Lock Rule)**: Việc mở khóa (unlock) đối với một monitor happens-before việc cài khóa (lock) tiếp theo đối với cùng monitor đó;
3. **Quy tắc biến volatile (Volatile Variable Rule)**: Thao tác ghi đối với một biến `volatile` happens-before thao tác đọc tiếp theo đối với cùng biến đó;
4. **Quy tắc bắc cầu (Transitivity Rule)**: Nếu A happens-before B, và B happens-before C, vậy thì A happens-before C;
5. **Quy tắc khởi động luồng (Thread Start Rule)**: Phương thức `start()` của đối tượng Thread happens-before mỗi một hành động của luồng này.

Danh sách này chưa hoàn chỉnh, còn bao gồm các quy tắc chấm dứt luồng, ngắt luồng... Giữa hai truy cập xung đột nếu không thể thông qua quy tắc hoàn chỉnh để suy ra mối quan hệ happens-before, có thể cấu thành Data Race (tranh chấp dữ liệu), tính nhìn thấy và thứ tự của nó không thể dựa vào trực giác đơn luồng để đảm bảo; điều này không tương đương với việc JVM có thể vô điều kiện trao đổi hai lệnh tùy ý, kết quả thực thi vẫn chịu sự ràng buộc của quy tắc nhất quán và tính nhân quả của JMM.

### Happens-before và JMM có mối quan hệ gì?

Mối quan hệ giữa happens-before và JMM như hình dưới đây:

![jmm-vs-happens-before](https://oss.javaguide.cn/github/javaguide/java/concurrent/jmm-vs-happens-before.png)

- JMM cung cấp cho programmer **"Quy tắc happens-before"** (như quy tắc thứ tự chương trình, quy tắc biến `volatile`...). Đây là một ảo tưởng **"Mô hình bộ nhớ mạnh"**: Programmer không cần quan tâm chi tiết sắp xếp lại phức tạp bên dưới, chỉ cần lập trình theo các quy tắc này là có thể đảm bảo tính nhìn thấy của bộ nhớ dưới đa luồng.
- JVM khi thực thi, sẽ ánh xạ quy tắc happens-before vào triển khai cụ thể. Để không làm mất hiệu năng trên tiền đề đảm bảo tính đúng đắn, JMM chỉ **"cấm các sắp xếp lại ảnh hưởng đến kết quả thực thi"**. Đối với các sắp xếp lại không ảnh hưởng đến kết quả thực thi đơn luồng, JMM là cho phép.
- Tầng dưới cùng nhất là **"Quy tắc sắp xếp lại"** thực sự của compiler và processor.

Tóm lại, JMM giống như một tầng trung gian: Hướng lên trên thông qua happens-before cung cấp mô hình lập trình đơn giản cho programmer; hướng xuống dưới thông qua việc cấm các sắp xếp lại đặc định, tận dụng hiệu năng phần cứng bên dưới. Thiết kế này vừa đảm bảo an toàn đa luồng, vừa giải phóng tối đa hiệu năng phần cứng.

## Nhìn lại ba đặc tính quan trọng của lập trình Concurrency

### Tính nguyên tử (Atomicity)

Một thao tác hoặc nhiều thao tác, hoặc là tất cả các thao tác đều được thực thi và không chịu sự can thiệp của bất kỳ yếu tố nào làm gián đoạn, hoặc là đều không thực thi.

Trong Java, có thể nhờ vào `synchronized`, các loại `Lock` cũng như các lớp Atomic để triển khai tính nguyên tử.

`synchronized` và các loại `Lock` có thể đảm bảo tại bất kỳ thời điểm nào chỉ có một luồng truy cập khối code đó, do đó có thể bảo đảm tính nguyên tử. Các lớp Atomic tận dụng thao tác CAS (compare and swap) (có thể cũng dùng đến từ khóa `volatile` hoặc `final`) để đảm bảo thao tác nguyên tử.

### Tính nhìn thấy (Visibility)

Khi một luồng tiến hành sửa đổi đối với một biến dùng chung, vậy thì các luồng khác đều lập tức có thể nhìn thấy giá trị mới nhất sau khi sửa đổi.

Trong Java, có thể nhờ vào `synchronized`, `volatile` cũng như các loại `Lock` để triểnkai tính nhìn thấy.

Nếu khai báo biến là `volatile`, thao tác ghi đối với biến đó và thao tác đọc tiếp theo sẽ thiết lập mối quan hệ happens-before. JVM bắt buộc phải đảm bảo tính nhìn thấy và ngữ nghĩa thứ tự tương ứng, nhưng triển khai cụ thể không yêu cầu mỗi lần đều phải truy cập Main Memory vật lý.

### Tính có thứ tự (Ordering)

Do vấn đề sắp xếp lại lệnh, thứ tự thực thi của code chưa chắc đã là thứ tự khi viết code.

Chúng ta phía trên khi giảng về sắp xếp lại cũng đã đề cập:

> **Sắp xếp lại lệnh có thể đảm bảo tính nhất quán ngữ nghĩa nối tiếp, nhưng không có nghĩa vụ đảm bảo ngữ nghĩa giữa các đa luồng cũng nhất quán**, nên trong đa luồng, sắp xếp lại lệnh có thể dẫn đến một số vấn đề.

Trong Java, `volatile` sẽ ràng buộc các sắp xếp lại liên quan đến đọc ghi biến đó mà có thể phá hỏng ngữ nghĩa bộ nhớ của nó, nhưng không phải cấm tất cả các tối ưu sắp xếp lại lệnh.

## Tóm tắt

- Java là ngôn ngữ đầu tiên thử cung cấp mô hình bộ nhớ, mục đích chính của nó là để đơn giản hóa lập trình đa luồng, tăng cường tính di động của chương trình.
- CPU có thể thông qua việc quy định giao thức nhất quán bộ nhớ đệm (ví dụ [MESI Protocol](https://zh.wikipedia.org/wiki/MESI%E5%8D%8F%E8%AE%AE)) để giải quyết vấn đề không nhất quán bộ nhớ đệm.
- Để nâng cao tốc độ/hiệu năng thực thi, máy tính khi thực thi code chương trình sẽ tiến hành sắp xếp lại các lệnh. Nói đơn giản chính là hệ thống khi thực thi code không nhất thiết phải thực thi lần lượt theo đúng thứ tự code bạn viết. **Sắp xếp lại lệnh có thể đảm bảo tính nhất quán ngữ nghĩa nối tiếp, nhưng không có nghĩa vụ đảm bảo ngữ nghĩa giữa các đa luồng cũng nhất quán**, nên trong đa luồng, sắp xếp lại lệnh có thể dẫn đến một số vấn đề.
- Bạn có thể xem JMM như một tập hợp các quy chuẩn liên quan đến lập trình concurrency do Java định nghĩa, ngoài việc trừu tượng hóa mối quan hệ giữa các luồng và bộ nhớ chính, nó còn quy định từ mã nguồn Java đến các lệnh CPU có thể thực thi phải tuân thủ những nguyên tắc và quy chuẩn liên quan đến concurrency nào, mục đích chính là để đơn giản hóa lập trình đa luồng, tăng cường tính di động của chương trình.
- JSR 133 đưa vào khái niệm happens-before để mô tả tính nhìn thấy của bộ nhớ giữa hai thao tác.

## Tham khảo

- 《Nghệ thuật lập trình Java Concurrency》 Chương 3 Mô hình bộ nhớ Java
- 《Dễ hiểu Java đa luồng》: <http://concurrent.redspider.group/RedSpider.html>
- Nghiên cứu về sắp xếp lại truy cập bộ nhớ Java: <https://tech.meituan.com/2014/09/23/java-memory-reordering.html>
- Này bạn, Mô hình bộ nhớ Java (JMM) bạn cần đến rồi đây: <https://xie.infoq.cn/article/739920a92d0d27e2053174ef2>
- JSR 133 (Java Memory Model) FAQ: <https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html>

<!-- @include: @article-footer.snippet.md -->
