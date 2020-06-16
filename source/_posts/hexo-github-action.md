---
title: "使用 GitHub Action 自動部屬 Hexo GitHub Page"
date: 2020-06-16 08:39:34
tags: [Hexo, web, GitHub, CI/CD]
categories: web
---

GitHub 在去年 11 月的時候發布了自家的 CI/CD 工具 GitHub Action 並提供 public repo 無限額度的免費使用，在過去雖然有整合 Travis CI 但設定起來還是有點麻煩，GitHub Action 強大的地方在於他的 Market place 功能，可以像安裝套件一樣直接使用別人寫好的 CI/CD Script。

以往在發部落格文章的時候都要自己手動下指令 `hexo g -d`，雖然已經夠簡便了，但人的懶惰是沒有極限的，因此這次就來嘗試使用看看這個 GitHub 的新功能來做自動部屬，在設定 Action 之前記得先將你的 hexo 的 git deploy 設定好，以下都假設你已經設定好 (就是你有辦法使用上面那個指令直接部屬你的 hexo blog)，且你知道你的 hexo 文章原始碼和靜態網站各放在哪個 repo 的 branch。

我從 Market place 找到了這個 action: [yrpang/github-actions-hexo](https://github.com/marketplace/actions/hexo-github-action)，看起來還行，而且支援自動 purge Cloudflare cache (雖然我沒設定)，就來試試看吧。

GitHub Action 的設定相當簡單，在你的文章原始碼的 repository 底下新增一個資料夾 `.github/workflows`，並加入檔案 `main.yml`，把 CI/CD 的設定檔寫進去就可以了，可參考我的 template

```yml
name: Hexo Build and Deploy

on: 
  push:
    branches: [source]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v1
        with:
          ref: source
      - name: Cache node modules
        uses: actions/cache@v1
        with:
          path: node_modules
          key: ${{runner.OS}}-${{hashFiles('**/package-lock.json')}}
      - uses: yrpang/github-actions-hexo@master
        with:
          deploykey: ${{secrets.DEPLOY_KEY}}
          username: USERNAME
          email: EMAIL
```

注意一下 `on: push:` 那邊和 `steps` 裡面的 Checkout 的部分的 branch 設定，我是將原始碼和產生的靜態網站檔案放在同一個 repo，原始碼在 `source`，網站在 `master` (如果不是在 master 的人請看文末的其他參數介紹)，Checkout 是要 checkout 到你的原始碼的 bracnh，而 `on: push:` 是設定哪一個 brach 的 push event 會觸發自動部屬，請大家把兩個 branch 都改成自己放 hexo 文章原始碼的 branch。

最下面可以看到該 action 有三個必要的參數，其中 username 和 email 是 git commit log 裡面的 name 跟 email，這部分其實不是很重要，而且這個 script 有一個小問題讓這兩個參數更不重要，待會會提，但總之如果你想讓你網站那邊的 commit 顯示正常的話就設定成你的 github username 跟 email 吧。

而 deploykey 是 `hexo deploy` 的時候用的 ssh key，設定方法如下

先在你的 commandline 產生 rsa key pair
```sh
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f hexo -N ""
```

產生成功應該會得到兩個檔案 `hexo` 和 `hexo.pub`，將 `hexo.pub` 的內容新增到靜態網站的 GitHub repo 設定中的 Deploy Keys 中 (Settings -> Deploy Keys -> Add deploy key)，title 隨意，但 Allow write access 一定要勾選，否則這個 ssh key 沒有辦法 push

![](https://imgur.com/76Ggz8v.png)

將 `hexo` 的內容新增到和要執行這個 action 相同的 GitHub repo 設定中的 Secret 中 (Settings -> Secrets -> New scret)，名稱 `DEPLOY_KEY` (或任何名稱，但記得去 `.yml` 裏修改成對應的 secret 名)

![](https://imgur.com/F2Raxfn.png)

因為我原始碼和網站在同一個 repository，所以我兩個設定都在同一個 repo 裡，如果兩個是分開的人小心不要搞錯。

完成後把該設定檔 push 上去就可以了，當你 `on: push:` 設定的 branch 有 push event 發生時就會觸發自動部屬，可以到 GitHub repo 裡面的 `Actions` tab 查看是否有部屬成功。

![](https://imgur.com/nrNObjb.png)

### 其他參數

| Name                 | Type    | Required | Default  | Description                                                             |
|----------------------|---------|----------|----------|-------------------------------------------------------------------------|
| deploykey            | secrets | **Yes**  |          | The deploy key of your GitHub Page repository                           |
| username             | string  | **Yes**  |          | Your user name                                                          |
| email                | string  | **Yes**  |          | Your email address                                                      |
| if_update_files      | boolean |          | false    | Whether update the source file after generate                           |
| github_token         | secrets |          |          | Token for the repo. Can be passed in using $\{{ secrets.GITHUB_TOKEN }} |
| branch               | string  |          | 'master' | The branch of the blog source code                                      |
| if_update_cloudflare | boolean |          | false    | Whether update cloudflare                                               |
| cloudflare_zone      | string  |          |          | the cloudflare zone                                                     |
| cloudflare_token     | secrets |          |          | Your cloudflare token                                                   |

比較值得注意的是 `branch`，如果你放靜態網站的 branch 不是 `master` (例如 `gh-pages`)，可以透過這個參數設定。

如果有需要 Clouldflare flush 的人可以設定最後那三個參數，你需要知道你的 zone id 跟 cloudflare token，token 記得要使用 secrets 來設定，設定方法和上面設定 ssh key 設定 private key 的部份一樣。

### 小問題
這個 action 有個小問題就是你的靜態網站的 repo 的 commit history 會被洗掉，`hexo deploy` 會把產生的靜態檔案複製到 repo 目錄底下的 `.deploy_git` 並 commit，預設這個資料夾是不會被 stage 的，解決方法是把這個資料夾從 `.gitignore` 裡面拿掉，或者是修改 action 的 script，讓他每次 deploy 前先把靜態網站的 repo clone 至 `.deploy_git` 資料夾內，後面這個方案可以參考[這篇文章](https://depp.wang/2020/02/17/use-github-actions-to-achieve-hexo-blog-auto-deploy/)。

我是不在意那邊的 commit history 所以沒有設定。

### 小結
感恩 GitHub，讚嘆 GitHub。