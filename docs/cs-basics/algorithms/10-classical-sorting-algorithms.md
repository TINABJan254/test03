---
title: "Tổng hợp 10 thuật toán sắp xếp kinh điển"
description: "Hệ thống hóa 10 thuật toán sắp xếp kinh điển, đính kèm so sánh độ phức tạp và tính ổn định, bao gồm nguyên lý cốt lõi và kịch bản hiện thực của sắp xếp so sánh và phi so sánh, giúp lựa chọn và tối ưu nhanh chóng."
category: Cơ sở máy tính
tag:
  - Thuật toán
head:
  - - meta
    - name: keywords
      content: Thuật toán sắp xếp,Quick Sort,Merge Sort,Heap Sort,Bubble Sort,Selection Sort,Insertion Sort,Shell Sort,Bucket Sort,Counting Sort,Radix Sort,Độ phức tạp thời gian,Độ phức tạp không gian,Tính ổn định
---

<!-- markdownlint-disable MD024 -->

## Dẫn nhập

Sắp xếp là thao tác sắp đặt một chuỗi các bản ghi theo thứ tự tăng dần hoặc giảm dần dựa trên giá trị của một hoặc một số từ khóa (keys). Thuật toán sắp xếp là phương pháp thực hiện việc sắp xếp các bản ghi đó theo đúng yêu cầu đề ra. Thuật toán sắp xếp nhận được sự quan tâm rất lớn trong nhiều lĩnh vực khoa học máy tính, đặc biệt là trong xử lý khối lượng dữ liệu lớn, nơi một thuật toán ưu tú có thể tiết kiệm một lượng khổng lồ tài nguyên tính toán.

## Giới thiệu tổng quan

### Bảng tổng hợp các thuật toán sắp xếp

Các thuật toán sắp xếp nội bộ (Internal Sorting) phổ biến bao gồm: **Insertion Sort (Sắp xếp chèn)**, **Shell Sort (Sắp xếp Shell)**, **Selection Sort (Sắp xếp chọn)**, **Bubble Sort (Sắp xếp nổi bọt)**, **Merge Sort (Sắp xếp trộn)**, **Quick Sort (Sắp xếp nhanh)**, **Heap Sort (Sắp xếp vun đống)**, **Counting Sort (Sắp xếp đếm)**, **Bucket Sort (Sắp xếp theo thùng)**, **Radix Sort (Sắp xếp cơ số)**. Bảng dưới đây tóm tắt đầy đủ các đặc tính cốt lõi:

| Thuật toán sắp xếp | Độ phức tạp thời gian (Trung bình) | Độ phức tạp thời gian (Xấu nhất) | Độ phức tạp thời gian (Tốt nhất) | Độ phức tạp không gian | In-place (Tại chỗ) | Tính ổn định (Stability) |
| -------- | ------------------ | ------------------ | ------------------ | ----------------------- | -------- | -------------- |
| Bubble Sort (Sắp xếp nổi bọt) | O(n^2) | O(n^2) | O(n) | O(1) | Có | Ổn định |
| Selection Sort (Sắp xếp chọn) | O(n^2) | O(n^2) | O(n^2) | O(1) | Có | Không ổn định |
| Insertion Sort (Sắp xếp chèn) | O(n^2) | O(n^2) | O(n) | O(1) | Có | Ổn định |
| Shell Sort (Sắp xếp Shell) | Phụ thuộc chuỗi khoảng cách | O(n^2) | O(n log n) | O(1) | Có | Không ổn định |
| Merge Sort (Sắp xếp trộn) | O(n log n) | O(n log n) | O(n log n) | O(n) | Không | Ổn định |
| Quick Sort (Sắp xếp nhanh) | O(n log n) | O(n^2) | O(n log n) | Trung bình O(log n), xấu nhất O(n) | Có | Không ổn định |
| Heap Sort (Sắp xếp vun đống) | O(n log n) | O(n log n) | O(n log n) | O(1) | Có | Không ổn định |
| Counting Sort (Sắp xếp đếm) | O(n + k) | O(n + k) | O(n + k) | O(n + k) | Không | Ổn định |
| Bucket Sort (Sắp xếp theo thùng) | Phụ thuộc phân phối dữ liệu | Phụ thuộc sắp xếp trong thùng | O(n + k) | O(n + k) | Không | Phụ thuộc sắp xếp trong thùng |
| Radix Sort (Sắp xếp cơ số) | O(d(n + r)) | O(d(n + r)) | O(d(n + r)) | O(n + r) | Không | Ổn định |

**Giải thích thuật ngữ**:

- **n**: Quy mô dữ liệu, biểu thị số lượng phần tử cần sắp xếp.
- **k**: Phạm vi giá trị đếm hoặc số lượng thùng, ý nghĩa cụ thể tùy thuộc vào từng thuật toán.
- **d**: Số chữ số tối đa cần xử lý trong Radix Sort.
- **r**: Cơ số sử dụng trong Radix Sort, ví dụ với hệ thập phân thì `r = 10`.
- **Sắp xếp nội bộ (Internal Sort)**: Toàn bộ dữ liệu cần sắp xếp có thể nạp trọn vẹn vào bộ nhớ RAM, các thao tác sắp xếp diễn ra hoàn toàn trong bộ nhớ.
- **Sắp xếp ngoài (External Sort)**: Khi lượng dữ liệu quá lớn không thể chứa hết trong bộ nhớ, phải nhờ đến sự hỗ trợ của các thiết bị lưu trữ ngoài (như ổ đĩa) để phân chia và xử lý theo từng đợt.
- **Tính ổn định (Stability)**: Nếu phần tử A ban đầu đứng trước B và có giá trị $A = B$, sau khi sắp xếp nếu A vẫn đảm bảo đứng trước B thì thuật toán đó gọi là **ổn định (Stable)**; ngược lại nếu vị trí tương đối có thể bị đảo lộn thì gọi là **không ổn định (Unstable)**.
- **Độ phức tạp thời gian**: Mô tả định tính lượng thời gian thuật toán tiêu tốn theo quy mô dữ liệu.
- **Độ phức tạp không gian**: Mô tả định tính lượng bộ nhớ bổ sung mà thuật toán yêu cầu trong quá trình thực thi.

