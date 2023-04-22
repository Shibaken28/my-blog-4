---
title: "RicercaCTF 2023 Writeup"
description: Crypto3問
slug: RicercaCTF
date: 2023-04-23T01:08:30+09:00
image: 
categories:
    - CTF
    - 数学
---

## Revolving Letters
### 問題
問題ファイル
```python
LOWER_ALPHABET = "abcdefghijklmnopqrstuvwxyz"

def encrypt(secret, key):
	assert len(secret) <= len(key)
  
	result = ""
	for i in range(len(secret)):
	if secret[i] not in LOWER_ALPHABET: # Don't encode symbols and capital letters (e.g. "A", " ", "_", "!", "{", "}")
		result += secret[i]
	else:
    	result += LOWER_ALPHABET[(LOWER_ALPHABET.index(secret[i]) + LOWER_ALPHABET.index(key[i])) % 26]
	return result

flag    = input()
key     = "thequickbrownfoxjumpsoverthelazydog"
example = "lorem ipsum dolor sit amet"
example_encrypted = encrypt(example, key)
flag_encrypted = encrypt(flag, key)

print(f"{key=}")
print(f"{example=}")
print(f"encrypt(example, key): {example_encrypted}")
print(f"encrypt(flag, key): {flag_encrypted}")
```

出力
```none
key='thequickbrownfoxjumpsoverthelazydog'
example='lorem ipsum dolor sit amet'
encrypt(example, key): evvug kztla qtzla exl vqvm
encrypt(flag, key): RpgSyk{qsvop_dcr_wmc_rj_rgfxsime!}
```
### 解答
シーザー暗号の亜種みたいな感じで，「アルファベットの何文字目(ただし`a`が$0$文字目)か」文字分だけずらしたものを出力している．
復元するには逆方向にずらせば良い．

```python

LOWER_ALPHABET = "abcdefghijklmnopqrstuvwxyz"

def decrypt(secret, key):
	assert len(secret) <= len(key)
	result = ""
	for i in range(len(secret)):
	if secret[i] not in LOWER_ALPHABET: # Don't encode symbols and capital letters (e.g. "A", " ", "_", "!", "{", "}")
    	result += secret[i]
	else:
    	result += LOWER_ALPHABET[(LOWER_ALPHABET.index(secret[i]) - LOWER_ALPHABET.index(key[i])) % 26]
	return result

ct = "RpgSyk{qsvop_dcr_wmc_rj_rgfxsime!}"
key ="thequickbrownfoxjumpsoverthelazydog"

print(decrypt(ct, key))
```

### 余談
こういうタイプの問題で`decrypt`関数を作るときは，`encrypt`関数を流用して少し書き換えると楽に作れることが多い．
また，各文字ごとに暗号化の処理が独立しているので，わざわざ逆変換を考えずに，各1文字について「`encrypt`してこの文字になる」ような文字をブルートフォースしても良い．

ちなみに，`example`のlorem ipsum...の文章は，Filler textとかDummy textと呼ばれるもので，文章の埋め込みやデザインの確認などに使われるもの．


## Rotated Secret Analysis
### 問題
$1024$bitの素数$p$と，$p$の上位$512$bitと下位$512$bitを入れ替えた値と等しい素数$q$でRSA暗号を構成している．$e,n,c$が与えられるので，$m$を復号する．

### 解答
$p$の上位$512$bitと下位$512$bitをそれぞれ$x,y$と置く．すると，$p$と$q$は以下のように表せる．
$$
\begin{align}
p &= 2^{512}x + y \\\\
q &= 2^{512}y + x 
\end{align}
$$
よって，$n$は以下のように表せる．
$$
\begin{align}
n &= pq \\\\
&= (2^{512}x + y)(2^{512}y + x) \\\\
&= 2^{1024}xy + 2^{512}(x^2+y^2) + xy
\end{align}
$$
$x$と$y$が$512$bitであることに注意すると，
- $n$の下位$512$bitは$xy$の下位$512$bitと等しい
- 上位$512$bitは，$xy$の上位$512$bit，または$xy$の上位$512$bitに$1$を足したもの[^1]と等しい

[^1]: 繰り上がりを考慮するため

