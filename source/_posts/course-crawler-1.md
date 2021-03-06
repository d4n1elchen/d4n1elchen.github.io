---
title: 又見課程爬蟲 (1)
tags: [crawler,NCKU,node,web,backend]
id: 622
categories: web
date: 2017-01-18 01:26:26
---

首先是久違的發文www雖然應該沒人看（

這學期應該是我大學最後一次選課，課程爬蟲這鬼東西我寫了不下三次了，第一次是用C#寫，那也是我第一次寫爬蟲，第二次是上次為了寫出漂亮的課程查詢（最後也沒多漂亮）而寫，然後就是這次了，這次是想要寫課表預排，浪打port過來的那個掛了，沒詳細去問發生什麼事情，不過那個也不是說非常好用，於是就又挖了個坑跳。

第二次的時候爬蟲是用node寫的，那次只有把系所代碼存起來而已，課程列表是即時爬的，但使用經驗非常的差，因為學校的網站load很慢，加上node爬蟲也沒有特別快，列表載入的延遲都是以數秒計算的，所以這次想要嘗試把資料存進資料庫，我選了之前一直很想用看看的mongodb，理所當然爬蟲語言還是選node，雖然用的語言一樣但是我重寫了，因為覺得之前寫得不是太漂亮。

至於我花了兩個小時接近快寫完的時候開始後悔為什麼要用node又是另一個故事了，文章結尾在講www

這次就來做個成大課程查詢的爬蟲筆記，repo的連結[在這](https://github.com/ccns/ncku-course-crawler)

node的爬蟲需要兩個套件：`request` `cheerio`

request用來載入頁面，cheerio則可以讓你用和jquery幾乎完全相容的語法寫爬蟲

安裝

```bash
npm install request cheerio
```

基本使用

```javascript
var request = require("request");
var cheerio = require("cheerio");

request(url, function(err, res, body) {
  if(!err && res.statusCode == 200) {
    console.log(body);
    var $ = cheerio.load(body);
    var div = $("div").text();
    console.log(div);
  }
});
```

基本上大致上使用的套件已經介紹完畢，沒錯，用node寫爬蟲就是這麼簡單，剩下就是jquery的事了

因為語法跟函式和jquery幾乎是一樣的，所以我習慣直接先用web console試jquery，然後再貼到node裡面
![](http://i.imgur.com/dPzpGhS.png)

分享一下我用了哪些selector爬到需要的資訊

首先系所代碼是從[首頁](http://course-query.acad.ncku.edu.tw/qry/index.php)爬的

`var lis = $("#dept_list li")`可以爬到12學院的列表，然後神奇的地方就來了，li裡面居然是class名稱是theader跟tbody的div，要不全部用div，要不都用ul&gt;li，還是第一次看到這樣寫的

接著遍歷12個學院列表可以得到各學院的系所列表，然後從列表的連結中擷取系所代碼

```javascript
var as = $(li).find(".dept a");
for (var i=0; i<as.length; i++) {
  var a = $(as[i]).attr("href").split("=")[1]; //?dept_no=XX
}
```

因為這份系所列表還要留著後面取得課程列表的，為了避免callback hell我這次嘗試使用了promise，語法如下

```javascript
function getDeptNo() {
  return new Promise(function(resolve, reject) {
    do crawler ...
    resolve(result)
  }
}

getDeptNo().then(function(depts) {
  console.log(depts)
})
```

簡單講就是本來return是執行callback function，改成return一個promise，本來要吃進callback function的結果餵給resolve，這樣結果就會被餵進後面的then裡面，不過到這邊還沒顯現出promise可避免callback function的特性，如果有興趣可以參考[這篇](http://huli.logdown.com/posts/292655-javascript-promise-generator-async-es6)

然後是爬課程列表，課程列表是透過餵不同的dept_no到這裡[http://course-query.acad.ncku.edu.tw/qry/qry001.php?dept_no=E1](http://course-query.acad.ncku.edu.tw/qry/qry001.php?dept_no=E1)取得的（話說首頁點擊連結之後是ajax載入這個頁面的內容，根本就是個完整頁面了，到底ajax目的是什麼啦）

`$("tr[class^=course_]")`首先把一行行的課程資料爬出來，這裡用了一個之前沒用過的selector

`class^=course_`意思是以course_開頭的class，因為好像不同年級會用不同的class名，一樣意味不明，style沒差，也沒有什麼js event啊

接著遍歷每個column把需要的資料取出來並做一些前處理，比較怪的地方是有些欄位會有一大堆多餘的空白，還要把那些東西濾掉，例如教室

```javascript
var classroom = $(tds).eq(17).text().replace(/\s+/, " ");
```

另外其實這個頁面有兩個蠻有用的連結，一個是課程地圖的連結，有些比較認真的老師會放一些課程大綱在裡面供學生參考，另外一個是這邊還會有該課程的moodle網址（不管老師有沒有開都會有連結，只是點進去會說未開放）

```javascript
var map_url = $(tds).eq(10).find("a").attr("href");
var moodle_url = "http://course-query.acad.ncku.edu.tw/qry/" + $(tds).eq(18).find("a").attr("href");
```

接著把完整我爬了哪些欄位貼出來好了，希望不會太佔版面（

```javascript
var dept_no = $(tds).eq(1).text();
var course_no = $(tds).eq(2).text();
var code = $(tds).eq(3).text();
var classes = $(tds).eq(5).text().replace(/\s+/, "");
var year = $(tds).eq(6).text();
var name = $(tds).eq(10).text();
var map_url = $(tds).eq(10).find("a").attr("href");
map_url = typeof map_url === &#039;undefined&#039;? &#039;&#039;: map_url;
var required = $(tds).eq(11).text();
var credit = $(tds).eq(12).text();
var teacher = $(tds).eq(13).text();
var selected = $(tds).eq(14).text();
var remain = $(tds).eq(15).text();
var time = $(tds).eq(16).text();
var classroom = $(tds).eq(17).text().replace(/\s+/, " ");
var memo = $(tds).eq(18).text();
var moodle_url = "http://course-query.acad.ncku.edu.tw/qry/" + $(tds).eq(18).find("a").attr("href");
var limit = $(tds).eq(19).text().replace(/\s+\//, "/");
```

兩個都寫完了，照理講可以開始爬然後寫進資料庫了

才怪

接下來才是惡夢的開始，首先是學校網站真的載入很慢，常常載到timeout，我不敢想像跑全校這麼多系所要花多久，如果timeout又要另外處理，要重試嗎？還是就跳過然後記起來下次重跑？超麻煩的啦

另外就是雖然有promise，但是還是要寫很多return then，我要traverse這麼多系所，總不可能一個一個寫return then吧，據說可以用generator來解決這個問題，讓他看起來就像sync的程式一樣，另一個解決方案是就不管了，讓他們同時執行然後各自寫進db，mongodb的話應該可以做到這樣的事情

如果用python就不會有async的問題啊...而且速度應該會比node快上許多，我到底為什麼堅持用node呢？

不過剛想想好像還是有辦法解決的，暫且還是先想辦法用node爬看看吧