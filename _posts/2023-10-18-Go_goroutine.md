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
