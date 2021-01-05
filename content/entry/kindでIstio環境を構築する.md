---
title: "KindでIstio環境を構築する"
date: 2021-01-04T22:02:55+09:00
slug: "kind-istio"
tags:
- kubernetes
- kind
- istio
draft: false
archives:
- 2021/01
---

[kind](https://kind.sigs.k8s.io/) は **K**ubernetes **in** **D**ocker の略。文字通り、Docker上にKubernetes環境を構築する。

- kind のインストール
- Kubernetes 環境の構築
- Istio 環境の構築

<!--more-->

# kind のインストール

kind は Docker 上に Kubernetes 環境を構築するツールなので、 `docker` と `kubernetes-cli` は別途インストールする必要がある。

```
$ brew install docker --cask
$ brew install kubernetes-cli
```

kind のインストールは以下。

```
$ brew install kind
```

# Kubernetes 環境の構築

## シングルノードクラスタ

以下のコマンドでデフォルトのクラスタ名 `kind` の k8s クラスタが作成される。

```
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Thanks for using kind! 😊

$ kind get clusters
kind

$ kind get nodes
kind-control-plane
# kind クラスタの Control Plane が作成されていることがわかる

$ kind get kubeconfig
apiVersion: v1
clusters:
- cluster:
（省略）
```

上記の通り、 Control Plane が 1 ノードのシングルノードクラスタが作成されていることがわかる。  
クラスタ作成時のメッセージでもある通り、コンテキスト名は `kind-kind`。（ kind-<クラスタ名> がコンテキスト名）

```
$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop
*         kind-kind        kind-kind        kind-kind

$ kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:52154
KubeDNS is running at https://127.0.0.1:52154/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   73m   v1.19.1

$ kubectl get po -n kube-system
NAME                                         READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-72k66                      1/1     Running   0          81m
coredns-f9fd979d6-c9wff                      1/1     Running   0          81m
etcd-kind-control-plane                      1/1     Running   0          81m
kindnet-dscxn                                1/1     Running   0          81m
kube-apiserver-kind-control-plane            1/1     Running   0          81m
kube-controller-manager-kind-control-plane   1/1     Running   1          81m
kube-proxy-cgk7n                             1/1     Running   0          81m
kube-scheduler-kind-control-plane            1/1     Running   1          81m
```

## マルチノードクラスタ

以下のような設定ファイル（ `cluster.yaml` ）を作成する。

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

上記では Control Plane を 1 ノード、 Woker Nodes を 2 ノード作成する設定ファイルとなる。  
なお、 Control Plane を複数ノードで HA 構成にすることも可能。

```
$ kind create cluster --name istio --config cluster.yaml
Creating cluster "istio" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-istio"
You can now use your cluster with:

kubectl cluster-info --context kind-istio

Have a nice day! 👋

$ kind get clusters
istio

$ kind get nodes --name istio
istio-control-plane
istio-worker
istio-worker2

$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop
*         kind-istio       kind-istio       kind-istio

$ kubectl get nodes
NAME                  STATUS   ROLES    AGE     VERSION
istio-control-plane   Ready    master   5m21s   v1.19.1
istio-worker          Ready    <none>   4m51s   v1.19.1
istio-worker2         Ready    <none>   4m50s   v1.19.1

$ kubectl get ns
NAME                 STATUS   AGE
default              Active   2m50s
kube-node-lease      Active   2m52s
kube-public          Active   2m52s
kube-system          Active   2m52s
local-path-storage   Active   2m44s

$ kubectl get po -n kube-system
NAME                                          READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-pt7j7                       1/1     Running   0          5m45s
coredns-f9fd979d6-x78xd                       1/1     Running   0          5m46s
etcd-istio-control-plane                      1/1     Running   0          5m49s
kindnet-d6txw                                 1/1     Running   0          5m35s
kindnet-nqrlk                                 1/1     Running   0          5m34s
kindnet-x7frv                                 1/1     Running   0          5m45s
kube-apiserver-istio-control-plane            1/1     Running   0          5m49s
kube-controller-manager-istio-control-plane   1/1     Running   0          5m49s
kube-proxy-6zkfz                              1/1     Running   0          5m34s
kube-proxy-f4jmr                              1/1     Running   0          5m46s
kube-proxy-gjvg8                              1/1     Running   0          5m35s
kube-scheduler-istio-control-plane            1/1     Running   0          5m49s
```

# Istio 環境の構築

kind で作成した istio クラスタに対して Istio の [Getting Started](https://istio.io/latest/docs/setup/getting-started/)を参考に構築していく。

## Istio のダウンロード

```
$ curl -L https://istio.io/downloadIstio | sh -

$ ls -1 | grep istio
istio-1.8.1

$ cd istio-1.8.1
```

## `istioctl` の設定

```
$ export PATH=$PWD/bin:$PATH

もしくは

$ echo \\nexport PATH='$PATH':$PWD/bin >> ~/.zshenv
```

## Istio のインストール by istioctl

```
$ istioctl install --set profile=demo -y

Detected that your cluster does not support third party JWT authentication. Falling back to less secure first party JWT. See https://istio.io/v1.8/docs/ops/best-practices/security/#configure-third-party-service-account-tokens for details.
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete

$ kubectl get po -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
istio-egressgateway-d84f95b69-f8r44     1/1     Running   0          7m25s
istio-ingressgateway-75f6d79f48-4m5mn   1/1     Running   0          7m25s
istiod-c9f6864c4-bq8cx                  1/1     Running   0          7m54s
```

isitod、Ingress Gateway、Egress Gatewayがインストールされているのがわかる。  
アプリケーションをデプロイする namespace に label `istio-injection=enabled` を設定して、 Envoy サイドカーを自動インジェクトするよう設定する。

```
$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
```

## Istio のインストール by helm

namespace `istio-system` を作成する。

```
$ kubectl create namespace istio-system
namespace/istio-system created
```

Istio Control Plane により利用されるリソースを含む Base Chart をインストールする。

```
$ helm install -n istio-system istio-base manifests/charts/base
NAME: istio-base
LAST DEPLOYED: Tue Jan  5 22:05:06 2021
NAMESPACE: istio-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

`istiod` をインストールする。

```
$ helm install --namespace istio-system istiod manifests/charts/istio-control/istio-discovery \
               --set global.hub="docker.io/istio" --set global.tag="1.8.1"
NAME: istiod
LAST DEPLOYED: Tue Jan  5 22:08:53 2021
NAMESPACE: istio-system
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ kubectl get po -n istio-system
NAME                      READY   STATUS    RESTARTS   AGE
istiod-7bc8f84cf6-v2blw   0/1     Pending   0          41s
```

Ingress Gateway をインストールする。

```
$ helm install --namespace istio-system istio-ingress manifests/charts/gateways/istio-ingress \
               --set global.hub="docker.io/istio" --set global.tag="1.8.1"
NAME: istio-ingress
LAST DEPLOYED: Tue Jan  5 22:12:41 2021
NAMESPACE: istio-system
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ kubectl get po -n istio-system
NAME                                    READY   STATUS              RESTARTS   AGE
istio-ingressgateway-6b68c5cd74-7x2vt   0/1     ContainerCreating   0          14s
istiod-7bc8f84cf6-v2blw                 0/1     Pending             0          4m1s
```

Egress Gateway をインストールする。

```
$ helm install --namespace istio-system istio-egress manifests/charts/gateways/istio-egress \
               --set global.hub="docker.io/istio" --set global.tag="1.8.1"
NAME: istio-egress
LAST DEPLOYED: Tue Jan  5 22:14:13 2021
NAMESPACE: istio-system
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ kubectl get po -n istio-system
NAME                                    READY   STATUS              RESTARTS   AGE
istio-egressgateway-86dc59b567-xkth9    0/1     ContainerCreating   0          110s
istio-ingressgateway-6b68c5cd74-7x2vt   0/1     ContainerCreating   0          3m23s
istiod-7bc8f84cf6-v2blw                 0/1     Pending             0          7m10s
```

アプリケーションをデプロイする namespace に label `istio-injection=enabled` を設定して、 Envoy サイドカーを自動インジェクトするよう設定する。

```
$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
```

## デモアプリをデプロイ

```
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

参考：[Getting Started](https://istio.io/latest/docs/setup/getting-started/)