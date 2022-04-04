---
title: "[教學] 使用Docker建立Nodejs開發環境範本"
date: 2022-04-03T22:22:27+08:00
# weight: 1
# aliases: ["/first"]
tags: ["Docker", "Nodejs"]

draft: false
---
大綱
- [前言](#前言)
- [為什麼要用Docker部屬](#為什麼要用docker部屬)
- [Node.js (Express.js)](#nodejs-expressjs)
  - [建置流程](#建置流程)
  - [資料夾結構 (啟動前)](#資料夾結構-啟動前)
  - [資料夾結構 (啟動後)](#資料夾結構-啟動後)
  - [建置說明](#建置說明)
      - [version](#version)
      - [services](#services)
      - [app (實際上可以取你自己喜歡的名稱web-app之類的)](#app-實際上可以取你自己喜歡的名稱web-app之類的)
- [結語](#結語)
# 前言
最近想紀錄一下可重複使用的程式碼片段，這樣之後找就從自己的部落格找會比較方便，尤其是最近常用Docker建立部屬環境。

# 為什麼要用Docker部屬
雖然網路上可以找到很多理由，但我的理由是：
1. 使用git版本控制，設定檔本身就取代環境建置說明文件
2. 不弄髒本地環境，想刪就刪
3. 替換依賴容易，Ex: MySQL -> Postgresl  

Docker基本上語法不複雜設定起來也很容易，尤其在建立不熟悉的環境亂試也可以，
且因為Docker有cache layer的關係，所以修改重新跑docker-compose up 也很快就可以建立。

廢話不多說，開始部屬吧...
# Node.js (Express.js) 
## 建置流程
1. 建立相關的檔案＆安裝Express（主要是為了產生package.json）
```bash
$ mkdir node-demo && cd node-demo 
$ mkdir app
$ touch app/package.json && touch app/index.js && touch docker-compose.yaml
```
2. 複製以下程式碼貼到對應的檔案  

app/package.json
```json
{
  "name": "node-demo",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "nodemon index.js"
  },
  "license": "MIT",
  "dependencies": {
    "express": "^4.17.3"
  },
  "devDependencies": {
    "nodemon": "^2.0.15"
  }
}
```
app/index.js
```javascript
const express = require("express");
const port = 8000;
const app = express();

app.get("/", (req, res) => {
  res.status(200).json({
    message: "Hello",
  });
});
app.listen(port);

```
docker-compose.yaml
```yaml
version: '3.1'
services:
  app:
    image: node:16-alpine
    command: sh -c "yarn install && yarn run start"
    ports:
      - 80:8000
    working_dir: /app
    volumes:
      - ./app:/app
```

3. 啟動服務、關閉服務
```bash
# 啟動Docker服務
$ docker-compose up -d
# 停止Docker服務
$ docker-compose down
```
4. 檢查實際結果 http://localhost/

## 資料夾結構 (啟動前)
```yaml
├── app
│   ├── index.js
│   └── package.json
└── docker-compose.yaml
```
## 資料夾結構 (啟動後)
```yaml
├── app
│   ├── index.js
│   ├── node_modules [程式依賴目錄]
│   ├── package.json
│   └── yarn.lock
└── docker-compose.yaml
```

## 建置說明
說明一下docker-compose.yaml內的細節，因為這些設定的屬性都是docker-compose up時參考的定義，一定要了解一下。

#### version  
  是說明使用docker-compose.yaml的語法版本，是給Docker CLI看的版本名稱，此處為3.1
#### services
  裡面可以看到app有開始往右靠，是因為app是這份文件的其中一個服務
#### app (實際上可以取你自己喜歡的名稱web-app之類的)
```yaml
image:
  定義app使用的基礎映像檔，比如說node就是一個已經裝好nodejs、npm的環境。
command:
  容器啟動每次都會執行的指令
  sh -c 只是從執行後面字串"yarn install && yarn run start"的指令，去安裝並執行package.json定義的start腳本
ports:
  80:8000
  代表本地主機的Port 80 會對應到容器的Port 8000
working_dir:
  指定容器內當前所在的目錄/app，因為這樣執行command執行
  命令時，才可以找到正確package.json後，正確執行相對位置的檔案
volumes:
  掛載本地端的./app到容器內的/app，修改本地端./app內的資料夾同時會修改到容器內的/app的程式碼，反之亦然。

  不使用DockerFile的COPY的方式，是因為nodemon會監控檔案的變化，COPY的方式只是一次性的複製到容器內，而非持續性。
```

# 結語
這篇文章先描述過程不解釋原理，只是想避免在操作過程、解釋原理這兩件事交錯，因為我想之後可能也會來看這篇文章，但有時真不會看什麼原理（畢竟已經很清楚了）。

雖然使用Node.js通常還會結合資料庫，但鑑於這篇是第一篇Docker文章，就先不提高文章知識難度，也囉唆多寫一點docker-compose細節的部份。

下一篇應該彙整併MongoDB，感謝你的閱讀。


