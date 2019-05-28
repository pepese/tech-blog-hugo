---
title: "Watcherまとめ"
date: 2019-05-28T13:37:09+09:00
slog: elasticsearch-watcher-basics
tags:
- aws
- elasticsearch
- kibana
- watcher
draft: true
---

[Watcher](https://www.elastic.co/guide/jp/x-pack/current/watcher-getting-started.html) についてまとめる。

<!--more-->

Kibana メニューの「 Management 」 -> Elasticsearch「Watcher」 。

# Watcher の設定

|構成要素|説明|
|---|---|
|Trigger|Watchの実行タイミングを規定。|
|Input|監視対象とするデータの取得方法を規定。|
|Condition|Inputで取得したデータから、Actionを実施するかどうかの識別・制御ルールを規定。基本的には監視対象のインデックスへ検索クエリを実行して情報を取得する。|
|Transform|Inputで取得したデータをActionsで利用するための処理を規定。Inputで取得したデータの整形等が必要ない場合は設定する必要はない。|
|Actions|Conditionで規定した識別・制御ルールを満たした場合の実行内容を規定。|

## Trigger

## Input

## Condition

## Transform

## Actions

# 参考

- https://qiita.com/zon_zone/items/93cda732d6c1ce74b8a3
- https://www.shin0higuchi.com/entry/2017/11/02/002955
- https://github.com/elastic/examples/tree/master/Alerting/Sample%20Watches