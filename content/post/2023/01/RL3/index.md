---
title: "三目並べを様々な強化学習で実装 #3 動的計画法"
description: 
slug: "RL-3"
date: 2023-01-06T22:01:36+09:00
categories:
    - 強化学習
tags:
image: 
draft : true
---
# 注：この記事は完成していません

## 動的計画法のアルゴリズム
　動的計画法は，状態数が少ない場合に有効な手法である．

### 反復方策評価
　ベルマン方程式を使って実際に各$s$における状態価値関数$v(s)$の値を求めることは，状態数が増えていくにつれ，難しくなる．そのため，$v(s)$の値をより正確な値に近づけていくような，値を**更新**させていくよな，手法を考える．

もう一度，式$(1)$の行動価値関数のベルマン方程式を振り返る．
$$
\begin{align}
v\_\pi(s)
&= \sum \_{a}  \sum \_{s'}\pi(a|s) p(s'|s,a) \left\\{ r(s,a,s') + \gamma v\_\pi(s') \right\\}
\end{align}
$$

このベルマン方程式を，更新の関数に置き換える．
$k$回目の更新を行った状態価値関数$V_{k}$から，$k+1$回目の更新によって得る状態価値関数を$V_{k+1}$とすると，式$(2)$のようになる．

$$
\begin{align}
V\_{k+1}(s)
&= \sum \_{a}  \sum \_{s'}\pi(a|s) p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k}(s') \right\\}
\end{align}
$$

ただし，この漸化式は，初期の関数$V_{0}$を与える必要がある．

　方策$\pi$を使い，$V_{0}$をもとにして，$V_{1}$を求め，$V_{1}$をもとにして$V_{2}$を求め，というように，$V_{k}$を求めていく．そして，収束したときにそれが状態価値関数$v_\pi (s)$になる[^1]．この手法を**反復方策評価**と呼ぶ

[^1]:証明は省略する．


### 方策反復法
　反復方策評価を使うと，方策$\pi$に対する状態価値関数$v_\pi (s)$を求めることができるが，本当に欲しいものは**最適な**方策$\pi_\*$と，その方策に対する状態価値関数$v\_\*(s)$である．

