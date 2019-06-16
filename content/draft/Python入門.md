---
title: Python入門
tags:
- Python
id: python-basics
draft: true
---

自分にとって最低限のことだけまとめる、というか整理。

- Windows での環境構築
- モジュールとパッケージ
- 起動モジュール
- 組み込み型
- 制御文
- 関数
- 内包表記

<!-- more -->

公式ドキュメントは以下。

- [公式サイト](https://www.python.org/)
- 公式ドキュメント一覧
    - [Python2系](https://docs.python.jp/2/index.html)
    - [Python3系](https://docs.python.jp/3/index.html)

# 環境構築

## Windows

1. Anaconda の導入
    - [公式](https://www.anaconda.com/download/)
2. Microsoft VisualC++ Build Tools の導入
    - [直リンク](http://go.microsoft.com/fwlink/?LinkId=691126&fixForIE=.exe.)
    - [公式](http://landinghub.visualstudio.com/visual-cpp-build-tools)

## Mac

anyenv の pyenv で構築する。

- [すべての**envを管理するanyenv](https://blog.pepese.com/entry/anyenv/)

## pip

パッケージ管理コマンドの `pip` 。  
`pip3` や `pip2` などがあるが、 Python 3 系を明示したい場合は `pip3` と打っておくのが安全。  
`pip` 自体をバージョンアップする方法は以下。

```
$ pip3 install -U pip setuptools
$ pip3 list
Package    Version
---------- -------
pip        19.0.3 
setuptools 40.8.0
```

## pipenv

`pip` と `virtualenv` と `requirements.txt` をラップするツール。もう悩まない。  
`Pipfile` および `Pipfile.lock` を使ってパッケージのバージョンと依存関係を管理する。

```bash
$ brew install pipenv
$ cd myproject
$ pipenv install
$ pipenv install <package>
$ pipenv shell
$ python --version
```

## VSCode

プロジェクトディレクトリで以下を実施してからVSCode に Python Extensions を入れる。

```bash
$ pipenv shell
$ pipenv install pylint --dev
```

# モジュールとパッケージ

## モジュール

他のプログラムから再利用できるようにした Python ソースコードファイルのことを **モジュール** という。（あくまで **ソースファイルがモジュール** ）  
デフォルトで組み込まれているモジュールのことを **組み込みモジュール** という。  
import の形式は複数あるが、例えば以下。

```Python
from モジュール名 import メンバ,メンバ,…
```

`from math import cos,sin,tan` と記載すると `math.cos` ではなく `cos` で呼び出すことができる。  
`from` 句でモジュール（つまり Python ソースファイル）まで読む。  
**メンバ** はソースコード内の **クラス** や **メソッド** を指す。  
以下注意事項。

- モジュールはimportによって読み込まれた時点で実行される
- グローバルスコープはモジュールに限定される
- グローバルスコープに宣言されたオブジェクトはモジュールオブジェクトの属性としてアクセスできる（ **モジュール変数** ）
    - `from` を利用した場合は、モジュール変数も同一スコープに入る

## パッケージ

Pythonではディレクトリに `__init__.py`  （ `_` 2つ）があれば、そのディレクトリを **パッケージ** として扱える。（ **ディレクトリがパッケージ**）  
import の際は以下の記述を行う。

```Python
import パッケージ名(.サブパッケージ名).モジュール名
```

## `PYTHONPATH`

カレントディレクトリ以外に参照されるディレクトリを列挙したのが `PYTHONPATH` 環境変数。  
ビルトインモジュールやサードパーティモジュールはPYTHONPATHに登録されたディレクトリに配置されている。

# 起動モジュール

```python
if __name__ == "__main__":
    print("Hello, World!")
```

`if __name__ == "__main__":` は、「このモジュールが(他のプログラムから呼び出されたのではなく)自身が実行された場合」という意味で、上記のモジュールを直接実行したい際の所謂 **メイン関数** となる。

# 組み込み系

## [組み込み関数](https://docs.python.jp/3/library/functions.html)

## [組み込み定数](https://docs.python.jp/3/library/constants.html)

## [組み込み型](https://docs.python.jp/3/library/stdtypes.html)

- 真偽値型( `bool` )
- 数値型
    - 整数( `int` )
        - 真偽値型は整数型のサブタイプ
    - 浮動小数点数( `float` )
    - 複素数( `complex` )
- シーケンス型
    - list, tuple, range
- テキストシーケンス型
    - シングルクォート: '"ダブル" クォートを埋め込むことができます'
    - ダブルクォート: "'シングル' クォートを埋め込むことができます"
    - 三重引用符: '''三つのシングルクォート''', """三つのダブルクォート"""
        - 改行を含めることができる
- バイナリシーケンス型
    - bytes, bytearray, memoryview
- set（集合）型
    - set, frozenset
    - 集合演算が可能
        - 和集合（union）、差集合（difference）、積集合（intersection）など
- マッピング型
    - dict
- イテレータ型
- ジェネレータ型
- コンテキストマネージャ型

## [組み込み例外](https://docs.python.jp/3/library/exceptions.html)

## シーケンスの扱い

### スライス

シーケンスの要素をコントロールする処理。

```python
シーケンス[開始インデックス:終了インデックス]
シーケンス[開始インデックス:終了インデックス:間隔]
```

```
>>> a = [1, 2, 3, 4, 5]
>>> a[1:4] # インデックス1から4未満まで取得
[2, 3, 4]
>>> a[::2] # 全てのインデックスを2間隔で取得
[1, 3, 5]
>>> a[::-1] # 全てのインデックスを-1間隔（つまり逆に）取得
[5, 4, 3, 2, 1]
```

### zip()

各シーケンスの要素を先頭から 1 つずつ取得し、タプルとして返却する。

```
zip(シーケンス1, シーケンス2,...)
```

```
>>> b1 = [1, 3, 5]
>>> b2 = [2, 4, 6]
>>> [x for x in zip(b1, b2)]
[(1, 2), (3, 4), (5, 6)]
```

### dict()

タプルのリストをディクショナリへ変換する。

```
>>> b1 = [1, 3, 5]
>>> b2 = [2, 4, 6]
>>> zip(b1, b2)
<zip object at 0x1035edd08>
>>> [x for x in zip(b1, b2)]
[(1, 2), (3, 4), (5, 6)]
>>> dict([x for x in zip(b1, b2)])
{1: 2, 3: 4, 5: 6}
```

つまり、 `zip()` と `dict()` を組み合わせれば、 2 つのリストからディクショナリを作成できる。

## イテレータとジェネレータ

- https://qiita.com/tomotaka_ito/items/35f3eb108f587022fa09

# 制御文

## 三項演算子

```
条件がTrueのときの値 if 条件 else 条件がFalseのときの値
```

# 関数

## アンパック

関数の引数にリストやディクショナリの要素を渡したい場合、 `*` `**` で **アンパック** して渡すことができる。

## 関数アノテーション（ Function Annotations ）

関数の引数と返り値の **型アノテーション** 。  
「 def 関数名(引数1<font color="red">: 型</font>, 引数2<font color="red">: 型</font>,...) <font color="red">-> 返り値の型</font>: 」。  
型アノテーションを理解する IDE などで警告がでるようになる。

# 内包表記（comprehension）

Pythonには **内包表記** という文法がある。  
これを使うと、リストやディクショナリなどの加工をするような処理をブロックを使用せずに簡潔に記述できる。  
内包表記には **リスト内包表記** 、 **ディクショナリ内包表記** 、 **set内包表記** がある。

## リスト内包表記（list comprehension）

基本構文

```
[ リストの要素 for シーケンスの要素 in シーケンス ]
```

例えば、 `[x**2 for x in range(10)]` は

- `range(10)`
    - 0から9までの整数のシーケンスを
- `x in range(10)`
    - 小さい順から取り出して
- `x**2 for x in range(10)`
    - 取り出した要素を二乗
- `[x**2 for x in range(10)]`
    - してできるリスト

となる。  
また、if文を使用してシーケンスの要素を絞ることができる。

```
[ リストの要素 for シーケンスの要素 in シーケンス if シーケンスの要素を絞る条件 ]
```

例えば、 `[x for x in data if data.count(x) > 1]` は

- `x in data`
    - dataリストの要素で
- `x in data if data.count(x) > 1`
    - dataリスト内で1つより多い要素
- `[x for x in data if data.count(x) > 1]`
    - のリスト

## ジェネレータ式

リスト内包表記の `[]` を `()` とするとジェネレータを返却する。

```
>>> xx = (x + 2 for x in range(5))
>>> xx
<generator object <genexpr> at 0x1035e2570>
>>> list(xx)
[2, 3, 4, 5, 6]
```

## セット内包表記

リスト内包表記の `[]` を `{}` とするとセットを返却する。

```
>>> {x + 2 for x in range(5)}
{2, 3, 4, 5, 6}
```

## 辞書内包表記

 セット内包表記の要素を `k: v` の形式にするとディクショナリを返却する。

```
>>> {x: x*x for x in range(5)}
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

# lambda式

基本構文。 `return` は不要。

```
lambda 引数のリスト : 引数を使った式
```

# 文字列の扱い

# Unicode 文字列

先頭に `u` をつけた文字列（ `u"文字列"` ）。  
文字列をユニコード（ UTF-8 ）で扱ってくれる。

## f 文字列

式を埋め込んだ文字列で、フォーマット済み文字列（ formatted string ）と呼ばれる。  
先頭に `f` をつけ、文字列内の `{}` で囲まれた部分が式として評価される。

```
>>> f'100+1={100+1}'  # 式を評価
'100+1=101'

>>> order={'spam':100, 'ham':200}
>>> f'spam: {order["spam"]}, ham: {order["ham"]}' # 変数を参照
'spam: 100, ham: 200'

>>> f'Hello, {input("名前を入力してください: ")}' # 関数の呼び出し
名前を入力してください: ぺーぺーSE
'Hello, ぺーぺーSE'
```

## raw 文字列

エスケープシーケンスを無効にしたい場合の書き方。  
例えば、 `\n` を出力したい場合 `\\n` とエスケープしなければならないが、 raw 文字列を使用すると、 `\n` のみでよい。  
先頭に `r` をつけると raw 文字列になる。

```
>>> print("Hello\\nPython")
Hello\nPython
>>> print("Hello\nPython")
Hello
Python
>>> print(r"Hello\nPython")
Hello\nPython
```

## 文字列置換

文字列置換には以下の方法がある。

- % 演算子
- `format()` 関数

### % 演算子

```
# 基本形
>>> print 'Hello, %s!' % 'World'
Hello, World!
# 複数ある場合はタプルで渡す
>>> print 'Hi, %s. You are %d years old' % ('John', 25)
Hi, John. You are 25 years old
```

|変換|意味|
|:---|:---|
|%d|符号つき 10 進整数|
|%e, %E|指数表記の浮動小数点数|
|%r|文字列（Python オブジェクトを repr() で変換したもの）|
|%s|文字列（Python オブジェクトを str() で変換したもの）|

### `format()` 関数

```
# 基本形
>>> print 'Hello, {}!'.format('World')
Hello, World!

# ポジション引数(インデックス)を使う
>>> print '{0}, {2}, {1}'.format('a', 'b', 'c')
a, c, b

>>> print '{0}, {1}, {0}'.format('a', 'b')
a, b, a

# キーワード引数を使う
>>> print '{language} has {number} quote types.'.format(language='Python', number=2)
Python has 2 quote types.

# タプル、リストでも
>>> coord = (3, 5)
>>> 'X: {0[0]};  Y: {0[1]}'.format(coord)
'X: 3;  Y: 5'
```

引数にdict型を使うことがでるが `**` でアンパックして渡す必要がある。

```
>>> coord = {'latitude': '37.24N', 'longitude': '-115.81W'}
>>> 'Coordinates: {latitude}, {longitude}'.format(**coord)
'Coordinates: 37.24N, -115.81W'
```

クラス属性へアクセスする。

```
>>> class Coord(object):
...     def __init__(self, lat, lon):
...         self.lat, self.lon = lat, lon
...
>>> coord = Coord('37.24N', '-115.81W')
>>> 'Coordinates: {0.lat}, {0.lon}'.format(coord)
'Coordinates: 37.24N, -115.81W'
```

## 文字列と真偽値の演算

任意の文字列と `False` との積は空文字になる。

```
>>> "hoge" * True
'hoge'
>>> "hoge" * False
''
```

## 文字コード変換（ `chr()` 、 `ord()` ）

Unicode コードポイントが整数 i である文字を表す文字列を返す。  
例えば chr(97) は文字列 'a' を、 chr(8364) は文字列 '€' を返す。  
`ord()` は `chr()` の逆。  
Python2 では `chr()` が ASCII で、 `unichr()` が Unicode だったが、 Python3 から `chr()` へ統合された？


# `locals()` と `globals()`

`locals()` は自身のローカル領域の変数の値を全て辞書形式で返してくれる。

```
>>> def hoge(str, num):
...     print(locals())
...
>>> hoge("moji", 123)
{'num': 123, 'str': 'moji'}
```

グローバル領域バージョンが `globals()` 。

```
>>> y = 30
>>> globals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__':
 None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'y': 30}
 ```

# `__xxx__` 系

## `__doc__`

`__doc__` を使うと、各クラスやメソッドの概要を閲覧出来る。  
概要の設定方法は、クラスやメソッドの直下にダブルクオート 3 つで囲んだ文字列を記載するだけ。

```
>>> class TestClass:
...     """This is TestClass.
...     """
...     def testMethod(self):
...         """This is testMethod
...         """
...         print( "hello" )
...         # comment2
...         """comment3
...         """
...
>>> print( TestClass.__doc__ )
This is TestClass.

>>> print( TestClass.testMethod.__doc__ )
This is testMethod
```

## `__call__`

`__init__` と同様にクラスに定義する関数。  
インスタンス生成では `__init__` しか呼び出されない。  
しかし、一度生成されたインスタンスを関数っぽく引数を与えて呼び出せば、 `__call__` が呼び出されるという仕組み。
もちろん、 `__call__` に返り値をつければ,インスタンスから得られた値を別の変数に使ったりもできるということ。

```
>>> class cls:
...     __call__ = "Hi. My name is {} and I'm {} years old".format
...
>>> say_hi = cls()
>>> say_hi("Alex", 32)
"Hi. My name is Alex and I'm 32 years old"
```

# 標準ライブラリ

## Template()

テンプレートでは、 `$` と変数名からなるプレースホルダ名を使う。  
プレースホルダの周りを `{}` で囲えば、プレースホルダの後ろにスペースを挟まず、英数文字を続けることができる。  
`$$` のようにすると、 `$` 自体をエスケープでる。

```
>>> from string import Template
>>> t = Template('${village}folk send $$10 to $cause.')
>>> t.substitute(village='Nottingham', cause='the ditch fund')
'Nottinghamfolk send $10 to the ditch fund.'
```

その他の標準ライブラリ

- https://docs.python.jp/3/tutorial/stdlib2.html

# ファイル操作

## 読み込み

一行ずつ読み込みは以下。

```python
f = open("input.txt", "r")
for row in f:
    print(row, end="")
f.close()
```

`close()` 忘れ防止のために `with` を使う。

```python
with open("input.txt", "r") as f:
    for row in f:
        print(row, end="")
```

全読み込みは `read()` を使う。

```python
with open("input.txt", "r") as f:
    print(f.read())
```

## 書き込み（上書き）

```python
with open("test.txt", "w") as f:
    #f.write("hoge")
    print("hoge", file=f)
```

## 追記

```python
with open("test.txt", "a") as f:
    print("fuge", file=f)
```

# その他

- [Pythonのvars()とdir()の違い](http://minus9d.hatenablog.com/entry/2016/11/13/222629)
