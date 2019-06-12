---
title: "Jupyter Notebook入門"
date: 2018-02-20T08:03:43+09:00
slug: "jupyter-notebook-basics"
tags:
- jupyter
- python
draft: false
archives:
- 2018/02
---

Jupyter Notebook 、ついでに JupyterHub について軽くまとめる。

- [公式](http://jupyter.org/)
- [公式ドキュメント](http://jupyter.org/documentation)

<!--more-->

# Jupyter Notebook

## インストール

```bash
$ pip install jupyter
```

## 起動

```bash
$ jupyter notebook
```

### 起動オプション

|オプション|説明|
|:---|:---|
| `--no-browser` |自動でブラウザを起動しない|
| `--port xxxx` |ポートの指定|
| `--help` |ヘルプ|

## ショートカットまとめ

自分がよく使うショートカットのみまとめる。

### コマンドモード（ `ESC` 押したとき）

|コマンド|説明|
|:---|:---|
| `Enter` |編集モード|
| `H` |ショートカット一覧|
| `DD` |セルの削除|
| `K` |上のセルへ移動|
| `J` |下のセルへ移動|
| `A` |セルを上に追加|
| `B` |セルを下に追加|
| `Y` |コードモード|
| `M` |マークダウンモード|
| `1` 〜 `6` |見出し１〜見出し６で書き出す|
| `00` |カーネルをリスタート|
| `S` or `Command + S` |保存|

### 編集モード（ `Enter` 押したとき）

|コマンド|説明|
|:---|:---|
| `ESC` or `Ctrl + M` |コマンドモード|
| `Ctrl + Enter` |セルの実行|
| `Command + S` |保存|

## その他小技

|コマンド|説明|
|:---|:---|
|実行結果のセルをダブルクリック|実行結果の最小化（閉じる）|

# JupyterHub

JupyterHub を導入することで以下が可能になる。

- マルチユーザ
- ユーザ管理・認証
- 高スペックサーバでリモートから利用
- などなど

<img src="https://jupyterhub.readthedocs.io/en/stable/_images/jhub-parts.png" />

## インストール

```bash
$ pip install jupyterhub notebook
$ npm install -g configurable-http-proxy
# test
$ jupyterhub -h
$ configurable-http-proxy -h
```

なお、 Docker 版も提供されている。

## 起動

```bash
$ jupyterhub
```

`https://localhost:8000` で起動する。