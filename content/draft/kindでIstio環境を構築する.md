---
title: "KindでIstio環境を構築する"
date: 2021-01-04T22:02:55+09:00
slug: ""
tags:
- tag_name
draft: true
archives:
- 2019/06
---

[kind](https://kind.sigs.k8s.io/) は **K**ubernetes **in** **D**ocker の略。文字通り、Docker上にKubernetes環境を構築する。

- kind のインストール
- Kubernetes 環境の構築
- Istio 環境の構築 ※[参考](https://istio.io/latest/docs/setup/platform-setup/kind/)

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

kind で作成した istio クラスタに対して Istio 環境を構築していく。