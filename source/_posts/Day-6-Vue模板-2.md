---
title: Day 6 Vue模板-2
date: 2019-03-05 15:19:31
tags: [web, frontend, vue.js]
categories: web
---

[原文章](https://ithelp.ithome.com.tw/articles/10185259)

昨天講到的模板語法都是跟插入內容有關，不管是插入文字內容、插入HTML內容、插入屬性內容

不過稍微修正一下昨天的說法，昨天說模板系統分成插入跟指令，今天仔細讀了一下發現原文"Directive"應該泛指所有`v-`開頭的特殊DOM屬性，所以這邊應該翻成「指示子」會比較好吧？（以下暫稱指示子，若有更好的翻譯歡迎指教）

今天要介紹的是這些特殊屬性的特性

# 指示子 v-
如前言所述，Vue的特殊DOM屬性都會以`v-`開頭，如`v-if`，這些特殊屬性的內容必須是單行的Javascript敘述（除了`v-for`之外），可以回憶一下之前的範例
```html
<p v-if="seen">Now you see me</p>
```

# 參數
有些指示子會有參數，語法是在指示子後面加上`:`，然後接著參數，如`v-bind`
```html
<a v-bind:href="url"></a>
```
這邊```href```是```v-bind```的參數，告訴```v-bind```要綁定的對象

又如`v-on`
```html
<a v-on:click="doSomething">
```
`click`是`v-on`的參數，表示`v-on`的綁定對象

# 修飾符
有些指示子除了參數外還會有後綴，語法是加上`.`然後加修飾符，如`v-on`
```html
<form v-on:submit.prevent="onSubmit"></form>
```
這可以讓事件被觸發時自動呼叫```event.preventDefault()```

# 總結
以上是有關指示子的介紹，要注意的地方是並沒有把所有的例子舉出，如修飾符不只有`v-on`有而已，其他的等有碰到再介紹囉，明天將介紹模板中的filter
