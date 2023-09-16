---
title: "Day 07 卡牌垂直重疊"
date: 2023-09-15T23:48:02Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 今日完成目標
- [x] 多張卡牌實現垂直重疊，但露出非交疊的部分

## 垂直重疊
為了產生重疊的效果，個人覺得最酷的方式應該是使用CSS的`grid`排版，
所以在這之前我利用一個[GRID GARDEN](https://cssgridgarden.com/)的網站練習了一下。

原本問ChatGPT是得知用每一張卡堆疊都還要套用不同的CSS，如果要多一張牌就要多一個class，或是用sass的寫法達成，但以上這些我都不想要，一來太麻煩、二來sass還要額外引入新依賴，畢竟我只是想堆疊卡牌而已。

接著說明實際我達成的方式是靠`display: grid;`要求格狀排列，然後設定`grid-template-rows`為 `repeat(13, 3rem);`，這樣就可以讓每一張牌所在的格子都只有3rem的高度，設定13是因為我認為這樣垂直重疊排列卡牌最多只有13張，畢竟紙牌接龍不同花色暫時串再一起也就13張，在現實還是虛擬我的認知都是這樣，當然在設定比13高一點也不會影響，但如果出現第14、15張就會有卡牌不重疊，這點還請注意。

那為什麼達成重疊呢?我在容器內裝13個元件，每一個元件都只裝一張牌，且元件高度限制都在`3rem`，但實際元件
牌的高度是超過`3rem`，所以當我裝入第二張牌就會擋住第一張牌，讓第一張牌只露出`3rem`的高度，以此類推，最後一張牌則會露出全部。
```vue
// CardColumn.vue
<template >
    <div class="card-box" :class="{ 'empty-card-box': isEmpty }">
        <div class="card" style="visibility: hidden;"></div>
        <div style="visibility: visible; position: absolute; z-index: 1;">
            <div v-if="isEmpty">沒牌</div>
            <div v-else style="; display: grid; grid-template-rows: repeat(13, 3rem);">
                <Card v-for="(card, index) in cards" @click="onClick" :key="card.value" :value="card.value"
                    :isOpen="card.isOpen" />
            </div>
        </div>
    </div>
</template>
```
![success card column stack](/images/day7-card-column.png)
額外補充可以注意到card-box內第一個元素是用來稱基本的寬高，
所以第二個元素我則讓他設定`position: absolute`這樣可以讓其中的格狀排列不會受到第一個元素的影響也不會影響到外部元素。

若少掉`position: absolute`的話，會變成底下這樣:
![fail card column stack](/images/day7-without-absolute.png)

## 本日小結
原本今天是要用水平堆疊，但看了撲克才發現接龍平常都是玩垂直的...
  
至於卡牌移動則留在明天吧，因為堆疊卡牌的關係讓我意識到沒有我想像動畫的只是水平移動那麼簡單。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day07

## 參考資料
- [GRID GARDEN](https://cssgridgarden.com/)
- [CSS Grid Layout](https://developer.mozilla.org/zh-TW/docs/Web/CSS/CSS_Grid_Layout)
