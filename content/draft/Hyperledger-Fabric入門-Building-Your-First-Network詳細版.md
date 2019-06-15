---
title: Hyperledger Fabric入門 Building Your First Network詳細版
tags:
- ブロックチェーン
- Hyperledger Fabric
id: hyperledger-fabric-basics-building-your-first-network-detail
draft: true
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
7. ログの確認

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

$ docker ps -a
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS              PORTS                                              NAMES
2c3a85d77efc        hyperledger/fabric-tools:latest     "/bin/bash"         11 seconds ago      Up 10 seconds                                                          cli
17f625da35f1        hyperledger/fabric-peer:latest      "peer node start"   12 seconds ago      Up 11 seconds       0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
a234851b44c9        hyperledger/fabric-orderer:latest   "orderer"           12 seconds ago      Up 10 seconds       0.0.0.0:7050->7050/tcp                             orderer.example.com
a8dafc97e0b6        hyperledger/fabric-peer:latest      "peer node start"   12 seconds ago      Up 11 seconds       0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
6593d9461ad5        hyperledger/fabric-peer:latest      "peer node start"   12 seconds ago      Up 10 seconds       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
95a680a1ac84        hyperledger/fabric-peer:latest      "peer node start"   12 seconds ago      Up 11 seconds       0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
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

2018-04-18 12:08:41.998 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:08:41.998 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-18 12:08:42.608 UTC [main] main -> INFO 003 Exiting.....
# 上記は Golang で実装した chaincode を `peer0.org1.example.com` へ配布する

@cli$ 
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/

2018-04-18 12:11:58.600 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:11:58.600 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-18 12:11:58.803 UTC [main] main -> INFO 003 Exiting.....
# 環境変数を指定して Golang で実装した chaincode を `peer0.org2.example.com` へ配布する


@cli$
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"

2018-04-18 12:14:19.776 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:14:19.776 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-18 12:14:37.943 UTC [main] main -> INFO 003 Exiting.....
# 上記のコマンドで chaincode を配布した `peer0.org2.example.com` の chaincode をインスタンス化する
# `-P "OR ('Org1MSP.peer','Org2MSP.peer')"` の部分で endorsement（承認）のポリシーを設定している
# Org1 か Org2 のどちらかが OK すればよい（ AND も指定できる）
# また、 `a` の値を 100、 `b` の値を 200 に設定している

# `peer0.org1.example.com` の chaincode はインスタンス化していないが chaincode を配布済みのため実行することができる
# ただし、 `peer1.org1.example.com` `peer1.org2.example.com` には chaincode を配布していないので実行することができない

# この時点で以下のコンテナが出現する
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
934ccf07c094        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.a..."   13 seconds ago      Up 13 seconds                                                          dev-peer0.org2.example.com-mycc-1.0
```

`mycc` は chaincode 名。  
chaincode はチャンネル毎にインスタンス化する必要あり。

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

2018-04-18 12:20:57.898 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:20:57.898 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 100
2018-04-18 12:21:15.610 UTC [main] main -> INFO 003 Exiting.....
# `a` の値を参照して、 100 が得られている
# また、上記は `peer0.org1.example.com` に対してクエリを発行している
# インスタンス化したのは `peer0.org2.example.com` だが、実行できることがわかる
# chaincode を配布していない `peer1.org1.example.com` `peer1.org2.example.com` へクエリを発行してもエラーとなる

# この時点で以下のコンテナが出現する
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
46a7b270dab3        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.a..."   11 seconds ago      Up 10 seconds                                                          dev-peer0.org1.example.com-mycc-1.0

@cli$ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

2018-04-18 12:22:06.706 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:22:06.708 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-18 12:22:06.721 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-04-18 12:22:06.722 UTC [main] main -> INFO 004 Exiting.....
# Invoke して a から b へ 10 送金する
# なお、 `peer0.org1.example.com` へ送信している

@cli$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

2018-04-18 12:23:21.070 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:23:21.070 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 90
2018-04-18 12:23:21.075 UTC [main] main -> INFO 003 Exiting.....
# a の値が 100 から 90 になっているのがわかる
# なお、 `peer0.org1.example.com` へ送信している

@cli$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","b"]}'

2018-04-18 12:24:00.126 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:24:00.126 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 210
2018-04-18 12:24:00.130 UTC [main] main -> INFO 003 Exiting.....
# b の値が 200 から 210 になっているのがわかる
# なお、 `peer0.org1.example.com` へ送信している

@cli$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

2018-04-18 12:25:33.163 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:25:33.164 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 90
2018-04-18 12:25:33.169 UTC [main] main -> INFO 003 Exiting.....
# `peer0.org2.example.com` へクエリを発行しても同様に a = 90 が得られる

@cli$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer1.org2.example.com:7051 peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

2018-04-18 12:27:15.236 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:27:15.236 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Error: Error endorsing query: rpc error: code = Unknown desc = cannot retrieve package for chaincode mycc/1.0, error open /var/hyperledger/production/chaincodes/mycc.1.0: no such file or directory - <nil>
# chaincode が配布されていない `peer1.org2.example.com` へクエリを発行してもエラーが発生する

@cli$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer1.org2.example.com:7051 peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/

2018-04-18 12:29:07.613 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:29:07.613 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-18 12:29:07.980 UTC [main] main -> INFO 003 Exiting.....
# `peer1.org2.example.com` へ chaincode を配布する
# インスタンス化は行なっていない

@cli$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer1.org2.example.com:7051 peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

2018-04-18 12:30:17.055 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-18 12:30:17.055 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 90
2018-04-18 12:30:34.774 UTC [main] main -> INFO 003 Exiting.....
# `peer1.org2.example.com` へ chaincode を配布してからクエリを実行すると正しく結果が返却されたことがわかる

# この時点で以下のコンテナが出現する
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
6100864a7cb5        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.a..."   9 seconds ago       Up 9 seconds                                                           dev-peer1.org2.example.com-mycc-1.0
```

