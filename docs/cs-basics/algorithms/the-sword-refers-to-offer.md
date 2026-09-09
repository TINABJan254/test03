---
title: Một số bài toán lập trình tiêu biểu trong Kiếm Chỉ Offer (Coding Interviews)
description: Tuyển chọn các bài toán lập trình phổ biến trong Kiếm Chỉ Offer (Coding Interviews), cung cấp nhiều hướng tiếp cận như đệ quy và lặp kèm ví dụ, giúp ôn tập hiệu quả các dạng bài tần suất cao.
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Kiếm Chỉ Offer,Coding Interviews,Fibonacci,Đệ quy,Lặp,LinkedList,Mảng,Bài toán phỏng vấn
---

# Một số bài toán lập trình tiêu biểu trong Kiếm Chỉ Offer

## Dãy số Fibonacci

**Mô tả bài toán:**

Mọi người đều biết về dãy số Fibonacci, hãy nhập vào một số nguyên n và xuất ra số hạng thứ n của dãy số Fibonacci. Với n <= 39.

**Phân tích bài toán:**

Chắc chắn bài toán này có thể giải quyết bằng phương pháp đệ quy, tuy nhiên đệ quy sẽ gặp một vấn đề rất lớn: Việc tính toán lặp đi lặp lại một lượng lớn bài toán con sẽ dẫn đến tiêu tốn tài nguyên và có thể tràn bộ nhớ/ngăn xếp. Ngoài ra, ta có thể dùng phương pháp lặp (iteration), sử dụng hai biến để lưu kết quả các bước trước và tái sử dụng chúng. Dưới đây là cả hai cách hiện thực:

**Code minh họa:**

Sử dụng phương pháp lặp:

```java
int Fibonacci(int number) {
    if (number <= 0) {
        return 0;
    }
    if (number == 1 || number == 2) {
        return 1;
    }
    int first = 1, second = 1, third = 0;
    for (int i = 3; i <= number; i++) {
        third = first + second;
        first = second;
        second = third;
    }
    return third;
}
```

Sử dụng phương pháp đệ quy:

```java
public int Fibonacci(int n) {
    if (n <= 0) {
        return 0;
    }
    if (n == 1 || n == 2) {
        return 1;
    }

    return Fibonacci(n - 2) + Fibonacci(n - 1);
}
```

## Bài toán nhảy bậc thang (Jump Floor)

**Mô tả bài toán:**

Một con ếch mỗi lần có thể nhảy lên 1 bậc hoặc 2 bậc thang. Hỏi con ếch đó có bao nhiêu cách để nhảy lên một cầu thang có n bậc.

**Phân tích bài toán:**

Phương pháp phân tích chuẩn:

> a. Nếu có hai cách nhảy: 1 bậc hoặc 2 bậc. Giả sử lần đầu tiên nhảy 1 bậc, thì số bậc thang còn lại là n - 1, số cách nhảy tiếp theo là f(n - 1);
> b. Giả sử lần đầu tiên nhảy 2 bậc, thì số bậc thang còn lại là n - 2, số cách nhảy tiếp theo là f(n - 2);
> c. Từ hai giả định a và b, ta suy ra tổng số cách nhảy là: f(n) = f(n - 1) + f(n - 2);
> d. Kết hợp với trường hợp thực tế cơ sở: Khi chỉ có 1 bậc thì f(1) = 1, khi có 2 bậc thì f(2) = 2.

Phương pháp tìm quy luật:

> f(1) = 1, f(2) = 2, f(3) = 3, f(4) = 5... Ta có thể rút ra quy luật f(n) = f(n - 1) + f(n - 2). Tại sao lại có quy luật này? Giả sử cầu thang có 6 bậc, ta có thể từ bậc 5 nhảy 1 bước lên bậc 6, như vậy có bao nhiêu cách nhảy đến bậc 5 thì sẽ có bấy nhiêu cách nhảy tiếp lên bậc 6; ngoài ra ta cũng có thể từ bậc 4 nhảy 2 bước lên bậc 6, có bao nhiêu cách nhảy đến bậc 4 thì cũng có bấy nhiêu cách nhảy lên bậc 6. Không thể nhảy từ bậc 3 hay các bậc khác lên bậc 6 chỉ trong 1 lần nhảy. Do đó tổng số cách nhảy lên bậc 6 là: f(6) = f(5) + f(4).

