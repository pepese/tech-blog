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

# 参考

- https://www.slideshare.net/takuyaueda967/2016-go
    - slideshare
- http://blog.amedama.jp/entry/2015/10/06/231038
    - Go の文法ではなく、標準の構造・エコシステムがよくわかる
- http://blog.nishimu.land/entry/2015/03/16/032222
    - 文法がわかる
- おすすめさいとまとめ
    - https://mayonez.jp/topic/1362