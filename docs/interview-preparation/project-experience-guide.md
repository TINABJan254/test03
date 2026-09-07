---
title: Hướng dẫn kinh nghiệm dự án
description: Hướng dẫn kinh nghiệm dự án: Dành cho ứng viên chưa có dự án hoặc dự án mờ nhạt, cung cấp phương pháp và lời khuyên lựa chọn để tích lũy kinh nghiệm dự án thực chiến, đồng thời làm rõ cách tạo điểm nhấn dự án, cách review và diễn đạt để nâng cao khả năng cạnh tranh của CV và phỏng vấn.
category: Chuẩn bị phỏng vấn
icon: "mdi:projector-screen-outline"
head:
  - - meta
    - name: keywords
      content: Kinh nghiệm dự án, Dự án tuyển dụng sinh viên, Dự án thực chiến, Điểm sáng dự án, Mô tả dự án trong CV, Dự án backend, Chuẩn bị dự án phỏng vấn, Review dự án
---

::: tip Lời nhắc thân thiện
Bài viết này được trích từ **[《Java 面试指北》](../zhuanlan/java-mian-shi-zhi-bei.md)**. Đây là chuyên mục hướng dẫn bạn cách chuẩn bị phỏng vấn hiệu quả hơn, nội dung bổ trợ cho JavaGuide, bao gồm các câu hỏi phỏng vấn cốt lõi thường gặp (Thiết kế hệ thống, Framework phổ biến, Hệ thống phân tán, High concurrency ……), kinh nghiệm phỏng vấn chất lượng cao, v.v.
:::

## Chưa có kinh nghiệm dự án thì phải làm sao?

Chưa có kinh nghiệm dự án là vấn đề mà phần lớn sinh viên mới tốt nghiệp gặp phải. Thậm chí, có rất nhiều lập trình viên đã đi làm nhưng không hài lòng với các dự án mình làm ở công ty và cũng muốn tìm một dự án có hàm lượng kỹ thuật cao hơn để làm.

Dưới đây là một số cách đáng tin cậy để tích lũy kinh nghiệm dự án, hy vọng sẽ mang lại gợi ý cho bạn:

### Xem video/chuyên mục dự án thực chiến

Tìm một video hoặc chuyên mục dự án thực chiến trên mạng phù hợp với năng lực và nhu cầu tìm việc của bạn, làm theo hướng dẫn của giảng viên.

Bạn có thể tìm kiếm các video/chuyên mục dự án thực chiến phù hợp thông qua các nền tảng như IMOOC (慕课网), Bilibili, Lagou, Geek Time, các trung tâm đào tạo (như Heima, Shangguigu), v.v.

