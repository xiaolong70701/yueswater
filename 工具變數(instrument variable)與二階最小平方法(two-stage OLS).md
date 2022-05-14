# 工具變數(instrument variable)與二階最小平方法(two-stage OLS)

###### tags: `Stats`

之前我們提到了最小平方法(OLS)，簡單來說，為了要找出最能夠配適(fit)所有樣本點的一條線性迴歸，我們透過將殘差平方和取一階條件的方法求出參數 $\beta_0$、$\beta_1 \cdots$。另外，我們也提到了二元變數的設定以及使用。而我們現在要著重在工具變數與二階最小平方法上。考慮以下關於美國勞動人口的「薪資方程式」(wage equation)：[^1]

$$
\log(\operatorname{wage}_i) = \beta_0 + \beta_1 \operatorname{exper}_i + \beta_2 \operatorname{exper}^2_i + \beta_3 \operatorname{educ}_i + \varepsilon_i
$$

其中， $\operatorname{wage}_i$ 代表每個人每月的薪資、$\operatorname{exper}_i$ 為每人的工作經驗年數，而 $\operatorname{edue}_i$ 代表其受教育的年數。首先我們可以在 $\texttt{R}$ 裡下載 `wooldridge` 這個資料庫，然後設定資料為 `wage2`。

```r
##   wage hours  IQ KWW educ exper tenure age married black south urban sibs brthord meduc
## 1  769    40  93  35   12    11      2  31       1     0     0     1    1       2     8
## 2  808    50 119  41   18    11     16  37       1     0     0     1    1      NA    14
## 3  825    40 108  46   14    11      9  33       1     0     0     1    1       2    14
## 4  650    40  96  32   12    13      7  32       1     0     0     1    4       3    12
## 5  562    40  74  27   11    14      5  34       1     0     0     1   10       6     6
## 6 1400    40 116  43   16    14      2  35       1     1     0     1    1       2     8

##   feduc    lwage
## 1     8 6.645091
## 2    14 6.694562
## 3    14 6.715384
## 4    12 6.476973
## 5    11 6.331502
## 6    NA 7.244227
```

接著我們就可以來進行迴歸：

```r
## Call:
## lm(formula = lwage ~ exper + I(exper^2) + educ, data = wage2)

## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.86696 -0.23886  0.03574  0.26112  1.30357 

## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 5.517432   0.124819  44.204   <2e-16 ***
## exper       0.016256   0.013540   1.201    0.230    
## I(exper^2)  0.000152   0.000567   0.268    0.789    
## educ        0.077987   0.006624  11.773   <2e-16 ***
## ---
## Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

## Residual standard error: 0.3932 on 931 degrees of freedom
## Multiple R-squared:  0.1309,	Adjusted R-squared:  0.1281 
## F-statistic: 46.75 on 3 and 931 DF,  p-value: < 2.2e-16
```

我們感興趣的變數為 `educ`，探討當其受教育年數增加時，其薪資的對數是否會增加。而我們估計出的係數為 $0.077987$，代表當其受教育年數越多時，薪資亦會相應的增加。但是，根據之前的假設，當我們的 OLS 估計式有一致性(consistency)時，

$$
\mathbb{E}(\mathbf{x}_i\varepsilon_i) = 0
$$

因此，對於我們的樣本而言，我們希望下列式子成立：

$$
\mathbb{E}(\operatorname{exper}_i\varepsilon_i) = \mathbb{E}(\operatorname{exper}^2_i\varepsilon_i) = \mathbb{E}(\operatorname{educ}_i\varepsilon_i) = 0
$$

但是，一般咸認，除了教育年數之外，一個人的能力、溝通能力、家庭背景等等皆會影響其收入，使得

$$
\mathbb{E}(\operatorname{educ}_i\varepsilon_i) \neq 0
$$

因此，「市儈」的迴歸模型應該是

$$
\log(\operatorname{wage}_i) = \beta_0 + \beta_1 \operatorname{exper}_i + \beta_2 \operatorname{exper}^2_i + \beta_3 \operatorname{educ}_i + \beta_4 \operatorname{ability}_i + u_i
$$

但因為「能力」這個特質是**難以量測**的，使得 $\varepsilon_i = \beta_4 \operatorname{ability}_i + u_i$，也因此

