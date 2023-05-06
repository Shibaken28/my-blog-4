---
title: "4月にやったcryptoまとめ"
description: ångstromCTF,umdctf
slug: cpctf2023
date: 2023-04-26T21:00:00+09:00
image: 
categories:
    - CTF
    - crypto

draft: true
---
## はじめに
スクリプトが長くて問題の意図を読み取るのに一苦労することが多かったです．

問題名の後にある括弧内の数字は解いたチーム数を表しています．

## ångstromCTF 2023
### ranch (895/1429)
#### 問題
`secret_shift.txt`に書かれた数だけフラグの各アルファベットをシフトしています(スクリプト中の97は`a`の文字コードである)．
```python
import string

f = open("flag.txt").read()

encrypted = ""

shift = int(open("secret_shift.txt").read().strip())

for i in f:
    if i in string.ascii_lowercase:
        encrypted += chr(((ord(i) - 97 + shift) % 26)+97)
    else:
        encrypted += i

print(encrypted)
```
#### 解答
ただのシーザー暗号なので全探索すれば良いです．ちなみに，このタイプの問題は暗号化したスクリプトの大部分を流用すると，復号のプログラムが書きやすいです．
```python
import string

ct = "rtkw{cf0bj_czbv_nv'cc_y4mv_kf_kip_re0kyvi_uivjj1ex_5vw89s3r44901831}"

for shift in range(26):
    decrypted = ""
    for c in ct:
        if c in string.ascii_lowercase:
            decrypted += chr(((ord(c) - 97 - shift) % 26)+97)
        else:
            decrypted += c

    print(decrypted)
```

```none
actf{lo0ks_like_we'll_h4ve_to_try_an0ther_dress1ng_5ef89b3a44901831}
```


### impossible (503/1429)
`zero_encoding(x,64)`と`one_encoding(y,64)`の出力(配列)に一致する要素がなく，`x>y,x>0,y>0`を満たすような`x,y`を入力すれば良い．
#### 問題
```python
#!/usr/local/bin/python

def fake_psi(a, b):
    return [i for i in a if i in b]

def zero_encoding(x, n):
    ret = []

    for i in range(n):
        if (x & 1) == 0:
            ret.append(x | 1)

        x >>= 1

    return ret

def one_encoding(x, n):
    ret = []

    for i in range(n):
        if x & 1:
            ret.append(x)

        x >>= 1

    return ret

print("Supply positive x and y such that x < y and x > y.")
x = int(input("x: "))
y = int(input("y: "))

if len(fake_psi(one_encoding(x, 64), zero_encoding(y, 64))) == 0 and x > y and x > 0 and y > 0:
    print(open("flag.txt").read())
```

#### 解答
`n`がせいぜい`64`なため，$2^n,n\geq 64$の数を入れると`one_encoding`の出力は空`[]`になります．
よって，例えば$x=2^100=1267650600228229401496703205376,y=1$とすれば良いです．

#### コメント
スクリプトが何をしているかの理解に時間がかかりました．

### Lazy Lagrange (150/1429)
#### 問題
スクリプトが長いので省略します．

簡単に問題を要約すると，
- フラグの長さは$N$である．$N\leq 18$である．
- 長さ$N$の，$(1,2,3\cdots,N)$を並べ替えた順列$P$がある．
- フラグの$i$文字目の文字コードを$C_i$とし，$A=(C_{P_1},C_{P_2},C_{P_3},\cdots,C_{P_N})$とする．
- ユーザーが$x$を入力すると，値$r(x)=(A_1x^0+A_2x^1+A_3x^2\cdots A_Nx^{N-1}を2^{127}-1で割ったあまり)$が得られる．
- 最大$10$個までの$x$が入力に対する値$r(x)$が得られる．

要するに，$N-1$次関数のパラメータ(各係数)を求めればよいわけです．

#### 解答
基本的に，$M$次関数のパラメータを求めるには，$M$個の点を知る必要があります．ですが，今回は最大$10$点の値しか知ることができないため，何か得策がないかを考えます．

