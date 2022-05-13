---
title: 強大的測試工具 Jmeter
date: 2020-11-26 09:30:00 +0800
category: others
img: cover/jmeter.png
description: 當專案開發到達一定規模之後，通常會開始引入自動化測試，自動測試可以幫助維護程式品質，確保一次次的變更之下還能夠按照預期的運行，在重購或是加入新功能上都有一定的幫助，今天就要來介紹一款好用的自動測試工具 Jmeter，各種強大的功能之下 Jmeter 都可以玩出一套學問了，今天這篇就先簡單講解基本用法
layout: post
tags: [test, jmeter]
redirect_from:
- /test/2020/11/powerful-test-tool-jmeter/
---

當專案開發到達一定規模之後，通常會開始引入自動化測試，自動測試可以幫助維護程式品質，確保一次次的變更之下還能夠按照預期的運行，在重購或是加入新功能上都有一定的幫助，今天就要來介紹一款好用的自動測試工具 Jmeter，各種強大的功能之下 Jmeter 都可以玩出一套學問了，今天這篇先簡單講解基本用法

# Jmeter 基本介紹

Jmeter 是一款由 Apache 開源的自動測試工具，本體用 Java 撰寫，可以用於功能以及效能上的自動測試，支持多線程執行可用作壓力測試，支援多種執行方式，包含:

+ 後端 API (HTTP、HTTPS、SOAP)
+ FTP
+ Database
+ Mail Server
+ TCP

... 等等其實還有蠻多功能是沒接觸過的 😅

自己最常用的還是用來測後端 API 吧，最大的好處是可以模擬多個使用者同時使用的情境，對於實際情況的模擬有蠻大的幫助的，也由於是用 Java 撰寫的，在各個 OS 上都可以使用，只要事先準備好 JRE 的執行環境就可以了

