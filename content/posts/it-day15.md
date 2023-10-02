---
title: "Day15 接龍發牌區功能實作(二)拖曳功能及CSS發牌動畫"
date: 2023-09-23T12:19:58Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---

## 前言
今天要來補強昨日缺少的`拖曳功能`和將整個頁面封裝成提供整合至接龍遊戲的元件`發牌區`。

## 移牌區改為可拖曳

首先將`移牌區`的部分調整成從原生HTML元素`div`替換成這幾天都在使用`draggable`元件，程式碼如下:
```html
<!-- 移牌區 - 左邊水平疊牌最多三張 -->
<draggable :list="canTakeCards" group="pokers" itemKey="value" class="list-group" >
    <template #item="{ element, index }">
        <Card :value="element.value" :isOpen="element.isOpen" />
    </template>
</draggable>
```

## 發牌區調整動畫
雖然`移牌區`感覺在加牌的時候應該要套用動畫，但無奈`vue.draggable.next`的transition-group有Bug存在且已被官方棄養，在issue也有許多類似問題[issue with transition-group in composition API](https://github.com/SortableJS/vue.draggable.next/issues/188)就不細講。

不論是`Vue2使用的vue.draggable`或`Vue3使用的vue.draggable.next`都是對[SortableJS](https://sortablejs.github.io/Sortable/)進行包裝的容器，基本上不滿意的話只能自己重封裝實現只是CP值不高，所以我不考慮這麼做。

這不代表選擇放棄動畫，打算以**CSS**來實現以下**動畫效果**。
![發牌向左位置&左右搖晃的GIF](/images/點擊發牌的浮動.gif)
### 實際調整
首先調整樣板程式碼，主要是針對有牌時添加對應的class`"card-back animtion"`，也順便將點擊事件`@click`移到外層div:
```jsx
<!-- 發牌堆 -->
<div class="card-box">
    <div class="card " style="visibility: hidden;">
        <div style="visibility: visible; width: 100%;height: 100%;" @click="clickCard">
            <Transition name="slide-left">
                <div v-if="deckState == 'empty'">無牌可用</div>
                <div v-else-if="deckState == 'full'" class="card">重新循環</div>
                <div v-else-if="deckState == 'normal'" class="card-back animation"></div>
            </Transition>
        </div>
    </div>
</div>
```
添加對應`CSS`實現點擊後往左快速位移一次的效果
```css
@keyframes move-left {
    from {
        transform: translateX(0rem);
    }

    to {
        transform: translateX(-100rem);
    }
}

.card-back.animation:active {
    animation: move-left 0.55s ease;
    animation-iteration-count: 1;
}
```
添加對應`CSS`當滑鼠移到`發牌堆`上方使其左右搖晃的效果
```css
@keyframes swing {
    0% {
        transform: rotate(-5deg);
    }

    50% {
        transform: rotate(5deg);
    }

    100% {
        transform: rotate(-5deg);
    }
}

.card-back:hover {
    cursor: pointer;
    animation: swing 1s ease infinite;
}
```
## 封裝成元件
在目錄 **src/components** 內新增檔案`DealerArea.vue`，並將`DealerAreaView.vue`的程式碼完全複製到新檔案`DealerArea.vue`，
但需要調整卡牌陣列`deck`從外部取得，所以`DealerArea.vue`程式碼需將變數`deck`改寫成以下形式:
```js
// DealerArea.vue
const { deck } = defineProps(
    {
        deck: {
            type: Array,
            default: () => []
        }
    }
)
```

### 實際在外部引用
因為卡牌是在`onMounted`時才會初始化`cardStacks`內的每個陣列，
但不清楚為什麼初始化結束後，`<DealerArea />`仍顯示無牌可用。
```vue
/** DragDemo.vue */
<DealerArea :deck="cardStacks.dealerStacks" />
```

查詢網路資料後，才知道原來子組件`props`如果會整個異動的話需要用`watch`去監控變化才會重新觸發渲染，
所以再次調整`DealerArea.vue`的程式碼:
```js
// DealerArea.vue
const props = defineProps(
    {
        deck: {
            type: Array,
            default: () => []
        },
        moveCard: {
            type: Function,
            default: () => { return false; }
        }
    }
)
const deck = ref([]);
watch(() => props.deck, (newVal) => {
    deck.value = newVal;
});
```

添加上方程式碼後，這裡`<DealerArea>`已可以根據`props.deck`的變動去做相對應的畫面更新。

但可以注意到`defineProps`突然多出`moveCard`屬性，實際上這是實作元件`<DealerArea>`忘記考慮被拖曳到其他區塊的行為。

所以再次調整`DealerArea.vue`的樣板，將`<draggable`的屬性`:move`對應到函數`props.moveCard`製作額外判斷是否可以被移出的函數，目前預設為回傳`false`的函數可以阻止元件內部`<draggable>`內的元素被移入到別的`<draggable`元件。
> 在樣板中使用`props`中的屬性`moveCard`為空，則會自動套用props屬性定義的預設值`default`
```html
// DealerArea.vue
<!-- 移牌區 - 左邊水平疊牌最多三張 -->
<draggable :list="canTakeCards" group="pokers" itemKey="value" class="list-group" :move="moveCard">
    <template #item="{ element, index }">
        <Card :value="element.value" :isOpen="element.isOpen" />
    </template>
</draggable>
```

## 小結
今天將昨天的頁面轉成元件`DealerArea`並添加一些**CSS動畫**，也學到子元件需要監聽外部`props`變動重新渲染的前端做法。
明日將實作讓`<DealerArea>`內移牌區內的牌有辦法實際拖曳至中間的7個牌堆上並一樣遵守拖曳的遊戲規則。

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day15
{{< youtube -OuTCuUUTHk >}}


## 參考
- [在 CSS 中模擬 Onclick 事件](https://www.delftstack.com/zh-tw/howto/css/css-onclick/)
- [CSS左右搖晃](https://codepen.io/ChrisMla/pen/jWMLWY)
- [Re-render a component when prop changes](https://www.reddit.com/r/vuejs/comments/w72g4y/rerender_a_component_when_prop_changes/)