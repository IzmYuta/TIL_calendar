---
layout: post
title: "Go_goroutine"
date: 2023-10-18
category: Go
excerpt: ""
---
# goroutine

## 0. 並行処理と並列処理
- 並行処理：一定期間の間に複数のプロセスを処理すること(同時であるかは気にしない)
- 並列処理：ある時点で**同時に**複数のプロセスを処理すること
- 並行性はコードの性質を指し、並列性は動作しているプログラムの性質を指します。
  - コード自体は並行性を表す。実行環境(CPUのコア数など)が並列性を作り出す。

<img width="1470" alt="スクリーンショット 2023-10-18 12 50 33" src="https://github.com/IzmYuta/TIL/assets/104307371/d2c5ca0b-8829-4de7-b777-1301ac2f8c3c">
(出典：https://speakerdeck.com/cyberagentdevelopers/ca-1day-youth-boot-camp-part-of-go)

## 1.基本
- goroutineとは、簡単に**並行処理**が実装できる軽量スレッド
- ブロッキング(IO待ちなど)すると自動で別のgoroutineが他で実行可能なスレッド(Go runtime上のスレッド)で実行される
- goroutineはGoプログラムの最小実行単位であり、entry pointのfunc mainはruntime/proc.go にあるfunc mainでコールされており、proc.goのfunc mainはmain goroutineと呼ばれる
  - なのでGoを実行するときは必ずgoroutineを使っている
- goroutineはコルーチン(coroutine)と似ている
  - コルーチンとの違い：goroutineは中断・再開の制御ができない。実行順の制御もできない。Go runtimeがスケジューリングしてやってくれる。
  - -> 制限が多い分、簡単に並行処理を扱うことができる

### 1.1 goscheduler
- P,M,Gの3つの要素がある
  - P：CPUのコアのようなもの
  - M：goroutineを実行する場所
  - G：goroutine
  - 画像ではPの数を1として実行
    - Pが1つであっても、複数個のMの中で複数個のGが実行されているので、M:Nスレッドモデルで動いていることがわかる
<img width="1011" alt="スクリーンショット 2023-10-18 14 15 16" src="https://github.com/IzmYuta/TIL/assets/104307371/e6200ff2-ef7e-400a-a313-c79cfc8b14bc">


### 1.2 Go channel
- Go channelとは並行に実行している関数同士が特定の方の値を送受信するためのもの
- channel<- や <-channelという構文でデータのコミュニケーションをとる
- 2回closeするとpanicを起こすので注意
  - 呼び出し側でcloseするように心がけること
<img width="1011" alt="BC435791-E470-4381-8188-7265CF9DB011" src="https://github.com/IzmYuta/TIL/assets/104307371/8e1ecf6f-891e-433d-be93-1935dca6acb9">


- Deadlockには注意すること
  - コンパイル時には判明せず、ランタイム実行しないとわからない

<img width="1011" alt="AE13BFD0-B09F-4C3E-9C5D-677B5A028D79" src="https://github.com/IzmYuta/TIL/assets/104307371/28fb4475-88ba-4689-8482-05947cb76edc">

### 1.3 WaitGroup
- 全てのルーチンが終了するまで待機したい時に使う
- Add: 終了待ちのgoroutineの数だけ内部のstateをインクリメントする
- Done: 終了したgoroutineが呼び内部のstateをデクリメントする
- Wait: 内部のstateが0になるまでブロックする

失敗するコード：
```go
package main

import (
	"log"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		i := i
		wg.Add(1)
		go func(wg sync.WaitGroup) {
			defer wg.Done()
			log.Println(i)
		}(wg)
	}
	wg.Wait()
	log.Println("end")
}
```
- `go func(wg sync.WaitGroup) {`に原因あり
  - 値渡しをするとコピーが作成されてしまう
  - 最初に定義したwgとは別のwgに対してデクリメントが行われてしまうのでDeadlockとなる
  - -> 関数にWaitGroupを渡すときはポインタで渡そう！
- 原因特定にはgo vetなどのlinterを使うと便利

<img width="1011" alt="スクリーンショット 2023-10-18 16 53 52" src="https://github.com/IzmYuta/TIL/assets/104307371/02db13c2-d3cd-48f9-b601-a77303029184">

修正後：
```go
package main

import (
	"log"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		i := i
		wg.Add(1)
                // ポインタを渡す
		go func(wg *sync.WaitGroup) {
			defer wg.Done()
			log.Println(i)
		}(&wg) // ここも忘れずに
	}
	wg.Wait()
	log.Println("end")
}
```

### 1.4 Mutex
- 次のコードの実行結果は？
  - 1000ではない
  - こうなるのは、複数のgoroutineがどの時点の数字をインクリメントするのかは、タイミングによるためである
  - -> 結果が一定でない
  - Mutexを使うことでこのようなData Race(データ競合)に対して、排他的処理を行うことができる

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c := 0
	for i := 0; i < 1000; i++ {
		go func() {
			c++
		}()
	}
	time.Sleep(time.Second)
	fmt.Println(c)
}
```
