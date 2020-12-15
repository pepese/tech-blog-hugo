---
title: "gRPC/Protocol Buffers入門"
date: 2020-12-10T15:14:28+09:00
slug: "grpc-protocolbuffers-basics"
tags:
- gRPC
- Protocol Buffers
draft: false
archives:
- 2020/12
---

今更ながら [gRPC](https://grpc.io/)/[Protocol Buffers](https://developers.google.com/protocol-buffers) に入門する。

<!--more-->

# gRPC/Protocol Buffers 概要

- [gRPC](https://grpc.io/) は Google 製の HTTP/2 を利用した RPC フレームワーク
    - RPC は Remote Procedure Call の略で、直訳すると「遠隔手続き呼び出し」
    - 従来のメソッド呼び出しをネットワーク越しで実行できるくらいのイメージ
- gRPC は [Protocol Buffers](https://developers.google.com/protocol-buffers) を利用してデータをシリアライズし、高速な通信を実現
    - .proto ファイルに IDL(Interface Definition Language) でAPI定義
    - .proto ファイルからサーバ・クライアント双方の雛形コードを自動生成

# gRPC

gRPCに入る前に以下の順で記載していく。

- HTTP/1.x の問題点
- HTTP/2
- gRPCの機能

## HTTP/1.x の問題点

従来の HTTP/1.x には以下の問題点があった。

- コネクション管理の問題（ 3 つのモデル）
    - ① Short-lived ※HTTP/1.0デフォルト
        - リクエストのたびにコネクションを確立し、 1 つレスポンスを受信したらコネクションをクローズ
        - パフォーマンスやリソース消費に問題あり
    - ② Persistent（所謂 Keep-Alive） ※HTTP/1.1のデフォルト
        - コネクションはしばらく開いたままにして、いくつかのリクエストで再利用
        - 1 つのリクエストに対してレスポンスが返却されないと、次のリクエストを送れない、所謂 Head of Line (HOL) Bolocking 問題
    - ③ Pipeline ※HTTP/1.1
        - 1 つのコネクション上で連続してリクエストを送信できるが、レスポンスはリクエストされた順に従い返却される（残 HOL 問題）
        - 実装が難しく不具合のあるサーバやプロキシも多いことから、ほとんどのブラウザでデフォルト無効化
- 通信サイズの問題
    - Webサイトのコンテンツ数増加やサイズ増大に伴い、 1 つのページを表示するのに大量のリクエスト送信が必要になり、コネクション管理とリクエスト送信の問題によりパフォーマンスの問題が顕著に
    - HTTP ヘッダサイズが増大してきた（特に Cookie やアクセストークンが含まれる場合）、圧縮せずにそのままテキストとして送受信するため不要なネットワークトラフィックが発生する

## HTTP/2

HTTP/2 は以下の仕組みにより、先に述べた HTTP/1.x の問題点を解消する。

- **ストリーム**
    - 1つのTCPコネクション上に仮想的な経路を複数作成し、フレームのリクエスト・レスポンスを並列でHOL問題無く処理
    - クライアントはストリーム開始時にサーバにストリームの **優先度付け** が可能で、サーバはストリームに対するリソース割当を変更
    - ストリームごとに受信側が受信可能なデータ量を制御する **フロー制御** が可能で、巨大ファイルダウンロードによるリソース占有を防いだり、アップ・ダウンストリームの回線速度差異抑制を可能に
- **フレーム**
    - テキストではないバイナリ形式のメッセージで、用途に応じてDATAやHEADERSといった10種類のフレームがある
    - HEADERSフレーム（HTTP/1.xのヘッダに相当）は、一連の通信で重複するヘッダ項目の省略やHPACKという方式で **ヘッダ圧縮**
- **サーバプッシュ**
    - クライアントからの1つのリクエストに対してサーバから複数のレスポンスを返せる機能で、例えば、HTML GET リクエストに対してJavaScript・CSSなど複数レスポンスするなど可能

## gRPC の機能

gRPC は先にも記載した通り、HTTP/2 と Protocol Buffers を利用した RPC フレームワーク。  
フレームワークとして、以下のような機能を有する。

- 4 つの RPC
    - ① Unary RPC ：1リクエスト1レスポンス
    - ② Server streaming RPC ：１つのリクエストに複数レスポンス
    - ③ Client streaming RPC ：複数のリクエストに一つのレスポンス
    - ④ Bidirectional streaming RPC ：双方向
- [Interceptor](https://grpc.io/blog/grpc-web-interceptor/)
    - RPC呼出に割り込んで事前・事後処理を実行可能
    - クライアント・サーバサイド双方で利用可能
    - ロギング、認証、エラー処理など
- [認証](https://grpc.io/docs/guides/auth/)
    - ビルドイン
        - SSL/TLS と Token-based authentication with Google
    - 拡張
        - WithPerRPCCredentials：クライアントがサーバーに接続する際、認証情報をメタデータとして送り、サーバーはその情報を認証に用いる
        - WithTransportCredentials：ハンドシェイクを行うような認証方式を実装できる
- [エラーハンドリング](https://grpc.io/docs/guides/error/)
    - エラーコード・エラーメッセージ・エラー詳細からなるエラーモデルが定義されている

# Protocol Buffers

ここでは [proto3](https://developers.google.com/protocol-buffers/docs/proto3) について記載する。

- ヘッダ部分
    - .proto ファイルのヘッダ部分の記載方法
- メッセージ定義
    - .proto ファイルでメッセージを定義する方法
    - 構造体チックに記載できる
- サービス定義
    - .proto ファイルでサービスを定義する方法
    - メッセージをリクエスト・レスポンス（In/Out）にしてメソッドチックに記載できる
- コード生成
    - .proto ファイルからコードを生成する方法

## ヘッダ部分

- `syntax = "proto3";`
    - 先頭行に記載
    - 記載しないと `proto2` 扱いになる
- `import`
    - 他の.proto ファイルをインポートできる
    - `import "other.proto";` : 直接インポートした.proto ファイルのみ参照できる
    - `import public "new.proto";` : public を含む .proto ファイル をインポートした場合はそれも多段インポートで参照できる
- `package`
    - プロトコルメッセージ型の間での名前衝突を避けるために，.proto ファイルに任意で package 指定子を加えることができる
- `option`
    - オプションによって注釈をつけることができる
    - ファイルレベル・メッセージレベル・フィールドレベル・サービスレベル・サービスメソッドレベルなど様々（[オプション](https://developers.google.com/protocol-buffers/docs/proto3#options)）
    - カスタムオプションもサポートする

## メッセージ定義（ **message** ）

```
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3; // コメント
}
```

- 型・フィールド名・フィールド番号 を記載
    - [スカラ値型](https://developers.google.com/protocol-buffers/docs/proto3#scalar)・[列挙型](https://developers.google.com/protocol-buffers/docs/proto3#enum)・複合型 などがある（[デフォルト値](https://developers.google.com/protocol-buffers/docs/proto3#default)）
    - フィールド番号は各フィールドにユニークな数値をタグ付けする
        - シリアライズしてバイナリ化した際にフィールドを区別するために利用される
        - 1 〜 15 までは 1 バイト、16 〜 2047 までは 2 バイトにエンコーディングされるため利用頻度の高いものは 1 〜 15 を採番する（[エンコーディング](https://developers.google.com/protocol-buffers/docs/encoding#structure)）
        - 今後の互換性を保つため、フィールドの削除を行った場合は `reserved` を用いて以降利用を禁止する（未来に同じフィールド名・番号が利用されたらバグる）

```
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

- コメントは `//`
- message 型は多段でネストできる

```
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

- 次のルールを守れば、message 型を更新・変更しても古い proto で生成されたコードも問題なく実行できる
    - 既に存在するフィールド名・番号を削除・再利用してはならない
    - 新規フィールドを追加した場合、古いバイナリはそれを無視する
    - 利用しないフィールドができたとしてもデフォルト値には注意して扱う
    - int32, uint32, int64, uint64, bool は全て互換
    - sint32 と sint64 は互換だが，他の整数型とは互換でない
    - stirng と bytes は，bytes が有効な UTF-8 である場合に限り互換
    - 埋め込まれたメッセージは，もし bytes がそのメッセージのエンコードされたバージョンを含んでいれば，bytes と互換
    - fixed32 は sfixed32 と互換、また、fixed64 と sfixed64 も互換
    - enum は int32, uint32, int64, uint64 と，ワイヤフォーマットの面において互換

### Any 型

```
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

`google.protobuf.Any` は型情報を持ったメッセージとして利用できる。（型定義が事情により出来なくて、送信時に決定したなど）  
プログラミング言語によるが、 `pack()` ・ `unpack()` といったアクセッサメソッドで変換可能。

### oneof

```
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

`oneof` は 「入れ子内のフィールドのうち **最大でどれか 1 つだけ** セットされている」と宣言できる機能。  
`oneof` は `repeated` を使えない。

### map 型

```
map<key_type, value_type> map_field = N;
```

- `key_type`：小数点型と bytes を除く任意のスカラー型
- `value_type`：他のマップを除く任意の型
- `map` フィールドは `repeated` を使えない

## サービス定義（ **service** ）

メッセージ型を RPCで使う場合、同様に .proto ファイルで RPC サービスのインターフェイスを定義することができる。

```
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

メッセージをリクエスト・レスポンスに設定でき、また、JSONエンコード機能も提供している。（[JSONマッピング](https://developers.google.com/protocol-buffers/docs/proto3#json)）  
rpc は以下の 4 方式をサポートしている。

```
service HelloService {
    // 1. Unary RPC ：1リクエスト1レスポンス
    rpc SayHello(HelloRequest) returns (HelloResponse);

    // 2. Server streaming RPC ：１つのリクエストに複数レスポンス
    rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);

    // 3. Client streaming RPC ：複数のリクエストに一つのレスポンス
    rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);

    // 4. Bidirectional streaming RPC ：双方向
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
}
```

## コード生成

以下コマンドで.proto ファイルから各言語のコードを生成できる。

```
% protoc --proto_path=IMPORT_PATH # .proto ファイルがあるディレクトリパス
         --cpp_out=DST_DIR        # C++言語の出力先
         --java_out=DST_DIR       # Java言語の出力先
         --python_out=DST_DIR     # Python言語の出力先
         --go_out=DST_DIR         # Go言語の出力先
         --ruby_out=DST_DIR       # Ruby言語の出力先
         --javanano_out=DST_DIR   # Android用の出力先
         --objc_out=DST_DIR       # ObjectiveC言語の出力先
         --csharp_out=DST_DIR     # C#言語の出力先
         path/to/file.proto       # 参照する.proto ファイル
```

入力として 1 つ以上の .proto ファイルが必要。（path/to/file.proto）  
.proto ファイルがカレントディレクトリからの相対名で与えられるとしても，コンパイラが正しい名前を決定できるように，各ファイルは IMPORT_PATH の中のどこかの場所存在する必要がある。