[官網](https://jmeter.apache.org/)
[Github](https://github.com/apache/jmeter)

## 下載使用
首先到上面的官網下載 Jmeter，只會有一個壓縮檔，解壓縮出來直接到 `bin/` 底下執行 `jmeter` 就會自己開啟了，個人喜歡把它加入到環境變數裡面，可以直接在任何地方用 cmd 開，如果像我一樣有加入環境變數的話，執行加入下面參數就可以直接開啟測試專案

```shell
jmeter -t "<jmx path>"
```

## 基本設定
剛開啟後會發現，預設介面的主題不知道為什麼設計的不是很好看，我們可以到選單中 `Option -> Look and Feel` 裡面去挑選想要的主題

![]({{site.baseurl}}/assets/img/jmeter-ui-setting.png)

但是每次重開的時候還是會載入預設選項，因此我會直接去更改預設，先去找到 `jmeter.properties`，然後找到下面那行

```properties
jmeter.laf.windows_xp=javax.swing.plaf.metal.MetalLookAndFeel
```

可以把預設主題更改為白色的，也可以替換成自己想要的主題，後面內容去替換掉就好

## Thread Group
主要的執行單位，顧名思義是由 Thread 組成的 Group，底下的所有執行操作都可以是獨立的 Thread，可以想成是獨立的使用者，可以看到底下的一些設定

![]({{site.baseurl}}/assets/img/jmeter-thread-group-setting.png)

可以指定這個 Group
1. 底下要有幾個 Thread
2. 執行間距時間
3. 迴圈執行次數

### Sampler
基本的執行單位，有非常多種的取樣器可以供我們使用，基本上就端看需要什麼測試行為，就去找自己想要的，如果真的找不到，也可以用 `BeanShell` 藉由自己寫腳本來達到目的，有關 `BeanShell` 之後也會額外開一篇來記錄

### Config Element
如果是像我主在用 `Http Request` 的話，應該要注意一下這類的元件，主要用來設定像是共同的根路由、header 還有 cookie 都會用到

### Controller
控制器主要則是提供一些基本的邏輯操作，主要常用的像是 迴圈、條件判斷，就像在寫程式一樣可以做到流程控制

其中 `Simple Controller` 應該是最常用到的，本身沒有功能，主要用來幫助分類 Sampler，專案管理上會比較直覺，視覺上也會清楚很多，就像是一個資料夾的概念

![]({{site.baseurl}}/assets/img/jmeter-simple-controller.png)

### Pre/Post Processor
Jmeter 也提供前處理以及後處理的行為，可以在 Sampler 執行之前或之後做一些特別的處理，比較常用到的像是，用 `JSON Extractor` 來解析 request 之後得到的結果阿，或是直接寫一個 `BeanShell` 對資料做一些特別的處理阿

`Pre/Post Processor` 比較複雜，之後也會另外撰寫一篇來講解一些用法

### Assertion
Sampler 雖然本身也有自己的 assertion，像 `HTTP Request` 就預設回傳 2xx 以及 3xx 才算是通過，但有時候我們預期的結果就是需要它失敗，或者不單純是看 Sampler 本身行為，那可能就需要加入自己客製化的 assertion，來制定我們所預期的結果

自己常用的像是 `Response Assertion`、`JSON Assertion`、`BeanShell Assertion`，也是可以另開一篇來講😅

## 執行測試
以上的設置都完成之後，就可以開始執行我們的測試啦，按下那個綠色的執行箭頭

![]({{site.baseurl}}/assets/img/jmeter-exec-test.png)

## View Result Tree
執行完測試之後有沒有莫名其妙，什麼都沒發生的感覺

對，我們需要一個看結果的地方，所以我們可以在專案中加入 `Listener -> View Result Tree`，這個元件會幫我們收集所有 Sampler 執行完的結果

![]({{site.baseurl}}/assets/img/jmeter-view-result.png)

這樣測試結果就很一目瞭然了

### 自動清除 log
每次執行完測試之後，最後的結果會直接 append 在紀錄上，會有點難以閱讀，還要手動清除有點麻煩，所以這邊我們寫一個腳本來幫助自動清除

+ 首先建立一個 `setUp Thread Group`，這個 Group 會優先於一般的 `Thread Group` 去執行
+ 然後在底下新增一個 `BeanShell Sampler`，加入下面的 code

```java
import org.apache.jmeter.gui.GuiPackage;
import org.apache.jmeter.gui.JMeterGUIComponent;
import org.apache.jmeter.gui.tree.JMeterTreeNode;
import org.apache.jmeter.samplers.Clearable;

log.info("Clearing All ...");

guiPackage = GuiPackage.getInstance();

guiPackage.getMainFrame().clearData();
for (JMeterTreeNode node : guiPackage.getTreeModel().getNodesOfType(Clearable.class)) {
    JMeterGUIComponent guiComp = guiPackage.getGui(node.getTestElement());
    if (guiComp instanceof Clearable){
        Clearable item = (Clearable) guiComp;
        try {
            item.clearData();
        } catch (Exception ex) {
            log.error("Can't clear: "+node+" "+guiComp, ex);
        }
    }
}
```

之後每次執行測試專案的時候，就都會先來執行這段腳本把全部的 Log 清除掉

### 自動覆寫
View Result Tree 也可以直接產出結果到檔案，但是每次產出都詢問要不要覆寫總有點令人煩躁，我們可以透過更改設定來解決這個問題

+ 找到 `jmeter.properties`，直接加入，或是找到下面內容把註解拿掉

```properties
# ASK : Ask user
# APPEND : Append results to existing file
# DELETE : Delete existing file and start a new file
resultcollector.action_if_file_exists=DELETE
```

這樣就可以直接執行不用被打斷囉~

## 結語

寫一寫都覺得有點心虛 😅，因為各個元件可以玩的東西真的蠻多的，也不希望太過專注一些內容，而導致篇幅過長或模糊掉重點，所以真的就是蜻蜓點水的提過一下，簡單講講使用流程，之後也會再詳細分享一些自己常用到的元件