### Phân loại thuật toán sắp xếp

10 thuật toán sắp xếp phổ biến được chia làm hai nhóm lớn: **Sắp xếp so sánh (Comparison Sort)** và **Sắp xếp phi so sánh (Non-comparison Sort)**.

![Phân loại thuật toán sắp xếp](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/sort2.png)

Các thuật toán như **Quick Sort**, **Merge Sort**, **Heap Sort**, **Bubble Sort**... đều thuộc nhóm **Sắp xếp so sánh**. Chúng xác định thứ tự tương đối giữa các phần tử thông qua phép so sánh trực tiếp. Theo lý thuyết mô hình so sánh, các thuật toán sắp xếp so sánh tổng quát trong trường hợp xấu nhất cần tối thiểu `Ω(n log n)` phép so sánh. Bubble Sort cần nhiều lượt quét nên thời gian trung bình là `O(n^2)`; Merge Sort và Quick Sort vận dụng chia để trị để đạt thời gian trung bình `O(n log n)`.

Ưu thế lớn nhất của sắp xếp so sánh là tính vạn năng: Áp dụng được cho mọi loại dữ liệu và không phụ thuộc vào phân phối dữ liệu.

Trong khi đó, **Counting Sort**, **Radix Sort**, **Bucket Sort** thuộc nhóm **Sắp xếp phi so sánh**. Chúng tận dụng các thông tin bổ sung như phạm vi giá trị, quy luật phân phối hoặc số lượng chữ số để vượt qua cận dưới so sánh, đạt được độ phức tạp thời gian tuyến tính. Tuy nhiên, chúng đòi hỏi tiêu tốn thêm không gian bộ nhớ và có yêu cầu khắt khe về phạm vi dữ liệu.

## 1. Sắp xếp nổi bọt (Bubble Sort)

Bubble Sort là thuật toán sắp xếp đơn giản. Nó liên tục duyệt qua dãy cần sắp xếp, so sánh từng cặp phần tử kề nhau, nếu sai thứ tự thì hoán đổi vị trí của chúng. Quá trình này lặp lại cho đến khi không còn cặp phần tử nào cần hoán đổi nữa, tức là mảng đã hoàn toàn có thứ tự. Thuật toán có tên gọi "nổi bọt" vì các phần tử nhỏ hơn/lớn hơn sẽ từ từ "nổi" lên đỉnh mảng giống như bọt khí dâng lên mặt nước.

### Các bước thuật toán

1. So sánh hai phần tử liền kề. Nếu phần tử trước lớn hơn phần tử sau, hoán đổi vị trí của chúng;
2. Thực hiện thao tác tương tự cho từng cặp phần tử liền kề tiếp theo, từ đầu đến cuối dãy. Sau lượt đầu tiên, phần tử lớn nhất sẽ nằm ở vị trí cuối cùng;
3. Lặp lại các bước trên cho tất cả các phần tử còn lại, trừ phần tử cuối cùng đã vào đúng vị trí;
4. Tiếp tục lặp lại các bước 1~3 cho đến khi dãy được sắp xếp hoàn tất.

### Minh họa thuật toán

![Minh họa Bubble Sort](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/bubble_sort.gif)

### Code hiện thực

```java
/**
 * Bubble Sort
 * @param arr
 * @return arr
 */
public static int[] bubbleSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        // Đặt cờ flag, nếu không có hoán đổi nào diễn ra trong vòng lặp
        // nghĩa là dãy đã có thứ tự hoàn tất, có thể kết thúc sớm.
        boolean flag = true;
        for (int j = 0; j < arr.length - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int tmp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = tmp;
                // Đổi cờ flag
                flag = false;
            }
        }
        if (flag) {
            break;
        }
    }
    return arr;
}
```

**Ở đây ta tối ưu một chi tiết nhỏ bằng cách thêm cờ `flag`: Khi mảng đầu vào đã được sắp xếp sẵn, thuật toán dừng ngay ở lượt đầu tiên và đạt độ phức tạp thời gian tốt nhất là `O(n)`.**

### Phân tích thuật toán

- **Tính ổn định**: Ổn định (Stable)
- **Độ phức tạp thời gian**: Tốt nhất: $O(n)$, Xấu nhất: $O(n^2)$, Trung bình: $O(n^2)$
- **Độ phức tạp không gian**: $O(1)$
- **Phương thức sắp xếp**: In-place (Tại chỗ)

## 2. Sắp xếp chọn (Selection Sort)

