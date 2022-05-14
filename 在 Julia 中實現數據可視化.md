# 在 Julia 中實現數據可視化

###### tags: `Julia`

隨著大數據時代的來臨，越來越多公司或學術領域的資料科學家紛紛開始使用數據可視化(data visualization)。那什麼是數據可視化呢？如果今天在一場關於台灣GDP的演講上，你看到的是

|    | data.0.x |       data.0.y      | data.1.x |       data.1.y      |
|:--:|:--------:|:-------------------:|:--------:|:-------------------:|
|  0 | 1951     | 0.14894134825991726 | 1951     | 0.1019712825504989  |
|  1 | 1952     | 0.14118668596237338 | 1952     | 0.0802315484804631  |
|  2 | 1953     | 0.1376106963014412  | 1953     | 0.08612606355270012 |
|  3 | 1954     | 0.147777997234841   | 1954     | 0.06454671143590757 |
|  4 | 1955     | 0.12498756754964692 | 1955     | 0.08205417233033849 |
|  5 | 1956     | 0.15773534840793724 | 1956     | 0.08969773880941394 |
|  6 | 1957     | 0.14661274014155712 | 1957     | 0.09590865372758885 |
|  7 | 1958     | 0.16605125500021978 | 1958     | 0.10305947514176447 |
|  8 | 1959     | 0.20665232374388362 | 1959     | 0.12514517449498316 |
|  9 | 1960     | 0.1876202795217213  | 1960     | 0.11344922232387923 |
|  ... | ... | ... | ... | ...|

基本上你大概聽不到 $10$ 分鐘就會走人。但如果你看到的是這樣的

<iframe width="850" height="800" frameborder="0" scrolling="no" src="//plotly.com/~xiaolong70701/19.embed"></iframe>

可以一眼看出這張圖所要傳達給觀眾的是什麼。基本上我們可以將數據可視化想成是把一堆雜亂的數字變成簡約的圖形，而講者或報告者可利用這些圖形的樣式、顏色可以告訴觀眾背後的邏輯與故事。這也是為什麼可視化重要的原因之一。所以，今天我們就要來藉著這個機會來好好的說明在 $\texttt{Julia}$ 中的可視化。

## 使用 `Plots` 套件繪圖

首先要安裝並引用 `Plots` 套件。

```julia
using Pkg; Pkg.add("Plots")
using Plots

Updating registry at `C:\Users\user\.julia\registries\General.toml`
Resolving package versions...
No Changes to `C:\Users\user\.julia\environments\v1.7\Project.toml`
No Changes to `C:\Users\user\.julia\environments\v1.7\Manifest.toml`
```

由於我已經安裝過了，所以會顯示上面的資訊。

### 基本圖形

假設我們想要畫出以下函數的圖形

$$
f(x) = x^2 + 2x +1, \;x \in [0, 100]
$$

我們首先要釐清一件事情：圖形到底是怎麼產生的？基本上，我們要先有輸入 $x$，才會有輸出 $f(x)$。

<a href="https://commons.wikimedia.org/wiki/File:Function_machine2.svg#/media/File:Function_machine2.svg"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Function_machine2.svg/1200px-Function_machine2.svg.png" width = "" height = "400" alt="Function machine2.svg"></a>
    
因此我們要先利用 `collect()` 產生樣本點，接著定義函數，然後再將我們產生的樣本點放入函數中取得函數值，最後再進行繪圖。

```julia
using Plots
x = collect(1:1:100)
f(x) = x^2 + 2x +1
y = f.(x)
plot(x, y, xlim = [0, 100])
```
   
