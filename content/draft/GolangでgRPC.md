---
title: GolangでgRPC
tags:
- golang
- go
- gRPC
id: golang-grpc
draft: true
---

[gRPC](https://grpc.io/) は、RPC (Remote Procedure Call) を実現するためにGoogleが開発したプロトコルの1つ。
[Protocol Buffers](https://developers.google.com/protocol-buffers/) を使ってデータをシリアライズ（異なるプログラミング言語間で XML や JSON といったデータフォーマットを介することなる透過的にデータをやり取り）し、HTTP/2 ベース高速な通信を実現できる。
プログラミング言語に依存しない IDL（インターフェース定義言語）を使ってあらかじめAPI仕様を `.proto` ファイルとして定義し、そこからサーバー側＆クライアント側に必要なソースコードのひな形を生成。

- 環境設定
- gRPC
- その他 Tips

<!-- more -->

# 環境設定

ここでは [公式](https://grpc.io/docs/quickstart/go/) とは異なる方法で設定して軽く動確する。

``` bash
$ go get -u google.golang.org/grpc
$ brew install protobuf # protoc コマンドが入る
$ brew install protoc-gen-go # go に対応するコードを出力するためのプラグイン
```

`$GOPATH/src/google.golang.org/grpc/examples` にサンプルあり。  
以下 2 つのターミナルで動かしてみる。

ターミナル１：サーバサイド
``` bash
$ cd $GOPATH/src/google.golang.org/grpc/examples/helloworld
$ go run greeter_server/main.go
```

ターミナル２：クライアントサイド
``` bash
$ cd $GOPATH/src/google.golang.org/grpc/examples/helloworld
$ go run greeter_client/main.go
2019/05/16 16:05:11 Greeting: Hello world
```

`.proto` ファイルからコードを生成してみる。

``` bash
$ cd $GOPATH/src/google.golang.org/grpc/examples/helloworld
$ protoc -I helloworld/ helloworld/helloworld.proto --go_out=plugins=grpc:helloworld
# helloworld/helloworld.pb.go が新しく生成され上書きされてる
```

`protoc <.proto file> --go_out=plugins=grpc:<out dir>` 。

# gRPC

HTTP/2 の stream もサポートしており、gRPCのサポートするRPC方式は以下の通り。

- Unary RPC ：1リクエスト1レスポンス
    - 
    ```
    rpc SayHello(HelloRequest) returns (HelloResponse){
    }
    ```
- Server streaming RPC ：１つのリクエストに複数レスポンス
    - 
    ```
    rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
    }
    ```
- Client streaming RPC ：複数のリクエストに一つのレスポンス
    - 
    ```
    rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
    }
    ```
- Bidirectional streaming RPC ：双方向
    - 
    ```
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
    }
    ```

## 認証

https://grpc.io/docs/guides/auth/

## エラーハンドリングとデバッグ

https://grpc.io/docs/guides/error/

## Protocol Buffers

https://developers.google.com/protocol-buffers/docs/overview

### .proto 3

- [Language Guide (proto3)](https://developers.google.com/protocol-buffers/docs/proto3)
- [和訳](https://qiita.com/CyLomw/items/9aa4551bd6bb9c0818b6)

`service` `rpc` `message` `enum` はアッパーキャメルケースで、 `message` のフィールドはスネークケースで、 `enum` のフィールドはコンスタントケースで記載する。

> #### 変数名の命名規則/xxケース
> - コンスタントケース：全部大文字、アンスコ区切り
> - アッパーキャメルケース：先頭大文字、大文字区切り
> - ローワーキャメルケース：先頭小文字、大文字区切り
> - スネークケース：全部小文字、アンスコ区切り
> - ケバブケース：全部小文字、ハイフン区切り

### Basics: Go

https://developers.google.com/protocol-buffers/docs/gotutorial

### Mappings

- https://github.com/grpc/grpc.github.io/wiki/Mapping
    - gRPC と REST のマッピング
- [Googleが公開しているAPIのスタンダード](https://cloud.google.com/apis/design/standard_methods)

## gRPC Gateway

- https://journal.lampetty.net/entry/implement-restful-api-with-grpc-gateway

# その他 Tips

- [Evans](https://github.com/ktr0731/evans) ： gRPC クライアントツール
- [gRPC から REST API Server をつくる](https://fisproject.jp/2018/09/translates-grpc-into-rest-json-api-with-go/)