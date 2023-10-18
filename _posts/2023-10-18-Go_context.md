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
