---
title: "強化学習勉強メモ #7 ニューラルネットワーク(実装編)"
description: 
slug: "RL-7"
date: 2023-01-09T15:00:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: true
---

$Q$学習によって本にあった迷路を解く．

## 特別な処理
### one-hotベクトル
カテゴリデータを扱う場合，それをone-hotベクトルに変換すると良いことが知られている[^1]．
例えば，$3$つのカテゴリがある場合，それぞれを$0,1,2$として，$0$の場合は$[1,0,0]$，$1$の場合は$[0,1,0]$，$2$の場合は$[0,0,1]$のような$3$次元のベクトルに変換する．
今回は，$3\times 4$の迷路のどこにいるかを$12$次元のone-hotベクトルで表現することにする．

[^1]: $1,2,3$と連番で振った場合に，その数の近さと遠さの情報がノイズとして入ってしまう，という問題がありそうなのが直感的な感想．

### Q関数の出力
$Q$関数の値を得るためのニューラルネットワークは，いくつかの方法が考えられる．
- 状態$S$と行動$A$を入力として，その$Q$値を出力する
- 状態$S$を入力として，全ての行動に関する，その状態での$Q$値を出力する
    - すなわち，行動の種類数だけの次元を持つベクトルが出力される．

$Q$学習では，$\max_a Q(s,a)$を求める必要があるが，前者の方法を採用すると，$Q$関数の出力を全ての$A$に対して，毎回順伝搬を行って見る必要がある．一方，後者の方法を採用すると，$Q$関数の出力を一度だけ求めれば良い．後者を採用する．

### 何を学習するか
　勾配降下法を使うには，損失関数が必要である．損失関数には，「正解」の値を与える必要がある．しかし，当然始めから正解の値はわかっていない．$Q$学習の式を振り返る．

$$
\begin{align}
Q(S_t,A_t) &\leftarrow Q(S_t,A_t) + \alpha \left( R_t + \gamma \max\_{a} Q(S\_{t+1},a) -Q(S_t,A_t) \right)
\end{align}
$$

$T=R_T + \gamma \max\_{a} Q(S\_{t+1},a)$とおく．$Q$学習は，$Q(S_t,A_t)$を$T$に近づけるアルゴリズムであった．そこで，現在の$Q$値に対して，$T$を正解にする．これは，$S,A$の関数の値$T$を求めるような回帰問題に帰着する．


## 実装
### モデルの確認
各情報を明確にしておくことは，実装の効率化につながる．

- 入力: 状態を表す$12$次元のone-hotベクトル
- 出力: 各$A$の価値を表す$4$次元のベクトル
- 中間層: $100$次元のベクトル
- 活性化関数: ReLU
- 損失関数: 平均二乗誤差
- 最適化手法: SGD

### Modelクラスの作成
　`Model`クラスを継承して，`QNet`クラスを作成する．
```python
class QNet(Model):
    def __init__(self):
        super().__init__()
        self.l1 = L.Linear(100)  # hidden_size
        self.l2 = L.Linear(4)  # action(out)_size

    def forward(self, x):
        x = F.relu(self.l1(x))
        x = self.l2(x)
        return x
```
最初の関数を`l1`，最後の関数を`l2`としている．それぞれ`Linear`クラスをインスタンス化したものであり，コンストラクタの引数は出力の次元数を表す．`forward`メソッドは，順伝搬を行うメソッドであり，線形関数，ReLU関数，線形関数の順に計算を行っている．

### 損失関数の実装
損失関数は，`dezero.functions.mean_squared_error`によって実装されている．

### one-hotベクトルの実装
```python
def one_hot(state):
    HEIGHT, WIDTH = 3, 4
    vec = np.zeros(HEIGHT * WIDTH, dtype=np.float32)
    y, x = state
    idx = WIDTH * y + x
    vec[idx] = 1.0
    return vec[np.newaxis, :]
```

### オプティマイザ
　オプティマイザにmodelを渡すことで，簡潔にニューラルネットワークの学習を行うことができる．初期設定は次のように行える．
```python
qnet = QNet()
optimizer = optimizers.SGD(lr=0.01)
optimizer.setup(qnet)
```
損失関数に正解の値と予測値を渡し，`backward`メソッドを呼び出し，`update`メソッドを呼び出すことで，パラメータの更新を行う．
```python
target = gamma * next_q + reward
qs = qnet(state)
q = qs[:, action]
loss = F.mean_squared_error(target, q)

self.qnet.cleargrads()
loss.backward()
self.optimizer.update()
```