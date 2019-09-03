---
title: "Flutter入門"
date: 2019-09-02T17:49:53+09:00
slug: "flutter-basics"
tags:
- flutter
- dart
draft: true
archives:
- 2019/09
---

[Flutter](https://github.com/flutter/flutter) に入門する。

<!--more-->

# 環境構築

## Flutter

公式にあるバイナリ取得ではなくて Github から取得して設定してみる。

```bash
$ git clone https://github.com/flutter/flutter.git ~/flutter
$ cd ~/flutter
$ git pull --tags
$ git tag
...
v1.9.7
$ git checkout refs/tags/v1.9.7
$ echo 'export PATH=$HOME/flutter/bin:$PATH' >> ~/.bash_profile
$ exec $SHELL -l
$ which flutter
$ flutter precache
$ flutter doctor
```

最後の `flutter doctor` の出力結果を基に iOS/Android の環境を作成する。

## プロジェクトの作成

```bash
$ flutter create flutter_sample
$ cd flutter_sample
$ open -a Simulator # iOS シミュレータ
$ flutter run
...
To hot reload changes while running, press "r". To hot restart (and rebuild state), press "R".
...
For a more detailed help message, press "h". To detach, press "d"; to quit, press "q".
```

上記の通り `flutter run` したターミナル上で `r` とかタイプするとリロードしたりする。

```bash
$ tree . -L 1
.
├── .idea        # IntelliJ 関連ファイル
├── .metadata    # Flutter プロジェクトのプロパティ
├── .packages    # アプリケーションで使用される依存関係のリスト
├── README.md
├── android      # Android 用
├── build        # ビルドの出力先？
├── ios          # iOS 用
├── lib          # 開発コード
├── pubspec.lock
├── pubspec.yaml # Pub Package Manager のパッケージ管理ファイル
├── test         # テストコード
└── user_app.iml # Android Studio 関連ファイル
```

# あれこれ

## StatelessWidget

Flutter の画面はテキストであろうがボタンであろうが全画面であろうが全て Widget で実装する。  
**StatelessWidget** は状態（ state ）を持たない Widget で、 親 Widget から渡された値をそのまま表示するだけ。  
以下の特徴がある。

- `StatelessWidget` を `extends` した `class` を作成
- `build()` メソッドを `@override` して Widget を `return` することで描画
- 子 Widget である StatelessWidget で使用した変数を親 Widget にて変更した場合、子であるStatelessWidgetは全て再描画（build）される

## StatefulWidget

状態（ state ）を持つ Widget 。  
自分の変数を更新することで自分自身を再ビルドすることができる。

- `StatefulWidget` を `extends` した `class` を作成（クラス①）
- `State<上記のclass>` を `extends` した `class` を作成（クラス②）
- クラス①の `createState()` メソッドを `@override` してクラス②をリターン
- クラス②の `build()` メソッドを `@override` して Widget を `return` することで描画

state のライフサイクルは以下の 10 ステップ。

1. createState
    - state を作成する
    - `StatefulWidget` のメソッドで `@override` して利用
2. mounted is true
    - state をマウントする
    - 作成された stateは `BuildContext` に紐づけられる
3. initState
    - Stateを初期化する
    - 最初に 1 度だけ実行される
4. didChangeDependencies
    - `InheritedWidget` の `BuildContext.inheritFromWidgetOfExactType` が実行された場合に呼び出される
    - `InheritedWidget` とは、Tree の下位の Widget から直接参照できる Widget のこと
5. build
6. didUpdateWidget
    - 親Widgetが変更されて、再ビルドされる必要がある場合に呼ばれる
7. setState
    - State 内の変数の値を変更したい場合に呼び出される
    - これが呼ばれた後、 `build()` メソッドが再実行される
8. deactivate
    - State がツリーから削除される場合に呼び出される
9. dispose
    - State オブジェクトが削除されると呼び出される
10. mounted is false
    - disposeが呼び出されると、Stateオブジェクトは再マウントされない
    - ここで `setState()` メソッドを呼び出した場合、エラーとなる

# 参考

- [Flutter 入門](https://flutter.keicode.com/)
  - [各種 Material ウィジェットの使い方](https://flutter.keicode.com/basics/index-material.php)
- [Flutter入門 - 簡単なアプリを作ってUI宣言やホットリロードなど便利機能の使い方を理解しよう](https://employment.en-japan.com/engineerhub/entry/2019/08/06/103000)
- [全てがここで分かる！！Flutter 超入門](https://tech-rise.net/category/programming/flutter/)
- [Flutterプロジェクトのディレクトリ構成とレイヤー案](https://issus.me/projects/1520/issues/35)
- [Flutterのコードはどのように分割したらわかりやすいだろう？](https://note.mu/nbht/n/n339076d40641)