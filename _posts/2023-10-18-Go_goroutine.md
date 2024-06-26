---
layout: post
title: "Go_goroutine"
date: 2023-10-18
category: Go
excerpt: ""
---
# goroutine

## TL;DR
- 容易に並行処理を実行することができるようにGoが提供する軽量スレッド
- 排他制御や同期するための標準パッケージを活用する
- goroutine leakに注意する


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
  - 関数の前に`go`と宣言するだけで使うことができる


```go
go func() {
  // 処理を記述
}()
```

### 1.1 goscheduler
- P,M,Gの3つの要素がある
  - P：CPUのコアのようなもの
  - M：goroutineを実行する場所
  - G：goroutine
  - 画像ではPの数を1として実行
    - Pが1つであっても、複数個のMの中で複数個のGが実行されているので、M:Nスレッドモデルで動いていることがわかる
<img width="1011" alt="スクリーンショット 2023-10-18 14 15 16" src="https://github.com/IzmYuta/TIL/assets/104307371/e6200ff2-ef7e-400a-a313-c79cfc8b14bc">

イメージ：
<img width="1470" alt="スクリーンショット 2023-10-20 18 00 52" src="https://github.com/IzmYuta/TIL/assets/104307371/71524c96-ecc4-4488-9055-236c398e515c">
(出典：https://speakerdeck.com/cyberagentdevelopers/ca-1day-youth-boot-camp-part-of-go)

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
  - Mutexにはsync.Mutexとsync.RWMutexの2種類がある
    - sync.Mutex：シンプルなロックを提供
    - sync.RWMutex：共有ロックを提供(White処理はロック解除まで待つが、Read処理は許可する)
- プロダクトレベルでの使用：
  - インメモリキャッシュ
  - グローバルリソースへのアクセス
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


### 1.5 select
- channelの受け取りを多重化できる
- switchのように上から評価されず受け取れたものから処理する
- どれにも該当しない場合はdefaultへ
- defaultがない場合はブロックされる

```go
go func() {
  ch1 <- 1
}()
go func() {
  ch2 <- 2
}()
select {
  case v1, ok := <-ch1:
    //
  case v2, ok := <-ch2:
    //
  default:
}
```

### 1.6 semaphore
- 制御なしでgoroutineを生成すると当然パフォーマンスは下がる
- rate limitが存在すると制御しないといけない
- gorutineの実行数を制御する:
  - bufferありのchannelを使用
  - semaphore package`golang.org/x/sync/semaphore`を利用する

```go
// semaphoreなしの実装
func longProcess(ctx context.Context) {
	fmt.Println("Wait...")
	time.Sleep(1 * time.Second)
	fmt.Println("Done")
}

func main() {
	ctx := context.TODO()
	go longProcess(ctx)
	go longProcess(ctx)
	go longProcess(ctx)
	time.Sleep(5 * time.Second)
}


// semaphoreありの実装
var s *semaphore.Weighted = semaphore.NewWeighted(1)
func longProcess(ctx context.Context) {
	// Acquireで取得するとcountが減る
	// 0になるとブロックする
	if err := s.Acquire(ctx, 1); err != nil {
		fmt.Println(err)
		return
	}
	// カウントを戻す
	defer s.Release(1)
	fmt.Println("Wait...")
	time.Sleep(1 * time.Second)
	fmt.Println("Done")
}

func main() {
	ctx := context.TODO()
	go longProcess(ctx)
	go longProcess(ctx)
	go longProcess(ctx)
	time.Sleep(5 * time.Second)
}
```

nosemaphore(一気にまとめて処理が行われてしまう)：
<img width="1011" alt="AD0B74EB-2C5B-4994-9DE2-75942DE99238" src="https://github.com/IzmYuta/TIL/assets/104307371/38f853e0-d32d-461f-ab94-a771e8555ad6">
semaphore(実行数を制限して、1個ずつ処理できるようにしている)：
<img width="1011" alt="88B04F73-57F2-4832-9AE6-CDD92A0B9921" src="https://github.com/IzmYuta/TIL/assets/104307371/c79cb59c-a013-48ab-a285-595a62c0c9e0">

### 1.7 errgroup
- goroutineで扱う関数は戻り値を受け取ることができない
  - 内部でerror型を使いたくてもreturnでerrorを渡せない
  - error型のchannelを渡すことで対応はできるが、ハンドリングが大変
  - -> errgroupを使う
- errgroupでは、sync.WaitGroup + error型を返せるgoroutineを使える
  - `errgroup.Group.Go`:error型を返すgoroutineを生成
  - `errgroup.Group.Wait`:sync.Waitのように待ちながら生成したgoroutineでerrorがないかチェックできる
  - `errgroup.WithContext`：contextも扱うことができる
- 注意点：
  - errgroupが取得できるエラーは一番最初に発生したもののみ
    - contextによる処理のキャンセルが発生するため
  - 複数のエラーをまとめたい場合は`hashicorp/go-multierror`がある
    - 発生した全てのエラーを受けとることができる
    - errgroupと異なりcontextを用いた処理のキャンセルが存在しない点は注意

### 1.8 goroutine leak
- goroutineはユーザーから制御できない
  - プロセスとして残り続ける可能性がある
  - 待ちのプロセスは残り続けるだけだが参照されている変数などがGCされずOOM(メモリ不足)に
- 対処法：
  - 後述のcontextとselectなどでしっかりと抜けるようにする
  - ブロッキングする処理やデーモン、ループ処理は終了する条件を正しく書く

GC：
> GCとは不要になったメモリを解放する機能です。
> 
> アプリケーションは常にサーバやPCのメモリを確保しつつ何かしらの処理を行なっています。
>
> GCは確保したメモリのうち、不要なメモリを解放します。
>
> 不要かどうかの判定は、メモリを使用しているオブジェクトが何かしら（ルート集合あるいは他のオブジェクト）から参照されているかどうかで行っています。
>
> GCがなければアプリケーションが使用可能なメモリが枯渇してしまい、処理を継続できなくなってしまいます。
>
> C言語などではmalloc関数とfree関数を使ってプログラマがメモリ管理する必要がありましたが、GCの登場により自動的にメモリ管理が行われるようになりました。
>
> GCには様々なアルゴリズムが存在し、プログラミング言語ごとに採用しているアルゴリズムの種類が異なります。

(引用：https://qiita.com/gold-kou/items/4431d3dd41606d41732b)
