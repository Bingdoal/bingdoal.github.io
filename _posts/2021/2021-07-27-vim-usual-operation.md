---
layout: post
title: "Vim 的常用操作"
date: 2021-07-27 09:55:24 +0800
category: dev-tools
img: cover/vim.png
description: 雖然現在有很多功能強大的 IDE 諸如 IntelliJ、VSCode 或是 Eclipse，但在 linux server 上時常都是只有 command line mode 的，以上的編輯器也就都派不上用場了，不可避免的需要使用到 vim 這類的純文字編輯器，因此筆記一下一些常用到的操作
lang: zh-TW
tags: [linux, ide, dev-tools]
---

{{page.description}}

## 前言

以前看過使用 vim 當主要編輯器的同事，裝上各種插件外掛後該有的功能一個也不少，絕對可以媲美 GUI 上的 IDE，但全部操作的快捷鍵都要背起來真的是有點硬，容我以後再研究，因此本篇會專注在個人常用的 vim 的原生操作，不會有額外插件外掛一類的

首先要先知道 vim 的操作被分為四個模式，分別為 `一般模式(Normal-mode)`、`插入模式(Insert-mode)`、`指令模式(Command-mode)`、`選取模式(Visual-mode)`，每個模式的操作視角都不相同，以下會分別為各個模式的操作作介紹

## 一般模式

一開始進入 `vim` 就預設為一般模式，此模式不可直接編輯檔案內容，主要透過一些指令操作內容或是對光標的操作，不過主要在此模式會進行的是模式的切換:

|       快捷鍵       | 功能                                                                 |
| :----------------: | :------------------------------------------------------------------- |
| `a`、`s`、`i`、`o` | 進入插入模式，每個快捷其實有些不同的行為，但基本是用來切換到插入模式 |
|   `:`、`/`、`?`    | 進入指令模式                                                         |
|  `v/V``ctrl + v`   | 進入選取模式                                                         |

而如果要從各個模式切換回一般模式只要輸入 `Esc` 就可以了，下面會介紹一般模式下對於內容的編輯以及光標的操作:

|   快捷鍵   | 功能                                                                      |
| :--------: | :------------------------------------------------------------------------ |
|    `dd`    | 剪下一行                                                                  |
|    `dw`    | 從光標開始，剪下一個單字                                                  |
|   `daw`    | 剪下目前所指的一整個單字                                                  |
|    `x`     | 剪下一個字母                                                              |
|    `yy`    | 複製一行                                                                  |
|   `yyp`    | 複製一行到下一行，不會進剪貼簿                                            |
|    `p`     | 貼上                                                                      |
|    `w`     | 移動到下一個單字                                                          |
|    `b`     | 移動到前一個單字                                                          |
|    `0`     | 移動到行首，也可以用功能鍵 `Home`                                         |
|    `$`     | 移動到行尾，也可以用功能鍵 `End`                                          |
|    `4G`    | 移動到第 4 行，行數可以指定，注意必須大寫，如果不輸入行數會移動到最後一行 |
|    `gg`    | 移動到第一行                                                              |
|    `*`     | 往下找到跟光標一樣的單字                                                  |
|    `#`     | 往上找到跟光標一樣的單字                                                  |
| `ctrl + f` | 往下翻頁，也可以用功能鍵 `Pg Dn`                                          |
| `ctrl + b` | 往上翻頁，也可以用功能鍵 `Pg Up`                                          |
|    `u`     | 返回上一步操作                                                            |
|    `~`     | 交換大小寫                                                                |
|    `ZZ`    | 保存並離開                                                                |

這些應該是比較常會用到的功能，其實一直都有點納悶為什麼沒有單純刪除的指令，只有剪下

輸入指令的時候可以在右下看到，幫助了解自己在輸入什麼

![Alt]({{site.baseurl}}/assets/img/vim-command-input.png)

## 插入模式

插入模式其實就是編輯模式，用來編輯檔案內容的，主要就是編輯，有個特別的功能是自動完成

輸入 `ctrl + p`，會有自動完成的功能或是清單顯示

