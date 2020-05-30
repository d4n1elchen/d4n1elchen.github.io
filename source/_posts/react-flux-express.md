---
title: React + Flux + Express
tags: [web,Express,Flux,Node,React,frontend,NCKU,NCKU Course Query]
id: 305
categories: web
date: 2016-09-13 01:07:42
---

嘛本來以為很簡單的東西被我搞得越來越複雜了阿wwwwwwwww

因為後來決定還是從後端爬資料下來寫新的網頁顯示，所以想說來用用看React

在查資料過程中看到這篇 [使用React、Node.js、MongoDB、Socket.IO开发一个角色投票应用](http://www.kancloud.cn/kancloud/create-voting-app)

<!--more-->

簡單略過前幾張覺得寫得蠻清楚的就照著做了，但沒想到React以前以為是可以簡化動態生成html的框架，但寫一寫發現還是頗複雜啊www

但我想只有React本身應該還算是簡單的，是加入React-route跟Flux之後便得複雜了

簡單做一些筆記

## ReactJS

ReactJS是14年發表的一套新框架，但相對於其他MVC框架而言，React專注在View上，將html物件模組化，使得動態生成新的前端物件更具有邏輯性

我大概抓幾個寫到目前為止已經理解的關鍵字，等之後比較有經驗了之後再來寫完整的筆記

React的元件稱為Component，每個Component都是一個Class(ES6才有Class的概念，ES5的話Class是用一個JS Object去實做，但其實概念一樣，文首的教學中有提到差異，可以去看看)

每個Component在生成之後有各自的生命週期，生命週期詳細可以參考[這篇](http://andyyou.logdown.com/posts/370308)，這個專案到目前為止用到了以下狀態(依照執行順序)

1.  初始化
* **componentDidMount()**
註冊該元件Store的callback(將onChange丟給store)
啟動初始化Action
* **render()**
顯示該元件的HTML內容

2.  state 發生改變時(由Store進行改變)
* **render()**
更新內容

3.  元件 unmount 卸載時
* **componentWillUnmount()**
註銷Store的callback
Action跟Store是Flux的東西，待會再解釋

此外component的初始化還有他的constructor，內容是將component的state和flux的state綁在一起，state可以理解成儲存在這個component裡面的data，例如說假設有一個使用者列表的元件，那麼state裡面可能就存有使用者的List，在render中存取state來取得內容，React在偵測到component的state改變時就會更新頁面，Flux架構中每個store也有自己的state，store透過state來修改component的內容，所以要將兩者綁在一起(還不懂Flux的概念可能看不太懂我在講啥，可以等了解之後再回來看這段)

constructor中還指定onChange函數(store用來更新state的callback，雖然沒看到有任何的文件提到Callback但這邊我以Callback理解好像解釋得通，求指正，詳細的作用會在Flux的地方解釋)

在Render部分React提供語法糖(Syntax sugar) - JSX，讓開發者可以用類似HTML的形式來寫React元件，語法糖的意思是在一些程式語言裡面，在特殊的狀況下會提供較簡單易懂或容易開發的語法讓開發者比較好寫出容易閱讀的程式，但該語法在編譯時仍會被轉譯成以基本語法編寫的片段。如下列兩段React的Code是等價的，JSX最後都會被轉換成JS的樣子。

JSX
```
render() {
  return (
    <ul>
      <li>Achura</li>
      <li>Civire</li>
      <li>Deteis</li>
    </ul>
  );
}
```
JS
```
render() {
  return React.createElement('ul', null,
    React.createElement('li', null, 'Achura'),
    React.createElement('li', null, 'Civire'),
    React.createElement('li', null, 'Deteis')
  );
}
```
目前還不知道React純前端怎麼讓元件顯示在頁面上

在這個範例中是使用React-route從後端來動態生成的

邏輯大概是這樣，Express收到http request之後丟給React-route處理生成主要的html，然後再丟給swig把這段由React-route生成的html塞進index.html裡面後丟回去給Express並傳送給前端

React-route還沒研究得很透徹，初步的理解是這樣，有錯歡迎指正

首先先由以下Code建立基本結構
```
<Route handler={App}>
  <Route path='/' handler={Home} />
</Route>
```
其中App和Home都是一個Component，也就是說每個Route都是由一個Component來handle

App內容如下(di=div，用div worpress會顯示不出來)
``` 
<di>
  <Navbar />
  <RouteHandler />
</di>
```
這邊是我不太確定的地方，我猜RouteHandler指的是包在App裡面的Route的Handler，也就是Home，確實最後顯示出來是div包裹著home的內容，但不太理解為什麼要這樣寫

Navbar則是選單列的Component，照這樣的邏輯來看這邊應該是可以直接塞Component進去，我想route的用意應該是不同的path可以用不同的Component來handle，然後Navbar是不管path是什麼都一律顯示

## Flux

這個之前一直看不懂，但寫一寫發現不難，不過腦筋還是要稍微轉一下才知道什麼東西要放哪

![flux-simple-f8-diagram-1300w](https://team6612.files.wordpress.com/2016/09/flux-simple-f8-diagram-1300w.png)

這是最新版的官方的圖解，簡單很多了，第一次看到之前的版本的時候複雜得要命，連讀都不想讀，就不貼出來了

簡單講就是你會在網頁的各個階段、各個地方觸發Action，Action完成一些動作之後會通知Dispatcher，Dispatcher偵測到之後會執行與Action綁定的Store，由Store來改變View

回到上面列出來的Component各階段做的事情

1.  初始化

    *   **componentDidMount()**
註冊該元件Store的callback(將onChange丟給store)
啟動初始化Action
    *   **render()**
顯示該元件的HTML內容

2.  state 發生改變時(由Store進行改變)

    *   **render()**
更新內容

3.  元件 unmount 卸載時

    *   **componentWillUnmount()**
註銷Store的callback
該元件初始化時會啟動一個Action，Action完成後會透過dispatcher觸發相對映的Store，Store會去改變Store中的state，這時會觸發onChange去更新Component中的state，Component的state發生改變時react會重新render，更新頁面

Action也可以從View中觸發，例如點擊事件，圖會變得像這樣

![flux-simple-f8-diagram-with-client-action-1300w](https://team6612.files.wordpress.com/2016/09/flux-simple-f8-diagram-with-client-action-1300w.png)

點擊事件處發後會啟動一個Action，Action完成後會透過dispatcher觸發相對映的Store(下略

Flux讓資料流變成單向的，不會像以前的MVC，若從View去處發Controller，Controller去改變其他的View又會觸發更多的Controller，當網頁規模一大起來會變得錯綜複雜難以管理，有時候連為什麼被觸發都不知道(我是沒遇過這樣的狀況啦...因為沒寫過大網站)

## Express

Express是node的一套web應用程式架構，他在這個專案裡的角色是負責處理http request，辨認路徑並分配給不同的react-route處理，並且將處理完的網頁再吐回去給瀏覽器，Express很簡單，就一些function call而已，就不多談了，有興趣可以自己去找範例程式來看

以上，在寫筆記的過程同時也釐清了不少觀念，希望我不會寫得太難懂

喔對另外那個教學還介紹了一些開發工具gulp browserify bower nodemon

目前只知道nodemon可以偵測檔案變更並自動重啟程式，相當方便的工具，其他的之後再研究吧