---
title: "[WIP]ABCの高度典型を解いてうはうはしたい"
description: 
slug: high-typical
date: 2023-08-20T12:08:30+09:00
image: 
categories:
    - 競プロ
    - 日記
---

## 謝罪
常体と敬体がごっちゃです。ごめんなさい。

## ABC212-G Power Pair
### 問題概要
素数$P$が与えられ、$x^n \equiv y \pmod{P}$を満たす$n$が存在する$(x,y)$の組$(0\leq x,y\leq P-1)$の個数を求める問題。

素数$P$に対する原始根$r$が必ず存在するため、$x=r^a,y=r^b$とか表現すると、
$$
x^n \equiv y \pmod{P} \Leftrightarrow r^{an} \equiv r^b \pmod{P} \Leftrightarrow an \equiv b \pmod{P-1}
$$
となるので(最後の変換はフェルマーの小定理より)、$an \equiv b \pmod{P-1}$を満たす$(a,b)$の組$(1\leq a,b\leq P-1)$を数えれば良い。

さて、ここからどう数えるかだが、
- $P-1$と$a$の最大公約数が$1$であるような$a$を見つけてきたとき、$n=1,2,\cdots P-1$のとき$b$は$P-1$通りの値を取る。
- $P-1$と$a$の最大公約数が$2$であるような$a$を見つけてきたとき、$n=1,2,\cdots P-1$のとき$b$は$(P-1)/2$通りの値を取る。

というように、$P-1$と$a$の最大公約数が$g$であるような$a$を見つけてきたとき、$n=1,2,\cdots P-1$のとき$b$は$(P-1)/g$通りの値を取ることがわかる。

$a=1,2,\cdots P-1$のそれぞれに対して$P-1$との最大公約数を取るのは$O(P\log P)$かかってしまうので、$GCD(P-1,a)$の値が$g$になるような$a$の個数を数えることにする。これは$g$が大きい順に数えるとうまくいく(説明がめんどいのでコードを貼ってごまかす)。

```cpp
mint ans = 1;
map<long,long> mp;
for(auto&i:f){ //fはP-1の約数(降順)
    // GCD(P-1, x) = iとなるxの個数を求める
    long n = P/i;
    for(auto&j:f){
        if(j<=i)continue;
        if(j%i==0){
            n -= mp[j];
        }
    }
    mint a = P/i;
    mint b = n;
    ans += a*b;
    mp[i] = n;
}
```