**Vì vậy, bài toán này thực chất chính là biến thể của dãy số Fibonacci.**

Code chỉ cần sửa đổi nhẹ từ bài Fibonacci ở trên. Điểm khác biệt duy nhất là các giá trị ban đầu là 1, 2, 3, 5, 8... thay vì 1, 1, 2, 3, 5... Vì đệ quy thuần túy có hiệu suất rất thấp nên dưới đây ta sử dụng phương pháp lặp.

**Code minh họa:**

```java
int jumpFloor(int number) {
    if (number <= 0) {
        return 0;
    }
    if (number == 1) {
        return 1;
    }
    if (number == 2) {
        return 2;
    }
    int first = 1, second = 2, third = 0;
    for (int i = 3; i <= number; i++) {
        third = first + second;
        first = second;
        second = third;
    }
    return third;
}
```

## Bài toán nhảy bậc thang biến thái (Jump Floor II)

**Mô tả bài toán:**

Một con ếch mỗi lần có thể nhảy lên 1 bậc, 2 bậc... hoặc nó cũng có thể nhảy một mạch n bậc. Hỏi con ếch có bao nhiêu cách để nhảy lên một cầu thang có n bậc.

**Phân tích bài toán:**

Giả sử `n >= 2`, ở bước đầu tiên có n cách nhảy: Nhảy 1 bậc, nhảy 2 bậc... cho tới nhảy n bậc.
- Nhảy 1 bậc, còn lại n - 1 bậc, số cách là f(n - 1)
- Nhảy 2 bậc, còn lại n - 2 bậc, số cách là f(n - 2)
- ...
- Nhảy n - 1 bậc, còn lại 1 bậc, số cách là f(1)
- Nhảy n bậc, còn lại 0 bậc, số cách là f(0) = 1

Do đó với `n >= 2`:
`f(n) = f(n - 1) + f(n - 2) + ... + f(1) + f(0)`
Vì `f(n - 1) = f(n - 2) + f(n - 3) + ... + f(1) + f(0)`
Nên thay thế vào ta được: `f(n) = 2 * f(n - 1)`.
Mà `f(1) = 1`, do đó suy ra công thức tổng quát: **f(n) = 2^(n - 1)**.

**Code minh họa:**

```java
int JumpFloorII(int number) {
    return 1 << --number; // 2^(number - 1) dùng phép dịch bit, tốc độ nhanh nhất
}
```

**Bổ sung:**

Trong Java có 3 toán tử dịch bit (bitwise shift):

1. `<<`: **Toán tử dịch trái**, tương đương với nhân với `2^n`.
2. `>>`: **Toán tử dịch phải có dấu**, tương đương với chia cho `2^n`.
3. `>>>`: **Toán tử dịch phải không dấu**, bất kể bit dấu ban đầu là 0 hay 1, các bit trống ở bên trái đều được bù bằng 0.

```java
int a = 16;
int b = a << 2; // Dịch trái 2 bit, tương đương 16 * 2^2 = 16 * 4 = 64
int c = a >> 2; // Dịch phải 2 bit, tương đương 16 / 2^2 = 16 / 4 = 4
```

## Tìm kiếm trong mảng hai chiều (Search in a 2D Array)

**Mô tả bài toán:**

Trong một mảng hai chiều, mỗi hàng đều được sắp xếp theo thứ tự tăng dần từ trái sang phải, mỗi cột đều được sắp xếp theo thứ tự tăng dần từ trên xuống dưới. Hãy hoàn thành một hàm: Nhận vào một mảng hai chiều như vậy và một số nguyên mục tiêu, xác định xem trong mảng có chứa số nguyên đó hay không.

**Phân tích bài toán:**

Bài này có một hướng tư duy rất trực quan và đạt hiệu năng tối ưu:

