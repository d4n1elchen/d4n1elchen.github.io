---
title: 大報社!
tags: [web,Node,frontend,backend]
id: 972
categories: web
date: 2017-02-12 00:07:38
---

http://大報社.club/
之前期中考的時候壓力太大買了這個網域，一直不知道要做什麼，結果前幾天腦洞大開刻了這鬼東西...

這篇記錄一些碰到的困難跟解決方法

首先是音樂撥放的部分，本來可以直接用html5 audio tag處理，但是這樣不潮
後來是找了方法自訂播放器的樣式
方法大致上是用range input當成播放進度條，然後再加上撥放跟暫停的按鈕
可以再另外做聲音大小聲，但我懶，不過之後如果哪天腦洞又開了會做個加速鈕

參考資料: [Create a Customized HTML5 Audio Player](https://webdesign.tutsplus.com/tutorials/create-a-customized-html5-audio-player--webdesign-7081)

再來是大報社的那個拋射特效，我是自己寫的，用到之前上計圖的時候用OpenGL做動畫學到的方法
簡單來說我讓他每隔一小段時間更新一次位置，位置的計算就是用高中學過的平面運動方程式
然後再隨機指定初速度跟射出的圖片，這邊嘗試使用了閉包，以下是主要的程式

```javascript
function parabola(v) {
  var random = Math.floor(Math.random()*shoots.length);
  var src = shoots[random];
  let img = $("<img>").attr('src',src).addClass('gif').css({width: '100px', height: '100px'});
  $("body").append(img);
  img.offset({top: y0-50, left: x0-50});
  setTimeout(updatePos.bind(null, 1), 23);
  function updatePos(t) {
    var x = x0 + v[0]*t;
    var y = y0 - (v[1]*t + 1/2*g*t*t);
    img.offset({top: y-50, left: x-50});
    t = t+1;
    if(img.position().top > window.innerHeight
        || img.position().top < -img.height()
        || img.position().left > window.innerWidth
        || img.position().left < -img.width())
      img.remove();
    else
      setTimeout(updatePos.bind(null, t), 23);
  }
}
```

設定的時間間隔是23毫秒，這是try and error測試出來看起來流暢的值
不過理論上人眼只要12fps以上就會感覺是順暢的動畫
一般電影是24fps，即1張畫面停留1000/24=41ms
但實際上我一開始設定33ms的時候感覺還是頓頓的不知道為什麼
可能gif本身會再拖累瀏覽器效能吧(不過更新頻率越高瀏覽器的負荷也越重，需在這之間取捨)

此外iOS的朋友們可能會發現目前網頁進去是沒有聲音的，因為iOS禁止自動撥放
還有就是按鈕連點會變成縮放，在iOS 10之後據說是為了增加網站的accessibility禁用了`user-scalable=no`....我心中是滿了問號阿!!! 參考資料: [css - disable viewport zooming iOS 10 safari? - Stack Overflow](http://stackoverflow.com/questions/37808180/disable-viewport-zooming-ios-10-safari)

不過應該還是有辦法用javascript來解決掉，等有心情再說吧