ということがわかる．図(?)にすると以下のような構造になっている．
```none
|-----------------------------------|
| 512bit | 512bit | 512bit | 512bit |
|-----------------------------------|
|       xy        |        0  	    |
|-----------------------------------|
|   0    |     x^2+y^2     |   0    |
|-----------------------------------|
|        0        |       xy        |
|===================================|
|                 n                 |
|-----------------------------------|
```
$xy$が求まれば，$x^2+y^2$も得られ，$2$つの変数$x,y$に関する$2$つの式が得られ，連立方程式として解ける．

あるいは，$\phi(n)$は次のように計算できるので，
$$
\begin{align}
\phi(n) &= (p-1)(q-1) \\\\
&= (2^{512}x + y - 1)(2^{512}y + x - 1) \\\\
&= 2^{1024}xy + 2^{512}(x^2+y^2) + xy - 2^{512}x - 2^{512}y - x - y + 1 \\\\
&= n - 2^{512}(x+y) - (x+y) + 1 \\\\
\end{align}
$$
$x+y=\sqrt{x^2+y^2+2xy}$より$\phi(n)$を直接求めることもできる．ソルバはこちらの方法で実装した．


```python
from Crypto.Util.number import bytes_to_long, getPrime, long_to_bytes, isPrime
from gmpy2 import iroot

n=24456513668907101359271796518022987404822072050667823923658615869713366383971188719969649435049035576669472727127263581903194099017975695864947929128367925596885753443249213201464273639499012909424736149608651744371555837721791748016889531637876303898022555235081004895411069645304985372521003721010862125442095042882100526577024974456438653686633405126923109918116756381929718438800103893677616376097141956262119327549521930637736951686117614349172207432863248304206515910202829219635801301165048124304406561437145821967710958494879876995451567574220240353599402105475654480414974342875582148522218019743166820077511
e=65537
c=18597341961729093099197297749831937867867316311655201999082918827905805371478429928112783157010654738161403312986940377995349388331953112844242407426040120302839420903486499187443737383169223520050969011318937950864196985991944523897440559547618789750180738003138383081085865616976666352985134179471231798760776607911573149993314296253654585181164097972479570867395976653829684069633563438561147707530130563531572708010593487686521808574459865586551335422619675302973576174518308347087901889923892503468385483111040271271572302540992212613766789315482719811321158322571666641755809592299352653626100918299699982602448

a = 1<<512
s = bin(n)[2::]

for i in range(0,2):
    up = int(s[0:512],2) - i # xyの上位512bit
    low = int(s[512*3::],2) # xyの下位512bit

    A = (up<<512) + low # xy
    print(len(bin(A))-2)

    B = n-(A<<1024)-A # x^2+y^2
    B = B>>512

    C = B + 2*A # x^2+y^2 + 2xy = (x+y)^2

    r,T = iroot(C,2)

    if T:
        phi = n + int(r)*(-a-1)+1    
        d = pow(e,-1,phi)
        m = pow(c,d,n)
        print(long_to_bytes(m)) 
```

### 余談
ビットを回転させるネタは作問で考えたことがあった．

## RSALCG
### 問題
次のように線形合同法($z \\% n$は$z$を$n$で割ったあまりを表す)で乱数を生成し，それを$e$乗して$n$で割ったあまりを使って暗号化をしている．
$$
\begin{align}
x_2 &= ax_1 + b \pmod n \\\\
x_3 &= ax_2 + b \pmod n \\\\
r_i &= x_{i}^e \pmod n \quad (i=1,2,3) \\\\
\end{align}
$$
$r_1$と$r_3$が得られるので$r_2$を求める問題．$a,b,n,e$も既知．

```
from Crypto.Util.number import getPrime, getRandomNBitInteger
import os

FLAG = os.getenv("FLAG", "RicSec{*** REDACTED ***}").encode()

def RSALCG(a, b, n):
    e = 65537
    s = getRandomNBitInteger(1024) % n
    while True:
        s = (a * s + b) % n
        yield pow(s, e, n)

def encrypt(rand, msg):
    assert len(msg) < 128
    m = int.from_bytes(msg, 'big')
    return int.to_bytes(m ^ next(rand), 128, 'big')

if __name__ == '__main__':
    n = getPrime(512) * getPrime(512)
    a = getRandomNBitInteger(1024)
    b = getRandomNBitInteger(1024)
    rand = RSALCG(a, b, n)
    print(f"{a = }")
    print(f"{b = }")
    print(f"{n = }")
    print(encrypt(rand, b"The quick brown fox jumps over the lazy dog").hex())
    print(encrypt(rand, FLAG).hex())
    print(encrypt(rand, b"https://translate.google.com/?sl=it&tl=en&text=ricerca").hex())
```

