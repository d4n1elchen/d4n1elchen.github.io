---
title: 又見課程爬蟲 (2) - JavaScript Promise & Generator
tags: [Web,Node,Backend,NCKU]
id: 716
categories: Web
date: 2017-01-21 19:55:10
---

今天居然用node把爬蟲寫出來了，覺得超不可思議的wwww

基本上爬蟲用同步的語言來寫會比較順暢一點，因為很常前後的步驟是有關連的，例如說要等到網頁整個載入完成才能開始爬資料，其實本來選js就是想試試看promise的用法，但是寫完之後還是奉勸大家，歹路不通走orz

這篇就來分享一下最後我使用的架構

promise如同前一篇簡介，可以將callback function從函式中抽離出來，避免層層堆疊的callback hell，promise寫出來的code大致上會長這樣(內容並非實際的code，只是示意而已)

```javascript
getDeptList().then((deptList) => {
  return storeToDb(deptList);
}).then((result) => {
  if (result=="success")
    console.log("success!")
});
```

邏輯是，取得系所列表之後在將列表存入db中，並檢查是否儲存成功

如果是同步語言，可能只要寫三行就好，因為下一行會等上一行做完才繼續做，資料的先後關係不會被打亂

```python
deptList = getDeptList()
result = storeToDb(deptList)
if (result=="success")
  print("success!")
```

這樣的寫法你用js跑跑看就會知道，getDeptList還沒執行完return，storeToDb就接著執行，也就讀不到deptList真正的值（一般來說會變成undefined），js會有這樣的設計據說是因為網頁比較重視載入的速度，如果說因為某行指令卡住的話會導致整個頁面卡在那邊，就像程式沒有回應那樣

所以過去的js會使用callback function來解決這個問題，但如果你的先後次序有很多層的話，就會變成callback層層堆疊，就是所謂的callback hell，有興趣可以去google看看這個詞

而promise讓callback可以不用寫在函式的參數中，而是丟給promise的一個method叫做then，當promise object執行完之後會去call then裡面的函式，然後這個函式可以回傳另外一個promise object，在繼續接then，如此一來callback都會在同一層，不會發生callback裡面還有callback的狀況

但就算如此then一多起來還是很可怕，在es7新增了async/await，寫起來感覺就像同步的程式一樣，簡潔有利，es6則有另外的解決方案: generator

在上一篇提到的(這篇)[http://huli.logdown.com/posts/292655-javascript-promise-generator-async-es6]有提到詳細的用法跟邏輯，可以去看看

generator是一種function，特色是可以執行到一半跳出去，然後再從跳出的點回來繼續執行，也可以中途把值進來使用

宣告generator function的方式如下

```javascript
function* run() {
  var a = 6;
  var b = yield a;
}
```

呼叫時大概是這種感覺

```javascript
var gen = run();
gen.next().value; // 6
```

只要注意一下執行順序應該很好理解，第一次呼叫該function會回傳一個generator物件，然後呼叫`next()`開始執行函式，跑到關鍵字`yield`的地方停下來，並把yield後方的值回傳出來，可以透過`.value`把值取出來，此時函式會卡在yield的地方不動，直到下次呼叫`next()`

除了把值從generator裡面丟出來之外，也可以把值從外面丟進去，呼叫`next()`時將要傳入的值餵給第一個參數`next(value)`，會回丟到上一次停下來的yield

```javascript
function* run() {
  var a = 6;
  var b = yield a; // b = 6
}

var gen = run();
var v = gen.next().value; // 6
gen.next(v);
```

透過generator不執行next()就不會繼續往下執行的特性可以寫出很類似同步的效果，詳細請參考上面提到的(這篇)[http://huli.logdown.com/posts/292655-javascript-promise-generator-async-es6]