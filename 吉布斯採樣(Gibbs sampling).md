# 吉布斯採樣(Gibbs sampling)

###### tags: `Numerical Methods`

對於單變數函數進行隨機抽樣或找出其累積分配函數(CDF)，我們可以利用反函數進行處理；但若同樣以反函數對於雙變數函數處理，取出隨機變數或找出 CDF，會發現結果會十分複雜。不過，透過吉布斯採樣，我們可以解決上面的問題：假設有一個聯合機率分配，可以先假定 $x$ 服從某個分配並對其抽樣，同時也假定 $y$ 也服從該分配，也就是說取其條件機率分配。我們可以透過下面的例子來了解：$X$ 代表性別，$Y$ 則是施打疫苗的種類

$$
\begin{array}{cccc}\hline Y/X & \textbf{male} & \textbf{female} & \textbf{total} \\ \textbf{AZ} &0.25& 0.15 &0.40\\ \textbf{BNT} &0.10 & 0.35 & 0.45 \\ \textbf{MVC} &0.10&0.05 & 0.15\\ \hline & 0.45 & 0.55 & 1.00\\\hline \end{array}
$$

上表中的 $0.40$ 、$0.45$ 與 $0.15$ 三者代表的意義是在給定疫苗的種類下，施打該疫苗的的人數加總（所有性別），記成 $\mathbb{P} (Y= \text{vaccine})$；而若給定男性的情況下，施打高端(MVC)的機率為 $0.10/0.45 = 0.22$，稱作條件機率分配，記作 $\mathbb{P} (Y = \text{vaccine}\mid \text{sex})$。了解完簡單的機率分配後，我們就可以來了幾吉布斯採樣的內容了。

## 吉布斯採樣的內容與性質

- 有相對應的組合，即給定 $x$ 可以對應到 $y$，反之亦可得到組合。
- 必須以容易、簡單的方式進行抽樣

而抽樣的步驟如下：

### 步驟 **1**

設定初始值$x$ 與 $y$ → $(x_0,y_0)$

### 步驟 2

利用 $x_1$ 從 $f(X|Y=y_0)$ 取樣，接著用 $y_1$ 從 $f(Y|X=x_1)$ 取樣，得到 $(x_1,y_1)$

### 步驟 **3**

利用 $x_2$ 從 $f(X|Y=y_1)$ 取樣，接著用 $y_2$ 從 $f(Y|X=x_2)$ 取樣，得到 $(x_2,y_2)$

### 步驟 **4**

重複步驟 2與步驟 3 $n$ 次.

### 步驟 **5**

確認其是否收斂(converge)

```julia
using Interact, WebIO
using Distributions, Plots, LinearAlgebra, Statistics

μ = [1,1]
σ = [2 0; 0 3]
min_N = 50
max_N = 2000

draw0 = [rand(Normal(0, 1), max_N)'; rand(Normal(0, 1), max_N)'];

@manipulate for num=min_N:max_N, corr=-1:0.1:1
    
    correlation = [1 corr; corr 1]
    covariance =  σ*correlation*σ
    C = cholesky(covariance)
    
    draw = C.L * draw0[:, 1:num] .+ μ  # take care of the mean, std, and correlation

    x = draw[1,:]
    y = draw[2,:]    
  
    layout = @layout [a{0.6w,0.4h} _
                      b{0.6w,0.6h} c{0.4w, 0.6h}]

    default(fillcolor=:lightgrey, markercolor=:white, grid=false, legend=false)
    plot(layout=layout, link=:both, size=(400, 400),  margin=-10Plots.pt)
    scatter!(x,y, subplot=2, framestyle =:box)
    histogram!([x y], subplot=[1 3], orientation=[:v :h], framestyle=:none, bins=min(num, 100), normalize=true)
end
```


上圖並非利用吉布斯採樣的方法產生的。若要實踐吉布斯採樣，我們可以依據上面的步驟勾勒出想法。

## ⌨️ 演練

假設 $x$ 與 $y$ 服從二元常態分配，其聯合機率分配如下

$$
f_{X Y}(x, y)=\frac{1}{2 \pi \sigma_{x} \sigma_{y} \sqrt{1-\rho^{2}}} \exp \left(-\frac{1}{2\left(1-\rho^{2}\right)}\left[\left(\frac{x-\mu_{x}}{\sigma_{x}}\right)^{2}+\left(\frac{y-\mu_{y}}{\sigma_{y}}\right)^{2}-2 \rho \frac{\left(x-\mu_{x}\right)\left(y-\mu_{y}\right)}{\sigma_{x} \sigma_{y}}\right]\right)
$$

其中 $\mu_x, \mu_y \in \mathbb{R}$，$\sigma_x, \sigma_y>0$， $\rho \in (-1,1)$，且均為一常數(constant)。其邊際機率分配為：

$$
\begin{aligned}f_{X}(x) &=\frac{1}{\sqrt{2 \pi} \sigma_{x}} e^{-\frac{1}{2}\left(\frac{x-\mu_{x}}{\sigma_{x}}\right)} \\f_{Y}(y) &=\frac{1}{\sqrt{2 \pi} \sigma_{y}} e^{-\frac{1}{2}\left(\frac{y-\mu_{y}}{\sigma_{y}}\right)}\end{aligned}
$$

條件機率分配則是

