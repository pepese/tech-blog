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
- Go の文法から設計思想を理解する
- 詳細理解のための標準パッケージと用途・意図
- 主要外部パッケージ
- テスト
- PR

## スタンス

- 端から端まで理解する必要はない（と思う）
    - てか無理ぽよ

## アーキテクチャ全体像、各コンポーネントの機能を理解する

https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613

## Go の文法から設計思想を理解する

- interface
    - 例えば、 `pkg/kubelet/server/server.go` の `type HostInterface interface` の実装を追えば中身わかりそう

## 詳細理解のための標準パッケージと用途・意図

- flag
- channel
    - API サーバの goroutine の使い方
    - 例えば kube-apiserver は `chan struct{}` の雨嵐になっている、なぜなのか
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