$$
\operatorname{cov}(\operatorname{educ}_i, \operatorname{ability}_i) > 0 \Rightarrow \mathbb{E}(\operatorname{educ}_i\varepsilon_i) \neq 0
$$

## 內生性(endogeneity)
在線性迴歸模型中，若解釋變數與誤差項的期望值不是 $0$，那麼我們就說該變數具有內生性，或稱該變數為**內生變數(endogenous variable)**，即

$$
y_i = \mathbf{x}_i^{\top}d\beta + \varepsilon_i, \; \mathbb{E}(\mathbf{x}_i\varepsilon_i) \neq 0
$$

而由於內生性，OLS 估計式也就不具有一致性：

$$
\begin{aligned}
\hat{\beta}-\beta &=\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1} \frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \epsilon_{i} \\
& \stackrel{p}{\to}\mathbb{E}\left(\mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1} \mathbb{E}\left(\mathbf{x}_{i} \epsilon_{i}\right) \\
& \neq \mathbf{0} .
\end{aligned}
$$

在實務上，內生性的問題隨處可見，會導致內生性有以下幾個原因：

- 遺漏變數(omitted varibale)：如果我們應該將某個變數放進模型中，但卻遺漏它，就會導致變數具有內生性。例如上述的能力，因為我們無法量測它，就沒有該筆資料，也就沒有理由將其納入模型當中。
- 測量誤差(measurement error)：在測量變數時產生誤差。
- 同時性(simultaneity)：在模型中，至少有一個解釋變數是與被解釋變數同時被決定的。

### 測量誤差(measurement error)

假設真實的資料產生機制如下：

$$
y_{i}=\mathbf{x}_{i}^{* \top} \beta+u_{i}, \; \mathbb{E}\left(\mathbf{x}_{i}^{*} u_{i}\right)=\mathbf{0}
$$

但是，由於某些原因，使得 $\mathbf{x}_i = \mathbf{x}_i^* + \mathbf{v}_i$，而其中 $\mathbf{v}_i$ 就是我們無法得知的測量誤差，因此，上述機制可以改寫為

$$
y_{i}=\mathbf{x}_{i}^{* \top} \beta+u_{i}=\left(\mathbf{x}_{i}-\mathbf{v}_{i}\right)^{\top} \beta+u_{i}=\mathbf{x}_{i}^{\top} \beta+\left(u_{i}-\mathbf{v}_{i}^{\top} \beta\right)
$$

因此，如果我們建立了一個如下的模型：

$$
y_i = \mathbf{x}^{\top}_i \beta + \varepsilon_i
$$

$\varepsilon_i$ 必定與 $\mathbf{x}_i$ 有關，因為

$$
\varepsilon_i = u_{i}-\mathbf{v}_{i}^{\top} \beta
$$

又，

$$
\begin{aligned}
\mathbb{E}\left(\mathbf{x}_{i} \varepsilon_i{i}\right) &=\mathbb{E}\left[\left(\mathbf{x}_{i}^{*}+\mathbf{v}_{i}\right)\left(u_{i}-\mathbf{v}_{i}^{\top} \beta\right)\right] \\
&=\mathbb{E}\left(\mathbf{x}_{i}^{*} u_{i}\right)-\mathbb{E}\left(\mathbf{x}_{i}^{*} \mathbf{v}_{i}^{\top}\right) \beta+\mathbb{E}\left(\mathbf{v}_{i} u_{i}\right)-\mathbb{E}\left(\mathbf{v}_{i} \mathbf{v}_{i}^{\top}\right) \beta
\end{aligned}
$$

無論前三項是否為 $0$，由於最後一項為測量誤差 $\mathbf{v}_i$ 的共變數矩陣，其必不等於 $0$，因此 $\mathbb{E}(\mathbf{x}_{i} \varepsilon_i)$ 一定不會是 $0$。假設我們考慮一個人的智商(intelligence quality, IQ) 會影響其所得，

