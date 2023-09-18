---
title: "Day 09 拖曳紙牌的效果(一)"
date: 2023-09-18T05:13:22Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

玩紙牌接龍最重要的就是卡牌會移來移去，之前都是用點的移動定點，
今天來試試看如何撰寫拖曳紙牌的功能。

## 安裝依賴
因為重頭學習理解拖曳，對我來說太麻煩也太無聊，
乾脆就使用現成的套件`Vue.Draggable`吧!

因為使用的是**Vue3**專案，所以必須安裝有標註**next**的版本。
```bash
npm i -S vuedraggable@next
```

## 單一列表的拖曳使用
我也還在理解該套件中，接下來的過程會盡可能去蕪存菁，但不失細節。  
首先一定要先引入`vuedraggable`，如下所示:
```vue
<script setup>
import draggable from 'vuedraggable'
// ...other template
</script>
```
樣板的部分則如下:
```vue
<template>
    <draggable :list="firstCardStack" itemKey="value"
        @change="console.log"
        style="display: grid; grid-template-columns: repeat(13, 3rem);">
        <template #item="{ element, index }">
            <Card :value="element.value" :isOpen="element.isOpen" />
        </template>
    </draggable>
</template>
```
這樣的寫法在通常就已可運用，接下來讓我們來逐一理解`<draggable>`每個欄位的意義和預設行為...

### :list
首先`:list`內設定的是一個參考到陣列的`ref`或`reactive`變數，只要裡面參考到的是Array即可，
這可以讓`<draggable>`明白當**列表項目被拖曳移動**時自動修改的對象。
```js
// 此處genearateDeck(20, true)會回傳20個物件的Array物件
const firstCardStack = ref(geneateDeck(20, true));
// 為求簡單也能寫成底下這樣
// const firstCardStack = ref([]);
```

### itemKey屬性
第二個重要的屬性是`itemKey`，代表前面陣列中每一個元素的唯一值，基本上就跟`v-for`內用到的`:key`有相同作用，可讓元件從`itemKey`明白內部元素的差異進而去做列表更新。

此處會設定`itemKey="value"`，是因為參考的陣列`firstCardStack`每一個元素的構造如下: 
```js
{ 
    value: number,  // 撲克牌值，Ex: 0 對應 ♣A
    isOpen: boolean // 開牌狀態，Ex: false 對應 蓋牌樣式
}
```
> 嘗試過不添加`itemKey`仍可拖曳且參考的陣列有更新，但會發生UI不會刷新的窘境🤣

### @change屬性
目前先使用`console.log`，可印出包含欄位`moved`的物件，`moved`內的構造如下:
```js
{
    newIndex: 13 // 拖曳後項目的新索引
    oldIndex: 0 // 拖曳項目的原索引
}
```

## `<draggable>`內部
傳進`draggable`內部的slot區塊，這部分我在重寫一次如下:
```vue
<template #item="{ element, index }">
    <Card :value="element.value" :isOpen="element.isOpen" />
</template>
```

可以方便理解成只是從下方`v-for`寫法換成上方的兩層式寫法:
```js
<Card v-for="(element, index) in firstCardStack" :value="element.value" :isOpen="element.isOpen" />
```
外層的`<template>`的屬性`#item`只是省略了要迭代哪個陣列的`v-for`寫法而已，至於為什麼要這樣設計就要問原作者了哈哈ˊWˋ

## 小結
目前的卡牌列表是可以任意切換順序，但真實的接龍是不能這樣隨意拖曳，
明日再來研究如何限制列表的拖曳、兩個列表的拖曳。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day09
{{< youtube ICQ-Lp9HIKw >}}
**參考資料**
- [vue.draggable的Github](https://github.com/SortableJS/vue.draggable.next)
- [vue.draggable 兩列表交換範例(SFC)](https://github.com/SortableJS/vue.draggable.next/blob/master/example/components/two-lists.vue)