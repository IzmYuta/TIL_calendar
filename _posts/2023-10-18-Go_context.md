---
layout: post
title: "Go_context"
date: 2023-10-18
category: Go
excerpt: ""
---
# Context

## 1.基本
- ListenAndServeのリクエストごとにgoroutineを生成する
- 実践的に考えるとそれぞれのgoroutineがさらにgoroutineを生成する可能性がある(DB, 外部APIへのリクエスト)
- 意識せず多くのgoroutineを生成している可能性がある(goroutine leakのリスク)
- Aは処理を止めたいがBは続けたい
  - channelを使用すると管理が煩雑になる
  - -> contextを利用する
- **contextのcanselFancは必ず呼ぶこと**
	- channelを2回以上cancelするとpanicするがcontextは制御してくれている
 	- なので安心して呼び出そうね
channelとcontextの比較：
```go
// channel
func proc(ch chan struct{}) {
	ch1 := make(chan struct{})
	ch2 := make(chan struct{})
	ch3 := make(chan struct{})
	go proc2(ch1)
	go proc2(ch2)
	go proc2(ch3)

	select {
	case _, ok := <-ch:
		if !ok {
			// procが増えるとcloseも増える
			// もし3rd libsがこのcloseを忘れてたら？
			close(ch1)
			close(ch2)
			// 試しにコメントアウトしてみると？
			close(ch3)
			log.Println("done from proc")
		}
	}
}

// context
func proc3(ctx context.Context) {
	go proc4(ctx)
	go proc4(ctx)
	go proc4(ctx)
}
```

### 1.1 context cancel
- cancelを用いることでAPIリクエストやある処理のキャンセルができる
  - 例：
  - 10秒以上かかったらキャンセルする
  - メインプロセスが終了したためgoroutineを全て終了する
- context.WithCancelを用いることでcancelCtxとcancelFuncが返ってくる
- cancelFuncを呼ぶことでcontext内部のchannelをcloseする
- context.Done()経由でcancelされたかどうかを確認できる
- キャンセルされるとcontext.Err()に値が入る
  - キャンセルされていない時はnilになる

```go
package main

import (
	"context"
	"log"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background()) // ctx=cancelCtx, cancel=cancelFunc
	defer cancel()
	go watch(ctx)
	log.Println("execute ...")
	time.Sleep(time.Second * 2)
	cancel() // cancelFuncの呼び出し
}

func watch(ctx context.Context) {
	for {
		select {
		case <-ctx.Done(): // ここで確認
			log.Println(ctx.Err())
			return
		default:
			time.Sleep(time.Second * 3)
			log.Println("watch ...")
		}
	}
}

```

### 1.2 context timeout
- context.WithTimeoutを用いることでtimerCtxとcancelFuncが返ってくる
- cancelFuncを呼ぶか、ある秒数が過ぎることでcontext内部のchannelをcloseする
- context.Done()経由でcancelされたかどうかを確認できる

### 1.3 context deadline
- context.WithDeadlineを用いることでtimerCtxとcancelFuncが返ってくる
- cancelFuncを呼ぶ、ある時刻から指定秒が過ぎることでcontext内部のchannelをcloseする
- context.Done()経由でcancelされたかどうかを確認できる

### 1.4 値の伝搬
- contextは特定の値をkey/valueで伝播させることも可能
	- 例：
 	- 分散システムでTrace IDを伝播させたい
	- 認証トークン
	- ユーザーID
	- Query Cache
- keyにempty structを使うことでメモリを節約できる
  - stringは16byte、empty structは0byte
- context.Valueのkeyはprivateにする
  - context.Valueに同じkeyをsetすると上書きされてしまうため
  - privateにすることでsetとgetを関数経由でのみ行えるようにできる

```go
// string
ctx := context.WithValue(ctx, “key”, “value)
ctx.Value(“key”)

//empty struct
ctx := context.WithValue(ctx, key, “value)
ctx.Value(key)
```

### 1.5 contextの親子関係・兄弟関係
キャンセル：
- 親子：親から子へ伝播する。逆は伝搬ない。
- 兄弟：伝搬しない。

```go

	ctxtimeout, _ := context.WithTimeout(context.Background(), time.Second*3)
	// timeoutが短い方が優先される
	childctxtimeout, _ := context.WithTimeout(ctxtimeout, time.Second*2)
	child2ctxtimeout, _ := context.WithTimeout(ctxtimeout, time.Second*4)
	// ----  2.5秒のライン  ----
	time.Sleep(time.Millisecond * 2500)
	log.Println("-----")
	log.Println(childctxtimeout.Err()) // context deadline exceeded 2.5秒経過したので、タイムアウトが2秒のchildの方はキャンセルされる
	log.Println(ctxtimeout.Err())  // nil 親にキャンセルは伝搬しない。
	// ----  3.5秒のライン  ----
	time.Sleep(time.Millisecond * 1000)
	log.Println(child2ctxtimeout.Err()) // context deadline exceeded  タイムアウトが4秒のchild2はキャンセルされないはずだが、親がタイムアウトすると子に伝搬するためキャンセルされている。

	parent, cancel := context.WithCancel(context.Background())
	child, _ := context.WithCancel(parent)
	cancel()
	// 親から子へはcancelが伝播する
	log.Println("-----")
	log.Println(parent.Err()) // context canceled
	log.Println(child.Err()) // context canceled

	parent2, _ := context.WithCancel(context.Background())
	child2, cancel := context.WithCancel(parent)
	cancel()
	// 子から親へは伝播しない
	log.Println("-----")
	log.Println(parent2.Err()) // nil
	log.Println(child2.Err()) // context canceled
```

context.Value：
- 親子：子から親へ探索する。逆はしない。
- 兄弟：探索しない。

イメージ図：
<img width="1470" alt="スクリーンショット 2023-10-20 0 38 46" src="https://github.com/IzmYuta/TIL/assets/104307371/bfac3ab8-b526-42b6-9c28-84808c6da110">
(出典：https://speakerdeck.com/cyberagentdevelopers/ca-1day-youth-boot-camp-part-of-go)
```go
	ctx := context.WithValue(context.Background(), "key", 1)
	log.Println("-----")
	log.Println(ctx.Value("key")) // 1
	log.Println(ctx.Value("key2")) // nil　親から子へ探索できない

	log.Println("-----")
	ctx2 := context.WithValue(ctx, "key2", 2)
	log.Println(ctx2.Value("key")) // 1 子から親の探索はできる
	log.Println(ctx2.Value("key2")) // 2 

	log.Println("-----")
	ctx3 := context.WithValue(ctx2, "key3", 3)
	log.Println(ctx3.Value("key")) // 1 孫から親への探索もできる
	log.Println(ctx3.Value("key2")) // 2

	log.Println("-----")
	ctx4 := context.WithValue(ctx, "key4", 4)
	log.Println(ctx4.Value("key3")) // nil 兄弟関係では探索はできない
```

### 1.6 contextのアンチパターン
- contextを構造体に含めてはいけない
  - contextにセットする値はリクエストスコープに閉じる(=呼び出す時にだけ参照できる)ものにする
  - structに含めるとリクエストスコープを超えてしまうため
    - 起こりうる問題：
    - トークンの漏洩
    - 意図しない値の上書き
  - 状態を管理したいときは、contextを**ctxという名前で第一引数**に指定して渡すこと

### 1.7 アンチパターンの例外
- net/http
- database/sql
