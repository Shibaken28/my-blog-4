---
title: "強化学習勉強メモ #9 方策勾配法(WIP)"
description: 
slug: "RL-9"
date: 2023-01-14T16:00:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: true
---

$Q$学習やSARSA学習では，状態の**価値**を求めて，その価値から方策を決定していた．ここでは，方策そのものを学習する方法，**方策勾配法**を考える．

## 導出
### 目的関数
　ニューラルネットワークの全ての重みのパラメータを$\theta$とする．方策$\pi\_{\theta}(a\mid s)$を学習する．このとき，方策勾配法では，方策のパラメータ$\theta$を更新する．正解がわからないため，損失関数を用意することはできないが，**目的関数**を用意してこれを最大化するように学習する[^1]．

[^1]:実は，損失関数にマイナス$1$を掛ければ，最大化する問題になるため，実質損失関数と目的関数は同じものである．

　目的関数を設定する．エピソードタスクで，方策$\pi\_{\theta}$を実行したときの軌道(trajectory)が次の通りであったとする．
$$
\tau = (S_0, A_0, R_0, S_1, A_1, R_1, \cdots, S_{T-1}, A_{T-1}, R_{T-1}, S_T)
$$
この$\tau$から収益は決定するため，次の通りに関数$G(\tau)$が定義できる．
$$
G(\tau) = R_0 + \gamma R_1 + \gamma^2 R_2 + \cdots + \gamma^{T-1} R_{T-1} + \gamma^T R_T
$$

目的関数$J(\theta)$は次のように表される．
$$
J(\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}[G(\tau)]
$$
波の記号$a\simA$は，$A$によって$a$が抽出(サンプリング，あるいは生成)されることを表す．
この式は$\pi\_{\theta}$から生成された軌道$\tau$の収益の期待値を表す．
あとは，目的関数の勾配がわかれば，勾配上昇法[^2]でパラメータを更新することができる．

[^2]:勾配降下法は勾配ベクトルの逆の方向にパラメータを更新したが，勾配上昇法は勾配ベクトルの方向にパラメータを更新する．

### 勾配の導出
では，目的関数の勾配$\nabla\_{\theta} J(\theta)$を求める．
先に結果を示すと，次のようになる．
$$
\nabla\_{\theta} J(\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} G(\tau) \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
$$

期待値の式からの変形によって導出する．

$$
\begin{align}
\nabla\_{\theta} J(\theta) &= \nabla\_\theta \mathbb{E}\_{\tau\sim\pi\_{\theta}} \left[ G(\tau) \right] \\\\
&= \nabla\_{\theta} \sum\_r \mathrm{Pr}(\tau | \theta)G(\tau) \\\\
&= \sum\_r \nabla\_{\theta} \mathrm{Pr}(\tau | \theta)G(\tau) \\\\
&= \sum\_r \left\\{ G(\tau)\nabla\_{\theta} \mathrm{Pr}(\tau | \theta) + \mathrm{Pr}(\tau | \theta)\nabla\_{\theta} G(\tau) \right\\} \\\\
&= \sum\_r G(\tau)\nabla\_{\theta} \mathrm{Pr}(\tau | \theta) \\\\
&= \sum \_{r} G(\tau) \mathrm{Pr}(\tau | \theta) \frac{\nabla\_{\theta} \mathrm{Pr}(\tau | \theta)}{\mathrm{Pr}(\tau | \theta)} \\\\
&= \sum \_{r} G(\tau) \mathrm{Pr}(\tau | \theta) \nabla\_{\theta} \log \mathrm{Pr}(\tau | \theta) \\\\
&= \mathbb{E}\_{\tau\sim\pi\_{\theta}} \left[ G(\tau) \nabla\_{\theta} \log \mathrm{Pr}(\tau | \theta) \right] \\\\
&= \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} G(\tau) \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
\end{align}
$$

