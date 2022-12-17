
---
title: picoCTF2022 crypto writeup
description: picoCTF2022の300点以上のcrypto問題のwriteup
slug: picoctf
#image: cover.jpg
categories:
    - CTF
    - 数学
tags:
---


## これはなに
picoCTF2022にチームSUSHi-ST0RMで参加しました．crypto問題(300点以上)のwriteupです．


## writeup
### Very Smooth(300pt)
#### 概要
RSA暗号で，$e,n,c$が与えられる．

gen.py
```python
#!/usr/bin/python

from binascii import hexlify
from gmpy2 import *
import math
import os
import sys

if sys.version_info < (3, 9):
    math.gcd = gcd
    math.lcm = lcm

_DEBUG = False

FLAG  = open('flag.txt').read().strip()
FLAG  = mpz(hexlify(FLAG.encode()), 16)
SEED  = mpz(hexlify(os.urandom(32)).decode(), 16)
STATE = random_state(SEED)

def get_prime(state, bits):
    return next_prime(mpz_urandomb(state, bits) | (1 << (bits - 1)))

def get_smooth_prime(state, bits, smoothness=16):
    p = mpz(2)
    p_factors = [p]
    while p.bit_length() < bits - 2 * smoothness:
        factor = get_prime(state, smoothness)
        p_factors.append(factor)
        p *= factor

    bitcnt = (bits - p.bit_length()) // 2

    while True:
        prime1 = get_prime(state, bitcnt)
        prime2 = get_prime(state, bitcnt)
        tmpp = p * prime1 * prime2
        if tmpp.bit_length() < bits:
            bitcnt += 1
            continue
        if tmpp.bit_length() > bits:
            bitcnt -= 1
            continue
        if is_prime(tmpp + 1):
            p_factors.append(prime1)
            p_factors.append(prime2)
            p = tmpp + 1
            break

    p_factors.sort()

    return (p, p_factors)

e = 0x10001

while True:
    p, p_factors = get_smooth_prime(STATE, 1024, 16)
    if len(p_factors) != len(set(p_factors)):
        continue
    ## Smoothness should be different or some might encounter issues.
    q, q_factors = get_smooth_prime(STATE, 1024, 17)
    if len(q_factors) != len(set(q_factors)):
        continue
    factors = p_factors + q_factors
    if e not in factors:
        break

if _DEBUG:
    import sys
    sys.stderr.write(f'p = {p.digits(16)}\n\n')
    sys.stderr.write(f'p_factors = [\n')
    for factor in p_factors:
        sys.stderr.write(f'    {factor.digits(16)},\n')
    sys.stderr.write(f']\n\n')

    sys.stderr.write(f'q = {q.digits(16)}\n\n')
    sys.stderr.write(f'q_factors = [\n')
    for factor in q_factors:
        sys.stderr.write(f'    {factor.digits(16)},\n')
    sys.stderr.write(f']\n\n')

n = p * q

m = math.lcm(p - 1, q - 1)
d = pow(e, -1, m)

c = pow(FLAG, e, n)

print(f'n = {n.digits(16)}')
print(f'c = {c.digits(16)}')
```
output.txt
```
e = 0x10001
n = 809fbd8b667d664f01fe1b0387e0b424efe2035e2dec4d249ace30563d0e1a50050020880c2f01bad63b22e21125d780d887cffdb1165268e6be788cd49ad8a9a1d27482f1a8ccbb37adc0deee65d09f312ebaab854782e2411d917181fef63d478b7e25391ac10d0330cafcb5c8d859ee1e403be029ce5dd75f864deabe5a65645b099afb7af4ed84dd75d1e4b966e2a0662ece5409feeacb2a277ddf05b72153ff6f36524f7693cc432269b56bd8ab3d601844aef6a6130eaaec08f92f9816ed0e7781a23a043570364807bef579c1e9175e3fe2b1d8f52356230feea244ce1b88b2342c9e40b25583a1fe558bdfb3a7115a4c71a6f06b706419ce8e21a3e1
c = 74ce97c4712bc3827a9f6021089c093a7540a6280330a9ec7c6f446a88093c33a6b9a0a1fdf2cad96e32344970adbf26601d9baf2c4e9892dde435dc994bde4754fddbac47a475b3907a455c6f671484b473b5481080224406b1d48d48da5ba0d9fccdc5732cb64c0f02c32ddc1413f66bd95b8e5a929e5b1f14843bd8f5d4747a4aabcc64217a187db6913facce48f2019b5524633153ee40a4376960b7f669f331da29227fa9a8c09a58a6f3db7453dd89a6093c062ff95502cc7cca5ee497c8ec6265413f5d05d1b720b4eb620875b6f6d2a7958e2391835497a106f4c280cd1ca8b9605bbef5952b54dffc028c160c1495e5cd2957f6f2bbb2e868823b6a
```



