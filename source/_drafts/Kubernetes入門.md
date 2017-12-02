---
title: Kubernetes入門
tags:
id:
---

**Kubernetes** （ **k8s** ）をさわってみる。

# はじめに

## Kubernetes にできること

- 複数ホストにコンテナをデプロイ
- 関連するコンテナ毎にグルーピング
- コンテナの死活監視
- コンテナ間のネットワーク
- コンテナの負荷分散
- コンテナのリソース管理

## 構成・概念

- ホストマシン
    - 物理サーバ、もしくは仮想マシン、インスタンス
- **Kubernetes Master**
    - 管理サーバ
    - ホストマシン 1 台
    - クライアントツールから API 経由でコントロール
    - apiserver 、 controller-manager 、　scheduler などのプロセスが稼働
- **Kubernetes Node** (旧名 Minion)
    - Master からコントロールされるワーカーサーバ
    - ホストマシン複数台
    - コンテナがマウントされる
    - kubelet 、 kube-proxy 、 flanneld 、 docker デーモンなどのプロセスが稼働
    - 複数 Node をまとめて **Cluster** と呼ぶ
- **Pod**
    - コンテナを起動するとデフォルトではコンテナ毎に仮想 NIC （プライベート IP ）が割り当てられる
    - 複数のコンテナ間で共有して使用する仮想 NIC とそれを利用するコンテナ（と Volume ）をまとめて Pod という
    - Kubernetes では Pod 単位でコンテナを起動する
    - Pod の中は Localhost の扱い
    - **Replication Controller** (rc) が Pod 内のコンテナの多重度をコントロールする
        - 障害検知、オートスケール
- **Service**
    - 1 つのものとして振る舞う複数 Pod をまとまり
    - L3ロードバランサのようなもの
    - Pod へアクセスをプロキシする
    - 「 IP + Port 」のアクセスを複数 Pod へ割り振る
    - Node へは Service の単位で配備される
- **コンテナ**
    - 所謂 Docker コンテナ
    - 1 つの Volume を複数コンテナで共有できる
- flanneld
    - 各ホストマシン上で動くプロセス
    - コンテナ間の通信を可能にする
    - Kubernetes 管理下のコンテナに一意の IP を割り振る
- etcd
    - flaneld が使用する共有データ（分散 KVS ）
    - 管理用コマンドは etcdctl
- Label
    コンポーネントを整理するための任意のメタデータ    

「 **Node > Service > Pod > コンテナ** 」なイメージ。  
[イメージ図](https://www.slideshare.net/yhokkey/kubernetes-google-container-engine-dockergke/42)

## ツール

- Kubectl
    - kubectl is the command line tool for Kubernetes. It controls the Kubernetes cluster manager.
- Kubeadm
    - kubeadm is the command line tool for easily provisioning a secure Kubernetes cluster on top of physical or cloud servers or virtual machines (currently in alpha).
- Kubefed
    - kubefed is the command line tool to help you administrate your federated clusters.
- Minikube
    - 1 台のローカル端末で シングル No-de の Kubernetes Cluster をお試しできるツール
- Dashboard
    - Dashboard, the web-based user interface of Kubernetes, allows you to deploy containerized applications to a Kubernetes cluster, troubleshoot them, and manage the cluster and its resources itself.

## ドキュメント

- 単体ホストで動かす
    - [公式 /Minikube](https://kubernetes.io/docs/getting-started-guides/minikube/)
- 複数ホストで動かす
    - [公式](https://kubernetes.io/docs/setup/pick-right-solution/#hosted-solutions)

ここでは単体ホストで動かす。

# 環境設定

## インストール

```sh
$ brew install kubectl
```

公式のインストールドキュメントは[ここ](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。
