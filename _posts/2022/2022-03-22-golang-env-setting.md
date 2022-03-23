---
layout: post
title: "[踩雷紀錄] Golang access denied: golang 環境變數 GOPATH、GOROOT"
date: 2022-03-22 10:46:06 +0800
category: backend
img: cover/golang.png
description: "最近在新的機器上重裝 golang 遇到一些問題，沒辦法正常使用，這次踩的雷主要是對於 Golang 環境變數的不了解，所以這邊就來看看 golang 環境變數的相關問題"
lang: zh-TW
tags: [踩雷紀錄, golang]
published: true
---

{{page.description}}

## 問題描述

新裝好 golang 之後將過往的專案從 git 拉下來悠悠地執行 `go run main.go` 卻發生下面的錯誤訊息

```bash
$ go run main.go
go: writing go.mod cache: mkdir C:\Program Files\go\pkg: Access is denied.
```

其實很好理解發生什麼事，因為要在 `C:\Program Files` 下建立資料夾是需要管理員權限的

可是為什麼 go 會跑到 `C:\Program Files` 下建立資料夾呢，這就牽扯到 go 的環境變數了

## go env

go 有自己管理一套環境變數，不過會優先使用系統的設定值，基本上應該都會自己設定好才對，透過 `go env` 可以看到完整內容

```bash
$ go env
set GO111MODULE=on
set GOARCH=amd64
set GOBIN=
set GOCACHE= ...
set GOENV= ...
set GOEXE=.exe
set GOFLAGS=
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOINSECURE=
set GOMODCACHE= ...
set GONOPROXY=
set GONOSUMDB=
set GOOS=windows
set GOPATH= ...
set GOPRIVATE=
set GOPROXY=https://proxy.golang.org,direct
set GOROOT=C:\go
set GOSUMDB=sum.golang.org
set GOTMPDIR=
set GOTOOLDIR=C:\go\pkg\tool\windows_amd64
set GCCGO=gccgo
set AR=ar
set CC=gcc
set CXX=g++
set CGO_ENABLED=1
set GOMOD= ...
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config
set GOGCCFLAGS=-m64 -mthreads -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=C:\Users\Essences\AppData\Local\Temp\go-build494662258=/tmp/go-build -gno-record-gcc-switches
```

透過參數 `-w` 可以寫入變數，`-u` 則可以移除變數

```bash
$ go env -w GOBIN=/somewhere/else/bin
$ go env -u GOBIN
```

## `GOPATH`

這便是這次問題的元兇了，不知道為什麼 `GOPATH` 被設定到了 `C:\Program Files\go`

`GOPATH` 的用意在於指定工作環境的根目錄，在 `go mod` 出來之前，所有的 go 專案都必須放在 `GOPATH` 之下才能執行，在 `go mod` 出來之後呢，`GOPATH` 依舊有它的作用，底下主要分為三個目錄

1. `bin`: 存放執行檔，在專案經過 `go install` 或是 `go get` 之後會編譯成執行檔放到這底下，只要把這個路徑加入到 `PATH` 之中，就可以在任何地方直接執行 go 的程式，也就是自製一個指令程式出來，非常便於建立一些工具指令
2. `pkg`: 可以用來將各種套件預編譯成 `.a` 檔存放，加快往後用到同一個套件時編譯的速度
3. `src`: 用來放各種原始碼，在 `go mod` 出現之前，開發專案都要放在這下面，而第三方的依賴的程式也會被抓到這裡面來

因此在進行 `go run` 的時候 go 想要在 `GOPATH` 之下建立 `pkg` 用來存放套件的預編譯檔，但是因為權限問題被擋下，因此執行失敗

所以這邊的解法就是把 `GOPATH` 設定成 `%USERPROFILE%\go`，抓到 windows 的使用者目錄下就可以確保權限一定沒問題了

## `GOROOT`

這次不知道為什麼 `GOPATH` 被設定到了 `GOROOT`，就順便講一下

`GOROOT` 其實就是指 golang 的安裝地點，存放有內建的函式庫與執行檔 `go` 指令也是放在裡面