```r
##
## Call:
## lm(formula = lwage ~ exper + I(exper^2) + educ + IQ, data = wage2)
##
## Residuals:
## Min 1Q Median 3Q Max
## -1.93666 -0.21283 0.02115 0.26028 1.26089
##
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)
## (Intercept) 5.2139647  0.1329475  39.218  < 2e-16 ***
## exper       0.0157135 0.0133006 1.181 0.238
## I(exper^2)  0.0001646 0.0005569 0.295 0.768
## educ        0.0573256 0.0073883 7.759 2.25e-14 ***
## IQ          0.0057867 0.0009801 5.904 4.97e-09 ***
## ---
## Signif. codes: 0 ’***’ 0.001 ’**’ 0.01 ’*’ 0.05 ’.’ 0.1 ’ ’ 1
##
## Residual standard error: 0.3863 on 930 degrees of freedom
## Multiple R-squared: 0.1623, Adjusted R-squared: 0.1587
## F-statistic: 45.05 on 4 and 930 DF, p-value: < 2.2e-16
```

可以發現 `educ` 這個變數的係數變為 $0.077987$。不過，一個人的能力並不只是被智商所決定，有可能受到原生家庭、成長背景、同儕相處等因素影響，其並不能完全代表這個人的能力之所在。

### 同時性(simultaneity)

舉例來說，我們關心一座城市的謀殺案的犯罪率 $y$，而解釋變數為該城市的警察數 $x$。直觀上來說，一座城市的如果駐有許多警察，犯罪率便會降低；但是反過來想，為什麼一座城市會需要如此多的警力呢？是否是因為該城市本身的犯罪率極高，導致政府不得不增派警力支援，進而防止犯罪。或是在經濟學的供需模型中，如果我們想要討論價格 $y$ 與產量 $x$ 之間的關係，應該會知道價格與產量是由供給與需求**同時決定**的，也因此迴歸模型中的解釋變數會具有內生性。

## 工具變數估計

我們的模型如下：

$$
y_i = \mathbf{x}^{\top}_i \beta + \varepsilon_i
$$

其中 $\mathbf{x}_i$ 為一個 $(k + 1) \times 1$ 的矩陣，括號中的 $k$ 代表解釋變數，$1$ 則為截距項。而在 $\mathbf{x}_i$ 中，有一些具有內生性的變數，使得 $\mathbb{E}(\mathbb{x}_i\varepsilon_i) \neq 0$。以下我們就要來設法（或說使用一些**技術性的**方法）試圖解決變數具有內生性的問題。假設存在另一個向量 $\mathbf{z}_i$，其維度(dimension)與 $\mathbf{x}_i$ 相同，但是其能夠使 $\mathbb{E}(\mathbb{z}_i\varepsilon_i) = 0$，且 $\operatorname{cov}(\mathbf{z}_i, \mathbf{x}_i) \neq 0$，而這個 $\mathbf{z}_i$ 我們稱之為**工具變數**。在處理 OLS 估計式時，我們會得到一組正規方程組，其中第一項可以寫成

$$
\mathbf{X}^{\top}(\mathbf{y} - \mathbf{X}\beta) = \sum^n_{i=1}\mathbf{x}_i (y_i - \mathbf{x}^{\top}_i \beta) = 0
$$

而工具變數估計則是要處理：

$$
\mathbf{Z}^{\top}(\mathbf{y} - \mathbf{X}\beta) = \sum^n_{i=1}\mathbf{z}_i (y_i - \mathbf{x}^{\top}_i \beta) = 0
$$


如果 $\mathbf{Z}^{\top}\mathbf{X}$ 具有滿秩的性質，那麼工具變數估計式就可以寫成

$$
\hat{\beta}_{\operatorname{IV}}=\left(\mathbf{Z}^{\top} \mathbf{X}\right)^{-1} \mathbf{Z}^{\top} \mathbf{y}=\left(\sum_{i=1}^{n} \mathbf{z}_{i} \mathbf{x}_{i}^{\top}\right)^{-1} \sum_{i=1}^{n} \mathbf{z}_{i} y_{i}
$$


### IV 估計式的一致性

回到薪資方程式的例子：

$$
\log(\operatorname{wage}_i) = \beta_0 + \beta_1 \operatorname{exper}_i + \beta_2 \operatorname{exper}^2_i + \beta_3 \operatorname{educ}_i + \varepsilon_i
$$

假設我們認為一名勞工的教育年數與其母親受教育的年數 $\operatorname{meduc}_i$ 有關，而母親受教育的年數與該名勞工之所得並無直接相關，不過當母親受教育時間越長，通常小孩的受教育年數也會越長。因此，

