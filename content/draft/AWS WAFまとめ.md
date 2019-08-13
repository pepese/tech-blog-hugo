---
title: "AWS WAFまとめ"
date: 2019-08-13T18:59:21+09:00
slug: "aws-waf-basics"
tags:
- aws
- waf
draft: true
archives:
- 2019/08
---

[AWS WAF](https://docs.aws.amazon.com/ja_jp/waf/latest/developerguide/waf-chapter.html) についてまとめる。

<!--more-->

# 基本

AWS WAF は、 **Web ACLs** 、 **Rules** 、 **Conditions** の 3 つの要素からなる。

- **Condistions** / 条件
- **Rules** / ルール
- **Web ACLs** / Web アクセスコントロールリスト

複数の Condistion を Rules にまとめ、複数の Rules を Web ACLs に登録するという包含関係になっている。

<img src="https://www.shadan-kun.com/blog/wp2/wp-content/uploads/2017/11/AWS-WAF1.png" />


Web ACLs を CloudFront 、 API Gateway 、 ALB に適用して利用する。

# Conditions / 条件

以下の条件を作成可能。

- Cross-site scripting / クロスサイトスクリプティング
- Geo match / 地域制限
- IP addresses / IP アドレス一致
- Size constraints / サイズ制約
- SQL injection / SQL インジェクション
- String and regex matching / 文字列と正規表現の一致

## Cross-site scripting / クロスサイトスクリプティング

悪意のあるスクリプトが含まれていないか検査する箇所を定義する。  
以下の組み合わせで検査箇所を定義する。

- Part of the request to filter on / 検査する場所の定義
  - Header / HTTP ヘッダ
  - HTTP method
  - Query string / クエリ文字列 : URL の `?` の後に続く文字列全て
  - Single query parameter (value only) / 単一クエリパラメータ (値のみ)
  - All query parameters (values only) / すべてのクエリパラメータ (値のみ)
  - URI /
  - Boby / リクエストボディ
- Transformation / 検査の前に行う変換を定義
  - None / 変換なし
  - Convert to lowercase / 小文字へ変換
  - HTML decode / エスケープ（ `&quot;` や `&#xhhhh;` など）のデコード
  - Normalize whitespace / `\f` `\t` `\n` `\r` `\v` の空白文字変換
  - Simplify command line / OS コマンドの場合の整形
  - URL decode / URL デコード、パーセントデコード

ここでは検査箇所を定義するだけで、検査内容・アルゴリズムについては定義しない。

## Geo match / 地域制限

どこの地域からのアクセスなのかを検査する。

- Location type : `Country` のみ
- Location : ロケールの指定

## IP addresses / IP アドレス一致

どの IP からのアクセスなのか検査する。  
そのまんまなので詳細割愛。

## Size constraints / サイズ制約

HTTP の各箇所のデータサイズを検査する。

- Part of the request to filter on / 検査する場所の定義
  - 「Cross-site scripting / クロスサイトスクリプティング」節の同項目に同じ
- Comparison operator
  - 「>」「<」といった比較演算子を定義
- Size (Bytes) / データサイズの定義
- Transformation / 検査の前に行う変換を定義
  - 「Cross-site scripting / クロスサイトスクリプティング」節の同項目に同じ

## SQL injection / SQL インジェクション

悪意のある SQL が含まれていないか検査する箇所を定義する。  
設定内容は「Cross-site scripting / クロスサイトスクリプティング」節に同じ。

## String and regex matching / 文字列と正規表現の一致

「Cross-site scripting / クロスサイトスクリプティング」節の設定と同様の箇所に対して、文字列・正規表現によるマッチングの検査を行う。  
ここまでで雰囲気分かると思うので詳細省略。

# Rules / ルール

- Rate-based rule / レートベースのアクセス制限
  - https://dev.classmethod.jp/cloud/aws/aws-waf-rate-based-rules/

おそらく「ブラックリストの条件」「ホワイトリストの条件」」をまとめる用途で Rules を利用する。

# Web ACLs

あああ。

# AWS Shield

AWS WAF を適用すると AWS Shield の 「 AWS Shield Standard 」がデフォルトで適用される。  
「AWS Shield Advanced」は有償で適用することができる。