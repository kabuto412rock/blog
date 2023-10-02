---
title: "Day 19 實作結算牌堆元件(三)整合至接龍頁面"
date: 2023-09-28T03:28:44Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今天會實作`發牌區`、`7牌堆`的牌可以拖曳到`結算牌堆`，且拖曳的過程需遵守`結算牌堆`的同色疊牌由A至K的規則。

## 整理重複的函數
先將昨天在`DealerAreaView.vue`撰寫的程式碼移動到拖曳練習的頁面`DragDemo.vue`。

樣板的部分是沒什麼問題，只是發現有太多函數宣告出現在`DragDemo.vue`，所以將常數`FOUR_SUITS`和判斷結算牌堆規則的`checkNextOk2`函數先移入工具目錄`utils/`內的程式碼，`DragDemo.vue`改用`import`的方式載入通用的常數、函數。
```js
// DragDemo.vue
import { FOUR_SUITS } from '../utils/constants';
import { geneateShuffleDeck, checkNextOk, checkNextOk2 } from "../utils/poker-helper";
```
## 設定結算牌堆用的資料
在原本的`cardStack`中添加針對結算牌堆四花色的撲克牌陣列
```js
// DragDemo.vue
const cardStacks = reactive({
    // 略...
     /** @type {Card[]} */ club: [],
     /** @type {Card[]} */ diamond: [],
     /** @type {Card[]} */ heart: [],
     /** @type {Card[]} */ spade: []
});
```
因為`<FinishedArea />`的屬性`:fourCard`只有要求傳入的物件需要有對應花色名稱的**KEY**就可以，沒有硬性要求不能有其他屬性，
所以樣板的傳值我就簡單設定`cardStacks`傳入，如下程式碼:
```vue
<!-- DragDemo.vue  -->
<div>
    <div class="text">發牌區</div>
    <DealerArea :deck="cardStacks.dealerStacks" :moveCard="dealerMove" />
    <div class="text">結算牌堆</div>
    <FinishedArea :fourCards="cardStacks" @doms="setFourCardDoms" :change="cardChange" />
</div>
```

在`DragDemo.vue`中對變數`cardStacks`新增四個key調整，而原本的函數`getDomName(dom)`也要跟著調整，
主要除了上層對應7個牌堆、也需要對應到`結算牌堆`的HTML元素。

底下程式碼中使用花色列表`FOUR_SUITS`一個個檢查，
若有相同的元素則回傳`結算牌堆`的花色名稱 (club/diamond/heart/spade)，否則依然回傳**none**。
```js
// DragDemo.vue
function getDomName(dom) {
    if (dom == first.value.targetDomElement) {
        return 'first';
    } else if (dom == second.value.targetDomElement) {
        return 'second';
    } else if (dom == third.value.targetDomElement) {
        return 'third';
    } else if (dom == fourth.value.targetDomElement) {
        return 'fourth';
    } else if (dom == fifth.value.targetDomElement) {
        return 'fifth';
    } else if (dom == sixth.value.targetDomElement) {
        return 'sixth';
    } else if (dom == seventh.value.targetDomElement) {
        return 'seventh';
    } else {
        for (let i = 0; i < FOUR_SUITS.length; i++) {
            const name = FOUR_SUITS[i];
            if (dom == fourCardsDom[name]) {
                return name;
            }
        }
        return 'none';
    }
}
```

上方程式碼寫到的變數`fourCardsDom`是一個字典，**KEY**會是花色(字串)，**VALUE**對應畫色的結算牌堆(HTML元素)。

底下的函數`setFourCardDoms`將花色、HTML元素儲存到變數`fourCardsDom`這個字典中這件事，會在`<FinishedArea />`發送的`@doms`事件被執行，這跟昨天實作在`DealerAreaView.vue`中一樣只會執行一次。
```js
// DragDemo.vue
const fourCardsDom = reactive({
    club: null,
    diamond: null,
    heart: null,
    spade: null,
});
function setFourCardDoms(cardDomMaps) {
    FOUR_SUITS.forEach(name => {
        const domElement = cardDomMaps[name];
        fourCardsDom[name] = domElement;
    });
}
```

## 調整拖曳至結算牌堆的規則
### 發牌區拖曳
主要是調整內部變數`result`的布林判斷，如果拖曳目標位置名稱`to`是結算牌堆的花色則走屬於`checkNextOk2`的判斷，否則是走`checkNextOk`的不同花色疊牌判斷，至於`if (result)`內的並沒有調整。
```js
/** 發牌區移動 */
function dealerMove(evt) {
    const to = getDomName(evt.to);
    const dealerCard = evt.draggedContext.element;
    const { futureIndex } = evt.draggedContext;
    let result = true;

    // 如果目標是結算盤堆，則套用結算盤堆的規則
    if (FOUR_SUITS.includes(to)) {
        result = result && checkNextOk2(to, cardStacks, dealerCard);
    } else {
        // 只能移動至目標牌堆的最後一張牌
        result = result && futureIndex == cardStacks[to].length;
        // 檢查疊牌順序、花色是否正確
        result = result && checkNextOk(cardStacks[to], dealerCard);
    }
    if (result) {
        changeOption.value = () => {
            cardStacks.dealerStacks = cardStacks.dealerStacks.filter(card => card.value !== dealerCard.value);
            changeOption.value = null;
        };
    }
    return result;
}
```

### 7牌堆拖曳
這部分跟**發牌區拖曳**一樣只改動`result`的布林判斷，如果是針對`結算牌堆`則改用`checkNextOk2`，目前測試應該是沒問題，直接上縮減後的程式碼:
```js
function limitLocalMove(evt) {
    // 略
    if (FOUR_SUITS.includes(to)) {
        result = result && checkNextOk2(to, cardStacks, element);
    } else {
        // 只能移動至目標牌堆的最後一張牌
        result = result && futureIndex == cardStacks[to].length;
        // 檢查疊牌順序、花色是否正確
        result = result && checkNextOk(cardStacks[to], element);
    }
    // 略
}
```

## 小結
已完成撲克牌都可以遵守規則拖曳至`結算牌堆`，但目前`結算牌堆`的牌移入後就無法移出，所以`結算牌堆`還不完全符合紙牌接龍的遊戲規則。

明日預計完成`結算牌堆`的牌應該要可以拖曳回`7牌堆`。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day19
{{< youtube gYt1kNES4xc >}}