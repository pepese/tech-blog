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

## 環境変数の設定

https://golang.org/doc/install/source#environment

- `GOROOT`
    - go のバイナリのホームまでのパス
	- `go env GOROOT`
- `GOPATH`
    - `go env GOPATH`
    - go のパスであってプロジェクトのパスでないことに注意
    - プロジェクトのパスは `$GOPATH/src/github.com/<あなたの名前>/<プロジェクト名>`
- `GOOS`
    - コンパイルして作成するバイナリの対象 OS を指定する
- `GOARCH`
    - コンパイルして作成するバイナリの対象 CPU を指定する

`.bash_profile` に以下を追記。

```
export GOROOT=`go env GOROOT`
export GOPATH=`go env GOPATH`
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

```bash
$ source .bash_profile
```

## 依存関係管理ツール dep

https://qiita.com/Azizjan/items/66564b5dc7597717932b

```bash
$ brew install dep
$ dep help
```

## ディレクトリ構造

```
$GOPATH
├─bin/
├─pkg/
│  └─darwin_amd64/
│    ├─github.com/
│    │  └─GitHubユーザ名
│    │    ├─`*.a`ファイル[^1]
│    │    └─GitHubレポジトリ名/`*.a`ファイル[^2]
│    └─pkg.in/
│      ├─パッケージ名/
│      └─`*.a`ファイル
└─src/
  ├─gopkg.in/
  │  └─パッケージ名/
  │    └─LICENSEとか`*.go`とかREADMEとか
  └─github.com/
    ├─GitHubユーザ名
    │  └─GitHubレポジトリ名/
    │    └─LICENSEとか`*.go`とかREADMEとか
    └─<あなたのGitHubユーザ名>
      └─GitHubレポジトリ名/ # プロジェクトディレクトリ（複数）
        ├─glide.yaml
        ├─main.go
        ├─その他、あなたが開発中のソフトウェアのコード
        └─vendor/依存先パッケージのコード(depでとってきたやつ)
```

## コマンドラインツール

- goimports
    - 過不足のimportの自動補完
	- `go get golang.org/x/tools/cmd/goimports`
- gocode
    - ヘルパー機能
	- `go get -u -v github.com/nsf/gocode`
- godef
    - 呼び出し関数へのジャンプなど
	- `go get -u -v github.com/rogpeppe/godef`
- gogetdoc
    - `go get -u -v github.com/zmb3/gogetdoc`
- golint
    - lint
	- `go get -u -v github.com/golang/lint/golint`
- go-outline
    - `go get -u -v github.com/lukehoban/go-outline`
- goreturns
    - `go get -u -v sourcegraph.com/sqs/goreturns`
- gorename
    - `go get -u -v golang.org/x/tools/cmd/gorename`
- gopkgs
    - `go get -u -v github.com/tpng/gopkgs`
- go-symbols
    - `go get -u -v github.com/newhook/go-symbols`
- guru
    - `go get -u -v golang.org/x/tools/cmd/guru`
- gotests
    - `go get -u -v github.com/cweill/gotests/...`

## プロジェクトの作成

```bash
$ mkdir -p $GOPATH/src/github.com/<あなたの名前>/<プロジェクト名>
$ cd $GOPATH/src/github.com/<あなたの名前>/<プロジェクト名>
```

初回は以下のような感じ。

```bash
$ mkdir -p $GOPATH/src/github.com/<あなたの名前>
$ cd $GOPATH/src/github.com/<あなたの名前>
$ git clone <golangプロジェクト> # プロジェクトディレクトリの作成
$ cd <golangプロジェクト>
$ echo "vendor/" > .gitignore
$ dep init # dep の初期化
$ touch app.go # 依存ライブラリ含め好きなコード書く
$ dep ensure -v # 依存ライブラリの解決
$ go run app.go
```

## デバッグ環境作成

- デバッガツール delve のインストール
    - `go get -u github.com/derekparker/delve/cmd/dlv`
- VSCodeにGo言語の拡張機能をインストール
    - `Rich Go language support for Visual Studio Code`

# A Tour of Go

https://go-tour-jp.appspot.com/welcome/1

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

## Variables

`var` は **変数宣言** 。

```go:variables.go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```

## Variables with initializers

初期化子が与えられている場合、型を省略できる。

```go:variables-with-initializers.go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```

## Short variable declarations

関数の中では、 `var` 宣言の代わりに、短い `:=` の代入文を使い、暗黙的な型宣言ができる。  
関数の外では、キーワードではじまる宣言( `var` , `func` など)が必要で、 `:=` での暗黙的な宣言は利用できない。

```go:short-variable-declarations
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}
```

## Basic types

- `bool`
- `string`
- `int` `int8` `int16` `int32` `int64`
- `uint` `uint8` `uint16` `uint32` `uint64` `uintptr`
- `byte`
    - `uint8` の別名
- `rune`
    - `int32` の別名
    - Unicode のコードポイントを表す
	- rune とは古代文字を表す言葉( runes )、 Go では文字そのものを表すためにruneという言葉を使う
- `float32` `float64`
- `complex64` `complex128`

```go:basic-types.go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

