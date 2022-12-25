---
layout: post
title: "gRPC 錯誤處理"
date: 2022-12-25 14:11:11 +0800
category: network
img: cover/gRPC.png
description: "前篇簡介了基本的 gRPC 以及基本 server、client 程式，這篇接續前篇的程式修改，會包含 gRPC 的錯誤處理"
lang: zh-TW
tags: [network, protocol, gRPC]
published: true
---

{{page.description}}

前一篇的 gRPC 簡介可以看 [這裡][grpc-intro]

[grpc-intro]: https://bingdoal.github.io/network/2022/12/grpc-protocol-intro/

## 錯誤處理

gRPC 基本有著與 http 類似的錯誤處理方式，同樣每個請求回傳會帶著 status 跟 message

status 定義在 [文件][grpc-status]

不過可想而知這種簡單的結構內容沒辦法符合實際的業務需求，讀取 message 判斷細節也不太合理，因此我們要使用 google 額外開發的 error model，下面主要紀錄這種範例

> 真不懂為什麼 google 不放在官方標準？

### protobuf

首先在 protobuf 中加入我們的客製化錯誤處理物件，還有錯誤代碼

```proto
message ErrorDetail{
    enum ErrorCode{
        NIL = 0;
        TEST = 1;
    }
    ErrorCode code = 1;
    string message = 2;
}
```

要注意到的是 `enum` 的類型其實就是 int32，而 int32 的預設值是 0，所以 `enum` 的序列必須要有設定 0，其他亂跳沒關係

### server
那我們改寫一下 server 的 code

```go
func (*TestGrpc) TestAdd(ctx context.Context, req *pb.TestAddReq) (*pb.TestAddResp, error) {
    st, err := status.New(codes.Unknown, "custom error").WithDetails(&pb.ErrorDetail{
        Code:    pb.ErrorDetail_NIL.Enum(),
        Message: "test error",
    })
    if err == nil {
        return nil, st.Err()
    } else {
        fmt.Printf("%+v\n", err)
    }
}
```

golang 的 grpc library 本身就自帶有 google error model 的實作，所以直接調用 `status.New(...).WithDetails(...)` 就可以帶入客製化的結構了

不過其他語言實作的話可能還沒有支援，不一定要自己動手接

### client

client 的部分也要修改一下

```go
func grpcTestAdd(client pb.TestClient) {
    fmt.Println("Staring gRPC request")
    req := &pb.TestAddReq{
        A: 5,
        B: 7,
    }

    res, err := client.TestAdd(context.Background(), req)
    if err != nil {
        st, ok := status.FromError(err)
        //fmt.Printf("error: %+v\n", st)
        if ok && len(st.Details()) > 0 {
            detail := st.Details()[0]
            fmt.Printf("detail: %+v\n", detail)

            errorDetail := detail.(*pb.ErrorDetail)
            fmt.Printf("error: %+v\n", errorDetail.Code)
        }
    }

    log.Printf("Response: %v", res.GetSum())
}
```

把客製化的 error struct 寫在 protobuf 的好處就是可以直接 casting 成需要的結構，這部分已經幫我們做好自動序列化了


[grpc-status]: https://grpc.github.io/grpc/core/md_doc_statuscodes.html

