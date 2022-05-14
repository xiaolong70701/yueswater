# 迴歸分析中有關 OLS 的漸進理論

###### tags: `Stats`

上一章提到，如果我們的誤差項真的不是服從常態分配，我們仍可以透過漸進理論來說明：就算誤差項服從均勻分配、二項分配 $\cdots$，我們仍可以假設誤差項服從常態分配。根據Gauss-Markov定理，給定下列迴歸模型，我們可以假設誤差項服從常態分配：

$$
y_i = \beta_0 + \beta_1 x_{i1} + \beta_1 x_{i2} + \cdots + \beta_K x_{iK} + \varepsilon_i
$$
其中 $\{x_{i1}, x_{i2}, \cdots, x_{iK}\}$ 為非隨機的，且
$$
\varepsilon_i \overset{i.i.d}{\sim} \mathcal{N}\left(0, \sigma^2\right)
$$
在上述假設下，參數矩陣可以寫作

$$
\hat{\beta}=\left(\mathbf{X}^{\top} \mathbf{X}\right)^{-1} \mathbf{X}^{\top} \mathbf{y} \sim \mathcal{N}\left(\beta, \sigma^{2}\left(\mathbf{X}^{\top} \mathbf{X}\right)^{-1}\right)
$$

但如果上述假設不成立呢？我們可以來做一個實驗：考慮一個迴歸模型 $\mathbf{y} = \mathbf{X} \beta + \varepsilon$，其中

$$
\mathbf{X}=\left[\begin{array}{ll}
1 & 1 \\
1 & 2 \\
1 & 3
\end{array}\right],\; \beta=\left[\begin{array}{l}
2 \\
1
\end{array}\right],\; \varepsilon \sim \mathcal{N}(\mathbf{0}, 0.25 \times \mathbf{I})
$$

```r
Z1 <- numeric(5000)
X <- cbind(rep(1,3), (1:3))
Xinv <- solve(t(X) %*% X)
for (r in 1:5000) {
    Y <- X %*% c(2, 1) + 0.5 * rnorm(3)
    beta.h <- Xinv %*% t(X) %*% Y
    Z1[r] <- (beta.h[2] - 1) / (0.5 * sqrt(Xinv[2,2]))
}
c(mean(Z1<qnorm(0.025)),mean(Z1<qnorm(0.05)),mean(Z<qnorm(0.1)))

## [1] 0.0240 0.0500 0.1028
```

接著，我們可以拿對於誤差項進行 kernel denisty 作圖，並與 $\mathcal{N}(0,1)$ 的 PDF 進行比較。

