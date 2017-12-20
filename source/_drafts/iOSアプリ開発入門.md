---
title: iOSアプリ開発入門
tags:
id: ios-app-basics
---

# 準備するもの・こと

- Mac PC
- Xcode
    - IDE
- Apple ID
    - [実機でテスト](https://i-app-tec.com/ios/device-test.html)する時に必要
- Apple Developer Program 登録 / 開発者登録
    - アプリを Apple Store で公開したいときに必要
    - iOSアプリを作って自分のiPhoneだけで使うのであれば、開発者登録は必要ない
    - 年間 $99
    - [参考](https://i-app-tec.com/ios/developer-registration.html)
- ターミナルで `xcode-select --install`
    - コマンドラインを有効化
    - `xcrun` 、 `swift` などのコマンドが有効になる

# Xcode の使い方

1. Xcode を起動
2. Create a new Xcode project.
3. Single View App を選択（なんでもいい）して「 Next 」
4. アプリ名や開発言語などを選択して「 Next 」
5. ディレクトリを指定して「 Create 」
    - この時点でプロジェクトの初期設定が完了する
6. 左上の再生ボタンでシミュレータが起動

# 参考

- [Swift実践入門 〜 今からはじめるiOSアプリ開発！ 基本文法を押さえて、簡単な電卓を作ってみよう](https://employment.en-japan.com/engineerhub/entry/2017/05/25/110000)
- [Swift/iOSアプリ開発 学習の進め方とソース](https://qiita.com/y-some/items/200db9ac37150effc8ed)
- [はてな研修用教科書-SwiftでのiOSアプリ開発](https://github.com/hatena/Hatena-Textbook/blob/master/swift-development-apps.md)
