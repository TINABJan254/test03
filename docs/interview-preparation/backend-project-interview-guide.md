---
title: Trình bày dự án Backend trong phỏng vấn như thế nào? Từ giới thiệu dự án đến điểm khó kỹ thuật và review sự cố
description: Hướng dẫn chuẩn bị phỏng vấn dự án Java Backend, làm rõ phương pháp chuẩn bị giới thiệu dự án, trách nhiệm cá nhân, luồng xử lý cốt lõi, lựa chọn công nghệ, tối ưu hiệu năng, sự cố online, chỉ số định lượng và các câu hỏi đào sâu thường gặp.
category: Chuẩn bị phỏng vấn
tag:
  - Phỏng vấn Java
  - Phỏng vấn Backend
  - Kinh nghiệm dự án
  - Đào sâu dự án
sitemap:
  changefreq: monthly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Phỏng vấn dự án backend, Phỏng vấn dự án Java, Giới thiệu dự án, Đào sâu dự án, Lựa chọn công nghệ, Khó khăn dự án, Sự cố online, Tối ưu hiệu năng, Phỏng vấn Java
---

"Hãy giới thiệu về dự án bạn đã từng làm."

"Đây là một dự án microservices phát triển dựa trên Spring Boot, sử dụng MySQL, Redis, Kafka, Elasticsearch……"

Rất nhiều phần giới thiệu dự án đến đây là không thể nói tiếp được nữa. Người phỏng vấn chỉ cần hỏi thêm một câu: "Tại sao lại dùng Kafka?", câu trả lời rất dễ bị nghẽn. Các câu hỏi phỏng vấn thường gặp về Java, MySQL, Redis có thể học thuộc trước, nhưng việc hỏi sâu về dự án sẽ luôn bám sát vào ràng buộc nghiệp vụ thực tế, triển khai code và kết quả kiểm chứng, học thuộc một đoạn văn mẫu cố định chỉ có thể đối phó được phần mở đầu.

## Người phỏng vấn muốn tìm hiểu điều gì từ dự án?

Người phỏng vấn sẽ xác nhận một vài điều qua dự án: Bạn có hiểu nghiệp vụ mà dự án phục vụ hay không, một request luân chuyển như thế nào; Bạn trực tiếp phụ trách những phần code, dữ liệu và upstream/downstream nào; Khi gặp vấn đề bạn định vị nguyên nhân, so sánh phương án và kiểm chứng kết quả ra sao; Công nghệ và các chỉ số trong CV có chịu được việc hỏi sâu hay không.

Phỏng vấn tuyển dụng sinh viên (Campus recruitment) sẽ không yêu cầu mỗi dự án đều phải đạt tới độ phức tạp của hệ thống production quy mô lớn. Một dự án monolithic do chính bạn tự làm và có thể trình bày thấu đáo, thường vững vàng hơn một dự án microservices dựng lên bằng cách làm theo hướng dẫn. Phỏng vấn tuyển dụng người có kinh nghiệm (Social recruitment) sẽ tiếp tục hỏi sâu về lưu lượng, dung lượng, sự cố, canary release (phát hành xám), rollback cũng như sự phối hợp trong team, khi trả lời cần cung cấp nhiều bằng chứng thực tế ở môi trường production hơn.

## Trước khi phỏng vấn hãy tổng hợp một bản phác thảo dự án

Đối với mỗi dự án trọng điểm trong CV, hãy chuẩn bị riêng một bản phác thảo dự án, không cần viết thành bài văn dài, chỉ cần bản thân hiểu được và trước khi phỏng vấn có thể nhanh chóng nhớ lại.

| Nội dung cần tổng hợp | Câu hỏi cần trả lời |
| --- | --- |
| Bối cảnh nghiệp vụ | Dự án phục vụ ai? Giải quyết vấn đề gì? Luồng nghiệp vụ quan trọng nhất là gì? |
| Phạm vi hệ thống | Gồm những module nào? Phụ thuộc hệ thống bên ngoài nào? Dữ liệu từ đâu đến, đi về đâu? |
| Trách nhiệm cá nhân | Những requirement, API hoặc module nào do bạn phụ trách? Tham gia ở mức độ nào? |
| Luồng xử lý cốt lõi | Một request sẽ đi qua những service, cache, database và message queue nào? |
| Lựa chọn công nghệ (Tech Selection) | Tại sao sử dụng giải pháp hiện tại? Đã so sánh những giải pháp nào? Trả giá bằng điều gì? |
| Khó khăn và Sự cố | Đã gặp vấn đề cụ thể gì? Định vị, khắc phục và kiểm chứng ra sao? |
| Chỉ số dự án | Lưu lượng, độ trễ, tỷ lệ lỗi, lượng dữ liệu, tiêu hao tài nguyên và kết quả tối ưu có ghi chép đáng tin cậy nào? |

