---
title: "DyanamoDB入門"
date: 2017-08-29T07:58:05+09:00
slug: "dynamodb-basics"
tags:
- aws
- dynamodb
- aws cli
draft: false
archives:
- 2017/08
---

ここでは、 DynamoDB の概要と AWS CLI でのアクセスまでまとめる。

- DynamoDB の概要
- DynamoDB の API
- AWS CLI で DynamoDB へアクセス

<!--more-->

# DynamoDB の概要

DynamoDB は AWS が提供する Key-Value 型 NoSQL のマネージドサービス。  
スキーマレスであるため、テーブル作成時に Key 以外の設定は必要ない。  
以下の順で説明していく。

- Value
- Key
- テーブル
- パーティション
- セカンダリインデックス
- キャパシティユニット

## Value

DynamoDB の Value は **Item** （ **項目** ）と呼ばれ、 1 つ以上の **Attribute** （ **属性** ）を持つ。  
Item は RDB でいうレコードに該当する。  
1つの Attribute につき、以下を指定する。

- Attribute Name
    - 属性名
- Attribute Type
    - 属性の型、以下の型がある
        - **スカラー型**
            - **1つ** の値を持つことができ、 **数値** 、**文字列** 、 **バイナリ** 、 **ブール** 、 **null** を選択できる
        - **ドキュメント型**
            - **入れ子** の値を持つことができ、 **リスト** 、 **マップ** を選択できる
            - リスト要素に保存できるデータ型に制限はなく、全て同じ型である必要はない
            - マップは JSON オブジェクトと同様
        - **セット型**
            - **複数** の値を持つことができ、 **数値** 、**文字列** 、 **バイナリ** を選択できる
            - **セット内の全ての要素は同じ型** である必要がある
- Attribute Value
    - Attribute Type に沿った属性の値

DynamoDBの各種キーは、 Item の中の Attribute から選択することになる。

### ドキュメント型/リストの例

```
FavoriteThings: ["Cookies", "Coffee", 3.14159]
```

### ドキュメント型/アップの例

```
{
    Day: "Monday",
    UnreadEmails: 42,
    ItemsOnMyDesk: [
        "Coffee Cup",
        "Telephone",
        {
            Pens: { Quantity : 3},
            Pencils: { Quantity : 2},
            Erasers: { Quantity : 1}
        }
    ]
}
```

### セット型の例

```
["Black", "Green" ,"Red"]

[42.2, -19, 7.5, 3.14]

["U3Vubnk=", "UmFpbnk=", "U25vd3k="]
```

## Key

DynamoDB では Key を **プライマリーキー** と呼び、以下の2種類の構成から選択できる。

1. **パーティションキー** のみの単純キー
2. **パーティションキー** と **ソートキー** の複合キー

パーティションキー、ソートキー共に Value で紹介した **Attribute と同様** の定義となる。  
パーティションキーは **Hash Key** 、 ソートキーは **Range Key** とも呼ばれることもある。

## テーブル

テーブルは **テーブル名** と **プライマリーキー** （オプションで、ローカルセカンダリインデックス ）で定義することができる。

## パーティション

DynamoDB はテーブルが作成された際に自動でパーティションが作成され、また、自動で管理される。
1つのパーティションには、 **スループット** と **データ容量が** が決まっており、それぞれ上限を超えたら自動的に新規にパーティションが作成される。
パーティションは **パーティションキー** と **ソートキー** から決定される。

DynamoDB はパーティションキーをハッシュ関数への入力し、その出力値に応じてパーティションを特定する。 Item はソートキーの値でソートされ、データ量が 10GB を超える場合はソートキーによりパーティション分割される。

## セカンダリインデックス

テーブルで 1 つ以上の[セカンダリインデックス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/SecondaryIndexes.html)（単に、インデックスともいう）を作成できる。  
セカンダリインデックスは必須ではないが、利用することにより query/scan の速度を向上することができる。

DynamoDB のインデックスには以下の 2 つある。

- [グローバルセカンダリインデックス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/GSI.html) （ **Global Secondary Index** / **GSI** ）
    - テーブルと異なるパーティションキー（必須）とソートキー（オプション）を持つインデックス
    - テーブル作成後に変更 **可能**
    - キャパシティーユニットの設定が **必要**
    - [ベストプラクティス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/GuidelinesForGSI.html)
