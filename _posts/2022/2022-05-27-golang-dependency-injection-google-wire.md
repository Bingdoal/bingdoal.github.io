---
layout: post
title: Golang 上的依賴注入框架 google/wire
date: 2022-05-27 10:26:54 +0800
category: backend
img: cover/golang.png
description: 依賴注入可以幫助日漸複雜的專案達到解耦合的效果，透過介面的注入讓重構以及測試可以更好進行，但要自己手動撰寫依賴注入十分費工，而且程式碼注定不會太好看，這時候就讓我們來利用一下框架之力，Wire 是 Google 開源的一個依賴注入的框架，透過事先生成程式碼的方式來幫助開發者進行依賴注入，簡單說就是幫你把要手動寫的部分自動生成
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## Google/wire

### 簡介

在系統日漸壯大的情況下，手動撰寫依賴注入的成本會越來越高，需要建立的實例越來越多，需要使用到的建構子也越來越多，很容易忘記注入的順序，雖然因為類型不同，不可能會寫錯還讓你編譯過，但是要一直查 code 也是很惱人，多利用框架之力可以幫助我們開發起來舒心許多，工程師心情好開發效率就好，下面看看 google/wire 是怎麼幫助我們心情愉悅的

一般來說我們要做依賴注入會需要做兩件事:

+ 撰寫用來注入的建構子

```golang
func NewUserRoute(userSvc *UserService) *UserRoute{
    return &UserRoute{
        userSvc: userSvc,
    }
}
```

+ 建立物件並注入

```golang
func Initialize(){
    userSvc := &UserService{}
    userRoute := NewUserRoute(userSvc)
}
```

google/wire 主要是幫忙完成第二件事，可以幫忙省下幾個麻煩:

1. 自動去建立各種依賴物件
2. 不需要記得每個建構子的參數順序

### 安裝

+ 附上官方連結: [https://github.com/google/wire](https://github.com/google/wire)

首先安裝指令工具

```bash
go install github.com/google/wire/cmd/wire@latest
```

然後在需要用到的專案底下引用

```bash
go get github.com/google/wire/cmd/wire@latest
```

### 實作(一)

首先我們來用到官方的例子說明一下:

```golang
func NewMessage() Message {
    return Message("Hi there!")
}
func NewGreeter(m Message) Greeter {
    return Greeter{Message: m}
}
func NewEvent(g Greeter) Event {
    return Event{Greeter: g}
}

func main() {
    message := NewMessage()
    greeter := NewGreeter(message)
    event := NewEvent(greeter)

    event.Start()
}
```

一般情況下手動寫依賴注入會長得像上面那樣，可以看到 `main` 裡面的各種實例建立跟注入就是我們要節省的部分

所以讓我們來開始使用 `wire`，首先建立一個 `wire.go` 的檔案並加入以下的 code

```golang
// wire.go

func InitializeEvent() Event {
    wire.Build(NewEvent, NewGreeter, NewMessage)
    return Event{}
}
```

然後在目錄下執行指令 `wire` 或是 `wire gen`，接著會生成檔案 `wire_gen.go`

```golang
func InitializeEvent() Event {
    message := NewMessage()
    greeter := NewGreeter(message)
    event := NewEvent(greeter)
    return event
}
```

有沒有發現跟我們剛剛在 `main` 裡做的事情一模一樣，只是這次只需要轉寫 `wire.go` 裡短短的 code 就可以幫我們生成了

從生成前後的行為可以了解到 google/wire 的基本原理

```golang
// wire.go

func InitializeEvent() Event {
    wire.Build(NewEvent, NewGreeter, NewMessage)
    return Event{}
}
```

其實 `wire.Build` 就是在進行建構子的註冊，`wire` 根據建構子需要的參數以及回傳的類型，去自動偵測哪個實例要被注入到哪個建構子之中

需要注意的就是回傳的類型必須是唯一的，不能夠有多個建構子回傳同一種類型

而這個建構子回傳類型的部分被 `wire` 稱作 `provider`，生成的初始化建構函示則稱為 `injector`

#### 補充

在執行完 `wire` 生成 code 之後會發現 `InitializeEvent` 亮紅字，而且其實沒辦法執行程式，因為 `InitializeEvent` 會由於產生出來的程式碼而重複宣告，解法是在 `wire.go` 上加入一個註解

```golang
//go:build wireinject
// +build wireinject

func InitializeEvent() Event {
    wire.Build(NewEvent, NewGreeter, NewMessage)
    return Event{}
}
```

這可以讓 `go build` 的時候忽略這個檔案，但生成的 code 依舊會被使用

### 實作(二)

接著我們小小修改一下上面的例子:

```golang
func NewMessage(msg string) Message {
    return Message(msg)
}
func NewGreeter(m Message) Greeter {
    return Greeter{Message: m}
}
func NewEvent(g Greeter) Event {
    return Event{Greeter: g}
}
```

現在最頂層的建構子 `NewMessage` 需要一個參數進行初始化，這在系統中是常見的，可能是環境的變更或是設定檔的路徑

這時候只要把 `wire.go` 做點小小改變

```golang
func InitializeEvent(msg string) Event {
    wire.Build(NewEvent, NewGreeter, NewMessage)
    return Event{}
}
```

沒辦法由 `provider` 所提供的實例，可以由外部參數來提供

### 實作(三)

有時候如果建構子要注入的參數太多了，也可以透過結構來包裝

```golang
type Options struct {
    M Message
    G Greeter
}

func NewEvent(opt Options) Event {
    return Event{Greeter: g, Message: m}
}

func InitializeEvent(msg string) Event {
    wire.Build(
        wire.Struct(new(Options),"*")
        NewEvent,
        NewGreeter,
        NewMessage,
    )
    return Event{}
}
```

如果有不想要用來注入的屬性可以加上 tag

```golang
type Options struct {
    M Message
    G Greeter

    NoInject string `wire:"-"`
}
```

---

## 結語

google/wire 是一個小巧好用的工具程式，透過預先生成 code 的方式可以減少執行的時間，生成的程式基本上跟自己手寫的是一樣的，還提供多種註冊 provider 的方式，可以彈性選擇注入的途徑，其實還有很多進階用法，不過目前還沒覺得需要使用到，之後有機會再介紹吧
