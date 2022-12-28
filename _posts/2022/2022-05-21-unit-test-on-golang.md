---
layout: post
title: Golang 上的 Unit test
date: 2022-05-21 12:11:06 +0800
category: backend
img: cover/golang.png
description: 開發一段時間後，系統發展日漸複雜，常會有重構或是修改的需求，這時候若是任意修改有可能會導致相關功能出現副作用，這時候確保測試的撰寫就很重要了，這篇就來簡介一下 golang 的 unit test 要怎麼進行
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## 內建 Testing

在 Golang 本身在設計時就考量進 Unit test 的需求，於是就已經有一套內建的 Testing 框架提供使用

簡單舉個例子我們要測試以下程式

```golang
// my_calculator.go

func add(a int, b int) int{
    return a + b
}

func time(a int, b int) int{
    return a * b
}
```

只要建立一個檔名結尾是 `_test` 就會被認定為測試檔，並且命名函式為 `Test` 開頭就會被認為是測試函式

```golang
// my_calculator_test.go

func TestAdd(t *testing.T){
    ans := add(1, 2)
    if ans != 3 {
        t.Errorf("Ans isn't correct. ans: %d", ans)
    }
}

func TestTime(t *testing.T){
    ans := time(2, 2)
    if ans != 4 {
        t.Errorf("Ans isn't correct. ans: %d", ans)
    }
}
```

然後只要在目錄下執行 `go test -v` 就會自動跑測試並輸出結果與相關細節，使用上非常簡單

## vscode

如果說是利用 vscode 進行開發的話，按下 `F1`，找指令 `>Go: Generate Unit Tests ...`

![Alt]({{site.baseurl}}/assets/img/vscode-generate-golang-test.png)

可以間單產出下面的測試程式

```golang
package main

import "testing"

func Test_add(t *testing.T) {
    type args struct {
        a int
        b int
    }
    tests := []struct {
        name string
        args args
        want int
    }{
        // TODO: Add test cases.
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := add(tt.args.a, tt.args.b); got != tt.want {
                t.Errorf("add() = %v, want %v", got, tt.want)
            }
        })
    }
}

func Test_time(t *testing.T) {
    type args struct {
        a int
        b int
    }
    tests := []struct {
        name string
        args args
        want int
    }{
        // TODO: Add test cases.
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := time(tt.args.a, tt.args.b); got != tt.want {
                t.Errorf("time() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

已經幫你把大致的測試框架寫好了，只要間單修改一下 `tests` 的結構內容就可以進行多個案例的測試了，真的是非常輕鬆啊

## Testify

另外簡介一個測試的工具 `Testify`，包含了各種各樣好用的測試功能

```bash
go get github.com/stretchr/testify
```

像是上面要檢查測試正確與否以及回報，會需要一個 `if` 以及一個 `t.Errorf` 結合，這樣寫起來其實略微有點麻煩

用到 `Testify` 就可以換成下面這樣

```golang
func TestAdd(t *testing.T){
    ans := add(1, 2)
    assert.Equal(t, 3, ans)
}

func TestTime(t *testing.T){
    ans := time(2, 2)
    assert.Equal(t, 4, ans)
}
```

寫起來簡潔也好理解，除了 `assert` 之外也有更多神奇的用法，待之後再介紹

---

## 結語

簡單介紹基本的 golang testing 的用法，不過實際上做測試還會遇到更多問題，等待之後再補幾篇來說明
