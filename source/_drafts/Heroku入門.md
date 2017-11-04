---
title: Heroku入門
tags:
id:
---

# Heroku Client

## インストール

```sh
$ brew install heroku/brew/heroku
```

[Trouble Shooting](https://devcenter.heroku.com/articles/heroku-cli#troubleshooting)  
[Uninstalling](https://devcenter.heroku.com/articles/heroku-cli#uninstalling-the-heroku-cli)

## 初期起動

```sh
$ heroku login # Heroku にログイン
$ cd /path/to/myApp
$ heroku create # Heroku 上に新しいアプリケーションを作成
heroku create
Creating app... done, ⬢ shielded-scrubland-xxxxx
https://shielded-scrubland-xxxxx.herokuapp.com/ | https://git.heroku.com/shielded-scrubland-xxxxx.git
$ git init
$ git remote add heroku https://git.heroku.com/shielded-scrubland-xxxxx.git
$ git add --all
$ git commit -m "first commit"
$ git push heroku master # Heroku へデプロイ
$ heroku open # ブラウザを立ち上げてページを表示
$ heroku help # USAGE
$ heroku apps help # apps の USAGE
```

## アドオン

```sh
$ heroku addons:create xxxx
```

## プロセスの確認、停止

```sh
$ heroku apps:info # アプリケーションの情報を見る
$ heroku ps # プロセスを見る
$ heroku logs # ログを見る
```

```sh
$ heroku ps:scale web=0 # 停止
$ heroku ps:scale web=1 # 起動
```

Webプロセスのスケール（Dyno、インスタンス台数）を 0 に指定して実質プロセスを停止している。

# node.js

`$ heroku open` でエラーが発生した場合は以下に対応。

- npm script に **start** を追加する
    - `"start": "node app/app.js"`
