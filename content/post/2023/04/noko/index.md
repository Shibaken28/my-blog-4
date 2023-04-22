---
title: "農工大編入試験過去問を解く"
description: 数学，物理，専門(論理回路，アルゴリズムを選択)
slug: noko-kakomon
date: 2023-04-07T12:08:30+09:00
image: 
categories:
    - 数学
hidden: true
---


## 2023年度実施
### 数学
#### 大問1
$$
f(x,y) = x^2y+xy^2+2x^2-xy-4y^2-6x-12y
$$
で偏微分の値が$0$になる点と極値を求めなさい．

---

$$
\begin{align*}
f_x &= 2xy+y^2+4x-y-6 = 0 \quad \cdots \quad (1) \\\\
f_y &= x^2 + 2xy - x -8y - 12 = 0\quad \cdots \quad (2)
\end{align*}
$$
$(1)$かつ$(2)$を満たせば良い．

$(1)$を因数分解すると，
$$
\begin{align*}
 2xy+y^2+4x-y-6 &= 2x(y+2)+y^2-y-6 \\\\
&= 2x(y+2) + (y-3)(y+2) \\\\
&= (y+2)(2x+y-3) = 0 \\\\
\end{align*}
\\\\
y + 2 = 0，または 2x+y-3 = 0
$$
$(2)$を因数分解すると，
$$
\begin{align*}
 x^2 + 2xy - x -8y - 12 &= 2y(x-4) + x^2 - x - 12 \\\\
 &= 2y(x-4) + (x+3)(x-4) \\\\
 &= (x-4)(x+3+2y) = 0 \\\\
\end{align*}
\\\\
x - 4 = 0，または x+3+2y = 0
$$
よって，$(1)$かつ$(2)$を満たす点は，
- $y+2=0$かつ$x-4=0$のとき
- $2x+y-3=0$かつ$x+3+2y=0$のとき
- $2x+y-3=0$かつ$y+2=0$のとき
- $x-4=0$かつ$x+3+2y=0$のとき

の$4$パターンある．
よって，$(x,y)=(4,-2),(4,-5),(1,-2),(3,-3)$が求める点である．

各点は極値の候補であるため，それぞれの点を確認する．なお，二次偏導関数は以下のようになる．

$$
\begin{align*}
f\_{xx} &= 2y+4 \quad \cdots \quad (3) \\\\
f\_{yy} &= 2x-8 \quad \cdots \quad (4) \\\\
f\_{xy} &= 2x+2y-1 \quad \cdots \quad (5) \\\\
H(x,y) &= f\_{xx}f\_{yy} - (f\_{xy})^2
\end{align*}
$$


(i) $(x,y)=(4,-2)$のとき$H(4,-2) = -9<0$より極値ではない． \
(ii) $(x,y)=(4,-5)$のとき$H(4,-5) = -9<0$より極値ではない．\
(iii) $(x,y)=(1,-2)$のとき$H(1,-2) = -9<0$より極値ではない．\
(iv) $(x,y)=(3,-3)$のとき$H(3,-3) = 3>0$より極値となる．また，$f_{xx}<0$．

よって，$f(x,y)$は$(3,-3)$で極大値$f(3,-3)=9$を取る．

コメント：\
　シンプルな極値を求める問題．計算量が多いので丁寧に計算する．

#### 大問2
$$
\begin{align*}
D &= {(x,y)|\frac{x^2}{4} + \frac{(y-3)^2}{9} \leq 1} \\\\
I &= \iint_D y\ dxdy
\end{align*}
$$
$I$を求めなさい．

---
$$
\begin{align*}
x =& 2r\cos\theta \quad \cdots \quad (1) \\\\
y-3 =& 3r\sin\theta \quad \cdots \quad (2)
\end{align*}
$$
と変換する．ヤコビアンは$6r$で積分範囲は$0\leq r\leq 1$，$\theta$は$0\leq\theta\leq 2\pi$である．

$$
\begin{align*}
I &= \int \_{0}^{2\pi} \int \_{0}^{1} (3r\sin\theta + 3)6r\ dr d\theta \\\\
&= 18 \int \_{0}^{2\pi} \int \_{0}^{1} (r^2\sin^2\theta + r) dr d\theta \\\\
&= 18 \int \_{0}^{2\pi}(6\sin\theta + 9) d\theta \\\\
&= 18 \pi
\end{align*}
$$


コメント：\
　積分範囲は楕円を表している．極座標に変換すると楽に解ける．ヤコビアン掛け忘れに注意．

#### 大問3

#### 大問4
$$
\begin{align*}
\frac{d^2x}{dx^2}
\end{align*}
$$
