---
title: "[WIP]代表的な分布の分散と期待値を導出"
description: モーメント母関数もあるよ
slug: EV
date: 2023-06-03T12:08:30+09:00
image: 
hidden: true
categories:
    - 数学
    - 確率統計
---

## 定義
- 離散型
$$
\begin{aligned}
E(X) &= \sum_{x} xP(X=x) \\\\
V(X) &= \sum_{x} (x-E(X))^2 \\\\
&= E(X^2) - (E(X))^2
\end{aligned}
$$

- 連続型

$$
\begin{aligned}
E(X) &= \int_{-\infty}^{\infty} xf(x)dx \\\\
V(X) &= \int_{-\infty}^{\infty} (x-E(X))^2 f(x)dx \\\\
&= E(X^2) - (E(X))^2
\end{aligned}
$$

- $E(X)$は期待値、$V(X)$は分散を表す。
- 分散の求め方は、二乗の期待値から期待値の二乗を引く手法の方が扱いやすいことが多い。

## モーメント母関数(積率母関数)
モーメント母関数というものを導入することで、簡単に期待値や分散を求めることができる場合がある。
確率変数$X$のモーメント母関数は以下で定義される。

$$
M_X(t) = E(e^{tX})
$$

また、マクローリン展開を用いれば

$$
\begin{aligned}
M_X(t) &= E(e^{tX}) \\\\
&= E \left( \sum_{k=0}^{\infty} \frac{(tX)^k}{k!} \right) \\\\
&= E \left( 1 + tX + \frac{(tX)^2}{2!} + \frac{(tX)^3}{3!} + \cdots \right) \\\\
&= 1 + tE(X) + \frac{t^2}{2!}E(X^2) + \frac{t^3}{3!}E(X^3) + \cdots \\\\
\end{aligned}
$$
となり、これを$t$で微分すると

$$
\begin{aligned}
M_X^\prime (t) &= E(X) + tE(X^2) + \frac{t^2}{2!}E(X^3) + \cdots \\\\
\end{aligned}
$$
であり、$t=0$を代入すると、$M_X^\prime (0) = E(X)$と期待値が求まる。

## 二項分布
二項分布は以下で定義される。

$$
P(X=x) = {}_n\mathrm{C}_x p^x q^{n-x}
$$

ただし、$p+q=1$である。

- $P(X)$は、確率$p$で成功する試行を$n$回行ったとき、$X$回成功する確率を表す。

### 期待値

$$
\begin{aligned}
E(X) &= \sum\_{k=0}^{n} kP(X=k) \\\\
&= \sum\_{k=0}^n k {}\_n\mathrm{C}\_k p^k q^{n-k} \\\\
&= \sum\_{k=1}^n k \frac{n!}{k!(n-k)!} p^k q^{n-k} \\\\
&= \sum\_{k=1}^n \frac{n!}{(k-1)!(n-k)!} p^k q^{n-k} \\\\
&= \sum\_{k=1}^n n \frac{(n-1)!}{(k-1)!(n-k)!} p\cdot p^{k-1} q^{n-k} \\\\
&= np \sum\_{k=1}^n \frac{(n-1)!}{(k-1)!(n-k)!} p^{k-1} q^{n-k} \\\\
&= np \sum\_{l=0}^{n-1} \frac{(n-1)!}{l!(n-1-l)!} p^l q^{n-1-l} \\\\
&= np(p+q)^{n-1} \\\\
&= np
\end{aligned}
$$

- 最後は二項定理(の逆)を利用した。

また、モーメント母関数を用いても求めることができる。

$$
\begin{aligned}
M_X(t) &= E(e^{tX}) \\\\
&= \sum\_{k=0}^n e^{tk} {}\_n\mathrm{C}\_k p^k q^{n-k} \\\\
&= \sum\_{k=0}^n {}\_n\mathrm{C}\_k (pe^t)^k q^{n-k} \\\\
&= (pe^t + q)^n \\\\
\end{aligned}
$$

$t$して$t=0$を代入すれば期待値が求まる($p+q=1$であることに注意)。

$$
\begin{aligned}
M_X^\prime (t) &= n(pe^t + q)^{n-1} pe^t \\\\
M_X^\prime (0) &= np
\end{aligned}
$$




### 分散

