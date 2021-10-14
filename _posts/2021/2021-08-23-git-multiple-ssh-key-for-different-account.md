---
layout: post
title: Git 設置 SSH Key 在一台裝置上管理不同帳號的專案
date: 2021-08-23 14:00:53 +0800
category: dev-tools
img: cover/git.png
description: 開始工作之後同時會有個人的 git repo 以及公司工作用的 git repo，一般來說會用一台專門處理工作的電腦來區分，但有時候會需要在個人裝置或是公司裝置上用到對方的專案，就像上班忙裡偷閒寫的這篇部落格一樣，如果要在不同專案間切換帳號也不是很方便，因此調整一下設定可以在一開始就指定好選用的帳號
lang: zh-TW
tags: [dev-tools, git]
---

{{page.description}}

# 產生 SSH key
這個步驟呢可以參考下[github 的官方教學](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

不過因為我們要用到兩把 key，務必記得分開命名不然會被取代掉

# Config

然後對 SSH key 作一點設定，新建一個設定檔 `~/.ssh/config`，預設沒有這個檔案，內容寫下:

```yaml
# 預設 key
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    User git
    IdentityFile ~/.ssh/id_rsa
# 工作用 key
Host work.github.com
    HostName github.com
    PreferredAuthentications publickey
    User git
    IdentityFile ~/.ssh/work_rsa
```

每個區塊第一行的 `Host` 是一種給 ssh 的識別，識別到這個 hostname 就會自動替換成下面設定的 `HostName`，並且按照下面 `IdentityFile` 設定的 key 去連線

# Remote Url

用這種方法需要在專案 clone 的時候或是事後設定專案的 url，就替換成需要的 host:

```bash
$ git clone git@work.github.com:compony/demo.git
$ git clone git@github.com:bingdoal/demo.git
```

以上就可以用不同的 key 去抓不同帳號的專案，在專案下的操作也會直接使用該 key 去操作

# 設定使用者

用以上的方法的確就可以操作不同的專案，但是每個專案都會使用 git 全域設定的 username 跟 email，有時候還是想區隔開來因此可以只設定該專案的 username 跟 email:

```bash
$ git clone --config user.name=forWork --config user.email=forWork git@work.github.com:compony/demo.git
```

也可以寫成 alias:
```bash
$ git config --global alias.clone-work clone --config user.name=forWork --config user.email=forWork
$ git clone-work git@work.github.com:compony/demo.git
```

這樣比較方便之後使用，也不會忘記名稱跟 email