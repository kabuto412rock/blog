---
title: "Day 18 實作結算牌堆元件(二) 整合拖曳相容不同規則"
date: 2023-09-27T06:51:23Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
昨天完成`結算牌堆`樣版的部分，今天接著處理整合拖曳到接龍之前的步驟，
先調整`結算牌堆`的內部結構、方法。

## 調整結算牌堆 FinishedArea
先調整結算牌堆`<FinishedArea />`內部需要的`props`結構，就底下這三種:
- `props.fourCards`對應四個牌堆的陣列
- `props.moveCard`對應`<draggable>`元件的屬性`:move`判斷是否可以拖曳成功的函數
- `props.change`對應`<draggable>`內的列表更新時觸發的函數(這部分等等再說明)

程式碼如下:
```js
// FinishedArea.vue
const props = defineProps({
    fourCards: {
        type: Object,
        required: true,
        validator: (value) => {
            return (
                value.hasOwnProperty('club') &&
                value.hasOwnProperty('diamond') &&
                value.hasOwnProperty('heart') &&
                value.hasOwnProperty('spade')
            );
        },
    },
    moveCard: {
        type: Function,
        default: () => { return false; }
    },
    change: {
        type: Function,
        default: () => { return false; }
    }
})
```
然後結算牌堆就像發牌區一樣，卡牌陣列都是由外部`props`傳入且props的陣列內容都會變動，
所以也需要`watch`監控**props.fourCards**變化調整元件內的`fourCards`內的值。
```js
// FinishedArea.vue
const fourCards = ref({
    club: [],
    diamond: [],
    heart: [],
    spade: [],
});
watch(props.fourCards, (newVal) => {
    fourCards.value = newVal;
});
```

另外設計一個用來將對應HTML元素(HTML Element)傳回父元件的`emit`事件，此事件只會在元件渲染成功時觸發一次，發送**doms**主要是讓父元件可以判斷是被拖曳至哪一個牌堆。
```js
// FinishedArea.vue
const club = ref(null);
const diamond = ref(null);
const heart = ref(null);
const spade = ref(null);
const emit = defineEmits(['doms']);
onMounted(() => {
    emit('doms', {
        club: club.value.targetDomElement,
        diamond: diamond.value.targetDomElement,
        heart: heart.value.targetDomElement,
        spade: spade.value.targetDomElement,
    })
})
```
## 整合拖曳相容不同規則
為了避免在嘗試的過程中影響先前的`發牌區`和七牌堆，
所以決定先將`結算牌堆 <FinishedArea>`跟`發牌區 <DealerArea>`整合在另一個頁面避免產生衝突。

樣板構造很簡單:
```vue
<!-- DealerAreaView.vue -->
<template>
    <main>
        <GameBoard>
            <div class="text">發牌區</div>
            <DealerArea :deck="deck" :moveCard="moveInFourCards" />
            <div class="text">結算牌堆</div>
            <FinishedArea :fourCards="fourCards" @doms="setFourCardDoms" :change="cardChange" />
        </GameBoard>
    </main>
</template>
```

資料處理的部分都放父元件`DealerAreaView.vue`裡面比較複雜一點，以下會先從簡單開始。

### 儲存對應結算牌堆內的HTM元素
我先宣告一個陣列`FOUT_SUITS`負責儲存對應結算的花色(梅花->黑桃)，
函數`setFouCardDoms`會在子元件觸發**@doms**事件回傳的名稱對應HTML元素的字典，存放在變數`fourCardsDom`提供後續拖曳判斷。
```js
// DealerAreaView.vue
const FOUR_SUITS = Object.freeze(['club', 'diamond', 'heart', 'spade']);
const fourCardsDom = reactive({
    club: null,
    diamond: null,
    heart: null,
    spade: null,
});
function setFourCardDoms(cardDomMaps) {
    FOUR_SUITS.forEach(name => {
        const domElement = cardDomMaps[name];
        fourCardsDom[name] = domElement;
    });
}
```
### 拖曳至結算牌堆
因為除了判斷是否可拖曳也需要設定拖曳後的資料處理，以下是函數`moveInFourCards(evt)`的步驟:
1. 首先判斷拖曳到的位置是結算牌堆的哪個花色`deckColor`? null 則不可拖曳
2. 判斷拖曳至牌堆的規則(必須是同花色的下一個數字)有點複雜我就包在函數`checkNextOk2`? false 則不可拖曳
3. 如果前面都判斷可拖曳，則先計算出來源、目標牌堆的變動結果並將修改執行包在箭頭函數，暫存至全局變數`changeOption`
> `changeOption`內的箭頭函數會在卡牌拖曳成功後，在函數`cardChange()`中被呼叫
```js 
/** 將發牌區的牌拖曳至 結算牌堆 中 */
function moveInFourCards(evt) {
    console.log(evt);
    const { element, index } = evt.draggedContext;

    // 檢查是否拖曳至 結算牌堆
    const { to } = evt;
    let deckColor = null;
    FOUR_SUITS.forEach(name => {
        if (fourCardsDom[name] === to) {
            deckColor = name;
        }
    });
    let result = deckColor != null;
    result = result && checkNextOk2(deckColor, element);
    if (result) {
        const fromCards = deck.value.filter((card) => card.value != element.value)
        const toCards = [...fourCards[deckColor], element];
        changeOption.value = () => {
            deck.value = fromCards;
            fourCards[deckColor] = toCards;
            changeOption.value = null;
        };
    }
    return result;
}
/** 檢查下一張牌是否可以放上去`集牌堆`
 * @param {String} pokerColor 牌堆花色
 * @param {Card} card 要放上去的牌
 * 
 * @returns {Boolean} 是否可以放上去
 */
function checkNextOk2(pokerColor, card) {
    const pokerColorIndex = Math.floor(card.value / 13);
    const pokerNumber = card.value % 13;
    const pokerColorName = FOUR_SUITS[pokerColorIndex];

    if (pokerColorName !== pokerColor) {
        return false;
    }
    const deckCards = fourCards[pokerColor];
    if (deckCards.length === 0 && pokerNumber !== 0) {
        return false;
    }

    const lastCardNumber = deckCards[deckCards.length - 1].value % 13;
    return lastCardNumber + 1 === pokerNumber;
}
```

## 小結
雖然還沒完全整合至接龍，但至少撰寫一個新的判斷規則函數 和 利用`emit`傳送HTML元素給上層判斷，這部分也是之後整合進接龍或許還需要`emit`再出場一次😂。

題外話:最近看Godot的官方文件提到`signal`時的提供的[參考文件Observer](https://gameprogrammingpatterns.com/observer.html)中有提到遊戲最常使用的模式**觀察者模式**，其實就跟Vue的`emit`一樣即使沒有人接收也會執行發送，但有訂閱的物件就會收到回應。

明天再來繼續加油!👍👍👍

程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day18
{{< youtube ima8w442psc >}}

