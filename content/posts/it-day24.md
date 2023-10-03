---
title: "Day24 紙牌接龍-結算畫面"
date: 2023-10-03T03:42:16Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
目前實作的紙牌接龍還沒有**結算畫面**，所以今天就來做!

## 初步思考
製作**結算畫面**本身不是問題，畢竟畫面沒有要做得超級漂亮的情況下都是沒問題的!

問題是何時跳出**結算畫面**? 我想到的情形有兩種:
1. `結算牌堆`四堆都集完13張的情況
2. 畫面中`7牌堆`的牌全部已經打開的情況

我個人認為是判斷第一種判斷`結算牌堆`的方式實作起來比較簡單。

## 實作邏輯
### 製作檢查完成牌組的函數
宣告一個函數`checkSolitaireGameDone`負責檢查紙牌接龍是否完成。
- 依序檢查各`結算牌堆`，若數量不為13就直接回傳 **否`false`**
- 最後就回傳 **是`true`**

程式碼如下:
```js
/** 
 * 檢查紙牌接龍是否完成
 * @param {CardStacks} cardStacks 
 */
function checkSolitaireGameDone(cardStacks) {
    for (let i = 0; i < FOUR_SUITS.length; i++) {
        if (cardStacks[FOUR_SUITS[i]].length !== 13) return false;
    }
    return true;
}
```

### 監控觸發檢查
不意外的又使用到`watch`這個關鍵字做監控，這部分就是跟Day4的連連看一樣，
當**牌堆`cardStacks`**發生變化就去觸發檢查，判定完成就使用`alert`跳出結算訊息。

> 使用者點擊`alert()`之後，才會執行重設遊戲的函數`resetGame()`

```js
// DragDemo.vue
watch(cardStacks, (newCardStacks) => {
    const isDone = checkSolitaireGameDone(newCardStacks);
    if (isDone) {
        alert(`遊戲結束，花費時間: ${gameTime.value} 秒 總分數: ${gameScore.value}!!!`);
        resetGame();
    }
})
```

## 小結
今天在實作**結算畫面**的過程雖然遇到一些Bug，但不影響遊戲可以正常跳出結算訊息並重置。

只是目前用alert`跳出那個結算訊息不太像一個正常遊戲的畫面，所以明天會繼續實作`真正的結算畫面`和修正目前發現的Bug:
- 昨日開發的**連點移動**的優先度不正常
  1. 預期優先移到**結算牌堆**的牌，自動移牌時竟然在`7牌堆`區中左右來回移動。
  2. **發牌區**的牌應該要自動移到**結算牌堆**，竟然先移動到**7牌堆**


程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day24
{{< youtube Hm254mth5Hw >}}

