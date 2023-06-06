---
title: "如何解決動態載入(Dynamic Import)不支援智能提示"
date: 2022-11-01T23:28:43Z
# weight: 1
# aliases: ["/first"]
tags: ["Nodejs"]

draft: false
---
# 前言
因為同事目前使用`動態載入`資料夾內多個程式檔案時，最後由一個特定檔案輸出，但VSCode編輯引用特定檔案的程式碼時，無法看到相對應的智能提示、引用路徑

我手上的NodeJS專案也有類似的例子，比如說多個Model檔定義在一個models資料夾，
最後統一由models/`index.js`做查詢檔案動態載入所有Model檔案，最後使用任一Model時只需要引用`index.js`

## 好處 
1. 只要在models新增一個新Model檔，不用特別修改`index.js`
2. 需要任一Model時只需要引用`index.js` 
## 壞處
失去了開發時可以享受靜態引用的`VSCode`提示(AutoComplete、Intellisense)

所以問題是如何保持好處且消除壞處

# 思考路境

`動態載入`不會被VSCode支援提示是因為Intellisense只支援靜態分析，
若改為`添加靜態引用`就失去原本的動態載入的方便。

除非`添加靜態引用`這件事本身是`自動`的...

# 解法
使用腳本`生成靜態引用`，如果行得通甚至可以使用類似nodemon去watch特定資料夾內的檔案變化，達到`自動生成靜態引用`的功能。

# 範例
https://github.com/kabuto412rock/gen-static-import







