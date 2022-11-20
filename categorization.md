# Tạo nhóm từ biến liên tục

```r
library(knitr)
opts_chunk$set(
    message = FALSE
)
```

# Mục đích

Trong nghiên cứu, bạn thường thu thập một số biến liên tục, sau đó dựa vào các điểm cắt để phân thành các nhóm. Chẳng hạn, chúng ta thường phân loại chỉ số khối cơ thể (BMI) thành các nhóm nhẹ cân (BMI < 18.5), bình thường (18.5 - <25), thừa cân (25 - <30), và béo phì (>30). Chúng ta không nên thu thập ngay phân loại BMI, mà nên thu thập các chỉ số chiều cao và cân nặng, sau đó sử dụng phần mềm để tính ra BMI và phân nhóm.

Chúng ta sẽ cùng xem một bộ số liệu như sau.


```r
library(dplyr)

set.seed(0)
n <- 10

d <- data.frame(
    id = seq(n),
    sex = sample(c(1, 2), n, replace = TRUE),   # 1=Nam 2=Nữ
    bmi = runif(n, 16.5, 35)
)

d %>% kable()
```



| id| sex|      bmi|
|--:|---:|--------:|
|  1|   2| 17.64305|
|  2|   1| 20.31053|
|  3|   2| 19.76630|
|  4|   1| 29.20992|
|  5|   1| 23.60592|
|  6|   2| 30.74207|
|  7|   1| 25.70744|
|  8|   1| 29.77594|
|  9|   1| 34.85026|
| 10|   2| 23.53065|


# Phân nhóm từ một biến liên tục

Để phân loại các nhóm BMI, chúng ta chỉ cần sử dụng một biến BMI là đủ. Đặc điểm của các nhóm phân loại từ BMI là mỗi cá thể chỉ được phân loại vào đúng một nhóm. Thông thường, với cách phân loại này, chúng ta sẽ mã hóa các nhóm tăng dần từ giá trị 1 (ví dụ, 1 đến 4 cho 4 nhóm BMI).

Bạn có thể làm rất nhanh việc phân nhóm này trong R bằng việc sử dụng hàm `cut()`. Cung cấp cho hàm này một đối số là các khoảng giá trị điểm cắt (bao gồm cả giá trị thấp nhất và cao nhất), hàm sẽ trả về cho bạn một factor của phân nhóm tạo ra từ biến liên tục. Các đối số khác trong hàm `cut()` bạn tự tham khảo trong phần documentation của R nhé (gõ `?cut` trong R console và ấn Enter).


```r
d$bmi %>% cut(c(0, 18.5, 25, 30, 100),
    labels = seq(4), right = FALSE, ordered_result = TRUE)
```

```
##  [1] 1 2 2 3 2 4 3 3 4 2
## Levels: 1 < 2 < 3 < 4
```

Một cách khác tuy nhìn không thuận tiện nhưng lại thuận lợi hơn về mặt tính toán là chỉ sử dụng các biểu thức logic và số học. Chúng ta sẽ xem kết quả trước (hãy tập trung vào nội dung của hàm `mutate()`), sau đó mình sẽ giải thích chi tiết.


```r
d %>%
    select(id, bmi) %>%
    mutate(
        bmi_group = 1 + (bmi >= 18.5) + (bmi >= 25) + (bmi >= 30)
    ) %>%
    kable()
```



| id|      bmi| bmi_group|
|--:|--------:|---------:|
|  1| 17.64305|         1|
|  2| 20.31053|         2|
|  3| 19.76630|         2|
|  4| 29.20992|         3|
|  5| 23.60592|         2|
|  6| 30.74207|         4|
|  7| 25.70744|         3|
|  8| 29.77594|         3|
|  9| 34.85026|         4|
| 10| 23.53065|         2|

Cách làm này dựa trên nguyên tắc về phân loại mình nêu trên, đấy là mỗi bệnh nhân chỉ được phân vào một trong 4 nhóm, và các nhóm đánh số từ 1 đến 4. Theo công thức trong hàm `mutate()`, những người có BMI cao sẽ thỏa mãn cả các điều kiện của BMI thấp hơn, và do đó, "tổng điểm" sẽ cao hơn. Nếu chưa mường tượng ra, bạn hãy nhìn bảng dưới đây và tự suy ngẫm.


```r
d %>%
    select(id, bmi) %>%
    mutate(
        bmi_2 = as.numeric(bmi >= 18.5),
        bmi_3 = as.numeric(bmi >= 25),
        bmi_4 = as.numeric(bmi >= 30),
        bmi_group = 1 + (bmi >= 18.5) + (bmi >= 25) + (bmi >= 30)
    ) %>%
    kable()
```



