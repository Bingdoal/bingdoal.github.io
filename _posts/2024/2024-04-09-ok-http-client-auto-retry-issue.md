---
layout: post
title: "[踩雷紀錄] Http 套件 OkHttpClient 非預期 retry"
date: 2024-04-09 10:44:35 +0800
category: backend
img: cover/okhttp.png
description: "最近公司的產品發生一件神秘的事情，明明從 log 或是程式碼來看我們都只有發出一次的 http 請求，但合作的廠商卻說我們重複送了一次一模一樣的請求，導致有幾百筆訂單被重複送出，又因為錯誤處理沒有處理到重複單號的問題，結果訂單處理跟三方有出入導致損失慘重"
lang: zh-TW
tags: [踩雷紀錄, backend]
published: true
---

{{page.description}}

查這個 issue 最大的困擾是從我們的程式碼跟 log 來看都應該只有送出一次請求才對，所以第一時間是懷疑可能有一些 middleware 如 `nginx` 或是什麼網通設備雞婆幫我們重送

但在諮詢了 DevOps team 的同仁過後明白這是在 Http 應用層級發生的問題，網通設備才不管這個，nginx 也沒有設定這種奇怪的重送，於是就把目光放回我們用的 Http 套件上面了

## [okhttp](https://github.com/square/okhttp) 

之前都是用 `SpringBoot` 內建的 `RestTemplate` 其實也是第一次碰 `okhttp`，看了一下官方文件，大抵優勢是可以共用連線資源，降低延遲，內建快取等等機制，提供高效的 Http 請求體驗

## 推論

不過在有了上面的懷疑之後嘗試的直接 google 看看有沒有 okhttp retry 的相關資料，發現還真的有，找到了這篇文章 [OkHttp is quietly retrying requests. Is your API ready?](https://medium.com/inloopx/okhttp-is-quietly-retrying-requests-is-your-api-ready-19489ef35ace) 

文章中寫到 okhttp 確實有內建一個 retry 機制，文章中寫到會在以下的情況發生：
1. domain 有多 host ip 時在取得其中之一的 ip 失敗時
2. socket `ConnectionPool` 的重用可以降低 latency，但可能導致某些非預期的 timeout
3. 取得 proxy server 失敗導致的重試

實際看上去會覺得這三個原因好像都不太對，第一點跟第三點的情況下，請求都不應該會送到三方才對，只有第二種比較有可能，但是這個感覺有點玄，因為實際上我們也有遇過 http timeout 的情況但沒有重試，所以這個 timeout 可能在特定情況才會觸發嗎？

實在是越想越不對勁，決定來翻 source code 看看到底什麼情況會重試

## source code 導讀

這次我們用的 `okhttp` 版本是 `3.14.9`

首先看到 request 發起的進入點 `okhttp3.RealCall#execute` 往下追到 `okhttp3.RealCall#getResponseWithInterceptorChain` 

![]({{site.baseurl}}/assets/img/okhttp-source-code-1.png)

可以發現這裡加入了一整大段的 `interceptor` 然後在下面 `chain.proceed` 就把請求的內容丟進這個 interceptor chain 之中，其中注意到一個 `RetryAndFollowUpInterceptor` 看上去非常可疑，進去看看

![]({{site.baseurl}}/assets/img/okhttp-source-code-2.png)

可以看到這裡有一段 `while(true)` 在 `realChain.proceed` 丟出 `RouteException` 或是 `IOException` 的情況下並且通過我們的 `recover` 的判斷之後有機會觸發這個 `continue`，而 `realChain.proceed` 其實就是往下執行後續的 `interceptor`，所以我們要繼續往下追，看看是哪裡有可能拋出 `RouteException` 跟 `IOException`

由於我們的情況是發生送了兩次一樣的 body 出去，所以我著重在尋找實際送出請求後讀取 response 這段，也就是 `okhttp3.internal.http.CallServerInterceptor#intercept` 在這裡才真正把 request 打出去

![]({{site.baseurl}}/assets/img/okhttp-source-code-3.png)

看到把 request body 完成送出後，開始讀 response header 這裡看到了一絲跡象，如果在這段有觸發 `IOException` 的話就會直接被丟出去了，外層也沒有其他的處理，會直接一路丟到 `RetryAndFollowUpInterceptor` 裡，再往裡面翻翻

![]({{site.baseurl}}/assets/img/okhttp-source-code-4.png)

看到了這段只要有 `EOFException` 就會噴 `IOException`，Http 的 EOF 大概就是換行了，抱持著如此想法，開始進行以下實驗

## 覆現實驗

首先我們準備一個 tcp 的 server，這裡我是用 node.js 架了一個非常簡陋的，收到請求之後印出內容，然直接斷線，什麼都不回傳

```javascript
var net = require('net');

const server = net.createServer(socket => {
    socket.on('data', data => {
        console.log(data.toString())
        socket.destroy()
    })
}).listen(3000)
```

接著用 `okHttpClient` 去打 `localhost:3000/test?t=123`，帶 path 跟 query parameter 是想看看這些參數會怎麼帶過來，沒想到運氣這麼好一次就被我矇到了，居然真的觸發 retry 了，噴錯的地方也跟我想的一樣

![]({{site.baseurl}}/assets/img/okhttp-source-code-5.png)

那該怎麼處理這個問題呢？我們回去翻翻 `okhttp3.internal.http.RetryAndFollowUpInterceptor#recover` 這裡是判斷要不要進行 retry 的地方，第一個條件式看到 `client.retryOnConnectionFailure()`

![]({{site.baseurl}}/assets/img/okhttp-source-code-6.png)

這個值可以在建立 `okHttpClient` instance 的時候做設定，透過方法 `retryOnConnectionFailure(false)` 就可以關掉了

實際測試後也的確在設定過後就不會再觸發 retry 了

## 結論

其實這個實驗跟調查也不能真正證明這就是我們遇到的情況，因為實際上我們不知道三方的服務有沒有這麼不合理的中斷連線

不過這可以證明，即便請求已經送到 application 而且處理完了，在送出完整的回傳之前可能發生了非預期的行為，還是有可能會觸發 okHttp 的重送，這就是我們遇到的情況了

所以最後我們就把 `retryOnConnectionFailure` 給關掉了，後續也沒有再發生這個問題了
