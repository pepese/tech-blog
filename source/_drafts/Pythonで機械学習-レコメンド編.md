---
title: Pythonで機械学習 レコメンド編
tags:
- Python
- Machine Learning
- scikit-surprise
- Collaborative filtering
id: python-ml-recommendation
---

代表的なレコメンドアルゴリズムと Python での実装をまとめる。  
スクラッチではなく、できるだけライブラリを利用する。

# レコメンドアルゴリズム

- 協調フィルタリング
    - ユーザの利用履歴を扱う
    - トランザクションデータ、ユーザ・アイテム行列
- ポピュラリティ
    - 所謂人気ランキング
- コンテンツベース（内容ベース）フィルタリング
    - アイテム間の類似度に基づいたレコメンド
        - アイテムの特徴ベクトルで類似度（ Cos 類似度など）ソートしてレコメンドする方法
        - 例：野球のバットを買った人には野球のボールをおすすめしよう
    - [参考](http://ohke.hateblo.jp/entry/2017/10/13/230000)
- 上記のハイブリッド

ここでは **協調フィルタリング** を扱う。

[参考](https://www.slideshare.net/takemikami/ss-76817490)  
[参考](http://d.hatena.ne.jp/EulerDijkstra/20130407/1365349866)  
[参考](http://www.kamishima.net/archive/recsysdoc.pdf)

レコメンドアルゴリズムでよく発生するネガティブが出来事は以下。

- 同じようなアイテムばかりレコメンドされる
- 人気のアイテム、長期間掲載しているアイテムばかりレコメンドされる
- 数年に一度しか購入しない物に対するレコメンド結果がずっと表示される
- ユーザー行動履歴が十分に蓄積されていないと精度がでない
- データの管理コストがでかい
- レコメンドの計算コストがでかい

## 協調フィルタリング

協調フィルタリングには主に以下の種類がある。

- メモリベース（近傍ベース）
    - ユーザ・商品行列をそのまま利用、モデルを作成しない
    - 以下の手法がある
        - ユーザベース
        - アイテムベース
    - スパース性（データサイズに対して意味のある情報が少ない、 0 が多い行列）の高いデータには適用し辛い
- モデルベース
    - ユーザ・商品行列をモデル構築に利用
    - 以下のような手法がある
        - 特異値分解（SVD）
        - 行列因子分解
        - 確率的（ナイーブベイズ、ベイジアンネットワークなど）
        - 分類、回帰
- 上記のハイブリッド

狭義の協調フィルタリングは、近傍ベースを指す。

### メモリベース協調フィルタリング

メモリベース協調フィルタリングのコンセプトは、「類似度」。  
過去の利用履歴から似たもの同士を明らかにし、この類似度を使ってオススメ商品を推定する。  
類似度といった場合、ユーザ同士の類似度と商品同士の類似度の 2 つが考えられ、前者を使う場合をユーザベース、後者をアイテムベースと言って区別する。

- ユーザベース
    - ユーザーの行動履歴からユーザー間の類似度を計算し、おすすめするアイテムを決める手法
    - 例
        - Aさんに何かおすすめしたい、Aさんは商品a、商品b、商品cを買っている。
        - 同じような行動履歴のBさんは商品a、商品b、商品c、商品dを買っている、ということはAさんには商品dをおすすめできる
- アイテムベース
    - ユーザーの行動履歴からアイテム間の類似度を計算し、類似するアイテムをおすすめする手法
    - 例
        - 商品Aは商品Bと一緒に購入されることが多いから、商品Aの購入者に商品Bをおすすめしよう

#### ユーザベース

ユーザベース協調フィルタリングのロジックは、いわゆるkNN回帰という機械学習手法になります。
kNN（k-Nearest-Neighbor、k近傍法）を簡単に説明すると、推定する対象に最も特徴が似ているk個の観測値を参考にし、値を推定しようというものです。kNNは一般に分類問題に適用されることが多いですが、オススメ度のような連続値を推定する場合、明示的にkNN回帰と呼んでいます。

アルゴリズムの概要は以下の通り。

1. あるユーザと他のユーザの **類似度** を計算
2. 他のユーザ利用したアイテムのうち、ユーザAがまだ利用していないアイテムの集合を抽出
3. それらのアイテム群のうち、おすすめ度が高いアイテムのリストを返却
    - この選定の際に、類似度が高いユーザが利用したアイテムほど重みが高くなるようにする

#### アイテムベース

仮定1： Aさんが未評価のアイテム（=推薦候補）のうち、気に入る可能性が高いのはAさんが高く評価しているアイテムと類似したものだろう。  
仮定2： アイテム同士は評価のパターンが近ければ、類似していると言えるだろう。

#### 用いられる類似度

類似度には以下のようなものがある。

- ユークリッド距離（ Euclidean distance ）
    - 平方ユークリッド距離（ Squared Euclidean distance ）
    - 2 点間の普通の距離
- コサイン類似度（ Cosine Similarity ）
- ピアソンの積率相関係数（ Pearson correlation coefficient ）
    - ユーザの評価をそのユーザの評価全体の平均を用いて正規化する
    - データが正規化されていないような状況でユークリッド距離よりも良い結果を得られることが多いとされる

ユーザベースでは、 **ピアソンの積率相関係数** を用いる。  
そうすることによって以下のように評価の傾向が似ている 2 ユーザ間で高い相関を得られる。

- ユーザ A ：「ラーメンはまずまずで 3 点だがカレーとチャーハンはイマイチだから 1.5 点だな……。」
- ユーザ B ：「ラーメンはうまくて 5 点だがカレーとチャーハンはふつうで 3.5 点だな……。」

また、アイテムベースでは **コサイン類似度** がよく用いられる。

[類似度の計算方法](https://qiita.com/ynakayama/items/59beb40b7c3829cc0bf2)

# Python ライブラリ

- [crab](https://github.com/muricoca/crab)
    - オワコン
- [python-recsys](https://github.com/ocelma/python-recsys)
    - オワコン
- [nimfa](https://github.com/marinkaz/nimfa)
    - 近年の流行りであるNMF を使った手法は、レコメンデーションのライブラリとしては存在しないようですが、実装において肝になる行列の演算はライブラリとして提供されているので、こちらを使うことでそれほど困難なく実装は実現できそうです。実装アルゴリズムはかなり豊富で、Factorizationの実装だけでも10種類以上ありました。
- **scikit-surprise**
    - デファクト臭
    - [Github](https://github.com/NicolasHug/Surprise)
    - [Doc](https://pypi.python.org/pypi/scikit-surprise)
- **fastFM**
    - 更新は頻繁だが、アカデミック臭が強いような、、、いいけど、ここでは使わない
    - [Github](https://github.com/ibayer/fastFM)
    - [論文](http://www.jmlr.org/papers/volume17/15-355/15-355.pdf)

# Python で実装

scikit-surprise で実装できるアルゴリズムの一覧は [ここ](http://surprise.readthedocs.io/en/stable/prediction_algorithms_package.html) 。

## 環境設定

環境設定は以下。

```sh
$ python -V
Python 3.6.1
$ pip -V
pip 9.0.1
$ pip install jupyter scikit-learn matplotlib scipy numpy scikit-surprise
```

## データセット

データセットには [SUSHI Preference Data Sets](http://www.kamishima.net/sushi/) を使用する。

```sh
$ wget http://www.kamishima.net/asset/sushi3-2016.zip
$ unzip sushi3-2016.zip
$ rm sushi3-2016.zip
$ ls ./sushi3-2016/
README-en.txt
sushi3.idata
sushi3b.5000.10.order
README-ja.txt
sushi3.udata
sushi3b.5000.10.score
README-stat-ja.txt
sushi3a.5000.10.order
```

`sushi3b.5000.10.score` は 5000 行 100 列( 5000 人 × 寿司ネタ 100 種類)の構成となっており、各要素には評価値 0 〜 4 (値が大きいほど、好き)と、欠測値 -1 がセットされている。

## データのロード

scikit-surprise でデータをロードするには、 `Dataset` クラスに対応した形式にする必要がある。  
形式は **1行1評価値** で、以下のようにする。

```
ユーザID アイテムID 評価値
```

`sushi3b.5000.10.score` を上記の形式に変換・Dataset形式でロードするコードは以下。  
なお、ユーザ ID は 0000 〜 4999 、アイテム ID は 00 〜 99 。

```python
def convert(input_file_name):
    # 返還後のファイル名
    output_file_name = input_file_name + '_converted'
    output = ''

    with open(input_file_name, mode='r') as f:
        lines = f.readlines()
        user_id = 0
        for line in lines:
            words = line.strip().split(' ')
            for item_id, word in enumerate(words):
                score = int(word)
                if score != -1:
                    output += '{0:04d} {1:02d} {2:01d}\n'.format(user_id, item_id, score)

            user_id += 1

    with open(output_file_name, mode='w') as f:
        f.write(output)

    return output_file_name

output_file_name = convert('sushi3-2016/sushi3b.5000.10.score')

# with open(output_file_name, mode='r') as f:
#     print(f.read())

from surprise import Reader, Dataset

reader = Reader(line_format='user item rating', sep=' ')
dataset = Dataset.load_from_file(output_file_name, reader=reader)

trainset = dataset.build_full_trainset()
```

## メモリベース協調フィルタリング

[scikit-surprise での実装](http://surprise.readthedocs.io/en/stable/knn_inspired.html)

### ユーザベース

```python
from surprise import KNNBasic

sim_options = {
    'name': 'pearson',
    'user_based': True
}
algo = KNNBasic(k=5, min_k=1,sim_options=sim_options)
algo.train(trainset)

user_id = '{:04d}'.format(0)
item_id = '{:02d}'.format(92)

pred = algo.predict(uid=user_id, iid=item_id)
print('Predicted rating(User: {0}, Item: {1}): {2:.2f}'.format(pred.uid, pred.iid, pred.est))
```

スクラッチでの実装例は以下。

- http://ohke.hateblo.jp/entry/2017/09/22/230000
- https://qiita.com/hik0107/items/96c483afd6fb2f077985

### アイテムベース

```python
from surprise import KNNBasic

sim_options = {
    'name': 'cosine',
    'user_based': False
}
algo = KNNBasic(k=5, min_k=1,sim_options=sim_options)
algo.train(trainset)

user_id = '{:04d}'.format(0)
item_id = '{:02d}'.format(92)

pred = algo.predict(uid=user_id, iid=item_id)
print('Predicted rating(User: {0}, Item: {1}): {2:.2f}'.format(pred.uid, pred.iid, pred.est))
```

スクラッチでの実装例は以下。

- http://ohke.hateblo.jp/entry/2017/09/29/230000
- https://qiita.com/kotaroito/items/6acb58bb16b68a460af9

## モデルベース協調フィルタリング

### 特異値分解（ SVD ）

```python
from surprise import SVD

algo = SVD()
algo.train(trainset)

user_id = '{:04d}'.format(0)
item_id = '{:02d}'.format(92)

pred = algo.predict(uid=user_id, iid=item_id)
print('Predicted rating(User: {0}, Item: {1}): {2:.2f}'.format(pred.uid, pred.iid, pred.est))
```

http://ohke.hateblo.jp/entry/2017/10/06/230000
