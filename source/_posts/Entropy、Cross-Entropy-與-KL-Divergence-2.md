---
title: Entropy、Cross-Entropy 與 KL-Divergence (2)
date: 2020-05-20 07:28:12
tags: [Machine Learning, Math]
categories: Machine Learning
mathjax: true
---

{% post_link Entropy、Cross-Entropy-與-KL-Divergence-1 %}

在上一篇我們討論了 Entroy 的觀念以及如何從 Entropy 延伸出 Cross Entropy 的概念，有了以上兩個背景知識之後 KL Divergence 其實相當簡單，KL Divergence 用資訊熵的角度來衡量兩個 Distribution 的差異程度，其定義如下

$$
D_{KL}(P || Q) = H(P, Q) - H(P)
$$

或

$$
\begin{align}
D_{KL}(P || Q) &= E_P[logQ(x)] - E_P[logP(x)] \\
&= E_P[logQ(x) - logP(x)] \\
&= E_P[log \frac{Q(x)}{P(x)}]
\end{align}
$$

表示「以分佈 P 為參考，P 和 Q 的差異程度」，要注意第一句話很重要不可省略，因為 KL Divergence 是非對稱的，即 $D\_{KL}(P || Q) \neq D\_{KL}(Q || P)$。

根據上一篇文章的觀點，$H(P, Q)$ 是為欲以分布 $Q$ 編碼分布 $P$ 所需的資料長度，又 $H(P)$ 為欲對分布 $P$ 編碼所需的理想(最小)資料長度，那麼 KL Divergence 即為以 $Q$ 編碼 $P$ 所需「額外」的資料長度，也就是其距離理想編碼還差多少，若 $Q = P$ 則 KL Divergence 為 0 且 $H(P, Q) = H(P, P) = H(P)$，因此當 P 和 Q 越接近，則 KL Divergence 會越小。

# KL Divergence 的非對稱性

雖然 KL Divergence 也常作為 loss function 使用，但必須注意到一點是 KL Divergence 是非對稱的，我們可以觀察其定義

$$
\begin{align}
D_{KL}(P || Q) &= E_P[log \frac{Q(x)}{P(x)}] \\
D_{KL}(Q || P) &= E_Q[log \frac{P(x)}{Q(x)}] \\
&= - E_Q[log \frac{Q(x)}{P(x)}]
\end{align}
$$

其中 $D\_{KL}(P || Q)$ 使用 $P$ 來計算期望值，而 $D\_{KL}(Q || P)$ 使用 $Q$ 來計算期望值，兩者並不相等，其意義也不同。

以 Supervised classification learner 觀點來看，我們的問題經常是要以類神經網路或其他模型去近似某一個真實的分布 $P\_{real}$，使用從 $P\_{real}$ 取樣的資料，樣本的資料分布為 $P\_{data}$，我們通常假設 (或可以證明當樣本數越多時) $P\_{real} \approx P\_{data}$，Supervised classification learner 就是盡量讓模型的機率分布 $Q$ 與 $P\_{data}$ 接近，因此我們可以用 $D\_{KL}(P\_{data} || Q)$ 來衡量 $Q$ 與 $P\_{data}$ 的差異，即以 $Q$ 來表達 $P\_{data}$ 距離最佳表達還差多少。

但單純以評估「兩個分布的差異」而言的話 KL Divergence 似乎不那麼理想，因為距離直覺上應該是對稱的，但以 Supervised classification learner 來說 KL Divergence 已足夠，因為我們有個明顯的參考對象，對於此問題有另一個評估兩分佈差異且具有對稱性值的 metrics 叫做 JS Divergence，即 GAN 所使用的 loss function，GAN 的成功有一部分說法是認為其採用了 JS Divergence，容我到下一篇再作介紹。

# KL Divergence 與 Cross-Entropy

延續上一個小節的範例，Supervised classification learner 的目標就是最小化 $D\_{KL}(P\_{data} || Q)$，但我們再次觀察 KL Divergence 的定義

$$
D_{KL}(P_{data} || Q) = H(P_{data}, Q) - H(P_{data})
$$

你可以發現在我們的問題裡 data 是已知的，因此 $H(P\_{data})$ 是定值，最小化 $D\_{KL}(P\_{data} || Q)$ 等價 $H(P\_{data}, Q)$。

我們更常使用 $H(P\_{data}, Q)$，因為 $D\_{KL}(P\_{data} || Q)$ 有除法運算，若分母值很小的話計算經常會出問題。

但是如果 $P$ 是不確定或不固定的，則兩者是不等價的，因為 $H(P, Q)$ 包含了 $P$ 的訊息量而 $D\_{KL}(P || Q)$ 沒有。

# 小節

KL Divergence 的概念與 Cross Entropy 有相當大的關聯，若能夠理解 Cross Entropy 則 KL Divergence 只是其延伸而已，回憶一下資訊熵最初欲描述的問題，即**對於編碼一已知機率分布的隨機事件，所需的最少位元數**，Entropy 為最佳解，Cross Entropy 為使用另一個分佈近似的最佳解，KL Divergence 則是使用另一個分佈近似與最佳解的差距。

下一篇有機會的話可以來講解 JS Divergence 與 Wasserstein distance，另外兩個評估兩分佈差異的指標。

# Reference

* [Demystifying Entropy - Naoki Shibuya](https://towardsdatascience.com/demystifying-entropy-f2c3221e2550)
* [Demystifying Cross-Entropy - Naoki Shibuya](https://towardsdatascience.com/demystifying-cross-entropy-e80e3ad54a8)
* [Demystifying KL Divergence - Naoki Shibuya](https://towardsdatascience.com/demystifying-kl-divergence-7ebe4317ee68)
* [What is the difference Cross-entropy and KL divergence?](https://stats.stackexchange.com/questions/357963/what-is-the-difference-cross-entropy-and-kl-divergence)
* [Why do we use Kullback-Leibler divergence rather than cross entropy in the t-SNE objective function?](https://stats.stackexchange.com/questions/265966/why-do-we-use-kullback-leibler-divergence-rather-than-cross-entropy-in-the-t-sne)
* [From GAN to WGAN](https://lilianweng.github.io/lil-log/2017/08/20/from-GAN-to-WGAN.html)