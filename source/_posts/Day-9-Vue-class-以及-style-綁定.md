---
title: Day 9 Vue class 以及 style 綁定
date: 2019-03-07 16:09:51
tags: [web, frontend, vue.js]
categories: web
---

[原文章](https://ithelp.ithome.com.tw/articles/10211132)

今天要講的是和網站外貌很有關係的 `class` 及 `style` 綁定

我們有幾種方式為 DOM 設定樣式，其中兩種常用的是 `.css` 檔案和 `inline style`，前者通常會使用 class 來作為 selector。

在 vue 中也有提供相關的綁定讓你可以將樣式的設定程式化。

# Class 綁定
class 也是一種 DOM 屬性，因此語法和屬性綁定一樣使用 `v-bind:`，但我們可以利用一些布林參數來切換 class

```html
<div v-bind:class="{ active: isActive }"></div>
```
```javascript
data: {
  isActive: true
}
```

我們用 `isActive` 這個 data 屬性來切換 `active` 這個 class

`v-bind:class` 也可以和一般固定寫死的 `class` 混用

```html
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
```
```javascript
data: {
  isActive: true,
  hasError: false
}
```

結果為

```html
<div class="static active"></div>
```

和其他各種綁定一樣，當 data 更新的時候也會做出相關的更新，例如將 `hasError` 設定為 true，則上述元素會變為

```html
<div class="static active text-danger"></div>
```

> 註: js object key name 不能使用 hypen，例如 `{ hey-yo: 666 }` 這是不合法的，如果要使用 hypen 的話必須明確用 string 當作 key，如 `{ 'hey-yo': 666 }`，這就是為什麼上面的 `text-danger` 要加引號其他的不用，當然其他的加引號也是沒問題的。

我們也可以直接在 data 宣告一個 object 綁進去
```html
<div v-bind:class="classObject"></div>
```
```javascript
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

前一篇講的 computed property 也是沒問題的

```html
<div v-bind:class="classObject"></div>
```
```javascript
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

## Array syntax

有另外一個寫法是傳一個 array 進去

```html
<div v-bind:class="[activeClass, errorClass]"></div>
```
```javascript
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

結果是

```
<div class="active text-danger"></div>
```

這種寫法如果你想做 class 的切換可以用三元運算子

```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```

但這樣很不直觀，可以改用最一開始介紹的 object 的寫法

```html
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

這屬性也可以在自定義的 vue component 中使用，到時候介紹完 component 再回來看。

# Inline style
一樣是對 `sylte` 使用 `v-bind:` 來做綁定

```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
```javascript
data: {
  activeColor: 'red',
  fontSize: 30
}
```

看起來和 css 非常相似，很方便，不過 css 中有一些用 hypen 連接的屬性名稱 (hypen 連接的變數命名方式稱為 `kebab-case`)，需要改寫成 `camelCase`，但其實也可以直接使用 `kebab-case`，只是需要記得加上引號 (和前面 class 使用 hypen 的例子一樣，可以翻回去看。

一樣可以做一個 object 丟進去
```html
<div v-bind:style="styleObject"></div>
```
```javascript
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

看起來更像 css 了，感覺還蠻像我們做一個一個小的樣式方塊，然後需要的時候可以套用進去，還蠻有趣的。

`v-bind:style` 同樣也支援 array syntax，用法和 class 一樣
```
<div v-bind:style="[baseStyles, blueText]"></div>
```
```javascript
data: {
  baseStyles: {
    color: 'red',
    fontSize: '13px'
  },
  blueText: {
    color: 'blue',
  }
}
```

有一些 css 屬性有針對瀏覽器相容性的 prefixes，在 Vue 2.3.0 以後的版本有支援這個部分

```html
<div v-bind:style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

render 時只會顯示瀏覽器支援的最後一個，例如多數瀏覽器目前應該都支援 `flex` 而不需要任何 prefixe，則上面的 code 執行結果為

```
<div style="display: flex"></div>
```


# 總結
Vue 提供給我們相當方便可以程式化樣式套用的方法，可以更靈活且模組化地去設計 css。

下一次將更詳細的介紹之前講解過的條件式 render: `v-if`
