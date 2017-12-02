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

Hypervisor (xhyve driver か VirtualBox か VMware Fusion) 、 **kuberctl** 、 **Minikuber** をインストールする。  
ここでは、Hypervisor に VirtualBox を選択する。

```sh
$ brew cask install virtualbox
$
$ brew install kubectl
$
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.24.1/minikube-darwin-amd64
$ chmod +x minikube
$ mv minikube /usr/local/bin/
$ which minikube
/usr/local/bin/minikube
```

公式のインストールドキュメントは[ここ](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 。

# 使い方

## Minikube

### クラスタ起動

VM 上にシングル Node Kubernetes Cluster が構築する。

```sh
$ minikube status
minikube:
cluster:
kubectl:
$
$ minikube start
$
$ VBoxManage list vms
"default" {b2aad115-9326-4545-bb47-45577d3c1242}
"minikube" {fc871584-7e67-4260-a64b-a39991102380}
$
$ minikube status
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

これで `kubectl` コマンドを使用して Kubernetes Cluster をコントロールできる。

### クラスタ停止

VM 上のシングル Node Kubernetes Cluster を停止する。

```sh
$ minikube stop
$ minikube status
minikube: Stopped
cluster:
kubectl:
```

### クラスタ削除

VM 上のシングル Node Kubernetes Cluster を削除する。

```sh
$ minikube delete
$ minikube status
minikube:
cluster:
kubectl:
$ VBoxManage list vms
"default" {b2aad115-9326-4545-bb47-45577d3c1242}
```

VM 上のゲスト OS も削除されているのがわかる。

## kubectl

クラスタを起動した時点で `kubectl` コマンドに対してコンテキストが設定されており、実行したコマンドが minikube VM の方を向いている。  
`kubectl` コマンドのリファレンスは[ここ](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) 。

### クライアント、サーバのバージョンを確認

```sh
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.4", GitCommit:"9befc2b8928a9426501d3bf62f72849d5cbcd5a3", GitTreeState:"clean", BuildDate:"2017-11-20T19:11:02Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.0", GitCommit:"0b9efaeb34a2fc51ff8e4d34ad9bc6375459c4a4", GitTreeState:"clean", BuildDate:"2017-11-29T22:43:34Z", GoVersion:"go1.9.1", Compiler:"gc", Platform:"linux/amd64"}
```

### Node の一覧を取得

```sh
$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    10m       v1.8.0
```

### コンテナをデプロイ

サンプルのコンテナをデプロイして、一覧を取得する。

```sh
$ kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080
deployment "kubernetes-bootcamp" created
$
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            0           58s
$
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-fwqf6   1/1       Running   0          10m
```

デフォルトでは、 Kubernetes Cluster 内でのみ通信可能で、外部から Master の apiserver へのアクセスができない。  
そのため、 `kubectl proxy` コマンドで外部から apiserver へアクセス可能なプロキシを作成する。

```sh
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

`ctrl + c` で終了する。  
終了しない状態で、別ターミナルで `curl` などを実行してアクセスを確認する。  
以下で apisever のバージョンを確認する。

```sh
$ curl http://127.0.0.1:8001/version
{
  "major": "1",
  "minor": "8",
  "gitVersion": "v1.8.0",
  "gitCommit": "0b9efaeb34a2fc51ff8e4d34ad9bc6375459c4a4",
  "gitTreeState": "clean",
  "buildDate": "2017-11-29T22:43:34Z",
  "goVersion": "go1.9.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

また、 apiserver に対して Pod 名を指定することによって、 Pod を経由してデプロイしたアプリケーションにアクセスできる。  
Pod 名は `kubectl get pods` で取得できる。

```sh
$ curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/kubernetes-bootcamp-6db74b9f76-fwqf6/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-6db74b9f76-fwqf6 | v=1
```

続きは以下。  
https://kubernetes.io/docs/tutorials/kubernetes-basics/explore-intro/
