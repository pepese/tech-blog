---
title: Golangのnewとmake
tags:
- golang
- go
id: golang-new-make
---

golang の new と make についてまとめる。

- new
- make

<!-- more -->

# new

# make

構造体のスライスを作るときは new ではない。

```go
a := make(struct{}, 0)
```