```r
x <- seq(-4, 4, length = 1000)
y <- dnorm(x)
plot(density(Z1),
     main = "",
     xlab = "",
     ylab = "",
     xlim = c(-4, 4),
     ylim = c(0, 0.4)
    )
par(new=TRUE)
plot(x, y, type="l", lty=1, lwd = 0.5,
     main = "",
     xlab = "",
     ylab = "", 
     xlim = c(-4, 4), 
     ylim = c(0, 0.4)
    )
```
![](https://i.imgur.com/DVBRlNe.png)

不難發現其圖形與常態分配的圖十分接近。如果我們將同個迴歸模型之誤差項設定為服從均勻分配，我們來看看會發生什麼事。

$$
\mathbf{X}=\left[\begin{array}{ll}
1 & 1 \\
1 & 2 \\
1 & 3
\end{array}\right],\; \beta=\left[\begin{array}{l}
2 \\
1
\end{array}\right],\;
x_i \overset{i.i.d}{\sim} U(0,3),\;
\varepsilon_i \overset{i.i.d}{\sim} U(-\sqrt{3}/2,\sqrt{3}/2)
$$

```r
Z2 <- numeric(5000)
for (r in 1:5000) {
    X <- cbind(rep(1,3), 3 * runif(3))
    Xinv <- solve(t(X) %*% X)
    Y <- X %*% c(2, 1) + runif(3,-sqrt(3)/2,sqrt(3)/2)
    beta.h <- Xinv %*% t(X) %*% Y
    Z2[r] <- (beta.h[2] - 1) / (0.5 * sqrt(Xinv[2,2]))
}
c(mean(Z2<qnorm(0.025)),mean(Z2<qnorm(0.05)),mean(Z2<qnorm(0.1)))

## [1] 0.0168 0.0502 0.1156
```

重複上面比較的動作，我們可以得到：

```r
x <- seq(-4, 4, length = 1000)
y <- dnorm(x)
plot(density(Z2), 
     main = "", 
     xlab = "", 
     ylab = "", 
     xlim = c(-4, 4), 
     ylim = c(0, 0.4)
)
par(new=TRUE)
plot(x, y, type="l", lty=1, lwd = 0.5,
     main = "", 
     xlab = "", 
     ylab = "", 
     xlim = c(-4, 4), 
     ylim = c(0, 0.4)
)
```

![](https://i.imgur.com/0YLKDzn.png)


可以發現，如果誤差項服從非常態以外的分配，我們就沒有理由相信 $Z$ 亦服從常態分配。

## 漸進理論

其實，我們的假設看起來有點**硬要**，畢竟服從常態分配這件事情非常主觀。那麼有沒有其他方法能夠在不武斷地主張誤差項的分配之下，進一步證明誤差項服從常態分配呢？回想到統計學基礎課程提到的漸進理論，它告訴我們當我們適當樣本足夠大時，便能夠得到一收斂的分配。因此以下要來介紹下方分析會使用到的漸進理論。

### Markov 不等式
對於一個隨機變數 $X$ 與一個極小的 $\eta$，其中 $\eta > 0$，根據 Markov 不等式，

$$
\mathbb{P}(|X| \geq \eta) \leq \frac{\mathbb{E}(|X|)}{\eta}
$$

我們可以透過指標函數進行證明。給定 $\mathbb{I}(|X| \geq \eta) \leq|X| / \eta$，則

$$
\mathbb{P}(|X| \geq \eta)=\mathbb{E}[\mathbb{I}(|X| \geq \eta)] \leq \mathbb{E}\left(\frac{|X|}{\eta}\right)=\frac{\mathbb{E}(|X|)}{\eta}
$$

### Chebyshev 不等式

對於一個隨機變數 $X$ 與一個極小的 $\eta$，其中 $\eta > 0$ 且 $\mathbf{E}(X)$，根據 Chebyshev 不等式，

$$
\mathbb{P}\left(\left|X-\mu_{X}\right| \geq \eta\right) \leq \frac{\operatorname{var}(X)}{\eta^{2}}
$$

其來自 Markov 不等式，

$$
\begin{aligned}
\mathbb{P}\left(\left|X-\mu_{X}\right| \geq \eta\right) &=\mathbb{P}\left[\left(X-\mu_{X}\right)^{2} \geq \eta^{2}\right] \leq \frac{\mathbb{E}\left(\left|\left(X-\mu_{X}\right)^{2}\right|\right)}{\eta^{2}} \\
&=\frac{\operatorname{var}(X)}{\eta^{2}} .
\end{aligned}
$$

其所代表的是隨機變數 $X$ 不會離 $\mu$ 太遠。

### 弱大數法則

給定彼此無相關的隨機變數 $\{X_1, X_2, \cdots\}$，即對於所有的 $i \neq j$，$\operatorname{cov}(X_i, X_j) = 0$，且我們已知 $\mathbf{E}(X_i) = \mu_X$，而 $\operatorname{var}(X_i) \leq C < \infty$。則
$$
\bar{X}_{n}=\frac{1}{n} \sum_{i=1}^{n} X_{i} \stackrel{P}{\rightarrow} \mu_{X}
$$

其證明如下。$\bar{X_n}$ 的變異數可以透過下列方式計算，

$$
\begin{aligned}
\operatorname{var}\left(\bar{X}_{n}\right)=& \operatorname{var}\left(\frac{X_{1}}{n}+\frac{X_{2}}{n}+\frac{X_{3}}{n}+\cdots\right) \\
=& \frac{\operatorname{var}\left(X_{1}\right)}{n^{2}}+\frac{\operatorname{var}\left(X_{2}\right)}{n^{2}}+\frac{\operatorname{var}\left(X_{3}\right)}{n^{2}}+\cdots \\
&+\frac{\operatorname{cov}\left(X_{1}, X_{2}\right)}{n^{2}}+\frac{\operatorname{cov}\left(X_{1}, X_{3}\right)}{n^{2}}+\cdots \leq \frac{n \cdot C}{n^{2}}=\frac{C}{n} .
\end{aligned}
$$

而根據 Chebyshev 不等式，對於 $\varepsilon > 0$，

$$
\mathbb{P}\left(\left|\bar{X}_{n}-\mu_{X}\right| \geq \varepsilon\right) \leq \frac{\operatorname{var}\left(\bar{X}_{n}\right)}{\varepsilon^{2}} \leq \frac{C}{n \varepsilon^{2}} \rightarrow 0
$$

![](https://i.imgur.com/yVAm6wr.png)


弱大數法則[^1]告訴我們一件事：當我們樣本數越大的時候，樣本平均數便會越接近母體平均數。


### 中央極限定理(Central Limit Theorem, CLT)

令 $X_1, X_2, \cdots, X_n \overset{i.i.d}{\sim} (\mathbb{E}(X_i) = \mu_X, \operatorname{var}(X_i) = \sigma^2_X)$。則
$$
\frac{1}{\sqrt{n}} \sum_{i=1}^{n}\left(X_{i}-\mu_{X}\right) \stackrel{D}{\rightarrow} \mathcal{N}\left(0, \sigma_{X}^{2}\right)
$$

或可以寫作

$$
\frac{\sqrt{n}\left(\bar{X}_{n}-\mu_{X}\right)}{\sigma_{X}} \stackrel{D}{\rightarrow} \mathcal{N}(0,1) .
$$

注意到其是**分配收斂**到常態分配，而非服從之。我們可以利用動差生成函數(MGF)證明之。$\mathcal{N}(0,1)$ 的 MGF為

$$
M_Z(t) = \exp(\frac{t^2}{2})
$$
我們可以令 $W_i = n^{-1/2}(X_i - \mu_X)/\sigma_X$，則 $\mathbb{E}(W_i) = 0$，$\operatorname{var}(W_i) = 1/n$。因此 $W_i$ 的 MGF 為
$$
\begin{aligned}
M_{W_{i}}(t) & \approx M_{W_{i}}(0)+M_{W_{i}}^{(1)}(0) t+\frac{1}{2} M_{W_{i}}^{(2)}(0) t^{2} \\
&=1+\mathbb{E}\left(W_{i}\right) \cdot t+\frac{1}{2} \mathbb{E}\left(W_{i}^{2}\right) t^{2} \\
&=1+\frac{t^{2}}{2 n}
\end{aligned}
$$
而因為

$$
Z_{n}=\frac{1}{\sqrt{n}} \sum_{i=1}^{n} \frac{X_{i}-\mu_{X}}{\sigma_{X}}=\sum_{i=1}^{n} W_{i}
$$

故
$$
\begin{aligned}
M_{Z_{n}}(t) &=\mathbb{E}\left\{\exp \left[\left(\sum_{i=1}^{n} W_{i}\right) \cdot t\right]\right\} \\
&=\mathbb{E}\left[\exp \left(W_{1} t\right) \times \exp \left(W_{2} t\right) \times \cdots \times \exp \left(W_{n} t\right)\right] \\
&=\mathbb{E}\left[\exp \left(W_{1} t\right)\right] \times \mathbb{E}\left[\exp \left(W_{2} t\right)\right] \times \cdots \times \mathbb{E}\left[\exp \left(W_{n} t\right)\right] \\
&=\left[M_{W_{i}}(t)\right]^{n} \approx\left(1+\frac{t^{2}}{2 n}\right)^{n} \rightarrow \exp \left(\frac{t^{2}}{2}\right)
\end{aligned}
$$

CLT 告訴我們：無論我們的樣本服從怎樣的分配，樣本平均都會**趨近於服從**常態分配。

### 連續映射定理(Continuous Mapping Theorem, CMT)

令 $g(\cdot)$ 為一個連續函數(continuous function)，則
- 若 $X_{n} \stackrel{D}{\rightarrow} X$, 則 $g\left(X_{n}\right) \stackrel{D}{\rightarrow} g(X)$。
- 若 $X_{n} \stackrel{P}{\rightarrow} \alpha$, 則 $g\left(X_{n}\right) \stackrel{P}{\rightarrow} g(\alpha)$。

## 漸進理論的實際應用

相對於統計的古典假設，我們現在可以設定一個統計「現代」假設。對於 $i=1,2, \ldots, n$，
$$
y_{i}=\beta_{0}+\beta_{1} x_{i 1}+\beta_{2} x_{i 2}+\cdots+\beta_{K} x_{i K}+\epsilon_{i}=\mathbf{x}_{i}^{\top} \beta+\epsilon_{i},
$$

其中 $\left\{x_{i 1}, x_{i 2}, \ldots, x_{i K}, \epsilon_{i}\right\}$ 是 i.i.d 的，而
$$
\mathbb{E}\left(\mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)=\mathbf{M},\; \forall i=1,2, \ldots, n .
$$

為一個正定矩陣；
$$
\mathbb{E}\left(\mathbf{x}_{i} \epsilon_{i}\right)=\mathbf{0},\; \forall i=1,2, \ldots, n
$$

且

$$
\mathbb{E}\left(\epsilon_{i}^{2} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)=\sigma^{2} \mathbf{M} ,\; \forall i=1,2, \ldots, n
$$

### 一致性

根據上面的假設，我們可以重新將參數矩陣改寫為

$$
\begin{aligned}
\hat{\beta} &=\left(\mathbf{X}^{\top} \mathbf{X}\right)^{-1} \mathbf{X}^{\top} \mathbf{y}=\left(\sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\sum_{i=1}^{n}\mathbf{x}_i \left(\mathbf{x}_i^{\top}\beta+\varepsilon_i\right) \\
&=\left(\sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\sum_{i=1}^{n}\mathbf{x}_i\sum_{i=1}^{n}\mathbf{x}_i^{\top}\beta + \left(\sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\sum_{i=1}^{n} \mathbf{x}_{i} \epsilon_{i}\right)\\
&=\beta+\left(\sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\sum_{i=1}^{n} \mathbf{x}_{i} \epsilon_{i}\right)\\
&=\beta+\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \epsilon_{i}\right)
\end{aligned}
$$

根據弱大數法則，當 $n \to \infty$ 時，

$$
\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top} \stackrel{p}{\rightarrow} \mathbf{M}, \; \frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \varepsilon_{i} \stackrel{p}{\rightarrow} \mathbf{0}
$$

而根據CMT，

$$
\hat{\beta}-\beta \stackrel{p}{\rightarrow} \mathbf{M} \cdot \mathbf{0}=\mathbf{0}
$$

也就是說，$\hat{\beta} \stackrel{p}{\rightarrow} \beta$，即 $\hat{\beta}$ 是 $\beta$ 的一致估計式。

![](https://i.imgur.com/MoNWFdh.png)

除了弱大數法則之外，一致性的性質還取決於 $\mathbb{E}(\mathbf{x}_i \varepsilon_i) = 0$ 的條件，即 $\varepsilon_i$ 與 $\mathbf{x}_i$ 兩者無相關。也就是說，當有足夠的資訊（夠大的樣本）時，OLS 估計式就越能夠接近真實的參數。

## 重要變數與無關變數

在設定迴歸模型時，通常會有兩種人：

- 把模型的參數設超多，以為這樣可以估計得很準 $\Rightarrow$ 放了很多無關的解釋變數
- 把模型的參數設超少，以為自己取的變數是準的 $\Rightarrow$ 忽略了許多相關的解釋變數

但，畢竟生而為人，我們不是上帝，無法知道真實的情況會與何種參數有關，因此我們在設定模型時，常常會出現以下兩種問題。

### 排除重要變數

給定模型如下

$$
y_i = \mathbf{x}_i^{\top} \beta + \varepsilon_i
$$

假設被解釋變數 $y_i$ 真的取決於諸多變數，我們姑且稱之為 $\mathbf{z}_i$，即真實的資料產生機制如下：

$$
y_i = \mathbf{x}_i^{\top} \mathbf{b} + \mathbf{z}_i^{\top} \mathbf{d} + u_i
$$

其中 $\mathbb{E}(\mathbf{x}_i u_i) = 0$ 且 $\mathbb{E}(\mathbf{z}_i u_i) = 0$，因此可以改寫為

$$
\begin{aligned}
\hat{\beta} &=\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} y_{i}\right) \\
&=\mathbf{b}+\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{z}_{i}^{\top} \mathbf{d}+\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} u_{i}\right)
\end{aligned}
$$

