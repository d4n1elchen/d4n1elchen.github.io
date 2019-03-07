---
title: Day 8 Vue 計算屬性及屬性監看
date: 2019-03-06 01:29:20
tags: [web, frontend, vue.js]
categories: Web
---

[原文章](https://ithelp.ithome.com.tw/articles/10185324)

前幾篇介紹了如何將 vue 物件中的屬性綁定到模板中，其中在 Day 5 提到我們可以在模板中直接插入 js 語句，可以對要顯示的資料進行處理，但 vue 並不建議你這麼做，因為模板會顯得很亂而且非常不值觀，如下範例

```html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

除了使用 `filter` 以外，vue 提供了兩個可以對資料進行處理的方式 `computed property` 以及 `watch`。

# 計算屬性 (Computed property)
當有對資料進行複雜的處理的需求時，我們可以在 vue 物件的 computed 屬性加入自定義的 computed property，並定義其對應的函式

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```
```javascript
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // a computed getter
    reversedMessage: function () {
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    }
  }
})
```

在 computed property 中，`this` 對應到的是該 vue 物件，可以看到這個例子裏面 `reversedMessage` 相依於 `message` 屬性，當 `message` 更新時，`reversedMessage` 就會重新計算，並更新到 DOM 中。

可以嘗試在 console 中修改 `message` 看看
```javascript
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```

# 計算快取 (Computed cache)
從上述範例可以發現 computed property 會去監聽在其相依資料的更新，並在更新時重新計算，若資料沒有更新則該屬性的 DOM 會顯示上一次計算好的內容，稱為 computed cache。

這邊就可以指出使用 computed property 和定義一個 method 並在模板中呼叫最主要的差異，那就是 method 在每次 re-render 時都會重新執行，先考慮以下用 method 實作的 `reverseMessage`

```html
<p>Reversed message: "{{ reverseMessage() }}"</p>
```
```javascript
// in component
methods: {
  reverseMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

每次 rerender 時 `reverseMessage` 會重新計算結果，因此如果 `messag` 在過程中有變更，則 `reverseMesage` 的結果也會跟著改變。

而因為我們更新了 `message`，這個操作會更新 computed cache，因此如果用 computed property 的話，從 render 出的結果來看會與 computed property 一致。

但如果我們考慮以下範例

```
computed: {
  now: function () {
    return Date.now()
  }
}
```

這個 computed property 所參考的是一個 Date 物件，當他被建立後就不會再更新了。如果我們用 method 來實作一樣的功能，則每次 render 該物件時都會重新執行一次，now 就會不斷被更新，可是如果用 computed property 的話，因為這個物件不會被更新，所以 rerender 時會從 computed cache 拿上一次計算的結果，now 就不會被更新。

使用 computed cache 的時機是，當有個需要消耗較多運算資源的資料（例如 AJAX）需要被顯示在很多地方時，若用 method 則每個地方該方法都會被重新運算，會造成相當大的資源消耗，此時有 computed cache 可以大幅增進效能。

# Computed setter
一般我們只會對 computed property 設定 getter，但我們也能為它指定 setter

```javascript
// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```

在 console 中執行 `vm.fullName = 'John Doe'`，可以觀察到 `firstName` 和 `lastName` 被更新了。

# 屬性監視 (Watched property)
和 computed property 類似，我們可以建立一個函式來偵測某個屬性的更新

```html
<div id="demo">{{ fullName }}</div>
```
```javascript
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

當 `firstName` 或 `lastName` 被更新時，watch 會去執行對應的兩個函式，進而更新 `fullName`。

但這個例子用 computed property 可以寫得更簡潔

```javascript
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

最後官方給了一個複雜的 [watch 範例](https://vuejs.org/v2/guide/computed.html#Watchers)，大家可以自己研究一下，簡單來說我們可以用 watch 來偵測使用者輸入，詳細的資料流我覺得大家看著 code 自己在腦中跑過一次會對這兩個工具更加熟悉，不過看不懂沒關係，因為裡面牽扯到一個後面才會提到的用法 `v-model`。

最後提一下，watch 的應用時機應是當我們想對資料的更新做出複雜的反應和操作時使用。

# 總結
今天介紹了 computed 以及 watch 屬性，並介紹了 computed 的 computed cache 特性以及 computed/method 的應用時機。

下次將介紹 class 和 style 的綁定，為你的 DOM 加上生動的樣式吧！
