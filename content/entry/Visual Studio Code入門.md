---
title: "Visual Studio Code入門"
date: 2018-01-31T18:09:30+09:00
slug: "vscode-basics"
tags:
- vscode
draft: false
archives:
- 2018/01
---

Microsoft が開発したエディタ[Visual Studio Code](https://code.visualstudio.com/)（以降、 VS Code ）のインストールから拡張機能の導入までをまとめる。

<!--more-->

# インストール（Mac）

```sh
$ brew cask install visual-studio-code
```

# 起動方法

アイコンクリックでもいいがコマンドラインで起動できる。

```sh
$ code
```

プロジェクト（カレントディレクトリ）で起動したい場合は以下。

```sh
$ code .
```

# ショートカット・操作

ショートカットの一覧・設定は `Comannd + K -> Command + S` で開く。  
（筆者の場合は「右側をすべて削除」が `Ctrl + K` になっていなかったので登録した。）

- ユーザ設定
    - `Command + ,`
- Markdown Preview
    - `Command + Shift + V`
- 統合ターミナル
    - `Control + Shift + @`
- ワークスペースにプロジェクトを追加
    - 「ファイル」->「ワークスペースにフォルダに追加」

# コマンドパレット

`Command + Shift + P` を押すと VS Code の上部に **コマンドパレット** が開く。  
VS Code で実行できる各種コマンドには名前が付いていて、その名前をこのコマンドパレットに入力することでそれらを実行できる。

# 拡張機能（ Extensions ）

アクティビティーバー（左上に縦に並んでるアイコン）の一番下にあるアイコンをクリックするとサイドバーが拡張機能の画面になる。（ `Command + Shift + X` でも）  
以下のような種類の拡張機能が存在する。

- ［Debuggers］（デバッガー）
- ［Languages］（言語）
- ［Linters］（Lintツール）
- ［Snippets］（スニペット）
- ［Themes］（テーマ）
- ［Other］（その他）

検索欄を空にすると現在インストール済みの拡張機能の一覧が表示される。  
なお、拡張機能の反映には VS Code を再起動するか再読み込みボタンを押す。

# アップデート

## VS Code

「 Code 」 -> 「更新の確認」。

## 拡張機能

拡張機能画面の右上の「・・・」をクリックして「更新の確認」。

# オススメの拡張機能

## EditorConfig

以下のコマンドを実行して VSCode の [EditorConfig](https://editorconfig.org/) 拡張機能を有効にする。

```bash
$ code --install-extension EditorConfig.EditorConfig
```

その後 `.editorconfig` をプロジェクトルートへ作成する。（設定内容はお好みで）

```
root = true

[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8

[*.{js,json}]
indent_style = space
indent_size = 2
```

## 視覚サポート

- Trailing Spaces
    - 改行部分の最後に入る半角スペースの強調、削除
- EvilInspector
    - 文章中の全角スペースを強調, 削除

## Markdown

- Auto-Open Markdown Preview
    - Markdown ファイルを開くときに自動でプレビューを開く
- Markdown+Math
    - `$$` 内に数式を書いて `Ctrl + Shift + .` と打てば、数式対応のプレビューが表示

## Git

- [Git Lens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)
    - 誰が・いつ編集したかをわかりやすく表示
- Git History
    - ツリー表示や差分表示など
- gi
    - [gitignore.io](https://github.com/joeblau/gitignore.io) から gitignore を追加
- gitignore
    - [github/gitignore.io](https://github.com/github/gitignore) から gitignore を追加

## マークアップ

- Auto Complete Tag
    - タグを自動で閉じて、開始・終了タグを変更したらもう片方のタグも自動で変更

## Javascript

- [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)
    - importしたファイルの容量を表示してくれる拡張機能
- [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)
    - ファイルパスの自動入力
- [Quokka.js](https://marketplace.visualstudio.com/items?itemName=WallabyJs.quokka-vscode)
    - エラーになる箇所を実行前に評価してくれる拡張機能

## その他

- Shortcuts
    - VS Code 最下部にショートカットボタンを追加
- [Material Icon Theme](https://github.com/PKief/vscode-material-icon-theme)
    - ファイルアイコンをマテリアルデザインになる
- [Indent Rainbow](https://github.com/oderwat/vscode-indent-rainbow)
    - インデントレベルごとに背景色が付く（超見やすいw）
- [Bracket Pair Colorizer](https://github.com/CoenraadS/BracketPair)
    - 対応するカッコに縦横線のハイライトを表示（超見やすいw）
- [Base16 Themes](https://marketplace.visualstudio.com/items?itemName=AndrsDC.base16-themes)
    - 気に入った Color Theme
    - Dark Monokai がよい
- [Trailing Spaces](https://marketplace.visualstudio.com/items?itemName=shardulm94.trailing-spaces)
    - 全角や余計なスペースをハイライトしてくれる拡張機能