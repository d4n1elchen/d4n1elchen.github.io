---
title: TensorFlow 學習筆記 (1) - feature_column
date: 2018-03-21 16:42:17
tags: [TensorFlow, machine learning]
categories: machine learning
---

前言
---
TensorFlow 應該大家都知道是啥就不詳細介紹了，從這篇開始往下我會整理自己讀官方的 documentation 的一些筆記，並以中文撰寫，希望可以幫助到有需要的人。

相信很多人初學 TensorFlow 的時候最困惑的部分之一是資料的輸入，TensorFlow 因為是通用化設計，為了滿足不同的資料類別需求，預設了幾種不同的資料輸入的方式，但我個人認為 TensorFlow 的框架感太重了，你一定要先讀懂框架的運作流程才有辦法開始寫程式，keras 相對來說流程就比較直觀一點，至於這是好是壞每個人的觀點可能不太一樣，我是認為以我目前的經驗來說，學習曲線是屬於懸崖型的工具當你爬上去之後工作效率會大幅提升，而緩坡型的可以讓你快速開始工作並從中獲得成就感。

TensorFlow 的資料輸入目前我整理下來大概有以下幾個關鍵字需要理解

* feature_column
* dataset
* input_fn

本篇從 [feature_column](https://www.tensorflow.org/versions/master/get_started/feature_columns) 開始介紹起


feature_column
---

類神經網路可視為一種 function emulator，建立輸入和輸出資料之間的對應關係。基本上類神經網路處理的是數值資料，但現實生活中的問題經常不是數值，例如類別類型的資料很可能是文字標籤，或自然語言處理會需要處理文字資料，我們必須先將這些資料轉換成數值資料才有辦法利用類神經網路進行建模。

舉例來說，我們有一組輸入資料是動漫人物的名字，如下表

1. 高木
1. 惠惠
1. 雛鶴愛

最簡單的方式就是對各類別進行編號，如上列表順序，若資料為「高木」則輸入`1`，結果為「惠惠」則輸入`2`。

另一種方式是以一組長度為類別個數的陣列，並以 0 和 1 來標示這組資料是哪個人物，例如「惠惠」會被轉換成 `[0, 1, 0]`，表示這組資料是屬於第二個分類(惠惠)，這種表示法又稱為 `one-hot vector`。

TensorFlow 的 `feature_column` 可以自動幫你做這樣的處理，且如果你的資料中包含不同種類的資料，例如輸入同時有標籤及數值，這些資料需要的前處理不一定相同，`feature_column` 會告訴 [`estimator`](https://www.tensorflow.org/versions/master/programmers_guide/estimators) 輸入的資料類型，並做出相對應的處理。

如下圖所示，左邊藍色可先想成一組輸入的資料結構，共有四個欄位，於是我們在 `feature_columns` 中定義這四個欄位分別的資料類型，並將類別定義餵給 `estimator`。

![](https://www.tensorflow.org/versions/master/images/feature_columns/inputs_to_model_bridge.jpg)

`feature_column` 有兩種 -- Categorical Column & Dense Column，文末會詳細討論這兩種 column 的差異及適用範圍，這邊可以先想成，Categorical 只給不同的類別標上數值標籤，Dense 則是轉換成矩陣輸入，全部共有 9 個 `feature_column`，其中 `bucketized_column` 較為特別，同時可以是 Categorical 和 Dense Column，後面會詳細介紹。

![](https://www.tensorflow.org/versions/master/images/feature_columns/some_constructors.jpg)

> **Tips**
> 其實如果輸入資料都是數值的話不太需要理解不同的 feature_column 用途是啥，一律使用 numeric_column 就好了，但 TensorFlow Estimator 只吃定義好 feature_column 的欄位，所以就算全部的欄位類型都一樣還是要寫，這也是為什麼我說 TensorFlow 框架感很重的原因。

在 TensorFlow 裡面將資料餵給神經網路的流程是

1. 撰寫 input_fn 將資料從外部載入並轉換成 TensorFlow 可以吃的格式
2. 定義 feature_column，指定 input_fn 回傳的資料每個資料欄位的類別
3. 將 feature_conlumn 餵給 Estimator 進行初始化
4. 將 input_fn 餵給 Estimator.train 或 Estimator.evaluate 進行運算

以下所有範例程式碼都假設 input_fn 所回傳的值有下列資料結構

```python
{
    "Length": [5.0, 6.0, ...],
    "Position": [[5.0, 5.0], 
                 [6.0, 6.0], ...],
    "SomeMatrix": [[[5.0, 5.0, 5.0], [5.0, 5.0, 5.0]], 
                   [[6.0, 6.0, 6.0], [6.0, 6.0, 6.0]], ...],
    "Character": ["Takagi", "Megumin", ...],
    "CharacterID": [0, 1, ...]
}
```

可解讀為，我的輸入資料有 5 個欄位，後面的陣列為 1~n 筆資料，按照順序一一對應，例如以上面的資料而言，我的第一組輸入就是

```python
{
    "Length": 5.0,
    "Position": [5.0, 5.0],
    "SomeMatrix": [[5.0, 5.0, 5.0], [5.0, 5.0, 5.0]],
    "Character": "Takagi",
    "CharacterID": 0
}
```

Dense Column
---
### numerical_column

數值資料，最基本不需要進行轉換的資料，預設的 `dtype` 為 `tf.float32`，呼叫時可指定 `dtype`。

```python
# Defaults to a tf.float32 scalar.
numeric_feature_column = tf.feature_column.numeric_column(key="Length")

# Represent a tf.float64 scalar.
numeric_feature_column = tf.feature_column.numeric_column(key="Length",
                                                          dtype=tf.float64)
```

可為純量或是陣列輸入，在呼叫時可以指定陣列的 `shape`，若未指定則預設為純量(1維且長度維1的矩陣)。

```python
# Represent a 10-element vector in which each cell contains a tf.float32.
vector_feature_column = tf.feature_column.numeric_column(key="Position",
                                                         shape=2)
# Represent a 10x5 matrix in which each cell contains a tf.float32.
matrix_feature_column = tf.feature_column.numeric_column(key="SomeMatrix",
                                                         shape=[2,3])
```

### indicator_column & embedding_column

這兩個函式的輸入是 Categorical Column，用途是將 Categorical Column 轉換成 Dense Column。

`indicator_column` 可將 Categorical Column 轉換成 one-hot vector。

```python
categorical_column = tf.feature_column.categorical_column_with_vocabulary_list(
    key="Character",
    vocabulary_list=["Takagi", "Megumin", "Hinatsuru Ai"])

# Represent the categorical column as an indicator column.
indicator_column = tf.feature_column.indicator_column(categorical_column)
```

`embedding_column` 則直接將每個標籤映射到 n 維的向量空間，這是一種 deep leaning 的技巧叫 embedding，自然語言處理好像很常用這樣的映射，這主要在處理標籤數量很多的狀況，例如我們如果將每個英文單字視為一個標籤的話，那全部總共會有上萬個標籤，若以 one-hot vector 表示會變成一組上萬維的向量，若有興趣可先參考 TensorFlow 官方提供的 [Embeddings](https://www.tensorflow.org/versions/master/programmers_guide/embedding) 文件。

```python
categorical_column = tf.feature_column.categorical_column_with_vocabulary_list(
    key="Character",
    vocabulary_list=["Takagi", "Megumin", "Hinatsuru Ai"])

# Represent the categorical column as an embedding column.
# This means creating a one-hot vector with one element for each category.
embedding_column = tf.feature_column.embedding_column(
    categorical_column=categorical_column,
    dimension=2) # 將角色名稱應射成 2 維向量
```

Bucketized Column
---
### bucketized_column

字面上意思就是將資料分成好幾個籃子，我們可以透過 `bucketized_column` 將連續的數值資料，透過設定區間的方式將資料群組起來，並標上標籤。

例如說我們有年份的資料，我們可將時間軸切成四份

![](https://www.tensorflow.org/versions/master/images/feature_columns/bucketized_column.jpg)

如此可將不同的年份分裝成四份，下表用 one-hot vector 表示

| Date Range | Represented as... |
| :-- | :-- |
| < 1960 | \[1, 0, 0, 0\] |
| >= 1960 but < 1980 | \[0, 1, 0, 0\] |
| >= 1980 but < 2000 | \[0, 0, 1, 0\] |
| \> 2000 | \[0, 0, 0, 1\] |

```python
# First, convert the raw input to a numeric column.
numeric_feature_column = tf.feature_column.numeric_column("Year")

# Then, bucketize the numeric column on the years 1960, 1980, and 2000.
bucketized_feature_column = tf.feature_column.bucketized_column(
    source_column = numeric_feature_column,
    boundaries = [1960, 1980, 2000])
```

要注意的地方有兩個，首先你要先建立 `numeric_column`，再把建好的 `numeric_column` 餵給 `bucketized_column`，另外 `bucketized_column` 是利用設定切分點的方式來設定區間，也就是說，3 個切分點會有 4 個區間。

Categorical Column
---

### Categorical vocabulary column

這個 column 會將輸入的文字標籤轉換成對應的編號。有兩種設定標籤列表的方式

-   [`tf.feature_column.categorical_column_with_vocabulary_list`](https://www.tensorflow.org/versions/master/api_docs/python/tf/feature_column/categorical_column_with_vocabulary_list)

```python
# Given input "feature_name_from_input_fn" which is a string,
# create a categorical feature by mapping the input to one of
# the elements in the vocabulary list.
vocabulary_feature_column =    tf.feature_column.categorical_column_with_vocabulary_list(
    key="Character",
    vocabulary_list=["Takagi", "Megumin", "Hinatsuru Ai"])
```

-   [`tf.feature_column.categorical_column_with_vocabulary_file`](https://www.tensorflow.org/versions/master/api_docs/python/tf/feature_column/categorical_column_with_vocabulary_file)

```python
# Given input "feature_name_from_input_fn" which is a string,
# create a categorical feature to our model by mapping the input to one of
# the elements in the vocabulary file
vocabulary_feature_column =    tf.feature_column.categorical_column_with_vocabulary_file(
    key="Character",
    vocabulary_file="class.txt",
    vocabulary_size=3)
```

In `class.txt`

```
Takagi
Megumin
Hinatsuru Ai
```

### Categorical identity column

像是 ID、編號，這種類型的資料，雖然是數值，但性質上比較像標籤，若要將編號資料轉換成標籤就需要用到這個 column，或是你已經事先將文字標籤轉換成數字了，你最後輸入的資料會是編號而不是文字標籤，也可用這個函式來建立 feature_column。

```python
# Create categorical output for an integer feature named "my_feature_b",
# The values of my_feature_b must be >= 0 and < num_buckets
identity_feature_column = tf.feature_column.categorical_column_with_identity(
    key="CharacterID",
    num_buckets=3) # Values [0, 3)
```

### Hashed Column

如果你的標籤有上千或上萬個，給每一個標籤一個編號並不是一個有效率的方法，除了前面有提到的 embedding 之外，還有另一個做法是將標籤轉換成 hash table 的 index，也就是計算該標籤的 hash 值之後對 hash_buckets_size 取餘數作為這個標籤的 ID。

```python
feature_id = hash(raw_feature) % hash_buckets_size
```

使用 hash 就會有碰撞的問題，但就經驗而言少量的碰撞不會太顯著的影響模型的效能，因為模型還是可以從其他的 feature 來分辨不同的標籤間的差異。

```python
hashed_feature_column =
    tf.feature_column.categorical_column_with_hash_bucket(
        key = "Character",
        hash_buckets_size = 100) # The number of categories
```

### Crossed Column

這是用來將不同的 feature_column 合成成單一個 feature_column 的，官方舉的例子是將經度和緯度資料合成成一組經緯度的座標資料，並且也可以指定 hash buckets size 將合成的結果分裝，詳細可參閱 [Crossed Column](https://www.tensorflow.org/versions/master/get_started/feature_columns#crossed_column)。

Categorical Column vs Dense Column
---

關於這兩個類別官方的教學文件中其實沒有太多解釋，上網查了一些資料目前整理如下，但我還是沒有很清楚兩者之間明確的差異，還請各位大大們補充跟指教。

* Categorical 會用 `tf.SparseTensor` 來儲存資料，而 Dense 是 `tf.Tensor` (DenseTensor)，有關稀疏矩陣的資料結構可以自己去 Google 一下。
* 標籤資料 Categorical 是儲存各種標籤的編號，而 Dense 則是 one-hot vector。
* 這點比較重要，內建的幾個 Estimator 可輸入的 feature_column 類別不同
    * [`LinearClassifier`](https://www.tensorflow.org/versions/master/api_docs/python/tf/estimator/LinearClassifier) and [`LinearRegressor`](https://www.tensorflow.org/versions/master/api_docs/python/tf/estimator/LinearRegressor) 
        * 兩種都可以
    * [`DNNClassifier`](https://www.tensorflow.org/versions/master/api_docs/python/tf/estimator/DNNClassifier) and [`DNNRegressor`](https://www.tensorflow.org/versions/master/api_docs/python/tf/estimator/DNNRegressor) 
        * 只吃 Dense，所以 Categorical 要透過 indicator_column 或 embedding_column 轉換成 Dense
    * [`DNNLinearCombinedClassifier`](https://www.tensorflow.org/versions/master/api_docs/python/tf/estimator/DNNLinearCombinedClassifier) and [`DNNLinearCombinedRegressor`](https://www.tensorflow.org/versions/master/api_docs/python/tf/estimator/DNNLinearCombinedRegressor)
        * `linear_feature_columns` 兩種都吃
        * `dnn_feature_columns` 只吃 dense

至於為什麼要分這兩個類別，目前我還沒有找到一個明確的解釋，還請各位大大補充。

後記
---

以上是 feature_column 的解釋，之前在嘗試都是使用數值資料在訓練，網路上的範例也大都使用數值資料，最近想研究看看能不能將不同種類的輸入資料前處理模組化，例如週期性資料可以先做頻域特徵轉換，時間序列可以讓他先通過 Convolution Autoencoder 等等，看到 feature_column 有這樣的感覺於是花了一些時間把之前跳過的部分重讀了，但最後發現和想像中的好像不太一樣，feature_column 主要在處理標籤資料的解讀，最後都還是回歸到數值資料。

下一篇應該會是 dataset 的探討。