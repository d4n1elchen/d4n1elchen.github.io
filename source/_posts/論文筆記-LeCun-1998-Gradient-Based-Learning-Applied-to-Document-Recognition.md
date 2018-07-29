---
title: 論文筆記 - LeCun 1998 - Gradient-Based Learning Applied to Document Recognition
date: 2018-07-29 22:59:08
tags: [statistics,math,machine learning,data analysis,deep learning,neural network]
categories: Machine Learning
---

Y. Lecun, L. Bottou, Y. Bengio and P. Haffner, "Gradient-based learning applied to document recognition," in Proceedings of the IEEE, vol. 86, no. 11, pp. 2278-2324, Nov 1998.
doi: 10.1109/5.726791

URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=726791&isnumber=15641

Full Text: http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf

LeCun's Website: http://yann.lecun.com/exdb/lenet/

前言
---
這篇是 LeCun 於 1998 年發表的論文，LeNet-5 在這篇論文中被提出，是卷積神經網路研究最早期的研究之一，可說是 CNN 的開山始祖 (卷積層的概念在更早 1989 也是 LeCun 的論文 [Backpropagation Applied to Handwritten Zip Code](https://ieeexplore.ieee.org/document/6795724) 就有提到過了，但 LeNet 的架構和名稱一直到這篇才正式被提出來)。

本次閱讀著重在 LeNet 以及 CNN、BP 的部分，有關手寫辨識的模組設計部分會略讀或跳過。

Abstract
---
作者提到多層神經網路可以擬合出複雜的平面，擅長於高維度的模式(Pattern)辨認，相較於傳統的辨識方法而言不必有太多的前處理就可以有很好的效果。

過去的文件識別方法多基於階層式的結構，需要有不同的模組和演算法來分別進行特徵萃取、辨識、語言模型等工作來完成文件識別，而這篇文章所提出的 Graph Transformer Network (GTN) 架構可建立出通用化的學習系統，可用相同的方法同時且自動的訓練不同功能的模型。

I. Introduction
---
作者先回顧了近年的機器學習方法發展對文件辨識帶來的好處，以及傳統 OCR 的架構，提出了新的 GTN 的概念；傳統 OCR 的方法的準確性多仰賴於前處理方法的設計，看前處理是否可以找出具有代表性的特徵，具有以下性質：(a) 能夠很好的區分不同的類別，(b) 變形對於萃取出的特徵影響相對較小，並將這些特徵再送至分類模型中進行識別。

過去的分類模型多只在低維度的特徵表達中有較好的表現，但近幾十年有三個重要的改變使得這些機器學習演算法開始能處理高維度的問題：(1) 低成本高效率的計算單元使得我們可以開始嘗試一些暴力的數值方法來對模型進行 refine，(2) 對於一些我們感興趣的問題有大量的資料可以被取得，(3) 最重要的是一些能夠處理高維度輸入的機器學習演算法的出現。

作者提到雖然我們想加入更好的通用性，但對學習演算法而言，能夠整合先驗知識可以使得模型更容易訓練成功，而在多層神經網路中加入先驗知識的方法是根據你對問題的瞭解來調整網路結構，CNN 即是運用此觀念，給權重加入限制條件，讓神經網路能夠針對 2D 的影像資料觀測出局部的圖型特徵。

### A. Learning from data
{% raw %}
這一小節主要在簡述一個 General 的機器學習模型及參數估計的概念，機器學習為透過資料找尋一個函式對應關係

$$
Y^p = F(Z^p, W)
$$

即給定第 p 個輸入 $Z^p$，給予對應的輸出 $Y^p$，以手寫辨識來說 $Y^p$ 就是對應的數字類別，$W$ 為可調整的參數；損失函式為計算 $Y^p$ 與正確的輸出 $D^p$ 之間的差異，可以以下公式表達

$$
E^p = \mathcal{D}(D^p, F(Z^p, W))
$$

一組訓練資料集的平均損失為 $E_{train}$，機器學習的過程，或參數估計，即在尋找使得 $E_{train}(W)$ 最小的 $W$。

通常模型的準確度我們用另外一組與訓練資料獨立的測試資料集，計算 $E_{test}$ 來評估，有一些理論研究表示 $E_{test}$ 和 $E_{train}$ 之間的差距與訓練樣本數有關，可近似於

$$
E_{test} - E_{train} = k(h/P)^\alpha
$$

其中 $P$ 為訓練樣本數，$h$ 為 Effective Capacity，即模型的複雜程度，$\alpha$ 介於 0.5~1.0 之間，$k$ 為常數。

這節後半段也有提到有關模型複雜度與模型誤差之間的 trade-off 問題，但目前我還看不太懂他的表達方式。
{% endraw %}
### B. Gradient-Based Learning
{% raw %}
這小節主要解釋梯度下降法的精神以及公式，若損失函數處處連續可微，我們可以透過梯度來估計改變某個參數對 $E_{train}$ 的影響程度，藉此來更新參數，最簡單的方式為梯度下降法

$$
W_k = W_{k-1} - \epsilon \frac{\partial E(W)}{\partial W}
$$

其中 $\epsilon$ 為一比例參數，可為常數，或複雜點可以是個變數。

這節也提到了 Stochastic Gradient Descent，即 SGD 演算法，SGD 通常有較好的收斂性，有關 SGD 的詳細說明可參考 Appendix B。
{% endraw %}
### C. Gradient Back-Propagation

這節講到了 gradient descent 過去主要用在線性的模型中，但近年有一些發現使得梯度下降有更廣泛的運用

1. 在實務運用中發現損失函式的局部最小值並非梯度下降的主要問題，且在 Boltzmann machine 中看到了梯度下降運用在非線性模型的可能性 (當時尚未有理論解釋，目前就我所知也還沒有一個很完整的證明梯度下降的收斂性)
2. Hinton 等人發現本來用在控制理論中的 Back-Propagation 演算法結合梯度下降可以有效的訓練多層的神經網路
3. BP 應用於基於 sigmoidal unit 的多層神經網路在解決複雜的學習問題的成功

### D. Learning in Real Handwriting Recognition Systems

這節簡述了手寫辨識的問題，並解釋了手寫文件辨識中一個很重要的問題: Segmentation，除了正確的辨識每個字以外，我們還需要能夠正確的從圖片中辨識文字所在的範圍，並簡述了本文章所提出的一些方法，分別為在 Section V 會提到的，先辨識整個句子；以及 Section IV 中提到的，透過移動辨識窗格的方式來找尋字元。

### E. Globally Trainable Systems

這節為描述如何整合各個傳統的手寫辨識架構中的各個模組，使演算法可以對整體的誤差進行最佳化，因主要在於針對手寫辨識的結構設計，因此暫且先略過。

II. Convolutional Neural Network for Isolated Character Recognition
---
在過去的文件辨識技術中，通常先用人工定義的 Feature Extractor 抽出特徵，再將這些特徵送進一組分類器中，在此架構下，全連接的神經網路可做為分類器使用，但作者認為我們能夠進一步嘗試讓演算法自己去學習特徵萃取的方法，如此一來我們不必做太多的前處理，甚至可以將原始圖片直接輸入至模型中。

作者首先分析如果我們使用全連接的神經網路來做為特徵萃取器會產生的問題

1. 影像原始資料維度相當高，若以一層 100 個神經元的全連接層來說，可能光第一層就數萬個參數，會需要更大量的資料來訓練，計算機記憶體的需求也更大 (需要儲存上萬個權重)
2. 未結構化的網路架構也無法適應圖像平移、扭曲、縮放的狀況，若我們給予足夠多的神經元個數有可能可以訓練出可以適應影像變異的網路，但需要的計算資源及訓練樣本數就更龐大，也很可能訓練出模式相近但位於不同位置的權重，來滿足在不同的影像位置上擷取相似的 Pattern
3. 影像的局部相關性對於萃取局部的特徵有很大的幫助，並已有很廣泛的應用，但全連接的網路忽略了影像的局部關聯性，而 CNN 則利用這個特性讓特徵萃取更有效率

### A. Convolutional Networks

卷積層有三個基本的想法使得他能夠解決上述使用全連接層的問題 `local receptive fields` `shared weights` `spatial or temporal sub-sampling`。

卷積層的輸入為前一層的局部區域，這樣只觀測局部區域的概念很早就有，在生物學上也有相應的理論，透過局部感知能有效地抽出局部的特徵，例如角落、線段結尾、邊線等等，然後將這接局部特徵組合成更高維度的特徵，若影像有扭曲、平移等現象只是使得這些特徵改變位置而已，不會改變特性，我們將這樣的特性應用到神經網路上，即使用一組相同權重的神經元遍歷整張圖片的各個位置，而卷積層的輸出稱為 Feature Map，這樣的操作和數學中的「[卷積 (Convolution)](https://en.wikipedia.org/wiki/Convolution)」是相同的，因此稱為卷積網路。

一個完整的卷積層可能包含數組 Feature Maps，即我們現在常講的 Channel，下圖為 LeNet-5 的結構，第一層的卷積 (C1) 即包含 6 組 Feature Maps，而每個 Feature Map 所對應那一組相同權重的神經元就是現在的卷積核 (kernel)，這篇論文裡面稱為 Receptive Field，如第一層卷積為 5x5 區域一共 25 個權重的 Receptive Field，在 LeNet-5 中 stride 為 1，沒有 padding。 (注: 原論文中沒有提到 stride 及 padding 等字，但有針對這兩個參數相似的概念進行說明，這兩個概念應該是後來才被提出來的)

![](https://i.imgur.com/5YcH2WU.png)

這樣的架構有個有趣的特性是，若我們將圖像平移，則特徵會在 Feature Map 中平移相對應的距離，而特徵較不明顯的部分則沒什麼改變，這證實了這個方法對影像平移、變形的 Robustness。

當特徵被辨識出來後其所在絕對位置就不太重要了，更重要的是其相對位置，因此接在卷積層後面的是 sub-sampling (S2)，可在維持特徵相對位置資訊的情況下進行降維，即現今的 pooling 層，降維的方法與卷積相似，但是是先取區域中的數值平均後乘上一權重後加上 bias，並將結果通過 sigmoid function，在 LeNet-5 中使用 2x2 的 receptive field，stride 為 2，個別對每個 Feature Map 進行 sub-sampling，因此前一層有幾個 Feature Map，就有幾個 sub-sampling 後的影像。

文中有提到 sub-sampling layer 的權重會影響 sigmoid function 的非線性程度，若權重小，則 sigmoid 線性的性質較明顯，若權重大，則會表現得像是 "noisy OR" 或 "noisy AND"。(若權重造成 sigmoid 的輸入為正應是 noisy OR，輸入為負是 noisy AND)

Convolution/sub-sampling 的架構是啟發於先前提到的生物學研究，且過去早有實作，是為 Fukushima 桑所實作的 Neocognitron，但 LeCun 是率先將 BP 拿來訓練卷積網路的。

共享權重的設計大幅降低了網路的參數總量，使得網路有更好的收斂性，也降低對資料量的需求。

### B. LeNet-5
{% raw %}
這節詳細的解釋了 LeNet-5 的架構，可參考上面的架構圖，其中 CX 表示卷積層 SX 表示 Sub-sampling 層，卷積和 Sub-sampling 暫時就先不詳細解釋了，之後有空再補，但要注意的事情是 S2 到 C3 的連接方式和我們現在實作 CNN 的方式有點不太一樣，有興趣的人可以先讀論文，從第 7 頁右下方開始。

這邊從後面的全連接層開始講起，現在的 CNN 在卷積網路後多直接接多層的全連接層，並使用 softmax 輸出來進行分類，但在 LeNet-5 中的設計不是這樣，LeNet-5 為單層的全連接層，一共 84 個神經元，神經元個數是固定的，因為要配合輸出層的設計，激發函數是 scaled hyperbolic tangent:

$$
x_i=f(a_i)=A \cdot tanh(S \cdot a_i)
$$

輸出層採用的是 Euclidean Radial Basis Function，即計算與特定位置點的距離

$$
y_i = \sum_{j=0}^{83} (x_j-w_{ij})^2
$$

這邊 j 即為剛才全連接層的 index，而 $w_{ij}$ 為 RBF 的權重，這邊的權重非可訓練的參數，而是一組事先以人工挑選過固定的參數，以手寫數字辨識而言會有 10 組參數，因此可以計算出 10 組 RBF 值的輸出，挑選參數的方法有很多種，而 LeNet 的挑選的方法為將數字畫在一組 7x12 的 bitmap 上，展開後即為一組維度為 84 的向量，且這組向量的值為 [-1, 1]，與全連接的輸出值域相同，如此可避免 sigmoid 太容易達到飽和，飽和會讓模型收斂得更慢。(參考下圖)
{% endraw %}
![](https://i.imgur.com/Vv2Xalc.png)

論文中有提到他為什麼不用 one-hot 的原因是，當 class 數目增加時模型的表現會變差，但近年來的研究多是用 one-hot 來編碼，且有更好的表現，這部分值得深究看看原因為何。

### C. Loss Function
{% raw %}
Loss function 一般採用 Maximum Likelihood Estimation (MLE)，在本文中等同於 Minimum Mean Squared Error (MSE)，最簡單的方式如下

$$
E(W) = \frac{1}{P} \sum_{p=1}^P y_{D_p}(Z^p, W) = \frac{1}{P} \sum_{p=1}^P \sum_{j=0}^{83} (x_j-w_{{D_p}j})^2
$$

其中 $D_p$ 為該樣本對應的正確類別，這個損失函數有一些不足之處
{% endraw %}
1. 若我們使得 RBF 的權重可調整，則此損失函數可能會解出無用的解 (這部分我還沒看懂原因，等看懂在補上)
2. 忽略了不同類別之間的關係比較
{% raw %}
為解決上述問題，加上懲罰項修正後的損失函數如下

$$
E(W) = \frac{1}{P} \sum_{p=1}^P y_{D_p}(Z^p, W) + log(e^{-j} + \sum_i e^{-y_i(Z^p, W)})
$$

這裡的 $j$ 與剛剛上面的無關，$j$ 是一個正的常數，目的是避免讓懲罰項過大，使得模型不易收斂，而觀察主要的懲罰項 $\sum_i e^{-y_i(Z^p, W)}$ 可發現，當其他無關的 RBF 輸出越小，會造成懲罰項越大，使得模型能夠在最小化輸出與正確類別的誤差的同時考慮到其他類別的影響，此舉可同時解決上述原損失函數的兩個問題。
{% endraw %}
結語
---
有關 CNN、BP 以及 LeNet 的部分到此結束，後面是實際訓練手寫辨識的案例，與傳統架構比較，以及後續 GTN 的設計，等之後有時間再來看。

LeNet 為最早期的 CNN，可說是 CNN 的發明者，其網路架構有一些和最近大家所研究的 CNN 有些許不同 (除了後來提出的BN、Dropout等正則化技巧之外，還有很多細節是不一樣的)，或許需要再找時間研究看看後來的這些改變從何而來，為什麼改變。

補充
---
- 和論文無關，只是第一次研究論文的引用格式，之後可能會寫一篇文歸納論文引用的規則，工程類的文章比較常用 IEEE 的格式，以下是 IEEE 的 Citation Guideline: https://ieee-dataport.org/sites/default/files/analysis/27/IEEE%20Citation%20Guidelines.pdf
