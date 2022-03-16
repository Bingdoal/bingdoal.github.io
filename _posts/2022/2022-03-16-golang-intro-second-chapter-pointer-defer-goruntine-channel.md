---
layout: post
title: "Golang 初探簡介(下)"
date: 2022-03-16 10:14:31 +0800
category: backend
img: cover/golang.png
description: "丄篇簡單講了 Golang 的基本語法，以及一些簡單的範例 code，這篇要來看看 Golang 比較具有特色的一些語法，也是 Golang 強大功能的一些重點"
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

### defer
defer 的用意是延後執行，後面要接上一段 function 執行，可以是匿名 function，使用上我覺得有點像 Java 的 finally，一般來說會在最後才執行

```golang
func main() {
	defer fmt.Println("World!")
	fmt.Println("Hello!")
}
```

這個例子會發現 `World!` 是在 `Hello!` 之後才顯示出來

使用情境上就像 finally，常用在開檔關檔或是連線關閉時，也可以事先寫好需要結束後執行的項目，讓邏輯可以寫在一起

defer 還不限定只能寫一個，多個 defer 執行時會從最後一個開始執行，簡單說是放入到一個 stack 再取出來

```golang
func main() {
	defer fmt.Println("1")
	defer fmt.Println("2")
	defer fmt.Println("3")
	defer fmt.Println("4")
    // 4 3 2 1
}
```

下面這種情境，在宣告 defer 之後才去更改傳入的參數

```golang
func main() {
	a := 50
	defer fmt.Println(a)
	a = 100
    // 50
}
```

會發現最後的輸出還是 50，defer 在宣告的時候就固定了，可以想成 defer 是一個 function，基本型態的參數會被 call by value 的方式傳入

那如果說是在 defer 裡更改回傳會發生什麼事，這可以分成兩種情境

+ 情境一

```golang
func test() int {
	a := 50
	defer func() {
		a = 100
	}()
	return a
}

func main() {
	fmt.Println(test())
}
// 50
```

+ 情境二

```golang
func test() (a int) {
	a = 50
	defer func() {
		a = 100
	}()
	return a
}

func main() {
	fmt.Println(test())
}
// 100
```

情境一是比較符合理解的，特別的是情境二，如果說有指定回傳變數的話，那 defer 裡的變數更改也會被套用到回傳

### pointer

Golang 也提供如 C 語言中的 Pointer，可以自由操控變數的位址，用法也跟 C 語言大同小異

```golang
func main() {
	var a *int
	a = new(int)
	*a = 10
	fmt.Println("a = ", a) // 位址
	fmt.Println("*a = ", *a) // 10
}
```

以 `*Type` 來宣告 pointer 變數，pointer 只是宣告一個位址，並不會實際給予記憶體空間，所以必須經過 `new` 的手續才能賦予變數值

並且取值時需加上 `*`

```golang
func main() {
	var a int
	a = 10
	fmt.Println("&a = ", &a) // 位址
	fmt.Println("a = ", a) // 10
}
```

反過來在一般宣告變數的時候，會自動分配記憶體空間，因此可以直接賦值，需要取位值時只要加上 `&`

簡單舉一個 pointer 應用情境 Swap

```golang
func Swap(a *int, b *int) {
	temp := *a
	*a = *b
	*b = temp
}

func main() {
	var a, b = 10, 20
	fmt.Println(a, b)
	Swap(&a, &b)
	fmt.Println(a, b)
}
```

Golang 中大量用到傳 pointer 的方式來進行 function 運作，常會有 function 不直接回傳，而是改變傳入的 pointer 的情況，這部分跟 Java 的設計就大有不同，需要特別習慣一下

### goroutine

goroutine 像是 Golang 中的 thread 概念，是用來進行非同步操作的，不過實際上，goroutine 是由單執行緒完成的，由於 thread 實際是由 OS 提供的，每個 thread 都會去搶占 CPU 的資源，比較起來 goroutine 的成本要低得多

使用上也非常簡便，只要在 function 執行前面加上 `go` 就可以使用了

```golang
func main() {
	for i := 0; i < 10; i++ {
		go test(i)
	}
	time.Sleep(time.Second * 1)
}

func test(i int) {
	fmt.Println(i)
}
```

