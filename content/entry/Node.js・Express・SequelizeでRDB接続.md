---
title: "Node.js・ExpressでブラウザからPUT・DELETEリクエストを発行する"
date: 2017-08-20T17:48:36+09:00
slug: "express-rdb-sequelize"
tags:
- node.js
- yarn
- express
- javascript
- sequelize
draft: false
archives:
- 2017/08
---

Node.js/ExpressアプリケーションからRDBへ接続してみる。  
使用するツール・ライブラリは以下。

- O/R Mapper
    - [Sequelize](http://docs.sequelizejs.com/)
- RDB
    - [SQLite3](https://sqlite.org/index.html)

以下の記事を読んだ前提で書く。

- [Express入門](https://blog.pepese.com/entry/express-basics/)

<!--more-->

# 環境設定

先の記事で紹介したプロジェクトにて以下を実行する。

```sh
$ yarn add sequelize sqlite3 morgan
$ mkdir app/db
```

`.gitignore` へ以下を追加。

```
node_modules
app/db/*.sqlite
.DS_Store
./**/.DS_Store
```

# 実装

- `app/repositories/sequelize.js`
    - Sequelizeを使ったRDB接続用モジュール
- `app/repositories/models.js`
    - Clientモデルを作成
    - テーブルの作成とサンプルデータのインサート
- `app/controllers/get_client`
    - 上記の `models.js` 経由でClientの検索


## `app/repositories/sequelize.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/repositories/sequelize.js?footer=0"></script>

## `app/repositories/models.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/repositories/models.js?footer=0"></script>

## `app/controllers/get_clients.js`

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_clients.js?footer=0"></script>

# 実行

以下で起動。

```sh
$ node app/app.js
```

`http://localhost:3000` にブラウザでアクセスして Client 情報が見れたら成功。

# Association

ちょっとハマったので、ここではモデル同士のリレーション・関連付けについて記載する。  
リレーションには以下がある。

- hasOne
    - 1:1 のリレーション
- belongsTo
    - 1:N のリレーション
    - `hasOne` や `hasMany` される側
- hasMany
    - 1:N のリレーション
- belongsToMany
    - N:M のリレーション

ここでは `belongsToMany` の例のみ記載する。  
[参考/チュートリアル](http://docs.sequelizejs.com/manual/tutorial/associations.html)、[参考/リファレンス](http://docs.sequelizejs.com/class/lib/associations/base.js~Association.html)

## 実装

- app/models/user.js
    - ユーザテーブルのモデル
- app/models/team.js
    - チームテーブルのモデル
- app/models/team-user.js
    - チームテーブルとユーザテーブルの N:M のリレーションを構築する中間テーブルのモデル
- app/repositories/sequelize.js
    - Sequelize のロード・設定を行う
    - Sqlite3 を使用した例になっている
- app/repositories/db.js
    - モデルのリレーション作成、DBとの同期を行う
- app/app.js
    - 適当な実行ファイル、全く参考にならない
    - `node app/app.js` で実行

個人的な好みだが、モデルは **単数形** で作成する。  
また、 **モデルのリレーションを実装してから `sync()` する** のがポイント。（ここでハマった）

### app/models/user.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/sequelize-sample/app/models/user.js?footer=0"></script>

`user_id` が主キー。

### app/models/team.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/sequelize-sample/app/models/team.js?footer=0"></script>

`team_id` が主キー。

### app/models/team-user.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/sequelize-sample/app/models/team-user.js?footer=0"></script>

`user_id` 、 `team_id` が複合キーで、それぞれ外部キーを参照しているのがポイント。

### app/repositories/sequelize.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/sequelize-sample/app/repositories/sequelize.js?footer=0"></script>

### app/repositories/db.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/sequelize-sample/app/repositories/db.js?footer=0"></script>

`belongsToMany` を使用して中間テーブルを介したモデルを構築する際は必ず `through` を記載する。  
`belongsToMany` でリレーションを構築するとユーザモデルから以下の関数を実行できるようになる。（チームモデルも同様）

- user.addTeam
    - 既存のteamテーブルのteamモデル（配列も可）をuserと関連付け。
- user.addTeams
    - 既存のteamテーブルのteamモデル（配列）をuserと関連付け。
- user.countTeams
    - userに関連付けされたteamを数え上げる。
- user.createTeam
    - teamオブジェクトを渡して、新規にteamテーブルにteamレコードを作成して且つ関連付け。
- user.getTeams
    - userと関連付けされたteamモデルを取得する。
- user.hasTeam
    - userに引数で与えたteamモデルが関連付けされているか確認する。（true/false）
- user.hasTeams
    - userに引数で与えたteamモデルが関連付けされているか確認する。（true/false）
- user.removeTeam
    - userと引数で与えたteamモデル（配列も可）の関連付けを削除する。
    - ただし、teamモデルは削除されない。
- user.removeTeams
    - userと引数で与えたteamモデル（配列）の関連付けを削除する。
    - ただし、teamモデルは削除されない。
- user.setTeams
    - **add との違いがわかってない。。。**

[参考](http://docs.sequelizejs.com/class/lib/associations/belongs-to-many.js~BelongsToMany.html)

### app/app.js

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/sequelize-sample/app/app.js?footer=0"></script>
