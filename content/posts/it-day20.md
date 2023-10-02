---
title: "Day20 實作結算牌堆元件(四) 實作移入7牌堆的拖曳&遊戲重置"
date: 2023-09-29T03:39:17Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今天會先實作`結算牌堆`的牌要可以拖曳至`7牌堆`，
另外因為每次要重新開始都要切換頁面或按F5重新整理也會實作一個`重置`遊戲的按鈕

## 拖曳結算牌堆的牌 至 7牌堆
原本以為會花點時間想怎麼做，但實際上思考、實作都沒花多少時間就完成。

因為`結算牌堆`拖曳的牌一次只會拖曳一張，且拖曳到`7牌堆`的過程判斷基本上抄之前`7牌堆`自己的拖曳判斷方法就差不多完成，只多了一個先判斷拖曳的目標牌堆是否為`七牌堆`，甚至不用像其他牌堆拖曳`:move`還要額外去寫『拖曳成功後要觸發修改陣列』的函數`changeOption.vlaue`。

以下是對應結算牌堆`<FinishedArea >`內元件的屬性`:move`實作程式碼:
```js
// DragDemo.vue
/** 結算牌堆移動 */
function finishedCardMove(evt) {
    const to = getDomName(evt.to);
    const { futureIndex, element } = evt.draggedContext;

    let result = validNames.includes(to);
    // 只能移動至目標牌堆的最後一張牌
    result = result && futureIndex == cardStacks[to].length;

    // 檢查疊牌順序、花色是否正確
    result = result && checkNextOk(cardStacks[to], element);

    return result;
}
```
調整樣板`<FinishedArea >`的屬性`:moveCard="finishedCardMove"`即可套用上方的函數。
```vue
<!-- DragDemo.vue -->
<div class="text">結算牌堆</div>
<FinishedArea :fourCards="cardStacks" :moveCard="finishedCardMove" @doms="setFourCardDoms"
    :change="cardChange" />
```

## 實作重置遊戲的按鈕
因為遊戲初始化就是寫在`onMounted`但重設的部分不夠完整，至少並沒有考慮到`結算牌堆的部分`，舊版初始化程式碼如下:
```js
onMounted(() => {
    const data = geneateShuffleDeck(52);
    const everyIndex = [0, 1, 3, 6, 10, 15, 21, 28];
    validNames.forEach((name, idx) => {
        cardStacks[name] = data.slice(everyIndex[idx], everyIndex[idx + 1]);
    });
    cardStacks.dealerStacks = data.slice(28).map(card => ({ ...card, isOpen: true }));
});
```

所以乾脆將初始化遊戲寫成函數`resetGame()`，這樣點擊重置按鈕、渲染元件`onMounted`時都可以呼叫同個函數。
```js
// DragDemo.vue
onMounted(() => {
    resetGame();
});
```

意外發現發牌區`<DealerArea>`開牌到第幾張的狀態是包在元件內不利於初始化，決定先將狀態提升父元件的程式碼中`DragDemo.vue`，將開牌到第幾張的索引改用`props`方式傳入通知元件`<DealerArea>`要更新索引。
```vue
// DragDemo.vue
<script setup>
let dealer = reactive({ index: 0 });
</script>
<template>
    <div class="text">發牌區</div>
    <DealerArea :dealer="dealer" :deck="cardStacks.dealerStacks" :moveCard="dealerMove" />
</template>
```

為了讓`<DealerArea>`取得發牌索引的變化，在`definProps`新增屬性`dealer`對應一個`reactive物件`，當物件`dealer`參考變動時則`發牌區`的索引`index`會被設值 【變動後`dealer`的index】，參考下方程式碼:
```js
// DealerArea.vue
const props = defineProps(
    {
        dealer: {
            type: Object,
            default: reactive({ index: 0 })
        },
        // ...略
    }
)
const index = ref(0);
watch(() => props.dealer, (newDealer) => {
    index.value = newDealer.index;
});
```
> 發牌區的開牌索引是`const index = ref(0);`宣告維持不變，讓`<DealerArea>`原本的`index`等操作程式碼不用調整。

回到**遊戲重置**的部分關於函數`resetGame`的實作，只需要比原本的`onMount`重設多上**重設開牌索引**、**重設結算牌堆的資料**，可以注意到變數`dealer`直接被賦予新物件`{ index : 0 }`這樣才可以刺激子元件`<DealerArea>`的**watch**觀測到變化進而調整**元件內部的開牌索引**。
```js
// DragDemo.vue
function resetGame() {
    const data = geneateShuffleDeck(52); // 洗亂的52張牌
    const everyIndex = [0, 1, 3, 6, 10, 15, 21, 28]// 7牌堆每個牌堆的起始index
    validNames.forEach((name, idx) => {
        cardStacks[name] = data.slice(everyIndex[idx], everyIndex[idx + 1]);
    });
    // 發牌區
    cardStacks.dealerStacks = data.slice(28).map(card => ({ ...card, isOpen: true }));
    // 結算牌堆區
    FOUR_SUITS.forEach(name => {
        cardStacks[name] = [];
    });
    dealer = { index: 0 };
}
```

## 小結
今天原本只要完善**結算牌堆**拖曳，但有些時間就補上**遊戲重置**的機制。

明日預計讓接龍顯示**分數**、**遊玩時間**。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day20
{{< youtube DVzhzKr8Sj4 >}}