- 式$(2)$: 期待値を確率$\times$収益の総和に変形
- 式$(3)$: $\nabla$を移動．足し算した後に微分しても，微分した後に足し算しても結果は変わらない．
- 式$(4)$: 積の微分の公式を用いる．
- 式$(5)$: $\nabla\_\theta G(\tau)$は，$0$である．なぜなら，$G(\tau)$は$\theta$に依存しないからである．
    - いやいや，$\tau$は$\theta$によって決まるから，$G(\tau)$は$\theta$に依存するだろ，と思われるかもしれない(筆者は思った)．確かに，$\theta$から$\tau$を取り出したときに，その$\tau$は$\theta$に依存する．しかし，今見ているのはある一つの特定の$\tau$である．様々な$\theta$が考えられるが，どの$\theta$から取り出した$\tau$でも，その$\tau$自身はどれも同じであるから，$G(\tau)$はいつも同じ，つまり定数である．
- 式$(6)$: $\mathrm{Pr}(\tau | \theta)/\mathrm{Pr}(\tau | \theta) = 1$を掛け算した．
    - $\nabla$が掛かっているものは変わっていないことに注意
- 式$(7)$: $\log$の合成関数の微分の逆を行っている[^3]．
$$
(\log f(x))' = \frac{f'(x)}{f(x)}
$$
- 式$(8)$: 確率$\times$そのときの値の形をしていたので期待値として表す．

[^3]:この変形はよく出てくるらしい．

式$(9)$は，少し飛躍がある．$\mathrm{Pr}(\tau | \theta)$を展開する．なお，$p(S\_0)$は，始めの状態が$S\_0$である確率を表す．
$$
\begin{align*}
\mathrm{Pr}(\tau | \theta) = p(S\_0) \prod\_{t=0}^{T-1} \pi\_\theta (A\_t | S\_t) p(S\_{t+1} | S\_t, A\_t)
\end{align*}
$$
対数をとると，積が和で表されるので，次のようになる．
$$
\begin{align*}
\log \mathrm{Pr}(\tau | \theta) = \log p(S\_0) + \sum\_{t=0}^{T-1} \log \pi\_\theta (A\_t | S\_t) + \sum\_{t=0}^{T-1} \log p(S\_{t+1} | S\_t, A\_t)
\end{align*}
$$
よって，$\nabla\_\theta \log \mathrm{Pr}(\tau | \theta)$は，
$$
\begin{align*}
\nabla\_\theta \log \mathrm{Pr}(\tau | \theta) = \nabla\_\theta \log p(S\_0) + \sum\_{t=0}^{T-1} \nabla\_\theta \log \pi\_\theta (A\_t | S\_t) + \sum\_{t=0}^{T-1} \nabla\_\theta \log p(S\_{t+1} | S\_t, A\_t)
\end{align*}
$$
このうち，$p(S_0)$と$p(S\_{t+1} | S\_t, A\_t)$は$\theta$に依存しないので，
$$
\begin{align*}
\nabla\_\theta \log \mathrm{Pr}(\tau | \theta) &= \sum\_{t=0}^{T-1} \nabla\_\theta \log \pi\_\theta (A\_t | S\_t) \\\\
&= \nabla\_\theta \sum\_{t=0}^{T-1} \log \pi\_\theta (A\_t | S\_t)
\end{align*}
$$
となる．以上で，導出は終わりである．


## 勾配上昇法
最も簡単な更新式は次である．$\alpha$は学習率である．
$$
\theta \leftarrow \theta + \alpha \nabla\_{\theta} J(\theta)
$$

