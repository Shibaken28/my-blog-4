---
title: "編入数学過去問特訓 別解(?)"
description: 
slug: hennyu-bekkai
date: 2023-03-16T15:04:39+09:00
image: 
categories:
    - 数学
---
　問題を解き，解説を見ると全然違うやり方で書いてあったものがありました．正当性が怪しいですのであくまで参考程度にいていただきたいです．

## 1C-02(3)
> $f\_n(x) = x^n \log x$とする，第$n+1$階導関数を求めよ(ただし$n$は自然数)．

解説ではライプニッツの公式を使っていますが，漸化式のように考えることもできます．
$$
\begin{aligned}
f\_n^{(n+1)}(x) &= \left( n x^{n-1} \log x + x^n \frac{1}{x} \right)^{(n)} \\\\
&= \left( n x^{n-1} \log x + x^{n-1} \right) ^{(n)}\\\\
&= \left( nf\_n(x) + (n-1) x^{n-1} \right)^{(n)} \\\\
&= nf\_n^{(n)}(x) + \left((n-1)x^{n-1}\right)^{(n)} \\\\
&= nf\_n^{(n)}(x)
\end{aligned}
$$
また，
$$
\begin{aligned}
f\_1^{(2)}(x) &= \left(\log x + 1 \right)^{\prime} \\\\
&= \frac{1}{x}
\end{aligned}
$$
であるので，
$$
f\_n^{(n+1)}(x) = \frac{n!}{x} 
$$
だとわかります．

