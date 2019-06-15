---
title: Pandas入門
tags:
- Python
- Pandas
id: pandas-basics
draft: true
---

Python ライブラリの Pandas の機能をまとめる。

# Pandas のオブジェクト

|オブジェクト|説明|
|:---|:---|
|pd.Series|インデックス付きの1次元データ|
|pd.DataFrame|インデックス付きの2次元データ|
|pd.Panel|インデックス付きの3次元データ|

# Series

インデックス付きの1次元データ。

# DataFrame

インデックス付きの2次元データ。

# Panel

インデックス付きの3次元データ

# Kaggle Titanic

|パラメータ|説明|
|:---|:---|
|PassengerID|乗客ID|
|Survived|生存結果 (1: 生存, 2: 死亡)|
|Pclass|乗客の階級 1が一番位が高いそう|
|Name|乗客の名前|
|Sex|性別|
|Age|年齢|
|SibSp|兄弟、配偶者の数。|
|Parch|両親、子供の数。|
|Ticket|チケット番号。|
|Fare|乗船料金。|
|Cabin|部屋番号|
|Embarked|乗船した港　Cherbourg、Queenstown、Southamptonの３種類があります|
