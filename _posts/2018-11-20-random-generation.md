---
title: 随机算法编程入门
categories: math prob
math: true
---

教程使用的语言是`C#`。

## 0 符号与用语

| 符号  | 含义  |
|---|---|
|$\newcommand{\P}{\mathbb{P}} \newcommand{\d}{\text{d}} \renewcommand{\R}{\mathbb{R}} \newcommand{\N}{\mathbb{N}} \newcommand{\Z}{\mathbb{Z}}$$\mathbb{R}$|实数|
|$\N$|自然数|
|$\Z$|整数|
|$\inf, \sup$|下确界，上确界|
|$\exp (x)$|指数函数，即$e^x$|
|$\ln, \log$|自然对数|
|$\forall x$|对于所有的$x$|
|$\exists x$|存在$x$|

<br>

-----

## 1 简介

### 1.1 均匀分布（Uniform Distribution）

均匀的随机变量$X$有概率密度

$$f(x) = \begin{cases} \displaystyle \frac{1}{b-a} & a \leq x \leq b \\ \\ 0 & 其它 \end{cases}$$

记作$\text{Unif}[a,b]$。特别的，在实际生成时，一般使用在0和1之间的伪随机过程，可以看做在0和1之间的均匀分布随机变量。

生成方式如下：
```csharp
using System;

public class Program
{
	public static void Main(string[] args)
	{
		var rand = new Random();
		double rv = rand.NextDouble();
	}
}
```

*之后只会写出方法，不会写出整体框架。

<br>

### 1.2 概率密度函数（PDF）与概率分布函数（CDF）

对于某随机变量$X$，如果它的概率分布函数$F_X(x) = \P (X \le x)$，那么它的概率密度函数为$f_X(x) = \displaystyle \frac{\d F_X(x)}{\d x}$。对于这两个函数，有以下性质：

1. $\displaystyle f_X(x) \ge 0$
2. $\displaystyle \int_{-\infty}^\infty f_X(x) \d x = 1$
3. $\displaystyle F_X(x) = \int^x_{-\infty}f_X(t) \d t$