> Ma trận đã được sắp xếp. Nếu nhìn từ góc dưới cùng bên trái: Đi lên trên thì giá trị giảm dần, đi sang phải thì giá trị tăng dần.
> Do đó, bắt đầu tìm kiếm từ góc dưới cùng bên trái:
> - Khi số cần tìm lớn hơn giá trị hiện tại: Dịch sang phải (tăng cột).
> - Khi số cần tìm nhỏ hơn giá trị hiện tại: Dịch lên trên (giảm hàng).
> Tìm kiếm theo cách này có tốc độ nhanh nhất với độ phức tạp `O(m + n)`.

**Code minh họa:**

```java
public boolean Find(int target, int[][] array) {
    // Ý tưởng: Bắt đầu từ góc dưới cùng bên trái
    int row = array.length - 1; // Hàng
    int column = 0;             // Cột
    while (row >= 0 && column < array[0].length) {
        if (array[row][column] > target) {
            row--;
        } else if (array[row][column] < target) {
            column++;
        } else {
            return true;
        }
    }
    return false;
}
```

## Thay thế khoảng trắng

**Mô tả bài toán:**

Hãy hiện thực một hàm để thay thế khoảng trắng trong chuỗi bằng "%20". Ví dụ: Khi chuỗi là `We Are Happy.`, kết quả sau khi thay thế là `We%20Are%20Happy.`.

**Phân tích bài toán:**

Bài này không khó, ta có thể lặp qua từng ký tự và dùng phương thức `append()` để nối chuỗi, hoặc sử dụng trực tiếp hàm `String.replace()`.

**Code minh họa:**

Cách làm tuần tự:

```java
public String replaceSpace(StringBuffer str) {
    StringBuffer out = new StringBuffer();
    for (int i = 0; i < str.length(); i++) {
        char b = str.charAt(i);
        if (b == ' ') {
            out.append("%20");
        } else {
            out.append(b);
        }
    }
    return out.toString();
}
```

Dùng một dòng lệnh:

```java
public String replaceSpace(StringBuffer str) {
    return str.toString().replace(" ", "%20");
}
```

## Lũy thừa nguyên của một số (Power / Fast Exponentiation)

**Mô tả bài toán:**

Cho một số thực `base` kiểu double và một số nguyên `exponent` kiểu int, hãy tính `base` lũy thừa `exponent` (`base^exponent`).

**Phân tích bài toán:**

Bài này có thể áp dụng thuật toán **Lũy thừa nhanh (Fast Exponentiation / Binary Exponentiation)**. Cần đặc biệt chú ý xử lý hai điều kiện biên:
1. Khi cơ số `base == 0.0` và số mũ là số âm thì không thể chia cho 0 (không lấy nghịch đảo được).
2. Khi `exponent = Integer.MIN_VALUE`, việc đổi dấu trực tiếp `-exponent` sẽ bị tràn số nguyên, do đó cần ép kiểu số mũ sang `long` trước.

Thuật toán lũy thừa nhanh chia đôi số mũ ở mỗi vòng lặp: Khi bit hiện tại của số mũ là 1, nhân cơ số hiện tại vào kết quả; sau đó bình phương cơ số và dịch phải số mũ 1 bit. Độ phức tạp thời gian là `O(log n)`.

**Code minh họa:**

```java
public class Solution {
    public double Power(double base, int exponent) {
        if (base == 0.0 && exponent < 0) {
            throw new ArithmeticException("zero cannot be raised to a negative exponent");
        }

        long exp = exponent;
        if (exp < 0) {
            base = 1.0 / base;
            exp = -exp;
        }

        double result = 1.0;
        while (exp > 0) {
            if ((exp & 1L) != 0) {
                result *= base;
            }
            base *= base;
            exp >>= 1;
        }
        return result;
    }
}
```

## Điều chỉnh thứ tự mảng: Số lẻ đứng trước số chẵn

**Mô tả bài toán:**

Cho một mảng số nguyên, hãy viết hàm điều chỉnh thứ tự các số trong mảng sao cho tất cả các số lẻ nằm ở nửa đầu, tất cả các số chẵn nằm ở nửa sau, và đảm bảo thứ tự tương đối giữa các số lẻ với nhau cũng như các số chẵn với nhau không bị thay đổi (tính ổn định - Stable).

