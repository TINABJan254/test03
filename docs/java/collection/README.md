---
title: Java Collection chuyên đề: List, Map, Queue, Concurrent Collection và phân tích mã nguồn
description: Lộ trình học phỏng vấn và phân tích mã nguồn Java Collection, bao gồm List, Set, Map, Queue, ArrayList, HashMap, ConcurrentHashMap, blocking queue và các vấn đề sử dụng phổ biến.
category: Java
tag:
  - Java
  - Java集合
  - Java面试
sitemap:
  changefreq: weekly
  priority: 0.9
head:
  - - meta
    - name: keywords
      content: Java集合,Java集合面试题,ArrayList,LinkedList,HashMap,ConcurrentHashMap,CopyOnWriteArrayList,ArrayBlockingQueue,PriorityQueue,DelayQueue,集合源码
---

Java Collection là một trong những thư viện nền tảng được sử dụng thường xuyên nhất trong phát triển ứng dụng, đồng thời cũng là module được kiểm tra nhiều nhất trong các buổi phỏng vấn Java. Khi học về collection, bạn cần biết mỗi container phù hợp với những tình huống nào, đồng thời hiểu được các đánh đổi thiết kế đằng sau cơ chế mở rộng (expansion), xung đột hash, Iterator, thread-safe và concurrent container.

## Dành cho ai

- Các backend developer muốn nắm vững Java Collection Framework một cách có hệ thống.
- Các bạn đang chuẩn bị cho các câu hỏi phỏng vấn về List, Map, Queue, concurrent collection và phân tích mã nguồn.
- Những người thường xuyên sử dụng collection nhưng chưa nắm rõ các chi tiết như cơ chế mở rộng, xung đột hash, fail-fast, thread-safe, v.v.
- Các kỹ sư muốn đọc mã nguồn JDK, bắt đầu từ các lớp collection thông dụng để xây dựng năng lực phân tích mã nguồn.

## Trọng tâm học tập

- Hệ thống interface và các lớp triển khai phổ biến của List, Set, Map, Queue.
- Cấu trúc dữ liệu nền tảng và cơ chế mở rộng của `ArrayList`, `LinkedList`, `HashMap`, `LinkedHashMap`.
- Tư tưởng thread-safe của các concurrent container như `ConcurrentHashMap`, `CopyOnWriteArrayList`, `ArrayBlockingQueue`.
- Các chi tiết phổ biến: xung đột hash, chuyển đổi sang cây đỏ-đen (red-black tree), fail-fast, xóa phần tử khi duyệt Iterator, kiểm tra collection rỗng và ước lượng dung lượng.
- Khi phân tích mã nguồn, cách tiếp cận từ bốn góc độ: cấu trúc dữ liệu, các trường quan trọng, phương thức cốt lõi và kiểm soát đồng thời.

## Thứ tự đọc đề xuất

1. [Tổng hợp câu hỏi phỏng vấn Java Collection phổ biến (Phần 1)](./java-collection-questions-01.md): Xây dựng danh sách câu hỏi về Collection Framework và các container thông dụng trước.
2. [Tổng hợp câu hỏi phỏng vấn Java Collection phổ biến (Phần 2)](./java-collection-questions-02.md): Tiếp tục bổ sung về Map, Queue, concurrent collection và các chi tiết mã nguồn.
3. [Tổng hợp lưu ý khi sử dụng Java Collection](./java-collection-precautions-for-use.md): Nắm vững các cách sử dụng dễ gây lỗi trong dự án thực tế.
4. [Phân tích mã nguồn ArrayList](./arraylist-source-code.md), [Phân tích mã nguồn LinkedList](./linkedlist-source-code.md), [Phân tích mã nguồn HashMap](./hashmap-source-code.md): Bắt đầu đọc mã nguồn từ các container thông dụng nhất.
5. [Phân tích mã nguồn ConcurrentHashMap](./concurrent-hash-map-source-code.md), [Phân tích mã nguồn CopyOnWriteArrayList](./copyonwritearraylist-source-code.md), [Phân tích mã nguồn ArrayBlockingQueue](./arrayblockingqueue-source-code.md): Tiếp tục tìm hiểu concurrent collection và blocking queue.

## Các bài viết cốt lõi

### Phỏng vấn và quy chuẩn sử dụng Collection

