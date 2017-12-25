---
title: Firebase 不支援多欄位查詢
date: 2017-12-25 19:50:02
tags: [firebase, web, backend, IoT]
categories: Web
---

這兩天本來打算重新設計一下之前寫的感測資料監控的專案，這個專案簡單來說是蒐集感測器的資料並上傳到資料庫，並在網頁上進行視覺化。

因為之前寫的幾個小專案用Firebase的體驗還不錯，API很方便呼叫，Realtime Database的功能也很棒，不用自己實作Websocket就可以達到Realtime的資料視覺化。

雖然Firebase在小型專案或Prototype開發很方便，但他查詢的API實在是不太實用。

我們在對資料庫進行查詢的時候常會進行條件的設定，以限縮查找範圍，提高查詢效率。

以我限在的這個專案來說，我目前每一筆樣本的資料結構設計如下

```javascript
{
    device: 裝置名稱,
    location: 裝置位置,
    time: Timestamp,
    values: {
        value1: 欄位1,
        value2: 欄位2,
          :
          .
        valuen: 欄位n
    }
}
```

當我有數個裝置放在不同位置時，我想用location來查找不同地點搜集到的資料，Firebase對單一欄位的查尋方法如下(省略前面setup)

```javascript
var ref = firebase.database().ref("/data");
ref.orderByChild("location").equalTo("裝置位置").once("value").then(...);
```

除此之外我還需要透過時間欄位來限縮資料範圍，若單獨查詢區間的話方法如下

```javascript
ref.orderByChild("time").startAt(timestamp_start).endAt(timestamp_end).once("value").then(...);
```

但如果我將兩個欄位的查詢合並，會噴Error，原因是不能同時有兩個orderBy

```javascript
ref.orderByChild("location").equalTo("裝置位置")
   .orderByChild("time").startAt(timestamp_start).endAt(timestamp_end).once("value").then(...);

// Error: Query.orderByChild: You can't combine multiple orderBy calls.
```

Stackoverflow上有一篇有討論到這個[問題](https://stackoverflow.com/questions/41041385/android-firebase-apply-multiple-queries)，有個建議的做法是，將兩個欄位預先合併成一個用來查詢的欄位

```javascript
{
    device: 裝置名稱,
    location: 裝置位置,
    time: Timestamp,
    location_time: 裝置位置_Timestamp
    values: {
        value1: 欄位1,
        value2: 欄位2,
          :
          .
        valuen: 欄位n
    }
}
```

```javascript
ref.orderByChild("location_time").startAt("裝置位置_" + timestamp_start).endAt("裝置位置_" + timestamp_end).once("value").then(...);
```

但這實在不太直觀而且若有更多欄位要查詢會變得很難處理。

不過Firebase最近有推出一個新的資料儲存服務[Cloud Firestore](https://firebase.google.com/docs/firestore/)，有提供類似SQL的where條件查詢，但仍在Beta release，接下來可能會先嘗試用Firestore，未來的話想把資料庫的部分抽象化，用XML等資料描述語言定義資料庫結構，再根據使用者需求依定義好的結構，在不同的資料庫上建立實例，以簡化資料庫設定的流程。