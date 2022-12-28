---
layout: post
title: "Golang Gin Gonic 簡介與基本操作"
date: 2022-03-17 10:01:08 +0800
category: backend
img: cover/gin-gonic.png
description: "Gin Gonic 是一個 Golang 非常好用也十分著名的 Http server 框架，這篇簡單說明一些基本用法與操作"
lang: zh-TW
tags: [golang, gin, backend]
published: true
---

{{page.description}}

## Gin Gonic

Gin 是一套用 golang 原生的 `net/http` package 封裝過後的框架，效能完全有保證，還有各種方便的 data binding 機制

接下來會主要用 Gin 來運行一個 RESTful 的 API server 不會有涉及前端的部分

### 安裝

安裝前記得要先執行 `go mod init <module-name>` 這樣依賴才會被記錄在 `go.mod` 中

```bash
go get github.com/gin-gonic/gin
```

簡單一行指令就可以安裝需要的套件了，只不過這個套件路徑不好記就是了

### 基本範例

```golang
type TestData struct {
    Hello string `json:"hello"`
}

func test(c *gin.Context) {
    data := new(TestData)
    data.Hello = "world!"
    c.JSON(http.StatusOK, data)
}

func main() {
    server := gin.Default()
    server.GET("/test", test)
    server.Run(":8080")
}
```

寫個基本的 GET 路由，讓它回傳一個 JSON 的資料，Gin 有幾種常用的 Http Method 作為 function 提供

1. GET
2. POST
3. PUT
4. PATCH
5. DELETE
6. OPTIONS

操作則由 callback 的方式傳入 `*gin.Context`

`gin.Context` 裡面包含有 request 以及 response 的內容與操作，並且支援直接回傳 JSON 自動進行轉換，根據 struct tag 給予的名稱來轉換

啟動上面程式會看見下面畫面

```bash
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /test                     --> main.test (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on :8080
```

很貼心的幫你把一些基本訊息印出來了，包含整個程式的路由

## Route

接著要來嘗試串接些路由，前面例子只有用到一層簡單路由，但實際情況常是有多層的路由，這樣可以讓同樣的邏輯被整理在一起

```golang
func main() {
    server := gin.Default()
    apiV1 := server.Group("api/v1/")
    apiV1.GET("test", test)
    server.Run(":8080")
}
```

實作起來也是十分簡單，延續上一個範例我們把 `GET /test` 改成 `GET /api/v1/test`

## 接收資料

在 Http request 中資料的傳遞是很重要的，Gin 是如何接收這些資料的呢

```golang
func main() {
    server := gin.Default()
    apiV1 := server.Group("api/v1/")
    apiV1.GET("hello/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.JSON(http.StatusOK, gin.H{"hello": name})
    })

    apiV1.GET("hello", func(c *gin.Context) {
        name := c.Query("name")
        c.JSON(http.StatusOK, gin.H{"hello": name})
    })

    apiV1.POST("hello", func(c *gin.Context) {
        body := map[string]string{}
        c.BindJSON(&body)
        c.JSON(http.StatusOK, gin.H{"hello": body["name"]})
    })
    apiV1.POST("hello/form", func(c *gin.Context) {
        name := c.PostForm("name")
        c.JSON(http.StatusOK, gin.H{"hello": name})
    })

    server.Run(":8080")
}
```

這邊用各種 hello 來示範串接 query、body、parameter 的方式，其中 body 要根據傳輸的格式來區分，主要最常用到的應該會是 JSON 以及 Form data，這邊兩個都寫個簡單範例，下面是各個請求的回傳

```bash
curl --request GET "localhost:8080/api/v1/hello/world"

curl --request GET "localhost:8080/api/v1/hello?name=world"

curl --request POST "localhost:8080/api/v1/hello" \
--header "Content-Type: application/json" \
--data-raw '{
    "name": "world"
}'

curl --request POST "http://localhost:8080/api/v1/hello/form" --form name="world"

{"hello": "world"}
```

其中 body 的部分提供了很多種 Bind 的 function，範例中的 `c.BindJSON()` 就是其中之一，這裡是寫一個簡單的 map 來乘載，也可以透過 struct 來乘載

```golang
type Test struct {
    Name string `json:"name"`
}

func main() {
    server := gin.Default()
    apiV1 := server.Group("api/v1/")
    apiV1.POST("hello", func(c *gin.Context) {
        body := new(Test)
        c.BindJSON(&body)
        c.JSON(http.StatusOK, gin.H{"hello": body.Name})
    })
    server.Run(":8080")
}
```

一樣透過 `json:"name"` 的 tag 來指定 binging 的 key

實際上 context 的 Binding 功能還有很多可以嘗試，之後有用到再詳細記錄吧

---

## 結語

Gin Gonic 是個簡單輕量的框架，寫起來的感覺其實更像是 node.js 的 express 框架寫法，設定簡單，撰寫簡單，沒有太多包袱也不用綁定一堆工具，比起 Spring boot 更好上手，這可以算是優點也算缺點

短周期可以快速開發，但長期來看要維護這樣的架構，專案一旦大了起來就需要各種規範，由於 Gin 提供了強大的彈性與自由度會導致需要開發團隊自己約束