[提出コード](https://atcoder.jp/contests/abc212/submissions/44847610)

### 感想
原始根って便利なんですね

## ABC212-H Nim Counting
### 問題概要

長さ$K$の数列$(A_1,A_2,\cdots,A_K)$が与えられ($1\leq A_i\leq 2^{16}$)、ここから$1$個以上$N$個以下の数を選び(重複可能)、それらの選んだ数のXORをとったとき、$0$に**ならない**ような選び方の個数を求める問題。

この問題ではxorの畳み込みという技術を使います。xorの演算子を$\oplus$、xor畳み込みの演算子を$*$とすると、xor畳み込みというのは、

$$
A = (a_1, a_2, \cdots, a_n) \\\\
B = (b_1, b_2, \cdots, b_n) \\\
c_i = \sum_{x\oplus y = i} a_x b_y
$$
となるようなベクトル$C = A * B = (c_1, c_2, \cdots, c_n)$を求めることで、$O(n\log n)$で行えます。すなわち、これでxor畳み込み後に添え字が0$以外の要素の総和を求めることでこの問題は解けそうです。

### Hadamard変換
天下り的ですが、
$$
\begin{aligned}
H_0 &= 1 \\\\
H_k &= \begin{pmatrix}
E_{k-1} & E_{k-1} \\\\
E_{k-1} & -E_{k-1} \end{pmatrix}
\begin{pmatrix}
H_k & 0 \\\\\
0 & H_k
\end{pmatrix}
\end{aligned}
$$
と行列$H_i$を定義します($E_k$は$k$次の単位行列)。このとき、$H_k$は$2^k$次の行列です。
$$
H_k H_k = 2^k E_{2^k}
$$
が成り立ちますので、$H_k^{-1} = 2^{-k} H_k$となります。また、
長さ$2^k$のベクトル$A$と$B$に対して次が成り立ちます。
$$
(H_k A) (H_k B) = H_k (A * B)
$$
左辺はベクトルの各要素ごとの積を表しています。これを変形すると、
$$
A * B = 2^{-k} H_k ((H_k A) (H_k B))
$$
が成り立ちます。しかし、これは長さ$2^k$のベクトルと$2^k$次の正方行列の積の計算をする必要がありますので、$n=2^k$として、結局$O(n^2)$かかってしまいます。
しかし、この行列$H_k$とベクトルの積はうまいことに高速化ができます。

### 高速Walsh-Hadamard変換

$H_k$を変形します。
$$
\begin{aligned}
H_k &=
\begin{pmatrix}
E_{k-1} & E_{k-1} \\\\
E_{k-1} & -E_{k-1}
\end{pmatrix}
\begin{pmatrix}
H_k & 0 \\\\\
0 & H_k
\end{pmatrix} \\\\
&=
\begin{pmatrix}
E_{k-1} & E_{k-1} \\\\
E_{k-1} & -E_{k-1}
\end{pmatrix}
\begin{pmatrix}
E_{k-2} & E_{k-2} & 0 & 0 \\\\
E_{k-2} & -E_{k-2} & 0 & 0 \\\\
0 & 0 & E_{k-2} & E_{k-2} \\\\
0 & 0 & E_{k-2} & -E_{k-2}
\end{pmatrix} 
\begin{pmatrix}
H_{k-2} & 0 & 0 & 0 \\\\
0 & H_{k-2} & 0 & 0 \\\\
0 & 0 & H_{k-2} & 0 \\\\
0 & 0 & 0 & H_{k-2}
\end{pmatrix} \\\
&= \cdots
\end{aligned}
$$
この行列の積への分解は高々$O(\log n)$回で終わります(上記の変形をし続けると最後に$H_0$が対角上に並んだ行列、すなわち単位行列が現れます)。
そして、各分解された行列をよく見ると、各行で$0$ではない要素は必ず$2$つです。すなわち、この分解された行列単体とベクトルの積は、$O(n)$で行うことができ、行列は$O(\log n)$個しかありませんので、結局$O(n\log n)$で行うことができます。

ピンと来ないという私のために、$n=8$のときの$H_3$を分解してみます。
$$
\begin{aligned}
H_3 &=
\begin{pmatrix}
E_2 & E_2 \\\\
E_2 & -E_2
\end{pmatrix}
\begin{pmatrix}
E_1 & E_1 & 0 & 0 \\\\
E_1 & -E_1 & 0 & 0 \\\\
0 & 0 & E_1 & E_1 \\\\
0 & 0 & E_1 & -E_1
\end{pmatrix} \\\\&
\begin{pmatrix}
E_0 & E_0 & 0 & 0 & 0 & 0 & 0 & 0 \\\\
E_0 & -E_0 & 0 & 0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & E_0 & E_0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & E_0 & -E_0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & E_0 & E_0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & E_0 & -E_0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 0 & E_0 & E_0 \\\\
0 & 0 & 0 & 0 & 0 & 0 & E_0 & -E_0
\end{pmatrix}
\begin{pmatrix}
H_0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\\\
0 & H_0 & 0 & 0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & H_0 & 0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & H_0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & H_0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 0 & H_0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 0 & H_0 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & H_0
\end{pmatrix} \\\\
&=
\begin{pmatrix}
1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\\\
0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 \\\\
0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 \\\\
0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 \\\\
1 & 0 & 0 & 0 & -1 & 0 & 0 & 0 \\\\
0 & 1 & 0 & 0 & 0 & -1 & 0 & 0 \\\\
0 & 0 & 1 & 0 & 0 & 0 & -1 & 0 \\\\
0 & 0 & 0 & 1 & 0 & 0 & 0 & -1
\end{pmatrix}
\begin{pmatrix}
1 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\\\
0 & 1 & 0 & 1 & 0 & 0 & 0 & 0 \\\\
1 & 0 & -1 & 0 & 0 & 0 & 0 & 0 \\\\
0 & 1 & 0 & -1 & 0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 1 & 0 & 1 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 1 & 0 & 1 \\\\
0 & 0 & 0 & 0 & 1 & 0 & -1 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 1 & 0 & -1
\end{pmatrix} \\\\&
\begin{pmatrix}
1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\\\
1 & -1 & 0 & 0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & 1 & 1 & 0 & 0 & 0 & 0 \\\\
0 & 0 & 1 & -1 & 0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 1 & 1 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 1 & -1 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 \\\\
0 & 0 & 0 & 0 & 0 & 0 & 1 & -1
\end{pmatrix}
\end{aligned}
$$
これを実装するのにも結構頭を使います。

```cpp
template<class T> vector<T> Hadamard(const vector<T>&A,const int k){
    // 2^k次行列
    if((int)A.size()!=(1<<k)){
        cout<<"A size should be 2^k"<<endl;
        return vector<T>();
    }
    vector<T> res = A;
    int h = 1;
    for(int i=0;i<k;i++){
        vector<T> tmp(1<<k,0);
        for(int j=0;j<(1<<k);j+=h*2){
            for(int l=j;l<j+h;l++){
                tmp[l] = res[l] + res[l+h];
                tmp[l+h] = res[l] - res[l+h];
            }            
        }
        h <<= 1;
        res = tmp;
    }
    return res;
}

template<class T> vector<T> xor_convolution(const vector<T>&A,const vector<T>&B,const int k){
    // xor convolution
    // A,Bのサイズは2^k
    vector<T> a = Hadamard(A,k);
    vector<T> b = Hadamard(B,k);
    vector<T> res(1<<k);
    for(int i=0;i<(1<<k);i++){
        res[i] = a[i] * b[i];
    }
    res = Hadamard(res,k);
    return res; //これを2^kで割る必要がある
}
```

このコードは最後に得た値の各要素を$2^k$で割り算してやる必要があるので注意です。

### 問題に戻る
問題で求めたいのは、$C=(c_1,c_2,\cdots,c_{2^{16}})$($c_i$は$i$個の石がある山の個数)があって、
$$
H(H(C)) +  H((H(C))^2) + H((H(C))^3) + \cdots + H((H(C))^N)
$$
です($C^i$は各要素の$i$乗を取ることを意味します)。Hadamard変換は線形変換なので、次のように変形できます。$D=H(C)$とすると、
$$
H( D + D^2 + D^3 + \cdots + D^N)
$$
かなりスッキリしました。あとは、等比数列の和の公式を使って、括弧の中身を計算することができます。

等比数列の和の公式は
$$
\frac{d_i(d_i^N-1)}{d_i-1}
$$
です。$d_i=0,1$の場合はこのまま計算すると値がおかしくなるのでそこだけ別処理することに注意します。

[提出コード](https://atcoder.jp/contests/abc212/submissions/45251406)


### 感想
これのAND版やOR版もあるらしい。考えた人天才。


## ABC213-G Connectivity 2
### 問題概要
$N$頂点$M$辺の単純無向グラフ$G$で、$0$本以上の辺を取り除いて新しいグラフ$H$を作る。各$k=2,3,\cdots,N$に対して、頂点$1$と$k$が$H$で連結になるような辺の取り除き方の個数を求める問題。

### 言い換え
問題で求められている「頂点$1$と$k$が連結(ry」の個数を$C(k)$とします。また、次のように$f(S),g(S)$を定義します。
- $f(S) := $ $S$を頂点集合とする$G$の連結部分グラフの個数
- $g(S) := $ $S$を頂点集合とする$H$の部分グラフの個数

すると、次のように$C(k)$を表現できます。ちなみに$V$は$G$の頂点集合です。
$$
C(k) = \sum_{\lbrace 1,k \rbrace \subset S \subset V} f(S) g(V\setminus S)
$$
全ての$f(S),g(S)$が既知ならば、これは$O(N2^N)$で計算できます。

### g(S)の計算
これは簡単です。
辺の両端が頂点集合$S$に含まれるような辺の個数を$E(S)$とすれば、$2^{E(S)}$が$g(S)$になります。
各頂点集合について$M$個の辺がそれぞれ含まれているかどうかを判定するので$O(M2^N)$で計算できます。

### f(S)の計算
問題はこちらです。$f(S)=g(S)-$(頂点集合が$S$であるような非連結なグラフの個数)です。$g(S)$は既知なので、後者を計算する必要があります。
連結でないグラフというのは、連結成分が$2$個以上あるグラフのことです。「この$2$個以上の連結成分」というのを、「$1$つの連結成分があって、残りの使ってない頂点たちを好きなようにしてもらう」という数え方をします。

$S$に含まれる頂点を$1$つ取ってきて、これを$v$とすると$f(S)$は次のようになります(この$v$がないとカウントが重複してしまいます)。
$$
f(S) = g(S) - \sum_{ v \in T \subsetneq S} f(T) g(S\setminus T)
$$

これを$S$についていい感じの順番で計算していくことで、$O(3^N)$で計算できます。

### 部分集合の部分集合の列挙
部分集合の$i$の部分集合$j$の列挙は$O(3^N)$でできます。
```cpp
for (int i = 0; i < (1 << N); i++) {
    for (int j = i; j >= 0; j--) {
        j &= i;
        // (i, j) は条件を満たす
    }
}
```
[提出コード](https://atcoder.jp/contests/abc213/submissions/45311948)

### 感想
$f(S),g(S)$に分けて$g(S)$の計算方法考えるのむずくね？？

## ABC213-H Stroll
### 問題概要
[問題](https://atcoder.jp/contests/abc213/tasks/abc213_h)

DPで解けそうな問題設定だが、時間が間に合わない。

### DP解
$d_{s,t}$を$t$キロメートル歩いて地点$s$にいる通り数とすると次のように計算できる。
$$
d_{s,t} = \sum \_{(s^\prime,i,x)} d_{s^\prime,t-x} \times p\_{i,t-x}
$$

ただし、シグマ記号は「$s^\prime$から$s$に向かう$i$番目の長さ$x$の道」を全てのペアについて足し合わせることを意味する。
これを$t$の小さい方から計算すれば$O(MT^2)$で計算できるがこれだと時間がかかりすぎる。

シンプルに$N=2$のときを考えると、
$$
d_{1,t} = \sum\_u d_{2,u} \times p\_{t-u}
$$
という式になって、畳み込みっぽい見た目をしている。実際には$d_{2,u}$が**定数ならば**畳み込みで計算できる。

### 分割統治FFT
**定数ならば畳み込みができる**ということで、半分を事前に計算して定数にし、いい感じに再帰的に計算するとなんと$O(T\log^2T)$で計算できてしまうのだ。言葉での説明が難しいので図を用意した。
図は単純に$1$次元のDPを分割統治FFTで解いたものである。

{{<figure src="./img/ABC213.jpg" width=50% title="分割統治FFT" >}}

ただ、計算に使用する数値は全て**確定**していなければならないので、以下のような順番で計算する必要がある。

{{<figure src="./img/ABC213-2.jpg" width=50% title="分割統治FFT(計算順序)" >}}

今回は、頂点ごとにDPの値を持つが、全ての辺について上記の図のような遷移があるので、各遷移を全ての辺ごとに行う。計算量は$O(MT\log^2T)$となる。

```cpp
void DCFFT(int l,int r,vector<edge>&E,vector<vector<mint>>&p, vector<vector<mint>>&dp){
    int m = (l+r)/2;
    if(l+1==r)return ;
    DCFFT(l,m,E,p,dp);
    for(int i=0;i<(int)E.size();i++){
        auto [u,v] = E[i];
        // [m,r)を更新
        /*
        本来だったら
        for(int i=m;i<r;i++){
            dp[v][i] += sum_x (dp[u][i-x]*p[v][x]);
        }
        をやるが、FFTを使う
        */
        auto dp2u = vector<mint>( dp[u].begin()+l, dp[u].begin()+m );
        auto p2 = vector<mint>( p[i].begin(), p[i].begin()+r-l );
        auto dp3u = convolution(dp2u,p2);
        for(int j=m;j<r;j++){
            dp[v][j] += dp3u[j-l];
        }
    }
    DCFFT(m,r,E,p,dp);
}
```

[提出コード](https://atcoder.jp/contests/abc213/submissions/45324097)

### 感想
添字をミスりまくってめっちゃ時間かかった。
地味に赤diff初AC。



