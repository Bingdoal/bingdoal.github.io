---
layout: post
title: "Shell script 讀取檔案並解析類 csv 格式"
date: 2021-07-23 15:14:31 +0800
category: linux
img: cover/bash.jpg
description: 有時候在撰寫 shell script 時，資料來源可能是一份預先產生的檔案，可能是 log 或是一些多行的輸出，那就沒辦法用一般讀值的方式去處理，如果又涉及特殊的格式像是 csv 這樣的又該怎麼處理呢
lang: zh-TW
tags: [linux, shell script]
---

{{page.description}}

## 讀取多行的資料源

說是讀取檔案，但對於 shell script 其實就是讀取多行的資料源，一般的字串輸入可以透過參數傳入，但是多行的資料通常會希望一行一行讀取，可以寫成像是:

```bash
# test.sh
count=0
while read line; do
    let count++
    echo "$count $line"
done
```

使用上

```bash
ls -l | test.sh
```

透過 `while read line` 的寫法可以把傳入 script 的資料來源一行一行讀入

## 讀取檔案

那要用到讀取檔案上也可以像上面的做法

```bash
cat "file.txt" | test.sh
```

也可以透過檔案名稱把指定的檔案加入到資料流當中

```bash
count=0
while read line; do
    let count++
    echo "$count $line"
done < "test.sh"
```

而就像我們常用的 `grep` 等指令都是可以讀檔案又讀 pipe 傳入資料的可以參考下面寫法

```bash
function addLineNumber(){
    let count=0
    while read line; do
        let count++
        echo "$count $line"
    done
}

for file in $@; do
    echo $file
    addLineNumber < $file
done

addLineNumber
```

`read` 在預設會去讀取 `stdin` 的來源內容，因此可以用來接收 pipe 的資料

## 解析 csv
如果今天有個檔案如下

```
1,test,hello world
```

那想要讀取個別元素時可以寫成下面，`IFS` 是一個特別的變數，是 `read` 用來分割字段的依據，所以我們可以預先設定好 `IFS` 的值來做到字段的切割

```bash
IFS=","
while read number title content; do
    echo "number: $number, title: $title, content: $content"
done < "test.csv"
```

或是直接寫在 `while` 中

```bash
while IFS="," read number title content; do
    echo "number: $number, title: $title, content: $content"
done < "test.csv"
```

## 其他

如果檔案內容不是簡單的用一個 `,` 去隔開，而是有複雜結構的話，可以參考用[前篇](https://bingdoal.github.io/linux/2021/07/linux-string-manipulation-and-output-proccessing-command/)所提到的字串處理，來分析複雜的資料結構