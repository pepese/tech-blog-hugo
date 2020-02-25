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

# 独自の Chart を作成する

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

- `Chart.yaml` ：Chartの名前やバージョンなどの情報が記述される。
- `values.yaml` ：Chartのパラメータのデフォルト値が記述されまる。
- `charts/` ：このChartが依存している別のChartを保存する。
- `templates/`
    -  `NOTES.txt` ：Chartのインストール後に表示するメッセージ。
    - `_helpers.tpl` ：テンプレート全般で再利用するための部分テンプレートを書く。
    - 残りの*.yamlがマニフェストのテンプレート。

上記は、 Deployment と Service を一つずつ作成して、 Ingress 経由で外部に公開するというk8sの基本構成で準備されている。必須じゃないので消していい。  
なお、テンプレートはほぼ go の `text/template` で記載する。  
やり方の超概要は以下。

1. `helm create`
2. `template/*` と values.yaml を削除する
3. 自分で作成したマニフェストをテンプレート化して `templates/` 配下に置く
4. 環境差分毎に `values.yaml` （ `values.dev.yaml` とか `values.prd.yaml` とか ）を作る
5. 適用する（ `$ helm install -f values.yaml sample-chart` ）

## テンプレートの書き方

テンプレートでは `{{}}` で囲った部分が他の値で書き換えられるのだが、以下のようになる。  
また、中括弧内側の前後にダッシュ `{{- -}}` をつけることができ、前に付けた場合はその部分より前の半角スペースを、後ろにつけた場合は改行コードを取り除くことができる。
これは値を展開しない構文に対して使われることがある。（要はその行自体消したいときとか）

### ビルドインオブジェクト（ [Built-in Objects](https://helm.sh/docs/chart_template_guide/builtin_objects/) ）

- `.Values`
  - `values.yaml` ファイルの値を参照する
  - テンプレートファイル内から `.Values.aaa.bbb` のように、".Values"を接頭辞として、そのYAMLのルート構造以下にアクセスすることで値を参照する
- `.Release`
  - リリースそのものを指し、さまざまな情報を持つ
  - Release.Name：リリース名
  - Release.Namespace：リリースされた Kubernetes クラスタのネームスペース
  - Release.IsUpgrade：現在のオペレーションが更新された場合 true となる
  - Release.IsInstall：現在のオペレーションがインストールされたら true となる
  - Release.Revision：リリースのリビジョン番号
- `.Chart`
  - `Chart.yaml` のコンテンツ
  - `{{ .Chart.Name }}-{{ .Chart.Version }}` のように使える
- `.Files`
- `.Capabilities`
- `.Template`

### 関数

Template構文内では事前定義された様々な関数を使うことができる。  
helm で利用できる関数がまとまっているページは見当たらないが、 Go のテンプレート関数は[ここ](http://masterminds.github.io/sprig/)。
関数はパイプ(|) を使うことでチェインすることができる。

```
{{ .Files.get "files/main.cf" | nindent 4}}
```

以下、代表的なもの。

- `nindent <numer> <string>`
  - 第2引数に渡した文字列の各行の先頭に第1引数の数値分だけ半角スペース(つまりインデント)を埋め込む
  - yaml等の文字列を展開する場合にこの関数でインデントを調整することができる
- `toYaml`
  - オブジェクトをyaml形式で展開する
  - values.yamlに記載されているyamlを直接マニフェストファイルに展開したい場合などにこの関数を使う
```
# values.yaml
test:
  hoge:
    fuga: piyo

# deployment.yaml
expand:
  {{- nindent 2 (toYaml .Values.test) }}

# ↓↓展開されると↓↓
expand:
  hoge:
    fuga: piyo
```
- `include <string> <object>`
  - _helpers.tpl に定義した名前付きtemplateを展開する
  - 第2引数のオブジェクトは名前付きtemplate側でのルートオブジェクトとして扱われ、基本的にはルートオブジェクトをそのまま渡すパターン `{{ inculude "fugafuga" . }}` が多い
```
# values.yaml
test:
  hoge:
    fuga: piyo

# _helpers.tpl
{{- define "fugafuga" -}}
- {{ toYaml . -}}
- {{ toYaml . -}}
{{- end }}

# deployment.yaml
expand:
  {{- include "fugafuga" .Values.test.hoge | nindent 2 }}

# ↓↓展開されると↓↓
expand:
  - fuga: piyo
  - fuga: piyo
```
- `tpl <string> <object>`
  - 第1引数の文字列をTemplateとして解釈し、第2引数をルートオブジェクトとして展開する
  - _helpers.tpl に定義しているものは include で展開すればTemplateとして解釈されるのでこちらは外部ファイルを取ってきたあとに適用する場合が多い
  - Template構文を展開するだけなのでyaml以外の文字列でも適用は可能
```
# values.yaml
nginxValues:
  nginxUser: "nginx"

# files/nginx.conf
user {{ .nginxUser }};
http{
    # いろいろ
}

# configmap.yaml
data:
  nginx.conf: |
{{ tpl (.Files.Get "files/nginx.conf") .Values.nginxValues | nindent 4 }}

# ↓↓展開されると↓↓
data:
  nginx.conf: |
    user nginx;
    http{
        # いろいろ
    }
```
- `.Files.Get <string>`
  - 外部からファイルを文字列として取り込む
  - 第1引数はファイルパスでChartのルートディレクトリ( Chart.yaml の置いてあるディレクトリ)からの相対パスを記載する

### アクション

関数のほか、Templateを制御するためいくつかのキーワードが用意されている。  
一般的なプログラミング言語でいう `if` や `for` などにあたり、これをHelmでは **アクション** と呼ぶ。

- if/else
- with
  - 引数に渡したオブジェクトをルートオブジェクトとしたスコープを展開
  - 引数に渡したオブジェクトが if/else アクションで紹介したfalseにあたる値の場合はwithブロック自体が展開されない、つまり簡易的な if としても使用できる
```
# values.yaml
configPaths:
  nginxConfPath: "files/nginx.conf"

# configmap.yaml
{{- $files = .Files -}}

data:
{{- with .Values.configPaths -}}
  nginx.conf: |
{{ $files.get .nginxConfPath | nindent 4 }}
{{- end -}}
```
- range
  - 引数に渡した配列でループを回す
  - ループ内のスコープでは配列の値自体がルートオブジェクトとなる
```
# values.yaml
configPaths:
  - key: nginx.conf
    path: "files/nginx.conf"
  - key: postfix.main.cf
    path: "files/main.cf"

# comfigmap.yaml
{{- $files = .Files -}}

data:
{{- range .Values.configPaths -}}
  {{ .key }}: |
{{ $files.get .path | nindent 4 }}
{{- end -}}

# ↓↓展開されると↓↓
data:
  nginx.conf: |
    # nginx.confの内容
  main.cf: |
    # main.cfの内容
```
- `template`
  - `_helpers.tpl` に定義してある名前付きtemplateを展開
  - `templates/_helpers.tpl` で `define` により定義した値を埋め込む
  - ただし、include関数と違いルートオブジェクトを渡すことができない

## `_helpers.tpl`

Chart内で共通で使うことのできるテンプレートスニペットを定義する。これは「名前付きテンプレート」と呼ばれる。  
values.yaml が差し替え可能な変数、ローカル変数が定義したTemplateファイル内でのみ使える変数、_helpers.tplはチャート内で自由に使える変数、という形に捉えることができる。  
`define` アクションを使うことで名前付きテンプレートを定義することができる。

```
{{- define "myapp.namespace" -}}
{{ .Chart.Name }}-space
{{- end }}
```

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