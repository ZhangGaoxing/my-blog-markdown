在第二章介绍了 R 语言中的基本数据类型，本章会将其组装起来，构成特殊的数据结构，即向量、矩阵与列表。这些数据结构在社交网络分析中极其重要，本质上对图的分析，就是对邻接矩阵的分析，而矩阵又是由若干个向量构成，因此需要熟练掌握这些特殊的数据结构。

* [向量]()
  * [向量的创建]()
  * [向量元素的访问]()
  * [向量的运算]()
  * [向量的其他常用操作]()
* [矩阵]()
  * [矩阵的创建]()
  * [矩阵元素的访问]()
  * [矩阵的运算]()
  * [矩阵的特征值与特征向量]()
* [列表]()
  * [列表的创建]()
  * [列表元素的访问]()

## 向量

### 向量的创建

向量（vector）作为 R 语言中最简单的数据结构，由一串有序的基本数据类型变量构成。

```R
x <- c(1, 2, 3, 4, 5)
```
上面一行代码就是创建一个包含 5 个元素的向量 `x`，而 `c()` 就是创建向量的函数。多个向量也可以使用 `c()` 进行拼接：
```R
x <- c(1, 2, 3, 4, 5)
y <- c(6, 7, 8, 9, 10)
z <- c(x, y)
```
代码中的向量 `z` 包含 10 个元素，即向量 `x` 和向量 `y` 的拼接。

向量的创建也可以通过面向对象的方式实现：
```R
x <- vector(mode = "integer", length = 5)
```
参数 `mode` 为向量中存储的数据类型，对应 R 语言中基本的数据类型，如整型 `integer`，浮点型 `numeric`, 字符串型 `character`，逻辑型 `logical` 等等；`length` 为初始向量的长度。向量作为一种无限长度的数据结构，此处的 `length` 是指向量初始化时的长度，后续仍然可以使用 `c()` 添加元素。
```R
x <- c(x, 0)  # 向 x 中添加元素 0
```

### 向量元素的访问

向量中的元素通过“`[索引]`”的形式访问。_**需要注意的是 R 语言中的索引不代表偏移量，而代表第几个，即索引从 1 开始。**_
```R
> x <- c(1, 2, 3, 4, 5)
> x[2]
[1] 2
```

在了解向量元素的访问后，也可以通过元素访问的形式向其中添加元素：
```R
> x[6] <- 6  # x 原长度为5
> x
[1] 1 2 3 4 5 6
```

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold">💡&nbsp;提示</p>
    <p><span>在 R 语言中任何使用索引的数据结构都可以使用元素访问的形式扩充。</span></p>
</div>

想要从向量中取出多个元素需要在方括号内传递索引的向量，即“`[c(索引)]`”。
```R
> x[2: 4]  # 取出第 2 到 4 项
[1] 2 3 4
> x[c(1, 3, 5)]  # 取出第 1,3,5 项
[1] 1 3 5
> x[c(-1, -5)]  # 去掉第 1,5 项
[1] 2 3 4
```

R 语言中还存在一种特殊的索引——名称索引。
```R
> x <- c(1, 2, 3, 4, 5)
> names(x) <- c("one", "two", "three", "four", "five")  # 对名称索引进行赋值
> x["three"]  # 使用名称索引访问元素
three
3
> names(x)  # 查看名称索引
[1] "one"   "two"   "three" "four"  "five"
```
名称索引相比数值索引的好处就是容易记忆，在对图中节点属性进行分析时，通常使用节点的名称去访问图中的节点，而不是使用节点的索引。

### 向量的运算

向量可以直接进行算数运算，运算时是向量的**对应元素**进行同样的算术运算。比如：
```R
> x <- c(1, 2, 3, 4, 5)
> y <- c(5, 4, 3, 2, 1)
> x + y
[1] 6 6 6 6 6
```

基本的算术运算包括：`+`、`-`、`*`、`/`、乘方`^`。还包括常用的数学函数：`log()`、`sin()`、`sqrt()`等等。还有一些特殊的统计函数：最大值 `max()`、最小值 `min()`、求和 `sum()`、平均值 `mean()`等等。
```R
> x <- c(1, 2, 3, 4, 5)
> max(x)
[1] 5
> mean(x)
[1] 3
```