**Phân tích bài toán:**

Đầu tiên ta đếm số lượng số lẻ trong mảng (gọi là `oddCount`). Sau đó tạo một mảng mới cùng độ dài, duyệt qua mảng gốc: Nếu là số lẻ thì thêm từ đầu mảng mới (bắt đầu từ chỉ số 0); nếu là số chẵn thì thêm từ vị trí `oddCount` trở đi.

**Code minh họa:**

Thuật toán với độ phức tạp thời gian `O(n)` và không gian `O(n)`:

```java
public class Solution {
    public void reOrderArray(int[] array) {
        if (array.length == 0 || array.length == 1)
            return;
        int oddCount = 0, oddBegin = 0;
        int[] newArray = new int[array.length];
        // Đếm số lượng số lẻ
        for (int i = 0; i < array.length; i++) {
            if ((array[i] & 1) == 1) oddCount++;
        }
        for (int i = 0; i < array.length; i++) {
            // Số lẻ ghi từ đầu (oddBegin), số chẵn ghi từ oddCount
            if ((array[i] & 1) == 1)
                newArray[oddBegin++] = array[i];
            else
                newArray[oddCount++] = array[i];
        }
        for (int i = 0; i < array.length; i++) {
            array[i] = newArray[i];
        }
    }
}
```

## Node thứ k tính từ cuối LinkedList

**Mô tả bài toán:**

Cho một LinkedList, hãy xuất ra node thứ k tính từ cuối LinkedList lên.

**Phân tích bài toán:**

Dùng hai con trỏ: Con trỏ `p1` chạy trước, sau khi `p1` đi được `k - 1` bước thì con trỏ `p2` mới bắt đầu di chuyển. Khi `p1` đi đến node cuối cùng của danh sách thì `p2` chính là node thứ k tính từ cuối lên.

**Code minh họa:**

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/

// Độ phức tạp thời gian O(n), một lần duyệt
public class Solution {
    public ListNode FindKthToTail(ListNode head, int k) {
        ListNode pre = null, p = null;
        p = head;
        pre = head;
        int a = k;
        int count = 0;
        while (p != null) {
            p = p.next;
            count++;
            if (k < 1) {
                pre = pre.next;
            }
            k--;
        }
        if (count < a) return null;
        return pre;
    }
}
```

## Đảo ngược LinkedList (Reverse Linked List)

**Mô tả bài toán:**

Cho một LinkedList, hãy đảo ngược danh sách và xuất ra tất cả các phần tử.

**Phân tích bài toán:**

Căn cứ vào đặc điểm của LinkedList, mỗi node trỏ tới node tiếp theo. Ta dùng biến tạm `next` để lưu node kế tiếp, sau đó đổi chiều trỏ của node hiện tại ngược về node đứng trước (`pre`), rồi tịnh tiến các con trỏ.

![Quá trình hoán đổi chiều trỏ của các node khi đảo ngược LinkedList](https://oss.javaguide.cn/p3-juejin/844773c7300e4373922bb1a6ae2a55a3~tplv-k3u1fbpfcp-zoom-1.png)

**Code minh họa:**

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode ReverseList(ListNode head) {
        ListNode next = null;
        ListNode pre = null;
        while (head != null) {
            next = head.next;
            head.next = pre;
            pre = head;
            head = next;
        }
        return pre;
    }
}
```

## Hợp nhất hai LinkedList đã sắp xếp

**Mô tả bài toán:**

Cho hai LinkedList đơn tăng dần, hãy hợp nhất hai danh sách này thành một LinkedList mới thỏa mãn thứ tự không giảm.

**Code minh họa:**

Phiên bản lặp (Iterative):

```java
public class Solution {
    public ListNode Merge(ListNode list1, ListNode list2) {
        if (list1 == null) return list2;
        if (list2 == null) return list1;
        ListNode mergeHead = null;
        ListNode current = null;
        while (list1 != null && list2 != null) {
            if (list1.val <= list2.val) {
                if (mergeHead == null) {
                    mergeHead = current = list1;
                } else {
                    current.next = list1;
                    current = list1;
                }
                list1 = list1.next;
            } else {
                if (mergeHead == null) {
                    mergeHead = current = list2;
                } else {
                    current.next = list2;
                    current = list2;
                }
                list2 = list2.next;
            }
        }
        if (list1 == null) {
            current.next = list2;
        } else {
            current.next = list1;
        }
        return mergeHead;
    }
}
```

