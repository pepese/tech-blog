---
title: k8s に貢献するために理解すること
tags:
- k8s
- kubernetes
id: understanding-for-contributing-k8s
---

k8s に貢献と書いているが、普通の Github OSS 貢献にもかなり重複すると思う。たぶん。

# ドキュメント

- Kubernetes 公式ドキュメントの全て
    - https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational
- 貢献
    - [Kubernetes Contributor Guide](https://github.com/kubernetes/community/blob/master/contributors/guide/README.md)
    - [stack overflow](https://stackoverflow.com/questions/tagged/kubernetes)
    - [Testing Guide](https://github.com/kubernetes/community/blob/master/contributors/devel/testing.md)

# 理解すること

- スタンス
- アーキテクチャ全体像、各コンポーネントの機能を理解する
- Golang らしい書き方
- 標準パッケージと使い所
- 主要外部パッケージ
- テスト
- PR

## スタンス

- 端から端まで理解する必要はない（と思う）
    - てか無理ぽよ

## アーキテクチャ全体像、各コンポーネントの機能を理解する

https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613

## Golang らしい書き方

- interface
    - 例えば、 `pkg/kubelet/server/server.go` の `type HostInterface interface` の実装を追えば中身わかりそう
- channel
    - API サーバの goroutine の使い方
    - 例えば kube-apiserver は `chan struct{}` の雨嵐になっている、なぜなのか
    - [goroutine を安全に止める方法](https://qiita.com/castaneai/items/7815f3563b256ae9b18d)
    - graceful stop
        - https://gist.github.com/rcrowley/5474430
    - signal を捕まえる処理
        - https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/server/signal.go

## 標準パッケージと使い所

- flag
- context
    - `golang context`
    - https://deeeet.com/writing/2016/07/22/context/

## 主要外部パッケージ

- pflag
- go-restful
    - `https://github.com/emicklei/go-restful`
    - `.To(` などから API の実装ロジックを追えたりするようになる
- openapi、swagger

## テスト

https://github.com/kubernetes/community/blob/master/contributors/guide/README.md#testing

大きく以下のテストがある。

- Unit
    - Golang の標準 testing パッケージ
- Integration
    - Golang の標準 testing パッケージ
- e2e
    - BDD テストパッケージの利用
        - [Ginkgo](https://github.com/onsi/ginkgo)
        - [Gomega](https://github.com/onsi/gomega)
- Conformance

## PR

- Pull Request の流れを理解する
    - http://kik.xii.jp/archives/179
- [GitHub初心者はForkしない方のPull Requestから入門しよう](https://blog.qnyp.com/2013/05/28/pull-request-for-github-beginners/)
    - upstream に push する権限無い可能性が高いので、fork型かな
- [一人プルリクエストのすゝめ](https://crieit.net/posts/9c710ef2383a3703649ee712a9eb86e6)
- 特に k8s では [k8s-ci-robot](https://github.com/k8s-ci-robot) の介在がポイント。
- [Kubernetes-specific github workflow](https://github.com/kubernetes/community/blob/master/contributors/guide/pull-requests.md#the-testing-and-merge-workflow)
- みんなの PR をみる
    - https://github.com/kubernetes/kubernetes/pulls


## その他未整理

- kube-apiserver は etcd に更新があったときに kubelet に hook するっぽい。
    - https://stackoverflow.com/questions/50689126/kubernetes-how-does-api-server-etcd-know-the-status-of-each-pod

# やってることメモ

ビルドはえぐそうなのでビルドのコードを追って以下のことがわかった。  
Kubernetes をビルドして作成されるもの一覧。各々に `package main` `func main` あり。

- https://github.com/kubernetes/kubernetes/blob/master/build/BUILD

```
release_filegroup(
    name = "docker-artifacts",
    srcs = [":%s.tar" % binary for binary in DOCKERIZED_BINARIES.keys()] +
           [":%s.docker_tag" % binary for binary in DOCKERIZED_BINARIES.keys()],
)

# KUBE_CLIENT_TARGETS
release_filegroup(
    name = "client-targets",
    srcs = [
        "//cmd/kubectl",
    ],
)

# KUBE_NODE_TARGETS
release_filegroup(
    name = "node-targets",
    srcs = [
        "//cmd/kube-proxy",
        "//cmd/kubeadm",
        "//cmd/kubelet",
    ],
)

# KUBE_SERVER_TARGETS
# No need to duplicate CLIENT_TARGETS or NODE_TARGETS here,
# since we include them in the actual build rule.
release_filegroup(
    name = "server-targets",
    srcs = [
        "//cluster/gce/gci/mounter",
        "//cmd/cloud-controller-manager",
        "//cmd/hyperkube",
        "//cmd/kube-apiserver",
        "//cmd/kube-controller-manager",
        "//cmd/kube-scheduler",
    ],
)

# kube::golang::test_targets
filegroup(
    name = "test-targets",
    srcs = [
        "//cmd/gendocs",
        "//cmd/genkubedocs",
        "//cmd/genman",
        "//cmd/genswaggertypedocs",
        "//cmd/genyaml",
        "//cmd/kubemark",  # TODO: server platforms only
        "//cmd/linkcheck",
        "//test/e2e:e2e.test",
        "//test/e2e_node:e2e_node.test",  # TODO: server platforms only
        "//vendor/github.com/onsi/ginkgo/ginkgo",
    ],
)

# KUBE_TEST_PORTABLE
filegroup(
    name = "test-portable-targets",
    srcs = [
        "//hack:e2e.go",
        "//hack:get-build.sh",
        "//hack:ginkgo-e2e.sh",
        "//hack/e2e-internal:all-srcs",
        "//hack/lib:all-srcs",
        "//test/e2e/testing-manifests:all-srcs",
        "//test/kubemark:all-srcs",
    ],
)
```

上記の `main` のそれぞれの配置（クライアント、MasterNode、WorkerNode）、機能概要を調べて、どの `main` を読むか決める。  
実際にコンテナ作ってるところが見たいなぁ、、、

## クライアントツール

- `kubectl`
    - マニフェストの設定を `kube-apiserver`  へ送信する

## サーバ

### Master Node

- `kube-apiserver`
    - kubectl から設定情報を受け付けて、 クラスタが保持すべき状態として情報を `etcd` へ保存する
    - apiserver 以外のコンポーネントは直接 etcd を参照せず、 apiserver を通してリソースにアクセスする
- `etcd` （ Master Node ではなく独立したクラスタでもいい ）
    - 全てのクラスタデータを保持する
- `kube-scheduler`
    - 新しい Pod を作成し、 Woker Node を選択・配置する
- `kube-controller-manager`
    - `etcd` の情報とクラスタの現在の状態を比較し、 `etcd` の状態へ更新する（ apiserver 経由での参照）
    - controller は自作可能
    - controller は複数プロセス存在するが、バイナリは 1 つにまとめられる
        - Node Controller: Worker Node の停止を検出する
        - Replication Controller: Pod の数を制御する
            - Replication Controller はリソース名自体に"Controller"とつくため、コントローラーは"RepplicationManager"という名前
        - Endpoints Controller: Service や Pod 内のエンドポイントオブジェクトを設定する
        - Service Account & Token Controllers: namespace のデフォルトアカウント、API アクセストークンを作成する
    - 各 controller は goroutine で起動
- `cloud-controller-manager` （α版）
    - クラウドプロバイダのリソースをコントロールする
- `hyperkube`
    - Kubernetes関連のバイナリを1つにまとめたall-in-oneバイナリ

### Worker Node

- `kubelet`
    - クラスタ内のそれぞれの Worker Node で稼働するエージェント（ Worker Node のメイン処理）
    - PodSpecs の設定を提供し、コンテナが Pod 内で稼働していることを管理する
- `kube-proxy`
    - k8s クラスタのルーティングを担当
    - kube-apiserver をポーリングして設定変更を検知する
    - ホスト上のネットワークルールを維持し、接続転送を実行することによって、Kubernetesサービスの抽象化を可能にする
    - iptable の nat table を設定して IP 変更やロードバランス（ClusterIP）の設定を行う
- `kube-dns` （ Worker Node ではない？）
    - k8s クラスタの名前解決を担当
    - kube-apiserver をポーリングして設定変更を検知する
- Container Runtime
    - コンテナの実行環境
    - `Docker` `rkt` `runc` など
- cAdvisor
    - 各ノード上にあるコンテナのCPU、メモリ、ファイル、ネットワーク使用量といった、リソースの使用量と性能の指標を監視・収集するエージェント

#### 重要な概念？

- Sync Loop (SyncLoop)
    - kube-apiserver に更新を確認するループ？
        - kubelet の ConfigMap 更新だけ？ Pod の更新は？
        - kubelet に `--node-status-update-frequency duration` というオプションがあり、デフォルト 10s で見てるっぽい

## 処理の流れ

1. kubectl で Deploment API を呼ぶ
2. Deployment Controller が検知し、 ReplicaSet API を呼ぶ
3. ReplicaSet Controller が検知し、 Pod API を呼ぶ
4. Scheduler が検知し、配置先 Node を決定し、 Pod API を呼ぶ
5. Kubelet が検知し、 Pod を作成
- 各 Controller 、 Scheduler 、 Kubelet は常にそれぞれが Reconcilation Loop を回している

## Addons

- https://kubernetes.io/docs/concepts/overview/components/#addons

