# Julia 基礎教學

###### tags: `Julia`

在開始實作之前，有一個小小的提醒。如果你使用的是 Jupyter notebook，那麼你可以利用 `\` 輸入由 Jupyter notebook 與 $\texttt{Julia}$ 支援的 [Unicode](https://docs.julialang.org/en/v1/manual/unicode-input/)，並按下 `tab` 鍵完成輸入。例如：

```julia
## 希臘字母
α β γ δ η μ σ ξ π

## 上、下標
x₁ σ²

## 一些運算符號
÷ × −
```

這些對於設定變數來說，是一個極為方便且易讀的形式。但切記，不要過度使用，否則在寫程式時很容易搞混。

## 變數與基本數學運算

在 $\texttt{Julia}$ 中，我們可以進行一些基本的運算：

### 四則運算：加、減、乘、除
```julia
@show 1 + 2
@show 4 - 3
@show 5 * 6
@show 7 / 8

## 1 + 2 = 3
## 4 - 3 = 1
## 5 * 6 = 30
## 7 / 8 = 0.875
```

上面的指令 `@show` 代表我們要將運算結果顯示出來。這麼做的目的是，如果我們只有寫 `1 + 2`，之後進行運算時，只會顯示最後一行執行的結果，不會出現 `1 + 2` 的結果。如果說我們想要計算許多數字，但又不想打那麼多運算符的話，可以使用下列的方式：

```julia
@show +(1, 20, 4)
@show -(1, 20)
@show *(1, 20, 4)
@show /(1, 20)
@show \(3, 6)

## 1 + 20 + 4 = 25
## 1 - 20 = -19
## 1 * 20 * 4 = 80
## 1 / 20 = 0.05
## 3 \ 6 = 2.0
```

注意到，減法與除法的括號內僅能輸入兩個數字。而 `\` 則是將前面的數字當作被除數，後者當除數。如果想要取一個數的倒數，我們可以用 `inv()`。

```julia
@show inv(2)

## inv(2) = 0.5
```

如果想要計算餘數，可使用 `mod()` 或 `%`，前面的數字放被除數，後面放除數。例如

```julia
@show mod(15, 4)
@show 15 % 4

## mod(15, 4) = 3
## 15 % 4 = 3
```

### 指數、對數與自然數運算

```julia
@show 2 ^ 10

## 2 ^ 10 = 1024
```

如果我們要將冪次項(power)變成分數形式，就必須以括號將其包起來，避免運算出錯。

```julia
@show 2 ^ (1/2)

## 2 ^ (1 / 2) = 1.4142135623730951
```

如果我們想要計算

$$
\log_2 64 = 6
$$

方法如下

```julia
@show log(2, 64)

## log(2, 64) = 6.0
```

前面的數字為對數的底(base)，後面則是真數(antilogarithm)。一般來說，如果沒有特別指定對數的底，那麼其底數便是 $e \approx 2.71828 \cdots$，即自然數。既然講到自然數，那我們就要來說說它的計算方式。如果我們想要計算

$$
\exp(10) = e^{10}
$$

可以寫成

```julia
@show exp(10)

## exp(10) = 22026.465794806718
```

特別的是，考慮以下函數

$$
2^x
$$

我們可以透過以下的方式運算。

```julia
@show exp2(10)

## exp2(10) = 1024.0
```

### 三角函數

三角函數有以下幾個常用的指令 `sin()`、`cos()`、`tan()`、`cot()`、`sec()`、`csc()`。

## 向量與矩陣

向量(vector)可以透過以下方式表達。

```julia
x = [1, 2, 3, 4]

## 4-element Vector{Int64}:
## 1
## 2
## 3
## 4
```

矩陣(matrix)則為

```julia
@show y = [1 2; 3 4]

