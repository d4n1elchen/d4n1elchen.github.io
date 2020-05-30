---
title: Day 3 Vue使用者輸入、自定義元件
date: 2019-03-05 01:39:39
tags: [web, frontend, vue.js]
categories: web
---

[原文章](https://ithelp.ithome.com.tw/articles/10184752)

昨天講了基本的邏輯方法—條件與迴圈，今天要來處理使用者輸入，動態網頁一定會碰到要和使用者互動的情境，如點擊、輸入資料等，此外今天也會講到vue怎麼自定義dom元件

# 事件綁定 v-on
最常見的事件綁定為點擊事件，即onclick事件，在vue裡面的寫法如下
```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>
```
```javascript
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```
如果執行成功的話將會產生一行字跟一個按鈕，當按下按鈕時第一行字會反轉。

這個範例同時展示了事件綁定以及更新內容的方法，在vue裡面更新內容不必直接存取dom，vue會自動偵測data的改變並更新dom，就像之前做的小實驗一樣。

vue能將物件中的methods綁定到dom事件中，方法和data綁定很類似，大家可以類比一下。

# 輸入框 v-model
vue提供一個雙向綁定的屬性`v-model`，用在輸入框，當輸入框內容從頁面中變更時，vue裡面相對應的data也會跟著修改。
```html
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
```
```javascript
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
```
這會輸出一行字，內容和message綁定，同時下面會有個用v-model綁定至message的輸入框，當輸入框內容更新時會更新message的內容，同時vue會更新與message綁定的那行字。

# 自定義元件
vue可以讓使用者自定義dom元件
```javascript
// Define a new component called todo-item
Vue.component('todo-item', {
  template: '<li>This is a todo</li>'
})
```
然後就可以在html中呼叫這個元件
```html
<ol>
  <!-- Create an instance of the todo-item component -->
  <todo-item></todo-item>
</ol>
```
但這樣的話內容是靜態的，於是我們可以將元素加上自定義屬性
```javascript
Vue.component('todo-item', {
  // The todo-item component now accepts a
  // "prop", which is like a custom attribute.
  // This prop is called todo.
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```
之後就可以利用`v-bind`（還記得嗎？在上一篇裡面用來將資料綁進屬性裡面）將data綁進元件中
```html
<div id="app-7">
  <ol>
    <!-- Now we provide each todo-item with the todo object    -->
    <!-- it's representing, so that its content can be dynamic -->
    <todo-item v-for="item in groceryList" v-bind:todo="item"></todo-item>
  </ol>
</div>
```
```javascript
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { text: 'Vegetables' },
      { text: 'Cheese' },
      { text: 'Whatever else humans are supposed to eat' }
    ]
  }
})
```
這個範例先使用`v-for`遍歷groceryList這個data，並將提出的元素用`v-bind`傳入todo這個自定義屬性裡面，vue就會透過template存取todo來動態生成元件。

透過自定義元素我們可以將前端的物件模組化，例如可以定義一個menu元件，當要顯示menu的時候只要在html中直接呼叫就可以了，若要顯示多個的話只要在呼叫一次就行了，不必把整段程式碼複製一遍。

# 過場動畫 Transition
vue的基礎架構大致上就到這邊，還記得在`v-if`一節曾提到vue有提供一些過場效果的語法，在此做個簡單介紹
```html
<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```
HTML部份有一個按鈕元素和段落元素，段落元素會根據show的值顯示或消失，按鈕被綁定至一個click事件，內容是將show的值反轉（這裡未用method宣告callback function，而是直接寫statement，本來猜想是會自動視為匿名函式，但它無法呼叫原生的javascript函式如alert，詳細的原因有待研究）
可以注意到段落元素被transition包住，宣告這裡面的元素要被apply過場動畫，過場動畫的名字是fade
```javascript
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```
js部份很簡單，只有宣告了一個show作為顯示切換
```javascript
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-active {
  opacity: 0
}
```
這邊多了一組css的語法，第一個是設定過場動畫的類型，有寫過css動畫的應該不陌生，`transition: opacity .5s`意思是透明度過場，長度是0.5秒，然後下面是設定顯示時的起始透明度值跟隱藏時的結束透明度值...很饒舌，官方有提供一張圖說明
![css語法說明](https://vuejs.org/images/transition.png)
v-enter: 元素將被顯示時的初始狀態，只套用在第一個frame，當元素開始顯示時會被移除
v-enter-active: 元素正在顯示及完成顯示的狀態，當元素完成顯示後會被移除
v-leave: 元素將被隱藏時的初始狀態，只套用在第一個frame，當元素開始隱藏時會被移除
v-leave-active: 元素正在隱藏及完成隱藏的狀態，當元素完成隱藏後會被移除

圖上的class名稱都是`v-`開頭，如果transition元素沒有命名的話會套用這個預設的class名稱，但如果有指定名稱的話就會是`name-`，如這個範例是`fade-`

所以整體解釋起來就是當元素要顯示時先將opacity初始化成0，然後套用transition動畫opacity 0.5s至opacity為1（原始狀態就是1所以不用特別設定），當元素要被隱藏時，opacity為1，然後套用transition動畫opacity 0.5s至opacity為0。

...好像還是很饒舌，大家自己改改看參數感受看看就會懂了～

# 總結
今天講解了如何用vue來處理使用者輸入，並介紹自定義dom元件，一樣希望大家都能實際操作看看～～

vue的基礎結構就到這邊，明天開始將進入vue的詳細架構講解，那麼明天見囉～～～

