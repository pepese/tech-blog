---
title: Node.js/Expressでテストする
tags:
- Node.js
- npm
- Express.js
- JavaScript
id: express-test
---

NodeJS、Expressアプリケーションのテストをしてみる。  
使用するライブラリは以下。

- テスティングフレームワーク
  - Mocha
- アサート
  - Chai
- モック
  - Sinon.JS
- カバレッジ
  - istanbul
- ルーティングのテスト
  - supertest

以下の記事を読んだ前提で書く。

- [Express入門](https://pepese.github.io/blog/express-basics/)

<!-- more -->

# インストール

## グローバル

```sh
$ npm install -g mocha chai mocha-sinon gulp
$ ndenv rehash
```

## ローカル

プロジェクトディレクトリで以下を実行。

```sh
$ npm install mocha should chai sinon supertest gulp-debug gulp-istanbul gulp-mocha@3.0.1 isparta --save-dev
```

```sh
$ yarn add mocha should chai sinon supertest gulp-debug gulp-istanbul gulp-mocha@3.0.1 isparta --dev
```

- **注意**
  - [*istanbul* は 2017/8/14 時点では *gulp-mocha* の4系には対応しておらず、 *3.0.1* にする必要がある](https://github.com/SBoudrias/gulp-istanbul)

## ディレクトリ作成

先に紹介した記事の通り、Expressアプリケーションのソースディレクトリは ```app/``` だけで完結するようにする。  
以下のようにテストスクリプト用のディレクトリを作成する。

```sh
$ mkdir app/spec
$ mkdir app/spec/controllers
```

# テストスクリプト作成

先に紹介した記事のスクリプトをテストする。

## gulpfile.js

<script src="http://gist-it.appspot.com/https://github.com/pepese/js-sample/blob/master/express-sample/gulpfile.js?footer=0"></script>

## モジュールのテスト

### app/spec/controllers/get_index.spec.js

<script src="http://gist-it.appspot.com/https://github.com/pepese/js-sample/blob/master/express-sample/app/spec/controllers/get_index.spec.js?footer=0"></script>

## ルーティングのテスト

**supertest** を使用してルーティングのテストを実装する。  
ただし、複雑なAPIや画面のテストはe2eテストでカバーするとして、ここでは簡易なルーティングのテストのみ。  
**なんか動かん。。。**

## 実行

```sh
$ gulp test
```

# ちょっと解説

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






# 以降ガンガン上記を無視してやり直し中

 タスクランナーは使わない。

- テスティングフレームワーク
  - Mocha
- アサート
  - Chai
- モック
  - Sinon.JS
- カバレッジ
  - istanbul
  - nyc
    - isparta ってやつの代わりっぽい。。。
- ルーティングのテスト
  - supertest

# インストール

## グローバル

グローバルいらん？

```sh
$ yarn global add mocha istanbul
$ ndenv rehash
```

## ローカル

```sh
$ yarn add mocha should chai sinon mocha-sinon supertest istanbul nyc --dev
```

## package.json

```
{
  "nyc": {
    "check-coverage": true,
    "include": [
      "app/**/*.js"
    ],
    "exclude": [
      "app/spec/**/*.spec.js",
      "app/config",
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
    "test": "nyc mocha app/spec/**/*.spec.js"
  },
}
```

テストの実行は `$ npm test` 。  
予約語に入らないスクリプトを作った場合の実行方法は `$ npm run-script test-cov` 。（定義が `test-cov` だとすると）

### Express テストの例

Expressのテストスクリプト（package.json）では以下のようになっている。

```
  "scripts": {
    "lint": "eslint .",
    "test": "mocha --require test/support/env --reporter spec --bail --check-leaks test/ test/acceptance/",
    "test-ci": "istanbul cover node_modules/mocha/bin/_mocha --report lcovonly -- --require test/support/env --reporter spec --check-leaks test/ test/acceptance/",
    "test-cov": "istanbul cover node_modules/mocha/bin/_mocha -- --require test/support/env --reporter dot --check-leaks test/ test/acceptance/",
    "test-tap": "mocha --require test/support/env --reporter tap --check-leaks test/ test/acceptance/"
  }
```

https://github.com/expressjs/express/blob/master/package.json

## メモ

- mocha（[公式](https://mochajs.org/)）
  - `--reporter` オプション
    - The --reporter option allows you to specify the reporter that will be used, defaulting to “spec”. This flag may also be used to utilize third-party reporters. For example if you npm install mocha-lcov-reporter you may then do --reporter mocha-lcov-reporter.
    - https://mochajs.org/#reporters
  - デフォルトテストディレクトリ
    - By default, mocha looks for the glob `./test/\*.js` and `./test/\*.coffee`, so you may want to put your tests in `test/` folder.
  - `mocha.opts` というファイルにオプションを設定できる模様
    - https://mochajs.org/#mochaopts
- istanbul（[公式](https://istanbul.js.org/)）
  - nyc
    - The nyc command-line-client for Istanbul works well with most JavaScript testing frameworks: tap, mocha, AVA, etc.
    - **重要！！** 設定方法は[ここ](https://github.com/istanbuljs/nyc)
  - [mocha用のチュートリアル](https://istanbul.js.org/docs/tutorials/mocha/)
