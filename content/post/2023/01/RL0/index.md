---
title: "強化学習勉強メモ (目次)"
description: 
slug: "RL-0"
date: 2023-01-15T16:00:36+09:00
categories:
    - 強化学習
tags:
image: 
hidden: false
---

## はじめに
　強化学習(Reinforcement learning)の勉強をするにあたり，オイラリー・ジャパンから出版されている[ゼロから作るDeep Learning ❹](https://www.oreilly.co.jp/books/9784873119755/)を読んだ．理解を深めるため，勉強メモを書いた．

## 目次
　各見出しがそのまま記事へのリンクになっている[^1]．番号が振られているが，本の章番号と対応しているわけではない．

[^1]:ブログがこれ関連の記事で埋め尽くされるのを回避するため，記事一覧にはこのページしか表示されない

1. 　[用語確認](/my-blog-4/contents/rl-1/)
1. 　[ベルマン方程式](/my-blog-4/contents/rl-2/)
1. 　[動的計画法](/my-blog-4/contents/rl-3/)
1. 　[モンテカルロ法](/my-blog-4/contents/rl-4/)
1. 　[TD法](/my-blog-4/contents/rl-5/)
1. 　[ニューラルネットワーク(基本編)](/my-blog-4/contents/rl-6/)
1. 　[ニューラルネットワーク(実装編)](/my-blog-4/contents/rl-7/)
1. 　[(WIP) DQN](/my-blog-4/contents/rl-8/)
1. 　[方策勾配法](/my-blog-4/contents/rl-9/)
1. 　[(WIP) タイトル未定(ケーススタディ的なやつを書く)](/my-blog-4/contents/rl-10/)
11. 　[Extra Material (付録)](/my-blog-4/contents/rl-ex/)

最後のExtra Materialには，記号や慣習のまとめや，数学に関する知識などが書いてある．

## その他
- [OpenAI Spinning Up](https://spinningup.openai.com/en/latest/index.html#)の内容も参考にしている．
    - このサイトは，[ゼロから作るDeep Learning ❹](https://www.oreilly.co.jp/books/9784873119755/)の参考文献にもなっている．英語に強い抵抗がないのであれば，とても参考になる．
- [ゼロから作るDeep Learning ❹](https://www.oreilly.co.jp/books/9784873119755/)はわかりやすかった．
- 「○○である」とか「○○してほしい」のように，おまえは何様なんだよという文体で書いてあるので，不快に感じたらごめんなさい．
    - 「である」調にした深い理由はない．

## 参考文献
- [ゼロから作るDeep Learning ❹](https://www.oreilly.co.jp/books/9784873119755/)
- [OpenAI Spinning Up](https://spinningup.openai.com/en/latest/index.html#)

## 感想
### 理論を理解する必要性
　私自身，習うより慣れ派なので，さっさと実装して結果を見て，コード見て何をやっているか理解したいという気持ちが強かった．しかし，強化学習に関しては，理論を先に理解する必要があると感じた．ある問題を解くプログラムをあって，別の問題を解くプログラムへと応用したい場合に，理屈を理解していないと何をどう変えればいいのかが全くわからない．分野の性質上，正しく実装できたとしても良い結果が得られるとは限らない．また，変えられる場所が多かったり，そもそも手法が使えなかったり，手探りで動かしていくのはかなり難しい(完全に理解してからじゃないと実装できないというわけではない．なんとなく理解した時点で実装するのがじぶんに合っていた，ここら辺は個人差がありそう)．

