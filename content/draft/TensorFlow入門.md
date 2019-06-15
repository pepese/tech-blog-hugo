---
title: TensorFlow入門
tags:
id: tensorflow-basics
draft: true
---

TensorFlow触ってみる。

- 環境構築
- 用語
- 代表的な関数
- MNISTサンプル

# 環境構築

Macにanyenvが導入されていること前提。

## python

```sh
$ anyenv install pyenv
$ exec $SHELL -l
$ which pyenv
/Users/xxxx/.anyenv/envs/pyenv/bin/pyenv
$ pyenv install --list
$ pyenv install 3.6.1
$ pyenv global 3.6.1
$ pyenv version
3.6.1 (set by /Users/xxxx/.anyenv/envs/pyenv/version)
$ pyenv versions
  system
* 3.6.1 (set by /Users/xxxx/.anyenv/envs/pyenv/version)
$ python --version
Python 3.6.1
$ which pip
/Users/xxxx/.anyenv/envs/pyenv/shims/pip
$ which pip3
/Users/xxxx/.anyenv/envs/pyenv/shims/pip3
$ pip --version
pip 9.0.1 from /Users/xxxx/.anyenv/envs/pyenv/versions/3.6.1/lib/python3.6/site-packages (python 3.6)
$ pip3 --version
pip 9.0.1 from /Users/xxxx/.anyenv/envs/pyenv/versions/3.6.1/lib/python3.6/site-packages (python 3.6)
```

## TensorFlow

pip/pip3 のバージョンが 8.1 以上である必要がある。

```sh
$ pip3 install --upgrade tensorflow
```

もしエラーがでたら以下を実行。

```sh
$ pip3 install --upgrade tfBinaryURL
```

正しくインストールできているか確認。

```sh
$ python
>>> import tensorflow as tf
```

Python3 しか入っていなければ、 `pip3` ではなく `pip` でおk。

