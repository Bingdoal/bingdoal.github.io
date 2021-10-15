---
title: 在 Git 上的檔案還原
date: 2020-11-10 09:30:00 +0800
category: dev-tools
img: cover/git.png
description: 常常在 GUI 上操作還原某段或是某個檔案的內容，但始終不知道背後的 git 是怎麼運作的，今天試著來研究並記錄一下實際的指令操作
layout: post
tags: [git,dev-tools]
---

{{page.description}}
## `git status`
了解還原的做法之前，先來弄懂 git 中的狀態有哪些，可以看到下面被分為
+ <span style="color: #00b700">Changes to be committed</span> (被加入暫存)
+ <span style="color: red">Changes not staged for commit</span> (尚未加入暫存)
+ <span style="color: red">Untracked files</span> (尚未被 git 追蹤的檔案)

![]({{site.baseurl}}/assets/img/git-status.png)

而根據不同的狀態，會需要使用到不同的指令去還原檔案

注意以下操作會使變更遺失，操作前請確定自己在做什麼
---

## `git checkout -- <file>`
`git checkout -- <file>` 是用來還原處於 <span style="color: red">Changes not staged for commit</span> 狀態的檔案，可以還原這個尚未被加入的變更

可以看到上圖中 test2.txt 同時處於 <span style="color: #00b700">Changes to be committed</span> 以及 <span style="color: red">Changes not staged for commit</span> 的狀態，當我對 test2.txt 下 `git checkout -- ` 之後只會還原 <span style="color: red">Changes not staged for commit</span> 的部分，而不會影響到 <span style="color: #00b700">Changes to be committed</span> 已經被加入到暫存的部分

![]({{site.baseurl}}/assets/img/git-checkout-status.png)

## `git clean -f <path>`
`git clean -f <path>` 是用來還原 <span style="color: red">Untracked files</span> 的檔案，會直接移除尚未被 git 追蹤的檔案，可以先用

```shell
git clean -n <path>
```

來預覽結果，確認好要移除的檔案之後再下 `git clean -f <path>`

![]({{site.baseurl}}/assets/img/git-clean-status.png)

可以確認一下 test3.txt 已經被從目錄下移除了

## `git reset <file>`

`git reset <file>` 則是用來還原**檔案的狀態**，將檔案從 <span style="color: #00b700">Changes to be committed</span> 還原到 <span style="color: red">Changes not staged for commit</span> 或是 <span style="color: red">Untracked files</span>，簡明之就是還原到 add 之前的狀態

![]({{site.baseurl}}/assets/img/git-reset-status.png)

然後再透過上面兩個方式去選擇要還原的方法

## `git reset --hard`

如果說真的這次的所有變更都不想要了，不管檔案目前是什麼狀態，就是想回到最後一次 commit 的樣子的話也可以下

```shell
git reset --hard
```

注意這會放棄所有變更，在尚未 commit 的情況下這些變更是找不回來的，一般情況下會建議用

```shell
git stash
```

來暫存這些變更，如果後續有需要還可以透過下面指令來還原變更

```shell
git stash pop
```
