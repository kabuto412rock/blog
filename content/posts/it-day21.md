---
title: "Day21 顯示接龍分數、遊戲時間"
date: 2023-09-29T23:38:40Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今天預計實作的項目**顯示分數**、**顯示遊玩時間**，
但實作**顯示分數**必須配合實作**累計分數**的功能，不然分數都不會變動也是尷尬😂。

儲存分數、遊玩時間的變數宣告:
```js
const gameScore = ref(0);
const gameTime = ref(0);
const gameTimer = ref(null);
```
## 實作遊戲分數
### 實作分數累計規則
先整理出接龍的分數在什麼情況會增加?
1. 從`發牌區`移出則加10分，因為`發牌區`的牌不會被重新移入所以不用擔心重複加分。
```js
// DragDemo.vue
/** 發牌區移動 */
function dealerMove(evt) {
    // 略
    if(result){
        changeOption.value = () => {
            // 略
            gameScore.value += 10;
        }
    }
}
```
2. `7牌堆`的牌被打開則加5分，因為被打開的牌不會被蓋回去。
> 原本程式就會將`7牌堆`最一張牌設為打開，改判斷最後一張原本是蓋牌才開牌、加5分避免分數重複累加。
```js
// DragDemo.vue
watch(cardStacks, (stacks) => {
    // 檢查每組牌堆最後一張
    validNames.forEach(cardName => {
        if (stacks[cardName].length > 0) {
            const lastCard = stacks[cardName][stacks[cardName].length - 1];
            if (!lastCard.isOpen) {
                lastCard.isOpen = true;
                gameScore.value += 5;
            }
        }
    });
});
```
3. 卡牌移入`結算牌堆`加15分，移出則扣15分。
   - 移出的部分直接調整`結算牌堆`移動進行扣分
   ```js
   /** 結算牌堆移動 */
   function finishedCardMove(evt) {
       // 略
       if (result) {
           changeOption.value = () => {
               gameScore.value -= 15;
               changeOption.value = null;
           };
       }
       return result;
   }
   ```
   - 移入結算牌堆則會需要調整`發牌區`和`7牌堆`的移動，一樣是調整`changeOption.value`的程式碼要加15分。
   > 因為第1點的規則(10分)，如果直接從發牌區拖曳至結算牌堆會加25分(15+10)
   ```js
   /** 發牌區移動 */
   function dealerMove(evt) {
       // 如果目標是結算盤堆，則套用結算盤堆的規則
       const isToFinishedArea = FOUR_SUITS.includes(to);
       // 略
       if (result) {
           changeOption.value = () => {
               cardStacks.dealerStacks = cardStacks.dealerStacks.filter(card => card.value !== dealerCard.value);
               gameScore.value += 10 + (isToFinishedArea ? 15 : 0);
               changeOption.value = null;
           };
       }
       return result;
   }
    /** 7牌堆移動 */
    function limitLocalMove(evt) {
        const isToFinishedArea = FOUR_SUITS.includes(to);
        // 略
        if (result) {
            // 略
            changeOption.value = () => {
                cardStacks[from] = newFromCards;
                cardStacks[to] = newToCards;
                if (isToFinishedArea) {
                    gameScore.value += 15;
                }
                changeOption.value = null;
            };
        }
        // 仍使用原生的拖曳效果
        return result;
    }

   ```

### 實作顯示樣板
這邊只是把分數、時間、重置按鈕都移動至同一區塊，部分程式碼如下:
```html
<!-- DragDemo.vue -->
<div
    style=" display: flex; flex-wrap: wrap; flex-direction: row;justify-content: space-around; align-items: center;  background-color: antiquewhite; font-size: large;">
    <div>累計分數: {{ gameScore }}</div>
    <div>經過時間: {{ gameTime }} 秒</div>
    <button style="font-size: 1.5rem;" @click="resetGame">重置</button>
</div>
```

## 實作遊玩時間
撰寫遊戲計時的函數`startTimer()`
- 變數`gameTime`負責儲存秒數
- 變數`gameTimer`負責儲存`intervalID`
```js
function startTimer() {
    if (!gameTimer.value) {
        gameTimer.value = setInterval(() => {
            gameTime.value++;
        }, 1000);
    }
}
```
當點擊GameBoard時則執行`startTimer()`的樣板:
```vue
<GameBoard style="display: flex;" @click="startTimer">
```
當**遊戲重置**時，使用`clearInterval(gameTimer.value)`清除遊玩時間的計時器。

## 小結
程式碼逐漸變得複雜，但依舊完成預期的功能`分數`、`時間`

明天想實作看看如何做到**下一步提示**。


程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day21
{{< youtube tJUj_NYbHww >}}
