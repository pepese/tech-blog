---
title: Kubernetes の概念整理
tags:
- Kubernetes
- k8s
id: k8s-concept
---

**Kubernetes** （ **k8s** ）の概念について整理する。

- k8s で実現できること
- 概念ざっくり
- k8s のリソース
- マニフェストファイル

# k8s で実現できること

- 複数ホストにコンテナをデプロイ
- 関連するコンテナ毎にグルーピング
- コンテナの死活監視
- コンテナ間のネットワーク
- コンテナの負荷分散
- コンテナのリソース管理

# 概念ざっくり

本来はリソース種別毎に整理したいところだが、いきなり感あるので一旦リソース気にせずざっくり。

- **Pod**
    - 複数のコンテナ間で共有して使用する仮想 NIC とそれを利用するコンテナ（と Volume ）をまとめて Pod という
    - Kubernetes ではコンテナを起動する際、 Pod 単位で起動する
    - Pod の中は Localhost の扱い
    - **Replication Controller** (rc) が Pod 内のコンテナの多重度をコントロールする
        - 障害検知、オートスケール
    - 複数種類のコンテナを 1 つの Pod に入れることができる
- **Service**
    - Pod の集合に対して外部と通信を行うための通り道
    - L3ロードバランサのようなもの
    - Pod へアクセスをプロキシする
    - 「 IP + Port 」のアクセスを複数 Pod へ割り振る
    - Node へは Service の単位で配備される
- **コンテナ**
    - 所謂 Docker コンテナ
    - 1 つの Volume を複数コンテナで共有できる

「 **Node > Service > Pod > コンテナ** 」なイメージ。  
[イメージ図](https://www.slideshare.net/yhokkey/kubernetes-google-container-engine-dockergke/42)

# k8s のリソース

- Workloads リソース
    - コンテナの実行に関するリソース
    - https://thinkit.co.jp/article/13610
- Discovery＆LB リソース
    - コンテナを外部公開するようなエンドポイントを提供するリソース
    - https://thinkit.co.jp/article/13738
- Config＆Storage リソース
    - 設定・機密情報・永続化ボリュームなどに関するリソース
    - https://thinkit.co.jp/article/14139
- Cluster リソース
    - セキュリティやクォータなどに関するリソース
- Metadata リソース
    - リソースを操作する系統のリソース

## Workloads リソース

- Pod
    - 複数のコンテナ間で共有して使用する 1 つの仮想 NIC と、それを利用する複数のコンテナ（と Volume ）をまとめて Pod という
    - k8s ではコンテナを起動する際、 Pod 単位で起動する
    - Pod 内の各コンテナは localhost の扱い
    - Pod 内には複数種類のコンテナを格納でき、メインのコンテナに加えて、補助的な役割を担うコンテナ（サブコンテナ）を加える構成のことを **サイドカー** と呼ぶ
- ReplicaSet
    - ReplicationController （今後廃止）の後継
    - 複数の Pods を管理
    - Podの レプリカを生成し、指定した数の Pod を維持し続けるリソース（ **セルフヒーリング** ）
    - 監視は、特定の **Label** がつけられた Pod の数をカウントする形で実現
        - レプリカ数が不足している場合は template から Pod を生成し、レプリカ数が過剰な場合は Label にマッチする Pod のうち1つを削除
    - selector をサポートする点において ReplicationController と異なる
    - set-based selector
    - ReplicaSet の特殊な形として「 DaemonSet 」「 StatefulSet 」がある
- Deployments
    - Pods と ReplicaSet を一括で管理する ReplicaSet の上位互換
    - Deployment は複数の ReplicaSet を管理することで、ローリングアップデートやロールバックなどを実現可能にするリソース
    - `kubectl` でリソース管理する際、基本的には ReplicaSet を直接操作することはなく Deployment を操作する
- Job
    - コンテナを利用して一度限りの処理を実行させるリソース
- CronJob
    - ScheduledJob の後継
    - スケジュールされた時間に Job を生成

Pod の管理・制御を行うリソース（オブジェクト）を **コントローラ** という。

## Discovery＆LB リソース

- Service
    - 1 つのマイクロサービスととらえることができる
    - 「 Pod 宛トラフィックのロードバランシング」「サービスディスカバリと内部 DNS 」を実現
- Ingress
    - 省略

### Service

Service には以下の種類がある。

- ClusterIP
    - k8s クラスタ内からのみ疎通可能な Service （なので、「Cluster」IP
- NodePort
- LoadBalancer
- ExternalIP
- ExternalName
- Headless（None）

## Config＆Storage リソース

コンテナに対して設定ファイル、パスワードなどの機密情報などをインジェクトしたり、永続化ボリュームを提供したりするためのリソース。  
Kubernetesでは、個別のコンテナに対する設定の内容は環境変数やファイルが置かれた領域をマウントして渡すことが一般的

- Secret
- ConfigMap
- PersistentVolumeClaim

# マニフェストファイル

参考：https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/

## Deployment

```yaml
apiVersion: v1
kind: Deployment
metadata: // ObjectMeta
  annotations:
  labels:
  name:
  namespace:
spec: // DeploymentSpec
  replicas:
  selector: // LabelSelector // replicas で指定した数冗長化する対象のラベルを指定
  strategy: // DeploymentStrategy
  template: // PodTemplateSpec
    metadata: ObjectMeta
    spec: // PodSpec
      containers: // Container
      - image:
        name:
        ports: // ContainerPort
        resources: // ResourceRequirements
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 100m
            memory: 100Mi
      dnsConfig: // PodDNSConfig
      volumes: // Volume
status: // DeploymentStatus // あまりわからない、、、
```