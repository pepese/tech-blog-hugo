---
title: "Prometheus入門"
date: 2019-10-04T11:02:49+09:00
slug: ""
tags:
- prometheus
draft: true
archives:
- 2019/10
---

今更 [Prometheus](https://prometheus.io/) に入門する。

<!--more-->

# 概要

- メトリクスベースモニタリングシステム
- Pull 型
- **exporter** （エクスポータ）で様々な製品のメトリクス取得が可能
  - Prometheus からリクエストを受け、アプリケーションから情報取得・フォーマット変換をしてレスポンス
- クエリ言語は PromQL
  - ビジュアライズ、アラートに利用可能
- k8s などの各種オーケストレーションツールのサービスディスカバリ機能を利用してメトリクス取得するサーバ・アプリを特定
  - サービスディスカバリから得たメトリクスデータがモニタリングターゲットやそのラベルとどのように対応しているかを *リラベル* によって設定できる
- サービスディスカバリによって得られた監視対象リストを *スクレイプ* してメトリクス収集することで監視を行う
- データはローカルのカスタムデータベースに格納される
  - 分散システム・クラスタリングされておらず、逆に信頼性が高い
  - 代わりにリモート読み出し・書き込み API がある
- Prometheus はダッシュボートを内臓するが、 Grafana を利用するのがよい
- レコーディングルールは PromQL を定期実行することで成り、アラートルールはその形態の 1 つ
- Alertmanager が Promethus からアラートを受信してアラート管理を行う

<img src="https://prometheus.io/assets/architecture.png" />

# exporter

- Node exporter
  - Linux マシンの CPU 、 Memory 、 Disc Space/IO 、 Network などのメトリクスを取得可能
  - 個別のプロセスのメトリクスには対応していない

上記のような感じで他にも [いろいろ](https://prometheus.io/docs/instrumenting/exporters/) 。

# ラベル

- 時系列データに関連づけられたキーバリューペア
- メトリクス名とともに時系列データを一意に識別

# [Prometheus Operator](https://github.com/coreos/prometheus-operator)

- Kubernetes の CustomResourceDefinition 機能を利用
- Prometheus や Alertmanager の実行も管理もしてくれる

つまり、 Kubernetes のマニフェストファイルを利用して Prometheus や Alertmanager の設定・定義を可能にしてくれるもの。

# [Prometheus Monitoring Mixins](https://monitoring.mixins.dev/)

- Grafanaのダッシュボード、Prometheusのルール・アラートを再利用可能な形でまとめたもの
- [jsonnet](https://jsonnet.org/) で記載されていて[jsonnet-bundler](https://github.com/jsonnet-bundler/jsonnet-bundler)でインストール・更新可能