---
layout: post
title: "gRPC 簡介"
date: 2022-12-24 13:47:58 +0800
category: network
img: cover/gRPC.png
description: "最近公司想引進新技術 gRPC，從之前就有聽說過這個協定，似乎在 golang 後端開法也是一個趨勢，今天就來記錄一下所學到的筆記，再補上一個簡單的 golang 實作"
lang: zh-TW
tags: [network, protocol, gRPC]
published: true
---

{{page.description}}

## 簡介

gRPC 簡言之就是 google 開發的 RPC(Remote Procedure Calls)，傳輸上是基於 HTTP/2 使用一種叫做 Protocol Buffers 的格式來作為介面描述語言，這點跟 GraphQL 有點像一樣要維護一份介面文件，不只可以作為文件也是執行程式必須的

主要會轉用 gRPC 我認為有幾個優點:
1. 透過 HTTP/2 並且用 binary 傳輸內容，傳送以及解析成本都比傳統 HTTP API 跟 json 快上不少
2. 透過 Protocol Buffers 維護統一介面，必須要有而不是隨心情而寫的文件
3. 可以透過 Protocol Buffers 直接產生程式碼，coding 的時候就是呼叫一個 function，不用額外記 url path，或是一直重複寫 http request 的 function
4. 支援雙向串流，不需要透過 WebSocket
5. 可以指定本次請求的預期時間，超時可以透過 server 端進行逾時處理

gRPC 缺點:
1. 生態系還不那麼完整，相比各個廠商都有提供 OpenAPI，gRPC 主要用在內部微服務溝通，也比較少第三方的各種方便 library 提供使用
2. 瀏覽器支援不足，目前瀏覽器還沒有原生支援 gRPC 的使用，我認為這是很大的原因限制了 gRPC 的發展
3. 沒有 Method 的分類，以及共同約定的 Header，像是驗證身份，或是語系的內容，都必須由雙方自行約定欄位並且自己實作機制
4. 由 binary 傳輸，不經過解碼的話沒辦法直接讀內容

## Protocol Buffers

Protocol Buffers 簡寫成 ProtoBuf 是 gRPC 的介面描述語言，除了用來作為統一的文件之外，也能用來產出 gRPC 的程式碼，一般檔案副檔名為 `.proto` 以下是一份範例檔

```proto
// test.proto

syntax = "proto3";

package test;

option go_package = "grpc.demo/test";

message TestAddReq{
    int64 a = 1;
    int64 b = 2;
}

message TestAddResp{
    int64 sum = 1;
}

service Test{
    rpc TestAdd(TestAddReq) returns (TestAddResp){};
}
```

開頭的 `syntax` 表明了 protobuf 的版本

其實寫起來還蠻像在寫 Swagger 的，定義介面跟傳參還有回傳

比較特別的是 `option` 這裡的 go_package 是用來定義 golang package 的路徑，可以理解到會根據實作的程式語言不同而有所不同

欄位後面的數字代表著他的 id，應該盡可能的將 id 縮減在 1~15 因為只會佔用 1 byte，其實我不太懂為什麼這個不能由 gRPC 編譯時再自行加入

## 建立基本 gRPC server

我們要建立一個新的 golang 專案來運行然後安裝所需的套件

先來抓一下 protobuf 的 [必要工具](https://github.com/protocolbuffers/protobuf/releases/)

然後透過下面指令確認版本跟安裝 golang 的套件

```bash
protoc --version
go get github.com/golang/protobuf/protoc-gen-go
go get -u google.golang.org/grpc
```

現在我們的專案大概長下面這樣

```
├ proto/
│   └ test.proto
├ go.mod
├ go.sum
└ server.go
```

然後執行下面指令編譯

```bash
mkdir grpc
protoc --go_out=./grpc --go_opt=paths=source_relative \
    --go-grpc_out=./grpc --go-grpc_opt=paths=source_relative \
    ./proto/*.proto
```

上面指令跑完之後的專案長這樣

```bash
├ grpc/proto/
│   ├ test_grpc.pb.go
│   └ test.pb.go
├ proto/
│   └ test.proto
├ go.mod
├ go.sum
└ server.go
```

生成的 code 就是幫你把 grpc 的 server 跟 client 基本架構還有介面寫好，我們只需要實作介面就好

下面是 server.go 的程式碼

```golang
package main

import (
    "context"
	"fmt"
	pb "grpc_demo/server/grpc/proto"
	"log"
	"net"

	"google.golang.org/grpc"
)

type TestGrpc struct {
	pb.UnimplementedTestServer
}

func (*TestGrpc) TestAdd(ctx context.Context, req *pb.TestAddReq) (*pb.TestAddResp, error) {
	return &pb.TestAddResp{
		Sum: req.GetA() + req.GetB(),
	}, nil
}


func main() {

	lis, err := net.Listen("tcp", "localhost:5000")
	if err != nil {
		log.Fatalf("failed to listen: %v \n", err)
	}

	grpcServer := grpc.NewServer()
	testGrpc := &TestGrpc{}
	pb.RegisterTestServer(grpcServer, testGrpc)

	fmt.Println("starting gRPC server on localhost:5000...")
	if err := grpcServer.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v \n", err)
	}
}

```

執行 `go run server.go` 簡單的 grpc server 就完成了

## 測試

接下來我們可以簡單利用 postman 來進行測試

![]({{site.baseurl}}/assets/img/postman-grpc-test.png)

正常回傳且結果正確

至此就完成我們第一個 gRPC server 了

---
## 結語

先簡介基本概念跟程式，之後會補上更多 gRPC 的用法，還有一些踩到的雷