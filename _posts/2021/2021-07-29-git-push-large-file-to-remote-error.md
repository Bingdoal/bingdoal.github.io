---
layout: post
title: "[踩雷紀錄] git 無法推送大型檔案至 repo"
date: 2021-07-28 16:48:11 +0800
category: dev-tools
img: cover/git.png
description: "最近在進行更版的時候一個不注意把一個 tar 檔一並加入到了 commit 當中，而當我要推上 repo 的時候就發生了錯誤，原因似乎是 git 不允許推送單一檔案超過 100 MB，當下真是有點慌張，因為加入這個檔案之後我又做了幾次變更，送了幾個 commit，心想著紀錄永遠不會從 git 中消失，那這個檔案不就無解了，還好還是有被我找到解方，做法也不複雜，特別筆記一下這個用法"
lang: zh-TW
tags: [dev-tools, git, 踩雷紀錄]
---

{{page.description}}

由於這個檔案已經被 commit 過了所以即使在新的 commit 中移除它也於事無補，git 會留存所有版本的紀錄，所以必須要另外想方法解才行

**在開始前一定要記得第一步一定要先編輯 `.gitignore`，再一次把大型檔案加入提交這種事情是一定要避免的**

## Error Message
先記錄下當下看到的錯誤訊息:

```
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com
remote: error: Trace: dff1555...
remote: error: See http://git.io/iEPt8g for more information.
remote: error: File test.tar is 261.83 MB; this exceeds GitHub's file size limit of 100.00 MB To https://github.com/.../.git

! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/.../.git'
```

屏蔽了一些不重要的資訊，明確的告知了哪個檔案過大導致了問題的發生，並且也有標示出發生的 commit，這讓我們很容易可以追蹤到問題

## 情境一: 剛剛才 commit 掉大型檔案
如果是這個情況，那恭喜，要做的事情簡單很多，只要兩行就可以解決了

```bash
git rm --cached test.tar
git commit --amend -C HEAD
```

+ `git rm --cached` 可以將檔案移出 git 的紀錄當中
+ 而 `git commit -amend` 則可以修改 commit 紀錄重新包一份給它

只要這兩行就可以簡單解決了

## 情境二: 大型檔案的 commit 在三個 commit 之前
這個情境就是筆者遇到的，在經過了幾個 commit 之後才發現曾經把不該加入的檔案提交了😢

那首先根據剛剛的錯誤訊息可以得知這個檔案是位在 commit `dff1555...`，讓我們先看看這個 commit 的位置在哪
```shell
git log --pretty=oneline --abbrev-commit
```

可以知道問題發生在前三個 commit，那我們要回朔到再往前一個 commit 來去進行修改

```
706d14191 最後一個 commit
810f4dbaf 倒數第二個 commit
dff155505 罪魁禍首
dd81500d8 問題發生前一個 commit
...
```

找到 commit 位置後輸入下面指令:
```shell
git rebase -i dd81500d8
```

`git rebase -i` 是用來幫忙整理 commit 紀錄的，可以用來修改 commit 的內容以及 commit 訊息，輸入之後會開啟 git 的預設編輯器並看到以下畫面:

```
pick dff1555 罪魁禍首
pick 810f4db 倒數第二個 commit
pick 706d141 最後一個 commit

# Rebase dd81500..dff1555 onto dd81500 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

接著在我們要修改的 commit 前面，把 `pick` 改成 `edit`，如下所示

```
edit dff1555 罪魁禍首
pick 810f4db 倒數第二個 commit
pick 706d141 最後一個 commit
...
```

接著就會進入 `rebase` 的階段，git 會帶你回到你要 edit 的 commit 階段，這時候就像是前一個情境一樣，你的 HEAD 回到了 `dff1555` 的位置，那操作也是一樣的，最後再離開 `rebase` 就好

```shell
git rm --cached test.tar
git commit --amend -C HEAD
git rebase --continue
```

## 情境三: 大型檔案的 commit 散佈在各處，且有多處變更
如果真的有個意外，讓這個大型檔案流傳了好幾個 commit，而且不同的 commit 間還有變更，那就不是改一個 commit 可以解決的問題了

```shell
git filter-branch --tree-filter "rm -f test.tar"
git push -f
```

其實這個解法好像更簡單，不過要確定操作正確，不然刪錯東西就麻煩了，下面簡單說明下指令:
+ `git filter-branch`: 這個指令的用途呢，其實就是 `checkout` 到每一個版本去做批次的操作，根據參數的 filter 去決定要做的操作
+ `--tree-filter`: 這個 filter 則是代表要針對每個版本的檔案去做修改
+ `"rm -f test.tar"`: 跟前面的指令與參數結合代表，切換到每個版本去刪除掉 `test.tar` 這個檔案
+ `git push -f`: 最後你更改了整個分支樹所以要強制推上 repo 去覆蓋

`git filter-branch` 其實還有蠻多用法的，如果有遇到特別的情境的話可以考慮再來寫一篇

---

## 結語
雖然是遇到這個 error 才特別查的做法，但其實不限於刪除大型檔案上，如果真的有一定要更動 commit 的需求也可以比照辦理，更加理解到了 git 的強大