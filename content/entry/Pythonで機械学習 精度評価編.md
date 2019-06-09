---
title: "Pythonで機械学習 精度評価編"
date: 2017-08-07T08:29:13+09:00
slug: "python-ml-metrics-score"
tags:
- python
- 機械学習
- scikit-learn
draft: false
archives:
- 2017/08
---

ここでは、機械学習で作成したモデルの精度評価・向上方法を記載する。

- 学習用データと評価用データの使い方
- 精度の指標
- 精度の向上

<!--more-->

ここで述べる手法は **scikit-learn** に実装されている。  
なお実装例については以下記事の後半で紹介している。

- [Pythonで機械学習 SVMで2クラス分類問題編](https://blog.pepese.com/entry/python-ml-dl-svm-2class/)
- [Pythonで機械学習 SVMで多クラス分類問題編](https://blog.pepese.com/entry/python-ml-dl-svm-multiclass/)

# 学習用データと評価用データの使い方

教師あり学習前提ではあるが、正解付けしたデータは学習用データと評価用データに分割して以下のような方法でモデルを評価する。

- ホールドアウト検証
    - データセットを学習用データと評価用データへ2分割する方法
    - 割合は任意
- k-分割交差検証
    - データセットを k 分割して、そのうち 1 個を評価用データ、その他の k-1 個を学習用データとして使用する方法
    - 評価用データを変更しながら k 回繰り返した結果の平均を精度に用いることが多い
    - scikit-learn 0.18 までは `sklearn.cross_validation.KFold` モジュールだったが DeprecationWarning となっており、0.20 からは `sklearn.model_selection.KFold` を使用する
        - [参考](http://segafreder.hatenablog.com/entry/2016/10/18/163925)


# 精度の指標

精度の指標には以下がある。

- 正答率（Accuracy）
    - 全体の事象の中で正解がどれだけあったかの割合
- 適合率（Precision）
    - モデルが Positive と判断した中で、本当に Positive なものの割合
- 再現率（Recall）
    - 本当に Positive なものの中で、モデルが Positive と判断できたものの割合
- F値（F-measure）
    - 適合率と再現率の調和平均
    - 適合率の再現理を総合的に見る際に使用

上記は以下の **混同行列** （ **Confusion Matrix** ）から算出することができる。

<table>
<tr>
<td></td><td></td><td colspan="2">予測</td>
</tr>
<tr>
<td></td><td></td><td>Positive</td><td>Negative</td>
</tr>
<tr>
<td rowspan="2">実際</td><td>Positive</td><td>True Positive (TP)</td><td>False Negative (FN)</td>
</tr>
<tr>
<td>Negative</td><td>False Positive (FP)</td><td>True Negative (TN)</td>
</tr>
</table>

- TP・・・Positive という予測が True （正解）だった数
- FP・・・Positive という予測が False （不正解）だった数
- TN・・・Negative という予測が True （正解）だった数
- FN・・・Negative という予測が False （不正解）だった数

上記を用いてそれぞれの指標値は以下のように算出できる。

$$ 正答率（Accuracy）=\frac{TP+TN}{TP+FP+FN+TN} $$

$$ 適合率（Precision）=\frac{TP}{TP+FP} $$

$$ 再現率（Recall）=\frac{TP}{TP+FN} $$

$$ F値（F-measure）=\frac{ 2 \times 適合率 \times 再現率 }{ 適合率 + 再現率 } $$

これらの使用値と混同行列は scikit-learn の metrics モジュールを用いて、以下のように算出できる。

- 正答率（Accuracy）
    - `metrics.accuracy_score(expected, predicted)`
- 適合率（Precision）
    - `metrics.precision_score(expected, predicted, pos_label=3)`
- 再現率（Recall）
    - `metrics.recall_score(expected, predicted, pos_label=3)`
- F値（F-measure）
    - `metrics.f1_score(expected, predicted, pos_label=3)`
- 混同行列（Confusion Matrix）
    - `metrics.confusion_matrix(expected, predicted)`

上記の `pos_label` は **Positive Label** のこと。  
2クラス分類問題などでは片方が `Positive` 、もう片方が `Negative` となるため、片方のラベルを Positive として設定してあげることになる。  
繰り返しになるが、実装例については以下記事の後半で紹介している。

- [Pythonで機械学習 SVMで2クラス分類問題編](https://blog.pepese.com/entry/python-ml-dl-svm-2class/)
- [Pythonで機械学習 SVMで多クラス分類問題編](https://blog.pepese.com/entry/python-ml-dl-svm-multiclass/)

# 精度の向上

精度向上の方法にはいくつかあるが、以下のような方法が考えられる。

- パラメータチューニング
- アンサンブル学習

## パラメータチューニング

パイパーパラメータの最適値を選ぶには以下の手法があり、うち **グリッドサーチ** は scikit-learn で実装されている。[参考](http://sucrose.hatenablog.com/entry/2013/05/25/133021)

- グリッドサーチ
    - 昔からある手法で、格子状の空間で最適なパラメータを探索する方法です
    - 格子の範囲を総当りするため、膨大な計算時間がかかるという課題がある
- ランダムサーチ
    - 無作為にパラメータを抽出して探索する
    - グリッドサーチよりも計算時間が短くて済むというメリットがある
- ベイズ最適化 (Bayesian Optimization)
    - 最初は無作為にパラメータを抽出して探索するが、その結果を使って次に探索する点を確率的に選ぶ方法
    - ディープラーニングを含む機械学習の手法で、比較的良いハイパーパラメータを探索できることが知られている
    - 特に、低次元のデータで大域的な解を求めたい場合や、グリッドサーチなどで時間がかかっている場合に有効な手法

## アンサンブル学習

**アンサンブル学習** とは、いくつかの性能の低い分類器（弱仮説器）を組み合わせて、性能の高い 1 つの分類器を作る方法。  
弱仮説器のアルゴリズムは決まっておらず、適宜選定する必要がある。  
アンサンブル学習のイメージは「多数決」であり、以下の 2 つに分類できる。

- バギング
    - 学習データを抜けや重複を許して複数個のグループに分割し、学習データのグループ毎に弱仮説器を生成する方法
    - 分類時は、各弱仮説器の出力した分類結果の多数決をとる
- ブースティング
    - 複数の弱仮説器を用意し、重み付きの多数決で分類を実現する方法
    - 難易度の高い学習データを正しく分類できる弱仮説器の半径結果を重視されるように重みを更新していく
    - ある弱仮説器が間違ったデータを難易度の高いデータとし、「難易度の高いデータの抽出」と「難易度の高いデータに特化した弱仮説器の重みの計算」を反復して弱仮説器の重みを決定する

**Random Forest** はバギング、 **AdaBoost** はブースティングの例である。
