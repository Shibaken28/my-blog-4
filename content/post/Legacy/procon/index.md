---
title: 第32回全国高専プロコン競技部門参戦記と好きなお茶ランキング
description: 2021年度高専プロコンに参加しました
slug: procon32
#image: cover.jpg
categories:
    - 日記
tags:
    - 高専プロコン
---




## これはなに
長野高専から競技部門に「高専の応用呪術IIB」 というチーム名で参加しました．それの参戦記です．

内容は，アルゴリズムの簡単な説明，当日の様子，感想，お茶の順番です．気になるところだけ読んでください．

結果や出題された問題は[公式サイト](https://www.procon.gr.jp/?p=77963)から見ることができます．



## 結果
予選(2組目)1位

準決勝(2組目)1位

決勝4位でした．総合では特別賞を頂きました．ありがとうございます．<s>コードが汚いことこの上ないので選ばれないと思っていた．</s>

上位3校に圧倒的に差をつけられて完敗でした．3位入賞はかなり技術力を上げないとまず届きそうにないですね......．

ほぼ1人参戦については，そもそもチーム戦なのに人を集めなかったのが(実際は集まらなかったのですが，主張が足りなかったのかも)悪いのです．

## アルゴリズムの説明
本番で使用したプログラムは全て私が作りました．第25回のプログラムを参考にしてあり明らかに独創性がありません．様々な解法のソルバを作ったら良かったのですが技量と時間とやる気が足りませんでした．要精進．

プログラム自体8月中旬から作りはじめ，9月の中旬頃にはほとんど完成し，あとはパラメータをチューニングするだけの状態でした．

### 画像復元
貪欲に，ある断片画像の周りに一番くっつきそうな断片画像を繋げていきます．

具体的には，

1. 適当な断片画像一つを「確定」させて配置し，その画像の4辺を集合$S$に追加する
2. $S$の各々について，最も親和度が高い断片画像の辺を「候補」として，親和度の値と一緒に集合$E$に追加する．
3. Eのうち最も親和度が高いものを「確定」した断片画像として配置して，追加した断片画像の辺を「候補として」$S$に追加する．2に戻る．

$E$はプライオリティキューで実装しているので追加，検索が高速$\mathrm{O}(\log N)$です．

親和度は辺の各ピクセルのRGB値を並べたベクトル同士のcos類似度で計算しました．cos類似度は，ベクトルの内積の等式に出てくる$\cos \theta$の値です(2つのベクトルのなす角)．
また，事前に全ての辺同士の親和度を計算しておき，いい感じにソートしておくことで高速化します．

回転可能という条件が予想以上に厄介で，向きや辺の番号の管理がややこしかったです．結局，向きとか辺を{0,1,2,3}で表して，剰余演算でごにょごにょしました．

GUI上では，次の断片画像の確定，任意の確定断片画像の削除，上端下端などの設定，強制的に任意の断片画像を候補から削除する，といった操作が可能です．
ちなみに，サイズの大きいppmファイルの中身を全てchar型の配列に突っ込んでいるので，読み込みが一番時間かかります．

### スライドパズル
端がつながっているルールをうまく使う方法が，案外思いつかないんですこれが．
だんだん小さな正方形の問題にしていく戦法と迷ったのですが，結局端っこから1列ずつ揃える→3×nのサイズになったら3×(n-1)にしていく戦法になりました．具体的な様子は
[youtube](https://youtu.be/KetMK80Mxa4?t=7807:title)
をご覧ください(再生開始場所が埋め込んであります)．

プログラムは至ってシンプルで，断片画像の揃える順番を決めて，一つの移動させる断片画像を決定したらあとはその断片画像を使って一つずつ断片画像を持って正しい場所へ持って行きます．ここで重要なのは移動経路です．
移動される断片画像の経路探索は，最短経路のうちのいくつかをランダムで選びました．選ばれた経路それぞれについて次に書いてある経路探索を行い，その中で評価が高いものを採用<s>す</s>しました．
移動させる断片画像の経路探索はダイクストラ法を使いました．具体的には，各断片画像をノード，交換可能な画像同士にエッジを張りとし，入れ替えたときに正しい位置に近づくか遠ざかるかでコストをつけます．ここのコストのチューニングが難しい．

盤面の評価は単純に正しい場所へのマンハッタン距離の総和になっています．本当は2乗した値のほうがいいことに割と最近気づいたんですが，なんかうまくできませんでした．

また，端っこの処理のプログラムが不十分で今でもたまに止まります．その場合は手動で揃えています．

結構効果的だったのは乱数要素を入れたことです．もちろんコストが上振れ(コストは低いほうがいい)することがありますが，下振れもします．1回回答を出力するまでに1分もかからないので，何度も探索することで，下振れスコアが出ることを願う作戦です．実は，この下振れがなければ負けてました．

この一連のプログラムはどれもSiv3dで作成しました．

余談ですがパンフレット見たときにGUIの画像が掲載されている高専が結構あって驚きました．以下GUIのスクショです．
![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Shibaken_8128/20211010/20211010233432.png)
![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Shibaken_8128/20211010/20211010233535.png)
![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Shibaken_8128/20211010/20211010233602.png)

内輪向けの話になるのですが，Siv3dで，配布されたppmファイルを読み込もうとするとエラーになってハマりました．自前で用意したppmファイルは正常に読み込むのになんでかなとバイナリを比較すると，0A(改行)の次の0D(復帰)の有無の違いがあり，0Dを削除したら読み込めるようになりました．0Dは改行として扱われることがあるらしくておそらくそれが原因な気がしますが，真相は不明．僕が使用したppmビュアーでは正常に表示できていたので原因を突き止めるのに1e9+7年はかかりました．

追記(10/11)
結構前に[issue](https://github.com/Siv3D/OpenSiv3D/issues/591)が出されていてv0.6で修正されたらしいです．



### その他
課題部門のメンバー([@shun_shobon](https://shunnotion.notion.site/7d5871af1af04de785fc8dd76727c078))が問題自動生成ツールを作ってくれました．公式で配布される問題が少ないのでめっちゃ役立ちました．ありがとう．

## 本番の様子
緊張で吐きそうでした．お腹痛い．緊張に弱いのどうにかならんの．
### 1日目朝
テストを回してたところ早速バグが発見されました．死にものぐるいで修正しますが一部は修正しきれず．

### 1日目1回戦
模擬試合で早速ビビりました．
![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Shibaken_8128/20211011/20211011001415.png)
なんだこれは．

復元後の画像
![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Shibaken_8128/20211011/20211011001607.png)

画像の周囲の背景色同士でくっついてしまいました．
しかも，ほぼ水色一色みたいな断片画像もあります．
用意していた修正機能を駆使してなんとか時間内に提出できましたが，16x16でやられたらまずいと思い，後で新機能を追加することになります．

本番の問題は全く問題なくすんなり揃いました．8x8の経路は20秒くらいで求まるので，何度も回すことで乱数ガチャをします．動画を観るとわかりますが1位通過でめちゃくちゃ喜んでます．gif画像にされそう．

### 1日目夜
新機能を追加します．バグ修正する元気がないのと疲れたので素直に寝ました．

### 2日目朝
\#procon32のハッシュタグのついたツイートを見ると，「完徹ww」とか「50%削減」などのワードが飛び交っています．うっそだろwwwと思いながら自分のプログラムも改善しなきゃと思い，試行錯誤しますが全くスコアが伸びずちょっと絶望してました．一応準決勝の組の中で，1回戦の成績はトップですが，そんな情報はあてにならなくなります．またもやバグが見つかったので，アドホックな修正をしていきます．そういえば朝ごはん食べてねぇ．

### 2日目準決勝
問題がこちらです．
![](https://cdn-ak.f.st-hatena.com/images/fotolife/S/Shibaken_8128/20211011/20211011004042.png)
面食らいます．けれどすんなり揃いました．正しく揃っていても間違ってるように見える箇所があり疑心暗鬼になりながらもaccepted 0 0(完全一致)の表示を見て一安心．乱数ガチャをし，1回目のコスト5081から，4477まで下げることができました．1位通過です．

### 2日目決勝
決勝に来れて満足！4位取れたら嬉しい！の気持ちで臨みました．画像復元に急遽実装した新機能が役立ち，4位を取ることができました．

後で試したら決勝の花火と模擬試合の画像は人の手を加えずに揃いました．画像が復元できないとその先がどうしよもないシステム，本当に怖いです．

## 本音と感想
正直，もともと競技部門にあまり乗り気ではありませんでした．
理由としては，

- アルゴリコンではなくマラソン系であること
- 1つの問題だけで勝敗が決まってしまうのでたまたま相性が悪い問題が出ると悲しくなること
- 制約が緩いので難しいこと
- まだなんも決まっていないアルゴリズムを書いた予選資料を提出をしなければならないこと

が挙げられます．そんな中，夏休み中になんとなく画像復元プログラムを作りはじめたや否や，そのままコーディングにハマり，数日後に16x16のアジサイが揃ったときの脳汁をエンジンに熱が入り，入賞するぞー！の気持ちになりました．一度書き始めるとノリノリにカタカタするんだけど，書き始めるまでが長い．

マラソンコンテストは積極的に出て，ヒューリスティックなアプローチに慣れていきたいなと思います．たぶん．

最後に，プロコンの関係者の皆様，観戦に来てくださった先生，応援してくださった方々，本当にありがとうございました．

## 好きなお茶ランキング
ところで，お茶は好きですか．私は好きです．Japanese like tea. year.

注：価格はスーパーなどで売ってるペットボトルのお茶のことを指しています．

### 1位：烏龍茶
この世で一番うまいお茶です．酔ったとき，気持ち悪いときに飲むと非常にすっきりします．あとだいたい価格がお手頃です．

### 2位：緑茶の濃い茶系
体脂肪云々はよくわかりませんが，こちらも気分が悪いときに飲むとすっきりします．こちらは割高なことが多い気がします．

### 3位：ほうじ茶
烏龍茶と風味の傾向が似ていて(当社比)しばしば比較されますが、私は烏龍茶派です．

### 4位：抹茶入り玄米茶
刺身のお供にしたいお茶第1位．某回転寿司のお茶の味と非常に近いです．

### 総括
一般に，お茶は美味しいです．

## まとめ
プロコンのお供は烏龍茶一択ですね．皆さんのおすすめのお茶はなんですか．
