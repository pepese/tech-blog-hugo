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

- Control Plane
  - `Mixer` : Envoy を通して各サービスのデータを収集し、その情報を元にアクセスコントロールやサービスメッシュに渡る制御を行う。
  - `Pilot` : サービスディスカバリと高度なトラフィックマネジメント(A/Bテスト、カナリアデプロイなど)を提供する。
  - `Galley` : Control Planeを代表してユーザー認証されたIstio APIを提供する。
  - `Citadel`: サービス間とエンドユーザー認証を行う。
- Data Plane
  - `Proxy` : Envoy を使って レイヤー4/7 のイン・アウト両方のサービスメッシュを提供し、 Pod に Sidecarとしてインジェクションされる。

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
$ kubectl get all --all-namespaces -o wide | grep tiller | wc -l                   # tiller の構築を確認
       4
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
$ kubectl label namespace default istio-injection=enabled                         # サイドカーを自動でつける設定
$ kubectl get namespace -L istio-injection                                        # 確認
$ kubectl apply -f <your-application>.yaml                                        # アプリのデプロイ
```

上記では default namespace に `istio-injection=enabled` ラベルを付与して自動でサイドカー（ envoy ）インジェクションするように設定している。  
なお、自動でサイドカーインジェクションされるための条件は [ここ](https://istio.io/docs/ops/setup/injection-concepts/) 。


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
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 # port forward
$ curl http://localhost:3000/dashboard/db/istio-mesh-dashboard
```

### Kiali

まずはユーザ/パスワードを作成して secret を適用する。  
（`createDemoSecret: true` に設定した場合、 ID/Pass は admin/admin ）

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
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001 # port forward
$ curl http://localhost:20001/kiali/console
```

### Istio Ingress Gateway

従来、Kubernetes は外部からクラスタに入るトラフィックを処理するために **Ingress Controller** 使用していた。Istioを使用する場合、これはもはや当てはまない。  
Istio は、使い慣れた Ingress リソースを新しい Gateway および VirtualServices リソースに置き換えた。  
これらは連携して動作し、トラフィックをメッシュにルーティングする。  
メッシュ内では、サービスはクラスタローカルサービス名で相互にアクセスできるため、ゲートウェイは必要ない。  
以下のように動作する。

<img src="https://blog.jayway.com/wp-content/uploads/2018/10/istio-networking.png" />

1. クライアントは特定のポートにリクエストを発行する
2. LoadBalancer がクラスタ内の woker node へトラフィックをフォワードする
  - ここでの LoadBalancer は状況（ IP が必要 or Nor ）によって NodePort でもよい
    - 例えば AWS の Autoscaling Group だと IP は意識しなくていいので NodePort でよい
  - Istio IngressGateway へトラフィックを流すためだけの役割で、手動もしくは自動で設定する
3. Istio IngressGateway の Service/Deployment(Pod) が LoadBalancer からのリクエストを受ける
  - Istio IngressGateway は type を LoadBalancer/NodePort/ClusterIP から選択できるため、 NodePort として前段の LoadBalancer を省略することもできる
4. Istio IngressGateway の Pod が Gateway および VirtualService の設定に応じてリクエストを処理する
  - Gateway では、ポート、プロトコル、および証明書の設定を行う
  - VirtualService は、 アプリケーションの Service へリクエストをルーティングするための設定を行う

Istio IngressGateway の他に **Istio EgressGateway** （外部への HTTP(S) 通信用）、 **Istio IblGateway** （クラスタ内 HTTP(S)/gRPC 通信用）が利用できる。

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
  - Istio IngressGateway/IblGateway/EgressGateway
- global
  - Istio の共通設定
- grafana
  - 可視化ツール、 Prometheus とともに
- istio_cni
  - CNI (Container Network Interface) の設定
- istiocoredns
  - Istio サービスの namespace で利用できるDNS の設定
  - kube-dns (CoreDNS) のサブドメインとして可動させたりできる
- kiali
  - リクエストのトレースを行う
- mixer
  - Istio のコンポーネント
- nodeagent
  - ノード単位のセキュリティ機能・設定を提供するエージェント
- pilot
  - Istio のコンポーネント
- prometheus
  - メトリクス監視ツール
- security
  - セキュリティの設定
- sidecarInjectorWebhook
  - [Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) という kube-api コールの際のインターセプト機能を利用
  - [Automatic sidecar injection](https://istio.io/docs/setup/additional-setup/sidecar-injection/#automatic-sidecar-injection) を実現する
- tracing
  - リクエストのレイテンシを追跡する
  - Jeager

設定については [ここ](https://istio.io/docs/reference/config/installation-options/) を参照。

# 参考

- Istio 公式
  - [helm 利用](https://istio.io/docs/setup/install/helm/)
  - [Installation Options](https://istio.io/docs/reference/config/installation-options/)
- 全般理解
  - [Istioの全貌](https://thinkit.co.jp/article/14640?page=0%2C1)
- 設定理解
  - [マイクロサービスアーキテクチャ向けにサービスメッシュを提供する「Istio」の概要と環境構築、トラフィックルーティング設定](https://knowledge.sakura.ad.jp/20489/)
  - [Istio 1.0 を試してみた！](https://medium.com/google-cloud-jp/istio-1-0-%E3%82%92%E8%A9%A6%E3%81%97%E3%81%A6%E3%81%BF%E3%81%9F-d74f75eeb1b1)
- Istio IngressGateway
  - [Understanding Istio Ingress Gateway in Kubernetes](https://blog.jayway.com/2018/10/22/understanding-istio-ingress-gateway-in-kubernetes/)
  - [Istio IngressGateway周辺を理解する](https://qiita.com/J_Shell/items/296cd00569b0c7692be7)
  - [Istio導入のメリットとハマりどころを、実例に学ぶ〜マイクロサービス化の先にある課題を解決する](https://employment.en-japan.com/engineerhub/entry/2019/05/21/103000)
- 可視化
  - [サービスメッシュを実現するIstioをEKS上で動かす - その4 データの可視化について](https://tech.recruit-mp.co.jp/infrastructure/post-19190/)