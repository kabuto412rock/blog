---
title: "[非教學] 學Hugo並發布到Github page"
date: 2021-11-27T23:03:49+08:00
draft: false
---

# 前言
如果你正在尋找一篇Hugo建立GithubPag的教學文，請你繼續搜尋其他文章，
因為這是我第一次使用Hugo跌跌撞撞的除錯紀錄，
而大部分問題都是我自己不照教學走導致需要Debug的文章，
但如果想知道怎麼出錯解錯就繼續看下去，但不建議跟著做下去。

## Bug1 前幾次hugo server -D啟動正常，後來卻出錯?
本來想說安裝extended版本比較好，所以重新用snap安裝穩定版，
結果啟動伺服器得到以下結果：
```bash
hugo v0.89.4 linux/amd64 BuildDate=2021-11-17T14:49:26Z
Error: Error building site: TOCSS: failed to transform "ananke/css/main.css" (text/css). Check your Hugo installation; you need the extended version to build SCSS/SASS.: this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information
Built in 14 ms
```
我是在Ubuntu的環境下用 ```snap install hugo```安裝Hugo，
而錯誤的原因是用extended版產生部落格，後面又重新切換Hugo穩定版，
但原本的部落格資料需要SCSS/SASS的支援所以才會出現這樣的錯誤。

## 解法
因為懶得重新生成檔案，乾脆就用SCSS版本的Hugo：
```bash
snap remove hugo
snap install hugo --channel=extended
```
或是使用官方提供的方法切換版本：
```bash
snap refresh hugo --channel=extended
```

## 產生一個GithubPage的儲存庫
基本上遇到的問題也只有上面的版本問題，
接著一路照著官方教學建立文章看結果，
最後使用hugo -D 建立靜態的HTML檔案，確認public資料夾是否產生資料。

接著來搞定github page的設定
我參考的是[GihubPage的官方教學](
https://docs.github.com/en/pages/quickstart)

反正照著做在Github建立出一個 username.github.io的儲存庫（Repository)，
後面發布的部落格就是存在這個儲存庫。
![建立GithubPage的儲存庫](/images/2021-12-05建立githubpage擷圖.png '建立GithubPage的儲存庫')

接著準備將本地端內容更新到GithubPage的儲存庫。
```bash
git init # 在本地端建一個Git儲存庫
git branch -m master main # 把本地預設的分支名稱從master改成main
git commit -am 'init blog'# 把本地端所有檔案都加入版本管理，並產生一個Git commit紀錄。
git remote add origin https://github.com/你的Github帳號/你的Github帳號.github.io.git
git push origin main #把本地儲存庫的main分支版本 -> 遠端origin這個儲存庫
```

### 但應該100%會出現以下這個錯誤
![無法gitpush](/images/2021-12-05無法直接gitpush.png '無法gitpush因為push無法直接合併')
當前的git push無法直接合併，因為git自動合併至少需要前面的commit是可以對得上的

## Bug2 所以第一件事就是把遠端的分支差異移到本地端
先說明一下底下的指令將會導致一些問題發生，因為這篇文章是篇Debug文章
```bash
git fetch # 
```
執行git fetch後的結果：
![git fetch後的結果](/images/2021-12-05gitfetch後的結果.png 'git fetch後的結果')
可以注意到本地端多了一個斷掉的分支(Branch)叫做origin/main，
且Commit的時間在比本地端的main還早，因為那是之前在Github上手動建時產生的Commit。 

但這時候如果直接下git merge origin/main會出現fatal: 拒絕合併無關的歷史，
我怎麼知道...因為我試過...早知道乖乖用git pull的方式就好。

## 挽救解法
重新git pull遠端，然後以rebase的方式解決分支衝突問題
```bash
git pull origin main --rebase --allow-unrelated-histories
```
可以注意到main和origin/main終於合併成一條線
![重新以git pull方式重做](/images/2021-12-05成功的gitmerge了.png '重新以git pull方式重做')

終於可以成功git push到遠端
```bash
git push origin main #將在本地端合併好的main分支，重新推上遠端origin
```

## 最終關卡是設定Github Action
01 在blog資料夾開一個檔案
.github/workflows/gh-pages.yml 
包含以下內容：
```bash
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

02 先讓git版本管理，重新
```bash
git add .
git commit -am '成功建立Github Action的設定檔'
git push
```