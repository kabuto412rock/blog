---
title: "Day 05 引入Vue Router切換頁面"
date: 2023-09-14T03:20:17Z
# weight: 1
# aliases: ["/first"]
tags: ["challenge"]
draft: false
---
因為想留著昨天完成的撲克牌連連看，預計將不同遊戲的頁面可以做保留並切換，
所以我打算在做牌堆之前，首先應該要了解Vue3在路由頁面的實作是如何切換頁面。

## 引入 Vue Router
接下來步驟我是參考『直接使用`npm create vue@latest`指令產生預設攜帶有用`Vue Router`專案的檔案架構』
下去調整的，所以步驟草率請敬請見諒。
1. 安裝依賴`Vue Router`
```bash
npm install vue-router@4
```
2. 在src底下新增兩個資料夾`views`和`router`
3. 在`views`資料夾底下新增兩個頁面檔`Game1View.vue`和`HomeView.vue`
```bash
src
├─App.vue
├─main.js
├─views
|   ├─Game1View.vue # 撲克連連看
|   └─HomeView.vue  # 首頁
├─utils
|   ├─constants.js
|   └─poker-helper.js
├─router
|   └─index.js
├─components
|     ├─Card.vue
|     ├─FoxyHeader.vue
|     ├─GameBoard.vue
| 略... 
```
> 如果想知道是怎麼產生樹狀目錄，我是在src目錄下執行`npx treer -e ./result.txt` 便會
> 將樹狀結構寫到當前的`result.txt`文件中
4. 定義路由應對應的元件，routes的部分可以有預先載入的功能
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path: '/game1',
      name: 'game1',
      // Lazy Loading
      component: () => import('../views/Game1View.vue')
    }
  ]
})
export default router
```
5. 接著`App.vue`修改成以下結構，`RouterView`就是渲染對應路由的元件，`RouterLink`則是像`<a>`的連結會修改網址的對應路由，只是不會重新整理整個網頁，而是動態載入去切換部分畫面、網址。
```vue
// App.vue
<script setup>
import { RouterView, RouterLink } from 'vue-router';
</script>
<template>
  <header>
    <nav>
      <RouterLink to="/">Home</RouterLink>
      <RouterLink to="/game1">撲克牌連連看</RouterLink>
    </nav>
  </header>
  <RouterView />
</template>
```
> 如果需要當前路由對應到對應連結時自動套用特定class的話，不一定要在每個`<RouterLink>`寫`:class=""`去做判斷，只需要在`<style>...</style>`中定義class `.router-link-active`即可
```css
.router-link-active {
  color: #ff4500;
  font-weight: bold;
}
```

1. 調整`main.js`，讓Vue實際掛載第四步驟定義的Router
```js
// main.js
import router from './router'
// 略
const app = createApp(App);
app.use(router);
app.mount('#app')
```

## 使用slot重構遊戲標題
一直以來都是用自訂義的Vue組件都是把`props`的方式傳值進去，
但其實還有一個`slot`的用法可以把外部組件傳進去。

slot用法非常簡單，只需要在元件中添加`<slot>...</slot>`，
以當前專案舉例，因為之後`<FoxyHeader>`可能會添加在每個遊戲上方，因為每個遊戲的標題都不相同，所以在此改用`slot`做調整。
```vue
// FoxyHeader.vue
<template>
    <header>
        <div>
            <!-- 以下slot就是標題位置 -->
            <slot>No child component passed in</slot>
        </div>
        <img src="../assets/imgs/foxy01.jpg" class="foxyHead" />
    </header>
</template>
```
然後在`<FoxyHeader>`和`</FoxyHeader>`中間添加想要的文字或HTML組件，就填充到對應的`slot`位置，如果沒填充才會顯示上方寫的**No child component passed in**。
```vue
// Game1view.vue
<template setup>
    <main>
        <FoxyHeader>撲克牌連連看</FoxyHeader>
        <GameBoard />
    </main>
</template>
```

## 更多的利用slot
將牌桌`<GameBoard>`改用slot，控制牌桌上卡片排序的職責也拆成一個新元件`<CardRow>`，
將撲克牌連連看的內容都移入全都移入`Game1View.vue`，所以後者的樣板變成以下結構:
```vue
// Game1View.vue
<template setup>
    <main>
        <FoxyHeader>撲克牌連連看</FoxyHeader>
        <button style="font-size: 1.5rem;" @click="resetGame">重置</button>
        <div style="font-size:1.5rem;">
            <div>當前分數: {{ gameScore }}</div>
            <div>時間經過: {{ timerFormat }}</div>
        </div>
        <GameBoard>
            <CardRow>
                <Card v-for="card in boardCards" :key="card.value" :value="card.value" :isOpen="card.isOpen"
                    :isDone="card.isDone" @poker-flip="toggleFlip" />
            </CardRow>
        </GameBoard>
    </main>
</template>
```

**小結** 
本來想先做牌堆沒做成，倒是學習如何應用`Vue Router`和`slot`
程式碼: https://github.com/kabuto412rock/ithelp-pokergame/tree/day05
{{< youtube 9W6I5WE7TN8 >}}

**參考**
- https://github.com/derycktse/treer
- https://router.vuejs.org/guide/
