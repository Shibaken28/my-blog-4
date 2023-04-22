---
title: "編入数学過去問特訓 別解(?)"
description: 
slug: hennyu-bekkai
date: 2023-03-16T15:04:39+09:00
image: 
categories:
    - 数学
hidden: true
---
　問題を解き，解説を見ると全然違うやり方で書いてあったものがありました．正当性が怪しいですのであくまで参考程度にいていただきたいです．嘘解法の可能性が大いにあります．

## 1C-02(3)
> $f\_n(x) = x^n \log x$とする，第$n+1$階導関数を求めよ(ただし$n$は自然数)．

解説ではライプニッツの公式を使っていますが，漸化式で考えることもできます．
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
だとわかります．関数の積の高次導関数が出てきたらだいたいライプニッツの公式を使うものなので，そちらの方法もできるべきだと思います．

## 2B-03(1)
置換積分を使います．
$$
\begin{aligned}
I\_1 =  \int ^1 \_0 (\sin ^{-1} x)^2 dx
\end{aligned}
$$
$t=\sin ^{-1}x$と置く．$\sin t = x$だから$\cos t dt = dx$．また，積分範囲は$0 \leq t \leq \pi/2$．
$$
\begin{aligned}
I\_1 &= \int ^{\frac{\pi}{2}} \_0 t^2 \cos t dt \\\\
&= \left[ t^2\sin t + 2t\cos t - 2\sin t \right]^{\frac{\pi}{2}} \_0 \\\\
&= \frac{\pi^2}{4} - 2
\end{aligned}
$$
最後の計算は部分積分を$2$回行います．やっていることはほぼ同じかもしれませんが計算ミスが少なさそう？


