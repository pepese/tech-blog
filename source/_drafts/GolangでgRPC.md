---
title: GolangでgRPC
tags:
- golang
- go
- gRPC
id: golang-grpc
---

[gRPC](https://grpc.io/) は、RPC (Remote Procedure Call) を実現するためにGoogleが開発したプロトコルの1つ。
[Protocol Buffers](https://developers.google.com/protocol-buffers/) を使ってデータをシリアライズ（異なるプログラミング言語間で XML や JSON といったデータフォーマットを介することなる透過的にデータをやり取り）し、HTTP/2 ベース高速な通信を実現できる。
プログラミング言語に依存しない IDL（インターフェース定義言語）を使ってあらかじめAPI仕様を `.proto` ファイルとして定義し、そこからサーバー側＆クライアント側に必要なソースコードのひな形を生成。

- 環境設定
- gRPC

# 環境設定

## gRPC のインストール

``` bash
$ go get -u google.golang.org/grpc
```

`$GOPATH/src/google.golang.org/grpc/examples` にサンプルあり。

## Protocol Buffers v3 のインストール

[ここ](https://github.com/protocolbuffers/protobuf/releases) から言語・環境に合わせた zip （ここでは `protoc-3.7.1-osx-x86_64.zip` ）をダウンロードし、 PATH の通ったディレクトリ（ここでは `$GOPATH/bin` ）に解凍したディレクトリの `/bin` の中のバイナリ（ `protoc` ）を移す。

## protoc plugin for Go のインストール

``` bash
$ go get -u github.com/golang/protobuf/protoc-gen-go
```

# gRPC

HTTP/2 の stream もサポートしており、gRPCのサポートするRPC方式は以下の通り。

- Unary RPC (1リクエスト1レスポンス)
- Server streaming RPC (１つのリクエストに複数レスポンス)
- Client streaming RPC (複数のリクエストに一つのレスポンス)
- Bidirectional streaming RPC(双方向)