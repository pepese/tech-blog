---
title: kubectl入門
tags:
- kubernetes
- k8s
- kubectl
id: kubectl-basics
---

# Kubernetes のリソース

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
    - Kubernetes ではコンテナを起動する際、 Pod 単位で起動する
    - Pod の中は Localhost の扱い
    - Pod 内には複数種類のコンテナを格納でき、メインのコンテナに加えて、補助的な役割を担うコンテナ（サブコンテナ）を加える構成のことを **サイドカー** と呼ぶ
- ReplicaSet
    - ReplicationController の後継
    - 複数の Pods を管理
    - Podのレプリカを生成し、指定した数のPodを維持し続けるリソース（ **オートヒーリング** ）
    - 監視は、特定の Label がつけられたPodの数をカウントする形で実現
        - レプリカ数が不足している場合は template から Pod を生成し、レプリカ数が過剰な場合は Label にマッチする Pod のうち1つを削除
    - selector をサポートする点において ReplicationController と異なる
    - set-based selector
    - ReplicaSet の特殊な形として「 DaemonSet 」「 StatefulSet 」がある
- ReplicationController
    - equality-based selector
- Deployments
    - Pods と ReplicaSets を一括で管理
    - Deployment は複数の ReplicaSet を管理することで、ローリングアップデートやロールバックなどを実現可能にするリソース
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

- Service
    - ClusterIP
    - NodePort
    - LoadBalancer
    - ExternalIP
    - ExternalName
    - Headless（None）
- Ingress

## Config＆Storage リソース

コンテナに対して設定ファイル、パスワードなどの機密情報などをインジェクトしたり、永続化ボリュームを提供したりするためのリソース。  
Kubernetesでは、個別のコンテナに対する設定の内容は環境変数やファイルが置かれた領域をマウントして渡すことが一般的

- Secret
- ConfigMap
- PersistentVolumeClaim

# kubectl コマンド

```
$ kubectl [command] [TYPE] [NAME] [flags]
```
- command
    - コマンドの種類
    - https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
- TYPE
    - リソースタイプ
    - https://kubernetes.io/docs/reference/kubectl/overview/#resource-types
- NAME
    - リソースの名前
- flags
    - オプション

# コマンド一覧

https://kubernetes.io/docs/reference/kubectl/overview/#operations

annotate	kubectl annotate (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]	Add or update the annotations of one or more resources.

api-versions	kubectl api-versions [flags]	List the API versions that are available.

- apply
    - `kubectl apply -f FILENAME [flags]`
    - ファイルもしくは標準入力からリソースの設定を行う

attach	kubectl attach POD -c CONTAINER [-i] [-t] [flags]	Attach to a running container either to view the output stream or interact with the container (stdin).
autoscale	kubectl autoscale (-f FILENAME \| TYPE NAME \| TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU] [flags]	Automatically scale the set of pods that are managed by a replication controller.
cluster-info	kubectl cluster-info [flags]	Display endpoint information about the master and services in the cluster.
config	kubectl config SUBCOMMAND [flags]	Modifies kubeconfig files. See the individual subcommands for details.

- create
    - `kubectl create -f FILENAME [flags]`
    - ファイルもしくは標準入力からリソース（オブジェクト？）を作成する

- delete
    - `kubectl delete (-f FILENAME \| TYPE [NAME \| /NAME \| -l label \| --all]) [flags]`
    - Delete resources either from a file, stdin, or specifying label selectors, names, resource selectors, or resources.

describe	kubectl describe (-f FILENAME \| TYPE [NAME_PREFIX \| /NAME \| -l label]) [flags]	Display the detailed state of one or more resources.
edit	kubectl edit (-f FILENAME \| TYPE NAME \| TYPE/NAME) [flags]	Edit and update the definition of one or more resources on the server by using the default editor.
exec	kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [-- COMMAND [args...]]	Execute a command against a container in a pod,
explain	kubectl explain [--include-extended-apis=true] [--recursive=false] [flags]	Get documentation of various resources. For instance pods, nodes, services, etc.

- expose
    - `kubectl expose (-f FILENAME \| TYPE NAME \| TYPE/NAME) [--port=port] [--protocol=TCP\|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type] [flags]`
    - レプリケーションコントローラ、サービス、またはポッドを新しいKubernetesサービスとして公開する
    - すでに存在するリソースに対して操作する

