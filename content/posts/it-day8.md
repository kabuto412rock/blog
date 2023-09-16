---
title: "Day 08 牌堆的卡牌移動動畫"
date: 2023-09-16T07:22:58Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 重點提醒: 沒有真的成功實作卡牌無中斷的移動
在實作的過程中遇到了一些問題，找到替代方案先記錄下來。

## 最初的思路
在前一篇文章中，我們已經完成了牌堆的製作，接下來我們要來製作牌堆的卡牌移動動畫。
在開始實作之前，我必須決定要用CSS或是JS實現?

如果牌堆的位置永遠是固定的，或許可以用CSS來實現，但是我們要移動的是牌堆位置不一定永遠在那，
而且牌堆的卡牌數量是不固定的，更加深了預設的座標位置，所以我想這必須用JS來實現，
至少需要JS取得動態元素的位置。

## 動畫的實作
第一個問題對我來說是，我要怎麼知道卡牌的位置?
起始和結束的位置都是不固定的，所以我必須要先取得卡牌的位置，才能夠進行動畫的實作。

取得當前牌堆的位置的方法有很多種，我在這邊使用的是`getBoundingClientRect()`，
這個方法可以取得當前元素的相對位置+寬高，但是這個方法是在DOM上的，所以我利用`const cardBox = ref(null)`設定一個參考，
Vue3會自動找到template中使用`ref="cardBox"`的元素，並且將其綁定到`cardBox`上，這樣我就可以在`<script setup>`中使用`cardBox.value`來取得HTML元素本身。

因為元件渲染掛載畫面上便會觸發`onMounted`事件，所以我在`onMounted`中取得元素的位置，並且透過`emit`傳遞卡片的絕對位置給父元件。

> 至於為什麼是絕對位置，因為我們要移動的是卡片，而不是牌堆，所以我們必須要知道卡片的絕對位置，才能夠進行移動。
> 雖然後來我失敗了，但是我還是想記錄一下這個方法，因為我覺得這個方法還是很有用的。
```html
// CardColumn.vue
<script setup>
// ...
const cardBox = ref(null);
onMounted(() => {
    const rect = cardBox.value.getBoundingClientRect();
    const x = window.scrollX + rect.left;
    const y = window.scrollY + rect.top;
    emit('position', { x, y });
})
</script>
<template >
    <div class="card-box" :class="{ 'empty-card-box': isEmpty }">
        <div class="card" style="visibility: hidden;"></div>
        <div style="visibility: visible; position: absolute; z-index: 5;" ref="cardBox">
            <div style="; display: grid; grid-template-rows: repeat(13, 2rem);">
                <Card v-for="(card, index) in cards" @click="(event) => onClick(event.target)" :key="card.value"
                    :value="card.value" :isOpen="card.isOpen" />
            </div>
        </div>
    </div>
</template>
```

然後在上層`HomeView.vue`就寫`@position`接收到x,y位置變數後，基礎工作就完成了。
```html
// HomeView.vue
<script setup>
// 第1, 2個卡堆的位置
const fstCardsPos = ref({ x: 0, y: 0 });
const secondCardsPos = ref({ x: 0, y: 0 });
</script>
<template>
    <CardColumn :onClick="moveCardFromAToB" :cards="fstCards" @position="(pos) => fstCardsPos = pos"></CardColumn>
</template>
```
## 悲劇開始、希望的結尾
打算在HomeView平常隱藏一個卡片，當卡片從卡堆移動出來時就顯示，並嘗試使用`requestAnimationFrame`並慢慢移動卡片的絕對位置，
但是我太天真了，程式碼變得一蹋糊塗，浪費一堆時間還沒有成功。
> requestAnimationFrame 似乎更適合用在Canvas動畫的更新上，而不是DOM移動的實作上

所以我決定先放棄這個方法，改用Vue3的`Transition-group`來更簡單實現。

基本上就是在`<TransitionGroup>`包裹`<Card v-for="..." />`來渲染卡片，並且在`<TransitionGroup>`中設定`name`屬性對應動畫class名稱的前綴 **fade**，還有因為`<TransitionGroup>`預設會渲染一個`<span>`，但是我們要渲染的是`<div>`，所以要設定tag為`<div>`。
```html
<template >
    <div class="card-box" :class="{ 'empty-card-box': isEmpty }">
        <div class="card" style="visibility: hidden;"></div>
        <div style="visibility: visible; position: absolute; z-index: 5;" ref="cardBox">
            <TransitionGroup name="fade" tag="div" style="; display: grid; grid-template-rows: repeat(13, 2rem);">
                <Card v-for="(card, index) in cards" @click="(event) => onClick(event.target)" :key="card.value"
                    :value="card.value" :isOpen="card.isOpen" />
            </TransitionGroup>
        </div>
    </div>
</template>
<style scoped>
/* 定義動畫的過渡效果 */
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.5s;
    transform: translateX(3rem);
}

.fade-enter,
.fade-leave-to {
    opacity: 0;
}
</style>
```

## 結論
雖然最後畫面上的卡片流暢的移動效果沒有實現，但還接觸到`requestAnimationFrame`、`getBoundingClientRect()`、`TransitionGroup`等等之前沒用過的知識，所以我覺得這次的實作還是很有收穫的。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day08
{{< youtube VBb-1RuIEIk >}}
## 參考資料
- [Shubo的程式開發筆記-深入了解 getBoundingClientRect() 函數，輕鬆獲得元素的大小和位置
](https://www.shubo.io/get-bounding-client-rect/)
- [Vue官網文件 transition-group](https://vuejs.org/guide/built-ins/transition-group.html)
- [30天實作線上簡報播放機制-使用requestAnimationFrame()](https://ithelp.ithome.com.tw/articles/10186735)