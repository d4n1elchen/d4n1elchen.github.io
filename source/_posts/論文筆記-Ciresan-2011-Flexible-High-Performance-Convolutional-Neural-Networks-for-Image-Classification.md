---
title: >-
  論文筆記 - Ciresan 2011 - Flexible, High Performance Convolutional Neural Networks
  for Image Classification
date: 2018-12-06 13:39:54
tags: [statistics,math,machine learning,data analysis,deep learning,neural network]
categories: Machine Learning
thumbnail: https://i.imgur.com/IYz0baq.png
---

Dan C. Cireşan et al., "Flexible, High Performance Convolutional Neural Networks for Image Classification," In Proc. International Joint Conference on Artificial Intelligence '02, 2011, pp. 1237-1242

URL: https://dl.acm.org/citation.cfm?id=2283603

Full Text: http://www.idsia.ch/~juergen/ijcai2011.pdf

這篇主要貢獻是在 AlexNet, VGG 前第一篇使用 GPU 來加速 CNN 運算的研究，並且因為加速了訓練，做了很多不同結構在一些經典資料集上的表現，結論是網路加深加寬對精度都有很好的改善。

對於沒有相關程式背景的人來說其實不用讀 GPU 的部分，可以直接看實驗結果，現在的深度學習計算框架 (如 PyTorch, TensorFlow) 都幫你把這些優化做好了，只要幾行程式碼就可以啟動 GPU 加速運算。

