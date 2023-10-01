---
title: "Day22 接龍移牌提示"
date: 2023-10-01T01:10:38Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今天要實作接龍移牌提示，以下是會需要處理的題目:
1. 怎麼取得場上牌的拖曳路線? 
2. 找到拖曳路線後，如何顯示要拖曳至哪個地方的提示(文字or動畫)?

## 取得拖曳路線
目前可知拖曳區塊有`7牌堆`、`發牌區`、`結算牌堆`，其中卡牌可拖曳的方向有:
- `7牌堆`可以內部自拖曳或`結算牌堆`
- `發牌區`只能拖曳至`7牌堆`、`結算牌堆`
- `結算牌堆`只能拖曳至`7牌堆`

初步分析: 可以先計算可以移入`7牌堆`、`結算牌堆`牌尾的撲克牌

### 預計執行步驟:
1. 計算出`7牌堆`、`結算牌堆`各自牌尾後能放什麼牌，儲存在Map
2. 從`發牌區`/`7牌堆`/`結算牌堆`依序判斷**可拖曳卡牌的數字**是否存在Map中?
    - 是，回傳比對成功的結果:
      ```js
      { "可拖曳卡牌所在的牌堆", "拖曳卡牌在牌堆的位置", "預計移入的牌堆"}
      ```
    - 否，繼續比對下一張直到無牌可比

### 實際程式碼
1. 參數帶入要計算的全部牌堆，計算回傳每張牌可被移入的牌堆。
    > 因為有可能出現**梅花A**可以移入`結算牌堆`或`7牌堆`的情況，所以實作設計成`一張牌`只會對應**一個牌堆**，此例**梅花A**會優先被移入`結算牌堆`。
    ```js
    // utils/poker-helper.js
    /**
     * 找出7牌堆、結算牌堆各牌尾後要接的牌
     * @param {CardStacks} cardstacks
     * @returns {Map<Number, String>} Map<撲克牌編號, 目標牌堆名稱>
     */
    function findTailCards(cardstacks) {
        const result = new Map();

        // 找出可拖曳至7牌堆尾巴的牌
        SEVEN_STACKS.forEach((name) => {
            const stack = cardstacks[name];
            if (stack.length === 0) {
                [12, 25, 38, 51].forEach((value) => {
                    result.set(value, name);
                });
                return;
            }

            const lastCard = stack[stack.length - 1];
            const lastCardNumber = lastCard.value % 13;
            const lastCardSymbol = Math.floor(lastCard.value / 13);

            // 檢查是否為A，則跳過
            if (lastCardNumber === 0) {
                return;
            }
            const matchNumber = lastCardNumber - 1;
            const isBlack = lastCardSymbol % 3 == 0;
            [matchNumber + (isBlack ? 13 : 0), matchNumber + (isBlack ? 26 : 39)].forEach((value) => {
                result.set(value, name);
            });
        });
        // 找出可拖曳至結算牌堆尾巴的牌
        FOUR_SUITS.forEach((name, index) => {
            const stack = cardstacks[name];
            if (stack.length === 0) {
                result.set(0 + index * 13, name);
                return;
            }

            const lastCard = stack[stack.length - 1];
            const lastCardNumber = lastCard.value % 13;
            // 檢查是否為K，則跳過
            if (lastCardNumber === 12) {
                return;
            }
            const matchNumber = lastCardNumber + 1;
            result.set(matchNumber + index * 13, name);
        });
        return result;
    }
    ```
