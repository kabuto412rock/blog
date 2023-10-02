---
title: "Day16 接龍發牌區功能實作(三)拖曳與疊牌區整合"
date: 2023-09-25T07:05:48Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今天要來處理`<DealerArea>`內的元素如何整合拖曳移動到7個牌堆，
這部分會拆成2個部分來看:
- 可將`<DealerArea>`卡牌移入至牌堆上方，且遵守移動遊戲規則的條件
- 其他牌堆的牌無法移入`<DealerArea>`牌堆中

## 實作將`<DealerArea>`卡牌移入牌堆
這邊在`DragDemo.vue`中設定提供給發牌區`<DealerArea>`判斷用的:move函數，因為這個函數不像函數`limitLocalMove`函數是同時給7個牌堆各自使用，所以可以看到沒有特別提到`from`的部分，因為不用特別想就可以知道一定是來自(`from`)發牌區。
> 如果不清楚`limitLocalMove`是做什麼的，這部分從 [第10天](#day-10-拖曳紙牌的效果二限制內部拖曳postsit-day10)~[第12天](#day-12-實作拖曳卡牌只能置放到目標牌堆的牌尾蓋牌無法拖曳postsit-day12) 的文章都有提到，主要是對應`<draggable>`元件的`:move`函數判斷使否可拖曳成功。

檢查疊牌遊戲規則的部分就由函數`checkNextOk`判斷，幾天前寫好的函數重新複用了👍
這邊只要`result`回傳`true`就會讓卡牌移動自動產生變化。

```js
// DragDemo.vue
/** 發牌區移動 */
function dealerMove(evt) {
    const to = getDomName(evt.to);
    const dealerCard = evt.draggedContext.element;

    // 檢查疊牌順序、花色是否正確
    const result = checkNextOk(cardStacks[to], dealerCard);
    return result;
}
```

樣板的部分就是小調整而已:
```vue
// DragDemo.vue
<DealerArea :deck="cardStacks.dealerStacks" :moveCard="dealerMove" />
```
結果看起來拖曳過去是沒問題，但本該移動過去的元素也仍留在原地，
如下圖GIF出現了2個梅花9，此為禁忌的二重身問題💀必須修正。
![Day16有Bug的拖曳.gif](/images/Day16有Bug的拖曳.gif)

## 修正陰魂不散的元素
因為還是有拖曳成功，只是舊資料殘留在發牌區的陣列`cardStacks.dealerStacks`，所以只要在拖曳完成時，將發牌區的陣列去除已經發出去的那一張牌即可，以下是修正後的程式碼:
```js
// DragDemo.vue
function dealerMove(evt) {
    const to = getDomName(evt.to);
    const dealerCard = evt.draggedContext.element;

    // 檢查疊牌順序、花色是否正確
    const result = checkNextOk(cardStacks[to], dealerCard);
    if (result) {
        changeOption.value = () => {
            cardStacks.dealerStacks = cardStacks.dealerStacks.filter(card => card.value !== dealerCard.value);
            changeOption.value = null;
        };
    }
    return result;
}
```
> `changeOption`的使用在[第11天](#day-11-拖曳紙牌的效果三如何一次拖曳多張卡牌postsit-day11實作邏輯)有提到的。

## 小結
今天主要就是完成單方面的拖曳和修Bug，本來還想說要如何防止其他卡堆的牌移入`<DealerArea>`的牌堆中，
後來發現這件事不會發生。

反而是實現`<DealerArea>`專屬的函數`dealerMove`後意外發現的Bug-二重身，
這可能是因為`<DealerArea>`內拖曳的撲克牌是使用`computed`計算出來的陣列，被移除了元素也不會影響原本上游的陣列。

明日預計實作 `結算牌堆`，敬請期待!

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day16
{{< youtube cN90ZJEpdP4 >}}

**參考**
- #### [Day 10 拖曳紙牌的效果(二)限制內部拖曳](/posts/it-day10)
- #### [Day 11 拖曳紙牌的效果(三)如何一次拖曳多張卡牌](/posts/it-day11/#實作邏輯)
- #### [Day 12 實作拖曳卡牌只能置放到目標牌堆的牌尾、蓋牌無法拖曳](/posts/it-day12/)