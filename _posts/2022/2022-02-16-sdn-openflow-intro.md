---
layout: post
title: "OpenFlow 之路(一) SDN、OpenFlow 簡介"
date: 2022-02-16 15:37:29 +0800
category: network
img: cover/openflow.jpg
description: "最近因緣際會接觸到一些 SDN 相關的工作，難得有這個機會就把所學到的紀錄一下吧，先簡單介紹下 SDN 以及 Openflow 的內容，後續則主要紀錄實作內容"
lang: zh-TW
tags: [network, openflow, sdn]
published: true
---

{{page.description}}

## SDN

Software-defined Networking 軟體定義網路，網路架構是由數台 Router 以及 Switch 彼此連接成為一張網子，傳統網路架構下封包的轉發邏輯需要每台 Switch 去個別設定，一旦網路架構複雜起來，難以對於每台的設定統一管理，但網路中只要有一個節點有問題就可能導致網路有重大疏漏，而這種複雜的拓樸如果要個別管理對於網管人員會是一種災難

SDN 最主要就是來解決這種問題，如下圖所示，SDN 架構主要把網路分成「控制層」跟「資料層」，簡單說就是抽離出一個統一的管理端來對各個 Switch 做設定，而要實現這種架構就必須要統一所有 Switch 的設定方式，這個方式就是 Openflow

![]({{site.baseurl}}/assets/img/sdn-structure.jpg)

因此網管人員可以透過一個控制端來操作所有的 Switch，也可以透過串接 OpenFlow 協議進一步實現更高層級的管理應用程式，更便利網管人員管理以及檢視網路拓樸的各種資訊與統計，Switch 本身也可以專注在資料傳輸的部分，對於效能提升、除錯都有顯著的幫助

## OpenFlow

OpenFlow 是 SDN 架構中 Controller 跟 OpenFlow Switch 溝通的重要 Protocol

![]({{site.baseurl}}/assets/img/openflow-intro.png)

OpenFlow 主要就是用來控制 OpenFlow Switch 中的各個元件，如圖中畫的 Controller 透過一個 Channel 溝通控制 Flow Table、Group Table、Meter Table

另外雖然只要是 OpenFlow Switch 都可以透過 OpenFlow 來操作，但是設定之後封包的 Pipeline 運作流程則視各家廠商的實作會有所不同，一些 OpenFlow Switch 同時也能處理非 OpenFlow 協議，這種混合式的 Switch 可能在封包轉送過程中就會用一些資訊來判別該封包是否要走向 OpenFlow 的 Pipeline

因此在實務使用上還是需要了解 Switch 本身的實作情境來調整

---

## 結語
由於 OpenFlow 各個元件希望可以搭配實作來做講解，這篇就簡單做個介紹而已，之後會針對 Flow Table、Group Table、Meter Table 各自寫一篇，實作上則是使用 OpenDaylight 以及 Mininet 來進行