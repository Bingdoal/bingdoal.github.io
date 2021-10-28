---
layout: post
title: "Linux 上的字串以及命令輸出處理 grep、awk、xargs、sed"
date: 2021-07-21 10:16:00 +0800
category: linux
img: cover/bash.jpg
description: 在 Linux 上工作時常會有需要針對 command 的輸出做一些處理，或針對我們需要的部分做字串操作，一般在程式語言中都會有原生或是第三方提供的 API 可以使用，那在 Linux 上的話也有類似的東西，就如同前一篇介紹的 shell script 一樣，用不同的 command 去做不同的處理，不僅僅可以在 bash 上使用，也可以套用到 script 之中，讓自動化工作更進一步
lang: zh-TW
tags: [linux, shell script]
---

{{page.description}}

## 原生操作
首先來介紹不需要用到額外 command 的一些字串操作

### length

```bash
test="12345"
echo ${#test} # 5
```

### substring

```bash
test="123456123456"
echo ${test:3} # 456123456
echo ${test:3:2} # 45  ${字串:起點:長度}
echo ${test#123} # 456123456 等同於 java 的 startWith 然後刪除最短 match 的部分，匹配不到則輸出完整字串
echo ${test#*3} # 456123456
echo ${test##*3} # 456 match 最長的部分
echo ${test%456} # 123456123 等同於 java 的 endWith 然後刪除最短 match 的部分，匹配不到則輸出完整字串
echo ${test%4*} # 123456123
echo ${test%%4*} # 123 match 最長的部分
array=("123" "456" "789")
echo ${array[@]} # 123 456 789
echo ${array array[1]} # 123 456
```

這裡比較特別的應該是 array 的部分，一定要用括號包起來並且用空格隔開元素

### replace

```bash
test="123456123456"
echo ${test/123/ttt} # ttt456123456 只取代第一個找到的
echo ${test//123/ttt} # ttt456ttt456 取代全部
echo ${test/#*3/ttt} # ttt456 取代開頭 match
echo ${test/%4*/ttt} # 123ttt 取代結尾 match
```

## 常用指令
接下來介紹下各種常運用的輸出處理指令，由於不是每次都需要撰寫 shell script 來處理輸出，所以這類指令就非常方便我們在 bash 中直接使用，當然也可以在 shell script 中去調用

### grep
首先介紹 grep 這應該是最常用的指令之一，簡單說就是用來做字串搜尋的

```bash
grep [參數] "關鍵字" [檔案1 檔案2 ...]
```

雖然可以直接用來搜尋檔案內容，不過最常用的還是搭配 pipe 來串流資料搜尋

```bash
ls -l | grep ".sh" # 輸出該行含有 .sh 的內容
```

還有幾個常用的參數如下
```bash
cat test.log | grep -A 10 "ERROR" # 輸出該行之外 多輸出後 10 行
cat test.log | grep -B 10 "ERROR" # 輸出該行之外 多輸出前 10 行
cat test.log | grep -C 10 "ERROR" # 輸出該行之外 多輸出前後 10 行
cat test.log | grep -P 10 "E\w+R" # 用 regex 來搜尋字串
cat test.log | grep -n "ERROR" # 輸出內容時帶上行數
cat test.log | grep -i "ERROR" # 不分大小寫比對
cat test.log | grep -v "ERROR" # 反向操作，不輸出匹配到的內容
cat test.log | grep --color=always "ERROR" # 把匹配到的內容上色，方便觀看
```

以上的參數操作都是可以混用的

### awk
awk 也是一個非常常用到的指令，主要功能就是用來篩選需要的 `欄位`

```bash
awk [參數] "腳本" [檔案1 檔案2 ...]
```

舉例來說今天我只想看到 `ps -aux` 輸出 PID 以及指令
```bash
ps -aux | awk {'print $2 "-" $11'}
```

這樣就可以只看到我想看到的訊息了，而不需要整行的其他資料

### xargs
xargs 是用來串聯參數帶入給其他指令的輔助指令，通常要跟其他指令串接結合使用，例如下面的範例就可以 `touch` 所有檔案，等同於把 `ls` 的全部檔案當作參數餵給 `touch`

```bash
ls | xargs touch
```

不過有時候指令不接受這麼多的參數，也可以去限制每次輸入的數量

```
ls | xargs -n 1 cat
```

其他參數還有:
+ `-r`: 忽略空字串為參數
+ `-t`: 顯示執行的指令
+ `-p`: 執行指令前顯示完整指令，並詢問是否執行

### wc
一個簡單用來計算的指令，可以計算行數、字數、位元組數，雖然是簡單的功能但挺實用的

```bash
wc [參數] [檔案1 檔案2 ...]
```

預設是行數、字數、位元組數都輸出
```bash
ls | wc
#      57      64     962
```

加上參數可以篩選自己想看的資訊:
+ `-l`: 只看行數
+ `-w`: 只看字數
+ `-c`: 只看位元組數
+ `-m`: 只看字元數，也就是半形字，中文的話換算成兩個字元

### sed
sed 則是一個比較萬用的字串處理工具，功能包含了取代、複製、刪除基本的處理都涵蓋了，只是操作上會稍微有點複雜

```bash
sed [參數] "腳本" [檔案1 檔案2 ...]
```

一樣地可以直接用來操作指定的檔案內容，但也可以用 pipe 的方式串聯在指令輸出之後

```bash
ls -l | sed "s/rwx/test/1"
```

#### 取代: s
最常用到的大概是這個所以寫在第一個，可以用來取代需要的字段，也可以視需求只輸出需要的行數
```bash
ls -l | sed "s/rwx/test/1" # 把每行找到的第 1 個 rwx 替換為 test
ls -l | sed "s/rwx/test/g" # 把每行找到的全部 rwx 替換為 test
ls -l | sed "s/rwx/test/g2" # 把每行找到的第二個之後的 rwx 都替換為 test
ls -l | sed -n "s/rwx/test/g2p" # 只輸出取代成功的行
```

#### 刪除: d
```bash
ls -l | sed "3d" # 刪除第三行內容
ls -l | sed "/rwx/d" # 刪除含有 rwx 字段的行數
```

#### 新增: a
```bash
ls -l | sed "a test" # 在每行輸出之後額外輸出一行 test
ls -l | sed "1a test" # 只在第一行後面一行輸出
ls -l | sed "1,5a test" # 在 1~5 行後面一行輸出
ls -l | sed "/rwx/a test" # 在含有 rwx 的行後面一行輸出
```

#### 插入: i
跟新增的操作基本一樣，不過是輸出在指定位置的前一行
```bash
ls -l | sed "i test"
ls -l | sed "1i test"
ls -l | sed "1,5i test"
ls -l | sed "/rwx/i test"
```

#### 替換: c
```bash
ls -l | sed "2c test" # 替換第三行整行的內容為 test
ls -l | sed "1,3c test" # 將 1~3 行的內容替換為 test，注意是三行取代成一行不是每行替換
ls -l | sed "/rwx/c test" # 把含有 rwx 的行都替換為 test
```

#### 其他
多個操作之間可以用 `;` 隔開
```bash
ls -l | sed "s/rwx/test/1"; "s/rw-/test2/1"
```
---
## 結語
這篇結合了多個自己在指令輸出時常用到的操作，最主要還是留個筆記，畢竟這麼多指令跟複雜的操作，一旦少用到就容易忘記，每次都要上網查也是麻煩，所以就在這裡做一個總整理