Selection Sort là thuật toán sắp xếp đơn giản và trực quan. Dù dữ liệu đầu vào như thế nào thì độ phức tạp thời gian của nó luôn là $O(n^2)$. Do đó, chỉ nên sử dụng khi quy mô dữ liệu rất nhỏ. Điểm cộng duy nhất là nó không tiêu tốn thêm không gian bộ nhớ. Nguyên lý hoạt động: Trước tiên tìm phần tử nhỏ nhất (hoặc lớn nhất) trong mảng chưa sắp xếp, đưa về vị trí đầu tiên của mảng; sau đó tiếp tục tìm phần tử nhỏ nhất trong phần còn lại và đặt vào vị trí tiếp theo. Lặp lại như vậy cho đến khi tất cả các phần tử được sắp xếp xong.

### Các bước thuật toán

1. Tìm phần tử nhỏ nhất trong dãy chưa sắp xếp, đưa về vị trí bắt đầu;
2. Tiếp tục tìm phần tử nhỏ nhất trong phần chưa sắp xếp còn lại, đặt vào cuối phần đã sắp xếp;
3. Lặp lại bước 2 cho đến khi toàn bộ các phần tử được sắp xếp hoàn tất.

### Minh họa thuật toán

![Mỗi lượt Selection Sort chọn phần tử nhỏ nhất đặt về cuối vùng đã sắp xếp](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/selection_sort.gif)

### Code hiện thực

```java
/**
 * Selection Sort
 * @param arr
 * @return arr
 */
public static int[] selectionSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        if (minIndex != i) {
            int tmp = arr[i];
            arr[i] = arr[minIndex];
            arr[minIndex] = tmp;
        }
    }
    return arr;
}
```

### Phân tích thuật toán

- **Tính ổn định**: Không ổn định (Unstable - vì phép tráo đổi xa có thể làm nhảy vọt qua phần tử bằng nhau đứng trước)
- **Độ phức tạp thời gian**: Tốt nhất: $O(n^2)$, Xấu nhất: $O(n^2)$, Trung bình: $O(n^2)$
- **Độ phức tạp không gian**: $O(1)$
- **Phương thức sắp xếp**: In-place

## 3. Sắp xếp chèn (Insertion Sort)

Insertion Sort hoạt động bằng cách xây dựng một dãy có thứ tự dần dần. Với mỗi phần tử chưa sắp xếp, ta quét ngược từ sau ra trước trong đoạn đã sắp xếp, tìm vị trí thích hợp và chèn phần tử đó vào. Trong quá trình quét ngược, các phần tử lớn hơn phần tử cần chèn sẽ được tịnh tiến lùi về sau một vị trí để dọn chỗ trống. Nguyên lý này rất giống cách chúng ta cầm và xếp bài Tây trên tay khi chơi bài.

### Các bước thuật toán

1. Xem phần tử đầu tiên là dãy đã có thứ tự;
2. Lấy phần tử tiếp theo, quét ngược từ sau ra trước trong dãy đã có thứ tự;
3. Nếu phần tử trong dãy lớn hơn phần tử mới, dịch phần tử đó sang phải một vị trí;
4. Lặp lại bước 3 cho đến khi tìm được vị trí mà phần tử trong dãy nhỏ hơn hoặc bằng phần tử mới;
5. Chèn phần tử mới vào vị trí đó;
6. Lặp lại các bước 2~5 cho đến hết mảng.

### Minh họa thuật toán

![Quá trình Insertion Sort](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/insertion_sort.gif)

### Code hiện thực

```java
/**
 * Insertion Sort
 * @param arr
 * @return arr
 */
public static int[] insertionSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        int preIndex = i - 1;
        int current = arr[i];
        while (preIndex >= 0 && current < arr[preIndex]) {
            arr[preIndex + 1] = arr[preIndex];
            preIndex -= 1;
        }
        arr[preIndex + 1] = current;
    }
    return arr;
}
```

### Phân tích thuật toán

- **Tính ổn định**: Ổn định (Stable)
- **Độ phức tạp thời gian**: Tốt nhất: $O(n)$ (khi mảng đã có thứ tự sẵn), Xấu nhất: $O(n^2)$, Trung bình: $O(n^2)$
- **Độ phức tạp không gian**: $O(1)$
- **Phương thức sắp xếp**: In-place

## 4. Sắp xếp Shell (Shell Sort)

Shell Sort do Donald Shell đề xuất vào năm 1959. Đây là phiên bản cải tiến hiệu năng cao hơn của Insertion Sort, còn được gọi là thuật toán sắp xếp với khoảng cách giảm dần (Diminishing Increment Sort). Hiệu năng của Shell Sort phụ thuộc rất lớn vào chuỗi khoảng cách (increment sequence). Chuỗi khoảng cách nguyên bản của Shell trong trường hợp xấu nhất vẫn là $O(n^2)$, nhưng các chuỗi khoảng cách tối ưu hơn về sau có thể đạt cận trên dưới bậc hai.

Ý tưởng cơ bản: Chia toàn bộ mảng cần sắp xếp thành nhiều dãy con cách đều nhau bởi một khoảng cách (gap), thực hiện Insertion Sort trên từng dãy con đó. Khi mảng đã "cơ bản có thứ tự", thu hẹp dần gap về 1 để thực hiện một lượt Insertion Sort cuối cùng.

### Các bước thuật toán

Ở đây ta chọn chuỗi khoảng cách Shell thông dụng: $gap = n / 2$, sau mỗi lượt giảm nửa $gap = gap / 2$, cho đến khi $gap = 1$.

1. Chọn khoảng cách ban đầu $gap = length / 2$;
2. Chia mảng thành các dãy con gồm các phần tử cách nhau đúng một khoảng $gap$, áp dụng Insertion Sort cho từng dãy con;
3. Giảm khoảng cách: $gap = gap / 2$;
4. Lặp lại bước 2 và 3 cho đến khi $gap = 1$, thực hiện lượt sắp xếp chèn cuối cùng để hoàn tất.

