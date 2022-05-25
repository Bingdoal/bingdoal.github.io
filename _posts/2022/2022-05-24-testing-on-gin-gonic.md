---
layout: post
title: Gin Gonic 上進行測試
date: 2022-05-21 15:02:50 +0800
category: backend
img: cover/gin-gonic.png
description: 使用 gin 進行後端開發上也是需要測試的，但沒有辦法簡單的使用 unit test 達到目的，一個完整的 http 請求流程會經過各種 middleware，為了測試這個情況，我們要來進行 gin 的 api 測試
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## 目標測試程式

```golang
// main.go

package main

import (
    "github.com/gin-gonic/gin"
    "fmt"
)

func main() {
    server := setupRoute()
    server.Run(":8888")
}

func setupRoute() *gin.Engine {
    router := gin.Default()

    router.GET("/hello", func(c *gin.Context){
        fmt.Printf("ctx.Request.Header: %v\n", ctx.Request.Header)
    },func(c *gin.Context) {
        c.JSON(200, gin.H{"hello": "world"})
    })

    return router
}
```

簡單寫個範例來測試，測試目的是模擬 API 打進來，會通過所有的 middleware

## 測試程式

一樣我們按照撰寫 unit test 的規則來撰寫一個 `main_test.go` 的檔案，並且 function 名稱定為 `TestHello`

```golang
// main_test.go

package main

import (
    "net/http"
    "net/http/httptest"
    "testing"
    "fmt"
    "github.com/stretchr/testify/assert"
)

func TestHello(t *testing.T) {
    server := setupRoute()

    req, _ := http.NewRequest("GET", "/hello", nil)
    req.Header.Set("Authorization", "Bearer testToken")
    w := httptest.NewRecorder()
    server.ServeHTTP(w, req)

    expectedStatus := http.StatusOK
    assert.Equal(t, expectedStatus, w.Code)

    fmt.Println("Body: %+v", w.Body)
}
```

gin 有提供 `ServeHTTP` 的 function 來讓使用者模擬請求丟入的情況

如果檔名跟 function 名稱有按照規則來的話，只要執行 `go test -v` 就會開始跑測試了

---
## 結語

gin 提供了易用的介面來進行 API 的模擬，簡單達到測試的目的，不過在測試上我們其實會有更多更複雜的需求，之後再詳細解說