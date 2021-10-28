---
layout: post
title: "Linux Shell Script 初學"
date: 2021-07-20 11:02:51 +0800
category: linux
img: cover/bash.jpg
description: 身為一個軟體工程師在 Linux 上操作指令是必須要學習的，雖然本人也是工作後才比較常用到😅，不過常常會有需要打一大串指令或是操作一些複雜邏輯的時候，這時候就可以撰寫 shell script 幫助處理複雜的指令，可以做到自動化的處理，是可以幫助工程師偷懶的利器，這篇主要做些簡單的筆記幫助自己不要忘記
lang: zh-TW
tags: [linux, shell script]
---

{{page.description}}

## 前言
首先要知道 shell script 其實不是一種程式語言，他的概念比較像是把指令腳本化，執行上類似直譯式的語言，因此 shell script 的使用上非常的彈性，只要是你可以在 bash 上直接使用的指令，shell script 也都可以使用，也因此要先對 linux 的基本指令操作有些了解才知道該撰寫什麼

## Hello world
第一個 script 不免俗地來寫個 `Hello world.sh`

```bash
#!/bin/bash
echo -e "Hello world!!!\n"
```

第一行的 `#!/bin/bash` 表示這個腳本的執行環境，然後撰寫完之後一定會發現不能執行，那是因為權限的問題，預設撰寫文本是不會有執行權限的，透過 `ls -l` 可以看到檔案的權限，然後透過下面指令來給予執行權限

```bash
chmod +x "Hello world.sh"
```

這樣就能成功執行了
```bash
$ ./test.sh
Hello world!!!
```

## 變數
寫程式當然要先知道怎麼做變數命名，不過要注意的是 shell script 的所有變數都是字串，沒有什麼複雜的型態問題
```bash
test_var=123
test_var="123"
test_var='123'
```

以上三個寫法是一樣的，要注意的是等號的左右不能有空格，使用變數的方式如下，加不加花括號都可以

```bash
echo "${test_var}"
echo "$test_var"
```

## 特殊變數
shell script 有一些預設的特別變數，用來幫助傳入參數進行判斷與處理

| 變數  | 描述                                    |
| :---: | :-------------------------------------- |
| `$0`  | 執行的 shell script 檔名                |
| `$n`  | n 從 1 開始，表示輸入的第幾個參數       |
| `$#`  | 表示總共傳入幾個參數                    |
| `$*`  | 表示傳入的所有參數，用空格隔開          |
| `$@`  | 表示傳入的所有參數，每個參數用 `"` 隔開 |
| `$?`  | 前一個指令的回傳狀態                    |
| `$$`  | 當前 shell script 的執行 PID            |

## 運算
前面說到 shell script 的變數都是字串，那要怎麼做數學運算呢，必須要透過一些特別的關鍵字才能做到

基本的像是 `expr`，要注意的是運算子的左右必須要空格，可以使用的運算子跟大部分的程式語言都大致相同

```bash
result = `expr 1 + 1`
echo ${result}
```

還有一種比較直覺的做法是用 `let`，這個寫法要注意不能有空白，不過寫起來較直覺

```bash
let a=1+1
let b=a+2
let a++
let a+=2
```

還有一種神奇的表示法 `$((...))` 或是 `$[...]`，可以在兩種表示法內寫入運算式就會轉成變數，而且有沒有空白都沒關係

```bash
result=$((1+1))
result=$((result+1))
((result+=10)) # 也可以直接在內做運算不加上 $ 號的話就不會轉成變數輸出，但結果一樣會留在 result，須注意用 [] 不能用此用法
((a=1+1,b=2+1,c=3+1)) # 多個運算式可以用 , 分開表示
```

雖然用括號的方式看起是最簡單的，但太多的括號個人覺得有點不好讀，所以自己比較喜歡的還是寫 `let`

## 流程控制
這部分個人覺得最麻煩的是，空白的部分都是不能夠省略的，只要有打錯的就會運作錯誤，這點要特別注意

### if then else
基本的 if 結構如下，有趣的是 if 的結尾是顛倒過來的 fi，這點蠻有趣的，然後判斷式是用 `[]` 來表示，如果要多條件判斷就需要多個 `[]`

```bash
if [ $a == $b ]; then
    echo "a == b"
elif [ $a != $b ]
    echo "a != b"
else
    echo "不可能發生"
fi
```

然後是邏輯運算子有點不一樣，不能直接用 `>` 跟 `<` 來表示，而是如下表

| 符號  | 意義     |
| :---: | :------- |
| `-gt` | 大於     |
| `-lt` | 小於     |
| `-ge` | 大於等於 |
| `-le` | 小於等於 |

示範一個多條件與比較運算子的用法

```bash
if [ $a -gt 0 ] && [ $a -lt 100 ]; then
    echo " 0 < a < 100 "
fi
```

### case
類似於 `switch case` 的感覺，用做多分支判斷
```bash
city='Kaohsiung'

case $language in
    Kaohsiung*) echo "高雄市"
    ;;
    Taipei*) echo "台北市"
    ;;
    Tainan*) echo "台南市"
    ;;
    *) echo "找不到"
esac
```

## 迴圈
迴圈也是在自動化工作中非常重要的一環，或者說絕大部分想要偷懶的時候都是要用到迴圈的時候XD

### for
shell script 的 for 有多種表示方法，也是有點混亂

```bash
for i in 1 2 3 4 5; do
    echo "number: $i"
done
```

上面寫法比較像是 foreach 的感覺，其中的 `1 2 3 4 5` 也可以用一個變數帶入

```bash
list="1 2 3 4 5"
for i in $list; do
    echo "number: $i"
done
```

或是有個比較直覺的寫法

```bash
for i in {1..5}; do
    echo "number: $i"
done
```

這個寫法也可以再變成，最後的 `..2` 代表 step，也就是每次加 2，所以會輸出 `1 3 5`

```bash
for i in {1..5..2}; do
    echo "number: $i"
done
```

還有一種基本上跟程式語言的寫法沒什麼不同

```bash
for ((i=1;i<=5;i++)); do
    echo "number: $i"
done
```

### while
while 也是一般程式語言常見的功能，寫法如下

```bash
let i=1
while [ $i -le 5 ]; do
    echo "number: $i"
    let i+=1
done
```

### until
until 比較特別一點，是一般程式語言沒有的功能，他其實就是 while 的另一種面向，表示直到條件成立前都執行

```bash
let i=1
until [ $i -gt 5 ]; do
    echo "number: $i"
    let i+=1
done
```

## function
隨著越來越複雜的自動化功能，有些 script 也可以被寫成函數並抽象化

```bash
function count(){
    for ((i=1;i<=5;i++)); do
        echo "number: $i"
    done
}

count # 使用時當作 command 來使用即可
```

呼叫 function 時就像使用 command 一樣，而傳遞參數的方式也相同直接帶在後面即可

```bash
function count(){
    for ((i=1;i<=$1;i++)); do
        echo "number: $i"
    done
}

count 5
```

而前面有提到一個特殊變數 `$?`，其實最常用到的地方就是 function 的回傳

```bash
function sum(){
    let sum=0
    for ((i=1;i<=$1;i++)); do
        let sum+=$i
    done
}

sum 5
echo "sum 1~5: $?"
```

---
## 結語
本篇是一個 shell script 的簡單筆記，開始接觸到 Linux 一定都會想使用 shell script 因為太多指令常常要下，實在很麻煩，但由於常常在寫的時候會忘記一些細節，導致常常要上網查，所以乾脆記錄一下