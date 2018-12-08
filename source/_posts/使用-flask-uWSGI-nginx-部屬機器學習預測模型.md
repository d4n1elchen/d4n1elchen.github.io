---
title: 使用 flask + uWSGI + nginx 部屬機器學習預測模型
date: 2018-12-08 11:31:55
tags: [Web, Machine Learning]
categories: Web
---

最近受朋友邀請參與創業團隊負責機器學習演算法模型部屬，這個團隊最早是從課堂專題的成員將專題內容帶出來創業的，因此過去 DEMO 的 Code 大部分都是在 Jupyter 上做出來的，我的工作就是將這些演算法服務化並部屬。

演算法從研發到上線大概會經過以下步驟

1. 離線演算法開發、測試
2. 定義輸入輸出
3. 服務化
4. 部屬到線上環境

本教學著重在服務化和上線而非機器學習演算法，所以不包含第一個步驟，我們會用個簡單的演算法作為範例，簡單到不行：接收一個浮點數，乘上 $\pi$ 之後計算其二次根號值；實際上應用只要把這個演算法替換成其他已經開發好的演算法就行了。

我們先創建一個資料夾 `model-server`
```shell
$ mkdir model-server
```

實作演算法

`model-server/algo.py`
```python
import math
a = 25
res = math.sqrt(a*math.pi)
```

如果是機器學習的模型的話，演算法可能會相當複雜，以圖像辨識來說，會先經過前處理、推論、後處理等，最後的輸出可能是辨識的類別。

這份教學將以上面開根號的例子為核心演算法，解釋要如何將模型包成服務、並將服務部屬到線上環境中。

此外線上環境可能會考量到運算時間，希望硬體能有效利用，降低反應時間。文末會介紹如何用 uWSGI 來做負載平衡。