### 解法
$x_1$と$x_3$は線形な関係であるので
$$
\begin{align}
x_3 &= ax_2 + b \\\\
&= a(ax_1 + b) + b \\\\
&= a^2x_1 + ab + b \\\\
\end{align}
$$
次のような同じ根を持つ多項式が得られ，Franklin-Reiter's Related Message Attackが使える(コードはネットからコピペ)．

$$
\begin{align}
ax_1 + b - r_1 &= 0 \pmod n \\\\
a^2x_1 + ab + b - r_3 &= 0 \pmod n \\\\
\end{align}
$$


```python
from Crypto.Util.number import bytes_to_long, getPrime, long_to_bytes, isPrime
from gmpy2 import iroot

def GCD(a,b):
    while b != 0:
        a, b = b, a % b
    return a

e = 65537
a = 104932596701958568145159429432079350581741243925294416012169671604384908382893445168447905864839450402111868722373005467040643335329799448356719960809485814400987619457043584576651627652936429829564657705560266433066823589229257859375942917575729874731586891094997845427952093627170472382405528285663530612106
b = 146908709759837063143862302770110984437045635655026319928249954800644806528614554086681623417268963974691959251767647958752898163761641238519061717835899588252518767306816402052353874469376243689011218283173950163484015487529897260943257598915903245695362042234335492571429369281809958738989439275152307290506
n = 68915438454431862553872087841423255330382510660515857448975005472053459609178709434028465492773792706094321524334359097372292237742328589766664933832084854448986045922250239618283612819975877218019020936022572963433202427817150998352120028655478359887600473211365524707624162292808256010583620102295206287739
c1 = 0x05d7913ff5cd9b6a706249ac05779f2501013ecc05caec697d9270a8a1d3bdaabf898d73410aa0ffbd361a6032adbbfa35386b2e19ec812e9f6bd52e6a2ca1b3760b3076a86ffc94dd6007d74a272e0e3d5326d9e5b01b9211a803338f5899ad6cc29877cc02ca2ff923db79e3ad477bf3820e73596088f54a8cfb187f812201
c2 = 0x1913ba387e6f847dce455dc47092bf83571c34914b7df5875da536f11e68c8a39c78dfe69517ef4b389ea51434e071ce033854fd27c831996aa214cdc02225747a517d44408fbd0232672679bc189f26f6e9b6852a1e68e93ac14e2ce5afc1e050a44733094fe68b0477d4c4b609043e4da4e58390c4f9cf372005653c7f2529
c3 = 0x45054a08d594bd8af1d0fac759ccc799214d0ccce8ae9c5183ef4fba296819bcdf6306f72ee34dcd5d85967fae314d6d3d65a7693b4187adce1d5375dd00c472c0310393cd5bb114602e24d481e276a4926e8886bdcfed96bb8bf9c5812d594f66e46b1737849e8e2f2c3f7b6a45e284c754cf6caf71df34efe143636b5e9079

m1 = bytes_to_long(b"The quick brown fox jumps over the lazy dog")
m3 = bytes_to_long(b"https://translate.google.com/?sl=it&tl=en&text=ricerca")

r1 = c1 ^^ m1
r3 = c3 ^^ m3

# https://crypto.stackexchange.com/questions/30884/help-understanding-basic-franklin-reiter-related-message-attack

s1 = (r1 * pow(a,e,n)) % n

R.<X> = Zmod(n)[]
f1 = (X - b)^e - s1
f2 = (a*X + b)^e - r3
# GCD is not implemented for rings over composite modulus in Sage
# so we'll do it ourselves. Might fail in rare cases, but we
# don't care.
def my_gcd(a, b): 
    return a.monic() if b == 0 else my_gcd(b, a % b)

x1 = int(- my_gcd(f1, f2).coefficients()[0])
print (x1)

r2 = pow((a*x1+b)%n,e,n)
m2 = c2 ^ r2
print(long_to_bytes(m2))
```

### 余談
$e = 65537$に対して計算量が$O(e^2)$らしく，私のパソコンでは実行に30分くらいかかった．HalfGCDというテクニックを使うことでより高速になるらしい．
あと，私はFranklin-Reiter's Related Message Attackの中身は全く理解していない！

