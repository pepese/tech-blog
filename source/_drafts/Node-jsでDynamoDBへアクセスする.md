---
title: Node.jsでDynamoDBへアクセスする
tags:
- Node.js
- yarn
- AWS
- DynamoDB
id: nodejs-dynamodb
---

ここでは、Node.js から DynamoDB にアクセスしてみる。

- DynamoDB の概要
- DynamoDB の API
- AWS CLI で DynamoDB へアクセス
- Node.js での実装

以下の記事を読んだ前提で書く。

- [Node.js/npm入門](https://pepese.github.io/blog/nodejs-basics/)

<!-- more -->

# DynamoDB の概要

DynamoDB は AWS が提供する Key-Value 型 NoSQL のマネージドサービス。  
スキーマレスであるため、テーブル作成時に Key 以外の設定は必要ない。  
以下の順で説明していく。

- Value
- Key
- テーブル
- パーティション
- セカンダリインデックス
- キャパシティユニット
    - まだ書いてない

## Value

DynamoDB の Value は **Item** （ **項目** ）と呼ばれ、 1 つ以上の **Attribute** （ **属性** ）を持つ。  
Item は RDB でいうレコードに該当する。  
1つの Attribute につき、以下を指定する。

- Attribute Name
    - 属性名
- Attribute Type
    - 属性の型、以下の型がある
        - **スカラー型**
            - **1つ** の値を持つことができ、 **数値** 、**文字列** 、 **バイナリ** 、 **ブール** 、 **null** を選択できる
        - **ドキュメント型**
            - **入れ子** の値を持つことができ、 **リスト** 、 **マップ** を選択できる
            - リスト要素に保存できるデータ型に制限はなく、全て同じ型である必要はない
            - マップは JSON オブジェクトと同様
        - **セット型**
            - **複数** の値を持つことができ、 **数値** 、**文字列** 、 **バイナリ** を選択できる
            - **セット内の全ての要素は同じ型** である必要がある
- Attribute Value
    - Attribute Type に沿った属性の値

DynamoDBの各種キーは、 Item の中の Attribute から選択することになる。

### ドキュメント型/リストの例

```
FavoriteThings: ["Cookies", "Coffee", 3.14159]
```

### ドキュメント型/アップの例

```
{
    Day: "Monday",
    UnreadEmails: 42,
    ItemsOnMyDesk: [
        "Coffee Cup",
        "Telephone",
        {
            Pens: { Quantity : 3},
            Pencils: { Quantity : 2},
            Erasers: { Quantity : 1}
        }
    ]
}
```

### セット型の例

```
["Black", "Green" ,"Red"]

[42.2, -19, 7.5, 3.14]

["U3Vubnk=", "UmFpbnk=", "U25vd3k="]
```

## Key

DynamoDB では Key を **プライマリーキー** と呼び、以下の2種類の構成から選択できる。

1. **パーティションキー** のみの単純キー
2. **パーティションキー** と **ソートキー** の複合キー

パーティションキー、ソートキー共に Value で紹介した **Attribute と同様** の定義となる。  
パーティションキーは **Hash Key** 、 ソートキーは **Range Key** とも呼ばれることもある。

## テーブル

テーブルは **テーブル名** と **プライマリーキー** （オプションで、ローカルセカンダリインデックス ）で定義することができる。

## パーティション

DynamoDB はテーブルが作成された際に自動でパーティションが作成され、また、自動で管理される。
1つのパーティションには、 **スループット** と **データ容量が** が決まっており、それぞれ上限を超えたら自動的に新規にパーティションが作成される。
パーティションは **パーティションキー** と **ソートキー** から決定される。

DynamoDB はパーティションキーをハッシュ関数への入力し、その出力値に応じてパーティションを特定する。 Item はソートキーの値でソートされ、データ量が 10GB を超える場合はソートキーによりパーティション分割される。

## セカンダリインデックス

テーブルで 1 つ以上の[セカンダリインデックス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/SecondaryIndexes.html)（単に、インデックスともいう）を作成できる。  
セカンダリインデックスは必須ではないが、利用することにより query/scan の速度を向上することができる。

DynamoDB のインデックスには以下の 2 つある。

- [グローバルセカンダリインデックス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/GSI.html) （ **Global Secondary Index** / **GSI** ）
    - テーブルと異なるパーティションキー（必須）とソートキー（オプション）を持つインデックス
    - テーブル作成後に変更 **可能**
    - [ベストプラクティス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/GuidelinesForGSI.html)
- [ローカルセカンダリインデックス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/LSI.html) （ **Local Secondary Index** / **LSI** ）
    - テーブルと同じパーティションキー（必須）と、異なるソートキー（必須）を持つインデックス
    - テーブル作成後に変更 **不可能**
    - [ベストプラクティス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/GuidelinesForLSI.html)

テーブルあたり最大 5 個の GSI と 5 個の LSI を定義できる。  
全てのセカンダリインデックスは **完全に 1 つのテーブルに関連付け** られ、そこからデータを取得します。  
また、ソートキーの Attribute に条件を適用して **Item をフィルタリング** することができる。

# DynamoDB の API

DynamoDB の API は大きく以下がある。

- コントロールプレーン
    - DynamoDB テーブルを作成、管理
- データプレーン
    - テーブルデータのCRUD
- DynamoDB ストリーム
    - テーブルのストリームを有効または無効にし、ストリームに含まれるデータ変更レコードにアクセス

ここでは、コントロールプレーン、データプレーンについて触れる。

## コントロールプレーン

コントロールプレーンには以下の API がある。

- **CreateTable**
    - テーブルを作成する
    - オプションで、1 つ以上のセカンダリインデックスを作成し、テーブルに対して DynamoDB ストリーム を有効にできる
- **DescribeTable**
    - プライマリキーのスキーマ、スループット設定、インデックス情報など、テーブルに関する情報を取得する
- **ListTables**
    - すべてのテーブル名のリストを取得する
- **UpdateTable**
    - テーブルまたはそのインデックスの設定を変更、テーブルの新しいインデックスを作成または削除、またはテーブルの DynamoDB ストリーム 設定を変更する
- **DeleteTable**
    - テーブルを削除する

## データプレーン

データプレーンには以下の API がある。

- データの作成
    - **PutItem**
        - テーブルに Item を 1つ書き込む
    - **BatchWriteItem**
        - テーブルに Item を25個書き込みむ
        - PutItem を複数回呼び出すよりも効率的
        - 1つ以上のテーブルを跨った書き込みも可能
- データの読み取り
    - **GetItem**
        - テーブルから Item を 1つ取得する
    - **BatchGetItem**
        - テーブルから Item を 100 個取得する
        - GetItem を複数回呼び出すよりも効率的
        - 1つ以上のテーブルを跨った読み込みも可能
    - **Query**
        - パーティションキーを指定し、そのパーティションキーがある全ての Item を取得する
        - Item 内の属性全体、もしくは指定した属性のみを取得できる
        - オプションでソートキーの値に条件を適用してフィルタリングできる
    - **Scan**
        - 指定したテーブル、もしくはインデックスの全ての Item を取得する
        - Item 内の属性全体、もしくは指定した属性のみを取得できる
        - オプションでソートキーの値に条件を適用してフィルタリングできる
- データの更新
    - **UpdateItem**
        - テーブルの Item を 1 つ以上更新する
        - 条件付き更新が可能
        - オプションで、アトミックカウンタ（他の書き込みリクエストを妨害することなく、数値属性をインクリメントまたはデクリメント）も可能
- データの削除
    - **DeleteItem**
        - テーブルから Item を 1 つ削除する
    - **BatchWriteItem**
        - テーブルから Item を最大 25 個削除する
        - DeleteItem を複数回呼び出すよりも効率的
        - 1つ以上のテーブルを跨った削除も可能

# AWS CLI で DynamoDB へアクセス

## 環境設定

```sh
$ pip install awscli
$ pyenv rehash
$ which aws
/Users/xxxx/.anyenv/envs/pyenv/shims/aws
$ aws configure
```

## CreateTable

```sh
$ aws dynamodb create-table \
  --attribute-definitions '[{"AttributeName":"test_hash","AttributeType":"S"},{"AttributeName":"test_range","AttributeType":"S"}]' \
  --table-name 'test_table' \
  --key-schema '[{"AttributeName":"test_hash","KeyType":"HASH"},{"AttributeName":"test_range","KeyType":"RANGE"}]' \
  --provisioned-throughput '{"ReadCapacityUnits":5,"WriteCapacityUnits":5}'
```

## DeleteTable

```sh
$ aws dynamodb delete-table \
  --table-name 'test_table'
```

## ListTables

```sh
$ aws dynamodb list-tables
```

## PutItem

```sh
$ aws dynamodb put-item \
  --table-name 'test_table' \
  --item '{"test_hash":{"S":"xxxxx"},"test_range":{"S":"yyyyy"},"test_value":{"S":"zzzzz"}}'
```

## GetItem

```sh
$ aws dynamodb get-item \
  --table-name 'test_table' \
  --key '{"test_hash":{"S":"xxxxx"},"test_range":{"S":"yyyyy"}}'
```
