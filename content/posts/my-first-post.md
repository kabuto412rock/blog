---
title: "[Debug] 學Hugo並發布到Github page"
date: 2021-11-27T23:03:49+08:00
draft: false
---

# 用Hugo寫部落格

## 第一次啟動出狀況
本來想說照著官方文件用一般的hugo穩定版就好，因為目前還沒有想用SCSS調整Blog的版面，結果啟動伺服器得到以下結果：
```bash
hugo v0.89.4 linux/amd64 BuildDate=2021-11-17T14:49:26Z
Error: Error building site: TOCSS: failed to transform "ananke/css/main.css" (text/css). Check your Hugo installation; you need the extended version to build SCSS/SASS.: this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information
Built in 14 ms
```
因為我是在Ubuntu的環境下用 ```snap install hugo```安裝Hugo，
但因為官方文件預設推薦的Theme是需要SCSS/SASS的支援，所以才會出現這樣的錯誤。

如果要重來，可以先刪除在重裝：
```bash
snap remove hugo
snap install hugo --channel=extended
```
或是使用官方提供的方法切換版本：
```bash
snap refresh hugo --channel=extended
```

基本上遇到的問題也只有上面的版本問題，
接著一路照著官方教學建立文章看結果，
最後使用hugo -D 建立靜態的HTML檔案。

接著來搞定github page的設定
我參考的是[GihubPage的官方教學](
https://docs.github.com/en/pages/quickstart)

反正照著做在Github建立出一個 username.github.io的儲存庫（Repository)，
後面發布的部落格就是存在這個儲存庫。
![建立GithubPage的儲存庫]](/images/2021-12-05建立githubpage擷圖.png '建立GithubPage的儲存庫')


接下來就是有趣的地方，使用Github Action去發布你的部落格。

原始的作法是在本地直接用hugo -D指令生成新的部落格檔案到public，
在本地儲存版本再git push到Github的儲存庫。

順便紀錄一下，我做的git動作。
```bash
git init
git branch -m master main
git commit -am 'init blog'
```
如果採用Github Action則是
