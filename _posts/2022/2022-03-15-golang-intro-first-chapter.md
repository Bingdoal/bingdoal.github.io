---
layout: post
title: "Golang 初探簡介(上)"
date: 2022-03-15 09:48:51 +0800
category: backend
img: cover/golang.png
description: "最近工作上使用 Golang 開發有一段時間了，在幾個周邊的小服務上慢慢使用 Golang 來取代 Java，對於 Golang 在小服務撰寫上的體驗是十分好的，可以達到快速開發、快速上線的效果，編譯後的執行檔也較輕便，執行效率也很好，目前來說比較可以挑剔的大概就是生態系還不如 Java 這樣完整，有一些功能還是需要自己打造，開發的概念上也跟 Java 有些不同，需要額外習慣一下，不過在各方面都可以知道這是個十分有潛力的語言"
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

先簡單介紹下 Golang 的特性，應該會分成上下兩篇，上篇簡介各種基本語法，下篇再來看到 Golang 獨有的寫法

## Golang

Go 是 Google 開發的一種編譯式語言，語法上接近 C 語言，並且支援垃圾回收。

當時 Google 設計 Go 目的在於提高程式的執行效率，並取其他程式語言的優點，例如:
+ 靜態型別和執行時效率（如：C++）
+ 可讀性和易用性（如：Python 和 JavaScript）
+ 高效能的網路和多行程

Golang 的可讀性一個原因在於撰寫風格的統一，如:
1. 每行程式結束後不需要撰寫分號
2. `{` 不能夠換行放置
3. if 判斷式和 for 迴圈不需要以小括號包覆起來
4. 使用 tab 做排版

第二點不符合時會在編譯時直接擋掉，其餘可以透過內建的 `gofmt` 自動整理程式碼，另外 Golang 的關鍵字極少，總共只有 25 個。

Golang 的執行效率以及編譯後的檔案大小都是目前頂尖的存在，因此有許多大廠都紛紛採用，只不過 Golang 的發展尚未十分成熟，有許多工具還沒有很良好的實作，有時需要自己多造點輪子

#### 缺點
目前 Golang 的發展最為人詬病的應該是:
1. 不支援泛型
2. 沒有好的異常處理手段

不支援泛型讓一些抽象化的寫法較難實現，讓模組化使用變得困難

而 Golang 目前的異常處理則是很陽春的 `if err!= nil`，這樣的寫法會讓程式碼比較難被閱讀，也難以修改，並且不好統一處理異常

