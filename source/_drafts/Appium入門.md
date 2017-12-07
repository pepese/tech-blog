---
title: Appium入門
tags:
- appium
- Android
- iOS
id: appium-basics
---

Appium は Selenium WebDriver の一種。  
Node.js 上でサーバーとして動作し、HTTP 経由で WebDriver API を通して操作を受け付けるという仕組み。  
Appium の背後には iOS 用, Android 用, Win 用などのドライバがある。

<!-- more -->

# 環境設定

## Appium

```sh
$ brew install node
$ yarn global add appium appium-doctor wd
```

以下で起動。

```sh
$ appium &
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

### シミュレータ

- Xcode を起動
- Xcode -> Open Developper Tool -> Simulator
- Window -> Scale -> 50%
- ホームボタンは Shift + Command + H

### 実機

TBD

## Android

```sh
$ brew cask install java
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

### エミュレータ

1. Android Studio を起動
2. AVD Manager を起動
3. エミュレータを作成

### 実機

Android 実機での自動テスト手順は以下。

1. Android 実機を USB で PC に繋ぎ、実機側で USB デバッグを許可
2. `$ adb devices` で device 番号が表示されることを
3. 実機でアプリを起動して、パッケージ名などを確認
    - `$ adb shell pm list packages | grep pepese`
        - `package:org.pepese`
    - `$ adb shell dumpsys activity | grep uniqlo | grep Intent`
        - `Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10200000 cmp=org.pepese/org.pepese.MainActivity }`
4. appium-desktop を起動
    - Capability を設定

# サンプルを実行

## iOS

コマンドラインの iOS ビルドツールの準備。

```sh
$ xcode-select --install
$ sudo xcode-select --switch /Applications/Xcode.app
$ xcodebuild -version
Xcode 9.1
Build version 9B55
```

サンプルアプリとテストを取得してセットアップ。

```sh
$ git clone https://github.com/appium/sample-code
$ git clone https://github.com/appium/ios-test-app
$ cd ios-test-app
$ xcodebuild -sdk iphonesimulator
$ cd ..
$ cp -R ios-test-app/build sample-code/sample-code/apps/TestApp/build
$ cd sample-code/sample-code/examples/python
```

`xcodebuild -version -sdk` コマンドで `PlatformVersion` を確認し、 `ios_simple.py` ファイルの 20 行目あたりの `PlatformVersion` を書き直す。  
別ターミナルを起動し `appium &` コマンドで Appium を起動してから以下を実行。

```sh
$ py.test ios_simple.py
```


## Android

```sh
$ py.test android_simple.py
```

動かない、参考で以下確認。

https://qiita.com/natsuki_summer/items/2d8d60114cdb95929dcb
