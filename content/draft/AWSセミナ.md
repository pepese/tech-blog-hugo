---
title: AWSセミナ
tags:
- AWS
id: aws-seminer
draft: true
---


AWS Black Belt Online Seminar / ソリューションアーキテクト 上原さん

# AWS Glue

- https://www.slideshare.net/AmazonWebServicesJapan/aws-black-belt-aws-glue

中身は **Spark** のマネージドサービス。ただし、加工変換部分の機能のみ。（ Spark Streaming ？）  
日本語ではノリという意味。  
「収集」「プリプロセス」の役割を担う。ポイントは以下。

- 分散処理
- サーバレス

大きく以下の機能がある。

- メタデータは **データカタログ** で管理
    - Hive のメタデータストア
    - メタデータ：デーブル、データ型、パーティションフォーマットなど
- データソースを **クロール** し、メタデータ取得
- メタデータを元に **ジョブ** を作成（ PySpark ）
    - ジョブは **サーバレス** な環境で実行される
    - コードは実装する必要がある（半自動生成はされる：雛形生成）
        - [サンプルコード](https://github.com/aws-samples/aws-glue-samples)

様々なデータソース（ JDBC で繋がる DB 、 S3 ）からデータを取得し、 **カラムナフォーマット** に変換して S3 へ保存する。  
以下で開発可能。

- Jupyter ： おすすめ
    - Sage Maker の notebook から繋ぐことも可能
- Zeppeline

# AWS Systems Manager

Hinemos 、 Zabbix と同レイヤ。