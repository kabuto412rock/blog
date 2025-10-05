---
title: "Gin Request Context導致的問題"
date: 2025-10-05T04:03:15Z
# weight: 1
# aliases: ["/first"]
tags: ["go", "context"]

draft: false
---
最近工作遇到將些服務的方法重構，為了調整了上下文的傳遞，結果意外導致Race Condition的問題。

正是因為 c.Request.Context()在回應HTTP Response的時候會取消掉Context，  
導致引用ctx的goroutine在Context被取消後無法繼續執行，從而引發了Race Condition的問題。
```go
func (c *gin.Context) SomeHandler() {
    ctx := context.WithValue(c.Request.Context(), "key", "value")

    go func(ctx context.Context) {
        // bug demonstration
    }(ctx)
    c.JSON(200, gin.H{"status": "ok"})
}
```

太久沒寫，內容短短只是想測試hugo部屬...
