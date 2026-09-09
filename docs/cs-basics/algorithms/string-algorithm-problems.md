---
title: Một số bài toán thuật toán chuỗi phổ biến
description: Tổng hợp các thuật toán và dạng bài chuỗi tần suất cao, tập trung giải thích nguyên lý KMP/BM, Sliding Window, giúp người đọc nắm vững so khớp hiệu quả và hiện thực chuẩn xác.
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Thuật toán chuỗi,KMP,BM,Sliding Window,Chuỗi con,So khớp chuỗi,Độ phức tạp
---

> Tác giả: wwwxmu
>
> Địa chỉ bài viết gốc: <https://www.weiweiblog.cn/13string/>

## 1. Thuật toán KMP

Nhắc đến các bài toán xử lý chuỗi, chắc chắn không thể bỏ qua thuật toán KMP (Knuth-Morris-Pratt). Thuật toán này dùng để giải quyết bài toán tìm kiếm chuỗi con (String Matching), có thể tìm kiếm vị trí xuất hiện của một chuỗi mẫu (pattern W) bên trong một chuỗi văn bản (text S). Thuật toán KMP thu hẹp độ phức tạp thời gian so khớp ký tự xuống còn `O(m + n)`, trong khi độ phức tạp không gian chỉ là `O(m)`. Phương pháp tìm kiếm vét cạn (brute-force) thông thường sẽ liên tục quay lui (backtrack) con trỏ trên chuỗi chính, dẫn đến hiệu suất rất thấp; trong khi đó thuật toán KMP tận dụng thông tin đã so khớp từng phần trước đó để giữ cho con trỏ trên chuỗi chính không bao giờ phải quay lui, chỉ cần điều chỉnh con trỏ trên chuỗi mẫu để dịch chuyển mẫu tới vị trí khớp tiềm năng tiếp theo một cách hiệu quả nhất.

Chi tiết về thuật toán, bạn có thể tham khảo thêm:

