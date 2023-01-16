---
title: "強化学習勉強メモ #ex Extra Material"
description: 
slug: "RL-ex"
date: 2023-01-15T16:00:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: true
---

なぜタイトルが英語なのかというと，Extra Materialのいい感じの日本語訳が思いつかなかったから．

## 記号
### 関数，変数まとめ
多くの変数が出てきたので一覧にする．まだ出てきてないものも載せてある．

| 文字 | 種類 | 意味| 英語 |
| --- | --- | --- | --- |
|$S$|変数|状態|State|
|$A$|変数|行動|Action|
|$R$|変数|報酬|Reward|
|$t$|変数|時刻|Time|
|$G$|変数|収益|Return|
|$\gamma$|定数|割引率|Discount Rate|
|$\alpha$|定数|学習率|Learning Rate|
|$v(s)$|関数|状態価値関数|State Value Function|
|$q(s,a)$|関数|行動価値関数|Action Value Function|
|$f(s,a)$|関数|状態遷移関数|State Transition Function|
|$p(s^{\prime}\|s,a)$|関数|状態遷移(確率)関数|State Transition (Probability)Function|
|$\mu(s)$|関数|決定的な方策の関数|Deterministic Policy Function|
|$\pi(a\|s)$|関数|確率的な方策の関数|Stochastic Policy Function|

また，組み合わせて使ったときの例も示す．
|文字|意味|英語|
|---|---|---|
|$v_\pi(s)$|方策$\pi$で，状態$s$の価値|State-Value of State $s$ under Policy $\pi$|
|$q_\pi(s,a)$|方策$\pi$，状態$s$で，行動$a$の価値|Action-Value of State $s$ and Action $a$ under Policy $\pi$|
|$v_{\*}(s)$|最適方策で，状態$s$の価値|State-Value of State $s$ under Optimal Policy|
|$q_{\*}(s,a)$|最適方策，状態$s$で，行動$a$の価値|Action-Value of State $s$ and Action $a$ under Optimal Policy|
|$r(s,a,s^{\prime})$|状態$s$で行動$a$をとり状態$s^{\prime}$に遷移したときの報酬|Reward of State $s$ and Action $a$ to State $s^{\prime}$|
|$f(s,a)$|状態$s$で行動$a$をとったときの次の状態|Next State of State $s$ and Action $a$|
|$p(s\|s^{\prime},a)$|状態$s$から状態$s^{\prime}$に遷移したときの確率|Probability of State $s$ to State $s^{\prime}$ by Action $a$|
|$\pi(a\|s)$|状態$s$で行動$a$をとる確率|Probability of Action $a$ in State $s$|

### 慣習
- プライム$^{\prime}$がついている変数は，次の状態を表すことが多い
    - $a^{\prime},s^{\prime}$など
- 最適な方策に関することにはアスタリスク$*$がつく
    - $\pi^\*,v\_\*,q\_\*$など
- 求めている途中の関数(真の値ではない)は，大文字で表す．
    - $V,Q$など


## ベクトル解析
### 偏微分
　複数変数の関数で，注目する1つの変数について微分したものが偏微分．注目した変数以外は定数として扱う．$x$で偏微分したものは$\frac{\partial f}{\partial x}$と表す(分数を書くのが面倒なので$f\_x$と書かれることもある)．
$$
\begin{align*}
f(x,y) &= x^2 + y^2 + xy \\\\
\frac{\partial f}{\partial x} &= 2x + y \\\\
\frac{\partial f}{\partial y} &= 2y + x
\end{align*}
$$

### 勾配
　多変数関数の偏微分をベクトルとしてまとめたもの．関数$f$の勾配は$\nabla f$で表す．
$$
\begin{align*}
f(x,y) &= x^2 + y^2 + xy \\\\
\nabla f &= \left(\frac{\partial f}{\partial x},\frac{\partial f}{\partial y}\right) = \left(2x + y,2y + x\right)
\end{align*}
$$
多変数関数$f(x_1,x_2,\cdots,x_n)$でも同様．
$$
\nabla f = \left(\frac{\partial f}{\partial x_1},\frac{\partial f}{\partial x_2},\cdots,\frac{\partial f}{\partial x_n}\right)
$$

## 期待値
期待値は英語でExpected ValueやExpectationと呼ばれる．
### 定義1
　当選するとコインが出てくるタイプのスロットを$10$回遊んだとき，次のような結果となったとする．

| 回数 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 結果[枚] | 0 | 1 | 1 | 0 | 1 | 1 | 1 | 0 | 0 | 1 |

期待値$E[X]$の定義は，次の式で与えられる．ただし，$P(x)$は確率変数$x$が起こる確率である．


$$
\begin{align}
E[X] &= \sum_{x} P(X=x) x
\end{align}
$$


この場合の獲得できるコインの枚数$X$も期待値は，次のように計算できる．
$$
\begin{align}
E[X]&=
\frac{1}{10} \times 1 + \frac{1}{10} \times 1 + \frac{1}{10} \times 1 + \frac{1}{10} \times 0 + \frac{1}{10} \times 1 \\\\
    & \quad + \frac{1}{10} \times 1 + \frac{1}{10} \times 1 + \frac{1}{10} \times 0 + \frac{1}{10} \times 0 + \frac{1}{10}\times 1\\\\
&= 0.6
\end{align}
$$
### 定義2
　確率変数$X$の取りうる値が連続の場合，次のように定義される．シグマでの定義とやっていることはかわらない．
$$
\begin{align}
E[X] &= \int_{-\infty}^{\infty} P(X=x) dx
\end{align}
$$

### 線形性
　期待値の和は和の期待値と等しい．
$$
E[X+Y] = E[X] + E[Y]
$$

https://manabitimes.jp/math/698 に面白い例が載っていました．
> 　各桁が独立に確率 $\frac{1}{2}$ で 1か2 であるような 9 桁の数字の並びがある。このとき「 $111$ 」と並ぶ部分の個数の期待値を求めよ。例えば「$111221111$」は1〜3，6〜8，7〜9番目の三箇所とみなす。


# *todo: 続きを書く*
