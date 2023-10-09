---
title: "Day30 使用Cloudflare部屬Vue靜態網站"
date: 2023-10-09T07:24:04Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

目前撲克牌遊戲網站都是在自己的電腦用`npm run dev`啟動，今天會介紹如何部屬`Vue專案`到`CloudFlare`提供對外連線的網站，操作有誤的地方再請多多指教。

## 使用Github Aciton建立
以下的操作需要事先註冊`Cloudflare`的帳號 和`GitHub`的帳號

### 第0步 建立CloudFlare Page的Project
我是參考這個YouTube影片學習如何建立`Cloudflare Page`專案，
但跟影片不同的部分我在**1:56**我是選**only-select-repositories**選擇單一Repo的權限並命名專案名稱`ithelp-game-test`。
> 影片看到5分鐘的時候，`Cloudflare`網站本身停止回應，後面的步驟就沒跟著影片教學做，後面第1~3步驟是我一步步看文件試出來的並非完全照官方建議走，因為需按專案本身調整。
{{< youtube MTc2CTYoszY  >}}

### 第1步 建立 Cloudflare API Token
參考[Cloudflare Pages GitHub Action說明](https://github.com/marketplace/actions/cloudflare-pages-github-action)以下只是我再額外自己截圖實作的步驟。

1. 登入帳號來到`Cloudflare`的儀表板，先點選左下角的`Workers & Pages` 
2. 接著點擊右手邊連結`Manage API tokens`進入管理`API Token`的頁面
![Alt text](/images/day30-step2.png)
3. 點擊藍色按鈕`Create Token`前往建立API Token的頁面
![Alt text](/images/day30-step3.png)
4. 來到API Tokens頁面後點選**Create Custom Token**旁的藍色按鈕`Get Started`
 ![Alt text](/images/day30-step4.png)
5. 填寫`Token name`這邊幫Token取名為**Deploy with github**
6. 在`Permisions`區塊點選`Add`新增一個權限`Account`/`Cloudflare`/`Edit` ，接著畫面拉到最下方點擊藍色按鈕`Continue to summary`。
![新增APIToken](/images/day30-step5.png)
7. 此時`Cloudflare`會讓你再次確認權限，只需要注意畫面上有出現`All accounts - Cloudflare Pages:Edit`這個，沒問題就繼續點擊藍色按鈕`Create Token`。
![Alt text](/images/day30-step7.png)
8. 至此`Cloudflare API Token`建立成功，點擊按鈕`Copy`先複製起來 
![點擊Copy按鈕複製](/images/day30-step8.png)

### 第2步 將Cloudflare API Token設置於Github Repo
> 此處Github儲存庫是[kabuto412rock/ithelp-pokergame](https://github.com/kabuto412rock/ithelp-pokergame)
1. 前往`Github`的儲存庫點擊`Setting`->`Secrets and Variables`->`Actions`
2. 來到`Actions secrets and variables`頁面後點擊綠色按鈕`New Repository secret`
   - 設置新的密鑰`CLOUDFLARE_API_TOKEN` 填入先前取得的**Cloudflare API Token**
   - 設置新的密鑰`CLOUDFLARE_ACCOUNT_ID`填入在Cloudflare的儀表板右手邊**Manage API tokens**上方的**Account ID**
  ![Alt text](/images/day30-setsecret.png)
3. 在分支`main`加入檔案`.github/workflows/publish.yml`定義Github Action
> 這邊要注意的是 `projectName`對應填寫的專案名稱`ithelp-game-test`是我在第0步已經預先建立的專案名稱，若亂填的話Github Action自動執行時將會在步驟**Publish to Cloudflare Pages**出錯。
```yaml
on: [push]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
    name: Publish to Cloudflare Pages
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      # Run a build step here if your project requires
      - name: Install and Build 🔧 # This example project is built using npm and outputs the result to the 'build' folder. Replace with the commands required to build your project, or remove this step entirely if your site is pre-built.
        run: |
          npm ci
          npm run build

      - name: Publish to Cloudflare Pages
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: ithelp-game-test
          directory: dist
          # Optional: Enable this if you want to have GitHub Deployments triggered
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}
          # Optional: Switch what branch you are publishing to.
          # By default this will be the branch which triggered this workflow
          branch: main
          # Optional: Change the Wrangler version, allows you to point to a specific version or a tag such as `beta`
          wranglerVersion: '3'
```

### 第3步 在Cloudflare儀表板確認成功部屬
回到儀表板後可以注意到專案`ithelp-game-test`出現 **visit site**，就代表Github Action已經正常部屬上環境囉。
> 部屬成功的網頁 https://ithelp-game-test.pages.dev
![在Cloudflare確認成功部屬](/images/day30-checkok.png)


## 30天小結
今天是我參加鐵人賽的第30天最後使用`Cloudflare Page`搭配`Github Action`把網頁發布出去並寫成一篇文章。

這30天對比他人的文章的知識含量遠遠不夠而且賽前又沒屯稿，中途也曾苦悶想要放棄比賽，幸好堅持寫文章不中斷，或許這就是一種鐵人精神吧!   

『在學習和創造之間不斷翻滾成長又分享過程給大家』

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day30
{{< youtube XpXum_AnOrM  >}} 

**參考文件**
- https://www.youtube.com/watch?v=MTc2CTYoszY
- https://github.com/marketplace/actions/cloudflare-pages-github-action
- https://developers.cloudflare.com/pages/platform/direct-upload/