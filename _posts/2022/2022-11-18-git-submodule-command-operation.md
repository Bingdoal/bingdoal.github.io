---
layout: post
title: "Git submodule 相關操作"
date: 2022-11-18 10:33:33 +0800
category: dev-tools
img: cover/git.png
description: "最近在公司碰到一些案子有在 git repo 中用到 submodule 的機制，但總覺得有點複雜而且有的操作不是太直覺，決定來紀錄一下"
lang: zh-TW
tags: [git,dev-tools]
published: true
---

{{page.description}}

## Git submodule

submodule 的概念就是在一個 git repo 中又嵌套另一個 repo，並且版控的單位會變成 repo 的版號

便利於將兩個相關聯的 repo 放在一起，大部分 submodule 可能會用於公用的 lib 或是整個大專案就是由數個 module 組成的，那有時候也會用這種方式管理定版

## 新增

只要在 root repo 的目錄下，指定 submodule 的 repo url 以及要放的路徑，輸入下面指令

```bash
git submodule add <repo-url> <path>
```

要注意的是路徑不能是已經存在的目錄，這個指令會幫你建好目錄

輸入完會發現多了一個 `.gitmodules` 的檔案，裡面會紀錄 submodule 的設定

詳細到 `.git/config` 也會看到 submodule 相關以及 `.git/modules` 也會新增相關的 repo 資訊

submodule 主要就是這三個地方的設定來綁定的

設定完之後還要記得 commit，這時候下 `git status` 會發現一個變更是以 repo 為單位

這個變更根據的就是 repo 的 commit id，之後有更新 submodule 的話也就按照一般變更處理

## 同步 submodule

在一個全新的機器上如果需要拉取 submodule 的內容，直接 `git clone` 是抓不下來的

會發現 submodule 目錄裏面是空的，需要下面指令

```bash
git submodule update --init
```

如果是已存在的 submodule 要更新

```bash
git submodule update
```

如果主 repo 跟 submodule 都有更動需要一起更新的話可以

```bash
git pull --recurse-submodules
```

## 移除 submodule

移除就比較麻煩了，一直很疑惑為什麼沒有個指令可以幫忙直接做完這件事

```bash
git submodule deinit <submodule path>
git rm --cached <submodule path>
rm -rf <submodule path>
rm -rf .git/modules/<submodule path>
```

必須要用四行指令才能完整移除 submodule 真的麻煩