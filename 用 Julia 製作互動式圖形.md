# 用 Julia 製作互動式圖形

###### tags: `Julia`

## 互動式功能：`@manipulate`

$\texttt{Julia}$ 在互動性上做得十分完善，搭配 Jupyter notebook 可堪稱是完美。而我們可以利用 `@manipulate` 這個功能建立互動式圖形。首先我們要先安裝能夠支援我們做出來的互動式圖形呈現在網頁上的套件：`Interact` 與 `WebIO`。安裝並引用套件之後，就可以來進行簡單的操作。

### 基本架構

我們可以試著寫一個「隨著 $n$ 大小改變斜率」的直線。

```=julia
using Interact, Plots

@manipulate for n=1:0.1:100
    x = range(0.0, 1.0, length = 100)
    plot(x, x -> n * x, ylim = [-10,50])
end
```

![](https://i.imgur.com/7MrvoQo.gif)


其中，`@manipulate` 代表撰寫程式者告訴系統允許使用者可以進行「操控」，畫面會出現滑桿(slider)，即下圖所示。而後方的 `for` 迴圈代表我們的滑桿可移動的範圍(range)以及步長(step)。而在 `plot()` 中的 `y` 值是透過指派函式的方式，也就是根據 `x` 產生 `y`。 

![](https://i.imgur.com/uFSTbjV.png)

### 產生動圖：`@animate` 與 `gif()`

產生上面的互動式圖表後，如果想要將這個圖表變成 `.gif` 的格式嵌入網頁或是不用透過手動調整滑桿。我們可以使用 `@animate` 這個指令先產生會自己運行的圖，接下來用 `gif()` 這個指令將畫好的圖用類似 `savefig()` 的方式儲存下來；其實不用太複雜的程式碼，我們只要將上面的 `@manipulate` 換掉即可。以下的動圖程式碼是上述圖表的程式碼：

```julia
using Interact, Plots

p = @animate for n=1:0.1:100
    x = range(0.0, 1.0, length = 100)
    plot(x, x -> n * x, ylim = [-10,50])
end

gif(p, "plot.gif", dpi = 1200)
```

其中 `gif()` 中，我們將指派(assign)的變數 `p` 儲存成 `"plot.gif"` 的檔案。而如果說我們想要將畫質(quality)調高呢？我們可以使用 `dpi` 這個指令進行設定，其最高值為 $4000$。

## 利用 `@layout` 呈現子圖(subplot)

製作完上述的圖表之後，我們可以得到很多張圖表。但是！假設我們今天想要將不同的圖表擺在一起進行比較，要怎麼辦呢？給予一個直觀的想法：我們從上一篇文章到這篇文章所繪製的圖表都是在**一張畫布(canva)** 上進行繪圖；那麼，其也可以從一張畫布被**分割(split)** 為許多張小畫布。

### 畫布設定

我們可以先畫一張空白的圖：

```julia
using Interact, Plots

plot()
```

![](https://i.imgur.com/n8GLgwS.jpg)


接著，我們就來設定要如何分割畫布。在 `plot()` 中，`layout` 這個指令是用來設定子圖應該如何呈現的。我們先來看看以下的程式碼：

```julia
layout = @layout [a{0.5w,0.5h} b{0.5w,0.5h}
                  c{0.5w,0.5h} d{0.5w, 0.5h}]
```

其中，`@layout` 用來進行初始設定，後方則是一個容納(contain)子圖高度(height)與寬度(width)的矩陣(matrix)。而原始的畫布寬度是 $1$，因此，當我們在設定做子圖時，便可透過比例的方式指定高度與寬度。

```julia
using Interact, Plots

layout = @layout [a{0.5w,0.5h} b{0.5w, 0.5h}
                  c{0.5w,0.5h} d{0.5w, 0.5h}]

plot(layout = layout)
```

![](https://i.imgur.com/TqII0lp.jpg)

可以看到，矩陣內的每個位置都與下圖的子圖位置相對應。我們可以將其無限分割（當然，如果你覺得你或是讀者的眼力夠好的話），擺放許多張子圖。如果我們只要擺放 $3$ 張圖，我們可以在矩陣裡的相對位置上放上 `_`，代表那個位置不要產生子圖。


![](https://i.imgur.com/fiNB6ih.jpg)

### 繪製子圖

有了上面的畫布之後，我們就可以將圖表畫上去了。還記得我們之前在使用疊圖(overlay)的指令嗎？沒錯，就是 `plot!()`，當然也可以是 `scatter!()` 或是 `histogram!()`。我們以[吉布斯採樣(Gibbs sampling)](https://hackmd.io/@yueswater/GibbsSampling)裡面的圖表作為例子。

```julia
using Distributions, Plots, LinearAlgebra, Statistics, Interact, WebIO

## step 1: create sliders for n, ρ, μy and σx
@manipulate for n=100:2000, ρ = -0.9:.1:0.9, μy = 0, σx = 1
## step 2: set default random seed and initial numbers of x and y
    rng = Xoshiro(1234)
    skip_num = 100
    init = [4,3]
    x = ones(1+skip_num+n) * init[1]
    y = ones(1+skip_num+n) * init[2]
## step 3: create a matrix for x and y
    for i in range(2,1+skip_num+n)
            x[i] = rand(rng, Normal(ρ * σx * (y[i-1] - μy), σx^2 * (1 - ρ ^ 2)))
            y[i] = rand(rng, Normal(μy + ρ * (1 / σx) * x[i], 1 - ρ ^ 2))
    end
    deleteat!(x, 2:skip_num)
    deleteat!(y, 2:skip_num)
## step 4: create the suplot in the layout
    layout = @layout [a{0.6w,0.4h} _
                      b{0.6w,0.6h} c{0.4w, 0.6h}]
    
## step 5: set plot default
    default(fillcolor=:lightgrey, markercolor=:white, grid=false, legend=false)
    plot(layout=layout, link=:both, size=(400, 400),  margin=-10Plots.pt)
    
    # Show scatter plot, located in position 2
    scatter!(x,y, subplot=2, framestyle =:box)
    # Show scatter plot, located in position 1 and 3, and make position 3 rotate through 90 degree
    histogram!([x y], subplot=[1 3], orientation=[:v :h], framestyle=:none, bins=min(n, 100), normalize=true)
end
```

我們來一步一步解析上述的程式碼中繪圖的部分，針對步驟 $1$ 到 $3$，請參考上面提供的文章。

```julia
layout = @layout [a{0.6w,0.4h} _
                      b{0.6w,0.6h} c{0.4w, 0.6h}]
```

同上述設定畫布的方式相同，此處就不再贅述。

```julia
default(fillcolor=:lightgrey, markercolor=:white, grid=false, legend=false)
```

我們透過 `default()` 進行圖片的基本設定，`fillcolor=:lightgrey` 代表我們將圖片的顏色全部設定為淺灰色；`markercolor=:white` 則是將標點(marker)設定為白色；`grid=false` 則是取消格線(grid)，而 `legend` 則是取消