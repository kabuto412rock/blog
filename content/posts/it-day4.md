---
title: "Day 04 調整翻牌效果&實作洗牌&外加撲克牌連連看"
date: 2023-09-13T06:25:37Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

## 改變翻牌效果
研究Transition的文件後發現跟我想像的動畫變換不同，官方用法`<Transition></Transition>` 包裹內的新舊元件其中一個在動畫過程會先被移除掉或新增，但我先前設計好的CSS會是兩個元素都存在只是一個會被轉到CSS效果轉到背後，因此我的水平翻轉動畫需要兩個元素都存在HTML上。

為了在時間內完成，我打消原本水平翻轉的作法，我改採`Transition`結合`v-if`的方式去顯示卡牌正反兩面，
發現用漸進式出現消失也是不錯的效果，以下是樣板的改變:
```jsx
// Card.vue
<template>
    <div class="card " @click="emit('poker-flip', value)">
        <Transition name="card-flip">
            <div v-if="isOpen" class="card-front" :class="numberClass">{{ content }}</div>
            <div v-else class="card-back"></div>
        </Transition>
    </div>
</template>
```
參考Vue官方的[範例](https://vuejs.org/guide/built-ins/transition.html#transition-between-elements)，
在style添加以下後綴是`-enter-active`/`leave-active`/`-enter-from`/`-leave-to`的class，是為了讓Transition知道動畫要如何改變。
```jsx
// Card.vue
<style scoped>
.card-flip-enter-active,
.card-flip-leave-active {
    transition: all 0.5s ease-out;
}

.card-flip-enter-from,
.card-flip-leave-to {
    opacity: 0;
}
{/* 略 */}
```
## 實作洗牌
1. 實作兩個函數存於`utils/poker-helper.js`
   - 洗牌的函數`shuffle`，洗牌採用的演算法參考看別人文章實作 **Fisher-Yates** 演算法，雖然洗牌用其他方法也可以，但看參考的文章說這個時間複雜度最低。
    ```js
    function shuffle(deck) {
        let length = deck.length;
        for (let i = 0; i < length; i++) {
            let rand_to_swap = Math.floor(Math.random()*(length-i));
            let tmp = deck[length-1-i];
            deck[length-1-i] = deck[rand_to_swap];
            deck[rand_to_swap] = tmp;
        }
        return deck;
    }
    ```
   - 產生52張洗亂的牌`geneateShuffleDeck`，這邊的`isDone`是為了儲存比對成功，
    原本在實作上是會消失，但後來發現連連看的版面會跑掉，所以設計`isDone`為true時卡片會變成顯示不可點擊的小綠卡。
    ```js
    function geneateShuffleDeck() {
        let deck = [];
        for (let i = 0; i < 52; i++) {
            deck.push({
                value: i,
                isOpen: false,
                isDone: false
            });
        }
        return shuffle(deck);
    }
    ```
## 製作撲克牌連連看
簡單就是52張牌洗亂擺在桌上，翻出兩張牌打開有相同的就等於連上，
因為我記憶力沒多好，所以遊戲規則調整成一次最多可以翻六張，超過張數的話牌就會蓋起來!

雖然樣式很醜，但也做了記錄了`分數`、`當前經過時間`和`遊戲重置`的功能。
因為有越來越多變數用ref存著，在`<script setup>`中每次對`ref`的變數取值/設值都要加 **.value** 越來越冗長，後面想到可以用`reactive`宣告一個遊戲狀態存多個屬性欄位，尤其是陣列的部分
```js
// GameBoard.vue
const boardCards = ref(geneateShuffleDeck());
const gameScore = ref(0);
const game = reactive({
    timer: 0, // 當前經過時間
    timerInterval: null, // 儲存執行 setInterval()回傳的intevealId，重置會需要用到
    deckCards: [], // 將目前打開的牌放到裡面，方便之後比對是否有數字相同的卡 
})
```

另外遊戲判定成功，我是使用Watcher去看有沒有分數(gameScore)達到182的，才去跳出結束訊息，
但看網路上watch好像都是用在接收API結果觸發時居多。
```js
// GameBoard.vue
watch(gameScore, (newScore, oldScore) => {
    if (newScore >= 182) {
        alert(`遊戲結束，花費時間: ${timerFormat.value} 總分數: ${newScore}!!!`);
        resetGame();
    }
});
```

實作完後覺得最複雜的可能就是點卡牌後觸發的翻轉功能，判定成功與否真的蠻麻煩，
傳值/取值也是如果可以有個簡單的中央儲存控管就好了。

最後成功實作出來也玩了幾回，至少真的像在玩一個遊戲的感覺，雖然最多可以點開六張是比較寬鬆。

以下是實際運作影片:
{{< youtube 3gS_K3Ab9D4 >}}

**參考**
- [簡單又複雜的洗牌演算法](https://medium.com/@asd757817/%E7%B0%A1%E5%96%AE%E5%8F%88%E8%A4%87%E9%9B%9C%E7%9A%84%E6%B4%97%E7%89%8C%E6%BC%94%E7%AE%97%E6%B3%95-7e7254bb9145)
- [Vue官網的Watchers](https://vuejs.org/guide/essentials/watchers.html)

明日預計會做紙牌接龍會用到的`牌堆`設計，感謝觀看!