#### 解法
$p$と$q$の作り方が鍵になってきそうです．
``gen.py``には親切にもDEGUB機能がついているため，これを使って$p$と$q$の生成過程を観察します．
```python
p_factors = [
    2,
    9277,
    16057,
    33223,
    33961,
(中略)
    61837,
    61961,
    62743,
    63577,
    65407,
]

q_factors = [
    2,
    33161,
    48751,
    67391,
    67399,
(中略)
    126227,
    127163,
    127607,
    128047,
]
```
プログラムも合わせて見ると，$p$と$q$は小さな素数の積に$1$を足して作られていることがわかります(もし$1$を足しても素数にならなければ作り直している)．また，$p$と$q$で使われている素数の範囲も異なっています．この状況に対して，$p-1$法という素因数分解の手法が使えます．

$p-1$法は，互いに素な$a,p$で，フェルマーの小定理$a^{p-1} = 1 \pmod p$が成り立つことを使います．

$p-1$の倍数である適当な数$ M $を持ってきて，
$ a^{M} $を計算すれば，$ a^{M} = 1 \pmod p $ が成り立ち，$ a^{M}-1 $ が $ p $ の倍数となります．

つまり，$ a^{M}-1 $と$ n $の最大公約数が$ p $となります．

今回は，$p-1$が$10^{5}$未満の素数の積であることから，$ M $を$10^{5}$以下の素数の積[^1]として，$a$は$2$を選びます．

[^1]:$ M $の値は小さな素数の積であれば何でもいいわけではなく，もし$ M $を$1.5\times 10^{5}$程度までの素数の積にしてしまうと，$ M $が$(p-1)(q-1)$の倍数になってしまい，$ a^{(p-1)(q-1)} =a^{M} = 1 \pmod n$が成り立つため，$n$と$a^{M}-1$の最大公約数が$n$になってしまいます．また，この問題は素数生成の際に同じ素数が2度以上使われていないことがわかっているので，ソルバでは各素数を1度しか掛け算していませんが，わかっていない場合は$p-1$が$2^{4}$や$3^{2}$のような約数を持っている可能性があるので，同じ素数も何度か掛けておくべきです．


この問題を解くにあたって下記の記事を参考にさせていただきました．

```
from Crypto.Util.number import *


n = #省略
c = #
e = 0x10001

a = 2
b = 1
for i in range(2,100000):
    if isPrime(i):
        b*=i
ab =pow(a,b,n)

p = GCD(ab-1,n)
q=n//p

phi=(p-1)*(q-1)
d=pow(e,-1,phi)
m=pow(c,d,n)
print(long_to_bytes(m))
```


### Sum-O-Primes(400pt)
#### 概要
RSA暗号で，$e,n,c$加えて$x=p+q$が与えられる．

```python
e = 65537
x = 1603fc8d929cb31edf62bcce2d06794f3efd095accb163e6f2b78941bd8c646d746369636a582aaac77c16a9486881a9e3db26d742e48c4adcc417ef98f310a0c5433ab077dd872530c3c3c77fe0c080d84154bfdb4c920df9617e986999104d9284516c7babc80dc53718d59032aefdf41b9be53957dea3f00a386b2666d446e
n = 75302ba292dc4bf47ffd690b8edc70ef1fcca5e148b2b9c1b60227788afcfe77a0097929ed3789fe51ac66f678c558244890a09ae4af3e7d098fd366a1c859edabbff1c9e164d5354968798107ae8518fcaab3743de58a141ffd26c1e16cb09fed1f6b0d68536ec7fba744ed120fea8c3a7ac1ebfa55d664d2f321fb44e814650147a9031f3bfa8f69d87393c7d88976d28d147398a355020bcb8e5613f0b29028b77db710e163ca1019fd3c3a065465ea457adec45243c385d12d3a1de3178f6ca05964be92e8b5bc24d420956de96ccc9ce39e70705660eb6b2f4e675aac7d6d7ba45c84223fc5819b37aa85beff1382f1c2c3b97603150f30c17f7e674441
c = 562888c70ce9a5c5ed9a0be1b6196f854ba2efcdb6dd0f79319ee9e1142659f90a6bae67481eb0f635f445d3c9889da84639beb84ff7159dcf4d3a389873dc90163270d80dbb9503cbc32992cb592069ba5b3eb2bbe410a3121d658f18e100f7bd878a25c27ab8c6c15b690fce1ca43288163c544bfce344bcd089a5f4733acc7dc4b6160718e3c627e81a58f650281413bb5bf7bad5c15b00c5a2ef7dbe7a44cce85ed5b1becd5273a26453cb84d327aa04ad8783f46d22d61b96c501515913ca88937475603437067ce9dc10d68efc3da282cd64acaf8f1368c1c09800cb51f70f784bd0f94e067af541ae8d20ab7bfc5569e1213ccdf69d8a81c4746e90
```

