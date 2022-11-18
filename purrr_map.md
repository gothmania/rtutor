---
title: Lặp trong R
author: Hoàng Bảo Long, MD MPH
output:
    pdf_document:
        keep_md: true
        keep_tex: false
        latex_engine: xelatex
        number_section: true
geometry: margin=1in
---


```r
library(knitr)
opts_chunk$set(
    message = FALSE
)
```

# Ví dụ 1

## Cách lặp truyền thống

Chúng ta sẽ đến với một bài toán đơn giản để mô phỏng cho việc lặp trong R. Chúng ta có một vector `v` chứa các phần tử là các dữ liệu số, và chúng ta muốn cộng thêm 1 cho mỗi phần tử. Chúng ta sẽ lưu kết quả cộng vào vector `v_a`.


```r
v <- c(1, 2, 3)
v_a1 <- v

for (i in v) {
    v_a1[i] <- v[i] + 1   # cộng 1 cho mỗi phần tử thứ i của vector v
}

print(v)
```

```
## [1] 1 2 3
```

```r
print(v_a1)
```

```
## [1] 2 3 4
```

Thao tác trên đây mô phỏng việc sử dụng vòng lặp `for` để thực hiện các tính toán giống nhau lặp đi lặp lại (thao tác cơ bản thì giống nhau, tham số đầu vào có thể khác nhau). Thông thường chúng ta sẽ gói gọn các công việc này vào trong một **hàm** để dễ tái sử dụng cho nhiều lần khác nhau, cũng như dễ quản lí mã lệnh và chỉnh sửa khi cần. Chẳng hạn, bạn có thể viết hàm `add_y()` để cộng thêm `y` vào mỗi phần tử trong vector, và mặc định `y` bằng 1.


```r
add_y <- function(x, y = 1) {
    x + y
}

add_y(1)
```

```
## [1] 2
```

Vòng lặp của chúng ta trở thành như sau.


```r
for (i in v) {
    v_a1[i] <- add_y(v[i])
}

print(v_a1)
```

```
## [1] 2 3 4
```

## Sử dụng `purrr::map_dbl()`

Bạn có thể đơn giản hóa vòng lặp này bằng hàm `map_dbl()` trong thư viện `purrr`. Hàm này sẽ lặp qua từng phần tử của vector `v`, thực hiện hàm `add_y()` trên từng phần tử đó, và trả về một vector là kết quả thực hiện trên toàn bộ vector `v`. Tốc độ lặp của hàm này nhanh hơn so với việc sử dụng vòng lặp `for`, do không phải truy xuất bộ nhớ liên tục và các tối ưu về vectorization khác.


```r
library(purrr)

v_a2 <- map_dbl(v, add_y)

print(v_a2)
```

```
## [1] 2 3 4
```

Nếu bạn chỉ muốn thực hiện một phép cộng đơn giản, chúng ta có thể làm nhanh hơn nữa như dưới đây. Cách viết thẳng hàm vào trong câu lệnh mà không khai báo hàm gọi là hàm lambda hay anonymous function. Hệ sinh thái Tidyverse cho phép bạn viết tắt việc khai báo hàm lambda như dòng lệnh tiếp theo (tính ra vector `v_a4`), `.x` là đại diện cho đối số của hàm lambda (tương tự `x` trong `function(x)`).


```r
v_a3 <- map_dbl(v, function(x) x + 1)
v_a4 <- map_dbl(v, ~ .x + 1)

print(v_a3)
```

```
## [1] 2 3 4
```

```r
print(v_a4)
```

```
## [1] 2 3 4
```


# Ví dụ 2

Trong ví dụ phức tạp hơn dưới đây, chúng ta sẽ cùng thao tác trên một data frame.


```r
library(dplyr)

set.seed(0)
d <- data.frame(
    id = seq(10),
    a = rnorm(10),
    b = rgamma(10, 1)
)

d %>% kable()
```



| id|          a|         b|
|--:|----------:|---------:|
|  1|  1.2629543| 1.1857109|
|  2| -0.3262334| 0.0946191|
|  3|  1.3297993| 0.1572015|
|  4|  1.2724293| 0.3108054|
|  5|  0.4146414| 0.4687319|
|  6| -1.5399500| 0.0681973|
|  7| -0.9285670| 1.2492921|
|  8| -0.2947204| 1.0081313|
|  9| -0.0057672| 1.3609450|
| 10|  2.4046534| 1.2059882|

Chúng ta sẽ tính tổng bình phương giá trị của tất cả các bản ghi trong một cột. Để làm việc này, chúng ta sẽ viết hàm `ssq()`. Hàm `sapply()` mà chúng ta sử dụng có tính năng tương tự hàm `map_dbl()`, và là một hàm sẵn có trong R.


```r
ssq <- function(v) {
    sum(sapply(v, function(x) x ^ 2))
}

ssq(d$a)
```

```
## [1] 14.36379
```

Bài toán phức tạp hơn là chúng ta muốn chạy hàm này cho nhiều cột khác nhau. Bên cạnh đó mình cũng muốn trả về trung bình của tổng bình phương, và thêm tên cột vào để dễ theo dõi. Vì vậy, mình tạo thêm một hàm `calc_ssq()` với đối số `name` là tên cột mà mình muốn thực hiện các phép tính toán. Hàm này sẽ trả về một data frame có một dòng, là kết quả tính toán tương ứng với cột trong `name`.


```r
calc_ssq <- function(d, name) {
    data.frame(
        name = name,
        ssq = ssq(d[name])
    ) %>%
        mutate(
            mean_ssq = ssq / nrow(d)
        )
}

calc_ssq(d, "a")
```

```
##   name      ssq mean_ssq
## 1    a 14.36379 1.436379
```

Cái hay của `purrr` là nó cung cấp hàm `map_df()`, tự động gộp các data frame sau mỗi lần chạy vào với nhau, và tạo thành một data frame duy nhất.


```r
vars_to_calc <- c("a", "b")
map_df(vars_to_calc, ~ calc_ssq(d, .x)) %>% kable()
```



|name |       ssq|  mean_ssq|
|:----|---------:|---------:|
|a    | 14.363786| 1.4363786|
|b    |  7.644174| 0.7644174|
