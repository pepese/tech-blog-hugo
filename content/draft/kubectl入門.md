---
title: kubectl入門
tags:
- kubernetes
- k8s
- kubectl
id: kubectl-basics
draft: true
---

- k8s のローカル実行環境
- kubectl コマンド
- 一通り

# k8s のローカル実行環境

k8s のローカル実行環境には以下のようなものがある。

- Minikube
- Docker Desktop ( Docker for Mac )

ここでは後者について記載する。

## Docker Desktop ( Docker for Mac )

**Docker for Mac** をインストールしている場合は、dockerアプリからpreferencesを開き、kubernetesを有効にすると利用できるようになる。  
`Kubernetes` と `Swarm` があるが、ここでは前者を選択する。

```
$ which kubectl
/usr/local/bin/kubectl

$ kubectl cluster-info
Kubernetes master is running at https://localhost:6443
KubeDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get node
NAME                 STATUS    ROLES     AGE       VERSION
docker-for-desktop   Ready     master    3m        v1.10.3
```

kubernetes master がローカルマシン（ docker-for-desktop ）であることがわかる。  
デフォルトで起動している Pod を確認してみる。

```
$ kubectl get po --all-namespaces
NAMESPACE     NAME                                         READY     STATUS    RESTARTS   AGE
docker        compose-7447646cf5-txjf9                     1/1       Running   0          27m
docker        compose-api-6fbc44c575-8txhh                 1/1       Running   0          27m
kube-system   etcd-docker-for-desktop                      1/1       Running   0          27m
kube-system   kube-apiserver-docker-for-desktop            1/1       Running   0          27m
kube-system   kube-controller-manager-docker-for-desktop   1/1       Running   0          27m
kube-system   kube-dns-86f4d74b45-95ftq                    3/3       Running   0          27m
kube-system   kube-proxy-6b26j                             1/1       Running   0          27m
kube-system   kube-scheduler-docker-for-desktop            1/1       Running   0          26m
```

クラスタを起動した時点で `kubectl` コマンドに対してコンテキストが設定されており、実行したコマンドが docker-for-desktop の方を向いている。  
`kubectl` コマンドのリファレンスは[ここ](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) 。

# kubectl コマンド

```
$ kubectl [command] [TYPE] [NAME] [flags]
```
- command
    - コマンドの種類
    - https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
- TYPE
    - リソースタイプ
    - https://kubernetes.io/docs/reference/kubectl/overview/#resource-types
- NAME
    - リソースの名前
- flags
    - オプション

## コマンド一覧

https://kubernetes.io/docs/reference/kubectl/overview/#operations

一応一覧するが、ボールドのものだけ見ればいい。

- annotate
    - `kubectl annotate (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]`
    - リソースのアノテーションを追加・更新する
- api-versions
    - `kubectl api-versions [flags]`
    - API の一覧とそのバージョンを表示する
- **apply**
    - `kubectl apply -f FILENAME [flags]`
    - マニフェストファイルもしくは標準入力からリソースの設定を行う
- attach
    - `kubectl attach POD -c CONTAINER [-i] [-t] [flags]`
    - Attach to a running container either to view the output stream or interact with the container (stdin).
- autoscale
    - `kubectl autoscale (-f FILENAME \| TYPE NAME \| TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU] [flags]`
    - Pod 数を自動調整する autoscaler を作成する
- cluster-info
    - `kubectl cluster-info [flags]`
    - k8s クラスタの Master やその他サービスのエンドポイントを表示する
- **config**
    - `kubectl config SUBCOMMAND [flags]`
    - **kubeconfig** ファイルを変更する
- create
    - `kubectl create -f FILENAME [flags]`
    - ファイルもしくは標準入力からリソースを作成する
    - apply を利用するため create は利用しない
- **delete**
    - `kubectl delete (-f FILENAME \| TYPE [NAME \| /NAME \| -l label \| --all]) [flags]`
    - マニフェストファイル名、リソース名、ラベル、セレクタなどを指定してリソースを削除する
- **describe**
    - `kubectl describe (-f FILENAME \| TYPE [NAME_PREFIX \| /NAME \| -l label]) [flags]`
    - k8s の各種リソース情報の詳細を表示する
- edit
    - `kubectl edit (-f FILENAME \| TYPE NAME \| TYPE/NAME) [flags]`
    - Edit and update the definition of one or more resources on the server by using the default editor.