- [ローカルセカンダリインデックス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/LSI.html) （ **Local Secondary Index** / **LSI** ）
    - テーブルと同じパーティションキー（必須）と、異なるソートキー（必須）を持つインデックス
    - テーブル作成後に変更 **不可能**
    - キャパシティーユニットは **テーブルのキャパシティーユニットと同じ**
    - [ベストプラクティス](http://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/GuidelinesForLSI.html)

テーブルあたり最大 5 個の GSI と 5 個の LSI を定義できる。  
全てのセカンダリインデックスは **完全に 1 つのテーブルに関連付け** られ、そこからデータを取得します。  
また、ソートキーの Attribute に条件を適用して **Item をフィルタリング** することができる。

## キャパシティユニット

DynamoDB では **キャパシティーユニット** という単位で、書き込みと読み込みのスループットキャパシティが決定される。  
DyanamoDB のテーブル作成時には **読み込みキャパシティーユニット** と **書き込みキャパシティーユニット** を指定する必要がある。

- **強力な整合性のある** 読み込みキャパシティーユニット
    - 『1 キャパシティーユニット』＝『「最大 **4** KB の Item 1 つ」を「 1 秒間」に「 1 回」の読み込む』
        - 4 KB より大きい項目を読み込む場合は、追加の読み込みキャパシティーユニットを消費する
- **結果的に整合性のある** 読み込みキャパシティーユニット
    - 『1 キャパシティーユニット』＝『「最大 **4** KB の Item 1 つ」を「 1 秒間」に「 **2** 回」の読み込む』
        - 4 KB より大きい項目を読み込む場合は、追加の読み込みキャパシティーユニットを消費する
- 書き込みキャパシティーユニット
    - 『1 キャパシティーユニット』＝『「最大 **1** KB の Item 1 つ」を「 1 秒間」に「 1 回」の書き込む』
        - 1 KB より大きい項目を書き込む場合は、追加の書き込みキャパシティーユニットを消費する

例えば、 **強力な整合性のある** 読み込みキャパシティーユニットであれば「最大 **4** KB の Item 1 つ」なので、「 3.5 KB の Item 1 つ を 1 秒に 1 回読み込み」でも「 1 キャパシティーユニット」になる。  
「 3.5 KB の Item 1 つ を 1 秒に 2 回読み込み」であれば「 2 キャパシティユニット」、「 0.5 KB の Item 1 つ を 1 秒に 10 回読み込み」であれば「 10 キャパシティユニット」、「 2.1 KB の Item 5 つ を 1 秒に 3 回読み込み」であれば「 15 キャパシティユニット」となる。  

**結果的に整合性のある** 読み込みキャパシティーユニットであれば、 **強力な整合性のある** 読み込みキャパシティーユニットの **2倍** のキャパシティー、つまりキャパシティユニットの数字は **半分** となるのみで考え方は同様。  
書き込みキャパシティーユニットであれば「最大 **1** KB の Item」部分異なるのみで考え方は同様。  

さらに例を挙げる。  
「 20 KB の Item 1 つを 1 秒間に強力な整合性のある読み込み」を行う場合は「 5 キャパシティーユニット」（ 20 KB = 4 KB × **5** なので）。  
「 20 KB の Item 1 つを 1 秒間に結果的に整合性のある読み込み」を行う場合は「 2.5 キャパシティーユニット」（ 20 KB = 4 KB × 5 の **2倍** のキャパシティなのでキャパシティーユニットの数字は **半分** ）。  
「 40 KB の Item 1 つを 1 秒間に結果的に整合性のある読み込み」を行う場合は「 5 キャパシティーユニット」。  
「 5 KB の Item 1 つを 1 秒間に書き込む」場合は「 5 キャパシティーユニット」。

# DynamoDB の API

DynamoDB の API は大きく以下がある。

- コントロールプレーン
    - DynamoDB テーブルを作成、管理
- データプレーン
    - テーブルデータのCRUD
- DynamoDB ストリーム
    - テーブルのストリームを有効または無効にし、ストリームに含まれるデータ変更レコードにアクセス

ここでは、コントロールプレーン、データプレーンについて触れる。

## コントロールプレーン

コントロールプレーンには以下の API がある。

- **CreateTable**
    - テーブルを作成する
    - オプションで、1 つ以上のセカンダリインデックスを作成し、テーブルに対して DynamoDB ストリーム を有効にできる
- **DescribeTable**
    - プライマリキーのスキーマ、スループット設定、インデックス情報など、テーブルに関する情報を取得する
- **ListTables**
    - すべてのテーブル名のリストを取得する
- **UpdateTable**
    - テーブルまたはそのインデックスの設定を変更、テーブルの新しいインデックスを作成または削除、またはテーブルの DynamoDB ストリーム 設定を変更する
- **DeleteTable**
    - テーブルを削除する

## データプレーン

データプレーンには以下の API がある。

- データの作成
    - **PutItem**
        - テーブルに Item を 1つ書き込む
    - **BatchWriteItem**
        - テーブルに Item を25個書き込みむ
        - PutItem を複数回呼び出すよりも効率的
        - 1つ以上のテーブルを跨った書き込みも可能
- データの読み取り
    - **GetItem**
        - テーブルから Item を 1つ取得する
    - **BatchGetItem**
        - テーブルから Item を 100 個取得する
        - GetItem を複数回呼び出すよりも効率的
        - 1つ以上のテーブルを跨った読み込みも可能
    - **Query**
        - 指定したパーティションキーを持つ全ての Item を取得する
            - パーティションキーは、「テーブル（＝ローカルセカンダリインデックス）のパーティションキー」もしくは「グローバルセカンダリインデックスのパーティションキー」
        - オプションで、ソートキーおよび比較演算子を指定して、検索結果を絞り込める
            - ソートキーは、パーティションキーに対応するソートキーから選択する
        - Item 内の属性全体、もしくは指定した属性のみを取得できる
    - **Scan**
        - 指定したテーブルもしくは（グローバル／ローカル）セカンダリインデックスの全ての Item を取得する
        - オプションで、パーティションキー・ソートキーおよび比較演算子を指定して、検索結果を絞り込める
        - Item 内の属性全体、もしくは指定した属性のみを取得できる
        - **フィルタ式があるかどうかにかかわらず、同じ量の読み込みキャパシティーを消費する**
- データの更新
    - **UpdateItem**
        - テーブルの Item を 1 つ以上更新する
        - 条件付き更新が可能
        - オプションで、アトミックカウンタ（他の書き込みリクエストを妨害することなく、数値属性をインクリメントまたはデクリメント）も可能
- データの削除
    - **DeleteItem**
        - テーブルから Item を 1 つ削除する
    - **BatchWriteItem**
        - テーブルから Item を最大 25 個削除する
        - DeleteItem を複数回呼び出すよりも効率的
        - 1つ以上のテーブルを跨った削除も可能

# AWS CLI で DynamoDB へアクセス

[公式ドキュメント](http://docs.aws.amazon.com/cli/latest/reference/dynamodb/index.html)

## 環境設定

**pip** が導入されている前提で書く。

```sh
$ pip install awscli
$ aws configure
# アクセスキー、シークレットキーなどを入力する
```

## テーブル定義

以降の AWS CLI によるオペレーションは以下のテーブル定義の前提とする。  
テーブル名は `test_table` 。

|属性名|属性型|プライマリーキー|
|:---|:---|:---|
|test_hash|スカラー型/文字列|HASH Key|
|test_range|スカラー型/文字列|RANGE Key|
|test_value|スカラー型/文字列||

## CreateTable

```sh
$ aws dynamodb create-table \
  --attribute-definitions '[{"AttributeName":"test_hash","AttributeType":"S"},{"AttributeName":"test_range","AttributeType":"S"}]' \
  --table-name 'test_table' \
  --key-schema '[{"AttributeName":"test_hash","KeyType":"HASH"},{"AttributeName":"test_range","KeyType":"RANGE"}]' \
  --provisioned-throughput '{"ReadCapacityUnits":5,"WriteCapacityUnits":5}'
```

## DeleteTable

```sh
$ aws dynamodb delete-table \
  --table-name 'test_table'
```

## ListTables

```sh
$ aws dynamodb list-tables
```

## PutItem

```sh
$ aws dynamodb put-item \
  --table-name 'test_table' \
  --item '{"test_hash":{"S":"xxxxx"},"test_range":{"S":"yyyyy"},"test_value":{"S":"zzzzz"}}'
```

## GetItem

```sh
$ aws dynamodb get-item \
  --table-name 'test_table' \
  --key '{"test_hash":{"S":"xxxxx"},"test_range":{"S":"yyyyy"}}'
```
