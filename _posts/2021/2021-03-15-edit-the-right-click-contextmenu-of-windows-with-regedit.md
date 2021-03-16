---
layout: post
title: 編輯 Windows 上的右鍵選單內容
date: 2021-03-15 09:59:03 +0800
category: others
img: cover/windows-regedit.png
description: 最近不曉得什麼原因個人電腦的右鍵選單一些內容遺失了，一些工具包含 Git、Cmder、IntelliJ 等等的右鍵選單內容都消失了，非常的不方便阿，在查明原因之前要先把他加回來才行，這篇就要來跑一遍如何在 windows 上手動編輯自己的右鍵選單
lang: zh-TW
tags: [others, windows, 踩雷紀錄]
---

{{page.description}}

#### 這邊用 Git 來做示範

+ 首先開啟 Regedit，然後找到以下路徑 `HKEY_CLASSES_ROOT\Directory`

![]({{site.baseurl}}/assets/img/regedit-directory.png)

可以看到展開的內容下有 shell 的資料夾，在登錄檔設定中被稱為`機碼`，這個 shell 的機碼裡面放的就是在目錄下的右鍵選單

`Background/shell` 則是代表進入資料夾之後在背景點右鍵的內容

![]({{site.baseurl}}/assets/img/directory-background-contextmenu.png)

#### 新增內容

那要新增自訂的功能只要在 `shell` 下面右鍵 `新增` => `機碼`，取的名稱會直接變為選單顯示的名稱，或是更改`機碼`內的`預設值`，如果想要選單顯示圖示的話可以在`機碼`內再新增一個`字串值` 命名為 `icon` 裡面填想要的 icon 路徑，也可以指定 Exe 為路徑也會套用一樣的圖示

![]({{site.baseurl}}/assets/img/regedit-create-contextmenu.png)

而要有實際的點選功能則要新增如圖中的 `command` 的`機碼`在該選單的下面，而這個 `command` 的`機碼``預設值`就是點選之後會執行的指令，並且執行路徑就預設在該目錄底下


#### 後記

按照上面步驟就可以簡單新增各種想要的指令到右鍵選單了，若不是遇到這個奇怪的問題也不會發現這件事，不過這讓我對 windows 的登錄檔有一個初步的了解，也許之後有機會可以深入看看還有什麼有趣的登錄檔可以調整，讓整個 windows 可以更加客製化