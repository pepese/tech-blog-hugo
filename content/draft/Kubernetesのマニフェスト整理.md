---
title: Kubernetesのマニフェスト整理
tags:
- kubernetes
id: k8s-manifest-basics
draft: true
---

kubernetes では `kubectl` に様々コマンドがあるが、基本的には `kubectl create/delete/apply` コマンドと **マニフェスト** とよぶ YAML/JSON ファイルを使用することで、リソースの作成/削除/更新できる。マニフェストのリファレンスは [ここ](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/) 。

- マニフェストの雛形
- マニフェストの理解

# マニフェストの雛形

ドライラン（ `kubectl run/create --dry-run` ）と yaml 表示（ `-o yaml` を）組み合わせるとマニフェストの雛形を標準出力できる。

``` bash
$ kubectl run sample --image nginx -o yaml --dry-run
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: sample
  name: sample
spec:
  replicas: 1
  selector:
    matchLabels:
      run: sample
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: sample
    spec:
      containers:
      - image: nginx
        name: sample
        resources: {}
status: {}
```


# マニフェストの理解

``` yaml:sample_pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: sample_pod
spec:
  containers:
    - name: nginx
      imege: nginx:1.7.9
      ports:
        - containerPort: 80
```

全てのマニフェストには以下の項目が含まれる。

- apiVersion
    - API のバージョン
- kind
    - リソースの種類
- metadata
    - リソースのメタデータ

Pod の場合は `spec` 以下に Pod の内容を定義する。

```
$ kubectl create -f sample_pod.yaml
pod "sample_pod" created

$ kubectl apply -f sample_pod.yaml
pod "sample_pod" configured

$ kubectl delete -f sample_pod.yaml
pod "sample_pod" deleted
```

`kubectl apply` はマニフェストに変更がある場合のみ変更処理を行う。  
削除済みのマニフェストに対しても差分を検出するため、新規作成でも `apply` を使用するほうがよい。  
また、マニフェストには複数のリソースを同時に記述することができ、 **`---`** で区切る。

```yaml:sample_pod.yml
---
apiVersion: v1
kind: Pod
metadata:
  name: sample_pod
spec:
  containers:
    - name: nginx
      imege: nginx:1.7.9
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
# LBの追加など
```

また、複数のマニフェストファイルを一気に適用したい場合は、ディレクトリにまとめ `-f` で指定して実行することができる。  
その歳、ファイル名辞書順で実行されることに注意。

## Label と Annotation

両者とも key-value 形式（ `key: value` ）でデータを保持する仕組みであるが、用途が異なる。

### Label

- 各リソースの `metadata` で設定する
- あるリソースが 1 つ以上のリソースを管理する場合、 `selector` にて Label を指定することにより対象リソースを発見する
  - ReplicaSet が Pod を管理する場合
  - Deployment が ReplicaSet を管理する場合
  - Service が受けた通信の転送先 Pod を指定する場合、など

### Annotation

- リソースオブジェクトに関する不特定の情報を保存する方法
  - オブジェクトの変更理由の記録
  - 特別なスケジューラへの、特別なスケジュールポリシーの伝達
  - リソースを更新したツールと、それがどのように更新したかの情報の付加
  - 他のツールからの更新検知などのため
  - Deployment オブジェクトによるロールアウトのための、ReplicaSet の追跡情報の保存、など

## namespace

namespace は k8s クラスタ上で論理的にシステムを分割する仕組み。  
namespace リソース（ `kind: Namespace` ）を作成し、各リソースの `metadata` にラベル付け（ `namespace: test` ）して利用する。

## Deployment

リソースには色々あるが、理解すべきリソースは少ない。そのうちの一つ。  
（その他は、Service、DaemonSet、CronJobくらい？）

```
apiVersion: app/v1
kind: Deployment
metadata: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#objectmeta-v1-meta
  annotations: # 任意のメタデータを保存および取得する。外部ツールなどによって設定・変更・保存される箇所でもある。
    key: value
  clusterName: string # オブジェクトが所属するクラスタ名。異なるクラスターで同じ名前と名前空間を持つリソースを区別するために使用。使わない。
  creationTimestamp: # このオブジェクトが作成されたサーバ時間。変更できない。
  deletionGracePeriodSeconds: # オブジェクトの正常終了までの時間。読み取り専用。
  deletionTimestamp: # オブジェクトが削除される時間。読み取り専用。
  finalizers: # リストからエントリを削除する責任のあるコンポーネントの識別子。削除のみ可能。
  generateName: # name が設定されていない場合にのみオプションで設定される名前。
  generation: # 状態の世代を表す。読み取り専用。
  initializers: # オブジェクトの作成時にシステムの不変条件を強制するコントローラー。DEPRECATED。
  labels: # ReplicaSet、Service の selector で利用。
    key: value
  managedFields: # 内部のワークフロー用でユーザは設定・理解の必要はない。
  name: # namespace 内で一意なオブジェクトの名前。
  namespace: # 名前空間名。空は default と同じ。 DNS_LABEL である必要があり、変更できない。
  ownerReferences: # このオブジェクトに依存するオブジェクトのリスト。コントローラにより管理される。
  resourceVersion: # オブジェクトがいつ変更されたかを判断するためにクライアントが使用できるこのオブジェクトの内部バージョン。読み取り専用。
  selfLink: # このオブジェクトを表すURL。読み取り専用。
  uid: # このオブジェクトの一意な ID 。システムにより管理される。
spec: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#deploymentspec-v1-apps
  minReadySeconds:
  paused:
  progressDeadlineSeconds:
  replicas:
  revisionHistoryLimit:
  selector: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#labelselector-v1-meta
    matchExpressions: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#labelselectorrequirement-v1-meta
    matchLabels:
  strategy: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#deploymentstrategy-v1-apps
    rollingUpdate: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#rollingupdatedeployment-v1-apps
      maxSurge:
      maxUnavailable:
    type:
  template: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#podtemplatespec-v1-core
    metadata:
    spec: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#podspec-v1-core
      activeDeadlineSeconds:
      affinity: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#affinity-v1-core
      automountServiceAccountToken:
      containers: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#container-v1-core
      dnsConfig: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#poddnsconfig-v1-core
      dnsPolicy:
      enableServiceLinks:
      hostAliases: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#hostalias-v1-core
      hostIPC:
      hostNetwork:
      hostPID:
      hostname:
      imagePullSecrets: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#localobjectreference-v1-core
      initContainers: #
      nodeName:
      nodeSelector:
      overhead:
      preemptionPolicy:
      priority:
      priorityClassName:
      readinessGates: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#podreadinessgate-v1-core
      restartPolicy:
      runtimeClassName:
      schedulerName:
      securityContext: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#podsecuritycontext-v1-core
      serviceAccount:
      serviceAccountName:
      shareProcessNamespace:
      subdomain:
      terminationGracePeriodSeconds:
      tolerations: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#toleration-v1-core
      volumes: # https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#volume-v1-core
```

上記、すごい量だが、結局意識するのは以下だけ。

```
apiVersion: app/v1
kind: Deployment
metadata:
  annotations:
    key: value
  labels:
    key: value
  name:
  namespace:
spec:
```

# 参考

- [Kubernetes Tips: kubectl でマニフェストの雛形を作る](https://qiita.com/tkusumi/items/4f63cea4c4d910b368c4)
- [Kubernetes流YAML職人になるために](https://qiita.com/bgpat/items/173f33ae2a9e21b24487)