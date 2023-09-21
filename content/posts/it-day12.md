---
title: "Day 12 實作拖曳卡牌只能置放到目標牌堆的牌尾、蓋牌無法拖曳"
date: 2023-09-21T04:36:25Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

今天預計實作接龍紙牌的拖曳規則
- ☑️ 拖曳卡牌只能置放到目標牌堆的牌尾 -> 只能置放至目標最後
- ☑️ 牌堆只有打開的牌才能進行拖曳 -> 蓋牌無法拖曳


## 拖曳卡牌只能置放到目標牌堆的牌尾
檢查方式很簡單，首先將昨日的`:move`函數內的變數`result`判斷先改成用`let`宣告，
因為不能任意插入前面的順序，所以增加判斷只有**新目標位置**`futureIndex`等於**目標牌堆陣列的長度**`cardStacks[to].length`時才可以進行移動。

- 如果要移動到的目標牌堆沒有牌，`futureIndex`就需要等於0
- 如果要移動到目標牌堆有1張牌，`futureIndex`就需要等於1
- 總之目標牌堆有**N**張牌，`futureIndex`就要為**N**，以此類推...
### 實際程式碼
```js
function limitLocalMove(evt) {
    // 限制同個牌堆無法拖曳
    let result = evt.from !== evt.to;

    // 取得牌堆的來源、目標名稱，對應reactive`cardStacks`內的名稱
    const from = getDomName(evt.from);
    const to = getDomName(evt.to);
    const { index, futureIndex } = evt.draggedContext;

    // 只能移動目標牌堆的最後一張牌
    result = result && futureIndex == cardStacks[to].length;
    
    if (result) {
        // 中間略
    }
    return result;
}
```

## 牌堆只有打開的牌才能進行拖曳
先初始化改為39張牌均分給3個牌堆且將所有牌都闔上，
這樣接下來才能測試闔牌狀態不能拖曳。
```js
const cardStacks = reactive({
    first: [],
    second: [],
    third: []
});
onMounted(() => {
    const data = geneateDeck(39, false);
    cardStacks.first = data.slice(0, 13);// 梅花A~梅花K
    cardStacks.second = data.slice(13, 26);// 方塊A~~方塊K
    cardStacks.third = data.slice(26); // 紅心A~~紅心K
});
```
因為多一個牌堆除了調整樣板，但同樣也在新牌堆樣板添加`ref="third"`，方便接下來在`:move`中判斷拖曳來源的`DOM`是對應`cardStacks`內的哪個陣列。

看到這邊聰明的讀者大概想到又要調整`:move`函數內的變數`result`，Bingo!  
只需要在函數內`if (result) {`的上方添加底下程式碼:
```js
// 移動的牌必須是開著的
result = result && cardStacks[from][index].isOpen;
```
到此闔牌無法移動已經完成，但今天的工作還沒結束。

因為目前全部的牌都處於蓋牌狀態，為了開發測試需要所以決定簡單調整樣板補上`@click`的事件，讓所有的牌只要點了就會開牌。
```jsx
 <Card :value="element.value" :isOpen="element.isOpen" @click="() => element.isOpen = true" />
```

## 小結
今天雖然實作拖曳至牌尾、預防隨意拖曳牌堆的限制，但在紙牌接龍的情況其實還有:
- 只能對牌尾進行開牌
- 牌的數字有順序接號且非同色系的情況才能移動至牌尾

這部分就留待明日繼續煩惱囉😁

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day12
{{< youtube 39HMsWGpiaU >}}