向量的逻辑运算包括两种情况，一种是对向量中的每一个元素，一种是对向量整体：
| 运算符 | 描述 |
| :-: | :- |
| & | 元素逻辑与运算符，将第一个向量的每个元素与第二个向量的相对应元素进行与运算 |
| \| | 元素逻辑或运算符，将第一个向量的每个元素与第二个向量的相对应元素进行或运算 |
| && | 逻辑与运算符，只对两个向量的第一个元素进行与运算 |
| \|\| | 逻辑或运算符，只对两个向量的第一个元素进行或运算 |

```R
> x <- c(T, T, F, F, F)
> y <- c(T, T, F, T, T)
> x & y
[1]  TRUE  TRUE FALSE FALSE FALSE
> x | y
[1]  TRUE  TRUE FALSE  TRUE  TRUE
> x && y
[1] TRUE
> x || y
[1] TRUE
```

### 向量的其他常用操作

获取向量的长度 `length()`：
```R
> length(c(1, 2, 3, 4, 5))
[1] 5
```

查找特定元素在向量中的索引 `which()`：
```R
> x <- c(1, 2, 3, 4, 5)
> which(x == 2)
[1] 2
```

使用 `%in%` 判断元素是否在向量中存在：
```R
> 2 %in% c(1, 2, 3, 4, 5)
[1] TRUE
```

对向量中的元素进行排序 `order()`，需要注意的是 `order()` 返回的排序结果是向量值的索引：
```R
> x <- c(10, 20, 30, 40, 50)
> order(x, decreasing = TRUE)
[1] 5 4 3 2 1
```

统计特定元素在向量中出现的次数 `table()`：
```R
> x <- c(T, T, F, F, F)
> table(x)
x
FALSE  TRUE
    3     2
```

## 矩阵

### 矩阵的创建

矩阵（matrix）作为社交网络分析中的一个重要工具，其并不算是一个基本的数据结构。你可以将矩阵看成一个二维数组（array），或是由多个向量（vector）构成。在 R 语言中使用 `matrix()` 函数来创建矩阵。
```R
matrix(data = NA, nrow = 1, ncol = 1, byrow = FALSE, dimnames = NULL)
```
其中 `data` 表示矩阵的填充元素，`nrow` 表示矩阵的行数，`ncol` 表示矩阵的列数，`byrow` 表示 `data` 的值是否按行填充，`dimnames` 给矩阵行列的名称赋值。

```R
> matrix(c(1:6), nrow = 2, ncol = 3, byrow = TRUE, dimnames = list(c("r1", "r2"), c("c1", "c2", "c3")))
   c1 c2 c3
r1  1  2  3
r2  4  5  6
```
上面即创建了一个 2 行 3 列的矩阵，通过按行填充元素的方式，并且给行和列赋予了名称。获取矩阵的行数和列数可以使用函数 `nrow()` 和 `ncol()`。

矩阵还可以通过组合向量的方式创建，使用 `rbind()` 函数按行组合向量，使用 `cbind()` 函数按列组合向量：
```R
> v1 <- c(1:3)
> v2 <- c(4:6)
> v3 <- c(7:9)

> rbind(v1, v2, v3)  # 按行组合
   [,1] [,2] [,3]
v1    1    2    3
v2    4    5    6
v3    7    8    9

> cbind(v1, v2, v3)  # 按列组合
     v1 v2 v3
[1,]  1  4  7
[2,]  2  5  8
[3,]  3  6  9
```

### 矩阵元素的访问

矩阵中的元素通过“`[行索引, 列索引]`”的形式访问。
```R
> m <- matrix(c(1:6), nrow = 3)
> m[3, 2]
[1] 6
```

想要从矩阵中取出行向量或者列向量，使用“`[行索引,]`”或者“`[,列索引]`”。
```R
> m[1,]  # 取第一行
[1] 1 4

> m[,2]  # 取第二列
[1] 4 5 6
```

