---
title: "Istio概念整理"
date: 2021-01-06T12:44:11+09:00
slug: "istio-concept"
tags:
- kubernetes
- istio
draft: true
archives:
- 2021/01
---

Istio でアプリケーションの設定する際の概念について整理する。

- トラフィック制御

<!--more-->

# トラフィック制御

- Inbound
    - Gateway、VirturlService
- Internal
    - DestinationRule、VirtualService
- Outbound
    - ServiceEntry

## Inbound

Istio クラスタ外からの通信について整理する。  
クラスタ外からの通信はまず **IngressGateway** が受信し、 **VirtualService** の設定に基づいてトラフィックが制御される。  
IngressGateway 事態の設定は **Gateway** にて行う。  
[参考](https://qiita.com/J_Shell/items/296cd00569b0c7692be7)

### Gateway

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: sample-gateway
spec:
  selector:
    istio: ingressgateway # デプロイされている IngressGateway のセレクタ
  servers:
  - port:                 # HTTP通信を全てのホストに許可する
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
  - port:                 # HTTPS通信をホストuk.bookinfo.comとeu.bookinfo.comのみ許可する
      number: 443
      name: https-443
      protocol: HTTPS
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      mode: SIMPLE # enables HTTPS on this port
      serverCertificate: /etc/certs/servercert.pem
      privateKey: /etc/certs/privatekey.pem
```

### VirtualService

Gateway の `spec.servers.hosts` 項目で設定したものを Service に流す設定を VirtualService で行う。

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo-rule
  namespace: bookinfo-namespace
spec:
  hosts:                                   # 対象となる Service 名を指定する
  - reviews.prod.svc.cluster.local
  - uk.bookinfo.com
  - eu.bookinfo.com
  gateways:
  - some-config-namespace/sample-gateway   # Gateway 名を指定すると、Gateway の VirtualServiceとして機能する。
  - mesh                                   # 「mesh」を指定すると Internal 通信として機能する。
  http:
  - match:                                 # 合致するリクエストの条件を記載する。
    - headers:
        cookie:
          exact: "user=dev-123"
    route:
    - destination:                         # トラフィックの流し先を指定。
        port:                              # 対象となる Service の Port。
          number: 7777
        host: reviews.qa.svc.cluster.local # 対象となる Service 名を指定する。Service が ClusterIP の場合、<service>.<namespace>.svc.cluster.local でもよい。
  - match:
    - uri:
        prefix: /reviews/
    route:
    - destination:
        port:
          number: 9080 # can be omitted if it's the only port for reviews
        host: reviews.prod.svc.cluster.local
      weight: 80
    - destination:
        host: reviews.qa.svc.cluster.local
      weight: 20
```

## Internal

Istio クラスタ内の通信について整理する。  
**DestinationRule** は対象となる Service 内に仮想敵にサブネットを定義し、 **VirturlService** にてどのサブネットにどの程度の流量を流すか設定する。  
[参考](https://blog.1q77.com/2020/03/istio-part3/)

### DestinationRule

以下は reviews.prod.svc.cluster.local という Service 内に `version: v1` `version: v2` という Label を持つ異なる 2 種類の Pod があり、それぞれを異なるサブネットに仮想的に分割する設定。

```
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews-destination-rule       # DestinationRule 名
spec:
  host: reviews.prod.svc.cluster.local # 対象の Service 名
  subsets:
  - name: v1                           # サブネット名
    labels:
      version: v1                      # サブネットに含める Pod を選択するセレクタ
  - name: v2
    labels:
      version: v2
```

### VirtualService

以下は先の DestinationRule で定義されたサブネットに対して 50:50 で流量を制御する VirturlService。

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews-virtual-service
spec:
  hosts:
  - reviews.prod.svc.cluster.local
  http:
  - route:
    - destination:
        host: reviews.prod.svc.cluster.local # 対象の Service 名
        subset: v1                           # 対象のサブネット名
      weight: 50                             # 全量を 100 とした流量の割合
    - destination:
        host: reviews.prod.svc.cluster.local
        subset: v2
      weight: 50
```

## Outbound

Istio クラスタ外への通信について整理する。

# 参考

- https://knowledge.sakura.ad.jp/20489/