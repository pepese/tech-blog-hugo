---
title: "Node.js・Expres・requestでHTTP(S)リクエストを発行する"
date: 2017-08-28T17:46:16+09:00
slug: "express-http-client-request"
tags:
- node.js
- yarn
- express
- javascript
- request
- promise
draft: false
archives:
- 2017/08
---

Node.js/ExpressアプリケーションからHTTP(S)を発行してみる。  
使用するツール・ライブラリは以下。

- HTTP(S)クライアント
    - [request](https://github.com/request/request)

以下の記事を読んだ前提で書く。

- [Express入門](https://blog.pepese.com/entry/express-basics/)

<!--more-->

ちなみに HTTP(S)クライアントとしては `request` 以外にも以下のようなものがある。

- [http](https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_http_methods) / [https](https://nodejs.org/dist/latest-v6.x/docs/api/https.html#https_https_get_options_callback)
    - Node.js のネイティブライブラリ
- [Unirest](http://unirest.io/nodejs.html)
- [axios](https://github.com/mzabriskie/axios)
    - `request` が嫌ならこれがいいかな？

# 環境設定

先の記事で紹介したプロジェクトにて以下を実行する。

```sh
$ yarn add request
```

# 実装

- `app/controllers/get_ip.js`
    - 自分のグローバルIPを取得するAPIにリクエストを発行するサンプル
    - `http://[FQDN]/ip` で動確

## `app/controllers/get_ip.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_ip.js?footer=0"></script>

# request の基本的な使い方

- `request(options, callback)`
    - コールバックで `options` に基づいたリクエストを発行する
    - サンプルはこれを使用
    - [ドキュメント](https://github.com/request/request#requestoptions-callback)
    - [optiong の設定](https://github.com/request/request#requestoptions-callback)
- `request.defaults(options)`
    - HTTP(S)クライアントのデフォルトの設定を行う
    - [ドキュメント](https://github.com/request/request#requestdefaultsoptions)
- `request.METHOD()`
    - HTTPメソッドを関数で表現する形式
    - [ドキュメント](https://github.com/request/request#requestmethod)

# Promise

コールバック地獄を避けたい人は [request-promise](https://github.com/request/request-promise) を使うと簡単に非同期 bluebird 版 Promise を実現できる。

## 導入

```sh
$ yarn add request-promise
```

## 実装 `app/controllers/get_ip_promise.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_ip_promise.js?footer=0"></script>