$$
y_{i}=\log \left(\operatorname{wage}_{i}\right), \;\mathbf{x}_{i}=\begin{bmatrix}
1 \\
\operatorname{exper}_{i} \\
\operatorname{exper}_{i}^{2} \\
\operatorname{educ}_{i}
\end{bmatrix}, \; \mathbf{z}_{i}=
\begin{bmatrix}
1 \\
\operatorname{exper}_{i} \\
\operatorname{exper}_{i}^{2} \\
\operatorname{meduc}_{i}
\end{bmatrix}
$$

我們可以將 IV 估計式寫成下面的形式，根據弱大數法則與連續映射定理，其會機率收斂到
$$
\begin{aligned}
\hat{\beta}_{\operatorname{IV}}-\beta &=\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{z}_{i} \mathbf{x}_{i}^{\top}\right)^{-1} \frac{1}{n} \sum_{i=1}^{n} \mathbf{z}_{i} \epsilon_{i} \\
& \stackrel{p}{\rightarrow}\left[\mathbb{E}\left(\mathbf{z}_{i} \mathbf{x}_{i}^{\top}\right)\right]^{-1} \mathbb{E}\left(\mathbf{z}_{i} \epsilon_{i}\right)=\mathbf{0}
\end{aligned}
$$

代表 IV 估計式具有一致性。那麼，IV 估計式的期望值是什麼呢？其是否為不偏的？大抵上來說，需要 IV 的原因在於 $\operatorname{cov}(\mathbf{x}_i, \varepsilon_i) \neq 0$，其影射了 $\mathbb{E}(\varepsilon|\mathbf{X}) \neq 0$。既然 $\operatorname{cov}(\mathbf{z}_i, \varepsilon_i) = 0$，那麼我們可以大膽假設 $\mathbb{E}(\varepsilon|\mathbf{Z}) = 0$。但是，
$$
\begin{aligned}
&\mathbb{E}\left(\hat{\beta}_{\operatorname{IV}}-\beta\right)=\mathbb{E}\left[\mathbb{E}\left(\hat{\beta}_{\operatorname{IV}}-\beta \mid \mathbf{X}, \mathbf{Z}\right)\right] \\
&=\mathbb{E}\left[\mathbb{E}\left(\left(\mathbf{Z}^{\top} \mathbf{X}\right)^{-1} \mathbf{Z}^{\top} \epsilon \mid \mathbf{X}, \mathbf{Z}\right)\right]
\end{aligned}
$$

我們無法說明 $\mathbb{E}(\varepsilon|\mathbf{X}, \mathbf{Z})$ 是否為 $0$；又或是更進一步地說，若真的要去細究其之間的關係，我們必須對於上述三者的聯合機率分配(joint distribution) 給出強烈的假設，然而這並非我們採用工具變數的目的，故我們鮮少會去討論其不偏性。

### 二階最小平方法(two-stage OLS)

我們已經假設 $\mathbf{x}_i$ 與 $\mathbf{z}_i$ 的維度是 $(k + 1) \times 1$，但事實上，我們沒必要也沒有規定要假設 $\mathbf{z}_i$ 的維度如上。如果 $\mathbf{z}_i$ 的維度為 $(l + 1) \times 1$，其中 $l$ 可能異於 $k$，那麼 IV 估計式

$$
\mathbf{Z}^{\top}(\mathbf{y}-\mathbf{X} \beta)=\mathbf{0}
$$

就會有以下三種情況：

- $l = k$：剛好辨識(just-identified)，會有一組解。
- $l < k$：低度辨識(under-identified)，通常為無限多組解。
- $l > k$：過度辨識(over-identified)，通常為無解。

如果說我們採用的 IV 個數比解釋變數的個數還要來得多，一個解套的辦法是考慮以下的方程式：

$$
\mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top}(\mathbf{y}-\mathbf{X} \beta)=\mathbf{0}
$$

經過化簡，我們可得到

$$
\hat{\beta}_{\operatorname{2SLS}}=\left[\mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{X}\right]^{-1} \mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{y}
$$

得出的估計式稱為**兩階段最小平方估計式(TSLS)**。

#### 第一階段(first stage)

在第一階段，我們以 $\mathbf{X}$ 對 $\mathbf{Z}$ 進行迴歸，即

$$
\tilde{\mathbf{X}}=\mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{X}
$$

其中 $\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{X}$ 是 $\mathbf{X}$ 對 $\mathbf{Z}$ 進行迴歸時的 OLS 估計式。