- [Tổng hợp câu hỏi phỏng vấn Java Collection phổ biến (Phần 1)](./java-collection-questions-01.md): Bao quát các câu hỏi cơ bản về Collection Framework, List, Set, Map, Queue.
- [Tổng hợp câu hỏi phỏng vấn Java Collection phổ biến (Phần 2)](./java-collection-questions-02.md): Tiếp tục hệ thống về hash table, concurrent collection, mã nguồn collection và các điểm dễ nhầm.
- [Tổng hợp lưu ý khi sử dụng Java Collection](./java-collection-precautions-for-use.md): Tổng hợp các lưu ý về khởi tạo collection, kiểm tra rỗng, xóa khi duyệt, thread-safe và hiệu năng.

### Mã nguồn List và Map

- [Phân tích mã nguồn ArrayList](./arraylist-source-code.md): Hiểu về mảng động, cơ chế mở rộng, truy cập ngẫu nhiên và Iterator.
- [Phân tích mã nguồn LinkedList](./linkedlist-source-code.md): Hiểu về danh sách liên kết đôi, thao tác đầu/cuối và các tình huống phù hợp.
- [Phân tích mã nguồn HashMap](./hashmap-source-code.md): Hiểu về mảng, danh sách liên kết, cây đỏ-đen, hàm nhiễu (perturbation function), cơ chế mở rộng và chuyển đổi sang cây.
- [Phân tích mã nguồn LinkedHashMap](./linkedhashmap-source-code.md): Hiểu về thứ tự truy cập, thứ tự chèn và tình huống LRU.

Nếu bạn chưa quen với cấu trúc nền tảng, hãy đọc trước [Giải thích chi tiết cấu trúc dữ liệu tuyến tính](../../cs-basics/data-structure/linear-data-structure.md), [Tổng hợp câu hỏi phỏng vấn về Hash Table](../../cs-basics/data-structure/hash-table.md), [Giải thích chi tiết cây đỏ-đen](../../cs-basics/data-structure/red-black-tree.md) và [Tổng hợp câu hỏi phỏng vấn về LRU Cache](../../cs-basics/data-structure/lru-cache.md), sau đó quay lại đọc mã nguồn collection sẽ dễ hiểu hơn rất nhiều.

### Concurrent Collection và Queue

- [Phân tích mã nguồn ConcurrentHashMap](./concurrent-hash-map-source-code.md): Hiểu sự tiến hóa từ Segment Lock đến CAS + synchronized.
- [Phân tích mã nguồn CopyOnWriteArrayList](./copyonwritearraylist-source-code.md): Hiểu về Copy-on-Write và tình huống đọc nhiều ghi ít.
- [Phân tích mã nguồn ArrayBlockingQueue](./arrayblockingqueue-source-code.md): Hiểu về bounded blocking queue, Lock và condition queue.
- [Phân tích mã nguồn PriorityQueue (trả phí)](./priorityqueue-source-code.md): Hiểu về cấu trúc heap và priority queue.
- [Phân tích mã nguồn DelayQueue](./delayqueue-source-code.md): Hiểu về delay queue, priority queue và tình huống tác vụ định thời.

## Câu hỏi thường gặp

- `ArrayList` và `LinkedList` có gì khác nhau? Tại sao trong nhiều tình huống người ta ưu tiên dùng `ArrayList` hơn?
- Cấu trúc dữ liệu nền tảng của `HashMap` là gì? Khi nào thì nó chuyển sang cây đỏ-đen?
- Tại sao `HashMap` không thread-safe? Có thể xảy ra vấn đề gì khi mở rộng?
- `HashMap` và `ConcurrentHashMap` có gì khác nhau?
- Triển khai của `ConcurrentHashMap` trong JDK 7 và JDK 8 có gì thay đổi?
- Tại sao `CopyOnWriteArrayList` phù hợp với tình huống đọc nhiều ghi ít?
- fail-fast và fail-safe khác nhau như thế nào?
- Làm thế nào để xóa phần tử an toàn khi duyệt collection?
- `ArrayBlockingQueue`, `PriorityQueue`, `DelayQueue` lần lượt phù hợp với những tình huống nào?

## Chủ đề liên quan

- [Hệ thống kiến thức Java](../)
- [Chuyên đề Java cơ bản](../basis/)
- [Chuyên đề lập trình đồng thời Java](../concurrent/)
- [Chuyên đề JVM](../jvm/)
- [Cấu trúc dữ liệu](../../cs-basics/data-structure/)
- [Tổng hợp câu hỏi phỏng vấn về Hash Table](../../cs-basics/data-structure/hash-table.md)
- [Tổng hợp câu hỏi phỏng vấn về LRU Cache](../../cs-basics/data-structure/lru-cache.md)


<!-- @include: @article-footer.snippet.md -->
