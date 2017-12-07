---
title: Python入門
tags:
id: python-basics
---

自分にとって最低限のことだけまとめる。

- Windows での環境構築
- モジュールとパッケージ
- 起動モジュール

# Windows での環境構築

1. Anaconda の導入
    - [公式](https://www.anaconda.com/download/)z
2. Microsoft VisualC++ Build Tools の導入
    - [直リンク](http://go.microsoft.com/fwlink/?LinkId=691126&fixForIE=.exe.)
    - [公式](http://landinghub.visualstudio.com/visual-cpp-build-tools)

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
