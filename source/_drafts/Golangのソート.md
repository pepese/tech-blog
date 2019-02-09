---
title: Golangのソート
tags:
- golang
- go
id: golang-sort
---

Golang におけるソートについてまとめる。

- sort パッケージ
- 独自のソートを作ってみる

<!-- more -->

# sort パッケージ

sort パッケージでは、以下の `Interface` に合わせた実装をしておけば、独自の type でも `sort.Sort(data Interface)` でソートしてくれる。

```go
type Interface interface {
	Len() int // 要素数
	Less(i, j int) bool // i 番目の要素が j 番目の要素より小さくなる条件
	Swap(i, j int) // i 番目の要素と j 番目の要素を入れ替えるロジック
}
```

とはいえ面倒なので、`IntSlice` （ `[]int` ）、`Float64Slice` （ `[]float64` ）、`StringSlice` （ `[]string` ）は用意されており、キャストして使用できる。

```go
var a []int
sort.Sort(sort.IntSlice(a))
```

キャストしたら、メンバに `Sort()` メソッドも実装されている。

```go
var a []int
sort.IntSlice(a).Sort()
```

もしくはキャストしなくとも、`Ints([]int)` 、 `Float64s([]float64)` 、 `Strings([]string)` が用意されている。

```go
var a []int
sort.Ints(a)
```

## Slice

less だけ実装すればよい、アレ。

- func Slice(slice interface{}, less func(i, j int) bool)
- func SliceStable(slice interface{}, less func(i, j int) bool)

# 独自のソートを作ってみる

## Interface の実装

```go
package main

import (
	"fmt"
	"sort"
)

type user struct {
	name string
	age  int
}

type users []user

func (u users) Len() int {
	return len(u)
}

func (u users) Less(i, j int) bool {
	return u[i].age < u[j].age
}

func (u users) Swap(i, j int) {
	u[i], u[j] = u[j], u[i]
}

func main() {
	list := []user{
		user{name: "A", age: 2},
		user{name: "B", age: 1},
		user{name: "C", age: 4},
		user{name: "D", age: 3},
	}
	us := users(list)
	sort.Sort(us)
	fmt.Println(us)
}
```

## Sliceの利用