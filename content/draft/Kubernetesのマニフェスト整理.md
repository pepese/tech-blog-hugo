---
title: Kubernetesのマニフェスト整理
tags:
- kubernetes
id: k8s-manifest-basics
draft: true
---

kubernetes では `kubectl` に様々コマンドがあるが、基本的には `kubectl create/delete/apply` コマンドと **マニフェスト** とよぶ YAML/JSON ファイルを使用することで、リソースの作成/削除/更新できる。マニフェストのリファレンスは [ここ](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/) 。

- マニフェストの雛形
- マニフェストの理解

# マニフェストの雛形

- https://qiita.com/tkusumi/items/4f63cea4c4d910b368c4
- https://qiita.com/bgpat/items/173f33ae2a9e21b24487

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