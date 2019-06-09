---
title: "Pythonで機械学習 SVMで2クラス分類問題編"
date: 2017-07-05T08:32:21+09:00
slug: "python-ml-svm-2class"
tags:
- python
- machine learning
- scikit-learn
- svm
draft: false
archives:
- 2017/07
---

ここでは、 **scikit-learn** の **SVM** モジュールを使用して **2クラス分類問題** を解いてみる。  
データセットは、以下で紹介している **digits データセット** を使用する。

- [Pythonで機械学習 データセット編](https://blog.pepese.com/entry/python-ml-dl-datasets/)

後半で実施している精度評価についての詳細は以下を参照。

- [Pythonで機械学習 精度評価編](https://blog.pepese.com/entry/python-ml-dl-metrics-scores/)

<!--more-->

# パッケージの導入

このブログにある Python コードを実行するためのパッケージをインストールする。

```sh
$ pip install jupyter scikit-learn matplotlib scipy
```

実行環境は Jupyter Notebook を想定。実行方法は `$ jupyter notebook` 。  
**matplotlib** の使い方は以下を参照。

- [matplotlib入門](http://blog.pepese.com/entry/2016/09/18/174407)

# SVM の実装

```python
import numpy as np
from sklearn import datasets
from sklearn import svm
from sklearn import metrics
# digits データセットのロード
digits = datasets.load_digits()
# digits データセットの中から 3 と 8 の位置を取得
# これを後に学習用データと評価用データに分割して使用する
flag_3_8 = (digits.target == 3) + (digits.target == 8)
# 3 と 8 の画像データを取得（一次元ベクトル）
data = digits.data[flag_3_8]
# 上記は下記のように 8x8 のimageを2次元ベクトルから 1次元ベクトルへ変換したものでもよい
# data = images.reshape(digits.images[flag_3_8].shape[0], -1)
# 3 と 8 の数字データを取得
labels = digits.target[flag_3_8]
# 以降、ホールドアウト検証にて正答率を算出する
# データセットの数を取得
num_of_samples = len(flag_3_8[flag_3_8])
# 学習データサイズの取得
training_data_size = int(num_of_samples * 3 / 5)
# SVMモデルの作成
classifier = svm.SVC(C=1.0, gamma=0.001)
# 学習用データをSVMへ適用
# データセットの前から 3/5 を学習用データとして使用
classifier.fit(data[:training_data_size], labels[:training_data_size])
# 正解データの取得
# データセットの後ろから 2/5 を評価用データとして使用
expected = labels[training_data_size:]
# 学習済み SVM へデータをインプットして結果を取得
predicted = classifier.predict(data[training_data_size:])
# 結果の出力
# SVMの出力と正解データを比較
print('Accuracy:\n', metrics.accuracy_score(expected, predicted)) # 0.937062937063
```

scikit-learn の SVM には、 LinearSVC 、 LinearSVR 、 NuSVC 、 NuSVR 、 OneClassSVM 、 SVC 、 SVR があり、整理すると以下のようになる。

- 分類問題に使用する SVM （Support Vector Classification）
    - SVC （C-Support Vector Classification）
        - 標準的なソフトマージン(エラーを許容する)SVM
        - [API Doc](http://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html)
    - LinearSVC （Linear Support Vector Classification）
        - カーネルが線形カーネルの場合に特化したSVM
        - [API Doc](http://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html)
    - NuSVC （Nu-Support Vector Classification）
        - エラーを許容する表現が異なるSVM
        - [API Doc](http://scikit-learn.org/stable/modules/generated/sklearn.svm.NuSVC.html)
- 回帰問題に使用する SVM （Support Vector Regression）[参考](http://d.hatena.ne.jp/saket/20130212/1360656405)
    - SVR
    - LinearSVR
    - NuSVR
- 異常検知に使用する SVM
    - OneClassSVM

# 精度の評価

k=5 の k-分割交差検証で、以下を算出する。

- 混同行列（Confusion Matrix）
- 正答率（Accuracy）
- 適合率（Precision）
- 再現率（Recall）
- F値（F-measure）

```python
import numpy as np
from sklearn import datasets
from sklearn import svm
from sklearn import metrics
digits = datasets.load_digits()
flag_3_8 = (digits.target == 3) + (digits.target == 8)
data = digits.data[flag_3_8]
labels = digits.target[flag_3_8]
# ここまではさきほどと同じ
# 交差検証用モジュールのロード
import sklearn.model_selection
# 交差検証の実施
# train は学習用データ、testは評価用データ
for train, test in sklearn.model_selection.KFold(n_splits=5).split(data, labels):
  # 学習用データ
  train_data = data[train]
  train_labels = labels[train]
  # 評価用データ
  test_data = data[test]
  test_labels = labels[test]
  # SVMモデルの作成
  classifier = svm.SVC(C=1.0, gamma=0.001)
  # 学習用データをSVMへ適用
  classifier.fit(train_data, train_labels)
  # 正解データの取得
  expected = test_labels
  # 学習済み SVM へデータをインプットして結果を取得
  predicted = classifier.predict(test_data)
  # ここまではさきほどと同じ
  # 結果の出力
  # SVMの出力と正解データを比較
  print('Confusion Matrix:\n', metrics.confusion_matrix(expected, predicted))
  print('Accuracy:\n', metrics.accuracy_score(expected, predicted))
  print('Precision:\n', metrics.precision_score(expected, predicted, pos_label=3))
  print('Recall:\n', metrics.recall_score(expected, predicted, pos_label=3))
  print('F-measure:\n', metrics.f1_score(expected, predicted, pos_label=3), '\n')
```

ここでは、数字の「3」か数字の「8」かを分類する問題であったので、 `pos_label=3` は Positive を数字の「3」に指定したことになる。（ Negative は数字の「8」）