根據弱大數法則，當 $n \to \infty$ 時，

$$
\frac{1}{n} \sum_{i=1}^{n} \mathbf{z}_{i} \mathbf{x}_{i}^{\top} \stackrel{p}{\rightarrow} \mathbb{E}(\mathbf{z}_i u_i), \; \frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} u_{i} \stackrel{p}{\rightarrow} \mathbf{0}
$$

因此，即使 $\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} u_{i} \stackrel{p}{\rightarrow} \mathbf{0}$，只要 $\frac{1}{n} \sum_{i=1}^{n} \mathbf{z}_{i} \mathbf{x}_{i}^{\top}$ 不是 $0$，那麼 $\hat{\beta}$ 會收斂到一個不是 $\mathbf{b}$ 的數值。這代表我們的解釋變數 $\mathbf{x}_i$ 與**遺漏變數** $\mathbf{z}_i$ 有關，即 $\operatorname{cov}(\mathbf{x}_i,\mathbf{z}_i) \neq 0$，因此 $\beta$ 的 OLS 估計式便不會是 $\mathbf{b}$，也就是不具有一致性；或是我們可以把$\mathbf{z}_i^{\top} \mathbf{d} + u_i$ 想成是 $\varepsilon_i$（因為兩者有關，所以違反了Gauss-Marlkov定理的假設）。這個現象稱為**遺漏變數偏誤**，而這個問題可以透過**工具變數(instrumental variable, IV)** 解決之。