| id|      bmi| bmi_2| bmi_3| bmi_4| bmi_group|
|--:|--------:|-----:|-----:|-----:|---------:|
|  1| 17.64305|     0|     0|     0|         1|
|  2| 20.31053|     1|     0|     0|         2|
|  3| 19.76630|     1|     0|     0|         2|
|  4| 29.20992|     1|     1|     0|         3|
|  5| 23.60592|     1|     0|     0|         2|
|  6| 30.74207|     1|     1|     1|         4|
|  7| 25.70744|     1|     1|     0|         3|
|  8| 29.77594|     1|     1|     0|         3|
|  9| 34.85026|     1|     1|     1|         4|
| 10| 23.53065|     1|     0|     0|         2|


# Phân loại từ nhiều biến

Nếu tiêu chí phân loại của bạn như sau:

* 1=Nam BMI >30
* 2=Nam BMI >28
* 3=Nam BMI >25 và <=30 hoặc Nữ BMI >23 và <=28
* 4=Còn lại

thì bạn sẽ gặp khó khăn trong việc dùng công thức ở trên. Tuy nhiên, chúng ta vẫn có thể tổng quát hóa công thức toán học đó như sau.


```r
d %>%
    mutate(
        ploai = 1 * (sex == 1) * (bmi > 30) +
            2 * (sex == 2) * (bmi > 28) +
            3 * (
                (sex == 1) * (bmi > 25) * (bmi <= 30) +
                (sex == 2) * (bmi > 23) * (bmi <= 28)
            ),
        ploai = ploai * (ploai > 0) + 4 * (ploai == 0)
    ) %>%
    kable()
```



| id| sex|      bmi| ploai|
|--:|---:|--------:|-----:|
|  1|   2| 17.64305|     4|
|  2|   1| 20.31053|     4|
|  3|   2| 19.76630|     4|
|  4|   1| 29.20992|     3|
|  5|   1| 23.60592|     4|
|  6|   2| 30.74207|     2|
|  7|   1| 25.70744|     3|
|  8|   1| 29.77594|     3|
|  9|   1| 34.85026|     1|
| 10|   2| 23.53065|     3|

Tư duy trong giải pháp này là bạn có thể tạo ra các phân nhóm chỉ từ phép cộng và phép nhân. Phép nhân sẽ tương đương với toán tử `AND` (nếu `A` và `B` chỉ là 0 hoặc 1 thì `A AND B` = 1 khi và chỉ khi `A` = `B` = 1), còn phép cộng sẽ tương đương với toán tử `OR` nếu như `A` và `B` không bao giờ đồng thời bằng 1 (khi đó `A OR B` = 0 khi và chỉ khi `A` = `B` = 0). Và khi một trường hợp là đúng (`TRUE`, =1), bạn chỉ cần nhân với code tương ứng của nó, thế là xong. Để thực hiện theo cách này, chúng ta cần đảm bảo các chuỗi điều kiện chỉ đúng cho một trong các phân nhóm; nếu có nhiều phân nhóm cùng đúng (một người có thể thuộc về nhiều nhóm) thì lệnh sẽ tạo ra các code mới ngoài dự kiến (là tổng của các phân nhóm thỏa mãn điều kiện).

Bạn có thể dùng hàm `case_when()` để đơn giản hóa biểu thức tính toán ở trên.


```r
d %>%
    mutate(
        ploai = case_when(
            (sex == 1) & (bmi > 30) ~ 1,
            (sex == 2) & (bmi > 28) ~ 2,
            (sex == 1) & (bmi > 25) | (sex == 2) & (bmi > 23) ~ 3,
            TRUE ~ 4
        )
    ) %>%
    kable()
```



| id| sex|      bmi| ploai|
|--:|---:|--------:|-----:|
|  1|   2| 17.64305|     4|
|  2|   1| 20.31053|     4|
|  3|   2| 19.76630|     4|
|  4|   1| 29.20992|     3|
|  5|   1| 23.60592|     4|
|  6|   2| 30.74207|     2|
|  7|   1| 25.70744|     3|
|  8|   1| 29.77594|     3|
|  9|   1| 34.85026|     1|
| 10|   2| 23.53065|     3|

Cách làm của hàm `case_when()` sẽ giúp bạn bớt đi được một số điều kiện (ví dụ, ở nhóm 3 bạn không cần thêm điều kiện BMI <=30 cho nam và <=28 cho nữ). Khi các điều kiện cho nhóm 1 và 2 không thỏa mãn, thì điều kiện BMI <= 30 hoặc 28 đã tự động được thỏa mãn.