- get
    - `kubectl get (-f FILENAME \| TYPE [NAME \| /NAME \| -l label]) [--watch] [--sort-by=FIELD] [[-o \| --output]=OUTPUT_FORMAT] [flags]`
    - リソースの情報を表示する

label	kubectl label (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]	Add or update the labels of one or more resources.
logs	kubectl logs POD [-c CONTAINER] [--follow] [flags]	Print the logs for a container in a pod.
patch	kubectl patch (-f FILENAME \| TYPE NAME \| TYPE/NAME) --patch PATCH [flags]	Update one or more fields of a resource by using the strategic merge patch process.
port-forward	kubectl port-forward POD [LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N] [flags]	Forward one or more local ports to a pod.
proxy	kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix] [--api-prefix=prefix] [flags]	Run a proxy to the Kubernetes API server.
replace	kubectl replace -f FILENAME	Replace a resource from a file or stdin.
rolling-update	kubectl rolling-update OLD_CONTROLLER_NAME ([NEW_CONTROLLER_NAME] --image=NEW_CONTAINER_IMAGE \| -f NEW_CONTROLLER_SPEC) [flags]	Perform a rolling update by gradually replacing the specified replication controller and its pods.

- run
    - `kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [flags]`
    - Cluster 上でコンテナイメージを起動する（ Pod も作成される）

scale	kubectl scale (-f FILENAME \| TYPE NAME \| TYPE/NAME) --replicas=COUNT [--resource-version=version] [--current-replicas=count] [flags]	Update the size of the specified replication controller.
stop	kubectl stop	Deprecated: Instead, see kubectl delete.
version	kubectl version [--client] [flags]	Display the Kubernetes version running on the client and server.

# 一通り

Nginx のコンテナを実行する。

```
$ kubectl run nginx --image=nginx:1.11.3

$ kubectl get pods
NAME                     READY     STATUS              RESTARTS   AGE
nginx-74b65c798c-9zj44   0/1       ContainerCreating   0          14s

$ kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/nginx-74b65c798c-9zj44   1/1       Running   0          3s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10h

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1         1         1            1           3s

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx-74b65c798c   1         1         1         3s
```

Pod 、 deployment 、 Replica Set が作成された。  
上記で Pod が `ContainerCreating` になっているのがわかる。（ `Running` になるまで待つ。）  
この状態では、 **Worker Node 上でコンテナが動いているだけ** なので、 `expose` コマンドで Service を作成して **公開** する。

```
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10h

$ kubectl expose deployment nginx --port 80 --type LoadBalancer
service "nginx" exposed

$ kubectl get services
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP        10h
nginx        LoadBalancer   10.111.65.40   localhost     80:30562/TCP   5s

$ kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/nginx-74b65c798c-9zj44   1/1       Running   0          1m

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        10h
service/nginx        LoadBalancer   10.98.192.178   localhost     80:30033/TCP   4s

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1         1         1            1           1m

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx-74b65c798c   1         1         1         1m
```

nginx という Service が作成されて、 80 番ポートで公開されていることがわかる。  
試しにアクセスしてみる。

```
$ curl localhost
〜省略〜
<title>Welcome to nginx!</title>
〜省略〜
```

終わったので `kubectl delete` コマンドで停止する。

```
$ kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           21m

$ kubectl delete service nginx
service "nginx" deleted

$ kubectl delete deployment nginx
deployment.extensions "nginx" deleted

$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10h
```

## `kubectl expose` について

`kubectl expse` では Workloads リソースをコントロールできる。  
`--type` では Service の TYPE を選択できる。

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

# yaml ファイル

```
# pod_sample.yml
kind: Pod
apiVersion: v1
metadata:
  name: example-app
  labels:
    app: example-app
spec:
  containers:
  - name: example-app
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

全てのリソースは以下を含む。

- kind
    - リソースの種類
- apiVersion
    - API のバージョン
- metadata
    - リソースのメタデータ

リソースが Pod の場合は `spec` 以下に Pod の内容を定義する。

```
$ kubectl create -f pod.yaml
pod "example-app" created
```