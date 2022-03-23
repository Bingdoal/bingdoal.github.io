---
layout: post
title: "在離開 ssh 連線之後持續在背景運作程式"
date: 2022-03-23 09:27:23 +0800
category: linux
img: cover/linux.png
description: "之前寫過在 Linux 的 terminal 下如何在背景運作程式，但是在離開 ssh 連線之後背景的程式也會跟著被關閉，尋尋覓覓終於找到一個方法可以在背景運作，還可以找回來的作法"
lang: zh-TW
tags: [linux]
published: true
---

{{page.description}}

## 問題描述

有個需求是要在 remote 的機器上背景執行一個 tty 程式類似於 `karaf`，並且在離開 ssh 之後還可以繼續執行，並且能夠回到該 tty 繼續使用

原本是朝向把程式弄成一個 daemon 在背景跑，但要能夠再次回到 terminal 操作就有點困難，而且要弄成 daemon 也不是個簡單的作法，因此誕生了這個方法

## tmux

這個解法是透過 `tmux` 這個工具程式完成的，之後有機會再詳細介紹，簡言之

`tmux` 是一款 terminal 下的多視窗管理工具，可以切分不同的 terminal 在同一個畫面，或是像頁籤一樣同時管理不同的 terminal 分頁，而每個分頁或視窗都可以獨立運行

這裡便是運用了 `tmux` 可以暫離 session 的特性，步驟如下:

1. 執行 `tmux`
2. 在 `tmux` 之中運行需要背景執行的程式
3. 輸入 `ctrl + b` 然後鍵入 `d`

這時候會發現你跳出 `tmux` 了，讓我們用 `ps -aux` 看看，會發現你的程式是有在運行的，並且離開 ssh 之後它還是在

而如果之後想要回到剛剛的 `tmux` 的話就輸入指令 `tmux attach`

如果上述的操作作了很多次，就會有很多 session 被丟到背景的話，可以輸入 `tmux ls` 可以看到所有的 session

並且透過 `tmux attach -t <session-name>` 可以回到指定的 session

### 其他操作
1. `ctrl + b` + `&`: 結束當前 window，如果是最後一個 window 就結束 session
2. `ctrl + b` + `c`: 建立新的 window
3. `ctrl + b` + `$`: 幫當前的 session 重新命名
4. `ctrl + b` + `s`: 用視覺化的方式切換 session 以及 window，還可以預覽內容，非常強大

應該會用到的就這幾個了

---

## 結語

這算是意外發現的收穫，之前就知道有 tmux 這款多視窗工具，但一直沒找到機會用，沒想到在這樣的場合用上了XD，tmux 其實是款非常強大的工具，希望日後有機會能多了解一下