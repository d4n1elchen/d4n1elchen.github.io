---
title: '[文章分享] 卷積神經網路 (Convolutional Neural Nets) 及物件偵測 (Object Detection) 方法演化史'
date: 2020-05-26 21:19:53
tags: [object detection, neural networks, machine learning]
categories: machine learning
---

分享 18 年的兩篇文章，有關 ConvNet 以及基於 ConvNets 的物件偵測方法的演化史，適合對 CNN 有一定程度了解的人作為深入學習 CNN 結構改進的方法與背後的數學思考的入門，以及有 CNN 基礎者的物件偵測方法入門，或是對這兩個領域都很熟悉，可以作為複習或參考筆記使用。

因為這兩篇是 18 年的文章，這兩年這兩個領域有更多後續的發展，但 18 年可說是 CNN 跟 ObjectDetection 的頂峰之年，許多關鍵的演算法都在 12~18 年間提出，若想了解整個領域的發展過程，節至 18 年的內容已經很全面，對最新的技術有興趣者可另外再找資料。

### [卷积神经网络结构演变（form Hubel and Wiesel to SENet）——学习总结，文末附参考论文](https://zhuanlan.zhihu.com/p/34621135)

這篇文章講解了 CNN 從最早啟發卷積神經網路研究，由 Hubel 和 Wiesel 等人發表建模貓視覺神經系統的文章開始，至 LeNet、AlexNet、VGG、Inception、ResNet、SENet，文末還有對輕量化網路如 MobileNet、ShuffleNet 的討論，可說是相當全面。文章中講解各個網路的發明的動機為何，解決了什麼問題，以及有哪些缺陷，對於理解 CNN 中各種工具的作用相當有幫助。

**[Table of Content]**
1. 早期的嘗試
    1. Hubel and Wiesel 的貓實驗 [[Hubel and Wiesel (1968)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1557912/pdf/jphysiol01104-0228.pdf)]
    2. NeoCognitron [[Fukushima (1980)](https://www.rctn.org/bruno/public/papers/Fukushima1980.pdf)]
    3. LeCun 的早期研究 [[LeCun (1989)](http://yann.lecun.com/exdb/publis/pdf/lecun-89e.pdf)]
    4. LeNet [[LeCun (1998)](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf)]
2. 歷史轉折
    1. AlexNet [Krizhevsky (2012)](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf)]
3. 網路加深
    1. VGG [[Simonyan (2014)](https://arxiv.org/pdf/1409.1556.pdf)]
    2. PReLU Net (MSRA-Net) [[He (2015)](https://arxiv.org/pdf/1502.01852.pdf)]
4. 卷積模組 (Conv Modules)
    1. NIN [[Lin (2013)](https://arxiv.org/pdf/1312.4400.pdf)]
    2. GoogleNet [[Szegedy (2014)](https://arxiv.org/pdf/1409.4842.pdf)]
    3. Inception V3 [[Szegedy (2015)](https://arxiv.org/pdf/1512.00567.pdf)]
    4. Inception V4 (Inception ResNet) [[Szegedy (2016)](https://arxiv.org/pdf/1602.07261.pdf)]
5. 集成線路 (應該是 Ensemble Connection)
    1. ResNet [[He (2015)](https://arxiv.org/pdf/1512.03385.pdf)]
    2. Identity Mappings in ResNet (改進 ResNet 的缺點) [[He (2016)](https://arxiv.org/pdf/1603.05027.pdf)]
    3. ResNeXt [[Xie (2016)](https://arxiv.org/pdf/1611.05431.pdf)]
    4. DenseNet [[Huang (2016)](https://arxiv.org/pdf/1608.06993.pdf)]
    5. Xception [[Chollet (2016)](https://arxiv.org/pdf/1610.02357.pdf)]
    6. SENet [[Hu (2017)](https://arxiv.org/pdf/1709.01507.pdf)]
6. 輕量化模型
    1. MobileNet V1 [[Howard (2017)](https://arxiv.org/pdf/1704.04861.pdf)]
    2. MobileNet V2 [[Sandler (2018)](https://arxiv.org/pdf/1801.04381.pdf)]
    3. SuffleNet [[Zhang (2017)](https://arxiv.org/pdf/1707.01083.pdf)]

### [關於影像辨識，所有你應該知道的深度學習模型](https://medium.com/cubo-ai/%E7%89%A9%E9%AB%94%E5%81%B5%E6%B8%AC-object-detection-740096ec4540)

這篇則是針對 NN-based 的物件偵測演算法的介紹，從 R-CNN、Fast R-CNN、Faster R-CNN、Mask R-CNN 至 YOLO，做詳細的演進過程介紹。

**[Table of Content]**
1. R-CNN [[Girshick (2014)](https://arxiv.org/abs/1311.2524)]
2. Fast R-CNN [[Girshick (2015)](https://arxiv.org/abs/1504.08083)]
3. Faster R-CNN [[Ren (2016)](https://arxiv.org/abs/1506.01497)]
4. Mask R-CNN [[He (2017)](https://arxiv.org/pdf/1703.06870.pdf)]
5. YOLO [[Redmon (2015)](https://arxiv.org/pdf/1506.02640.pdf)]
6. YOLOv2 [[Redmon (2016)](https://arxiv.org/pdf/1612.08242.pdf)]

另外補充一篇 19 年的文章 [A 2019 Guide to Object Detection](https://heartbeat.fritz.ai/a-2019-guide-to-object-detection-9509987954c3)，英文的，但內容大致上相同，除了下列幾個模型 

1. SSD [[Liu (2015)](https://arxiv.org/pdf/1512.02325)]
2. CenterNet [[Zhou (2019)](https://arxiv.org/pdf/1904.07850v2.pdf)]

此外下列論文是比較新的，也值得去看看

1. RetinaNet [[Lin (2017)](https://arxiv.org/pdf/1708.02002.pdf)]
2. FPN [[Lin 2017](https://arxiv.org/pdf/1612.03144.pdf)]
3. YOLOv3 [[Redmon (2018)](https://arxiv.org/pdf/1804.02767.pdf)]
4. Casecade R-CNN [[Cai (2019)](https://arxiv.org/pdf/1906.09756.pdf)]
5. ResNeSt [[Zhang (2020)](https://arxiv.org/pdf/2004.08955.pdf)]

Object Detection 的 Network Architecture Search (NAS)

1. NAS-FPN [[Ghaisi (2019)](https://arxiv.org/pdf/1904.07392.pdf)]
2. NAS-FCOS [[Wang (2019)](https://arxiv.org/pdf/1906.04423.pdf)]
3. DetNAS [[Chen (2019)](https://arxiv.org/pdf/1903.10979.pdf)]

### Other references
* [Object Detection for Dummies Part 1: Gradient Vector, HOG, and SS](https://lilianweng.github.io/lil-log/2017/10/29/object-recognition-for-dummies-part-1.html)
* [Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html)
* [Object Detection for Dummies Part 2: CNN, DPM and Overfeat](https://lilianweng.github.io/lil-log/2017/12/15/object-recognition-for-dummies-part-2.html)
* [Object Detection Part 4: Fast Detection Models](https://lilianweng.github.io/lil-log/2018/12/27/object-detection-part-4.html)
* [DeepLearning Tutorial](http://deeplearning.net/tutorial/contents.html)
* [Object Detection and Tracking in 2020](https://blog.netcetera.com/object-detection-and-tracking-in-2020-f10fb6ff9af3)
* [Object Detection | Paper with Code](https://paperswithcode.com/task/object-detection)