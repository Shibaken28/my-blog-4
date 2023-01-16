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


## 偏微分

## 期待値

