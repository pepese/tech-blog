---
title: AWS Lambdaまとめ
tags:
- aws
- lambda
id: aws-lambda-basics
---

[AWS](https://docs.aws.amazon.com/ja_jp/index.html) [Lambda](https://docs.aws.amazon.com/ja_jp/lambda/) についてまとめる。

# サーバーレス概要

差別化につながるビジネス実装に集中したい、つまり、以下のような 差別化に繋がらない重い仕事 を避けたい。

- サーバーのセットアップ： OS・ネットワーク・ミドルウェア・ランタイムなどのセットアップ
- キャパシティやスケーラビリティの管理
- 耐障害性を確保するための冗長化
- セキュリティパッチの適用
- 差別化に繋がらない機能の実装：スロットリング、認証認可など

上記の労力は「物理マシン > 仮想マシン > コンテナ > サーバーレス」となる。  
[サーバーレス](https://aws.amazon.com/jp/serverless/) には以下の特徴がある。

- サーバーの管理が不要
    - インフラのプロビジョニング・管理・保守
- 柔軟かつ自動スケーリング
- 高可用性の自動化
    - 可用性と耐障害性

AWS では Lambda 以外にも以下のようなサーバーレスサービスがある。

- AWS Step Functions
- AWS CodePipeline / AWS CodeBuild
- AWS Serverless Application Repository
- AWS IAM / Amazon Cognito / Amazon VPC
- Lambda@Edge / AWS Greengrass
- AWS Fargate
- Amazon S3 / Amazon EFS
- Amazon DynamoDB / Amazon Aurora Serverless
- Amazon API Gateway
- Amazon SNS / Amazon SQS / AWS AppSync
- Amazon Kinesis / Amazon Athena

AWS Lambda は **イベントソース** から Lambda 関数（ファンクション）が呼び出され **ダウンストリーム** （サービス）へアクセスする構成となる。（イベントソース -> Lambda 関数 -> ダウンストリーム）

# AWS Lambda の基本

## Lambda 関数（ファンクション）

- 標準もしくは **カスタムラインタイム** でサポートされている言語で記述
- それぞれが隔離されたコンテナ内で実行され、 1 コンテナ＝ 1 イベントを処理する
- 実行時は言語の関数もしくはハンドラーを指定し、 JSON 形式のイベントデータを渡すことが可能
    - イベントソースが S3 の場合、バケット名などがイベントデータとなる
- コードは依存ライブラリをパッケージングした上でアップロードする（ **デプロイパッケージ** ）
    - 内部的には S3に保存されており、S3 の ARN で指定することも可能
- 以下の基本設定が可能
    - メモリ：64MBごとに128MB〜3008MBで指定可能で、CPU能力もこれに比例する
    - タイムアウト：Lambda関数の実行時間（最大 900 秒）
    - 実行ロール：Lambda関数からアクセス可能な AWS リソースをコントロールするための IAM ロール
- [制限](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/limits.html) あり
- OS のネイティブライブラリを利用する場合は、AWS Lambdaの実行環境（Amazon Linux）を要確認
- [料金](https://qiita.com/Keisuke69/items/e3f79b50b6039175401b)

## イベントソース

- イベントの発生元となるAWSサービスもしくはユーザが開発したアプリケーション
- 以下の 3 タイプある
    - ポーリング・ストリームタイプ：Lambda関数自体がイベントを検知
        - Kinesis、DynamoDBなど
    - ポーリングタイプ：Lambda関数自体がイベントを検知
        - SQSなど
    - プッシュタイプ：Lambda関数にイベントを通知
- 以下の分類がある
    - タイプ：
        - 同期：リクエストの処理結果含め返却
        - 非同期：リクエストが正常に受信できたかのみ返却
    - ベース：
        - ポーリングベースかつストリームベース：Lambda関数自体がイベントを検知
        - ポーリングベースかつストリームベースでない：Lambda関数自体がイベントを検知
        - それ以外（プッシュベース？）：Lambda関数にイベントを通知
    - 対応サービス
        - 同期・ポーリングベースかつストリームベース：
            - Amazon DynamoDB、Amazon Kinesis Data Streams
        - 同期・ポーリングベースかつストリームベースでない：
            - Amazon SQS
        - 同期・プッシュベース：
            - Amazon Cognito、AWS CloudFormation、Amazon Cloud Watch Logs、Amazon Alexa、Amazon Lex、Amazon API Gateway、Amazon CloudFront、Amazon Kinesis Data Firehose
        - 非同期：
            - Amazon S3、Amazon SNS、Amazon SES、Amazon Cloud Watch Events、AWS CodeCommit、AWS Config、AWS IoT ボタン
    - リトライの動き
        - 同期・ポーリングベースかつストリームベース：
            - データの有効期限が切れるまでリトライ
            - 失敗の間は同一イベントソースからの新規イベントはブロックされる
        - 同期・ポーリングベースかつストリームベースでない：
            - （Amazon SQSなので） Visibility Timeout のちに再実行される
            - 新規イベントの実行はブロックされない
        - 同期・プッシュベース：
            - エラーのステータスコード・内容が返却される
            - リトライはイベントソースの設定による
        - 非同期：
            - 2 回まで自動リトライ（遅延あり）され、その後イベントは破棄される
            - Dead Letter Queue（DLQ）を設定することでイベントを Amazon SQS / Amazon SNS へ移動可能

## VPC アクセス

- VPC 内のリソースへインターネットを経由せずアクセス可能
- Lambda 関数に対して VPC サブネットおよびセキュリティグループを設定する
    - 既存のものから変更も可能
    - 可用性確保のため、各AZ毎のサブネットを指定しておく
- 内部的には ENI がフェッチされている
    - IAM ロールに `AWSLambdaVPCAccessExecutionRole` ポリシーをアタッチしておく
    - 初回アクセスつまりコールドスタート時にはレイテンシ（10〜60秒）が発生する
    - ENI は複数の Lambda 関数から共有される
- 設定したタイミングからインターネットアクセスは不可能
    - パブリック IP がないため
    - NAT の準備が必要
- VPC の制限により ENI または サブネット IP が確保できない場合は失敗する
    - 非同期呼び出しの場合は CloudWatch Logs に記録されない

## 実行ロール

- IAM ロールで指定する
- 最低でも CloudWatch Logs へのアクセス許可が必要

## 同時実行数

- デフォルトは 1000 で、超えるとスロットリングエラーとなる
- ConcurrentExecutions と UnreservedConcurrentExecutions メトリクスで確認

## ライフサイクル

1. （ ENI の作成）
2. コンテナの作成
3. デプロイパッケージのロード
4. デプロイパッケージの展開
5. ランタイム起動・初期化
6. 関数の実行
7. コンテナの破棄

利用できるコンテナがない場合はコールドスタートし、再利用できるコンテナがある場合ウォームスタートする。

# AWS Lambda の使い方

## プログラミングモデル

- ハンドラー
    - 言語の関数もしくはメソッドで、実行時のエントリーポイントとなる
    - 1 つ目のパラメータとして JSON 形式のイベントデータが渡される
- コンテキスト
    - ラインタイムに関する情報が含まれる
    - 2 つ目のパラメータとして渡される
- ロギング
    - 標準出力が CloudWatch Logs に書き込まれる
    - CloudWatch Logs の制限を受けるため、スロットリングによって失われる可能性あり
- 例外処理
    - 言語仕様によりことなる
    - 同期呼び出しの場合はレスポンスされる
- 注意点
    - ステートレスにする
    - 同じインスタンスで実行されるとは限らない
        - ローカルリソースの利用・アクセスは処理実行中のみ有効
    - データは他の永続化サービスへ保存する

## Lambda 関数の設定

- VPC アクセスの設定
- 同時実行数の管理
    - アカンウト毎に割り当てられたデフォルト同時実行数 1000 を関数毎に割り当て可能
    - ダウンストリーム（ DB とか）に対する流量制御に利用
- 環境変数
    - キーと値のペアで設定可能
        - キーと値の合計が 4kB 以下、英字開始で英数字とアンスコのみ
    - デフォルトの環境変数もあり、関数の中から参照可能（ `AWS_REGION` など）
        - リザーブドがありの場合は上書き不可能
    - 関数のバージョニング機能に環境変数も保存される
    - AWS Key Management Service によりデプロイ後に自動的に暗号化
        - デフォルトでキーが自動生成されるが、独自キーの適用も可能
- バージョニング
    - Lambda 関数と環境変数などの情報を PublishVersion に保存可能
    - バージョンに対してエイリアス設定・付け替え可能
    - routing-config を利用することでエイリアスに対してトラフィックの割合を管理可能（カナリア的な）
- Lambda Layers
    - 各 Lambda 関数から共通で参照可能なコンポーネントを定義・参照可能
    - ZIP でアップロード
    - カスタムランタイムを Layer としてアップロードも可能
    - `/opt` の下に指定した順に展開される（同名の場合後勝ち）
    - 非圧縮で 250MB まで（ Lambda 関数と同じ）
- タグ付け
    - 他の EC2 などのサービスと同様でタグ付け可能で、グループ化とフィルタリングに利用
- デッドレターキュー（ **DLQ** ）
    - 非同期実行時に 2 回リトライ行われた後にも失敗した場合にイベントを退避可能
        - Amazon SQS/SNS
    - 退避先へアクセスする権限付与が必要
    - DLQ 自体に書き込めない場合 `DeadLetterErrors` メトリクスに記録されイベントは破棄される
- Custom Runtimes
    - Linux 互換のランタイムを持ち込み可能 -> 任意の言語を実行可能になる
    - Custom Runtimes を利用する場合は Lambda 関数に `bootstrap` と呼ばれる実行ファイルを含める必要あり
        - Lambda Layers として登録も可能
    - `bootstrap` に以下の 2 つを実装する必要あり
        - `Initialization Tasks` ：ループしてイベントを待ち受ける
            - 環境変数の読み取り、関数の初期化、エラーハンドリングなどを実装する
        - `Processing Tasks` ：請求時間とタイムアウトの対象
            - イベントの取得、トレーシングヘッダの伝播、コンテキストオブジェクトの作成、関数ハンドラ呼び出し、レスポンス・エラーのハンドリング、クリーンアップなどを実装
- モニタリング
    - CloudWatch を利用したメトリクス監視
    - X-Ray を利用したトラブルシューティング
- 注意点
    - Labmda 関数が保証するのは最低 1 回実行されること
        - 冪等性を確保するなど設計しておく必要あり

## アプリケーションの管理

- AWS Serverless Application Repository
    - Lambda アプリケーションのリポジトリ
- AWS Serverless Application Model
    - サーバーレスアプリケーションの定義とパッケージング・デプロイ

# ペストプラクティス / アンチパターン

# セキュリティ

# ユースケース・事例