### Minh họa thuật toán

![Quy trình Shell Sort phân nhóm theo khoảng cách và sắp xếp chèn](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/shell_sort.png)

### Code hiện thực

```java
/**
 * Shell Sort
 *
 * @param arr
 * @return arr
 */
public static int[] shellSort(int[] arr) {
    int n = arr.length;
    int gap = n / 2;
    while (gap > 0) {
        for (int i = gap; i < n; i++) {
            int current = arr[i];
            int preIndex = i - gap;
            // Insertion Sort trên các phần tử cách nhau một khoảng gap
            while (preIndex >= 0 && arr[preIndex] > current) {
                arr[preIndex + gap] = arr[preIndex];
                preIndex -= gap;
            }
            arr[preIndex + gap] = current;
        }
        gap /= 2;
    }
    return arr;
}
```

### Phân tích thuật toán

- **Tính ổn định**: Không ổn định (Unstable - vì các phần tử bằng nhau ở các nhóm con khác nhau có thể bị đổi thứ tự tương đối)
- **Độ phức tạp thời gian**: Tốt nhất: $O(n \log n)$, Xấu nhất: $O(n^2)$, Trung bình phụ thuộc vào chuỗi khoảng cách
- **Độ phức tạp không gian**: $O(1)$

## 5. Sắp xếp trộn (Merge Sort)

Merge Sort là thuật toán sắp xếp áp dụng tư duy Chia để trị (Divide and Conquer) vô cùng điển hình. Merge Sort là thuật toán ổn định, chia mảng thành các mảng con, sắp xếp đệ quy các mảng con đó rồi hợp nhất lại để thu được mảng hoàn chỉnh có thứ tự.

Hiệu năng của Merge Sort hoàn toàn không bị ảnh hưởng bởi hình thái dữ liệu đầu vào: Luôn luôn giữ vững độ phức tạp thời gian $O(n \log n)$ trong mọi tình huống, cái giá phải trả là cần thêm $O(n)$ không gian bộ nhớ phụ trợ.

### Các bước thuật toán

Merge Sort là một quy trình đệ quy với điều kiện dừng là khi mảng con chỉ còn 1 phần tử:

1. Nếu mảng con chỉ có 1 phần tử, trả về ngay; ngược lại chia mảng độ dài $n$ thành hai mảng con có độ dài $n/2$;
2. Đệ quy thực hiện Merge Sort trên hai mảng con;
3. Thiết lập hai con trỏ trỏ vào đầu hai mảng con đã có thứ tự;
4. So sánh hai phần tử được trỏ tới, chọn phần tử nhỏ hơn đưa vào mảng kết quả tạm thời và dịch con trỏ tương ứng sang phải;
5. Lặp lại bước 4 cho đến khi một trong hai con trỏ đi đến cuối mảng con;
6. Sao chép toàn bộ các phần tử còn lại của mảng con kia vào cuối mảng kết quả.

### Minh họa thuật toán

![Merge Sort chia nhỏ mảng và hợp nhất các mảng con](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/merge_sort.gif)

### Code hiện thực

```java
import java.util.Arrays;

public static int[] mergeSort(int[] arr) {
    if (arr.length <= 1) {
        return arr;
    }
    int middle = arr.length / 2;
    int[] arr_1 = Arrays.copyOfRange(arr, 0, middle);
    int[] arr_2 = Arrays.copyOfRange(arr, middle, arr.length);
    return merge(mergeSort(arr_1), mergeSort(arr_2));
}

/**
 * Hợp nhất hai mảng đã sắp xếp
 */
public static int[] merge(int[] arr_1, int[] arr_2) {
    int[] sorted_arr = new int[arr_1.length + arr_2.length];
    int idx = 0, idx_1 = 0, idx_2 = 0;
    while (idx_1 < arr_1.length && idx_2 < arr_2.length) {
        if (arr_1[idx_1] <= arr_2[idx_2]) {
            sorted_arr[idx] = arr_1[idx_1];
            idx_1 += 1;
        } else {
            sorted_arr[idx] = arr_2[idx_2];
            idx_2 += 1;
        }
        idx += 1;
    }
    while (idx_1 < arr_1.length) {
        sorted_arr[idx] = arr_1[idx_1];
        idx_1 += 1;
        idx += 1;
    }
    while (idx_2 < arr_2.length) {
        sorted_arr[idx] = arr_2[idx_2];
        idx_2 += 1;
        idx += 1;
    }
    return sorted_arr;
}
```

### Phân tích thuật toán

- **Tính ổn định**: Ổn định (Stable)
- **Độ phức tạp thời gian**: Tốt nhất, Xấu nhất, Trung bình đều là $O(n \log n)$
- **Độ phức tạp không gian**: $O(n)$

## 6. Sắp xếp nhanh (Quick Sort)

Quick Sort cũng vận dụng tư duy Chia để trị (Divide and Conquer). Khác với Merge Sort (chia đôi mảng đơn thuần trước rồi mới so sánh trong bước gộp), Quick Sort phân chia mảng thành hai nửa: một nửa gồm các phần tử nhỏ hơn một mốc chốt (pivot), nửa còn lại gồm các phần tử lớn hơn pivot. Nhờ đó, sau khi đệ quy sắp xếp hai nửa xong, ta không cần thêm bước so sánh hợp nhất nữa. Tuy nhiên, tính chất phụ thuộc vào việc chọn pivot khiến độ phức tạp thời gian của Quick Sort có sự dao động.

