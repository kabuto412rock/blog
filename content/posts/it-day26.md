---
title: "Day26 實作功能【返回上一步】"
date: 2023-10-05T02:04:38Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

## 前言
因為玩接龍有時會有按錯步，這時候沒有**返回上一步**的機制就只能硬著頭皮玩下去或按**重置**，今天想解決這個問題。

## 開發前的思考
在`ReactJS`官方學習文件中OOXX遊戲`Tic-Tac-Toe`也有提到時光旅行的實作，基本上就是每一步更動後的結果狀態都推入(Push)陣列變數`history`裡面，時光回朔就是將結果狀態直接設為`history[index]`。

目前開發下來的程式碼大概要回到上一步的只有`分數`、`牌組`、`發牌的索引`，
經過的時間應該就不用`上一步`。


## 實作過程
### 儲存遊戲變化的歷史
首先宣告一個負責儲存歷史的ref變數`history`
```js
const history = ref([]);
```

宣告函數`pushStateToHistory()`負責撰寫把最新的狀態推入history
- 累積超過30個後會將最舊的狀態移除再推入最新的狀態，避免暫存太多的上一步驟。
- 因為`reactive`的關係所以不得不手動深度複製每張卡牌，這也是為什麼要做`elemFunc`的原因
```js
/** 儲存當前狀態到歷史紀錄 */
function pushStateToHistory() {
    if (history.value.length > 30) {
        const startIndex = history.value.length - 30;
        history.value = history.value.slice(startIndex, history.value.length);
    }
    const elemFunc = (card) => ({
        "value": card.value,
        "isOpen": card.isOpen,
        "isDone": card.isDone,
    });
    history.value = [
        ...history.value,
        {
            "cardStacks": {
                first: cardStacks.first.slice().map(elemFunc),
                second: cardStacks.second.slice().map(elemFunc),
                third: cardStacks.third.slice().map(elemFunc),
                fourth: cardStacks.fourth.slice().map(elemFunc),
                fifth: cardStacks.fifth.slice().map(elemFunc),
                sixth: cardStacks.sixth.slice().map(elemFunc),
                seventh: cardStacks.seventh.slice().map(elemFunc),
                dealerStacks: cardStacks.dealerStacks.slice().map(elemFunc),
                club: cardStacks.club.slice().map(elemFunc),
                diamond: cardStacks.diamond.slice().map(elemFunc),
                heart: cardStacks.heart.slice().map(elemFunc),
                spade: cardStacks.spade.slice().map(elemFunc),
            },
            "gameScore": JSON.parse(JSON.stringify(gameScore.value)),
            "dealer": { index: dealer.index },
        }
    ];
}
```

接著在程式碼中【分數、卡牌有變動】的情況都補上執行`pushStateToHistory();`去儲存當下的狀態:
- `發牌區移動`/`結算牌堆移動`/`7牌堆移動`成功移動後
- 遊戲初始化`resetGame`
- 連點自動拖曳`clickAutoMove`只要成功移動，最後也會執行
> 發牌區`<DealerArea />`點擊開牌也會執行`pushStateToHistory();`，因為發牌索引`dealer.index`產生變化

### 實作返回上一步函數
返回上一步函數`undo`的演算法:
1. 取得`history`變數的陣列的長度，若長度大於1繼續往下執行，否則不執行後續步驟。
2. 因為`history`目前最後一個元素就是當下的狀態，所以從`history`克隆出原本長度減一的陣列暫存至變數`prevHistory`
3. 將`history`的值更新為`prevHistory`
4. 將`prevHistory`最後一個元素內的狀態`發牌區`/`結算牌堆`/`7牌堆`/`遊戲分數`/`發牌索引`的值全都覆蓋到對應變數
```js
/** 返回上一步 */
function undo() {
    if (history.value.length > 1) {
        const prevHistory = history.value.slice(0, history.value.length - 1);
        history.value = prevHistory;
        const prevState = prevHistory[prevHistory.length - 1];
        cardStacks.dealerStacks = prevState.cardStacks.dealerStacks;
        FOUR_SUITS.forEach((deckName) => {
            cardStacks[deckName] = prevState.cardStacks[deckName];
        })
        SEVEN_STACKS.forEach((deckName) => {
            cardStacks[deckName] = prevState.cardStacks[deckName];
        })
        gameScore.value = prevState.gameScore;
        dealer = prevState.dealer;
    }
}
```

## 小結
今日除了完成`返回上一步`的功能使得遊戲玩起來更有趣😎也盡可能以**演算法**的角度更明確的步驟去說明程式碼的實現。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day26
{{< youtube EhYPGI4SNJ8  >}}

**參考**
- https://stackoverflow.com/questions/597588/how-do-you-clone-an-array-of-objects-in-javascript
- https://react.dev/learn/tutorial-tic-tac-toe#implementing-time-travel
