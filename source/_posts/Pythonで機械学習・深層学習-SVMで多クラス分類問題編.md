---
title: Pythonで機械学習・深層学習 SVMで多クラス分類問題編
date: 2017-07-06 22:26:58
tags:
- Python
- Machine Learning
- Deep Learning
- SVM
id: python-ml-dl-svm-multiclass
---

ここでは、 **scikit-learn** の **SVM** モジュールを使用して **多クラス分類問題** を解いてみる。  
SVMを使用した2クラス分類問題は以下。

- [Pythonで機械学習・深層学習 SVMで2クラス分類問題編](https://pepese.github.io/blog/python-ml-dl-svm-2class/)

データセットは、以下で紹介している **digits データセット** を使用する。

- [Pythonで機械学習・深層学習 データセット編](https://pepese.github.io/blog/python-ml-dl-datasets/)

<!-- more -->

# パッケージの導入

このブログにある Python コードを実行するためのパッケージをインストールする。

```sh
$ pip install jupyter scikit-learn matplotlib scipy
```

実行環境は Jupyter Notebook を想定。実行方法は `jupyter notebook` 。  
**matplotlib** の使い方は以下を参照。

- [matplotlib入門](http://blog.pepese.com/entry/2016/09/18/174407)

# SVM の実装

```python
import numpy as np
from sklearn import datasets
from sklearn import svm
from sklearn import multiclass
from sklearn import metrics
# digits データセットのロード
digits = datasets.load_digits()
# 画像データを取得（一次元ベクトル）
data = digits.data
# 数字データを取得
labels = digits.target
# データセットの数を取得
num_of_samples = labels.shape[0]
# 学習データサイズの取得
training_data_size = int(num_of_samples * 3 / 5)
# 2クラス分類問題を解くSVMモデルの作成
estimator = svm.SVC(C=1.0, gamma=0.001)
# 2クラス分類問題を解くSVMモデルを他クラス問題へ対応
classifier = multiclass.OneVsRestClassifier(estimator)
# 学習用データをSVMへ適用
# データセットの前から 3/5 を学習用データとして使用
classifier.fit(data[:training_data_size], labels[:training_data_size])
# 正解データの取得
# データセットの後ろから 2/5 を評価用データとして使用
expected = labels[training_data_size:]
# 学習済み SVM へデータをインプットして結果を取得
predicted = classifier.predict(data[training_data_size:])
# 結果の出力
# SVMの出力と正解データを比較
print('Accuracy:\n', metrics.accuracy_score(expected, predicted))
```

[scikit-learnによる多クラスSVM](http://qiita.com/sotetsuk/items/3a5718bb1f945a383ceb)

k-分割交差検証について書く。
