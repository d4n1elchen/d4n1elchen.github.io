---
title: 又見課程爬蟲 (4) – mongo-connector
tags: [NCKU Course Query,Web,Node,Backend,NCKU]
id: 880
categories: Web
date: 2017-02-07 22:33:57
---

最近在調整作息，但還是有時候不小心寫一個小功能就弄到很晚，這裡的紀錄進度落後太多。

先紀錄一些零碎的事項，然後列一些需要詳細研究的東西在這篇。

上一篇之後增加的進度大致上有以下
- 重構react的code（更了解react的運作機制以及alt的使用方法）
- 增加時間篩選（重新設計了課程時間的儲存格式以利比較）
- 增加搜尋功能（用mongo-connector整合elasticsearch作為搜尋引擎）
- 重構爬蟲的code

這次在整合搜尋功能的時候，雖然之前在測試elasticsearch的時候已經設定過了
但重開機之後好像就跑掉了不知道為啥，於是我又重新設定了一遍
只是很多東西都忘記了又重投查一遍，這邊紀錄一下

elasticsearch是什麼有興趣的自己google看看
因為mongo本身的搜尋功能較弱，所以我用es來作為搜尋引擎
用mongo-connector為中介程式幫忙同步mongo跟es
es本身好像就已經可以作為資料庫使用了
但我對他的了解還不太夠，暫時還是先用mongo+es的方式

# Installation

首先必須安裝mongo跟es，這邊就不詳細說明
按照官方的安裝指示安裝即可
不過這裡我還有一個問題沒解決
mongo跟es的開機啟動設定不好，等之後再來研究

## mongo-connector

參考[官方文件](https://github.com/mongodb-labs/mongo-connector/wiki/Installation)
我選擇透過github安裝

```bash
cd your/installation/directory
git clone https://github.com/mongodb-labs/mongo-connector.git
cd mongo-connector
python setup.py install
```

# Configurations

參考[Getting Started](https://github.com/mongodb-labs/mongo-connector/wiki/Getting-Started)

## mongodb

要先設定replica set
官方的教學[在這](https://docs.mongodb.com/manual/tutorial/deploy-replicaset/)，還沒有詳細讀過
我用mongo-connector給的指令去下會失敗，我自己各方查找勉強設定到可以用
但我不知道這個步驟詳細在設定什麼，推測replica應該是建立分散式的節點
mongo-connector貌是透過replica的推播事件來同步資料，所以資料有更新才會進行同步
設定方法如下

首先打開`/etc/mongod.conf`，增加以下設定

```text
replication:
  replSetName: myDevReplSet
```

重啟mongod並進入mongo shell

```bash
$ sudo service mongod restart
$ mongo
```

輸入以下指令

```javascript
rs.initiate({
    _id : "myDevReplSet",
     members : [
         {_id : 0, host : "localhost:27017"},
     ]
})
```

完成

es跟mongo-connector都不用特別設定

#Start
接著就可以執行mongo-connector

```bash
mongo-connector -m localhost:27017 -t localhost:9200 -d elastic2_doc_manage
```

如果你的mongo有該auth的話

```bash
mongo-connector -m localhost:27017 -t localhost:9200 -d elastic2_doc_manager --admin-username <username> --password <password>;
```

注意你登入的帳號要有repl的管理權限才行，repl相關的role可以參考[這裡](https://docs.mongodb.com/manual/reference/built-in-roles/#cluster-administration-roles)，我是直接設定成clusterAdmin

啟動後就可以在mongo那邊insert資料進去測試看看會不會同步到es裡面了

以上

最後整理一下之後必須詳細研究的東西
- Mongo replication
- Elasticsearch query
- Mongo and ES 開機啟動

下一篇沒意外是寫mongo的一些設定，authentication還有remote connection等等