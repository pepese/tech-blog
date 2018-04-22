---
title: Golang入門
tags:
- go
id: golang-basics
---

# 環境構築

```bash
$ brew install go
```

`.bash_profile` に以下を追記。

```
export GOROOT=`go env GOROOT`
export GOPATH=`go env GOPATH`
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

```bash
$ source .bash_profile
```

# 環境変数

https://golang.org/doc/install/source#environment

- `GOPATH`
    - プロジェクトのパスを指定する
    - どこでもいい
- `GOOS`
    - コンパイルして作成するバイナリの対象 OS を指定する
- `GOARCH`
    - コンパイルして作成するバイナリの対象 CPU を指定する

# 依存関係管理ツールdep

https://qiita.com/Azizjan/items/66564b5dc7597717932b

```bash
$ brew install dep
$ dep help
```

# A Tour of Go

## hello.go

```bash
$ mkdir $GOPATH/src/github.com/tour
$ cd $GOPATH/src/github.com/tour
$ touch hello.go
$ vi hello.go
```

```go:hello.go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

```bash
$ go run hello.go
Hello, 世界
$ gofmt hello.go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

## packages.go

```go:packages.go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
}
```

## import.go

```go:import.go
package main

// import (
// 	"fmt"
// 	"math"
// )
import "fmt"
import "math"

func main() {
	fmt.Printf("Now you have %g problems.", math.Sqrt(7))
}
```

## exported-names.go

Go では、最初の文字が大文字で始まる名前は、外部のパッケージから参照できる公開された名前( *exported name* )。  
例えば、 `Pi` は `math` パッケージでエクスポートされている。  
`pi` （小文字）ではないことに注意。

```go:exported-name.go
package main

import (
	"fmt"
	"math"
)

func main() {
	//fmt.Println(math.pi)  // Error になる
    fmt.Println(math.Pi)
}
```

## functions.go

```go:functions.go
package main

import "fmt"

// func add(x , y int) int { // 省略可能
func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}
```

## multiple-results.go

```go:multiple-results.go
package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

## named-results.go

返り値となる変数に名前をつけることができる。

```go:named-results.go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```


# 参考

- https://www.slideshare.net/takuyaueda967/2016-go
    - slideshare
- http://blog.amedama.jp/entry/2015/10/06/231038
    - Go の文法ではなく、標準の構造・エコシステムがよくわかる
- http://blog.nishimu.land/entry/2015/03/16/032222
    - 文法がわかる
- おすすめさいとまとめ
    - https://mayonez.jp/topic/1362