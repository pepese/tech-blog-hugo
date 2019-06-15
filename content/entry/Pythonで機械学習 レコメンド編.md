---
title: "Pythonで機械学習 レコメンド編"
date: 2017-11-26T18:02:36+09:00
slug: "python-ml-recommendation"
tags:
- python
- machine learning
- scikit-surprise
- 協調フィルタリング
draft: false
archives:
- 2017/11
---

代表的なレコメンドアルゴリズムと Python での実装をまとめる。  
スクラッチではなく、できるだけライブラリ（ [scikit-surprise](http://surpriselib.com/) ）を利用する。

<!--more-->

# レコメンドアルゴリズム

- ポピュラリティ
    - 所謂人気ランキング
- コンテンツベース（内容ベース）フィルタリング
    - アイテム間の類似度に基づいたレコメンド
        - アイテムの特徴ベクトルで類似度（ Cos 類似度など）ソートしてレコメンドする方法
        - 例：野球のバットを買った人には野球のボールをおすすめしよう
    - [参考](http://ohke.hateblo.jp/entry/2017/10/13/230000)
- 協調フィルタリング
    - ユーザの利用履歴を扱う
    - トランザクションデータ、ユーザ・アイテム行列
- 上記のハイブリッド

ここでは **協調フィルタリング** を扱う。

[参考：推薦システムのアルゴリズム](http://www.kamishima.net/archive/recsysdoc.pdf)

レコメンドアルゴリズムでよく発生するネガティブが出来事は以下。

- 同じようなアイテムばかりレコメンドされる
- 人気のアイテム、長期間掲載しているアイテムばかりレコメンドされる
- 数年に一度しか購入しない物に対するレコメンド結果がずっと表示される
- ユーザー行動履歴が十分に蓄積されていないと精度がでない
- データの管理コストがでかい
- レコメンドの計算コストがでかい

## 協調フィルタリング

協調フィルタリングには主に以下の種類がある。

- メモリベース（近傍ベース）
    - ユーザ・アイテム行列をそのまま利用、モデルを作成しない
    - 類似度の高いユーザ、アイテムを集め、実際の値と類似度の **加重平均** で評価値を算出する
    - 以下の手法がある
        - ユーザベース
        - アイテムベース
    - スパース性（データサイズに対して意味のある情報が少ない、 0 が多い行列）の高いデータには適用し辛い
- モデルベース
    - 事前に調べておいたデータの規則性を使って予測
    - ユーザ・アイテム行列をモデル構築に利用
    - モデルの種類には以下がある。
        - 行列因子分解（Matrix Factorization）モデル
            - ユーザ・アイテム行列をユーザ行列（user × k）とアイテム行列（item × k）に分解する際、既に値があるセルの値の誤差が最小になるようにする
                - 分解した後の行列の積をとって元に戻した際、値の入っていなかったセルに値が入っており、その値を評価値とする
            - 特異値分解（SVD：Singular Value Decomposition）
            - 非負値行列因子分解（NMF：Non-negative Matrix Factorization）
                - SVDと異なり、分解した行列の要素が全て正の数
                - 交互最小二乗法（ALS：Alternative Least Squares）や確率的勾配降下法（SGD：Stochastic Gradient Descent）を用いて実施
        - クラスタモデル
            - 嗜好が類似した利用者のグループごとに推薦をする
        - 関数モデル
            - 利用者の嗜好パターンから，アイテムの評価値を予測する関数
        - 確率モデル
            - 行動分布型：どの利用者が，どのアイテムを，どう評価したかの分布をモデル化
            - 評価分布型：全アイテムに対する評価値の同時分布をモデル化
            - ナイーブベイズ、ベイジアンネットワークなど
        - 時系列モデル
            - マルコフ過程：アイテムを評価した時間的順序も考慮
            - マルコフ決定過程(MDP：Markov Decision Process)：加えて，利用者の行動もモデル化
- 上記のハイブリッド

狭義の協調フィルタリングは、近傍ベースを指す。

### メモリベース協調フィルタリング

メモリベース協調フィルタリングのコンセプトは、「類似度」。  
過去の利用履歴から似たもの同士を明らかにし、この類似度を使ってオススメ商品を推定する。  
類似度といった場合、ユーザ同士の類似度と商品同士の類似度の 2 つが考えられ、前者を使う場合をユーザベース、後者をアイテムベースと言って区別する。

- ユーザベース
    - ユーザーの行動履歴からユーザー間の類似度を計算し、おすすめするアイテムを決める手法
    - 例
        - Aさんに何かおすすめしたい、Aさんは商品a、商品b、商品cを買っている。
        - 同じような行動履歴のBさんは商品a、商品b、商品c、商品dを買っている、ということはAさんには商品dをおすすめできる
- アイテムベース
    - ユーザーの行動履歴からアイテム間の類似度を計算し、類似するアイテムをおすすめする手法
    - 例
        - 商品Aは商品Bと一緒に購入されることが多いから、商品Aの購入者に商品Bをおすすめしよう

#### ユーザベース

ユーザベース協調フィルタリングのロジックは、いわゆるkNN回帰という機械学習手法を利用する。  
kNN（k-Nearest-Neighbor、**k近傍法** ）を簡単に説明すると、推定する対象に最も特徴が似ているk個の観測値を参考にし、値を推定しようというもの。  
アルゴリズムの概要は以下の通り。

1. あるユーザと他のユーザの **類似度** を計算
2. レコメンドしたい対象ユーザ（ユーザ A ）と類似度の高い k 名のユーザを選ぶ
3. 他のユーザ利用したアイテムのうち、ユーザAがまだ利用していないアイテムの集合を抽出
4. それらのアイテム群のうち、おすすめ度が高いアイテムのリストを返却
    - この選定の際に、類似度が高いユーザが利用したアイテムほど重みが高くなるようにする

#### アイテムベース

ユーザベースはユーザ同士の類似度を考えたのに対して、アイテムベースはアイテム同士の類似度を考える。  
それ以外の考え方はユーザベースに同じ。

#### 用いられる類似度

類似度には以下のようなものがある。

- ユークリッド距離（ Euclidean distance ）
    - 平方ユークリッド距離（ Squared Euclidean distance ）
    - 2 点間の普通の距離
- コサイン類似度（ Cosine Similarity ）
- ピアソンの積率相関係数（ Pearson correlation coefficient ）
    - ユーザの評価をそのユーザの評価全体の平均を用いて正規化する
    - データが正規化されていないような状況でユークリッド距離よりも良い結果を得られることが多いとされる

ユーザベースでは、 **ピアソンの積率相関係数** を用いる。  
そうすることによって以下のように評価の傾向が似ている 2 ユーザ間で高い相関を得られる。

- ユーザ A ：「ラーメンはまずまずで 3 点だがカレーとチャーハンはイマイチだから 1.5 点だな……。」
- ユーザ B ：「ラーメンはうまくて 5 点だがカレーとチャーハンはふつうで 3.5 点だな……。」

また、アイテムベースでは **コサイン類似度** がよく用いられる。

# 実装

scikit-surprise ベースで実装する。

## 環境設定

環境設定は以下。

```sh
$ python -V
Python 3.6.1
$ pip -V
pip 9.0.1
$ pip install jupyter scikit-learn matplotlib scipy numpy scikit-surprise
$ pip freeze | grep scikit-surprise
scikit-surprise==1.0.5
```

## データセット

データセットには [SUSHI Preference Data Sets](http://www.kamishima.net/sushi/) を使用する。

```sh
$ wget http://www.kamishima.net/asset/sushi3-2016.zip
$ unzip sushi3-2016.zip
$ rm sushi3-2016.zip
$ ls ./sushi3-2016/
README-en.txt
sushi3.idata
sushi3b.5000.10.order
README-ja.txt
sushi3.udata
sushi3b.5000.10.score
README-stat-ja.txt
sushi3a.5000.10.order
```

`sushi3b.5000.10.score` は 5000 行 100 列( 5000 人 × 寿司ネタ 100 種類)の構成となっており、各要素には評価値 0 〜 4 (値が大きいほど、好き)と、欠測値 -1 がセットされている。

## データのロード

scikit-surprise でデータをロードするには、 `Dataset` クラスに対応した形式にする必要がある。  
形式は **1行1評価値** で、以下のようにする。

```
ユーザID アイテムID 評価値
```

`sushi3b.5000.10.score` を上記の形式に変換・Dataset形式でロードするコードは以下。  
なお、ユーザ ID は 0000 〜 4999 、アイテム ID は 00 〜 99 。

```python
def convert(input_file_name):
    # 返還後のファイル名
    output_file_name = input_file_name + '_converted'
    output = ''

    with open(input_file_name, mode='r') as f:
        lines = f.readlines()
        user_id = 0
        for line in lines:
            words = line.strip().split(' ')
            for item_id, word in enumerate(words):
                score = int(word)
                if score != -1:
                    output += '{0:04d} {1:02d} {2:01d}\n'.format(user_id, item_id, score)
            user_id += 1

    with open(output_file_name, mode='w') as f:
        f.write(output)

    return output_file_name

# 「ユーザID アイテムID 評価値」形式にデータファイルを変換
output_file_name = convert('sushi3-2016/sushi3b.5000.10.score')

# with open(output_file_name, mode='r') as f:
#     print(f.read())
```

データフォーマットを変換した後、以下のように `Dataset` 形式でデータをロードし、 `trainset` を作成する。

```python
from surprise import Reader, Dataset

# ファイルからデータをロードし、Dataset形式に
# dataset.raw_ratings にデータが格納される
dataset = Dataset.load_from_file(output_file_name, reader=reader)
# dataset を計算し、trainset 形式に
trainset = dataset.build_full_trainset()
```

`dataset` から `trainset` へ変換される際、元の `user_id` `item_id` から読み込んだ順に 0 からインクリメントされた内部 ID へ変換される。

`dataset` には以下のようなフィールドがある。（面倒なので説明は一部のみ）

- build_full_trainset
- construct_testset
- construct_trainset
- folds
- load_builtin : scikit-surprise が準備しているデータセットの読み込み
- load_from_df : DataFrame 形式のデータを Dataset 形式へ変換するメソッド
- load_from_file : ファイルからデータをロードし、 Dataset 形式へ変換するメソッド
- load_from_folds : ファイルからデータをロードし、 Dataset 形式へ変換するメソッド（クロスバリデーションなどで利用）
- n_folds
- ratings_file
- raw_folds
- raw_ratings : `ユーザID アイテムID 評価値 None` 形式のデータ
- read_ratings
- reader
- shuffle
- split

`trainset` には以下のようなフィールドがある。（面倒なので説明は一部のみ）

- all_items : n_items の range を返却するメソッド
- all_ratings
- all_users
- build_anti_testset
- build_testset
- global_mean : 評価値の平均（ `_global_mean` ）を計算するメソッド
- ir : ユーザ ID をインデクスとした各アイテムの評価値のタプル
- knows_item
- knows_user
- n_items : アイテム数
- n_ratings : 評価数（ユーザ数 x アイテム数、欠損は考えられていない）
- n_users : ユーザ数
- offset
- rating_scale
- to_inner_iid : 元の `item_id` を入力すると `trainset` 内部管理の ID を返却する
- to_inner_uid：元の `user_id` を入力すると `trainset` 内部管理の ID を返却する
- to_raw_iid： `trainset` 内部管理の ID を入力すると元の `item_id` を返却する
- to_raw_uid： `trainset` 内部管理の ID を入力すると元の `user_id` を返却する
- ur : アイテム ID をインデクスとした各ユーザの評価値のタプル

## 協調フィルタリング

scikit-surprise で実装できるアルゴリズムの一覧は [ここ](http://surprise.readthedocs.io/en/stable/prediction_algorithms_package.html) 。

scikit-surprise の各アルゴリズムのクラスは `AlgoBase` クラスを継承して作成されている。  
このクラスには、 `compute_similarities` や `algo.train(trainset)` など全てのアルゴリズムで共通して利用される実装されている。  
`sim_options` オプションを利用することで、類似度の計算方法（ cosine 、 msd 、 pearson 、 pearson_baseline ）やユーザベースかアイテムベース（ user_based ）などを指定することができる。

### メモリベース協調フィルタリング

以下のような形で各アルゴリズムの学習を実行する。

```python
from surprise import KNNBasic

sim_options = {
    'name': 'pearson', # 類似度を計算する方法を指定（ cosine,msd,pearson,pearson_baseline ）
    'user_based': True # False にするとアイテムベースに
}
algo = KNNBasic(k=5, min_k=1,sim_options=sim_options)
algo.fit(trainset)
```

上記で、 **k近傍法** の学習が完了する。  
`algo.compute_similarities()` を実行するとユーザ間の類似度を計算してくれ、「ユーザ数 x ユーザ数」の 2 次元配列を返却する。  
なお、 `algo.train` を実行すると内部で `algo.compute_similarities()` が実行されており、その後 `algo.sim` で「ユーザ数 x ユーザ数」の 2 次元配列でユーザ間の類似度を取得できる。

```python
print(train.sim)
```

また、以下のように学習後のモデルを使用して評価の予測値を取得できる。

```python
user_id = '{:04d}'.format(0)
item_id = '{:02d}'.format(92)

pred = algo.predict(uid=user_id, iid=item_id)
print('Predicted rating(User: {0}, Item: {1}): {2:.2f}'.format(pred.uid, pred.iid, pred.est))
```

スクラッチでの実装例は以下。

- ユーザベース
    - http://ohke.hateblo.jp/entry/2017/09/22/230000
    - https://qiita.com/hik0107/items/96c483afd6fb2f077985
- アイテムベース
    - http://ohke.hateblo.jp/entry/2017/09/29/230000
    - https://qiita.com/kotaroito/items/6acb58bb16b68a460af9

### モデルベース協調フィルタリング

#### 特異値分解（ SVD ）

```python
from surprise import SVD

algo = SVD()
algo.fit(trainset)
```

## 精度評価

精度の評価には以下のものがある。

- MAE（Mean Abusolute Error）
    - 予測値と実測値の差の絶対値を算出し、平均したもの
- MSE (Mean Squared Error)
    - 予測値と実測値の差を二乗した値の平均値
- RMSE（Root Mean Squared Error）
    - MSEの平方根

# 参考

- [scikit-surpriseを使ってレコメンドする](http://ohke.hateblo.jp/entry/2017/10/06/230000)
- [推薦システム](http://www.kamishima.net/archive/recsys.pdf)
    - P.134：協調フィルタリング、メモリベースのチューニングのヒント
    - p.137：モデルベースの種類
    - P.215：ショートヘッド、ロングテールについて
- 精度評価
    - [レコメンドつれづれ ～第3回 レコメンド精度の評価方法を学ぶ～](http://blog.brainpad.co.jp/entry/2017/08/25/140000)
    - [レコメンド研究のあれこれ](https://www.slideshare.net/MasahiroSato2/ss-65646039)
    - [レコメンドアルゴリズムの基本と周辺知識と実装方法](https://www.slideshare.net/takemikami/ss-76817490)
    - [推薦システムの基本的な評価指標について整理してみた](https://datahotel.io/archives/4778)
    - [MatrixFacorization を使った評価予測](https://ameblo.jp/principia-ca/entry-10980281840.html)