#### 解法
連立方程式と見て，$p$と$q$を求めることもできますが，直接$ \phi = (p-1)(q-1)$を求めるほうが早いです．
$$ \phi = (p-1)(q-1) = pq - p - q + 1 = n - x + 1$$
```python
from Crypto.Util.number import *

phi = n-x+1
d=pow(e,-1,phi)
m=pow(c,d,n)
print(long_to_bytes(m))
```

### Sequences(400pt)
#### 概要
下の関数で，``m_func(20000000)``の値の下10000桁を求める問題．
```python
def m_func(i):
    if i == 0: return 1
    if i == 1: return 2
    if i == 2: return 3
    if i == 3: return 4

    return 55692*m_func(i-4) - 9549*m_func(i-3) + 301*m_func(i-2) + 21*m_func(i-1)
```

つまり，

$$ a_0 = 1, a_1 = 2, a_2 = 3, a_3 = 4 $$

$$ a_i = 21a\_{i-1} + 301a\_{i-2} - 9549\_{i-3} + 55692 a\_{i-4} \quad (i>4) $$

のとき，$a\_{20000000}$を$10^{10000}$で割った余りを求める．

#### 解法
線形漸化式であるため，行列を使うテクニックを使って第n項を$O(\log n)$で計算することができます．

任意の項は次の行列を使った式によって表されます．
<img src="https://latex.codecogs.com/svg.image?\begin{pmatrix}21&301&space;&-9549&55692\\1&0&0&0\\0&1&0&0\\0&0&1&0\end{pmatrix}&space;^{n-3}\begin{pmatrix}a_3\\a_2\\a_1\\a_0\end{pmatrix}=\begin{pmatrix}a_n\\a_{n-1}\\a_{n-2}\\a_{n-3}\\\end{pmatrix}"/>

この行列を累乗をナイーブに計算すると結局$O(n)$かかってしまいますが，
繰り返し2乗法(ダブリングと呼ぶこともある)，という手法を使うことで$O(\log n)$で計算することが可能です．

この問題を解くにあたって下記の記事を参考にさせていただきました．

```python
import math
import hashlib
import sys
from tqdm import tqdm
import functools

ITERS = int(2e7)
VERIF_KEY = "96cc5f3b460732b442814fd33cf8537c"
ENCRYPTED_FLAG = bytes.fromhex("42cbbce1487b443de1acf4834baed794f4bbd0dfe08b5f3b248ef7c32b")

def mat_mul(a, b) :
    I, J, K = len(a), len(b[0]), len(b)
    c = [[0] * J for _ in range(I)]
    for i in range(I) :
        for j in range(J) :
            for k in range(K) :
                c[i][j] += a[i][k] * b[k][j]
            c[i][j] %= 10**10000
    return c


def mat_pow(x, n):
    y = [[0] * len(x) for _ in range(len(x))]

    for i in range(len(x)):
        y[i][i] = 1

    while n > 0:
        if n & 1:
            y = mat_mul(x, y)
        x = mat_mul(x, x)
        n >>= 1

    return y


d0 = 0
ret = [[4], [3], [2],[1]]
mat = [[21,301,-9549,55692], [1, 0, 0, 0], [0, 1, 0, 0],[0,0,1,0]]
#ret = mat_mul(mat_pow(mat, ITERS), ret)
#ret = [[1],[1]]
#mat = [[1,1], [1,0]]
ret = mat_mul(mat_pow(mat, ITRES), ret)
print(ret)


## Decrypt the flag
def decrypt_flag(sol):
    sol = sol % (10**10000)
    sol = str(sol)
    sol_md5 = hashlib.md5(sol.encode()).hexdigest()

    if sol_md5 != VERIF_KEY:
        print("Incorrect solution")
        sys.exit(1)

    key = hashlib.sha256(sol.encode()).digest()
    flag = bytearray([char ^ key[i] for i, char in enumerate(ENCRYPTED_FLAG)]).decode()

    print(flag)

if __name__ == "__main__":
    sol = A
    decrypt_flag(sol)
```



