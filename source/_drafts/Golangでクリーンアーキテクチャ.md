---
title: Golangでクリーンアーキテクチャ
tags:
- go
- golang
- クリーンアーキテクチャ
id: golang-clean-architecture
---

ググるといくつか記事が散見されるが、自分でも整理する。

- 階層定義
- 実装例

# 階層定義

<img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg" />

勝手ながら各層に名前をつけて整理する。

- 第 1 層：青色： **infrastructure** 層：外の世界とシステムを相互に繋ぐ層
	- [INPUT] 外からのアクセス
	    - **server** ディレクトリ： FW に依存したリクエスト受信・返却の実装
	        - view（HTMLとか）、api(JSONとかXML)
		- **batch** ディレクトリ：FW に依存したバッチ・ジョブの実装
	- [OUTPUT] 内（ interface ）からのアクセス
	    - **datastore** ディレクトリ：データベース製品に依存したアクセス、 repository の実装
		- **httpclient** ディレクトリ：他のマイクロサービスが外部サービスの API 呼び出し、 repository の実装
- 第 2 層：緑色： **interface** 層： infrastructure と usecase を相互に繋ぐ層
    - [INPUT] **controller** ディレクトリ：infrastructure からの呼び出しを usecase にマッピングする **handler** を実装する
	- [OUTPUT] **presenter** ディレクトリ：usecase からの 呼び出され、 infrastructure のdatastore や httpclient のインターフェースとなる **repository** をインターフェースとして定義する、ロジックの実装はない
- 第 3 層：赤色： **usecase** 層
    - INPUT/OUTPUT の分けはない
    - interface/controller より呼び出される
	- interface/presenter を経由して 1 つ以上のドメインモデルを操作・ドメインロジックを実行し、ビジネスロジックを成立させる
	- ビジネスロジックの結果を controller へ返却（ return ）する
- 第 4 層：黄色： **domain** 層
    - usecase 層によって扱われるドメインモデルとドメインロジック
    - ユースケースを実行するビジネスロジックは usecase 層に実装するが、ドメイン固有のルール・ロジックはここに実装する
	- **model** ディレクトリ
	    - ドメインモデルを入れる
- その他の層
    - **registry** ： DI とか
    - **cmd** ： cobra とかで作ったコマンド
    - **view** ： html template とか
    - **public** ： css とか js とか

例えば Hello Go は以下のようになる。（意図的に並び替えている。）

```bash
.
├── infrastructure   # 第 1 層
│   ├── server         # INPUT
│   │   └── httpserver.go
│   └── datastore      # OUTPUT
│       └── hello_datastore.go
├── interface        # 第 2 層
│   ├── controller     # INPUT
│   │   └── httphandler.go
│   └── presenter      # OUTPUT
│       └── hello_repository.go
├── usecase          # 第 3 層
│   └── hello_usecase.go
├── domain           # 第 4 層
│   ├── hello_logic.go # Logic
│   └── model          # Model
│       └── hello.go
└── main.go
```


- https://qiita.com/yoshinori_hisakawa/items/f934178d4bd476c8da32
- https://eng-blog.iij.ad.jp/archives/2442