$$
\begin{aligned}&(X \mid Y=y) \sim N\left(\mu_{x}+\rho \frac{\sigma_{x}}{\sigma_{y}}\left(y-\mu_{y}\right), \quad \sigma_{x}^{2}\left(1-\rho^{2}\right)\right) \\&(Y \mid X=x) \sim N\left(\mu_{y}+\rho \frac{\sigma_{y}}{\sigma_{x}}\left(x-\mu_{x}\right), \quad \sigma_{y}^{2}\left(1-\rho^{2}\right)\right)\end{aligned}
$$

假設 $\mu_x = \mu_y = 0$，$\sigma_x = \sigma_y = 1$，$\rho = \rho$。請根據上述機率分配寫出一個進行吉布斯抽樣的函式。首先我們要先決定初始值：

```julia
init = [4,3] # Set the initial number of x0 and y0
```

接著我們產生一個由初始值組成的陣列用來做等一下的運算。

```julia
x = ones(1+skip_num+n) * init[1] # Create an array of x0 = 4
y = ones(1+skip_num+n) * init[2] # Create an array of y0 = 3

##
## 1001-element Vector{Float64}:
## 3.0
## 3.0
## 3.0

## ⋮
## 3.0
## 3.0
## 3.0
```

上面的`skip_num` 代表我們想要丟棄的值(burn-in points)，原因在於如果一開始的初始值在該分配發生機率極小的地方，例如常態分配的最左邊或最右邊，當它收斂回來的時候所經過的值都不會是我們想要的。接著我們寫一個迴圈，利用**取代(replace)陣列元素**的方式進行抽樣。此處不透過`append!` 的原因在於，當我們使用`append!` 時，迴圈每跑一次就要重新檢測陣列長度，這樣會增加運算時間，也會佔記憶體。

```julia
for i in range(2,1+skip_num+n)
        x[i] = rand(Normal(ρ * y[i-1], 1 - ρ ^ 2)) # Sample x1 by given y0
        y[i] = rand(Normal(ρ * x[i], 1 - ρ ^ 2)) # Sample y1 by given x1
end
```

我們可以來看看`x` 跟 `y` 長怎樣（上面設定 `ρ = 0.2`，`n = 1000` ，`skip_num = 100` ）。

```julia
x

## 1101-element Vector{Float64}:
## 4.0
## -0.04705256736833596
## 1.3052037427528813
## -0.06677596402993682
## ⋮
## -0.05473150108855082
##  0.5575666596686812
## -0.8040756956021986

y
## 1101-element Vector{Float64}:
## 3.0
## 1.136943703212021
## 0.1674852002420904
## ⋮
## 1.236202469133087
## -1.5420349774737756
## 0.0679277556673149
```

看起來好像是我們想要的結果，但我們有個地方要檢測一下，就是 $x$ 跟 $y$ 的相關係數(correlation)。

```julia
# using Pkg; Pkd.add("Statistics")
using Statistics
cor(x, y)

##
## 0.2596178739181784
```

誒？怎麼跟我們設定的有點差距啊！不過出現這個結果並不意外，我們試著把樣本數調整到 1000000，並重新計算其相關係數：

```julia
cor(x, y)

##
## 0.20038077119079106
```

很明顯的，當樣本數越多的時候，相關係數就會越接近我們設定的值。不過接下來還有一個問題：那我們之前設定的`skip_num` 到哪去了？可以看到我們陣列中其實總共有1101筆資料，但其中有100筆（上面設定的burn-in points）是我們不要的資料，所以我們必須把它從陣列裡頭清除（利用`deleteat!`）。

```julia
deleteat!(x, 2:skip_num)
deleteat!(y, 2:skip_num)
```

不過其實可以直接使用索引將這些burn-in points取走：

```julia
x = x[1+skip, n]
y = y[1+skip, n]
```

這樣得到的結果就會是在有burn-in points之下我們要的資料了。最後，我們就用`@manipulate` 建立一個具有互動功能的可視化圖型。

```julia
using Distributions, Plots, LinearAlgebra, Statistics, Interact, WebIO

@manipulate for n=100:2000, ρ = -0.9:.1:0.9, μy = 0, σx = 1
		rng = Xoshiro(1234)
    skip_num = 100
    init = [4,3]
    x = ones(1+skip_num+n) * init[1]
    y = ones(1+skip_num+n) * init[2]

    for i in range(2,1+skip_num+n)
            x[i] = rand(rng, Normal(ρ * σx * (y[i-1] - μy), σx^2 * (1 - ρ ^ 2)))
            y[i] = rand(rng, Normal(μy + ρ * (1 / σx) * x[i], 1 - ρ ^ 2))
    end
    deleteat!(x, 2:skip_num)
    deleteat!(y, 2:skip_num)
    # Create the suplot in the layout
    layout = @layout [a{0.6w,0.4h} _
                      b{0.6w,0.6h} c{0.4w, 0.6h}]
    
    # Set plot default
    default(fillcolor=:lightgrey, markercolor=:white, grid=false, legend=false)
    plot(layout=layout, link=:both, size=(400, 400),  margin=-10Plots.pt)
    
    # Show scatter plot, located in position 2
    scatter!(x,y, subplot=2, framestyle =:box)
    # Show scatter plot, located in position 1 and 3, and make position 3 rotate through 90 degree
    histogram!([x y], subplot=[1 3], orientation=[:v :h], framestyle=:none, bins=min(n, 100), normalize=true)
end
```

注意到我們在程式碼上加上了`rng = Xoshiro(1234)` 的功能，便是讓結果可以是「疊加」的：當 $n$ 增加時，圖型會如何改變。