### 納入無關變數

給定以下模型，

$$
y_{i}=\mathbf{x}_{i}^{\top} \beta+\mathbf{z}_{i}^{\top} \delta+\epsilon_{i}
$$

假設 $y_i$ 真的取決於 $\mathbf{x}$，也就是真實的資料產生機制如下:

$$
y_{i}=\mathbf{x}_{i}^{\top} \mathbf{b}+u_{i}
$$

其中 $\mathbb{E}(\mathbf{x}_i u_i) = 0$ 且 $\mathbb{E}(\mathbf{z}_i u_i) = 0$，但我們卻把另一個解釋變數 $\mathbf{z}$ 納入模型中，得到
$$
y_{i}=\mathbf{x}_{i}^{\top} \mathbf{b}+\mathbf{z}_{i}^{\top} \mathbf{d}+u_{i}
$$
其中令$\mathbf{d} = 0$，此時 OLS 估計式具有一致性的，即
$$
\hat{\beta} \stackrel{p}{\rightarrow} \mathbf{b}, \;\hat{\delta} \stackrel{p}{\rightarrow} \mathbf{d}=\mathbf{0}
$$
那這樣是否代表我們還是可以放超多變數進去模型中呢？答案是不行！以下的漸進分配理論會告訴我們其中原因，以及加入無關的變數後，估計式的變異數變大所帶來的後果。


