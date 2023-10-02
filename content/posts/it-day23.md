---
title: "Day23 連點2下自動移牌"
date: 2023-10-02T03:45:32Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今天要實做的是**點擊自動移牌**的功能也算是昨天提示的延伸，差別只在會**實際移動卡牌**。

我打算**連點移牌**功能只做在`七牌堆`和`發牌區`，結算牌堆就不提供此功能。

## 實作過程
### 處理連點事件
調整`發牌區`/`七牌堆`的卡片元件`<Card>`添加對應的屬性`@dblclick="emit('card-click', element)"`，
這會讓卡牌元件`<Card>`被連點兩下(Double Click)時，向父元件發送事件`card-click`然後攜帶的參數**element**則是**Card**物件
```jsx
// Card
{
    value: 0, // 卡牌對應數值，Ex: 梅花A
    isOpen: false, // 是否已開牌
}
```
然後修改上層樣板(DragDemo.vue)的部分:
- 為了接收`card-click`事件進行處理，在**發牌區**的樣板修改成有添加`@card-click="(card) => clickAutoMove('dealerStacks', card)`
- 在**七牌堆**的樣板，這七行依序添加屬性`@dblcick`
  -  第一牌堆 `@dblclick="clickAutoMove('first', element)"`
  -  第二牌堆 `@dblclick="clickAutoMove('second', element)"`
  -  中間略...
  -  第七牌堆的屬性 `@dblclick="clickAutoMove('seventh', element)"`

### 處理自動移動的邏輯
這邊出現的新函數`clickAutoMove(fromName, card)`，主要是用來處理**自動移動**的邏輯，流程如下:
1. 先利用`findFollowDeckName`找出**card**可以拖曳到的牌堆名稱，然後依照優先順序排序(結算牌堆排第一)。
2. 如果沒有找到對應的牌堆，則不執行後續。
3. 取出第一個牌堆名稱當作目標牌堆
4. 接著就判斷來源牌堆是`發牌堆`還是`七牌堆`，來做不同的處理(修改對應牌組陣列 還有 加分等等)

可以參考下方的程式碼片段:
```js
// DragDemo.vue
/**
 * 自動移動
 * @param {String} fromName 來自的牌堆名稱
 * @param {Card} card 想移動的牌
 */
function clickAutoMove(fromName, card) {
    const toNames = findFollowDeckName(cardStacks, card).sort((a, b) => {
        const aOrder = a.length + FOUR_SUITS.includes(a) ? -100 : 0;
        const bOrder = b.length + FOUR_SUITS.includes(b) ? -100 : 0;
        return aOrder - bOrder;
    })
    // 如果沒找到對應牌堆，則不執行
    if (toNames.length == 0) {
        console.log(`卡牌${PokerValuesMap[card.value].content}沒有符合移動的規則`);
        return;
    }

    const toName = toNames[0];
    const isToFinishedArea = FOUR_SUITS.includes(toName);
    if (fromName == 'dealerStacks') {
        // 來自`發牌堆`
        const fromIndex = cardStacks[fromName].findIndex(c => c.value == card.value);

        const newFromCards = [
            ...cardStacks[fromName].slice(0, fromIndex),
            ...cardStacks[fromName].slice(fromIndex + 1)
        ];
        const newToCards = [
            ...cardStacks[toName],
            cardStacks[fromName][fromIndex]
        ];
        cardStacks[fromName] = newFromCards;
        cardStacks[toName] = newToCards;
        gameScore.value += isToFinishedArea ? 25 : 10;
    } else if (SEVEN_STACKS.includes(fromName)) {
        // 來自7牌堆
        const fromLength = cardStacks[fromName].length;
        const fromIndex = cardStacks[fromName].findIndex(c => c.value == card.value);

        if (isToFinishedArea) {
            if (fromIndex != fromLength - 1) {
                console.log(`卡牌${PokerValuesMap[card.value].content}不是${fromName}的最後一張牌，不可移入結算牌堆`);
                return;
            }
            const newFromCards = cardStacks[fromName].slice(0, fromIndex);
            const newToCards = [
                ...cardStacks[toName],
                card
            ];
            cardStacks[fromName] = newFromCards;
            cardStacks[toName] = newToCards;
            gameScore.value += 15;
        } else {
            const newFromCards = cardStacks[fromName].slice(0, fromIndex);
            const newToCards = [
                ...cardStacks[toName],
                ...cardStacks[fromName].slice(fromIndex)
            ];
            cardStacks[fromName] = newFromCards;
            cardStacks[toName] = newToCards;
        }
    }
}
```

另外在`poker-helper.js`新增函數`findFollowDeckName(cardstacks, targetCard)`，主要是用來找出**targetCard**可以拖曳到的牌堆名稱，因為可能可以拖曳到複數個牌堆，所以回傳**字串組成的一維陣列**:

```js
// utils/poker-helper.js
/**
 * 找出`指定牌`可以接在哪一個牌堆(7牌堆or結算牌堆)的後面
 * @param {CardStacks} cardstacks 牌堆
 * @param {Card} targetCard 指定牌
 * @returns {Array<String>} Array<目標牌堆名稱>
 */
function findFollowDeckName(cardstacks, targetCard) {
    const result = new Set();

    // 找出可拖曳至7牌堆尾巴的牌
    SEVEN_STACKS.forEach((name) => {
        const stack = cardstacks[name];
        if (stack.length === 0) {
            if ([12, 25, 38, 51].includes(targetCard.value)) {
                result.add(name);
            }
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
        const isInclude = [matchNumber + (isBlack ? 13 : 0), matchNumber + (isBlack ? 26 : 39)].includes(targetCard.value);
        if (isInclude) result.add(name);
    });


    // 找出可拖曳至結算牌堆尾巴的牌
    FOUR_SUITS.forEach((name, index) => {
        const stack = cardstacks[name];
        if (stack.length === 0) {
            if (index * 13 == targetCard.value) result.add(name);
            return;
        }
        const lastCard = stack[stack.length - 1];
        const lastCardNumber = lastCard.value % 13;
        // 檢查是否為K，則跳過
        if (lastCardNumber === 12) {
            return;
        }
        const matchNumber = lastCardNumber + 1;
        if (matchNumber + index * 13 == targetCard.value) {
            result.add(name);
        }
    });
    return Array.from(result);
}
```

## 小結
今天完成的功能**連點2下自動移牌**可以取代部分**拖曳卡牌**的操作，讓遊戲更加順暢。
這部分也是參考曾經玩過的接龍遊戲功能，當時就覺得這樣很方便😁。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day23
{{< youtube yvOmRSRhWOY >}}