　確率的方策$\pi$から，最適な方策$\pi_\*$へ近づける鍵は，決定論的方策に変換することである．
具体的には，$v_\pi (s)$をもとに，$s$における最適な行動$a_\*$を選択する．
そして，その行動をとる確率を1に，それ以外の行動をとる確率を0にする．これにより，決定論的方策$\mu(s)$に変換される．数式で表すと，次のようになる．
$$
\begin{align}
\mu(s) &= \text{argmax} \_{a} q (s,a) \\\\
&= \text{argmax} \_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma v (s') \right\\}
\end{align}
$$
$\mu(s)$は，$v(s)$から得られた，**貪欲**的な方策，**greedy** policyと呼ばれる．**greedy 化**と呼ばれることもある．
先程説明した反復方策評価とgreedy化を交互に繰り返すと，greedy化しても方策が変わらなくなる地点に到達する．それが，最適方策となる．このような手法を**方策反復法**と呼ぶ．

### 価値反復法
　**価値反復法**は，方策反復法で別々に行っていた反復方策評価とgreedy化を同時に行う手法である．式で表すと，次のようになる．
$$
\begin{align}
V\_{k+1}(s) = \max_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k} (s') \right\\}
\end{align}
$$
これによって更新がストップした時点で，greedy化を行えば最適方策$\mu\_*(s)$が得られる．
今回は，この価値反復法を用いて実装を行う．

## 実装
言語はC++を使う．また，`include <hoge>`や`using namespace std;`などの記述を省略している．詳しいコンパイルについてはMakefileと一緒にソースコードをGitHubに置いているので，そちらを参照してほしい．
### ルールの確認
- 盤面は$3\times 3$のマスで構成される
- 先手はマル，後手はバツ
- 交互に空いているマスに自分の記号を置く
- 縦横斜めのいずれかのラインに$3$つ自分の記号が並んだ時点で勝ち
- 盤面が埋まっていても勝敗がつかない場合は引き分け

### アルゴリズムの確認
各変数を三目並べに適用すると，次のようになる．
- 環境$s$ : 盤面
- 行動$a$ : 手
- 報酬$r(s,a,s')$ : 勝敗
- 遷移確率$p$ : 三目並べは決定的な挙動を取るので，使わない．

状態$s$は，丸かバツか空白の$3$通りが$9$マス分あるので高々$3^9=19683$通りある(この中には全てが丸で埋め尽くされた盤面などが存在し，実際はもっと少ない)．この程度なら，全ての状態の価値をメモリに乗せておくことができる．よって，動的計画法が適用そうである．

　価値反復法のアルゴリズムを再掲する．

1. $V(s)$を初期化する．
1. $V(s)$が更新されなくなるまで，全ての$s$について以下を繰り返す．
    1. 状態$s$から全ての行動$a$を試す
    1. その中で最も価値の高いものを新しい価値$V(s)$とする．

### 状態の表現
$3\times 3$の$9$マスの情報を，整数で表現する．具体的には，
- 空白 : 0
- マル : 1
- バツ : 2

とする．
$3\times 3$の二次元配列に格納するのも良いが，今回は，$1$次元配列に格納することにする．

手始めに，盤面を表示する関数を作る．
```cpp
using Board = vector<int>;
constexpr int width = 3;
constexpr int height = 3;
constexpr int cells = width * height;

// 盤面を表示
void print_board(const Board &board)
{
    for (int i = 0; i < cells; i++)
    {
        if (board[i] == 0)
        {
            cout << " ";
        }
        else if (board[i] == 1)
        {
            cout << "o";
        }
        else if (board[i] == 2)
        {
            cout << "x";
        }
        if (i % width == width - 1)
        {
            cout << endl;
        }
        else
        {
            cout << "|";
        }
    }
}
```

### 盤面の判定
ゲームの進行状況を判定する関数を作る．
`win`という変数に，勝利条件を表す配列を用意しておく(この配列は盤面の`width`と`height`を変えたときに一緒に変えなければならない．謝罪)．
```cpp
int judge(const Board &board)
{
    static const vector<vector<int>> win = {
        {0, 1, 2},
        {3, 4, 5},
        {6, 7, 8},
        {0, 3, 6},
        {1, 4, 7},
        {2, 5, 8},
        {0, 4, 8},
        {2, 4, 6},
    };
    // 0: 続行
    // 1: oの勝ち
    // 2: xの勝ち
    // 3: 引き分け
    for (const auto &w : win)
    {
        if (board[w[0]] == board[w[1]] && board[w[1]] == board[w[2]])
        {
            if (board[w[0]] == 1)
            {
                return 1;
            }
            else if (board[w[0]] == 2)
            {
                return 2;
            }
        }
    }
    for (const int &b : board)
    {
        if (b == 0)
        {
            return 0; // 続行
        }
    }
    return 1; // 引き分け
}
```
一緒に報酬を返す関数も作っておく．引き分けでも正の報酬を与える．
簡単のために，コンピュータはマルのみを打つことにする．ちなみに，後述するように，コンピュータがバツを打つ場合は，盤面のマルとバツを入れ替えることで同じコードで実装できる．

```cpp
// 報酬
double reward(const Board &board)
{
    int j = judge(board);
    static const vector<double> r = {0.0, 1.0, -1.0, 0.3};
    // 0: 続行
    // 1: oの勝ち
    // 2: xの勝ち
    // 3: 引き分け
    return r[j];
}
```

### 盤面の生成
さて，$3^9$通りの盤面を全て生成すると，全てのマスがマルで埋まっているような盤面であったり，マルとバツが両方揃っているような盤面であったり，そもそも存在しない盤面まで生成されてしまう．そんな盤面を除外するために，`is_valid`という関数を作る(残しておいても問題がないのかもしれないが，なんとなく)．
マルとバツの個数を数え，マルの個数がバツの個数より1個多いか，同じ個数のときには`true`を返す．
```cpp
// 盤面が有効かどうか
bool isVaild(const Board &board)
{
    int cnt = 0;
    for (auto &b : board)
    {
        if (b == 1)
        {
            cnt++;
        }
        if (b == 2)
        {
            cnt--;
        }
    }
    return 0 <= cnt && cnt <= 1;
}
```

状態価値関数$v$は，`std::map`を用いて実装する．
`map<Board, double>`とすると，`Board`をキーとして，`double`を値として扱うことができる．全ての有効な状態で，$v(s) = 0$としておく．
この酷いコードは再帰関数を使って後で直す．
```cpp
int main(){
    // コンピュータは常にoとする
    //  V[s] := 状態sにおける価値
    map<Board, double> V;
    // 全ての状態を生成
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            for (int k = 0; k < 3; k++)
            {
                for (int l = 0; l < 3; l++)
                {
                    for (int m = 0; m < 3; m++)
                    {
                        for (int n = 0; n < 3; n++)
                        {
                            for (int o = 0; o < 3; o++)
                            {
                                for (int p = 0; p < 3; p++)
                                {
                                    for (int q = 0; q < 3; q++)
                                    {
                                        Board board = {i, j, k, l, m, n, o, p, q};
                                        if (isVaild(board))
                                        {
                                            V[board] = 0.0;
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
```

### 学習
学習フェーズの雛形は以下のようになる．完全に値の更新がしなくなるまで回すというのは，浮動小数点型の精度の問題から難しいため，ある程度の差分が出なくなったら終了するということにする．
```cpp
    double gamma = 0.8;
    double threshold = 1e-5; // 閾値;更新したときの差分がこの値以下になったら終了
    while (true){
        double max_diff = 0.0;
        /*
        全ての状態sについて，価値を更新する
        */
        if (max_diff < threshold)
        {
            break;
        }
    {
```

### 状態価値関数の更新
式$(6)$の通りに実装すると，以下のようになるが，このままだと問題がある．
$$
\begin{align}
V\_{k+1}(s) = \max_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k} (s') \right\\}
\end{align}
$$
```cpp
        double max_diff = 0.0;
        for (auto &[prevS, prevV] : V)
        {
            // cout<<"prevS:"<<endl;
            // print_board(prevS);
            double maxV = -1e9;
            if (judge(prevS) != 0)
            {
                // 終了状態
                maxV = reward(prevS);
            }
            for (int i = 0; i < cells; i++)
            {
                if (prevS[i] != 0)
                    continue;
                
                // 置いたときの盤面
                Board nextS = prevS;
                nextS[i] = 1;

                double nextV = reward(nextS) + gamma * V[nextS];
                if (nextV > maxV)
                {
                    maxV = nextV;
                }
            }
            double diff = abs(prevV - maxV);
            if (diff > max_diff)
            {
                max_diff = diff;
            }
            V[prevS] = maxV;
        }
```
問題は，$v(s)$を更新するときに，次の状態$s'$における$v(s')$を使っていることである．
この，$v(s')$というのは，相手の手番である．相手の手番の価値関数は求めていない．
そもそも，$s$が相手の手番の場合にもマルを置いてしまっていて，色々破綻している．

$v(s)$を更新するときに使う$v(s')$は，自分がマルを置いて，更に相手がバツを置いた状態の$s'$である必要がある．

状況を整理する．実装したいのは次のことだ．

- 状態$s$が，次マルを置く盤面の場合
    - マルを置いた後，相手がバツを置く
    - その状態の報酬と価値を使って，$v(s)$を更新する
- 状態$s$が，次バツを置く盤面の場合
    - バツを置いた後，相手がマルを置く
    - その状態の報酬と価値を使って，$v(s)$を更新する

問題は，
- 相手がバツを置くとき，どこに置かれるかがわからない
- バツを置く盤面の価値が必要

相手の状態価値関数を用意したくなるが，今回はコンピュータ自身の分身を使うことにする．つまり，状態価値観数は$1$つだけのままにする．

コンピュータはマルを置く体で実装をしてきたので，コンピュータがバツの場合の実装をしなければならない．これは，盤面の状態を反転させるだけで実装できる．バツの置き場所は，マルとバツを入れ替えた盤面で，マルを置いたときに最も価値関数の高い場所に置くことにする．

```cpp
Board inverse(const Board &board)
{
    Board ret = board;
    for (int i = 0; i < cells; i++)
    {
        if (ret[i] == 1)
        {
            ret[i] = 2;
        }
        else if (ret[i] == 2)
        {
            ret[i] = 1;
        }
    }
    return ret;
}
```
また，その次の手番がどちらなのかを判定する関数も必要になる．
```cpp
bool isFirst(const Board &board)
{
    int cnt = 0;
    for (int i = 0; i < cells; i++)
    {
        if (board[i] != 0)
            cnt++;
    }
    return cnt % 2 == 0;
}
```

これらを用いると，次を実装すれば良いことになる．

- 状態$s$が，次マルを置く盤面の場合
    1. マルを置く
    1. 盤面のマルとバツを反転させる
    1. 最も価値の高い状態になるような場所に，マルを置く
    1. 盤面のマルとバツを反転させる
    1. その状態の報酬と価値を使って，$v(s)$を更新する
- 状態$s$が，次バツを置く盤面の場合
    1. 盤面のマルとバツを反転させる
    1. 「状態$s$が，次マルを置く盤面の場合」を実行する
    
バツの手番である場合には，盤面を反転させることで，マルの手番と考えることができる．
なお，プログラムでは反転のタイミングがちょっと異なることに注意する(ごめんなさい)．

注意深く実装すると，次のようになる．
```cpp

    double gamma = 0.8;
    double threshold = 1e-5; // 閾値;更新したときの差分がこの値以下になったら終了
    // 価値反復法
    while (true)
    {
        double max_diff = 0.0;
        for (auto &[prevS, prevV] : V)
        {
            // cout<<"prevS:"<<endl;
            // print_board(prevS);
            double maxV = -1e9;
            if (judge(prevS) != 0)
            {
                // 終了状態
                maxV = reward(prevS);
            }
            for (int i = 0; i < cells; i++)
            {
                if (prevS[i] != 0)
                    continue;
                // 置いたときの盤面
                Board nextS = prevS;

                if (!isFirst(prevS))
                {
                    nextS = inverse(nextS);
                }

                nextS[i] = 1;
                if (judge(nextS) == 0)
                {
                    // 相手の番
                    Board nextinvS = inverse(nextS);
                    double maxoppV = -1e9;
                    int put = -1;
                    for (int j = 0; j < cells; j++)
                    {
                        // 0のときに置ける
                        if (nextinvS[j] != 0)
                            continue;
                        // bに置いたときの盤面
                        Board nextnextS = nextinvS;
                        nextnextS[j] = 1;
                        // 価値が最大のものを選ぶ
                        if (V[nextnextS] > maxoppV)
                        {
                            maxoppV = V[nextnextS];
                            put = j;
                        }
                    }
                    // 相手の番で価値が最大になる手を打ったときの盤面
                    nextS[put] = 2;
                }

                if (!isFirst(prevS))
                {
                    nextS = inverse(nextS);
                }

                double nextV = reward(nextS) + gamma * V[nextS];
                if (nextV > maxV)
                {
                    maxV = nextV;
                }
            }
            double diff = abs(prevV - maxV);
            if (diff > max_diff)
            {
                max_diff = diff;
            }
            V[prevS] = maxV;
        }
        if (max_diff < threshold)
        {
            break;
        }
    }
```


### 対戦する
これにて学習部分の実装は終わりである．
適当な対話型の対戦プログラムを書いて遊ぶことができる．

先手がコンピュータでマル，後手が人間でバツである．
```cpp
 | | 
 | | 
 | | 

o| | 
 | | 
 | | 
put:1

o|x| 
 | | 
 | | 

o|x| 
o| | 
 | | 
put:6

o|x| 
o| | 
x| | 

o|x| 
o|o| 
x| | 
put:5

o|x| 
o|o|x
x| | 

o|x| 
o|o|x
x| |o
```
なかなか賢いんじゃないかと思う．
初手に真ん中に置かないところが不思議ではあるが．


# うまくいきません助けてください

