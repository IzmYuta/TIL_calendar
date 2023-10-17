---
layout: post
title: "Go_interface"
date: 2023-10-18
category: Go
excerpt: ""
---
# Interface型

## 1.基本
interfaceは2つの使い方がある。
1. 振る舞いを抽象化する
2. あらゆる型(interface以外)を代入することができる

### 1.1 振る舞いを抽象化する
- Pythonでいうクラスのようなイメージ
  - 型にメソッドを実装できるというGo言語の性質を利用している

例：(https://go.dev/play/p/T77psnc50IN)
```go
type Hello interface {
	Hello()
}

type hello struct{}

func (h *hello) Hello() {
	fmt.Println("Hello from Go")
}

func main() {
	var h hello
	h.Hello()
}
```

### 1.2 あらゆる型を代入できる
- 型に応じて動的に動作を変えることができる
