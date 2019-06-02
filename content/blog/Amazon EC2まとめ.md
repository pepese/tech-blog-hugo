---
title: "Amazon EC2まとめ"
date: 2019-06-02T08:50:23+09:00
slug: "aws-ec2-basics"
tags:
- aws
- ec2
draft: true
archives:
- 2019/06
---

[Amazon EC2](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/concepts.html)についてまとめる。

<!--more-->

## インスタンスタイプ

EC2 **インスタンスタイプ** のネーミングポリシーは以下の通り。

```
# フォーマット
[インスタンスファミリー][インスタンス世代][(追加機能)].[インスタンスサイズ]
# 例
c5d.xlarge
```

各桁の意味は以下の通り。

- インスタンスファミリー（ 1 桁目）
- インスタンス世代（ 2 桁目）
- 追加機能（ 3 桁目）
- インスタンスサイズ（ドット以降）

[インスタンスタイプ一覧](https://aws.amazon.com/jp/ec2/instance-types/)

## EC2 のストレージ

- インスタンスストア
    - ホストコンピュータに内臓されたディスク
    - EC2 インスタンスと分割不可能
    - stop/terminate するとクリアされる
    - 追加費用無し
- Amazon Elastic Block Store (EBS)
    - ネットワークで接続
    - EC2 インスタンスとは独立して管理
    - stop/terminate してもクリアされない
    - Volume ごとに性能・容量を設定可能
    - 別途費用発生
    - Snapshot を取得して S3 に保存可能

### EBS 最適化オプション

通常のネットワークとは別に EBS 専用帯域を確保するオプション。

- 起動時に有効/無効を洗濯可能
- 帯域はインスタンスサイズによって異なる
- インスタンスタイプによっていはデフォルトで有効

[参考](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/EBSOptimized.html)

## Key Pair

[Key Pair](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-key-pairs.html) は、 EC2 インスタンス上の OS に対する安全な認証を提供する。  
Key Pair を作成すると秘密鍵をダウンロードでき、 EC2 起動時に公開鍵が EC2 に自動的に付与される。  
EC2 ログイン時に Key Pair （秘密鍵と公開鍵のペア）が照合され、ログイン可能に。

- 認証鍵は、ユーザ名・パスワードの認証よりも安全な認証方式
- AWS では公開鍵のみ保持し、起動時に公開鍵を EC2 にコピーする
- 秘密鍵は、ユーザにて適切に管理・保管する必要がある

## IP の種類

- Privete IP
    - 必ず割り当てられる IP アドレス
    - EC2 作成時に設定可能
    - stop/start しても変化なく固定の IP
- Public IP
    - ランダムに割り当てられる Public IP
    - stop/start すると別の IP が割り当てられる
    - 付与しないこともできる
- Elastic IP (EIP)
    - 別インスタンスへ再マップも可能な固定の Public IP
    - stop/start しても変化なく固定の IP
    - 利用していなくても課金される

[参考](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/using-instance-addressing.html)

## ENI

ENI ( Elastic Network Interfaces ) は、 VPC 上で実現する仮想ネットワークインターフェース。  
以下を ENI に紐づけて管理する。

- Private IP
- Elatic IP
- MAC アドレス
- セキュリティグループ

インスタンス障害時に NIC 付け替えなどの運用が可能に。  
また、インスタンスによって割り当て可能な数が異なることに注意。  
[参考](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/using-eni.html)

## 拡張ネットワーキング

EC2 インスタンスの持つ通信性能を最大化する機能。  
以下の 2 タイプある。

- Intel 82599VF (ixgbevf)
- Elastic Network Adapter (ENA)

[参考](https://aws.amazon.com/jp/premiumsupport/knowledge-center/enable-configure-enhanced-networking/)

## ネットワーク帯域

EC2 間のネットワーク帯域には以下の制限がある。

- シングルフロー通信
    - 最大 5 Gbps または 10 Gbps （同一 Cluster Placement Group の場合）
- マルチフロー通信
    - 各インスタンスタイプが持つ通信帯域の最大

EC2 間で 5 Gbps 以降の帯域幅を実現するには、通信の多重化とマルチコアへの分散を意識する必要がる。（ RPS/RFS の設定など）

## AMI の分類

- アーキテクチャ
    - x86
    - Arm
- ビット数
    - 32bit
    - 64bit
- 仮想化方式
    - 準仮想化（ Paravirtual : PV ）※古いので利用は推奨しない
    - 完全仮想化（ Hardware-assisted VM : HVM ）
- ブートストレージ
    - EBS Backed
    - Instance Store-Backed (S3 Backed)

推奨は、 64bit HVM EBS-Backed 。

[参考](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/AMIs.html)