Cột cuối cùng của bản phác thảo hãy dành riêng để ghi những phần mình không tham gia. Ví dụ bảng database do bạn thiết kế, nhưng việc deploy và capacity planning do team hạ tầng phụ trách, thì hãy trả lời đúng thực tế. Người phỏng vấn thường chấp nhận phạm vi trách nhiệm có giới hạn, việc bịa đặt trải nghiệm tham gia rủi ro sẽ lớn hơn nhiều.

## Giới thiệu dự án nên trình bày như thế nào?

Cần chuẩn bị cả 2 phiên bản: 30 giây và 3 phút. Khi người phỏng vấn chỉ muốn nắm nhanh thì dùng bản ngắn; khi đối phương yêu cầu bạn giới thiệu chi tiết, hãy bổ sung kiến trúc, trách nhiệm và các công việc trọng tâm.

### Phiên bản 30 giây

Bản 30 giây chỉ giữ lại 4 nội dung: Vấn đề dự án giải quyết, Người dùng hoặc nghiệp vụ chính, Trách nhiệm của bạn, Một công việc dự định trình bày sâu.

Ví dụ:

> Đây là hệ thống đơn hàng dành cho nhân viên thu mua nội bộ doanh nghiệp, chủ yếu bao gồm tra cứu hàng hóa, đặt hàng, phê duyệt và thực hiện đơn hàng. Trong dự án, tôi phụ trách 2 luồng tạo đơn hàng và tự động đóng đơn hàng khi quá hạn, bao gồm thiết kế cấu trúc bảng, phát triển API, xử lý Idempotency và tích hợp giám sát. Phần tôi dành nhiều thời gian nhất trong dự án là tối ưu hóa hiệu năng của luồng tạo đơn hàng, lát nữa tôi có thể giới thiệu chi tiết về cách định vị các request chậm lúc đó.

Đoạn này không liệt kê tất cả các middleware, nhưng đã để lại cho người phỏng vấn một vài điểm có thể hỏi tiếp: Trạng thái đơn hàng, Idempotency, Đóng đơn hàng timeout và Tối ưu hiệu năng.

### Phiên bản 3 phút

Bản 3 phút triển khai theo thứ tự sau:

1. Bối cảnh nghiệp vụ và người dùng của dự án.
2. Hệ thống gồm các module chính nào, luồng request cốt lõi di chuyển ra sao.
3. Module bản thân phụ trách và phạm vi trách nhiệm.
4. Một hoặc hai điểm khó hoặc kết quả có bằng chứng chứng minh.

Vẫn lấy ví dụ hệ thống đơn hàng:

> Hệ thống chủ yếu phục vụ nhân viên thu mua và kế toán, chịu trách nhiệm tra cứu sản phẩm, tạo đơn hàng, phê duyệt, đồng bộ trạng thái thanh toán và tra cứu thực hiện đơn. Backend được chia tách theo các module Đơn hàng, Tồn kho và Phê duyệt. Khi tạo đơn hàng, trước tiên sẽ validate request và giá cả, sau đó tạo đơn và giữ trước (pre-occupy) tồn kho; sau khi thành công sẽ gửi message để downstream hoàn thành các tác vụ async như thông báo phê duyệt.
>
> Tôi phụ trách việc tạo đơn hàng và đóng đơn hàng khi timeout. Tạo đơn hàng cần xử lý submit lặp lại, tồn kho không đủ và gửi message thất bại; đóng đơn timeout cần tránh việc đóng nhầm các đơn hàng đã thanh toán. Tôi chủ yếu hoàn thành API, luồng chuyển trạng thái (State Machine), Idempotency và các tác vụ bù đắp (compensation tasks), đồng thời tích hợp monitoring liên quan. Sau này trong một lần release phiên bản mới, độ trễ đuôi (long-tail latency) của API tra cứu đơn hàng tăng cao, tôi đã tham gia định vị và tối ưu hóa vấn đề. Chúng tôi dựa vào Trace và Slow SQL để tìm ra vấn đề điều kiện truy vấn không khớp với Composite Index, sau khi điều chỉnh index đã tiến hành stress test so sánh dưới cùng một lượng dữ liệu.

