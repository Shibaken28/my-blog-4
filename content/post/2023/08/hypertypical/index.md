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

## ABC212-G Power Pair
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

## 感想
原始根って便利なんですね

## ABC212-H Nim Counting 
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




