---
title: "Day27 當牌全開時自動結算"
date: 2023-10-06T05:25:00Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
如標題所言，今天要做的就是補上『當牌全開時視為已完成遊戲』自動結算遊戲時間、分數，
有實際在現實世界中用撲克牌玩過**接龍**的應該知道其實最麻煩的就是收牌結尾，因為最後只是慢慢把所有牌都移到`結算牌堆`。

> 事先說明: 此篇不會實作自動移牌的動畫效果。

## 開發前思考
判斷牌`全開`不難! 不需要判斷全部牌是否已打開，
只需要判斷`7牌堆`每堆最上面第一張牌是否已打開，若有蓋著那就不能結算，
若每堆第一張都沒蓋著就可以開始結算。

結算分數時只需要考慮補上`發牌區`、`移牌區`有幾張牌，再用加分的方式計算最後的總分。

## 實作過程
### 修改遊戲結算時機
原本函數`checkSolitaireGameDone(cardStacks)`是判斷`結算牌堆`集滿13張牌則當作遊戲結束，但思考過後`結算牌堆都集滿13張`等同於`七牌區的牌全空`一樣會回應`true`，因此放心調整為`檢查七牌堆的最上方開牌的狀態`當作遊戲結束的依據。
```js
// poker-helper.js
/** 
 * 檢查紙牌接龍是否完成
 * @param {CardStacks} cardStacks 
 */
function checkSolitaireGameDone(cardStacks) {
    // 檢查每組牌堆第一張牌覆蓋著，就代表遊戲還沒結束
    const isDone = SEVEN_STACKS.reduce((prev, stackName) => {
        const stack = cardStacks[stackName];
        if (stack.length > 0 && (!stack[0].isOpen)) {
            return false;
        }
        return prev;
    }, true);
    return isDone;
}
```
### 計算剩餘牌數&結算分數
新增函數`getRemainCardCount(cardStacks)`計算並返回`發牌區`和`七牌堆`的剩餘牌數
```js
// poker-helper.js
/** 
 * 計算剩餘在發牌區和7牌堆的張數
 * @param {CardStacks} cardStacks 
 * @returns {Object} {dealer: number, seven: number}
 */
function getRemainCardCount(cardStacks) {
    let dealerStacksCount = cardStacks['dealerStacks'].length;

    let sevenCount = 0;
    SEVEN_STACKS.forEach((name) => {
        sevenCount += cardStacks[name].length;
    });
    return {
        dealer: dealerStacksCount,
        seven: sevenCount,
    };
}
```

在接龍頁面使用`computed`宣告一個變數`remainCardCounts`，負責返回剩餘的牌數。
```js
// DragDemo.vue
// 發牌區、七牌堆的剩餘牌數
const remainCardCounts = computed(() => {
    const { dealer, seven } = getRemainCardCount(cardStacks);
    return { dealer, seven };
});
```

然後調整結算畫面的樣板，把`發牌區`/`7牌堆`的剩餘牌數印出來並計算最後的加權總分(結算分數)，不更動原本的累計分數。
```vue
<BModal v-model="doneModal" title="結算畫面" hide-footer @close="resetGame" @hide="resetGame">
    <h1>玩家 xxx</h1>
    <div>完成花費時間: {{ gameTime }} 秒</div>
    <div>累計分數: {{ gameScore }} 分</div>
    <div>發牌區剩餘 {{ remainCardCounts.dealer }} 張 x 35分</div>
    <div>7牌堆剩餘 {{ remainCardCounts.seven }} 張 x 20分</div>
    <div>加權總分: {{ gameScore + remainCardCounts.dealer * 35 + remainCardCounts.seven * 20 }} 分</div>
</BModal>
```


## 小結
今天實作功能`全開牌自動結算`雖然沒有一張張牌自動移回`結算牌堆`的動態感，但優點就是無須等待看動畫的時間。

雖然上方沒提到今天除了實作也有修正`連點自動移動`的Bug，
Bug主因就是當牌可移回的選擇有`結算牌堆`/`7牌堆`兩種，因為優先選`結算牌堆`然而卡牌不是最後一個不能移，卻沒有繼續讓卡牌移動到`7牌堆`導致。

修復方式就是把邏輯換成優先檢查移動到`結算牌堆`，若不行會繼續檢查是否可以移動`7牌堆`。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day27
{{< youtube oyqWd4N_OM4  >}}
