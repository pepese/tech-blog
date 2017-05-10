---
title: Macで開発環境を作る
date: 2017-05-09 19:38:08
tags:
- Mac
- Homebrew
- anyenv
- Ruby
- Node.js
- Python
- Perl
- Java
- STS
- Maven
id: mac-dev-environment
---

2016年下期にMacbook Proを買ったので開発環境の設定をまとめてみた。  
以下のようなことを書く。

- OSの設定
- 開発環境作成
  - 開発言語、ビルドツールとか
- エディタ・IDEの設定

<!-- more -->

# OSの設定

## Dockの大きさを調整する

- 「りんごマーク」->「システム環境設定」->「Dock」
    - ここで好きなように設定（サイズは最小、拡大をチェックして最大、がオススメ）

## トラックパッドの設定

- 「りんごマーク」->「システム環境設定」->「トラックパッド」
    - ここで好きなように設定（「タップでクリック」を選択）

## ドラッグ＆ドロップの設定

- 「りんごマーク」->「システム環境設定」->「アクセシビリティ」->「マウスとトラックパッド」を選択
-  「トラックパッドオプション」-> 「ドラッグを有効にする」->「ドラッグロックあり」

これで２タップ目でちょっと動かしたらファイル・ウィンドウをホールドでき、もう１タップしたリリースできる。

## デスクトップの整理

- デスクトップで右クリック（二本指クリック）->「表示順序」->「名前」を選択

## ライブ変換オフ

タイピング中に勝手に変換される、あれ。

- http://sbapp.net/appnews/osxelcapitan-33422

## Finderで隠しファイルを表示

```sh
$ defaults write com.apple.finder AppleShowAllFiles TRUE
$ killall Finder
```


# 開発環境作成

## Homebrewをインスール

以下を参照。

- https://pepese.github.io/blog/homebrew-basics/

## iTerm2をインストール

```sh
$ brew cask search iterm2
==> Exact match
iterm2
$ brew cask install iterm2
==> Moving App 'iTerm.app' to '/Applications/iTerm.app'
```

Macデフォルトのターミナルより便利。  
`Command + D` でウィンドウ分割できたりする。

## cURLをインストール

- ターミナルで`brew install curl`
- `brew list` でcURLが含まれているか確認

## Gitをインストール

- ターミナルで `brew install git`
- `brew list` でgitが含まれているか確認
- `git --version` で確認

Gitの使い方は以下。

- http://blog.pepese.com/entry/2017/02/17/182311

## anyenvをインストール

- [公式](https://github.com/riywo/anyenv)を参照
- スクリプト系プログラミング言語のインストール・バージョン管理ができるようになる

```sh
$ git clone https://github.com/riywo/anyenv ~/.anyenv
$ echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(anyenv init -)"' >> ~/.bash_profile
$ exec $SHELL -l
```

## Rubyをインストール

```sh
$ anyenv install rbenv
$ git clone https://github.com/sstephenson/rbenv-gem-rehash.git ~/.anyenv/envs/rbenv/plugins/rbenv-gem-rehash
$ exec $SHELL -l
$ rbenv install 2.3.0
$ rbenv global 2.3.0
$ ruby -v
```

「rbenv-gem-rehash」を導入することで「rbenv rehash」（~/.rbenv/versions/2.x.y/bin/ 以下に置いてあるコマンド群を ~/.rbenv/shims/以下に置いて使えるようにする）を実行する必要が無くなる。  

ついでいgemの確認とbundlerの導入。

```sh
$ which gem
/Users/xxx/.anyenv/envs/rbenv/shims/gem
$ gem install bunlder
$ which bundle
/Users/xxx/.anyenv/envs/rbenv/shims/bundle
```

## node.jsをインストール

```sh
$ anyenv install ndenv
$ ndenv install v4.2.5
$ exec $SHELL -l
$ ndenv shell v4.2.5
$ npm update -g npm
$ ndenv shell --unset
$ ndenv global v4.2.5
```


## その他スクリプト言語をインストール(\*\*env）

```sh
$ anyenv install plenv
$ anyenv install pyenv
$ plenv install 5.25.7
$ pyenv install 3.5.2
$ plenv global 5.25.7
$ pyenv global 3.5.2
$ plenv rehash
$ pyenv rehash
$ anyenv varsions
```

## Oracle Javaをインストール

```sh
$ brew cask install java
```

インストールされたJavaの確認は以下。

```java
$ /usr/libexec/java_home -V
Matching Java Virtual Machines (2):
    1.8.0_112, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_112.jdk/Contents/Home
    1.8.0_111, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/jdk1.8.0_112.jdk/Contents/Home
```

PATHが通ってなかったら `~/.bash_profile` に以下を加筆して `source ~/.bash_profile` 。

```sh
export JAVA_HOME=`/usr/libexec/java_home -v 1.8`
```

`1.8` の部分を `1.7` にするとJava7を使用できる。  
消す時は以下。

```sh
$ rm -rf /Library/Java/JavaVirtualMachines/jdk1.8.0_25.jdk/Contents/Home
```

## Mavenをインストール

```sh
$ brew search maven
maven                                    maven-shell
homebrew/completions/maven-completion    homebrew/versions/maven32
homebrew/versions/maven31                Caskroom/cask/mavensmate
$ brew install homebrew/versions/maven32
$ mvn -v
Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-15T02:29:23+09:00)
Maven home: /usr/local/Cellar/maven32/3.2.5/libexec
Java version: 1.8.0_112, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_112.jdk/Contents/Home/jre
Default locale: ja_JP, platform encoding: UTF-8
OS name: "mac os x", version: "10.12.2", arch: "x86_64", family: "mac"
```

# エディタ・IDEの設定

## Atom

[http://blog.pepese.com/entry/2016/11/23/173602:embed:cite]

## STS (Spring Tool Suite)

```sh
$ brew cask install sts
```

# その他ツール

## FileZilla

FTP、FTPS、SFTPのGUIクライアント。

```sh
$ brew cask install filezilla
```
