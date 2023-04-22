---
title: "編入数学 知識・テクニックまとめ"
description: 
slug: hennyu 
date: 2023-03-16T15:04:39+09:00
image: 
categories:
    - 数学
hidden: true
draft: true
---

　編入数学の勉強をしていると，特有の式変形だったり，変数変換だったり，「テクニック」のようなものがしばしば見受けられる．また，罠なども紹介している．

## 極限
- 指数がついている場合は対数を取ってみる
    - 自然対数の底$e$の存在$\lim_{x\to\infty}(1+x)^\frac{1}{x}$も忘れずに
        - 変形してこの式に持っていく場合もある
- 累乗に二項定理を途中で打ち切ったものではさみうちの原理を使うことがある
    - $(1+h)^n > 1 + nh + \frac{n(n-1)}{2}h^2$など
        - $n$が$1$のときの場合分けを忘れない
- 階乗もはさみうちの原理を使うことがある
    - $n! = 1\cdot 2\cdot 3\cdot \cdots \cdot n \leq 1\cdot n\cdot n \cdot \cdots \cdot n \leq n^{n-1}$など
- 三角関数もロピタルかはさみうちの原理を疑う
    - $\sin x/x$は$\sin x$の取りうる範囲より$-1 / x \leq \sin x / x \leq 1/ x$なので，$\lim_{x \to \infty } \sin x / x = 1$となる．

## 微分
- 極値は1階微分が0をとることが**必要条件**
    - 微分値=0は極値ではない場合がある($y=x^3$の$x=0$みたいな)
    - 本当に極値であるか，極大極小を調べるには，増減表または2階微分の符号で判断できる
- チェインルールマジで大事
    - 特に多変数の場合
## 積分
- パターンがたくさんあるので経験を積む
    - 部分分数分解
    - 部分積分
    - 変数変換
    - 三角関数の相互関係，各種公式
- 微分積分学の基本定理は重要
$$
\begin{align*}
\int^x\_a f(t) dt &= F(x) \\\\
\frac{d}{dx} F(x) &= f(x)
\end{align*}
$$
- よく見るタイプの次の式は$s=x-t$のような変換で見やすくなるかも
$$
\begin{align*}
g(x) = \int^x\_a P(t)f(x-t) dt 
\end{align*}
$$
- 区間内に存在しているかの証明は最小値と最大値を使って不等号で挟む


### 指数関数と三角関数の積
　$I = \int e^{ax} \sin bx$ の積分の方法は何通りかある．どの方法も覚えておいて損はないだろう(私は1つだけ覚えていたが，他の方法を説明なく使われたときに意味がわからなかった)．

#### 1. 部分積分を2回行う
よく説明される方法(もっとも，他の方法は結果は割と天下り的であるためそうなる)．
$$
\begin{align*}
I &= \int e^{ax} \sin bx dx \\\\
&= \frac{1}{a} e^{ax} \sin bx - \int \frac{1}{a} e^{ax} b \cos bx dx \\\\
&= \frac{1}{a} e^{ax} \sin bx - \frac{b}{a^2} e^{ax} \cos bx - \int \frac{1}{a^2} e^{ax} b^2 \sin bx dx \\\\
&= \frac{1}{a} e^{ax} \sin bx - \frac{b}{a^2} e^{ax} \cos bx - \frac{b^2}{a^2} I 
\end{align*}
$$
よって，$I$について解くと
$$
\begin{align*}
\frac{a^2}{a^2+b^2}I &= \frac{1}{a} e^{ax} \sin bx - \frac{b}{a^2} e^{ax} \cos bx \\\\
I &= \frac{a^2}{a^2+b^2} \left( \frac{1}{a} e^{ax} \sin bx - \frac{b}{a^2} e^{ax} \cos bx \right) \\\\
&= \frac{1}{a^2+b^2} \left( ae^{ax} \sin bx - b e^{ax} \cos bx \right) + C \\\\
\end{align*}
$$
とわかる．

#### 2. 微分したら$I$になりそうなものを探す
　$D_1=e^{ax}\sin bx$と$D_2=e^{ax}\cos bx$をそれぞれ微分する．
$$
\begin{align*}
\left( e^{ax}\sin bx \right)^\prime &= a e^{ax} \sin bx + b e^{ax} \cos bx \\\\
\left( e^{ax}\cos bx \right)^\prime &= a e^{ax} \cos bx - b e^{ax} \sin bx \\\\
\end{align*}
$$
ここで，微分したら$I=e^{ax}\sin bx$になるものが欲しいので，うまい具合に係数を調整して$D_1$と$D_2$を組み合わせて$e^{ax}\sin bx$を作る．
$$
\begin{align*}
\left( aD_1 - bD_2 \right)^\prime &= (a^2 + b^2) e^{ax} \sin bx \\\\
\therefore \quad I &= \frac{1}{a^2 + b^2} \left( a e^{ax} \sin bx - b e^{ax} \cos bx \right) + C \\\\
\end{align*}
$$

