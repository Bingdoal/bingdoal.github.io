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

不過如果像我一樣有在 github 上部屬的需求的話，由於 github 並不全面的支援 jekyll 的所有 plugin，所以一些主題在 github 上可能功能會殘缺不全，所以建議關鍵字改成 `github page jekyll theme` 或者直接到 [github-pages-themes](https://jekyllthemes.io/github-pages-themes){:target='_blank'} 去找，往後搜尋相關功能的時候也記得加入 `github page` 的關鍵字，才不容易走冤枉路    

挑好之後把整個專案拉下來，然後把你的 `_post` 換上去，然後把 `_config.yml` 的相關內容改一改，基本上這樣就套用好別人的主題啦~

# 客製化
抓下別人做好的主題之後，接下來就是慢慢來研究到底是怎麼運作的，進而客製化自己想要的功能跟版面啦，這裡用我套用的 [flexible-jekyll](https://github.com/artemsheludko/flexible-jekyll){:target='_blank'} 主題來舉例  

## Jekyll & Liquid template
既然要來改裡面的東西，免不了要來了解一下使用的工具  

### Jekyll
Jekyll 是個輕量級的靜態網站生成並附帶一個 hosting server，Jekyll 不使用資料庫而是透過解析 Markdown 和 Liquid 的語法來生成靜態 html  

個人是想像成輕量版的 php 吧，就是巨集式的生成需要的文本內容，對 html 有一定了解的話掌握起來還蠻快的，Liquid 寫起來也很直覺  

不確定詳細 Jekyll 是怎麼運作的，但就使用上來推測是將執行目錄下的所有檔案都用 liquid 來解析，然後生成並且 host `_site` 的資料夾，生成時會透過 `_config.yml` 來讀取設定像是:  

+ 環境變數
+ 略過的檔案與資料夾
+ 一些 Plugin 與其相關的設定

### Liquid
來講講 Liquid 的一些內容，先來看看[官方文件](https://shopify.github.io/liquid/){:target='_blank'}  

可以發現文件內容其實不多，很容易就可以上手了  

{% raw %}
+ 變數命名  

```
{% assign a = 10 %}
```
+ 輸出內容

```
{{ a }}
```
+ 流程控制
  
```html
<!-- if else -->
{% if a == 10 %}
    ...
{% elseif a == 5 %}
    ...
{% endif %}

 <!-- unless 就是 if not 的概念 -->
{% unless a == 10 %}
    ...
{% endless %}

<!-- case when -->
{% case a %}
    {% when 10 %}
        ...
    {% when 5 %}
        ...
{% endcase %}
```
+ 迴圈

```
{% for a in array %}
    ...
{% endfor %}
```
+ 迴圈基本常用的就是 for 不過還有很多神奇用法，只是在寫部落格不常用到，詳細就看官方文件吧
+ 其他精隨還有在於 pipe filter 的串接用法例如:

```html
{% assign users = "bingdoal, peter, lisa" | split:", " %}
{% for user in users %}
    <p>{{ user }}</p>
{% endfor }
```
liquid 提供的 filter 語法非常多，詳細用法就請查看官方文件去使用吧  

還有個重要的是讓我能在這裡順利嵌入 liquid 語法的 [raw](https://shopify.github.io/liquid/tags/raw/){:target="_blank"}，可以讓包在其中的 liquid 語法跳脫不被解析     
> 這邊有踩到一個雷，raw 跨越的行數好像不能太多，跨越太多後面會亂掉

## layout
接下來研究一下 layout 的部分，先來看到 `index.html` 會發現頂部會有以下設定  

```
---
layout: main 
---
```
這個 main 的 layout 呢，我們可以在 `_layout` 的資料夾底下找到，然後可以發現在一團 html 跟 liquid 的語法之中，有一個 `content` 的變數 

```html
    ...
<div class="content-box clearfix">
  {{ content }}
</div>
    ...
```
這應該是 jekyll 幫我們做好的，會根據設定的 layout 把文本內容帶到 `{{ content }}` 的位置，而 layout 裡面也可以再嵌套一層 layout，可以看到 main layout 還有再嵌入 `layout: default`，層層嵌套進去組合而成我們要的頁面，主要可以讓你不需要重複撰寫一堆 html，也方便統一設計跟維護，也可以專注在單一功能的開發  

{% endraw %}

{% raw %}

## include
在看到 `default.html` 的 layout 時，可以發現有像是 `{% include head.html %}` 的語法在其中，這段會去找 `_include` 資料夾底下的文本去嵌入到指定位置，跟 layout 的功能有點相反的感覺，可以用來引入一些共用的設定，像是 css style 或是第三方的工具，像這個主題就有內建 Disqus 和 google analytics  

{% endraw %}

---
這篇大概說明一下客製化主題的過程，下一篇會講講自己想要弄一個 Category 的功能的歷程，就是現在頁面左邊的那排，根據設定的分類產出分類頁面，找了幾個方法，也踩了不少的雷，最終為什麼會決定用這個方法等等的，下一篇再分享分享。