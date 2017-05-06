---
title: Hexoで作ったブログをGithub Pagesで公開
date: 2017-05-02 17:05:58
tags:
- Hexo
- Github Pages
id: hexo-github-pages
---

はてなブログが一向に一向に *https* 対応しないし、そのほかのリスクも考えて流行りの **静的サイトジェネレータ** の [HEXO](https://hexo.io/) を使ってブログを作成し、[GitHub Pages](https://pages.github.com/)で公開してみる。  
前提として **npm** と **Git** 環境が必要なため以下参照。

[http://blog.pepese.com/entry/2017/02/16/141653:embed:cite]

以降、[お名前.com]か何かでドメイン（ここでは **pepese.com** ）を取得した前提で書く。

# Hexoのインストールから起動まで

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
|   ├── _drafts  // これは無いかも
|   └── _posts
└── themes
```

## 設定ファイルの編集

**_config.yml** を以下のように設定する。

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
url: https://pepese.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# 〜（省略）〜
```

[設定ファイルの公式ドキュメント](https://hexo.io/docs/configuration.html)

## 記事の作成

```sh
$ hexo new [layout] <title>
```

**layout** には **post** 、 **page** 、 **draft** の3種類あり、それぞれ以下のように別のパスに作成される。

|layout|パス|説明|
|:---|:---|:---|
|post|source/_posts|公開記事として作成される|
|page|source|imageやjavascriptなどのアセット？|
|draft|source/_drafts|非公開記事として作成される|

上記のpost、page、draftは、 **scaffolds** 配下の **post.md** 、 **page.md** 、 **draft.md** が雛形となって生成される。  
この雛形で記事生成時のMarkdownのヘッダ部分（例えば以下）の初期値を設定することができる。

```
---
title: {{ title }}
date: {{ date }}
tags:
id:
---
```

「 **{{ title }}** 」へは記事のファイル名が、「 **{{ date }}** 」へは **source/_posts** 配下へ記事が作成された段階の日時が自動で設定される。  
「 **tags:** 」は以下のように記述することで記事へタグを付与することができ、タグ単位でリンクが作成される。

```
tags:
- Hexo
- Github Pages
```

「 **id:** 」は、 **_config.yml** へ以下（「 **:id** 」部分）のように設定することでURLとして扱うことができる。（デフォルトは記事名）

```
permalink: :year/:month/:day/:id/
```

記事の削除は、「 **rm source/_post/title.md** 」などのコマンドで直接削除する。  
ドラフトで作成していた記事は以下のコマンドで公開（つまりpostへ移動）される。

```sh
$ hexo publish [layout] <title>
```

### 静的ファイルの生成

```sh
$ hexo generate
```

上記のコマンドで「 **public/** 」配下に静的ファイル（HTTP/CSS/JS）が作成される。  
後述のブログのデプロイ時には、「public/」配下のファイルが公開されることになる。

## テーマの設定

### テーマのインストール

**themes/** ディレクトリ配下にテーマを配置する。  
Githubで公開されているCasperを使用する場合は以下のように取得・配置する。

```sh
$ git clone https://github.com/cgmartin/hexo-theme-bootstrap-blog.git themes/bootstrap-blog

# USAGE
# git clone [Githubリポジトリ] themes/[テーマ名]
```

ブログの見た目を変更したいときは、 **themes/[テーマ名]/layout** 配下のファイルを編集する。  
テーマを反映させたいときは、 **_config.yml** の **theme** 項目にテーマ名を設定する。

```yml
# 〜（省略）〜

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: bootstrap-blog

# 〜（省略）〜
```

### テーマの編集

**themes/[テーマ名]/layout** 配下を編集することでタイトル、サイドバーメニューの編集が可能。  
ここ、詳細はまだ。


# ブログの公開

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
**hexo deploy** コマンドでGithubリポジトリに反映される。

```sh
$ rm -r public/
$ hexo generate
$ hexo deploy # これがデプロイコマンド
```

「 **https://pepese.github.io** 」へアクセスするとブログがGithub Pagesとして公開されていることがわかる。


## HTTPS＋独自ドメインでブログの公開

ここでは独自ドメインをお名前.comで取得した「pepese.com」とする。    
なお、独自ドメインを使用するとGithubが用意してくれている証明書が使えないため **https** にはならない。  


お名前.comを利用している場合は[こちら](http://qiita.com/tiwu_official/items/d7fb6c493ed5eb9ee4fc)を参考。  
[Cloudflare](https://www.cloudflare.com/)を使用したサイトのHTTPS化を以下を参考に行う。  
（お名前.comの有料オプションを使ってHTTPS化も可能だが、ここでは無料でできるCloudflareを使用する。）

- https://rcmdnk.com/blog/2017/01/03/blog-github-web/
- https://www.kaitoy.xyz/2016/07/01/https-support-by-cloudflare/
- http://qiita.com/noraworld/items/89dd85a434a7b759e00c

手順は以下。

**(1)** [Cloudflare](https://www.cloudflare.com/)のアカウント作成

Sign upのリンクからメアドとパスワードを渡してアカウントを作成。

**(2)** Cloudflareにサイトを登録

「Add Your First Domain」にて **pepese.com** を入力して「Begin Scan」。

**(3)** CloudflareのDNSの設定

以下のようになっていればいい。

|Type|Name|Value|TTL|
|:---|:---|:---|:---|
|A|pepese.com|192.30.252.153|Automatic TTL|
|A|pepese.com|192.30.252.154|Automatic TTL|
|CNAME|techblog|pepese.github.io|Automatic TTL|

**(4)** プランの選択

無料のFree Websiteを選択。  
以上を完了すると「Please visit your registrar's dashboard to change your nameservers to the following.」と出て、レジストラのサイト（ここではお名前.com）のネームサーバを変更するように指示される。  
ここでは以下のように指示された。

|Current Nameservers|Change Nameservers to:|
|:---|:---|
|01.dnsv.jp|dina.ns.cloudflare.com|
|02.dnsv.jp|james.ns.cloudflare.com|
|03.dnsv.jp|Remove this nameserver|
|04.dnsv.jp|Remove this nameserver|

上記の指示通りにお名前.comの設定（ネームサーバの変更）を行う。  
この設定を行うことで、お名前.comのDNSサーバではなく、CloudflareのDNSサーバが使用されるようになる。

**(5)** CNAMEファイルの作成

以下のようにCNAMEファイルを配置して、Github Pagesを更新しておく。

```sh
$ echo 'techblog.pepese.com' > source/CNAME
```

**(6)** Hexoの設定ファイル（ **_config.yml** ）の編集

ドメイン変更に伴って設定を変更する。

```yml
# 〜（省略）〜

url: https://techblog.pepese.com # ここを変更
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# 〜（省略）〜
```

**(7)** その他の設定

好みに応じて以下の設定をする。

- CloudflareのPage Rulesタブで常にHTTPSアクセスとなるように以下を設定する。
  - If the URL matches: http://techblog.pepese.com*
  - Then the settings are: Always use HTTPS
- SpeedタブのAuto Minify項目でCloudflareでキャッシュする時にJavaScript/CSS/HTMLのソースをminifyする設定が可能。

以上で「 **https://techblog.pepese.com** 」へアクセスすると独自ドメインでブログが公開されていることがわかる。

# その他のHexoの設定

## sitemapの作成

```sh
$ npm install hexo-generator-sitemap --save --no-optional
```

**_config.yml** に以下の設定を追加する。

```yml
sitemap:
  path: sitemap.xml
```    

# 調べることメモ

- タグ
- Hexoで作ったブログにAdsenseを設定する
  - [hexo に google adsense 設置&バナー編集](http://tofu.hatenadiary.com/entry/2017/01/14/025420)
- Hexoで作ったブログにAnalyticsを設定する
