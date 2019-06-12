---
title: "Atomのインストールとパッケージの導入"
date: 2017-05-11T07:45:58+09:00
slug: "atom-install-packages"
tags:
- atom
draft: false
archives:
- 2017/05
---

Githubが開発したエディタ[Atom](https://atom.io/)のインストールからパッケージの導入までをまとめる。

<!--more-->

# インストール（Mac）

- [公式](https://atom.io)へアクセス
- 「Download For Mac」をクリック
- ダウンロードした ファイルがzipであれば解凍する
- Macアプリ「Atom」が表示されるので、それを「アプリケーション」フォルダへドラッグ＆ドロップ
- 「アプリケーション」フォルダへ移動し、「Atom」を起動

brew caskが使える人は以下でもよい。

```sh
$ brew cask install atom
```

上記がなんのことかわからない人は以下参照。

- https://pepese.github.io/blog/homebrew-basics/

# 起動方法

アイコンクリックでもいいがコマンドラインで起動できる。

```sh
$ atom file
```

プロジェクト（カレントディレクトリ）で起動したい場合は以下。

```sh
$ atom .
```

# パッケージ導入方法

- メニュータブの「Packages」->「Settings View」を選ぶと「Settings」タブが開く
    - 「Ctrl+,」（Win）、「Command+,」（Mac）でも開く
- 「Install」メニューを選択し、欲しいパッケージを検索して「Install」ボタンを押すS
- 設定が反映されない場合はAtomを再起動する

# アップデート

## Atom

メニューの「 Atom 」 -> 「 Restart and Install Update 」。

## パッケージ

メニューの「パッケージ」 -> 「 Settings View 」 -> 「 Updates 」。

## トラブルシューティング

アップデートできない人は以下を実行。

```sh
$ sudo chown -R $(whoami) /Applications/Atom.app/
```

# オススメのパッケージ

- japanese-menu
    - メニューバーとコンテキストメニューを日本語化
- japanese-wrap
    - 日本語を折り返してくれるパッケージ
    - 現在のAtomは日本語の折り返しがうまく機能しないため
- goto-defitinion
    - 「Ctrl+Alt+Enter」（Win）、「Option+Cmd+Enter」（Mac）でクラス、メソッドの実装へ飛べるようになる
- terminal-plus（ **platformio-ide-terminal** ）
    - ターミナル画面を追加できるパッケージ
        - 画面を切り替えてターミナル開くのが面倒になった人にオススメ
    - 下部の「+」をクリックするか、 `Ctrl + Shift + @` でターミナルが開く
    - しかしながら、2017/01/19時点では動かなかったため、terminal-plusをforkして作成された **platformio-ide-terminal** がいい
- markdown-preview-enhanced
    - マークダウン表示機能の拡張
    - Pandoc を導入（ `brew install pandoc` ）することで、 Tex 形式のマークダウンも表示可能になる

# ATOM IDE

## インストール

パッケージの導入から **atom-ide-ui** を導入する。  
その後、利用したい開発言語に合わせて以下のものを導入する。

- ide-typescript
- ide-flowtype
- ide-csharp
- ide-java (Java 8 runtime required)
- ide-php

[ここ](https://atom.io/packages/search?q=IDE)で言語対応のパッケージを検索できる。