在这里不详细解释概率论。有兴趣请阅读[维基百科](https://en.wikipedia.org/wiki/Cumulative_distribution_function)有关CDF与PDF的内容。

<br>

-----

## 2 逆抽样算法

### 2.1 逆抽样（Inverse Sampling)

逆抽样是一个生成随机变量的算法。根据所需要的CDF，生成符合该CDF的随机变量。要求只有一个：能够生成在区间$[0,1]$上的均匀分布随机变量，并且该CDF为可逆的，即它单调递增，$f_X(x) > 0$。由此可知，如果有均匀分布的随机变量$U \sim \text{Unif}[0,1]$，那么我们可以得出

$$\P(F^{-1}_X(U) \le t) = \P(U \le F_X(t)) = F_X(t),$$

因为$U$是均匀分布的，它的CDF在该区间就是$F_X(x) = x$。于是通过该式我们得出结论：$F_X^{-1}(U)$与$F_X(x)$具有相同的CDF。那么，我们很容易就能生成符合该CDF的变量，只要找出它的反函数$F^{-1}_X(x)$，并代入$U$即可。对于离散型随机变量的CDF（是阶梯型右连续函数），我们定义它的逆

$$F^{-1}_X(x) = \inf_{t \in \R} \{t : F_X(t) \ge x\}.$$

### 2.2 逆抽样示例

假设我们需要生成[指数分布（Exponential Distribution）](https://baike.baidu.com/item/指数分布)的变量$X > 0$，有$X \sim \text{Exp}(\beta)$，那么我们有$X$的PDF

$$f_X(x) = \beta^{-1} e^{-x/\beta}.$$

那么，我们能通过积分得到它的CDF

$$F_X(x) = 1 - e^{-x/\beta}.$$

于是，我们求它的反函数

$$F_X^{-1}(y) = -\beta \ln (1-y).$$

生成$U \sim \text{Unif}[0,1]$，代入就能得到

$$X = -\beta \ln U.$$

### 2.3 代码示例

假设生成参数$\beta = 2$的指数分布随机变量$X \sim \text{Exp}(2)$。代码如下：

```csharp
double Generate(double beta)
{
    var rand = new Random();
    double u = rand.NextDouble();

    return -beta * Math.Log(u);
}
```

<br>

-----

## 3 Box-Muller（博克斯穆勒）算法

### 3.1 正态分布(Normal/Gaussian Distribution)

**正态分布**，又称**高斯分布**，有两个参数，记作$\mathcal{N}(\mu, \sigma^2)$，其中$\mu$叫做**平均值**（Mean Value)，而$\sigma^2$叫做**方差**（Variance），$\sigma$叫做**标准差**（Standard Deviation）。如果随机变量$X \sim \mathcal{N}(\mu, \sigma^2)$，那么有PDF

$$f_X(x) = \frac{1}{\sqrt{2\pi}\sigma} \exp \left (-\frac{(x-\mu)^2}{2\sigma^2} \right ),$$

其中$x \in \R$，$\sigma > 0$，$\mu \in \R$，表达式$\exp(x) = e^x$。

<br>

### 3.2 标准正态分布（Standard Normal Distribution）

标准正态分布指$\mu=0$，$\sigma^2 = 1$的正态分布，即$\mathcal{N}(0,1)$。它的PDF是

$$f_X(x) = \frac{1}{\sqrt{2\pi}} \exp \left ( -\frac{x^2}{2} \right ).$$

我们还能知道它的CDF

$$\Phi(z) = \frac{1}{\sqrt{2\pi}} \int^z_{-\infty} \exp \left ( -\frac{x^2}{2} \right ) \d x.$$

图像以及其它性质可以参考[百度百科](https://baike.baidu.com/item/标准正态分布)关于正态分布的内容。它的图像是铃铛形（Bell Shape）的。

<br>

### 3.3 Box-Muller 算法

正态分布是统计学一大重要分布。在游戏设计中也广泛使用。不详细解释该算法的过程，只讨论如何生成正态分布变量。

首先，生成$U_0, V_0 \sim \text{Unif}[0,1]$。然后，计算得到$U = -2 \ln U_0$和$V = 2\pi V_0$。于是，我们得到一组标准正态分布变量

$$X = \sqrt{U} \cos(V)$$

$$Y = \sqrt{U} \sin(V)$$

它们是独立的，即我们得到了两个标准正态变量。原理请读者自行摸索。提示：$U \sim \text{Exp}(2)$，$V \sim \text{Unif}[0, 2\pi]$。

用代码实现：

```csharp
public double StdGaussian()
{
    var rand = new Random();
    double u = -2 * Math.Log(rand.NextDouble());
    double v = 2 * Math.PI * rand.NextDouble();

    return Math.Sqrt(u) * Math.Cos(v);
}
```

<br>

-----

## 4 接受拒绝算法

### 4.1 接受拒绝算法（Accept-Reject Algorithm）

假设我们能够生成PDF为$g(x)$的随机变量$Y$，我们想要生成PDF为$f(x)$的随机变量$X$。根据接受拒绝算法，我们能够生成更多的随机变量，把在$f(x)$外的拒绝即可。

<br>

### 4.2 算法步骤

使用该算法的前提是

1. 当$f(x)>0$时，$g(x)>0$。
2. 对于任意的$x$，存在$M >0$，使得$\displaystyle \frac{f(x)}{g(x)} \le M$。即该比值有上界，称之为$M$。

于是，我们可以使用以下步骤生成变量：

1. 先生成符合$g(x)$的随机变量$Y$与$U \sim \text{Unif} [0,1]$。
2. 如果$\displaystyle \frac{f(Y)}{M \cdot g(Y)} \ge U$，接受$X = Y$。
3. 如果上一步不成立，则回到1。

<br>

### 4.3 简单应用

**例题：**假设我们需要生成伽马分布的变量$\text{Gamma}(\frac{7}{2},1)$，并且我们知道它的PDF与

$$f(x) = x^{5/2}e^{-x}$$

成正比。生成该变量。（提示：接受拒绝算法可以不用管该比例。）

**解答：**首先，选择一个我们能生成的变量，并且我们知道它的PDF。在这里我选择指数分布变量$X \sim \text{Exp}(2)$。它的PDF是$g(x) = \frac{1}{2} e^{-x/2}$。我们需要找到$M$使得

$$\max_{x \ge 0} \frac{f(x)}{g(x)} \le M, \ \forall x \ge 0.$$

这个上界很好找。令$h(x) = f(x) / g(x)$，我们有$h(x) = 2x^{5/2}e^{-x/2}$。对它求导，得

$$h'(x) = -(x-5)x^{\frac{3}{2}}e^{-\frac{x}{2}}.$$

于是，使导数等于$0$，得$x=0$或$x=5$，计算$h(0) = 0$，$h(5) = 9.1773$，于是它的最大值为$9.1773$。我们可以选择一个上界$M = 10$。接下来，我们就能使用该算法生成变量。代码如下（假设生成1000个变量）：

```csharp
public List<double> GenGamma()
{
    var rand = new Random();
    int total = 0;
    int valid = 0;
    int size = 1000;
    double M = 10;
    var results = new List<double>();

    while (valid < size)
    {
        double unif = rand.NextDouble();
        double y = -2 * Math.Log(rand.NextDouble());

        double fY = Math.Pow(y, 2.5) * Math.Exp(-y);
        double gY = 0.5 * Math.Exp(-y / 2);
        if (fY / (M * gY) >= unif)
        {
            results.Add(y);
            valid++;
        }

        total++;
    }
    return results;
}
```

其中，生成的随机变量储存在`results`中，一共$1000$个。我运行了该程序，结果共生成了$2698$个指数分布随机变量(`total`)，其中$37.06\%$被接受作为$\text{Gamma}(7/2, 1)$（即那$1000$个）。

<br>

-----

#### 引用

- [1] Cumulative Distribution Function. Wikipedia. 2018.
- [2] 指数分布. 百度百科. 2018.
- [3] Probability and Statistics. Shuyang Ling, New York University. 2018.
- [4] 概率论与数理统计. 浙江大学, 高等教育出版社. June, 2008.
