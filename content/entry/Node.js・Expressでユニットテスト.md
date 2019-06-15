---
title: "Node.js・Expressでユニットテスト"
date: 2017-08-16T17:52:59+09:00
slug: "express-test"
tags:
- node.js
- npm
- express
- javascript
- mocha
- chai
- sinon.js
- istanbul
- nyc
draft: false
archives:
- 2017/08
---

Node.js/Expressアプリケーションのテストをしてみる。  
使用するツール・ライブラリは以下。

- テスティングフレームワーク
    - [mocha](https://mochajs.org/)
        - デフォルトでは `./test/*.js` 、 `./test/*.coffee` をテストスクリプトとして認識
        - `mocha.opts` というファイルにオプションを設定できる模様（[参考](https://mochajs.org/#mochaopts)）
- アサート
    - [Chai](http://chaijs.com/)
- モック
    - [sinon.js](http://sinonjs.org/)
- カバレッジ
    - [Istanbul](https://istanbul.js.org/)
        - 公式の案内にもある通り、以下の `nyc` 経由で `istanbul` を利用する
    - [nyc](https://github.com/istanbuljs/nyc)
        - tap、mocha、AVA といったJSテスティングフレームワークと Istanbul をうまく連携させるコマンドラインツール
        - [mocha用のチュートリアル](https://istanbul.js.org/docs/tutorials/mocha/)

なお、タスクランナーは使用せず、npmスクリプトを使用する。

<!--more-->

以下の記事を読んだ前提で書く。

- [Express入門](https://pepese.github.io/blog/express-basics/)

# 環境設定

## グローバルインストール

```sh
$ yarn global add nyc
$ ndenv rehash
```

## ローカルインストール

[Express入門](https://pepese.github.io/blog/express-basics/)で作成したプロジェクトにて以下を導入する。

```sh
$ yarn add mocha chai sinon nyc rimraf --dev
```

## ディレクトリ作成

先に紹介した記事の通り、Expressアプリケーションのソースディレクトリは `app/` だけで完結するようにする。  
以下のようにテストスクリプト用のディレクトリを作成する。

```sh
$ mkdir app/spec
$ mkdir app/spec/controllers
```

# ユニットテストの実装

先に紹介した記事のスクリプトをテストする。

## packege.json

以下を加筆する。

```
{
  // 上記省略
  "nyc": {
    "check-coverage": true,
    "per-file": true,
    "lines": 90,
    "statements": 90,
    "functions": 90,
    "branches": 90,
    "include": [
      "app/"
    ],
    "exclude": [
      "app/spec/**/*.spec.js",
      "app/app.js",
      "app/controllers/router.js",
      "app/config",
      "app/coverage",
      "app/log",
      "app/public"
    ],
    "reporter": [
      "html",
      "text"
    ],
    "require": [],
    "extension": [
      ".js"
    ],
    "cache": true,
    "all": true,
    "report-dir": "app/coverage"
  },
  "scripts": {
    "clean": "rimraf .nyc_output ./app/coverage",
    "test": "mocha app/spec/*.spec.js app/spec/**/*.spec.js"
  },
  // 下記省略
}
```

## テストスクリプト `app/spec/controllers/get_index.spec.js`

`app/controllers/get_index.js` を対象としたテストスクリプトを以下のように作成する。

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/spec/controllers/get_index.spec.js?footer=0"></script>

## テストスクリプト `app/spec/controllers/get_users.spec.js`

`app/controllers/get_users.js` を対象としたテストスクリプトを以下のように作成する。

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/spec/controllers/get_users.spec.js?footer=0"></script>

## テストの実行

npmスクリプトで以下のようにテストを実行する。

```sh
$ npm test

  /
    ✓ Index画面が1回レンダリングされること

  /users
    ✓ Users画面が1回レンダリングされること


  2 passing (8ms)
```

カバレッジレポートは `nyc` を用いて以下のように出力する。

```sh
$ nyc npm test

  /
    ✓ Index画面が1回レンダリングされること


  /users
    ✓ Users画面が1回レンダリングされること


  2 passing (10ms)

--------------|----------|----------|----------|----------|----------------|
File          |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
--------------|----------|----------|----------|----------|----------------|
All files     |      100 |      100 |      100 |      100 |                |
 get_index.js |      100 |      100 |      100 |      100 |                |
 get_users.js |      100 |      100 |      100 |      100 |                |
--------------|----------|----------|----------|----------|----------------|
```

`package.json` の `nyc` の設定の通りだが、HTML形式のレポートが `app/coverage` 配下に出力される。  
なお、 `$ npm run-script clean` で `nyc` が出力する `.nyc_output` 、 `app/coverage` を削除する。

# ライブラリ・ツールの概念を解説

個々のライブラリを軽く解説する。  
なお、各ライブラリを連携するプラグインライブラリに以下のようなものがある。

- mocha-sinon
    - [Github](https://github.com/elliotf/mocha-sinon)
- [sinon-chai](http://chaijs.com/plugins/sinon-chai/)
    - [Github](https://github.com/domenic/sinon-chai)
    - chai-sinon というのもあるが、違いがわからん。。。

## mocha

mocha では以下のような関数を使用してテストを記述していく。

|関数|概要|
|:---|:---|
|describe|ネスト管理可能な階層。テスト対象を宣言する。|
|before|describeの前に実行される。前提条件を記述する。|
|after|describeの後に実行される。後処理を記述する。|
|beforeEach|各describeの前に実行される。前提条件を記述する。|
|afterEach|各describeの後に実行される。後処理を記述する。|
|it|検証内容を記述する。|

## Chai

Chai の代表的な機能には以下がある。

- Should
- Expect
- Assert

Should と Expect は基本同じインターフェースだが、Sholudはオブジェクト自身を拡張してsholudメソッドを追加して実装されているのに対して、expectは関数で引数に評価する値を渡すことでオブジェクトの検査を行う。

```
// hogeオブジェクトとfuge文字列が等しいかどうか検査

// Should
hoge.should.equal('fuge');

// Expcet
expect(hoge).to.equal('fuge');
```

## sinon.js

sinon.js の代表的な機能には以下がある。

- スパイ
    - 関数がどのように呼び出されたかを記録する
- スタブ
    - 関数の戻り値をあらかじめ設定し、その結果でテストを行う
- モック
    - 実行前に関数の実行回数など期待する結果を指定しておく
- フェイク
    - 問い合わせるDBやサーバ処理などを単純な実装に置き換える

[API documentation - Sinon.JS - v3.2.0](http://sinonjs.org/releases/v3.2.0/)

### スパイ

スパイオブジェクトの作成方法は以下。

|呼び出し方法|概要|
|:---|:---|
|let spy = sinon.spy();|空のスパイオブジェクトを生成。主にコールバックで渡す無名関数などに対して使用する。|
|let spy = sinon.spy(myFunc);|特定の関数に対してスパイオブジェクトを作成する。|
|let spy = sinon.spy(object, "method");|オブジェクト内のメソッドに対してスパイオブジェクトを生成する。|

スパイオブジェクトの主な関数は以下。

|呼び出し方法|概要|
|:---|:---|
|spy.calledWith(arg1, arg2, ...);|指定した引数で関数が呼び出されたかを確認する。|
|spy.callCount|呼び出された回数を返す。|
|spy.called|呼び出された場合にtrueを返す。|
|spy.calledOn(obj);|指定したオブジェクトで関数が実行された場合trueを返す。|
|spy.calledOnce|1回呼び出された場合trueを返す。|
|spy.calledTwice|2回呼び出された場合trueを返す。|
|spy.exceptions|発生したexceptionを返す。|
|spy.returnValues|実行後の戻り値を返す。|
|spy.withArgs(arg1, arg2, ...);|指定した引数で関数が呼び出された場合にのみ、スパイオブジェクトを返す。|

### スタブ

スタブオブジェクトの作成方法は以下。

<table>
<thead>
<tr class="header">
<th align="left">呼び出し方法</th>
<th align="left">概要</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">let stub = sinon.stub();</td>
<td align="left">空のスタブオブジェクトを生成。主にコールバックで渡す無名関数などに対して使用する。</td>
</tr>
<tr class="even">
<td align="left">let stub = sinon.stub(object, “method”);</td>
<td align="left">特定の関数に対してスタブオブジェクトを作成する。</td>
</tr>
<tr class="odd">
<td align="left">let stub = sinon.stub(object, “method”, func);</td>
<td align="left">特定の関数に対してスタブオブジェクトを作成し、指定した関数で処理を上書きします。</td>
</tr>
<tr class="even">
<td align="left">let stub = sinon.stub(object);</td>
<td align="left">対象オブジェクトのメソッドをすべてスタブにする。</td>
</tr>
</tbody>
</table>

スタブオブジェクトの主な関数は以下。

|呼び出し方法|概要|
|:---|:---|
|stub.returns(obj);|呼び出し時のリターン値を指定する。|
|stub.throws(obj);|呼び出し時に指定したExceptionを発生させる。|
|stub.onCall(n);|n回目のスタブオブジェクトを返す。|
|stub.onFirstCall();|1回目のスタブオブジェクトを返す。|
|stub.onSecondCall();|2回目のスタブオブジェクトを返す。|

### モック

モックオブジェクトの作成方法は以下。

<table>
<thead>
<tr class="header">
<th align="left">呼び出し方法</th>
<th align="left">概要</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">let mock = sinon.mock(object);</td>
<td align="left">オブジェクトのモックを作成する。</td>
</tr>
<tr class="even">
<td align="left">let expectation = mock.expects(“method”);</td>
<td align="left"><code>object.method</code> をオーバーライドしてモック関数を作成する。</td>
</tr>
<tr class="odd">
<td align="left">let expectation = sinon.expectation.create(“method”);</td>
<td align="left">モックオブジェクト無しでモック関数を作成する。</td>
</tr>
</tbody>
</table>

スタブオブジェクトの主な関数は以下。

|呼び出し方法|概要|
|:---|:---|
|mock.restore();|全てのモック関数をリストアする。|
|mock.verify();|全てのモック関数の expectation をベリファイする。|
|expectation.atLeast(number);|モック関数が少なくとも呼び出される回数を指定する。|
|expectation.atLeast(number);|モック関数が多くとも呼び出される回数を指定する。|
|expectation.never();|モック関数が決して呼び出されないことを指定する。|
|expectation.once();|モック関数が1回呼び出されることを指定する。|
|expectation.twice();|モック関数が2回呼び出されることを指定する。|
|expectation.exactly(number);|モック関数が呼び出される回数を指定する。|
|expectation.withArgs(arg1, arg2, ...);|モック関数呼び出し時に含まれる引数を指定する。|
|expectation.withExactArgs(arg1, arg2, ...);|モック関数呼び出し時の引数を指定する。|
|expectation.on(object);|モック関数が object から呼び出されることを指定する。|
|expectation.verify();|モック関数が上記の指定にマッチしていたか確認する。|

## フェイク

フェイクには、時間・XMLHttpRequest・サーバーオブジェクトの置き換えができる。  
ここでは省略する。
