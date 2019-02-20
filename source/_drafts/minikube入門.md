---
title: minikube入門
tags:
- Kubernetes
- k8s
id: k8s-minikube
---

minikube を構築してみる。 Mac 前提で書きます。

- minikube の意義
- minikube の構築

# minikube の意義

[Docker for Mac](https://docs.docker.com/docker-for-mac/) あるからいらねーじゃん、と思っていた。  
だがしかし、minikube と Docker for Mac では仮想マシンの種類が異なる。  
仮想マシンの種類は以下。（呼び方がいろいろあるみたいなので多少の違いはご愛嬌）

- ファームウェア型： LPAR 、 vPars
    - ハードウェアにファームウェアをインストールし、ハードウェア資源を区切ってOSに使わせる方式
- ハイパーバイザ型： Hyper-V 、 Xen 、 bHyve
    - ハードウェアにゲスト OS を直接コントロールするソフトウェアをインストールし、ハードウェア資源を小さなハードウェアの形で標準化・抽象化してOSに使わせる方式
- アプリケーション型： VMwarePlayer 、 VirtualBox
    - ハードウェアに何らかのホスト OS （ Linux など）がインストールされていて、その上にゲスト OS をコントロールするソフトウェアをインストールする方式

それぞれ以下のようなイメージ。

ファームウェア型
<table>
<tr>
<td align="center">ゲストOS</td><td align="center">ゲストOS</td>
</tr>
<tr>
<td colspan="2" align="center">ファームウェア</td>
</tr>
<tr>
<td colspan="2" align="center">ハードウェア</td>
</tr>
</table>

ハイパーバイザ型
<table>
<tr>
<td align="center">管理OS</td><td align="center">ゲストOS</td><td align="center">ゲストOS</td>
</tr>
<tr>
<td colspan="3" align="center">仮想マシンソフトウェア</td>
</tr>
<tr>
<td colspan="3" align="center">ハードウェア</td>
</tr>
</table>

アプリケーション型
<table>
<tr>
<td align="center">ゲストOS</td><td align="center">ゲストOS</td>
</tr>
<tr>
<td colspan="2" align="center">仮想マシンソフトウェア</td>
</tr>
<tr>
<td colspan="2" align="center">ホストOS</td>
</tr>
<tr>
<td colspan="2" align="center">ハードウェア</td>
</tr>
</table>

Docker for Mac はハイパーバイザー型で、 minikube はアプリケーション型になる。  
といっても今回はコンテナ文脈なので、ゲストOSはコンテナ、仮想マシンソフトウェアはコンテナランタイムに読み替える。  
Docker for Mac は MacOS においてLinux KVMに相当する仮想化ハイパーバイザを利用する `Hypervisor.framework` を使って実現してるらしい。  
何が言いたいかというと、 Mac 上で **コンテナランタイムを構築したホスト OS ( Linux ) にログインして何か検証したい場合は minikube 使う** 、だ。