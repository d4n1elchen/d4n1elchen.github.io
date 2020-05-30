---
title: 'Re: 從零開始的網頁設計之路'
tags: [CSS,Web,Hahow,frontend]
id: 3
categories: web
date: 2016-09-09 00:59:41
---

[Re: 從零開始的網頁設計之路](http://codepen.io/team6612/pen/ALjJZj)

暑假前在hahow上買了這堂課 [動畫互動網頁程式入門](https://hahow.in/courses/56189df9df7b3d0b005c6639) ，會知道這個是因為五月底的時候去了複雜生活節，聽了這位大大的講題，活動也有hahow的演講，那次真的認識不少新東西，也見識到同年齡層的等級差距Orz<!--more-->

總之這個月開始課程陸續上傳上來了，我覺得他們這樣一次上傳多堂課的做法還蠻棒的，不然之前看公開課最後都是隔一個禮拜不是忘了就是懶得看，而且每個影片的長度都不長，累了可以很好抓段落休息。

蠻推這位講者的，前面觀念敘述很清楚，而且他有設計的底子，在解釋一些排版上的觀念的時候比較有整體式的思考，不會像一般的網頁教學都亂排，只教你怎麼打語法。

這是我昨天做的第一份作業（其實不算作業啦，課程有分Project跟Homework，Project比較像範例，Homework是要上繳到系統上的），是用純CSS寫的名片，很廚的用了Re:0的配色www

以下記錄幾個重點
```
*
  //border: solid 1px black
  font-family: 微軟正黑體
  font-weight: 100
  letter-spacing: 1px
  position: relative
```
他交了一招我覺得蠻實用的，`*`這個selector以前從來沒用過，他是選擇所有元素的意思，在這個選擇器下設定border可以為每個元素加框線，以確定各元素的位置以及實際的大小，排版時先加上框線，將每個元素定位到想要的位置之後再開始寫樣式，可以節省很多調整的麻煩

這邊學到兩個新的屬性: `letter-spacing`、`font-weight`，前者用來調整字距，把字距調大可以讓視覺比較放鬆一點，之後若是有需要大字距的設計時可以用；後者是調整字級，數字課堂上是說100~900，但不太確定為何是這個數字，要在調查看看

另外這邊設定`position: relative`的原因待會一起講
```
a 
  color: inherit
  text-decoration: none
```
這段也是重設的語法，連結的預設樣是大家都知道，就藍底還有白線，超醜，`color: inherit`是把顏色設定成和上級一樣，`text-decoration: none`是把底線拿掉
```
html,body 
  box-sizing: border-box
  width: 100%
  height: 100%
  padding: 0px
  margin: 0px

body
  background-color: #F9F7F7
  border-top: solid 50px #F7B3CC
  border-bottom: solid 50px #8DD3F5
```
`box-sizing`這邊的用途是讓border成為元素空間的一部分，而不是凸出元素之外造成額外的空間

然後這邊又是css一個很怪的預設，預設的html（還是body忘了）會跟最外層的邊框有預設的margin，如果要做完全滿版的頁面的話要把margin設0不然會有白邊
```
.vertical-center
  height: 100%
  display: flex
  align-items: center
  justify-content: center
```
這是垂直置中的語法，用的是flexbox，詳情可以參考[這裡](http://zh-tw.learnlayout.com/flexbox.html)
```
.namecard
  background-color: #45464B
  color: white 
  width: 400px
  height: 250px 
  border-radius: 6px 
  margin: auto
  padding: 10px 20px 
  box-shadow: 10px 10px 10px rgba(0,0,0,0.15) 
  overflow: hidden
```
這是主要中間那張卡片的樣式，以前不太用`border-radius`，不過這次用完發現圓角框蠻有質感的，以後可以嘗試多用

`margin: auto`的作用是讓元素水平置中

`box-shadow`雖然這邊沒寫到但是課程有講到，他有第四個位置參數，是設定內縮或外放，可以用來設計`material desin`那種卡片漂浮的感覺，用法像這樣

`box-shadow: 0px 8px 10px -5px rgba(0,0,0,0.15)`
<div style="margin:20px;">
<div style="background-color:white;border:solid 1px #DDD;text-align:center;width:150px;box-shadow:0 8px 10px -5px rgba(0,0,0,0.15);">這是一張卡片</div></div>
先把陰影內縮之後往下調整，這樣就會有向上漂浮的感覺

最後的`overflow: hidden`是為了把那兩個裝飾的小方塊超出元素範圍的部分隱藏，以前從來沒有想過要這樣做，包含用div去畫圖，之前總是覺得在網頁上畫圖一定要用svg，svg的語法又很麻煩很懶得用，但沒想到用div加上`background-color`、`border`、`transform`等屬性就組合出很多種形狀了，也算是大開眼界，感覺功力大增

## Position &amp; Display

然後來講一下這次看前22堂課最大的收穫吧，終於搞懂position和display的用法，display之前除了none知道是元素會完全消失之外其他完全沒概念wwww，然後position的話是有時候不管怎麼調都不在想要的位置上，很困擾QQ

### Display

[CSS display property](http://www.w3schools.com/cssref/pr_class_display.asp)，這是W3 school關於display這個屬性的頁面，剛剛才發現除了好不容易搞懂的block/inline/inline-block之外還有一大票東西www淦wwwww，總之先記錄這三個

`block`是div元素的預設值，html的概念是元素會一個區塊一個區塊向下排列，所以不同的`block`會各自佔有完整的一行，因此設定`block`的元素會自動往下跳一行，即便他是某段文字的子元素，像這樣
<div style="margin:20px;">同一行文字
<div>這邊卻斷行了</div>
太過分啦！</div>
如果設定成inline或inline-block的話就可以讓他們在同一行

`inline`跟`inline-block`的差別在於`inline`的話會將元素視為該上級元素的一部分，因此無法調整其左右的內外距（但位置可），只能調整上下的（我也不懂為什麼要這樣設定），`inline-block`的話會變成包覆內容的一個區塊，就沒有這樣的限制了，因此若有區塊需要符合內文大小也可以設定成`inline-block`

### Position

position就有趣了，不過這觀念我剛好前幾天在寫聊天室的時候有釐清過了，我是看[這邊](http://zh-tw.learnlayout.com/position.html)學到的

position基本上有4種值，`static/relative/absolute/fixed`，意思是設定當你使用`top/bottom/left/right`來調整位置時的參考，分別是`不變/相對/絕對/固定`

`static`跟`relative`和`fixed`比較沒問題

`static`意思是這個元素完全不會動，無法透過`top/bottom/left/right`來調整位置

`relative`的話是相對於元素當下的位置來調整

`fixed`是相對於整個瀏覽視窗並將其固定在畫面上不隨卷軸捲動

問題出在`absolute`，`absolute`顧名思義是絕對定位，但絕對是絕什麼對呢？`absolute`是相對於該元素的上級元素做定位，但前提是上級元素是可被定位的（也就是`position`屬性值非`static`），如果上級元素是不可被定位的（也就是`position`是`static`）的話元素會相對於body來定位，但和`fixed`的差別是他會隨卷軸捲動，媽的超奇葩設定，完全不知道用意為何wwwww

這就是為什麼最一開始要把所有元素都設定成`relative`的原因，若是碰到需要用到`absolute`定位的情況才不用在那邊看上級元素是否可被定位，全部設定成`relative`不會有任何改變，因為預設的`top/bottom/left/right`都是0，這招蠻方便的，因為設定成absolute要相對body定位的狀況比較少

嘛，大概就這樣，另外提一個可能大家沒注意到的地方，背景那個Re:我刻意調得跟Re:0官方的Logo很像，調蠻久的但是好像不太明顯wwwwww，讓他在這邊再現！！
<div style="margin-bottom:40px;">
<div style="line-height:normal;font-weight:100;font-size:150px;color:rgba(0,0,0,0.15);font-family:serif;text-shadow:0 0 15px rgba(0,0,0,0.1);height:130px;overflow:hidden;letter-spacing:-15px;">Re<span style="position:relative;font-size:100px;color:rgba(0,0,0,0.5);bottom:10px;left:5px;">:</span></div>
</div>