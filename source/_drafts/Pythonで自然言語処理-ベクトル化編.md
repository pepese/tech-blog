---
title: Pythonで自然言語処理 ベクトル化編
date: 2017-07-07 20:42:56
tags:
- Python
- gensim
- word2vec
- doc2vec
id: python-nlp-vector
---

- [word2vec 参考](https://foolean.net/p/71)
  - なんと **gensim** の中に word2vec モジュールがある模様
  - コーパスが必要な予感
- [doc2vec 参考](https://deepage.net/machine_learning/2017/01/08/doc2vec.html)
  - なんと doc2vec も **gensim** から使える模様

  <!-- more -->

# word2vec/Mecab を用いて意味ベクトルを作成する

Mecab については以下を参考。

- [形態素解析システムMeCab入門](https://pepese.github.io/blog/mecab-basics/)

## word2vec 導入

Macでの導入方法。

```sh
$ git clone https://github.com/svn2github/word2vec.git
$ cd word2vec/
$ make
gcc word2vec.c -o word2vec -lm -pthread -O3 -march=native -Wall -funroll-loops -Wno-unused-result
gcc word2phrase.c -o word2phrase -lm -pthread -O3 -march=native -Wall -funroll-loops -Wno-unused-result
gcc distance.c -o distance -lm -pthread -O3 -march=native -Wall -funroll-loops -Wno-unused-result
distance.c:18:10: fatal error: 'malloc.h' file not found
#include <malloc.h>
         ^~~~~~~~~~
1 error generated.
make: *** [distance] Error 1
```

Macでは `<malloc.h>` は必要ないので、以下コマンドでソースコードから除去する。

```sh
$ sed -ie 's/#include <malloc.h>//g' *.c
```

以下で動作確認。（ wget が無かったら `brew install wget` などで導入しておく）  
（ちなみに自分の環境では動かなかったwww）

```sh
$ ./demo-word.sh
```

なお、コンパイルして作成した `word2vec` コマンドは `make` を実行したカレントに作成されており、パスは通っていない。

## コーパスの収集

コーパスとして Wikipedia のデータを使用する。  
word2vec へのインプットとして上記のコーパスが分かち書きされた 1つのテキストデータを作成する必要がある。

```sh
$ wget https://dumps.wikimedia.org/jawiki/latest/jawiki-latest-pages-articles.xml.bz2
```

上記で取得したデータは XMLファイルなので、テキストファイルに整形する必要がある。  
`wp2txt` という Ruby 系のツールを使用する。  
インストールは以下。

```sh
$ gem install wp2txt
```

実行。

```sh
$ mkdir jawiki-latest-pages-articles
$ cd jawiki-latest-pages-articles
$ wp2txt --input-file ../jawiki-latest-pages-articles.xml.bz2
```

出力されたファイルを結合して分かち書きファイルを作成する。

```sh
$ cd ..
$ cat jawiki-latest-pages-articles/*.txt | mecab -O wakati > jawiki-latest-pages-articles-wakati-ipadic.txt
```

コーパスを使って学習する。

```sh
$ time {path to}/word2vec -train jawiki-latest-pages-articles-wakati-ipadic.txt -output jawiki-latest-pages-articles-wakati-ipadic.bin -size 200 -window 5 -sample 1e-3 -negative 5 -hs 0 -binary 1
```
