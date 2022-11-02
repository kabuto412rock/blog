---
title: "Gen Import File"
date: 2022-11-01T23:28:43Z
# weight: 1
# aliases: ["/first"]
tags: ["Nodejs"]

draft: false
---

# 前言
因為同事說`動態載入`資料夾內多個程式檔案時，雖然過程變成只要在資料夾內新增一個Model，執行狀態時就可以自動使用那個Model，但卻失去了開發時可以享受靜態引用的`VSCode`提示(AutoComplete、Intellisense)

雖然那位朋友使用的是PHP，但我手上的NodeJS專案也會有這個問題。

# 思考路境
因為Intellisense只支援靜態分析，所以`動態載入`不會被VSCode支援提示。

但改為`添加靜態引用`就失去原本的動態載入的方便。

# 解法
執行腳本`生成靜態引用`，如果行得通甚至可以使用類似nodemon去watch特定資料夾內的檔案變化，達到`自動生成靜態引用`的功能。

# 範例
https://github.com/kabuto412rock/gen-static-import







