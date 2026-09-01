---
title: Java集合常见面试题总结(上)
description: Java集合框架面试题总结：深入解析Collection/List/Set/Queue接口，对比ArrayList/LinkedList/HashMap等常用集合类，掌握集合底层数据结构与使用场景。
category: Java
tag:
  - Java集合
head:
  - - meta
    - name: keywords
      content: Java集合,Collection,List,Set,Queue,ArrayList,LinkedList,HashMap,集合框架,Java面试题
---

<!-- markdownlint-disable MD024 -->

## Tổng quan về Collection

### Tổng quan về Java Collection

Java Collection, còn được gọi là container (tập hợp/vùng chứa), chủ yếu được dẫn xuất từ hai interface chính: một là interface `Collection`, chủ yếu dùng để lưu trữ các phần tử đơn lẻ; interface còn lại là `Map`, chủ yếu dùng để lưu trữ các cặp key-value. Đối với interface `Collection`, bên dưới có ba interface con chính: `List`, `Set`, `Queue`.

Khung Java Collection (Java Collection Framework) được thể hiện như hình dưới đây:

![Java 集合框架概览](https://oss.javaguide.cn/github/javaguide/java/collection/java-collection-hierarchy.png)

Lưu ý: Hình trên chỉ liệt kê các quan hệ kế thừa/dẫn xuất chính, không liệt kê tất cả các mối quan hệ. Ví dụ đã bỏ qua các abstract class như `AbstractList`, `NavigableSet` và các class bổ trợ khác. Nếu muốn tìm hiểu sâu hơn, bạn có thể tự mình xem source code.

### ⭐️ Hãy nêu sự khác biệt giữa List, Set, Queue và Map?

- `List` (Trợ thủ đắc lực xử lý thứ tự): Các phần tử lưu trữ có thứ tự và có thể trùng lặp.
- `Set` (Chú trọng tính duy nhất): Các phần tử lưu trữ không được trùng lặp.
- `Queue` (Máy gọi số xếp hàng): Xếp thứ tự trước sau theo quy tắc xếp hàng nhất định, các phần tử lưu trữ có thứ tự và có thể trùng lặp.
- `Map` (Chuyên gia tìm kiếm bằng key): Sử dụng cặp key-value để lưu trữ, tương tự như hàm số toán học y=f(x), "x" đại diện cho key, "y" đại diện cho value, key không có thứ tự, không được trùng lặp; value không có thứ tự, có thể trùng lặp, mỗi key ánh xạ tối đa tới một value. Lưu ý rằng "không có thứ tự" ở đây đề cập đến các implementation như `HashMap` — không có thứ tự liên kết hiển thị giữa các cặp key-value. Các implementation như `LinkedHashMap` và `TreeMap` lại có thứ tự, chúng duy trì thứ tự của các cặp key-value thông qua các cấu trúc dữ liệu bổ sung (danh sách liên kết đôi hoặc cây đỏ đen).

### Tổng kết cấu trúc dữ liệu bên dưới của Collection Framework

Trước tiên hãy cùng xem các collection bên dưới interface `Collection`.

#### List

- `ArrayList`: Mảng `Object[]`. Chi tiết có thể xem: [Phân tích source code ArrayList](./arraylist-source-code.md).
- `Vector`: Mảng `Object[]`.
- `LinkedList`: Danh sách liên kết đôi (trước JDK1.6 là danh sách liên kết vòng, JDK1.7 đã hủy bỏ tính chất vòng). Chi tiết có thể xem: [Phân tích source code LinkedList](./linkedlist-source-code.md).

#### Set

- `HashSet` (Không thứ tự, duy nhất): Được triển khai dựa trên `HashMap`, bên dưới sử dụng `HashMap` để lưu trữ phần tử.
- `LinkedHashSet`: `LinkedHashSet` là class con của `HashSet`, và bên trong nó được triển khai thông qua `LinkedHashMap`.
- `TreeSet` (Có thứ tự, duy nhất): Cây đỏ đen (cây nhị phân tìm kiếm tự cân bằng).

#### Queue

- `PriorityQueue`: Mảng `Object[]` để triển khai min-heap. Chi tiết có thể xem: [Phân tích source code PriorityQueue](./priorityqueue-source-code.md).
- `DelayQueue`: `PriorityQueue`. Chi tiết có thể xem: [Phân tích source code DelayQueue](./delayqueue-source-code.md).
- `ArrayDeque`: Mảng động hai chiều có thể mở rộng kích thước.

Tiếp theo hãy xem các collection bên dưới interface `Map`.

#### Map

- `HashMap`: Trước JDK1.8, `HashMap` được cấu thành từ mảng + danh sách liên kết, mảng là thành phần chính của `HashMap`, còn danh sách liên kết chủ yếu tồn tại để giải quyết xung đột hash (phương pháp "chaining/kết nối" giải quyết xung đột). Từ JDK1.8 trở đi, việc giải quyết xung đột hash đã có sự thay đổi lớn: khi độ dài danh sách liên kết lớn hơn ngưỡng (mặc định là 8) (trước khi chuyển danh sách liên kết thành cây đỏ đen sẽ kiểm tra, nếu độ dài mảng hiện tại nhỏ hơn 64 thì sẽ chọn mở rộng mảng trước chứ không chuyển thành cây đỏ đen), danh sách liên kết sẽ được chuyển đổi thành cây đỏ đen để giảm thời gian tìm kiếm. Chi tiết có thể xem: [Phân tích source code HashMap](./hashmap-source-code.md), khái niệm cơ bản có thể xem trước [Tổng kết câu hỏi phỏng vấn Bảng băm (Hash Table)](../../cs-basics/data-structure/hash-table.md).
- `LinkedHashMap`: `LinkedHashMap` kế thừa từ `HashMap`, vì vậy cấu trúc bên dưới của nó vẫn dựa trên cấu trúc băm dạng chaining, tức là gồm mảng và danh sách liên kết hoặc cây đỏ đen. Ngoài ra, trên cơ sở cấu trúc trên, `LinkedHashMap` bổ sung thêm một danh sách liên kết đôi, giúp giữ nguyên thứ tự chèn của các cặp key-value. Đồng thời thông qua các thao tác tương ứng trên danh sách liên kết, nó triển khai logic liên quan đến thứ tự truy cập. Chi tiết có thể xem: [Phân tích source code LinkedHashMap](./linkedhashmap-source-code.md), bài tập tự viết LRU có thể xem [Tổng kết câu hỏi phỏng vấn LRU Cache](../../cs-basics/data-structure/lru-cache.md).
- `Hashtable`: Gồm mảng + danh sách liên kết, mảng là thành phần chính của `Hashtable`, còn danh sách liên kết chủ yếu tồn tại để giải quyết xung đột hash.
- `TreeMap`: Cây đỏ đen (cây nhị phân tìm kiếm tự cân bằng).

### Làm thế nào để lựa chọn Collection phù hợp?

Chúng ta chủ yếu dựa vào đặc điểm của các collection để lựa chọn collection phù hợp. Ví dụ:

- Khi cần lấy giá trị phần tử dựa trên key, ta chọn collection thuộc interface `Map`; khi cần sắp xếp chọn `TreeMap`, khi không cần sắp xếp chọn `HashMap`, khi cần đảm bảo thread-safe chọn `ConcurrentHashMap`.
- Khi chỉ cần lưu trữ các giá trị phần tử, ta chọn collection triển khai interface `Collection`; khi cần đảm bảo phần tử là duy nhất, chọn collection triển khai interface `Set` như `TreeSet` hoặc `HashSet`; khi không cần duy nhất, chọn collection triển khai interface `List` như `ArrayList` hoặc `LinkedList`, sau đó dựa vào đặc điểm của các collection triển khai các interface này để lựa chọn.

### Tại sao nên sử dụng Collection?

Khi cần lưu trữ một nhóm dữ liệu cùng kiểu, mảng (array) là một trong những container cơ bản và phổ biến nhất. Tuy nhiên, việc sử dụng mảng để lưu trữ đối tượng tồn tại một số hạn chế, vì trong thực tế phát triển phần mềm, kiểu dữ liệu cần lưu trữ rất đa dạng và số lượng không cố định. Lúc này, Java Collection phát huy tác dụng. So với mảng, Java Collection cung cấp phương pháp linh hoạt và hiệu quả hơn để lưu trữ nhiều đối tượng dữ liệu. Các class và interface trong Java Collection Framework có thể lưu trữ các đối tượng có kiểu dáng và số lượng khác nhau, đồng thời cung cấp nhiều phương thức thao tác đa dạng. So với mảng, ưu điểm của Java Collection là kích thước có thể thay đổi, hỗ trợ Generics, tích hợp sẵn các thuật toán,... Nhìn chung, Java Collection nâng cao tính linh hoạt trong việc lưu trữ và xử lý dữ liệu, thích ứng tốt hơn với nhu cầu dữ liệu đa dạng trong phát triển phần mềm hiện đại và hỗ trợ viết code chất lượng cao.

## List

### ⭐️ Sự khác biệt giữa ArrayList và Array (Mảng)?

`ArrayList` bên trong dựa trên mảng động (dynamic array) để triển khai, linh hoạt hơn nhiều so với `Array` (mảng tĩnh):

- `ArrayList` sẽ tự động mở rộng dung lượng (resize/扩容) dựa theo số lượng phần tử thực tế lưu trữ, cũng có thể chủ động thu nhỏ mảng bên dưới thông qua `trimToSize()`, trong khi `Array` sau khi được tạo ra thì không thể thay đổi chiều dài của nó nữa.
- `ArrayList` cho phép bạn sử dụng Generics để đảm bảo type safety (an toàn kiểu), còn `Array` thì không.
- `ArrayList` chỉ có thể lưu trữ đối tượng (Object). Đối với dữ liệu kiểu nguyên thủy (primitive types), cần sử dụng wrapper class tương ứng (như `Integer`, `Double`,...). `Array` có thể trực tiếp lưu trữ dữ liệu kiểu nguyên thủy và cũng có thể lưu trữ đối tượng.
- `ArrayList` hỗ trợ các thao tác phổ biến như chèn, xóa, duyệt,... và cung cấp các phương thức API phong phú như `add()`, `remove()`,... `Array` chỉ là một mảng có độ dài cố định, chỉ có thể truy cập phần tử theo chỉ số (index), không có khả năng thêm, xóa phần tử một cách động.
- `ArrayList` khi tạo không cần chỉ định kích thước, trong khi `Array` khi tạo bắt buộc phải chỉ định kích thước.

Dưới đây là so sánh đơn giản về cách sử dụng giữa hai loại:

`Array`:

```java
 // Khởi tạo một mảng kiểu String
 String[] stringArr = new String[]{"hello", "world", "!"};
 // Sửa giá trị phần tử trong mảng
 stringArr[0] = "goodbye";
 System.out.println(Arrays.toString(stringArr));// [goodbye, world, !]
 // Xóa phần tử trong mảng, cần tự di chuyển các phần tử phía sau
 for (int i = 0; i < stringArr.length - 1; i++) {
     stringArr[i] = stringArr[i + 1];
 }
 stringArr[stringArr.length - 1] = null;
 System.out.println(Arrays.toString(stringArr));// [world, !, null]
```

`ArrayList`:

```java
// Khởi tạo một ArrayList kiểu String
 ArrayList<String> stringList = new ArrayList<>(Arrays.asList("hello", "world", "!"));
// Thêm phần tử vào ArrayList
 stringList.add("goodbye");
 System.out.println(stringList);// [hello, world, !, goodbye]
 // Sửa phần tử trong ArrayList
 stringList.set(0, "hi");
 System.out.println(stringList);// [hi, world, !, goodbye]
 // Xóa phần tử trong ArrayList
 stringList.remove(0);
 System.out.println(stringList); // [world, !, goodbye]
```

### Sự khác biệt giữa ArrayList và Vector? (Chỉ cần tham khảo)

- `ArrayList` là implementation chính của `List`, bên dưới sử dụng `Object[]` để lưu trữ, thích hợp cho việc tìm kiếm thường xuyên, không thread-safe.
- `Vector` là implementation cũ của `List`, bên dưới sử dụng `Object[]` để lưu trữ, thread-safe.

### Sự khác biệt giữa Vector và Stack? (Chỉ cần tham khảo)

- `Vector` và `Stack` đều thread-safe, cả hai đều sử dụng từ khóa `synchronized` để xử lý đồng bộ hóa.
- `Stack` kế thừa từ `Vector`, là một stack LIFO (vào sau ra trước), còn `Vector` là một list.

Cùng với sự phát triển của lập trình đa luồng (concurrent programming) trong Java, `Vector` và `Stack` đã bị loại bỏ/lạc hậu, khuyến nghị nên sử dụng các class collection đồng thời (ví dụ như `ConcurrentHashMap`, `CopyOnWriteArrayList`,...) hoặc tự triển khai phương thức thread-safe để hỗ trợ thao tác đa luồng an toàn.

### ArrayList có thể thêm giá trị null không?

Trong `ArrayList` có thể lưu trữ bất kỳ đối tượng kiểu nào, bao gồm cả giá trị `null`. Tuy nhiên, không khuyến khích thêm giá trị `null` vào `ArrayList`, giá trị `null` không có ý nghĩa và làm cho code khó bảo trì, ví dụ nếu quên kiểm tra null sẽ dẫn đến `NullPointerException`.

Ví dụ code:

```java
ArrayList<String> listOfStrings = new ArrayList<>();
listOfStrings.add(null);
listOfStrings.add("java");
System.out.println(listOfStrings);
```

Output:

```plain
[null, java]
```

### ⭐️ Độ phức tạp thời gian khi chèn và xóa phần tử trong ArrayList?

Đối với thao tác chèn (insert):

- Chèn ở đầu: Do cần phải dịch chuyển tất cả phần tử lùi về sau một vị trí, nên độ phức tạp thời gian là O(n).
- Chèn ở cuối: Khi dung lượng của `ArrayList` chưa đạt giới hạn, việc chèn phần tử vào cuối danh sách có độ phức tạp thời gian là O(1), vì nó chỉ cần thêm phần tử vào cuối mảng; khi dung lượng đã đạt giới hạn và cần mở rộng (resize/扩容), sẽ cần thực hiện một thao tác O(n) để copy mảng cũ sang mảng mới lớn hơn, sau đó mới thực hiện thao tác O(1) để thêm phần tử.
- Chèn tại vị trí chỉ định: Cần dịch chuyển tất cả phần tử phía sau vị trí mục tiêu lùi về sau một vị trí, sau đó mới đặt phần tử mới vào vị trí chỉ định. Quá trình này trung bình cần di chuyển n/2 phần tử, do đó độ phức tạp thời gian là O(n).

Đối với thao tác xóa (delete):

- Xóa ở đầu: Do cần dịch chuyển tất cả phần tử tiến lên trước một vị trí, nên độ phức tạp thời gian là O(n).
- Xóa ở cuối: Khi phần tử cần xóa nằm ở cuối danh sách, độ phức tạp thời gian là O(1).
- Xóa tại vị trí chỉ định: Cần dịch chuyển tất cả phần tử phía sau phần tử mục tiêu tiến lên trước một vị trí để lấp khoảng trống bị xóa, do đó trung bình cần di chuyển n/2 phần tử, độ phức tạp thời gian là O(n).

Dưới đây là một ví dụ minh họa đơn giản:

```java
// Mảng bên dưới của ArrayList có kích thước là 10, hiện tại đang lưu trữ 7 phần tử
+---+---+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 | 6 | 7 |   |   |   |
+---+---+---+---+---+---+---+---+---+---+
  0   1   2   3   4   5   6   7   8   9
// Chèn phần tử 8 vào vị trí index 1, tất cả phần tử đằng sau phần tử đó phải dịch sang phải 1 vị trí
+---+---+---+---+---+---+---+---+---+---+
| 1 | 8 | 2 | 3 | 4 | 5 | 6 | 7 |   |   |
+---+---+---+---+---+---+---+---+---+---+
  0   1   2   3   4   5   6   7   8   9
// Xóa phần tử tại vị trí index 1, tất cả phần tử đằng sau phần tử đó phải dịch sang trái 1 vị trí
+---+---+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 | 6 | 7 |   |   |   |
+---+---+---+---+---+---+---+---+---+---+
  0   1   2   3   4   5   6   7   8   9
```

### ⭐️ Độ phức tạp thời gian khi chèn và xóa phần tử trong LinkedList?

- Chèn/Xóa ở đầu: Chỉ cần sửa đổi con trỏ của node đầu là có thể hoàn thành thao tác chèn/xóa, do đó độ phức tạp thời gian là O(1).
- Chèn/Xóa ở cuối: Chỉ cần sửa đổi con trỏ của node cuối là có thể hoàn thành thao tác chèn/xóa, do đó độ phức tạp thời gian là O(1).
- Chèn/Xóa tại vị trí chỉ định: Cần di chuyển đến vị trí chỉ định trước, sau đó mới sửa con trỏ của node chỉ định để hoàn thành việc chèn/xóa. Tuy nhiên do có con trỏ đầu và cuối, có thể xuất phát từ con trỏ gần hơn, nên trung bình cần duyệt n/4 phần tử, độ phức tạp thời gian là O(n).

Dưới đây là một ví dụ đơn giản: Giả sử chúng ta muốn xóa node 9, trước tiên cần duyệt danh sách liên kết để tìm node đó. Sau đó thực hiện thay đổi hướng chỉ của các con trỏ node tương ứng, source code cụ thể có thể tham khảo: [Phân tích source code LinkedList](https://javaguide.cn/java/collection/linkedlist-source-code.html).

![Logic của phương thức unlink](https://oss.javaguide.cn/github/javaguide/java/collection/linkedlist-unlink.jpg)

### Tại sao LinkedList không thể triển khai interface RandomAccess?

`RandomAccess` là một marker interface (interface đánh dấu), được dùng để biểu thị class triển khai interface này hỗ trợ truy cập ngẫu nhiên (tức là có thể truy cập phần tử nhanh chóng qua index). Do cấu trúc dữ liệu bên dưới của `LinkedList` là danh sách liên kết, địa chỉ bộ nhớ không liên tục, chỉ có thể định vị thông qua con trỏ, không hỗ trợ truy cập ngẫu nhiên nhanh chóng, do đó không thể triển khai interface `RandomAccess`.

### ⭐️ Sự khác biệt giữa ArrayList và LinkedList?

- **Có đảm bảo thread-safe hay không:** Cả `ArrayList` và `LinkedList` đều không được đồng bộ hóa (unsynchronized), nghĩa là không đảm bảo thread-safe;
- **Cấu trúc dữ liệu bên dưới:** `ArrayList` sử dụng **mảng `Object`**; `LinkedList` sử dụng **danh sách liên kết đôi** (trước JDK1.6 là danh sách liên kết vòng, JDK1.7 đã hủy bỏ tính chất vòng. Chú ý sự khác biệt giữa danh sách liên kết đôi và danh sách liên kết vòng đôi, sẽ được giới thiệu ở bên dưới!)
- **Thao tác chèn và xóa có bị ảnh hưởng bởi vị trí phần tử hay không:**
  - `ArrayList` sử dụng mảng để lưu trữ, do đó độ phức tạp thời gian khi chèn và xóa phần tử chịu ảnh hưởng bởi vị trí phần tử. Ví dụ: khi thực thi phương thức `add(E e)`, `ArrayList` mặc định sẽ thêm phần tử chỉ định vào cuối danh sách này, trường hợp này độ phức tạp thời gian là O(1). Nhưng nếu muốn chèn hoặc xóa phần tử tại vị trí chỉ định `i` (`add(int index, E element)`), độ phức tạp thời gian sẽ là O(n). Bởi vì khi thực hiện các thao tác trên, phần tử thứ `i` và (n-i) phần tử đằng sau phần tử thứ `i` trong collection đều phải thực hiện thao tác dịch chuyển lùi/tiến 1 vị trí.
  - `LinkedList` sử dụng danh sách liên kết để lưu trữ, nên việc chèn hoặc xóa phần tử ở đầu/cuối không bị ảnh hưởng bởi vị trí phần tử (`add(E e)`, `addFirst(E e)`, `addLast(E e)`, `removeFirst()`, `removeLast()`), độ phức tạp thời gian là O(1). Nếu muốn chèn hoặc xóa phần tử tại vị trí chỉ định `i` (`add(int index, E element)`, `remove(Object o)`, `remove(int index)`), độ phức tạp thời gian là O(n), vì cần phải di chuyển đến vị trí chỉ định trước rồi mới chèn/xóa.
- **Có hỗ trợ truy cập ngẫu nhiên nhanh hay không:** `LinkedList` không hỗ trợ truy cập phần tử ngẫu nhiên hiệu quả, còn `ArrayList` (triển khai interface `RandomAccess`) thì hỗ trợ. Truy cập ngẫu nhiên nhanh là việc thông qua index của phần tử để lấy nhanh đối tượng phần tử (tương ứng với phương thức `get(int index)`).
- **Dung lượng bộ nhớ tiêu tốn:** Việc lãng phí không gian của `ArrayList` chủ yếu thể hiện ở việc dự phòng một khoảng dung lượng nhất định ở cuối danh sách list, trong khi chi phí không gian của `LinkedList` thể hiện ở việc mỗi phần tử của nó đều tiêu tốn nhiều không gian hơn `ArrayList` (vì phải lưu trữ con trỏ trỏ đến phần tử kế tiếp, phần tử phía trước và dữ liệu).

Trong dự án, chúng ta thường không sử dụng `LinkedList`. Các kịch bản cần dùng `LinkedList` hầu như đều có thể thay thế bằng `ArrayList`, và hiệu năng thường sẽ tốt hơn! Ngay cả tác giả của `LinkedList` là Josh Bloch cũng từng nói rằng bản thân chưa bao giờ sử dụng `LinkedList`.

![](https://oss.javaguide.cn/github/javaguide/redisimage-20220412110853807.png)

Ngoài ra, đừng nghĩ theo bản năng rằng `LinkedList` là danh sách liên kết thì sẽ thích hợp nhất cho các kịch bản thêm/xóa phần tử. Như tôi đã nói ở trên, `LinkedList` chỉ có độ phức tạp thời gian xấp xỉ O(1) khi chèn hoặc xóa phần tử ở đầu/cuối, các trường hợp thêm/xóa phần tử khác độ phức tạp thời gian trung bình đều là O(n).

#### Nội dung bổ sung: Danh sách liên kết đôi và danh sách liên kết vòng đôi

**Danh sách liên kết đôi (Doubly Linked List):** Chứa hai con trỏ, một con trỏ `prev` trỏ đến node phía trước, một con trỏ `next` trỏ đến node phía sau.

![Danh sách liên kết đôi](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-linkedlist.png)

**Danh sách liên kết vòng đôi (Circular Doubly Linked List):** Con trỏ `next` của node cuối cùng trỏ đến `head`, còn con trỏ `prev` của `head` trỏ đến node cuối cùng, tạo thành một vòng khép kín.

![Danh sách liên kết vòng đôi](https://oss.javaguide.cn/github/javaguide/cs-basics/data-structure/bidirectional-circular-linkedlist.png)

#### Nội dung bổ sung: Interface RandomAccess

```java
public interface RandomAccess {
}
```

Xem source code chúng ta thấy thực tế trong interface `RandomAccess` không định nghĩa bất kỳ điều gì. Vì vậy, theo tôi interface `RandomAccess` chỉ là một đánh dấu (marker). Đánh dấu điều gì? Đánh dấu class triển khai interface này có khả năng truy cập ngẫu nhiên.

Trong phương thức `binarySearch()`, nó sẽ kiểm tra list truyền vào có phải là instance của `RandomAccess` hay không. Nếu phải thì gọi phương thức `indexedBinarySearch()`, nếu không thì gọi phương thức `iteratorBinarySearch()`.

```java
    public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

`ArrayList` triển khai interface `RandomAccess`, trong khi `LinkedList` thì không triển khai. Tại sao lại như vậy? Tôi nghĩ điều này liên quan đến cấu trúc dữ liệu bên dưới! `ArrayList` bên dưới là mảng, còn `LinkedList` bên dưới là danh sách liên kết. Mảng tự nhiên hỗ trợ truy cập ngẫu nhiên với độ phức tạp thời gian là O(1), nên gọi là truy cập ngẫu nhiên nhanh. Danh sách liên kết cần phải duyệt đến vị trí cụ thể mới truy cập được phần tử tại vị trí đó, độ phức tạp thời gian là O(n), nên không hỗ trợ truy cập ngẫu nhiên nhanh. `ArrayList` triển khai interface `RandomAccess` nhằm biểu thị rằng nó có chức năng truy cập ngẫu nhiên nhanh. Interface `RandomAccess` chỉ là đánh dấu, không phải nói rằng `ArrayList` triển khai interface `RandomAccess` thì mới có chức năng truy cập ngẫu nhiên nhanh!

### ⭐️ Hãy nói về cơ chế mở rộng (resize/扩容) của ArrayList

Chi tiết xem tại bài viết này của tác giả: [Phân tích cơ chế mở rộng ArrayList](https://javaguide.cn/java/collection/arraylist-source-code.html#arraylist-扩容机制分析).

### ⭐️ Fail-fast và fail-safe trong Collection là gì?

`fail-fast` (thất bại nhanh) và `fail-safe` (thất bại an toàn) là hai triết lý thiết kế và chiến lược chịu lỗi hoàn toàn khác nhau của Java Collection Framework khi xử lý các vấn đề sửa đổi đồng thời (concurrency modification).

Về `fail-fast`, trích dẫn phát biểu trong một bài viết trên Medium về `fail-fast` và `fail-safe`:

> Fail-fast systems are designed to immediately stop functioning upon encountering an unexpected condition. This immediate failure helps to catch errors early, making debugging more straightforward.

Tư tưởng thất bại nhanh chính là chủ động báo lỗi và dừng chạy ngay lập tức đối với các ngoại lệ có thể xảy ra. Thông qua việc phát hiện và dừng lỗi sớm nhất có thể, giúp giảm thiểu rủi ro sự cố lan truyền trong hệ thống (cascading failures).

Hầu hết các collection trong package `java.util` (như `ArrayList`, `HashMap`) đều không thread-safe. Để có thể sớm phát hiện các rủi ro thread-safe do thao tác đồng thời gây ra, người ta đề xuất việc duy trì một biến `modCount` để ghi lại số lần sửa đổi. Trong quá trình lặp (iteration), bằng cách so sánh số lần sửa đổi dự kiến `expectedModCount` với `modCount` xem có nhất quán hay không để xác định xem có tồn tại thao tác đồng thời hay không, từ đó triển khai fail-fast, đảm bảo tránh việc thực thi code phức tạp không cần thiết khi xảy ra bất thường.

**Ví dụ ArrayList (fail-fast):**

```java
     // Sử dụng ArrayList không thread-safe, đây là một loại collection fail-fast
      List<Integer> list = new ArrayList<>();
      CountDownLatch latch = new CountDownLatch(2);

      for (int i = 0; i < 5; i++) {
          list.add(i);
      }
      System.out.println("Initial list: " + list);

      Thread t1 = new Thread(() -> {
          try {
              for (Integer i : list) {
                  System.out.println("Iterator Thread (t1) sees: " + i);
                  Thread.sleep(100);
              }
          } catch (ConcurrentModificationException e) {
              System.err.println("!!! Iterator Thread (t1) caught ConcurrentModificationException as expected.");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              latch.countDown();
          }
      });

      Thread t2 = new Thread(() -> {
          try {
              Thread.sleep(50);
              System.out.println("-> Modifier Thread (t2) is removing element 1...");
              list.remove(Integer.valueOf(1));
              System.out.println("-> Modifier Thread (t2) finished removal.");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              latch.countDown();
          }
      });

      t1.start();
      t2.start();
      latch.await();

      System.out.println("Final list state: " + list);
```

Output:

```
Initial list: [0, 1, 2, 3, 4]
Iterator Thread (t1) sees: 0
-> Modifier Thread (t2) is removing element 1...
-> Modifier Thread (t2) finished removal.
!!! Iterator Thread (t1) caught ConcurrentModificationException as expected.
Final list state: [0, 2, 3, 4]
```

Chương trình sau khi thread t2 sửa đổi danh sách, lần lặp tiếp theo của thread t1 ngay lập tức ném ra `ConcurrentModificationException`. Điều này là do iterator của `ArrayList` trong mỗi lần gọi `next()` đều kiểm tra xem `modCount` có bị thay đổi hay không. Một khi phát hiện collection bị sửa đổi mà iterator không hay biết, nó sẽ lập tức "fail-fast" để ngăn chặn việc tiếp tục thao tác trên dữ liệu không nhất quán gây ra những hậu quả không lường trước được.

Về điều này chúng ta cũng đưa ra phương thức `next` của iterator bên dưới vòng lặp `for` khi lấy phần tử tiếp theo, có thể thấy `checkForComodification` bên trong nó chứa logic so sánh số lần sửa đổi:

```java
 public E next() {
 			// Kiểm tra xem có tồn tại sửa đổi đồng thời hay không
            checkForComodification();
            //......
            // Trả về phần tử tiếp theo
            return (E) elementData[lastRet = i];
        }

final void checkForComodification() {
		// Khi số lần lặp hiện tại và số lần sửa đổi dự kiến không nhất quán, sẽ ném ra ConcurrentModificationException
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }

```

Còn `fail-safe` tức là thất bại an toàn, nó nhằm mục đích ngay cả khi đối mặt với tình huống bất ngờ vẫn có thể khôi phục và tiếp tục chạy, điều này khiến nó đặc biệt phù hợp với các môi trường không xác định hoặc không ổn định:

> Fail-safe systems take a different approach, aiming to recover and continue even in the face of unexpected conditions. This makes them particularly suited for uncertain or volatile environments.

Tư tưởng này thường được ứng dụng trong các concurrent container, triển khai kinh điển nhất chính là `CopyOnWriteArrayList`. Thông qua tư tưởng Sao chép khi ghi (Copy-On-Write), đảm bảo khi thực hiện thao tác sửa đổi sẽ sao chép ra một bản snapshot. Sau khi hoàn thành thao tác thêm hoặc xóa dựa trên bản snapshot này, tham chiếu mảng bên dưới của `CopyOnWriteArrayList` sẽ trỏ đến vùng không gian mảng mới này, từ đó tránh được sự can thiệp của việc sửa đổi đồng thời khi đang lặp gây ra rủi ro an toàn thao tác đồng thời. Tất nhiên cách làm này cũng có nhược điểm, đó là khi thực hiện thao tác duyệt sẽ không thể nhận được kết quả thời gian thực (real-time):

![](https://oss.javaguide.cn/github/javaguide/java/collection/fail-fast-and-fail-safe-copyonwritearraylist.png)

Tương ứng chúng ta cũng đưa ra code lõi của `CopyOnWriteArrayList` khi triển khai `fail-safe`. Có thể thấy cách triển khai của nó là thông qua `getArray` lấy tham chiếu mảng, sau đó thông qua `Arrays.copyOf` thu được một bản snapshot của mảng. Sau khi hoàn thành thao tác thêm dựa trên snapshot này, biến `array` bên dưới sẽ được sửa đổi để trỏ đến địa chỉ tham chiếu mới, từ đó hoàn thành thao tác Copy-On-Write:

```java
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
        	// Lấy mảng ban đầu
            Object[] elements = getArray();
            int len = elements.length;
            // Sao chép ra một bản snapshot bộ nhớ dựa trên mảng ban đầu
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            // Thực hiện thao tác thêm
            newElements[len] = e;
            // array trỏ đến mảng mới
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

## Set

### Sự khác biệt giữa Comparable và Comparator

Interface `Comparable` và interface `Comparator` đều là các interface dùng để sắp xếp trong Java, chúng đóng vai trò quan trọng trong việc so sánh kích thước và sắp xếp giữa các đối tượng của class triển khai:

- Interface `Comparable` thực chất thuộc package `java.lang`, nó có phương thức `compareTo(Object obj)` dùng để sắp xếp.
- Interface `Comparator` thực chất thuộc package `java.util`, nó có phương thức `compare(Object obj1, Object obj2)` dùng để sắp xếp.

Thông thường khi cần sử dụng sắp xếp tùy chỉnh cho một collection, chúng ta sẽ override phương thức `compareTo()` hoặc `compare()`. Khi cần triển khai hai cách sắp xếp cho cùng một collection, ví dụ một đối tượng `song` có tên bài hát và tên ca sĩ cần áp dụng hai phương pháp sắp xếp khác nhau, chúng ta có thể override phương thức `compareTo()` đồng thời sử dụng phương thức `Comparator` tự định nghĩa, hoặc dùng hai `Comparator` để lần lượt triển khai sắp xếp tên bài hát và sắp xếp tên ca sĩ. Cách thứ hai đồng nghĩa với việc chúng ta chỉ có thể sử dụng phiên bản 2 tham số của `Collections.sort()`.

#### Sắp xếp tùy chỉnh bằng Comparator

```java
ArrayList<Integer> arrayList = new ArrayList<Integer>();
arrayList.add(-1);
arrayList.add(3);
arrayList.add(3);
arrayList.add(-5);
arrayList.add(7);
arrayList.add(4);
arrayList.add(-9);
arrayList.add(-7);
System.out.println("Mảng ban đầu:");
System.out.println(arrayList);
// void reverse(List list): Đảo ngược
Collections.reverse(arrayList);
System.out.println("Collections.reverse(arrayList):");
System.out.println(arrayList);

// void sort(List list): Sắp xếp tăng dần theo thứ tự tự nhiên
Collections.sort(arrayList);
System.out.println("Collections.sort(arrayList):");
System.out.println(arrayList);
// Cách dùng sắp xếp tùy chỉnh
Collections.sort(arrayList, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2.compareTo(o1);
    }
});
System.out.println("Sau khi sắp xếp tùy chỉnh:");
System.out.println(arrayList);
```

Output:

```plain
Mảng ban đầu:
[-1, 3, 3, -5, 7, 4, -9, -7]
Collections.reverse(arrayList):
[-7, -9, 4, 7, -5, 3, 3, -1]
Collections.sort(arrayList):
[-9, -7, -5, -1, 3, 3, 4, 7]
Sau khi sắp xếp tùy chỉnh:
[7, 4, 3, 3, -1, -5, -7, -9]
```

#### Override phương thức compareTo để sắp xếp theo tuổi

```java
// Đối tượng Person chưa triển khai interface Comparable nên bắt buộc phải triển khai, như vậy mới không bị lỗi và giúp dữ liệu trong TreeMap xếp theo thứ tự
// Class String ở ví dụ trước đã mặc định triển khai interface Comparable, chi tiết xem tài liệu API của String. Các class khác như Integer cũng đã triển khai Comparable nên không cần triển khai thêm
public  class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     * Override phương thức compareTo để sắp xếp theo tuổi
     */
    @Override
    public int compareTo(Person o) {
        if (this.age > o.getAge()) {
            return 1;
        }
        if (this.age < o.getAge()) {
            return -1;
        }
        return 0;
    }
}

```

```java
    public static void main(String[] args) {
        TreeMap<Person, String> pdata = new TreeMap<Person, String>();
        pdata.put(new Person("张三", 30), "zhangsan");
        pdata.put(new Person("李四", 20), "lisi");
        pdata.put(new Person("王五", 10), "wangwu");
        pdata.put(new Person("小红", 5), "xiaohong");
        // Lấy key đồng thời lấy giá trị tương ứng với key
        Set<Person> keys = pdata.keySet();
        for (Person key : keys) {
            System.out.println(key.getAge() + "-" + key.getName());

        }
    }
```

Output：

```plain
5-小红
10-王五
20-李四
30-张三
```

### Ý nghĩa của tính không thứ tự (Unordered) và tính không trùng lặp (Unique) là gì?

- Tính không thứ tự (Unordered) không đồng nghĩa với tính ngẫu nhiên (Randomness). Tính không thứ tự có nghĩa là dữ liệu lưu trữ không được thêm vào mảng bên dưới theo thứ tự chỉ số (index) của mảng, mà được quyết định dựa trên giá trị hash của dữ liệu.
- Tính không trùng lặp (Unique) có nghĩa là phần tử được thêm vào khi so sánh bằng `equals()` sẽ trả về `false`. Cần phải đồng thời override cả phương thức `equals()` và phương thức `hashCode()`.

### So sánh sự giống và khác nhau giữa HashSet, LinkedHashSet và TreeSet

- `HashSet`, `LinkedHashSet` và `TreeSet` đều là các implementation class của interface `Set`, đều đảm bảo tính duy nhất của phần tử và đều không thread-safe.
- Sự khác biệt chính giữa `HashSet`, `LinkedHashSet` và `TreeSet` nằm ở cấu trúc dữ liệu bên dưới. Cấu trúc dữ liệu bên dưới của `HashSet` là bảng băm (dựa trên `HashMap`). Cấu trúc dữ liệu bên dưới của `LinkedHashSet` là danh sách liên kết và bảng băm, thứ tự chèn và lấy phần tử tuân theo FIFO. Cấu trúc dữ liệu bên dưới của `TreeSet` là cây đỏ đen, các phần tử có thứ tự, cách thức sắp xếp bao gồm sắp xếp tự nhiên và sắp xếp tùy chỉnh.
- Cấu trúc dữ liệu bên dưới khác nhau dẫn đến kịch bản ứng dụng của ba loại này cũng khác nhau. `HashSet` dùng cho kịch bản không cần đảm bảo thứ tự chèn và lấy phần tử. `LinkedHashSet` dùng cho kịch bản cần đảm bảo thứ tự chèn và lấy phần tử tuân theo FIFO. `TreeSet` dùng cho kịch bản cần hỗ trợ quy tắc sắp xếp tùy chỉnh cho các phần tử.

## Queue

### Sự khác biệt giữa Queue và Deque

`Queue` là hàng đợi đơn (single-ended queue), chỉ có thể chèn phần tử ở một đầu và xóa phần tử ở đầu còn lại, về mặt triển khai thường tuân theo quy tắc **Vào trước ra trước (FIFO)**.

`Queue` mở rộng interface `Collection`. Dựa trên **sự khác biệt về cách xử lý khi thao tác thất bại do vấn đề dung lượng**, các phương thức được chia thành hai nhóm: một nhóm sẽ ném ra ngoại lệ (exception) khi thất bại, nhóm còn lại sẽ trả về giá trị đặc biệt.

| Interface `Queue` | Ném ra Exception | Trả về giá trị đặc biệt |
| ------------ | --------- | ---------- |
| Chèn vào cuối hàng đợi | add(E e) | offer(E e) |
| Xóa ở đầu hàng đợi | remove() | poll() |
| Truy vấn phần tử đầu hàng đợi | element() | peek() |

`Deque` là hàng đợi hai đầu (double-ended queue), có thể chèn hoặc xóa phần tử ở cả hai đầu hàng đợi.

`Deque` mở rộng interface `Queue`, bổ sung các phương thức chèn và xóa ở đầu và cuối hàng đợi, cũng được chia làm hai nhóm dựa trên cách xử lý khi thất bại:

| Interface `Deque` | Ném ra Exception | Trả về giá trị đặc biệt |
| ------------ | ------------- | --------------- |
| Chèn vào đầu hàng đợi | addFirst(E e) | offerFirst(E e) |
| Chèn vào cuối hàng đợi | addLast(E e) | offerLast(E e) |
| Xóa ở đầu hàng đợi | removeFirst() | pollFirst() |
| Xóa ở cuối hàng đợi | removeLast() | pollLast() |
| Truy vấn phần tử đầu hàng đợi | getFirst() | peekFirst() |
| Truy vấn phần tử cuối hàng đợi | getLast() | peekLast() |

Trên thực tế, `Deque` còn cung cấp các phương thức khác như `push()` và `pop()`, có thể dùng để mô phỏng stack.

### Sự khác biệt giữa ArrayDeque và LinkedList

`ArrayDeque` và `LinkedList` đều triển khai interface `Deque`, cả hai đều có tính năng của hàng đợi, vậy giữa chúng có những điểm gì khác biệt?

- `ArrayDeque` được triển khai dựa trên mảng có độ dài biến đổi và con trỏ kép (double pointer), trong khi `LinkedList` được triển khai thông qua danh sách liên kết.

- `ArrayDeque` không hỗ trợ lưu trữ dữ liệu `null`, nhưng `LinkedList` thì có hỗ trợ.

- `ArrayDeque` được đưa vào từ JDK1.6, trong khi `LinkedList` đã tồn tại từ JDK1.2.

- `ArrayDeque` khi chèn có thể xảy ra quá trình mở rộng (resize/扩容), tuy nhiên thao tác chèn sau khi chia đều (amortized) vẫn là O(1). Mặc dù `LinkedList` không cần mở rộng dung lượng, nhưng mỗi lần chèn dữ liệu đều cần cấp phát không gian heap mới, hiệu năng trung bình chậm hơn.

Xét từ góc độ hiệu năng, lựa chọn `ArrayDeque` để triển khai hàng đợi sẽ tốt hơn `LinkedList`. Ngoài ra, `ArrayDeque` cũng có thể được dùng để triển khai stack.

### Hãy nói về PriorityQueue

`PriorityQueue` được đưa vào từ JDK1.5. Sự khác biệt giữa nó và `Queue` thông thường nằm ở chỗ thứ tự xuất hàng đợi (dequeue) của phần tử có liên quan đến độ ưu tiên, tức là phần tử có độ ưu tiên cao nhất luôn được xuất hàng đợi trước.

Dưới đây là một số điểm chính liên quan:

- `PriorityQueue` sử dụng cấu trúc dữ liệu binary heap để triển khai, bên dưới dùng mảng có độ dài biến đổi để lưu trữ dữ liệu.
- `PriorityQueue` thông qua việc vun lên (swim/up-heap) và chìm xuống (sink/down-heap) của các phần tử trong heap để đạt được độ phức tạp thời gian O(log n) khi chèn phần tử và xóa phần tử đỉnh heap.
- `PriorityQueue` không thread-safe và không hỗ trợ lưu trữ `null`. Khi không cung cấp `Comparator`, các phần tử cần triển khai `Comparable`; khi cung cấp `Comparator`, các phần tử phải có thể so sánh lẫn nhau thông qua bộ so sánh đó.
- `PriorityQueue` mặc định là min-heap, nhưng có thể nhận một `Comparator` làm tham số constructor để từ đó tùy chỉnh độ ưu tiên trước sau của phần tử.

`PriorityQueue` trong các buổi phỏng vấn thường xuất hiện nhiều hơn ở phần viết thuật toán trực tiếp (coding interview), các bài toán điển hình bao gồm Heap Sort, tìm phần tử lớn thứ K, duyệt đồ thị có trọng số,... do đó bạn cần phải sử dụng nó thật thành thạo.

Nếu muốn bổ sung mẫu thuật toán Heap và Top K trước, bạn có thể xem [Giải thích chi tiết về Heap](../../cs-basics/data-structure/heap.md) và [Tổng kết câu hỏi phỏng vấn bài toán Top K](../../cs-basics/algorithms/top-k.md).

### BlockingQueue là gì?

`BlockingQueue` (hàng đợi nghẽn/hàng đợi chặn) là một interface kế thừa từ `Queue`. Nó cung cấp 4 cách xử lý khác nhau cho thao tác chèn và xóa: ném ngoại lệ, trả về giá trị đặc biệt, tiếp tục block (chặn) và chờ hết thời gian timeout. Trong đó, `take()` có thể block khi hàng đợi rỗng, còn `put()` có thể block khi hàng đợi có giới hạn dung lượng đã đầy.

```java
public interface BlockingQueue<E> extends Queue<E> {
  // ...
}
```

`BlockingQueue` thường được sử dụng trong mô hình Producer-Consumer (Nhà sản xuất - Người tiêu dùng). Thread producer sẽ thêm dữ liệu vào hàng đợi, còn thread consumer sẽ lấy dữ liệu từ hàng đợi ra để xử lý.

![BlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/blocking-queue.png)

### Các implementation class của BlockingQueue gồm những loại nào?

![Các implementation class của BlockingQueue](https://oss.javaguide.cn/github/javaguide/java/collection/blocking-queue-hierarchy.png)

Trong Java, các implementation class của hàng đợi nghẽn (blocking queue) thường dùng bao gồm các loại sau:

1. `ArrayBlockingQueue`: Hàng đợi nghẽn có giới hạn (bounded blocking queue) được triển khai bằng mảng. Khi tạo cần chỉ định kích thước dung lượng, và hỗ trợ cơ chế truy cập khóa (lock access) theo hai phương thức fair (công bằng) và non-fair (không công bằng).
2. `LinkedBlockingQueue`: Hàng đợi nghẽn có giới hạn tùy chọn được triển khai bằng danh sách liên kết đơn. Khi tạo có thể chỉ định kích thước dung lượng, nếu không chỉ định thì mặc định là `Integer.MAX_VALUE`. Khác với `ArrayBlockingQueue`, nó chỉ hỗ trợ cơ chế truy cập khóa non-fair.
3. `PriorityBlockingQueue`: Hàng đợi nghẽn không giới hạn (unbounded blocking queue) hỗ trợ sắp xếp theo độ ưu tiên. Phần tử bắt buộc phải triển khai interface `Comparable` hoặc truyền đối tượng `Comparator` vào constructor, đồng thời không được chèn phần tử `null`.
4. `SynchronousQueue`: Hàng đợi đồng bộ, là một loại hàng đợi nghẽn không lưu trữ phần tử. Mỗi thao tác chèn bắt buộc phải chờ thao tác xóa tương ứng, ngược lại thao tác xóa cũng bắt buộc phải chờ thao tác chèn. Do đó `SynchronousQueue` thường được sử dụng để truyền dữ liệu trực tiếp giữa các thread.
5. `DelayQueue`: Hàng đợi trì hoãn, các phần tử trong đó chỉ khi đến thời gian trì hoãn (delay time) chỉ định mới có thể xuất hàng đợi.
6. ……

Trong phát triển hàng ngày, các hàng đợi này thực ra ít khi được sử dụng, chỉ cần nắm thông tin tham khảo là được.

### ⭐️ Sự khác biệt giữa ArrayBlockingQueue và LinkedBlockingQueue?

`ArrayBlockingQueue` và `LinkedBlockingQueue` là hai implementation của hàng đợi nghẽn thường dùng trong package concurrent của Java, cả hai đều thread-safe. Tuy nhiên giữa chúng tồn tại những điểm khác biệt dưới đây:

- Triển khai bên dưới: `ArrayBlockingQueue` dựa trên mảng để triển khai, còn `LinkedBlockingQueue` dựa trên danh sách liên kết (linked list) để triển khai.
- Có giới hạn hay không: `ArrayBlockingQueue` là hàng đợi có giới hạn (bounded), bắt buộc phải chỉ định kích thước dung lượng khi tạo. `LinkedBlockingQueue` khi tạo có thể không cần chỉ định dung lượng, mặc định là `Integer.MAX_VALUE` (tức là không giới hạn). Tuy nhiên cũng có thể chỉ định kích thước hàng đợi để trở thành có giới hạn.
- Khóa (lock) có tách biệt hay không: Lock trong `ArrayBlockingQueue` không được tách biệt, tức là sản xuất (produce) và tiêu thụ (consume) dùng chung một lock; Lock trong `LinkedBlockingQueue` được tách biệt, tức là sản xuất dùng `putLock`, tiêu thụ dùng `takeLock`, điều này giúp ngăn chặn tranh chấp lock giữa thread producer và thread consumer.
- Dung lượng bộ nhớ tiêu tốn: `ArrayBlockingQueue` cần phân bổ trước bộ nhớ mảng, trong khi `LinkedBlockingQueue` phân bổ động bộ nhớ cho các node danh sách liên kết. Điều này có nghĩa là `ArrayBlockingQueue` khi tạo ra sẽ chiếm một lượng không gian bộ nhớ nhất định, và thường bộ nhớ xin cấp phát lớn hơn thực tế sử dụng; còn `LinkedBlockingQueue` thì dựa vào sự tăng thêm của các phần tử mà dần dần chiếm thêm bộ nhớ.

## Đọc thêm về Cấu trúc dữ liệu

Phỏng vấn Java Collection thường hay hỏi sâu vào cấu trúc dữ liệu bên dưới. Khuyến nghị kết hợp ôn tập cùng các bài viết dưới đây:

- [Giải thích chi tiết về Cấu trúc dữ liệu tuyến tính](../../cs-basics/data-structure/linear-data-structure.md): Hiểu mối quan hệ giữa mảng, danh sách liên kết, stack, queue và `ArrayList`, `LinkedList`, `ArrayDeque`.
- [Tổng kết câu hỏi phỏng vấn Bảng băm (Hash Table)](../../cs-basics/data-structure/hash-table.md): Hiểu về xung đột hash, mở rộng dung lượng (resize) và tư tưởng bên dưới của `HashMap`.
- [Giải thích chi tiết về Cây đỏ đen (Red-Black Tree)](../../cs-basics/data-structure/red-black-tree.md): Hiểu về `TreeMap`, `TreeSet` cũng như cây đỏ đen liên quan khi `HashMap` biến đổi danh sách liên kết thành cây (treeify).
- [Giải thích chi tiết về Heap](../../cs-basics/data-structure/heap.md): Hiểu về cấu trúc bên dưới của `PriorityQueue` và các dạng bài tập Top K.

<!-- @include: @article-footer.snippet.md -->