#### 第二階段(second stage)

接下來，我們以 $\mathbf{y}$ 對 $\tilde{\mathbf{X}}$ 進行迴歸：

$$
\begin{aligned}
\hat{\beta}_{2 S L S}=&\left(\tilde{\mathbf{X}}^{\top} \tilde{\mathbf{X}}\right)^{-1} \tilde{\mathbf{X}}^{\top} \mathbf{y} \\
=& {\left[\mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{X}\right]^{-1} } \\
& \mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{y} \\
&=\left[\mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{X}\right]^{-1} \mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{y}
\end{aligned}
$$

直覺上來說，當 $l>k$ 時，我們有一個過度辨識(over-identified)的方程組。而要進行 IV 估計，一個簡單的方法是從這些 IV 中挑出一個與解釋變數最相關、最有關係的，並考慮一個所有 IV 的線性組合(linear combination)，也就是拿 $\mathbf{Z}$ 對 內生變數 $\mathbf{X}$ 進行迴歸。重新考慮上面的兩階段迴歸，並根據以下的性質：

$$
(\mathbf{ABC})^{-1} = \mathbf{C}^{-1}\mathbf{B}^{-1}\mathbf{A}^{-1}
$$

則兩階段迴歸可進一步寫成：

$$
\begin{aligned}
\hat{\beta}_{\operatorname{2SLS}} &= \left[\mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{X}\right]^{-1} \mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{y}\\
&= \left(\mathbf{Z}^{\top} \mathbf{X}\right)^{-1}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)\left(\mathbf{X}^{\top} \mathbf{Z}\right)^{-1} \mathbf{X}^{\top} \mathbf{Z}\left(\mathbf{Z}^{\top} \mathbf{Z}\right)^{-1} \mathbf{Z}^{\top} \mathbf{y} \\
&=\left(\mathbf{Z}^{\top} \mathbf{X}\right)^{-1} \mathbf{Z}^{\top} \mathbf{y}
\end{aligned}
$$

可以發現，$\hat{\beta}_{\operatorname{IV}}$ 是 $\hat{\beta}_{\operatorname{2SLS}}$ 的一個特例。 

### 如何執行 IV 估計

我們令 $\operatorname{meduc}_i$ 為該名勞工母親受教育的年數、$\operatorname{feduc}_i$ 則是父親的受教育年數，則我們可以重新寫下迴歸模型：

$$
y_{i}=\log \left(\operatorname{wage}_{i}\right), \;\mathbf{x}_{i}=\begin{bmatrix}
1 \\
\operatorname{exper}_{i} \\
\operatorname{exper}_{i}^{2} \\
\operatorname{educ}_{i}
\end{bmatrix}, \; \mathbf{z}_{i}=
\begin{bmatrix}
1 \\
\operatorname{exper}_{i} \\
\operatorname{exper}_{i}^{2} \\
\operatorname{meduc}_{i} \\
\operatorname{feduc}_{i}
\end{bmatrix}
$$

根據上面得出的計算方式，在第一階段中，我們要找到一個與 $\mathbf{x}_i$ 最相關的 $\mathbf{z}_i$，對於變數 $1$、$\operatorname{exper}_{i}$、$\operatorname{exper}_{i}^{2}$ 來說，它們自己就是最相關的變數；不過，對於 $\operatorname{educ}_{i}$，我們不清楚哪個對其而言是最相關的，因此可以拿 $\operatorname{educ}_{i}$ 對 $\mathbf{z}_i$ 進行迴歸，得到 $\tilde{\operatorname{educ}_{i}}$，並重新放回模型中進行迴歸。而具體上怎麼操作呢？我們可以先安裝 `AER` 這個套件(Applied Econometrics)，然後執行以下這行：