我的做法主要參考[這裡](https://hackernoon.com/a-guide-to-scaling-machine-learning-models-in-production-aa8831163846)。

定義輸入輸出
---

我們首先要將你的演算法的輸入輸出定義出來，以開根號的例子來說的話，輸入為一個浮點數，輸出也是一個浮點數，藉此我們將上述演算法寫成一個 function。

`model-server/algo.py`
```python
import math

def pi_sqrt(a):
    return math.sqrt(a*math.pi)
```

如果是影像辨識的話，輸入會是一張圖片，在 Python 裡可能會是一個 PIL 的物件、Numpy 矩陣、或其他可能的格式，輸出則是類別的編號或是類別的名字，取決於專案的需求。

```python
def predict(im):
    # preprocessing ...
    cls_id = resnet(im_preprocessed)
    return id_to_text(cls_id)
```

服務化
---

顧名思義我們要將演算法包裝成網路服務，你可以選擇不同的通訊協定來做到這件事，最常用的是 HTTP，我們這邊用 [flask](http://flask.pocoo.org/) 來做 HTTP Server。

### 安裝方法
```shell
$ pip install Flask
```

### 基本使用
`model-server/server.py`
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```
```shell
$ python server.py
```

用瀏覽器開啟 http://localhost:5000/ ，可以看到熟悉的 `Hello World!` 就算是成功囉！

這個範例啟動了一個基本的 HTTP Server，只有一個路徑 `/` 也就是 `http://localhost:5000/`，監聽 `/` 的 GET Request 並回傳 `Hello World!`。

可以透過 `@app.route()` 來定義新的路徑，例如：

```python
@app.route('/predict')
def predict():
    return 'This route may predict something'
```

重新啟動後訪問 http://localhost:5000/predict 可以看到上面 return 的那行字。

如果未指定的話預設只接收 GET Request，如果想要用 POST Request 的話可以在 `@app.route()` 中指定

```python
@app.route('/predict', methods=['POST'])
def predict():
    return 'This route may predict something'
```

### 接收參數

HTTP 有兩種傳 Request 資料的方式，一是 query string，二是 HTTP message body，前者可以透過 request url 來傳輸資料，後者則需要將資料帶在 HTTP 封包中的 body 欄位，可以是純文字或 json 或其他支援的格式。

這邊因為我們的情境較簡單所以使用 query string，可以透過 `request.args.get()` 來存取，記得 import request 物件

```python
from flask import request

@app.route('/pisqrt', methods=['GET'])
def pisqrt():
    a = request.args.get('a')
    return 'This route will compute sqrt(pi*{})'.format(a)
```

訪問 http://localhost:5000/pisqrt?a=25 會得到 `This route will compute sqrt(pi*25)`。

### 處理輸入參數並將計算結果輸出

因為接收到的參數一律是字串格式，根據需求需要做一些處理，然後丟給一開始寫好的演算法處理。回傳一般會採用 JSON 格式，或要直接將結果回傳也可以，依需求而定。

最後 `server.py` 完整的內容

`model-server/server.py`
```python
from flask import Flask
from flask import request
from flask import jsonify
import algo # algo.py

app = Flask(__name__)

@app.route('/pisqrt', methods=['GET'])
def pisqrt():
    a = request.args.get('a')
    a = float(a)
    res = algo.pi_sqrt(a)
    return jsonify({'result': str(res)})

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

執行
```shell
$ python server.py
```

若訪問 http://localhost:5000/pisqrt?a=25 (或 a 可帶入任何數字) 得到以下回應
```json
{
    "result": "8.86226925452758"
}
```
表示你成功將演算法包成服務了

部屬到線上環境
---

將服務開發好之後最後一步就是部屬到上線的環境中，首先你需要能夠存取欲作為上線環境的系統存取權限。將服務部屬的方式有數種，基本想法就是複製開發環境過去，然後讓伺服器能夠從公網存取。

因為我們的專案中使用到 Python 和相關套件，首先需要在伺服器中安裝 Python 及專案中有用到的相關套件，可以用 pip freeze 或 anaconda 來做到這件事。

再來需要將專案檔案傳上伺服器，可以將整個專案資料夾打包壓縮後上傳再解壓，或是如果有 git server 可以直接用 git 來部屬。以下假設專案在上線伺服器中的路徑為 `/home/ubuntu/model-server/`

最後是啟動服務，可直接執行 `python server.py` 來啟動，或是用 `systemd` 來設定開機自動啟動，以下是 `systemd` 的範例設定

在 `/home/ubuntu/model-server/server.py` 中加入 shebang (#!)
```python
#!/usr/bin/env python

from flask import Flask
from flask import request
from flask import jsonify
import algo # algo.py

app = Flask(__name__)

@app.route('/pisqrt', methods=['GET'])
def pisqrt():
    a = request.args.get('a')
    a = float(a)
    res = algo.pi_sqrt(a)
    return jsonify({'result': str(res)})

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```
給 `/home/ubuntu/model-server/server.py` 執行權限
```shell
$ chmod +x ~/model-server/server.py
```

`/etc/systemd/system/modelserver.service`
```
[Unit]
Description=Model Server
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/model-server/   # 使用絕對路徑
ExecStart=/home/ubuntu/model-server/server.py # 使用絕對路徑
Restart=on-failure
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
```shell
# systemctl enable modelserver
# systemctl start modelserver
```

可用
```shell
# systemctl status modelserver
```
來觀察執行狀況

成功啟動後可以訪問 http://[ip-to-your-server]:5000/pisqrt?a=25 來測試看看是否成功。

使用 uWSGI + nginx 做負載平衡
---
uwsgi 是一套 Web Server (如 nginx) 與 Web Application (如 flask) 之間的通信協定，可將 Web Server 與 Web Application 分離，讓 load balance、proxy 等網路管理交給 Web Server，在應用端只要管接到請求後怎麼處理資料就好。想更深入了解可以參考[這篇](https://hk.saowen.com/a/5eefef9f3a3bbc08f81e99a6bdc4106ba15df92aaebc93378afeb87836abd499)。

而 uWSGI 是一套實作了 uwsgi、http 等協定的軟體，你可以直接拿 uWSGI 來當 http server，或是用 nginx 架 http server 並將請求透過 uwsgi 介面 proxy 到 uWSGI。

這邊一堆看起來很像只差大小寫的東西，看不懂不用太擔心，你只要知道三件事

1. uWSGI 是一套網頁伺服器軟體
2. uWSGI 可以作為 middleware 把你的 service 包裝起來，讓 nginx 可以將請求轉發給你的 service，以 nginx 做為主要的 http server 提升安全性和效能
3. uWSGI 可以讓你可以同時開多個 service 來處理多人同時請求的情境，他自動會幫你做 load balance

其中 3 是我們想要的，因為可能會面臨同時大量的使用者要進行請求，如果沒有做 load balance，所有的請求全部都塞在同一個 Process 上的話，效能會相當低落，latency 也會提高。

### uWSGI
我們先將之前開起來的服務暫停
```shell
# systemctl stop modelserver
```

安裝 uWSGI 套件
```shell
$ pip install uwsgi
```

撰寫 wsgi 進入點
`model-server/wsgi.py`
```python
from server import app

if __name__ == "__main__":
    app.run()
```

用指令啟動測試看看，`-w` 可以指定你要啟動哪個 python module (.py 檔)，並指定裡面哪個物件是 callable，以我們的例子來看是 app 這個物件，所以可以看到 `-w wsgi:app`。
```shell
$ uwsgi --socket 0.0.0.0:5000 --protocol=http -w wsgi:app
```

成功的話訪問 http://[ip-to-your-server]:5000/pisqrt?a=25 應該會得到跟剛剛一致的結果。

我們可以將 uWSGI 相關設定參數寫在設定檔中，並在啟動時將設定檔餵進去。

編輯 `model-server/uwsgi.ini`
```ini
[uwsgi]
module = server:app
socket = 0.0.0.0:5000
protocol = http
```
```shell
$ uwsgi --ini uwsgi.ini
```
以上操作和前面一行指令的結果應完全一樣。

嘗試加入數個 worker 看看 (下面每次做 `uwsgi.ini` 都需要重開 uWSGI 才會生效，大家自己注意一下)

`model-server/uwsgi.ini`
```ini
[uwsgi]
module = server:app
socket = 0.0.0.0:5000
protocol = http

master = true
processes = 5
```

因為在 local 上跑的，除非去找壓力測試的軟體來玩，不然其實感受不出差意，不過應該可以看到提示現在有幾個 worker 正在運作，如果 flask 有寫 log 的話應該也會看到同時輸出 5 個 log。

### nginx

nginx 是一套網頁伺服器，和 apache 一樣，但在效能和安全性上略勝 apache 一籌，最近有許多架站教學都是以 nginx 為主。

在本專案裡面 nginx 是負責從外面接 http request，然後轉發給 uWSGI 進行工作分配。

先安裝 nginx
```shell
# apt-get install nginx
```

因為接下來 uWSGI 接收的請求不會來自外部而是來自 nginx，我們不需要 uWSGI 自己做為 http server 了，改用欲設的 uwsgi，並使用 unix socket 來監聽請求，可提升速度及安全性。

修改 `model-server/uwsgi.ini`
```ini
[uwsgi]
module = server:app

master = true
processes = 5

socket = model-server.sock
chmod-socket = 666
vacuum = true
```
```shell
$ uwsgi --ini uwsgi.ini
```
啟動 uWSGI 後會產生 `model-server.sock` 這個檔案，nginx 將透過他來轉送請求，其中 `vacuum` 參數表示 uWSGI 停止後會自動清除 socket。

編輯檔案 `/etc/nginx/sites-available/modelserver` (注意檔案權限問題，nginx 的設定檔都需要 root 權限才能編輯)
```conf
server {
    listen 5555; # 你想監聽的 port
    server_name your_domain www.your_domain; # 如果有指定網域需要設定上來，
                                             # 沒有網域的話有沒設定沒什麼差別

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/ubuntu/model-server/model-server.sock; # 使用絕對路徑
    }
}
```

啟用此 config 檔
```shell
# ln -s /etc/nginx/sites-available/modelserver /etc/nginx/sites-enabled
```

檢查設定檔正確性
```shell
# nginx -t
```

沒問題的話重啟 nginx
```shell
# systemctl restart nginx
```

接著訪問 http://[ip-to-your-server]:[port-for-nginx]/pisqrt?a=25 可以得到和剛剛一致的結果。

### systemd
我們前面設定的 systemd 是用 python 指令直接啟動 flask server，但我們現在要啟動的是 uWSGI server，因此原來的設定檔需要做一些修改

`/etc/systemd/system/modelserver.service`
```
[Unit]
Description=Model Server
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/model-server/   # 使用絕對路徑
ExecStart=/path/to/bin/uwsgi --ini uwsgi.ini  # 使用絕對路徑
Restart=on-failure
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
重新載入設定檔然後啟動
```shell
# systemctl daemon-reload
# systemctl start modelserver
```

觀察是否成功啟動
```
# systemctl status modelserver
```

如果一切正常的話可以重新啟動看看服務會不會自動啟動。

以上就是如何使用 flask + uWSGI + nginx 部屬具備負載平衡的模型預測服務，未來可能可以嘗試用 docker 來進行部屬，達到更高的 scalability 和部屬的效率。

謝謝大家，有什麼問題歡迎提問～

Ref
---
- [A Guide to Scaling Machine Learning Models in Production](https://hackernoon.com/a-guide-to-scaling-machine-learning-models-in-production-aa8831163846)
- [uWSGI Options](https://uwsgi-docs.readthedocs.io/en/latest/Options.html)
- [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04)