### Các bước thuật toán

1. **Chọn chốt (Pivot)**: Chọn một phần tử trong mảng làm pivot (để tránh trường hợp xấu nhất, nên chọn ngẫu nhiên);
2. **Phân vùng (Partition)**: Sắp xếp lại dãy sao cho tất cả phần tử nhỏ hơn pivot nằm bên trái pivot, tất cả phần tử lớn hơn nằm bên phải pivot;
3. **Đệ quy (Recurse)**: Đệ quy áp dụng Quick Sort cho hai mảng con bên trái và bên phải của pivot.

**Về hiệu năng:**

- **Trường hợp trung bình và tốt nhất:** Độ phức tạp thời gian là $O(n \log n)$, đạt được khi pivot chia mảng thành hai nửa tương đối đồng đều.
- **Trường hợp xấu nhất:** Thời gian thoái hóa về $O(n^2)$, xảy ra khi mỗi lần chọn pivot đều rơi đúng vào phần tử nhỏ nhất hoặc lớn nhất của mảng (ví dụ mảng đã có thứ tự sẵn mà luôn chọn phần tử đầu tiên làm pivot). Do đó, **việc chọn pivot ngẫu nhiên** là cực kỳ quan trọng.

### Minh họa thuật toán

![Quick Sort ngẫu nhiên chọn pivot và phân vùng đệ quy](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/random_quick_sort.gif)

### Code hiện thực

```java
import java.util.concurrent.ThreadLocalRandom;

class Solution {
    public int[] sortArray(int[] a) {
        quick(a, 0, a.length - 1);
        return a;
    }

    // Hàm đệ quy cốt lõi của Quick Sort
    void quick(int[] a, int left, int right) {
        if (left >= right) { // Điều kiện dừng: khoảng có 0 hoặc 1 phần tử
            return;
        }
        int p = partition(a, left, right); // Phân vùng và trả về chỉ số pivot
        quick(a, left, p - 1);             // Đệ quy mảng con bên trái
        quick(a, p + 1, right);            // Đệ quy mảng con bên phải
    }

    // Hàm phân vùng: chia mảng thành hai phần (nhỏ hơn pivot bên trái, lớn hơn bên phải)
    int partition(int[] a, int left, int right) {
        // Chọn ngẫu nhiên pivot để tránh trường hợp xấu nhất
        int idx = ThreadLocalRandom.current().nextInt(right - left + 1) + left;
        swap(a, left, idx); // Đưa pivot về đầu khoảng
        int pv = a[left];   // Giá trị pivot
        int i = left + 1;   // Con trỏ trái
        int j = right;      // Con trỏ phải

        while (i <= j) {
            while (i <= j && a[i] < pv) {
                i++;
            }
            while (i <= j && a[j] > pv) {
                j--;
            }
            if (i <= j) {
                swap(a, i, j);
                i++;
                j--;
            }
        }
        // Đặt pivot vào đúng vị trí phân vùng
        swap(a, j, left);
        return j;
    }

    void swap(int[] a, int i, int j) {
        int t = a[i];
        a[i] = a[j];
        a[j] = t;
    }
}
```

### Phân tích thuật toán

- **Tính ổn định**: Không ổn định (Unstable)
- **Độ phức tạp thời gian**: Tốt nhất: $O(n \log n)$, Xấu nhất: $O(n^2)$, Trung bình: $O(n \log n)$
- **Độ phức tạp không gian**: Trung bình $O(\log n)$, Xấu nhất $O(n)$ (không gian ngăn xếp đệ quy)

## 7. Sắp xếp vun đống (Heap Sort)

Heap Sort là thuật toán sắp xếp dựa trên cấu trúc dữ liệu Heap. Heap là một cây nhị phân gần như hoàn chỉnh thỏa mãn **tính chất đống**: Giá trị của node con luôn nhỏ hơn (hoặc lớn hơn) node cha của nó.

### Các bước thuật toán

1. Xây dựng mảng ban đầu thành một **Max-Heap (Đống cực đại)**;
2. Đổi chỗ phần tử đỉnh đống (phần tử lớn nhất) với phần tử cuối cùng của mảng, lúc này phần tử lớn nhất đã nằm ở vùng có thứ tự;
3. Giảm kích thước đống đi 1, tiến hành điều chỉnh (heapify) đỉnh đống mới để khôi phục lại tính chất Max-Heap;
4. Lặp lại bước 2 và 3 cho đến khi kích thước đống chỉ còn 1 phần tử.

### Minh họa thuật toán

![Heap Sort xây dựng Max-Heap và lần lượt rút phần tử đỉnh đống](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/heap_sort.gif)

### Code hiện thực

```java
public class HeapSort {
    static int heapLen;

    private static void swap(int[] arr, int i, int j) {
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }

    // Xây dựng Max-Heap ban đầu
    private static void buildMaxHeap(int[] arr) {
        for (int i = arr.length / 2 - 1; i >= 0; i--) {
            heapify(arr, i);
        }
    }

    // Vun đống / Điều chỉnh đống tại vị trí i
    private static void heapify(int[] arr, int i) {
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        int largest = i;
        if (right < heapLen && arr[right] > arr[largest]) {
            largest = right;
        }
        if (left < heapLen && arr[left] > arr[largest]) {
            largest = left;
        }
        if (largest != i) {
            swap(arr, largest, i);
            heapify(arr, largest);
        }
    }

    public static int[] heapSort(int[] arr) {
        heapLen = arr.length;
        buildMaxHeap(arr);
        for (int i = arr.length - 1; i > 0; i--) {
            // Đưa phần tử lớn nhất ở đỉnh về cuối
            swap(arr, 0, i);
            heapLen -= 1;
            heapify(arr, 0);
        }
        return arr;
    }
}
```