Phiên bản đệ quy (Recursive):

```java
public ListNode Merge(ListNode list1, ListNode list2) {
    if (list1 == null) return list2;
    if (list2 == null) return list1;
    if (list1.val <= list2.val) {
        list1.next = Merge(list1.next, list2);
        return list1;
    } else {
        list2.next = Merge(list1, list2.next);
        return list2;
    }
}
```

## Dùng hai Stack để hiện thực một Queue

**Mô tả bài toán:**

Dùng hai Stack để hiện thực một Queue, hoàn thành hai thao tác `push` và `pop`. Các phần tử trong Queue là số nguyên kiểu int.

**Phân tích bài toán:**

Đặc điểm cơ bản:
- **Stack:** Vào sau ra trước (LIFO)
- **Queue:** Vào trước ra trước (FIFO)

Khi `push`: Ta luôn đẩy phần tử vào `stack1`.
Khi `pop`: Nếu `stack2` đang rỗng, ta lần lượt lấy toàn bộ phần tử từ `stack1` sang `stack2` (đảo ngược thứ tự), sau đó thực hiện `pop` trên `stack2`. Nếu `stack2` đã có sẵn phần tử thì cứ trực tiếp `pop` từ `stack2`.

![Một số phương thức thông dụng của class Stack](https://oss.javaguide.cn/github/javaguide/cs-basics/algorithms/5985000.jpg)

**Code minh họa:**

```java
import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();

    public void push(int node) {
        stack1.push(node);
    }

    public int pop() {
        if (stack1.empty() && stack2.empty()) {
            throw new RuntimeException("Queue is empty!");
        }
        if (stack2.empty()) {
            while (!stack1.empty()) {
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }
}
```

## Thứ tự đẩy vào và lấy ra của Stack

**Mô tả bài toán:**

Cho hai chuỗi số nguyên: Chuỗi thứ nhất biểu thị thứ tự đẩy vào (push) của Stack, hãy xác định xem chuỗi thứ hai có thể là thứ tự lấy ra (pop) tương ứng của Stack hay không. Giả sử tất cả các số được đẩy vào Stack đều không trùng nhau. Ví dụ chuỗi `1, 2, 3, 4, 5` là thứ tự đẩy vào, thì chuỗi `4, 5, 3, 2, 1` là một chuỗi lấy ra hợp lệ, nhưng chuỗi `4, 3, 5, 1, 2` không thể là chuỗi lấy ra hợp lệ.

**Phân tích bài toán:**

Sử dụng một Stack phụ trợ: Duyệt qua thứ tự đẩy vào, lần lượt đẩy các phần tử vào Stack phụ trợ. Sau mỗi lần đẩy một phần tử, kiểm tra xem đỉnh Stack có bằng với phần tử hiện tại của chuỗi lấy ra hay không. Nếu bằng nhau thì lấy phần tử ra khỏi Stack (`pop`) và tăng chỉ số của chuỗi lấy ra lên 1, tiếp tục lặp lại kiểm tra cho đến khi đỉnh Stack không còn bằng nữa. Sau khi duyệt hết chuỗi đẩy vào, nếu Stack phụ trợ rỗng hoàn toàn thì chứng minh chuỗi lấy ra là hợp lệ.

**Code minh họa:**

```java
import java.util.Stack;

public class Solution {
    public boolean IsPopOrder(int[] pushA, int[] popA) {
        if (pushA.length == 0 || popA.length == 0)
            return false;
        Stack<Integer> s = new Stack<Integer>();
        int popIndex = 0;
        for (int i = 0; i < pushA.length; i++) {
            s.push(pushA[i]);
            while (!s.empty() && s.peek() == popA[popIndex]) {
                s.pop();
                popIndex++;
            }
        }
        return s.empty();
    }
}
```

<!-- @include: @article-footer.snippet.md -->
