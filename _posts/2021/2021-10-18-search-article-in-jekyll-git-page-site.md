---
layout: post
title: "在用 Jekyll 架的 Git page 中實現搜尋功能"
date: 2021-10-18 09:35:45 +0800
category: blog
img: cover/jekyll-github.png
description: Git page 上的文章開始多了起來之後連自己想要找文章都不太方便，但在 Jekyll 產生的靜態網頁中是沒有一個後端伺服器來提供文章列表的 API，用一般去 fetch 後端 API 是行不通的，下面介紹是怎麼實作出這個功能的
lang: zh-TW
tags: [blog, jekyll, github page]
---

{{page.description}}

既然是靜態網頁那能夠想到的實作方式只能是純前端的作法，那首先要先想辦法產生我們的文章列表的 `json` 供 `javascript` 使用，參考以下寫法:

{% raw %}

```liquid
[
    {% for post in site.posts %} {
        "title": "{{ post.title | escape }}",
        "tags": "{{ post.tags | join: ', ' }}",
        "url": "{{ site.url }}{{ post.url }}",
        "description": "{{ post.description | truncate: 100 }}",
        "date": "{{ post.date | date: '%d %b %Y' }}"
    } {% unless forloop.last %}, {% endunless %}
    {% endfor %}
]
```

{% endraw %}

經過 liquid 轉換後大概長的像:

```json
[
    {
        "title": "Git 設置 SSH Key 在一台裝置上管理不同帳號的專案",
        "tags": "dev-tools, git",
        "url": "https://bingdoal.github.io/dev-tools/2021/08/git-multiple-ssh-key-for-different-account/",
        "description": "開始工作之後同時會有個人的 git repo 以及公司工作用的 git repo，一般來說會用一台專門處理工作的電腦來區分，但有時候會需要在個人裝置或是公司裝置上用到對方的專案，就像上班忙裡偷閒...",
        "date": "23 Aug 2021"
    } ,
    ...
]
```

有了這個結構就可以很容易的幫助我們對 `json` 操作的方式來達到搜尋的目的，部分程式碼如下:

{% raw %}

```javascript
var allArticle = JSON.parse(`[
    {% for post in site.posts %} {
        "title": "{{ post.title | escape }}",
        "tags": "{{ post.tags | join: ', ' }}",
        "url": "{{ site.url }}{{ post.url }}",
        "description": "{{ post.description | truncate: 100 }}",
        "date": "{{ post.date | date: '%d %b %Y' }}"
    } {% unless forloop.last %}, {% endunless %}
    {% endfor %}
]`);
function startSearch(event) {
    let inputTxt = event.target.value.toLowerCase().trim();
    if (inputTxt == "" || inputTxt == null) {
        return;
    }
    // 多字段分析
    let searchTxts = inputTxt.split(" ");
    let articles = allArticle;
    let searchResultTemplate = "";
    let searchedArticle = [];
    // 根據 title description tags 的內容做搜尋
    for (let j = 0; j < searchTxts.length; j++) {
        searchedArticle = [];
        for (const idx in articles) {
            let article = articles[idx];
            const searchTxt = searchTxts[j];
            if (article.title.toLowerCase().includes(searchTxt) ||
                article.description.toLowerCase().includes(searchTxt) ||
                article.tags.toLowerCase().includes(searchTxt)) {
                searchedArticle.push(article);
            }
        }
        articles = searchedArticle;
    }
    // Html 組合
    for (let i = 0; i < searchedArticle.length; i++) {
        const article = searchedArticle[i];
        searchResultTemplate += getArticleCardHtml(article);
    }
    let searchResultDiv = document.querySelector("#search-result-div");
    if (searchResultTemplate != "") {
        searchResultDiv.innerHTML = searchResultTemplate;
    } else {
        searchResultDiv.innerHTML = `
                <div class="search-not-found">
                    查無結果
                </div>`;
    }
}
```

{% endraw %}

搜尋的時候包含 title、tags、description 的內容都去做查詢，盡可能的找出使用者想要的內容，還加入的以空格為分割符號的多字段搜尋，希望可以做到盡可能智慧的搜尋功能

想實際看看功能的可以到 [Search](https://bingdoal.github.io/search/) 玩玩看，有任何建議也可以在下方留言告訴我

---

## 結語

以上是個人部落格搜尋上的做法，是直接把 liquid 嵌入在 javascript 的變數裡，而不是寫成另外的 json 然後再用 fetch 去取得內容，有看過其他人是這樣實作的，感覺就像是真的有個 API 在那邊一樣，但既然都是用 liquid 來產生的，這樣好像有點不必要，還會浪費額外的流量，那不如一開始就放在 javascript 的變數之中，也省了一個步驟
