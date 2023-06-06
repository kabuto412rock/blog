---
title: "使用Docker建立Node容器+MySQL容器的範本"
date: 2023-06-06T14:08:22Z
# weight: 1
# aliases: ["/first"]
tags: ["Docker", "Nodejs", "MySQL"]

draft: false
---

# 前言
距離上一篇文章[使用Docker建立Nodejs開發環境範本
](../node-docker)已經有1年啦! 
那時竟然還擔心部落格太多文章，Github page是否會被限制

# Node.js (Express.js + MySQL Server) 
## 建置流程(快速克隆 -> 趕時間)
1. 直接使用git clone語法下載範例
  ```bash 
  $ git clone https://github.com/kabuto412rock/node-demo.git --branch express-mysql --single-branch
  ```
2. 啟動服務、關閉服務
```bash
# 進到目錄底層
$ cd node-demo
# 啟動Docker服務
(node-demo) $ docker-compose up -d
# 停止Docker服務
(node-demo) $ docker-compose down
```
## 相對上篇文章的差異
- 使用sequelize-cli進行Sequelize的設定，畢竟手寫migration、seeds太累
- docker-compose.yaml多加MySQL容器、Adminer容器
    ```yaml
    express-mysqldb:
    image: mysql:8.0
    # WARNIGN:正式環境請不要直接使用原生密碼，這只是開發偷懶用
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    ports:
        - "3308:3306"
    environment:
        # root的密碼
        MYSQL_ROOT_PASSWORD: PyoA2hGpSDQordaDAbuiQIIDS
        # 預設建立的DB名稱
        MYSQL_DATABASE: mydb
        # DB使用者的帳號
        MYSQL_USER: dbuser
        # DB使用者的密碼
        MYSQL_PASSWORD: youiIIDSA2hGpassword
    ```
- my-app資料夾可以注意到`config/config.json`的host值與MySQL容器名稱相同，這是因為容器的`hostname`預設與名稱相同，其他password等設定可以參考上面對照
```bash
{
  "development": {
    "username": "dbuser",
    "password": "youiIIDSA2hGpassword",
    "database": "mydb",
    "host": "express-mysqldb",
    "dialect": "mysql"
  },
  // 略...
}
```

## 結語
還好當初有在blog的repo設定好Github Aciton和寫README備忘，才可以像現在簡單git push一下就能生成新文章。

雖然這篇有點水，但至少是好習慣的開始。