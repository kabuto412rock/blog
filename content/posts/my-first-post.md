---
title: "[非教學] 學Hugo並發布到Github page"
date: 2021-11-27T23:03:49+08:00
draft: false
---

# 用Hugo寫部落格

## 第一次啟動出狀況
本來想說照著官方文件用一般的hugo穩定版就好，所以重新用snap安裝從extended切回穩定版，
結果啟動伺服器得到以下結果：
```bash
hugo v0.89.4 linux/amd64 BuildDate=2021-11-17T14:49:26Z
Error: Error building site: TOCSS: failed to transform "ananke/css/main.css" (text/css). Check your Hugo installation; you need the extended version to build SCSS/SASS.: this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information
Built in 14 ms
```
因為我是在Ubuntu的環境下用 ```snap install hugo```安裝Hugo，
懷疑是我用extended版產生部落格，後面又重新切換原本的穩定版，
但原本的部落格資料需要SCSS/SASS的支援所以才會出現這樣的錯誤，
喜歡多嘗試不同的操作，這是個人的壞習慣戒不掉啊。

因為懶得重新生成檔案，乾脆就用SCSS版本的Hugo：
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
當前的git push無法直接合併，因為git自動合併至少需要前面的commit是可以對得上的。

### 所以第一件事就是把遠端的分支差異移到本地端
```bash
git fetch # 
```
執行git fetch後的結果：
![git fetch後的結果](/images/2021-12-05gitfetch後的結果.png 'git fetch後的結果')
可以注意到本地端多了一個斷掉的分支(Branch)叫做origin/main，
且Commit的時間在比本地端的main還早，因為那是之前在Github上手動建時產生的Commit。

### 合併branch在推回

如果採用Github Action則是
