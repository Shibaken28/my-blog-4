---
title: "部分空間への正射影を求める"
description: 
slug: syaei
date: 2023-05-20T12:08:30+09:00
image: 
categories:
    - 数学
    - 線形代数
---

# この記事は執筆途中です


## 問題
次の問題を考える．
> 　線形独立な$n$個の$m$次元実列ベクトル$\boldsymbol{a}_1,\boldsymbol{a}_2,\cdots,\boldsymbol{a}_n$(ただし，$n<m$)によって貼られる$m$次元実数空間がある．この$m$次元実数空間の部分空間を考え，その部分空間への正射影$q$はどう求めるか．


## 正射影とは
### 次元$1$への正射影
想像のしやすいように，まずは$2$次元の場合を考える．
適当な二次元のベクトル$\boldsymbol{b},\boldsymbol{a}$を考える．このとき，$\boldsymbol{b}$を$\boldsymbol{a}$へ正射影するとは，$\boldsymbol{b}$を$\boldsymbol{a}$に垂直に落としたときの足の位置を指す．

ベクトル$\boldsymbol{b}$と$\boldsymbol{a}$のなす角を$\theta$とすれば，射影の長さは$|\boldsymbol{b}|\cos\theta$である．よって，次のような式変形により，正射影$q$の長さを求めることができる．

$$
\begin{aligned}
|q| &= |\boldsymbol{b}|\cos\theta \\\\
&= \frac{|\boldsymbol{a}||\boldsymbol{b}|\cos\theta}{|\boldsymbol{a}|} \\\\
&= \frac{\boldsymbol{a}^T\boldsymbol{b}}{|\boldsymbol{a}|}
\end{aligned}
$$

あとはベクトルの向きは$\boldsymbol{a}$と同じなので，$\boldsymbol{a}$の単位ベクトル$\boldsymbol{u}$を用いて，正射影$q$は次のように表せる．

$$
\begin{aligned}
\boldsymbol{q} &= \frac{\boldsymbol{a}^T\boldsymbol{b}}{|\boldsymbol{a}|}\boldsymbol{u} \\\\
&= \frac{\boldsymbol{a}^T\boldsymbol{b}}{|\boldsymbol{a}|} \frac{\boldsymbol{a}}{|\boldsymbol{a}|} \\\\
&= \frac{\boldsymbol{a}^T\boldsymbol{b}}{|a|^2}\boldsymbol{a}
\end{aligned}
$$
さらに，スカラー量の掛け算は順序を入れ替えてもよいので，次のように行列$A$を用いて表せる．
$$
\begin{aligned}
\frac{\boldsymbol{a}^T\boldsymbol{b}}{|a|^2}\boldsymbol{a} &= \boldsymbol{a}\frac{\boldsymbol{a}^T\boldsymbol{b}}{|a|^2} \\\\
&=  \frac{1}{|a|^2}\boldsymbol{a}\boldsymbol{a}^T\boldsymbol{b} \\\\
&= \boldsymbol{A}\boldsymbol{b}
\end{aligned}
$$

同様に，空間(三次元)のベクトル(要素が$3$つ)を，別の空間(三次元)のベクトルへの正射影も考えることができる．

ここまでは高校数学の範囲だが，残念ながら，次元$2$以上のベクトル空間への正射影は，このように簡単に求めることができない．

### 部分空間への正射影
正射影は別の言い方で直交射影とも呼ばれる．この**直交**というのはまさに正射影の性質を表している．

では，$3$次元のベクトル空間において，$2$次元の部分空間への正射影を考える．

(ここに図を入れる)

見ての通り，正射影$\boldsymbol{q}$は，$\boldsymbol{q}$から$\boldsymbol{b}$へ向かうベクトル$\boldsymbol{r}$と直交する．

そして，正射影$\boldsymbol{q}$は，部分空間の基底ベクトル$\boldsymbol{a}_1,\boldsymbol{a}_2$の線形結合で表せる．
$$
\boldsymbol{q} = x_1\boldsymbol{a}_1 + x_2\boldsymbol{a}_2
$$
これは行列で表すと，
$$
\boldsymbol{q} = A\boldsymbol{b}
$$
となる．
ただし，
$$
A = \begin{pmatrix}
\boldsymbol{a}_1 & \boldsymbol{a}_2
\end{pmatrix} \\\\
\boldsymbol{x} = \begin{pmatrix}
x_1 \\\\
x_2
\end{pmatrix}
$$
である．

正射影とベクトル$\boldsymbol{r}$が直交するということは，$\boldsymbol{r}$は$\boldsymbol{a}_1,\boldsymbol{a}_2$のそれぞれと直行するということである．式にすると，
$$
\begin{aligned}
\boldsymbol{a}_1^T\boldsymbol{r} &= 0 \\\\
\boldsymbol{a}_2^T\boldsymbol{r} &= 0
\end{aligned}
$$
となる．
$\boldsymbol{r}$は$\boldsymbol{b}$から$\boldsymbol{q}$へ向かうベクトルなので，$\boldsymbol{r} = \boldsymbol{b} - \boldsymbol{q}$である．これを上の式に代入すると，
$$
\begin{aligned}
\boldsymbol{a}_1^T(\boldsymbol{b} - \boldsymbol{q}) &= 0 \\\\
\boldsymbol{a}_2^T(\boldsymbol{b} - \boldsymbol{q}) &= 0
\end{aligned}
$$
実は，この式は行列で表すことができる．
$$
\begin{pmatrix}
\boldsymbol{a}_1^T \\\\
\boldsymbol{a}_2^T
\end{pmatrix}
(\boldsymbol{b} - \boldsymbol{q}) = 
A^T(\boldsymbol{b} - \boldsymbol{q}) = \boldsymbol{0}
$$

$\boldsymbol{q}=A\boldsymbol{b}$であるから，

$$
\begin{aligned}
A^T(\boldsymbol{b} - \boldsymbol{q}) &= \boldsymbol{0} \\\\
A^T\boldsymbol{b} - A^T\boldsymbol{q} &= \boldsymbol{0} \\\\
A^T\boldsymbol{b} &= A^T\boldsymbol{q} \\\\
A^T\boldsymbol{b} &= A^TA\boldsymbol{x} \\\\
\boldsymbol{x} &= (A^TA)^{-1}A^T\boldsymbol{b}
\end{aligned}
$$

こうして，正射影$\boldsymbol{q}$を求めるために必要な$\boldsymbol{x}$が求まった．



