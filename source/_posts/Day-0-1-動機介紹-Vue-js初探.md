---
title: Day 0-1 動機介紹&Vue.js初探
date: 2019-03-05 01:20:58
tags: [web, frontend, vue.js]
categories: Web
---

# 還債
此為[前年的鐵人賽參賽作品](https://ithelp.ithome.com.tw/users/20103396/ironman/1030)，當時只寫了七天就不行了，最近又想重新學習 vue.js，因此將之前寫的東西搬運過來並從未完成的地方繼續，我仍主要會在鐵人賽那邊發表，但會同步在 Blog 這邊發布，只是不一定會每天寫就是了。

[原文章](https://ithelp.ithome.com.tw/articles/10184750)

# 前言
大家好，我是DC，機械系學生自學網頁程式設計，過去前端和後端都接觸過，後端寫過Node.js、PHP、ROR、Django，前端只用過Bootstrap之類的css框架，之前嘗試過AngularJs跟React等JS框架，但都是修改網路上找到的教學中的範例，我今年第一次知道有鐵人賽，也是第一次參加，這次我想嘗試直接閱讀vuejs的document學習，並從零開始建構一個vuejs的專案。

# 30天規劃
以下是[官方教學文件](https://vuejs.org/v2/guide/index.html)的目錄，前半個月將試著把教學讀完，後半個月會使用vue製作一個專案。

* Installation
* Introduction
* What is Vue.js
* The Vue Instance
* Template Syntax
* Computed Properties and Watchers
* Class and Style Bindings
* Conditional Rendering
* List Rendering
* Event Handling
* Form Input Bindings
* Components

所以前半月會是我讀文件的筆記，後半月是專案的工作進度報告。

# What is Vue?
教學第一篇應該要先介紹一下這個框架，但是我也是第一次接觸js框架，可能得等我看完文件後才能寫這個部份，簡單來講Vue是一套前端框架，主要目的是讓網頁可以動態Load資料，有別於傳統靜態的頁面將內容寫死在source code裡面，目前只知道vue跟react一樣有virtual DOM的架構，在此僅將原文介紹翻譯。

Vue (發音 /vjuː/, like view) 是個發展中的前端框架。 和其他整體性的框架不同，Vue 從底層開始逐漸的擴展。 Vue 的底層核心專注在View上，可以輕易的和其他library或現有專案整合。 另外使用Vue並整合其他工具也很適合用來寫複雜的一頁式網頁。

# Hello Vue
Vue.js和其他js框架一樣，只要在頁面上load框架的主程式就可以使用。

```
<script src="https://unpkg.com/vue/dist/vue.js"></script>
```

Vue.js的寫法和Angular.js很像，在html中直接宣告模板，然後用js填入內容
以下是個簡單的[範例](https://jsfiddle.net/chrisvfritz/50wL7mdz/)

```
<div id="app">
  {{ message }}
</div>
```

```
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

成功的話網頁將輸出

```
Hello Vue!
```

Vue會將data link進DOM，並且這個link是雙向的，當data的值改變了，Vue會自動更新DOM，這邊可以做個小實驗：打開瀏覽器的Console並修改`app.message`的值，你會發現頁面上的文字跟著改變了。

以上就是Vue.js基本的樣子，下一篇將開始介紹更進階一點點的用法。