- **exec**
    - `kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [-- COMMAND [args...]]`
    - Pod 内のコンテナ上でコマンドを実行する
- explain
    - `kubectl explain [--include-extended-apis=true] [--recursive=false] [flags]`
    - リソースの説明ドキュメントを表示する（pods, nodes, services, etc.
- expose
    - `kubectl expose (-f FILENAME \| TYPE NAME \| TYPE/NAME) [--port=port] [--protocol=TCP\|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type] [flags]`
    - k8s Service を新しく作成して公開する（すでに存在するリソースに対しても可能）
- **get**
    - `kubectl get (-f FILENAME \| TYPE [NAME \| /NAME \| -l label]) [--watch] [--sort-by=FIELD] [[-o \| --output]=OUTPUT_FORMAT] [flags]`
    - k8s の各種リソース情報のを表示する
- label
    - `kubectl label (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]`
    - リソースにラベルを追加・更新する
- **logs**
    - `kubectl logs POD [-c CONTAINER] [--follow] [flags]`
    - Pod 内のコンテナのログを取得する
- patch
    - `kubectl patch (-f FILENAME \| TYPE NAME \| TYPE/NAME) --patch PATCH [flags]`
    - Update one or more fields of a resource by using the strategic merge patch process.
- port-forward
    - `kubectl port-forward POD [LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N] [flags]`
    - Forward one or more local ports to a pod.
- proxy
    - `kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix] [--api-prefix=prefix] [flags]`
    - Run a proxy to the Kubernetes API server.
- replace
    - `kubectl replace -f FILENAME`
    - Replace a resource from a file or stdin.
- rolling-update
    - `kubectl rolling-update OLD_CONTROLLER_NAME ([NEW_CONTROLLER_NAME] --image=NEW_CONTAINER_IMAGE \| -f NEW_CONTROLLER_SPEC) [flags]`
    - Perform a rolling update by gradually replacing the specified replication controller and its pods.
- run
    - `kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [flags]`
    - Cluster 上でコンテナイメージを起動する（ Pod も作成される）
- scale
    - `kubectl scale (-f FILENAME \| TYPE NAME \| TYPE/NAME) --replicas=COUNT [--resource-version=version] [--current-replicas=count] [flags]`
    - Update the size of the specified replication controller.
- stop
    - Deprecated: delete を使う
- top
    - `kubectl top [node/pod] [options]`
    - ノード、 Pod のリソール使用率を確認できる
- version
    - `kubectl version [--client] [flags]`
    - クライアントとサーバで起動している k8s のバージョンを表示する

参考：https://qiita.com/hana_shin/items/ef1a20239001ac83a78d

# 一通り

Nginx のコンテナを実行する。

```
$ kubectl run nginx --image=nginx:1.11.3

$ kubectl get pods
NAME                     READY     STATUS              RESTARTS   AGE
nginx-74b65c798c-9zj44   0/1       ContainerCreating   0          14s

$ kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/nginx-74b65c798c-9zj44   1/1       Running   0          3s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10h

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1         1         1            1           3s

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx-74b65c798c   1         1         1         3s
```

Pod 、 deployment 、 Replica Set が作成された。  
上記で Pod が `ContainerCreating` になっているのがわかる。（ `Running` になるまで待つ。）  
この状態では、 **Worker Node 上でコンテナが動いているだけ** なので、 `expose` コマンドで Service を作成して **公開** する。

```
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10h

$ kubectl expose deployment nginx --port 80 --type LoadBalancer
service "nginx" exposed

$ kubectl get services
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP        10h
nginx        LoadBalancer   10.111.65.40   localhost     80:30562/TCP   5s

$ kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/nginx-74b65c798c-9zj44   1/1       Running   0          1m

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        10h
service/nginx        LoadBalancer   10.98.192.178   localhost     80:30033/TCP   4s

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1         1         1            1           1m

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx-74b65c798c   1         1         1         1m
```

nginx という Service が作成されて、 80 番ポートで公開されていることがわかる。  
試しにアクセスしてみる。

```
$ curl localhost
〜省略〜
<title>Welcome to nginx!</title>
〜省略〜
```

終わったので `kubectl delete` コマンドで停止する。

```
$ kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           21m

$ kubectl delete service nginx
service "nginx" deleted

$ kubectl delete deployment nginx
deployment.extensions "nginx" deleted

$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10h
```