$\nabla\_\theta J(\theta)$を計算するために，モンテカルロ法を用いる．期待値の式で，
$$
\begin{align*}
\nabla\_{\theta} J(\theta) &=  \mathbb{E}\_{\tau\sim\pi\_{\theta}} \left[ \sum\_{t=0}^{T} G(\tau) \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
\end{align*}
$$
$\theta$に従い，$\tau$をサンプリングし，そのときエピソードで計算をする．
$$
\text{sampling:} \tau\_i \sim \pi\_\theta \quad (i = 1, \dots, N) \\\\
x^{(i)} = \sum\_{t=0}^{T} G(\tau\_i) \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \\\\
\nabla\_{\theta} J(\theta) \approx \frac{1}{N} \sum\_{i=1}^{N} x^{(i)}
$$
サンプルが$N=1$つだけの場合，次の式になる．これを使う場合は，エピソードの各行動をするたびに，$\nabla\_{\theta} \log \pi\_\theta (A_t |  S_t)$を計算し，その値$G(\tau)$を足していくような実装となる．
$$
\begin{align*}
\nabla\_{\theta} J(\theta) \approx \sum\_{t=0}^{T} G(\tau) \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t)
\end{align*}
$$

## 実装
例のポールを倒れないようにするやつを実装する．
### ソフトマックス関数
$n$個の要素を持つベクトル$x$を入力すると，次のようなベクトルを返す関数である．
$$
\begin{align*}
\mathrm{softmax}(x) = \left( \frac{\exp(x\_1)}{\sum\_{i=1}^{n} \exp(x\_i)}, \dots, \frac{\exp(x\_n)}{\sum\_{i=1}^{n} \exp(x\_i)} \right)
\end{align*}
$$
$\exp(a)$は$e^a$を表し，これは各$e^{x\_i}$を$\sum\_{i=1}^{n} \exp(x\_i)$でわっているので，全ての値を足すと1になる．また，$e^a$は正の値であるので，各要素は0から1の間に収まる．この値は確率として解釈できる．

### モデル
ReLU関数とソフトマックス関数の$2$層のネットワークを用いる．
```python
class Policy(Model):
    def __init__(self, action_size):
        super().__init__()
        self.l1 = L.Linear(128)
        self.l2 = L.Linear(action_size)

    def forward(self, x):
        x = F.relu(self.l1(x))
        x = F.softmax(self.l2(x))
        return x
```
「行動の種類数」次元の出力をソフトマックス関数に通すことで，各行動の確率として解釈できるようにしている．

　各行動の報酬とその確率を保存しておき，勾配の式にしたがって目的関数を計算する．プログラムでは，目的関数を負にして，損失関数として扱っている．計算が終わったら最後にoptimizerのupdateメソッドを呼び出して，パラメータを更新する．
```python
def update(self):
    self.pi.cleargrads()

    G, loss = 0, 0
    for reward, prob in reversed(self.memory):
        G = reward + self.gamma * G

    for reward, prob in self.memory:
        loss += -F.log(prob) * G

    loss.backward()
    self.optimizer.update()
    self.memory = []
```
全体のコードは本の作者様の[GitHub](https://github.com/Shibaken28/deep-learning-from-scratch-4/blob/master/ch09/simple_pg.py)に置いてあるので，そちらを参照してください．

ただし，この方法では効率が悪い．

## REINFORCEアルゴリズム
### 概要
$$
\nabla\_{\theta} J(\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} G(\tau) \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
$$
　さっきまで使っていたこの式の問題点を考えると，どの時刻$t$に対しても$G(\tau)$との積を取っている点が気になる．というのも，$t=T$のときにも$G(\tau)$が使われてしまっているのである．$t=T$は，最後の行動であり，その行動の重みとして，それまでのすべての行動$\tau$に関する収益$G(\tau)$を使っていることになる．つまり，$t=T$の行動をする前の情報が入ってきてしまっている．これは，他の$t$も同じで，それ以前どんな動きをしていたのかも知らないのに，それまでの行動込みの収益$G(\tau)$を使ってしまっている．そこで，$t$以降の行動のみを考慮するようにする．
$$
\nabla\_{\theta} J(\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} G_t \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right] \\\\
G\_t = R\_t + \gamma R\_{t+1} + \dots + \gamma^{T-t} R\_{T}
$$

### 証明

