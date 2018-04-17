---
title: Hyperledger Fabric入門 Building Your First Network詳細版
tags:
- ブロックチェーン
- Hyperledger Fabric
id: hyperledger-fabric-basics-building-your-first-network-detail
---

[公式サイトのチュートリアル](https://hyperledger-fabric.readthedocs.io/en/release-1.1/tutorials.html)の[Building Your First Network](https://hyperledger-fabric.readthedocs.io/en/release-1.1/build_network.html)を実施する。  
ここでは `byfn.sh` スクリプトを使用せず、 Hyperledger Fabric のコマンドを使用して実施する。

 <!-- more -->

# サンプルの入手

省略する。  
簡易版の記事を確認のこと。

# 実行

以下の順で記載する。

1. 証明書・鍵の作成
2. 設定用トランザクションの作成
3. コンテナの作成
4. 各コンテナの設定
5. chaincode の配布
6. chaincode の実行
7. ネットワークの停止

## 1. 証明書・鍵の作成

[`cryptogen`](https://hyperledger-fabric.readthedocs.io/en/release-1.1/commands/cryptogen-commands.html) は `crypto-config.yaml` ファイルを参照して **Membership Service Providers(MSP)** をコントロールするコマンド。  
ここでは、 `cryptogen` コマンドを使用して各種証明書・鍵を作成する。

```bash
$ cryptogen generate --config=./crypto-config.yaml

org1.example.com
org2.example.com
# カレントディレクトリに `crypto-config` ディレクトリが作成される
# 証明書と鍵が出力されている
# Orderer には「example.com」単位で、 Peer には「org1.example.com」「org2.example.com」単位で作成される
```

上記のコマンドを実行することで `crypto-config` ディレクトリ配下に以下のように各種証明書・鍵が作成される。

```bash
./crypto-config
├── ordererOrganizations
│   └── example.com
│       ├── ca
│       │   ├── 6594c6e88ed6c899209184f8671badb8c2473b4c2604d39e2f9b01011c0554b9_sk
│       │   └── ca.example.com-cert.pem
│       ├── msp
│       │   ├── admincerts
│       │   │   └── Admin@example.com-cert.pem
│       │   ├── cacerts
│       │   │   └── ca.example.com-cert.pem
│       │   └── tlscacerts
│       │       └── tlsca.example.com-cert.pem
│       ├── orderers
│       │   └── orderer.example.com
│       │       ├── msp
│       │       │   ├── admincerts
│       │       │   │   └── Admin@example.com-cert.pem
│       │       │   ├── cacerts
│       │       │   │   └── ca.example.com-cert.pem
│       │       │   ├── keystore
│       │       │   │   └── 37672aae00e404fcfa7a9a50228746d3928c55bee536acbeb93ea323dfa4a9e9_sk
│       │       │   ├── signcerts
│       │       │   │   └── orderer.example.com-cert.pem
│       │       │   └── tlscacerts
│       │       │       └── tlsca.example.com-cert.pem
│       │       └── tls
│       │           ├── ca.crt
│       │           ├── server.crt
│       │           └── server.key
│       ├── tlsca
│       │   ├── a82e87bd4a20b1a1475d61c9c94eec900e1916bf56f59bbcd46ec738f026613d_sk
│       │   └── tlsca.example.com-cert.pem
│       └── users
│           └── Admin@example.com
│               ├── msp
│               │   ├── admincerts
│               │   │   └── Admin@example.com-cert.pem
│               │   ├── cacerts
│               │   │   └── ca.example.com-cert.pem
│               │   ├── keystore
│               │   │   └── 38931bf0da00a4424d92b12a8384eb759b951d01dad84da7249fecebbc60cfa7_sk
│               │   ├── signcerts
│               │   │   └── Admin@example.com-cert.pem
│               │   └── tlscacerts
│               │       └── tlsca.example.com-cert.pem
│               └── tls
│                   ├── ca.crt
│                   ├── client.crt
│                   └── client.key
└── peerOrganizations
    ├── org1.example.com
    │   ├── ca
    │   │   ├── 61d58e75199a11337f0da5ec4db8728380a84735eb2b08613e26cd9986589b70_sk
    │   │   └── ca.org1.example.com-cert.pem
    │   ├── msp
    │   │   ├── admincerts
    │   │   │   └── Admin@org1.example.com-cert.pem
    │   │   ├── cacerts
    │   │   │   └── ca.org1.example.com-cert.pem
    │   │   ├── config.yaml
    │   │   └── tlscacerts
    │   │       └── tlsca.org1.example.com-cert.pem
    │   ├── peers
    │   │   ├── peer0.org1.example.com
    │   │   │   ├── msp
    │   │   │   │   ├── admincerts
    │   │   │   │   │   └── Admin@org1.example.com-cert.pem
    │   │   │   │   ├── cacerts
    │   │   │   │   │   └── ca.org1.example.com-cert.pem
    │   │   │   │   ├── config.yaml
    │   │   │   │   ├── keystore
    │   │   │   │   │   └── 41cac932bf09365c5b4331f4c61d53f9bf522d5badde017d0b62657e2c49bb56_sk
    │   │   │   │   ├── signcerts
    │   │   │   │   │   └── peer0.org1.example.com-cert.pem
    │   │   │   │   └── tlscacerts
    │   │   │   │       └── tlsca.org1.example.com-cert.pem
    │   │   │   └── tls
    │   │   │       ├── ca.crt
    │   │   │       ├── server.crt
    │   │   │       └── server.key
    │   │   └── peer1.org1.example.com
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   │   └── Admin@org1.example.com-cert.pem
    │   │       │   ├── cacerts
    │   │       │   │   └── ca.org1.example.com-cert.pem
    │   │       │   ├── config.yaml
    │   │       │   ├── keystore
    │   │       │   │   └── 2497443163abf92685cab8c124b4d7e59a374f8d90f521e4b13714ca5eadc83b_sk
    │   │       │   ├── signcerts
    │   │       │   │   └── peer1.org1.example.com-cert.pem
    │   │       │   └── tlscacerts
    │   │       │       └── tlsca.org1.example.com-cert.pem
    │   │       └── tls
    │   │           ├── ca.crt
    │   │           ├── server.crt
    │   │           └── server.key
    │   ├── tlsca
    │   │   ├── 9ddd1dd2be0f2a376f700fa27847ee5165ea4320533ac2707f67ba9989dd62c2_sk
    │   │   └── tlsca.org1.example.com-cert.pem
    │   └── users
    │       ├── Admin@org1.example.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── Admin@org1.example.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.org1.example.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── 137ba18e6a54f935b6bfb86b3b9d396eb171bf10e15cd9458e50d5ab53aa1fc7_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── Admin@org1.example.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.org1.example.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       └── User1@org1.example.com
    │           ├── msp
    │           │   ├── admincerts
    │           │   │   └── User1@org1.example.com-cert.pem
    │           │   ├── cacerts
    │           │   │   └── ca.org1.example.com-cert.pem
    │           │   ├── keystore
    │           │   │   └── b22a85c54d4c77309d8b66320234a60c6d9cc30651b7da8a23ff6667852e7bec_sk
    │           │   ├── signcerts
    │           │   │   └── User1@org1.example.com-cert.pem
    │           │   └── tlscacerts
    │           │       └── tlsca.org1.example.com-cert.pem
    │           └── tls
    │               ├── ca.crt
    │               ├── client.crt
    │               └── client.key
    └── org2.example.com
        ├── ca
        │   ├── 60e21383946a8a147a4ffdb2d62caa47672826aa6efcddb0414fdc39e3a32bbf_sk
        │   └── ca.org2.example.com-cert.pem
        ├── msp
        │   ├── admincerts
        │   │   └── Admin@org2.example.com-cert.pem
        │   ├── cacerts
        │   │   └── ca.org2.example.com-cert.pem
        │   ├── config.yaml
        │   └── tlscacerts
        │       └── tlsca.org2.example.com-cert.pem
        ├── peers
        │   ├── peer0.org2.example.com
        │   │   ├── msp
        │   │   │   ├── admincerts
        │   │   │   │   └── Admin@org2.example.com-cert.pem
        │   │   │   ├── cacerts
        │   │   │   │   └── ca.org2.example.com-cert.pem
        │   │   │   ├── config.yaml
        │   │   │   ├── keystore
        │   │   │   │   └── 682ff8f0f8ccbd0474c10b43c9898a12c75f82878062cb06797fa1adf50965ec_sk
        │   │   │   ├── signcerts
        │   │   │   │   └── peer0.org2.example.com-cert.pem
        │   │   │   └── tlscacerts
        │   │   │       └── tlsca.org2.example.com-cert.pem
        │   │   └── tls
        │   │       ├── ca.crt
        │   │       ├── server.crt
        │   │       └── server.key
        │   └── peer1.org2.example.com
        │       ├── msp
        │       │   ├── admincerts
        │       │   │   └── Admin@org2.example.com-cert.pem
        │       │   ├── cacerts
        │       │   │   └── ca.org2.example.com-cert.pem
        │       │   ├── config.yaml
        │       │   ├── keystore
        │       │   │   └── 7ba445bb224849b5e067f314158912335222c47db4bb9ee02ddd4e9133a97c94_sk
        │       │   ├── signcerts
        │       │   │   └── peer1.org2.example.com-cert.pem
        │       │   └── tlscacerts
        │       │       └── tlsca.org2.example.com-cert.pem
        │       └── tls
        │           ├── ca.crt
        │           ├── server.crt
        │           └── server.key
        ├── tlsca
        │   ├── 86ba2dfd3cf06b5aa75c1f9a4229eb72786cacb992cdcde1b966227bc0461551_sk
        │   └── tlsca.org2.example.com-cert.pem
        └── users
            ├── Admin@org2.example.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   │   └── Admin@org2.example.com-cert.pem
            │   │   ├── cacerts
            │   │   │   └── ca.org2.example.com-cert.pem
            │   │   ├── keystore
            │   │   │   └── 7c93795f786c9c2a0e9b07aeb594492fe8e504c656371b71fb14debc3f946560_sk
            │   │   ├── signcerts
            │   │   │   └── Admin@org2.example.com-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.org2.example.com-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── client.crt
            │       └── client.key
            └── User1@org2.example.com
                ├── msp
                │   ├── admincerts
                │   │   └── User1@org2.example.com-cert.pem
                │   ├── cacerts
                │   │   └── ca.org2.example.com-cert.pem
                │   ├── keystore
                │   │   └── 39fb41c4c90482de2675c7225d2a54843045ae3efa161aa1cf42fa5a06c93b43_sk
                │   ├── signcerts
                │   │   └── User1@org2.example.com-cert.pem
                │   └── tlscacerts
                │       └── tlsca.org2.example.com-cert.pem
                └── tls
                    ├── ca.crt
                    ├── client.crt
                    └── client.key
```

Orderer 、 Peer の組織・ドメイン・ホスト単位で証明書・鍵が作成されているのがわかる。

## 2. 設定用トランザクションの作成

[`configtxgen`](https://hyperledger-fabric.readthedocs.io/en/release-1.1/commands/configtxgen.html) は `configtx.yaml` ファイルを参照して、 各種設定を行うためのトランザクションファイルを作成コマンド。  
ここでは、 **Genesis Block** の作成、 `mychannel` チャンネルの設定、 `Org1` の **Anchor Peer** を設定、 `Org2` の Anchor Peer を設定するトランザクションファイルを作成する。

```bash
$ export FABRIC_CFG_PATH=`pwd`
# `configtx.yaml` の位置を知らせるために上記の環境変数を設定

$ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

2018-04-16 19:18:20.642 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-04-16 19:18:20.653 JST [msp] getMspConfig -> INFO 002 Loading NodeOUs
2018-04-16 19:18:20.653 JST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-04-16 19:18:20.653 JST [common/tools/configtxgen] doOutputBlock -> INFO 004 Generating genesis block
2018-04-16 19:18:20.655 JST [common/tools/configtxgen] doOutputBlock -> INFO 005 Writing genesis block
# 上記で `channel-artifacts` ディレクトリ配下に Orderer の genesis block が作成される

$ export CHANNEL_NAME=mychannel
# 環境変数にチャンネル名を登録する

$ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

2018-04-16 19:24:13.431 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-04-16 19:24:13.440 JST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-04-16 19:24:13.441 JST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-04-16 19:24:13.441 JST [msp] getMspConfig -> INFO 004 Loading NodeOUs
2018-04-16 19:24:13.466 JST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx
# 上記で `channel-artifacts` ディレクトリ配下に mychannel の channel transaction artifact が作成される（ `./channel-artifacts/channel.tx` ）

$ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

2018-04-16 19:28:21.391 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-04-16 19:28:21.401 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-04-16 19:28:21.401 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
# 上記で、 mychannel 上の Org1 の anchor peer を定義するトランザクションを作成する（ `./channel-artifacts/Org1MSPanchors.tx` ）

$ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

2018-04-16 19:32:11.947 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-04-16 19:32:11.956 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-04-16 19:32:11.956 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
# 上記で、 mychannel 上の Org2 の anchor peer を定義するトランザクションを作成する（ `./channel-artifacts/Org2MSPanchors.tx` ）
```

## 3. コンテナの作成

**Docker Compose** を使用してコンテナを作成する。

```bash
$ docker-compose -f docker-compose-cli.yaml up -d

Creating network "net_byfn" with the default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating volume "net_peer1.org2.example.com" with default driver
Creating volume "net_peer1.org1.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_orderer.example.com" with default driver
Creating orderer.example.com ... 
Creating peer1.org1.example.com ... 
Creating peer1.org2.example.com ... 
Creating peer0.org1.example.com ... 
Creating peer0.org2.example.com ... 
Creating peer1.org1.example.com
Creating orderer.example.com
Creating peer1.org2.example.com
Creating peer0.org2.example.com
Creating peer0.org1.example.com ... done
Creating cli ... 
Creating cli ... done
# 上記で Docker Compose が `./channel-artifacts/genesis.block` を参照・使用して `byfn` ネットワーク上に以下のコンテナを構築する
# orderer.example.com
# peer0.org1.example.com
# peer1.org1.example.com
# peer0.org2.example.com
# peer1.org2.example.com
# cli
## cliコンテナは、1000秒間アイドル状態に留まります。 
## 必要なときに消えた場合は、簡単なコマンドで再起動できます：
## $ docker start cli
```

なお、 `hyperledger/fabric-tools` イメージから作成される `cli` コンテナは、 Orderer/Peer に対して設定用のトランザクションやクエリを発行する際に使用する。

## 4. 各コンテナの設定

`cli` コンテナにログインして各種設定用トランザクションを発行する。

```bash
$ docker exec -it cli bash
# cli コンテナへログイン
# ログインすると以下のコマンドラインが以下のようになる
# root@a96cfc7bb90b:/opt/gopath/src/github.com/hyperledger/fabric/peer#
# ここでは省略のため cli へログイン中は「 @cli$ 」と記載する

@cli$ export CHANNEL_NAME=mychannel
# チャンネル名を環境変数へ設定

@cli$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2018-04-17 12:01:56.035 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-04-17 12:01:56.078 UTC [channelCmd] InitCmdFactory -> INFO 002 Endorser and orderer connections initialized
2018-04-17 12:01:56.282 UTC [main] main -> INFO 003 Exiting.....
# Orderer に対して Orderer の証明書を用いて mychannel を作成する命令を発行
# このコマンドを実行すると `mychannel` が作成され、`<channel-ID.block>` の形式で genesis block が返却される（ここでは `mychannel.block` ）
# このブロックには設定情報が定義された `channel.tx` が含まれている

@cli$ peer channel join -b mychannel.block

2018-04-17 12:04:54.043 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-04-17 12:04:54.089 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-04-17 12:04:54.090 UTC [main] main -> INFO 003 Exiting.....
# このコマンドで `peer0.org1.example.com` を `mychannel` へ参加させる
# 対象などをオプションで指定していないが、これは以下の環境変数が登録されているため。
## CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
## CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
## CORE_PEER_LOCALMSPID=Org1MSP
## CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
## CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
## CORE_PEER_ADDRESS=peer0.org1.example.com:7051
# この環境変数を変更することで他の Peer を登録できる

@cli$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org1MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp CORE_PEER_ADDRESS=peer1.org1.example.com:7051 peer channel join -b mychannel.block

2018-04-17 12:16:04.771 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-04-17 12:16:04.814 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-04-17 12:16:04.814 UTC [main] main -> INFO 003 Exiting.....
# 上記は環境変数を指定しつつコマンドを実行して `peer1.org1.example.com` を mychannel へ参加させている。

@cli$ 
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 peer channel join -b mychannel.block

2018-04-17 12:19:24.602 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-04-17 12:19:24.650 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-04-17 12:19:24.650 UTC [main] main -> INFO 003 Exiting.....
# 上記は環境変数を指定しつつコマンドを実行して `peer0.org2.example.com` を mychannel へ参加させている。

@cli$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer1.org2.example.com:7051 peer channel join -b mychannel.block

2018-04-17 12:22:55.622 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-04-17 12:22:55.675 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-04-17 12:22:55.675 UTC [main] main -> INFO 003 Exiting.....
# 上記は環境変数を指定しつつコマンドを実行して `peer1.org2.example.com` を mychannel へ参加させている。

@cli$ peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2018-04-17 12:24:52.771 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-04-17 12:24:52.785 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
2018-04-17 12:24:52.785 UTC [main] main -> INFO 003 Exiting.....
# 上記で `peer0.org1.example.com` を `mychannel` 上の `Org1` の Anchor Peer に設定する
# ここでも以下の環境変数があることにより `peer0.org1.example.com` が Anchor Peer に設定される
## CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
## CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
## CORE_PEER_LOCALMSPID=Org1MSP
## CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
## CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
## CORE_PEER_ADDRESS=peer0.org1.example.com:7051

@cli$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2018-04-17 12:28:51.379 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-04-17 12:28:51.392 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
2018-04-17 12:28:51.392 UTC [main] main -> INFO 003 Exiting.....
# 上記は環境変数を指定しつつコマンドを実行して `peer0.org2.example.com` を Anchor Peer へ変更させている。
```

## 5. chaincode の配布

Peer への chaincode の配布は以下。

```bash
@cli$ peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

2018-04-17 12:33:12.393 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-17 12:33:12.393 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-17 12:33:12.947 UTC [main] main -> INFO 003 Exiting.....
# 上記は Golang で実装した chaincode を `peer0.org1.example.com` へ配布する

@cli$ peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"

2018-04-17 12:33:39.969 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-17 12:33:39.970 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-17 12:33:58.287 UTC [main] main -> INFO 003 Exiting.....
# 上記のコマンドで配布した chaincode をインスタンス化する
# `-P "OR ('Org1MSP.peer','Org2MSP.peer')"` の部分で endorsement（承認）のポリシーを設定している
# Org1 か Org2 のどちらかが OK すればよい（ AND も指定できる）
# また、 `a` の値を 100、 `b` の値を 200 に設定している
```

ここでは、 `peer0.org1.example.com` にのみしか chaincode の配布・インスタンス化を実施していないが、 `peer1.org1.example.com` `peer0.org2.example.com` `peer1.org2.example.com` にも必要か要確認。  
ちなみに試しに他に投げてみたが、実行できなかった。。。？

上記は Go の chaincode を配布・インスタンス化したが、 Node の場合は以下。

```bash
@cli$ peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
# `-l node` を指定することで Node で実装した chaincode を配布する

@cli$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"
# Node の場合のインスタンス化
```

## 6. chaincode の実行

chaincode の実行は以下。

```bash
@cli$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

2018-04-17 12:34:35.362 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-17 12:34:35.362 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 100
2018-04-17 12:34:35.367 UTC [main] main -> INFO 003 Exiting.....
# `a` の値を参照して、 100 が得られている

@cli$ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

2018-04-17 12:35:00.231 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-17 12:35:00.231 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-17 12:35:00.238 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-04-17 12:35:00.239 UTC [main] main -> INFO 004 Exiting.....
# a から b へ 10 送金する

@cli$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

2018-04-17 12:35:28.617 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-17 12:35:28.617 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 90
2018-04-17 12:35:28.622 UTC [main] main -> INFO 003 Exiting.....
# a の値が 100 から 90 になっているのがわかる

@cli$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","b"]}'

2018-04-17 12:36:04.170 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-17 12:36:04.170 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 210
2018-04-17 12:36:04.174 UTC [main] main -> INFO 003 Exiting.....
# b の値が 200 から 210 になっているのがわかる
```


続きは以下。

- http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#what-s-happening-behind-the-scenes

なお、ネットワークの停止は `byfn.sh` でやる。

```bash
$ ./byfn.sh -m down
```