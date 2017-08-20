---
title: Node.js/Express/requestでHTTP(S)リクエストを発行する
tags:
- Node.js
- yarn
- Express.js
- JavaScript
- request
id: express-http-client-request
---

Node.js/ExpressアプリケーションからHTTP(S)を発行してみる。  
使用するツール・ライブラリは以下。

- HTTP(S)クライアント
    - [request](https://github.com/request/request)

以下の記事を読んだ前提で書く。

- [Express入門](https://pepese.github.io/blog/express-basics/)

<!-- more -->

# 環境設定

先の記事で紹介したプロジェクトにて以下を実行する。

```sh
$ yarn add request
```

# 実装

- ``

# request の基本的な使い方

- `request(options, callback)`
    - コールバックで `options` に基づいたリクエストを発行する
    - https://github.com/request/request#requestoptions-callback
- `request.defaults(options)`
    - HTTP(S)クライアントのデフォルトの設定を行う
    - https://github.com/request/request#requestdefaultsoptions
- `request.METHOD()`
    - https://github.com/request/request#requestmethod

## `options` の設定

- https://github.com/request/request#requestoptions-callback
