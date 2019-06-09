---
title: "Helm入門"
date: 2019-05-28T08:02:13+09:00
slog: k8s-helm-basics
tags:
- kubernetes
- helm
draft: true
archives:
- 2019/05
---

今更ながら [Helm](https://github.com/helm/helm) に入門する。（ [公式ドキュメント](https://helm.sh/docs/) ）

- 環境構築
- 使い方

<!--more-->

# 環境構築

コマンドのインストールとコンテキストの確認。

``` bash
$ brew install kubernetes-helm
$ which helm
/usr/local/bin/helm
$ kubectl config current-context
docker-for-desktop
```

**Tiller** を kubernetes クラスタへインストールする。（ TLS authentication が必要な場合は [ここ](https://helm.sh/docs/using_helm/#using-ssl-between-helm-and-tiller) を参照）

``` bash
$ helm init --history-max 200
Creating /Users/xxxx/.helm
Creating /Users/xxxx/.helm/repository
Creating /Users/xxxx/.helm/repository/cache
Creating /Users/xxxx/.helm/repository/local
Creating /Users/xxxx/.helm/plugins
Creating /Users/xxxx/.helm/starters
Creating /Users/xxxx/.helm/cache/archive
Creating /Users/xxxx/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /Users/xxxx/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
```

上記で `$HELM_HOME` 環境変数（初期値は `$HOME/.helm` ）以下に所定のディレクトリとファイル群が生成されると同時に、 Tiller に関するリソースがkubernetesクラスタのkube-system名前空間に作成される。  
Tiller をアンインストールするには `$ helm reset` 。  

# 使い方

|コマンド|概要|
|:---|:---|
| `helm update repo` |リポジトリ情報の更新|
| `helm search` | Chartの検索|
| `helm inspect` | Chartの詳細情報の表示|
| `helm install` | Chartのインストール（デプロイ）|
| `helm list` |クラスタ上のChartインスタンス(Release)の一覧を表示|
| `helm upgrade` | Chartの更新のデプロイ|
| `helm delete` | ReleaseとAPIオブジェクトの削除|
| `helm template` | Chartから作成されるマニフェストファイルの出力|

## 構築・確認・削除

**Chart** （ Helm のパッケージ）をインストールするには `$ helm install` を利用する。

``` bash
$ helm repo update
$ helm install stable/mysql # これだけでクラスタに MySQL が構築される、恐ろしい、、、
$ helm list
$ helm delete wintering-rodent
```

- `$ helm install` でクラスタ上にデプロイされた構成を **リリース** と呼ぶ
    - このリリースには必ず名前がつけられ `--name` オプションで指定する（ [stable/mysql](https://github.com/helm/charts/tree/master/stable/mysql) ）
    ```
    $ helm install --name my-release stable/mysql
    ```
    - オプション指定が無い場合はランダムで命名（上記でいう `wintering-rodent` ）される
    - 同じチャートのより新しいリビジョンを使って内容を書き換えることが更新
    - `$ helm install` コマンドは、 `$ helm upgrade` コマンドの `--install` オプションで代替できるので、実際には `$ helm install` コマンドを使わず、 `$ helm upgrade` コマンドだけで統一可能
    - 事前にチェックするために `--dry-run` オプションを指定して初回は実行することをお勧め
- `$ helm rollback` コマンドによって、指定したリビジョンの内容に復帰させるということが可能
- `$ helm delete` コマンドでリリースを削除
    - `--purge` オプションをつけない限り、「DELETED」の状態としてそのリリース定義自体は維持される
    - この振る舞いによって、 `$ helm rollback` で「リリース削除の取り消し」も可能
- リリース後は `$ helm list` コマンドで一覧取得
    - 「DELETED」な状態のリリースを表示するには、 `--all` オプション

## Chart の探し方

``` bash
$ helm search
NAME                                 	CHART VERSION	APP VERSION                 	DESCRIPTION
stable/acs-engine-autoscaler         	2.2.2        	2.1.1                       	DEPRECATED Scales worker nodes within agent pools
stable/aerospike                     	0.2.6        	v4.5.0.5                    	A Helm chart for Aerospike in Kubernetes
...
$ helm search mysql
NAME                            	CHART VERSION	APP VERSION	DESCRIPTION
stable/mysql                    	1.1.1        	5.7.14     	Fast, reliable, scalable, and easy to use open-source rel...
...
stable/mariadb                  	6.1.0        	10.3.15    	Fast, reliable, scalable, and easy to use open-source rel...
```

## Chart のカスタマイズ

`$ helm inspect values <Chart>` コマンドで Chart のパラメータを確認できる。

``` bash
$ helm inspect values stable/mysql
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"

busybox:
  image: "busybox"
  tag: "1.29.3"

testFramework:
  image: "dduportal/bats"
  tag: "0.4.0"

## Specify password for root user
##
## Default: random 10 character string
# mysqlRootPassword: testing

## Create a database user
##
# mysqlUser:
## Default: random 10 character string
# mysqlPassword:
...
```

上記を基に `.yaml` ファイルを作成して以下のように構築する。

``` bash
$ helm install --name my_release -f values.yaml stable/mysql
```

なお、インストール対象の Chart には、アーカイブ・ディレクトリ・URL も指定できる。

## 独自の Chart を作成する

|コマンド|概要|
|:---|:---|
| `helm create` |新しいChartの作成|
| `helm template` |テンプレートのレンダリング|
| `helm upgrade –install` |作ったChartのデプロイ|

[詳細](https://helm.sh/docs/chart_template_guide/)

``` bash
$ helm create sample-chart
Creating sample-chart
$ ls -la
total 0
drwxr-xr-x   5 xxxx  xxxx   160  5 21 19:10 .
drwxr-xr-x+ 32 xxxx  xxxx  1024  5 21 18:37 ..
drwxr-xr-x   7 xxxx  xxxx   224  5 21 19:10 sample-chart
$ tree sample-chart/
sample-chart/
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
$ helm lint sample-chart    # リンター
$ helm package sample-chart # パッケージング
```

各ディレクトリ・ファイルは以下の役割。

-  `Chart.yaml` ：Chartの名前やバージョンなどの情報が記述される。
- `values.yaml` ：Chartのパラメータのデフォルト値が記述されまる。
- `charts/` ：このChartが依存している別のChartを保存する。
- `templates/`
    -  `NOTES.txt` ：Chartのインストール後に表示するメッセージ。
    - `_helpers.tpl` ：テンプレート全般で再利用するための部分テンプレートを書く。
    - 残りの*.yamlがマニフェストのテンプレート。

上記は、 Deployment と Service を一つずつ作成して、 Ingress 経由で外部に公開するというk8sの基本構成で準備されている。必須じゃないので消していい。  
なお、テンプレートはほぼ go の `text/template` で記載する。  
やり方の超概要は以下。

1. 自分で作成したマニフェストをテンプレート化して `templates/` 配下に置く。
2. 環境差分毎に `values.yaml` （ `values.dev.yaml` とか `values.prd.yaml` とか ）を作る
3. 適用する（ `$ helm install -f values.yaml sample-chart` ）

### テンプレートの書き方

テンプレートでは `{{}}` で囲った部分が他の値で書き換えられるのだが、以下のようになる。

- `.Values`
    - `values.yaml` ファイルの値を参照する
    - テンプレートファイル内から `.Values.aaa.bbb` のように、".Values"を接頭辞として、そのYAMLのルート構造以下にアクセスすることで値を参照する
- `template`
    - 定義済みの部分テンプレートを呼び出すAction
    - `templates/_helpers.tpl` で `define` により定義した値を埋め込む
- `.Release`
- `include`
- `.Chart`
- `toYaml`


# プラグイン・ツール

## helm-diff

```
$ helm plugin install https://github.com/databus23/helm-diff --version master
```

## [Helmfile](https://github.com/roboll/helmfile)

``` bash
$ brew install helmfile
```

`helmfile.yaml` にデプロイしたい Chart を書いて `helmfile sync` を実行するだけでインストールやアップグレードをよしなにやってくれる。

```
releases:
  - name: logstash
    namespace: kube-system
    chart: ./charts/logstash
    values:
      - ./values/production/logstash.yaml

  - name: fluentd-kinesis
    namespace: fluentd-kinesis
    chart: ./charts/fluentd-kinesis
    values:
      - ./values/production/fluentd-kinesis.yaml

  - name: backend-app
    namespace: backend-app
    chart: ./charts/backend-app
    values:
      - ./values/production/backend-app.yaml
```

差分チェックしてから適用。

``` bash
$ helmfile -f helmfile.yaml diff  # 差分チェック
$ helmfile -f helmfile.yaml apply # 適用
```

# 参考

- https://chopschips.net/blog/2018/05/30/helm-with-rails/