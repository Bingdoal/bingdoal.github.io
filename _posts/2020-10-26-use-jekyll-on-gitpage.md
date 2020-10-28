---
layout: post
title:  "用 jekyll 在 gitpage 上架 Blog"
date:   2020-10-26 14:21:13 +0800
category: blog
img: jekyll-github.png
description: 用 Jekyll 架設在 github 上用 markdown 舒服撰寫的 Blog
lang: zh-TW
tags: [blog, jekyll, gitpage]
---
# 前言
想了好久終於是弄了一個 Blog 出來了，一直以來都是用 [hackmd.io]{:target="_blank"} 在撰寫筆記想說哪天 [hackmd.io]{:target="_blank"} 出個部落格功能再說，拖了這麼久終於還是自己搞了一個

## 選擇 gitpage 跟 jekyll 的原因
1. 個人寫作喜好
+ 最主要的點就是，個人實在是非常喜歡用 markdown 寫作的感覺，可以在手不離開鍵盤的情況下完成整篇文章，不過據我了解沒有什麼 Blog 系統是可以完全用 markdown 寫的，在這之前則是都把筆記整理在 hackmd.io
   
1. 客製化程度高  
+ 雖然本身沒有接觸過 ruby 但經過一些 survey 之後覺得最好的方案應該還是用 jekyll 自架是比較好的選擇，之後客製化上也比較方便，也不需要學習操作別人弄好的後台
   
3. 架設方便  
+ 架在 gitpage 上則單純不想租機器或雲端空間來弄這個，git 的系統也能幫我做好備份跟更改記錄管理，實在也是蠻理想的

## [jekyll](https://jekyllrb.com/)
+ jekyll 的功用就是幫助將 markdown、liquid 等語法轉換為靜態 HTML 的工具，重點是 Gitpage 原生支援
+ 因為 jekyll 本身則是用 Ruby 開發的，因此使用之前我們先來裝 [Ruby]{:target="_blank"}，官網直接下載安裝，然後鍵入

```
$ gem install jekyll
```

這樣就有了 jekyll 的 cli 了，接著鍵入
```
$ jekyll new <project-name>
```

建立你的靜態網頁專案，跑完之後專案結構如下
```
├ _post
│   └ 2010-10-26-title.markdown
├ _config.yml
├ .gitignore
├ 404.html
├ about.markdown
├ Gemfile
├ Gemfile.lock
└ index.markdown
```

大致應該算好理解，可以直接執行
```
$ jekyll serve
```

來看看執行結果的網頁看起來怎樣
+ 跑完後會發現多了兩個資料夾 `_site`, `.jekyll-cache` 這兩個是 jekyll 自動根據專案目錄的檔案產生的

+ 最主要的文章會根據 `_post` 底下的 markdown 解析出來，依據檔名格式 `yaer-month-date-title.md` 來解析，只根據檔名，所以 `_post` 下的目錄結構要怎麼放都隨你

+ 再來就是 `_config.yml` 應該算是重點，各種設定還有換主題都在這邊，***不過有點複雜之後再研究***，可以先改上自己的部落格名稱跟介紹

## Gitpage
+ 專案建起來了，接下來就是上架

1. 在自己的 github 上開一個 `<username>.github.io` 的 repo，這個專案名稱會默認為你的 gitpage
2. 第二步就很簡單了，把你的專案 push 上去，接著到 `<username>.github.io` 下就能看到結果了

## 開寫
將要發表的文章放在 `_post` 裡並且檔名按照 `year-month-date-<title>.md` 來命名， Jekyll 會根據這個格式來解析存入環境變數，並且在 `_config.yml` 裡面也可以用來設定路徑相關  

撰寫的時候也可以為你的頁面撰寫 meta data 可以促進 SEO，也可以用來解析文章的 tag、category、description 方便用來分類或檢視大綱，以這篇文章的 meta 舉例:  
```markdown
---
layout: post
title:  "用 jekyll 在 gitpage 上架 Blog"
date:   2020-10-26 14:21:13 +0800
category: blog
img: jekyll-github.png
description: 用 Jekyll 架設在 github 上用 markdown 舒服撰寫的 Blog
lang: zh-TW
tags: [blog, jekyll, gitpage]
---
```

然後下面寫下你的 markdown 文章，然後 run `jekyll serve` 運行沒問題，就可以推上去啦。
[hackmd.io]: http://hackmd.io
[Ruby]: https://www.ruby-lang.org/zh_tw/ 