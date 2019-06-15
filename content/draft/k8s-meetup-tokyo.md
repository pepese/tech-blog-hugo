---
title: k8s meetup tokyo
tags:
- Kubernetes
- k8s
id: k8s-meetup-tokyo
draft: true
---

- https://k8sjp.connpass.com/event/116799/

# 19:05~19:35 Z氏は、Kubernetesクラスタを手中に収めることができたのか (30min)

CVE-2018-1002105 の話。  
**reconciliation loop** ： kube-apiserver を中心としたループ。
しかし以下の例外がある。

1. exec/attach/port_forward
    - Pod に対する直接的な命令（kube-apiserver -> kubelet API）
    - kube-apiserver が適切に命令を
2. Kubernetes API の拡張方法
    1. Custom Resource xxxx （CRD）：
    2. Aggregated API Server：kube-apiserver のリバプロ的存在
        - Service Catalog、Metrix Server
        - Aggregator が認証プロキシとして動作する（ Aggregator -> kube-apiserver の構成）

今回は Aggregator の脆弱性。  
Aggregator に対してTPC接続（sslクライアントでできる）の後 HTTP Upgrade が失敗しても、Aggregator のエラーハンドリングミスで kube-apiserver に対する TCPコネクションが残っている。

## Aggregator の参考

- http://bbrfkr.hatenablog.jp/entry/2018/05/26/211800
- https://github.com/kubernetes/kube-aggregator

# 19:35~20:05 CNI 入門 (30min)：LINE 市原ゆうし

Worker Node 内には「Router + FW + LB + Switch」の仕組みが据えて詰まっている。  
CNI(Container Network Interface) として IF が整理されている。

- CNI Spec
- CNI Plugins

CNM(Container Network Model)はあるが、 CNI 一強状態かな。

kubelet -> CNI Plugin という流れで、 CNI Plugin に対するインプットは標準入力と環境変数、アウトプットは標準出力。  
また、 CNI チェインという形で複数の CNI Plugin を数珠つなぎで実行できる。

- ADD：ネットワーク追加
- DEL：ネットワーク削除
- CHECK（旧GET）
- VERSION

## インプット：環境変数

- CNI_COMMAND
- CNI_CONTAINERID
- CNI_NETNS
- CNI_IFNAME
- CNI_ARGS
- CNI_PATH：CNI Plugin のパス

## インプット：標準入力（基本 JSON ）

- cniVersion (string)
- name (string)
- type (string)
- args (dictionary, optional)
- ipMasq (boolean, optional)
- ipam (dictionary, optional)
- dns (dictionary, optional)

## CNI base plugins

- Main
    - bridge, macvlan, ipvlan, host-devicd, ptp, Windows
- IPAM
    - host-local, DHCP, static
- Meta
    - bandwidth, flannel, portmap, tuning, sbr

## CNI Plugin の作り方

cnitool

```bash
$ go install github.com/containernetworking/cni/cnitool


$ sudo ip netns add testing
```

# 大規模Kubernetesクラスタ向けにCNIプラグインを自作した話 (30min)

分散ストレージソフトウェア [ceph](https://ceph.com/) とかも使ってる。

- https://github.com/cybozu-go/neco
- ネットワーク構成
    - CLOS アーキテクチャ
    - BGP Router

以下を実装した。

- [coil](https://github.com/cybozu-go/coil)
- [Neco プロジェクトのスキルチェックシート](https://gist.github.com/ymmt2005/bd92296166e52d1beba9df8ac516a9db)

# CNI

https://qiita.com/yuanying/items/68b2a32b4d217e679955

## 下準備

Mac の場合は `ip` コマンド（ネットワークデバイズの IP アドレスを管理）をインストールする。

```
$ brew install iproute2mac
$ ip
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip -V
where  OBJECT := { link | addr | route | neigh }
       OPTIONS := { -4 | -6 }

iproute2mac
Homepage: https://github.com/brona/iproute2mac
This is CLI wrapper for basic network utilities on Mac OS X inspired with iproute2 on Linux systems.
Provided functionality is limited and command output is not fully compatible with iproute2.
For advanced usage use netstat, ifconfig, ndp, arp, route and networksetup directly.
```

`ip netns` とか無いので `ifconfig` で[代用](https://www.atmarkit.co.jp/ait/articles/0109/29/news004.html)しよう。