![Khóa học thực chiến IMOOC](https://oss.javaguide.cn/javamianshizhibei/mukewangzhiazhanke.png)

Hãy cố gắng chọn một dự án phù hợp với bản thân, không nhất thiết phải làm dự án phân tán (distributed) / microservices. Đối với phần lớn các bạn, làm thật tốt một dự án đơn thể (monolithic) đã là rất tuyệt vời rồi.

Tôi đã phỏng vấn rất nhiều ứng viên, CV nhìn có vẻ có kinh nghiệm dự án microservices, nhưng chỉ cần hỏi hai câu là biết ngay không phải tự mình làm hoặc khi làm hoàn toàn không suy nghĩ nghiêm túc. Tình huống này sẽ để lại ấn tượng rất xấu với người phỏng vấn.

Tôi cũng đã từng chia sẻ trong phần "Chuẩn bị phỏng vấn" của **[《Java 面试指北》](../zhuanlan/java-mian-shi-zhi-bei.md)**:

> Cá nhân tôi nghĩ cũng không nhất thiết phải cố làm dự án microservices hay distributed, chưa chắc đã có lợi cho phỏng vấn của bạn. Dự án microservices hoặc distributed liên quan đến quá nhiều điểm kiến thức, người bình thường rất khó thấu hiểu hết. Hơn nữa, loại dự án này đối với sinh viên mới tốt nghiệp thực ra hơi vượt chuẩn một chút. Dù bạn có làm ra, nhiều người phỏng vấn cũng sẽ cho rằng không phải do bạn độc lập hoàn thành.
>
> Thực ra, bạn làm một dự án monolithic đến mức tối ưu nhất cũng rất tốt, đối với việc nâng cao năng lực cá nhân không hề thua kém việc làm dự án microservices hay distributed. Làm sao để đạt mức tối ưu nhất? Chất lượng code ở đây không cần bàn tới, quan trọng hơn là bạn phải cố gắng tạo cho dự án của mình một vài điểm nhấn (chẳng hạn như bạn nâng cao hiệu năng dự án như thế nào, giải quyết một điểm nghẽn (pain point) trong dự án ra sao), thành quả đạt được từ kinh nghiệm dự án nên cố gắng lượng hóa, ví dụ: tôi đã sử dụng công nghệ xxx giải quyết vấn đề xxx, QPS của hệ thống từ xxx tăng lên xxx.

Trong quá trình làm theo giảng viên, bạn nhất định phải có tư duy của riêng mình, đừng chỉ dừng lại ở mức cưỡi ngựa xem hoa. Đối với nhiều điểm kiến thức, giải thích của người khác có thể chỉ vừa đủ đáp ứng dự án, nếu bản thân muốn biết nhiều hơn, đối với những điểm kiến thức quan trọng bạn phải tự học cách đào sâu tìm hiểu.

### Dự án mã nguồn mở thực chiến

Trên GitHub hoặc Gitee có rất nhiều dự án thuộc thể loại thực chiến, bạn có thể chọn một dự án để nghiên cứu. Để hiểu rõ hơn về dự án này, trên cơ sở hiểu code ban đầu, bạn có thể cải tiến hoặc thêm tính năng cho dự án.

Bạn có thể tham khảo các dự án open-source thực chiến được đề xuất tại [Dự án thực chiến mã nguồn mở Java chất lượng cao](https://javaguide.cn/open-source-project/practical-project.html "Java 优质开源实战项目"). Chất lượng đều rất cao, thể loại dự án cũng tương đối toàn diện, bao gồm hệ thống Blog/Diễn đàn, hệ thống Thi cử/Luyện đề, hệ thống E-commerce, hệ thống Phân quyền, Scaffold phát triển nhanh và các thư viện tự dựng (wheel).

![Dự án thực chiến open source Java chất lượng](https://oss.javaguide.cn/javamianshizhibei/javaguide-practical-project.png)

Nhất định phải nhớ: **Không chỉ làm, mà còn phải cải tiến, hoàn thiện. Dù là video dự án thực chiến, chuyên mục hay dự án open-source thực chiến, chắc chắn đều có rất nhiều điểm có thể hoàn thiện và nâng cấp.**

### Tự phát triển từ đầu (From Scratch)

Tự tay làm một thứ mà bản thân muốn hoàn thành, gặp chỗ nào chưa biết thì vừa học vừa làm, học đến đâu áp dụng ngay đến đó.

Cách này đòi hỏi trình độ tương đối cao, tôi khuyên bạn sau khi đã có kinh nghiệm làm qua một dự án thì mới nên áp dụng phương pháp này. Nếu bạn chưa từng làm dự án nào, tốt nhất vẫn nên chọn hai phương pháp trên.

### Tham gia các cuộc thi lớn do các công ty công nghệ tổ chức

Nếu tham gia các cuộc thi như thế này mà đạt giải thì hàm lượng giá trị của dự án là rất cao. Dù không đạt giải cũng không sao, bạn vẫn có thể đưa vào CV.

![Cuộc thi Alibaba Cloud Tianchi](https://oss.javaguide.cn/xingqiu/up-673f598477242691900a1e72c5d8b26df2c.png)

### Tham gia dự án thực tế

Thông thường, bạn có các kênh sau để tiếp cận với việc phát triển dự án thực tế của doanh nghiệp:

1. Dự án do giảng viên nhận về;
2. Dự án freelance tự nhận;
3. Dự án tiếp xúc được khi đi thực tập / làm việc;

Dự án của thầy cô và freelance thường là các dự án thuần nghiệp vụ, hiếm khi liên quan đến tối ưu hiệu năng. Trong trường hợp này, bạn có thể cân nhắc cải tiến dự án, đừng ngại tốn thời gian, dành thời gian làm tốt một việc là được, chẳng hạn như bạn cải tiến mô hình dữ liệu của dự án, đưa vào bộ nhớ cache để tăng tốc độ truy cập, v.v.

Dự án tiếp xúc khi thực tập / làm việc cũng tương tự, nếu gặp các dự án thiên về nghiệp vụ, bạn cũng nên tự mình tìm cách cải tiến và tối ưu hóa dự án ở bên ngoài.

Cố gắng tối ưu hóa dự án một cách thực sự, bản thân điều này cũng là sự nâng cao năng lực cá nhân. Nếu bạn thực sự không có thời gian thực hành thì cũng không sao, hãy nắm thật chắc phương pháp tối ưu dự án này, chuẩn bị trước một số câu hỏi có thể gặp phải trong phỏng vấn.

## Có dự án nào tốt được khuyến nghị không?

Trong phần "Chuẩn bị phỏng vấn" của **[《Java 面试指北》](../zhuanlan/java-mian-shi-zhi-bei.md)** có một bài viết chuyên tổng hợp các dự án thực chiến chất lượng cao, bao gồm dự án nghiệp vụ, dự án tự dựng công cụ (wheel), Lab khóa học mở quốc tế và các video hướng dẫn thực chiến, rất thích hợp để học tập hoặc dùng làm kinh nghiệm dự án.

![Gợi ý dự án thực chiến Java chất lượng cao](https://oss.javaguide.cn/javamianshizhibei/project-experience-guide.png)

Bài viết này giới thiệu tổng cộng 15+ dự án thực chiến, có cả loại nghiệp vụ lẫn loại công cụ, cả dự án mã nguồn mở lẫn video hướng dẫn. Đối với các bạn tham gia tuyển dụng sinh viên, tôi khuyên nên làm một dự án nghiệp vụ kết hợp với một dự án công cụ (wheel).

## Tôi làm dự án theo video hướng dẫn thì có bị người phỏng vấn chê không?

Rất nhiều sinh viên mới ra trường làm dự án theo video hướng dẫn, điều này hầu hết người phỏng vấn đều hiểu rõ.

Không loại trừ việc quả thực có một số người phỏng vấn không thích điều này, việc này cũng tùy người. Nhưng tôi tin đa số người phỏng vấn đều có thể thông cảm, vì khi còn ở trường đại học thực tế bạn không có nhiều cơ hội để tiếp cận các dự án thực tế.

Kinh nghiệm dự án của phần lớn sinh viên đều là tự tìm trên mạng hoặc mua khóa học trả phí rồi làm theo giống bạn, rất ít người có dự án thực tế doanh nghiệp. Việc bạn chủ động làm một dự án thực chiến là một khởi đầu tốt, thực sự giúp bạn học hỏi được kiến thức. Tuy nhiên, điều quan trọng là bạn đã nắm vững được bao nhiêu phần. Điều tối kỵ nhất khi xem video là tiếp thu thụ động, hãy tự mình cải tiến nhiều hơn, suy nghĩ nhiều hơn! Ngay cả khi bạn làm dự án theo video, nó vẫn hoàn toàn có thể được tối ưu hóa!

**Nếu bạn thực sự muốn học được kiến thức, khuyên bạn không chỉ đơn thuần cho dự án chạy được, mà còn phải tự mình thử nghiệm tối ưu hóa!**

Điểm qua một vài hướng tối ưu tương đối dễ thực hiện:

1. **Xử lý ngoại lệ toàn cục (Global Exception Handling)**: Rất nhiều dự án làm phần này chưa tốt, có thể tham khảo bài viết: [《Sử dụng Enum đóng gói xử lý ngoại lệ toàn cục thanh lịch trong Spring Boot!》](https://mp.weixin.qq.com/s/Y4Q4yWRqKG_lw0GLUsY2qw) để tối ưu.
2. **Tối ưu lựa chọn công nghệ (Tech Selection)**: Ví dụ nơi dùng Guava làm local cache có thể đổi thành **Caffeine**. Hiệu năng các mặt của Caffeine tốt hơn rất nhiều! Hoặc kiểm tra xem tầng Controller có chứa quá nhiều logic nghiệp vụ hay không.
3. **Về phía Database**: Thiết kế database có thể tối ưu không? Index đã sử dụng đúng chưa? Câu lệnh SQL có thể tối ưu không? Có cần thực hiện Read-Write Separation (tách đọc - ghi) không?
4. **Cache**: Dự án có dữ liệu nào thường xuyên được truy cập không? Có nên đưa cache vào để tăng tốc độ phản hồi không?
5. **Bảo mật (Security)**: Dự án có tồn tại lỗ hổng bảo mật không?
6. ……

Ngoài ra, tôi đã từng chia sẻ trong Tinh Cầu các trường hợp thực hành tối ưu hiệu năng phổ biến liên quan đến Multi-threading, Asynchronous, Index, Cache, rất khuyên bạn nên xem: <https://t.zsxq.com/06EqfeMZZ>.

Cuối cùng, **xin giới thiệu với mọi người một mẹo nhỏ để tối ưu code trong IntelliJ IDEA, cực kỳ hữu ích!**

Phân tích code của bạn: Chuột phải vào project -> Analyze -> Inspect Code

![](https://oss.javaguide.cn/xingqiu/up-651672bce128025a135c1536cd5dc00532e.png)

Sau khi quét xong, IDEA sẽ chỉ ra một số Code Smell tiềm ẩn như vấn đề đặt tên.

![](https://oss.javaguide.cn/xingqiu/up-05c83b319941995b07c8020fddc57f26037.png)

Hơn nữa, bạn còn có thể tùy chỉnh các quy tắc kiểm tra (inspection rules).

![](https://oss.javaguide.cn/xingqiu/up-6b618ad3bad0bc3f76e6066d90c8cd2f255.png)

## Sau khi làm xong dự án thì chuẩn bị cho việc đào sâu phỏng vấn như thế nào?

Làm xong dự án chỉ mới giải quyết được bài toán "có dự án hay không". Phỏng vấn kỹ thuật sẽ tiếp tục đào sâu hỏi về kiến trúc dự án, trách nhiệm cá nhân, lựa chọn công nghệ, luồng xử lý cốt lõi, chỉ số hiệu năng và sự cố online. Khuyên bạn nên chuẩn bị cho mỗi dự án trọng điểm trong CV hai phiên bản giới thiệu: 30 giây và 3 phút, sau đó tự đặt câu hỏi đào sâu theo từng công nghệ mình sử dụng như Database, Cache, Message Queue và Thread Pool.

Phương pháp chuẩn bị cụ thể có thể tham khảo: [《Trình bày dự án Backend trong phỏng vấn như thế nào? Từ giới thiệu dự án đến điểm khó kỹ thuật và review sự cố》](./backend-project-interview-guide.md). Hệ thống quản lý đơn hàng trong bài viết chỉ mang tính minh họa cấu trúc trả lời, trách nhiệm dự án và các chỉ số vẫn cần thay thế bằng dữ liệu thực tế của chính bạn.

## Dự án AI Agent chuẩn bị như thế nào?

Nếu bạn chuẩn bị dự án AI Agent, ngoài việc trình bày rõ mình đã làm những gì, còn phải trả lời được tại sao lại sử dụng Agent, luồng yêu cầu di chuyển như thế nào, thao tác ghi của Tool được bảo đảm an toàn (fallback) ra sao, và một trường hợp thất bại (bad case) được định vị và sửa chữa như thế nào. Bạn có thể xem thêm bài viết [《Trình bày dự án Agent trong phỏng vấn như thế nào? Từ kiến trúc hệ thống, lựa chọn công nghệ đến review Badcase》](../ai/interview-questions/agent-project-interview-guide.md). Các case study trong bài viết chỉ dùng làm tài liệu tham khảo để tổ chức câu trả lời, trách nhiệm và chỉ số dự án vẫn phải dựa trên kinh nghiệm thực tế của chính bạn.
