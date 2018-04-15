---
title: Hyperledger fabric入門
tags:
id: hyperledger-fabric-basics
---

Hyperledger Fabric（ハイパーレッジャー ファブリック）は、 Hyperledger といいう ブロックチェーン技術推進コミュニティ により共同検証された OSS で単に Fabric と呼ばれる。  
The Linux Foundation のプロジェクトの 1 つで Hyperledger Project 。

- [Hyperledger Fabric 1.0 概要](https://www.slideshare.net/Hyperledger_Tokyo/hyperledger-fabric-10)
- [公式サイト](https://www.hyperledger.org/)
- [公式 Github リポジトリ](https://github.com/hyperledger/fabric)
- [公式ドキュメント](http://hyperledger-fabric.readthedocs.io/en/latest/index.html)

<!-- more -->

# 特徴

Fabric の特徴は以下の通り。

- Industry-focused Design
    - ブロックチェーンは元々不特定多数を対象としたビットコインに利用される技術であったが、知っている者同士で契約を行う場面などビジネスにおける多様なユースケースに応える設計となっている
    - 参加者の登録と証明書発行、権限設定を担うメンバーシップ・サービスを実装しており、トランザクションは個々の参加者の証明書を用いて暗号化されています。プライベートな複数のブロックチェーンネットワークを有効にすることもできます。
- Modular Architecture
    - **メンバーシップサービス** 、 **ブロックチェーンサービス** 、 **チェーンコードサービス** という3つのカテゴリのコアコンポーネント
    - 個別のプロセス・名前空間・仮想マシンを持った物理的な個々の部品になっており、それぞれプラグアンドプレイが可能
    1. メンバーシップサービス
        - 参加者のアイデンティティと権限を管理
        - 参加者に対して登録証明書(Enrollment Certificates, ECert)や取引証明書(Transaction Certificates, TCert)を発行する認証局(CA)の役割を担う
    2. ブロックチェーンサービス
        - HTTP/2 上に構築された P2P プロトコルを通じて、ブロックチェーンとトランザクションを管理
        - 異なるコンセンサス・分散合意形成アルゴリズム(PBFT, Raft, PoW, PoS)にプラグイン可能で、デプロイ毎に設定
    3. チェーンコードサービス
        - ブロックチェーンにトランザクションの一部として保存されるアプリケーションレベルのコード
        - 各ノードでトランザクションを実行する方法(= チェーンコードの実行環境)を提供する
        - 実行環境は OS と言語、Runtime と SDK をセットにした Docker イメージになっており、Go, Java, Node.js をサポート

# 構成

- クライアント
    - 全PeerにTransaction Proposal(トランザクションのお伺い)を送って署名を集める
    - Ordererにトランザクションと署名とreadset/writeset*を送る
- Membership Service Providers
    - ユーザー認証や証明書の発行
    - ネットワークに複数存在(Org毎に1つ)
    - ※詳細は [こちら](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)
- Orderer
    - Proposal response(トランザクション+署名+readset/writeset*)複数Channelでリクエストを受け付ける
    - ブロック(トランザクション+署名+readset/writeset*)作成および整列
    - Channelに参加しているPeerにブロックを配布
- Peer
    - チェーンコード実行(台帳書き込みの代わりにreadset/writeset*を生成)および署名を実施
    - Ordererから受け取ったブロックで台帳書き込み
    - ※チェーンコード実行後に即台帳書き込み、ではないので注意

> - readset/writeset:
>     - readset: Peerがチェーンコードを実行した時点でのワールドステートのバージョン(キー毎)
>     - writeset: ワールドステートの更新内容

# 環境構築

## 準備

### cURL

```bash
$ brew install curl
```

### Docker ToolBox

```bash
$ brew cask install docker-toolbox
```

### Golang

```bash
$ brew install go
```

### npm

```bash
$ brew install node
```

`.bash_profile` に以下を追記。

```
export GOROOT=`go env GOROOT`
export GOPATH=`go env GOPATH`
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

```bash
$ source .bash_profile
```

### Python

```bash
$ brew install python
```

### 本体

```bash
$ mkdir fabric_sample
$ cd fabric_sample
```

Mac は Linux カーネルが無いので VM を立ててそこに Docker デーモンを起動させておく必要があるので以下を実行。

```bash
$ docker-machine create --driver virtualbox fabric
$ docker-machine ls
# fabric があることを確認
$ docker-machine start fabric
$ eval $(docker-machine env fabric)
```

以下を実行することで、必要なバイナリの入手および Docker Image を pull してくれる。

```bash
$ curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0
===> Downloading platform specific fabric binaries
===> Downloading platform specific fabric-ca-client binary
# ここまでがバイナリ入手ログ（一部省略
# ここから Docker Image pull ログ（一部省略
===> Pulling fabric Images
==> FABRIC IMAGE: peer
x86_64-1.1.0: Pulling from hyperledger/fabric-peer
==> FABRIC IMAGE: orderer
x86_64-1.1.0: Pulling from hyperledger/fabric-orderer
==> FABRIC IMAGE: ccenv
x86_64-1.1.0: Pulling from hyperledger/fabric-ccenv
==> FABRIC IMAGE: javaenv
x86_64-1.1.0: Pulling from hyperledger/fabric-javaenv
==> FABRIC IMAGE: tools
x86_64-1.1.0: Pulling from hyperledger/fabric-tools
===> Pulling fabric ca Image
==> FABRIC CA IMAGE
x86_64-1.1.0: Pulling from hyperledger/fabric-ca
===> Pulling thirdparty docker images
==> THIRDPARTY DOCKER IMAGE: couchdb
x86_64-0.4.6: Pulling from hyperledger/fabric-couchdb
==> THIRDPARTY DOCKER IMAGE: kafka
x86_64-0.4.6: Pulling from hyperledger/fabric-kafka
==> THIRDPARTY DOCKER IMAGE: zookeeper
x86_64-0.4.6: Pulling from hyperledger/fabric-zookeeper

===> List out hyperledger docker images
hyperledger/fabric-ca          latest              72617b4fa9b4        2 days ago          299MB
hyperledger/fabric-ca          x86_64-1.1.0        72617b4fa9b4        2 days ago          299MB
hyperledger/fabric-tools       latest              b7bfddf508bc        2 days ago          1.46GB
hyperledger/fabric-tools       x86_64-1.1.0        b7bfddf508bc        2 days ago          1.46GB
hyperledger/fabric-orderer     latest              ce0c810df36a        2 days ago          180MB
hyperledger/fabric-orderer     x86_64-1.1.0        ce0c810df36a        2 days ago          180MB
hyperledger/fabric-peer        latest              b023f9be0771        2 days ago          187MB
hyperledger/fabric-peer        x86_64-1.1.0        b023f9be0771        2 days ago          187MB
hyperledger/fabric-javaenv     latest              82098abb1a17        2 days ago          1.52GB
hyperledger/fabric-javaenv     x86_64-1.1.0        82098abb1a17        2 days ago          1.52GB
hyperledger/fabric-ccenv       latest              c8b4909d8d46        2 days ago          1.39GB
hyperledger/fabric-ccenv       x86_64-1.1.0        c8b4909d8d46        2 days ago          1.39GB
hyperledger/fabric-zookeeper   latest              92cbb952b6f8        4 weeks ago         1.39GB
hyperledger/fabric-zookeeper   x86_64-0.4.6        92cbb952b6f8        4 weeks ago         1.39GB
hyperledger/fabric-kafka       latest              554c591b86a8        4 weeks ago         1.4GB
hyperledger/fabric-kafka       x86_64-0.4.6        554c591b86a8        4 weeks ago         1.4GB
hyperledger/fabric-couchdb     latest              7e73c828fc5b        4 weeks ago         1.56GB
hyperledger/fabric-couchdb     x86_64-0.4.6        7e73c828fc5b        4 weeks ago         1.56GB
```

ちなみに Hyperledger の Docker Hub は [ここ](https://hub.docker.com/u/hyperledger/) 。  
`bin` 配下にはコマンドがあるのでパスを通しておく。

```bash
$ ls
bin	config
$ export PATH=`pwd`/bin:$PATH
```

# チュートリアル

以下のチュートリアルサイトを実施する。

- https://hyperledger-fabric.readthedocs.io/en/release-1.1/tutorials.html

## [Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)

[サンプル](https://github.com/hyperledger/fabric-samples) を入手して実行する。

```bash
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ git checkout v1.1.0
$ cd first-network
```

`byfn.sh` というスクリプトがあり、 4 つの Peer （2 つの異なる組織） 、 1 つの Orderer を含む Hyperledger Fabric network を作成するラッパースクリプトになっている。

```bash
$ ./byfn.sh --help
Usage: 
  byfn.sh up|down|restart|generate|upgrade [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-i <imagetag>]
  byfn.sh -h|--help (print this message)
    <mode> - one of 'up', 'down', 'restart' or 'generate'
      - 'up' - bring up the network with docker-compose up
      - 'down' - clear the network with docker-compose down
      - 'restart' - restart the network
      - 'generate' - generate required certificates and genesis block
      - 'upgrade'  - upgrade the network from v1.0.x to v1.1
    -c <channel name> - channel name to use (defaults to "mychannel")
    -t <timeout> - CLI timeout duration in seconds (defaults to 10)
    -d <delay> - delay duration in seconds (defaults to 3)
    -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
    -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
    -l <language> - the chaincode language: golang (default) or node
    -i <imagetag> - the tag to be used to launch the network (defaults to "latest")

Typically, one would first generate the required certificates and 
genesis block, then bring up the network. e.g.:

	byfn.sh generate -c mychannel
	byfn.sh up -c mychannel -s couchdb
        byfn.sh up -c mychannel -s couchdb -i 1.1.0-alpha
	byfn.sh up -l node
	byfn.sh down -c mychannel
        byfn.sh upgrade -c mychannel

Taking all defaults:
	byfn.sh generate
	byfn.sh up
	byfn.sh down
```

サンプル実行の流れは以下。

1. ネットワークの生成 / Generate Network Artifacts
2. ネットワークの起動 / Bring Up the Network
3. ネットワークの停止 / Bring Down the Network
4. クエリの発行

## ネットワークの生成

以下のコマンドを実行する。（ログは一部省略している）

```bash
$ ./byfn.sh -m generate

Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y

# Generate certificates using cryptogen tool
+ cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
+ res=0
+ set +x

#  Generating Orderer Genesis block
+ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
2018-03-18 16:35:08.285 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.296 JST [msp] getMspConfig -> INFO 002 Loading NodeOUs
2018-03-18 16:35:08.297 JST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-03-18 16:35:08.297 JST [common/tools/configtxgen] doOutputBlock -> INFO 004 Generating genesis block
2018-03-18 16:35:08.298 JST [common/tools/configtxgen] doOutputBlock -> INFO 005 Writing genesis block
+ res=0
+ set +x

# Generating channel configuration transaction 'channel.tx'
+ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
2018-03-18 16:35:08.319 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.330 JST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-03-18 16:35:08.331 JST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-03-18 16:35:08.331 JST [msp] getMspConfig -> INFO 004 Loading NodeOUs
2018-03-18 16:35:08.354 JST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx
+ res=0
+ set +x

# Generating anchor peer update for Org1MSP
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
2018-03-18 16:35:08.375 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.387 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-03-18 16:35:08.387 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
+ res=0
+ set +x

# Generating anchor peer update for Org2MSP
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
2018-03-18 16:35:08.408 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.420 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-03-18 16:35:08.420 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
+ res=0
+ set +x
```

`mychannel` チャンネルで証明書と **genesis block** を作成する。  
以下のコマンドが実行されている。

1. `cryptogen generate --config=./crypto-config.yaml`
    - Orderer と Peer の証明書を作成
    - [`cryptogen`](https://hyperledger-fabric.readthedocs.io/en/release-1.1/commands/cryptogen-commands.html) は Membership Service Providers(MSP) をコントロールするコマンド
    - `crypto-config.yaml` ファイルで組織のドメイン(MSP ID)を定義
2. `configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block`
    - 2 つの組織に対して Orderer の Genesit Block を作成
    - [`configtxgen`](https://hyperledger-fabric.readthedocs.io/en/release-1.1/commands/configtxgen.html) は artifacts に関連するチャンネル設定の作成と検査を実行するコマンド
3. `configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel`
4. `configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP`
5. `configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP`

`org1.example.com` と `org2.example.com` の 2 つの組織が作られる？

## ネットワークの起動

```bash
$ ./byfn.sh -m up

Starting with channel 'mychannel' and CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
2018-03-18 07:40:40.515 UTC [main] main -> INFO 001 Exiting.....
LOCAL_VERSION=1.1.0
DOCKER_IMAGE_VERSION=1.1.0
Creating network "net_byfn" with the default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating volume "net_peer1.org2.example.com" with default driver
Creating volume "net_peer1.org1.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_orderer.example.com" with default driver
Creating peer1.org2.example.com ... 
Creating peer1.org2.example.com
Creating peer0.org1.example.com ... 
Creating peer1.org1.example.com ... 
Creating orderer.example.com ... 
Creating peer0.org2.example.com ... 
Creating peer0.org1.example.com
Creating peer1.org1.example.com
Creating peer0.org2.example.com
Creating peer0.org1.example.com ... done
Creating cli ... 
Creating cli ... done

 ____    _____      _      ____    _____ 
/ ___|  |_   _|    / \    |  _ \  |_   _|
\___ \    | |     / _ \   | |_) |   | |  
 ___) |   | |    / ___ \  |  _ <    | |  
|____/    |_|   /_/   \_\ |_| \_\   |_|  

Build your first network (BYFN) end-to-end test

Channel name : mychannel
Creating channel...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org1MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
+ peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
+ set +x
2018-03-18 07:40:44.590 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-03-18 07:40:44.638 UTC [channelCmd] InitCmdFactory -> INFO 002 Endorser and orderer connections initialized
2018-03-18 07:40:44.842 UTC [main] main -> INFO 003 Exiting.....
===================== Channel "mychannel" is created successfully ===================== 

Having all peers join the channel...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org1MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
+ peer channel join -b mychannel.block
+ res=0
+ set +x
2018-03-18 07:40:44.928 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-03-18 07:40:44.978 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-03-18 07:40:44.978 UTC [main] main -> INFO 003 Exiting.....
===================== peer0.org1 joined on the channel "mychannel" ===================== 

CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org1MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer1.org1.example.com:7051
+ peer channel join -b mychannel.block
+ res=0
+ set +x
2018-03-18 07:40:48.068 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-03-18 07:40:48.120 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-03-18 07:40:48.121 UTC [main] main -> INFO 003 Exiting.....
===================== peer1.org1 joined on the channel "mychannel" ===================== 

CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org2MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
+ peer channel join -b mychannel.block
+ res=0
+ set +x
2018-03-18 07:40:51.217 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-03-18 07:40:51.263 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-03-18 07:40:51.263 UTC [main] main -> INFO 003 Exiting.....
===================== peer0.org2 joined on the channel "mychannel" ===================== 

CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org2MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer1.org2.example.com:7051
+ peer channel join -b mychannel.block
+ res=0
+ set +x
2018-03-18 07:40:54.353 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-03-18 07:40:54.408 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-03-18 07:40:54.408 UTC [main] main -> INFO 003 Exiting.....
===================== peer1.org2 joined on the channel "mychannel" ===================== 

Updating anchor peers for org1...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org1MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
+ peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
+ set +x
2018-03-18 07:40:57.490 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-03-18 07:40:57.510 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
2018-03-18 07:40:57.510 UTC [main] main -> INFO 003 Exiting.....
===================== Anchor peers for org "Org1MSP" on "mychannel" is updated successfully ===================== 

Updating anchor peers for org2...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org2MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
+ peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
+ set +x
2018-03-18 07:41:00.602 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-03-18 07:41:00.616 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
2018-03-18 07:41:00.617 UTC [main] main -> INFO 003 Exiting.....
===================== Anchor peers for org "Org2MSP" on "mychannel" is updated successfully ===================== 

Installing chaincode on peer0.org1...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org1MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
+ peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
+ res=0
+ set +x
2018-03-18 07:41:03.716 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-03-18 07:41:03.716 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-03-18 07:41:04.348 UTC [main] main -> INFO 003 Exiting.....
===================== Chaincode is installed on peer0.org1 ===================== 

Install chaincode on peer0.org2...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org2MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
+ peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
+ res=0
+ set +x
2018-03-18 07:41:04.493 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-03-18 07:41:04.493 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-03-18 07:41:04.686 UTC [main] main -> INFO 003 Exiting.....
===================== Chaincode is installed on peer0.org2 ===================== 

Instantiating chaincode on peer0.org2...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org2MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
+ peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P 'OR	('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
+ res=0
+ set +x
2018-03-18 07:41:04.770 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-03-18 07:41:04.770 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-03-18 07:41:33.573 UTC [main] main -> INFO 003 Exiting.....
===================== Chaincode Instantiation on peer0.org2 on channel 'mychannel' is successful ===================== 

Querying chaincode on peer0.org1...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org1MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
===================== Querying on peer0.org1 on channel 'mychannel'... ===================== 
Attempting to Query peer0.org1 ...3 secs
+ peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
+ res=0
+ set +x

2018-03-18 07:41:36.715 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-03-18 07:41:36.715 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 100
2018-03-18 07:41:58.903 UTC [main] main -> INFO 003 Exiting.....
===================== Query on peer0.org1 on channel 'mychannel' is successful ===================== 
Sending invoke transaction on peer0.org1...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org1MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
+ peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
+ res=0
+ set +x
2018-03-18 07:41:59.155 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-03-18 07:41:59.155 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-03-18 07:41:59.203 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-03-18 07:41:59.203 UTC [main] main -> INFO 004 Exiting.....
===================== Invoke transaction on peer0.org1 on channel 'mychannel' is successful ===================== 

Installing chaincode on peer1.org2...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org2MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer1.org2.example.com:7051
+ peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
+ res=0
+ set +x
2018-03-18 07:41:59.371 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-03-18 07:41:59.371 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-03-18 07:42:00.022 UTC [main] main -> INFO 003 Exiting.....
===================== Chaincode is installed on peer1.org2 ===================== 

Querying chaincode on peer1.org2...
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org2MSP
CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ID=cli
CORE_LOGGING_LEVEL=INFO
CORE_PEER_ADDRESS=peer1.org2.example.com:7051
===================== Querying on peer1.org2 on channel 'mychannel'... ===================== 
Attempting to Query peer1.org2 ...3 secs
+ peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
+ res=0
+ set +x

2018-03-18 07:42:03.120 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-03-18 07:42:03.120 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 90
2018-03-18 07:42:25.987 UTC [main] main -> INFO 003 Exiting.....
===================== Query on peer1.org2 on channel 'mychannel' is successful ===================== 

========= All GOOD, BYFN execution completed =========== 


 _____   _   _   ____   
| ____| | \ | | |  _ \  
|  _|   |  \| | | | | | 
| |___  | |\  | | |_| | 
|_____| |_| \_| |____/  
```

Docker コンテナの起動を確認。

```bash
$ docker ps
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
550a9a7b6c2e        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.a..."   5 minutes ago       Up 5 minutes                                                           dev-peer1.org2.example.com-mycc-1.0
2417b3f56dae        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.a..."   5 minutes ago       Up 5 minutes                                                           dev-peer0.org1.example.com-mycc-1.0
d04699a9f796        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.a..."   5 minutes ago       Up 5 minutes                                                           dev-peer0.org2.example.com-mycc-1.0
5490a6300dcd        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              6 minutes ago       Up 6 minutes                                                           cli
eab001704cfc        hyperledger/fabric-peer:latest                                                                         "peer node start"        6 minutes ago       Up 6 minutes        0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
b9d1d8d067f4        hyperledger/fabric-orderer:latest                                                                      "orderer"                6 minutes ago       Up 6 minutes        0.0.0.0:7050->7050/tcp                             orderer.example.com
993f134e866a        hyperledger/fabric-peer:latest                                                                         "peer node start"        6 minutes ago       Up 6 minutes        0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
a180a341e56d        hyperledger/fabric-peer:latest                                                                         "peer node start"        6 minutes ago       Up 6 minutes        0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
57018d77573f        hyperledger/fabric-peer:latest                                                                         "peer node start"        6 minutes ago       Up 6 minutes        0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
```

1 つの `fabric-tools` 、 4 つの `fabric-peer` 、 1 つの `fabric-orderer` が起動しているのがわかる。

## ネットワークの停止

```bash
$ ./byfn.sh -m down
```

## クエリの発行

```bash
$ 
```

# リクエストの流れ

クライアント->Peer->クライアント->Orderer->Peer 。

1. クライアントは自分が所属するOrganizationのMSPでEnrollし、Peerとの通信に必要な証明書を取得する
2. クライアントはPeerにTransaction Proposalを送る
    - 台帳書き込みを伴う場合はEndorsement policyを満たすよう複数Peerに送信
3. Peerがチェーンコードを実行し、署名とreadset/writesetを生成してクライアントに返す
    - ※Queryの場合は台帳検索結果が返ってくる。台帳書き込みを伴わないのでここで終了
4. クライアントは各Peerからの戻りと元のトランザクションをまとめてOrdererに送る
5. Ordererは複数クライアントからのリクエストをChannel毎に整列してブロックを生成し、Channelに参加しているPeerに配布する
6. 各Peerで下記を確認してから台帳書き込みを実行
    - ブロックに含まれている署名がEndorsement policyを満たしていること
    - readsetのワールドステートのバージョンが最新であること
7. 各Peerが台帳書き込み完了イベントをクライアントに通知

# HyperLedger Fabric(v1.00) におけるトランザクションの理解

https://qiita.com/hakozaki/items/9c7d1e00c4cf486a8f1f

# 応用

## Bringing up a Kafka-based Ordering Service

http://hyperledger-fabric.readthedocs.io/en/latest/kafka.html


# 疑問メモ

## World State DB

State databese はデフォルトで Google の LevelDB （ key value store ）。  
オプションで CouchDB （ document DB / JSON ）へ変更できる。  

If you model assets as JSON and use CouchDB, you can also perform complex rich queries against the chaincode data values, using the CouchDB JSON query language within chaincode.  
チェインコードから直接 CouchDB へリッチなクエリを発行できる？  

[上記に関する公式 Doc](https://hyperledger-fabric.readthedocs.io/en/latest/couchdb_as_state_database.html)

## CouchDB について

REST ベース。

- リビジョン番号
    - ドキュメントは削除されずに追加のみ
    - DocID + RevID （リビジョン番号）で一意
    - DocID のみ指定で最新が取得される

```
{"id":"hoge", "name":"test", "rev":"1-123"}
```

- ドキュメント追加
    - POST は DocID 自動採番
    - PUT は DocID 指定
- 一覧取得
    - `GET /menbers/_all_docs`
    - `_all_docs` は予約語
- **view** とは？
    - クエリを発行したり、レポートを作ったりする機能
    - 1 つの **view 属性** は 1 つの **map** を持ち、オプションで 1 つの **reduce** を持てる
    - いくつかの view 属性が 1 つの **デザイン・ドキュメント** に格納される
    - `{dbName}/_design/{デザイン名}/_view/{view名}`

## Hyperledger Fabric CAのＤＢの中身をみてみる

http://memoyasu.blogspot.jp/2017/11/hyperledger-fabric-ca.html

## Hyperledger Fabric & Composer 環境の導入手順（2018/2月版）

http://dotnsf.blog.jp/tag/blockchain  
http://dotnsf.blog.jp/archives/1069641731.html

## SDK の機能

- fabric client
- fabric-ca client

key-value なので、 key 指定でしか検索できない？

```
Successfully queried  chaincode function: request={"chaincodeID":"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX","fcn":"query","args":["a"]}, value=90
```
