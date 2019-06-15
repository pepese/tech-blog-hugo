---
title: React入門
tags:
id: react-basics
draft: true
---

Facebookが開発、OSSとして公開しているJavaScriptライブラリ。  
フレームワークではなく、UIを構築するためのライブラリ。

# はじめに

## 用語とか

- Virtual DOM
    - メモリ上に保存された、DOM のコピー
    - イベントによりデータ更新が発生し、 DOM を再描画する必要が出た場合、画面の DOM ではなく Virtual DOM を更新
- コンポネントベース
    - コンポーネントごとに状態を管理、複雑なUIを一つの部品としてまとめる
- サーバサイドレンダリング
    - クライアントサイドの描画だけでなく、Nodeを使ったサーバーサイドレンダリングに対応
- JSX
    - 一見 DOM が混じったような JS の記法
    - HTML のように見えるが、 XML
    - 通常の XML とは以下が異なる
        - 属性をダブルクオートではなく中括弧で囲む
        - DOM の最上位要素は 1 つ
- Flux
    - クライアントサイドのWebアプリケーション（ユーザーインターフェース）を開発するためのアプリケーションアーキテクチャ
    - Action 、 Dispatcher 、 Store 、 View の機能がある
- React Router ( `react-router-dom` )
    - ページ遷移を司る
    - ページ全体もページの一部の変更も可能

## React コンポーネントのライフサイクル

大きく分けて 3 フェーズある。

- Mounting フェーズ
    - コンポーネントが生成されて DOM に要素が挿入される時
- Updating フェーズ
    - PropsやStateの値に変更が発生した時
- Unmountingフェーズ
    - コンポーネントが DOM から取り除かれる時

`componentWillMount` メソッドは初期化時と属性変更時に、 `render` メソッドは描画時に呼ばれる。

[React TutorialでComponentを別ファイルに分けてやってみた](https://qiita.com/taigamikami/items/4ea2813df24e12f14662)

# 環境設定

## create-react-app の導入

```sh
$ brew install yarn
$ yarn global add create-react-app
```

## プロジェクトの作成

```sh
$ create-react-app react-sample
Inside that directory, you can run several commands:

  yarn start
    Starts the development server.

  yarn build
    Bundles the app into static files for production.

  yarn test
    Starts the test runner.
  yarn eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd react-sample
  yarn start

Happy hacking!
$ cd react-sample
$ yarn start
You can now view react-sample in the browser.

  Local:            http://localhost:3000/
  On Your Network:  http://192.168.150.33:3000/

Note that the development build is not optimized.
To create a production build, use yarn build.
```

## SASS 環境

[ここ](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#adding-a-css-preprocessor-sass-less-etc)。  
[その他テーマ](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md)。


# メモ

- [Reactの概要と基礎技術要素を理解する](https://codezine.jp/article/detail/9878)
- [イマドキWebフロントエンド環境とReactを触りながらサンプルを10本書いてみた](https://qiita.com/akimach/items/af3ba7841bcb789b75a5)
- [ReactJSで作る今時のSPA入門（基本編）](https://qiita.com/teradonburi/items/fb91e5feacab5071cfef)