[参考](https://www.tensorflow.org/install/install_mac)

# 用語

- テンソル（Tensor）
    - 線形的な量または線形的な幾何概念を一般化したもの
    - 規定を選べば、多次元の配列として表現できるようなもの
    - テンソル自身は、特定の座標系によらないで定まる対象
    - **多次元配列** でおk
    - テンソルの種類
        - 0階テンソル・・・スカラ、ただの数値
        - 1階テンソル・・・ベクトル、配列
        - 2階テンソル・・・行列、二次元配列
        - 3階テンソル・・・行列が複数個あるもの、三次元配列
        - 4階テンソル・・・行列が複数個あるものが複数個あるもの、四次元配列
- セッション（Session）
    - ニューラルネットワークを実行する単位のこと
    - プログラム上ではセッションを作成して、そこに実行するノードを指定することになる
- ニューラルネットワーク
    - 層の名前
        - 入力層
            - 値を入力するノードの層
        - 隠れ層、中間層
            - 入力層と出力層の間のノードの層
            - ここが多層になるとディープラーニングと言われる
        - 出力層
            - 結果を出力するノードの層


# 代表的な関数

TensorFlow の代表的な関数とその使い方を記載する。  
詳細は[API Doc](https://www.tensorflow.org/api_docs/python/)を参照。

## import

関数ではないがメモ的に書いておく。

```python
import tensorflow as tf
```

## 定数

```python
# const1 と命名した値 3 の定数ノード
node1 = tf.constant(3, name="const1")
```

## 変数

```python
# val1 と命名した初期値 0 の変数ノード
node2 = tf.Variable(0, name="val1")
```

TensorFlow では **重み $w$** を変数（ Variable ）で置く。

## 行列

```python
x = tf.placeholder(tf.float32, [None, 5])
```

TensorFlowでは **学習データ/入力値 $x$** をプレースホルダ（ placeholder : 入力値を色々変更できるノード ）で置く。  
第一引数は要素のデータ型、第二引数は行列のサイズ（ None にすることで任意のサイズを指定できる）。  
データ型には以下がある。

|データ型|説明|
|:---|:---|
|tf.float32|32 ビット浮動小数点型|
|tf.float64|64 ビット浮動小数点型|
|tf.int8|8 ビット符号付き整数型|
|tf.int16|16 ビット符号付き整数型|
|tf.int32|32 ビット符号付き整数型|
|tf.int64|64 ビット符号付き整数型|
|tf.string|文字列型|
|tf.bool|真偽値型|
|tf.complex|複素数型|


変数には初期化が必要なので注意が必要。初期化の方法については後述。

## 足し算

```python
# node1 と node2 を足し算するノード
node3 = tf.add(node1, node2)
```

## 変数の初期化

```python
init = tf.global_variables_initializer()
```

かつては `initialize_all_variables` だったが、 r0.12 から Deprecated になっているので `global_variables_initializer` を使用する。

## セッション（Session）

### 作成

```python
sess = tf.Session()
```

> TensorFlow は CPU の拡張命令を使用可能なのだが、これが有効になっていない場合、下記のような警告がでる。
> ```sh
> 2017-06-20 16:32:03.465042: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
> 2017-06-20 16:32:03.465069: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
> 2017-06-20 16:32:03.465078: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX2 instructions, but these are available on your machine and could speedup CPU computations.
> 2017-06-20 16:32:03.465086: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use FMA instructions, but these are available on your machine and could speed up CPU computations.
> ```

### 変数を初期化

```python
init = tf.global_variables_initializer()
sess.run(init)
```

もしくはシンプルに以下。

```python
sess.run(tf.global_variables_initializer())
```

### 実行

ノードは作成してもセッション上で実行しなければ値は出力されない。  
セッションの実行方法は以下。

```python
print(sess.run([node1, node2]))
# 出力：[3, 0]
```

下記は node1 と node2 の出力を足し算する node3 を実行した結果となる。  

```python
print(sess.run(node3))
# 出力：3
```

## TensorFlow の実行

```python
tf.app.run()
```

## assign

あるノードのアウトプットを別のノードへインプットする。

```python
assign = tf.assign(from_node, to_node)
```

## プレースホルダへの値の入力

```python
input = tf.placeholder(tf.int32, name="input")
```

以下の `feed_dict` でセッション実行時に入力値を指定できる。

```python
sess.run(run_node, feed_dict={input:3})
```

# MNISTサンプル

公式の[MNISTサンプル](https://www.github.com/tensorflow/tensorflow/blob/r1.2/tensorflow/examples/tutorials/mnist/mnist_softmax.py)を解説する。

```python
import argparse
import sys

from tensorflow.examples.tutorials.mnist import input_data

import tensorflow as tf

FLAGS = None

def main(_):
  # データセットの読み込み
  mnist = input_data.read_data_sets(FLAGS.data_dir, one_hot=True)
  # mnist.train.images        ・・・訓練用入力データの配列
  # mnist.train.labels        ・・・訓練用正解データの配列
  # mnist.train.next_batch(50)・・・訓練データを50個取り出して、(入力データ,正解データ)のタプルで返却

  # 入力層 28x28=784 の画像
  x = tf.placeholder(tf.float32, [None, 784])
  # 隠れ層の重み 入力が784 出力が10
  W = tf.Variable(tf.zeros([784, 10]))
  # バイアス 固定の入力 出力が10
  b = tf.Variable(tf.zeros([10]))
  # 出力層 [None, 10]のベクトル
  y = tf.matmul(x, W) + b

  # 教師データの入れ物
  y_ = tf.placeholder(tf.float32, [None, 10])

  # The raw formulation of cross-entropy,
  #
  #   tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(tf.nn.softmax(y)),
  #                                 reduction_indices=[1]))
  #
  # can be numerically unstable.
  #
  # So here we use tf.nn.softmax_cross_entropy_with_logits on the raw
  # outputs of 'y', and then average across the batch.

  # 交差エントロピーの作成
  # logits：最終的な推計値。softmaxはする必要ない
  # labels：教師データ
  # tf.reduce_mean：平均値
  cross_entropy = tf.reduce_mean(
      tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y))
  train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)

  sess = tf.InteractiveSession()
  tf.global_variables_initializer().run()
  # Train
  for _ in range(1000):
    batch_xs, batch_ys = mnist.train.next_batch(100)
    sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})

  # Test trained model
  correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
  accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
  print(sess.run(accuracy, feed_dict={x: mnist.test.images,
                                      y_: mnist.test.labels}))

if __name__ == '__main__':
  parser = argparse.ArgumentParser()
  parser.add_argument('--data_dir', type=str, default='/tmp/tensorflow/mnist/input_data',
                      help='Directory for storing input data')
  FLAGS, unparsed = parser.parse_known_args()
  tf.app.run(main=main, argv=[sys.argv[0]] + unparsed)
```

# その他 TensorFlow 系ライブラリ・ツール

## TFLearn

**TFLearn** はTensorFlowをScikit-learnライクに使えるライブラリのことで、Githubにサンプルコードが幾つか掲載されている。  
[参考](http://qiita.com/kenta1984/items/4452e91db806ee765a78)

## TensorBoard

TensorFlow と一緒にインストールされている。

```sh
$ which tensorboard
/Users/xxxx/.anyenv/envs/pyenv/shims/tensorboard
```

### 使い方

コード中に `writer = tf.summary.FileWriter('cnn', sess.graph)` のような感じで表示対象を出力しておき、下記のコマンドで TensorBoard を起動する。

```sh
$ tensorboard --logdir=./cnn
```

- [TensorFlowとTensorBoardでニューラルネットワークを可視化](http://qiita.com/sergeant-wizard/items/fdf4d64a0d221a81da34)

# 参考

- [深層学習とTensorFlow入門](https://www.slideshare.net/tak9029/tensorflow-67483532)
    - 超いいこれ
- 良書
    - [TensorFlowで学ぶディープラーニング入門 ~畳み込みニューラルネットワーク徹底解説~](https://www.amazon.co.jp/TensorFlow%E3%81%A7%E5%AD%A6%E3%81%B6%E3%83%87%E3%82%A3%E3%83%BC%E3%83%97%E3%83%A9%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0%E5%85%A5%E9%96%80-%E7%95%B3%E3%81%BF%E8%BE%BC%E3%81%BF%E3%83%8B%E3%83%A5%E3%83%BC%E3%83%A9%E3%83%AB%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%E5%BE%B9%E5%BA%95%E8%A7%A3%E8%AA%AC-%E4%B8%AD%E4%BA%95-%E6%82%A6%E5%8F%B8/dp/4839960887/ref=sr_1_1?s=books&ie=UTF8&qid=1506557557&sr=1-1&keywords=tensorflow%E3%81%A7%E5%AD%A6%E3%81%B6%E3%83%87%E3%82%A3%E3%83%BC%E3%83%97%E3%83%A9%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0%E5%85%A5%E9%96%80)

# MNISTチュートリアル

- [TensorFlow 日本語 MNIST](http://www.tensorflow-partner.jp/mnist-beginner)
- [TensorFlow 公式 MNIST](https://www.tensorflow.org/get_started/mnist/beginners)
- [MNISTチュートリアルの日本語解説](http://qiita.com/sergeant-wizard/items/55256ac6d5d8d7c53a5a)
- [MNISTチュートリアルの日本語解説](http://qiita.com/haminiku/items/36982ae65a770565458d)
- [for Beginners と for Experts の間を埋めたい](http://qiita.com/TomokIshii/items/92a266b805d7eee02b1d)
- [交差エントロピー（cross entropy）](https://ja.wikipedia.org/wiki/%E4%BA%A4%E5%B7%AE%E3%82%A8%E3%83%B3%E3%83%88%E3%83%AD%E3%83%94%E3%83%BC)
    - コスト関数として使用する
- [確率的勾配降下法](https://ja.wikipedia.org/wiki/%E7%A2%BA%E7%8E%87%E7%9A%84%E5%8B%BE%E9%85%8D%E9%99%8D%E4%B8%8B%E6%B3%95)
    - 学習時にトレーニングセットを **ミニバッチ** に分割してそれぞれのミニバッチでパラメータが収束するまで学習を繰り返す方法
