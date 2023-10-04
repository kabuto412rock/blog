---
title: "Day25 紙牌接龍-結算畫面採用Modal和修正移牌優先權"
date: 2023-10-04T03:05:48Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]

draft: false
---
## 前言
今天會調整結算畫面的顯示、修正**連點移牌**的優先權錯誤(應該最優先移入`結算牌堆`而非`七牌堆`)。

## 結算畫面調整
### 安裝套件 bootstrap-vue-next
昨日完成的結算畫面是跳出來的瀏覽器訊息，畫面完全看各家的瀏覽器制式化只能點確認，即使擋住原本的遊戲畫面是我想要的效果，但更想要的是可客製化頁面的互動視窗`Modal`。

雖然可以自己土炮撰寫`Modal`但看帳號名字就知道我很懶，我打算撿現成的套件看能不能快速客製化介面...然後就找到`bootstrap-vue-next`這個套件，聽名字就知道是針對`Vue3`特別拉出來的實現。

先照著官方文件安裝依賴:
```bash
npm i bootstrap bootstrap-vue-next
npm i unplugin-vue-components -D
```
> 這個 **unplugin-vue-components** 主要是方便自動載入有副作用(side effect)的功能到你的元件中，詳細可參考官方說明，畢竟`Bootstrap`有副作用才方便?! 

接著調整`vite.config.js`的內容，主要是在`plugins`屬性對應的陣列中添加`Components 包裹 BootstrapVueNextResolver`的依賴:
```js
// vite.config.js
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite'
import {BootstrapVueNextResolver} from 'unplugin-vue-components/resolvers'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    Components({
      resolvers: [BootstrapVueNextResolver()],
    }),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```
最後在`main.js`載入相關的CSS，記得把`bootstrap`移到上方避免之前撰寫的css被蓋掉:
```js
// main.js/ts
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue-next/dist/bootstrap-vue-next.css'
// 略
```

前面都完成後，發現接龍畫面的卡片的英文、符號都變得比較Q版，上方`導覽列 NavgationBar`CSS有點跑版。
![載入vue-bootstrap後的接龍畫面](/images/itday25-css.png)

### 修改邏輯
從套件`bootstrap-vue-next`引入要使用的`BModal`元件，宣告變數`doneModal`表示顯示結算畫面與否。
```js
// DragDemo.vue
const doneModal = ref(false);
import { BModal } from 'bootstrap-vue-next';
```

使用`<BModal>`元件去包住結束畫面的HTML樣板:
- `v-model`為true則會跳出結算畫面
- `title`為交互視窗的上方標題
- `hide-footer`設定交互視窗的確認取消區塊隱藏
- `@close`設定點擊交互視窗關閉x按鈕時觸發的事件方法
- `@hide`當交互視窗隱藏時觸發的事件方法
```vue
<!-- DragDemo.vue -->
<BModal v-model="doneModal" title="結算畫面" hide-footer @close="resetGame" @hide="resetGame">
    <h1>玩家 xxx</h1>
    <div>花費時間: {{ gameTime }} 秒</div>
    <div>總分數: {{ gameScore }} 分</div>
</BModal>
```

## 修正`點擊自動移牌`的優先權問題
修正自動移動的函數`clickAutoMove`中的`toName`寫法，原本是用`FOUR_SUITS`優先排序接著取第0個元素，
改寫為直接用`Array.find`取得想要優先的牌堆(結算盤堆名稱)，如果沒有才取得第0個元素(7牌堆名稱)。
> 結算牌堆名稱 FOUR_SUITS = ['club', 'diamond', 'heart', 'spade']
```js
/**
 * 自動移動
 * @param {String} fromName 來自的牌堆名稱
 * @param {Card} card 想移動的牌
 */
function clickAutoMove(fromName, card) {
    const toNames = findFollowDeckName(cardStacks, card);
    // 略
    const toName = toNames.find((name) => FOUR_SUITS.includes(name)) ?? toNames[0];
    console.log(`可移動至的牌堆有: ${toNames}, 選擇移動至: ${toName}`);
    // 略
}
```

## 小結
今天使用的`bootstrap-vue-next`因為是`Alpha`版本對應文件還不完整，但像`BModal`的部分參考原`bootstrap-vue`文件的範例發現也是可以通的。

另外寫到這邊才發現如果`Vue3`專案使用`bootstrap-vue`還是有相容支援到`bootstrap v4`，只是需要額外的設定`{ compatConfig: { MODE: 3 }}`，詳細情形可參考[官方文件](https://bootstrap-vue.org/vue3)。



程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day25
{{< youtube ZrEgZ2mySDM >}}

**參考文件**
- https://bootstrap-vue-next.github.io/bootstrap-vue-next/docs.html
- https://bootstrap-vue.org/docs/components/modal 
