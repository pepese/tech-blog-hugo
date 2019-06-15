---
title: Kubernetes のアーキテクチャ整理
tags:
- Kubernetes
- k8s
id: k8s-architecture
draft: true
---

**Kubernetes** （ **k8s** ）のアーキテクチャについて整理する。

# サーバ構成

k8s ではホストマシン（物理サーバ、もしくは仮想マシン、インスタンス）として **Master Node(s)** と **Worker Node(s)** がある。  
それぞれで実行されるコンポーネントは以下の通り。各コンポーネントはそれぞれ各ノードで独立したプロセスとして常駐する。

- **Master Node(s)** ： 管理サーバ、クライアントツール（ **kubectl** ）からこのノードを経由して Worker をコントロールする
    - `etcd` （ Master Node ではなく独立したクラスタでもいい ）
    - `kube-apiserver`
    - `kube-scheduler`
    - `kube-controller-manager`
- **Worker Node(s)** ： Master からコントロールされるワーカーサーバ、複数台で構成されるクラスタ構成をとり、コンテナがマウントされる
    - `kubelet`
    - `kube-proxy`
    - `kube-dns` （ Worker Node ではない？）
- その他のコンポーネント
    - `cloud-controller-manager` （α版）
        - クラウドプロバイダのリソースをコントロールする
    - `hyperkube`
        - k8s 関連のバイナリを1つにまとめたall-in-oneバイナリ
    - cAdvisor
        - 各ノード上にあるコンテナのCPU、メモリ、ファイル、ネットワーク使用量といった、リソースの使用量と性能の指標を監視・収集するエージェント
- クライアントツール
    - kubectl
        - k8s クラスタを管理するコマンド
    - kubeadm
        - 物理もしくは仮想サーバに対して Kubernetes 環境を構築するツール
    - kubefed
        - Federation を管理するコマンド
        - Federationは複数のリージョンやクラウドに配置されているKubernetesのクラスタを一括で管理できるようにする機能
    - Minikube
        - 1 台のローカル端末で シングル Node の Kubernetes Cluster をお試しできるツール
    - Dashboard
        - k8s クラスタの可視化を行う Web ベースのツール

## コンポーネント

ここでは先ほど紹介した主要 7 コンポーネントについて記載する。

### etcd

k8s クラスタの全ての情報が保存される分散 KVS 。  
`etcd` へアクセスするコンポーネントは `kube-apiserver` のみで、他のコンポーネントは `kube-apiserver` を介してアクセスする。  
分散合意アルゴリズム Raft の特性上、奇数台で冗長構成にする必要がある。  
`etcdctl shapshot` コマンドなどで定期的にバックアップしておく。  
なお、 Master Node 上ではなく、独立した `etcd` クラスタ構成としてもよい。

### kube-apiserver

k8s クラスタをコントロールする API を提供するコンポーネント。  
REST API を提供しているが、通常はクライアントツール `kubectl` でコントロールする。`kubectl` から設定情報を受け付けて、 クラスタが保持すべき状態として情報を `etcd` へ保存する。  
`kube-apiserver` 以外のコンポーネントは直接 `etcd` を参照せず、 `kube-apiserver` を通してリソースにアクセスする。  
コンテナが作成されるまでのフロー例は以下のようになる。

1. `kubectl` で Deploment API を呼ぶ
2. Deployment Controller が検知し、 ReplicaSet API を呼ぶ
3. ReplicaSet Controller が検知し、 Pod API を呼ぶ
4. Scheduler が検知し、配置先 Node を決定し、 Pod API を呼ぶ
5. Kubelet が検知し、 Pod を作成
    - 各 Controller 、 Scheduler 、 Kubelet は常にそれぞれが Reconcilation Loop を回している

### kube-scheduler

`kube-scheduler` の仕事は、新規に `etcd` に登録された Pod に対して最適な Worker Node を選択して割り当てること。  
`go routine` の無限ループ（間隔 0 ）で以下の処理を繰り返す。

1. `kube-apiserver` 経由で `etcd` にアクセスし、ノードへ未割り当ての Pod 情報を 1 つ取得する
2. Pod の設定情報から最適な Woker Node を選択する
3. 選択した Worker Node の情報を`kube-apiserver` 経由で `etcd` にアクセスし、保存する

