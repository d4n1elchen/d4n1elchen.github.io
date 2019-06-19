---
title: Entropy、Cross-Entropy 與 KL-Divergence (1)
date: 2019-06-19 11:23:00
tags: [Machine Learning, Math]
categories: Machine Learning
mathjax: true
---

在過去我曾 {% post_link 亂數 撰文 %} 討論過亂數的計量方式，其中提到可利用資訊熵來衡量一組資料的亂數程度，在當時的文章中提到了一些熵的基本特性，例如與發生機率成反比、疊加性等，但對於熵的實際數值的意義並沒有太深的理解，當熵的概念延伸到交叉熵時便碰到了障礙。

最近讀到了幾篇寫得很棒的文章，從 Entropy 開始談，推到 Cross-Entropy，最後是 KL-Divergence，讀完後總算是有比較合理的理解，因此重新撰文討論這些概念，內容和思考邏輯基本和我參考的那幾篇文章相同，但稍微用我自己的理解方式敘述，有興趣可以直接跳到最後 Refereces 去看原文。

Cross-Entropy 在 ML 中評估分類模型的 Loss 用得相當多，在多數的 ML 教學中多略過 Cross-Entropy 的解釋，但其實如果了解 Entropy 的實際意義之後來看並不是那麼難理解。

# Entropy

Entropy 由資訊理論之父 Claude Shannon 發明，因此資訊熵又稱 Shannon's Information Entropy，資訊理論當時發展的背景是通訊技術發展初期，傳輸訊息的資源有限，人們在研究怎樣傳遞訊息是最有效率的，要提升資訊的傳輸效率，有兩個切入點，一是提升訊息傳遞技術本身的效率，如更快的傳輸設備，另一是提升單位資料傳輸可傳遞的資訊量，而後者就是資訊理論研究的對象。

換句話說，我們在訊找一個最有效率的編碼方式，用最少的資料表達最多的資訊，資訊熵的提出就是用來評估一套編碼系統的效率。

假設我們現在要對英文字母 A~Z 以 0 和 1 的位元資料編碼，直覺來說我們至少可以用 5 個 bits 來進行編碼，因為 5 個 bits 最多可以表達 32 個值，選其中 26 個出來一一對應就可以表達出 A~Z。

這個例子裡面 A~Z 這 26 個字母所使用的編碼長度都是 5 bits，因此對於這個編碼系統而言，我們平均需要 5 bits 來表達 A~Z。

