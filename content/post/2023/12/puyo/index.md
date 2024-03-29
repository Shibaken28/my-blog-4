---
title: "強化学習でぷよぷよを学習させたかった(願望)"
description: 
slug: puyo-dqn
date: 2023-12-02T12:08:30+09:00
image: 
categories:
    - 強化学習
tags:
    - DQN
    - 強化学習
---

## これは何
強化学習の手法のうちの一つであるDeep Q-Network (DQN)で、ぷよぷよを学習させるという記事です。本記事はガッツリぷよぷよAIについて研究したとかではなく、こんなことやってみました程度のものです。私自身、強化学習について最近興味を持った素人で、「強化学習ってどのくらいできるもんなのか」という動機でやりました。

**結果として、サルより強いAIが完成しました**

また、本記事は[長野高専アドベントカレンダー](https://qiita.com/advent-calendar/2023/nnct)の2日目の記事です。


## DQNとは
Deep Q-Network(DQN)をいきなり説明するのは無理なので、次の順で説明します。

1. 用語
2. Q学習
3. ニューラルネットワーク

### 用語
強化学習特有の用語をいくつか説明します。

|用語|説明|ぷよぷよでの例|
|:--|:--|:--|
|エージェント|学習する本人のこと|ぷよぷよを操作する人|
|状態|エージェントに与える状況|盤面の状態|
|行動|エージェントが取れる行動|ぷよを置く場所と向き|
|報酬|エージェントが行動を取ったときに与えられる値|ぷよを消した数など|

エージェントは、報酬の総和(正確には、総和ではないこともある)を最大化するように行動を選択します。ですので、学習の目的によって報酬の付け方が変わります。

報酬は、各行動を一回行うごとに与えられます。つまり、次のサイクルで学習が行われます。

1. エージェントが状態を観測する
2. エージェントが行動を選択する
3. エージェントが行動を実行し、状態が遷移する
4. エージェントが報酬を受け取る
5. 1に戻る

また、報酬は負の値にすることも可能です。例えば、ゲームオーバーになってしまった場合には報酬$-1$を与える、ということもできます。

### Q学習
#### 定義
$Q$学習は$Q$関数を用います。$Q$関数は次のように定義されます。

$$
Q(s,a) = (状態sで行動aを取ったときの評価値)
$$

シンプルな定義です。「評価値」は大きければ大きいほど、その先で得られる報酬が大きいということを表します。

もし、完全な$Q$関数が完成していたとしましょう。このときは、次の手順で最適な行動ができます。

1. 現在の状態$s$で行える全ての行動$a_1,a_2,\dots,a_n$について、$Q(s,a_i)$を計算する
2. $Q(s,a_i)$が最大となる行動$a_i$を選択する

#### Q関数の作り方
ランダムに行動して、その結果得られた報酬を$Q$関数に反映させることで、$Q$関数の各値を求めます。

ある状態$s$で行動$a$を取ったときに、状態$s^\prime$に遷移し、報酬$r$を得たとします。このとき、$Q$関数の値を次のように更新します。

$$
Q(s,a) \leftarrow Q(s,a) + \alpha(r + \gamma \max_{a^\prime}Q(s^\prime,a^\prime) - Q(s,a))
$$

ここで、$\alpha$は学習率、$\gamma$は割引率と呼ばれる値です(ただし$0\leq \alpha,\gamma \leq 1$)。

この式は実はそんなに難しいことは言っていないのですが、説明するのが面倒なので省略します。

理屈はさておき、この更新式を使って、ランダムに動くことを繰り返して$Q$関数を求めることはできます。しかし、状態$s$の場合の数が非常に大きいな場合、$Q$関数を求めるのに時間がかかってしまいます。例えば、ぷよぷよの盤面の場合、縦$12$横$6$の計$72$マスがあり、各マスは$4$色のぷよか、空白のいずれかであるため、$5^{72}$通りの状態があります[^1]。これは約$10^{50}$通りであり、現時点で世界に存在するコンピューターで計算するのは不可能なオーダーです。

そこで、ニューラルネットワークの出番です。

### ニューラルネットワーク
簡単に説明すると関数を予測する構造を持ちます。例えるならば、グラフに点をプロットしていくと、その点を通るような関数を予測してくれます。ニューラルネットワークの中では、偏微分やら何やらを使って(誤差逆伝播法やBackPropagationで調べると出てきます)、誤差が小さくなるようにパラメータが調節されます。

{{<figure src="./img/net.jpg" width=80% title="ニューラルネットワークのイメージ" >}}

ちなみにこの図ですが、卒研の中間発表の際に使ったものを持ったきたものです。中間発表は良い思い出ではないので、あまりこの図を見たくありません。

[^1]:空白の上にぷよが置かれる場合が含まれてしまっているため、正確にはもうちょっと少ないですが、大きい数であることには間違いないです。

これを使うことで、全パターンの状態を試さなくても、$Q$関数の値を予測してくれるんじゃないか、というのがDeep Q-Network(DQN)のアイデアです。



## ぷよぷよの実装
ぷよぷよを学習させるためにはぷよぷよのシステムを作らなければなりません。作りました。

DQNにおいて必要なのは「行動を与えたときに、次の状態と報酬を返すメソッド」です。これさえあればDQNに突っ込めば学習ができます。[OpenAIGym](https://www.gymlibrary.dev/)に倣って、そのゲームの「環境」クラスをインスタンス化して使えるようにします。

以下は主要部分だけ抜粋したものです。
```python
def step(self, action):
    reward = 0
    done = False
    win = False
    self.prev_action = action
    puyo = self.puyo_list[self.turn]
    self.turn += 1

    isDropped = self.drop_puyo(action, puyo)

    if not isDropped:
        # 落とせなかったら負け
        done = True
        reward = -1
    else:
        _dis , _chain = self.process()
        if self.turn >= self.turn_max:
            # 全ターン経過で勝ち
            done = True
            win = True
            reward = 1
    # print(self.board)
    return self.states(), reward, done, self.info(), win
```
`step()`が「行動を与えたときに次の状態と報酬を返す」メソッドです。



## 学習方法
[『ゼロから作るDeep Learning ❹ 強化学習編』(オライリー・ジャパン)のサポートサイトのコード](https://github.com/oreilly-japan/deep-learning-from-scratch-4/blob/master/pytorch/dqn.py)をベースに書きました。機械学習ライブラリのPyTorchを使っています。

### 状態
状態としてエージェントに与えたい要素は次の通りです。
- 盤面の状態
- 次に落とすぷよ$3$つ

これをもとに状態を返すメソッドを書きます。

```python
def states(self):
    # 現在の盤面の状態
    state = []
    for row in range(self.height):
        for col in range(self.width):
            state.append(self.board[row][col])
    # 次の3ターンで降るぷよの色
    for i in range(3):
        state.append(self.puyo_list[self.turn+i][0])
        state.append(self.puyo_list[self.turn+i][1])
    # print(state)

    assert len(state) == STATE_SPACE

    # dtypeをfloat32に変換
    state = torch.tensor(state, dtype=torch.float32)
    return state
```

これらをすべて1次元ベクトルとして入力にします。$2$次元の盤面をどう$1$次元に変換させるのかというと、単純に各行を横につなげただけです。

### epsilon-greedy法
epsilon-greedy法は学習テクニックの一つで、学習初期にはランダムに行動を選択し、学習が進むにつれてランダムに行動を選択する確率を下げていく方法です。これを使うことで、学習初期にはランダムに行動を選択することで、より多くの状態を試し、偏りなく学習させることが期待されます。


## 学習
ソースコードは[Gist](https://gist.github.com/Shibaken28/5b794fc0ad75ed42e5402b6a8a8135ef)に置いてあります。
### 実験A 生き残れるか
#### 条件
次の条件で学習させます。
- 盤面の大きさは縦$8$横$6$の計$48$マス
- $30$回ぷよを落とすことができればゲームクリア
	- 少なくとも、$30\times 2=60$個のうち$60-48=12$個は消さなければならない

- ゲームクリアの場合報酬$1$、ゲームオーバーの場合報酬$-1$を与える

#### 結果
$2000$回の試行を行ったところ、最終的には$4$から$6$割程度の成功率となりました。
{{<figure src="./img/servive.png" width=60% title="実験Aの直近100回の成功率" >}}
なお、$1000$回らへんで急激に成功率が上がっているのは、epsilon-greedy法の影響です。学習初期にはランダムに行動を選択する確率が高いため、成功率が極端に低くなっています。

学習の効果が出ているのかを確かめるため、試しに全てランダムに行動させてみました。すると、成功率は$1$割にも満たない結果となりました。
よって、精度はともかく、少なくとも学習はできていると言えます。サルより強い。
{{<figure src="./img/servive-saru.png" width=60% title="実験Aの直近100回の成功率(ランダム)" >}}


### 実験B 2連鎖以上を起こせるか
#### 条件
次の条件で学習させます。
- 盤面の大きさは縦$8$横$6$の計$48$マス
- $2$連鎖以上を起こした時点でゲームクリア
- ただし、$30$回ぷよを落とした時点で$2$連鎖以上が起こっていなかった場合ゲームオーバー
- ゲームクリアの場合報酬$1$、ゲームオーバーの場合報酬$-1$を与える

#### 結果
成功率は案外伸びずに2割程度でした。
{{<figure src="./img/2.png" width=60% title="実験Bの直近100回の成功率" >}}
一応ランダムの場合は成功率が1割未満なため、学習自体はできているようです。
{{<figure src="./img/2-saru.png" width=60% title="実験Bの直近100回の成功率(ランダム)" >}}



### 実験C ばよえ～～ん
#### 条件
次の条件で学習させます。
- 盤面の大きさは縦$12$横$6$の計$96$マス
- $7$連鎖を起こした時点でゲームクリア
- ただし、$100$回ぷよを落とした時点で$7$連鎖以上が起こっていなかった場合ゲームオーバー
- 報酬は、起こした連鎖数の$2$乗
    - これにより、「$1$連鎖$2$回」よりも「$2$連鎖$1$回」の方が良く評価される

要するに[ばよえ～ん](https://dic.nicovideo.jp/a/%E3%81%B0%E3%82%88%E3%81%88%E3%80%9C%E3%82%93)を唱えられればクリアです


#### 結果

$5000$回の試行のうち、$11$回成功しました。成功率は非常に低いです。

$1600$回目の試行で$7$連鎖を起こすことができました。

{{< iframe-puyo url="https://www.pndsng.com/puyo/view.html?a11dadedbdbdbceadecbeadcdebae2dbcacdb2cabcdcdaecb2cd2ecbedc2bec2bceb2d" >}}

別のパターン($2593$回目)。※ゲームオーバーの判定は原作と少し違い、「盤面の1番上の行にぷよが既に存在していて、さらにその上に置こうとするとゲームオーバー」となっています。

{{< iframe-puyo url="https://www.pndsng.com/puyo/view.html?a7d2a2dae3aea2cdaea2ce2ba2dbdca2ececa2ecbeaeb2dcabd2edac2edead2ecd2e2dec" >}}

$4905$回目の試行。こちらは$8$連鎖起こっていますし、残ったぷよの数が少なめです。

{{< iframe-puyo url="https://www.pndsng.com/puyo/view.html?a6baeba2caeda4deba3bcea2ecbca2cdecabcecdbecedebd3ce2bc2ecb2d2b2de2dc2e" >}}

報酬の総和の平均は次のようになりました。
{{<figure src="./img/7.png" width=60% title="実験Bの直近100回の報酬の総和の平均" >}}


## 考察
成功率がそこまで高くならないものの、ランダムの場合よりは良い結果が得られているのは確かではあります。ただ、ぷよぷよというゲームの性質上、次のような行動をすれば案外連鎖を起こせてしまいます。
- 盤面から溢れない程度にぷよを積んでおく
- 盤面がほとんどぷよでいっぱいで、ゲームオーバーになりそうな場合は適当なぷよを消す
- すると、それなりに連鎖が起こる場合がある

いわゆる「カエル積み」です。強化学習によって、「盤面から溢れない程度に置く」「満杯になったら適当に消す」ことが良いと認識され、「カエル積み」が生まれているのではないか、という考えです。

実験Cではカエル積みが何度も試行され、試行回数を大量に重ねた結果、$7$連鎖を起こすことができたと考えます。

## 感想と謝罪文と言い訳
遅れてしまいました。ごめんなさい！！！！

元々この記事はZennに投稿する予定だったのですが、学習の結果があんまりぱっとしなかったことや、強化学習への理解が足りないことなどの理由から、記事のクオリティが低くなりそうだったので、個人ブログに投げることにしました。個人ブログは何をやっても許されるので便利ですね。


感想ですが、機械学習は難しいですね。私が普段触れている競プロやアルゴリズムは、厳密さや再現性が重要視される分野であることに対し、機械学習は「ニューラルネットワークを使うとなんかうまくいく」とか、「パラメータを調節すると学習率が上がる」みたいな、どこかふんわりとした印象があります。手元で調整したパラメータが、直接結果に現れるのではなく、「ニューラルネットワーク」という得体の知れないものを通して出てくる、謎の仲介者がいる、直接やり取りしたいのに、という気持ちになります。そこがかなりとっつきづく、私はまだまだ機械学習について理解が足りないと感じました。