2. 參數帶入要計算的全部牌堆、發牌區發到的位置，一旦檢查到有一個卡牌符合則返回`拖曳路線的資訊`，
   若無則返回`null`值。
    ```js
    // utils/poker-helper.js
    /** 取得一個移動提示
    * @param {CardStacks} cardStacks 
    * @param {number} dealerIndex 
    * @returns {MoveHint | null} 移動提示
    */
    function getMoveHint(cardStacks, dealerIndex) {
        const tailValuesMap = findTailCards(cardStacks);
        let hintAnswer = null;
        // 發牌區
        let startIndex = dealerIndex < 3 ? 0 : dealerIndex - 3;
        const dealerCards = cardStacks['delaerStacks'].slice(startIndex, dealerIndex);
        dealerCards.forEach((card) => {
            if (tailValuesMap.has(card.value)) {
                hintAnswer = {
                    fromName: 'delaerStacks',
                    card: card,
                    fromIndex: cardStacks['delaerStacks'].findIndex((c) => c.value === card.value),
                    toName: tailValuesMap.get(card.value),
                };
            }
        });
        if (hintAnswer != null) return hintAnswer;
        // 7個牌堆
        SEVEN_STACKS.forEach((name) => {
            let len = cardStacks[name].length;
            for (let i = 0; i < len; i++) {
                let card = cardStacks[name][i];
                // 由上往下找，遇到未開牌就跳過
                if (!card.isOpen) continue;
                if (tailValuesMap.has(card.value)) {
                    const toName = tailValuesMap.get(card.value);
                    // 只能拿最後一張牌放 結算牌堆
                    if (FOUR_SUITS.includes(toName) && i !== len - 1) continue;
                    hintAnswer = {
                        fromName: name,
                        card: card,
                        fromIndex: i,
                        toName: toName,
                    };
                    break;
                }
            }
        });
        if (hintAnswer != null) return hintAnswer;

        // 結算牌堆
        FOUR_SUITS.forEach((name) => {
            let len = cardStacks[name].length;
            if (len == 0) return;
            let card = cardStacks[name][len - 1];
            if (tailValuesMap.has(card.value)) {
                hintAnswer = {
                    fromName: name,
                    card: card,
                    fromIndex: len - 1,
                    toName: tailValuesMap.get(card.value),
                };
            }
        });

        return hintAnswer;
    }
    ```

## 執行拖曳提示動畫
目前已經可以呼叫函數`getMoveHint`取得拖曳路線的資訊
```jsx
{
    fromName: String, // 來源牌堆的名稱
    fromIndex: Number,// 撲克牌在來源牌堆中的位置 
    card: Card,       // 應拖曳的撲克牌
    toName: String,   // 目標牌堆的名稱
}
```

雖然理想上是產生CSS動畫漸變位移過去，嘗試過但沒找到流暢的解法XD   
因此實作目標改用`標示兩個位置`的方式做拖曳提示😁

### 實作函數`showHint`
- 使用了`document.querySelector`取得來源/目標所在的HTML元素，這邊是抓包含`dcid`屬性的`<div>`元素
- 屬性`dcid`是我寫在每個要取得`HTML`元素的HTML TAG
- 函數`animateMoveDom`則是依據傳入的來源/目標HTML元素進行一秒的顯示提示動畫
- 以下都是用`setTimeout`去做個一秒的定時動畫
```js
/** 顯示移牌提示 */
function showHint(e) {
    const btnElement = e.target;
    const info = getMoveHint(cardStacks, dealer.index);
    if (info) {
        const { card, toName } = info;
        const fromDom = document.querySelector('div[dcid="card' + card.value + '"]');
        let toDom;
        if (cardStacks[toName].length == 0) {
            toDom = document.querySelector('div[dcid="' + toName + '"]');
        } else {
            toDom = document.querySelector('div[dcid="card' + cardStacks[toName][cardStacks[toName].length - 1].value + '"]');
        }
        animateMoveDom(fromDom, toDom);
    } else {
        btnElement.disabled = true;
        const orginalContent = btnElement.textContent;
        btnElement.textContent = '沒有可移動的牌';

        setTimeout(() => {
            btnElement.disabled = false;
            btnElement.textContent = orginalContent;
        }, 1000);
    }
}
function animateMoveDom(element1, element2) {
    const { x, y, height } = element2.getBoundingClientRect();
    const element1Clone = element1.cloneNode(true);
    const app = document.body.querySelector("#app");
    app.appendChild(element1Clone);
    element1Clone.style.position = 'absolute';
    element1Clone.style.zIndex = 9999;
    element1Clone.style.top = Math.floor(y + height / 3) + 'px';
    element1Clone.style.boxShadow = '0 0 10px 5px limegreen';
    element1Clone.style.left = Math.floor(x) + 'px';
    element1.style.opacity = 0.5;
    setTimeout(() => {
        app.removeChild(element1Clone);
        element1.style.opacity = 1;
    }, 1000);
}
```

## 小結
今天實作`接龍移牌提示`，實作**取得拖曳路線**都很順利，
但要純用JS控制元素一格格移動會很不自然非常不順利，所以最後以`標註起點/終點`的方式去完成`拖曳提示`功能😎。


程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day22
{{< youtube VfG2t97jyvo >}}

