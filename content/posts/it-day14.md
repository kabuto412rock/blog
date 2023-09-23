---
title: "Day14 接龍發牌區功能實作(一)發牌循環"
date: 2023-09-23T00:33:59Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

為了實現接龍發牌區功能，必須先思考如何讓撲克牌循環利用，這部分程式碼我是先拆一個頁面來練習實作，避免單一頁面的程式碼邏輯太過混亂。

## 發牌區的樣板
今日實作的目標會是一個左邊`移牌區`右邊`發牌堆`結合在一起的，
點擊`發牌堆`會將牌發到`移牌區`顯示，`移牌區`最多同時顯示三張牌。

### 實作的樣板程式碼
```html
// DealerAreaView.vue
<GameBoard>
    <div class="text">發牌區</div>
    <div style="display: grid; grid-template-columns: 1.5fr 1fr; gap:3rem; width: fit-content;">
        <!-- 移牌區 - 左邊水平疊牌最多三張 -->
        <div style="display: grid; grid-template-columns: repeat(3, 3rem);">
            <div v-for="card in canTakeCards" :key="card.value">
                <Card :value="card.value" :isOpen="true" />
            </div>
        </div>
        <!-- 發牌堆 -->
        <div class="card-box">
            <div class="card" style="visibility: hidden;">
                <div style="visibility: visible; width: 100%;height: 100%; ">
                    <Transition name="slide-left">
                        <div v-if="deckState == 'empty'">無牌可用</div>
                        <div v-else-if="deckState == 'full'" class="card" @click="clickCard">重新循環</div>
                        <div v-else-if="deckState == 'normal'" @click="clickCard" class="card-back"></div>
                    </Transition>
                </div>
            </div>
        </div>
    </div>
</GameBoard>
```

可以注意到`發牌堆`使用`<Transition>`包裹，裡面的元件使用`v-if`和`v-else-if`去判斷三種情況顯示元件，
這邊使用到的`deckState`是一個計算結果。

在說明`deckState`之前，先講一下此範例使用到的變數`index`、`deck`。
```js
const index = ref(0);
const deck = ref(geneateDeck(14, true));
```
`deck`就是包含著`發牌堆`+`移牌區`的卡牌陣列，日後會改用屬性`props`傳進元件，
但因為今天在練習實作就先自己產生。

`index`則是對應發到第幾張牌的**指標(Pointer)**，採用指標的原因是可以不用將把`發牌堆`、`移牌區`分成兩個陣列儲存，當指標`index`為0時代表移牌區沒有牌，`index`為1時則代表已發到第1張牌，以此類推。

回到變數`deckState`就是用指標位置和牌堆數量計算出來`發牌堆`會有3種顯示狀態:
1. `發牌堆`和`移牌區`都沒牌時，顯示`無牌可用` (empty)
2. `發牌堆`已發完牌但`移牌區`仍有牌，則顯示`重新循環` (full)
3. `發牌堆`還有牌可以發，顯示撲克卡背  (normal)
```js
const deckState = computed(() => {
    if (index.value === 0 && deck.value.length == 0) return 'empty';
    if (index.value === deck.value.length) return 'full';
    return 'normal';
});
```

`移牌區`顯示的撲克陣列`canTakeCards`也是用`index`和`deck`計算出來的最多回傳3個元素的陣列，
因為`Array.slice`只會回傳複製陣列，所以不會影響`deck`陣列本身。
```js
/** 玩家可拿取的牌 */
const canTakeCards = computed(() => {
    let startIndex = index.value < 3 ? 0 : index.value - 3;
    return deck.value.slice(startIndex, index.value);
});
```

點擊`發牌堆`觸發的函數也只是改變指標`index`往前移動，間接影響上面使用`computed`計算出來的結果。
```js
function clickCard() {
    index.value++;
    if (index.value > deck.value.length) {
        index.value = 0;
    }
}
```
## 小結
發牌區內部的部分使用陣列**指標**的方式去判斷，可以減少對陣列本身的`push`和`pop`等異動原堆疊資料的行為。

目前`移牌區`沒有使用到`<draggable>`包裹，實際上`移牌區`的牌應該要可被拖曳至中心的集牌區進行移動，但中心的集牌區的牌不能被拖回發牌區的`移牌區`，這部分明天才會做但還是先提一下。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day14
{{< youtube vtsZPyt9KdQ >}}