---
title: "Day 02 調整css調整桌面&產生52張紙牌"
date: 2023-09-11T11:36:54Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

## 動工前的準備
因為昨天標題在畫面縮放的情況下會擋住放卡牌的地方，所以早上就先看CSS相關的網站學習並且如何在Vue專案中使用。

意外發現原來App.vue檔(放置卡牌區的元件)在`<style scoped></style>`定義的class雖然會套用到App.vue的`<template>`的元件中，但卻不影響App.vue內引入的GameBoard.vue`<template>`內使用相同class名稱的元素

## 修正重疊問題
關於排版的部分身為前端菜雞，就現學現賣使用將元件內部**flex-wrap: wrap**將包裹**標題**和**卡牌區**的main元素進行調整，如下所示:
```css
main {
  display: flex;
  flex-wrap: wrap;
}
```

另外並且避免標題的狐狸圖太大，直接對包裹狐狸圖的header元素進行以下的配置，
採用 **overflow: hidden;** 這可以避免header內的元素超出限制範圍的部分進行隱藏。
```css
header {
    padding: 1rem;
    display: flex;
    align-items: center;
    width: fit-content;
    max-height: 100px;
    text-align: center;
    overflow: hidden;
}
```

## 顯示卡牌在牌桌
處理完之後想到今天好像約定要怎麼將卡牌一張張顯示在畫面，但腦袋不知道為什麼想到製作卡牌花色，
所以就使用線上工具vectr製作卡牌花色分別製作成svg檔案
https://vectr.com/design/editor/1dd3ff02-4dc1-4de6-84bb-17b9446450b8

因為我昨天刻的桌子似乎不太適合顯示一堆卡牌，
馬上詢問ChatGPT『能用HTML和CSS做出撲克牌桌的樣子嗎?』

複製得到的回應(HTML和CSS)渲染在瀏覽器上看看，發現它的作法花色其實是用字元表示。
我就放棄顯示svg在div元素上的想法，還有渲染出來的牌桌顏色、卡牌的外框也都採用ChatGPT的作法。

## 卡牌元件
每一張卡元件都會攜帶的資訊包含一個數字，使用0~51依序去表示梅花A至黑桃K。
另外使用布林值表示當前狀態是開牌/蓋牌，如下所示:
```js
// Card.vue
const props = defineProps({
    value: Number,
    isOpen: Boolean
});
```

另外也學到如果要在`<script setup>`中拿取屬性資料 defineProps() 需要先存在自訂義的變數，不能像  `<template>` 那麼自由直接取值 ，原本還想說怎麼白畫面且開發人員工具一直報Error跟我說『value is not defined』
```js
// Card.vue
const pokerValue = props.value;
// 對應撲克花色符號，Ex: ♣A
const content = PokerValuesMap[pokerValue].content;
// 對應撲克顏色class
const numberClass = PokerValuesMap[pokerValue].isRed ? 'card card-red' : 'card';
```

卡片也實際使用到 **:class** 去決定要顯示指定的class，畢竟紅心/梅花是套用紅色。
Card.vue的樣板很簡單是因為把一些轉換顏色物件的部分移到我自訂義的**uitls/constants.js**進行map對照產生Card實際顯示的花色符號。
```js
// Card.vue
<template>
    <div :class="numberClass">
        <div>{{ content }}</div>
    </div>
</template>
```

使用到**v-for**去執行迴圈在GameBoard元件產生52張牌
```jsx
// GameBoard.vue
<template>
    <div class="game-board">
        <div class="card-row">
            <Card v-for="card in boardCards" key="card.value" :value="card.value" />
        </div>
    </div>
</template>
```
## 今日進度
![52張的牌桌](/images/day2-result.png)
明日預計製作**發牌**和**卡牌自動翻面**的效果

參考: 
學習 CSS 版面配置 https://zh-tw.learnlayout.com/
ChatGPT