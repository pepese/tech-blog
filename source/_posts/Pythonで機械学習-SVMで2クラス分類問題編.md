---
title: Pythonで機械学習 SVMで2クラス分類問題編
date: 2017-07-05 14:43:37
tags:
- Python
- Machine Learning
- scikit-learn
- SVM
id: python-ml-svm-2class
---

ここでは、 **scikit-learn** の **SVM** モジュールを使用して **2クラス分類問題** を解いてみる  
データセットは、以下で紹介している **digits データセット** を使用する。

- [Pythonで機械学習 データセット編](https://pepese.github.io/blog/python-ml-dl-datasets/)

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
from sklearn import metrics
# digits データセットのロード
digits = datasets.load_digits()
# digits データセットの中から 3 と 8 の位置を取得
# これを後に学習用データと評価用データに分割して使用する
flag_3_8 = (digits.target == 3) + (digits.target == 8)
# 3 と 8 の画像データを取得（一次元ベクトル）
data = digits.data[flag_3_8]
# 上記は下記のように 8x8 のimageを2次元ベクトルから 1次元ベクトルへ変換したものでもよい
# data = images.reshape(digits.images[flag_3_8].shape[0], -1)
# 3 と 8 の数字データを取得
labels = digits.target[flag_3_8]
# 以降、ホールドアウト検証にて正答率を算出する
# データセットの数を取得
num_of_samples = len(flag_3_8[flag_3_8])
# 学習データサイズの取得
training_data_size = int(num_of_samples * 3 / 5)
# SVMモデルの作成
classifier = svm.SVC(C=1.0, gamma=0.001)
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
print('Accuracy:\n', metrics.accuracy_score(expected, predicted)) # 0.937062937063
```

scikit-learn の SVM には、 LinearSVC 、 LinearSVR 、 NuSVC 、 NuSVR 、 OneClassSVM 、 SVC 、 SVR があり、整理すると以下のようになる。

- 分類問題に使用する SVM （Support Vector Classification）
  - SVC （C-Support Vector Classification）
    - 標準的なソフトマージン(エラーを許容する)SVM
    - [API Doc](http://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html)
  - LinearSVC （Linear Support Vector Classification）
    - カーネルが線形カーネルの場合に特化したSVM
    - [API Doc](http://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html)
  - NuSVC （Nu-Support Vector Classification）
    - エラーを許容する表現が異なるSVM
    - [API Doc](http://scikit-learn.org/stable/modules/generated/sklearn.svm.NuSVC.html)
- 回帰問題に使用する SVM （Support Vector Regression）[参考](http://d.hatena.ne.jp/saket/20130212/1360656405)
  - SVR
  - LinearSVR
  - NuSVR
- 異常検知に使用する SVM
  - OneClassSVM

パイパーパラメータの最適値を選ぶには以下の手法があり、うち **グリッドサーチ** は scikit-learn で実装されている。[参考](http://sucrose.hatenablog.com/entry/2013/05/25/133021)

- グリッドサーチ
  - 昔からある手法で、格子状の空間で最適なパラメータを探索する方法です
  - 格子の範囲を総当りするため、膨大な計算時間がかかるという課題がある
- ランダムサーチ
  - 無作為にパラメータを抽出して探索する
  - グリッドサーチよりも計算時間が短くて済むというメリットがある
- ベイズ最適化 (Bayesian Optimization)
  - 最初は無作為にパラメータを抽出して探索するが、その結果を使って次に探索する点を確率的に選ぶ方法
  - ディープラーニングを含む機械学習の手法で、比較的良いハイパーパラメータを探索できることが知られている
  - 特に、低次元のデータで大域的な解を求めたい場合や、グリッドサーチなどで時間がかかっている場合に有効な手法

# 精度の評価方法

## 学習用データと評価用データの使い方

- ホールドアウト検証
  - データセットを学習用データと評価用データへ2分割する方法
  - 割合は任意
- k-分割交差検証
  - データセットを k 分割して、そのうち 1 個を評価用データ、その他の k-1 個を学習用データとして使用する方法
  - 評価用データを変更しながら k 回繰り返した結果の平均を精度に用いることが多い
  - scikit-learn 0.18 までは `sklearn.cross_validation.KFold` モジュールだったが DeprecationWarning となっており、0.20 からは `sklearn.model_selection.KFold` を使用する
    - [参考](http://segafreder.hatenablog.com/entry/2016/10/18/163925)

先ほどの SVM の結果を k=5 の k-分割交差検証で評価すると以下のようになる。

```python
import numpy as np
from sklearn import datasets
from sklearn import svm
from sklearn import metrics
digits = datasets.load_digits()
flag_3_8 = (digits.target == 3) + (digits.target == 8)
data = digits.data[flag_3_8]
labels = digits.target[flag_3_8]
# ここまではさきほどと同じ
# 交差検証用モジュールのロード
import sklearn.model_selection
# 交差検証の実施
# train は学習用データ、testは評価用データ
for train, test in sklearn.model_selection.KFold(n_splits=5).split(data, labels):
  # 学習用データ
  train_data = data[train]
  train_labels = labels[train]
  # 評価用データ
  test_data = data[test]
  test_labels = labels[test]
  # SVMモデルの作成
  classifier = svm.SVC(C=1.0, gamma=0.001)
  # 学習用データをSVMへ適用
  classifier.fit(train_data, train_labels)
  # 正解データの取得
  expected = test_labels
  # 学習済み SVM へデータをインプットして結果を取得
  predicted = classifier.predict(test_data)
  # 結果の出力
  # SVMの出力と正解データを比較
  print('Accuracy:\n', metrics.accuracy_score(expected, predicted))
```

## 精度の指標

精度の指標には以下がある。

- 正答率（Accuracy）
  - 全体の事象の中で正解がどれだけあったかの割合
- 適合率（Precision）
  - モデルが Positive と判断した中で、本当に Positive なものの割合
- 再現率（Recall）
  - 本当に Positive なものの中で、モデルが Positive と判断できたものの割合
- F値（F-measure）
  - 適合率と再現率の調和平均
  - 適合率の再現理を総合的に見る際に使用

上記は以下の **混同行列** （ **Confusion Matrix** ）から算出することができる。

<table>
<tr>
<td></td><td></td><td colspan="2">予測</td>
</tr>
<tr>
<td></td><td></td><td>Positive</td><td>Negative</td>
</tr>
<tr>
<td rowspan="2">実際</td><td>Positive</td><td>True Positive (TP)</td><td>False Negative (FN)</td>
</tr>
<tr>
<td>Negative</td><td>False Positive (FP)</td><td>True Negative (TN)</td>
</tr>
</table>

- TP・・・Positive という予測が True （正解）だった数
- FP・・・Positive という予測が False （不正解）だった数
- TN・・・Negative という予測が True （正解）だった数
- FN・・・Negative という予測が False （不正解）だった数

上記を用いてそれぞれの指標値は以下のように算出できる。

$$ 正答率（Accuracy）=\frac{TP+TN}{TP+FP+FN+TN} $$

$$ 適合率（Precision）=\frac{TP}{TP+FP} $$

$$ 再現率（Recall）=\frac{TP}{TP+FN} $$

$$ F値（F-measure）=\frac{ 2 \times 適合率 \times 再現率 }{ 適合率 + 再現率 } $$

これらの使用値と混同行列は scikit-learn の metrics モジュールを用いて、以下のように算出できる。

- 正答率（Accuracy）
  - `metrics.accuracy_score(expected, predicted)`
- 適合率（Precision）
  - `metrics.precision_score(expected, predicted, pos_label=3)`
- 再現率（Recall）
  - `metrics.recall_score(expected, predicted, pos_label=3)`
- F値（F-measure）
  - `metrics.f1_score(expected, predicted, pos_label=3)`
- 混同行列（Confusion Matrix）
  - `metrics.confusion_matrix(expected, predicted)`

上記の `pos_label` は **Positive Label** のこと。  
ここでは、数字の「3」か数字の「8」かを分類する問題であったので、 `pos_label=3` は Positive を数字の「3」に指定したことになる。（ Negative は数字の「8」）  

以上の指標値を先ほど実行したSVMのサンプルで k-分割交差検証をした場合のコードは以下のようになる。

```python
import numpy as np
from sklearn import datasets
from sklearn import svm
from sklearn import metrics
digits = datasets.load_digits()
flag_3_8 = (digits.target == 3) + (digits.target == 8)
data = digits.data[flag_3_8]
labels = digits.target[flag_3_8]
import sklearn.model_selection
for train, test in sklearn.model_selection.KFold(n_splits=5).split(data, labels):
  train_data = data[train]
  train_labels = labels[train]
  test_data = data[test]
  test_labels = labels[test]
  classifier = svm.SVC(C=1.0, gamma=0.001)
  classifier.fit(train_data, train_labels)
  expected = test_labels
  predicted = classifier.predict(test_data)
  # ここまではさきほどと同じ
  # 結果の出力
  # SVMの出力と正解データを比較
  print('Confusion Matrix:\n', metrics.confusion_matrix(expected, predicted))
  print('Accuracy:\n', metrics.accuracy_score(expected, predicted))
  print('Precision:\n', metrics.precision_score(expected, predicted, pos_label=3))
  print('Recall:\n', metrics.recall_score(expected, predicted, pos_label=3))
  print('F-measure:\n', metrics.f1_score(expected, predicted, pos_label=3), '\n')
```

# アンサンブル学習

**アンサンブル学習** とは、いくつかの性能の低い分類器（弱仮説器）を組み合わせて、性能の高い 1 つの分類器を作る方法。  
弱仮説器のアルゴリズムは決まっておらず、適宜選定する必要がある。  
アンサンブル学習のイメージは「多数決」であり、以下の 2 つに分類できる。

- バギング
  - 学習データを抜けや重複を許して複数個のグループに分割し、学習データのグループ毎に弱仮説器を生成する方法
  - 分類時は、各弱仮説器の出力した分類結果の多数決をとる
- ブースティング
  - 複数の弱仮説器を用意し、重み付きの多数決で分類を実現する方法
  - 難易度の高い学習データを正しく分類できる弱仮説器の半径結果を重視されるように重みを更新していく
  - ある弱仮説器が間違ったデータを難易度の高いデータとし、「難易度の高いデータの抽出」と「難易度の高いデータに特化した弱仮説器の重みの計算」を反復して弱仮説器の重みを決定する

**Random Forest** はバギング、 **AdaBoost** はブースティングの例である。
