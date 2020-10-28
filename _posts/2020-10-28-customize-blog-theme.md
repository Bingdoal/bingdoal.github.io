---
layout: post
title:  "客製化自己的 blog 主題與樣式"
date:   2020-10-28 14:55:00 +0800
category: blog
img: jekyll-github.png
description: blog 架好了，但是預設的樣式當然不能滿足我們，既然都自己架 blog 了接下來當然就是客製化的時間啦!!
lang: zh-TW
tags: [blog, jekyll, liquid]
---
blog 架好了，但是預設的樣式當然不能滿足我們，既然都自己架 blog 了接下來當然就是客製化的時間啦!!  

不過由於自己以前也沒有接觸過 Ruby 和 Liquid template 所以就找些網路資源來拼拼湊湊吧  

# 主題
首先來挑個別人寫好的主題吧，直接 google `jekyll theme` 就可以找到很多結果，也有很多的網站都幫你收集好可以用了  

不過如果像我一樣有在 github 上部屬的需求的話建議關鍵字改成 `github page jekyll theme` 或者直接到 [github-pages-themes](https://jekyllthemes.io/github-pages-themes){:target='_blank'} 找  

挑好之後把整個專案拉下來，然後把你的 `_post` 換上去，然後把 `_config.yml` 的相關內容改一改，基本上這樣就套用好別人的主題啦~

# 客製化
抓下別人做好的主題之後，接下來就是慢慢來研究到底是怎麼運作的，進而客製化自己想要的功能跟版面啦，這裡用我套用的 [flexible-jekyll](https://github.com/artemsheludko/flexible-jekyll){:target='_blank'} 主題來舉例  

## Jekyll & Liquid template
既然要來改裡面的東西，免不了要來了解一下使用的工具  

### Jekyll
Jekyll 是個輕量級的靜態網站生成並附帶一個 hosting server，Jekyll 不使用資料庫而是透過解析 Markdown 和 Liquid 的語法來生成靜態 html  

個人式想像成輕量版的 php 吧，就是巨集式的生成需要的文本內容，對 html 有一定了解的話掌握起來還蠻快的，Liquid 寫起來也很直覺  

不確定詳細 Jekyll 是怎麼運作的，但就使用上來推測是將執行目錄下的所有檔案都用 liquid 來解析，然後生成並且 host `_site` 的資料夾，生成時會透過 `_config.yml` 來讀取設定像是:  

+ 環境變數
+ 略過的檔案與資料夾
+ 一些 Plugin 與其相關的設定

### Liquid
來講講 Liquid 的一些內容，先來看看[官方文件](https://shopify.github.io/liquid/){:target='_blank'}  

可以發現文件內容其實不多，很容易就可以上手了  

原本想舉例，但發現寫上來會被 jekyll 解析到😂，所以範例就作罷吧。