---
title: "強化学習勉強メモ #3 動的計画法"
description: 
slug: "RL-3"
date: 2023-01-06T22:01:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: true
---

## 動的計画法のアルゴリズム


### 反復方策評価
　ベルマン方程式を使って実際に各$s$における状態価値関数$v(s)$の値を正確に求めることは，状態数が増えていくにつれ，計算量の問題により難しくなる．そのため，$v(s)$の値をより正確な値に近づけていくような，値を**更新**させていくような，別の手法を考える．

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
このような，方策を求めるフェーズを**方策制御**と呼ぶ．

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

混乱しやすいので箇条書きで同じことを書くと，次のようになる．

1. 初期の適当な方策$\pi$を決める
1. 全ての状態$s$に対して$V(s)$を更新する(方策評価)
1. greedy化を行い$\pi$を更新する(方策制御)
1. $V$を更新したときの差分が十分小さくなっていなければ 2. に戻る

### 価値反復法
　**価値反復法**は，方策反復法で別々に行っていた反復方策評価とgreedy化を同時に行う手法である．式で表すと，次のようになる．
$$
\begin{align}
V\_{k+1}(s) = \max_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k} (s') \right\\}
\end{align}
$$
これによって更新がストップした時点で，greedy化を行えば最適方策$\mu\_*(s)$が得られる．
今回は，この価値反復法を用いて実装を行う．

## 三目並べの実装
　言語はC++を使う．また，`include <hoge>`や`using namespace std;`などの記述を省略している．全体のソースコードは[gist](https://gist.github.com/Shibaken28/41d44940567dac6dd8721e549af46d46)に公開している．

コンパイルは，`g++ -std=c++17 hoge.cpp`で行う．Makefileを適当に作っておくと便利．
```
# g++ の c++17でコンパイル
CXX = g++
CXXFLAGS = -std=c++17 -Wall -Wextra -Wpedantic
```
### ルールの確認
- 盤面は$3\times 3$のマスで構成される
- 先手はマル，後手はバツ
- 交互に空いているマスに自分の記号を置く
- 縦横斜めのいずれかのラインに$3$つ自分の記号が並んだ時点で勝ち
- 盤面が埋まっていても勝敗がつかない場合は引き分け

また，今回は，先手を学習することにする．

### アルゴリズムの確認
　各変数を三目並べに適用すると，次のようになる．
- 環境$s$ : 盤面
- 行動$a$ : 手
- 報酬$r(s,a,s')$ : 勝敗
- 遷移確率$p$ : ?

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
簡単のために，コンピュータはマルのみを打つことにする．ちなみに，コンピュータがバツを打つ場合は，盤面のマルとバツを入れ替えることで同じコードで実装できる．

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
　さて，$3^9$通りの盤面を全て生成すると，全てのマスがマルで埋まっているような盤面であったり，マルとバツが両方揃っているような盤面であったり，そもそも存在しない盤面まで生成されてしまう．そんな盤面を除外するために，`is_valid`という関数を作る(残しておいても問題がないのかもしれないが，なんとなく)．また，`judge(board) != 0`のときは，その盤面は終了盤面であるので，その盤面は除外する．
今回は，先手を学習するため，マルとバツの個数が同じ盤面のみ考える．
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
    return cnt == 0;
}
```

状態価値関数$v$は，`std::map`を用いて実装する．
`map<Board, double>`とすると，`Board`をキーとして，`double`を値として扱うことができる．全ての有効な状態で，$v(s) = 0$としておく．
全ての盤面の生成にはカッコよく再帰関数を用いる．
```cpp
void initAllBoard(Board &board, vector<Board> &allBoard,int depth=0)
{
    if (depth == cells)
    {
        if (isVaild(board))
        {
            allBoard.push_back(board);
        }
        return;
    }
    for (int i = 0; i < 3; i++)
    {
        board[depth] = i;
        initAllBoard(board, allBoard, depth + 1);
    }
}


int main(void)
{
    // コンピュータは常にoとする
    //  V[s] := 状態sにおける価値
    map<Board, double> V;
    // 全ての状態を生成
    vector<Board> allBoard;
    Board empty = {0, 0, 0, 0, 0, 0, 0, 0, 0};
    initAllBoard(empty, allBoard);
    for (const auto &b : allBoard)
    {
        V[b] = 0.0;
    }
```

### 学習
　学習フェーズの雛形は以下のようになる．完全に値の更新がしなくなるまで回すというのは，浮動小数点型の精度の問題から難しいため，ある程度の差分が出なくなったら終了するということにする．
式$(6)$の通りに，`maxAction`関数で，$\sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k} (s') \right\\}$が最大になる行動$a$を選択する．

$$
\begin{align}
V\_{k+1}(s) = \max_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k} (s') \right\\}
\end{align}
$$
```cpp
    double gamma = 0.8;
    double threshold = 1e-5; // 閾値;更新したときの差分がこの値以下になったら終了
    while (true){
        double max_diff = 0.0;
        /*
        全ての状態sについて，価値を更新する
        */
        for (auto &[prevS, prevV] : V)
        {
            auto [act, nextV] = maxAction(prevS, V, gamma);
            //maxActionは最も収益の高い行動とそのときの価値を返す
            double diff = abs(prevV - nextV);
            max_diff = max(max_diff, diff);
            V[prevS] = nextV;
        }
        if (max_diff < threshold)
        {
            break;
        }
    {
```

### 状態価値関数の更新
　`maxAction`関数を作る．ここで，注意するのは，相手の行動を考慮する必要があることである．
行動$a$を選択し，さらに相手ターンが終了したときの盤面が$s'$となる．そこで，相手の行動を全て試して，それらの状態価値の平均を取る．

少し複雑なので，言語化すると以下のようになる．

1. 最大の価値とそのときの行動を$v_{max} = -\infty$，$a_{max} = -1$で初期化する．
1. 現在の状態$s$からできる全ての行動$a$について次を行う．
    1. 行動$a$を選択したときの盤面を$t$とする．
    1. $V_{k+1} = 0$と初期化する．
    1. $t$からできる全ての相手の行動について次を行う．行動の数を$n$とする．
        1. 相手の行動を選択したときの盤面を$s'$とする．
        1. $V_{k+1}$に$r(s,a,s') + \gamma V(s')$を加算する
    1. $V_{k+1}$を$n$で割って平均にする．
    1. $V_{k+1}$が$v_{max}$より大きければ$v_{max} = V_{k+1}$，$a_{max} = a$とする．

ただし，相手の置く場所がなかった場合はその時点の状態の報酬をそのまま返す．

数式で表現すると，`maxAction`関数は，それぞれ式$(7)(8)$を返していることになる．
$$
\begin{align}
\text{argmax}_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k} (s') \right\\} \\\\
\max\_{a} \sum \_{s'} p(s'|s,a) \left\\{ r(s,a,s') + \gamma V\_{k} (s') \right\\}
\end{align}
$$

```cpp
pair<int,double> maxAction(const Board &prevS,const map<Board, double> &V,const double gamma)
{
    double maxV = -1e9;
    int act = -1;
    for (int i = 0; i < cells; i++)
    {
        if (prevS[i] != 0) continue;
        // 置いたときの盤面
        Board nextS = prevS;
        nextS[i] = 1;

        double nextV = 0;

        vector<int> put;//相手の置ける場所
        for(int j=0;j<cells;j++){
            if(nextS[j]==0){
                put.push_back(j);
            }
        }
        for(const auto &p : put){
            nextS[p] = 2;
            nextV += reward(nextS) + gamma * V.at(nextS);
            nextS[p] = 0;
        }
        if(put.empty()){//相手の置ける場所がないとき
            nextV = reward(nextS);   
        }else{
            nextV /= put.size();
        }

        if (nextV > maxV)
        {
            maxV = nextV;
            act = i;
        }
    }
    return {act,maxV};
}
```

### 最適方策の取得
　以上のプログラムによって，状態価値関数が求まったので，最適方策を取得する．
これは，`maxAction`関数の戻り値を使えば実装できる．
```cpp
    map<Board, int> mu;
    for (auto &[prevS, prevV] : V)
    {
        auto [act, nextV] = maxAction(prevS, V, gamma);
        mu[prevS] = act;
    }
```

適当なコードで出力してみる．
```cpp
    for(auto &[prevS, prevV] : V){
        cout<<"mu ["<<endl;
        print_board(prevS);
        cout<<"] = "<<mu[prevS]<<endl<<endl;
    }
```

出力結果(抜粋)は以下の通り．
```cpp
mu [
 | | 
 |o| 
x|o|x
] = 1

mu [
 | | 
 |o| 
x|x|o
] = 0

mu [
 | | 
 |o|o
 |x|x
] = 3

mu [
 | | 
 |o|o
x| |x
] = 3
```
次に置くべき場所が出力されている．

こういう感じの，次に相手の置く場所を阻止しないと行けない場合もしっかりと出力されている．
```cpp
mu [
 |x| 
x|o|o
x|o| 
] = 0

mu [
 | |x
 |x|o
 |o| 
] = 6
```

対戦してみると普通に賢い．
```cpp
 | | 
 | | 
 | | 

o| | 
 | | 
 | | 
put:8

o| | 
 | | 
 | |x

o| |o
 | | 
 | |x
put:1

o|x|o
 | | 
 | |x

o|x|o
 | | 
o| |x
put:3

o|x|o
x| | 
o| |x

o|x|o
x|o| 
o| |x
```