我們來解析一下上面這段程式碼：
- `collect()`：它是將指定的等差數列集合在一起形成向量的函式。如果輸入 `collect(1:10)`，就會產生一個 $1$ 到 $10$ 的向量；如果輸入 `collect(1:2:10)`，便會產生一個由奇數 $1,3,5,7,9$ 組成的向量，其中第二個冒號後面的數字稱為步長(step)，可以想像成為等差數列裡的公差(common difference)。
- `f.()`：使用 `f.()` 的原因是因為我們產生的樣本點為一個向量，故使用 dot syntax 的方式進行逐元(element-wise)的運算。
- `plot()`：其中 `x` 與 `y` 就分別對應上數的輸入值與函數值；而 `xlim = [<x_min>, <x_max>]` 則是設定圖形中 $x$ 的值域。同理，我們也可以設定 $y$ 的值域，`ylim = [<y_min>, <y_max>]`

如果我們不想要手刻 $x$ 的集合，可以用 `rand()` 產生隨機變數。

```julia
x = rand(10)
```

代表我們產生了 $10$ 個隨機變數。詳細的說明請參考[官網的文檔](https://docs.julialang.org/en/v1/stdlib/Random/)。我們也產生兩個函數圖形，

```julia
using Plots
x = 1:10; y = rand(10, 2) # 2 columns means two lines
plot(x, y)
```

![](https://i.imgur.com/UyqfEjR.png)


當然，我們也可以繪製一般常見函數的圖形，例如 $\log(x)$：

```julia
using Plots, LaTeXStrings
x = collect(1:1:100)
g(x) = log(x)
y = g.(x)
plot(x, y, xlim = [0, 100], title = L"\log(x)", label = L"\log(x)")
```
![](https://i.imgur.com/8Xrfpde.png)


這裡多了幾個新的東西！

- `using LaTeXStrings`：引用 `LaTeXStrings` 套件，支援 $\LaTeX$ 的字體。使用方法為在 `""` 之前加上 `L`，代表文字內要使用 $\LaTeX$ 字體。 
- `title = ""`：設定圖片標題。
- `label = ""`：設定圖例(legend)。



### 儲存圖片

想要將我們畫好的美圖存下來嗎？沒問題！`savefig()`可以幫你做到。我們只要在畫圖的指令下方打上

```julia
savefig("<檔案名稱>.<副檔名>")
```

就可以將圖片存下來囉。至於怎麼考慮副檔名又是個學問，可以參考[這篇文章](https://www.cool3c.com/article/146971#:~:text=%E7%B0%A1%E5%96%AE%E7%9A%84%E5%9C%96%E7%89%87%E8%BE%A8%E8%AD%98%E6%8C%87%E5%8D%97,%E8%A3%BD%E4%BD%9C%E5%B0%8F%E5%9E%8B%E5%8B%95%E6%85%8B%E5%9C%96%E7%89%87%E6%AA%94%E6%A1%88%E3%80%82)，裡面有詳細的解說。

### 統計圖表

#### 折線圖(line chart)

折線圖又可以稱為趨勢圖(trend graph)，也就是說它可以呈現資料隨著時間、劑量、年份 $\cdots$ 等的走勢。以下我們利用Plotly 官網所提供的[主要新聞來源資料](https://plotly.com/julia/line-charts/)。首先我們先進行基本設定。

```julia
title = "主要新聞來源"
labels = ["電視", "報紙", "網路", "廣播"]
colors = ["rgb(67,67,67)" "rgb(115,115,115)" "rgb(49,130,189)" "rgb(189,189,189)"]
linewidths = [2 2 4 2]

x_data = [
    2001 2002 2003 2004 2005 2006 2007 2008 2009 2010 2011 2012 2013
    2001 2002 2003 2004 2005 2006 2007 2008 2009 2010 2011 2012 2013
    2001 2002 2003 2004 2005 2006 2007 2008 2009 2010 2011 2012 2013
    2001 2002 2003 2004 2005 2006 2007 2008 2009 2010 2011 2012 2013
]

y_data = [
    74 82 80 74 73 72 74 70 70 66 66 69
    45 42 50 46 36 36 34 35 32 31 31 28
    13 14 20 24 20 24 24 40 35 41 43 50
    18 21 18 21 16 14 13 18 17 16 19 23
]
```

然後我們就可以利用上面設定好的數據進行繪圖。

```julia
using Plots, Dates

plot(Date.([x[1, :] x[2, :] x[3, :] x[4, :]]),
     [y[1, :] y[2, :] y[3, :] y[4, :]], 
     color = colors,
     linewidth = linewidths,
     label = labels,
     ylabel = "Percentage(%)"
     )
```

而 `Date` 這個指令是用來將數據變成日期的形式，使用的套件為 `Dates`。

![](https://i.imgur.com/gOQDsWW.png)

#### 直方圖(histogram)

直方圖可以說是統計圖表中最常見的一種類型，基本上直方圖在高中數學課本就已經出現過。那直方圖代表的是什麼意思呢？簡單來講，直方圖的橫軸代表數據的類型，縱軸代表數據的分佈情況。我們採用 *Introduction to Statistics and Data Analysis - with exercises, solutions and applications in R* 這本書裡面的[pizza delivery data](https://chris.userweb.mwn.de/book/pizza_delivery.csv)作為我們分析之用。
![](https://i.imgur.com/x1EDC1v.png)

##### $\texttt{pizza} 資料$
下載完畢後，我們可以來看一下資料的內容有哪些。我們可以打開 excel 或是將資料載入到目前 $\texttt{Julia}$ 工作的環境裡。這時候就要使用兩個套件： `CSV` 與 `DataFrame`。顧名思義， `CSV` 就是拿來讀取、處理 `.csv` 檔案類型的；而 `DataFrame` 則是用來處理表格。我們可以透過 `CSV.read()` 這個指令來讀取 `.csv` 檔案，格式為 

```julia
CSV.read("<要讀取的檔案>", DataFrame)
```

就是將檔案讀取進來之後，以 DataFrame 的方式呈現。

```julia
using CSV, DataFrames

df = CSV.read("/<儲存父目錄>/pizza_delivery.csv", DataFrame)
```


| day |	date|	time	|operator	|branch|	driver|	temperature|	bill|	pizzas|	free_wine|	got_wine|	discount_customer|
| -------- | -------- | -------- |-------- |-------- |-------- |-------- |-------- |-------- |-------- |-------- |-------- |


而這麼多資料當中，只有連續變數的資料才可以拿來進行繪製直方圖。我們這邊選擇時間 `time` 這筆資料。而在資料中，`time` 位於表格的第三欄(column)，因此我們可以先定位這一欄的位置：

```julia
df[!, 3]

## 1266-element Vector{Float64}:
## 35.12836705
## 25.20307368
## 45.64340414
## 29.3742975
  ⋮
## 36.88054621
## 35.73753383
```

其中，`df[]` 代表我們要索引(indexing)該筆資料，而 `!` 則是因為索引值的第一個參數為列(row)，但我們是要取整欄的資料，無論是哪一列。而 `3` 代表第三欄。接著使用 `histogram()` 這個指令來繪製直方圖。

```julia
histogram(df[!, 3], label = L"\texttt{time}")
```

![](https://i.imgur.com/G9KL5bI.png)


我們就得到了 `time` 這筆資料的直方圖。


##### 中央極限定理(central limit theorem, CLT)
在統計學中，這個重要的定理代表著：隨著樣本數趨近於無限，**相互獨立且來自同分配(independent and identically distributed, i.i.d)** 的樣本會**分配收斂**到常態分配。假設給定一組隨機變數 $x$ 服從常態分配，那麼只要樣本數夠大，我們就可以看到一開始的圖形不會是常態分配的圖形，不過只要樣本數越大，就會越接近常態分配，也就是一個鐘型(bell-shape)的線。我們可以利用上面學到的 `rand()` 指令來產生隨機變數，接著看看他會長什麼樣子。而這邊我們可以利用另一個套件 `Distributions` 來產生常態分配的PDF，看看當樣本數越大的時候，是不是真的會符合中央極限定理。

```julia
using Plots, LaTeXStrings

N = <自行調整> # 設定樣本數
μ = 0; σ = 1 # 設定 μ 與 σ
d = Normal(μ, σ^2) # 設定常態分配
sample = rand(d,N) # 建立樣本

histogram(sample, normalize=true, bins=100, xlim = [μ-4σ, μ+4σ], ylim = [0, 0.5], title = L"n=10000")  # hjw
plot!(d, linewidth = 3)
```

注意到上面的 `plot!()` 加上 `!` 的目的是為了進行疊圖(overlay)。 

![](https://i.imgur.com/cGIT1qF.png)

我們透過動圖來看看其是如何隨著樣本數的增加，分配收斂到常態分配的。

![](https://i.imgur.com/9vHBNhv.gif)

#### 長條圖(bar chart)

不同於直方圖，雖然長條圖長得很像直方圖，但還是有些差別。上面提到，直方圖是用於描述連續的資料，而長條圖是用來描述離散的資料。在 `pizza` 的資料裡面，`branch` 與 `driver` 都是連續的資料，我們就以 `branch` 作為我們分析之用。讀取資料的方式跟剛剛一樣，但這次我們要用另一個套件處理資料，因為 `branch` 這筆資料是描述哪間分店處理訂單的，因此我們畫出的長條圖應該是呈現不同分店的頻次(frequency)。

```julia
using FreqTables
pizza_freq = freqtable(df[!, 5])

## 3-element Named Vector{Int64}
## Dim1     │ 
## ─────────┼────
## "Centre" │ 421
## "East"   │ 410
## "West"   │ 435
```

使用 `FreqTables` 套件就可以做出頻次表了。接著利用上面索引值的技巧，令 `centre`、`east` 與 `west` 變數分別是各個分店的頻次，然後使用 `Plots.bar()` 繪製長條圖。[^1]

```julia
centre = pizza_freq[1];
east = pizza_freq[2];
west = pizza_freq[3];

branch = ["Centre" "East" "West"]
freq = [centre east west]
colors = ["lightgray" "lightgray" "lightgray"]

Plots.bar(branch, freq, color = colors)
```

![](https://i.imgur.com/ML9A8rm.png)


#### 圓餅圖(pie chart)

圓餅圖是用來描述量、頻率或百分比之間的相對關係，因此適合呈現離散的資料。我們就延續長條圖的資料，只是我們這次要把 `freq` 後面以用逐元除法的方式除上總數，進而得到比例，再來用 `pie()` 繪製圓餅圖即可。

```julia
using CSV, DataFrames, Plots, LaTeXStrings, FreqTables

df = CSV.read("/Users/anthonysung/Julia tutorial/pizza_delivery.csv", DataFrame)
pizza_freq = freqtable(df[!, 5])
centre = pizza_freq[1];
east = pizza_freq[2];
west = pizza_freq[3];

branch = ["Centre", "East", "West"]
freq = [centre, east, west] ./ nrow(df)
pie(branch, freq)
```

![](https://i.imgur.com/N4dOaac.png)


#### 散佈圖(scatter graph)

通常我們會利用散佈圖來呈現一筆資料的相關性(correlation)，如果圖形偏向左上右下，我們會稱這是負相關(negative correlation)；而如果是偏向右上左下，則稱為正相關(positive correlation)；而如果呈現的圖形類似圓形，那麼則稱為無相關(uncorrelated)。以下我們使用來自 $\texttt{R}$ 的資料－「鳶尾花集(iris datasets)」。我們需要安裝 `RDatasets`，接著進行以下步驟：

```julia
using RDatasets
iris = dataset("datasets", "iris")
x = iris[!, 2]; y = iris[!, 1]
scatter(x, y, label = "")
```

![](https://i.imgur.com/YkGPsKl.jpg)


[^1]:用 `bar()` 這個指令似乎無法繪圖，系統會報錯，顯示找不到這個指令。