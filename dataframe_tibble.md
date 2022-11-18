# So sánh data frame và tibble

Khi thao tác với bảng số liệu trong R, bạn có thể lựa chọn sử dụng cấu trúc data frame có sẵn hoặc tibble, một kiểu dữ liệu mở rộng từ data frame được phát triển trong hệ sinh thái Tidyverse. Hai cấu trúc dữ liệu này có một số khác biệt mà bạn cần lưu ý khi thao tác.

```r
library(dplyr)
library(knitr)

opts_chunk$set(
    message = FALSE
)

d <- data.frame(
    a = c(1, 0, 1, 0, 0),
    b = seq(5)
)

d
```

```
##   a b
## 1 1 1
## 2 0 2
## 3 1 3
## 4 0 4
## 5 0 5
```

## Tên hàng

Trong khi data frame có tên hàng, tibble không có.


```r
rownames(d) <- c("a", "b", "c", "d", "e")
d
```

```
##   a b
## a 1 1
## b 0 2
## c 1 3
## d 0 4
## e 0 5
```


```r
d_tib <- tibble(d)
d_tib
```

```
## # A tibble: 5 x 2
##       a     b
##   <dbl> <int>
## 1     1     1
## 2     0     2
## 3     1     3
## 4     0     4
## 5     0     5
```

Muốn thêm tên hàng cho tibble, chúng ta tạo một cột mới.


```r
d_tib <- tibble(tibble::rownames_to_column(d, "id"))
d_tib
```

```
## # A tibble: 5 x 3
##   id        a     b
##   <chr> <dbl> <int>
## 1 a         1     1
## 2 b         0     2
## 3 c         1     3
## 4 d         0     4
## 5 e         0     5
```

Cũng vì lí do này, data frame có thể slice theo tên hàng, còn tibble thì không.


```r
d[c("a", "b"), ]
```

```
##   a b
## a 1 1
## b 0 2
```


```r
d_tib %>% filter(id %in% c("a", "b"))
```

```
## # A tibble: 2 x 3
##   id        a     b
##   <chr> <dbl> <int>
## 1 a         1     1
## 2 b         0     2
```

## Slice cột

Để truy cập vào dữ liệu của một cột trong data frame, chúng ta có những cách sau.


```r
d$a
```

```
## [1] 1 0 1 0 0
```


```r
d[, "a"]
```

```
## [1] 1 0 1 0 0
```


```r
d[, c("a")]
```

```
## [1] 1 0 1 0 0
```

Những cách này trả về vector nếu chỉ slice một cột. Muốn giữ nguyên định dạng data frame (gọi là subset), chúng ta làm như sau.


```r
d["a"]
```

```
##   a
## a 1
## b 0
## c 1
## d 0
## e 0
```

Thêm một cặp ngoặc vuông nữa, bạn cũng sẽ lấy được vector giá trị của cột.


```r
d[["a"]]
```

```
## [1] 1 0 1 0 0
```

Slicing bằng giá trị chuỗi kí tự của tên cột thuận lợi cho lập trình.


```r
col_name <- "a"
d[[col_name]]
```

```
## [1] 1 0 1 0 0
```

Đối với tibble, slicing luôn trả về subset.


```r
d_tib["a"]
```

```
## # A tibble: 5 x 1
##       a
##   <dbl>
## 1     1
## 2     0
## 3     1
## 4     0
## 5     0
```


```r
d_tib[, "a"]
```

```
## # A tibble: 5 x 1
##       a
##   <dbl>
## 1     1
## 2     0
## 3     1
## 4     0
## 5     0
```


```r
d_tib[, c("a")]
```

```
## # A tibble: 5 x 1
##       a
##   <dbl>
## 1     1
## 2     0
## 3     1
## 4     0
## 5     0
```

Để lấy vector, bạn có thể dùng các cách sau.


```r
d_tib$a
```

```
## [1] 1 0 1 0 0
```


```r
d_tib[["a"]]
```

```
## [1] 1 0 1 0 0
```


```r
d_tib %>% pull(a)
```

```
## [1] 1 0 1 0 0
```

Do vậy, muốn lập trình với tibble, bạn có thể làm như sau.


```r
d_tib[[col_name]]
```

```
## [1] 1 0 1 0 0
```

Hoặc


```r
d_tib %>% pull(!!rlang::sym(col_name))
```

```
## [1] 1 0 1 0 0
```

Tương tự, nếu muốn subset, bạn có thể làm như trên với hàm `select()`.


```r
d_tib %>% select(!!rlang::sym(col_name))
```

```
## # A tibble: 5 x 1
##       a
##   <dbl>
## 1     1
## 2     0
## 3     1
## 4     0
## 5     0
```