## 7. ログの確認

```bash
@cli$ exit
$ docker logs -f cli # 簡易版じゃないと見えれないくさい
```

## 疑問

```bash
$ docker ps
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
f737b18f1dc9        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.a..."   4 minutes ago       Up 4 minutes                                                           dev-peer1.org2.example.com-mycc-1.0
69c887024bf1        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.a..."   13 minutes ago      Up 13 minutes                                                          dev-peer0.org1.example.com-mycc-1.0
e7942020176e        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.a..."   20 minutes ago      Up 20 minutes                                                          dev-peer0.org2.example.com-mycc-1.0
427e97f7d0aa        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              28 minutes ago      Up 28 minutes                                                          cli
3db8f43d05b2        hyperledger/fabric-peer:latest                                                                         "peer node start"        28 minutes ago      Up 28 minutes       0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
50c425aae9ae        hyperledger/fabric-peer:latest                                                                         "peer node start"        28 minutes ago      Up 28 minutes       0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
70445348149b        hyperledger/fabric-orderer:latest                                                                      "orderer"                28 minutes ago      Up 28 minutes       0.0.0.0:7050->7050/tcp                             orderer.example.com
b390d5194441        hyperledger/fabric-peer:latest                                                                         "peer node start"        28 minutes ago      Up 28 minutes       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
ea37514af045        hyperledger/fabric-peer:latest                                                                         "peer node start"        28 minutes ago      Up 28 minutes       0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
```

`dev-peer1.org2.example.com-mycc-1.0` とかいうコンテナが出現している理由がわからない。。。
```
The chaincode is then “instantiated” on peer0.org2.example.com. Instantiation adds the chaincode to the channel, starts the container for the target peer, and initializes the key value pairs associated with the chaincode. The initial values for this example are [“a”,”100” “b”,”200”]. This “instantiation” results in a container by the name of dev-peer0.org2.example.com-mycc-1.0 starting.
```
これってもしかして **LevelDB** プロセス？


# CouchDB でやってみる

## 1. 証明書・鍵の作成

同様。

## 2. 設定用トランザクションの作成

同様。

## 3. コンテナの作成

```bash
$ docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d

Creating network "net_byfn" with the default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating volume "net_peer1.org2.example.com" with default driver
Creating volume "net_peer1.org1.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_orderer.example.com" with default driver
Creating couchdb2 ... 
Creating orderer.example.com ... 
Creating orderer.example.com
Creating couchdb3 ... 
Creating couchdb0 ... 
Creating couchdb1 ... 
Creating couchdb2
Creating couchdb0
Creating couchdb3
Creating couchdb2 ... done
Creating peer0.org2.example.com ... 
Creating couchdb0 ... done
Creating peer0.org1.example.com ... 
Creating couchdb3 ... done
Creating peer1.org2.example.com ... 
Creating peer1.org1.example.com ... 
Creating peer1.org2.example.com
Creating peer1.org1.example.com ... done
Creating cli ... 
Creating cli ... done

$ docker ps -a
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS              PORTS                                              NAMES
f6421edf9e70        hyperledger/fabric-tools:latest     "/bin/bash"              31 seconds ago      Up 30 seconds                                                          cli
9a091ca026c2        hyperledger/fabric-peer:latest      "peer node start"        31 seconds ago      Up 31 seconds       0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
38710929e0c7        hyperledger/fabric-peer:latest      "peer node start"        31 seconds ago      Up 31 seconds       0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
e9fc8f117800        hyperledger/fabric-peer:latest      "peer node start"        32 seconds ago      Up 31 seconds       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
3ccf85005fed        hyperledger/fabric-peer:latest      "peer node start"        32 seconds ago      Up 31 seconds       0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
a5c1674156ef        hyperledger/fabric-couchdb          "tini -- /docker-e..."   33 seconds ago      Up 31 seconds       4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp         couchdb3
c164108eafd1        hyperledger/fabric-couchdb          "tini -- /docker-e..."   33 seconds ago      Up 31 seconds       4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp         couchdb1
4bfa549c6242        hyperledger/fabric-couchdb          "tini -- /docker-e..."   33 seconds ago      Up 32 seconds       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp         couchdb0
8dc770e2004f        hyperledger/fabric-couchdb          "tini -- /docker-e..."   33 seconds ago      Up 32 seconds       4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp         couchdb2
20f473476693        hyperledger/fabric-orderer:latest   "orderer"                33 seconds ago      Up 32 seconds       0.0.0.0:7050->7050/tcp                             orderer.example.com
# 4 つの Peer 分の 4 つの CouchDB コンテナが起動する
```

