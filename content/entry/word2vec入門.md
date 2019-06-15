---
title: "Word2vec入門"
date: 2018-02-04T18:10:43+09:00
slug: "word2vec-basics"
tags:
- python
- gensim
- word2vec
draft: false
archives:
- 2018/02
---

Mecab については以下を参考。

- [形態素解析システムMeCab入門](https://blog.pepese.com/entry/mecab-basics/)

<!--more-->

# コマンドで word2vec

## 導入

Macでの導入方法。

```sh
$ git clone https://github.com/svn2github/word2vec.git
$ cd word2vec/
$ make
gcc word2vec.c -o word2vec -lm -pthread -O3 -march=native -Wall -funroll-loops -Wno-unused-result
gcc word2phrase.c -o word2phrase -lm -pthread -O3 -march=native -Wall -funroll-loops -Wno-unused-result
gcc distance.c -o distance -lm -pthread -O3 -march=native -Wall -funroll-loops -Wno-unused-result
distance.c:18:10: fatal error: 'malloc.h' file not found
#include <malloc.h>
         ^~~~~~~~~~
1 error generated.
make: *** [distance] Error 1
```

Mac では上記の通りエラーが発生する。  
Mac では `<malloc.h>` ではなく `<stdlib.h>` を使用するので以下を実行してから make する。

```sh
$ sed -ie 's/#include <malloc.h>/#include <stdlib.h>/g' *.c
```

なお、コンパイルして作成した `word2vec` などのコマンドは `make` を実行したカレントに作成されており、パスは通っていない。

## 実行

動作確認用の `./demo-word.sh` を実行する。

```sh
$ brew install wget
$ brew install gzip
$ ./demo-word.sh
$ ./distance vectors.bin
Enter word or sentence (EXIT to break): dog

Word: dog  Position in vocabulary: 1902

                                              Word       Cosine distance
------------------------------------------------------------------------
                                              dogs		0.637071
                                           spaniel		0.604613
                                            borzoi		0.582073
                                             hound		0.579149
                                           mastiff		0.573205
                                           terrier		0.563618
                                              hund		0.552174
                                               ...
Enter word or sentence (EXIT to break):
```

上記が成功したら英語コーパス（text8）による学習は完了しているので、次回以降は `./distance vectors.bin` で実行できる。  
また `word2vec` コマンドの学習インプットとなるテキストは **分かち書き** である必要がある。  
上記のデモは英語をコーパスとして使用しているため、もともと分かち書きである。  
日本語コーパスをインプットとする場合は分かち書きした状態にする必要がある。

## word2vec の機能

- distance
    - 入力した単語の類義語や同義語を返す
- analogy
    - 単語の足し算、引き算ができる

## 日本語学習済みモデル

```sh
$ wget http://www.cl.ecei.tohoku.ac.jp/~m-suzuki/jawiki_vector/data/20170201.tar.bz2
$ tar xf 20170201.tar.bz2
$ ./distance 20170201
```

## 日本語のコーパス

青空文庫

```sh
$ brew install nkf
$ wget http://www.aozora.gr.jp/cards/001847/files/57347_ruby_57225.zip
$ unzip 57347_ruby_57225.zip
$ nkf -g rojinto_umi.txt
Shift_JIS
$ nkf -w --overwrite rojinto_umi.txt
$ nkf -g rojinto_umi.txt
UTF-8
$ cat rojinto_umi.txt | mecab -Owakati > rojinto_umi_wakati.txt
$ time ./word2vec -train rojinto_umi_wakati.txt -output rojinto_umi_wakati.model -size 200 -window 5 -sample 1e-3 -negative 5 -hs 0 -binary 1
$ ./distance rojinto_umi_wakati.model
Enter word or sentence (EXIT to break): 老人

Word: 老人  Position in vocabulary: 22

                                              Word       Cosine distance
------------------------------------------------------------------------
                                               　		0.913845
                                         分かっ		0.904279
                                         無かっ		0.865596
```

[参考](http://www.cl.ecei.tohoku.ac.jp/~m-suzuki/jawiki_vector/)

## コーパスの収集して学習・実行する

コーパスとして Wikipedia のデータを使用する。  
word2vec へのインプットとして上記のコーパスが分かち書きされた 1つのテキストデータを作成する必要がある。

```sh
$ nohup wget https://dumps.wikimedia.org/jawiki/latest/jawiki-latest-pages-articles.xml.bz2 &
$ less nohup.out
Redirecting output to ‘wget-log’.
$ tail -f wget-log
```

上記で取得したデータは XMLファイルなので、テキストファイルに整形する必要がある。  
`wp2txt` という Ruby 系のツールを使用する。  
インストールは以下。

```sh
$ gem install wp2txt
```

実行。

```sh
$ mkdir jawiki-latest-pages-articles
$ cd jawiki-latest-pages-articles
$ wp2txt --input-file ../jawiki-latest-pages-articles.xml.bz2
```

出力されたファイルを結合して分かち書きファイルを作成する。

```sh
$ cd ..
$ cat jawiki-latest-pages-articles/*.txt | mecab -Owakati > jawiki-latest-pages-articles-wakati-ipadic.txt
```

コーパスを使って学習する。

```sh
$ time ./word2vec -train jawiki-latest-pages-articles-wakati-ipadic.txt -output jawiki-latest-pages-articles-wakati-ipadic.bin -size 200 -window 5 -sample 1e-3 -negative 5 -hs 0 -binary 1
```

オプションは以下。（ `$ ./word2vec` コマンドで表示される）

- -train
    - 学習に使用するファイル。分かち書きが必要。
- -output
    - 学習結果を出力するファイル名
- -size
    - ベクトルの次元数
- -window
    - 指定した数値の分だけ、単語の前後にある単語を文脈として判断させる
- -sample
    - 単語を無視する頻度の閾値。1e-3は「頻出度が高め」 の意味。
    - あまりに高い頻度で出現する単語は意味のない単語である可能性が高いので、無視する。
- -hs
    - 学習に階層化ソフトマックスを使用するかどうか
- -negative
    - ネガティブサンプリングに用いる単語数、ランダムに間違った解答として判断させる
- -threads
    - 学習に使用するスレッド数
- -iter
    - トレーニング反復回数
- -min-count
    - n回未満登場する単語を破棄
- -alpha
    - 学習率
    - 高いほど収束が速いですが、高すぎると発散します。低いほど精度が高いですが、収束が遅くなります。
- -classes
    - ベクトルよりもワードクラスを優先させる
- -debug
    - デバッグモード
- -binary
    - バイナリ形式で出力するかどうか
    - `-binary 0` で出力ファイルを見ると各単語のベクトルが見れる
- -save-vocab
    - 語彙をファイル保存
- -read-vocab
    - 語彙をファイルから使用
- -cbow
    - 学習モデル CBOW を使うか、Skip-gram を使うか

以下で学習データを使って実行。

```sh
$ ./distance jawiki-latest-pages-articles-wakati-ipadic.bin
Enter word or sentence (EXIT to break):
```

# Pythonで word2vec

```sh
$ pip install gensim
```

学習の実装は以下。

```python
# -*- coding: utf-8 -*-
from gensim.models import word2vec
import logging

logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

sentences = word2vec.Text8Corpus('jawiki_wakati.txt')

model = word2vec.Word2Vec(sentences, size=200, min_count=20, window=15)

model.save("jawiki_wakati.model")
```

2単語の類似度を出力する実装は以下。  
（どうやら Cos 類似度の模様）

```python
# -*- coding: utf-8 -*-
from gensim.models import word2vec
import logging
import sys

logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

model = word2vec.Word2Vec.load("jawiki_wakati.model")
argvs = sys.argv
print model.similarity(argvs[1], argvs[2])
```

```sh
$ python similarity.py 日本 フィリピン
```

単語のベクトルを出力する実装は以下。

```python
# -*- coding: utf-8 -*-
from gensim.models import word2vec
import logging
import sys

logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

model = word2vec.Word2Vec.load("jawiki_wakati.model")
argvs = sys.argv
print model[argvs[1]]
```