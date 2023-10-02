---
title: "Day 17 實作結算牌堆元件(一)樣板&資料結構"
date: 2023-09-26T03:54:14Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今日要製作`結算牌堆`，跟中間的7疊牌不同，`結算牌堆`只有4堆且只能允許同花色疊在上面，必須由小到大(A->K)放上去，放上去的牌會擋住底下的牌。

![結算牌堆示意圖](/images/20230926結算牌堆示意圖.png)

## 修正發牌區拖曳Bug
在開發今日功能前，測試~~遊玩~~`發牌區`到中間七牌堆發現能任意插到七牌堆的中間，
在函數`dealerMove`中補上`evt.draggedContext.futureIndex == cardStacks[to].length`的判斷後才正常，以下是修正後的程式碼:
```js
/** 發牌區移動 */
function dealerMove(evt) {
    const to = getDomName(evt.to);
    const dealerCard = evt.draggedContext.element;
    // 只能移動至目標牌堆的最後一張牌
    let result = futureIndex == cardStacks[to].length;

    // 檢查疊牌順序、花色是否正確
    result = result && checkNextOk(cardStacks[to], dealerCard);
    if (result) {
        changeOption.value = () => {
            cardStacks.dealerStacks = cardStacks.dealerStacks.filter(card => card.value !== dealerCard.value);
            changeOption.value = null;
        };
    }
    return result;
}
```

## 製作結算牌堆樣板
看一下草稿圖，接著想像畫面應該會有四個長方塊並排，四個長方都有個底圖對應花色，
底圖上方都可以擺一張牌。

跟製作發牌區`<DealerArea>`相同，打算先做出元件`<FinishedArea>`在整合進原本的接龍區塊，
以下是目前的樣板程式碼:
```vue
// FinishedArea.vue
<template>
    <div style="display: flex;">
        <div class="card club"></div>
        <div class="card diamond"></div>
        <div class="card heart"></div>
        <div class="card spade"></div>
    </div>
</template>
```

補上拖曳的樣板前，這部分會需要考慮四個牌堆的資料，
所以資料結構就設計成四個花色各自對應1個Card陣列的狀態
> 為了之後測試方便，僅先將各花色A放入對應陣列。
```js
// FinishedArea.vue
const fourCards = reactive({
    club: [], // 梅花
    diamond: [],// 方塊
    heart: [], // 紅心
    spade: [], // 黑桃
});
onMounted(() => {
    const cards = geneateDeck(52, true);
    fourCards.club = cards.slice(0, 1);
    fourCards.diamond = cards.slice(13, 14);
    fourCards.heart = cards.slice(26, 27);
    fourCards.spade = cards.slice(39, 40);
})
```
接著將樣板調整成有添加`draggable`元件，因為沒有添加`:move`屬性所以預設同個group的`<draggable>`內的元素是可以互相堆疊。
```vue
// FinishedArea.vue
<template>
    <div style="display: flex;">
        <div class="card club">
            <draggable :list="fourCards.club" group="pokers" itemKey="value" class="drag-cards">
                <template #item="{ element, index }">
                    <Card :value="element.value" :isOpen="element.isOpen" />
                </template>
            </draggable>
        </div>
        <div class="card diamond">
            <draggable :list="fourCards.diamond" group="pokers" itemKey="value" class="drag-cards">
                <template #item="{ element, index }">
                    <Card :value="element.value" :isOpen="element.isOpen" />
                </template>
            </draggable>
        </div>
        <div class="card heart">
            <draggable :list="fourCards.heart" group="pokers" itemKey="value" class="drag-cards">
                <template #item="{ element, index }">
                    <Card :value="element.value" :isOpen="element.isOpen" />
                </template>
            </draggable>
        </div>
        <div class="card spade">
            <draggable :list="fourCards.spade" group="pokers" itemKey="value" class="drag-cards">
                <template #item="{ element, index }">
                    <Card :value="element.value" :isOpen="element.isOpen" />
                </template>
            </draggable>
        </div>
    </div>
</template>
```

上方有添加`class="drag-cards"`在`<draggable>`上面是為了讓無拖曳元素的牌堆區可以自動撐到最高最寬，
因為預設拖曳目標的有效空間就是`<draggable>`元件本身的寬高，所以才特地補這個CSS。
```css
/* FinishedArea.vue */
.drag-cards {
    width: 100%;
    height: 100%;
}
```

## 小結
今天先定義`結算牌堆<FinishedArea>`的樣板、資料結構，明日再來嘗試整合進接龍頁面，研究要如何相容不同的疊牌拖曳規則😊

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day17
{{< youtube fs6n5MZqg0c >}}