## 4. 各コンテナの設定

同様。

## 5. chaincode の配布

**CouchDB** を使用する場合 CouchDB はドキュメント DB なので、値は JSON 形式である必要がある。  
そのため、先ほどと異なる chaincode `fabric/examples/chaincode/go` を配布する。

```bash
@cli$ peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go

2018-04-19 09:06:57.640 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:06:57.640 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:06:58.093 UTC [main] main -> INFO 003 Exiting.....
# `peer0.org1.example.com` へ chaincode を配布する

@cli$ 
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go

2018-04-19 09:10:25.585 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:10:25.585 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:10:25.882 UTC [main] main -> INFO 003 Exiting.....
# `peer0.org2.example.com` へ chaincode を配布する


@cli$ peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"

2018-04-19 09:11:05.346 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:11:05.346 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:11:24.283 UTC [main] main -> INFO 003 Exiting.....
# `peer0.org1.example.com` へ chaincode のインスタンス化を実行

# この時点で以下のコンテナが生成される
CONTAINER ID        IMAGE                                                                                                     COMMAND                  CREATED             STATUS              PORTS                                              NAMES
f6a6f86513fd        dev-peer0.org1.example.com-marbles-1.0-f57d84df5eaddfab2c22f5896ebc44f52af3829d7f9d8a271b41651a0d4cb353   "chaincode -peer.a..."   48 seconds ago      Up 48 seconds                                                          dev-peer0.org1.example.com-marbles-1.0
```

## 6. chaincode の実行

```bash
@cli$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'

2018-04-19 09:14:18.769 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:14:18.769 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:14:18.781 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-04-19 09:14:18.781 UTC [main] main -> INFO 004 Exiting.....

@cli$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'

2018-04-19 09:14:45.663 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:14:45.663 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:14:45.689 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-04-19 09:14:45.689 UTC [main] main -> INFO 004 Exiting.....

@cli$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'

2018-04-19 09:15:15.529 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:15:15.529 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:15:15.544 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-04-19 09:15:15.545 UTC [main] main -> INFO 004 Exiting.....

@cli$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'

2018-04-19 09:15:51.163 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:15:51.163 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:15:51.173 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-04-19 09:15:51.174 UTC [main] main -> INFO 004 Exiting.....

@cli$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'

2018-04-19 09:16:13.337 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:16:13.337 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:16:13.359 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 payload:"Transferred 2 blue marbles to jerry" 
2018-04-19 09:16:13.360 UTC [main] main -> INFO 004 Exiting.....

@cli$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'

2018-04-19 09:16:38.328 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:16:38.328 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-04-19 09:16:38.341 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
2018-04-19 09:16:38.341 UTC [main] main -> INFO 004 Exiting.....

@cli$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'

2018-04-19 09:19:41.449 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:19:41.449 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}
2018-04-19 09:19:41.458 UTC [main] main -> INFO 003 Exiting.....

@cli$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'

2018-04-19 09:20:15.702 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:20:15.702 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: [{"TxId":"4912fad318b7906a68018736db704a9e502cafe657c29189c9ee9413e51ded7c", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}, "Timestamp":"2018-04-19 09:14:18.77004991 +0000 UTC", "IsDelete":"false"},{"TxId":"0b293d7efb30a129bd0c649054886197cbcbb78cbffb158c0b66d83aad004776", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}, "Timestamp":"2018-04-19 09:16:13.337399502 +0000 UTC", "IsDelete":"false"},{"TxId":"02d23e2976ef00d1c13a64bf1b02e65f386d2c5bfa290dfb210498b540f59c20", "Value":null, "Timestamp":"2018-04-19 09:16:38.328712661 +0000 UTC", "IsDelete":"true"}]
2018-04-19 09:20:15.712 UTC [main] main -> INFO 003 Exiting.....

@cli$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'

2018-04-19 09:20:46.549 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-19 09:20:46.549 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]
2018-04-19 09:20:46.595 UTC [main] main -> INFO 003 Exiting.....
```

## 7. ログの確認

CouchDB のパスは以下？

```
http://localhost:5984/_utils
```

## 8. ネットワークの停止

なお、ネットワークの停止は `byfn.sh` でやる。

```bash
$ ./byfn.sh -m down
```