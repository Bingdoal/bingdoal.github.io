---
layout: post
title:  "用 jekyll 在 gitpage 上架 Blogger"
date:   2020-10-26 14:21:13 +0800
categories: [blogger]
---

## 先來說下使用 gitpage 跟 jekyll 的原因
1. 最主要第一點就是，個人實在是非常喜歡用 markdown 寫作的感覺，可以在手不離開鍵盤的情況下完成整篇文章，不過據我了解沒有什麼 Blogger 系統是可以完全用 markdown 寫的，在這之前則是都把筆記整理在 hackmd.io
   
2. 雖然本身沒有接觸過 ruby 但經過一些 survey 之後覺得最好的方案應該還是用 jekyll 自架是比較好的選擇，之後客製化上也比較方便，也不需要學習操作別人弄好的後台
   
3. 架在 gitpage 上則單純不想租機器或雲端空間來弄這個，git 的系統也能幫我做好備份跟更改記錄管理，實在也是蠻理想的

## [jekyll](https://jekyllrb.com/)
+ jekyll 的功用呢就是幫助將，markdown、liquid 等語法轉換為靜態 HTML 的工具，且 Gitpage 本身支援使用
+ 而 jekyll 本身則是用 Ruby 開發的，因此使用之前我們先來裝 [Ruby](https://www.ruby-lang.org/zh_tw/)，官網直接下載安裝，然後鍵入

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

+ 再來就是 `_config.yml` 應該算是重點，各種設定還有換主題都在這邊，***不過之後再研究***，可以先改上自己的部落格名稱跟介紹

## Gitpage
+ 專案建起來了，接下來就是上架

1. 在自己的 github 上開一個 `<username>.github.io` 的 repo，這個專案名稱會默認為你的 gitpage
2. 第二步就很簡單了，把你的專案 push 上去，接著到 `<username>.github.io` 下就能看到結果了


{% if page.comments %}
{% endif %} 