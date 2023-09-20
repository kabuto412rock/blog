---
title: "Day 11 拖曳紙牌的效果(三)如何一次拖曳多張卡牌"
date: 2023-09-20T03:25:18Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

今日預計實作只實作如何`一次拖曳` `多張卡牌`

## 多張卡牌拖曳的考察研究
這部分可能會讓很多人(我)失望，因為`vue.draggable.next`最近一次的合併更新在2021年8月，
所以目前`Vue3`無法像原本`Vue2`能使用`vue.draggable`的`Multi-drag`的擴充功能，所以我捨棄使用`套件原生多筆拖曳`的想法和 拖曳多張牌`完美的畫面效果`。

如果願意改變資料結構為巢狀`Vue3`版本還是有辦法對[巢狀物件進行一次性的拖曳](https://sortablejs.github.io/vue.draggable.next/#/nested-example)，但對我來說在未來資料處理的靈活性降低又提高判斷卡牌順序的複雜度因此不考慮。

### 想法邏輯
1. 從`來源牌堆`先拖曳一張牌`A`移動到`目標牌堆`的指定位置
2. 將`來源牌堆`中`A牌`後的剩餘卡片複製到`目標牌堆`的指定位置後方
3. 刪除`來源牌堆`原`A`位置後的剩餘元素
> 實作後發現其實可以先複製一份`來源牌堆`、`目標牌堆`移動後的結果，後續處理會更為靈活。

### 實作邏輯
1. 在`:move`對應的函數中判斷可拖曳時，產生『若拖曳成功後，來源/目標陣列的新狀態』並封裝成一個函數儲存至ref變數`changeOption`。
> 
```js
function limitLocalMove(evt) {
    // 限制同個牌堆無法拖曳
    const result = evt.from !== evt.to;
    if (result) {
        // 取得牌堆的來源、目標名稱，對應reactive`cardStacks`內的名稱
        const from = getDomName(evt.from);
        const to = getDomName(evt.to);
        const draggedContext = evt.draggedContext
        const { index, futureIndex } = draggedContext;
        // 產生多筆拖曳後，來源牌堆、目的牌堆的陣列變動後的結果
        const newFromCards = cardStacks[from].slice(0, index);
        const newToCards = [
            ...cardStacks[to].slice(0, futureIndex),
            ...cardStacks[from].slice(index),
            ...cardStacks[to].slice(futureIndex)
        ];
        // 將變動牌堆的函數暫存，預計等到拖曳完成後執行
        changeOption.value = () => {
            cardStacks[from] = newFromCards;
            cardStacks[to] = newToCards;
            changeOption.value = null;
        };
    }
    // 仍使用原生的拖曳效果
    return result;
}
```
2. 當`@change`事件發生時，若定義ref變數`changeOption`有值則執行第1點暫存的函數，將最終結果更新對應的牌堆陣列之中。
> `@change`只要在卡牌拖曳完成對陣列產生異動時才會自動被執行，此為`draggable`原始機制
 ```js
function cardChange(event) {
    if (changeOption.value) {
        changeOption.value();
        changeOption.value = null;
    } else {
        console.log(`no trigger changeOption`);
    };
}
```
3. 樣板調整除了`<draggable>`的屬性`:move`、`@change`之外，眼尖的朋友可以注意到兩個`<draggable>`分別添加了`ref="first"` 和`ref="second"`對應到第一/第二牌堆的元素，主要是為了第1點在`:move`函數執行中可以判斷來源和目標陣列對應到`cardStacks`內的哪一個陣列。
```vue
<GameBoard>
    <div>
        <h4 class="title">牌堆1</h4>
        <draggable :list="cardStacks.first" group="pokers" itemKey="value"
            style="display: grid; grid-template-columns: repeat(13, 3rem); background-color: yellow;"
            :move="limitLocalMove" @change="cardChange" ref="first">
            <template #item="{ element, index }">
                <Card :value="element.value" :isOpen="element.isOpen" />
            </template>
        </draggable>
    </div>
    <div>
        <h4 class="title">牌堆2</h4>
        <draggable :list="cardStacks.second" group="pokers" itemKey="value"
            style="display: grid; grid-template-columns: repeat(13, 3rem); background-color: yellow;padding: 1px;"
            :move="limitLocalMove" @change="cardChange" ref="second">
            <template #item="{ element, index }">
                <Card :value="element.value" :isOpen="element.isOpen" />
            </template>
        </draggable>
    </div>
</GameBoard>
```
因為`:move`本身攜帶的資訊無法得到拖曳來源、目標的`:list`，但卻有紀錄拖曳列表來源、目標的DOM(`from`,`to`)，所以撰寫一個函數可取得DOM所對應的牌組名稱。
```js
function getDomName(dom) {
    if (dom == first.value.targetDomElement) {
        return 'first';
    } else if (dom == second.value.targetDomElement) {
        return 'second';
    } else {
        return 'none';
    }
}
```
上方的`first`/`second`都有先定義成`ref(null)`否則執行可是會失敗的喔!  
如下所示:
```html
<script setup>
const first = ref(null); 
const second = ref(null);
// ...
</script>
```

## 小結
雖然原本除了多筆拖曳還想要額外判斷特定順序，但花了蠻多時間在`vue.draggable.next`要如何正常實現畫面，
最終還是只撰寫/實作多筆拖曳的功能。

實作前思考最理想的狀態，實作中則逐步去做取捨先挑重要的去做，若一昧追求完美便無法完成作品/寫稿，但身為軟體工程師同時需要考慮到作品的未來運行和功能上的調整。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day11
{{< youtube KI6fuhtqZHk >}}

**參考**
- [SortableJS/Sortable](https://github.com/SortableJS/Sortable#event-object-demo)
- [SortableJS/vue.draggable.next](https://github.com/SortableJS/vue.draggable.next)