Đây là một ví dụ minh họa, đừng bê nguyên trách nhiệm và sự cố trong này đổi tên dự án rồi nhét vào CV. Dự án của bạn không có Message Queue thì nói về gọi đồng bộ (Sync call); không có số liệu stress test thực tế cũng có thể giải thích môi trường test và phương pháp kiểm chứng, đừng bịa tạm một con số QPS.

## Trình bày sơ đồ kiến trúc như thế nào?

Sơ đồ kiến trúc dự án thích hợp trình bày theo một request thực tế: Request của người dùng đi vào từ API Gateway, qua service nào, đọc những cache và database nào, tác vụ nào đi vào Message Queue, sau khi lỗi xử lý ra sao. Nói xong luồng chính (Happy Path), bổ sung thêm một luồng ngoại lệ (Exception Path).

Mỗi component xuất hiện trong kiến trúc, tốt nhất đều trả lời được 3 câu hỏi:

- Nó đảm nhận công việc gì trong luồng này?
- Nếu bỏ nó đi thì điều gì sẽ xảy ra?
- Khi nó không khả dụng, hệ thống xử lý thế nào?

Nếu dự án dùng Redis cache thông tin sản phẩm, còn phải chuẩn bị việc query database khi cache miss, chiến lược cache expiration, Hotspot data, Data consistency và phương án fallback khi Redis gặp sự cố. Nếu chỉ trả lời "Redis hiệu năng cao nên dùng Redis", thường sẽ rất nhanh rơi vào điểm mù kiến thức.

Sơ đồ kiến trúc cũng đừng vẽ quá đồ sộ. Nhồi nhét vào một hình cả API Gateway, Registry Center, Config Center, hàng chục Microservices và tất cả các Middleware, khi giới thiệu sẽ rất khó tìm thấy trọng tâm. Sơ đồ kiến trúc dùng phỏng vấn chỉ cần giữ lại phạm vi dự án và một luồng cốt lõi là đủ, dự án phức tạp có thể chuẩn bị thêm một sơ đồ module hoặc Sequence Diagram.

## Làm sao để làm rõ trách nhiệm của bản thân?

"Phụ trách phát triển module đơn hàng" cung cấp rất ít thông tin. Hãy nói rõ tiếp phạm vi requirement, phạm vi code và phạm vi phối hợp:

- Phạm vi requirement: Tạo đơn, hủy đơn, đóng đơn timeout hay toàn bộ nghiệp vụ đơn hàng.
- Phạm vi code: API, cấu trúc bảng, State Machine, Scheduled Task, Message Consumption lần lượt tham gia ở mức độ nào.
- Phạm vi phối hợp: Có tham gia review giải pháp không, phối hợp với các bên Tồn kho, Thanh toán, Test, DevOps như thế nào.
- Phạm vi kết quả: Release, Canary rollout, Monitoring và bảo trì về sau có do mình theo sát hay không.

Dự án tuyển dụng sinh viên thì hãy nói thẳng đây là dự án cá nhân hoặc đồ án môn học, và những phần nào tham khảo từ hướng dẫn. Những phần bản thân tự thêm tính năng, bổ sung test, sửa cấu trúc bảng dựa trên hướng dẫn thì tập trung nói về những thay đổi đó. Người phỏng vấn quan tâm đến việc bạn có thực sự bắt tay vào làm và suy nghĩ hay không, không cần phải đóng gói dự án cá nhân thành hệ thống production của các tập đoàn lớn.

## Trả lời về Lựa chọn công nghệ (Tech Selection) như thế nào?

Khi trả lời về lựa chọn công nghệ, hãy trình bày đầy đủ: Vấn đề, Ràng buộc, Các phương án ứng viên và Kết quả kiểm chứng.

Giả sử người phỏng vấn hỏi: "Tại sao việc đóng đơn hàng timeout lại sử dụng Delayed Message?"

Khi trả lời cần nêu rõ:

