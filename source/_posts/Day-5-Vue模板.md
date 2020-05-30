---
title: Day 5 Vue模板
date: 2019-03-05 15:01:55
tags: [web, frontend, vue.js]
categories: web
---

[原文章](https://ithelp.ithome.com.tw/articles/10185163)

今天要介紹Vue的模板語法，Vue使用HTML-based的模板語法，並將模板compile成virtual DOM，然後和vue物件中的data進行雙向綁定，這邊常會看到一個單字: reactivity，直譯的話是反應，我感覺意思大概有點像是DOM會對data的改變進行反應，Vue會自動計算更新DOM的最短路徑。

Vue其實也有提供JSX的Support，如果對JSX熟悉的話可以嘗試使用看看，這邊只介紹模板系統。

今天要介紹的是插入內容的語法。

# 文字
插入element的內容
```html
<span>Message: {{ msg }}</span>
```
Vue預設會對內容進行雙向綁定，若不想這麼做的話可使用`v-once`這個元素屬性
```html
<span v-once>This will never change: {{ msg }}</span>
```
這樣的話內容就只會被更新一次，不會隨著data的更新而更新

# HTML
雙大括號的內容會以純文字的形式插入，如果想插入HTML的話要使用`v-html`屬性
```html
<div v-html="rawHtml"></div>
```
但注意動態插入HTML會有XSS攻擊的風險，所以記住插入的HTML一定要是被信任的，且盡量避免讓使用者插入HTML

# 屬性
插入屬性的方式是`v-bind:attr`
```html
<div v-bind:id="dynamicId"></div>
```
若綁定的資料型態是boolean的話，可以做屬性的切換
```html
<button v-bind:disabled="someDynamicCondition">Button</button>
```
若值是false的話這個屬性會被移除

# Javascript
你可以在模板語法中使用JS的語法
```html
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-' + id"></div>
```
但只能是單行的敘述，像以下語法就是不合法的(不會執行)
```html
<!-- this is a statement, not an expression: -->
{{ var a = 1 }}
<!-- flow control won't work either, use ternary expressions -->
{{ if (ok) { return message } }}
```
而且只能執行在白名單內的全域函式像是Math跟Date，無法使用自定義函式

> 這點回應了 {% post_link Day-3-Vue使用者輸入、自定義元件 %} 中無法直接使用 alert() 的原因

以上是內容插入的部分，明天將繼續討論指令語法
