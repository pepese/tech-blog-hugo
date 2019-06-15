---
title: "Node.jsでDynamoDBへアクセスする"
date: 2017-08-29T17:55:56+09:00
slug: "nodejs-dynamodb"
tags:
- node.js
- yarn
- aws
- dynamodb
draft: false
archives:
- 2017/08
---

ここでは、Node.js から DynamoDB にアクセスしてみる。  
以下の記事を読んだ前提で書く。

- [Node.js/npm入門](https://blog.pepese.com/entry/nodejs-basics/)
- [DynamoDB入門](https://blog.pepese.com/entry/dynamodb-basics/)

<!--more-->

# 環境構築

```sh
$ mkdir dynamodb-js
$ cd dynamodb-js
$ yarn init
$ yarn add aws-sdk
$ touch .gitignore
$ aws dynamodb create-table \
  --attribute-definitions '[{"AttributeName":"test_hash","AttributeType":"S"},{"AttributeName":"test_range","AttributeType":"S"}]' \
  --table-name 'test_table' \
  --key-schema '[{"AttributeName":"test_hash","KeyType":"HASH"},{"AttributeName":"test_range","KeyType":"RANGE"}]' \
  --provisioned-throughput '{"ReadCapacityUnits":5,"WriteCapacityUnits":5}'
```

`.gitignore` を下記のように編集しておく。

```
node_modules
.DS_Store
./**/.DS_Store
```

# 実装

[API Document](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html)を参考のこと。  
以下を作成する。

- app/dynamodb.js
- app/putItem.js
- app/getItem.js
- app/deleteItem.js

[DynamoDB入門](https://pepese.github.io/blog/dynamodb-basics/) で作成したテーブルにアクセスする前提。

## app/dynamodb.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/dynamodb-js/app/dynamodb.js?footer=0"></script>

## app/putItem.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/dynamodb-js/app/putItem.js?footer=0"></script>

## app/getItem.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/dynamodb-js/app/getItem.js?footer=0"></script>

## app/deleteItem.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/dynamodb-js/app/deleteItem.js?footer=0"></script>

# 実行

```sh
$ node app/putItem.js
$ node app/getItem.js
$ node app/deleteItem.js
```
