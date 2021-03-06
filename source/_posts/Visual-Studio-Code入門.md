---
title: Visual Studio Code入門
date: 2018-01-31 18:15:32
tags:
- VS Code
id: vscode-basics
---

<!--移行済み-->
Microsoft が開発したエディタ[Visual Studio Code](https://code.visualstudio.com/)（以降、 VS Code ）のインストールから拡張機能の導入までをまとめる。

<!-- more -->

# インストール（Mac）

```sh
$ brew cask install visual-studio-code
```

# 起動方法

アイコンクリックでもいいがコマンドラインで起動できる。

```sh
$ code
```

プロジェクト（カレントディレクトリ）で起動したい場合は以下。

```sh
$ code .
```

# ショートカット・操作

ショートカットの一覧・設定は `Comannd + K -> Command + S` で開く。  
（筆者の場合は「右側をすべて削除」が `Ctrl + K` になっていなかったので登録した。）

- ユーザ設定
    - `Command + ,`
- Markdown Preview
    - `Command + Shift + V`
- 統合ターミナル
    - `Control + Shift + @`
- ワークスペースにプロジェクトを追加
    - 「ファイル」->「ワークスペースにフォルダに追加」

# コマンドパレット

`Command + Shift + P` を押すと VS Code の上部に **コマンドパレット** が開く。  
VS Code で実行できる各種コマンドには名前が付いていて、その名前をこのコマンドパレットに入力することでそれらを実行できる。

# 拡張機能（ Extensions ）

アクティビティーバー（左上に縦に並んでるアイコン）の一番下にあるアイコンをクリックするとサイドバーが拡張機能の画面になる。（ `Command + Shift + X` でも）  
以下のような種類の拡張機能が存在する。

- ［Debuggers］（デバッガー）
- ［Languages］（言語）
- ［Linters］（Lintツール）
- ［Snippets］（スニペット）
- ［Themes］（テーマ）
- ［Other］（その他）

検索欄を空にすると現在インストール済みの拡張機能の一覧が表示される。  
なお、拡張機能の反映には VS Code を再起動するか再読み込みボタンを押す。

# アップデート

## VS Code

「 Code 」 -> 「更新の確認」。

## 拡張機能

拡張機能画面の右上の「・・・」をクリックして「更新の確認」。

# オススメの拡張機能

## 視覚サポート

- Trailing Spaces
    - 改行部分の最後に入る半角スペースの強調、削除
- EvilInspector
    - 文章中の全角スペースを強調, 削除

## Markdown

- Auto-Open Markdown Preview
    - Markdown ファイルを開くときに自動でプレビューを開く
- Markdown+Math
    - `$$` 内に数式を書いて `Ctrl + Shift + .` と打てば、数式対応のプレビューが表示

## Git

- Git History
    - ツリー表示や差分表示など
- gi
    - [gitignore.io](https://github.com/joeblau/gitignore.io) から gitignore を追加
- gitignore
    - [github/gitignore.io](https://github.com/github/gitignore) から gitignore を追加

## マークアップ

- Auto Complete Tag
    - タグを自動で閉じて、開始・終了タグを変更したらもう片方のタグも自動で変更

## その他

- Shortcuts
    - VS Code 最下部にショートカットボタンを追加