- [Hiểu cặn kẽ thuật toán KMP từ đầu đến cuối (Blog CSDN)](https://blog.csdn.net/v_july_v/article/details/7041827)
- [Làm sao để hiểu và nắm vững thuật toán KMP tốt hơn? (Zhihu)](https://www.zhihu.com/question/21923021)
- [Phân tích chi tiết thuật toán KMP](https://blog.sengxian.com/algorithms/kmp)

**Bên cạnh đó, hãy cùng tìm hiểu thêm về thuật toán BM (Boyer-Moore)!**

> Thuật toán BM cũng là một thuật toán so khớp chuỗi chính xác rất nổi tiếng. Nó so sánh các ký tự theo chiều từ phải sang trái, đồng thời áp dụng hai quy tắc heuristic: quy tắc ký tự xấu (bad character rule) và quy tắc hậu tố tốt (good suffix rule) để quyết định khoảng cách nhảy sang phải. Ý tưởng cơ bản là so khớp ký tự từ phải sang trái, khi gặp ký tự không khớp thì tra cứu trong bảng ký tự xấu và bảng hậu tố tốt để tìm giá trị dịch chuyển sang phải lớn nhất, sau đó dịch chuyển chuỗi mẫu sang phải và tiếp tục so khớp.

## 2. Thay thế khoảng trắng

> Kiếm Chỉ Offer: Hãy hiện thực một hàm để thay thế mọi ký tự khoảng trắng trong chuỗi bằng "%20". Ví dụ: Khi chuỗi là `We Are Happy.`, chuỗi sau khi thay thế sẽ là `We%20Are%20Happy.`.

Ở đây có hai phương pháp tiếp cận: ① Phương pháp thông thường duyệt mảng; ② Sử dụng trực tiếp API.

```java
public class Solution {

  /**
   * Phương pháp 1: Duyệt thông thường. Sử dụng String.charAt(i) duyệt qua chuỗi
   * và kiểm tra xem ký tự có phải khoảng trắng hay không. Nếu có thì nối "%20", ngược lại nối chính ký tự đó.
   */
  public static String replaceSpace(StringBuffer str) {
    int length = str.length();
    StringBuffer result = new StringBuffer();
    for (int i = 0; i < length; i++) {
      char b = str.charAt(i);
      if (b == ' ') {
        result.append("%20");
      } else {
        result.append(b);
      }
    }
    return result.toString();
  }

  /**
   * Phương pháp 2: Sử dụng API thay thế toàn bộ khoảng trắng, chỉ một dòng code.
   */
  public static String replaceSpace2(StringBuffer str) {
    return str.toString().replace(" ", "%20");
  }
}
```

Đối với trường hợp thay thế ký tự cố định (như khoảng trắng), phương pháp 2 sử dụng phương thức `replace` có sẵn trong Java, vừa tiện lợi vừa cho hiệu năng rất tốt!

```java
str.toString().replace(" ", "%20");
```

## 3. Tiền tố chung dài nhất (Longest Common Prefix)

> LeetCode: Viết một hàm để tìm chuỗi tiền tố chung dài nhất trong một mảng chuỗi. Nếu không tồn tại tiền tố chung, hãy trả về chuỗi rỗng `""`.

Ví dụ 1:

```plain
Đầu vào: ["flower","flow","flight"]
Đầu ra: "fl"
```

Ví dụ 2:

```plain
Đầu vào: ["dog","racecar","car"]
Đầu ra: ""
Giải thích: Không tồn tại tiền tố chung giữa các chuỗi.
```

Ý tưởng rất đơn giản! Trước tiên dùng `Arrays.sort(strs)` để sắp xếp mảng chuỗi theo thứ tự từ điển, sau đó chỉ cần so sánh từng ký tự từ đầu đến cuối giữa phần tử đầu tiên và phần tử cuối cùng của mảng!

```java
import java.util.Arrays;

public class Main {
  public static String longestCommonPrefix(String[] strs) {
    // Nếu kiểm tra mảng không hợp lệ thì trả về chuỗi rỗng
    if (!checkStrs(strs)) {
      return "";
    }
    int len = strs.length;
    StringBuilder res = new StringBuilder();
    // Sắp xếp mảng chuỗi theo thứ tự tăng dần
    Arrays.sort(strs);
    int m = strs[0].length();
    int n = strs[len - 1].length();
    int num = Math.min(m, n);
    for (int i = 0; i < num; i++) {
      if (strs[0].charAt(i) == strs[len - 1].charAt(i)) {
        res.append(strs[0].charAt(i));
      } else {
        break;
      }
    }
    return res.toString();
  }

  private static boolean checkStrs(String[] strs) {
    if (strs == null || strs.length == 0) {
      return false;
    }
    for (String s : strs) {
      if (s == null || s.length() == 0) {
        return false;
      }
    }
    return true;
  }

  // Kiểm thử
  public static void main(String[] args) {
    String[] strs = {"customer", "car", "cat"};
    System.out.println(Main.longestCommonPrefix(strs)); // "c"
  }
}
```

## 4. Chuỗi đối xứng (Palindrome)

### 4.1. Chuỗi đối xứng dài nhất có thể tạo (Longest Palindrome)

> LeetCode: Cho một chuỗi chứa các chữ cái in hoa và in thường, hãy tìm độ dài của chuỗi đối xứng dài nhất có thể tạo thành từ các chữ cái đó. Trong quá trình tạo, hãy chú ý phân biệt chữ hoa và chữ thường. Ví dụ `"Aa"` không được tính là chuỗi đối xứng. Chú ý: Giả định độ dài chuỗi không vượt quá 1010.
>
> Chuỗi đối xứng (Palindrome): Là một chuỗi đọc xuôi hay đọc ngược đều hoàn toàn giống nhau, chẳng hạn như "level" hay "noon".

Ví dụ 1:

```plain
Đầu vào: "abccccdd"
Đầu ra: 7
Giải thích: Một trong những chuỗi đối xứng dài nhất có thể tạo ra là "dccaccd", có độ dài là 7.
```

Điều kiện để các ký tự ghép thành chuỗi đối xứng:

- Các ký tự có số lần xuất hiện chẵn có thể phân bổ đều về hai bên.
- Có thể có tối đa **một ký tự lẻ duy nhất** đặt ở chính giữa chuỗi.

Ta đếm số lần xuất hiện của các ký tự. Duyệt mảng ký tự, dùng `HashSet` để lưu trữ: Nếu ký tự chưa có trong set thì thêm vào; nếu đã có sẵn thì tăng `count++` (nghĩa là tìm được một cặp ký tự) rồi xóa ký tự đó khỏi set. Kết quả nếu set không rỗng (còn ký tự đơn lẻ), ta cộng thêm 1 vào độ dài.

```java
import java.util.HashSet;

class Solution {
  public int longestPalindrome(String s) {
    if (s.length() == 0)
      return 0;
    HashSet<Character> hashset = new HashSet<>();
    char[] chars = s.toCharArray();
    int count = 0;
    for (int i = 0; i < chars.length; i++) {
      if (!hashset.contains(chars[i])) {
        hashset.add(chars[i]);
      } else {
        hashset.remove(chars[i]);
        count++;
      }
    }
    return hashset.isEmpty() ? count * 2 : count * 2 + 1;
  }
}
```

### 4.2. Kiểm tra chuỗi đối xứng (Valid Palindrome)

> LeetCode: Cho một chuỗi, hãy xác thực xem nó có phải là chuỗi đối xứng hay không, chỉ xét các ký tự chữ và số, bỏ qua sự phân biệt chữ hoa và chữ thường. Chú thích: Chuỗi rỗng được định nghĩa là chuỗi đối xứng hợp lệ.

Ví dụ 1:

```plain
Đầu vào: "A man, a plan, a canal: Panama"
Đầu ra: true
```

Ví dụ 2:

```plain
Đầu vào: "race a car"
Đầu ra: false
```

```java
class Solution {
  public boolean isPalindrome(String s) {
    if (s.length() == 0)
      return true;
    int l = 0, r = s.length() - 1;
    while (l < r) {
      // Bỏ qua ký tự không phải chữ và số từ hai đầu
      if (!Character.isLetterOrDigit(s.charAt(l))) {
        l++;
      } else if (!Character.isLetterOrDigit(s.charAt(r))) {
        r--;
      } else {
        // So sánh hai ký tự sau khi chuyển về chữ thường
        if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r)))
          return false;
        l++;
        r--;
      }
    }
    return true;
  }
}
```

### 4.3. Chuỗi con đối xứng dài nhất (Longest Palindromic Substring)

> LeetCode: Cho một chuỗi `s`, hãy tìm chuỗi con đối xứng dài nhất trong `s`. Có thể giả định độ dài tối đa của `s` là 1000.

Ví dụ 1:

```plain
Đầu vào: "babad"
Đầu ra: "bab"
Lưu ý: "aba" cũng là một đáp án hợp lệ.
```

Ví dụ 2:

```plain
Đầu vào: "cbbd"
Đầu ra: "bb"
```

Phương pháp mở rộng từ tâm (Center Expansion): Chọn từng vị trí làm tâm đối xứng, lần lượt tính toán độ dài đối xứng tối đa cho trường hợp tâm lẻ (đối xứng qua 1 ký tự) và tâm chẵn (đối xứng qua 2 ký tự kề nhau).

```java
class Solution {
  private int index, len;

  public String longestPalindrome(String s) {
    if (s.length() < 2)
      return s;
    for (int i = 0; i < s.length() - 1; i++) {
      palindromeHelper(s, i, i);     // Tâm lẻ
      palindromeHelper(s, i, i + 1); // Tâm chẵn
    }
    return s.substring(index, index + len);
  }

  public void palindromeHelper(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) {
      l--;
      r++;
    }
    if (len < r - l - 1) {
      index = l + 1;
      len = r - l - 1;
    }
  }
}
```

### 4.4. Dãy con đối xứng dài nhất (Longest Palindromic Subsequence)

> LeetCode: Cho một chuỗi `s`, hãy tìm dãy con đối xứng dài nhất trong `s`. Có thể giả định độ dài tối đa của `s` là 1000.
>
> **Điểm khác biệt cốt lõi giữa Dãy con đối xứng (Subsequence) và Chuỗi con đối xứng (Substring) ở bài trước là: Chuỗi con là một đoạn ký tự liên tục, trong khi dãy con chỉ cần giữ nguyên thứ tự tương đối của các ký tự mà không yêu cầu phải liên tục kề nhau. Ví dụ: "bbbb" là dãy con của "bbbab" nhưng không phải là chuỗi con.**

Ví dụ 1:

```plain
Đầu vào: "bbbab"
Đầu ra: 4
Giải thích: Một dãy con đối xứng dài nhất có thể là "bbbb".
```

Ví dụ 2:

```plain
Đầu vào: "cbbd"
Đầu ra: 2
Giải thích: Một dãy con đối xứng dài nhất là "bb".
```

**Quy hoạch động (DP):**
- Nếu `s.charAt(i) == s.charAt(j)`: `dp[i][j] = dp[i + 1][j - 1] + 2`
- Ngược lại: `dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1])`

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        int len = s.length();
        int[][] dp = new int[len][len];
        for (int i = len - 1; i >= 0; i--) {
            dp[i][i] = 1;
            for (int j = i + 1; j < len; j++) {
                if (s.charAt(i) == s.charAt(j))
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                else
                    dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
            }
        }
        return dp[0][len - 1];
    }
}
```

## 5. Độ sâu lồng nhau của dấu ngoặc

> Bài toán tuyển dụng mùa thu Java:
> Một chuỗi ngoặc hợp lệ được định nghĩa như sau:
>
> 1. Chuỗi rỗng `""` là chuỗi ngoặc hợp lệ.
> 2. Nếu "X" và "Y" là các chuỗi ngoặc hợp lệ, thì "XY" cũng là chuỗi ngoặc hợp lệ.
> 3. Nếu "X" là chuỗi ngoặc hợp lệ, thì "(X)" cũng là chuỗi ngoặc hợp lệ.
>
> Độ sâu của một chuỗi ngoặc hợp lệ được định nghĩa như sau:
>
> 1. Chuỗi rỗng `""` có độ sâu là 0.
> 2. Nếu độ sâu của "X" là x, độ sâu của "Y" là y, thì độ sâu của "XY" là `max(x, y)`.
> 3. Nếu độ sâu của "X" là x, thì độ sâu của "(X)" là `x + 1`.
>
> Ví dụ: Độ sâu của "()()()" là 1, độ sâu của "((()))" là 3. Hãy tính độ sâu của chuỗi ngoặc hợp lệ được cho.

```plain
Mô tả đầu vào:
Đầu vào gồm một chuỗi ngoặc hợp lệ s, độ dài length (2 <= length <= 50), chuỗi chỉ chứa '(' và ')'.

Mô tả đầu ra:
Xuất ra một số nguyên dương đại diện cho độ sâu của chuỗi ngoặc đó.
```

Ví dụ:

```plain
Đầu vào: (())
Đầu ra: 2
```

Đoạn code như sau:

```java
import java.util.Scanner;

public class Main {
  public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    String s = sc.nextLine();
    int cnt = 0, max = 0;
    for (int i = 0; i < s.length(); ++i) {
      if (s.charAt(i) == '(')
        cnt++;
      else
        cnt--;
      max = Math.max(max, cnt);
    }
    sc.close();
    System.out.println(max);
  }
}
```

## 6. Chuyển đổi chuỗi thành số nguyên (String to Integer)

> Kiếm Chỉ Offer: Hãy chuyển đổi một chuỗi thành một số nguyên (tương tự như hàm `Integer.valueOf(string)`), nhưng khi chuỗi không đáp ứng quy chuẩn số thì trả về 0. Yêu cầu không được sử dụng các hàm thư viện có sẵn để chuyển đổi chuỗi thành số nguyên. Nếu giá trị là 0 hoặc chuỗi không phải là một biểu diễn số hợp lệ thì trả về 0.

```java
public class Main {

