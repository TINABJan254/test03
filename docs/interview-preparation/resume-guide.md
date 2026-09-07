---
title: Hướng dẫn viết CV cho lập trình viên
description: Hướng dẫn viết CV cho lập trình viên: Xuất phát từ logic sàng lọc CV để làm rõ cấu trúc CV, cách viết kinh nghiệm dự án và mô tả kỹ năng, cung cấp mẫu CV và lời khuyên tránh bẫy, giúp bạn nâng cao tỷ lệ đậu CV và giúp người phỏng vấn đào sâu các điểm sáng của bạn tốt hơn.
category: Chuẩn bị phỏng vấn
icon: "mdi:account-tie-outline"
head:
  - - meta
    - name: keywords
      content: CV lập trình viên, CV Java, Tối ưu CV, Cách viết kinh nghiệm dự án, Mẫu CV, CV sinh viên, CV người có kinh nghiệm, Chuẩn bị phỏng vấn
---

::: tip Lời nhắc thân thiện
Bài viết này được trích từ **[《Java 面试指北》](../zhuanlan/java-mian-shi-zhi-bei.md)**. Đây là một cẩm nang hướng dẫn bạn cách chuẩn bị phỏng vấn hiệu quả hơn, bao gồm các câu hỏi phỏng vấn cốt lõi (Thiết kế hệ thống, Framework phổ biến, Hệ thống phân tán, High concurrency ……), các bài review phỏng vấn chất lượng cao, v.v.
:::

## Lời nói đầu

Một bản CV tốt có thể đóng vai trò cực kỳ quan trọng trong toàn bộ quá trình ứng tuyển và phỏng vấn.

**Tại sao nói CV lại quan trọng như vậy?** Chúng ta có thể nhìn nhận qua các điểm sau:

**1. CV giống như bộ mặt của chúng ta, nó quyết định phần lớn việc bạn có nhận được cơ hội phỏng vấn hay không.**

- Nếu bạn nộp đơn trực tuyến (apply online), CV của bạn chắc chắn sẽ qua vòng lọc của HR. Một bản CV, HR có thể chỉ dành khoảng 10 giây để lướt qua rồi quyết định bạn có được vào vòng phỏng vấn hay không.
- Nếu bạn được giới thiệu nội bộ (referral), nếu CV không có điểm nổi bật thì dù người giới thiệu có nhiệt tình đến đâu cũng đành bất lực.

Ngoài ra, ngay cả khi bạn đã vượt qua vòng lọc đầu tiên để nhận lời mời phỏng vấn, trong các vòng phỏng vấn tiếp theo, người phỏng vấn cũng sẽ dựa vào CV để đánh giá xem bạn có xứng đáng để họ dành nhiều thời gian phỏng vấn hay không.

**2. Nội dung trên CV quyết định phần lớn trọng tâm câu hỏi của người phỏng vấn.**

- Thông thường, những thứ ghi trên CV bạn biết mới được hỏi đến (Java cơ bản, Collections, Concurrency, MySQL, Redis, Spring, Spring Boot là những thứ gần như ai cũng bị hỏi). Ví dụ nếu ghi bạn sử dụng thành thạo Redis, người phỏng vấn rất có thể sẽ hỏi bạn các câu hỏi về Redis; nếu ghi bạn có sử dụng Message Queue trong dự án, người phỏng vấn rất có thể sẽ hỏi nhiều câu hỏi liên quan đến Message Queue.
- Mức độ thành thạo kỹ năng cũng quyết định phần lớn độ sâu của câu hỏi phỏng vấn.

Trong điều kiện không phóng đại năng lực của bản thân, viết được một bản CV chất lượng cũng là một năng lực tuyệt vời. Thông thường, những người có năng lực kỹ thuật và khả năng tự học tốt thì CV viết ra cũng rất ấn tượng!

## Mẫu CV (Resume Template)

Hình thức và bố cục của CV thực sự rất, rất quan trọng! Nếu phong cách trình bày CV của bạn xấu không thể tả, người phỏng vấn thực sự không có hứng thú để đọc tiếp. Nỗi khổ của việc xử lý hàng trăm bản CV mỗi ngày, bạn không hiểu được đâu!