勝手に$G\_t$に変えていいのか疑問に思うが，これは証明ができる．
証明は本には載っておらず，証明は[こちら](https://spinningup.openai.com/en/latest/spinningup/extra_pg_proof1.html)に書いてある．

*todo: 証明を理解する*

## ベースライン
### ベースラインとは
*todo: 書く*
### Actor-Critic
$G(\tau)$から$b(S\_t)$(ベースライン)を引き算する．$b(S\_t)$は$S\_t$の関数であれば何でもいい．
$$
\begin{align*}
\nabla\_\theta J(\pi\_\theta) &= \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} G(\tau) \nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right] \\\\
\nabla\_\theta J(\pi\_\theta)&= \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} (G(\tau) - b(S_t))\nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
\end{align*}
$$
予測の精度が高くなるため，分散が小さくなる．

さて，肝心の$b(S\_t)$であるが，これは状態価値関数を使う．新たな変数として$w$を価値関数を表すニューラルネットワークのパラメータ，$V\_w(S\_t)$をそれに基づいた状態価値関数とする．
$$
\nabla\_\theta J(\pi\_\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} (G(\tau) - V\_w(S_t))\nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
$$
そして，これをTD法にする．
$$
\nabla\_\theta J(\pi\_\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} (G(\tau) + \gamma V\_w(S\_{t+1})-V\_w(S\_t))\nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
$$


### 証明
いきなりだが，全事象の各確率の合計は$1$であるため次の式は成り立つ．
$$
\sum\_x P\_\theta(x) = 1
$$
この式の勾配を求めると，$0$になる．
$$
\nabla\_\theta \sum\_x P\_\theta(x) = \nabla\_\theta 1 = 0
$$
値が$0$である式を変形すると，次のようになる(途中で$\log$の合成関数の微分の逆を行っている)．
$$
\begin{align*}
\nabla\_\theta \sum\_x P\_\theta(x) &= \sum\_x \nabla\_\theta P\_\theta(x) \\\\
&= \sum \_{x} P\_\theta(x) \frac{\nabla\_{\theta} P\_\theta (x)}{P\_\theta (x)} \\\\
&= \sum\_{x}P\_\theta(x) \nabla\_\theta \log  P\_\theta (x) \\\\
&= \mathbb{E}\_{x\sim P\_\theta} [\nabla\_\theta \log P\_\theta(x)]
\end{align*}
$$
これはつまり，次が成り立つということだ．
$$
\mathbb{E}\_{A_t\sim \pi\_\theta} [\nabla\_\theta \log \pi\_(A\_t|S\_t)] = 0
$$
好きな定数を掛け算しても$0$のままだ．定数である$S_t$の関数を掛け算しても問題ない(これは$A_t$に関する期待値だから)．
$$
\mathbb{E}\_{A_t\sim \pi\_\theta} [b(S\_t)\nabla\_\theta \log \pi\_\theta(A\_t|S\_t)] = 0
$$

### 実装
*todo: 書く*

## Vの代わりにQ関数を使う
これも本には載っておらず，[こちら](https://spinningup.openai.com/en/latest/spinningup/extra_pg_proof2.html)に書いてある．

### 結論
$$
\nabla\_\theta J(\pi\_\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} \left( \nabla\_\theta \log \pi\_\theta (a\_t|s\_t) \right)
Q^{\pi\_\theta}(s\_t,a\_t) \right]
$$

### 証明
*todo:書く*


## まとめ
方策勾配法は，基本的に次の式で表される．
$$
\nabla\_\theta J(\pi\_\theta) = \mathbb{E}\_{\tau\sim\pi\_{\theta}}
\left[ \sum\_{t=0}^{T} \Phi\_\tau\nabla\_{\theta} \log \pi\_\theta (A_t |  S_t) \right]
$$
$\Phi\_\tau$は，手法によって異なる．

- $\Phi\_\tau = G(\tau)$の場合
- $\Phi\_\tau = G\_t$の場合
- $\Phi\_\tau = G(\tau) - V\_w(S\_t)$の場合
- $\Phi\_\tau = G(\tau) + \gamma V\_w(S\_{t+1})-V\_w(S\_t)$の場合

