---
title: "Git入門"
date: 2017-05-9T07:55:01+09:00
slug: "git-basics"
tags:
- tag_name
draft: false
archives:
- 2017/05
---

Git/Githubのメモ。  
インストールから簡単な使い方まで書いてみる。

- 環境構築
- 一通りの使い方
- その他 Tips
    - git submodule

<!--more-->

# 環境構築

## GitHubのアカウントを作成

下記でGitHubのアカウントを作成。

- https://github.com/
- ちなみに自分のアカウント
    - https://github.com/pepese
- リポジトリを作る
    - https://github.com/pepese/Sample

## Gitクライアント環境の作成

### Windowsへのインストール

- GitクライアントアプリをWindowsPCへインストール
    - http://www.git-scm.com/downloads

### Macへのインストール

以下を参照。

- [Macで開発環境を作る](https://blog.pepese.com/entry/mac-dev-environment/)

### Gitクライアント設定

- ユーザ名とメールアドレスを登録（ **コミットログに残るユーザ名なので必ず設定する！** ）
    - `git config --global user.name "hoge"`
    - `git config --global user.email "hoge@email.sample.com"`
- プロキシがある環境の場合は下記を叩く
    - `git config --global http.proxy http://[FQDN]:[PORT]`
    - `git config --global https.proxy http://proxy.example.com:8080`
- プロキシがある環境の場合は下記を叩く（ID、Passwordが必要な場合）
    - `git config --global http.proxy http://[ID]:[Password]@[FQDN]:[PORT]`
    - `git config --global https.proxy http://[ID]:[Password]@[FQDN]:[PORT]`
- プロキシがある環境の場合は下記を叩く
    - `git config --global url."https://".insteadOf git://`
    - これでgitスキームを使えるようになる
- Windowsのgitではデフォルトで改行コードがCR+LFへ変換されてしまうのでオフにする
    - `git config --global core.autocrlf false`
- 設定が反映されていることを確認 `git config --global -l`

グローバル設定ではなく、単一Gitプロジェクトにのみ反映させたい場合は、 `--global` を外してコマンドを打つ。

```
core.symlinks=false
core.autocrlf=true
color.diff=auto
color.status=auto
color.branch=auto
color.interactive=true
pack.packsizelimit=2g
help.format=html
http.sslcainfo=/bin/curl-ca-bundle.crt
sendemail.smtpserver=/bin/msmtp.exe
diff.astextplain.textconv=astextplain
rebase.autosquash=true
http.proxy=http://[ID]:[Password]@proxy.example.com:8080
https.proxy=http://[ID]:[Password]@proxy.example.com:8080
url.https://.insteadof=git://
```

# 一通りの使い方

## リモートリポジトリを作って、ローカルリポジトリにコピーして編集して更新するパターン

1. リモートリポジトリをローカルリポジトリへコピー（clone）
    - GitHub用のローカルリポジトリ領域を作成してカレントを移動
        - `cd /d C:\Git\GitHub`
    - リモートリポジトリ（GitHub）をローカルリポジトリへコピー（初回のみ）
        - `git clone https://github.com/pepese/Sample.git`
        - `C:\Git\GitHub\Sample」と「C:\Git\GitHub\Sample\.git` ができる
1. ローカルリポジトリへファイルを登録（add/commit）
    - コピーしたローカルリポジトリへ移動
        - `cd /d C:\Git\GitHub\Sample`
    - ファイルを作成
        - `touch sample.txt`
            - `C:\Git\GitHub\Sample\sample.txt`
        - このままだとファイルを作っただけでまだローカルリポジトリへは反映されていない
    - ステージング領域へ反映
        - `git add sample.txt`
        - 全ての変更ファイルをaddしたい場合は `git add --all`
    - ローカルリポジトリへコミット
        - `git commit`
        - Vimが起動してコミットコメントを求められるので適当に編集
        - コメント付きでコミットする場合は下記
            - `git commit -m "コミットコメント"`
1. ローカルリポジトリの内容をリモートリポジトリへ反映（push）
    - ローカルリポジトリの内容をリモートリポジトリ（GitHubのMasterブランチ）へ反映
        - `git push origin master`
    - 確認
        - 下記を見ると「sample.txt」が追加されているのがわかる
            - https://github.com/pepese/Sample
        - 「sample.txt」のリンクを押すと下記のURIへ飛び、中身が確認できる（今回は空）
            - https://github.com/pepese/Sample/blob/master/sample.txt
1. ファイルを更新する
    - 下記のファイルをエディタで更新
        - `C:\Git\GitHub\Sample\sample.txt`
    - ステージング領域へ反映
        - `git add sample.txt`
    - ローカルリポジトリへコミット
        - `git commit`
    - ローカルリポジトリの内容をリモートリポジトリ（GitHub）へ反映
        - `git push origin master`
        - GitHubアカウントのIDとPasswordを求められるので入力
    - 確認
        - 下記を見ると「sample.txt」があることがわかる
            - https://github.com/pepese/Sample
        - 「sample.txt」のリンクを押すと下記のURIへ飛び、中身が確認できる
            - https://github.com/pepese/Sample/blob/master/sample.txt

## ローカルリポジトリを作ってリモートリポジトリへ反映するパターン

リモートリポジトリが存在している前提で。

- `cd /d C:\Git\GitHub`
- `mkdir Sample2`
- `cd Sample2`
- `git init`
- `git remote add origin https://github.com/pepese/Sample2.git`
    - リポジトリの参照先（origin）の設定
    - もちろんGithub上にあらかじめリポジトリを作成しておく必要がある
- `touch sample2.txt`
- `git add sample2.txt`
- `git commit`
- `git push origin master`

## ブランチの作成からPull Request完了まで

```sh
$ git branch <branchname>
```

`<branchname>` を指定せずに実行すると現在のブランチ（\*がついてる）が確認できる。  
また、ブランチの切り替えは以下。

```sh
$ git checkout <branchname>
```

一連の流れは以下。

```sh
$ git branch issue1
$ git branch
  issues1
* master
$ git checkout issue1
$ git branch
* issues1
  master
```

Pull Requestマージ後の作業。

```sh
$ git checkout master
$ git branch -d issue1      #ローカルブランチの削除
$ git push origin :issue1   #リモートブランチの削除
# もしくは git push --delete origin issue1
```

## ファイルの変更を戻す

以下でコミットログのハッシュ値を取得。

```bash
git log [ファイルパス]
```

以下でコミットのハッシュ値を指定して戻す。

```sh
$ git checkout [コミット番号] [ファイルパス]
```

## 特定のコミットをマージする

以下でリモートリポジトリの変更をローカルリポジトリへ持ってくる。（ワーキングディレクトリの変更は行われない）

```sh
$ git fetch
```

コミットログのハッシュ値を指定してマージする。

```sh
$ git cherry-pick [ハッシュ値]
```

## ブランチをマージする

issue1ブランチにmasterをマージしたい場合。

```sh
$ git checkout issue1
$ git merge master
```

## ファイル、ディレクトリをgitの管理対象から外す

### ファイルをgitの管理対象から外して且削除

```sh
$ git rm [削除したいファイル]
```

### ファイルをgitの管理対象から外すがファイルを削除しない

```sh
$ git rm —-cached [削除したいファイル]
```

### ディレクトリをgitの管理対象から外して且削除

```sh
$ git rm -r [削除したいディレクトリ]
```

もうちょっと知りたい人は以下を参照。

- [Gitコマンド整理](https://blog.pepese.com/entry/git-commands/)

# その他 Tips

## git submodule

`git submodule` は、外部の git リポジトリを、自分の git リポジトリのサブディレクトリとして登録し、特定の commit を参照する仕組み。（ [参考](https://qiita.com/sotarok/items/0d525e568a6088f6f6bb) ）