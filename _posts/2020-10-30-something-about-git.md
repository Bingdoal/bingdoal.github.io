---
title: 一些 Git 的使用心得
date: 2020-10-30 23:32:00 +0800
category: Dev tools
img: git.png
description: 最近因為工作，還有架設部落格的關係，才又開始頻繁接觸使用 Git，想把最近的一些使用上的心得，還有一些設定，紀錄也分享一下，方便以後回顧
layout: post
tags: [git,dev tools]
---

最近因為工作，還有架設部落格的關係，才又開始頻繁接觸使用 Git，想把最近的一些使用上的心得，還有一些設定，紀錄也分享一下，方便以後回顧 

# 簡介
## 版本控制
先簡單解釋一下 Git 的概念，Git 是一個版本控制的軟體，最初是由 Linux 的作者為了更好的維護和管理 Linux 而設計出來的。  

版本控制很常聽到，而目的上大概可以想成為了把產品的每個穩定版本備留下來，而做的一個行為。傳統方式可能透過不同的資料夾命名、FTP、壓縮... 等等方法，但是這會導致混亂，每個版本間的差異沒辦法被體現出來，人為控管的方式也可能導致版本不可信，而且也更占用空間資源  

所以才需要用自動化管理的軟體來做這件事，Git 就是為此而生  

## Git
Git 的設計在程式碼修改上有很突出的表現，每次版本間的差異用 commit 為單位，也就是每次變更的提交  

而變更是以程式碼每一行的差異，或是檔案為單位去記錄，在版本間的比較上會非常明確，可以知道是哪個檔案，哪一行有出現差異，提交本身就成為最好的版本標記，再微小的提交都可以成為一個版本，而也因為是記錄變更，而不需要占用到龐大的硬碟空間  

![]({{site.baseurl}}/assets/img/git-diff.png)

Git 也可以在各個提交點中，自由切換也可以開分支，做到垂直、平行的版本切換，也就達到了版本控制的目的並且不會遺失任何版本  

![]({{site.baseurl}}/assets/img/git-graph.png)

## Git 平台
Git 是一套軟體，基本上是運作在本機並且以一個專案的根目錄為範圍，去監控這一整個專案，去做到這個專案的版本控制，那要怎麼團隊合作開發上達到版本控制且安全的協作呢?  

這就要仰賴以 Git 為核心的雲端服務平台，較知名的有 `GitLab`、`GitHub`、`Bitbucket` 等等，這些平台幫你把專案放在雲端上，並且用 Git 為核心軟體來管理，就像是大家遠端到平台上面利用 Git command 一起互動  

![]({{site.baseurl}}/assets/img/git-platforms.png)
# Git Command
## 基礎流程
簡單紀錄說明下常用的 Git 指令的使用與目的  

首先要設定一下你的基本資料，這些資料會與你的變更一起被 commit 上去，明確的表示出，這個變更是由誰所進行的  
```shell
git config --global user.name "<your-name>"
git config --global user.email "<your-email>"
```

接下來在你修改程式碼或變更任何檔案之後，再來查看現在變更中的檔案  

```shell
git status
```

也可以看到變更的細節，到底是哪一行程式碼被更動了  

```shell
git diff
```

確認好更動之後，利用下面兩個指令，去提交你的變更  

```shell
git add <filepath that you want to commit>
git commit -m "<commit message>"
```
  
如果有利用到 Git 平台的話，最後再 push 上去，就可以將本地的變更，同步到平台上去了

```shell
git push
```

## 版本控制
會了基本同步之後，要來看看到底是怎麼做到版本控制的  
首先來看看之前我們做的 commit 到底都被記錄在哪裡了  

```shell
git log
```
![]({{site.baseurl}}/assets/img/git-log.png)  

這可以看到之前所有的 commit 紀錄，每個 commit 都會帶一個很長的 hash 值，代表這個 commit 的位置，基本上我們只需要看到前 7 碼就好了
> log 很長的話要退出只要輸入 q 就好  

這個 hash 可以用來查看這個階段的變更，一樣透過前面的指令  
```shell
git diff de986d9
```

或者也可以用來做版本的切換  
```shell
git checkout de986d9
```

也可以直接讓專案回到這個版本  
```shell
git reset de986d9
```

要注意這會讓 `de986d9` 之後的 commit 直接在視線中消失，比較建議的會是下面的指令

```shell
git revert de986d9
```

差異在於， revert 是把重設到 `de986d9` 的這件事，當成另一個變更，你不會遺失任何版本  