### Phân tích thuật toán

- **Tính ổn định**: Không ổn định (Unstable)
- **Độ phức tạp thời gian**: Tốt nhất, Xấu nhất, Trung bình đều là $O(n \log n)$
- **Độ phức tạp không gian**: $O(1)$

## 8. Sắp xếp đếm (Counting Sort)

Counting Sort là thuật toán sắp xếp phi so sánh có độ phức tạp tuyến tính. Cốt lõi của Counting Sort là chuyển giá trị của phần tử đầu vào thành chỉ số (index) của một mảng phụ trợ `C` để đếm tần suất xuất hiện. **Counting Sort yêu cầu dữ liệu đầu vào bắt buộc phải là các số nguyên nằm trong một phạm vi xác định**.

### Các bước thuật toán

1. Tìm giá trị lớn nhất `max` và nhỏ nhất `min` trong mảng;
2. Khởi tạo mảng đếm `C` có kích thước `max - min + 1` với toàn bộ giá trị bằng 0;
3. Duyệt mảng gốc, dùng `A[i] - min` làm chỉ số, tăng giá trị đếm `C[A[i] - min]++`;
4. Biến đổi mảng `C` thành mảng tiền tố tổng: `C[i] = C[i] + C[i - 1]`;
5. Tạo mảng kết quả `R` có cùng độ dài với mảng gốc;
6. **Duyệt ngược từ cuối về đầu** mảng gốc `A`, tra cứu vị trí chính xác trong mảng `C`, đặt phần tử vào `R` và giảm số đếm tương ứng trong `C` đi 1 (duyệt ngược để đảm bảo tính ổn định).

### Minh họa thuật toán

![Counting Sort thống kê số lần xuất hiện để xác định vị trí](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/counting_sort.gif)

### Code hiện thực

```java
private static int[] getMinAndMax(int[] arr) {
    int maxValue = arr[0];
    int minValue = arr[0];
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] > maxValue) {
            maxValue = arr[i];
        } else if (arr[i] < minValue) {
            minValue = arr[i];
        }
    }
    return new int[] {minValue, maxValue};
}

public static int[] countingSort(int[] arr) {
    if (arr.length < 2) {
        return arr;
    }
    int[] extremum = getMinAndMax(arr);
    int minValue = extremum[0];
    int maxValue = extremum[1];
    int[] countArr = new int[maxValue - minValue + 1];
    int[] result = new int[arr.length];

    for (int i = 0; i < arr.length; i++) {
        countArr[arr[i] - minValue] += 1;
    }
    for (int i = 1; i < countArr.length; i++) {
        countArr[i] += countArr[i - 1];
    }
    for (int i = arr.length - 1; i >= 0; i--) {
        int idx = countArr[arr[i] - minValue] - 1;
        result[idx] = arr[i];
        countArr[arr[i] - minValue] -= 1;
    }
    return result;
}
```

### Phân tích thuật toán

- **Tính ổn định**: Ổn định (Stable)
- **Độ phức tạp thời gian**: $O(n + k)$ trong mọi trường hợp (với `k` là độ dài phạm vi `max - min + 1`)
- **Độ phức tạp không gian**: $O(n + k)$

## 9. Sắp xếp theo thùng (Bucket Sort)

Bucket Sort là phiên bản nâng cấp của Counting Sort. Nó sử dụng một hàm ánh xạ để phân chia dữ liệu đầu vào vào các thùng (buckets) có thứ tự. Sau đó, mỗi thùng được sắp xếp riêng lẻ (bằng cách gọi đệ quy Bucket Sort hoặc dùng thuật toán sắp xếp khác như Insertion Sort), cuối cùng ghép các thùng lại với nhau.

Để Bucket Sort đạt hiệu quả tối đa:
1. Trong điều kiện bộ nhớ cho phép, số lượng thùng nên càng lớn càng tốt;
2. Hàm ánh xạ phải phân phối đều $N$ phần tử đầu vào vào $K$ thùng.

### Các bước thuật toán

1. Thiết lập `bucket_size` (kích thước dải giá trị của mỗi thùng);
2. Duyệt qua dữ liệu đầu vào, ánh xạ từng phần tử vào thùng tương ứng;
3. Sắp xếp các phần tử bên trong từng thùng không rỗng;
4. Nối tuần tự các phần tử từ các thùng đã có thứ tự để thu được mảng kết quả cuối cùng.

### Minh họa thuật toán

![Bucket Sort phân phối dữ liệu vào các thùng rồi sắp xếp và ghép lại](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/bucket_sort.gif)

### Code hiện thực

