---
title: 好用的 Command Line 工具 Cmder
date: 2020-11-12 09:30:00 +0800
category: Dev tools
img: cmder.jpg
description: 開發工作中經常會需要使用到 cmd 來進行操作，但是 windows 的 cmd、PowerShell 都不是太美觀，操作上也沒有很順手，通常都是為了用某些工具才會打開來用，今天就來看看這個功能強大的 command line 工具 cmder，用過之後甚至可以都在 command line 上面做操作就像在 linux 一樣
layout: post
tags: [dev tools, cmder]
---

開發工作中經常會需要使用到 cmd 來進行操作，但是 windows 的 cmd、PowerShell 都不是太美觀，操作上也沒有很順手，通常都是為了用某些工具才會打開來用，今天就來看看這個功能強大的 command line 工具 cmder，用過之後甚至可以都在 command line 上面做操作就像在 linux 一樣  

## 安裝
安裝就直接到 [Cmder 官網](https://cmder.net/){:target="_blank"} 去下載並解壓縮吧，Cmder 本身是可攜帶的應用程式，設定也是跟著 cmder 的目錄走的，所以可以放進隨身碟裡隨插隨用  

至於 Mini 跟 Full 的差別主要在於有沒有附帶 Git 的相關工具，個人是都選 Full，反正也沒差那一點空間  

解壓縮完我的習慣會加入執行路徑到環境變數 `PATH`    
然後也新增一下跟目錄的環境變數 `CMDER_ROOT`  
## 介面
先看看這個精美的介面  

![]({{site.baseurl}}/assets/img/cmder2.png)

> 因為我有調整過一些設定跟主題所以會長的不太一樣啦 😅  

不過預設的樣子看著就順眼許多了，除此之外可以看到底下，這是一個頁籤列，也就是你可以在同一個視窗開多個命令列工具，按下右邊的 ➕ 就可以新增想要的命令列，還可以新增不同來源或是帶著參數的命令列  

![]({{site.baseurl}}/assets/img/cmder3.png)
![]({{site.baseurl}}/assets/img/cmder4.png)
## 功能
cmder 內建了許多 linux 的指令功能，自己比較常用的像是 `ls`、`grep`、`cat`、`alias` 還有 pipe 的功能  

其中 `alias` 應該是我覺得最好用的，常常要打一些落落長的指令很麻煩，就可以藉由 `alias` 來設定   

```shell
dc=docker $*
dcc=docker container $*
dci=docker image $*
ll=ls -al --show-control-chars -F --color $*
ls=ls -a --show-control-chars -F --color $*
```

`$*` 是指接收後面所有的參數，也可以指定只接收一個或兩個參數在裡面指定就好 `$1`, `$2`...  

## 更改主題

cmder 跟 bash 一樣可以做到自定義樣式，這邊我是直接套用別人的主題 [](https://github.com/AmrEldib/cmder-powerline-prompt){:target="_blank"}  
把 source code 抓下來，把目錄下的 `.lua` 檔都放到 `%CMDER_ROOT%/config` 底下就好了，不過字體上還有點問題，所以要去抓一下 [Fira Code](https://github.com/tonsky/FiraCode/releases){:target="_blank"} 的字型來安裝，這個字型在 coding 的時候我也蠻喜歡用的  

下載好之後可以到設定裡去更改字型   

![]({{site.baseurl}}/assets/img/cmder6.png)

弄好後就像下圖這樣啦，賞心悅目多了，也可以到 github 找找還蠻多其他人自製的主題的  

![]({{site.baseurl}}/assets/img/cmder5.png)

## 設定
下面紀錄一下自己在用 cmder 的時候習慣的設定  

### 更換前綴字元
cmder 用了一個蠻特別的前綴字元 `λ`，但這個字元某些情況下會導致 bug，可能會讓第一個字元殘留刪不掉，所以這邊會把它換成 Linux 常用的 `$`  

更改方法呢去到 `%CMDER_ROOT%/vendor` 底下，找到 `clink.lua` 以及 `profile.ps1` 兩個檔案，分別是 base on cmd 跟 base on PowerShell 下的執行腳本，找到  

+ `clink.lua`  

```lua
local lambda = "λ" -- 改成 $
```

+ `profile.ps1`  

```powershell
Microsoft.PowerShell.Utility\Write-Host "`nλ " -NoNewLine -ForegroundColor "DarkGray" # 一樣改成 $
```

如果是跟我一樣套用 `cmder-powerline-prompt` 主題的話，要到 `%CMDER_ROOT%/config` 底下，找到  

+ `powerline_core.lua`  

```lua
plc_prompt_lambSymbol = "λ" -- 改成 $
```

這樣就完成囉  

### 右鍵快速啟動
有時候想要在檔案總管中直接開啟 cmder，希望可以像 git bash 一樣右鍵點開就有選項，然後可以在當前目錄打開  

那首先把 cmder 的執行檔目錄加入到環境變數的 PATH 底下，然後用管理員權限開啟 cmd 鍵入  

```shell
Cmder.exe /REGISTER ALL
```

![]({{site.baseurl}}/assets/img/cmder7.png)

右鍵選單出現啦  

### vim 設定位置
開始在用 vim 編輯文件後就會想改一些設定，讓操作上更方便一點，我們可以直接找到 vimrc 的位置在 `%CMDER_PATH%/vender/git-for-windows/etc` 下面，直接在裏頭更改個人喜歡的設定，如果發現改了沒用，有可能是也裝了 Git，那就直接到 Git 的資料夾底下，一樣 etc 下面可以找到 vimrc  

### 在 VSCode 使用 Cmder
cmder 用習慣之後，就會想到在其他地方用 terminal 的時候也能有一致的體驗，這邊寫一下在 VSCode 中使用的設定  

```json
{
    "terminal.integrated.shell.windows": "cmd.exe",
    "terminal.integrated.env.windows": {
        "CMDER_ROOT": "D:\\tools\\cmder"
    },
    "terminal.integrated.shellArgs.windows": [
        "/k D:\\tools\\cmder\\vendor\\init.bat"
    ]
}
```
