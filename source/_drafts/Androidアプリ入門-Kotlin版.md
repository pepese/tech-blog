---
title: Androidアプリ入門 Kotlin版
tags:
- Android
- Kotlin
id: android-kotlin-basics
---

[公式ドキュメント](https://developer.android.com/studio/intro/index.html)

# Android Studio の設定

Java 1.8 はインストール済みな前提。  
バージョンは 3.0.1 for Mac 。

## インストール

```sh
$ brew update
$ brew cask install android-studio
```

## 初回起動時の設定

android-studio でインストールした android-sdk 系のツールは `~/Library/Android/sdk` 配下に配備される。  
以下の初回起動時の設定後にダウンロードされる。

1. Android Studio 起動
2. 「 Do not import settings 」 -> 「 OK 」
3. 「 Next 」
4. 「 custom 」 -> 「 Next 」
5. 「 Default 」 -> 「 Next 」
6. 以下を選択して「 Next 」
    - Android SDK
    - Android SDK Platform
    - Performance (Intel HAXM)
    - Android Virtual Device
7. 任意の Emulator Settings して「 Next 」 -> 「 Finish 」

「[ダウンロードしたアプリケーションの実行許可]の下の方に intel なんたらかんたら」って出たら、Mac の「システム環境設定　> セキュリティとプライバシー」を開いて許可する。  
`~/.bash_profile` に以下を加筆した後 `source ~/.bash_profile` を実行。

```
export JAVA_HOME=`/usr/libexec/java_home`
export ANDROID_HOME=$HOME/Library/Android/sdk
PATH=$PATH:$JAVA_HOME/bin
PATH=$PATH:$ANDROID_HOME/platform-tools
PATH=$PATH:$ANDROID_HOME/tools/bin

export PATH
```

`$ adb devices` 、 `$ sdkmanager --help` が実行できることを確認しておく。

## アンインストール

```
$ brew cask uninstall android-studio
$ rm -Rf ~/Library/Preferences/AndroidStudio*
$ rm -Rf ~/Library/Logs/AndroidStudio*
$ rm -Rf ~/Library/Caches/AndroidStudio*
$ rm ~/Library/Preferences/com.android.Emulator.plist
$ rm -Rf ~/Library/Android*
$ rm -Rf ~/AndroidStudioProjects
$ rm -Rf ~/.gradle
$ rm -Rf ~/.android
```

# Android Studio の使い方

1. Android Studio 起動
2. 「 Start a new Android Project 」
3. アプリ名、ドメイン名、「 Include Kotlin Support 」にチェックを入れて「 Next 」
4. アプリのプラットフォームを選択して「 Next 」
5. コンポーネント（ Basic Activity でよい）を選択して「 Next 」
6. メインとなる Activity の名前などを設定して「 Next 」->「 Finish 」
    - この時点でプロジェクトの初期設定環境
    - エラーメッセージが出た場合は、メッセージに従い SDK Manager などからツールをダウンロードする
6. エミュレータの作成
7. 右上の再生ボタンでアプリを実行可能

「 Shift 」を2回押すと、 Search Everywhere が起動する。

# プロジェクト構造

Android プロジェクト構造は以下。  
実際のディレクトリ構造と異なることに注意。

- app
    - manifest ： AndroidManifest.xml がある
    - java ： テストコードを含む Java ソースコードのファイルがある
    - res ： XML レイアウト、UI 文字列、ビットマップ イメージなど、コード以外のすべてのリソースがある（ resource の略）
        - drawable ： 画像
        - layout ： Activity のレイアウト
        - menu ： アプリのメニューのレイアウト
        - mipmap ： 拡大縮小アニメーションに対応した画像
        - values ： 文字列や値を XML で管理
- Gradle Script ： ビルドファイルがある
    - build.gradle (Project: xxxx)
    - build.gradle (Module: app)
    - gradle-wrapper.properties
    - proguard-rules.pro
    - gradle.properties
    - settings.gradle
    - local.properties