## 漸進常態性(asymptotic normality)
雖說一開始我們設定的「統計現代假設」並沒有提到誤差項必須服從任何分配，但當樣本足夠大時，$\hat{\beta}$ 會機率收斂到 $\beta$，那麼 $\hat{\beta}$ 的分配又是什麼呢？已知
$$
\hat{\beta}-\beta=\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \varepsilon_{i}\right)
$$
現在我們故意在前面乘上 $\sqrt{n}$，得到
$$
\begin{aligned}
\sqrt{n}(\hat{\beta}-\beta) &=\sqrt{n}\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \varepsilon_{i}\right) \\
&=\left(\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)^{-1}\left(\frac{1}{\sqrt{n}} \sum_{i=1}^{n} \mathbf{x}_{i} \varepsilon_{i}\right)
\end{aligned}
$$

根據弱大數法則，

$$
\frac{1}{n} \sum_{i=1}^{n} \mathbf{x}_{i} \mathbf{x}_{i}^{\top} \stackrel{p}{\rightarrow} \mathbf{M}, \; \frac{1}{\sqrt{n}} \sum_{i=1}^{n} \mathbf{x}_{i} \varepsilon_{i} \stackrel{A}{\sim} \mathcal{N}\left(\mathbf{0}, \sigma^{2} \mathbf{M}\right)
$$

同時，根據CMT，

$$
\sqrt{n}(\hat{\beta}-\beta) \stackrel{A}{\sim} \mathcal{N}\left(\mathbf{0}, \sigma^{2} \mathbf{M}^{-1}\right)
$$

假設 $\sigma^2 = 0.25$，

$$
\mathbf{M}=\mathbb{E}\left(\mathbf{x}_{i} \mathbf{x}_{i}^{\top}\right)=\left[\begin{array}{cc}
1 & 3 / 2 \\
3 / 2 & 3
\end{array}\right],\;
\mathbf{M}^{-1}=\left[\begin{array}{cc}
4 & -2 \\
-2 & 4 / 3
\end{array}\right]
$$



而上面的結果計算如下。

$$
\begin{aligned}
\mathbf{x_i} =
\begin{bmatrix}
1\\
x_i
\end{bmatrix}
\sim U(0,3)
\end{aligned}
$$

接著

