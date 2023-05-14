---
title: "偏微分思ったより単純な話じゃなかった"
description: (個人の感想)
slug: henbibun
date: 2023-05-13T15:01:00+09:00
image: 
categories:
    - 数学

---

## 偏微分の定義
$$
\frac{\partial f(x,y)}{\partial x} = \lim_{h \to 0} \frac{f(x+h,y)-f(x,y)}{(x+h)-x}
$$

よく「ある関数のある変数に注目して，他の変数を定数として微分することである．」と説明され，その通りである．
そして，私は「じゃあいつもの微分(常微分)がわかれば何ら難しいことはないんだな」と思っていた．しかし，他にも常微分との区別で注意すべき点がある．

## 問題の式
$z(x,y)$と$f(x,y,z)$がある．このとき，$f(x,y,z(x,y))$を$x$で偏微分すると次のように表される．
$$
\begin{align}
\frac{\partial f(x,y,z(x,y))}{\partial x} &= \frac{\partial f}{\partial x} + \frac{\partial f}{\partial z} \frac{\partial z}{\partial x} \\\\
\end{align}
$$
この式は私が問題の解説を見ているときに出てきた．そして私は，左辺には$f$を$x$で偏微分したものがあり，右辺にも$f$を$x$で偏微分したものがある，おかしいじゃないかと思った．ほかにも，次のような変形をして，余計に混乱をしていた．
$$
\begin{align}
\frac{\partial f(x,y,z(x,y))}{\partial x} &= \frac{\partial f}{\partial x} + \frac{\partial f}{\partial z} \frac{\partial z}{\partial x}  \\\\
&= \frac{\partial f}{\partial x} + \frac{\partial f}{\partial x} \quad ??? \\\\
&= \frac{\partial f}{\partial x} \quad ???
\end{align}
$$
当然この式変形は間違っているのだが，その理由が説明できなかった．

## 定義に戻って導出
今考えているのは，$f(x,y,z(x,y))$で$x$が極小$\Delta x$だけ変化したとき，$f$の変化量$\Delta f$を求めることである．
$$
\begin{align}
\Delta f &= f(x+\Delta x,y,z(x+\Delta x,y)) - f(x,y,z(x,y)) \\\\
&= f(x+\Delta x,y,z(x+\Delta x,y)) - f(x,y,z(x+\Delta x,y)) \nonumber
\\\\ & \quad + f(x,y,z(x+\Delta x,y)) - f(x,y,z(x,y))
\\\\ &= \frac{f(x+\Delta x,y,z(x+\Delta x,y)) - f(x,y,z(x+\Delta x,y))}{\Delta x} \Delta x \nonumber
\\\\ & \quad \quad +  \frac{f(x,y,z(x+\Delta x,y)) - f(x,y,z(x,y))}{\Delta x} \Delta x \\\\
\nonumber \\\\
\lim_{\Delta x \to 0} \Delta f&= \lim_{\Delta x \to 0} \frac{f(x+\Delta x,y,z(x,y)) - f(x,y,z(x,y))}{\Delta x} \Delta x \nonumber
\\\\ & \quad \quad + \lim_{\Delta x \to 0} \frac{f(x,y,z(x+\Delta x,y)) - f(x,y,z(x,y))}{\Delta x} \Delta x \\\\
\end{align}
$$
式変形を説明すると，
- 式$(5)$：これが求めたい値である．
- 式$(6)$：余計な項を付け足した．足して引いているので，値は変わらない．
- 式$(7)$：$\Delta x$を余分にかけている．分子と分母に同じ値をかけているので，値は変わらない．
- 式$(8)$：$\Delta x\to 0$のとき，$f(x+\Delta x,y,z(x+\Delta x,y))-f(x,y,z(x+\Delta x,y)) \to f(x+\Delta x,y,z(x,y))-f(x,y,z(x,y))$であることを用いた．
    - 随分乱雑な変形に見えるかもしれないし，実際そうである気がする．しかし，この後のこの項の扱われ方を見ると，この変形でも問題なさそうである．

式$(8)$の一項目は，$f(x,y,z)$を$x$で偏微分したものであるが，このときに$z$は**動いていない**．二項目は，$f(x,y,z)$うち$z(x,y)$の$x$で偏微分したものである．このときに$f$の第一引数$x$は**動いている**．

二項目は，
$$
\begin{align}
& \quad \frac{f(x,y,z(x+\Delta x,y)) - f(x,y,z(x,y))}{\Delta x} \Delta x　\\\\
&= \frac{f(x,y,z+\Delta z) - f(x,y,z)}{\Delta z} \Delta z
\end{align}
$$
であるような，$z$の変化量$\Delta z$を考えると，$z$の変化量$\Delta z$は$x$の変化量$\Delta x$によって決まり，
$$
\begin{align}
\Delta z &= z(x+\Delta x,y) - z(x,y) \\\\
&= \frac{z(x+\Delta x,y) - z(x,y)}{\Delta x} \Delta x \\\\
&=  \frac{z(x+\Delta x,y) - z(x,y)}{\Delta x} \Delta x \\\\
&=  \frac{z(x+\Delta x,y) - z(x,y)}{\Delta x} \Delta x \\\\
\end{align}
$$
よって，
$$
\begin{align}
&\quad \lim_{\Delta x \to 0} \frac{f(x,y,z+\Delta z) - f(x,y,z)}
{\Delta z} \Delta z 
\\\\ &=
\lim_{\Delta x \to 0} \frac{f(x,y,z+\Delta z) - f(x,y,z)}
{\Delta z} \frac{z(x+\Delta x,y) - z(x,y)}{\Delta x} \Delta x \\\\
&=
\frac{\partial f}{\partial z} \frac{\partial z}{\partial x} \Delta x
\end{align}
$$
一番初めの式に戻って，
$$
\begin{align}
\Delta f &= \lim_{\Delta x \to 0} \frac{f(x+\Delta x,y,z(x,y)) - f(x,y,z(x,y))}{\Delta x} \Delta x \nonumber
\\\\ & \quad \quad + \lim_{\Delta x \to 0} \frac{f(x,y,z(x+\Delta x,y)) - f(x,y,z(x,y))}{\Delta x} \Delta x \\\\
&= \frac{\partial f}{\partial x} \Delta x + \frac{\partial f}{\partial z} \frac{\partial z}{\partial x} \Delta x \\\\
\end{align}
$$

$\Delta$を$d$に変えて
$$
df = \frac{\partial f}{\partial x} dx + \frac{\partial f}{\partial z} \frac{\partial z}{\partial x} dx
$$
であることが示せた．ただし，これをもっと厳密に書くならば，
$$
df(x,y,z(x,y)) = \frac{\partial f(x,y,z)}{\partial x} dx + \frac{\partial f(x,y,z)}{\partial z} \frac{\partial z(x,y)}{\partial x} dx
$$
であり，両辺を$dx$で割ると，
$$
\begin{align}
\frac{df(x,y,z(x,y))}{dx} &= \frac{\partial f(x,y,z)}{\partial x} + \frac{\partial f(x,y,z)}{\partial z} \frac{\partial z(x,y)}{\partial x} \\\\
&= \frac{\partial f}{\partial x} + \frac{\partial f}{\partial z} \frac{\partial z}{\partial x}
\end{align}
$$
という式が導出される．この式では，右辺では常微分の$df/dx$が，左辺では偏微分の$\partial f/\partial x$が登場している．

## 注意点
常微分のように，通分して$\partial z$を消すことはできない．
$$
\frac{\partial f}{\partial z} \frac{\partial z}{\partial x}
\ne \frac{\partial f}{\partial x} 
$$