## 2×2 Matrix{Int64}:
## 1  2
## 3  4
```

更多關於向量與矩陣的運算，請參考[線性代數](https://hackmd.io/@yueswater/LinearAlgebra)這篇文章。

## 函式

給定以下函數，

$$
f(x) = 3x^3 - 5x^2 + \sin(x) - x
$$

我們希望每次只要輸入 `f(x)` 即可，不用重複輸入整串函數，這時候我們就需要函式幫助我們達成這個目的。

```julia
function f(x)
    return 3x^3 - 5x^2 + sin(x) - x
end

## f (generic function with 1 method)
```

比如我們要計算 $f(10)$，

```julia
f(10)

## 2489.4559788891106
```

我們也可以寫成一行的函式，

```julia
g(x) = 9x^3 - 4x^2 + cos(x) - 6
```

若要計算 $g(10)$，

```julia
g(10)

## 8593.160928470923
```

但如果今天給定下列函數：

$$
P(x, y) = x + y
$$
我們一樣可以透過一行解決！

```julia
P(x, y) = x + y
```

則計算 $P(5, 6)$

```julia
z(5, 6)

## 11
```

就變得輕鬆多了。但如果我們想要將一整個矩陣的元素都進行某個函數的運算呢？假設我們直接丟進上面的函數 `P(x, y)`，來看看會發生什麼結果。首先設定矩陣 $\mathbf{A}$

```julia
A = [1 3 7
     4 7 2
     0 1 1]
```

接著將其丟入函數中，

```julia
P(A, 4)

MethodError: no method matching +(::Matrix{Int64}, ::Int64)
For element-wise addition, use broadcasting with dot syntax: array .+ scalar
Closest candidates are:
  +(::Any, ::Any, ::Any, ::Any...) at C:\Users\user\AppData\Local\Programs\Julia-1.7.2\share\julia\base\operators.jl:655
  +(::T, ::T) where T<:Union{Int128, Int16, Int32, Int64, Int8, UInt128, UInt16, UInt32, UInt64, UInt8} at C:\Users\user\AppData\Local\Programs\Julia-1.7.2\share\julia\base\int.jl:87
  +(::Rational, ::Integer) at C:\Users\user\AppData\Local\Programs\Julia-1.7.2\share\julia\base\rational.jl:311
  ...

Stacktrace:
 [1] P(x::Matrix{Int64}, y::Int64)
   @ Main .\In[3]:1
 [2] top-level scope
   @ In[3]:2
 [3] eval
   @ .\boot.jl:373 [inlined]
 [4] include_string(mapexpr::typeof(REPL.softscope), mod::Module, code::String, filename::String)
   @ Base .\loading.jl:1196
```

會發生錯誤！原因是我們定義的函數基本上只能進行數字的運算，並沒有規定其可以進行矩陣的運算。為了解決這個問題，我們可以在函數內先定義變數型別，從而能夠讓變數在運算時讓程式知道我們可以進行不同型別的運算。

```julia
P(A::Matrix, y::Number) = A + y*I
```

接著神奇的事情發生了！

```julia
P(A, 4)

## 3×3 Matrix{Int64}:
## 5   3  7
## 4  11  2
## 0   1  5
```

如此一來就可以將矩陣代入函數中進行計算了！

## 安裝套件
如果想要安裝套件，我們要使用以下指令。

```julia
using Pkg; Pkg.add("<套件名稱>")
```

如果想要同時安裝許多套件，可以參考以下的方式：

```julia
using Pkg; Pkg.add(["<套件1>", "<套件2>", "<套件3>", ...])
```

當然也可以用 `import` 取代 `using`[^1]。安裝完套件之後，就可以用 `using <套件名稱>` 引用套件。

    
## 小結
今天的教學就到這邊，如果有不清楚或是不了解的地方，都歡迎詢問我，當然google 大神肯定比我厲害，希望大家喜歡今天的教學！我們下回見。
    
[^1]: 不過 $\texttt{Julia}$ 在區分 `using` 跟 `import` 上不如 $\texttt{python}$ 清楚。