$$
\begin{aligned}
\mathbb{E}(\mathbf{x}_i\mathbf{x}_i^{\top}) = 
\mathbb{E}
\begin{bmatrix}
1 & x_i\\
x_i & x_i^2
\end{bmatrix}=
\begin{bmatrix}
1 & 3/2\\
3/2 & 3
\end{bmatrix}
\end{aligned}
$$

而 $x_i^2$ 的期望值為

$$
\mathbb{E}(x_i^2) = \int^3_0 x^2 \cdot \frac{1}{3} dx = \left.\frac{1}{9} x^3 \right|^3_0 = 3
$$
 
 
又

$$
\begin{aligned}
\mathbf{M}^{-1} &= \frac{1}{1\cdot 3 - \frac{3}{2} \cdot \frac{3}{2}}
\begin{bmatrix}
1 & 3/2\\
3/2 & 3
\end{bmatrix}\\
&= 
\begin{bmatrix}
4 & -2\\
-2 & 4/3
\end{bmatrix}
\end{aligned}
$$

最後
$$
\sigma^2 \mathbf{M}^{-1} = 0.25 \times
\begin{bmatrix}
4 & -2\\
-2 & 4/3
\end{bmatrix}=
\begin{bmatrix}
1 & -1/2\\
-1/2 & 1/3
\end{bmatrix}
$$

其中 $\operatorname{var}(\sqrt{n} \cdot \hat{\beta_0}) = 1$，$\operatorname{var}(\sqrt{n} \cdot \hat{\beta_1}) = \frac{1}{3}$，而 $\operatorname{cov}(\sqrt{n} \cdot \hat{\beta_0}, \sqrt{n} \cdot \hat{\beta_1}) = -\frac{1}{2}$。


我們可以試著將樣本數由 $10$ 增加到 $100$

```r
Z <- numeric(5000)
for (r in 1:5000) {
    X <- 3 * runif(10)
    Y <- 2 + X + runif(10,-sqrt(3)/2,sqrt(3)/2)
    OLS <- lm(Y~X)
    Z[r] <- sqrt(10) * (OLS$coefficients[2] - 1) / (1/sqrt(3))
}
c(mean(Z<qnorm(0.025)),mean(Z<qnorm(0.05)),mean(Z<qnorm(0.1)))

## [1] 0.0428 0.0752 0.1272
```

![](https://i.imgur.com/K9XbrF1.png)


```r
Z <- numeric(5000)
for (r in 1:5000) {
    X <- 3 * runif(50)
    Y <- 2 + X + runif(50,-sqrt(3)/2,sqrt(3)/2)
    OLS <- lm(Y~X)
    Z[r] <- sqrt(50) * (OLS$coefficients[2] - 1) / (1/sqrt(3))
}
c(mean(Z<qnorm(0.025)),mean(Z<qnorm(0.05)),mean(Z<qnorm(0.1)))

## [1] 0.0230 0.0466 0.1042
```
![](https://i.imgur.com/PJUkXmz.png)


```r
Z <- numeric(5000)
for (r in 1:5000) {
    X <- 3 * runif(100)
    Y <- 2 + X + runif(100,-sqrt(3)/2,sqrt(3)/2)
    OLS <- lm(Y~X)
    Z[r] <- sqrt(100) * (OLS$coefficients[2] - 1) / (1/sqrt(3))
}
c(mean(Z<qnorm(0.025)),mean(Z<qnorm(0.05)),mean(Z<qnorm(0.1)))

## [1] 0.0244 0.0500 0.0980
```

![](https://i.imgur.com/XbuNiOs.png)

可以看到誤差項的 kernel density 圖形越來越接近常態分配的結果。事實上，即使我們對於誤差項的分配沒有任何要求，或要求誤差項的變異數為**同質性(homoskedasticity)**，同時也不要求 $x_i$ 為隨機或非隨機，只要服從統計現代假設，根據弱大數法則與CLT，OLS 估計式便有一致性與漸進常態性的性質；然而，我們**不能夠說此即為 $\hat{\beta}$ 真正的分配**。而了解這些性質背後的原因，便是要去進行假設檢定。但我們完全不知道 $\sigma^2$

[^1]:Ray在課堂的小提醒：弱大數法則不是**大數據法則**，也不是**30-大數法則**，如果有人膽敢在考卷上寫下弱大數法則的本質是樣本數超過 $30$，樣本平均數會接近母體平均數的話，考卷分數就是 $30$ 分！