1. Đơn hàng sau khi tạo cần kiểm tra trạng thái thanh toán sau khoảng thời gian chỉ định, lượng tác vụ và độ chính xác về độ trễ có yêu cầu gì.
2. Việc định kỳ quét database, thông báo hết hạn của Redis, Hashed Wheel Timer và Delayed Message lần lượt có những hạn chế gì.
3. Tại sao dự án hiện tại chọn Delayed Message, hạ tầng sẵn có, chi phí vận hành bảo trì và kinh nghiệm của team có ảnh hưởng đến quyết định hay không.
4. Xử lý như thế nào khi Message bị lặp lại, bị trễ, bị mất hoặc Consumer gặp sự cố.

Rời khỏi điều kiện cụ thể của dự án, lựa chọn công nghệ không có câu trả lời chuẩn duy nhất. Dự án nhỏ dùng Scheduled Task quét các đơn hàng chờ thanh toán, kết hợp với index và sharding phù hợp có thể đã đủ dùng; khi team đã có sẵn Message Queue hoàn thiện và lượng đơn hàng tương đối lớn, có thể cân nhắc Delayed Message. Câu trả lời phỏng vấn nên giải thích tại sao lại chọn như vậy trong điều kiện hiện tại, đồng thời thừa nhận những hạn chế của giải pháp.

Việc đưa Redis, Kafka hoặc Elasticsearch vào chỉ chứng minh dự án có sử dụng các component này. Giải thích được các vấn đề dưới đây mới chứng minh bạn đã làm chủ phần công việc này:

- Redis cache những dữ liệu nào, Key thiết kế ra sao, thời gian hết hạn đặt thế nào.
- Topic và Partition của Kafka thiết kế ra sao, gửi thất bại và tiêu thụ lặp lại xử lý thế nào.
- Document trong Elasticsearch được mô hình hóa như thế nào, Index cập nhật ra sao, kết quả truy vấn tại sao đáng tin cậy.
- Sau khi phân cơ sở dữ liệu và bảng (Sharding) thì chọn Sharding Key thế nào, mở rộng dung lượng và truy vấn cross-shard xử lý ra sao.

Khi dự án chưa đạt đến quy mô tương ứng, có thể giải thích một công nghệ nào đó dưới dạng học tập và thử nghiệm, đừng tuyên bố nó đã giải quyết điểm nghẽn production vốn không hề tồn tại.

## Trình bày điểm khó của dự án như thế nào?

"Thời gian tương đối gấp", "Requirement thay đổi liên tục" quả thực làm tăng độ khó công việc, nhưng phỏng vấn kỹ thuật thường muốn nghe một vấn đề kỹ thuật (engineering problem) có thể tiếp tục hỏi sâu.

Tìm tư liệu từ những việc mình thực sự đã từng làm, phổ biến gồm có:

- Vấn đề tính chính xác: Đặt trùng đơn, bán vượt tồn kho (overselling), trạng thái hỗn loạn, độ chính xác tiền tệ, dữ liệu không nhất quán.
- Vấn đề hiệu năng: Slow SQL, Cache hit rate giảm, tranh chấp Lock (Lock contention), ứ đọng Thread Pool, GC Pause.
- Vấn đề tính ổn định: Dependency timeout, tích tụ message, cạn kiệt Connection Pool, lỗi khi release, lưu lượng tăng đột biến.
- Vấn đề kỹ thuật: Refactor code cũ, migration dạng canary, tương thích dữ liệu cũ, thay đổi API liên team.

Một điểm khó ít nhất phải trình bày rõ quy trình sau:

```text
Hiện tượng & Ảnh hưởng → Ràng buộc đã biết → Điều tra hoặc Phân tích → So sánh phương án
→ Quá trình thực hiện → Kết quả kiểm chứng → Vấn đề tồn đọng
```

Ví dụ "API tra cứu rất chậm" vẫn chưa phải là điểm khó hoàn chỉnh. Hãy nói rõ tiếp những request nào chậm, bắt đầu từ khi nào, P95/P99 biến đổi ra sao, database đã quét bao nhiêu dòng, cuối cùng sửa index hay sửa logic truy vấn, kiểm chứng thế nào dưới cùng lượng dữ liệu và lưu lượng. Nếu không lưu lại con số lúc đó, hãy giải thích đã quan sát những chỉ số nào và kết luận đến từ bằng chứng gì.

