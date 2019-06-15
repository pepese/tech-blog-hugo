---
title: OpenShift入門
tags:
- OpenShift
id: openshift-basics
draft: true
---

OpenShift は Kubernetes の RedShift による拡張。  
`kubectl` ではなく `oc` コマンドを使う。

# 環境設定

```
$ brew install openshift-cli
```

# Istio on Kubernetes

- https://bit.ly/istio-intro
- https://github.com/redhat-developer-demos/istio-tutorial

## Service Mesh

サービスメッシュは、サービス間通信を処理するための専用インフラレイヤ。 最新のクラウドネイティブアプリケーションを構成する複雑なサービストポロジを使用して、リクエストを確実に配信する役割を担う。 通常、サービスメッシュは、通常、アプリケーションコードと一緒に展開される軽量ネットワークプロキシの配列として実装され、アプリケーションは認識する必要がない。

- Sidecar
    - 概念
    - Pod の中に Envoy のコンテナを置く
- Circuit Breakers

- Istio
    - Data Plane
    - Control Plane

- Istio Pilot が設定を配信
- Istio Mixer サービルやリクエストの情報をもってる
- Istio Citadel は TLS などでセキュリティを担保

# https://learn.openshift.com/training/servicemesh

## Istio Introduction

Istio は IP Table をいじって通信を横取りをする。