## `kubectl expose` について

`kubectl expose` では Workloads リソースをコントロールできる。  
`--type` では Service の TYPE を選択できる。

- ClusterIP (default)
    - Kubernetes Cluster 内に閉じる Internal IP を公開する
- NodePort
    - 指定したPort に対して Node の NAT を構成する
    - Kubernetes Cluster 外部に公開され、 `<NodeIP>:<NodePort>` でアクセスできる
    - ClusterIP のスーパーセット
- LoadBalancer
    - Kubernetes Cluster 外部に公開されるロードバランサ
    - 外部 IP アドレスが作成され、アクセス可能となる
    - NodePort のスーパーセット
- ExternalName
    - 任意の名前と設定できる
    - kube-dns の CNAME レコードとなる






# 以下、未整理

### クライアント、サーバのバージョンを確認

```sh
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.3", GitCommit:"2bba0127d85d5a46ab4b778548be28623b32d0b0", GitTreeState:"clean", BuildDate:"2018-05-21T09:17:39Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.3", GitCommit:"2bba0127d85d5a46ab4b778548be28623b32d0b0", GitTreeState:"clean", BuildDate:"2018-05-21T09:05:37Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

### 設定を確認

コンテキストの確認。

```
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
*         docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
```

コンテキストを切り替える。

```
$ kubectl config use-context docker-for-desktop
```

### Node の一覧を取得

```sh
$ kubectl get nodes
NAME                 STATUS    ROLES     AGE       VERSION
docker-for-desktop   Ready     master    57m       v1.10.3
```

### コンテナをデプロイ

サンプルのコンテナをデプロイして、コンテナ、 Pod の一覧を取得する。

```sh
$ kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080
deployment "kubernetes-bootcamp" created
$
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            0           58s
$
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-fwqf6   1/1       Running   0          10m
```

デフォルトでは、 Kubernetes Cluster 内でのみ通信可能で、外部から Master の apiserver へのアクセスができない。  
そのため、 `kubectl proxy` コマンドで外部から apiserver へアクセス可能なプロキシを作成する。

```sh
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

`ctrl + c` で終了する。  
終了しない状態で、別ターミナルで `curl` などを実行してアクセスを確認する。  
以下で apisever のバージョンを確認する。

```sh
$ curl http://127.0.0.1:8001/version
{
  "major": "1",
  "minor": "8",
  "gitVersion": "v1.8.0",
  "gitCommit": "0b9efaeb34a2fc51ff8e4d34ad9bc6375459c4a4",
  "gitTreeState": "clean",
  "buildDate": "2017-11-29T22:43:34Z",
  "goVersion": "go1.9.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

また、 apiserver に対して Pod 名を指定することによって、 Pod を経由してデプロイしたアプリケーションにアクセスできる。  
Pod 名は `kubectl get pods` で取得できる。

```sh
$ curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/kubernetes-bootcamp-6db74b9f76-fwqf6/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-6db74b9f76-fwqf6 | v=1
```

Pod 内のコンテナ上でコマンドを実行してみる。

```sh
$ kubectl exec kubernetes-bootcamp-6db74b9f76-5c4p4 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-6db74b9f76-5c4p4
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root
```

### コンテナへログイン

bash セッションを作成してコンテナへアクセスする。  
`exit` でぬける。

```sh
$ kubectl exec -ti kubernetes-bootcamp-6db74b9f76-5c4p4 bash
root@kubernetes-bootcamp-6db74b9f76-5c4p4:/#
root@kubernetes-bootcamp-6db74b9f76-5c4p4:/# curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-6db74b9f76-5c4p4 | v=1
root@kubernetes-bootcamp-6db74b9f76-5c4p4:/# exit
```

### Service

Pod は IP を持つが、その IP は Kubernetes Cluster の外部へ公開されない。  
Service は Pod へのトラフィックを中継する。  
Service は ServiceSpec という YAML もしくは JSON ファイルで設定する。  
Service には以下の種類（ **type** という）がある。

- ClusterIP (default)
    - Kubernetes Cluster 内に閉じる Internal IP を公開する
- NodePort
    - 指定したPort に対して Node の NAT を構成する
    - Kubernetes Cluster 外部に公開され、 `<NodeIP>:<NodePort>` でアクセスできる
    - ClusterIP のスーパーセット
- LoadBalancer
    - Kubernetes Cluster 外部に公開されるロードバランサ
    - 外部 IP アドレスが作成され、アクセス可能となる
    - NodePort のスーパーセット
- ExternalName
    - 任意の名前と設定できる
    - kube-dns の CNAME レコードとなる

あるService へ所属する Pod はラベル（ **LabelSelector** ）によって決定される。

```sh
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   43m
```

上記の kubernetes は master から node に通信する用の Service 。  
新しく Service を作成してみる。

```sh
$ kubectl expose deployment kubernetes-bootcamp --type="NodePort" --port 8080
service "kubernetes-bootcamp" exposed
$
$ kubectl get services
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP          46m
kubernetes-bootcamp   NodePort    10.102.125.240   <none>        8080:31130/TCP   10m
$
$ kubectl describe services kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.102.125.240
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31130/TCP
Endpoints:                172.17.0.4:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$
$ kubectl get services kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.102.125.240   <none>        8080:31130/TCP   13m
```

`-l` オプションを使用して、ラベルを指定してリソースを取得可能。

```sh
$ kubectl get pods -l run=kubernetes-bootcamp
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          54m
$ kubectl get services -l run=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.102.125.240   <none>        8080:31130/TCP   20m
```

ラベルは以下のように付与する。

```sh
$ kubectl label pod kubernetes-bootcamp-6db74b9f76-5c4p4 app=v1
pod "kubernetes-bootcamp-6db74b9f76-5c4p4" labeled
$
$ kubectl describe pods kubernetes-bootcamp-6db74b9f76-5c4p4
Name:           kubernetes-bootcamp-6db74b9f76-5c4p4
Namespace:      default
Node:           minikube/192.168.99.100
Start Time:     Sun, 03 Dec 2017 09:10:57 +0900
Labels:         app=v1
                pod-template-hash=2863065932
                run=kubernetes-bootcamp
