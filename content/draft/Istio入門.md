---
title: Istio入門
tags:
- kubernetes
- Istio
- Kiali
id: k8s-istio-basics
draft: true
---

- https://qiita.com/toshiki1007/items/896c79d94755686e57cd

[Istio](https://istio.io/) についてまとめる。

<img src=""https://istio.io/docs/concepts/what-is-istio/arch.svg />

[参考](https://www.1915keke.com/entry/2018/10/06/042621)

- `Pilot` : サービスディスカバリと高度なトラフィックマネジメント(A/Bテスト、カナリアデプロイなど)を提供する。
- `Proxy` : Envoy Proxyを使ってSidecarとしてデプロイをしている。
- `Mixer` : プラットフォームに依存するもの。アクセスコントロールやサービスメッシュに渡る制御をしたり、それぞれのEnvoyやサービスからテレメトリーを収集する。
- `Citadal`: サービス間とエンドユーザー認証を行う。
- `Galley` : Control Planeを代表してユーザー認証されたIstio APIを提供する。


# サイドカーインジェクト

これが重要。

```bash
#istioctl kube-injectでサイドカー(istio-proxy)を組み込んだManifestを作成し、そのManifestを使ってデプロイする
$ kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
```