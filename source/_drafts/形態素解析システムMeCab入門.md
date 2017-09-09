---
title: 形態素解析システムMeCab入門
tags:
- MeCab
id: mecab-basics
---

[公式](http://taku910.github.io/mecab/)

# 環境構築

## インストール

```sh
$ brew install mecab
$ mecab --version
mecab of 0.996
```

## 辞書の追加

このままでは辞書がないので追加する。  
辞書の場所は `/usr/local/lib/mecab/dic/` 配下。

### mecab-ipadic-neologd のインストール

```sh
$ git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd.git
$ cd mecab-ipadic-neologd/
$ ./bin/install-mecab-ipadic-neologd -n -p /usr/local/lib/mecab/dic/neologd
```

最後に本当にインストールするか聞かれるから `yes` or `no` をタイプする。  
なお、筆者の環境では足りなかった以下のコマンドを追加した。

```sh
$ brew install xz
```

### デフォルトの辞書を変更する

`mecab -d /usr/local/lib/mecab/dic/neologd/` でも辞書を指定して実行できるが、めんどくさいのでデフォルトを変更する。  
`/usr/local/etc/mecabrc` の以下を変更する。

```
dicdir =  /usr/local/lib/mecab/dic/ipadic
# 以下のように変更
dicdir =  /usr/local/lib/mecab/dic/neologd
```

# 使い方

インタラクティブモードは以下。（そのままコマンドを打つ）

```sh
$ mecab
おはよう
おはよう	感動詞,*,*,*,*,*,おはよう,オハヨウ,オハヨー
EOS
```

第一引数にファイルを指定することもでき、 `-o` オプションで出力も可能。

```
$ mecab [入力ファイル] -o [出力ファイル]
```

## 形態素の見方

```
表層形\t品詞,品詞細分類1,品詞細分類2,品詞細分類3,活用型,活用形,原形,読み,発音
```

「表層形」は文章からそのまま抜き出した形。  
なお MeCab の品詞体系は [IPA品詞体系](http://www.unixuser.org/~euske/doc/postag/#chasen)というものが使われている。（茶筅と同じ）

## 未知語推定

MeCabには未知語を推定する機能があり、デフォルトでは有効。  
未知語を抽出したい場合は `-x` （ `--unk-feature` ）で未知語の表示形式を指定して実行する。

```sh
$ mecab -x "undef"
にゃほにゃほたまくろー
に      助詞,格助詞,一般,*,*,*,に,ニ,ニ
ゃほにゃほたまくろ      undef
ー      undef
EOS
```

## ユーザ辞書の追加

- https://taku910.github.io/mecab/dic.html

自分で単語を登録したいときの方法。  
csv ファイルに 1 行 1 単語で以下の形式で作成する。

```
表層形,左文脈ID,右文脈ID,コスト,品詞,品詞細分類1,品詞細分類2,品詞細分類3,活用形,活用型,原形,読み,発音
```

- 左文脈ID
- 右文脈ID
- コスト