# (省略)
$
$ kubectl get pods -l app=v1
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          57m
```

削除は以下。

```sh
$ kubectl delete service -l run=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1h
```

### スケーリング

スケーリングの設定を行うことで Pod がスケールする。

```sh
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           1h
```

- DESIRED：設定されたレプリカ数
- CURRENT：起動中のレプリカ数
- UP-TO-DATE： DESIRED の状態になったレプリカ数
- AVAILABLE：ユーザからアクセス可能なレプリカ数

Pod を 4 つまでスケールさせてみる。

```sh
$ kubectl scale deployments/kubernetes-bootcamp --replicas=4
deployment "kubernetes-bootcamp" scaled
$
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           1h
$
$ kubectl get pods -o wide
NAME                                   READY     STATUS    RESTARTS   AGE       IP           NODE
kubernetes-bootcamp-6db74b9f76-2ppvx   1/1       Running   0          10m       172.17.0.6   minikube
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          1h        172.17.0.4   minikube
kubernetes-bootcamp-6db74b9f76-ct7lp   1/1       Running   0          10m       172.17.0.5   minikube
kubernetes-bootcamp-6db74b9f76-vkhbl   1/1       Running   0          10m       172.17.0.7   minikube
```

### コンテナをローリングアップデート

```sh
$ kubectl describe pods
```

現状は `v1` の Pod が 4 つあるのがわかる。  
`kubectl set image` コマンドでコンテナイメージを更新する。

```sh
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-2ppvx   1/1       Running   0          23m
kubernetes-bootcamp-6db74b9f76-5c4p4   1/1       Running   0          1h
kubernetes-bootcamp-6db74b9f76-ct7lp   1/1       Running   0          23m
kubernetes-bootcamp-6db74b9f76-vkhbl   1/1       Running   0          23m
$
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
deployment "kubernetes-bootcamp" image updated
$
$ kubectl get pods
# (省略)
$ kubectl rollout status deployments/kubernetes-bootcamp
deployment "kubernetes-bootcamp" successfully rolled out
```

`kubectl get pods` コマンドでステータスが `Terminating` 、 `ContainerCreating` 、 `Running` となっていき、ローリングアップデートされていくのがわかる。  
`kubectl rollout status` でローリングアップデートが成功したか確認する。  
なお、なんらかが原因で失敗している場合、以下のコマンドで UNDO できる。

```sh
$ kubectl rollout undo deployments/kubernetes-bootcamp
```