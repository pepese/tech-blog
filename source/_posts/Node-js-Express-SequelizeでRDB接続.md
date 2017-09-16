---
title: Node.js/Express/SequelizeでRDB接続
date: 2017-08-20 15:30:40
tags:
- Node.js
- yarn
- Express.js
- JavaScript
- Sequelize
id: express-rdb-sequelize
---

Node.js/ExpressアプリケーションからRDBへ接続してみる。  
使用するツール・ライブラリは以下。

- O/R Mapper
    - [Sequelize](http://docs.sequelizejs.com/)
- RDB
    - [SQLite3](https://sqlite.org/index.html)

以下の記事を読んだ前提で書く。

- [Express入門](https://pepese.github.io/blog/express-basics/)

<!-- more -->

# 環境設定

先の記事で紹介したプロジェクトにて以下を実行する。

```sh
$ yarn add sequelize sqlite3 morgan
$ mkdir app/db
```

`.gitignore` へ以下を追加。

```
node_modules
app/db/*.sqlite
.DS_Store
./**/.DS_Store
```

# 実装

- `app/repositories/sequelize.js`
    - Sequelizeを使ったRDB接続用モジュール
- `app/repositories/models.js`
    - Clientモデルを作成
    - テーブルの作成とサンプルデータのインサート
- `app/controllers/get_client`
    - 上記の `models.js` 経由でClientの検索


## `app/repositories/sequelize.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/repositories/sequelize.js?footer=0"></script>

## `app/repositories/models.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/repositories/models.js?footer=0"></script>

## `app/controllers/get_clients.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_clients.js?footer=0"></script>

# 実行

以下で起動。

```sh
$ node app/app.js
```

`http://localhost:3000` にブラウザでアクセスして Client 情報が見れたら成功。
