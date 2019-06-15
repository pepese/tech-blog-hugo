---
title: Pandas入門 Series編
tags:
- Python
- Pandas
- Series
id: pandas-basics-series
draft: true
---

Python ライブラリである Pandas の Series についてまとめる。

- [公式ドキュメント](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.html)

<!-- more -->

# 基本操作

# StringMethods

- [公式ドキュメント](https://pandas.pydata.org/pandas-docs/stable/api.html#string-handling)

`series.str` で Python の `str` 同様の扱いができる。  
この `str` が持つメソッドを StringMethods という。

## 正規表現（ `match` ）

正規表現にマッチすれば `True` 、そうでなければ `False` の Series を返却する。

```python
pat = r"正規表現"
series.str.match(pat, na=False)
```

# 代表的なメソッド

|メソッド|説明|
|:---|:---|
| `value_counts()` |ユニークな値がインデックスでその値の出現数が値の Series が返却される|
| `count()` | NA/null でない行数|
| `isin([リスト])` |セルの値がリストに含まれていれば True 、そうでなければ False の Series を返却する|
| `isnull()` |セルが None であれば True 、そうでなければ False の Series を返却する|
| `sort_values()` |値でソートされた Series を返却する|