## Trình bày tối ưu hiệu năng như thế nào?

Việc tối ưu hiệu năng trong dự án rất dễ bị hỏi dồn, vì phía sau câu "Thời gian phản hồi của API giảm từ 2 giây xuống 200 ms" còn rất nhiều câu hỏi:

- 2 giây và 200 ms được đo ở vị trí nào?
- Lượng dữ liệu, số lượng concurrency và cấu hình máy có giống nhau không?
- Đang xem thời gian phản hồi trung bình hay P95/P99?
- Điểm nghẽn rốt cuộc nằm ở Application, Database, Cache hay Downstream service?
- Sau khi tối ưu có làm tăng rủi ro không nhất quán dữ liệu và chi phí bảo trì hay không?

Khi chuẩn bị case tối ưu hiệu năng, hãy giữ lại một bộ tài liệu có thể đối chiếu: Trace trước và sau tối ưu, Execution Plan, GC Log, cấu hình stress test, ảnh chụp màn hình monitoring hoặc báo cáo test. Nếu tài liệu công ty không tiện mang đi, có thể ghi lại kết luận và quá trình điều tra đã được ẩn danh dữ liệu nhạy cảm (desensitized).

Khi trả lời hãy kết nối các thông tin này lại:

> Một API nào đó dưới lưu lượng và lượng dữ liệu bao nhiêu xuất hiện vấn đề gì; Tôi dựa vào những chỉ số nào để thu hẹp vấn đề vào khâu nào; Đã so sánh những giải pháp nào, cuối cùng sửa cái gì; Sử dụng môi trường và chỉ số nào để kiểm chứng; Sau khi release tiếp tục quan sát điều gì; Thay đổi này mang lại những chi phí/đánh đổi nào.

Cache, Asynchronous và Parallel processing thường có thể cải thiện thời gian phản hồi, nhưng cũng sẽ đưa vào các vấn đề về Cache consistency, Message reliability, Thread safety và áp lực lên downstream. Nói ra được những cái giá phải trả này sẽ đáng tin cậy hơn nhiều so với việc chỉ nhấn mạnh vào con số hiệu năng.

## Trình bày sự cố Online như thế nào?

Một sự cố trả lời theo trình tự thời gian:

1. **Phát hiện vấn đề**: Cảnh báo (Alert), phản hồi của user hoặc quan sát khi release phát hiện ra điều gì.
2. **Xác nhận ảnh hưởng**: Những API, user và instance nào bị ảnh hưởng, tỷ lệ lỗi và độ trễ biến đổi ra sao.
3. **Cầm máu khẩn cấp (Emergency Mitigation)**: Rollback, cô lập instance, Rate limit, Circuit break/Degrade hoặc tắt tính năng.
4. **Lưu giữ bằng chứng**: Log, Trace, Thread dump, Heap dump, GC log và lịch sử thay đổi.
5. **Định vị nguyên nhân gốc rễ (Root Cause)**: Đã đưa ra những giả thuyết nào, loại trừ các hướng sai ra sao, cuối cùng dùng bằng chứng gì để xác nhận.
6. **Sửa đổi & Kiểm chứng**: Code hoặc cấu hình đã sửa những gì, test, release canary và quan sát ra sao.
7. **Tránh tái phát**: Bổ sung những monitoring, unit test, giới hạn dung lượng hoặc checklist khi release nào.

Khi khắc phục sự cố có áp dụng biện pháp tạm thời thì cũng phải nêu rõ tác dụng phụ của nó. Ví dụ restart instance có thể tạm thời hồi phục service, nhưng sẽ làm mất hiện trường lỗi; scale out mù quáng có thể tiếp tục truyền áp lực xuống database. Phương pháp điều tra hoàn chỉnh có thể tham khảo: [Xử lý sự cố Online trong Java Backend](../java/jvm/jvm-in-action.md).

## Chuẩn bị chỉ số dự án như thế nào?

Chỉ số phải chứng minh được quy mô dự án, ảnh hưởng của vấn đề hoặc kết quả thay đổi, không cần nhồi nhét số liệu chỉ để làm cho dự án có vẻ to tát.

