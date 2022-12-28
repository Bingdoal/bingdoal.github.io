---
layout: post
title: 利用 ngrok 代理本地服務到外網，並套用 https
date: 2022-08-15 11:47:12 +0800
category: dev-tools
img: cover/ngrok.png
description: 後端工作中常會有需要將本地的測試環境給外部服務存取，而很多第三方服務由於安全性問題大多都只支援 https，這時候就可以透過今天要介紹的工具 ngrok 來達到目的
lang: zh-TW
tags: [dev-tools, https]
published: true
---

{{page.description}}

## ngrok

ngrok 是一個第三方服務用來進行本地服務的反代理

簡單說就是用戶端會先訪問到 ngrok 然後再由 ngrok 轉封包到本地

而這個轉導不限於 http protocol 也支援 tcp

### 安裝

[官網](https://ngrok.com/)下載安裝即可

### 事前準備

在官網註冊登入後應該會進入到個人的 dashboard

使用前需要拿到個人的 auth token，並且設定 ngrok

```sh
ngrok config add-authtoken <your-token>
```

### 指令

上面設定好後使用上非常簡單只需要

```sh
ngrok http 80
```

接著會看到精美的執行畫面

```sh
ngrok                                                                                                                          (Ctrl+C to quit)

Hello World! https://ngrok.com/next-generation

Session Status                online
Account                       bingdoal (Plan: Free)
Version                       3.0.6
Region                        Japan (jp)
Latency                       -
Web Interface                 http://127.0.0.1:4040
Forwarding                    https://1bcc-114-34-167-55.jp.ngrok.io -> http://localhost:80

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

可以看到 ngrok 已經幫我們把本地的 80 port 代理出去了，只要訪問 `https://1bcc-114-34-167-55.jp.ngrok.io` 就可以連通我們的服務，並且幫我們加上了 https

只不過免費方案只能夠同時代理一個服務，也沒辦法固定 domain 每次重啟都是隨機的，只能用來暫時測試