#### 3. 複素数(オイラーの公式)を使う
　$\int e^{ax} e^{ibx} dx$を計算する
$$
\begin{align*}
\int e^{ax} e^{ibx} dx &= \int e^{(a+ib)x} dx \\\\
&= \frac{1}{a+ib} e^{(a+ib)x} \\\\
&= \frac{a-ib}{a^2+b^2} e^{ax}e^{ibx} \\\\
&= \frac{a-ib}{a^2+b^2} \left( e^{ax} \cos bx + e^{ax} i\sin bx \right) \\\\
\end{align*}
$$
両辺の虚部に注目すると結果が得られる．
$$
\begin{align*}
\int e^{ax} e^{ibx} dx &= \frac{a-ib}{a^2+b^2} \left( e^{ax} \cos bx + e^{ax} i\sin bx \right) \\\\
\int e^{ax} \left( \cos bx + i\sin bx \right) dx &= \frac{a-ib}{a^2+b^2} \left( e^{ax} \cos bx + e^{ax} i\sin bx \right) \\\\
\int e^{ax} \sin bx dx &= \frac{1}{a^2+b^2} \left( a e^{ax} \sin bx - b e^{ax} \cos bx \right) \quad (虚部に注目) \\\\
\end{align*}
$$
実部を見れば$e^{ax}\cos bx$にもなり，楽でお得な方法である．


## 確率
### 極限値が存在している場合の漸化式

## 複素関数
### 留数定理中に出てくる微分


## 全般・その他
- 問題を見て答えがすぐに分かる天才は少ない，試行錯誤は重要である，とあの[ヨビノリさん](https://www.youtube.com/channel/UCqmWJJolqAgjIdLqK3zD1QQ)も言っている！場合分けも億劫がらずにやる，とヨビノリさんが言ってた！
- 条件や例外などをしっかり確認する．
    - $a$は正の数なのか，実数なのか，$0$はありえるのか．
    - 必要によっては場合分けを行う．
        - 例：$1/x^a$の積分($a$は整数)
        - 例：$\sin mx \sin nx$の定積分($m$と$n$は自然数，フーリエ変換で出てくるアレ)
### 二乗ルートの罠
　$\sqrt{a^2} = a$は負の数$a$に対しては成り立たない．$\sqrt{a^2}=|a|$である．そんなの当たり前じゃないかと思うかもしれないが，たまに牙を向いてくるから油断はできない．
例えば次の積分だ.
$$
\int^{\frac{\pi}{2}}\_{-\frac{\pi}{2}} \sqrt{1-\cos^2 x}  dx
$$
ああ，ルートが出てきてビビったがこれは$\sin x$に変形できるな．
$$
\int^{\frac{\pi}{2}}\_{-\frac{\pi}{2}} \sqrt{1-\cos^2 x} dx = \int^{\frac{\pi}{2}}\_{-\frac{\pi}{2}} \sin x  dx = 0 \quad (?)
$$
としてしまうのは間違いである．というのも，$\sin x$はこの積分範囲では負の数を取るからだ．つまり，これは
$$
\begin{align*}
\int^{\frac{\pi}{2}}\_{-\frac{\pi}{2}} \sqrt{1-\cos^2 x} dx &= \int^{\frac{\pi}{2}}\_{-\frac{\pi}{2}} \sqrt{\sin^2 x} dx \\\\
&= \int^{\frac{\pi}{2}}\_{-\frac{\pi}{2} }|\sin x| dx \\\\
&= 2\int^{\frac{\pi}{2}}\_0 \sin x dx \\\\
&=2 \left[-\cos x \right]^\frac{\pi}{2}\_0 \\\\
&= 2 \\\\
\end{align*}
$$
と計算するのが正しい．

### 元の方程式でも成り立つか
　ルートと似たトピックだが，ルートや分母を払ったあとに出てきた方程式は，元の方程式と解が一致しないことがある．
例えば，次の方程式を考える．
$$
\begin{align*}
\sqrt{2-x^2} &= x \\\\
2-x^2 &= x^2 \\\\
2 &= 2x^2 \\\\
x &= \pm 1 \\\\
\end{align*}
$$
元の方程式は$\sqrt{2-x^2}=x$は$x=-1$を満たさない．$x>0$であるから，$x=\sqrt{2}$のみが解である．

