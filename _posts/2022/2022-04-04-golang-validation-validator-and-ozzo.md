---
layout: post
title: "Validation in Go: ozzo-validation、go-playground/validator"
date: 2022-04-03 17:49:21 +0800
category: backend
img: cover/golang.png
description: "在撰寫 API server 的時候一定會遇到參數驗證的問題，這次會介紹兩種常見的套件 ozzo-validation、go-playground/validator 看看各自是如何實踐在 golang 的 validation 的"
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## 前言
其實 gin gonic 有內建 validation，也就是下面要介紹的 `go-playground/validator`，不過今天多介紹一個套件提供更多做法可以挑選

## go-playground/validator

+ 附上 [官方 Github](https://github.com/go-playground/validator) 連結

網路上看到絕大多數的教學也都是用這套，主要原因應該是使用上比較簡單，且 gin 官方指定，不需要太多的額外操作，範例 code 如下

```golang
package main
import (
	"fmt"
	"github.com/go-playground/validator/v10"
)
type User struct {
	Name     string `validate:"required,min=2"`
	Email    string `validate:"required,email"`
	Password string `validate:"required,min=6`
}

func main(){
    user := User{
        Name: "",
        Email: "",
        Password: "",
    }
    if err:=validate.Struct(user);err!=nil{
        fmt.Println(err)
    }

    email := "12345"
    if err := validate.Var(email, "required,email"); err != nil {
        fmt.Println(err)
    }
}
```

主要是利用幫參數撰寫 tag 的方式設定驗證的規則，然後透過 `validate` 提供的 function 進行驗證
+ 優點:
  1. 使用便利，利用 tag 的方式撰寫相容性極高
  2. 幾乎不需要額外撰寫程式碼
+ 缺點:
  1. tag 撰寫是字串，容易寫錯，且寫錯沒辦法馬上得知
  2. 由於 tag 是綁定在 struct 上，所以不能隨 Create 或是 Update 改變，必須撰寫多個 struct 來套用

## ozzo-validation

+ 附上 [官方 Github](https://github.com/go-ozzo/ozzo-validation) 連結

`ozzo` 其實本身就是一個後端框架，並且有各種搭套的框架一起，如 ORM 的 `ozzo-dbx`，設定載入的 `ozzo-config`，感覺上有想要統一天下的雄心壯志，如果做的好的話應該會像 spring 那樣子變成一個生態系，之後有機會再來完整的研究研究，今天只專注在 `ozzo-validation` 上

`ozzo-validation` 是站在 `go-playground/validator` 的對面，認為 tag 由於是字串，所以容易出錯不易除錯，因此改而用程式碼的方式進行驗證，寫一段等價於上面的範例 code

```golang
package main

import(
    "fmt"
    "github.com/go-ozzo/ozzo-validation/v4"
    "github.com/go-ozzo/ozzo-validation/v4/is"
)

type User struct {
	Name     string
	Email    string
	Password string
}

func main(){
    user := User{
        Name: "",
        Email: "",
        Password: "",
    }
    if err := validation.ValidateStruct(user,
		validation.Field(&user.Name, validation.Required, v.Min(2)),
		validation.Field(&user.Email, validation.Required, is.Email),
		validation.Field(&user.Password, validation.Required, v.Min(6)),
	);err!=nil{
        fmt.Println(err)
    }

    email := "12345"
    if err := validation.Validate(email, validation.Required, is.Email); err != nil {
        fmt.Println(err)
    }
}
```

不需要撰寫 tag ，但把驗證規則寫在程式碼之中，透過 `validation` 提供的各種方法，根據需求靈活運用，但彈性有點不足，而且寫起來有點冗長

+ 優點:
  1. 程式碼實作時才決定規則，靈活套用在各種情境
  2. 程式碼撰寫會有快速完成的提示，寫錯也會在 compile 就被擋下來
+ 缺點:
  1. 必須把規則寫在程式碼之中，稍嫌有點彈性不足
  2. 要多寫很多 code，比較麻煩
  3. 如果結構較為複雜，規則寫起來也會有點雜亂

---

## 結論

其實端看各個團隊的使用習慣去選擇不同的套件即可，沒有一定孰優孰劣，個人是偏好 `go-playground/validator` 這套，利用 tag 的方式可以達到更好的彈性，不必把規則寫死在程式碼中，寫錯的問題只好多多測試了