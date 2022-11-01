---
layout: post
title: "Linux Shell Script 常用語法 read, select, getopts"
date: 2022-10-27 14:03:31 +0800
category: linux
img: cover/bash.jpg
description: 繼上次的 shell script 初學之後又瞭解到更多實用的語法，目的是讓 shell script 更像個 command line tool 可以有參數或是使用者互動，主要會介紹 read, select, getopts 的用法
lang: zh-TW
tags: [linux, shell script]
published: true
---

{{page.description}}

## read
簡單實用的指令，就是用來讀取使用者輸入的，雖然多數 command line 都是直接帶 argument 來讀取參數

但有些情境下可以讓使用者有個互動的輸入會比較方便使用，下面是指令用法，應該看過就知道用途了

```bash
read INPUT
read -p "input: " INPUT
read -sp "input silent: " INPUT
read -ap "input array: " INPUT_ARRAY

echo $INPUT
echo $INPUT_ARRAY[0] $INPUT_ARRAY[1]
```

需要特別解釋的應該是 `-s`
是用來屏蔽敏感資訊的，例如密碼之類的

## select

用來給使用者提供選擇題

```bash
PS3="Select by number:"
select value in opt1 opt2 opt3 opt4; do
    echo "select $REPLY $value"
done
```
輸出會像下面

```
1) opt1
2) opt2
3) opt3
4) opt4
Select by number: 1
select 1 opt1
```

要注意的是 select 是一個迴圈，輸入完一個後會繼續要你輸入第二次

常會搭配 `case` 或是 `if` 來使用，如果不需要繼續回圈可以用 `break` 跳出

`PS3` 則是用來設定問題的提示，`$REPLY` 可以取得使用者輸入的內容，即便不屬於任何選項

## getopts

這應該是最常會看到的指令，多數 command line 工具都有參數選項的功能，這個指令就是提供這個功能，參考以下腳本

```bash
#!/bin/bash
while getopts "a:b:c" argv; do
  echo "$argv: $OPTARG"
done
```

執行如下

```bash
./test.sh -a 123 -b 456 -c
a: 123
b: 456
c:
```

直覺來看會有點誤會的是，其實這個參數選項是 `a:`, `b:`, `c` 這樣三組設定

`:` 表示這個選項需要參數，如果僅執行 `./test.sh -a` 則會有以下錯誤

```bash
./test.sh: option requires an argument -- a
```

不想要有這個錯誤或是想對錯誤進行額外處理可以參考下面腳本

```bash
while getopts ":a:b:c" argv; do
  echo "$argv: $OPTARG"
done
```

其實就是在選項多加一個 `:` 在最前面，執行如下

```bash
./test.sh -a
:: a
```

如此就不會有錯誤，而且還可以接收到需要的值

常搭配 `case` 使用，主要會用來設定一些旗標跟值

還有一個常用的參數 `$OPTIND` 可以拿到 getopts 總共的參數數量，如果需要選項以外的參數帶入，則可以參考以下操作

```bash
while getopts ":a:b:c" argv; do
  echo "$argv: $OPTARG"
done

shift (($OPTIND-1))
echo $1
```

執行如下

```bash
./test.sh -a 123 -b 456 ttttt
a: 123
b: 456
ttttt
```