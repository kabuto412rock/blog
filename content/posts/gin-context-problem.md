---
title: "使用Gin Request Context可能遇到的問題(附解法)"
date: 2025-10-05T04:03:15Z
# weight: 1
# aliases: ["/first"]
tags: ["go", "context", "gin", "debug"]

draft: false
---
- [撰文原因](#撰文原因)
- [問題程式碼(範例)](#問題程式碼範例)
- [解法](#解法)
	- [1. 暫時先傳 `context.TODO()`](#1-暫時先傳-contexttodo)
	- [2. 使用 `context.WithoutCancel`](#2-使用-contextwithoutcancel)
	- [3. 不要用**go**關鍵字](#3-不要用go關鍵字)
- [結語](#結語)
- [參考資料](#參考資料)

## 撰文原因
最近工作遇到服務使用的新方法需要傳遞context到函數中執行，
結果導致本來要跑在背景任務沒完成(ex: 通知訊息/發送郵件)。

## 問題程式碼(範例)
事情是因為傳入的是 c.Request.Context()，然而這個gin所實作的Context在回應HTTP Response結束時會自動取消，  
導致引用ctx的do函數在Context連帶著被取消後內部的函數也跟著取消。

程式碼範例如下:
```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
)

func main() {
	// Create a Gin router with default middleware (logger and recovery)
	r := gin.Default()

	// Define a simple GET endpoint
	r.GET("/ping", func(c *gin.Context) {
        
		go doWork(c.Request.Context())
		time.Sleep(2 * time.Second)
		// Return JSON response
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	r.Run("127.0.0.1:8080")
}

func doWork(ctx context.Context) {
	defer ctx.Done()
	elapsed := 0 * time.Millisecond
	for {
		select {
		case <-ctx.Done(): // Check if the context is done (cancelled or timed out)
			fmt.Printf("worker stopped err: %v\n", ctx.Err())
			return
		default:
			secs := 500 * time.Millisecond
			fmt.Printf("working... %v passed\n", elapsed)
			time.Sleep(secs)
			elapsed += secs
		}
		if elapsed >= 5*time.Second {
			fmt.Println("worker completed")
			break
		}
	}
}
```

實際跑起來會發現doWork只是每0.5秒印出一段訊息，原本預期最多跑5秒就會結束，
然而實際上2秒就會因為gin response被標記為完成自動結束，
這邊只是粗略模擬尊重context機制的函式庫的行為(qmgo...)，
當select捕捉到<-ctx.Done()會停止或取消後續的行為。

所以內部函數的ctx.Err() 回傳 `context canceled`這個error也是合理的，
畢竟parent context都取消

阿...講那麼多，要如何避免呢??

## 解法
### 1. 暫時先傳 `context.TODO()`
   當你不知道該傳什麼context時，可以使用 `context.TODO()` 來表示這是一個尚未決定的context。
   如果確定這是個背景任務，可以使用 `context.Background()`。
   ```go
   go doWork(context.TODO())
   ```
### 2. 使用 `context.WithoutCancel`
當你需要在context中的相同資料但不受parent context的取消影響，
可以使用 `context.WithoutCancel` 來創建一個新的context。
```go
ctx := context.WithoutCancel(c.Request.Context())
go doWork(ctx)
```

### 3. 不要用**go**關鍵字
如果其實可以做完事情再回應，那就等事情做完再回應。
```go
doWork(c.Request.Context())
```

## 結語
在使用Web框架時，正確地處理context非常重要，特別是在處理背景任務時。
透過上述3種方法，我們可以有效地避免因為context被取消而導致的問題。

應該還有更多解，但我懶得學了😆


## 參考資料
- [官方go文件 context](https://pkg.go.dev/context)
- 我的大腦