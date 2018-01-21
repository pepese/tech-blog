---
title: Pythonで機械学習 ダミー変数編
tags:
id:
---

# 参考

- [Kaggle事始め](https://qiita.com/taka4sato/items/802c494fdebeaa7f43b7)
- [データ分析入門としてのKaggleコンペ「タイタニック乗客の生存予測」](http://simplestem.hatenablog.com/entry/2016/11/30/230641)
- [Kaggleのtitanic問題で上位10%に入るまでのデータ解析と所感](http://www.mirandora.com/?p=1804)


# メモ

- 変数同士の相関を見る（説明変数も目的変数も）
- 欠損値をうめる

# データのこと

https://www.slideshare.net/canard0328/ss-44288984

## 変数整理

- 説明変数、特徴量
    - 特徴量の分類１
        - 名義尺度：名前、電話番号など
        - 順序尺度：レースの順位など
        - 間隔尺度：温度など
        - 比例尺度：身長、体重、年齢など
    - 特徴量の分類２
        - 数値データ：比例尺度、（間隔尺度）
        - カテゴリデータ：名義尺度、順序尺度、（間隔尺度）
- 目的変数

## ダミー変数

カテゴリデータを数値データへ変換して作成する変数。  
機会学習データ数値データを前提としているアルゴリズムが多い。

- One-Hot Encoding
    - n種からなるカテゴリー変数をnつの0-1変数に置き換えたもの
    - 1 つの変数が三種類のカテゴリを持つ場合、0/1 のみになりうる 3 つの変数へ分解する
- ordinal encording of categorical variable
    - [Ordinal](http://contrib.scikit-learn.org/categorical-encoding/ordinal.html)

Pandas では `pd.get_dummies` 関数でダミー変数を取得できる。

## 欠損値の扱い

- 捨てる
    - 欠損が多い、データが大量
- 置換する
    - 最頻度、中央値、平均値
- 補完する
    - 時系列データ

完全情報最尤推定法、多重代入法

## 正規化 ( Normalization )

```
(x - xの平均)/xの標準偏差
```

平均 0 、標準偏差 1 にするのが一般的。]

L0正規化、L1正規化、L2正規化

## 次元の呪い

**特徴選択** や **次元削減** で説明変数を減らす。

- 特徴選択 ( Feature Selection )
    - 前向き法
        - 有効な特徴量を増やしていく
    - 後向き法
        - 不要な特徴量を減らしていく
- 次元削減 ( Dimension Reduction )
    - 主成分分析 ( Principal Component Analysis : PDA )
    - （線型）判別分析 ( Linear Discriminant Analysis : LDA )

## 醜いアヒルの子定理

# スパース性への対応

スパースモデリング、スパースコーディング  
スパース性回避のためにL1正規化を使う？  

Lasso の推定アルゴリズム、スパース推定してくれる  
LARS

# データへのアプローチ

1. 欠損値の処理
2. カテゴリ変数の処理
3. データの標準化
4. 特徴量の作成・選択