```java
import java.util.ArrayList;
import java.util.List;

public static List<Integer> bucketSort(List<Integer> arr, int bucket_size) {
    if (bucket_size <= 0) {
        throw new IllegalArgumentException("bucket_size must be positive");
    }
    if (arr.size() < 2) {
        return arr;
    }
    int minValue = arr.get(0);
    int maxValue = arr.get(0);
    for (int i : arr) {
        if (i > maxValue) maxValue = i;
        else if (i < minValue) minValue = i;
    }
    int bucket_cnt = (maxValue - minValue) / bucket_size + 1;
    List<List<Integer>> buckets = new ArrayList<>();
    for (int i = 0; i < bucket_cnt; i++) {
        buckets.add(new ArrayList<Integer>());
    }
    for (int element : arr) {
        int idx = (element - minValue) / bucket_size;
        buckets.get(idx).add(element);
    }
    for (int i = 0; i < buckets.size(); i++) {
        if (buckets.get(i).size() > 1) {
            buckets.get(i).sort(Integer::compareTo);
        }
    }
    ArrayList<Integer> result = new ArrayList<>();
    for (List<Integer> bucket : buckets) {
        for (int element : bucket) {
            result.add(element);
        }
    }
    return result;
}
```

### Phân tích thuật toán

- **Tính ổn định**: Phụ thuộc vào thuật toán sắp xếp bên trong từng thùng (nếu dùng hàm sort ổn định thì tổng thể là ổn định)
- **Độ phức tạp thời gian**: Tốt nhất $O(n + k)$; khi dữ liệu phân phối đều, kỳ vọng đạt xấp xỉ $O(n + k)$; xấu nhất có thể lên tới $O(n \log n + k)$ hoặc $O(n^2)$ nếu tất cả dồn vào 1 thùng
- **Độ phức tạp không gian**: $O(n + k)$

## 10. Sắp xếp cơ số (Radix Sort)

Radix Sort cũng là thuật toán sắp xếp phi so sánh, sắp xếp các phần tử bằng cách xét từng chữ số từ vị trí có trọng số thấp nhất (LSD - Least Significant Digit) lên vị trí có trọng số cao nhất (MSD). Giả sử mảng có độ dài $n$, số chữ số tối đa là $d$, cơ số là $r$, độ phức tạp thời gian là $O(d(n + r))$.

Nguyên lý cơ bản: Trước tiên sắp xếp theo chữ số hàng đơn vị và thu gom lại; sau đó sắp xếp theo chữ số hàng chục và thu gom lại; cứ thế tiếp tục cho đến chữ số cao nhất. Vì ở mỗi lượt đều sử dụng sắp xếp đếm ổn định (phân phối riêng, thu gom riêng) nên thứ tự tương đối trước đó vẫn được bảo toàn trọn vẹn.

### Các bước thuật toán

1. Tìm giá trị lớn nhất trong mảng để xác định số lượng chữ số tối đa $N$ (ví dụ số lớn nhất là 1000 thì $N = 4$);
2. Bắt đầu từ chữ số thấp nhất (hàng đơn vị), phân phối các phần tử vào 10 thùng (với hệ thập phân $r = 10$);
3. Thu gom tuần tự các phần tử từ các thùng trả lại mảng;
4. Lặp lại bước 2 và 3 cho các hàng chục, hàng trăm... cho đến khi hết $N$ chữ số.

### Minh họa thuật toán

