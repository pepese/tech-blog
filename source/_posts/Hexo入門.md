---
title: Hexo入門
date: 2017-05-02 17:05:58
tags:
---

はてなブログが一向に一向に *https* 対応しないし、そのほかのリスクも考えて流行りの **静的サイトジェネレータ** の [HEXO](https://hexo.io/) を使ってみる。  
前提として **npm** と **Git** 環境が必要。

[http://blog.pepese.com/entry/2017/02/16/141653:embed:cite]

# インストールから起動まで

```sh
$ npm install -g hexo-cli --no-optional // インストール
$ ndenv rehash
$ hexo -version                         // 確認
hexo: 3.3.5
hexo-cli: 1.0.2
os: Darwin 16.5.0 darwin x64
http_parser: 2.7.0
node: 7.6.0
v8: 5.5.372.40
uv: 1.11.0
zlib: 1.2.11
ares: 1.10.1-DEV
modules: 51
openssl: 1.0.2k
icu: 58.2
unicode: 9.0
cldr: 30.0.3
tz: 2016j
$ hexo init tech-blog                   // ブログプロジェクト作成
$ cd tech-blog
$ npm install --no-optional
$ hexo server                           // 起動
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

筆者がインストールした時（2017/05/02）は、 **--no-optional** オプション無しだと **Error: Cannot find module ./build/default/DTraceProviderBindings** というエラーがでた。

```sh
$ tree
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

## _config.yml

- https://hexo.io/docs/configuration.html

以下を設定する。

```yml
# Site
title: ぺーぺーSEのブログ
subtitle: 備忘録・メモ用サイト。
description: 備忘録・メモ用サイト。
author: ぺーぺーSE
language: ja
timezone: Japan

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://pepese.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# 〜（省略）〜
```

## テーマの設定

### テーマのインストール

**themes/** ディレクトリ配下にテーマを配置する。  
Githubで公開されているCasperを使用する場合は以下のように取得・配置する。

```sh
$ git clone https://github.com/kywk/hexo-theme-casper.git themes/casper

# USAGE
# git clone [Githubリポジトリ] themes/[テーマ名]
```

ブログの見た目を変更したいときは、 **themes/[テーマ名]/layout** 配下のファイルを編集する。  
テーマを反映させたいときは、 **_config.yml** の **theme** 項目にテーマ名を設定する。

## タイトル、メニューの設定

## 記事の作成

```sh
$ hexo new [layout] <title>
```

**layout** には **post** 、 **page** 、 **draft** の3種類あり、それぞれ以下のように別のパスに作成される。

|layout|パス|説明|
|:---|:---|:---|
|post|source/_posts|公開記事として作成される|
|page|source|アセット|
|draft|source/_drafts|非公開記事として作成される|

記事の削除は、 **rm source/_post/title.md** で直接削除する。  
ドラフトで作成していた記事は以下のコマンドで公開（つまりpostへ移動）されるj。

```sh
$ hexo publish [layout] <title>
```

## Github Pagesへの公開

以下のライブラリを導入する。

```sh
$ npm install hexo-deployer-git --save --no-optional
```

さらに **_config.yml** に以下の設定を追加する。

```yml
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://pepese@github.com/pepese/pepese.github.io.git
  branch: master
  message: update
```

なお、対応するGithubリポジトリは事前に作成しておくこと。  
以下のコマンドでGithubリポジトリに反映される。

```sh
$ hexo deploy
```

## 静的ファイルの作成

```sh
$ hexo generate
```

## 独自ドメインの設定

ドメインを取得したサービスで以下のように設定する。

<table>
<tr><th>サブドメイン</th><td>techblog</td></tr>
<tr><th>種別</th><td>CNAME</td></tr>
<tr><th>内容</th><td>github.io</td></tr>
</table>

さらに以下のようにCNAMEファイルを配置する。

```sh
$ echo 'techblog.pepese.com' > source/CNAME
```

お名前.comを利用している場合は[こちら](http://qiita.com/tiwu_official/items/d7fb6c493ed5eb9ee4fc)を参考。  
なお、この方法でやると **https** にはならない。

- http://dev.shikakun.com/post/hexo-init/

# 調べることめも

- sitemap
- タグ

# 参考

- [hexo に google adsense 設置&バナー編集](http://tofu.hatenadiary.com/entry/2017/01/14/025420)
