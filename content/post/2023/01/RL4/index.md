---
title: "強化学習勉強メモ #4 モンテカルロ法"
description: 
slug: "RL-4"
date: 2023-01-07T13:00:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: true
---

## モンテカルロ法の説明
### 動的計画法の問題点
　動的計画法は，状態遷移確率($p$)と報酬関数($r$)が既知である必要があった．他にも，エピソードの途中の状態を再現しなければならない点も厄介であった．

### モンテカルロ法のアイデア
　モンテカルロ法のアイデアは，全ての行動を試すのが無理ならば，いくつかサンプリングを行って，それの平均値を取ることで，期待値を近似しようというものである[^1].
例えば，$10$個のサイコロを振ったときの出る目の合計の期待値を解析的に求めるには，$6^{10}=60466176$通りの計算をしなければならない．モンテカルロ法によれば，実際に$10$個のサイコロを振る動作を適当な回数(例えば$1000$回)くらい繰り返して，出た目の平均を取れば十分近似できる．

　余談ではあるが，モンテカルロ法は強化学習の分野以外でも使われる．例えば，平面上にランダムな点をたくさん打って，円の中に入っている点の数を数えることで円周率を近似する手法[^2]も，モンテカルロ法と呼ばれている．

[^1]:大数の法則より，サンプリング数が増えると，平均値は期待値に近づいていく．
[^2]:https://manabitimes.jp/math/1182

### 状態価値関数をモンテカルロ法で求める
　状態$s$における価値$v(s)$を求めたい場合は，状態$s$から始まるエピソードをたくさん生成し，その平均を取れば良い．これはエピソードタスクでのみ可能で，連続タスクには適用できない．
エピソード$1$の収益が$G_1$，エピソード$2$の収益が$G_2$，$\cdots$，エピソード$N$の収益が$G_N$とすると，状態$s$における価値$v(s)$は式$(1)$のようになる．

$$
\begin{align}
v(s) = \frac{G_1 + G_2 + \cdots + G_N}{N}
\end{align}
$$

### 最適方策を見つける
　動的計画法と同じく，状態価値関数$V$から方策を改善する**方策制御**のターンが必要である．
動的計画法と同じく，greedy化を行って方策制御を行うことができる．箇条書きにすると次のようになる．

1. 初期の適当な方策$\pi$を決める
1. 十分な回数次を繰り返す．
    1. モンテカルロ法によりエピソードを$1$回通す
    1. greedy化を行い$\pi$を更新する(方策制御)

ただし，このままだといくつか問題点があるため，次の手法で改善する．

