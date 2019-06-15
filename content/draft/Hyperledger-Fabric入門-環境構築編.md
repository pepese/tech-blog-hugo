---
title: Hyperledger Fabric入門 環境構築編
tags:
- ブロックチェーン
- Hyperledger Fabric
id: hyperledger-fabric-basics-env
draft: true
---

Hyperledger Fabric（ハイパーレッジャー ファブリック）は、 Hyperledger といいう ブロックチェーン技術推進コミュニティ により共同検証された OSS で単に Fabric と呼ばれる。  
The Linux Foundation のプロジェクトの 1 つで Hyperledger Project 。

- [Hyperledger Fabric 1.0 概要](https://www.slideshare.net/Hyperledger_Tokyo/hyperledger-fabric-10)
- [公式サイト](https://www.hyperledger.org/)
- [公式 Github リポジトリ](https://github.com/hyperledger/fabric)
- [公式ドキュメント](http://hyperledger-fabric.readthedocs.io/en/latest/index.html)

ここでは Hyperledger Fabric の特徴などを記載した後、 Mac 上に環境を構築する手順を記載する。

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
`bin` 配下に Hyperledger Fabric をコントロールするコマンドがあるのでパスを通しておく。

```bash
$ ls
bin	config
$ export PATH=`pwd`/bin:$PATH
```