在给矩阵的行列赋值名称后，可以使用名称索引访问。
```R
> rownames(m) <- c("r1", "r2", "r3")  # 定义行的名称
> colnames(m) <- c("c1", "c2")  # 定义列的名称

> m["r2", "c2"]
[1] 5
```

### 矩阵的运算

矩阵直接进行算术运算时，是两个矩阵**对应位置的元素**做运算。数学函数和统计函数在矩阵中的用法与在向量中的用法相同。
```R
> m1 <- matrix(c(1:4), nrow = 2)
> m2 <- matrix(c(5:8), nrow = 2)
> m1 * m2
     [,1] [,2]
[1,]    5   21
[2,]   12   32
```

矩阵还包括一些特有的运算，比如内积 `%*%`，外积 `%o%`。
```R
> m1 <- matrix(c(1:6), nrow = 2)
> m2 <- matrix(c(1:6), nrow = 3)
> m1 %*% m2  # 矩阵的内积
     [,1] [,2]
[1,]   22   49
[2,]   28   64

> m1 <- c(1, 2, 3)
> m2 <- c(4, 5, 6)
> m1 %o% m2  # 矩阵的外积
     [,1] [,2] [,3]
[1,]    4    5    6
[2,]    8   10   12
[3,]   12   15   18
```

矩阵的转置使用函数 `t()`。
```R
> m <- matrix(c(1:4), nrow = 2)
> t(m)
     [,1] [,2]
[1,]    1    2
[2,]    3    4
```

### 矩阵的特征值与特征向量

特征值与特征向量作为矩阵的重要属性，不仅在传统的图分析中有重要的意义，在图卷积中也有重要的应用。R 语言为我们提供了计算函数 `eigen()`：
```R
> v1 <- c(1, 0, 0)
> v2 <- c(2, 3, 0)
> v3 <- c(4, 5, 6)
> m <- cbind(v1, v2, v3)

> eigen(m)              
eigen() decomposition
$values  # 特征值
[1] 6 3 1

$vectors  # 特征向量
          [,1]      [,2] [,3]
[1,] 0.6023442 0.7071068    1
[2,] 0.6844821 0.7071068    0
[3,] 0.4106893 0.0000000    0
```

随着网络规模的变大，`eigen()` 函数的计算速度会变得很慢，此时通常会使用 `RSpectra` 包来加快计算速度。在 `RSpectra` 包中使用 `eigs()` 函数计算特征值与特征向量：
```R
> library(RSpectra)
> eigs(m, 3)  # 这里的 3 是指要计算特征值与特征向量的个数
$values
[1] 6 3 1

$vectors
          [,1]      [,2] [,3]
[1,] 0.6023442 0.7071068    1
[2,] 0.6844821 0.7071068    0
[3,] 0.4106893 0.0000000    0
```

当网络规模继续变大，邻接矩阵中的节点数量到达数十万以上的规模时，`RSpectra` 包仍然有些捉襟见肘。这时使用 `Rcpp` 包调用 C++ 的代码，采用并行计算的方式加快计算速度。对于矩阵的计算操作，安装 `Rcpp` 包的同时还需要安装 `RcppEigen` 包。依赖的包安装完成后，新建一个 `matrix.cpp` 文件，将下面的代码复制到该文件中保存。
```cpp
// [[Rcpp::depends(RcppEigen)]]
#include <RcppEigen.h>

// [[Rcpp::export]]
SEXP eigenValues(const Eigen::Map<Eigen::MatrixXd> A){
    Eigen::SelfAdjointEigenSolver<Eigen::MatrixXd> es(A);
    return Rcpp::wrap(es.eigenvalues());
}

// [[Rcpp::export]]
SEXP eigenVectors(const Eigen::Map<Eigen::MatrixXd> A){
    Eigen::SelfAdjointEigenSolver<Eigen::MatrixXd> es(A);
    return Rcpp::wrap(es.eigenvectors());
}
```

紧接着在工作区中引入 `Rcpp` 包与 `matrix.cpp` 文件，此时就可以调用特征值计算函数 `eigenValues()` 和特征向量计算函数 `eigenVectors()`。

