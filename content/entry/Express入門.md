---
title: "Express入門"
date: 2017-08-14T07:59:50+09:00
slug: "express-basics"
tags:
- express
- node.js
- npm
- javascript
draft: false
archives:
- 2017/08
---

Node.jsのWebフレームワーク `Express` 触ってみた。  
下記の記事を読んだテイで書く。

- [Node.js・npm入門](https://blog.pepese.com/entry/nodejs-basics/)

<!--more-->

# 構築

## Expressプロジェクトの作成

プロジェクト作成は以下。

```sh
$ mkdir express-sample          // プロジェクトディレクトリの作成
$ cd express-sample
$ npm init
```

Angular CLI 等で既にプロジェクトを作成している場合は、上記は行わず以下から実行。

```sh
$ npm i -S express@5.0.0-alpha.6 body-parser cookie-parser debug morgan pug serve-favicon request fs file-stream-rotator node-sass-middleware
$ touch .gitignore
$ mkdir app                     // サーバサイドExpressアプリ用のソースディレクトリ作成
$ touch app/app.js              // Expressアプリケーション起動ポイントの作成
$ mkdir app/models              // Mongoose（MongoDB）用のモデルディレクトリ作成
$ touch app/models/.gitkeep
$ mkdir app/repositories        // DAO/Repository用のディレクトリ作成
$ touch app/repositories/.gitkeep
$ mkdir app/config              // 設定ファイル用ディレクトリ作成
$ touch app/config/config.json  // 環境差分ファイル作成
$ mkdir app/log                 // ログ出力用ディレクトリ作成
$ touch app/log/.gitkeep
$ mkdir app/spec                // テストスクリプト用のディレクトリ作成
$ touch app/spec/.gitkeep
```

Expressアプリケーションのソースディレクトリは `app/` だけで完結するようにする。  
また、 forever を導入しておく。

```sh
$ npm install -g forever
$ ndenv rehash
```

さらに `.gitignore` に以下を追加しておく。

```
# Logs
/app/log
*.log

# dependencies
/node_modules

# System Files
.DS_Store
Thumbs.db
```

### フロントエンドアプリケーションをプロジェクトに同梱する場合、REST APIアプリを作成する場合

この場合、以下のようにディレクトリを作成する。

```sh
$ mkdir app/api           // REST API用のコントローラディレクトリ作成
$ touch app/api/.gitkeep
```

フロントエンドアプリケーションのトランスパイル結果は `express-sample/dist` に出力されるものとし、サーバサイドアプリケーション（ `express-sample/app` ）は `express-sample/dist` をExpressの公開ディレクトリとする。  
フロントエンドアプリケーションは `app/api` に実装したAPIにアクセスする。

### フロントエンドアプリケーションをプロジェクトに同梱せず、ExpressからテンプレートエンジンVIEWを使用する場合

この場合、以下のようにディレクトリを作成する。  
なお、 SCSS を使用する。

```sh
$ mkdir app/controllers                  // VIEW用のコントローラディレクトリ作成
$ touch app/controllers/.gitkeep
$ mkdir app/views                        // 画面・テンプレート（Pugなど）用のディレクトリ作成
$ touch app/views/.gitkeep
$ mkdir app/public                       // 静的コンテンツ用のディレクトリ作成
$ mkdir app/public/javascripts           // JS用のディレクトリ作成
$ touch app/public/javascripts/.gitkeep
$ mkdir app/public/stylesheets           // SCSS用のディレクトリ作成
$ touch app/public/stylesheets/style.scss
$ mkdir app/public/images                // 画像用のディレクトリ作成
$ touch app/public/images/.gitkeep
```

## ソースコードの作成

 「フロントエンドアプリケーションをプロジェクトに同梱する場合」は以下を参照。

- [所謂MEANスタックを作る](https://blog.pepese.com/entry/mean-stack-basics/)

ここでは、「フロントエンドアプリケーションをプロジェクトに同梱せず、Expressからテンプレートエンジンを使用する場合」で、最低限動くViewアプリを構築するために以下のスクリプトを実装する。

- `app/app.js`
    - Expressアプリケーションのミドルウェアの設定と起動処理
- `app/controllers/router.js`
    - Viewを処理するモジュールへのルーティングを行う
- `app/controllers/get_index.js`
    - Index画面を表示する
- `app/controllers/get_users.js`
    - Users画面を表示する
- `app/public/stylesheets/style.scss`
- `app/views/layout.pug`
- `app/views/index.pug`
- `app/views/error.pug`
- `app/config/config.json`
    - 環境差分設定ファイル

### app/app.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/app.js?footer=0"></script>

### app/controllers/index.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/router.js?footer=0"></script>

### app/controllers/get_index.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_index.js?footer=0"></script>

### app/controllers/get_users.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_users.js?footer=0"></script>

### app/public/stylesheets/style.scss

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/public/stylesheets/style.scss?footer=0"></script>

### app/views/layout.pug

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/views/layout.pug?footer=0"></script>

### app/views/index.pug

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/views/index.pug?footer=0"></script>

### app/views/error.pug

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/views/error.pug?footer=0"></script>

### app/config/config.json

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/config/config.json?footer=0"></script>

### 起動

```
$ NODE_ENV=production forever start app/app.js
```

`NODE_ENV` で環境を指定することができる。デフォルトは `development` 。


# ちょっとExpressの機能解説

## Expressアプリケーション

Expressアプリケーションの最小単位は以下のように作成する。

```javascript
const express = require('express');
const app = express();
```

## ミドルウェア関数

Expressは、単独では最小限の機能を備えたルーティングとミドルウェアのWebフレームワーク。  
Expressアプリケーションは様々な **ミドルウェア関数** 呼び出しを行うことで様々な機能を実現する。  
ミドルウェア関数はコールバック関数で以下の種類がある。

- アプリケーション・レベルのミドルウェア
- ルーター・レベルのミドルウェア
- エラー処理ミドルウェア
- 標準装備のミドルウェア
- サード・パーティー・ミドルウェア

### アプリケーション・レベルのミドルウェア

以下のように記述する。

```javascript
app.use([path,] callback [, callback...])
```

`path` を記載しない場合は **全てのリクエスト** に対してミドルウェア関数（ `callback` :コールバック関数）が実行される。  
コールバック関数には `(req, res, next)` の3つを渡すことができる。
`next()` を実行すると次のミドルウェア関数に処理が移る。（ **`next()` を実行しないと処理は終わる** ）

```javascript
app.use( (req, res, next) => {
  console.log('Time:', Date.now());
  next();
});
```

`path` を記載すると **指定したパスへのリクエスト** に対してミドルウェア関数が実行される。  

```javascript
app.use('/user/:id', (req, res, next) => {
  console.log('Request Type:', req.method);
  next();
});
```

```javascript
app.get('/user/:id', (req, res, next) => {
  res.send('USER');
});
```

**`app.use` は `app.[all|get|post|put|delete|etc]` よりも全て前に記述する必要がある** 。

### ルーター・レベルのミドルウェア

ルーター・レベルのミドルウェアは、express.Router() のインスタンスにバインドされる点を除き、アプリケーション・レベルのミドルウェアと同じ。

```javascript
const express = require('express');
const router = express.Router();
```

正直、 **アプリケーション・レベルとルーター・レベルの違いはよくわからない** 。  
以下の方針で使い分けることにする。

- アプリケーション・レベル
    - **アプリ全体** 、 **全てのHTTPメソッド** へ影響する処理
    - `app.use()` を使用する
- ルーター・レベル
    - **特定のパス** 、 **特定のHTTPメソッド** へのルーティング
    - `router.[all|get|post|put|delete](path, callback [, callback...])` を使用する

### エラー処理ミドルウェア

引数は ` (err、req、res、next)` となり `err` が追加される。

```javascript
app.use( (err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

### 標準装備のミドルウェア

Expressの標準装備のミドルウェア関数は `express.static` のみ。  
過去はもったあったらしいが、個別モジュール化されたらしい。

```javascript
express.static(root, [options])
```

静的コンテツを提供する機能。  
`root` 引数は、静的コンテンツを提供するルートディレクトリを指定する。  
`options` オブジェクトにはプロパティを指定することができる。

```javascript
var options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false,
  maxAge: '1d',
  redirect: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now());
  }
}

