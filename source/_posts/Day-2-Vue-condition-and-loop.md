---
title: Day 2 Vue條件判斷、迴圈
date: 2019-03-05 01:30:17
tags: [web, frontend, vue.js]
categories: web
---

[原文章](https://ithelp.ithome.com.tw/articles/10184749)

昨天介紹了基本語法，將data link進DOM element的內容中，今天將介紹如何將data link到element的attribute裡以及條件、迴圈等語法。

# 元素屬性 v-bind
除了將data載進內容之外，vue還可以將內容link到元素屬性中:
```html
<div id="app-2">
  <span v-bind:title="message">
    Hover your mouse over me for a few seconds to see my dynamically bound title!
  </span>
</div>
```
```javascript
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: 'You loaded this page on ' + new Date()
  }
})
```
只要在元素屬性前面加上`v-bind:`，屬性內容填上data的名稱，就可以將data綁進屬性中，成功的話會產生一個span標籤，當你把滑鼠移到上面時會顯示一個懸浮視窗`You loaded this page on Fri Dec 02 2016 13:23:19 GMT+0800 (CST)`，後面的時間會根據你載入頁面的時間不同喔～

這邊一樣可以透過`app2.message`來修改屬性的值。

這裡可以注意到`v-`這個prefix，vue的所有特殊屬性都會以`v-`開頭。

# 條件判斷 v-if
這裡的條件判斷用來決定元素是否顯示
```html
<div id="app-3">
  <p v-if="seen">Now you see me</p>
</div>
```
```javascript
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```
成功的話p這個element會正常顯示，這時在console中輸入`app3.seen = false`就會讓p消失，接著輸入` app3.seen = true`又會重新顯示哦～

此外vue還提供一些基本的過場效果，如淡出、淡入等等，等講到事件綁定時會以範例形式跟大家介紹。

# 迴圈 v-for
vue可以利用迴圈根據內容產生重複的element
```html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
```
```javascript
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: 'Learn JavaScript' },
      { text: 'Learn Vue' },
      { text: 'Build something awesome' }
    ]
  }
})
```
成功的話會輸出一個list
1. Learn JavaScript
2. Learn Vue
3. Build something awesome

理解這段程式需要一點javascript object的概念（雖然之前好像也需要，但比較直觀，這邊稍微複雜一點），如果看不懂的話這邊稍微做點粗略的補充，懂的人可以直接略過。

js object結構類似於json，最外面以一組大括號`{}`包起來，各組資料間以逗點`,`分隔，資料是以key-value的格式儲存，差別在於json只能儲存字串資料跟另一組json結構，js object可以存任何js中的資料型態，包含數值、字串、函式、陣列，當然也可以存另一個js object。

以下是一個簡單的範例
```javascript
var obj = {
    key1: 10,         // 數值
    key2: "string1",  // 字串
    key3: function() { alert(); },  // 函式
    key4: [ 55, 66 ], // 陣列
    key5: {           // Object
        key5_1: 87
    }
}
```
若要存取裡面的內容可以用`.`來存取，如：
```javascript
console.log(obj.key2); // string1
```
會被稱作object是因為我們可將裡面的每一個元素視為這個object的屬性，屬性的名稱就是他的key的名子。

初學者很容易將js array跟js object搞混（至少我搞混了好一段時間），因為js array跟js object都用逗點分隔，且array也不限儲存內容的資料型態，array中可以存ojbect，object中也可以存array，兩個混在一起寫更容易亂掉，在此做個小比較。（補充：其實在 js 裡面 array 的型別也是 object，這邊指的是初始化的語法不同）
```javascript
      | 宣告方式 |   儲存結構   | 範例
------+---------+------------+---------------------
js obj| 大括號{} | key: value | { num1: 1, num2: 2 }
------+---------+------------+---------------------
js arr| 中括號[] |      value | [ 1, 2 ]
```
再回來看剛剛的範例
```javascript
var app4 = new Vue({           // 宣告一個vue物件並傳入一個js object
  el: '#app-4',                // 屬性el的值是個字串
  data: {                      // 屬性data的值是個object
    todos: [                         // todos是data中的一個屬性，值是個array
      { text: 'Learn JavaScript' },  // array中有三個元素，各是一組object
      { text: 'Learn Vue' },
      { text: 'Build something awesome' }
    ]
  }
})
```
有了object跟array的概念後再回來看template
```html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
```
有點像foreach，遍歷todos這個array，並將每個值提出為todo，然後在template中呼叫todo.text。

所以要注意的地方是todos要是個array，且array中的每一個object都要有text這個屬性。

當然array中不一定要是object，也可以放一般的資料型態，如下面這個範例
```html
<div id="app-4">
  <ol>
    <li v-for="member in members">
      {{ member }}
    </li>
  </ol>
</div>
```
```javascript
var app4 = new Vue({
  el: '#app-4',
  data: {
    members: [
      "Daniel Chen",
      "David Cheng",
      "Destiny Chain"
    ]
  }
})
```
將輸出一個list
1. Daniel Chen
2. David Cheng
3. Destiny Chain

js array可以透過push這個method來新增元素，可以試著在console中輸入`app4.members.push("Donuts Chicken")`，頁面上的list會自動新增一個項目。

# 總結
今天介紹了vue的attribute綁定，並介紹了兩個特殊的attribute: v-if跟v-for，大家可以直接在昨天的那個[JS fiddle](https://jsfiddle.net/chrisvfritz/50wL7mdz/)中測試跟練習。

目前都只介紹vue的語法以及一些特性，若順利進入專題製作階段會嘗試介紹一下開發環境的架設。

明天見囉～