### NSA-backdoor(500pt)

#### 概要
離散対数問題


```python
#!/usr/bin/python

from binascii import hexlify
from gmpy2 import *
import math
import os
import sys

if sys.version_info < (3, 9):
    math.gcd = gcd
    math.lcm = lcm

_DEBUG = False

FLAG  = open('flag.txt').read().strip()
FLAG  = mpz(hexlify(FLAG.encode()), 16)
SEED  = mpz(hexlify(os.urandom(32)).decode(), 16)
STATE = random_state(SEED)

def get_prime(state, bits):
    return next_prime(mpz_urandomb(state, bits) | (1 << (bits - 1)))

def get_smooth_prime(state, bits, smoothness=16):
    p = mpz(2)
    p_factors = [p]
    while p.bit_length() < bits - 2 * smoothness:
        factor = get_prime(state, smoothness)
        p_factors.append(factor)
        p *= factor

    bitcnt = (bits - p.bit_length()) // 2

    while True:
        prime1 = get_prime(state, bitcnt)
        prime2 = get_prime(state, bitcnt)
        tmpp = p * prime1 * prime2
        if tmpp.bit_length() < bits:
            bitcnt += 1
            continue
        if tmpp.bit_length() > bits:
            bitcnt -= 1
            continue
        if is_prime(tmpp + 1):
            p_factors.append(prime1)
            p_factors.append(prime2)
            p = tmpp + 1
            break

    p_factors.sort()

    return (p, p_factors)

while True:
    p, p_factors = get_smooth_prime(STATE, 1024, 16)
    if len(p_factors) != len(set(p_factors)):
        continue
    ## Smoothness should be different or some might encounter issues.
    q, q_factors = get_smooth_prime(STATE, 1024, 17)
    if len(q_factors) == len(set(q_factors)):
        factors = p_factors + q_factors
        break

if _DEBUG:
    import sys
    sys.stderr.write(f'p = {p.digits(16)}\n\n')
    sys.stderr.write(f'p_factors = [\n')
    for factor in p_factors:
        sys.stderr.write(f'    {factor.digits(16)},\n')
    sys.stderr.write(f']\n\n')

    sys.stderr.write(f'q = {q.digits(16)}\n\n')
    sys.stderr.write(f'q_factors = [\n')
    for factor in q_factors:
        sys.stderr.write(f'    {factor.digits(16)},\n')
    sys.stderr.write(f']\n\n')

n = p * q
c = pow(3, FLAG, n)

print(f'n = {n.digits(16)}')
print(f'c = {c.digits(16)}')

```


#### 解法

<img src="https://latex.codecogs.com/svg.image?p-1"> と <img src="https://latex.codecogs.com/svg.image?q-1"> はどちらも小さな素数の積であるため，<img src="https://latex.codecogs.com/svg.image?\phi(n)&space;=&space;(p-1)(q-1)&space;"> も小さな素数の積に分解できます．
つまり，Pohlig–Hellman algorithm が使えます．
Pohlig–Hellman algorithm は，現実的な時間で解くことのできる小さな離散対数問題に分割し，中国剰余定理で結果をもとの問題の答えとして復元する，という手法です．




