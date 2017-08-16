---
title: Node.js/Expressでテストする
tags:
- Node.js
- npm
- Express.js
- JavaScript
id: express-test
---

Node.js/Expressアプリケーションのテストをしてみる。  
使用するツール・ライブラリは以下。

- テスティングフレームワーク
  - [mocha](https://mochajs.org/)
    - デフォルトでは `./test/*.js` 、 `./test/*.coffee` をテストスクリプトとして認識
    - `mocha.opts` というファイルにオプションを設定できる模様（[参考](https://mochajs.org/#mochaopts)）
- アサート
  - Chai
- モック
  - Sinon.JS
- カバレッジ
  - [Istanbul](https://istanbul.js.org/)
    - 公式の案内にもある通り、以下の `nyc` 経由で `istanbul` を利用する
  - [nyc](https://github.com/istanbuljs/nyc)
    - tap、mocha、AVA といったJSテスティングフレームワークと Istanbul をうまく連携させるコマンドラインツール
    - [mocha用のチュートリアル](https://istanbul.js.org/docs/tutorials/mocha/)

なお、タスクランナーは使用せず、npmスクリプトを使用する。  
以下の記事を読んだ前提で書く。

- [Express入門](https://pepese.github.io/blog/express-basics/)

<!-- more -->

# 環境設定

## グローバルインストール

```sh
$ yarn global add nyc
$ ndenv rehash
```

## ローカルインストール

[Express入門](https://pepese.github.io/blog/express-basics/)で作成したプロジェクトにて以下を導入する。

```sh
$ yarn add mocha should chai sinon mocha-sinon istanbul nyc --dev
```

## ディレクトリ作成

先に紹介した記事の通り、Expressアプリケーションのソースディレクトリは ```app/``` だけで完結するようにする。  
以下のようにテストスクリプト用のディレクトリを作成する。

```sh
$ mkdir app/spec
$ mkdir app/spec/controllers
```

# ユニットテストの実装

先に紹介した記事のスクリプトをテストする。

## packege.json

以下を加筆する。

```
{
  // 上記省略
  "nyc": {
    "check-coverage": true,
    "per-file": true,
    "lines": 90,
    "statements": 90,
    "functions": 90,
    "branches": 90,
    "include": [
      "app/"
    ],
    "exclude": [
      "app/spec/**/*.spec.js",
      "app/app.js",
      "app/controllers/router.js",
      "app/config",
      "app/coverage",
      "app/log",
      "app/public"
    ],
    "reporter": [
      "html",
      "text"
    ],
    "require": [],
    "extension": [
      ".js"
    ],
    "cache": true,
    "all": true,
    "report-dir": "app/coverage"
  },
  "scripts": {
    "test": "mocha app/spec/*.spec.js app/spec/**/*.spec.js"
  },
  // 下記省略
}
```

## テストスクリプト `app/spec/controllers/get_index.spec.js`

`app/controllers/get_index.js` を対象としたテストスクリプトを以下のように作成する。

<script src="http://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/spec/controllers/get_index.spec.js?footer=0"></script>

## テストスクリプト `app/spec/controllers/get_users.spec.js`

`app/controllers/get_users.js` を対象としたテストスクリプトを以下のように作成する。

<script src="http://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/spec/controllers/get_users.spec.js?footer=0"></script>

## テストの実行

npmスクリプトで以下のようにテストを実行する。

```sh
$ npm test

  /
    ✓ Index画面が1回レンダリングされること

  /users
    ✓ Users画面が1回レンダリングされること


  2 passing (8ms)
```

カバレッジレポートは `nyc` を用いて以下のように出力する。

```sh
$ nyc npm test

  /
    ✓ Index画面が1回レンダリングされること


  /users
    ✓ Users画面が1回レンダリングされること


  2 passing (10ms)

--------------|----------|----------|----------|----------|----------------|
File          |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
--------------|----------|----------|----------|----------|----------------|
All files     |      100 |      100 |      100 |      100 |                |
 get_index.js |      100 |      100 |      100 |      100 |                |
 get_users.js |      100 |      100 |      100 |      100 |                |
--------------|----------|----------|----------|----------|----------------|
```

`package.json` の `nyc` の設定の通りだが、HTML形式のレポートが `app/coverage` 配下に出力される。

# ライブラリ・ツールの概念を解説

## Sinon

- https://codezine.jp/article/detail/7729

- スパイ
  - 関数がどのように呼び出されたかを記録する
- スタブ
  - 関数の戻り値をあらかじめ設定し、その結果でテストを行う
- モック
  - 実行前に関数の実行回数など期待する結果を指定しておく
- フェイク
  - 問い合わせるDBやサーバ処理などを単純な実装に置き換える

### スパイ

スパイオブジェクトの作成方法は以下。

|呼び出し方法|概要|
|:---|:---|
|var spy = sinon.spy();|空のスパイオブジェクトを生成。主にコールバックで渡す無名関数などに対して使用する。|
|var spy = sinon.spy(myFunc);|特定の関数に対してスパイオブジェクトを作成する。|
|var spy = sinon.spy(object, "method");|オブジェクト内のメソッドに対してスパイオブジェクトを生成する。|

スパイオブジェクトの主な関数は以下。

|呼び出し方法|概要|
|:---|:---|
|spy.calledWith(arg1, arg2, ...);|指定した引数で関数が呼び出されたかを確認する。|
|spy.callCount|呼び出された回数を返す。|
|spy.called|呼び出された場合にtrueを返す。|
|spy.calledOn(obj);|指定したオブジェクトで関数が実行された場合trueを返す。|
|spy.calledOnce|1回呼び出された場合trueを返す。|
|spy.calledTwice|2回呼び出された場合trueを返す。|
|spy.exceptions|発生したexceptionを返す。|
|spy.returnValues|実行後の戻り値を返す。|
|spy.withArgs(arg1, arg2, ...);|指定した引数で関数が呼び出された場合にのみ、スパイオブジェクトを返す。|

### スタブ

スタブオブジェクトの作成方法は以下。

|呼び出し方法|概要|
|:---|:---|
|var stub = sinon.stub();|空のスタブオブジェクトを生成。主にコールバックで渡す無名関数などに対して使用する。|
|var stub = sinon.stub(object, "method");|特定の関数に対してスタブオブジェクトを作成する。|
|var stub = sinon.stub(object, "method", func);|特定の関数に対してスタブオブジェクトを作成し、指定した関数で処理を上書きします。|
|var stub = sinon.stub(obj);|対象オブジェクトのメソッドをすべてスタブにする。|

スタブオブジェクトの主な関数は以下。

|呼び出し方法|概要|
|:---|:---|
|stub.returns(obj);|呼び出し時のリターン値を指定する。|
|stub.throws(obj);|呼び出し時に指定したExceptionを発生させる。|
|stub.onCall(n);|n回目のスタブオブジェクトを返す。|
|stub.onFirstCall();|1回目のスタブオブジェクトを返す。|
|stub.onSecondCall();|2回目のスタブオブジェクトを返す。|

### モック


# 参考

- MochaとChai
  - http://qiita.com/y_hokkey/items/f73ea6b3d5f6902396b6