![Radix Sort phân phối và thu gom theo từng chữ số](https://oss.javaguide.cn/github/javaguide/cs-basics/sorting-algorithms/radix_sort.gif)

### Code hiện thực

```java
import java.util.ArrayList;
import java.util.List;

public static int[] radixSort(int[] arr) {
    if (arr.length < 2) {
        return arr;
    }
    for (int element : arr) {
        if (element < 0) {
            throw new IllegalArgumentException("radixSort only supports non-negative integers");
        }
    }
    int N = 1;
    int maxValue = arr[0];
    for (int element : arr) {
        if (element > maxValue) {
            maxValue = element;
        }
    }
    while (maxValue / 10 != 0) {
        maxValue = maxValue / 10;
        N += 1;
    }
    for (int i = 0; i < N; i++) {
        List<List<Integer>> radix = new ArrayList<>();
        for (int k = 0; k < 10; k++) {
            radix.add(new ArrayList<Integer>());
        }
        for (int element : arr) {
            int idx = (element / (int) Math.pow(10, i)) % 10;
            radix.get(idx).add(element);
        }
        int idx = 0;
        for (List<Integer> l : radix) {
            for (int n : l) {
                arr[idx++] = n;
            }
        }
    }
    return arr;
}
```

### Phân tích thuật toán

- **Tính ổn định**: Ổn định (Stable)
- **Độ phức tạp thời gian**: Tốt nhất, Xấu nhất, Trung bình đều là $O(d(n + r))$
- **Độ phức tạp không gian**: $O(n + r)$

**So sánh Radix Sort vs Counting Sort vs Bucket Sort:**

Cả 3 thuật toán này đều sử dụng khái niệm "thùng", nhưng cơ chế hoàn toàn khác biệt:
- **Radix Sort**: Phân chia thùng dựa trên từng chữ số của giá trị.
- **Counting Sort**: Mỗi thùng chỉ tương ứng với một giá trị duy nhất.
- **Bucket Sort**: Mỗi thùng lưu trữ một khoảng dải giá trị nhất định.

## Tài liệu tham khảo

- [Tổng hợp thuật toán sắp xếp (Nguồn tham khảo chính)](https://www.cnblogs.com/guoyaohua/p/8600214.html)
- <https://en.wikipedia.org/wiki/Sorting_algorithm>
- <https://sort.hust.cc/>

## Trọng tâm ôn tập phỏng vấn

Trong phỏng vấn, người phỏng vấn thông thường sẽ không bắt bạn viết tay toàn bộ 10 thuật toán sắp xếp, nhưng bạn bắt buộc phải trình bày rõ ràng: Độ phức tạp thời gian, không gian, tính ổn định, sắp xếp tại chỗ (in-place) và ngữ cảnh áp dụng phù hợp của từng thuật toán.

| Thuật toán sắp xếp | Độ phức tạp thời gian (Trung bình) | Độ phức tạp thời gian (Xấu nhất) | Độ phức tạp không gian | Tính ổn định | In-place (Tại chỗ) |
| -------- | -------------- | -------------- | --------------------------- | -------------- | -------- |
| Bubble Sort (Nổi bọt) | `O(n^2)` | `O(n^2)` | `O(1)` | Ổn định | Có |
| Selection Sort (Chọn) | `O(n^2)` | `O(n^2)` | `O(1)` | Không ổn định | Có |
| Insertion Sort (Chèn) | `O(n^2)` | `O(n^2)` | `O(1)` | Ổn định | Có |
| Merge Sort (Trộn) | `O(n log n)` | `O(n log n)` | `O(n)` | Ổn định | Không |
| Quick Sort (Nhanh) | `O(n log n)` | `O(n^2)` | Trung bình `O(log n)`, xấu nhất `O(n)` | Không ổn định | Có |
| Heap Sort (Vun đống) | `O(n log n)` | `O(n log n)` | `O(1)` | Không ổn định | Có |
| Counting Sort (Đếm) | `O(n + k)` | `O(n + k)` | `O(n + k)` | Ổn định | Không |
| Bucket Sort (Theo thùng) | Phụ thuộc phân phối | Phụ thuộc sort trong thùng | `O(n + k)` | Phụ thuộc sort trong thùng | Không |
| Radix Sort (Cơ số) | `O(d(n + r))` | `O(d(n + r))` | `O(n + r)` | Ổn định | Không |

Một số câu hỏi mở rộng có tần suất xuất hiện rất cao:

- **Tại sao Quick Sort trong trường hợp xấu nhất là `O(n^2)`? Làm sao để giảm thiểu xác suất thoái hóa?** Khi pivot chọn phải giá trị cực tiểu hoặc cực đại liên tục. Có thể giảm thiểu bằng cách chọn pivot ngẫu nhiên (Randomized) hoặc lấy trung vị 3 số (Median-of-three).
- **Tại sao Merge Sort lại ổn định?** Vì trong quá trình hợp nhất hai mảng con, khi gặp hai phần tử có giá trị bằng nhau, ta luôn ưu tiên lấy phần tử ở mảng con bên trái trước.
- **Tại sao Heap Sort lại không ổn định?** Vì quá trình vun đống và hoán đổi phần tử đỉnh với phần tử cuối cùng sẽ làm xáo trộn thứ tự ban đầu của các phần tử có giá trị bằng nhau.
- **Khi nào Insertion Sort hoạt động hiệu quả nhất?** Khi mảng dữ liệu đã gần như có thứ tự sẵn hoặc quy mô dữ liệu nhỏ.
- **Tại sao Counting Sort, Bucket Sort, Radix Sort không phải là thuật toán sắp xếp vạn năng?** Vì chúng bị ràng buộc bởi phạm vi giá trị, quy luật phân phối hoặc số chữ số của dữ liệu.

## Java Code Template

Trong phỏng vấn, hai thuật toán thường được yêu cầu viết tay trực tiếp nhất là **Quick Sort** và **Merge Sort**. Với Quick Sort, cần đặc biệt lưu ý xử lý phân vùng biên:

```java
void quickSort(int[] nums, int left, int right) {
    if (left >= right) {
        return;
    }
    int pivotIndex = partition(nums, left, right);
    quickSort(nums, left, pivotIndex - 1);
    quickSort(nums, pivotIndex + 1, right);
}

int partition(int[] nums, int left, int right) {
    int pivot = nums[right];
    int less = left;
    for (int i = left; i < right; i++) {
        if (nums[i] <= pivot) {
            swap(nums, less, i);
            less++;
        }
    }
    swap(nums, less, right);
    return less;
}

void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

Nếu lo ngại mảng đã có thứ tự khiến Quick Sort bị thoái hóa, có thể chọn ngẫu nhiên pivot trước khi phân vùng và hoán đổi nó về vị trí `right`:

```java
int randomIndex = left + new Random().nextInt(right - left + 1);
swap(nums, randomIndex, right);
```

## Minh họa quy trình và các trường hợp biên

Một lượt phân vùng của Quick Sort có thể hình dung như sau:

```text
Khoảng mảng gốc: [left ... right]
pivot: Chọn nums[right]
less: Trỏ vào vị trí kế tiếp của "vùng <= pivot"
i: Quét từ left đến right - 1

Sau khi quét xong:
[left ... less - 1] <= pivot
[less ... right - 1] > pivot
Đổi pivot về vị trí less, sau đó đệ quy hai nửa bên trái và bên phải của pivot
```

Các trường hợp biên cần rà soát trước khi viết code:

- Mảng rỗng hoặc chỉ có 1 phần tử: Trả về trực tiếp.
- Mảng đã sắp xếp xuôi hoặc ngược: Chọn cố định phần tử đầu/cuối dễ gây thoái hóa.
- Mảng chứa lượng lớn phần tử trùng lặp: Phân vùng 2 chiều thông thường có thể chưa tối ưu, có thể tìm hiểu thêm 3-way Quick Sort (Quick Sort 3 đường).
- Khi người phỏng vấn hỏi về tính ổn định, tuyệt đối không được trả lời Quick Sort là ổn định.

<!-- @include: @article-footer.snippet.md -->
