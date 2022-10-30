---
title: "Linux指令-設定檔案讀寫權限 chmod"
date: 2022-10-30T04:00:14Z
# weight: 1
# aliases: ["/first"]
tags: ["linux"]

draft: false
---
## 問題:
WSL2跑Docker也可以寫原本在Ubuntu寫的hugo，又不用額外安裝`homebrew`
想用`VSCode`編輯Docker容器產生的文章後，發現我`VSCode`竟然沒有權限修改檔案。

## 解法:
辦法很簡單，修改檔案權限全開
```bash
chmod -R 777 content/
```

這篇是用來測試Github action是否還有正常運作


