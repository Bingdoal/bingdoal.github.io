---
layout: post
title: Golang 的 Error Handling
date: 2022-05-11 11:31:10 +0800
category: backend
img: cover/golang.png
description: 撰寫系統的時候錯誤處理是非常重要的一環，系統的穩定度基本取決於對於錯誤處理是否全面，好的錯誤處理也可以產生適當的錯誤訊息，讓 Debug 更加容易，golang 在錯誤處理這方面跟其他語言的設計有些不同，特別來介紹一下
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## Go 的錯誤處理機制

golang 不像現行多數的程式語言有 `try catch` 的機制，Go 的錯誤處理方式大致分成兩種：
1. 回傳錯誤的物件，由後續程式自行處理
2. 觸發 panic 事件，強制中斷程式

參考以下程式碼

```golang
package main

import (
    "fmt"
    "strconv"
)

func main(){
    strVar := "abc"
	intVar, err := strconv.Atoi(strVar)
    if err != nil {
        fmt.Println(strVar + " 不是整數。")
    }

    t := 3/0 // panic
}
```

一般常用的函式庫幾乎都會帶有一個 `error` 的物件回傳，給使用者用來自行做額外處理

而像是 `3/0` 這種嚴重錯誤則會直接跳 `panic` 把程式中斷

以下也會分成這兩種錯誤拋出的方式來分別說明各自的處理方法

## error 處理

如同上面的範例程式，我們一般直接判斷 `error` 物件是否為 nil 然後再做進一步處理，不過如果我們確定程式不會出錯當然也可以不處理

實際在使用上我們可能會有客製化 `error` 物件的需求，可以通過一些 golang 提供的方法

```golang
var err error
err = errors.New("這是一個客製化錯誤訊息")
err = fmt.Errorf("這是一個客製化錯誤訊息: %s", "Hello world!")
```

我們也可以實作自己的 `error` 物件，只要符合介面就可以

```golang
type error interface{
    Error() string
}
```

其實介面非常單純，就一個 `Error` 的 `function`，以下是使用的範例 code：

```golang
type MyError struct{
    Status int
    Message string
}

func (e MyError) Error() string{
    return fmt.Sprintf("%d: %s", e.Status, e.Message)
}

func testMyError() error{
    return MyError{
        Status: 400,
        Message: "這是一個客製化錯誤訊息",
    }
}

func main(){
    if err := testMyError();err!=nil{
         fmt.Println(err)
    }
}
```

只要實作 `Error` 這個 `function` 就能夠當作 `error` 物件來回傳

並且如果 `function` 只有 `error` 回傳的話，可以用一行 `if` 來做處理，並且可以把 `error` 物件的 scope 限制在這個 `if` 內

如果需要更複雜的錯誤處理，也可以判斷一下 `error` 的實際類型

```golang
func main(){
    if err := testMyError();err!=nil{
        myE, isMyE := err.(*MyError)
        fmt.Println(myE)
        fmt.Println(isMyE)
    }
}
```

或更複雜有多種錯誤類型的話可以

```golang
func main(){
    if err := testMyError();err!=nil{
        switch t:=err.(type){
            case *MyError:
                myE,_ = err.(*MyError)
                fmt.Println(myE)
            default:
                fmt.Println("Unknown type: ", t)
        }
    }
}
```

## panic 處理

`panic` 除了像上面例子之外，也能夠自行觸發，通過同名函式

```golang
func letsPanic(){
    panic("Panic!!!")
    fmt.Println("print something")
}

func main(){
    letsPanic()
}
```

`panic` 的特性除了會中斷目前的函示執行之外，也會一連串向外引發 `panic`，如此一來就可以追蹤錯誤來源

不過有些時候我們不希望程式完全中斷，我們需要攔截這個 `panic` 可以透過 `recover` 的方式

```golang

func getRecover(){
    defer func() {
        err := recover()
        if err != nil {
            fmt.Println("Recover from panic: ", err)
        }
    }()
    letsPanic()
}

func main(){
    getRecover()
    fmt.Println("Still working.")
}
```

這邊要注意 `recover` 只有在 `defer` 中才有用，因為後續程式都不會執行了

但函式被 `panic` 結束後還是會進入到 `defer` 之中，並且在經過 `recover` 之後就不會再向外拋出了，可以確保程式不會被強制中斷，把錯誤限縮在可控的範圍之中

雖說 `panic` 搭配 `recover` 有一點 `throw` 跟 `try catch` 的味道在，但實際使用上差異還是挺大的

golang 的 `panic` 比較是真的發生嚴重錯誤才會拋出，相應的 `recover` 只是一個應急的容錯手段，理想上應該是盡可能不要有 `panic` 發生

---

## 結語

golang 的錯誤處理與過去各種語言的方式大不相同，這方面有點難以適應，且每次都要判斷 `err!=nil` 也讓程式變得有點亂，沒辦法統一處理錯誤，這方面還要多思考要怎麼設計

目前覺得這種設計思維是想要開發者盡可能單一化每個 function 的功能，這樣一來每個 function 也只需要做最少的 error handling 那這樣子的回傳方式反而可以讓流程簡單化