![Alt]({{site.baseurl}}/assets/img/vim-auto-complete.png)

## 指令模式

指令模式顧名思義可以輸入 vim 相關的指令操作，一樣列出常用到的指令

|      指令       | 功能                                            |
| :-------------: | :---------------------------------------------- |
|      `:q!`      | 退出檔案不儲存                                  |
|      `:wq`      | 儲存並退出                                      |
|      `:w`       | 儲存                                            |
| `:w <filename>` | 另存新檔                                        |
|     `:undo`     | 返回上一動                                      |
|   `:! <cmd>`    | 有點像在 IDE 中開啟一個 terminal 執行指令的概念 |

除了以 `:` 開頭的指令之外，指令模式下也可以進行搜尋功能

|   指令    | 功能                                                                |
| :-------: | :------------------------------------------------------------------ |
| `/<word>` | 往下搜尋字段，字段後面加上 `\c` 代表不分大小寫，`\C` 則代表大小敏感 |
| `?<word>` | 往上搜尋字段                                                        |
|    `n`    | 找下一處                                                            |
|    `N`    | 找上一處                                                            |

指令模式下的也可以進行取代功能，取代的方式有點像 `sed` 的操作模式，指令模式下輸入

```bash
:<range>s/<source>/<replace>/<g,c,i>
```

+ `range`: 範圍可以是 `%` 代表整份檔案，或是 `1,10` 代表 1~10 行，或是不指定範圍表示光標在的那行
+ `g,c,i`: 最後的參數有三個選項，可以都選也可以都不選，都不選表示只取代第一個
  + `g`: 代表全部取代
  + `i`: 表示不分大小寫
  + `c`: 在每個字段被取代前進行確認是否取代
    + `y`、`n`: 這兩個選項應該不需要解釋
    + `a`: 代表從這個之後全部同意
    + `q`: 代表從這個之後全部不同意

## 選取模式

選取模式就像是用滑鼠在文件上拖曳一樣，會把光標移動的範圍反白，然後就可以針對反白的部分進行操作，針對光標的操作可以參考一般模式的部分，而下面會列出選取模式的專屬操作

首先講到進入選取模式時:

+ `v`: 一般選取模式，就像一般用鼠標拖曳一樣
+ `V`: 行選取模式，根據光標的移動把一整行反白
+ `ctrl + v`: 區塊選取模式，在換行選取時會像一些 IDE 的多游標選取一樣

| 指令  | 功能           |
| :---: | :------------- |
|  `d`  | 剪下選取內容   |
|  `y`  | 複製選取內容   |
|  `<`  | 選取行向左縮排 |
|  `>`  | 選取行向右縮排 |

## vim 設定

vim 本身也有許多可以客製化設定的部分，而這些設定也都可以在指令模式下進行變更

|          指令           | 功能                        |
| :---------------------: | :-------------------------- |
|        `:set nu`        | 顯示行數                    |
|       `:set nonu`       | 關閉行數                    |
|     `:set bg=dark`      | 黑暗主題                    |
|     `:set bg=light`     | 明亮主題，預設              |
| `:set tabstop=<number>` | 設定 tab 的空白長度，預設 4 |
|    `:set autoindent`    | 開啟自動縮排，預設不開啟    |
|   `:set noautoindent`   | 關閉自動縮排                |
|       `:syntax on`       | highlight 語法              |
|      `:syntax off`       | 關閉 highlight 語法         |

透過指令模式的設定會再下一次開啟 vim 之後就全部重置，如果想要持久化設定，可以將需要的設定寫到 `/usr/share/vim/vimrc`，這是全域的 vim 設定檔，可以在每個使用者的加目錄下自己新增一個 `.vimrc`，這可以只有在該使用者使用時套用，寫到檔案內的話設定前可以不需要加 `:`

---

## 結語

做為一個純文字編輯器，其實 vim 的功能是比想像中豐富的，操作的過程中其實也可以理解他的擁護者的想法，可以在操作文本的時候雙手不離開鍵盤真的是個還蠻舒服的體驗，往後有機會再來研究看看 vim 的各種插件。
