---
title: 行列累乗まとめ
description: 行列累乗の原理と例をひたすら列挙
slug: matrix-pow
date: 2022-12-06 02:00:00+0000
#image: cover.jpg
categories:
    - 競プロ
    - 数学
tags:
---

## これはなに
　行列累乗と呼ばれる競プロのテクニックの概要と，それを用いる例をひたすら挙げていきます．競プロチックな話題ではありますが，行列が役立つ場面の例の紹介として，非競プロerでも楽しめると思います．ただし，数列と行列に関する用語がある程度わかっている人向けです．

なお，この記事は[長野高専 Advent Calender 2022](https://qiita.com/advent-calendar/2022/nnct)の2日目の記事です。

## 原理(フィボナッチ数列の例)
次の式で表される数列を考えます．

$$
\begin{align}
F_1 &= 1 \\\\
F_2 &= 1 \\\\
F_n &= F_{n-1} + F_{n-2} \quad　(n \geq 3)
\end{align}
$$

これはフィボナッチ数列という名で知られている，$F=(1,1,2,3,5,8,13,21,34,\cdots)$という前の$2$項の和が次の項になる数列です．これの第$n$項目をプログラミングで求めることにします．第$1$項目から第$n$項目までを求めるのではなく，第$n$項目のみ求めれば良いことに注意です．それっぽいコードを次に示します．
```cpp
//F[i]:=フィボナッチ数列のi項目
F[1] = 1;
F[2] = 1;
for(int i=3;i<=n;i++){
	F[i] = F[i-1] + F[i-2];
}
```

`for`文を$n$回くらい回すので計算量は$O(n)$です(計算量についてこちら[^c]を参照してください)．

[^c]:APG4b 計算量 https://atcoder.jp/contests/APG4b/tasks/APG4b_w


では，$n=10^{15}$項目を求めたい場合はどうでしょうか．このコードでは，実行が終わりません．
#### 行列で表現する

　ここで，行列です．フィボナッチ数列の定義より，次の式は成り立ちます．

$$
\begin{pmatrix}
F_{n} \\\\
F_{n-1}
\end{pmatrix}
\=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
F_{n-1} \\\\
F_{n-2}
\end{pmatrix}
$$

具体的に計算してみます．

$$
\begin{pmatrix}
2 \\\\
1
\end{pmatrix}
\=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
1 \\\\
1
\end{pmatrix}
$$

$$
\begin{pmatrix}
3 \\\\
2
\end{pmatrix}
\=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
2 \\\\
1
\end{pmatrix}
$$

$$
\begin{pmatrix}
5 \\\\
3
\end{pmatrix}
\=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
3 \\\\
2
\end{pmatrix}
$$

前の$2$項から次の項が生成されていることがわかりますね．
これらの結果を使って次のような変形を考えます．

$$
\begin{align}
\begin{pmatrix}
5 \\\\
3
\end{pmatrix}
&=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
3 \\\\
2
\end{pmatrix}
\\\\
&=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
2 \\\\
1
\end{pmatrix}
\\\\
&=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
\begin{pmatrix}
1 \\\\
1
\end{pmatrix}
\\\\
&=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
^3
\begin{pmatrix}
1 \\\\
1
\end{pmatrix}
\end{align}
$$

始めの$2$項に同じ行列を$3$回掛け算することで3つ後の項が出てきました．これを繰り返すと次が成り立つことがわかります．

$$
\begin{align}
\begin{pmatrix}
F_n \\\\
F_{n-1}
\end{pmatrix}
&=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
^{n-2}
\begin{pmatrix}
F_2 \\\\
F_1
\end{pmatrix}
\end{align}
$$

第$10$項目を求めたいときは次のように計算できます．

$$
\begin{align}
\begin{pmatrix}
F_{10} \\\\
F_{9}
\end{pmatrix}
&=
\begin{pmatrix}
1 & 1 \\\\
1 & 0
\end{pmatrix}
^{8}
\begin{pmatrix}
F_2 \\\\
F_1
\end{pmatrix}
\\\\
&=
\begin{pmatrix}
89 & 55 \\\\
55 & 34
\end{pmatrix}
\begin{pmatrix}
1 \\\\
1
\end{pmatrix}
\\\\
&=
\begin{pmatrix}
144 \\\\
89
\end{pmatrix}
\end{align}
$$

フィボナッチ数列は$1,1,2,3,5,8,13,21,34,55,89,144,\cdots$であるため，確かにあっています．
便利そうな式が行列によって完成しました．漸化式の悪いところは前の項の値がわかっていないと次の項が計算できないことですが，この式は特定の項をダイレクトに表すことに成功しています．行列の$n$乗が高速に計算できればフィボナッチ数列の第$n$項が高速に計算できそうです．

#### 累乗の高速計算
行列$A$の$n$乗である$A^n$を計算するのに，ナイーブな方法だと$n-1$回の行列同士の掛け算が発生します．この行列同士の掛け算の回数が少なくなることを高速化と呼ぶことにします．
$A^{128}$を考えてみましょう．
- $A$に$A$をかけて$A^2$
- $A^2$に$A$をかけて$A^3$
$\cdots$
- $A^{127}$に$A$をかけて$A^{128}$

全部で$127$回の掛け算が必要ですね．では，$128$という特徴的な数に注目して次のように計算したらどうでしょうか．
- $A$に$A$をかけて$A^2$
- $A^2$に$A^2$をかけて$A^4$
- $A^4$に$A^4$をかけて$A^8$
- $A^{8}$に$A^{8}$をかけて$A^{16}$
- $A^{16}$に$A^{16}$をかけて$A^{32}$
- $A^{32}$に$A^{32}$をかけて$A^{64}$
- $A^{64}$に$A^{64}$をかけて$A^{128}$

全部で，$7$回の掛け算で$A^{128}$が計算できました．$127$回から$7$回という驚異の回数削減です．
実は，これは$128$のような$2$の冪数に限った話ではありません．先程，$A^1,A^2,A^4,A^8,A^{16},\cdots$を求めましたが，これらを掛け合わせることで任意の$A^n$が計算できます(指数法則$A^aA^b=A^{a+b}$が成り立つことに注意)．
- $A^{12} = A^{8} A^{4}$
- $A^{39} = A^{32} A^{4} A^{2} A^{1}$
- $A^{127} = A^{64}A^{32}A^{16}A^{8}A^{4}A^{2}A^{1}$

これらは，$Aの(2進数表記したとき，1である位)乗$を掛け算したものになっています．
上の例では，$12_{(10)}=1100_{(2)}$であり，$1$である位は$4$の位と$8$の位なので$12=8+4$と$12$を$2$の冪数の和で表すことができます．
このような手法をダブリングや繰り返し二乗法と呼びます．

それっぽいコードを次に示します．計算量は$O(\log N)$です．$O(N)$に比べて非常に高速です[^9]．

[^9]:実はこの計算量解析はあまり意味のあるものではありません．なぜなら，フィボナッチ数列は後の項に行けば行くほど桁数が増えるからです．後の項に行くほど，かけ算やたし算の計算コストは上がっていきます．ですので，$O(\log N)$といっても$N=10^7$の時点で結構時間がかかってしまいます．競プロでは$10^9+7$では割ったあまりを求めなさい，のように桁数を気にせずに計算できる場合が多いため，今回は考えないことにしています(競プロに限った話ではなく，大きな数を求めるときに異なる素数で割ったあまりを求めておいて，中国剰余原理で復元する，等の手法が使われる)．

```cpp
// AのB乗の計算結果をCの格納
C = I //Iは単位行列
tmp = A
while(B>0){
	if(B%2==1){
		C = C * tmp
	}
	B/=2;
	tmp = tmp * tmp
}
```

Pythonでの汚い実装例も示します．

##### 愚直パターン
```python
n = 10000
f = [0]*(n+1)
f[1] = 1
f[2] = 1
for i in range(3, n+1):
    f[i] = f[i-1] + f[i-2]

print(f"第{n}項目は{f[n]}です")
```
##### 高速化
```python

def mat_mul(a, b) :
    I, J, K = len(a), len(b[0]), len(b)
    c = [[0] * J for _ in range(I)]
    for i in range(I) :
        for j in range(J) :
            for k in range(K) :
                c[i][j] += a[i][k] * b[k][j]
    return c


def mat_pow(x, n):
    y = [[0] * len(x) for _ in range(len(x))]

    for i in range(len(x)):
        y[i][i] = 1

    while n > 0:
        if n & 1:
            y = mat_mul(x, y)
        x = mat_mul(x, x)
        n //= 2

    return y


ret = [[1],[1]]
mat = [[1,1],[1,0]]
n = 10000
ret = mat_mul(mat_pow(mat, n-2), ret)
print(f"第{n}項目は{ret[0][0]}です")
```

## 線形漸化式
フィボナッチ数列は$2$項間の漸化式でしたが，$k$項間の線形漸化式でも可能です．
#### 3項の場合
> 次で定義される数列$a$の$n$項目を求めよ．

$$
\begin{align}
a_1 &= a_2 = a_3 = 1 \\\\
a_n &= 2a_{n-1} - 4a_{n-2} + 3a_{n-3}\quad　(n \geq 4)
\end{align}
$$

この漸化式は次のような行列で表されます．

$$
\begin{align}
\begin{pmatrix}
a_n \\\\
a_{n-1} \\\\
a_{n-2}
\end{pmatrix}
&=
\begin{pmatrix}
2 & -4 & 3\\\\
1 & 0 & 0 \\\\
0 & 1 & 0 
\end{pmatrix}
^{n-3}
\begin{pmatrix}
a_3 \\\\
a_2 \\\\
a_1 
\end{pmatrix}
\end{align}
$$

実際に行列を計算すると確かに成立していることがわかります．

一般に，$k+1$項間の漸化式は$k\times k$の行列を使って表され，$k\times k$の行列同士の掛け算は$k^3$回の掛け算が必要であるため，第$n$項を求めるのに$O(k^3\log n)$かかります．

なお，$k+1$項間の線形漸化式の第$n$項はkitamasa法と呼ばれる手法で$(k^2\log n)$で求めることが可能です．こちらもダブリングを使った手法です．

#### 定数項がある場合

> $a,b,m,x_0$を使って線形合同法で乱数を生成する．このときの$n$番目の乱数$x_n$を求めよ
> ただし，線形合同法とは，

$$
x_{i+1} = ax_i + b \pmod m
$$

> の漸化式で疑似乱数を生成する手法である．

定数項がある場合は$1$の行を付け足して次のように作ることができます．

$$
\begin{align}
\begin{pmatrix}
x_n \\\\
1 \\\\
\end{pmatrix}
&=
\begin{pmatrix}
a & b\\\\
0 & 1 
\end{pmatrix}
^{n}
\begin{pmatrix}
x_0 \\\\
1
\end{pmatrix}
\end{align}
$$

## 複数変数の線形漸化式
複数の変数で表される漸化式にも応用できます
#### 平方根を含む数の累乗
次の問題を考えます．
> $(2+\sqrt{3})^n$はいくつか？$a+b\sqrt{3}$の形式になるので$a,b$を求めよ．

$n$が大きいと展開が厄介になりそうなことが想像できますね．
とりあえず，$2+\sqrt{3}$の累乗を次のように文字で置きます．

$$
X_n = (a_n + b_n\sqrt{3})
$$

$X_n$から$X_{n+1}$を計算してみます．

$$
\begin{align}
X_{n+1} &= X_n (2 + \sqrt{3}) \\\\
	&= (a_n + b_n \sqrt{3})(2 + \sqrt{3}) \\\\
	&= 2a_n + a_n \sqrt{3} + 2b_n \sqrt{3} + 3b_n \\\\
	&= (2a_n + 3b_n) + (a_n + 2b_n)\sqrt{3} \\\\
	&= a_{n+1} + b_{n+1} \sqrt{3}
\end{align}
$$

最後の二行の係数を比較して，次の漸化式が立ちます．

$$

\begin{align}
a_1&=b_1=1 \\\\
a_{n+1} &= 2a_{n} + 3b_n \\\\
b_{n+1} &= a_n + 2b_n
\end{align}

$$

これを行列にすると次のようになります．

$$
\begin{align}
\begin{pmatrix}
a_n \\\\
b_n 
\end{pmatrix}
&=
\begin{pmatrix}
2 & 3\\\\
1 & 2
\end{pmatrix}
^{n-1}
\begin{pmatrix}
a_1 \\\\
b_1
\end{pmatrix}
\end{align}
$$


#### 確率
次の問題を考えます．
> エアコンのスイッチの状態は$OFF$である．遠隔操作を1回行うと確率$p$でエアコンの$ON$と$OFF$が入れ替わる．$n$回遠隔操作をしたときエアコンが$ON$である確率を求めよ．($N\leq 10^{18}$)　出典 [CODE FESTIVAL 2014 Middle C - eject](https://atcoder.jp/contests/code-festival-2014-morning-middle/tasks/code_festival_morning_med_c)

次のように変数を設定します．
$a_i = i$回目の遠隔操作をしたときにエアコンが$ON$である確率
$b_i = i$回目の遠隔操作をしたときにエアコンが$OFF$である確率
漸化式は次のように立ちます．

$$
a_i = p b_{i-1} + (1-p)a_{i-1} \\\\
b_i = p a_{i-1} + (1-p)b_{i-1} \\\\
$$

行列にします．ただし，$a_0=0,b_0=1$です．

$$
\begin{align}
\begin{pmatrix}
a_n \\\\
b_n 
\end{pmatrix}
&=
\begin{pmatrix}
1-p & p\\\\
p & 1-p
\end{pmatrix}
^{n}
\begin{pmatrix}
a_0 \\\\
b_0
\end{pmatrix}
\end{align}
$$

今までのものは全て，小さな問題の答えから大きな問題の答えを導くdp(dynamic programming:動的計画法)に分類されます．dpの漸化式が立ったときに，それが線形であり，同じ遷移をする場合に行列累乗が有効なことが多いです．

## グラフのパス数
[EDPC R - Walk](https://atcoder.jp/contests/dp/tasks/dp_r)
> $N$頂点の単調有効グラフ$G$で，長さ$1$の有向辺が長さ$K$のパスは何通りありますか．ただし同じ頂点を複数回通っても良い

$N$次の正方行列で，頂点$i$から頂点$j$までの有向辺があれば$A_{ij}=1$，なければ$A_{ij}=0$となるような隣接行列を考えます．

例えば，次の$4$頂点のグラフを隣接行列にすると次のようになります．

![](https://storage.googleapis.com/zenn-user-upload/1c6bfa766abe-20221107.png)

$$
A = 
\begin{align}
\begin{pmatrix}
0 &1 &0 &1 \\\\
0 &0 &1 &0 \\\\
0 &1 &0 &0 \\\\
0 &0 &0 &1 
\end{pmatrix}
\end{align}
$$

このとき，頂点$i$から頂点$j$へのパス数は次のように計算できます．

$$
(頂点iから頂点jへのパス数) = \sum _{k=1}^4 A_{ik} A_{kj}
$$

これは，行列の積の定義とよく似ていて，$A^2$の$(i,j)$成分は，頂点$i$から頂点$j$への長さ$2$のパス数を表します．よって，$A^n$の$i,j$成分から頂点$i$から$j$までの長さ$n$のパス数を求めることができます．

なお，無向グラフの場合は双方向に有向辺が張っていると考え，$A_{ij}=A_{ji}$とすることで同様に処理できます．

## 行列の級数
> 正方行列$A$がある．$A+A^2+A^3+A^4+A^5+\cdots+A^N$を求めよ．


次のように$X_n$を定義します．

$$
X_n = A^1 + A^2 + A^3 + \cdots + A^n
$$

漸化式は次のように作れます．

$$
\begin{align}
X_1 &= A\\\\
X_n &= A+AX_{n-1} \quad (n\geq 2)
\end{align}
$$

よって，

$$
\begin{align}
\begin{pmatrix}
X_n \\\\ \hline
I 
\end{pmatrix}
&=
\begin{pmatrix}
\begin{array}{c|c} 
A & A\\\\ \hline
O & I
\end{array}
\end{pmatrix}
^{n-1}
\begin{pmatrix}
A \\\\ \hline
I
\end{pmatrix}
\end{align}
$$

です．

## 半環
普通，行列の掛け算は次のように定義されています．

$$
A = 
\begin{align}
\begin{pmatrix}
a_{11} & a_{12} \\\\
a_{21} & a_{22}
\end{pmatrix}
\begin{pmatrix}
b_{11} & b_{12} \\\\
b_{21} & b_{22}
\end{pmatrix}
\=
\begin{pmatrix}
a_{11}\cdot   b_{11}+a_{12}\cdot   b_{21} & a_{11}\cdot   b_{12}+a_{12}\cdot   b_{22} \\\\
a_{21}\cdot   b_{11}+a_{22}\cdot   b_{21} & a_{21}\cdot   b_{12}+a_{22}\cdot   b_{22}
\end{pmatrix}
\end{align}
$$

それぞれ要素の積を計算し，和を計算しています．ここで，「和」と「積」を別のものに置き換えることを考えます．例えば次のように$\max$と$+$に変えるとどうなるでしょうか．ここで，$\max(a_1,a_2,a_3,\cdots)$は，$a_1,a_2,a_3,\cdots$のうちの最大値を表します．

$$
A = 
\begin{align}
\begin{pmatrix}
a_{11} & a_{12} \\\\
a_{21} & a_{22}
\end{pmatrix}
\begin{pmatrix}
b_{11} & b_{12} \\\\
b_{21} & b_{22}
\end{pmatrix}
\=
\begin{pmatrix}
\max(a_{11}+b_{11},a_{12}+b_{21}) & \max(a_{11}+b_{12},a_{12}+b_{22}) \\\\
\max(a_{21}+b_{11},a_{22}+b_{21}) & \max(a_{21}+b_{12},a_{22}+b_{22})
\end{pmatrix}
\end{align}
$$

この$\max,+$で計算される行列ですが，実はこの行列においても高速な行列累乗の計算が可能です．こんな変な演算の世界で行列累乗をしてどうするんだと思いますが，例は後述します．もちろん，演算子を好き勝手変えていいわけではありません．いくつか条件があります．次に示す条件を満たす，集合と加法$+$と乗法 $\cdot$ の$2$つの二項演算，である必要があります．この性質を満たす集合と$+$と$\cdot$の組を半環と言います．また，集合の要素を元と言います．

- 加法において，結合法則が成り立ち，可換であり，単位元$0$を持つ．
	- $(a + b) + c = a + (b + c)$
	- $a+b = b+a$
	- $0 + a = a + 0 = 0$
- 乗法において，結合法則が成り立ち，単位元$1$を持つ．
	- $(a \cdot   b) \cdot   c = a \cdot   (b \cdot   c)$
	- $1 \cdot   a = a \cdot   1 = a$
- 分配法則が成り立つ．
	- $a \cdot   (b+c) = (a\cdot   b) + (a\cdot   c)$
	- $(a+b)\cdot   c = (a\cdot   c) + (b\cdot   c)$
- 乗法で，$0$(加法の単位元)と任意の元の積は$0$になる．
	- $0\cdot   a = a\cdot   0 = 0$

例えば，有理数の集合を$\mathbb{R}$とすると，$(\mathbb{R},+,\cdot)$は半環をなす，という言い方をします．

では，先程の$\max$と$+$の例は本当に半環をなしているかを確認してみましょう．この場合の集合も$\mathbb{R}$(有理数全体)としておきます．先述した半環の定義だと，加法が$\max$で，乗法が$+$に対応しています．
まずは加法$\max$について見ていきます．

- $\max(\max(a,b),c) = \max(a,\max(b,c))$
- $\max(a,b) = \max(b,a)$
- $\max(\infty,a) = \max(a,\infty) = \infty$

$\max$は単位元を$\infty$にすることで成立します．次に乗法$+$です．

- $(a + b) + c = a + ( b+c)$
- $0 + a = a + 0 = a$

$+$の単位元は$0$です．
次に分配法則の確認です．ややこしいですが，次の式が成り立つことがわかります．

- $a+\max(b,c) = \max(a+b,a+c)$
- $\max(a,b)+c = \max(a+c,b+c)$

最後に加法$\max$の単位元$\infty$を乗算$+$すると$\infty$になることを確認します．

- $\infty +a = a + \infty = \infty$

以上，厳密性には欠けますが$(\mathbb{R},\max,+)$が半環をなしていることがわかりました．さて，次の節からは，いつもと違う半環の世界での行列累乗の例を見ていきます．

### ANDとXOR
[ABC009 D - 漸化式](https://atcoder.jp/contests/abc009/tasks/abc009_4)
> 定数$C_1,C_2,C_3,\cdots,C_K$があり，$K$項間の漸化式が次のように定まる
> $A_i \quad (1\leq i\leq K)$は与えられる．

$$
A_{K+1} = (C_1 \mathrm{AND} A_{K}) \mathrm{XOR} (C_2 \mathrm{AND} A_{K-1}) \mathrm{XOR}  \cdots \mathrm{XOR} (C_K \mathrm{AND} A_{1})
$$

> このとき，$A_N$を求めよ($1\leq M\leq 10^{9}$)．
> なお，$\mathrm{AND}$はビットごとの論理積，$\mathrm{XOR}$はビットごとの排他的論理和を表す．

この問題は$(\mathbb{N},\mathrm{AND},\mathrm{XOR})$が半環をなすことを利用して，ただの線形漸化式と見ることができます．なお，$\mathrm{AND}$と$\mathrm{XOR}$はビットごとで独立した演算であるため，各bitで$\mod 2$の掛け算とたし算をしていると見ることでも解くことができます．

### ワーシャルフロイド法


ワーシャルフロイド法は，$(\mathbb{R},\min,+)$の世界での行列累乗だと捉えることができます．

MojaCoderで類題を見つけました．
[Dungeon Attack (Hard)](https://mojacoder.app/users/magurofly/problems/dungeon-attack)


## 関連問題
行列累乗がストレートに出ることは珍しく，考察の末に最後の一捻りとして出てくることが多い印象があります．全体的に難しいです．
- [フィボナッチ数列の第N項をMで割った余りを求める](https://yukicoder.me/problems/no/526)
- [ABC204 F - Hanjo 2](https://atcoder.jp/contests/abc204/tasks/abc204_f)
- [ABC199 F - Graph Smoothing](https://atcoder.jp/contests/abc199/tasks/abc199_f)
- [ABC256 G - Blakc and Whte Stones](https://atcoder.jp/contests/abc256/tasks/abc256_g)
- [ABC129 F - Takahashi's Basics in Education and Learning](https://atcoder.jp/contests/abc129/tasks/abc129_f)
- [ABC271 G - Access Counter ](https://atcoder.jp/contests/abc271/tasks/abc271_g)
- [ABC212 H - Nim Counting](https://atcoder.jp/contests/abc212/tasks/abc212_h)
- [ARC025 D - コンセント](https://atcoder.jp/contests/arc025/tasks/arc025_4)
- [DISCO2020 B - Hawker on Graph](https://atcoder.jp/contests/ddcc2020-final/tasks/ddcc2020_final_b)