```r
library(AER)
summary(ivreg(lwage ~ exper + I(experˆ2) + educ,
              ~ exper + I(experˆ2) + meduc, wage2))

##
## Call:
## ivreg(formula = lwage ~ exper + I(exper^2) + educ | exper + I(exper^2) +
## meduc, data = wage2)
##
## Residuals:
##      Min       1Q  Median      3Q     Max
## -1.70244 -0.25416 0.02051 0.26390 1.36063
##
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)
## (Intercept) 4.3946778  0.3596507  12.219  < 2e-16 ***
## exper       0.0170421  0.0152462    1.118    0.264
## I(exper^2)  0.0009528  0.0006718    1.418    0.156
## educ        0.1518448  0.0229241    6.624 6.19e-11 ***
## ---
## Signif. codes: 0 ’***’ 0.001 ’**’ 0.01 ’*’ 0.05 ’.’ 0.1 ’ ’ 1
##
## Residual standard error: 0.4121 on 853 degrees of freedom
## Multiple R-Squared: 0.03018, Adjusted R-squared: 0.02676
## Wald test: 15.11 on 3 and 853 DF, p-value: 1.371e-09
```

其中 `ivreg` 代表我們進行工具變數的迴歸，而括號內的第一行代表我們的薪資方程式，而第二行則代表第一階段迴歸。可以發現 `educ` 的參數變得更大了。另外，我們可以考慮兩個 IV：

```r
summary(ivreg(lwage ~ exper + I(experˆ2) + educ,
~ exper + I(experˆ2) + meduc + feduc, wage2))
##
## Call:
## ivreg(formula = lwage ~ exper + I(exper^2) + educ | exper + I(exper^2) +
## meduc + feduc, data = wage2)
##
## Residuals:
##      Min       1Q  Median      3Q     Max
## -1.70736 -0.26176 0.01261 0.26410 1.36497
##
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)
## (Intercept) 4.5228705  0.3108540  14.550  < 2e-16 ***
## exper       0.0082983  0.0162034   0.512   0.6087
## I(exper^2)  0.0013114  0.0007179   1.827   0.0682 .
## educ        0.1457095  0.0196955   7.398  3.87e-13 ***
## ---
## Signif. codes: 0 ’***’ 0.001 ’**’ 0.01 ’*’ 0.05 ’.’ 0.1 ’ ’ 1
##
## Residual standard error: 0.4122 on 718 degrees of freedom
## Multiple R-Squared: 0.03813, Adjusted R-squared: 0.03411
## Wald test: 18.81 on 3 and 718 DF, p-value: 9.374e-12
```

而回憶我們之前在進行 OLS 估計時，根據中央極限定理，

$$
\frac{1}{\sqrt{n}} \sum_{i=1}^{n} \tilde{\mathbf{x}}_{i} \varepsilon_{i} \stackrel{A}{\sim} \mathcal{N}\left(\mathbf{0}, \sigma^{2} \tilde{\mathbf{M}}\right)
$$

其中 $\sigma^2 = \mathbb{E}(\varepsilon_i^2)$，而 $\tilde{\mathbf{M}} = \mathbb{E}(\tilde{\mathbf{x}}_{i}\tilde{\mathbf{x}}_{i}^{\top})$。則

$$
\begin{aligned}
\sqrt{n}\left(\hat{\beta}_{\operatorname{2SLS}}-\beta\right) &=\left(\frac{1}{n} \sum_{i=1}^{n} \tilde{\mathbf{x}}_{i} \tilde{\mathbf{x}}_{i}^{\top}\right)^{-1} \frac{1}{\sqrt{n}} \sum_{i=1}^{n} \tilde{\mathbf{x}}_{i} \varepsilon_{i} \\
& \stackrel{A}{\sim} \mathcal{N}\left(\mathbf{0}, \sigma^{2} \tilde{\mathbf{M}}^{-1}\right)
\end{aligned}
$$

對於 $\tilde{\mathbf{M}}$，我們可以利用 $n^{-1}\sum^n_{i=1}\tilde{\mathbf{x}}_{i}\tilde{\mathbf{x}}_{i}^{\top}$ 進行估計，而通常來說對於不知道的 $\sigma^2$，使用 樣本變異數估計：

$$
s^{2}=\frac{1}{n-k-1} \sum_{i=1}^{n} \hat{\varepsilon}_{i}^{2}
$$

其中 $\hat{\epsilon}_{i}=y_{i}-\tilde{\mathbf{x}}_{i}^{\top} \hat{\beta}_{\operatorname{2SLS}}$。

### 找到一個好的 IV

