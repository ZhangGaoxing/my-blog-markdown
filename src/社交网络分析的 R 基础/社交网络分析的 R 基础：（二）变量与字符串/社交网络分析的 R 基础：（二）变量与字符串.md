本章会从 R 语言中最基本的数据类型开始介绍，在此之后就可以开始 R 语言实践了。对社交网络分析而言，我们在处理字符串上所花费的时间要远远大于处理数字的时间，因此本章还会介绍常用的字符串处理操作。

* [变量]()
* [字符串]()
  * [字符串的创建]()
  * [特殊字符的转义]()
  * [字符串的其他常用操作]()

## 变量

R 语言中基本的数据类型包括：
1. 整型：整数，如 `100`；
2. 浮点型：小数，如 `3.14`；
3. 字符串型：R 语言中的字符串可以使用 `"` 或者 `'` 定义，如 `"abc"`，`'abc'`；
4. 逻辑型（Logical）：其他编程语言中常称为布尔型，在 R 语言中使用严格区分大小写的 `TRUE` 和 `FALSE` 表示，或者使用缩写 `T` 和 `F`。

变量就是对数据类型的引用，比如有一个整型值 100，想在程序中使用它并用 a 来表示，将 100 赋值给 a 后（`a <- 100`），a 就称之为变量。R 语言对变量的定义并不像强类型的语言一样，需要在定义变量时声明变量的数据类型。当进行赋值操作时，就定义了一个新的变量。下面这段程序就是声明了一个变量 a，并且将 100 赋值给了变量 a，这三行代码的操作是等价的：
```R
a <- 100
a = 100
100 -> a
```
在 R 语言中标准的赋值符号为 `<-`，这其中包含两个字符 `<` 和 `-`。当然也可以使用 `=` 进行代替。从上面的代码也可以观察到，赋值符号 `<-` 是有方向性的，指向被赋值的对象。

变量的名称不是随意的，一个有效的变量名由**字母**开头，后面跟上任意数量的字母，数字以及下划线。下面是一些合法的变量名：`a`、`a1`、`a_b`。下面是一些非法的变量名称：`1`、`1a`、`_a`。当然，也不要使用关键字作为变量名，关键字是用于描述 R 语言的语法的。

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold">💡&nbsp;提示</p>
    <p><span>下面给出一些特殊的运算符：</span></p>
    <p>
        <table>
        <tr>
            <th>运算符</th><th>描述</th><th>示例</th><th>输出</th>
        </tr>
        <tr>
            <td>^</td><td>乘方</td><td>2^3</td><td>8</td>
        </tr>
        <tr>
            <td>%%</td><td>求余</td><td>3 %% 2</td><td>1</td>
        </tr>
        <tr>
            <td>%/%</td><td>整除</td><td>5 %/% 2</td><td>2</td>
        </tr>
        </table>
    </p>
</div>

## 字符串

### 字符串的创建

R 语言中的字符串既可以使用双引号 `"` 定义，也可以使用单引号 `'` 定义。但是为什么要使用两种引号定义字符串？
```R
> '这是包含"双引号"的字符串'
[1] "这是包含\"双引号\"的字符串"

> "这是包含'单引号'的字符串"
[1] "这是包含'单引号'的字符串"
```
这样做的好处是可以在不转义引号的情况下，创建本身就包含引号的字符串。可以在双引号 `"` 定义的字符串中使用单引号 `'`，也可以在单引号 `'` 定义的字符串中使用双引号 `"`。

使用 `as.character()` 也可以将其他的数据类型转换成字符串:
```R
> as.character(3.14)
[1] "3.14"

> as.character(T)
[1] "TRUE"
```

### 特殊字符的转义

转义是指输出具有特殊意义的字符，比如想要在双引号定义的字符串中使用双引号，或者在字符串中使用换行操作。在 R 语言中使用反斜杠 `\` 进行转义操作，常见的转义字符有换行符 `\n`，引号 `\" \'`，以及对反斜杠本身进行转义 `\\`。
```R
> writeLines("Use \"double quotation\" in a string")
Use "double quotation" in a string

> writeLines("Use line breaks\nin a string")
Use line breaks
in a string
```

### 字符串的其他常用操作

获取字符串的长度 `nchar()`：
```R
> nchar("Social Network")
[1] 14
```

字符串的拼接 `paste()`，`sep` 参数为连接的字符：
```R
> paste("Social", "Network", sep = "-")
[1] "Social-Network"
```

字符串的分割 `strsplit()`：
```R
> strsplit("Social-Network", "-")
[[1]]
[1] "Social"  "Network"
```

字符串的截取 `substr()`，要注意的是，和大多数语言不同，R 语言的索引从 1 开始：
```R
> substr("Social Network", 1, 6)
[1] "Social"
```

字符串的格式化输出 `sprintf()`，在 R 语言中也采用类似 C 语言的风格对变量进行格式化：
1. `%s`：字符串
2. `%f`：浮点型
3. `%d`：整数
4. `%e`：科学计数法
```R
> sprintf("The degree of the node is %d\n", 4)
[1] "The degree of the node is 4\n"
```
当需要输出转义字符串时，可以在外层套用 `cat()`：
```R
> cat(sprintf("The degree of the node is %d\n", 4))
The degree of the node is 4
```

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e0f2ff;color: #002b4d;margin: 10px">
    <p style="margin-top:0;font-weight: bold">✏️&nbsp;练习</p>
    <p><span>1. 第一章留下的问题 <code>"a"+"b"</code> 会输出 <code>ab</code> 吗，如何将<code>"a"</code> 和 <code>"b"</code> 拼接成 <code>"ab"</code> ；</span></p>
    <p><span>2. 截取 <code>"Social Network"</code> 中的 <code>"Network"</code>。</span></p>
</div>

## 参考

1. <a id="1" target="_blank" href="https://cran.r-project.org/doc/manuals/r-release/R-intro.html">An Introduction to R</a>
2. <a id="2" target="_blank" href="https://www.runoob.com/r/r-string.html">R 字符串 | 菜鸟教程</a>