ここで，フラグのすべての文字がprintableでascii文字，すなわち文字コードが$2^7$未満であることに注目すると，$x=2^7$とすると都合が良いです．つまり，
$$
r(2^7) = A_1 + 2^7A_2 + (2^7)^2A_3 + \cdots + (2^7)^{N-1} A_N
$$
となります．よって，$x=2^7$として，得られた結果を$7$ビットごとに分割して，それぞれを文字に変換すればパラメータ$A$がわかります．
```
a = 68322490381546798839241596580868495416
s = bin(a)[2:].zfill(7*16)
f = []
for i in range(0, len(s), 7):
    f.append(int(s[i:i+7], 2))

f.reverse()
for x in f:
    print(x, end=' ')
```
これで数列$A$を復元でき，サーバーに$A$を投げると$P$が得られます．
```
p = "11 7 3 6 16 13 9 4 10 2 1 0 17 14 12 15 5 8".split()
p = [int(x) for x in p]
flag = [0]*18

for i in range(18):
    flag[int(p[i])] = chr(f[i])

print(''.join(flag))
```

#### コメント
　RSAの問題よりも点数が低く設定されていましたが，solvesはそちらよりも少なかった問題です．
僕もすぐにわからなかったため，RSAの問題を先に解きました．


### Royal Society of Arts (446/1429)
#### 問題
RSAの問題です．$n,e,c$の他に$(p-2)(q-1)$と$(p-1)*(q-2)$がもらえます．
```python
from Crypto.Util.number import getStrongPrime, bytes_to_long
f = open("flag.txt").read()
m = bytes_to_long(f.encode())
p = getStrongPrime(512)
q = getStrongPrime(512)
n = p*q
e = 65537
c = pow(m,e,n)
print("n =",n)
print("e =",e)
print("c =",c)
print("(p-2)*(q-1) =", (p-2)*(q-1))
print("(p-1)*(q-2) =", (p-1)*(q-2))
```
#### 解答
与えられた$(p-2)(q-1)=A$と$(p-1)(q-2)=B$は$p,q$に関する方程式であるため，$n=pq$であることを利用しながら$p,q$を求めることができます．
ここでは，少し技巧的に$\phi(n)$を直接求めます．

$\phi(n)$の値をまず確認しておきましょう．
$$
\begin{aligned}
\phi(n)&=(p-1)(q-1)\\\\
&=pq-p-q+1 \\\\
&=n-p-q+1 \\\\
&=n-(p+q)+1
\end{aligned}
$$
最後の行は，我々が$p+q$さえ求めればいいことを表していますね．

さて，$A,B$を展開して，

$$
\begin{aligned}
A&=pq-p-2q+2 \\\\
&= n -p -2q + 2 \\\\
B&=pq-2p-q+2 \\\\
&= n -2p -q + 2 \\\\
\end{aligned}
$$
$p$と$q$に対称性があるので足します．
$$
\begin{aligned}
A+B&=2n-3p-3q+4\\\\
&=2n-3(p+q)+4
\end{aligned}
$$
ここで$p+q$が出現しました．あとは，$p+q$について解いて代入すれば$\phi(n)$は求まります．
最終的には次のようになります．
$$
\phi(n) = n-\frac{1}{3}(2n+4-A-B) + 1
$$
```python
from Crypto.Util.number import bytes_to_long, getPrime, long_to_bytes, isPrime
from gmpy2 import iroot

n = 125152237161980107859596658891851084232065907177682165993300073587653109353529564397637482758441209445085460664497151026134819384539887509146955251284230158509195522123739130077725744091649212709410268449632822394998403777113982287135909401792915941770405800840172214125677106752311001755849804716850482011237
e = 65537
c = 40544832072726879770661606103417010618988078158535064967318135325645800905492733782556836821807067038917156891878646364780739241157067824416245546374568847937204678288252116089080688173934638564031950544806463980467254757125934359394683198190255474629179266277601987023393543376811412693043039558487983367289
A = 125152237161980107859596658891851084232065907177682165993300073587653109353529564397637482758441209445085460664497151026134819384539887509146955251284230125943565148141498300205893475242956903188936949934637477735897301870046234768439825644866543391610507164360506843171701976641285249754264159339017466738250
B = 125152237161980107859596658891851084232065907177682165993300073587653109353529564397637482758441209445085460664497151026134819384539887509146955251284230123577760657520479879758538312798938234126141096433998438004751495264208294710150161381066757910797946636886901614307738041629014360829994204066455759806614

C = (2*n+4-A-B)//3
phi = n - C + 1

d = pow(e, -1, phi)
m = pow(c, d, n)
print(long_to_bytes(m))
```

### Royal Society of Arts 2
#### 問題
RSAの問題です．$c,e$のみ与えられています．次のことが$1$度だけできます．
- $x$を入力する．$x^d \mod n$が出力される．

ただし，次の条件を満たす$x$は入力することができません．
- $x$が$m$と等しい
- $x^d \mod n$の値をbytes型にしたときに`actf{`で始まる．

