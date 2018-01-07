---
title: Python入門
tags:
- Python
id: python-basics
---

自分にとって最低限のことだけまとめる。

- [Windows での環境構築](#Windowsでの環境構築)
- [モジュールとパッケージ](#モジュールとパッケージ)
- [起動モジュール](#起動モジュール)

<!-- more -->

公式ドキュメントは以下。

- [公式サイト](https://www.python.org/)
- [公式ドキュメント一覧](https://docs.python.jp/3/index.html)

<a id="Windowsでの環境構築"></a>
<a href="#Windowsでの環境構築"></a>
# Windows での環境構築

1. Anaconda の導入
    - [公式](https://www.anaconda.com/download/)
2. Microsoft VisualC++ Build Tools の導入
    - [直リンク](http://go.microsoft.com/fwlink/?LinkId=691126&fixForIE=.exe.)
    - [公式](http://landinghub.visualstudio.com/visual-cpp-build-tools)

<a id="モジュールとパッケージ"></a>
<a href="#モジュールとパッケージ"></a>
# モジュールとパッケージ

## モジュール

他のプログラムから再利用できるようにした Python ソースコードファイルのことを **モジュール** という。（あくまで **ソースファイルがモジュール** ）  
デフォルトで組み込まれているモジュールのことを **組み込みモジュール** という。  
import の形式は複数あるが、例えば以下。

```Python
from モジュール名 import メンバ,メンバ,…
```

`from math import cos,sin,tan` と記載すると `math.cos` ではなく `cos` で呼び出すことができる。  
`from` 句でモジュール（つまり Python ソースファイル）まで読む。  
**メンバ** はソースコード内の **クラス** や **メソッド** を指す。

## パッケージ

Pythonではディレクトリに `__init__.py`  （ `_` 2つ）があれば、そのディレクトリを **パッケージ** として扱える。  
import の際は以下の記述を行う。

```Python
import パッケージ名(.サブパッケージ名).モジュール名
```

http://note.crohaco.net/2014/python-module/


<a id="起動モジュール"></a>
<a href="#起動モジュール"></a>
# 起動モジュール

```python
def fibo(n):
    result = []
    a = 1
    b = 1

    while b < n:
        result.append(b)
        val = a + b
        b = a
        a = val
    return result

if __name__ == "__main__":
    print("{0}".format(fibo(100)))
```

`if __name__ == "__main__":` は、「このモジュールが(他のプログラムから呼び出されたのではなく)自身が実行された場合」という意味で、上記のモジュールを直接実行したい際の所謂 **メイン関数** となる。

# 組み込み系

Python には標準で以下が組み込まれている。

- [組み込み関数](https://docs.python.jp/3/library/functions.html)
- [組み込み定数](https://docs.python.jp/3/library/constants.html)
- [組み込み型](https://docs.python.jp/3/library/stdtypes.html)
- [組み込み例外](https://docs.python.jp/3/library/exceptions.html)

# 制御文

## 三項演算子

```
条件がTrueのときの値 if 条件 else 条件がFalseのときの値
```

# 関数

## アンパック

関数の引数にリストやディクショナリの要素を渡したい場合、 `*` `**` で **アンパック** して渡すことができる。

## 関数アノテーション（ Function Annotations ）

関数の引数と返り値の **型アノテーション** 。  
「 def 関数名(引数1<font color="red">: 型</font>, 引数2<font color="red">: 型</font>,...) <font color="red">-> 返り値の型</font>: 」。  
型アノテーションを理解する IDE などで警告がでるようになる。

# 内包表記（comprehension）

Pythonには **内包表記** という文法がある。  
これを使うと、リストやディクショナリなどの加工をするような処理をブロックを使用せずに簡潔に記述できる。  
内包表記には **リスト内包表記** 、 **ディクショナリ内包表記** 、 **set内包表記** がある。

## リスト内包表記（list comprehension）

基本構文

```
[ リストの要素 for シーケンスの要素 in シーケンス ]
```

例えば、 `[x**2 for x in range(10)]` は

- `range(10)`
    - 0から9までの整数のシーケンスを
- `x in range(10)`
    - 小さい順から取り出して
- `x**2 for x in range(10)`
    - 取り出した要素を二乗
- `[x**2 for x in range(10)]`
    - してできるリスト

となる。  
また、if文を使用してシーケンスの要素を絞ることができる。

```
[ リストの要素 for シーケンスの要素 in シーケンス if シーケンスの要素を絞る条件 ]
```

例えば、 `[x for x in data if data.count(x) > 1]` は

- `x in data`
    - dataリストの要素で
- `x in data if data.count(x) > 1`
    - dataリスト内で1つより多い要素
- `[x for x in data if data.count(x) > 1]`
    - のリスト

# lambda式

基本構文

```
lambda 引数のリスト : 引数を使った式
```

# 文字列の扱い

## f 文字列

式を埋め込んだ文字列で、フォーマット済み文字列（ formatted string ）と呼ばれる。  
先頭に `f` をつけ、文字列内の `{}` で囲まれた部分が式として評価される。

```
>>> f'100+1={100+1}'  # 式を評価
'100+1=101'

>>> order={'spam':100, 'ham':200}
>>> f'spam: {order["spam"]}, ham: {order["ham"]}' # 変数を参照
'spam: 100, ham: 200'

>>> f'Hello, {input("名前を入力してください: ")}' # 関数の呼び出し
名前を入力してください: ぺーぺーSE
'Hello, ぺーぺーSE'
```

## raw 文字列

エスケープシーケンスを無効にしたい場合の書き方。  
例えば、 `\n` を出力したい場合 `\\n` とエスケープしなければならないが、 raw 文字列を使用すると、 `\n` のみでよい。  
先頭に `r` をつけると raw 文字列になる。

```
>>> print("Hello\\nPython")
Hello\nPython
>>> print("Hello\nPython")
Hello
Python
>>> print(r"Hello\nPython")
Hello\nPython
```

## 文字列置換

文字列置換には以下の方法がある。

- % 演算子
- `format()` 関数

### % 演算子

```
# 基本形
>>> print 'Hello, %s!' % 'World'
Hello, World!
# 複数ある場合はタプルで渡す
>>> print 'Hi, %s. You are %d years old' % ('John', 25)
Hi, John. You are 25 years old
```

|変換|意味|
|:---|:---|
|%d|符号つき 10 進整数|
|%e, %E|指数表記の浮動小数点数|
|%r|文字列（Python オブジェクトを repr() で変換したもの）|
|%s|文字列（Python オブジェクトを str() で変換したもの）|

### `format()` 関数

```
# 基本形
>>> print 'Hello, {}!'.format('World')
Hello, World!

# ポジション引数(インデックス)を使う
>>> print '{0}, {2}, {1}'.format('a', 'b', 'c')
a, c, b

>>> print '{0}, {1}, {0}'.format('a', 'b')
a, b, a

# キーワード引数を使う
>>> print '{language} has {number} quote types.'.format(language='Python', number=2)
Python has 2 quote types.

# タプル、リストでも
>>> coord = (3, 5)
>>> 'X: {0[0]};  Y: {0[1]}'.format(coord)
'X: 3;  Y: 5'
```

引数にdict型を使うことがでるが `**` でアンパックして渡す必要がある。

```
>>> coord = {'latitude': '37.24N', 'longitude': '-115.81W'}
>>> 'Coordinates: {latitude}, {longitude}'.format(**coord)
'Coordinates: 37.24N, -115.81W'
```

クラス属性へアクセスする。

```
>>> class Coord(object):
...     def __init__(self, lat, lon):
...         self.lat, self.lon = lat, lon
...
>>> coord = Coord('37.24N', '-115.81W')
>>> 'Coordinates: {0.lat}, {0.lon}'.format(coord)
'Coordinates: 37.24N, -115.81W'
```

## 文字列と真偽値の演算

任意の文字列と `False` との積は空文字になる。

```
>>> "hoge" * True
'hoge'
>>> "hoge" * False
''
```


# `locals()` と `globals()`

`locals()` は自身のローカル領域の変数の値を全て辞書形式で返してくれる。

```
>>> def hoge(str, num):
...     print(locals())
...
>>> hoge("moji", 123)
{'num': 123, 'str': 'moji'}
```

グローバル領域バージョンが `globals()` 。

```
>>> y = 30
>>> globals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__':
 None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'y': 30}
 ```

# `__xxx__` 系

## `__doc__`

`__doc__` を使うと、各クラスやメソッドの概要を閲覧出来る。  
概要の設定方法は、クラスやメソッドの直下にダブルクオート 3 つで囲んだ文字列を記載するだけ。

```
>>> class TestClass:
...     """This is TestClass.
...     """
...     def testMethod(self):
...         """This is testMethod
...         """
...         print( "hello" )
...         # comment2
...         """comment3
...         """
...
>>> print( TestClass.__doc__ )
This is TestClass.

>>> print( TestClass.testMethod.__doc__ )
This is testMethod
```

## `__call__`

`__init__` と同様にクラスに定義する関数。  
インスタンス生成では `__init__` しか呼び出されない。  
しかし、一度生成されたインスタンスを関数っぽく引数を与えて呼び出せば、 `__call__` が呼び出されるという仕組み。
もちろん、 `__call__` に返り値をつければ,インスタンスから得られた値を別の変数に使ったりもできるということ。

```
>>> class cls:
...     __call__ = "Hi. My name is {} and I'm {} years old".format
...
>>> say_hi = cls()
>>> say_hi("Alex", 32)
"Hi. My name is Alex and I'm 32 years old"
```

# 標準ライブラリ

## Template()

テンプレートでは、 `$` と変数名からなるプレースホルダ名を使う。  
プレースホルダの周りを `{}` で囲えば、プレースホルダの後ろにスペースを挟まず、英数文字を続けることができる。  
`$$` のようにすると、 `$` 自体をエスケープでる。

```
>>> from string import Template
>>> t = Template('${village}folk send $$10 to $cause.')
>>> t.substitute(village='Nottingham', cause='the ditch fund')
'Nottinghamfolk send $10 to the ditch fund.'
```

その他の標準ライブラリ

- https://docs.python.jp/3/tutorial/stdlib2.html

# その他

- [Pythonのvars()とdir()の違い](http://minus9d.hatenablog.com/entry/2016/11/13/222629)
