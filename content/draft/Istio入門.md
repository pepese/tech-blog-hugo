---
title: Istio入門
tags:
- kubernetes
- Istio
- Kiali
id: k8s-istio-basics
draft: true
---

[Istio](https://istio.io/) についてまとめる。

# 概要

<img src="https://istio.io/docs/concepts/what-is-istio/arch.svg" />

[参考](https://www.1915keke.com/entry/2018/10/06/042621)

- `Pilot` : サービスディスカバリと高度なトラフィックマネジメント(A/Bテスト、カナリアデプロイなど)を提供する。
- `Proxy` : Envoy Proxyを使ってSidecarとしてデプロイをしている。
- `Mixer` : プラットフォームに依存するもの。アクセスコントロールやサービスメッシュに渡る制御をしたり、それぞれのEnvoyやサービスからテレメトリーを収集する。
- `Citadal`: サービス間とエンドユーザー認証を行う。
- `Galley` : Control Planeを代表してユーザー認証されたIstio APIを提供する。

# ローカル環境構築

Homebrew で構築できるものは以下。

```bash
$ brew install awscli
$ brew install terraform
$ brew install kubernetes-cli
$ brew install kubernetes-helm
$ brew install helmfile
$ helm plugin install https://github.com/databus23/helm-diff --version master
```

Istio のコマンドライン資材の設定は以下。

```bash
$ cd
$ curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.3.1 sh -
$ cd istio-1.3.1
$ export PATH=$PWD/bin:$PATH
$ istioctl verify-install
```

# k8s 環境構築

k8s クラスタは構築されている前提。

## Tiller 構築

```bash
$ kubectl apply -f ~/istio-1.3.1/install/kubernetes/helm/helm-service-account.yaml # tiller の Service Account 作成
$ helm init --history-max 200 --service-account tiller                             # tiller の作成
$ kubectl get all --all-namespaces -o wide | grep tiller                           # tiller の構築を確認
kube-system   pod/tiller-deploy-7695cdcfb8-2xf4l   1/1     Running   0          40s    10.0.65.144   ip-10-0-65-119.ap-northeast-1.compute.internal   <none>           <none>
kube-system   service/tiller-deploy   ClusterIP   172.20.27.245   <none>        44134/TCP       40s    app=helm,name=tiller
kube-system   deployment.apps/tiller-deploy   1/1     1            1           40s    tiller       gcr.io/kubernetes-helm/tiller:v2.14.3                                  app=helm,name=tiller
kube-system   replicaset.apps/tiller-deploy-7695cdcfb8   1         1         1       40s    tiller       gcr.io/kubernetes-helm/tiller:v2.14.3                                  app=helm,name=tiller,pod-template-hash=7695cdcfb8
```

## istio-init

```bash
$ helm install ~/istio-1.3.1/install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
$ kubectl get crds | grep 'istio.io' | wc -l
      23
$ helm ls --all                              # 確認２
NAME      	REVISION	UPDATED                 	STATUS  	CHART           	APP VERSION	NAMESPACE
istio-init	1       	Thu Oct  3 21:02:53 2019	DEPLOYED	istio-init-1.3.1	1.3.1      	istio-system
```

## Istio の構築

```bash
$ helm install ~/istio-1.3.1/install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values ~/istio-1.3.1/install/kubernetes/helm/istio/values-istio-demo.yaml # Istio 構築
$ helm install ~/istio-1.3.1/install/kubernetes/helm/istio --name istio --namespace istio-system # Istio 構築
$ kubectl label namespace default istio-injection=enabled                       # サイドカーを自動でつける設定
$ kubectl get namespace -L istio-injection                                      # 確認
$ kubectl apply -f <your-application>.yaml                                      # アプリのデプロイ
```

### Prometheus

```bash
$ kubectl -n istio-system get pod -l app=prometheus                          # Pod 名を確認
NAME                          READY   STATUS    RESTARTS   AGE
prometheus-776fdf7479-zhqbc   1/1     Running   0          7m15s
$ kubectl -n istio-system get svc -l app=prometheus                          # ポートを確認
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
prometheus   ClusterIP   172.20.55.63   <none>        9090/TCP   12m
$ kubectl -n istio-system port-forward prometheus-776fdf7479-zhqbc 9090:9090 # port forward
$ curl http://localhost:9090                                                 # ブラウザでアクセス
```

一撃だと `$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090` 。

### Grafana

```bash
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000
$ curl http://localhost:3000/dashboard/db/istio-mesh-dashboard
```

### Kiali

まずはユーザ/パスワードを作成して secret を適用する。

```bash
$ KIALI_USERNAME=$(read -p 'Kiali Username: ' uval && echo -n $uval | base64)
Kiali Username: # ユーザ名を入力する
$ KIALI_PASSPHRASE=$(read -p 'Kiali Passphrase: ' pval && echo -n $pval | base64)
Kiali Passphrase: # パスワードを入力する
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```

以下でアクセスして、上記で作成したユーザ/パスワードを利用する。

```bash
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
$ curl http://localhost:20001/kiali/console
```

なんかログインできない、、、

# 解説

```bash
.
├── README.md
├── helmfile.yaml
└── istio-values.yaml
```

```yaml:helmfile.yaml
repositories:
  - name: elastic
    url: https://helm.elastic.co
  - name: kiwigrid
    url: https://kiwigrid.github.io
  - name: codecentric
    url: https://codecentric.github.io/helm-charts

releases:
  # 前提として、istio-init が完了している必要がある
  # https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio
  - name: istio
    namespace: istio-system
    chart: istio.io/istio
    values:
    - istio-values.yaml
```

```yaml:istio-value.yaml
gateways:
  enabled: true
  istio-ingressgateway:
    type: NodePort

global:
  proxy:
    autoInject: disabled

grafana:
  enabled: true
  ingress:
    enabled: true
    hosts:
      - istio-grafana.sample.com
  contextPath: /
  security:
    enabled: true

kiali:
  enabled: true
  ingress:
    enabled: true
    hosts:
      - istio-kiali.sample.com
  dashboard:
    jaegerURL: https://istio-jaeger.sample.com
  createDemoSecret: true

tracing:
  enabled: true
  ingress:
    enabled: true
    hosts:
      - istio-jaeger.sample.com
  contextPath: /
  jaeger:
    persist: true
    accessMode: ReadWriteOnce
```

Helm から構築する Istio は、 Istio では飽き足らず、様々な機能を構築することが可能。

- certmanager
  - TLSの証明書を自動で生成し管理するK8sのアドオン
- galley
  - Istio のコンポーネント
- gateways
  - Istio のコンポーネント
  - Istio Ingress Gateway
- global
- grafana
- istio_cni
- istiocoredns
- kiali
- mixer
  - Istio のコンポーネント
- nodeagent
- pilot
  - Istio のコンポーネント
- prometheus
- security
- sidecarInjectorWebhook
- tracing

設定については [ここ](https://istio.io/docs/reference/config/installation-options/) を参照。

# 参考

- Istio 公式
  - [helm 利用](https://istio.io/docs/setup/install/helm/)
  - [Installation Options](https://istio.io/docs/reference/config/installation-options/)
- 参考
  - [マイクロサービスアーキテクチャ向けにサービスメッシュを提供する「Istio」の概要と環境構築、トラフィックルーティング設定](https://knowledge.sakura.ad.jp/20489/)
  - [Istio入門 その1 -Istioとは?-](https://qiita.com/Ladicle/items/979d59ef0303425752c8)
  - [Istio導入のメリットとハマりどころを、実例に学ぶ〜マイクロサービス化の先にある課題を解決する](https://employment.en-japan.com/engineerhub/entry/2019/05/21/103000)
  - [Istio IngressGateway周辺を理解する](https://qiita.com/J_Shell/items/296cd00569b0c7692be7)
  - [サービスメッシュを実現するIstioをEKS上で動かす - その4 データの可視化について](https://tech.recruit-mp.co.jp/infrastructure/post-19190/)