app.use(express.static('public', options));
```

### サード・パーティー・ミドルウェア

`npm` でモジュールをインストールして、アプリケーション・レベルまたはルーター・レベルでロードして使用する。

```javascript
const express = require('express');
const bodyParser = require('body-parser'); // サード・パーティー・ミドルウェア

const app = express();

// Parsers for POST data
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```

## モジュール呼び出し

`module.exports` と `require` を使用すると、モジュールの外部参照が可能になる。  
以下のような `index.js` があったとすると

```javascript
router.post('/', (req, res) => {
  ...
}
module.exports = router; // 外部モジュールからrequireできるようになる
```

`app.js` から以下のように `index.js` の機能を呼び出すことができる。

```javascript
const index = require('./index');
```

## Expressアプリケーションの設定

Expressでは、以下の記載で「key-value」形式でアプリケーションに対して様々な設定ができる。

```javascript
app.set(name, value);
app.get(name);
```

代表的な設定項目としては以下（[詳細](http://expressjs.com/ja/api.html#app.settings.table)）があるが、単にインメモリキーバリューストア（好きな「key-value」を設定できる）としても使用できる。

|Property|Type|Description|Default|
|:---:|:---:|:---|:---:|
|case sensitive routing|Boolean|ルーティング時、URLの大文字・小文字を区別する。|N/A (undefined)|
|env|String|環境名。productionとかdevelopmentとか。|process.env.NODE_ENV or “development”|
|etag|Varied| `Etag` レスポンスヘッダの設定を行う。|weak|
|jsonp callback name|String|JSONP callback名を設定する。|“callback”|
|json replacer|Varied|`JSON.stringify` で利用される引数 `replacer` の設定。JSONの各メンバの値の変換の設定ができる。|N/A (undefined)|
|json spaces|Varied|`JSON.stringify` で利用される引数 `space` の設定。JSONパース時に見やすいようにインデントのスペースを設定できる。|N/A (undefined)|
|query parser|Varied|URLクエリパラメータのパースの設定ができる。 `simple` 、 `extended` 、カスタマイズから選択できる。 `simple` はNodeJSが内包するNativeのクエリパーサ。 `extended` は サードパーティモジュール `qs` 。カスタマイズは関数を設定し、その関数はクエリパラメータの文字列を受け取り、キー・バリューのオブジェクトを返却するよう実装する。|"extended"|
|strict routing|Boolean|ルーティングにおいてURLの `/foo` と `/foo/` を区別するかどうか設定する。|N/A (undefined)|
|subdomain offset|Number|サブドメインにアクセスするために削除するホストのドット区切り部分の数。|2|
|trust proxy|Varied|信頼するWebプロキシの設定を行う。詳細は、[ここ](http://expressjs.com/ja/guide/behind-proxies.html)を参照。|false (disabled)|
|views|String or Array|Viewを配置するディレクトリを文字列もしくは配列で設定する。|process.cwd() + '/views'|
|view cache|Boolean|Viewテンプレートコンパイルキャッシュを有効にする。|true in production, otherwise undefined.|
|view engine|String|Viewエンジンの設定を行う。|N/A (undefined)|
|x-powered-by|Boolean|"X-Powered-By: Express" HTTPヘッダを有効にする。|true|

## モジュール間で変数をやりとりする方法

- global変数を使う
    - `const global.hoge = 'hogehoge';` みたいに、頭にglobalを付けるとglobalスコープで扱われる。
    - ただし、 `require('./global.js');` のようにグローバル変数を定義したモジュールを使用するモジュールでロードする必要がある。
    - グローバル領域を汚染するので注意が必要
- 共有変数用のモジュールを作成する
    - `module.exports = {};` とだけ書いた `common.js` を作成する
    - 他モジュールからは `require('./common').hoge = hoge;` で変数を代入でき、 `const hoge = require('./common').hoge` で変数を参照できる
    - どんな変数を作ったからわからなくなるので、 `common.js` にはコメントくらい残しておく必要がある