```R
> library(Rcpp)
> sourceCpp("matrix.cpp")
> eigenValues(m)
[1] 1 3 6
> eigenVectors(m)  
     [,1]      [,2]      [,3]
[1,]    1 0.7071068 0.6023442
[2,]    0 0.7071068 0.6844821
[3,]    0 0.0000000 0.4106893
```

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold">💡&nbsp;提示</p>
    <p><span>要实现其他的矩阵计算操作可以查看 RcppEigen 的教程：https://cran.r-project.org/web/packages/RcppEigen/vignettes/RcppEigen-Introduction.pdf</span></p>
</div>

## 列表

### 列表的创建

列表（list）在 R 语言中是由一个个对象所构成的集合，这些对象可以是不同的数据类型，比如数值、字符串、向量、矩阵等等。如果为列表元素定义名称的话，列表更像是 Python 中的字典，但 R 语言中的列表中的元素是有序的。在 R 语言中使用 `list()` 函数来创建列表。
```R
list(name = "ruby", age = 18, scores = c(100, 88.5, 82))
```

上面一行代码创建了一个包含数值、字符串与向量的列表，同时为每一个元素定义了名称。将其输入到 R 终端中，细心的你会发现这与矩阵计算特征值和特征向量的函数 `eigen()` 返回的类型一致。这种定义了名称的列表对于包含多个返回值的函数非常方便。
```R
> list(name = "ruby", age = 18, scores = c(100, 88.5, 82))
$name
[1] "ruby"

$age
[1] 18

$scores
[1] 100.0  88.5  82.0
```

列表还可以通过多个列表合并的方式创建，合并使用函数 `c()`。下面的代码展示了两个列表的合并，同时使用了未定义元素名称的列表创建方式。注意观测列表的输出结果，输出的索引表明了列表是有序的。
```R
> l1 <- list(matrix(c(1:4), nrow = 2))
> l2 <- list(c("a", "b", "c"), 12345)
> c(l1, l2)
[[1]]
     [,1] [,2]
[1,]    1    3
[2,]    2    4

[[2]]
[1] "a" "b" "c"

[[3]]
[1] 12345
```

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold">💡&nbsp;提示</p>
    <p><span><code>c()</code> 本质上并不是创建向量的函数，c 是 combine 的缩写，是一个合并函数。</span></p>
</div>

### 列表元素的访问

列表中的元素通过“`[[索引]]`”的形式访问，当列表元素定义了名称后可以使用“`$名称`”或者“`[["名称"]]`”的形式访问。
```R
> student <- list(name = "ruby", age = 18, scores = c(100, 88.5, 82))
> student[[1]]
[1] "ruby"
> student$age
[1] 18
> student[["scores"]]
[1] 100.0  88.5  82.0
```

对于在创建时没有定义名称的列表，仍然可以使用 `names()` 定义名称。
```R
l <- list(c("a", "b", "c"), 12345)
names(l) <- c("name1", "name2")
```

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e0f2ff;color: #002b4d;margin: 10px">
    <p style="margin-top:0;font-weight: bold">✏️&nbsp;练习</p>
    <p><span>1. 试着创建一个向量，看看向量能否包含不同类型的元素，比如 <code>c(1, "a")</code> 会创建一个什么向量；</p>
    <p><span>2. 试着对矩阵进行运算，能否求出一个矩阵的最大元素；</span></p>
    <p><span>3. 列表通过“<code>[索引]</code>”与“<code>[[索引]]</code>”有什么不同，输出看看；</span></p>
    <p><span>4. <code>list(c("a", "b", "c"))</code> 该列表的长度是多少。</span></p>
    <p><span>5. 试着对任意一个非空列表使用 <code>unlist()</code> 函数，看看会发生什么。</span></p>
</div>

## 参考

1. <a id="1" target="_blank" href="https://cran.r-project.org/doc/manuals/r-release/R-intro.html">An Introduction to R</a>
2. <a id="2" target="_blank" href="https://www.runoob.com/r/r-data-types.html">R 数据类型 | 菜鸟教程</a>
3. <a id="3" target="_blank" href="https://www.runoob.com/r/r-matrix.html">R 矩阵 | 菜鸟教程</a>
4. <a id="4" target="_blank" href="https://www.runoob.com/r/r-list.html">R 列表 | 菜鸟教程</a>