  public static int StrToInt(String str) {
    if (str.length() == 0)
      return 0;
    char[] chars = str.toCharArray();
    // Kiểm tra có dấu âm dương hay không
    int flag = 0;
    if (chars[0] == '+')
      flag = 1;
    else if (chars[0] == '-')
      flag = 2;
    int start = flag > 0 ? 1 : 0;
    int res = 0; // Lưu kết quả
    for (int i = start; i < chars.length; i++) {
      if (Character.isDigit(chars[i])) {
        int temp = chars[i] - '0';
        res = res * 10 + temp;
      } else {
        return 0;
      }
    }
    return flag != 2 ? res : -res;
  }

  public static void main(String[] args) {
    String s = "-12312312";
    System.out.println("Dùng hàm thư viện: " + Integer.valueOf(s));
    int res = Main.StrToInt(s);
    System.out.println("Dùng hàm tự viết: " + res);
  }
}
```

## Trọng tâm ôn tập phỏng vấn

Các bài toán chuỗi nhìn bề ngoài rất phong phú đa dạng, nhưng thực tế các template phổ biến không quá nhiều: Đếm bằng Hash/Array, Two Pointers, Sliding Window, KMP, Palindrome, Mô phỏng bằng Stack.

| Dạng bài | Phương pháp phổ biến | Bài toán tiêu biểu |
| ---------- | -------------------- | -------------------------------- |
| Đếm ký tự | Mảng hoặc HashMap | Valid Anagram, Group Anagrams |
| Bài toán chuỗi con (Substring) | Sliding Window | Longest Substring Without Repeating Characters, Minimum Window Substring |
| Bài toán đối xứng (Palindrome) | Two Pointers, Center Expansion, DP | Valid Palindrome, Longest Palindromic Substring |
| Khớp chuỗi (Pattern Matching) | KMP, Rolling Hash | Implement `strStr()` |
| Ngoặc và mã hóa | Stack | Valid Parentheses, Decode String |
| Chuyển đổi định dạng số | Mô phỏng (Simulation) | String to Integer (atoi) |

Khi giải bài toán chuỗi, hãy tự trả lời 3 câu hỏi trước:

1. Đề bài quan tâm đến chuỗi con (substring - liên tục) hay dãy con (subsequence - không cần liên tục)?
2. Phạm vi tập ký tự lớn cỡ nào? Nếu chỉ toàn chữ cái thường tiếng Anh (`a-z`), dùng mảng kích thước 26 sẽ đơn giản và nhanh hơn nhiều so với `HashMap`.
3. Có cần xử lý tràn số (overflow), chuỗi rỗng, khoảng trắng, hoặc dấu `+`/`-` ở các trường hợp biên không?

Một số lỗi thường gặp cần lưu ý:

- Trong Java, `String` là bất biến (immutable), việc nối chuỗi liên tục trong vòng lặp nên ưu tiên dùng `StringBuilder`.
- Kiểu `char` trong một số trường hợp không đủ để biểu diễn trọn vẹn ký tự Unicode đặc biệt; tuy nhiên các bài phỏng vấn thông thường chủ yếu khảo sát trên bảng mã ASCII hoặc chữ cái latin thường.
- Chuỗi con đối xứng và Dãy con đối xứng là hai dạng bài hoàn toàn khác nhau: Dạng trước thường dùng Center Expansion, dạng sau thường dùng Dynamic Programming.
- Trong phỏng vấn, nhà tuyển dụng thông thường không bắt bạn phải tính tay từng giá trị mảng `next` của KMP, nhưng bạn cần hiểu bản chất của nó là để bỏ qua các tiền tố đã khớp, tránh việc so khớp lại từ đầu.

<!-- @include: @article-footer.snippet.md -->
