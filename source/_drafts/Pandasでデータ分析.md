---
title: Pandasでデータ分析
tags:
- Python
- Pandas
id: pandas-data-analytics
---

基本操作参考：  
https://qiita.com/tanemaki/items/2ed05e258ef4c9e6caac

Pandas を用いたデータ分析を行う際のメモ。  
以下について記載する。

- ライブラリのロード
- ライブラリの設定
- データのロード

Pandas はデータを全てメモリに展開して処理するため、メモリに乗り切らない大規模なデータを扱う場合は SQL などで前処理してから。  
Apache Impala とか使える？

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

# ダミー変数の作成

```python
pd.get_dummies(df["Sex"], prefix = "Sex", drop_first = True)
```
