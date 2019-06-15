---
title: "Pythonで機械学習 K Meansでクラスタリング編"
date: 2017-07-05T18:00:50+09:00
slug: "python-ml-k-means-clustering"
tags:
- python
- machine learning
- scikit-learn
- k-means
draft: false
archives:
- 2017/07
---

ここでは、 **scikit-learn** で **k-means** を実行してみる。  
データセットは、以下で紹介している **iris データセット** を使用する。

- [Pythonで機械学習 データセット編](https://blog.pepese.com/entry/python-ml-dl-datasets/)

<!--more-->

# パッケージの導入

このブログにある Python コードを実行するためのパッケージをインストールする。

```sh
pip install jupyter scikit-learn matplotlib scipy
```

実行環境は Jupyter Notebook を想定。実行方法は `jupyter notebook` 。  
**matplotlib** の使い方は以下を参照。

- [matplotlib入門](http://blog.pepese.com/entry/2016/09/18/174407)

# k-means の実装

```python
import matplotlib.pyplot as plt
from sklearn import cluster
from sklearn import datasets
# iris データセットをロード
iris = datasets.load_iris()
data = iris['data']
# k-means モデルの作成
# クラスタ数は 3 を指定
model = cluster.KMeans(n_clusters=3)
model.fit(data)
# クラスタリング結果ラベルの取得
labels = model.labels_
# 以降、結果の描画
# 1 番目のキャンバスを作成
plt.figure(1)
# ラベル 0 の描画
ldata = data[labels == 0]
plt.scatter(ldata[:, 2], ldata[:, 3], color='green')
# ラベル 1 の描画
ldata = data[labels == 1]
plt.scatter(ldata[:, 2], ldata[:, 3], color='red')
# ラベル 2 の描画
ldata = data[labels == 2]
plt.scatter(ldata[:, 2], ldata[:, 3], color='blue')
# x軸、y軸の設定
plt.xlabel(iris['feature_names'][2])
plt.ylabel(iris['feature_names'][3])
plt.show()
```

<img src="../../images/k-means_result.png"  alt="iris データセット">

model の各項目は以下の通り。

- `cluster_centers_`
    - クラスタの中心座標
- `labels_`
    - 各点に付与されたラベル、つまり、クラスタリングの結果
- `inertia_`
    - 各点からそれぞれが属するクラスタの中心までの距離の挿話

# その他のクラスタリング手法

## 階層的凝集型クラスタリング

```python
model = cluster.AgglomerativeClustering(n_clusters=3, linkage='ward')
```

## 非階層的クラスタリング Affinity propagation

```python
model = cluster.AffinityPropagation()
```
