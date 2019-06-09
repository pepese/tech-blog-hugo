---
title: "Node.js・npm入門"
date: 2017-08-14T08:10:36+09:00
slug: "nodejs-basics"
tags:
- node.js
- npm
- javascript
draft: false
archives:
- 2017/08
---

Node.js（npm）についてまとめる。

npm（Node Packege Manager）は、Node.js用のパッケージ管理コマンド。  
JavaScriptエコシステムツール・ライブラリはnpmで導入できるものが多い。  
npmはNode.jsと同時にインストールされるので、Node.jsを入れておけばいい。  
また、Node.jsはJavaScriptの実行エンジン。

ちなみに[ここ](http://node.green)でNode.jsのES6対応状況がわかる。  
v8.3.0で **99%** カバーしている。

<!--more-->

# インストール

Node.jsのインストール方法は以下の「anyenv」を参照。

- [すべての\*\*envを管理するanyenv](https://blog.pepese.com/entry/anyenv/)

## ndenv

上記の手順でNode.jsを導入した場合、Node.jsのバージョンを `ndenv` でコントロールする。

```sh
$ ndenv install -l # インストールできるNode.jsのバージョン確認
$ ndenv install v8.3.0 # バージョンを指定してインストール
$ ndenv versions # インストールされたことを確認
$ ndenv global v8.3.0 # バージョンを切り替え
$ ndenv local v8.3.0 # 今いるプロジェクトだけバージョンを指定したい場合はこっち
$ node -v # 確認
$ npm update -g npm # npm のバージョンアップ
$ npm -v # 確認
```

# npmの使い方

公式ドキュメントは[ここ](https://docs.npmjs.com)。  
プロキシの設定が必要な場合は以下。

```sh
$ npm config set proxy http://<username>:<password>@<proxy-host>:<proxy-port>
$ npm config set https-proxy http://<username>:<password>@<proxy-host>:<proxy-port>
```

## プロジェクトの作成

任意のディレクトリを作成して `npm init` を実行すると `package.json` が作成され、プロジェクトとなる。  
いろいろ聞かれるが、全部Enterで問題無い。あとで変更可能。

```sh
$ mkdir nodejs-sample
$ cd nodejs-sample
$ npm init
```

`package.json` はMavenでいう `pom.xml` みたいなもの。（Javaの人向けでごめん）

## パッケージのインストール

### グローバルインストール

`npm install -g <package_name>@<version>` でパッケージのインストールが可能。  
`-g` オプションがついてるのがポイント。  
アンインストールは `npm uninstall -g <package_name>` 。  
インストールパッケージ一覧の確認は `npm list -g` 。  

インストール先は以下で確認できる。

```
# グローバルインストール先のディレクトリ
$ npm root -g

# グローバルインストール先の、コマンドディレクトリ
$ npm bin -g
```

### ローカルインストール

`npm install <option> <package_name>@<version>` でパッケージのインストールが可能。  
インストールしたパッケージは `package.json` に追記され、 `node_modules/` 配下に配備される。  
オプションは以下の通り。

- `--save` 、 `-S`
    - `package.json` の `dependencies` に追記される
    - `npm install  --production` した時に `mode_modules/` に配備される
    - 開発したソフトウェアを動かすのに必要な依存ライブラリ
- `--save-dev` 、 `-D`
    - `package.json` の `devDependencies` に追記される
    - `npm install --dev` した時に `mode_modules/` に配備される
    - ソフトウェアを開発する際に必要になるツール・ライブラリ
- `--save-optional` 、 `-O`
    - `package.json` の `optionalDependencies` に追記される
    - `npm install` した時に `mode_modules/` に配備される（失敗してもスルーされる）

デフォルト設定の場合、 `npm install` （オプション無）すると、 `dependencies` 、 `devDependencies` 、 `optionalDependencies` の全てがインストールされる。  
アンインストールは `npm uninstall <package_name>` 。  
インストールパッケージ一覧の確認は `npm list` 。

### パッケージの使い方

npmで導入したJavaScriptライブラリ（ `node_modules/` 配下にある）は、JavaScriptのソースコードからほぼ以下の形式で呼び出すことができる。

```javascript
var req = require('request')
```

上記は `request` モジュールを呼び出して `req` 変数へ代入している例。

## npmスクリプト

`package.json` の `scripts` にスクリプトを登録することにより簡易に実行することができる。  
例えば以下。

```javascript
"scripts": {
  "test": "mocha --require test/support/env --reporter spec --bail --check-leaks test/ test/acceptance/",
  "test-ci": "istanbul cover node_modules/mocha/bin/_mocha --report lcovonly -- --require test/support/env --reporter spec --check-leaks test/ test/acceptance/",
  "test-cov": "istanbul cover node_modules/mocha/bin/_mocha -- --require test/support/env --reporter dot --check-leaks test/ test/acceptance/",
  "test-tap": "mocha --require test/support/env --reporter tap --check-leaks test/ test/acceptance/"
  }
```

上記のようにスクリプトを定義することにより `npm test` 、 `npm test-ci` といった具合で登録したスクリプトを実行することができる。

### `package.json` の更新

手動で `package.json` 内のライブラリバージョンを変更してもよいが、 `npm-check-updates` を利用すると楽。

```bash
$ npm install -g npm-check-updates
$ which ncu
/usr/local/bin/ncu
```

`ncu` コマンドで `package.json` のライブラリの最新バージョンを確認できる。

```bash
$ ncu
Checking /path/to/project/package.json
[====================] 17/17 100%

 hexo-deployer-git       ^0.3.1  →  ^1.0.0 
 hexo-generator-archive  ^0.1.4  →  ^0.1.5 
 hexo-generator-index    ^0.2.0  →  ^0.2.1 
 hexo-renderer-ejs       ^0.3.0  →  ^0.3.1 
 hexo-renderer-marked    ^0.3.2  →  ^1.0.1 
 hexo-renderer-pandoc    ^0.2.6  →  ^0.3.0 
 hexo-renderer-stylus    ^0.3.1  →  ^0.3.3 
 hexo-server             ^0.2.0  →  ^0.3.3 

Run ncu -u to upgrade package.json
```

`ncu -u` で `package.json` が書き換えられる。

# Node.jsの使い方

Node.jsのAPIリファレンスは[ここ](https://nodejs.org/api/)。

## シンプルに動かしてみる

まず、Node.jsをJavaScriptエンジンとしてシンプルに動かしてみる。

■hello.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/nodejs-sample/hello.js?footer=0"></script>

■main.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/nodejs-sample/main.js?footer=0"></script>

上記JSファイルを作成して下記を実行。

```sh
$ node main.js
Hello World !
```

`node <JSファイル>` コマンドで実行できる。

## HTTPサーバ

Node.jsでHTTPサービスを立ち上げてみる。

■app.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/nodejs-sample/app.js?footer=0"></script>

下記を実行してブラウザで `http://127.0.0.1:1337/` へアクセスすると「Hello World !」と表示される。

```sh
$ node app.js
```

`Ctr+C` でサービスを停止できる。  
この方法でHTTPサービスを起動した場合、ロジックでエラーが発生するとプロセスが終了する。  
エラーが発生してもプロセスが終了しないようにする方法の１つとして **forever** の導入がある。

■foreverのインストール、起動、停止

```sh
$ npm install -g forever
$ forever start app.js
$ forever stop app.js
```

## Node.jsに関する話題

- [Node.jsでのJavaScriptメモリリークを発見するための簡単ガイド](http://postd.cc/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/)
    - JavaScriptは **ガベージコレクション** を行う言語。
