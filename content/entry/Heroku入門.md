---
title: "Heroku入門"
date: 2017-12-07T07:56:43+09:00
slug: "heroku-basics"
tags:
- heroku
draft: false
archives:
- 2017/12
---

Heroku は salesforce が運営する PaaS 。  
Free プランではクレカ登録無しで月に 550 dyno hours （約 23 日分、 1 ヶ月分無い）、クレカ登録すると月に 1000 dyno hours を無料で利用できる。  
Free プランの落とし穴は、 30 分アクセスが無いとインスタンス（ dyno ）が Sleep してしまうところ。　　
以下、 Free プランでアカウントを取得している前提で記載する。

<!--more-->

# Heroku CLI

Heroku の Web ページで各種設定できるが、ここでは CLI ベースの方法を記載する。

## インストール

```sh
$ brew install heroku/brew/heroku
```

[Trouble Shooting](https://devcenter.heroku.com/articles/heroku-cli#troubleshooting)  
[Uninstalling](https://devcenter.heroku.com/articles/heroku-cli#uninstalling-the-heroku-cli)

## 初期起動

以下の手順で Heroku へアプリケーションをデプロイ・起動する。  
なお、 `/path/to/myApp` は Heroku が対応している言語で実装されたローカルのアプリケーションプロジェクトをさす。  
また、 Heroku へのアプリケーションのデプロイは Git 経由で行われる。

```sh
$ heroku login # Heroku にログイン
$ cd /path/to/myApp
$ heroku create # Heroku 上に新しいアプリケーションを作成
Creating app... done, ⬢ shielded-scrubland-xxxxx
https://shielded-scrubland-xxxxx.herokuapp.com/ | https://git.heroku.com/shielded-scrubland-xxxxx.git
$ git init # アプリケーションプロジェクトの Git 初期化
$ git remote add heroku https://git.heroku.com/shielded-scrubland-xxxxx.git
$ git add --all
$ git commit -m "first commit"
$ git push heroku master # Heroku へアプリケーションをデプロイ
$ heroku open # ブラウザを立ち上げてページを表示
$ heroku help # USAGE
$ heroku apps help # apps の USAGE
```

## アドオン

アドオンを CLI から適用する方法は以下。

```sh
$ heroku addons:create xxxx
```

## プロセスの確認、停止

```sh
$ heroku apps:info # アプリケーションの情報を見る
$ heroku ps # プロセスを見る
$ heroku logs # ログを見る
```

```sh
$ heroku ps:scale web=0 # 停止
$ heroku ps:scale web=1 # 起動
```

Webプロセスのスケール（Dyno、インスタンス台数）を 0 に指定して実質プロセスを停止している。

# 小ネタめも

## node.js アプリケーション

`$ heroku open` でエラーが発生した場合は以下に対応。

- npm script に **start** を追加する
    - `"start": "node app/app.js"`

Heroku ではアプリケーションの起動方法に各プログラミング言語毎にルールがあるので注意。

## １つのアプリケーションを複数のHeroku環境へデプロイ

```sh
$ heroku create アプリ名 --remote 新環境名
$ git push 新環境名 master
```