Ở đây, tôi khuyên mọi người nên dùng cú pháp Markdown để viết CV, sau đó chuyển định dạng Markdown sang file PDF để nộp. Nếu bạn chưa hiểu rõ cú pháp Markdown, có thể dành nửa tiếng xem hướng dẫn cú pháp Markdown: <http://www.markdown.cn/>.

Dưới đây là một số mẫu CV khá tốt mà tôi đã tổng hợp:

- Mẫu CV phù hợp tiếng Trung/tiếng Việt (Khuyên dùng, open source miễn phí): <https://github.com/dyweb/awesome-resume-for-chinese>
- Muji CV (Khuyên dùng, một phần miễn phí): <https://www.mujicv.com/>
- Easy CV (Khuyên dùng, một phần miễn phí): <https://easycv.cn/>
- Polebrief CV (Miễn phí): <https://www.polebrief.com/index>
- Công cụ dàn trang CV Markdown (Open source miễn phí): <https://resume.mdnice.com/>
- Jianli Chinaz (Trả phí, hỗ trợ tạo bằng AI): <https://jianli.chinaz.com/>
- Typora + Markdown + CSS Custom Template: <https://github.com/Snailclimb/typora-markdown-resume>
- Wonder CV (Một phần trả phí): <https://www.wondercv.com/>

Hầu hết các mẫu CV trên chỉ có 1 trang, rất khó thể hiện đủ lượng thông tin. Nếu bạn không phải là "pro đỉnh cấp" (chẳng hạn như đạt giải ACM), tôi khuyên bạn nên viết nhiều hơn một chút các nội dung có thể làm nổi bật năng lực của mình (Sinh viên mới ra trường trong vòng 2 trang, người đã đi làm trong vòng 3 trang, nhớ cô đọng ngôn ngữ, không dài dòng vô ích).

Tổng kết một số **lưu ý về cách dàn trang CV**:

- Cố gắng súc tích, không nên quá màu mè hoa mỹ.
- Thuật ngữ kỹ thuật nên chuẩn hóa cách viết hoa - thường, ví dụ `java` -> `Java`, `spring boot` -> `Spring Boot`. Dù một số người phỏng vấn không để bụng, nhưng rất nhiều người phỏng vấn sẽ để ý chi tiết này.
- Giữa chữ và số/tiếng Anh nên có khoảng trắng để nhìn thoải mái hơn.

Ngoài ra, trong Tinh Cầu Tri Thức còn có các mẫu CV thực tế để tham khảo: <https://t.zsxq.com/12ypxGNzU> (cần tham gia [Tinh Cầu Tri Thức](https://javaguide.cn/about-the-author/zhishixingqiu-two-years.html)).

