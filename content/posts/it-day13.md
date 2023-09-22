---
title: "Day13 牌尾自動翻牌、限制疊牌順序"
date: 2023-09-22T04:58:26Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
今日預計實作項目:
- ☑️ 只能對牌尾進行翻牌
- ☑️ 牌尾自動翻牌
- ☑️ 限制疊牌順序

## 只能對牌尾進行翻牌
新增函數`openCard(cards, element)`因為需要確認點擊卡牌確實是在牌尾，
所以需要參數`cards`對應原本的卡牌陣列、參數`element`對應點擊的卡牌元素。

檢查牌尾就單純檢查`element`的數字(value)是否跟陣列`cards`的最後一筆數字相同，
在這邊的情況不用擔心`cards`的長度為`0`，因為所在牌堆沒有卡牌自然就不會渲染使用`openCard`函數的`<Card>`元件。
```js
/** 開牌函數 
 * @param {Card[]} cards 
 * @param {Card} element 
 */
function openCard(cards, element) {
    let same = cards[cards.length - 1].value == element.value;
    if (same) {
        element.isOpen = true;
    }
}
```
調整每一個`<Card>`元件的`@click`監聽，以下以第1牌堆的樣板舉例:
```jsx
<Card :value="element.value" 
      :isOpen="element.isOpen"
      @click="openCard(cardStacks.first, element)" />
```
第2、3牌堆只需將`cardStacks.first`中的**first**替換成對應陣列**second**和**third**，
雖然下一步牌尾會自動翻牌不需要人去手動翻開、但還是要避免有人去對非牌尾的開牌。

##  牌尾自動翻牌
這部分我是依賴Vue本身的`watch`去監聽牌堆的變化，
跟上一個步驟很像的取得最後一張牌`lastCard`並打開`isOpen = true`，不一樣的是需要真的檢查牌堆是否為空的情況，避免出現`undefined`的Error
```js
watch(cardStacks, (stacks) => {
    // 檢查每組牌堆最後一張
    ['first', 'second', 'third'].forEach(cardName => {
        if (stacks[cardName].length > 0) {
            const lastCard = stacks[cardName][stacks[cardName].length - 1];
            lastCard.isOpen = true;
        }
    });
});
```
## 限制疊牌順序
這邊先解釋以下在紙牌接龍的中心會有七個`疊牌區`，
疊牌的規局會是不同花色以黑紅交錯，黑牌後面只能接紅牌，紅牌後面只能接黑牌，
且數字必須是前面的減一，不能多也不能少，例如`梅花K`後面只能接`方塊Q`或`紅心Q`
> 例外: 疊牌區若為空則可以放任意花色的K

為了開發方便測試，我決定直接把畫面改成有七個牌堆並且改回垂直疊牌，且牌堆的牌數數量依序為1、2、...7張，
這部分程式就略過。

主要是做了一個**檢查這一張牌是否能放上去**的函數，以刪去法的邏輯實現檢查:
- 卡堆為空，擺放數字不為K則失敗
- 卡牌顏色相同則失敗
- 原卡堆`最後一張的數字減1`等於`擺放數字`則成功，否則失敗

以下是實現的程式碼:
> 程式碼中以回傳布林值`true`、`false`表達成功、失敗
```js
/** 檢查下一張牌是否可以放上去
 * @param {Card[]} targetDeck 
 * @param {Card} card 
 * 
 * @returns {Boolean} 是否可以放上去
 */
function checkNextOk(targetDeck, card) {
    // 如果目標為空牌堆，則只有K可以放上去
    if (targetDeck.length === 0) {
        return (card.value % 13) === 12;
    }
    // 如果目標非空牌堆，則檢查最後一張
    const lastCard = targetDeck[targetDeck.length - 1];
    // 取得兩張牌花色必須一紅一黑
    const lastCardSymbol = Math.floor(lastCard.value / 13);
    const cardSymbol = Math.floor(card.value / 13);
    // 兩張牌顏色相同則移動無效
    if (lastCardSymbol == cardSymbol || (lastCardSymbol + cardSymbol) == 3) {
        return false;
    }
    // 檢查數字是否連續
    return (lastCard.value % 13 - 1) == (card.value % 13)
}
```

## 小結
今天主要是撰寫邏輯去`限制撲克牌的移動`，過程中`推導公式`也蠻開心的，
甚至還為專案補上`@types`強化VSCode的開發體驗，同時也將畫面調整接近紙牌接龍的狀態。

目前持續快兩週的一邊開發一邊寫稿，文章變得更像Side Project的錨點讓我隨時找回開發的方向，而程式碼也在約束文章不能太空泛，這種彼此牽制的開發模式目前感受良好也從中學習許多。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day13
{{< youtube 8Lsjnw_3C34 >}}

**參考**
- [Wiki 紙牌接龍](https://zh.wikipedia.org/zh-tw/接龍_(紙牌遊戲))
- [虛擬加載-typescript-jsdoc-定義檔](https://medium.com/codememo/虛擬加載-typescript-jsdoc-定義檔至-javascript-b01f87acf3e0)