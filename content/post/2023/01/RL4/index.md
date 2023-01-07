---
title: "三目並べを様々な強化学習で実装 #4 モンテカルロ法"
description: 
slug: "RL-4"
date: 2023-01-07T13:00:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: true
---

## モンテカルロ法とは

### 動的計画法の問題点
　動的計画法は，状態遷移確率($p$)と報酬関数($r$)が既知である必要があった．他にも，エピソードの途中の状態を再現しなければならない点も厄介であった．

### モンテカルロ法の概要
　モンテカルロ法のアイデアは，たくさんのサンプリングを行って，それの平均値を取ることで，期待値を近似するというものである[^1].
例えば，$10$個のサイコロを振ったときの出る目の合計の期待値を解析的に求めるには，$6^{10}=60466176$通りの計算をしなければならない．モンテカルロ法によれば，実際に$10$個のサイコロを振る動作を適当な回数(例えば$1000$回)くらい繰り返して，出た目の平均を取れば十分近似できる．

[^1]:大数の法則より，サンプリング数が増えると，平均値は期待値に近づいていく．

### 状態価値関数をモンテカルロ法で求める
　状態$s$における価値$v(s)$を求めたい場合は，状態$s$から始まるエピソードをたくさん生成し，その平均を取れば良い．これはエピソードタスクでのみ可能で，連続タスクには適用できない．
エピソード$1$の収益が$G_1$，エピソード$2$の収益が$G_2$，$\cdots$，エピソード$N$の収益が$G_N$とすると，状態$s$における価値$v(s)$は以下のようになる．

$$
v(s) = \frac{G_1 + G_2 + \cdots + G_N}{N}
$$

## 実装
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
$s_1$の次の状態を$s_2$，その次の状態を$s_3$，その次が$s_4$で，そこでエピソードが終了したとすると，各状態$s_i$における収益$G_{s_i}$は式$(1)$から$(4)$で表される．
$$
\begin{align}
G_{s_1} =& r_{1} + \gamma r_{2} + \gamma^2 r_{3} + \gamma^3 r_{4} \\\\
G_{s_2} =&                r_{2} + \gamma r_{3}   + \gamma^2 r_{4} \\\\
G_{s_3} =&                         r_{3}   + \gamma r_{4} \\\\
G_{s_4} =&                                  r_{4} \\\\
\end{align}
$$
ナイーブな実装では，$s_1$から始まるエピソードの収益を計算するたびに，$s_2$から始まるエピソードの収益を計算しなければならないが，実は，式$(5)$のように，$G_{s_1}$の中には，$G_{s_2}$が含まれている．

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

さらに，今回はエピソードが終了するときにしか報酬が発生しないので，式$(10)$から$(13)$の通り，等比数列のように計算できる．
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
            int act;
            act = rand() % cells;
            if (board[act] != 0)
                continue;
            board[act] = turn + 1;
            if (judge(board) != 0)
                break;
            turn = 1 - turn;
        }
        history.push_back(board);
        double r = reward(board);
        reverse(history.begin(), history.end());
        for (const auto &h : history)
        {
            V[h] += r;
            count[h]++;
            r *= gamma;
        }
    }

    for (auto &[prevS, prevV] : V)
    {
        prevV /= count[prevS];
    }
```
価値反復法のときには相手の手番を考慮していたが，今回は簡単のために完全にランダムにする．
各状態からの試行回数をカウントし，最後に回数で割って平均をとる．

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

