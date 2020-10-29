---
layout: post
title:  "在 github page 上實現 category archive"
date:   2020-10-29 18:21:13 +0800
category: blog
img: jekyll-github.png
description: 總是看別人的部落格都有些酷酷的功能，自己的也想來一些，一開始就先從幫文章分類開始吧
lang: zh-TW
tags: [blog, jekyll, github page]
---

總是看別人的部落格都有些酷酷的功能，自己的也想來一些，一開始就先從幫文章分類開始吧  

其實一開始套上主題的時候覺得蠻空虛的，還以為整套功能都已經被別人實作好了~~(到底要多懶~~，總之有需多想要的功能還是得自己想辦法  

第一個想要的功能就是幫文章分類，可以藉由類別來查找文章，方便查找也方便歸類，就像現在左邊的樣子  

![]({{site.baseurl}}/assets/img/blog-category-shot.png)

然後點選連結就可以到一個分類的專用頁面，然後列出這個分類下的文章

![]({{site.baseurl}}/assets/img/blog-category-shot2.png)

---

那就來看看應該怎麼實作這件事

## [jekyll-archives](https://github.com/jekyll/jekyll-archives){:target="_blank"}
其實 jekyll 有個好用的插件可以簡單幫我們達到這件事，只要利用這個 Plugin 再加上他的設定:  

```yml
plugins:
  - jekyll-archives
jekyll-archives:
  enabled: all
  layout: archive
  permalinks:
    year: '/:year/'
    month: '/:year/:month/'
    day: '/:year/:month/:day/'
    tag: '/tag/:name/'
    category: '/category/:name/'
```

以及相關的 layout 撰寫好
{% raw %}
```liquid
<h1>
    Archive of posts with 
    {{ page.type }} '{{ page.title }}'
</h1>
<ul class="posts">
    {% for post in page.posts %}
    <li>
        <span class="post-date">
            {{ post.date | date: "%b %-d, %Y" }}
        </span>
        <a class="post-link" 
            href="{{ post.url | relative_url }}">
            {{ post.title }}
        </a>
    </li>
    {% endfor %}
</ul>
```
{% endraw %}
然後記得加入 Plugin 後要鍵入 `$ bundle` 可以自動幫忙下載跟引用，以上都完成後，功能就完成了  

對，就這麼簡單，而且還包好 tag 還有時間分類，你只要加上連結就好，只不過....**github page 不支援 😢**  

前篇文章有說過 github page 不支援所有的 jekyll plugin，我只是沒想到這感覺很基本的功能居然也不支援，好吧，是我功課做的不足，不過如果你不是在 github page 上架設的話，這個 Plugin 真的是很好用  

## Github Action

既然 Plugin 辦不到，那還有什麼辦法做到 category archive 的功能呢?  

找著找著，我找到了這篇文章 [Automated Jekyll Archives for GitHub Pages](https://aneejian.com/automated-jekyll-archives-github-pages/){:target="_blank"}  

他是利用 Github 上的 Action 功能來達到的，過去從沒用過 Action 功能的我也被吸引了，感覺像是什麼黑魔法一樣，實際上我們來試試看  

首先創建一個資料夾 `_archives` 並在裡面創建一個檔案 `archivedata.txt` 裡面寫入以下內容

{% raw %}
```liquid
---
---
{
    "categories": [
        {% for category in site.categories %}
        "{{ category[0]}}"{% unless forloop.last %},{% endunless %}
        {% endfor %}
    ],
    "tags": [
        {% for tag in site.tags %}
        "{{ tag[0] }}"{% unless forloop.last %},{% endunless %}
        {% endfor %}
    ],
    "years": [
        {% for post in site.posts %}
        "{{ post.date | date: "%Y" }}"{% unless forloop.last %},{% endunless %}
        {% endfor %}
    ]
}
```
{% endraw %}

然後建立三個 layout 在 `_layout` 的資料夾裡，分別是  
+ `archive-categories.html`
+ `archive-tags.html`
+ `archive-years.html`

這邊提供我自己的 category layout

{% raw %}
```liquid
<div class="category-title">
    <strong>Category:</strong><span>{{ page.title }}</span>
</div>
{% for post in site.categories[page.category] %}
<article class="post">
    {% if post.img %}
    <a class="post-thumbnail"
        style="background-image: url({{"/assets/img/" | prepend: site.baseurl | append : post.img}})"
        href="{{post.url | prepend: site.baseurl}}"></a>
    {% else %}
    {% endif %}
    <div class="post-content">
        <h2 class="post-title"><a href="{{post.url | prepend: site.baseurl}}">{{post.title}}</a></h2>
        <p class="post-desc">{{ post.description | strip_html | truncatewords: 15 }}</p>
        <div class="post-info">
            <div class="post-tags">
                Tags:
                {% for tag in post.tags %}
                <span class="post-tag">
                    <a href="{{site.baseurl}}/tags#{{tag}}"># {{tag}}</a>
                </span>
                {% endfor %}
            </div>
            <p class="post-date">{{post.date | date: '%Y, %b %d'}}</p>
        </div>
    </div>
</article>
{% endfor %}
```
{% endraw %}

然後在專案目錄下創建一個資料夾 `.github` 裡面再創一個目錄 `workflows` 裡面塞一個檔案 `add_archives.yml` 內容如下  

```yml
name: Generate Jekyll Archives
# description: Generate categories, tags and years archive files.
on:
  workflow_dispatch:
  push:
    paths:
      - "_posts/**"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Generate Jekyll Archives
        uses: kannansuresh/jekyll-blog-archive-workflow@master
        with:
          archive_url: "<your-site-case-url>/archives/archivedata"
          archive_folder_path: "_archives"

      - name: setup git config
        run: |
          git config --global user.name "GitHub Actions Bot"
          git config --global user.email "<>"
          git pull
      - name: commit
        run: |
          git add --all
          git commit -m "Created and updated archive files." || echo "No changes to commit."
          git push origin master || echo "No changes to push."
```
記得在