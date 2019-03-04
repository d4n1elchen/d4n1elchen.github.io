---
title: Day 4 Vue物件
date: 2019-03-05 01:54:34
tags: [web, frontend, vue.js]
categories: Web
---

[原文章](https://ithelp.ithome.com.tw/articles/10185036)

vue的基本介紹與語法前三天已經介紹完了，基本過程是先載入vue.js，然後在html裡面寫templete，在js裡面宣告vue物件綁定至dom並寫入data，或是要使用自定義的dom元件的話要先在js裡面宣告元件。

第四天開始將深入了解前三天所提到的語法結構與各屬性的意義。

# Vue物件
本節的原文是Vue Instance，第一次見到Instance這個單字，拿去google翻譯是翻實例，不過詳細查了一下它有物件的意思，但又跟Object有些為差距，有興趣的人可以研究看看，在這邊先不討論。

Vue物件本身可視為[MVVM](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)架構中的ViewModel（雖然vue本身並非完全遵照MVVM架構設計），所以有時候會以`vm`作為變數名稱指向vue物件。

vue物件扮演著data(model)及dom(view)之間的溝通橋樑，建立雙向的綁定。

# 建構子
Vue物件的建構子大致長得像這樣
```javascript
var vm = new Vue({
  // options
})
```
只有一個傳入參數: Options object

options的屬性有: data, template, element to mount on(el), methods, lifecycle callbacks 及[其他](https://vuejs.org/v2/api/)

前幾次的範例有用到的有: el, data, methods, template(宣告componet時）

可以利用`.extent`方法來自定義建構子並加入一些預設的options
```javascript
var MyComponent = Vue.extend({
  // extension options
})
// all instances of `MyComponent` are created with
// the pre-defined extension options
var myComponentInstance = new MyComponent()
```
但vue建議以自定義元件的方式來做這件事，並在template裡面直接使用自定義元件，過幾天會來討論vue的component系統，到時候會有詳細的解說

# Vue物件的屬性與方法
vue物件會將data裡面的屬性proxy到vue物件本身
```javascript
var data = { a: 1 }
var vm = new Vue({
  data: data
})
vm.a === data.a // -> true
// setting the property also affects original data
vm.a = 2
data.a // -> 2
// ... and vice-versa
data.a = 3
vm.a // -> 3
```

> 註: javascript中===和==的差別在於三個等於會檢查兩者是否為同樣的類型，最常舉的例子就是，`1=='1'`: true，`1==='1'`: false

除了data之外，vue也expose了一些方便的屬性跟方法，為了和被proxy的data屬性區隔，這些其他的屬性跟方法前面會加上$來區隔

```javascript
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})
vm.$data === data // -> true
vm.$el === document.getElementById('example') // -> true
// $watch is an instance method
vm.$watch('a', function (newVal, oldVal) {
  // this callback will be called when `vm.a` changes
})
```

> Vue在這節特別提到不要使用[arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)作為vue物件方法的傳入參數或callback，詳細原因可以參考[原文](https://vuejs.org/v2/guide/instance.html)

# 生命週期
下圖展示了一個vue物件的生命週期
![](https://vuejs.org/images/lifecycle.png)
大致先看過就行了，不必全部都理解，要注意的地方只有用紅線拉出來的框框，這些表示各個生命週期的state，我們可以在option中為這些state加上callback
```javascript
var vm = new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` points to the vm instance
    console.log('a is: ' + this.a)
  }
})
// -> "a is: 1"
```

# 總結
今天簡介了vue物件、他的建構子、屬性方法，最後介紹了vue物件的生命週期

下週將介紹vue的模板系統