```python
from Crypto.Util.number import *

n = 0x71c27455f38b75f08868b5965d7afba3d81bff38f3b63271ad9250b9d7dc8c909d3555593c2eff9c27a3c259f8e95da41d55544a362494476141c8ccb93fc7d9019d965a20e16d55daf57b5663ede8d5ad97b7be239ecacb2636621ef997854f18f6da1394101dfb8229a2253dbc3ffc995cc6197bd85455f6178c14dbb9a611b3b42530fcdc5c36c5f63fd3796efdfc440a76cf966ff8c56e7e55872a57aa3a335c2b10a82421bcd1cd0d238496f2830d6524f6ba8e9890e30c4e6ad11df8948f4b428d8089a5d9455baca34cee61cb238042bcf8293aab13595aeb90fedabf23b1d0e82c6882824aa0f78c2208de641d9592a170ed839728f6c7e6b6bdf831
y = 0x2560971fdf742d398ae3e677082ab950e99edde5577abcc4d704d65577ec287169d209f2033c82e7574f7e6c27540bb07416cd12b5fca1bb5c7ae23e80bb00b81a5c49116fa3cca6ab72f4a56b2bf0d51c58eedb918faa1e88d6fddb7dd358c1cdaa6e61964284014919662f75adaad5065a3633067b2297cd4657d39c8e2cdb02fd80ba33447abb8bfcd4dd68166f487094108afe5b4378f5f6eb9209f503b718dec9c841089551db648f5b5a84357b2319eb1b27935c3bc47c645f732d36cfffcb0e7f1c8ec5859413e6d62f7ed9af27f4712ca91bbdb9526ea19414c82090a52a78e6bbc2a756b5756017ee08326cd7b1d5dd9fff6afb12bcb93fb541a542

print(f'n = {n}')
print(f'c = {y}')

## get p and q with using p-1 method
p=133120514134071565184901374403906104857402594315193452979400334844456988039351029748429497538981454221984106135005043996378098395846705709422658663585864970394230279241392567216599860405945469882804839379543093459226432299183847151658790842646530907008354991523758458009950957111989465047584759069188272154703
q=107878320716069936845347261730222923402619282584236808136469656719645067255143372538361375857163594280233925663413568686520703251291382637679919197208920902793419007945364505474185442005931731898039583502138819092649272834779913753377381462765827012568092442233669634217190652686190269458879367972776287369087
assert p*q==n
## factors of phi = (p-1)(q-1)
phi=[2, 2, 10369, 11437, 11969, 12491, 33343, 34369, 34687, 34939, 35969, 36467, 36709, 36919, 36973, 36997, 37361, 37379, 37561, 38867, 40897, 41203, 41593, 41801, 42221, 43189, 43481, 43951, 44029, 44953, 45161, 45751, 46649, 46703, 47017, 47221, 49409, 49499, 49783, 50321, 50539, 52081, 53077, 53299, 54367, 54601, 54829, 55147, 55399, 55457, 55661, 56039, 56237, 56267, 56299, 57089, 57373, 57637, 57731, 58897, 59753, 60223, 60733, 61673, 61781, 62459, 62969, 63781, 63901, 64399, 65551, 67651, 68207, 68947, 72287, 74653, 74857, 75011, 77081, 77153, 77239, 78467, 78691, 78877, 81343, 83701, 85009, 88037, 88117, 88397, 89269, 89363, 89477, 90403, 90901, 91009, 94057, 95701, 98387, 100853, 104161, 105097, 106657, 107021, 109121, 109807, 110681, 111599, 112901, 113797, 114883, 115163, 115727, 116009, 117037, 117413, 118799, 120413, 123229, 123973, 124067, 124427, 
125863, 125887, 126631, 127481, 128311, 129671, 129793, 130589]
m=1
for i in phi:
    m*=i

assert m==(p-1)*(q-1)

g=3

def extgcd(a, b):
    if b:
        d, y, x = extgcd(b, a % b)
        y -= (a // b) * x
        return d, x, y
    return a, 1, 0

## V = [(X_i, Y_i), ...]: X_i (mod Y_i)
def remainder(V,W):
    x = 0; d = 1
    for i in range(len(V)):
        X=V[i]
        Y=W[i]
        g, a, b = extgcd(d, Y)
        x, d = (Y*b*x + d*a*X) // g, d*(Y // g)
        x %= d
    return x, d


## Baby-step giant-step
def baby_step_giant_step(g, y, p, q):
    m = int(q**0.5 + 1)
    
    ## Baby-step
    baby = {}
    b = 1
    for j in range(m):
        baby[b] = j
        b = (b * g) % p

    ## Giant-step
    gm = pow(inverse(g, p), m, p)
    giant = y
    for i in range(m):
        if giant in baby:
            x = i*m + baby[giant]
            print("Found:", x)
            return x
        else:
            giant = (giant * gm) % p
    print ("not found")
    return -1


## Pohlig-Hellman algorithm
def pohlig_hellman(p1,p2, g, y, Q):
    print ("[+] Q:", Q)
    X = []
    for q in Q:
        x = baby_step_giant_step(pow(g,((p1-1)*(p2-1))//q,p1*p2), pow(y,((p1-1)*(p2-1))//q,p1*p2), p1*p2, q)
        X.append(x)
    print ("[+] X:", X)
    x ,d= remainder(X,Q)
    return x,d


x,d = pohlig_hellman(p,q, g, y, phi)
print(long_to_bytes(x))
print(long_to_bytes(d))
```

## 感想
400点のSum-O-Primesよりも300点のVery-Smoothの方が難しいと感じました．あと，全完できたので嬉しいです．