| Chỉ số | Thích hợp để nói lên điều gì | Sai lầm thường gặp |
| --- | --- | --- |
| QPS/TPS | Lưu lượng và thông lượng hệ thống | Chỉ báo con số đỉnh (peak), không nói vị trí thống kê và khoảng thời gian |
| P95/P99 | Trải nghiệm của các request đuôi dài (long-tail) | Chỉ nhìn vào thời gian phản hồi trung bình (average latency) |
| Tỷ lệ lỗi (Error Rate) | Tình trạng request thất bại | Gộp chung lỗi nghiệp vụ từ chối (business rejection) và lỗi hệ thống |
| Lượng dữ liệu | Quy mô truy vấn, lưu trữ và migration | Chỉ nói tổng lượng, không nói tốc độ tăng trưởng và phân bổ nóng/lạnh |
| CPU, Memory, GC | Tài nguyên ứng dụng và trạng thái JVM | Chỉ báo con số sau tối ưu, không có điều kiện đối chứng |
| Tích tụ hàng đợi, Chờ Connection Pool | Xếp hàng bên trong hệ thống | Chỉ mở rộng dung lượng, không xác nhận khả năng xử lý của downstream |

Dự án thực tế không có sẵn chỉ số thì có thể đo kiểm bổ sung trong môi trường test, nhưng phải nêu rõ dữ liệu đến từ môi trường test. Cấu hình máy, lượng dữ liệu, mô hình concurrency và thời gian test cũng phải ghi lại cùng, số liệu test không thể mạo danh dữ liệu production.

## Chuẩn bị các câu hỏi đào sâu thường gặp như thế nào?

Đặt CV trước mặt, tự hỏi dồn theo từng công nghệ trong tech stack. Bộ câu hỏi dưới đây phù hợp với đa số dự án Java Backend:

### Nghiệp vụ và Kiến trúc

- Luồng nghiệp vụ quan trọng nhất của dự án là gì?
- Khâu nào trong hệ thống dễ xảy ra vấn đề nhất?
- Nếu lưu lượng tăng gấp nhiều lần hiện tại, điểm nghẽn có khả năng xuất hiện đầu tiên ở đâu?
- Monolithic và Microservices được lựa chọn như thế nào? Sau khi chia tách service đã làm tăng những chi phí nào?

### Database và Cache

- Các bảng cốt lõi thiết kế ra sao? Tại sao chọn Primary Key và Index này?
- Slow SQL được phát hiện như thế nào? Trong Execution Plan đã xem những trường nào?
- Những dữ liệu nào được đưa vào cache? Sau khi cache mất hiệu lực xử lý thế nào?
- Khi Cache và Database không nhất quán, nghiệp vụ có thể chấp nhận trong bao lâu?

### Message và Tính nhất quán

- Tại sao phải gửi message này, gọi đồng bộ (Sync call) có khả thi không?
- Producer gửi thất bại, Consumer xử lý lặp lại và tích tụ message lần lượt xử lý ra sao?
- Local transaction commit thành công nhưng gửi message thất bại thì xử lý thế nào?
- Tính Idempotency của API dựa vào định danh duy nhất (Unique Identifier) nào của nghiệp vụ?

### Concurrency và Tính ổn định

- Các tham số của Thread Pool được thiết lập dựa trên cơ sở nào? Khi hàng đợi đầy điều gì sẽ xảy ra?
- Khi downstream API bị chậm, Timeout, Retry, Rate Limiting và Circuit Breaker phối hợp với nhau ra sao?
- Khi Redis, Database hoặc Message Queue không khả dụng, dự án còn có thể cung cấp những năng lực nào?
- Khi release xảy ra sự cố, làm thế nào để rollback và xác nhận dữ liệu không bị hư hại?

Không cần chuẩn bị một đoạn câu trả lời mẫu cho từng câu hỏi. Hãy liên hệ câu hỏi với code, cấu trúc bảng, cấu hình và monitoring của chính mình, khi trả lời sẽ tự nhiên hơn rất nhiều.

## Trọng tâm chuẩn bị cho Campus và Social recruitment khác nhau thế nào?

Dự án tuyển dụng sinh viên (Campus) thường tiếp tục hỏi sâu về kiến thức nền tảng và chi tiết triển khai. Ví dụ sử dụng HashMap, Thread Pool, Redis, người phỏng vấn có thể từ dự án chuyển sang cấu trúc dữ liệu, concurrency và các vấn đề cache. Khi chuẩn bị phải đảm bảo các công nghệ xuất hiện trong CV đều nắm vững nguyên lý cơ bản và có thể tìm thấy code tương ứng.