在實證研究中，要能夠找到一個能夠跟解釋變數高度相關，且與誤差項沒有關係的 IV 是很困難的。但是，由於誤差項是我們看不見摸不著的東西，我們很難去量測經過選擇的 IV 是否跟誤差項沒有任何的關係。不過一個想法是，因為 IV 跟內生變數有關（是**唯一**影響該內生變數的管道），而內生變數又與被解釋變數有關，因此 IV 必定與被解釋變數有關；但是驗證 IV 與內生變數是否有關就簡單多了。

```r
wage3 <- wage2[!is.na(wage2$meduc),]
cor(wage3$educ, wage3$meduc)
```

因為 `meduc` 的資料中有一些遺漏值(missing value)，我們可以用 `!is.na()` 將其排除。不過！母親受教育的年數不一定完全跟小孩受教育的年數與其薪資有關係。怎麼說呢？可以想像，該名母親受教育的年數可能與其自身的能力有關，而母親良好的能力又遺傳給孩子，就又回到我們最初提及的質疑：一個人的薪資除了與其受教育年數有關之外，更與其自身能力有關係。更進一地步說，在 $1960$ 年代，台灣普遍家庭的所得都不高，造成許多家庭的孩子無法受到良好的教育，但是這些家庭又有一些背景，例如他們家是田僑仔[^2]，或親戚從事高所得的工作，從而引薦孩子去那邊工作，使他們的薪水較高。話說回來，要怎麼找到一個好的 IV 呢？Wooldridge 教科書提到一個十分有用但也十分沒用的方法：
> Be clever!

不過我們可以參考其他經濟學者如何研究薪資的例子，看看他們怎麼找 IV 的。[Angrist and Krueger (1991)](https://uh.edu/~adkugler/Angrist&Krueger_CS.pdf) 提出了使用該名勞工於一年的哪個時期(quarter)出生的這個變數當作 IV。根據美國法律規定，義務教育年數為 $12$ 年，到了 $16$ 歲就可以輟學(dropout)出去工作。[^3] 而 Angrist 與 Krueger 認為，一個年級之中年紀最大的學生（大約是 $9/2$ 出生的）與另一個年級中年紀最小的學生（大約是 $9/1$ 出生的），前者受教育的年數會比較少，因為只要年紀一到，就可以輟學去工作，但是後者則還要在學校中待幾個月才能達到合法輟學、工作的年紀。

### 自然實驗(natural experiments)

要找到一個良好且有效(valid)的 IV，我們可以透過所謂的**自然實驗**來達成。[Angrist (1990)](https://sites.duke.edu/niou/files/2011/06/Angrist_lifetime-earningsmall.pdf) 在其研究越戰退伍軍人(Vietnam War veterans) 是否有得到國家補償的論文中提到，如果將當時參戰的年輕人與未參戰的年輕人的薪資進行比較，這個方法是有問題的。原因是此二類人在本質上便有所不同，例如男性參戰的可能性遠大於女性、參戰的男性為大學學歷的可能性很低（畢竟是在學），轉換為統計學的語言，就是**參戰不必然是外生變數**。而 Angrist 找到一個良好的 IV：當時美國政府為了徵召人民參戰，但由於志願者太少，於是就制定一項政策，即在電視上公開使用抽獎的方式，若其得到的號碼較小就必須參戰。而這個就是很純粹的自然實驗：與未來薪資無關，但與是否參戰有關。最終，Angrist 得出的結論為：退伍軍人的薪資較同年齡的人少約 $15\%$。

## 工具變數的更多例子

在底下的例子中，我們會使用 Scott Cunningham 的教科書 ***[Causal Inference: The Mixtape](https://mixtape.scunning.com/index.html)*** 中的第七章：工具變數。

![](https://i.imgur.com/f0chPBN.jpg)




[^1]:Wooldridge, 2002, 5.1, *Instrumental Variables for Education in a Wage Equation*

[^2]:[田僑仔](https://twblg.dict.edu.tw/holodict_new/result_detail.jsp?n_no=2003&curpage=0&sample=%E7%94%B0%E5%83%91%E4%BB%94&radiobutton=0&querytarget=0&limit=1&pagenum=0&rowcount=0)(*tshân-kiâu-á*)，代表擁有許多田地，靠地價暴漲而致富的土財主。

[^3]:Glavin, Chris. “[School Leaving Age.](https://www.k12academics.com/dropping-out/school-leaving-age#:~:text=In%20the%20United%20States%2C%20most,at%20ages%2016%20and%2017)” School Leaving Age | K12 Academics, February 6, 2014. 