但實際上 A~Z 出現的頻率並不是一致的，如果我們用較少的 bits 來表達較常見的字母，是不是能夠降低平均所需要的位元數呢？根據維基百科條目 [Letter frequency](https://en.wikipedia.org/wiki/Letter_frequency) 中所提供的資料，我們有 26 個字母發生的機率

|Letter|P(l)|Coding|
|------|----|------|
|a|8.167%|00001|
|b|1.492%|00010|
|c|2.782%|00011|
|d|4.253%|00100|
|e|12.702%|00101|
|f|2.228%|00110|
|g|2.015%|00111|
|h|6.094%|01000|
|i|6.966%|01001|
|j|0.153%|01010|
|k|0.772%|01011|
|l|4.025%|01100|
|m|2.406%|01101|
|n|6.749%|01110|
|o|7.507%|01111|
|p|1.929%|10000|
|q|0.095%|10001|
|r|5.987%|10010|
|s|6.327%|10011|
|t|9.056%|10100|
|u|2.758%|10101|
|v|0.978%|10110|
|w|2.360%|10111|
|x|0.150%|11000|
|y|1.974%|11001|
|z|0.074%|11010|

有了機率分布可以計算一下編碼系統的位元數期望值

$$
\mathbb{E}_{l \in letters}[\text{required bits for letter l}] = \sum_{l \in letters} P(l) (\text{required bits for letter l})
$$

對於剛剛的 5 bits 系統，應該不難計算出期望值等於 5。

觀察下公式可以發現我們如果用越少的 bits 數表達頻率高的字母，整體的期望值會越小，但我們可以這樣任意的刪減位元數嗎？

假設今天我們想減少頻率最高的字母 e 所需要位元數到 4 bits，計算一下會發現位元數的期望值降低了，大約是 4.87 (bits)，但很快的你會發現問題，假設我們從原本的編碼去做刪減，例如去掉第一個 bit，變成 0100，那麼當 e 和其他字母並排在一起時，會發生混淆，例如

```
010010100
```
有兩種解碼方式
```
0100 10100 et
01001 0010 ie
```

當然你可以選擇不會混淆的編碼，例如 1111，這是因為我們並沒有充足的使用 5 bits 的編碼系統，還有一些額外的空間可以使用，但很快地當你想要在進一步縮減時就會出現問題。

一個可能的解決方法是在可能會混淆的編碼中加上額外的 bit 來標示區別，但你可能會想，這樣不就增加了編碼長度了嗎？別忘了我們的目標是降低總體期望值，我們可以把這些比較長的編碼指定給頻率較低的字母，這樣整體而言期望值還是降低的，也就是說，頻率越高的我們用較少的位元數表達，頻率越低的用較多的位元數表達，整體的利用效率可以最大化。

那麼怎樣才是理想上最佳的表達呢？考慮上表中 e 的出現機率 12.7%，為了簡化運算和直觀理解，我們近似成 12.5%，這機率的意思是，每 8 個字母中可能出現 e 一次，也就是說我們的編碼系統至少需要能分辨 8 個值，才有可能表達出機率是 12.5% 的事件，而分辨 8 個值所需要的 bit 數為 $log_2(8) = 3$，整理一下這個最低所需 bit 數的公式，假設我們需要分辨 N 個值

$$
log N = - log \frac{1}{N} = - log P
$$

也就是說要能夠分辨發生機率為 P 的事件所需要的位元數是 $- log P$ 個，這就是前篇文中所提到的 Self information 的公式。

有了這個我們可以根據上表中每個字母的發生機率計算要分辨該字母的發生最少需要的位元數，乘上機率後相加，便是最理想的編碼系統，最小所需的位元數，這就是 Entropy，公式如下

$$
Entropy = \mathbb{E}_{X}[-log P(X)] = -\sum_{x \in X} P(x) log P(x)
$$

用一句話來說的話，資訊熵表示對於編碼一**已知機率分布**的隨機事件，所需的最少位元數。

對於機率分布 P 的 Entropy 也常被寫作 $H(P)$。

# 資訊量？亂度？

在前一篇文章中我們提到我們可以用熵來評估亂數程度，一些直觀的理解我已經在前一篇文中敘述過了，可以過去看看，這邊根據上述的定義嘗試討論看看以資訊理論的角度來看熵的亂度涵義，我們可以透過數學證明均勻分布的資訊熵為最大值，證明如下

我們知道一個離散的完全隨機的事件其機率分布應為均勻分布，我們可以計算其資訊熵之值

$$
Entropy(\text{Uniform distribution}) = - \sum^N \frac{1}{N} log \frac{1}{N} = log N
$$

如果分布不均勻的話，假設我們有以下分布

$$
\{P_1, P_2, P_3, P_4, ...\}=\{\frac{1}{N} + \epsilon_1, \frac{1}{N} + \epsilon_2, \frac{1}{N} + \epsilon_3, \frac{1}{N} + \epsilon_4, \dots\}
$$

其中
$$
-\frac{1}{N} \leq \epsilon_n \leq \frac{N-1}{N} \\
\sum \epsilon_n = 0
$$

計算其資訊熵
$$
\begin{align}
Entropy(\{P_1, P_2, P_3, P_4, ...\}) &= - \sum_{n=1}^N P_n log P_n \\
&= - [(\frac{1}{N} + \epsilon_1) log(\frac{1}{N} + \epsilon_1) + (\frac{1}{N} + \epsilon_2) log(\frac{1}{N} + \epsilon_2) + \dots] \\
&= - [\frac{1}{N}(log(\frac{1}{N} + \epsilon_1) + log(\frac{1}{N} + \epsilon_2) + \dots) \\
& \qquad + \epsilon_1 log(\frac{1}{N} + \epsilon_1) + \epsilon_2 log(\frac{1}{N} + \epsilon_2) + \dots] \\
&= - [\frac{1}{N}(Nlog \frac{1}{N} + log(1 + N\epsilon_1) + log(1 + N\epsilon_2) + \dots) \\
& \qquad + \sum \epsilon_n log \frac{1}{N} + \epsilon_1 log(1 + N\epsilon_1) + \epsilon_2 log(1 + N\epsilon_2) + \dots] \\
&= log N - [\frac{1}{N}(log(1 + N\epsilon_1) + log(1 + N\epsilon_2) + \dots) \\
& \qquad + \epsilon_1 log(1 + N\epsilon_1) + \epsilon_2 log(1 + N\epsilon_2) + \dots] \\
&= log N - [(\frac{1}{N} + \epsilon_1)log(1 + N\epsilon_1) + (\frac{1}{N} + \epsilon_2)log(1 + N\epsilon_2) + \dots] \\
&= log N - \frac{1}{N}[(1 + N\epsilon_1)log(1 + N\epsilon_1) + (1 + N\epsilon_2)log(1 + N\epsilon_2) + \dots] \\
\end{align}
$$

其中因為
$$
-\frac{1}{N} \leq \epsilon_n \leq \frac{N-1}{N} \\
0 \leq 1 + N\epsilon_n \leq N
$$

又
$$
\lim_{x \to 0} x log(x) = 0
$$

我們得到
$$
(1 + N\epsilon_n) log (1 + N\epsilon_n) > 0
$$

使得
$$
[(1 + N\epsilon_1)log(1 + N\epsilon_1) + (1 + N\epsilon_2)log(1 + N\epsilon_2) + \dots] > 0 \\
\begin{align}
Entropy(\{P_1, P_2, P_3, P_4, ...\}) &= log N - \frac{1}{N}[(1 + N\epsilon_1)log(1 + N\epsilon_1) + (1 + N\epsilon_2)log(1 + N\epsilon_2) + \dots] \\
&< logN
\end{align}
$$

我們證明對於離散的隨機事件，完全隨機的 Entropy 會是最大的，此外當機率集中於某個事件時，考慮極端狀況 $\{P_k = 1, P_{n\neq k}=0\}$，Entropy 會等於零，也就是說事件越確定 Entropy 會越低，這性質亦可以推廣至連續的分佈。

總之到目前為止我們總算清楚了 Entropy 的實際涵義以及其可測量亂度的特性。

# Cross-Entropy

上一個章節我們計算了一個隨機事件的熵，但各位要注意到熵的計算假設我們知道事件的機率分布，可實際上很多時候我們並不知道實際的機率分布，我們可能會假設一個機率分布來做編碼系統的設計。

例如說我們有個假定的機率分布 Q

我們可以計算 Q 的 Entropy

$$
Entropy(Q) = \mathbb{E}_{x\in Q}[-logQ(x)] = - \sum_{x\in Q} Q(x)logQ(x)
$$

是為編碼以 Q 為機率分布的隨機事件所需的最小位元數。

但當我們知道實際的機率分布為 P 之後，我們可以評估看看以 Q 所設計出的編碼系統在真實分布下的位元數期望值

$$
CrossEntropy(P,Q) = \mathbb{E}_{x\in P}[-logQ(x)] = - \sum_{x\in P} P(x)logQ(x)
$$

這就是交叉熵，用來評估以另一個機率分布設計編碼系統描述真實機率分布的能力，又記做 $H(P,Q)$。

# $H(P,Q) \ge H(P)$

交叉熵一個重要的特性是其值必大於機率分布 P 的資訊熵，證明大家可以自己嘗試看看，但直觀上意義就是，機率分布 P 的最理想編碼長度就是 H(P)，用其他不準確的分布來編碼其效率必小於此編碼 (也就是說需要更多的位元數來表達)。

# Cross-Entropy 作為 loss function

在分類問題中最常被拿來當作 loss function 的就是 Cross-Entropy，但通常一般的 ML 教學都會略過 Cross Entropy 的解釋，不過理解了 Entropy 和 Cross Entropy 的概念之後其實不難理解為什麼可以用交叉熵來作為分類問題的 loss function。

分類模型其實可以看成是去近似一個條件機率分布 Q，給定資料 X，每個 Class 發生的機率為何，即 $Q(Class_i|X)$，因次每次訓練時，我們都有實際的 (GT) 和近似的 (Model) 機率分布，實際的分布以 One-hot encoding 表達，例如某個資料 $x$ 屬於 $Class_k$，其實際的條件機率分布為 $P(Class_{i\ne k}|x) = 0, P(Class_{k}|x) = 1$，而模型的輸出為 $Q(Class_i|x) = \{Q_1, Q_2, \dots, Q_n\}$

透過計算 H(P,Q)，我們可以得到以 Q 來表達 P 的能力，當 Q 越接近 P 時 H(P,Q) 會越小，因此可以將其作為衡量 P、Q 兩分佈相似的程度。

而 Logistic Regression 所用的 Binary Cross-Entropy 公式也可以從 Cross-Entropy 推得

$$
\begin{align}
H(P,Q) &= - \sum_{x\in P} P(x)logQ(x) \\
&= - (P(1)logQ(1) + P(0)logQ(0)) \\
&= - (P(1)logQ(1) + (1-P(1))log(1-Q(1))) \\
&= - (y\ log\ y' + (1-y)\ log\ (1-y'))
\end{align}
$$

# 小結

以上，應該對 Entropy 和 Cross Entropy 有比較有條理的理解了，KL Divergence 留到下次再寫，KL Divergence 是 Generative model 中用非常多的一個指標，其概念也是從 Entropy 延伸出去的，欲了解 Generative model 必須先理解 KL Divergence，KL Divergence 和 Cross Entropy 類似，都可以評估兩個分佈的差異程度，但在一些特殊情況下 KL Divergence 可以更好的評估兩分佈實際上的差異。

# References
### Demystifying Entropy - Naoki Shibuya
https://towardsdatascience.com/demystifying-entropy-f2c3221e2550
### Demystifying Cross-Entropy - Naoki Shibuya
https://towardsdatascience.com/demystifying-cross-entropy-e80e3ad54a8
### Demystifying KL Divergence - Naoki Shibuya
https://towardsdatascience.com/demystifying-kl-divergence-7ebe4317ee68
### Why do we use Kullback-Leibler divergence rather than cross entropy in the t-SNE objective function?
https://stats.stackexchange.com/questions/265966/why-do-we-use-kullback-leibler-divergence-rather-than-cross-entropy-in-the-t-sne/265989
