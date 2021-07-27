---
layout: post
title: 最好的 Java IDE --- Intellij
date: 2021-01-14 17:24:50 +0800
category: dev-tools
img: cover/intellij.png
description: 個人通常用來撰寫 Java 及 Dart，最大的缺點大概就是 IDE 重開很花時間，但是不影響他的強大使用體驗，Intellisense 速度快且精確，語法支援上很友善，自動完成的提示可以有效優化程式碼，程式碼檢查功能嚴謹，啟動後運行速度快，使用體驗相當良好，各種插件也是功能強大，下面就慢慢來介紹各個強大之處
lang: zh-TW
tags: [ide, java, intellij, dev-tools]
---

{{page.description}}

## 優點
### 追蹤程式碼的能力極強
你只需要 `ctrl + 左鍵` 可以追蹤參考、追蹤使用，還可以追蹤外部引入的 library，即使被編譯成 jar 依舊可以 trace，十分強大。

另外強烈建議使用 IntelliJ 要使用有側鍵的滑鼠，側鍵的功能可以返回上一個 focus，當你在頻繁 trace code 的時候比較不容易迷失。

### 輕鬆一鍵重構
寫程式不免需要重構程式碼，常見的可能有重新命名、抽換方法、移除功能、移動功能 ...，IntelliJ 都內建好這些功能了，這種苦工式的重構就交給工具來辦吧。

### 最懂你需求的 Intellisense
這應該是 IntelliJ 最強大的地方，跳出的語法提示基本上前一兩個就會是你要的，舉例來說下面例子，IntelliJ 第一個提示會跳出 `getName` 而不是按照字母排序，也不會是 `setName`，這就是它強大的地方，IntelliJ 會根據前後文判斷提示顯示的優先順序。

![]({{site.baseurl}}/assets/img/intellij-intellisense1.png)

甚至在命名變數的時候還會給你適當的名稱提示，根本就是一個最好的 coding 助手。

### 內建 Git 的整合
IntelliJ 對於 Git 的支援度非常的高，它本身就是一個不錯用的 Git GUI 工具，包含 commit graph 或是查看 history 都有非常漂亮的介面提供

## 常用快鍵
不免俗要善用工具，一些快鍵要盡可能的熟用，以下是我常用到的快捷鍵

+ `ctrl + Y`: 刪除一行
+ `ctrl + alt + L`: 自動排版
+ `ctrl + D`: 複製到下一行
+ `ctrl (+ shift) + F`: (全域)搜尋
+ `ctrl (+ shift) + R`: (全域)取代
+ `ctrl (+ shift) + N`: 找 class (找檔案)
+ `ctrl + G`: 跳至指定行數
+ `ctrl + E`: 查詢最近開啟的檔案
+ `ctrl + shift +  ↑/↓`: 移動行到 上/下
+ `ctrl + shift + ←/→`: 往 左/右 整段選取
+ `alt + 鼠標拖曳`: 多行選取
+ `shift + enter`: 在行尾新增換行
+ `shift + F6`: 重構重新命名
+ `alt + enter`: 快速完成
+ `alt + F12`: 開啟/關閉 Terminal
+ `ctrl + /`: 選取行註解
+ `ctrl + 左鍵` 或 `中鍵`: 追蹤定義
+ `alt + ←/→`: 切換左右分頁
+ `alt + 1`: 開關專案結構

## 好用插件
+ `Lombok`: 算是開發 java 必裝，是 java 的一個 library 用法，運用 annotation 的方式去自動產生一些常見的 methods 例如: getter、setter...，不過在一般開發中，會缺乏這些方法導致沒有語法提示，還會出現錯誤，加入這個 plugin 可以避免這個狀況
+ `.ignore`: 對於 gitignore 的支援，可以幫被 ignore 的檔案上灰色，也可以對語法做檢查
+ `ideolog`: 檢視 log 檔可以 trace code 跟 highlight 十分方便。
+ `Grep Console`: 幫 console 上色，預設會抓一些 log 的關鍵字，也可以自定義要 match 的字去上色
+ `rainbow bracket`: 幫助不同層的括號上不同的顏色
+ `Code miniMap`: 可以觀看到 code 的迷你地圖
+ `Json Editor`: 幫助 json 格式轉換

### 有趣的插件
一些惡搞得插件，有興趣可以裝來玩玩，不過拿來開發的話，個人是覺得有點煩躁😂
+ `Rainbow Fart`: 可以偵測開發時的關鍵字，播對應的語音鼓勵開發者
+ `active-power-mode`: 打字會累積 combo，combo 越高特效越華麗
+ `Power Mode II`: 不用累積 combo 就會有打字特效