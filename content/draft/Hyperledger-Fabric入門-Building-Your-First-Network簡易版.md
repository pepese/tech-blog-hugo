---
title: Hyperledger Fabric入門 Building Your First Network簡易版
tags:
- ブロックチェーン
- Hyperledger Fabric
id: hyperledger-fabric-basics-building-your-first-network-easy
draft: true
---

[公式サイトのチュートリアル](https://hyperledger-fabric.readthedocs.io/en/release-1.1/tutorials.html)の[Building Your First Network](https://hyperledger-fabric.readthedocs.io/en/release-1.1/build_network.html)を実施する。  
ここでは後述の `byfn.sh` スクリプトを使用して簡易に実施する。

 <!-- more -->

# サンプルの入手

[サンプル](https://github.com/hyperledger/fabric-samples) を入手する。

```bash
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ git checkout v1.1.0
$ cd first-network
```

`byfn.sh` というスクリプトがあり、 4 つの Peer （2 つの異なる組織） 、 1 つの Orderer を含む Hyperledger Fabric network を作成してクエリを発行するラッパースクリプトになっている。

```bash
$ ./byfn.sh --help
# Usage が表示される
```

このスクリプトを利用することで簡易に Hyperledger Fabric の network を作成して、クエリが発行されている様子が見てとれるが詳細は理解できない。  
詳細確認は詳細版の記事を確認のこと。

# 実行

サンプル実行の流れは以下。

1. ネットワークの生成 / Generate Network Artifacts
2. ネットワークの起動 / Bring Up the Network
    - ついでにクエリも発行されている
3. ネットワークの停止 / Bring Down the Network

## 1. ネットワークの生成

以下のコマンドを実行する。（ログは一部省略している）

```bash
$ ./byfn.sh -m generate

Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y

# Generate certificates using cryptogen tool
+ cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com

#  Generating Orderer Genesis block
+ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
2018-03-18 16:35:08.285 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.296 JST [msp] getMspConfig -> INFO 002 Loading NodeOUs
2018-03-18 16:35:08.297 JST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-03-18 16:35:08.297 JST [common/tools/configtxgen] doOutputBlock -> INFO 004 Generating genesis block
2018-03-18 16:35:08.298 JST [common/tools/configtxgen] doOutputBlock -> INFO 005 Writing genesis block

# Generating channel configuration transaction 'channel.tx'
+ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
2018-03-18 16:35:08.319 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.330 JST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-03-18 16:35:08.331 JST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-03-18 16:35:08.331 JST [msp] getMspConfig -> INFO 004 Loading NodeOUs
2018-03-18 16:35:08.354 JST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx

# Generating anchor peer update for Org1MSP
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
2018-03-18 16:35:08.375 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.387 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-03-18 16:35:08.387 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

# Generating anchor peer update for Org2MSP
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
2018-03-18 16:35:08.408 JST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-03-18 16:35:08.420 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-03-18 16:35:08.420 JST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

`mychannel` チャンネルで証明書と **genesis block** を作成する。  
以下のコマンドが実行されている。

1. `cryptogen generate --config=./crypto-config.yaml`
    - [`cryptogen`](https://hyperledger-fabric.readthedocs.io/en/release-1.1/commands/cryptogen-commands.html) は `crypto-config.yaml` ファイルを参照して **Membership Service Providers(MSP)** をコントロールするコマンド
    - 証明書、鍵を作成することができる
    - ここでは、Orderer と Peer の証明書を作成している
2. `configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block`
    - [`configtxgen`](https://hyperledger-fabric.readthedocs.io/en/release-1.1/commands/configtxgen.html) は `configtx.yaml` ファイルを参照して、 各種設定を行うためのトランザクションを作成コマンド
    - `Org1` `Org2` の 2 つの組織に対して Orderer の Genesit Block を作成
3. `configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel`
    - `Org1` `Org2` の 2 つの組織が参加する `mychannel` チャンネルを設定するトランザクション（ `./channel-artifacts/channel.tx` ）を作成する
4. `configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP`
    - `mychannel` 上の `Org1` 組織に **Anchor Peer** を設定するトランザクション（ `./channel-artifacts/Org1MSPanchors.tx` ）を作成する
5. `configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP`
    - `mychannel` 上の `Org2` 組織に **Anchor Peer** を設定するトランザクション（ `./channel-artifacts/Org2MSPanchors.tx` ）を作成する

`org1.example.com` と `org2.example.com` の 2 つの組織が作られる？

## 2.ネットワークの起動

以下のコマンドを実行する。（ログは一部省略している）

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

## 3.ネットワークの停止

```bash
$ ./byfn.sh -m down
```