Worker Node の選択は、 `predicates` と呼ばれる条件にによるフィルタと、 `priorities` という優先度付けによって行われる。

- `predicates`
    - Worker Node をフィルタする条件。
    - 例えば nodeSelector によるノードの選択や request に対するリソースの空き状況によるフィルタがある。
- `priorities`
    - Worker Node の優先度付け。
    - 例えば Pod を出来る限りノード、ゾーンを分散させるような優先度付けがある。

参考：https://qiita.com/tkusumi/items/58fdadbe4053812cb44e

### kube-controller-manager

`etcd` の情報とクラスタの現在の状態を比較し、 `etcd` の状態へ更新する（ `kube-apiserver` 経由での参照）。
Controller は自作可能であるが、デフォルトで以下のような Controller がある。

- Node Controller
    - Worker Node の停止を検出する
- Deployment Controller
- Replication Controller （ ReplicaSet Controller ？）
    - Pod の数を制御する
    - Replication Controller はリソース名自体に"Controller"とつくため、コントローラーは"RepplicationManager"という名前
- Endpoints Controller
    - Service や Pod 内のエンドポイントオブジェクトを設定する
- Service Account & Token Controllers
    - namespace のデフォルトアカウント、API アクセストークンを作成する

なお、 Controller は複数プロセス存在するが、バイナリとしては 1 つにまとめられる。
また、各 Controller は `go routine` で起動し、 `kube-apiserver` 経由で `etcd` へアクセスし、登録情報と現在のクラスタの状態を比較し続ける。  
`kube-controller-manager` は状態の差分を検出、 `etcd` へ登録するが、実際のクラスタへの反映作業は `kubelet` が行う。

### kubelet

クラスタ内のそれぞれの Worker Node で稼働するエージェントで、実際にコンテナを起動・停止する。PodSpecs の設定に基づいて、 Pod 内のコンテナの稼働を管理する。  
`kube-scheduler` がによって Pod が起動される Worker Node が決定した事を検知して処理を行う。  
また、 ConfigMap や Secret の設定更新は `kubelet` の Sync Loop （ `--node-status-update-frequency duration` というオプションがあり、デフォルト 10s ）のタイミングで反映される。  
`kubelet` は複数のコンテナランタイムを扱うことができ、以下のようなものがある。

- 高位コンテナランタイム： `kubelet` から利用されるランタイム
    - docker-shim (Docker)
    - containerd （ Docker 内部でも利用されている）
    - cri-o
    - Frakti
    - rktlet (rkt)
- 定位コンテナランタイム： 高位コンテナランライムから利用されるランタイム
    - runc
    - runv
    - runsc (gVisor)
    - kata-runtaime ( Kata Containers )

上記のような複数のコンテナランタイムを利用できる背景には、 OCI ( Open Container Initiative) や CRI ( Container Runtime Interface ) などの標準化が進んでいることが挙げられる。

### kube-proxy

k8s クラスタの各 Worker Node に配備され、トラフィックのルーティングを担当する。  
`kube-apiserver` をポーリングして設定変更を検知する。  
ホスト上のネットワークルールを維持し、接続転送を実行することによって、Kubernetesサービスの抽象化を可能にしている。  
転送方式には以下の 3 つがあり、 `--proxy-mode` で指定できる。

- userspace mode
    - 受信したトラフィックを userspace 上のプロセスが処理する
- iptables mode
    - カーネルの iptable の nat table を設定して IP 変更やロードバランス（ClusterIP）の設定を行う
- ipvs mode

また、  Worker Node に跨った Pod 間での通信を可能にするため **オーバーレイネットワーク** を構築する必要がある。  
`kube-proxy` は以下のような CNI Plugin と連携してオーバーレイネットワークを構築する。

- Flannel
- Open vSwitch
- Project Calico
- Cilium
- Contiv (Cisco が開発？)
- Romana
- Weave Net

### kube-dns

k8s クラスタの名前解決やサービスディスカばりを担当するクラスタ内の DNS サーバ。  
`kube-apiserver` をポーリングして設定変更を検知する。  
`kube-dns` の後継として CoreDNS がある。