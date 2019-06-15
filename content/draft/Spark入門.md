---
title: Spark入門
tags:
- Spark
- Hadoop
id: spark-basics
draft: true
---

Spark さわってみた。

# Hadoop と Spark

<img src="../../images/hadoop-spark.png"  alt="Hadoop と Spark">

Cluster Manager のソフトウェアとしては以下も選択可能。

- Apache Mesos
    - 一般的な Cluster Manager で、 Hadoop MapReduce の実行も可能
    - クラスタ・マネージャ：分散システム（としてのソフトウェア）を実行するための管理システムであり、対障害性・スケーラブル
    - 分散システム・カーネル：データセンタのための小さな（マイクロ）カーネルかつメタ（汎用）スケジューラ
- Kubernetes
    - `apache-spark-on-k8s`　で Spark の Cluster Manager として利用可能
    - まだ experimental

# 構成

<img src="https://spark.apache.org/docs/latest/img/cluster-overview.png"  alt="Spark の構成">

`start-master.sh`　で起動するか `start-slave.sh` で起動するかによって Master ( Cluster Manager ) か Slave ( Worker Node ) かが決まる。  
Spark Shell から Master ( Cluster Manager ) 経由でインタラクティブに Spark　にアクセス可能（ Scala ）。  
アプリケーションは Master ( Cluster Manager ) にデプロイする。

# Spark のパッケージ

<img src="../../images/spark-packages.png"  alt="Hadoop と Spark">

Data Source は Hadoop HDFS 以外のものにも対応している。

# インストール

```sh
$ brew install apache-spark
```

# 起動

- [Spark Standalone Mode](https://spark.apache.org/docs/latest/spark-standalone.html)

# 参考

[Apache Spark で分散処理入門](https://qiita.com/Hiroki11x/items/4f5129094da4c91955bc)
