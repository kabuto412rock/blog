---
title: "Day 01 開發遊戲前先設定目標策略"
date: 2023-09-10T07:51:40Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
# 前言
最終目標是做一個個人版的紙牌接龍，中途也會嘗試做一些延伸的紙牌遊戲。  
遊戲的畫面預計會使用我不熟悉的前端框架 Vue3，若有充足的時間我也會結合後端去做整合，
此次參賽也是為了督促自己在前端的技術進步。

原本是想單純參加自我挑戰組，
因為曾在2014參賽第9天就中斷，所以接下來想先在Day 1設定一些目標讓自己可以達成，
過了9年後的我可不想在同個地方掛彩。

## 中短期目標
目前預計想完成的目標如下:
- 前期目標
  - 可以玩牌的畫面
  - 產生紙牌
  - 拖曳紙牌
  - 洗牌
- 中期目標
  - 紙牌動畫移動
  - 紙牌翻轉
  - 使用前面的技術實現心臟病遊戲
- 後期目標
  - 紙牌接龍  

## 今日目標-建好專案
1. 建立Vue3專案並啟動成功
```bash 
npm create vue@latest
cd ithelp-pokergame && npm run dev
```
![begin-project](/images/2023-09-10begin-project.png)
2. 在新專案中建好git repo
```bash
git init
git commit -am 'initial'
```
3. 在Github建好一個新的[儲存庫 ithelp-pokergame](https://github.com/kabuto412rock/ithelp-pokergame)
4. 將本地的git庫跟Github的庫連上對應 
```bash
git remote add origin https://github.com/kabuto412rock/ithelp-pokergame.git
```
5. 決定聽從Github老大的main分支選擇，後來又比較奇怪的方式推上Github的Repo過程
```bash
# 先在本地建一個main分支
git checkout -b main 
# 砍了本地原始master分支
git branch -D master
# 嘗試拖拉儲存庫
git pull
# 設定本地main和遠端main分支對應
git branch --set-upstream-to=origin/main main
# 乾脆直接把本地分支全部強推上遠端
git push -f
```
6. 自己使用平板畫了一隻小狐狸，也替換到了畫面上，但畫面徹底跑版...
   照理來說狐狸圖應該要上下對齊才對!
![vuePagePicture with foxy](/images/day1-result.png)

## 小結
記錄好自己的目標後，Vue建立專案和元件的方式跟React很像，
後面git的步驟只是記錄我是雷包，還好是因為空專案才可以放心執行**git push -f**

明日預計來研究CSS怎麼排版...太久沒碰都忘記了QwQ  
還要想怎麼顯示一張張卡牌在畫面上，真的是毫無信心，就看明天能否靠ChatGPT拯救這一切吧

## 參考
[Vue建立專案](https://vuejs.org/guide/quick-start.html)
