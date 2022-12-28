---
layout: post
title: "[踩雷紀錄] Vscode markdown snippets 無作用"
date: 2021-12-10 14:47:28 +0800
category: others
img: cover/vscode.jpg
description: "部落格文章的內容都是用 markdown 來撰寫的，在 Jekyll 的框架之下，頁面一些 Metadata 的欄位其實是蠻固定的，而固定的 template 就會想用 vscode 的 snippets 來幫忙，可以不用每次都複製貼上，但之前在設定過後一直都沒有辦法正常運作，這問題困擾了好一陣子，最近才試著找看看有沒有辦法，沒想到很簡單就搞定了"
lang: zh-TW
tags: [踩雷紀錄, others]
---

{{page.description}}

## 問題描述

首先在 vscode 的 markdown snippets 設定好下面的 snippet，可以幫助建立一個基本的 post 版面，包含時間都幫忙寫上

{% raw %}

```json
{
    "jekyll blog post template": {
        "prefix": "post",
        "body": [
            "---",
            "layout: post",
            "title: \"$1\"",
            "date: $CURRENT_YEAR-$CURRENT_MONTH-$CURRENT_DATE $CURRENT_HOUR:$CURRENT_MINUTE:$CURRENT_SECOND +0800",
            "category: $2",
            "img: cover/$3",
            "description: $4",
            "lang: zh-TW",
            "tags: [$5]",
            "---",
            "",
            "{{page.description}} ",
            "$0"
        ],
        "description": "markdown blog post template"
    }
}
```

{% endraw %}

但實際在 markdown 撰寫的時候會發現，打 `post` 不會有任何提示跳出

原來 vscode 預設 markdown 的提示功能是關閉的，想想也是合理，可以更專注在寫作上，如果想要開啟的話請在設定裡加上

```json
{
    "[markdown]": {
        "editor.quickSuggestions": true
    }
}
```

不過加上這個之後雖然我們的 `post` 會有提示也可以作用了，但是還會有其他內建的 Markdown 語法提示，稍微有點影響寫作體驗

這邊建議可以安裝一個 extension [Control Snippets](https://marketplace.visualstudio.com/items?itemName=svipas.control-snippets)

功能就像下面看到的，可以選擇想要開啟跟關閉的 snippet 功能，而且會區分內建跟 extension 提供的，如果是使用者自訂的則不能開關，可以利用這個把內建的 markdown 提示關掉

![Alt]({{site.baseurl}}/assets/img/control-snippets.png)

這樣就又回到我們乾淨的寫作環境了，還可以享有客製化的 snippets，真是太舒服了
