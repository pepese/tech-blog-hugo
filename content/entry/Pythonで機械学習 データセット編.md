---
title: "Pythonで機械学習 データセット編"
date: 2017-07-05T08:38:14+09:00
slug: "python-ml-datasets"
tags:
- python
- machine learning
- scikit-learn
draft: false
archives:
- 2017/07
---

ここでは、機械学習で利用するデータセットを紹介する。

<!--more-->

# パッケージの導入

このブログにある Python コードを実行するためのパッケージをインストールする。

```sh
pip install jupyter scikit-learn matplotlib scipy
```

実行環境は Jupyter Notebook を想定。実行方法は `jupyter notebook` 。  
**matplotlib** の使い方は以下を参照。

- [matplotlib入門](http://blog.pepese.com/entry/2016/09/18/174407)

# digits データセット

digits データセットは手書き数字のデータセットで、8x8 ピクセルのモノクロ画像 1,797 枚。

```python
import matplotlib.pyplot as plt
from sklearn import datasets
# digits データセットをロード
digits = datasets.load_digits()
type(digits) # sklearn.datasets.base.Bunch
dir(digits) # ['DESCR', 'data', 'images', 'target', 'target_names']
type(digits.DESCR) # データセットの説明 # str
type(digits.data) # numpy.ndarray
type(digits.data) # numpy.ndarray
print(digits.data.shape) # (1797, 64) # 8x8ではなく8x8=64の形式の1次元ベクトルデータ
type(digits.images) # numpy.ndarray
print(digits.images.shape) # (1797, 8, 8) # 8x8の行列データ
print(digits.data == digits.images.reshape(digits.images.shape[0], -1)) # True ..., True
type(digits.target) # 番号の種類のID　# numpy.ndarray
print(digits.target.shape) # 1,797
type(digits.target_names) # 番号の種類のIDに対応する番号 # numpy.ndarray
print(digits.target_names) # [0 1 2 3 4 5 6 7 8 9]
# 画像データを matplotlib で表示
# 1 番目のキャンバスを作成
plt.figure(1)
# digits データセットから先頭 10 枚を取得
# label にはその画像の数字、imgには画像が入る
for label, img in zip(digits.target[:10], digits.images[:10]):
  # キャンバスを 2 行 5 列に分割した label+1 番目のグラフ
  plt.subplot(2, 5, label + 1)
  # 軸の設定を off
  plt.axis('off')
  # 画像をグレースケールで subplot した箇所に貼り付け
  plt.imshow(img, cmap=plt.cm.gray_r, interpolation='nearest')
  # 画像のタイトルを label （その数字）で設定
  plt.title('Digit: {0}'.format(label))
# キャンバスの描画
plt.show()
```

<img src="/img/digits_datasets.png"  alt="digits データセット">

# iris データセット

iris データセットは植物の「あやめ」に関するデータセット。  
「萼片（がくへん）の長さ」「萼片の幅」「花びらの長さ」「花びらの幅」に関する特長量と、アヤメの種類（0:setosa、1:versicolor、2:virginica）の値を持つ 50 個のデータセットである。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn import datasets
# iris データセットをロード
iris = datasets.load_iris()
type(iris) # sklearn.datasets.base.Bunch
dir(iris) # ['DESCR', 'data', 'feature_names', 'target', 'target_names']
type(iris.DESCR) # データセットの説明 # str
type(iris.data) # アヤメに関する特長量 # numpy.ndarray
type(iris.feature_names) # 特長良のカラム名 # list
print(iris.feature_names) # ['sepal length (cm)', 'sepal width (cm)', 'petal length (cm)', 'petal width (cm)']
type(iris.target) # アヤメの種類のID # numpy.ndarray
type(iris.target_names) # アヤメの種類のIDに対応する名前 # numpy.ndarray
print(iris.target_names) # ['setosa', 'versicolor', 'virginica']
# 画像を matplotlib で表示
# 1 番目のキャンバスを作成
plt.figure(1)
features = iris.data
target = iris.target
target_names = iris.target_names
labels = target_names[target]
#
setosa_petal_length = features[labels == 'setosa', 2]
setosa_petal_width = features[labels == 'setosa', 3]
setosa = np.c_[setosa_petal_length, setosa_petal_width]
versicolor_petal_length = features[labels == 'versicolor', 2]
versicolor_petal_width = features[labels == 'versicolor', 3]
versicolor = np.c_[versicolor_petal_length, versicolor_petal_width]
virginica_petal_length = features[labels == 'virginica', 2]
virginica_petal_width = features[labels == 'virginica', 3]
virginica = np.c_[virginica_petal_length, virginica_petal_width]
#
plt.scatter(setosa[:, 0], setosa[:, 1], color='red')
plt.scatter(versicolor[:, 0], versicolor[:, 1], color='blue')
plt.scatter(virginica[:, 0], virginica[:, 1], color='green')
#
plt.show()
```

<img src="/img/iris_datasets.png"  alt="iris データセット">

# オリジナルデータを CSV で扱う

CSV形式のデータを取り扱う場合、基本的に以下の4通りある。

1. テキストファイル読み込む
2. Python csv モジュールを使う
3. NumPy モジュールを使う
4. pandas モジュールを使う
