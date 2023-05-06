---
title: "CPCTF2023 writeup"
description: Medium以下のPPCとCrypto
slug: cpctf2023
date: 2023-04-26T21:00:00+09:00
image: 
categories:
    - CTF
    - 競技プロ

draft: true
---

PPCはそれなりに健闘しましたが，Cryptoがダメでした．

## Crypto

### Entrance to the Crypto World (61solves)
```none
CJ, YGNEQOG.JQY FKF AQW NKMG VJKU UKORNG UWDUVKVWVKQP EKRJGT?
YKVJ C NKVVNG KPIGPWKVA, UWEJ C UWDUVKVWVKQP EKRJGT ECP DGEQOG C XGTA FKHHKEWNV EKRJGT.
QPG GZCORNG KU VJG GPKIOC ETGCVGF DA VJG IGTOCPU. YJA FQP'V AQW EJGEM KV QWV?
PQY, JGTG KU VJG HNCI. EREVH{3PL0A_7J3_Y0TNF_0H_ETAR70!}
```

換え字式の暗号．ネットでsubstitution cipher solverなどと調べて出てくるいい感じのサイトで解いてもらう．
私は[https://quipqiup.com/](https://quipqiup.com/)を使用した．

```none
AH, WELCOME. HOW DID YOU LIKE THIS SIMPLE SUBSTITUTION CIPHER?
WITH A LITTLE INGENUITY, SUCH A SUBSTITUTION CIPHER CAN BECOME A VERY DIFFICULT CIPHER.
ONE EXAMPLE IS THE ENIGMA CREATED BY THE GERMANS. WHY DON'T YOU CHECK IT OUT?
NOW, HERE IS THE FLAG. CPCTF{3NZ0Y_7H3_W0RLD_0F_CRYP70!}
```
スペルを適当に補正して，`CPCTF{3NJ0Y_7H3_W0RLD_0F_CRYP70}`が得られる．

### Simple (6solves)
解けませんでした．

次の数が与えられる．$i$は虚数単位で，$\mod n$は実部と虚部をそれぞれ$n$で割った余りを表す．
$$
\begin{align}
c_1 &= a+ib &= (p + im)^e \mod n \\\\
c_2 &= c+id &= (q + im)^e \mod n 
\end{align}
$$



### Misuse (6solves)
解けませんでした．


## PPC

### 注意
言語はC++を使用する．また，以下のコードが先頭にあるものとする．スペースの都合で一部の関数などを省略する．
```cpp
#include <iostream> // cout, endl, cin
#include <string> // string, to_string, stoi
#include <vector> // vector
#include <algorithm> // min, max, swap, sort, reverse, lower_bound, upper_bound
#include <utility> // pair, make_pair
#include <tuple> // tuple, make_tuple
#include <cstdint> // int64_t, int*_t
#include <cstdio> // printf
#include <map> // map
#include <queue> // queue, priority_queue
#include <set> // set
#include <stack> // stack
#include <deque> // deque
#include <unordered_map> // unordered_map
#include <unordered_set> // unordered_set
#include <bitset> // bitset
#include <cctype> // isupper, islower, isdigit, toupper, tolower
#include <iomanip>
#include <climits>
#include <cmath>
#include <functional>
#include <numeric>
#include <regex>
#include <array>
#include <fstream>
#include <sstream>

#define rep(i, n) for (int i = 0; i < (int)(n); i++)
#define rep1(i, n) for (int i = 1; i <= (int)(n); i++)
#define repl(i,l,r) for (int i = l; i < (int)(r); i++)
#define all(a) a.begin(),a.end()
#define Pii pair<int,int>
#define Pll pair<long,long>
#define INFi 1000000001
#define INFl 1000000000000000001
#define ll long long
using namespace std;
```


### Count Coins (59solves)
- 問題リンク： https://cpctf.space/challenges/59584b8c-827a-4acc-b6f4-303eeb1ed950

問題文の通りに実装するだけ．コインに最大値と最小値があることに注意．
```cpp
int main(void){
    int N,X;cin>>N>>X;
    int coins = X;
    rep(i,N){
        int a;cin>>a;
        if(a==1){
            coins += 1;
        }else{
            coins -=3;
        }
        if(coins<0)coins=0;
        if(coins>10)coins=10;
    }
    cout<<coins<<endl;
}
```


### Transfer (52solves)
- 問題リンク： https://cpctf.space/challenges/ee925a4c-2d8f-46ca-97a8-49aa5824a85c

読解すると，$A+B$と$C+D$のうち小さい方を出力すれば良いことがわかる．

```cpp
int main(void){
    int A,B,C,D;
    cin>>A>>B>>C>>D;
    cout<<min(A+B,C+D)<<endl;
}
```

### Find cpctf (33solves)
- 問題リンク： https://cpctf.space/challenges/a9efb9c7-93a5-471f-9e41-b757f16cfddc

ある連続した$5$文字を見たときに，それを`cpctf`にできるかどうか，という判定問題を考える．
この判定問題は，`cpctf`にするのに必要なシフトする数が$A$に含まれているかどうか，という問題に帰着できる．

```cpp
int main(void){
    int N;cin>>N;
    map<int,int> A;
    rep(i,N){
        int a;cin>>a;
        A[a]++;
    }
    bool ok=false;
    string S;cin>>S;
    //cout<<A<<endl;
    for(int i=0;i<S.size()-4;i++){
        string T="cpctf";
        map<int,int>need;
        for(int j=0;j<5;j++){
            int from = S[i+j]-'a';
            int to = T[j]-'a';
            need[(to-from+26)%26]++;
        }
        bool yes=true;
        for(auto&[k,v]:need){
            if(A[k]<v){
                yes=false;
                break;
            }
        }
        if(yes){
            ok=true;
            break;
        }
    }
    cout<<(ok?"Yes":"No")<<endl;
}
```

### Floor Sqrt (34solves)
- 問題リンク： https://cpctf.space/challenges/31703a20-e604-4450-93e2-16961b1d06bc

実験をしてみる．
- $n=1$のとき，$\lfloor \sqrt{1} \rfloor=1, \lfloor \frac{1}{1} \rfloor=1$より成立
- $n=2$のとき，$\lfloor \sqrt{2} \rfloor=1, \lfloor \frac{2}{1} \rfloor=2$より不適
- $n=3$のとき，$\lfloor \sqrt{3} \rfloor=1, \lfloor \frac{3}{1} \rfloor=3$より不適
- $n=4$のとき，$\lfloor \sqrt{4} \rfloor=2, \lfloor \frac{4}{2} \rfloor=2$より成立
- $n=5$のとき，$\lfloor \sqrt{5} \rfloor=2, \lfloor \frac{5}{2} \rfloor=2$より成立
- $n=6$のとき，$\lfloor \sqrt{6} \rfloor=2, \lfloor \frac{6}{2} \rfloor=3$より不適
- $n=7$のとき，$\lfloor \sqrt{7} \rfloor=2, \lfloor \frac{7}{2} \rfloor=3$より不適
- $n=8$のとき，$\lfloor \sqrt{8} \rfloor=2, \lfloor \frac{8}{2} \rfloor=4$より不適
- $n=9$のとき，$\lfloor \sqrt{9} \rfloor=3, \lfloor \frac{9}{3} \rfloor=3$より成立
- $n=10$のとき，$\lfloor \sqrt{10} \rfloor=3, \lfloor \frac{10}{3} \rfloor=3$より成立

数字を眺めると，$\lfloor \sqrt{n}\rfloor$の変わり目で不適から成立に変わっていることがわかる．
$\lfloor \sqrt{n}\rfloor$が同じ数ごとにまとめて考えるのが良さそうである．
$\lfloor \sqrt{x}\rfloor=a$のとき， $\lfloor \frac{x}{\lfloor \sqrt{x}\rfloor} \rfloor = \lfloor \frac{x}{a} \rfloor=a$を成り立たせるためには，明らかに$a\leq x < 2a$である必要があり，その個数は$2a-a=a$である． $a$の値ごとに計算していくことで，$O(\sqrt{N})$で解ける．

```cpp
int main(void){
    std::cin.tie(0)->sync_with_stdio(0);
    long N;cin>>N;
    long cnt = 0;
    for(long i=1;i*i<=N;i++){
        long s = i*i;
        long e = min(i*i+i-1,N);
        if(s>e)break;
        cnt += (e-s+1);
    }
    cout<<cnt<<endl;
}
```

### Geometric Progression Sum (24solves)
- 問題リンク： https://cpctf.space/challenges/5a22698d-64a8-4fc1-8e39-8f29bba9da6b

等比数列は，前の数がわかれば次の数もわかる．そして，各クエリで公比が同じなところをうまく使えないかを考える．
そこで，次のような数列$S$を作る．

- $A_{i-1}$に$S_i$を加算して，$X$倍すると$A_i$になる．

クエリ$(B,L,R)$が来たときには，$S_L$に$BX^{-1}$を加算，$S_{R+1}$に$-BX^{(R-L)}$を加算することで表すことができる．よって各クエリ$O(1)$，最後の$A$の計算に$O(N)$で処理できる．


```cpp
using mint = Fp<998244353>; //modint構造体は省略

int main(void){
    long N,X,Q;cin>>N>>X>>Q;
    vector<mint> sub(N+1,0);
    mint x = X;
    rep(i,Q){
        long B,L,R;cin>>B>>L>>R;
        mint b = B;
        sub[L-1] += b*1/x;
        sub[R] -= b*modpow(x,R-L);
    }
    mint now = 0;
    rep(i,N){
        now += sub[i];
        now *= x;
        cout<<now<<" ";
    }
    cout<<endl;
}
```

### Whisper Sucu (24solves)
- 問題リンク： https://cpctf.space/challenges/396492ab-d9f1-4ff8-8b20-4aff2982121e

信号の条件がなければ，単純なダイクストラ法で解くことができる．
そこで，ダイクストラ法を次のように改造する．
- ある頂点までの奇数分かかったときの最短の距離と，偶数分かかったときの最短の距離をそれぞれ管理する．
- 頂点までの距離の更新は，奇数分と偶数分の別々に行う．

```cpp
struct Edge{
    int to;
    long type;
};

struct WeightedVertex{
    int v;
    long cost;
    int type;
};

using Graph = vector<vector<Edge>>;

int main(void){
    std::cin.tie(0)->sync_with_stdio(0);
    int N,M; cin>>N>>M;
    Graph G(N);
    rep(i,M){
        int u,v,s;cin>>u>>v>>s;
        u--;v--;s--;
        G[u].push_back({v,s});
        G[v].push_back({u,s});
    }

    vector<vector<long>> D(N,vector<long>(2,INFl));
    D[0][0] = 0; // 0:even, 1:odd

    auto comp = [](WeightedVertex &l,WeightedVertex &r){return l.cost > r.cost;};
    priority_queue < 
        WeightedVertex,
        vector<WeightedVertex>,
        function<bool(WeightedVertex&,WeightedVertex&)>
        > qu (comp);

    qu.push({0,0,0});
    while(!qu.empty()){
        auto a = qu.top(); qu.pop();
        int f = a.v;
        long c = a.cost;
        int t = a.type;
        int nt = 1-t;
        for(auto&e:G[f]){
            if(e.type == 2 || e.type == t){
                if(chmin(D[e.to][nt],c + 1)){
                    qu.push({e.to,c + 1,nt});
                }
            }
        }
    }
    long ans = min(D[N-1][0],D[N-1][1]);
    if(ans == INFl) ans = -1;
    cout<<ans<<endl;
}
```

### Max Min GCD (14solves)
- 問題リンク：https://cpctf.space/challenges/6986b85f-9849-4d44-8304-218374cd2847
次のアルゴリズムで解くことができる
- 約数の集合$S$と$T$を用意しておく．はじめは空集合．
- $k=N,N-1,N-2,\cdots,2,1$について順番に
    - $A_k$の約数を$S$に追加する．計算量は$O(\sqrt{A_k})$
    - $B_k$の約数を$T$に追加する．計算量は$O(\sqrt{B_k})$
    - 次の数のうち最大値を$C_k$とする．計算量は$O(\sqrt{A_k}\log|T|+\sqrt{B_k}\log|S|)$
        - $A_k$の約数を列挙していき，$T$に含まれているもののうち，最大のもの．
        - 同様に，$B_k$の約数を列挙していき，$S$に含まれているもののうち，最大のもの．

全体では，$A$の最大値を$A_{\rm{max}}$，$B$の最大値を$B_{\rm{max}}$とすると，計算量は$O(N(\sqrt{A\_{\rm{max}}}\log|T|+\sqrt{B\_{\rm{max}}}\log|S|))$となる．$\log |T|$がそんなに大きくないことを祈りつつ提出すると通る．


なお，横着してゴミコードを書いてしまった(ごめんなさい)．

```cpp

long GCD(long a,long b){
    if(a<b)return GCD(b,a);
    if(b==0)return a;
    return GCD(b,a%b);
}

long GCD(vector<long>&A){
    long gcd = A.front();
    for(auto&a:A)gcd = GCD(gcd,a);
    return gcd;
}

long LCM(long a,long b){
    return (a/GCD(a,b))*b;
}


long LCM(vector<long>&A){
    long lcm = 1;
    for(auto&a:A)lcm = LCM(lcm,a);
    return lcm;
}


int main(void){
    int N;cin>>N;
    vector<int> A(N);
    vector<int> B(N);
    rep(i,N)cin>>A[i];
    rep(i,N)cin>>B[i];
    set<int> ad;
    set<int> bd;
    vector<int> C(N,1);
    for(int x=N-1;x>=0;x--){
        for(int i=1;i*i<=A[x];i++){
            if(A[x]%i==0){
                ad.insert(i);
                ad.insert(A[x]/i);
            }
        }
        for(int i=1;i*i<=B[x];i++){
            if(B[x]%i==0){
                bd.insert(i);
                bd.insert(B[x]/i);
            }
        }
        for(int i=1;i*i<=A[x];i++){
            if(A[x]%i==0){
                if(bd.count(i)){
                    chmax(C[x],i);
                }
                if(bd.count(A[x]/i)){
                    chmax(C[x],A[x]/i);
                }
            }
        }
        for(int i=1;i*i<=B[x];i++){
            if(B[x]%i==0){
                if(ad.count(i)){
                    chmax(C[x],i);
                }
                if(ad.count(B[x]/i)){
                    chmax(C[x],B[x]/i);
                }
            }
        }
    }
    printArray(C);
}
```

### God Field (13solves)
- 問題リンク：https://cpctf.space/challenges/19222f0c-c5da-4af9-a2e3-ccf138a23899

制約からbitDPを疑う．
- $dp[i][S]$ := $i$回目の攻撃を受けたとき，防具の集合$S$を使用したときの残りの最大HP
- $dp[i][S]$は$dp[i-1][T]$から計算できる．
- $dp[0][0] = X$で初期化．

```cpp
int main(void){
    int N,M,X;cin>>N>>M;
    cin>>X;
    vector<long> A(N);
    vector<long> B(M);
    vector<bool> C(M);
    rep(i,N){
        cin>>A[i];
    }
    rep(i,M){
        cin>> B[i];
        int c;cin>>c;
        C[i] = (c==1);
    }
    vector<vector<long>> dp(M+1,vector<long>(1<<N,0));
    //dp[i][S] := i番目の攻撃までで防具Sを使用したときの最大HP
    dp[0][0] = X;
    for(int i=1;i<=M;i++){
        for(int j=0;j<(1<<N);j++){
            for(long T=j; ; T=(T-1)&j) {
                int S = j^T;
                long d = 0;
                for(int k=0;k<N;k++){
                    if(T&(1<<k)){
                        d+=A[k]; //ディフェンス力を加算
                    }
                }
                //cout<<d<<endl;
                long a = max(0L,B[i-1]-d);//ダメージを計算
                if(C[i-1]){
                    if(a>0)chmax(dp[i][j],0L); //ダメージがあったら死ぬ
                    else chmax(dp[i][j],dp[i-1][S]-a);
                }
                else{
                    chmax(dp[i][j],dp[i-1][S]-a);
                }
                if(T==0)break;
            }
        }
    }
    long ans = 0;
    for(int i=0;i<(1<<N);i++){
        chmax(ans,dp[M][i]);
    }
    cout<<ans<<endl;
}
```

### Digital Clock (12solves)
- 問題リンク：https://cpctf.space/challenges/22317b83-25fc-47a2-91d1-43e6fe3c67a7

愚直コードで実験をすると次のことがわかります．証明はできない！
- 答えは$\rm{min}(\rm{LCM}(H,M),K+1)$
```cpp
int main(void){
    std::cin.tie(0)->sync_with_stdio(0);
    long H,M,K;cin>>H>>M>>K;
    long d = LCM(H,M);
    cout<<min(d,K+1)<<endl;
    /*
    愚直コード
    set<pair<long,long>>st;
    for(int a=0;a<=K;a++){
        long h = a%H;
        long m = (K-a)%M;
        st.insert({h,m});
    }
    cout<<st.size()<<endl;
    cout<<st<<endl;
    */
}
```

### Move Road (12solves)
- 問題リンク：https://cpctf.space/challenges/0f0cdab0-4295-4012-b6fe-47d31e4dbe4b

車の移動の処理を毎回行うのは大変なので，言い換えをします．
- Sotatsu君は時刻$t=0.5,1.5,2.5,\dots$に左に移動する．ただし，左端にいた場合は右端に移動する．
- Sotatsu君は時刻$t=1,2,3,\dots$に左，左上，左下のいずれかに移動する．ただし，右端と左端はつながっている(トーラス状)．

これにより，シンプルな探索問題に帰着できます．
```cpp

int main(void){
    std::cin.tie(0)->sync_with_stdio(0);
    int H,W;cin>>H>>W;
    vector<string> M(H);
    rep(i,H){
        string S;
        cin>>S;
        M[i]=S;
    }
    int y=0,x=(int)M[0].size()-1;
    vector<int> dy={-1, 0, 1};
    vector<int> dx={-1,-1,-1};
    vector<vector<int>> dis(H,vector<int>(M[0].size(),INFi));
    dis[y][x]=0;
    queue<Pii> Q;
    Q.push({y,x});
    while(!Q.empty()){
        Pii p = Q.front();Q.pop();
        int y=p.first,x=p.second;
        rep(i,3){
            if(y>0)if(i==0&&M[y-1][x]=='D')continue;
            if(y<H-1)if(i==2&&M[y+1][x]=='D')continue;
            int ny=y+dy[i],nx=x+dx[i];
            if(ny<0||ny>=H)continue;
            if(nx==-1)nx=W-1;
            if(nx>=W)nx=0;
            if(M[ny][nx]=='D')continue;
            if(dis[ny][nx]<=dis[y][x]+1)continue;
            dis[ny][nx]=dis[y][x]+1;
            Q.push({ny,nx});
        }
    }
    bool ok = false;
    rep(i,M[0].size()){
        if(dis[H-1][i]<INFi)ok=true;
    }
    if(ok)cout<<"Yes"<<endl;
    else cout<<"No"<<endl;
}
```

