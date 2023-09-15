---
title: "Day 06 實現自訂義牌堆"
date: 2023-09-15T04:08:21Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
# 今日目標
- [x] 研究如何製作牌堆

## 製作牌堆
製作`CardBox`元件負責用來擺放卡片的容器，所以結構上就只是比原本卡片元件的稍寬，所以設計成可放子元件進去的樣板，然後當沒放牌時會產生跟牌一個寬高的隱藏區塊到slot裡面。
```vue
// CardBox.vue
<template >
    <div class="card-box">
        <slot>
            <div class="card" style="visibility: hidden;"></div>
        </slot>
    </div>
</template>
```
所以目前首頁是長這樣，放了兩個空的CardBox佔位置。
```html
// HomeView.vue
<div style="display: grid; grid-template-columns: 1fr 1fr;">
    <CardBox></CardBox>
    <CardBox></CardBox>
</div>
```
![cardbox.vue demo image](/images/day06-cardbox.png)

接著思考如何實作卡牌從A點發到B點，看著畫面思考發現，
不如今天就來實作兩邊卡堆點擊後會移動到對方卡堆的功能。

這邊為了簡單驗證想法，定義函數`geneateDeck(5, true)`生成五張卡牌依序是梅花A~5(設為開牌)，
我預計實驗兩個卡堆，所以函數也就設計以下兩個非常相似的函數，
功用正是將卡堆的最後一張卡彈出，並推到另一個牌堆。
```js
// HomeView.vue
const fstCards = ref(geneateDeck(5, true));
const secondCards = ref([]);
const moveCardFromAToB = () => {
  const card = fstCards.value.pop();
  if (card === undefined) return;
  secondCards.value.push(card);
};
const moveCardFromBToA = () => {
  const card = secondCards.value.pop();
  if (card === undefined) return;
  fstCards.value.push(card);
};
```

在經過一番折騰，發現`CardBox.vue`應該要內部自帶`Card`元件，
由外面用`slot`的方式注入進去畫面反而不好寫版面，當然也有可能是我排版功力不夠，改由`props`傳入cards陣列，由外部`HomeView.vue`處理資料傳遞，但由`CardBox.vue`負責卡片的顯示。

所以可以看到以下`CardBox.vue`會判斷當前有牌沒牌渲染不同的樣式。
```vue
// CardBox.vue
<script setup>
import { computed } from 'vue';
import Card from '../components/Card.vue';
const { cards } = defineProps(['cards']);
const isEmpty = computed(() => {
    return cards.length == 0;
});
</script>
<template >
    <div class="card card-box" :class="{ 'empty-card-box': isEmpty }">
        <div v-if="isEmpty">沒牌啦</div>
        <div v-else class="card">
            <Card v-for="(card, index) in cards" :key="card.value" :value="card.value" :isOpen="card.isOpen" />
        </div>
    </div>
</template>
```

遊戲大廳`HomeView.vue`負責處理資料的部分依舊如上，但樣板改成傳簡單的卡片陣列進到`<CardBox />`
```html
// HomeView.vue
<div style="display: grid;  grid-template-columns: 1fr 1fr;">
    <div>第一卡堆: </div>
    <div>第二卡堆: </div>
    <CardBox @Click="moveCardFromAToB" :cards="fstCards" />
    <CardBox @Click="moveCardFromBToA" :cards="secondCards" />
</div>
```

## 小結
本來理想上是要有卡片飛過去特效，但至少有先完成牌堆顯示的部分，
今天開發學習進度不理想或許也卡在我想要如何顯示，而忘記先去思考資料如何改變。

預計明天會來研究該如何處理牌堆之間飛越的效果，以及牌如何實現水平重疊的部分。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day06

{{< youtube BLVe-KFV_b0 >}}

**參考**
- ChatGPT
