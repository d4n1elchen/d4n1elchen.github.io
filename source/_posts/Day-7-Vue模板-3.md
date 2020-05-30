---
title: Day 7 Vue模板-3
date: 2019-03-05 15:29:32
tags: [web, frontend, vue.js]
categories: web
---

[原文章](https://ithelp.ithome.com.tw/articles/10185322)

昨天談到了指示子的一些性質與用法，今天要介紹filter，寫過angular的應該對這語法不陌生

# filter
filter直譯是過濾器，但寫過濾器好像有種要把不要的東西過濾掉的意思（雖然可以這麼做），不過一時也想不到更好的翻譯，簡單來說filter的用途是在資料顯示之前對資料進行前處理，以下是個簡單的範例

```html
<!-- in mustaches -->
{{ message | capitalize }}
<!-- in v-bind -->
<div v-bind:id="rawId | formatId"></div>
```

```javascript
new Vue({
  // ...
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
})
```

這個範例將要顯示的字串的開頭第一個字都改成大寫，注意到formatId並沒有實做，那邊只是展示如果要在屬性裡面使用filter要怎麼寫而已

filter可以連續使用

```html
{{ message | filterA | filterB }}
```

filter是個js function，vue會將filter的對象當成第一個參數傳入filter，你也可以使用一個以上的參數

```html
{{ message | filterA('arg1', arg2) }}
```

`'arg1'`這個字串會作為第二個參數（第一個是message）傳入filterA，`arg2`是第三個參數...以此類推

要注意的一點是filter在vue 2.x裡面只能使用於插入本文`{% raw %}{{}}{% endraw %}`與插入屬性`v-bind`中，因為filter是被設計來進行文字處理的，如果要進行更複雜的計算或轉換，可使用`computed`屬性

# 縮寫
vue提供了縮寫給兩個最常用的語法：`v-bind` `v-on`
```html
<!-- full syntax -->
<a v-bind:href="url"></a>
<!-- shorthand -->
<a :href="url"></a>
```

```html
<!-- full syntax -->
<a v-on:click="doSomething"></a>
<!-- shorthand -->
<a @click="doSomething"></a>
```

# 總結
vue的模板系統語法和其他框架大同小異，基本的功能就是內容插入、條件判斷、迴圈、filter，模板是動態生成內容一個很重要的部份，有了模板系統我們可以直接在html文本裡面寫入邏輯

模板系統大致上就到這邊，明天將提到兩個新的vue options，compute＆watch