Tuyển dụng người có kinh nghiệm (Social) sẽ quan tâm đến quy mô dự án và sự đánh đổi kỹ thuật (engineering trade-offs): Requirement do ai đề xuất, giải pháp review ra sao, release canary thế nào, sự cố xử lý ra sao, chỉ số có được cải thiện không, phối hợp với các team khác như thế nào. Chỉ nói về việc viết code thường là không đủ, còn phải bổ sung phần giải pháp và giai đoạn sau khi go-live.

Số năm kinh nghiệm càng nhiều, người phỏng vấn càng dễ hỏi sâu: "Tại sao lúc đó không chọn phương án khác?" và "Nếu làm lại từ đầu bạn sẽ thay đổi như thế nào?". Loại câu hỏi này có thể trả lời thành thật về bối cảnh lịch sử, thời gian, năng lực của team, hạ tầng và quy mô nghiệp vụ lúc đó đều có thể ảnh hưởng đến sự lựa chọn.

## Làm sao để tránh việc "đóng gói" dự án quá đà?

Những mô tả dưới đây rất dễ dẫn đến các câu hỏi đào sâu vượt quá phạm vi chuẩn bị:

- Viết dự án của team thành do cá nhân độc lập phụ trách.
- Viết việc mới đọc qua giải pháp thành đã triển khai thực tế trên production.
- Viết kết quả test trên máy đơn thành QPS của production.
- Thêm vào các middleware thực tế không hề sử dụng chỉ để làm cho dự án có vẻ phức tạp.
- Chỉ nhớ kết luận trong video hướng dẫn, chưa từng xem code và cấu hình của dự án.

Dự án có thể tối ưu cách diễn đạt, nhưng trách nhiệm và kết quả không được hư cấu. Một giải pháp nào đó chỉ mới làm nghiên cứu (research) hoặc Demo, hãy nói thẳng: "Trên production cuối cùng không áp dụng, tôi từng phụ trách kiểm chứng giải pháp, kết luận thu được là……". Câu trả lời như vậy cũng hoàn toàn chịu được việc hỏi sâu tiếp theo.

## Tự kiểm tra trước khi phỏng vấn

Trước khi phỏng vấn hãy lấy một tờ giấy trắng, hoàn thành những việc dưới đây trong tình trạng không nhìn tài liệu:

- Dùng 30 giây và 3 phút lần lượt giới thiệu về dự án.
- Vẽ một luồng request cốt lõi và một luồng xử lý thất bại.
- Trình bày rõ 3 tính năng do bản thân phụ trách và vị trí code tương ứng.
- Chuẩn bị một lựa chọn công nghệ, một vấn đề hiệu năng và một lần fix sự cố/bug.
- Bổ sung tiêu chuẩn thống kê và tài liệu kiểm chứng cho từng con số trong CV.
- Trả lời "Tại sao sử dụng, Thất bại ra sao, Giám sát thế nào" cho từng middleware.

Nếu câu hỏi nào chỉ trả lời được định nghĩa của component, hãy quay lại code hoặc tài liệu dự án tra cứu lại một lần nữa. Giới thiệu dự án không cần phải nói quá hoa mỹ, chỉ cần làm cho người phỏng vấn có thể tiếp tục hỏi dựa trên câu trả lời của bạn, mà trong tay bạn lại có chi tiết thực tế để tiếp nhận câu hỏi, về cơ bản là đã đủ dùng.

## Đọc thêm

- [Hướng dẫn kinh nghiệm dự án](./project-experience-guide.md)
- [Hướng dẫn viết CV cho lập trình viên](./resume-guide.md)
- [Xử lý sự cố Online trong Java Backend](../java/jvm/jvm-in-action.md)
- [Câu hỏi phỏng vấn Thiết kế hệ thống hiệu năng cao](../high-performance/high-performance-system-interview-questions.md)
- [Câu hỏi phỏng vấn Thiết kế hệ thống độ sẵn sàng cao](../high-availability/high-availability-system-interview-questions.md)
- [Nhập môn Performance Testing và Stress Testing](../high-availability/performance-test.md)

<!-- @include: @article-footer.snippet.md -->