## Zero values

変数に初期値を与えずに宣言すると、ゼロ値( **zero value** )が与えられる。  
ゼロ値は型によって以下のように与えられる。

- 数値型(int,floatなど): 0
- bool型: false
- string型: "" (空文字列( empty string ))

```go:zero.go
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}
```

## Type conversions

従来の **キャスト** とほぼ同じ。

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

ただし、C言語とは異なり、 *Goでの型変換は明示的な変換が必要* 。

## Type inference

明示的な型を指定せずに変数を宣言する場合( `:=` や `var =` のいずれか)、変数の型は右側の変数から型推論される。

```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

## Constants

定数は、 `const` キーワードを使って変数と同じように宣言。  
定数は、文字(character)、文字列(string)、boolean、数値(numeric)のみで使える。  
なお、定数は `:=` を使って宣言できない。

## For

C言語 や Java、JavaScriptの for ループとは異なり、 for ステートメントの3つの部分を括る括弧 `( )` はない。なお、中括弧 `{ }` は必要。

```go:for.go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```

## For is Go's "while"

`for ` はセミコロン(;)を省略することもできる。  
つまり、C言語などにある `while` は、Goでは `for` だけを使う。

```go:for-is-gos-while.go
package main

import "fmt"

func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```

## If

Go 言語の `if` ステートメントは、 `for` ループと同様に、括弧 `( )` は不要で、中括弧 `{ }` は必要。

## If with a short statement

`if` ステートメントは、 `for` のように、条件の前に、評価するための簡単なステートメントを書くことができる。  
ここで宣言された変数は、 if のスコープ内だけで有効。

```go:if-with-a-short-statement.go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}
```

## If and else

`if` ステートメントで宣言された変数は、 `else` ブロック内でも使うことができる。

```go:if-and-else.go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// can't use v here, though
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}
```

## Switch

Go の `switch` は C や C++、Java、JavaScript、PHP の `switch` と似ているが、 Go では選択された `case` だけを実行してそれに続く全ての `case` は実行されない。 これらの言語の各 `case` の最後に必要な `break` ステートメントが Go では *自動的に提供される* 。 もう一つの重要な違いは Go の `switch` の `case` は定数である必要はなく、 関係する値は整数である必要はない。

```go:switch.go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.", os)
	}
}
```

## Defer

`defer` ステートメントは、 `defer` へ渡した関数の実行を、呼び出し元の関数の終わり( `return` する)まで遅延させる。  
`defer` へ渡した関数の引数は、すぐに評価されるが、その関数自体は呼び出し元の関数が `return` するまで実行されない。

```go:defer.go
package main

import "fmt"

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
// hello
// world
// と出力される
```

## Stacking defers

`defer` へ渡した関数が複数ある場合、その呼び出しはスタックされ、 LIFO の順番で実行される。

```go:defer-multi.go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}
```

[Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover)

## つづき

https://go-tour-jp.appspot.com/moretypes/1

# 参考

- https://www.slideshare.net/takuyaueda967/2016-go
    - slideshare
- http://blog.amedama.jp/entry/2015/10/06/231038
    - Go の文法ではなく、標準の構造・エコシステムがよくわかる
- http://blog.nishimu.land/entry/2015/03/16/032222
    - 文法がわかる
- おすすめさいとまとめ
    - https://mayonez.jp/topic/1362