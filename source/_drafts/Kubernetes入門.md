---
title: Kubernetes入門
tags:
id: kubernetes-basics
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
    - apiserver 、 controller-manager(Replication Controller?) 、　scheduler などのプロセスが稼働
- **Kubernetes Node** (旧名 Minion)
    - Master からコントロールされるワーカーサーバ
    - ホストマシン複数台で構成されるクラスタ
    - コンテナがマウントされる
    - kubelet 、 kube-proxy 、 flanneld 、 docker デーモンなどのプロセスが稼働
    - 複数 Node をまとめて **Cluster** と呼ぶ
- **Pod**
    - 複数のコンテナ間で共有して使用する仮想 NIC とそれを利用するコンテナ（と Volume ）をまとめて Pod という
    - Kubernetes ではコンテナを起動する際、 Pod 単位で起動する
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
    - Node 上で動くプロセス
    - コンテナ間の通信を可能にする
    - Kubernetes 管理下のコンテナに一意の IP を割り振る
- etcd
    - Node 上で動くプロセス（違うかも、Masterからアクセスされる？）
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
代表的なものは以下。

- `kubectl get`
    - Kubernetes の各種リソース情報の一覧を取得
- `kubectl describe`
    - Kubernetes の各種リソース情報の詳細を取得
- `kubectl logs [pod_name]`
    - Pod 内のコンテナのログを取得
- `kubectl exec`
    - Pod 内のコンテナ上でコマンドを実行する

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

サンプルのコンテナをデプロイして、コンテナ、 Pod の一覧を取得する。

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

Pod 内のコンテナ上でコマンドを実行してみる。

```sh
$ kubectl exec kubernetes-bootcamp-6db74b9f76-5c4p4 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-6db74b9f76-5c4p4
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root
```

### コンテナへログイン

bash セッションを作成してコンテナへアクセスする。  
`exit` でぬける。

```sh
$ kubectl exec -ti kubernetes-bootcamp-6db74b9f76-5c4p4 bash
root@kubernetes-bootcamp-6db74b9f76-5c4p4:/#
root@kubernetes-bootcamp-6db74b9f76-5c4p4:/# curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-6db74b9f76-5c4p4 | v=1
root@kubernetes-bootcamp-6db74b9f76-5c4p4:/# exit
```

### Service

Pod は IP を持つが、その IP は Kubernetes Cluster の外部へ公開されない。  
Service は Pod へのトラフィックを中継する。  
Service は ServiceSpec という YAML もしくは JSON ファイルで設定する。  
Service には以下の種類（ **type** という）がある。

- ClusterIP (default)
    - Kubernetes Cluster 内に閉じる Internal IP を公開する
- NodePort
    - 指定したPort に対して Node の NAT を構成する
    - Kubernetes Cluster 外部に公開され、 `<NodeIP>:<NodePort>` でアクセスできる
    - ClusterIP のスーパーセット
- LoadBalancer
    - Kubernetes Cluster 外部に公開されるロードバランサ
    - 外部 IP アドレスが作成され、アクセス可能となる
    - NodePort のスーパーセット
- ExternalName
    - 任意の名前と設定できる
    - kube-dns の CNAME レコードとなる

あるService へ所属する Pod はラベル（ **LabelSelector** ）によって決定される。

```sh
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   43m
```

Service を作成してみる。

```sh
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
service "kubernetes-bootcamp" exposed
$
$ kubectl get services
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP          46m
kubernetes-bootcamp   NodePort    10.102.125.240   <none>        8080:31130/TCP   10m
$
$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.102.125.240
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31130/TCP
Endpoints:                172.17.0.4:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$
$ kubectl get services/kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.102.125.240   <none>        8080:31130/TCP   13m
```

`-l` オプションを使用して、ラベルを指定してリソースを取得可能。

```sh
$ kubectl get pods -l run=kubernetes-bootcamp
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          54m
$ kubectl get services -l run=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.102.125.240   <none>        8080:31130/TCP   20m
```

ラベルは以下のように付与する。

```sh
$ kubectl label pod kubernetes-bootcamp-6db74b9f76-5c4p4 app=v1
pod "kubernetes-bootcamp-6db74b9f76-5c4p4" labeled
$
$ kubectl describe pods kubernetes-bootcamp-6db74b9f76-5c4p4
Name:           kubernetes-bootcamp-6db74b9f76-5c4p4
Namespace:      default
Node:           minikube/192.168.99.100
Start Time:     Sun, 03 Dec 2017 09:10:57 +0900
Labels:         app=v1
                pod-template-hash=2863065932
                run=kubernetes-bootcamp
# (省略)
$
$ kubectl get pods -l app=v1
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          57m
```

削除は以下。

```sh
$ kubectl delete service -l run=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1h
```

### スケーリング

スケーリングの設定を行うことで Pod がスケールする。

```sh
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           1h
```

- DESIRED：設定されたレプリカ数
- CURRENT：起動中のレプリカ数
- UP-TO-DATE： DESIRED の状態になったレプリカ数
- AVAILABLE：ユーザからアクセス可能なレプリカ数

Pod を 4 つまでスケールさせてみる。

```sh
$ kubectl scale deployments/kubernetes-bootcamp --replicas=4
deployment "kubernetes-bootcamp" scaled
$
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           1h
$
$ kubectl get pods -o wide
NAME                                   READY     STATUS    RESTARTS   AGE       IP           NODE
kubernetes-bootcamp-6db74b9f76-2ppvx   1/1       Running   0          10m       172.17.0.6   minikube
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          1h        172.17.0.4   minikube
kubernetes-bootcamp-6db74b9f76-ct7lp   1/1       Running   0          10m       172.17.0.5   minikube
kubernetes-bootcamp-6db74b9f76-vkhbl   1/1       Running   0          10m       172.17.0.7   minikube
```

### コンテナをローリングアップデート

```sh
$ kubectl describe pods
```

現状は `v1` の Pod が 4 つあるのがわかる。  
`kubectl set image` コマンドでコンテナイメージを更新する。

```sh
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-2ppvx   1/1       Running   0          23m
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          1h
kubernetes-bootcamp-6db74b9f76-ct7lp   1/1       Running   0          23m
kubernetes-bootcamp-6db74b9f76-vkhbl   1/1       Running   0          23m
$
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
deployment "kubernetes-bootcamp" image updated
$
$ kubectl get pods
# (省略)
$ kubectl rollout status deployments/kubernetes-bootcamp
deployment "kubernetes-bootcamp" successfully rolled out
```

`kubectl get pods` コマンドでステータスが `Terminating` 、 `ContainerCreating` 、 `Running` となっていき、ローリングアップデートされていくのがわかる。  
`kubectl rollout status` でローリングアップデートが成功したか確認する。  
なお、なんらかが原因で失敗している場合、以下のコマンドで UNDO できる。

```sh
$ kubectl rollout undo deployments/kubernetes-bootcamp
```
