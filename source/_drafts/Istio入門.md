---
title: Istio入門
tags:
- kubernetes
- Istio
id: k8s-istio-basics
---

- https://qiita.com/toshiki1007/items/896c79d94755686e57cd


# サイドカー作成方法

これが重要。

```bash
#istioctl kube-injectでサイドカー(istio-proxy)を組み込んだManifestを作成し、そのManifestを使ってデプロイする
$ kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
```