![](https://oss.javaguide.cn/javamianshizhibei/image-20230918073550606.png)

## Nội dung CV

### Thông tin cá nhân

- Cơ bản nhất: Họ tên, Tuổi, Số điện thoại, Địa chỉ/Quê quán, Email
- Điểm cộng tiềm năng: Địa chỉ GitHub, Địa chỉ Blog kỹ thuật (Nếu Blog và GitHub chưa có nội dung gì nổi bật thì không nên ghi)

Ví dụ:

![](https://oss.javaguide.cn/zhishixingqiu/20210428212337599.png)

**Có nên để ảnh trong CV không?** Rất nhiều người khi viết CV đều băn khoăn câu hỏi này.

Thực ra để hay không đều được, không ảnh hưởng nhiều, hoàn toàn không cần quá bận tâm. Trừ khi vị trí bạn ứng tuyển có yêu cầu rõ ràng phải kèm ảnh. Tuy nhiên, nếu để ảnh, đừng dùng ảnh đời thường mà nên dùng ảnh trang trọng, chỉn chu (ảnh thẻ/chân dung nghề nghiệp).

### Mục tiêu nghề nghiệp (Career Objective)

Vị trí bạn muốn ứng tuyển là gì, mong muốn làm việc ở thành phố nào. Ngoài ra, bạn cũng có thể gộp phần mục tiêu nghề nghiệp vào phần thông tin cá nhân.

Ví dụ:

![](https://oss.javaguide.cn/zhishixingqiu/20210428212410288.png)

### Quá trình học vấn (Education)

Quá trình học vấn cũng không thể thiếu. Qua phần này, bạn cần đảm bảo người phỏng vấn biết được bằng cấp, chuyên ngành, trường tốt nghiệp và ngày tốt nghiệp của bạn.

Ví dụ:

> Đại học Bách Khoa Hà Nội | Thạc sĩ, Kỹ thuật Phần mềm | 2019.09 - 2022.01
> Đại học Bách Khoa Hà Nội | Cử nhân, Công nghệ Thông tin | 2015.09 ~ 2019.06

### Kỹ năng chuyên môn (Professional Skills)

Trước hết hãy tự hỏi bản thân biết những gì, sau đó xem công ty mục tiêu cần những gì. Thông thường HR có thể không quá am hiểu kỹ thuật, nên khi lọc CV họ sẽ nhìn chăm chú vào các từ khóa kỹ năng chuyên môn. Đối với những kỹ năng công ty yêu cầu mà bạn chưa biết, bạn có thể dành vài ngày để tìm hiểu rồi ghi trong CV là mình "có hiểu biết" về kỹ năng đó.

Dưới đây là danh sách kỹ năng phát triển Java Backend mẫu, bạn có thể điều chỉnh linh hoạt theo tình hình bản thân và yêu cầu tuyển dụng, tư tưởng cốt lõi là cố gắng đáp ứng tối đa các yêu cầu kỹ năng của công việc.

![Mẫu kỹ năng Java Backend](https://oss.javaguide.cn/zhishixingqiu/jinengmuban.png)

Dưới đây là một phần giới thiệu kỹ năng của một bạn ứng viên, chúng ta cùng chỉ ra các vấn đề:

![](https://oss.javaguide.cn/zhishixingqiu/up-a58d644340f8ce5cd32f9963f003abe4233.png)

Các vấn đề tồn tại trong ảnh trên:

- Thuật ngữ kỹ thuật viết hoa - thường chưa chuẩn (ví dụ `java` -> `Java`, `spring boot` -> `Spring Boot`).
- Giới thiệu kỹ năng quá dàn trải, không có điểm nhấn. Nhà tuyển dụng không cần người biết tuốt mọi thứ mà cần người làm thật tốt trong một mảng!
- Đối với một số kỹ năng quan trọng của Java Backend như Spring Boot mà chỉ ở mức "tìm hiểu qua" thì khó đáp ứng yêu cầu doanh nghiệp.

### Kinh nghiệm thực tập / Kinh nghiệm làm việc (Quan trọng)

Kinh nghiệm làm việc dành cho người đã có kinh nghiệm (Social), kinh nghiệm thực tập dành cho sinh viên (Campus).

Kinh nghiệm làm việc nên sắp xếp theo thứ tự thời gian đảo ngược (mới nhất lên đầu). Cả hai đều cần làm nổi bật một cách ngắn gọn mình đã làm những gì trong thời gian làm việc.

Ví dụ:

> **Công ty ABC (202X.XX ~ 202X.XX)**
>
> - **Vị trí**: Kỹ sư phát triển Java Backend
> - **Nội dung công việc**: Phụ trách chính về XXX

### Kinh nghiệm dự án (Quan trọng)

Trên CV có 1-2 kinh nghiệm dự án là bình thường, nhưng thực sự có thể trình bày tốt kinh nghiệm dự án cho người phỏng vấn thì rất ít.

Nhiều ứng viên khi giới thiệu dự án thường gặp các vấn đề: quá dài dòng, quá sơ sài, không làm nổi bật được điểm sáng.

Template giới thiệu kinh nghiệm dự án như sau:

> Tên dự án (Cỡ chữ to hơn một chút)
>
> 2017-05 ~ 2018-06 | Kỹ sư phát triển Java Backend
>
> - **Mô tả dự án**: Mô tả ngắn gọn dự án làm về cái gì.
> - **Tech Stack**: Đã sử dụng những công nghệ gì (Ví dụ: Spring Boot + MySQL + Redis + MyBatis-Plus + Spring Security + OAuth2)
> - **Trách nhiệm cá nhân**: Mô tả ngắn gọn bản thân đã làm những gì, giải quyết vấn đề gì, mang lại cải thiện thực chất nào. Làm nổi bật năng lực bản thân, tránh trần thuật quá mờ nhạt.
> - **Thu hoạch cá nhân (Tùy chọn)**: Từ dự án này bạn học được những gì, áp dụng những công nghệ nào, làm chủ việc sử dụng công nghệ mới ra sao. Thông thường không cần viết vì phần trách nhiệm đã thể hiện điều này.
> - **Thành quả dự án (Tùy chọn)**: Mô tả ngắn gọn dự án đạt được thành tích gì.

**1. Kinh nghiệm dự án nên làm nổi bật bản thân đã làm gì, khái quát ngắn gọn tình hình cơ bản của dự án.**

Phần giới thiệu dự án cố gắng gói gọn trong 2 dòng, không cần giới thiệu quá nhiều nhưng cũng đừng viết qua loa vài chữ.

Ngoài ra, thu hoạch cá nhân và thành quả dự án là tùy chọn, nếu viết thì đừng chiếm quá nhiều dung lượng, hãy nhớ trọng tâm là nội dung công việc / trách nhiệm cá nhân.

**2. Kiến trúc kỹ thuật chỉ cần ghi trực tiếp tên công nghệ, không cần giải thích công nghệ đó để làm gì, điều đó vô nghĩa.**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/46c92fbc5160e65dd85c451143177144.png)

**3. Giảm bớt các mô tả trách nhiệm thuần túy về nghiệp vụ (CRUD đơn thuần). Cố gắng khai thác thêm điểm sáng (khoảng 6~8 gạch đầu dòng trách nhiệm là vừa), tốt nhất có thể thể hiện tố chất tổng hợp của bản thân: ví dụ bạn điều phối các thành viên cùng phát triển ra sao, khi gặp bài toán hóc búa bạn giải quyết thế nào, hoặc bạn tối ưu hiệu năng module nào trong dự án.**

Ngay cả khi tính năng hoặc vấn đề đó không phải do bạn trực tiếp làm, chỉ cần bạn thấu hiểu cặn kẽ thì hoàn toàn có thể lấy làm kinh nghiệm của mình, trau chuốt lại câu chữ là được!

Các điểm sáng về tối ưu hiệu năng trước khi phỏng vấn cũng tương đối dễ chuẩn bị, nhưng cũng đừng biến toàn bộ thành tối ưu hiệu năng, đó cũng là một thái cực cực đoan.

Ngoài ra, thành quả tối ưu kỹ thuật nên cố gắng lượng hóa bằng con số:

- Sử dụng công nghệ xxx giải quyết vấn đề xxx, QPS hệ thống từ xxx tăng lên xxx.
- Sử dụng công nghệ xxx tối ưu API xxx, QPS hệ thống từ xxx tăng lên xxx.
- Sử dụng công nghệ xxx giải quyết vấn đề xxx, tốc độ truy vấn tối ưu xxx, QPS hệ thống đạt 10w+.
- Sử dụng công nghệ xxx tối ưu module xxx, thời gian phản hồi từ 2s giảm xuống 0.2s.
- ……

Ví dụ mô tả trách nhiệm cá nhân (Đây chỉ là ví dụ tham khảo, đừng copy nguyên xi, hãy tự viết theo trải nghiệm dự án của mình):

- Dựa trên Spring Cloud Gateway + Spring Security OAuth2 + JWT triển khai xác thực và phân quyền tập trung cho Microservices, sử dụng mô hình RBAC để kiểm soát quyền động.
- Tham gia phát triển module đơn hàng, phụ trách các tính năng tạo, hủy, tra cứu đơn hàng, dựa trên Spring StateMachine để quản lý luồng chuyển trạng thái đơn hàng.
- Tích hợp Elasticsearch cho kịch bản tìm kiếm sản phẩm và đơn hàng, hiện thực tính năng gợi ý tìm kiếm và gợi ý sản phẩm liên quan.
- Tích hợp Canal + RabbitMQ để đồng bộ dữ liệu gia tăng của MySQL (dữ liệu sản phẩm, đơn hàng) sang Elasticsearch.
- Sử dụng plugin Delayed Message của RabbitMQ để triển khai các kịch bản tác vụ hẹn giờ như tự động hủy đơn hàng timeout, nhắc nhở coupon hết hạn, xử lý hoàn tiền.
- Tích hợp RabbitMQ vào hệ thống gửi thông báo để xử lý bất đồng bộ, san phẳng đỉnh tải (peak shaving) và giảm phụ thuộc dịch vụ (decoupling), tốc độ gửi tối đa đạt 10w/s, lượng tin nhắn tối đa 20 triệu/ngày.
- Sử dụng công cụ Memory Analyzer Tool (MAT) phân tích Heap Dump để khắc phục sự cố cảnh báo timeout dịch vụ hàng loạt sau khi release phiên bản mới của dịch vụ quảng cáo.
- Điều tra và giải quyết vấn đề Deadlock trong module trừ phí do task cha trừ phí và task con chống gian lận dùng chung một Thread Pool, loại bỏ triệt để nguy cơ bằng chiến lược Thread Pool Isolation.
- Dựa trên EasyExcel triển khai import/export dữ liệu chạy quảng cáo, tối ưu insert dữ liệu bằng MyBatis Batch, xử lý bất đồng bộ dựa trên bảng tác vụ (task table).
- Phụ trách phát triển module thống kê người dùng, sử dụng CompletableFuture để tải song song dữ liệu đa chiều, thời gian phản hồi trung bình giảm từ 3.5s xuống 1s.
- Tích hợp Sentinel để Rate Limiting và Circuit Breaking cho các kịch bản cốt lõi (đăng nhập, đăng ký, tra cứu địa chỉ nhận hàng), bảo vệ hệ thống và nâng cao trải nghiệm người dùng.
- Dữ liệu hot (trang chủ, bài viết nổi bật) sử dụng 2 tầng cache Redis + Caffeine, giải quyết vấn đề Cache Breakdown và Cache Penetration, tốc độ truy vấn đạt mức mili-giây, QPS 30w+.
- Sử dụng CompletableFuture tối ưu module giỏ hàng, phối hợp bất đồng bộ các RPC call lấy thông tin user, chi tiết sản phẩm, coupon, thời gian phản hồi giảm từ 2s xuống 0.2s.
- Dựng dịch vụ EasyMock để giả lập API của bên thứ ba, hỗ trợ công tác tích hợp API trong điều kiện mạng bị cô lập.
- Dựng hệ thống Distributed Tracing dựa trên SkyWalking + Elasticsearch để giám sát toàn bộ chuỗi request.

**4. Nếu bạn thấy công nghệ trong dự án của mình tương đối cũ, bạn có thể tự mình cải tiến bên ngoài. Quan trọng là làm cho dự án có điểm sáng, bằng cách nào không quá quan trọng.**

Kinh nghiệm dự án rất quan trọng trong CV. Phần "Chuẩn bị phỏng vấn" của [《Java 面试指北》](https://javaguide.cn/zhuanlan/java-mian-shi-zhi-bei.html) có nhiều bài viết hướng dẫn tối ưu kinh nghiệm dự án, rất khuyên bạn nên đọc kỹ.

![](https://oss.javaguide.cn/zhishixingqiu/4e11dbc842054e53ad6c5f0445023eb5~tplv-k3u1fbpfcp-zoom-1.png)

**5. Tránh việc tất cả mô tả trách nhiệm chỉ xoay quanh một điểm kỹ thuật duy nhất.**

![](https://oss.javaguide.cn/zhishixingqiu/image-20230424222513028.png)

**6. Tránh các mô tả mơ hồ, giới thiệu phải cụ thể (Công nghệ + Bối cảnh + Hiệu quả), đồng thời chú ý súc tích ngôn từ (tránh nhồi nhét từ khóa kỹ thuật, lược bỏ mô tả không cần thiết).**

![](https://oss.javaguide.cn/github/javaguide/interview-preparation/project-experience-avoiding-ambiguity-descriptio.png)

### Giải thưởng & Danh hiệu (Tùy chọn)

Nếu bạn có các giải thưởng giá trị cao trong các cuộc thi (như ACM, Cuộc thi Tianchi của Alibaba), phần này nhất định phải viết! Bạn còn có thể đẩy phần này lên vị trí nổi bật hơn ở phía trên.

### Hoạt động ngoại khóa / Trải nghiệm tại trường (Tùy chọn)

Nếu có hoạt động nào thực sự nổi bật thì viết ngắn gọn, không có thì bỏ qua!

### Tự đánh giá bản thân (Personal Evaluation)

**Đánh giá bản thân là cách bạn tự định vị mình, nhất định phải dùng ngôn ngữ súc tích làm nổi bật đặc điểm và ưu thế của bản thân, tránh nói suông!** Những từ sáo rỗng như chăm chỉ, chịu khó thì không nên đưa vào, người phỏng vấn nhìn rất ngán ngẩm.

Chúng ta có thể viết đánh giá bản thân từ các góc độ sau:

- Khả năng viết tài liệu, khả năng tự học, giao tiếp và làm việc nhóm
- Thái độ đối với công việc và tinh thần trách nhiệm cá nhân
- Khả năng chịu áp lực công việc và thái độ khi đối mặt khó khăn
- Tinh thần cầu tiến về kỹ thuật, sự chỉn chu đối với chất lượng code
- Kinh nghiệm phát triển hoặc vận hành bảo trì hệ thống Distributed, High Concurrency

3 ví dụ thực tế:

- Khả năng tự học tốt, năm 3 đại học khi tham gia Cuộc thi Thiết kế Phần mềm Quốc gia đã nhanh chóng học Python để viết một hệ thống Crawler có khả năng cấu hình linh hoạt.
- Có tinh thần làm việc nhóm, khi tham gia thi đấu đã điều phối 5 thành viên phát triển trong nhóm, hỗ trợ các bạn gặp khó khăn khi code, cuối cùng hoàn thành các tính năng cốt lõi thuận lợi trong 1 tháng.
- Kinh nghiệm dự án phong phú, từng chủ trì phát triển nhiều dự án cấp doanh nghiệp trong thời gian học đại học.

## Nguyên tắc STAR và Nguyên tắc FAB

### Nguyên tắc STAR (Situation Task Action Result)

Chắc hẳn mọi người đều đã nghe qua nguyên tắc STAR. Đối với phỏng vấn, bạn có thể áp dụng nguyên tắc này vào CV và trong quá trình giao tiếp với người phỏng vấn:

- **Situation (Bối cảnh):** Sự việc xảy ra trong bối cảnh/tình huống nào?
- **Task (Nhiệm vụ):** Nhiệm vụ của bạn là gì?
- **Action (Hành động):** Bạn đã làm những gì?
- **Result (Kết quả):** Kết quả cuối cùng ra sao?

### Nguyên tắc FAB (Feature Advantage Benefit)

Ngoài STAR, bạn cũng nên biết thêm nguyên tắc FAB thường dùng trong ngành Sales:

- **Feature (Đặc điểm):** Đặc điểm/thế mạnh của bạn là gì?
- **Advantage (Ưu thế):** Vượt trội hơn người khác ở những điểm nào?
- **Benefit (Lợi ích):** Nếu tuyển dụng bạn, nhà tuyển dụng sẽ nhận được lợi ích gì?

Nói một cách đơn giản, **nguyên tắc FAB giúp người phỏng vấn biết được thế mạnh của bạn và giá trị bạn có thể mang lại cho công ty.**

## Lời khuyên

### Tránh số trang quá dài

Diễn đạt súc tích, làm nổi bật điểm sáng. CV sinh viên khuyên không quá 2 trang, CV người có kinh nghiệm không quá 3 trang. Nếu nội dung nhiều, không nhất thiết phải ép vào 1 trang, chỉ cần giữ bố cục sạch sẽ, ngay ngắn là được.

Đã đọc hàng nghìn bản CV, có một số bạn làm CV dài gần 10 trang, thực sự làm người đọc "hoa mày chóng mặt".

![CV quá nhiều trang](https://oss.javaguide.cn/zhishixingqiu/image-20230508223646164.png)

### Tránh diễn đạt mơ hồ

Cố gắng tránh các diễn đạt mang tính chủ quan, bớt dùng các tính từ mơ hồ. Diễn đạt cần ngắn gọn rõ ràng, cấu trúc CV phải mạch lạc.

Ví dụ:

- Diễn đạt chưa tốt: Tôi đóng vai trò rất quan trọng trong nhóm.
- Diễn đạt tốt: Với vai trò Tech Lead backend, tôi dẫn dắt nhóm hoàn thành thiết kế và phát triển dự án backend.

### Chú ý hình thức CV

Hình thức CV cũng cực kỳ quan trọng! Không cần theo đuổi sự hoa mỹ màu mè, nhưng phải đảm bảo cấu trúc rõ ràng và dễ đọc.

### Các lưu ý khác

- Nhất định phải nộp file định dạng PDF, không dùng Word hay các định dạng khác. Đây là điều cơ bản nhất!
- Thứ gì không biết thì đừng ghi vào CV. Chú ý tính chân thực của CV, trau chuốt hợp lý thì hoàn toàn bình thường.
- Kinh nghiệm làm việc nên xếp theo thứ tự thời gian đảo ngược, kinh nghiệm thực tập nên để cái có giá trị nhất lên đầu.
- Thể hiện hoàn hảo kinh nghiệm dự án là rất quan trọng, trọng tâm là làm nổi bật bản thân đã làm gì (khai thác điểm sáng), chứ không phải giới thiệu dự án làm về cái gì.
- Dự án không nằm ở số lượng (tinh tuyển 2~3 dự án là đủ) mà nằm ở việc có điểm nhấn.
- Quá trình chuẩn bị phỏng vấn nên lấy những thứ bạn viết trên CV làm trọng tâm, đặc biệt là kinh nghiệm dự án và danh mục kỹ năng.
- Phỏng vấn và làm việc là hai việc khác nhau. Người thông minh sẽ dẫn dắt người phỏng vấn vào lĩnh vực sở trường của mình, người khác thì bị người phỏng vấn dắt mũi. Tuy nhiên, muốn nhận được offer ưng ý thì thực lực bản thân phải đủ mạnh.

## Sửa CV

Tính đến nay, tôi đã giúp hơn **6000+** bạn đọc sửa CV. Do thời gian có hạn, dịch vụ sửa CV chỉ dành cho thành viên Tinh Cầu. Nếu cần hỗ trợ xem CV, bạn có thể tham gia [**JavaGuide 官方知识星球**](https://javaguide.cn/about-the-author/zhishixingqiu-two-years.html#%E7%AE%80%E5%8E%86%E4%BF%AE%E6%94%B9).

![img](https://oss.javaguide.cn/xingqiu/%E7%AE%80%E5%8E%86%E4%BF%AE%E6%94%B92.jpg)

Mặc dù học phí chỉ bằng 1% so với các trung tâm đào tạo, nhưng nội dung chất lượng cao hơn và dịch vụ toàn diện hơn, rất phù hợp với các bạn đang chuẩn bị phỏng vấn Java và học Java.

Dưới đây là một số dịch vụ trong Tinh Cầu:

[![Dịch vụ Tinh Cầu](https://oss.javaguide.cn/xingqiu/xingqiufuwu.png)](../about-the-author/zhishixingqiu-two-years.md)

Coupon ưu đãi giới hạn:

![Coupon giảm giá 30 tệ Tinh Cầu](https://oss.javaguide.cn/xingqiu/xingqiuyouhuijuan-30.jpg)
