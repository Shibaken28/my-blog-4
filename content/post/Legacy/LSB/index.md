---
title: LSB Decryption Oracle Attackの実装
description: 
slug: 何度も復元でき，下位ビットが得られるときに使える攻撃
#image: cover.jpg
categories:
    - CTF
    - crypto
    - 数学
tags:
    - RSA
---


## LSB Decryption Oracle Attack
RSA暗号において，何度も暗号文を復元してくれて，その最も下のビットがわかっている場合，LSB Decryption Oracle Attackが使える．
ここでは，入力$x$に対して，$x^d\mod n$が出力されるものとする．
(RSAの各パラメータの説明については省略する．)

### 理論
$$(2m)^e = 2^em^e = 2^ec\mod n$$
が成り立つため，$2^ec\mod n$を入力として与えると，$(2m)^e\mod n$を復元することになるため，$2m \mod n$が返ってくる．
この値の偶奇で$m$の範囲がわかる．
- もし，$0\leq2m<n$の場合，$2m\mod n$と$2m$は等しいため，$2m \mod n$は必ず偶数になる．
- もし，$n\leq2m<2n$の場合，$2m\mod n$は$2m-n$と等しいため，$n$が奇数であることから$2m\mod n$は必ず奇数になる．

$0<m<n$であるから，$2m$がこれ以外の値の範囲をとることはない．
これにより，$2m\mod n$が偶数なら$0\leq m<\frac{1}{2}n$，奇数なら$\frac{1}{2}n\leq m<n$であることがわかる．

次に，$4^ec\mod n$を入力に与えることについて考える．同様に，$4^ec = (4m)^c\mod n$が成り立つから，$4m\mod n$の値が返ってくる．
例えば，$2m\mod n$の出力が偶数で，$0\leq m<\frac{1}{2}n$であることが，わかっていたとする．このとき，$4m\mod n$の偶奇は次のように決まっている．
- $0\leq4m<n$の場合，$4m\mod n$は$4m$と等しいため，$4m\mod n$は必ず偶数になる．
- $n\leq4m<2n$の場合，$4m\mod n$は$4m-n$と等しいため，$4m\mod n$は必ず奇数になる．

よって，$2m\mod n$が偶数で，
かつ$4m\mod n$が偶数のときは，$0\leq m<\frac{1}{4}n$，奇数のときは$\frac{1}{4}n\leq m<\frac{1}{2}n$であることがわかる．

同様に，$2m\mod n$の出力が奇数で，$\frac{1}{2}n\leq m<n$であることが，わかっていたとする．このとき，$4m\mod n$の偶奇は次のように決まっている．
- $2n\leq4m<3n$の場合，$4m\mod n$は$4m-2n$と等しいため，$4m\mod n$は必ず偶数になる．
- $3n\leq4m<4n$の場合，$4m\mod n$は$4m-3n$と等しいため，$4m\mod n$は必ず奇数になる．

よって，$2m\mod n$が奇数で，
かつ$4m\mod n$が偶数のときは，$\frac{1}{2}n\leq m<\frac{3}{4}n$，奇数のときは$\frac{3}{4}n\leq m<n$であることがわかる．

これを繰り返していくと，二分探索の要領でどんどん$m$の取りうる値が半分になっていき，最終的に$m$の値が求まる．

### 実装例
整数で`l,r`を動かすと誤差が出てしまうため，有理数を使っている．
```python
from Crypto.Util.number import *
from fractions import Fraction
from math import ceil

p = getPrime(256)
q = getPrime(256)
n = p*q
m = bytes_to_long(b"flag{this_is_flag}")
e = 0x10001
c = pow(m,e,n)
d = pow(e,-1,(p-1)*(q-1))
assert m<n
assert pow(c,d,n)==m

def dec(x):
       return pow(x,d,n)

l = 0
r = n
a = 1
while abs(r-l)>1:
       b=(c*pow(2,a*e,n))%n
       de = dec(b)
       mid = Fraction(l + r, 2)
       if de%2==0:
              r = mid
       else:
              l = mid
       a+=1
print(ceil(l))
print(m)
```


### 参考サイト

- [plain RSAに対するLSB decryption oracle attackをやってみる](https://inaz2.hatenablog.com/entry/2016/11/09/220529)
- [LSB Leak Attackを実装した](https://kmyk.github.io/blog/blog/2017/06/24/lsb-leak-attack/)

を参考にさせて頂きました．