$$
\begin{aligned}
V(X) &= E(X^2) - (E(X))^2 \\\\
&= \sum\_{k=0}^n k^2 P(X=k) - (np)^2 \\\\
&= \sum\_{k=1}^n k^2 \frac{n!}{k!(n-k)!} p^k q^{n-k} - n^2p^2 \\\\
&= \sum\_{k=1}^n k(k-1) \frac{n!}{k!(n-k)!} p^k q^{n-k} + \sum\_{k=1}^n k \frac{n!}{k!(n-k)!} p^k q^{n-k} - n^2p^2 \\\\
&= \sum\_{k=1}^n k(k-1) \frac{n!}{k!(n-k)!} p^k q^{n-k} + np - n^2p^2 \\\\
&= \sum\_{k=1}^n \frac{n!}{(k-2)!(n-k)!} p^k q^{n-k} + np - n^2p^2 \\\\
&= \sum\_{k=1}^n n(n-1) \frac{(n-2)!}{(k-2)!(n-2-(k-2))!} p^2 p^{k-2} q^{n-2-(k-2)} + np - n^2p^2 \\\\
&= n(n-1)p^2 \sum\_{k=1}^n \frac{(n-2)!}{(k-2)!(n-2-(k-2))!} p^{k-2} q^{n-2-(k-2)} + np - n^2p^2 \\\\
&= n(n-1)p^2(p+q)^{n-2} + np - n^2p^2 \\\\
&= n(n-1)p^2 + np - n^2p^2 \\\\
&= np(1-p)
\end{aligned}
$$

- $k^2=k(k-1)+k$を利用した。
- やっていることは期待値のときと似ている。

## ポアソン分布

ポアソン分布は以下で定義される。
$$
P(X=k) = \frac{\lambda^k}{k!}e^{-\lambda} \quad (k=0,1,2,\cdots)
$$

- $P(X)$は、単位時間あたりに平均$\lambda$回起こる事象が、$k$回起こる確率を表す。

### 期待値

$$
\begin{aligned}
E(X) &= \sum\_{k=0}^{\infty} kP(X=k) \\\\
&= \sum\_{k=0}^{\infty} k \frac{\lambda^k}{k!}e^{-\lambda} \\\\
&= \sum\_{k=1}^{\infty} \frac{\lambda^k}{(k-1)!}e^{-\lambda} \\\\
&= \lambda e^{-\lambda} \sum\_{k=1}^{\infty} \frac{\lambda^{k-1}}{(k-1)!} \\\\
&= \lambda e^{-\lambda} \sum\_{l=0}^{\infty} \frac{\lambda^l}{l!} \\\\
&= \lambda e^{-\lambda} e^{\lambda} \\\\
&= \lambda
\end{aligned}
$$

- 途中で$e^x$のマクローリン展開を利用した。

### 分散

$$
\begin{aligned}
V(X) &= E(X^2) - (E(X))^2 \\\\
&= \sum\_{k=0}^{\infty} k^2 P(X=k) - \lambda^2 \\\\
&= \sum\_{k=1}^{\infty} k^2 \frac{\lambda^k}{k!}e^{-\lambda} - \lambda^2 \\\\
&= \sum\_{k=1}^{\infty} k(k-1) \frac{\lambda^k}{k!}e^{-\lambda} + \sum\_{k=1}^{\infty} k \frac{\lambda^k}{k!}e^{-\lambda} - \lambda^2 \\\\
&= \sum\_{k=1}^{\infty} \frac{\lambda^k}{(k-2)!}e^{-\lambda} + \lambda - \lambda^2 \\\\
&= \sum\_{k=1}^{\infty} \lambda(\lambda-1) \frac{\lambda^{k-2}}{(k-2)!}e^{-\lambda} + \lambda - \lambda^2 \\\\
&= \lambda(\lambda-1)e^{-\lambda} \sum\_{k=1}^{\infty} \frac{\lambda^{k-2}}{(k-2)!} + \lambda - \lambda^2 \\\\
&= \lambda(\lambda-1)e^{-\lambda} e^{\lambda} + \lambda - \lambda^2 \\\\
&= \lambda(\lambda-1) + \lambda - \lambda^2 \\\\
&= \lambda
\end{aligned}
$$

- $k^2=k(k-1)+k$を利用した。

