---
title: Pandasでデータ分析
tags:
- Python
- Pandas
id: pandas-data-analytics
---

# はじめに

Pandas はデータを全てメモリに展開して処理するため、メモリに乗り切らない大規模なデータを扱う場合は SQL などで前処理してから。  
（Apache Impala とか使える？）

- 小規模データ（ローカルマシンのメモリにデータが乗る）
    - Python、Pandas などを利用
- 中規模データ（ローカルマシンのメモリにデータがギリ乗らない）
    - Windows の場合：MS Access
    - Mac の場合：Base（ Libre Office にある）
        - HSQLDB Embedded
- 大規模データ（数十 GB 以上のデータ）
    - Hadoop、Apache Impala など、ガッツリ

また、ローカルマシンのメモリが小さく、別途メモリサイズの大きいサーバを用意して Jupyter Notebook を起動して外部からアクセスしたい場合は以下。

Jupyter Notebook で外部から接続を許可する方法について。  
サーバにて以下のコマンドで Jupyter Notebook を起動する。

```
$ jupyter notebook --ip=* --no-browser
```

表示されたパス（例えば以下）の localhost 部分をサーバの IP に書き換えてブラウザからアクセスする。

```
http://localhost:8888/?token=5a497cc7665e9fa60f20097e166c13c904958e0fe2e08cbb
```

基本操作参考：  
https://qiita.com/tanemaki/items/2ed05e258ef4c9e6caac

- Jupyter Notebook の拡張
    - [extensionを追加してもっと快適なJupyter環境を構築する](https://qiita.com/sasaki77/items/30a19d2be7d94116b237)
    - [Jupyter Notebookの拡張機能を使ってみる](http://cartman0.hatenablog.com/entry/2016/03/28/170319)
    - [Jupyter Notebookの次世代版「JupyterLab」を紹介する](http://www.monthly-hack.com/entry/2016/07/15/152726)

Pandas を用いたデータ分析を行う際のメモ。  
以下について記載する。

- ライブラリのロード
- ライブラリの設定
- データのロード

# ライブラリのロード

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
```

# ライブラリの設定

```python
dir(pd.options)
#dir(pd.options.display)
```

# データのロード

```python
# データのロード
df = pd.read_csv("train.csv")
```

# データの確認

```python
df.describe()
df.info()

# 欠損の確認
print(len(df.index) - df["PassengerId"].count())

# ユニークな値の数
print(df["PassengerId"].value_counts().count())

# 変数間の相関
plt.figure(figsize=(8, 6)) #heatmap size
sns.heatmap(df.corr(), annot=True, cmap='plasma', linewidths=.5) # annot:値を表示するかどうか linewidths: しきり線

# ある範囲のデータを取得
print(len(df.loc[df["Age"] <= 10 ].index))
print(len(df.loc[(df["Age"] > 10) & (df["Age"] <= 20)].index))
```

# 前処理

## データの整形

Pandas の文字列メソッドで置換や空白削除などの処理を行うことができる。  
Pandas では、 `pandas.DataFrame` の列（つまり、 `pandas.Series` ）に対して一括で処理を行うために、 `pandas.Series` に文字列メソッドが準備されている。

- 置換
    - str.replace()
- 空白削除
    - str.strip()
    - str.lstrip()
    - str.rstrip()
- 大文字小文字変換
    - str.lower()
    - str.upper()
    - str.capitalize()
    - str.title()

```python
# 全ての列を文字列として読み込み、空白を削除する処理
df = pd.read_csv("data.csv", dtype = "object")
for _col in range(len(df.colomns)):
  df.iloc[:, _col] = df.iloc[:, _col].str.strip()
```

## ダミー変数の作成

```python
pd.get_dummies(df["Sex"], prefix = "Sex", drop_first = True)
```