而在較有規模的專案開發中，大多數都會有切分支的需求，分支的概念就是從現在的版本拉一個平行世界出去，主要用來區隔不同功能或開發者的變更，不會直接衝突到一起  
```shell
git branch <branch-name>       # 建立分支
git checkout <branch-name>     # 切換到分支
git checkout -b <branch-name>  # 建立並且切換到分支
```

分支開發到一個階段之後，還是要回歸主版本那就要靠  
```shell
git merge <branch-name>
```

merge 可能會發生 collision，也就是所謂的衝突，有相同的檔案一起做了變更  

這時候會進入 MERGE 的模式，Git 會幫你把 collision 的區段畫起來，由開發者自己去決定是哪個變更要留下，或是再重新撰寫新的變更  

然後再提交一包上去作為 Merge 的 commit，至此就算是解完衝突了  

# 其他

## Alias
在用 git 開發的時候會很頻繁的用到很多 git 的指令，要一直打非常的麻煩阿，所以可以利用 git alias 的功能，幫指令取別名，打起來就快速多了，下面列出我自己設定的別名  

```shell
git config --global alias.st status
git config --global alias.ch checkout
git config --global alias.ls "log --format='%C(yellow)%h - %C(magenta)%an%C(green)<%ae>, %C(cyan)%ad : %C(auto)%C(ul)%s' --date=short"
git config --global alias.ignore "!gi() { curl -sL https://www.gitignore.io/api/$@ ;}; gi"
```

使用上只要  
```shell
git st   # 就等於 git status
```
第二個 alias 可以看到幫 git log 寫了一堆參數，因 git log 的部分直接秀出來的排版實在有點難看，所以可以用參數格式化一下，詳細可以看看這篇文 [git log 進階應用](http://jamestw.logdown.com/posts/238719-advanced-git-log){:target="_blank"} 還有一些 filter 的功能方便查找  

可以看到第三個 alias 最前面有個 ! 意思是這個是不屬於 git 的內部指令的，連這都可以 alias，使用起來真的是相當方便。這個指令的作用，是去 [gitignore.io](https://www.toptal.com/developers/gitignore){:target="_blank"} 的網站抓取常用的 `.gitignore` 的內容，也是很方便的東西  

如果想要取消別名只要  
```shell
git config --global --unset alias.st 
```
就好囉

## Tag

前面有講到要定位到想要的 commit 可以透過自動產生的 hash，但這個 hash 並不是這麼好記本身也不含有意義  

這時候可以利用 `git tag` 的指令為 commit 貼上標記  

```shell
git tag <tag-name>                              # 在最後的 commit 處貼上 tag
git tag <tag-name> de986d9                      # 在 commit de986d9 處貼上 tag
git tag <tag-name> de986d9 -a -m "tag message"  # 在 commit de986d9 處貼上 tag 並且附帶 "tag message" 的訊息
git tag -d <tag-name>                           # 刪除 tag
git tag                                         # 列出所有的 tag
git tag -l 'v1.*'                               # 列出符合 v1.* 的所有 tag
```
那之後使用這個 tag 就等於會指向到被貼上的 commit 處了，有點像是幫 commit 取別名的概念  
而如果有用到 git 平台的話，一般的 push 並不會把 tag 推上去，要透過一些參數  

```shell
git push origin <tag-name>          # 把 <tag-name> 推上去
git push origin --tags              # 把全部的 tag 同步上去
git push --delete origin <tag-name> # 刪除遠端的 tag
```
tag 應用上比較常見的是貼上 stable 的版號，也方便統一管理或比較版本間的差異  

## .gitignore
在你的專案底下建立 `.gitignore` 的檔案，git 在對專案監控檔案變更的時候，就會忽略底下的檔案，不過也是可以強制加入的  
```shell
git add -f <filepath>
```

如果說檔案在 `.gitignore` 建立前就被加入變更了，那 git 是不會捨棄對這個檔案的監控的，可以透過下面指令來除去對他的監控  
```shell
git rm --cached <filepath>
```

或是用下面指令，一次掃除所有應該被忽略的對象  
```shell
git clean -fx
```

## .gitkeep
這個算是一個大家約定俗成的東西，而不是強制的  

有時候你的專案會需要這個資料夾，但底下可能是沒有檔案的，預設在 git 變更中是不會加入空資料夾的，這時候就可以在裡面新增一個 `.gitkeep` 的檔案，來表示這個資料夾要留著，裡面不需要任何內容  