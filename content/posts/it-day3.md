---
title: "Day 03 完成卡牌自動翻面的效果，但還不完整"
date: 2023-09-12T03:51:05Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
原本今天預計製作發牌和卡牌自動翻面的效果，希望兩個都能有動畫效果，
但現實的能力讓我想先說說今天的開發方式
  
# 實際思考/開發的過程
## 卡牌翻面
### 想像的實作過程
1. 沿用原本的`Card.vue`內的樣板，在裡面分成兩個div區塊，
一個div區塊放正面有花色數字的牌面，另一個則放一張背圖。
2. 在`Card.vue`內撰寫 **@click** 去改變卡片的狀態
3. 撰寫CSS的動畫去實現翻面的過程
### 現實開發
1. 先調整撲克牌背面的樣板，花了些時間讓圖片不會超出卡牌寬高 `background-size: cover;`
2. 上網查詢是怎麼做翻轉撲克的效果，甚至查到有人做出6個div欄位做出[3D方塊](https://eyesofkids.gitbooks.io/css3/content/contents/transform3d.html)，但嘗試改成只有前後的div欄位但發現景深會產生兩張卡片交疊的一點位移就放棄這個寫法
3. 意外查到IT邦上有個純CSS挑戰撲克翻轉，所以就先照著原作的CSS翻轉寫法調整Card.vue樣板
4. 並且添加`@click` 會觸發`emit`將點到的卡牌數字回傳至上層進行更新
```jsx
// Card.vue
<template>
    <div class="card " @click="emit('poker-flip', value)" :class="backCardClass">
        <div class="card-front" :class="numberClass">{{ content }}</div>
        <div class="card-back"></div>
    </div>
</template>
```
5. 因為我Vue3是採用[SFC(Single-File Components)](https://vuejs.org/guide/scaling-up/sfc.html#single-file-components)的寫法，所以定義emit的方式如下所示:
    ```jsx
    // Card.vue
    const emit = defineEmits(["poker-flip"]);
    ```
    在上層的元件要接會使用剛剛定義在defineEmits的名稱 `@poker-flip` 
    ```jsx
    // GameBoard.vue 其他都略
    <Card v-for="card in boardCards" key="card.value" :value="card.value" :isOpen="card.isOpen"
    @poker-flip="toggleFlip" />
    ```
    可能因為練習過ReactJS官網的的關係，知道狀態提升(Lifting State Up)的概念，所以目前所有卡牌資料依然是先放在上層的GameBoard.vue裡面，`toggleFlip`是我撰寫
    ```jsx
    // GameBoard.vue
    function toggleFlip(num) {
        // 找到對應的開牌狀態且翻轉
        const targetIdx = boardCards.value.findIndex((item) => item.value === num)
        boardCards.value[targetIdx].isOpen = !boardCards.value[targetIdx].isOpen;
    }
    ```
    > 今天也有犯一點蠢，發現開牌的狀態一直傳不下去，但toggleFlip函數又有拿到對應的數字和改變的狀態，  
    > 原來是GameBoard.vue中寫的 `<Card />` 元件沒寫屬性 `:isOpen` 導致程式沒報錯，但有執行異常的情況...

### 今日進度
- 卡牌翻面有了，點擊後可翻開/覆蓋卡牌，但CSS水平反轉的動畫只套用到蓋牌(開牌就沒動畫)，
  雖然有發現Vue3的`Transition`寫法但奈何悟性不足今天無法應用在上面
  > Vue3的Transition的寫法也是ChatGPT跟我說的，雖然是錯的居多(範例給我 `<transition>` 但第一字母其實應該大寫才對XD)，但身為前端菜逼八也是先把它當成關鍵字搜尋引擎使用
- 發牌排到明天再做

{{< youtube z-89io-k1vk >}}

**參考資料**
- [2018鐵人賽文章-撲克翻轉](https://ithelp.ithome.com.tw/articles/10204869)