因為在做演算法優化，這篇和 CNN FP 和 BP 的演算法實作有很大的關係，如果想讀懂這篇可能要有一些實作這些演算法的基礎，尤其是 BP。此外牽涉 CUDA 的實作，所以可能還需要有一些 CUDA 的基礎觀念。BP 部分可以參考這份[投影片](https://dlsys.cs.washington.edu/pdf/lecture4.pdf)，CUDA 補充在這篇筆記的下面。

1 Introduction
---
比較傳統影像辨識和CNN，並提到GPU加速運算可以加速對CNN的研究，因為可以快速嘗試不同的網路結構對訓練成果的影響。

2 CNN
---
介紹 CNN，包含 Pre-processing, Conv layer, Max-pooling layer, classification layer (FC layer)。

3 GPU implementation
---
本篇的重點，解釋如何最佳化 GPU 加速 CNN 運算，主要用了下面三個技巧，可能需要一些 CUDA 和計算機組織基礎才看得懂

1. pre-computed expressions
2. unrolled loops within template kernels
3. strided matrices to obtain coalesced mem access

大多是在講如何有效運用 GPU 的運算資源。部分和記憶體、快取操作有關，簡單講是說，記憶體越大存取速度越慢，所以 CPU 或 GPU 核心會有階層式的快取設計，將常用的資料存在空間較小但速度較快的儲存區，以減少資料傳輸帶來的延遲。

下圖為常見的 CPU Cache 結構

![](https://i.imgur.com/FNpOO4p.png)

GPU Cache 和 CPU 概念基本上一樣，只是 GPU 運算核心比 CPU 多上很多倍，所以有大量的 L1 Cache

![](https://i.imgur.com/zHLFviQ.png)

如果所取用的資料經常可以在 Cache 就拿到不用到 Memory 去找的話，可以大幅降低 latency。例如一個核心負責矩陣中一個 row 的加法運算，如果核心的 L1 Cache 剛好可以存一個 row 的資料，那整個過程就只需要向 Memory 請求一次資料。

### 3.1 Data structures

1. {% raw %}$y$ and $\delta$ are 2D strided with a pitch of 32 floats{% endraw %}
2. {% raw %}Forward connection matrix: $C^n${% endraw %}
3. {% raw %}Backward connection matrix: $C_{BP}${% endraw %}
4. {% raw %}$WIDX_{BP}${% endraw %}
    - (還沒讀懂，應和 Chain rule 有關)

2, 3 都是方便運算時可以快速知道連接關係用的。

### 3.2 Forward propagation
平行化FP運算最值觀的方式就是每個 feature map 分配一個 block 來計算，每個 thread 負責一個神經元的計算。但 CUDA 每個 block 上限的 thread 數是 1024，如果 feature map 的神經元個數超過 1024 則需要再進行分組，如分配一個 block 來計算每個 feature map 的某一行算。

![](https://i.imgur.com/IYz0baq.png)

### 3.3 Backward propagation
#### Push vs Pull (論文是用 Pull)
> 註: 這段還沒有讀得很清楚，如果有發現錯誤還麻煩指正

計算梯度 (論文中指的 delta) 的方法有兩種：Push 和 Pull
- Push 指當你要計算某一層的梯度時從後一層取得相關的梯度來計算
- Pull 指反過來先將當前層算好的梯度往前主動推送給需要這層梯度的神經元

因為當前層可能會影響前一層很多個神經元，如果用 Push 會造成很多次重複的記憶體存取，論文採用的是 Pull，論文中的數學式主要在計算會影響的前後層神經元之間 index 的關係，也就是在計算神經元的連接關係。

### 3.4 Adjusting weights
{% raw %}
1D grid 對應到層跟層之間的 filter，2D block 對應到每個 filter 的權重，但每個 filter 還需要額外多一次操作更新 bias，所以每個 block 一共有 $(K_x + 1) \times K_y$ 個 thread。通常多出來的那 $K_y$ 個 thread 會需要等待。
{% endraw %}

4 Experiments
---
這篇使用 Core i7-920, 12 GB DDR3, 2 x GTX 480, 2 x GTX 580 來進行實驗，並使用 Test for best Validation (TfbV) 進行模型的準度評估，意為在不同次訓練中取最佳 Validation 的模型進行 Testing。

### 4.1 MNIST
![](https://i.imgur.com/fBdGI7t.png)

在 MNIST 上主要測試網路加深的影響，M 代表該層的 feature map 個數，N 代表 FC 層的神經元個數，可以很明顯看到隨著網路加深精度的增加。

雖然加深網路會大幅增加參數個數，但收斂的速度也增加了，在 7 層的架構中，在第 1, 2, 7 個 epochs 就分別有 2.42% 0.97% 0.48% 的表現；相較之下 4 層的結構為 4.71% 1.58% 0.68%。

### 4.2 [NORB](https://cs.nyu.edu/~ylclab/data/norb-v1.0/)
這是一個視差立體圖的資料集，由 Yann LeCun 在 NYU 時的團隊整理發表。

因為是黑白的立體視差圖像所以輸入有 2 個 chennel。在這個資料集上主要測試前處理 (Translation & Contrast Extraction) 的影響，網路架構是固定的 5 層結構: 300M-MaxPooling-500M-MaxPooling-500N。

![](https://i.imgur.com/eiifq2l.png)

結果顯示前處理 (或為後來的 Data Augmentation 的概念) 對精度提升有幫助。

但論文中也提到，因為 NORB 每個類別的物體各數過少，變異不夠，造成訓練時很快就收斂但一般化的程度不夠，使得測試的準確度無法提升，也因此 Data Augmentation 才能夠有這麼明顯的提升。在 MNIST 及 CIFAR 10 中，因為訓練集的變異夠大，Data Augmentation 的幫助就不那麼明顯了。

### 4.3 CIFAR 10
CIFAR 10 中影像裡的物體沒有置中，甚至有的只有部分出現在圖像中，背景也是五花八門，再加上他是彩色圖片，是為三個資料集中最困難的一個。

CIFAR 10 的實驗中採用的是 8 層的架構: ?M-MaxP-?M-MaxP-?M-MaxP-300N-100N，其中每個捲積層的 filter 個數是變數，因此這個實驗還比較了網路的寬度帶來的影響。

![](https://i.imgur.com/DCo79Go.png)

實驗結果可以首先看到的是 Translation 帶來了大幅度的準確率提升，從本來的 28% 左右提升到了 20%，而 Contract Extraction 的影響卻是負面的；而寬度從 100 提高到 300 有小幅度的改善，但到了 400 卻略微下降，寬度對精確度的影響並不明顯。

Notes: CUDA 基本概念
---
### 異質多核心運算
對於複雜的運算來說 CPU 有較大的優勢，但 CPU 不擅長多線程平行處裡；而 GPU 則反過來適合非常大量但簡單的同步運算，例如大矩陣運算。異質運算就是同時利用 CPU 和 GPU 的優勢來加速整體的運算速度，CUDA 就是一套由 nVidia 開發的異質運算框架函式庫，只能用在 nVidia 的 GPU 上。

![](https://i.imgur.com/SQ8I9HC.png)

### host & device
host 指 CPU 而 device 指 GPU 上的多個核心

### 運算流程
> 1. 分配host内存，并进行数据初始化；
> 2. 分配device内存，并从host将数据拷贝到device上；
> 3. 调用CUDA的核函数在device上完成指定的运算；
> 4. 将device上的运算结果拷贝到host上；
> 5. 释放device和host上分配的内存。

### kernel function
Kernel function 為使用者定義要在 GPU 上進行運算的 function，用 `__global__` 關鍵字來定義，並在呼叫時以 `<<<grid, block>>>` 來指定同時要啟用的線程數。

例:
```C
// Kernel definition
__global__ void VecAdd(float* A, float* B, float* C)
{
    int i = threadIdx.x;
    C[i] = A[i] + B[i];
}

int main()
{
    ...
    // Kernel invocation with N threads
    VecAdd<<<1, N>>>(A, B, C);
    ...
}
```

### grid, block, thread 之間的關係
這是 CUDA 中很核心的觀念，可以想像把矩陣運算切細再切細，然後用很多個 thread 同時撒下去除理這些小塊的運算，可以參考《[【CUDA】grid、block、thread的关系及thread索引的计算](https://blog.csdn.net/hujingshuang/article/details/53097222)》中提供的範例。

Ref
---
- [CUDA 入門](https://zhuanlan.zhihu.com/p/34587739)