### Hello world
環境建置只要到 [Golang 官網](https://go.dev/) 下載對應的作業系統安裝解包好就可以用了，一開始不免俗要來 Hello 一下

```golang
// main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, world!")
}
```

看到基本程式執行的進入點 `func main()` 除此之外還必須加上 `package main`，這才會被視為是主要的執行模組

執行時只要下指令 `go run main.go` 就可以看到 `Hello, world!` 的訊息

#### build
如果說今天要部屬到其他環境下，只需要執行檔的情況，可以下指令:

```bash
go build main.go
```

會得到一個 `main.exe` 這時候就可以直接執行 `main.exe` 了

### 基本型態

|               型態                | 描述                                                 |
| :-------------------------------: | :--------------------------------------------------- |
|              `bool`               | 布林                                                 |
|   `int/int8/int16/int32/int64`    | 有號整數，如果只有 int 會自動判斷需要 int32 或 int64 |
| `uint/uint8/uint16/uint32/uint64` | 無號整數                                             |
|         `float32/float64`         | 浮點數                                               |
|      `complex64/complex128`       | 複數 `1+2i`，應該是用不太到                          |
|             `string`              | 字串                                                 |
|              `byte`               | 位元組                                               |
|              `rune`               | `rune` 是個特別的型態，會把字串用 unicode 的方式呈現 |
|      `[]Type`，被稱作 slice       | 類似 array 與 Linked list 的存在，有特別多的奇技淫巧 |
|          `map[Type]Type`          | 就是 Map，Key & Value 的對應表                       |
|            `interface`            | 代表的是泛用型，以 Java 舉例就是 Object              |

### 變數宣告
Golang 的變數宣告，有兩種方式可以進行，彼此還有一些不同的差異

```golang
var a int
var a int = 16
a := 16
a, b := 10, 16
```

變數宣告可以透過上面的兩種方式，不過之中有些許差異

像是當宣告全域變數時只能用 `var`，並且用 `var` 跟 `:=` 宣告的變數會被視為不同的變數

+ 常數宣告

```golang
const YEAR = 2022
const HELLO = "Hello"
```

### 流程控制

#### if

```golang
if Condition {
	...
} else {
    ...
}
```

#### switch case

````golang
var i int = 2
switch i {
case 1:
	fmt.Println("i is 1")
case 2:
	fmt.Println("i is 2")
case 3:
	fmt.Println("i is 3")
default:
	fmt.Println("i is not 1, 2, 3")
}
````

interface 的特殊用法

````golang
var i interface{}
switch i.(type) {
case int:
	fmt.Println("i is int")
case string:
	fmt.Println("i is string")
case bool:
	fmt.Println("i is bool")
default:
	fmt.Println("i is not int, string, bool")
}
````

特別注意到 Golang 不需要特別寫 break

#### loop

+ for

```golang
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

+ while

Go 並沒有 while 只能用 for 的方式實現

```golang
i := 0
for i < 10 {
    fmt.Println(i)
    i++
}

```

+ foreach

```golang
slice := []int{1, 2, 3, 4, 5}
for index, num := range slice {
	fmt.Println(index, num)
}

mapDemo := map[string]string{"a": "apple", "b": "banana"}
for key, value := range mapDemo {
	fmt.Println(key, value)
}
```


### function
function 的使用跟其他程式語言也沒太大差別，上面的 `func main()` 就是一種 function

```golang
func hello(){
    fmt.Println("Hello World")
}

func main(){
    hello()
}
```

Golang 的 function 有點像是 javascript 可以被 assign 給變數，也可以使用匿名函式

```golang
func main() {
	func() {
		fmt.Println("Hello World")
	}()
}

func main() {
	var test = func() {
		fmt.Println("Hello World")
	}
	test()
}
```

函式傳參方式有幾種如下

```golang
func test1() string {
	return fmt.Sprintln("Hello World")
}

func test2() (a string) {
	a = fmt.Sprintln("Hello World") // 注意這裡的 a 不需要經過 var 或是 :- 來新建變數
	return
}

func test3(a int, b int) int {
	return a + b
}

func test4(a int, b int) (int, int) {
	return a, b
}

func test5(a func()) {
	a()
}
```

傳入的部分跟一般程式語言差不多，並且能夠傳入 function，做到 callback 的功能

但回傳有幾種比較特別的，像是能夠回傳多參數，以及能在回傳直接宣告變數名稱

### struct
簡單說就是自定義一個資料結構，這個資料結構可以包含其他資料結構，也可以包含基本型態

```golang
type Person struct {
    Name string
    Age int
}

func main(){
    var p Person
    p.Name = "John"
    p.Age = 20
    fmt.Println(p)
}
```

#### function in struct

寫過物件導向程式語言一定會希望能夠在物件內部使用 function，這樣的 function 可以被稱為 method

寫法如下

```golang
func (person *Person) doIntro() {
    fmt.Printf("Hello, my name is %s", person.Name)
}
```

執行上如下

```golang
func main(){
    var p Person
    p.Name = "John"
    p.Age = 20
    p.doIntro()
}
```

### Go module & package

今天當我們要開始一個專案開發的時候總不可能都寫在一份 main 裡面，那要怎麼進行程式的引用呢

首先我們要先執行指令

```bash
go mod init go-demo
```

指令執行過後會產生一個 `go.mod` 的檔案，裏頭會紀錄 module 名稱，以及這個 module 有用到的其他 module，就像是 node.js 的 package.json，或是 java maven 的 pom.xml

這樣才能把專案納入到 golang 的 module 當中，也才能夠引用，不然預設只會去抓 `GOPATH` 底下的 module

接著按照下面結構擺放檔案

```
├── pkg
│   └─── test.go
├── go.mod
└── main.go
```

程式碼如下

```golang
// test.go
package pkg

import "fmt"

func init(){
    fmt.Println("test.go init.")
}

func Test() {
	fmt.Println("Hello, World!")
}

// main.go
package main

import "go-demo/pkg"

func main() {
	pkg.Test()
}
```

必須將專案納入到 module 中，才能夠引用，並且 package 必須跟目錄名稱一樣，package 在被引用時，會執行 `init()` 進行初始化，並且整個執行週期只會被初始化一次

+ golang 中的變數包含 function，只要開頭是大寫就是 public，小寫就是 private

### 其他

補充一個小知識，常會看到 Go 的圖片中有一隻下面的東西，這隻東西是地鼠，叫做 Gopher，是 Golang 的吉祥物，是不是很可愛啊

![]({{site.baseurl}}/assets/img/gopher.jpg)

---
## 結語

把 Golang 的基本語法跑過一遍，會發現內容真的不多，因為 Golang 當初設計的初衷就是希望能夠簡化各種繁瑣的操作以及關鍵字，因此可以用最單純的語法去組合需要的邏輯，下篇再來了解一下 Golang 的強大併發機制以及指標操作