這邊看到在呼叫 `go test(i)` 之後還進行了 `time.Sleep` 的原因是，如果不讓主線程睡一下的話，由於 goroutine 是叫完就跑，會導致主線程馬上就結束了，主線程一旦結束，程式就會被跳出，那自然 goroutine 也會直接結束

這段程式跑完應該會發現不是從 0 跑到 9，順序會被打亂，且每次都長的不一樣，這就是 goroutine 並發的證據

### channel

channel 是一個跟 goroutine 常並用的東西，主要目的在於多線程之間的通訊，由於讓不同線程去共用記憶體是一件危險的事，容易發生 race condition 造成不可預期的錯誤，因此誕生了 channel 的用法

首先 channel 的寫法也是相當簡便的

```golang
func main() {
	ch := make(chan int)

	go func() {
		ch <- 1
	}()

	fmt.Println(<-ch)
    close(ch)
}
```
藉由 `<-` 來表示資料的讀入跟輸出，其實是非常直覺的，並且在用完之後記得關閉，減少資源的消耗

可以看到這邊沒有讓主線程睡，但是可以正常輸出資料，這是因為在讀出 channel 的時候，會進入等待，會等待 channel 有 input 才會通過 channel 的 output，否則就會卡住

接著試試看加入了 buffer 的 channel

```golang
func main() {
	ch := make(chan int, 3)
    defer close(ch)
	ch <- 1
	ch <- 2
	ch <- 3
	fmt.Println(<-ch) // 1
	fmt.Println(<-ch) // 2
	fmt.Println(<-ch) // 3
}
```

在宣告 channel 的時候，可以加入參數指定 channel 的長度，在塞滿之前可以暫存在 buffer 之中，因此可以在一個線程中使用，預設沒指定的話則視為 0，就如前一個例子，有一方輸入就必須有一方讀出，沒有辦法暫存

若是放入或是讀出超出 buffer 長度的內容，，channel 會發生 Block，並且無法解除 Block 的話就會發生 Deadlock 會強制跳出程式

從輸出也可以看到 channel 的 output 是 FIFO(First In First out)，所以其實也可以當作 queue 來使用

#### select

select 是一個 channel 的特殊用法，目的在於同時監聽不同的 channel，並進行不同的行為

```golang
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	defer close(ch1)
	defer close(ch2)
	go func() {
		for {
			select {
			case v := <-ch1:
				fmt.Println("ch1:", v)
			case v := <-ch2:
				fmt.Println("ch2:", v)
			}
		}
	}()

	for i := 1; i <= 10; i++ {
		if i%2 == 0 {
			ch1 <- i
		}
		if i%3 == 0 {
			ch2 <- i
		}
	}
}
```

輸出會像是下面這樣

```
ch1: 2
ch2: 3
ch1: 4
ch1: 6
ch2: 6
ch1: 8
ch2: 9
ch1: 10
```

可以看到只有在該 channel 有資料時，才會跑到對應的 case 去執行，若是都沒有的話則 select 預設其實也是 Block 住的，一直等到有 channel 被觸發才會進行下一步

用來作為觀察者模式的觸發應該是蠻不錯的

### 其他

Golang 本身其實也是主打跨環境開發與執行，若是今天想要在 windows 開發然後編譯成執行檔在 Linux 或是 Mac 執行該怎麼作呢

首先 Golang 在執行 `go build` 的時候，其實有參考兩個環境變數 `GOOS` 跟 `GOARCH`，可以先用 `go env` 來看自己主機的環境變數長什麼樣子

```bash
$ go env
...
set GOARCH=amd64
...
set GOOS=windows
```

我以 windows64 位元的電腦輸出是這兩個值，那如果今天我要 build 到其他環境的話其實就是改這兩個變數就好

```bash
SET GOOS=darwin&SET GOARCH=arm64&go build . # MAC
SET GOOS=linux&SET GOARCH=arm64&go build . # Linux
```

不過 `GOARCH` 應該是會跟著機器 CPU 有所不同而要更改，這就等到遇到再說了，各自的詳細值則可以參考[這邊](https://gist.github.com/asukakenji/f15ba7e588ac42795f421b48b8aede63)

---
## 結語

Golang 的 goroutine 以及 channel 實在是非常好用的兩個特性，在處理非同步的情況時調用簡便，且效率非常好，defer 的運用也是十分廣泛，感覺得出來都是經過很好的設計才誕生的用法，期望之後 Golang 的發展也能越來越方便並且維持它的高效能
