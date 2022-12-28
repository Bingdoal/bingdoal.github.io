---
layout: post
title: "gRPC 雙向串流"
date: 2022-12-25 19:54:57 +0800
category: network
img: cover/gRPC.png
description: "之前簡介的時候寫過 gRPC 是有提供雙向串流的，也就是說可以由 server 主動推播訊息過去，也可以由 client 不間斷的傳送訊息過來，這篇就來記錄一下相關的用法，實作一個簡單的訂閱推播系統"
lang: zh-TW
tags: [network, protocol, gRPC]
published: false
---

{{page.description}}
