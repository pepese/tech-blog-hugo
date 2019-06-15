---
title: Hyperledger fabric入門
tags:
id: hyperledger-fabric-basics
draft: true
---





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
