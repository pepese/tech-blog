---
title: Appium入門
tags:
id:
---


Appium は Selenium WebDriver の一種。  
Node.js 上でサーバーとして動作し、HTTP 経由で WebDriver API を通して操作を受け付けるという仕組み。  
Appium の背後には iOS 用, Android 用, Win 用などのドライバがある。

# 環境設定（公式）

```sh
$ brew install node      # get node.js
$ npm install -g appium  # get appium
$ npm install wd         # get appium client (Web Driver)
$ appium &               # start appium
$ node your-appium-test.js
```

# 実際の環境構築メモ

## Appium

```sh
$ yarn global add appium appium-doctor wd
```

## appium-desktop

[ここ](https://github.com/appium/appium-desktop/releases/) から最新版を取得してインストール。

## iOS

Xcode を App Store でインストールしてから以下を実行。

```sh
$ sudo mkdir /usr/local/Frameworks
$ sudo chown -R $(whoami):admin /usr/local/Frameworks
$ brew install carthage
$ brew link carthage
$ appium-doctor --ios # インストール、設定が正しいかチェック
```

`brew install carthage` を実行する際、権限不足で `/usr/local/Frameworks` ディレクトリ作成に失敗する。  
 そのため、 `brew link carthage` に失敗するので、あらかじめディレクトリを作ってあげてからインストールする。


## Android

```sh
$ brew cask install android-sdk
```

`~/.bash_profile` に以下を加筆した後 `source ~/.bash_profile` を実行。

```
export JAVA_HOME=`/usr/libexec/java_home`
export ANDROID_HOME=/usr/local/share/android-sdk
export PATH=$JAVA_HOME/bin:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH
```

```sh
$ appium-doctor --android # インストール、設定が正しいかチェック
$ brew cask install android-studio
```

Android Studio を起動し、 Standard を選択してダウンロード。  
「[ダウンロードしたアプリケーションの実行許可]の下の方に intel なんたらかんたら」って出たら、Mac の「システム環境設定　> セキュリティとプライバシー」を開いて許可する。
