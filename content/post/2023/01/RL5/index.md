---
title: "強化学習勉強メモ #5 TD法 [WIP]"
description: 
slug: "RL-5"
date: 2023-01-07T15:00:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: true
---

## TD法
### 概要
　モンテカルロ法(以下MC法と略す)の固定値$\alpha$方式の更新式は式$(1)$で表される．
$$
\begin{align}
V(S_t) \leftarrow V(S_t) + \alpha \left( G_t - V(S_t) \right)
\end{align}
$$

TD(Temporal Difference)法の更新式は，式$(2)$である．
$$
\begin{align}
V(S_t) \leftarrow V(S_t) + \alpha \left( R_t + \gamma V\_\pi(S\_{t+1}) -V\_\pi(S_t) \right)
\end{align}
$$

これらの異なる点は，$G_t$が$R_t + \gamma V\_\pi(S\_{t+1})$に置き換わっていることである．この変更はどこから来たのか．式$(3)$で表されるdp法を思い出して欲しい．

$$
\begin{align}
V\_\pi(S_t) \leftarrow \mathbb{E}\left[ R_t + \gamma V\_\pi(S\_{t+1}) \mid s = S_t \right]
\end{align}
$$

DP法は，このように，一つ先の状態の価値を使って，現在の状態の価値を更新していた．

つまり，TD法は，

- MC法のように，環境から得られた報酬を使って，現在の状態の価値を更新する
- DP法のように，一つ先の状態の価値を使って，現在の状態の価値を更新する

という特徴を持つ，まさにMC法とDP法のミックスさせた方法だ．

他にも，次のような利点が考えられる．

- MC法はデータのばらつきが大きくなってしまうが，TD法は$1$ステップ先の状態の価値を使っているので，データのばらつきが小さくなる
- MC法はエピソード終了まで待たなければならないが，TD法は$1$ステップ先の状態の価値のみが必要なので，エピソード終了まで待つ必要がない(=連続タスクに対応できる)

エピソードタスクの場合，プログラムの流れとしては，
1. 適当な方策$\pi$を決める．
1. 十分な回数次を繰り返す．
    - エピソードタスクを一度走らせる(ただし，行動のたびに$V$が更新される)．

となる．

### 方策制御，SARSA(サルサ)
方策制御をするためには，モンテカルロ法と同じく$Q$値を求める必要がある．式$(4)$の状態価値関数の更新式で，$V$を行動価値関数$Q$に置き換えると式$(5)$となる．
$$
\begin{align}
V(S_t) &\leftarrow V(S_t) + \alpha \left( R_t + \gamma V\_\pi(S\_{t+1}) -V\_\pi(S_t) \right) \\\\
Q(S_t,A_t) &\leftarrow Q(S_t,A_t) + \alpha \left( R_t + \gamma Q\_\pi(S\_{t+1},A_{t+1}) -Q\_\pi(S_t,A_t) \right)
\end{align}
$$
必要な情報$S_t,A_t,R_t,S_{t+1},A_{t+1}$の文字を取ってSARSAと呼ぶ．

求められた$Q$値によって，式$(6)$で最適方策が定まる．
$$
\begin{align}
\mu(s) &= \text{argmax} \_{a} q (s,a)
\end{align}
$$

### その他の改善
モンテカルロ法と同じく，$\varepsilon$-greedy法を使うべきである．

### 方策オフ型のSARSA
モンテカルロ法と同様に，方策オフ型のSARSAを考えることができる．
モンテカルロ法の項で説明したので，詳しいことは省略するが，挙動方策$b$からターゲット方策$\pi$の価値関数$Q$を求めるには，式$(7)$を使えば良い．
$$
\begin{align}
Q(S_t,A_t) &\leftarrow Q(S_t,A_t) + \alpha \left( \frac{\pi(A\_{t+1}|S\_{t+1})}{b(A\_{t+1}|S\_{t+1})}(R_t + \gamma Q\_\pi(S\_{t+1},A_{t+1})) -Q\_\pi(S_t,A_t) \right)
\end{align}
$$

重点サンプリングを用いることで，挙動方策とターゲット方策を別々にできて便利なことが多いが，残念ながら，$\pi$と$b$の違いが大きいほど誤差が大きくなってしまう．

## Q学習
