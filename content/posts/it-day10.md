---
title: "Day 10 拖曳紙牌的效果(二)限制內部拖曳"
date: 2023-09-19T03:49:59Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

今天來研究`兩個列表的拖曳`和`如何限制列表的拖曳`，學習內容主要來自`vue.draggable`文件的內容和親身實作進行分析。

## 先實現兩個列表的拖曳
昨天已學過如何製作單一列表的拖曳，先將相同的`<draggable>`內容複製往下貼，
複製的另一個`<draggable>`只需要調整屬性`:list`成想要對應的另一個陣列，並且兩個`<draggable>`添加相同的屬性`group`則自動實現兩個列表拖曳的關聯。
```vue
<template>
    <div>
        <draggable :list="firstCardStack" group="pokers" itemKey="value">
            <template #item="{ element, index }">
                <Card :value="element.value" :isOpen="element.isOpen" />
            </template>
        </draggable>
    </div>
    <div>
        <draggable :list="secondCardStack" group="pokers" itemKey="value">
            <template #item="{ element, index }">
                <Card :value="element.value" :isOpen="element.isOpen" />
            </template>
        </draggable>
    </div>
</template>
```

## 如何限制列表的拖曳
前面已先實現了兩列表的拖曳，但為求理解仍保留著`@change="console.log"`，在限制拖曳之前，我必須先觀察拖曳完成前能得到的資訊，所以我必須先觀察變動後`@change`實際印了什麼有用的資訊。

因為如果只是要單純限制一個`<draggable>`不能拖曳移入移出，只需要設定屬性`:move="() => false"`，但我想做的不僅僅如此。

### 不同牌堆的移動
將牌堆1的梅花A移動到牌堆2後，可觀察到開發人員工具的Console依序印出兩個物件:
1. 內含`added`屬性，且added物件本身包含陣列移動的元素`element`和目標陣列內的新位置`newIndex`
2. 內含`removed`屬性，且removed物件本身包含陣列移動的元素`element`和原陣列內的舊位置`oldIndex`
![從牌堆1移動到牌堆2則會觸發change的added和remove事件](/images/20230919實驗一.png)
> 這兩個@change是由各自受影響的Draggable所觸發，但內含的元素是相同的

### 同牌堆的移動
將同牌堆的第三張牌(梅花3)移動到第六張牌(梅花6)的位置，則只印出一個包含屬性`moved`的物件，
且moved物件本身包含陣列移動的元素本身`element`和新位置`newIndex`和舊位置`oldIndex`的資訊。
![同一個牌堆內部移動只會觸發change的moved事件](/images/20230919實驗二.png)

> 從此可推測，如果要禁止同牌堆內的移動，只需要禁止`moved`的行為。

### 禁止同牌堆的內部拖曳

從官方文件中可查到屬性`:move`對應的函數只要回傳`false`即可取消此次的拖曳。
```js
function onMoveCallback(evt, originalEvent){
   ...
    // return false; — for cancel
}
```
這邊我只印出參數`evt`得到的資料，內容跟`@change`不一樣的超乎想像多資訊😫，而且跟`@change`不同的是，這個`:move`對應函數只要拖曳卡牌到某一張卡的上面就會觸發一次
![:move內參數evt印出的資訊超多](/images/20230919move內參數evt.png)

但好險...如果只需要判斷是否在同個牌堆只需要看evt內的屬性`from`, `to`是否為相同DOM元素即可，
因為`from`、`to`分別代表拖曳的元素是 **來自哪一個draggable元件** 和 **即將放置在哪一個draggable元件**。
```js
function limitLocalMove(evt, originalEvent) {
    // 限制同個牌堆無法拖曳
    return evt.from !== evt.to;
}
```

## 小結
今天慢慢花時間理解也希望能幫助打算使用`vue.draggable`，也理解如何限制拖曳發生。

明天預計實作牌堆需要在特定順序才能進行拖曳、如何一次拖曳多筆元素。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day10
{{< youtube 1jFp0j0CjJ0 >}}

**參考**
- [vue.draggable.next#move](https://github.com/SortableJS/vue.draggable.next#move)