```python
from Crypto.Util.number import getStrongPrime, bytes_to_long, long_to_bytes
f = open("flag.txt").read()
m = bytes_to_long(f.encode())
p = getStrongPrime(512)
q = getStrongPrime(512)
n = p*q
e = 65537
c = pow(m,e,n)
print("n =",n)
print("e =",e)
print("c =",c)

d = pow(e, -1, (p-1)*(q-1))

c = int(input("Text to decrypt: "))

if c == m or b"actf{" in long_to_bytes(pow(c, d, n)):
    print("No flag for you!")
    exit(1)

print("m =", pow(c, d, n))

# nc challs.actf.co 32400
```

#### 解答
復号したものが$m$と等しいとダメなので，$c+kn$を投げる作戦($k$は整数)は使えません．計算結果が$m$とは異なるけど，$m$に関係する値になってもらえば良さそうです．

私は$c^2$を投げました．
$$
\begin{aligned}
(c^2)^d &= (c^d)^2 \mod n \\\\
&= m^2 \mod n
\end{aligned}
$$
$m^2$が返ってくるので，平方根を取ればよいです[^1]．

```python
from Crypto.Util.number import bytes_to_long, getPrime, long_to_bytes, isPrime
from gmpy2 import iroot

n = 122795322639273335183407547988355717667682402969887021662853040822068784429099512092859819366313760861569995191111354903609641912851543073534721229141621278828813949497504819929457406006733164210839718812846052764823313825358868347677787273455595487060611890626341637500852692386233883685753371899288189050001
e = 65537
c = 13928861579959579258021132975729463470137322017208366862354271326027744432029596343875946043929040177248697424435441871947184336560233969116972059874696951272071826812809162130296606007924269097875864150634901807435906603676769414008842283735798617363866797245521482942082487171162820892424992296802798328435
c2 = pow(c, 2, n)
print(c2)

m2 = 3428404974453772204440872767975262611897324377204891195496459106442411519036871510586566155464308744925877387175539403921484579647158452704134120793457058489885113163402935234588733969216791423316030005787704096339163272966409
m = iroot(m2, 2)[0]
print(long_to_bytes(m))
```

#### コメント
今回は，運良く$m^2$が$n$を超えていかったので平方根を簡単に求められました．
$m^2$が$n$を超えていた場合，$\rm{mod}$上での平方根を求める問題になります．
この場合でも，Tonelli-Shanksのアルゴリズムなどを使うことで求めることができます．


## UMDCTF2023

### CBC-MAC 1(108/745)
#### 問題
CBCモードを使った問題です．スクリプトが長いので省略します．

この問題は，サーバーに接続すると，次のことが何回もできます．
- 任意のブロックを入力し，それをCBCモードで暗号化したときの末尾ブロックが得られる．

そして，次のことをするとフラグがもらえます．
- ブロック列$B$とブロック$b$を入力し，$B$をCBCモードで暗号化した結果が$b$になる．
    - ただし，$B$は今まで暗号化したブロック列とは異なるものでなければならない．

ただし，keyは固定でiv=$0$です．

#### 解答
まず，`00000000000000000000000000000000`を暗号化させます．
```python
Team Rocket told me CBC-MAC with arbitrary-length messages is safe from forgery. If you manage to forge a message you haven't queried using my oracle, I'll give you something in return.

What would you like to do?
        (1) MAC Query
        (2) Forgery
        (3) Exit

Choice: 1
msg (hex): 00000000000000000000000000000000
CBC-MAC(msg): dd736ee59e5f5209a514666d4e15fe2e

What would you like to do?
        (1) MAC Query
        (2) Forgery
        (3) Exit

Choice: 1                                                               
msg (hex): 0000000000000000000000000000000000000000000000000000000000000000
CBC-MAC(msg): 29071fe0f5b20c9a373d7004b304478a

What would you like to do?
        (1) MAC Query
        (2) Forgery
        (3) Exit

Choice: 2
msg (hex): dd736ee59e5f5209a514666d4e15fe2e
tag (hex): 29071fe0f5b20c9a373d7004b304478a
If you reach this point, I guess we need to find a better MAC (and not trust TR). UMDCTF{Th!s_M@C_Sch3M3_1s_0nly_S3cur3_f0r_f!xed_l3ngth_m3ss4g3s_78232813}

What would you like to do?
        (1) MAC Query
        (2) Forgery
        (3) Exit

Choice: 3
bye
```

### 

