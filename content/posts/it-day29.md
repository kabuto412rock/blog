---
title: "Day29 簡單評估是否還有活路"
date: 2023-10-08T00:23:02Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

目前所製作的經典紙牌接龍其實是源自於`Klondike Solitaire`的規則，只是發牌區是一次抽一張的循環制，因為網路上有看到每次都是抽3張但只能移動最上面那一張的規則，那種非常難玩挑戰性也很大。

**判斷紙牌接龍無解**這個問題這兩天困擾我很久，在查過無數資料略讀幾篇論文後，尤其實際有用的資料大多是英文論文😭，發現這絕對不是一兩天的空餘時間就可以理解的目標，所以決定降低目標不去判斷是否已經死局，改為【簡單評估是否還有活路】。

## 簡單評估是否還有活路

尋找活路的最容易想到的3種可能移動方式:
1. `七牌堆`最後一張後面要接的牌存在於`發牌區`中
2. 任一`七牌堆`壓在隱藏牌上放的那張可以接在其他`七牌堆`的後面
3. `發牌區`任一張牌或`七牌堆`的最後一張 可移動`結算牌堆`

### 開始實作
按照簡單評估的三項規則依序檢查，若有檢查可移動的方式為**true**就不會繼續檢查後續的規則，以下為程式碼:
```js
// poker-helper.js
/** 檢查是否還有效的移動卡牌
 * @param {CardStacks} cardStacks
 * @returns {boolean} 有有效移動為true 可能沒有為false
 */
function checkValidMove(cardStacks) {
    const dealerStacksValues = cardStacks['dealerStacks'].map((card) => card.value);

    const seventLastValues = {};

    // 1. `七牌堆`最後一張後面要接的牌存在於`發牌區`中
    let haveMove1 = SEVEN_STACKS.some((name) => {
        const stack = cardStacks[name];
        if (stack.length === 0) {
            return false;
        }
        const lastCard = stack[stack.length - 1];
        const lastCardNumber = lastCard.value % 13;
        seventLastValues[name] = lastCard.value;
        // A後面沒有要接的，但可移動到結算牌堆(回應成功)
        if (lastCardNumber === 0) return true;
        const targetPokerValues = getDifferentColorPokerValues(lastCard.value - 1);

        return dealerStacksValues.some((value) => {
            return targetPokerValues.includes(value);
        });
    });
    if (haveMove1) {
        console.log("第1活局: `七牌堆`最後一張後面要接的牌存在於`發牌區`");
        return true;
    }
    // 2. 任一`七牌堆`壓在隱藏牌上放的那張可以接在其他`七牌堆`的後面(不包含本身牌堆)
    let haveMove2 = SEVEN_STACKS.some((name) => {
        const stack = cardStacks[name];
        if (stack.length === 0) {
            return false;
        }
        let firstOpenCard = null;
        for (let i = 1; i < stack.length; i++) {
            if (stack[i].isOpen && (!stack[i - 1].isOpen)) {
                firstOpenCard = stack[i];
                break;
            }
        }
        // 沒有壓在隱藏牌上的牌或 放的那張是K無法接在其他牌堆後面(回應失敗)
        if (firstOpenCard === null || firstOpenCard.value % 13 === 12) {
            return false;
        }
        // 檢查是否有可以接的牌
        const targetPokerValues = getDifferentColorPokerValues(firstOpenCard.value + 1)
        return SEVEN_STACKS.some((name2) => {
            if (name === name2) return false;
            if (seventLastValues[name2]) {
                return targetPokerValues.includes(seventLastValues[name2]);
            }
            return false;
        });
    });

    if (haveMove2) {
        console.log("第2活局: 任一`七牌堆`壓在隱藏牌上放的那張可以接在其他`七牌堆`的後面(不包含本身牌堆)");
        return true;
    }
    // 3. `發牌區`任一張牌或`七牌堆`的最後一張 可移動至 `結算牌堆`
    let haveMove3 = FOUR_SUITS.some((suit, index) => {
        const stackLen = cardStacks[suit].length;
        if (stackLen === 13) return false;

        let targetValue = index * 13 + stackLen;
        return dealerStacksValues.includes(targetValue) || Object.values(seventLastValues).includes(targetValue);
    });
    if (haveMove3) {
        console.log("第3活局: `發牌區`任一張牌或`七牌堆`的最後一張 可移動至 `結算牌堆`");
    }
    return haveMove3;
}
/** 取得相同數字不同顏色的撲克牌編碼
 * @param {Number} pokerValue 對應撲克牌的編號
 * @returns {Array} [number1, number2] 
 */
function getDifferentColorPokerValues(pokerValue) {
    const red = (Math.floor(pokerValue / 13) % 3) == 0
    const number = pokerValue % 13;
    return [number + (red ? 13 : 0), number + (red ? 26 : 39)];
}
```

然後在`DragDemo.vue`中使用函數`checkValidMove(cardStacks)`將結果【是否有可移動的牌(推測)】儲存在計算ref變數。
```js
// DragDemo.vue
const maybeHaveValidMove = computed(() => checkValidMove(cardStacks));
```
並使用以下方式顯示在Vue樣板
```vue
// DragDemo.vue
<span>
    {{ maybeHaveValidMove ? '(遊戲還有解)' : '(可能無解😎)' }}
</span>
```


## 小結
今天放棄**判斷是否無解**的實現，改為實現**可能無解去提示玩家**是我目前能做到的部分。

這讓我想到如果要去實現一個執行起來會很沒效率的功能，中間尋找答案的過程會求助網路、同事，但最終預期無法在時限內完成最後好像也是看PM能不能協調修改規格，但自己的**SideProject**客戶/PM/RD都是同一人的情況下就可以自由的修改規格😂👍

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day29
{{< youtube v2gyMViaYrc  >}}

**參考文件(想讓大腦受苦的請看下面論文)** 
- https://www.theseus.fi/bitstream/handle/10024/501330/Voima_Mikko.pdf?sequence=2&isAllowed=y
- https://core.ac.uk/download/pdf/82132940.pdf