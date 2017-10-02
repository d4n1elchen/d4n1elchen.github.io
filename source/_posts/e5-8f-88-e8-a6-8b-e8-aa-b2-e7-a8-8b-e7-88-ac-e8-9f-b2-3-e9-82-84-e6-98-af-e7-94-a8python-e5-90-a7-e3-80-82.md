---
title: 又見課程爬蟲 (3) - 還是用python吧。
tags:
  - Web
  - NCKU Course Query
  - python
  - Backend
  - NCKU
id: 824
categories:
  - Web
date: 2017-01-21 20:28:06
---

本來第三篇要寫mongodb的，但是今天沒心情寫（？

上一篇提到我好不容易把爬蟲用node搞出來了，但因為還是不太熟promise的關係，加上db操作之後還是碰到了一些問題，大崩潰Orz

於是~~我一氣之下就把整個專案砍了改用python寫~~我想了一下，會造成異步處理困難主要是因為http request的延遲跟不上，檔案還沒載完、爬蟲還沒跑完，就要寫入資料庫，我想了一個方法來解決這個問題，就是先把頁面都下載下來然後再跑爬蟲，用檔案IO的方式延遲會比較小一點

雖然下載檔案沒有先後次序的問題，但我想試試看python的multithreading所以就用python寫了，只有用到一個額外函式庫叫requests，可以簡化送http reqests的操作（原本的話要用內建的urllib來操作，比較不值關），事先先把系所列表先寫到一個檔案裡面，讀出來之後開multithreading去下載檔案

這篇就簡介一下threading的用法吧

開n個thread來跑run這個function

```python
import threading

def run():
  do something ...

for i in range(1:n)
  threading.Thread(target=run).start()
```

如果要傳參數的話，可以用args這個參數

```python
import threading

def run(a1):
  do something ...

for i in range(1:n)
  threading.Thread(target=run, args=(a1,)).start()
```

注意`(a1,)`後面的逗點，那是必要的，因為args是吃tuple，加逗點才能宣告成tuple，不然括號會變成是statement

本來還想做內容更新檢查的，就是要寫入檔案之前先檢查是否內容有變更，原本想用hash來比對，但是後來想想檔案都載下來了才去比對否有更新好像沒什麼意義，直接覆蓋掉也不會怎樣（目前資料庫更新也是這麼做，每次更新都把過去的資料蓋掉，不過資料庫那邊好像比較值得做這件事）

另外選課系統的首頁每次刷新會有些許的內容差異這點也很令人崩潰（據說是有設計分流才會這樣），最後索性就不做內容比對了

到這邊好像蠻順利的，但是學校的網站什麼時候給你驚喜你不會知道www，課程查詢的頁面不知道為什麼有時後會request timeout，過一陣子之後重連又正常，如果這是刻意設計的話應該是防止ddos的機制吧？希望是刻意設計的www

這個問題我的解決方式是將還沒下載到檔案的系所代碼存起來，然後寫shell script來跑下載器並設定執行時間上限，超過就重新執行，第二次以後讀的系所代碼列表就會是retry的列表，一直跑到全部都跑完為止，然後再執行node寫的那個爬蟲

目前測試完整流整ok，然後設定每個禮拜更新一次資料庫這樣