#### 行動価値関数を求める
　状態価値関数$v(s)$を求めていたが，これを行動価値関数$q$に変更する．状態価値関数から最適方策を求めようとすると，式$(2)$のようになる．
$$
\begin{align}
\mu(s) &= \text{argmax} \_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma v (s') \right\\}
\end{align}
$$
これには，$v$に加えて，$p,r$の情報が必要になってしまう．$p,r$を直接求めることが困難な場合，最適方策がわからない．そもそも，$p,r$を直接求めることが困難であったから，ランダムにエピソードを走らせるモンテカルロ法をしているのであり，ここで$p,r$が必要になってしまったら本末転倒である．

　代わりに$q$関数を使うとどうだろうか．式$(3)$のように$p$も$r$も必要なく非常にシンプルである！
$$
\begin{align}
\mu(s) &= \text{argmax} \_{a} q (s,a)
\end{align}
$$

求めるものが$v$から$q$へと変わったが，モンテカルロ法の適用のさせ方はほとんど変わらず，始めの状態$s$に加えて最初の行動$a$を記憶しておくだけである．

#### $\varepsilon$-greedy法 (方策評価フェーズ)
　モンテカルロ法で方策の評価をした後，単にgreedy化し，それを使ってまたモンテカルロ法を行うのは危険である．
なぜなら，モンテカルロ法はあくまでサンプリングをしているのであり，収益が悪いサンプルを引いてしまうとgreedy化しても悪い方策が出てくるだけである．そこで，greedy化された方策に加え，たまにランダムな動きをするように修正を加えると，微妙な局所解に陥ってしまうことを逃れられる．
具体的には，確率$\varepsilon$でランダムな行動を選択し，確率$1-\varepsilon$でgreedy化した行動を選択する．

#### 固定値$\alpha$方式
　$Q$値の更新を式$(4)$で行っても良いが，これに疑問を抱く．
$$
\begin{align}
Q(s,a) = \frac{G_1 + G_2 + \cdots + G_N}{N}
\end{align}
$$
方策の評価と改善(制御)を交互に行うことで，その方策は段々と最適なものに近づいていくと考えられる．例えば，初期の頃ある行動は$G_1=-4$と評価されていたが，方策の改善が進むに連れ，$G_{10}=16,G_{11}=15$と実はもっと良い行動であったとわかったとする．このときに式$(4)$で$Q$値を更新すると，$G\_1,G\_2,\cdots G\_N$の値は全て平等に扱われる．より良い方策によって求まった$G_{10}$や$G_{11}$の方が重要な情報であるのだから，最新の$G$であればあるほど重みをつけて$Q$値を更新すべきである．
最新の情報に重みを置いた$Q$値の計算方法を式$(5)$に示す．$0\leq \alpha \leq 1$である．式
$(6)$は式$(5)$と等価である．
$$
\begin{align}
Q(s,a) &\leftarrow Q(s,a) + \alpha(G - Q(s,a)) \\\\
Q(s,a) &\leftarrow (1-\alpha)Q(s,a) + \alpha G
\end{align}
$$
これだけだと何が起こっているのかわかりにくいので，$Q_k$を$k$回目の$Q$値の値として$G\_1,G\_2,G\_3,\cdots$の値で更新していく様子を見る．式$(6)$を使う．
$$
\begin{align}
Q_0 &= 0 \\\\
Q_1 &= (1-\alpha)Q_0 + \alpha G_1 = \alpha G_1 \\\\
Q_2 &= (1-\alpha)Q_1 + \alpha G_2 = (1-\alpha)\alpha G_1 + \alpha G_2 \\\\
Q_3 &= (1-\alpha)Q_2 + \alpha G_3 = (1-\alpha)^2\alpha G_1 + (1-\alpha)\alpha G_2 + \alpha G_3 \\\\
Q_k &= \sum_{i=0}^{k-1}(1-\alpha)^i\alpha G_{k-i}
\end{align}
$$
このように，$Q_k$の更新が終わったとき，
- $G_k$の重みは$\alpha$
- $G_{k-1}$の重みは$(1-\alpha)\alpha$
- $G_{k-2}$の重みは$(1-\alpha)^2\alpha$
- $\cdots$

と，古い情報であればあるほど重みが小さくなる．
このように，**新しい情報に重みをおきたいときに固定値$\alpha$方式**は有効である．



## 方策の種類
　これまで方策と呼んできたものは，次のように分類することができる．
- いまから改善していく方策 = **ターゲット方策**
- エージェントが実際に行動するときの方策 = **挙動方策**

一つの方策がターゲット方策と挙動方策の両方の役割を持つ場合，それは**方策オン型**のアプローチと呼ぶ．逆に，別々の方策の場合は**方策オフ型**と呼ぶ．前回のモンテカルロ法は，始めに決めた方策に従ってエピソードを走らせ，その方策を改善し，改善された方策でまたエピソードを走る，というように一つの方策で行っているため，方策オン型である．

　では，方策オフ型のモンテカルロ法があったとすると，それは，挙動方策が探索を行い，ターゲット方策にはそれを活用させるようなものである．このとき，挙動方策から得られたサンプリングデータ$x$は確率分布$b(x)$に従うが，実際のターゲット方策は確率分布$\pi(x)$に従うという状況になる．このとき，$\pi$における$x$の期待値$\mathbb{E}\_\pi[x]$を求めるには少し工夫が必要である．

### 重点サンプリング
　この問題を数学的に表現すると，次のようになる．
> ある確率分布$\pi(x)$の期待値$\mathbb{E}_\pi[x]$を，別の確率分布$b(x)$からのサンプリングで得られたデータでどう表現するか．

次の式変形を行う．
$$
\begin{align} 
\mathbb{E}\_\pi &= \sum x\pi(x) \nonumber \\\\
&= \sum x\pi(x) \frac{b(x)}{b(x)} \nonumber \\\\
&= \sum x\frac{\pi(x)}{b(x)} b(x) \nonumber \\\\
&= \mathbb{E}\_b \left[x \frac{\pi(x)}{b(x)} \right] \nonumber
\end{align}
$$
これにより，$\pi(x)$の期待値は，$x$に$\pi(x)$と$b(x)$の比をかけたものの期待値と等しいことがわかる．これだけだとありがたみが分かりづらいので具体例としてモンテカルロ法を説明する．

### 方策オフ型のモンテカルロ法
　重点サンプリングを用いることで，ある行動をする確率分布を，ターゲット方策をgreedyな方策の$\pi$，
挙動方策を$\varepsilon$-greedyな方策の$b$に設定する，という方策オフ型のアルゴリズムの実現が可能になる．
まず，簡単な例として，次のように時刻$S_t$から行動$A_t$を取り，報酬$R_t$を得て，状態$S_t$で終了したとする．これを$\mathrm{trajectory}$と呼ぶことにする．
$$
\mathrm{trajectory} = S_{t} \rightarrow A_{t} \rightarrow R_{t} \rightarrow S_{t+1}
$$
これが方策$\pi$で行った結果であるのなら収益$G_t$が得られ，これがたくさん得られれば，それらの平均を取り，$\mathbb{E}[G_t]$の計算にそのまま使用できる．一方，方策$b$で行った場合であればそうはいかない．
方策$\pi$で行った場合，この$\mathrm{trajectory}$を取る確率は次のようになる．
$\mathrm{Pr}(\mathrm{trajectory}|\pi)$は方策$\pi$のときに一連の流れ$\mathrm{trajectory}$が起こる確率を表す．
$$
\mathrm{Pr}(\mathrm{trajectory}|\pi) = p(S\_{t+1}|S\_{t},A\_{t})\pi(A\_{t}|S\_{t})
$$
一方，方策$b$で行った場合，この$\mathrm{trajectory}$を取る確率は次のようになる．
$$
\mathrm{Pr}(\mathrm{trajectory}|b) = p(S\_{t+1}|S\_{t},A\_{t})b(A\_{t}|S\_{t})
$$

よって，$G_t$の期待値は次のようになる．
$$
\mathbb{E}\_{\pi}[G\_{t}] = \mathbb{E}\_{b} \left[ G\_{t} \frac{\mathrm{Pr}(\mathrm{trajectory}|\pi)}{\mathrm{Pr}(\mathrm{trajectory}|b)} \right]
 = \mathbb{E}\_{b} \left[ G\_{t} \frac{\pi(A\_{t}|S\_{t})}{b(A\_{t}|S\_{t})} \right]
$$

一般化する．時刻$t$から始まったエピソードは時刻$T$の状態$S_T$で終了したとする．
$$
\mathrm{trajectory} = S_{t} \rightarrow A_{t} \rightarrow R_{t} \rightarrow S_{t+1} \rightarrow A_{t+1} \rightarrow R_{t+1} \rightarrow \cdots \rightarrow S_{T-1} \rightarrow A_{T-1} \rightarrow R_{T-1} \rightarrow S_{T}
$$
先ほどと同様に，各方策で$\mathrm{trajectory}$を取る確率は次のようになる．
$$
\mathrm{Pr}(\mathrm{trajectory}|\pi) = \prod_{\tau=t}^{T-1} p(S\_{\tau+1}|S\_{\tau},A\_{\tau})\pi(A\_{\tau}|S\_{\tau}) \\\\
\mathrm{Pr}(\mathrm{trajectory}|b) = \prod_{\tau=t}^{T-1} p(S\_{\tau+1}|S\_{\tau},A\_{\tau})b(A\_{\tau}|S\_{\tau})
$$
よって，得られた$G_t$に
$$
\frac{\mathrm{Pr}(\mathrm{trajectory}|\pi)}{\mathrm{Pr}(\mathrm{trajectory}|b)} = \prod_{\tau=t}^{T-1} \frac{\pi(A\_{\tau}|S\_{\tau})}{b(A\_{\tau}|S\_{\tau})}
$$
を掛け算し，それらの平均を取れば方策$\pi$の期待値に近似できる．





## 三目並べの実装
　$Q$値を$\varepsilon$-greedy法と固定値$\alpha$方式を使いながら求めるプログラムを本当は書きたかったのだが，うまくいかなかった(うまくいかない理由がよくわかっていないのでいつかやりたい)．

完成版のソースコードは[こちら](https://gist.github.com/Shibaken28/db7f1f7c0e992bfdfab7d8c52d5bf5b8)．
　
ここから下にあるのは方策制御を行わないモンテカルロ法で，三目並べを実装したものである．
### 雛形
　モンテカルロ法は，適当な回数だけ試行を行うため，プログラムの形は次のようになる．
```cpp
int main(){
    // コンピュータは常にoとする
    //  V[s] := 状態sにおける価値
    map<Board, double> V;
    map<Board, int> count;
    
    //モンテカルロ法
    const int N = 100000;
    double gamma = 0.9;
    for(int i=0;i<N;i++){
        Board board = {0, 0, 0, 0, 0, 0, 0, 0, 0};
        // エピソードを生成，収益を計算
    }
```
これだと一番始めの状態からの試行しかできないのではないかと思われるかもしれないが，途中の状態における収益も，同時に計算を行うことができる．

### 収益の計算
　$G_{s}$を状態$s$から始まるエピソードの収益とする．
$s_1$の次の状態を$s_2$，その次の状態を$s_3$，その次が$s_4$で，そこでエピソードが終了したとすると，各状態$s_i$における収益$G_{s_i}$は式$(12)$から$(15)$で表される．
$$
\begin{align}
G_{s_1} =& r_{1} + \gamma r_{2} + \gamma^2 r_{3} + \gamma^3 r_{4} \\\\
G_{s_2} =&                r_{2} + \gamma r_{3}   + \gamma^2 r_{4} \\\\
G_{s_3} =&                         r_{3}   + \gamma r_{4} \\\\
G_{s_4} =&                                  r_{4} \\\\
\end{align}
$$
ナイーブな実装では，$s_1$から始まるエピソードの収益を計算するたびに，$s_2$から始まるエピソードの収益を計算しなければならないが，実は，式$(16)$のように，$G_{s_1}$の中には，$G_{s_2}$が含まれている．

$$
\begin{align}
G_{s_1} = r_{1} + \gamma G_{s_2}
\end{align}
$$

同様に，$G_{s_2}$の中には，$G_{s_3}$が含まれ，$G_{s_3}$の中には，$G_{s_4}$が含まれている．よって，エピソードが終了したときに，各報酬と各状態を逆順にたどっていけば，各状態における収益を計算することができる．

$$
\begin{align}
G_{k} &= r_{k}\\\\
G_{s_{k-1}} &= r_{k-1} + \gamma G_{s_{k}} \\\\
\cdots \nonumber \\\\
G_{s_2} &= r_{2} + \gamma G_{s_3} \\\\
G_{s_1} &= r_{1} + \gamma G_{s_2}
\end{align}
$$

さらに，今回はエピソードが終了するときにしか報酬が発生しないので，式$(21)$から$(24)$の通り，等比数列のように計算できる．
$$
\begin{align}
G_{s_1} = \gamma^0 r \\\\
G_{s_2} = \gamma^1 r \\\\
G_{s_3} = \gamma^2 r \\\\
\cdots \nonumber \\\\
G_{s_{k}} = \gamma^{k-1} r 
\end{align}
$$

これに基づき，次のように実装する．

```cpp
    map<Board, double> V;
    map<Board, int> count;

    const int N = 100000;
    double gamma = 0.9;
    for (int i = 0; i < N; i++)
    {
        Board board = {0, 0, 0, 0, 0, 0, 0, 0, 0};
        vector<Board> history;
        int turn = 0;
        while (true)
        {
            history.push_back(board);　
            int act
            while(true){
                act = rand() % cells;
                if (board[act] == 0)
                    break;
            }
            board[act] = turn + 1; //自分の記号を置く
            if (judge(board) != 0)
                break;
            turn = 1 - turn; //次の手番
        }
        history.push_back(board);
        double r = reward(board);
        reverse(history.begin(), history.end()); //逆順にする
        for (const auto &h : history)
        {
            V[h] += r;
            count[h]++;
            r *= gamma; //割引率をかける
        }
    }

    for (auto &[prevS, prevV] : V)
    {
        prevV /= count[prevS];
    }
```
価値反復法のときには相手の手番を考慮していたが，今回は簡単のために完全にランダムにする．
各状態からの試行回数をカウントし，最後に回数で割って平均をとる．

　プログラムを詳しく解説していく．
`while`文では，ランダムな空いている場所を選ぶまで繰り返している[^3]．
空いている場所を列挙して，その中からランダムに選ぶ実装でも良いが，今回は実行時間に余裕があるので実装をサボった(読者への課題としよう！)．
```cpp
            while(true){
                act = rand() % cells;
                if (board[act] == 0)
                    break;
            }
```

[^3]:最後の$1$マスは$1/9$の確率を引くまでループし続ける．マス数が多くなると当然空いているマスを選ぶまで時間がかかるので注意．

`history`には，盤面の情報$s_1,s_2,s_3,\cdots,s_n$が順番に記録されていく．一番最後の盤面の報酬が$r$だったとき，各盤面の収益は，後ろから$r,\gamma r,\gamma^2 r,\cdots$となる．
```cpp
        double r = reward(board);
        reverse(history.begin(), history.end()); //逆順にする
        for (const auto &h : history)
        {
            V[h] += r;
            count[h]++;
            r *= gamma; //割引率をかける
        }
```

### 最適方策
　最も価値が高くなるような行動を選び，最適方策$\mu$を決定する．
```cpp
    map<Board, int> mu;
    for (auto &[prevS, prevV] : V)
    {
        if (!isVaild(prevS))
            continue;
        int act;
        double nextV = -1e9;
        for (int i = 0; i < cells; i++)
        {
            if (prevS[i] != 0)
                continue;
            Board nextS = prevS;
            nextS[i] = 1;
            cout << i << " : " << V[nextS] << endl;
            if (V[nextS] > nextV)
            {
                nextV = V[nextS];
                act = i;
            }
        }
        mu[prevS] = act;
        cout << "mu [" << endl;
        print_board(prevS);
        cout << "] = " << mu[prevS] << endl
             << endl;
    }
```

### 結果 
各状態の価値を表示すると，以下のようになる(抜粋)．
```cpp
0 : -0.121793
4 : -0.20035
5 : -0.662595
mu [
 |x|x
o| | 
x|o|o
] = 0

0 : 0.323512
4 : 1
6 : -0.0316175
7 : -0.0757296
8 : -0.291569
mu [
 |x|x
o| |o
 | | 
] = 4
```
対戦してみると，以下のようになる．
```cpp
 | | 
 | | 
 | | 

 | | 
 |o| 
 | | 
put:5

 | | 
 |o|x
 | | 

 | | 
 |o|x
 | |o
put:0

x| | 
 |o|x
 | |o

x| | 
 |o|x
o| |o
put:7

x| | 
 |o|x
o|x|o

x| |o
 |o|x
o|x|o
```
初手は真ん中に置いてきて，こちらが辺に置くと相手は勝ってくれる．




