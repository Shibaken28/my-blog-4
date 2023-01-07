---
title: "三目並べを様々な強化学習で実装 #5 TD法"
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

Temporal Difference法(以下TD法)は、モンテカルロ法(以下MC法)と動的計画法(以下DP法)のいいとこ取りをしたような手法．

MC法の固定$\alpha$法の更新式は，
$$
V(S_t) \leftarrow V(S_t) + \alpha \left( G_t - V(S_t) \right)
$$
となっていた．

TD法の更新式は，
$$
V(S_t) \leftarrow V(S_t) + \alpha \left( R_t + \gamma V\_\pi(S\_{t+1}) -V\_\pi(S_t) \right)
$$

である．

異なる点は，$G_t$が$R_t + \gamma V\_\pi(S\_{t+1})$に置き換わっていることである．
これはどこから来たのか．DP法を思い出して欲しい．

$$
V\_\pi(S_t) \leftarrow \mathbb{E}\left[ R_t + \gamma V\_\pi(S\_{t+1}) \mid s = S_t \right]
$$

DP法は，このように，一つ先の状態の価値を使って，現在の状態の価値を更新していた．

つまり，TD法は，

- MC法のように，環境から得られた報酬を使って，現在の状態の価値を更新する
- DP法のように，一つ先の状態の価値を使って，現在の状態の価値を更新する

というまさにMC法とDP法のミックスさせた方法だ．

他にも，

- MC法はデータのばらつきが大きくなってしまうが，TD法は$1$ステップ先の状態の価値を使っているので，データのばらつきが小さくなる
- MC法はエピソード終了まで待たなければならないが，TD法は$1$ステップ先の状態の価値のみが必要なので，エピソード終了まで待つ必要がない(=連続タスクに対応できる)


## SARSA(サルサ)
​state–action–reward–(next)state–actionの略である．
