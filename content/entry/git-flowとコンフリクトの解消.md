---
title: "Git Flowとコンフリクトの解消"
date: 2017-05-24T07:51:52+09:00
slug: "git-flow-fix-conflict"
tags:
- git
- github
draft: false
archives:
- 2017/05
---

git-flowの概要とマージ時のコンフリクトの解消についてまとめる。  
gitについては以下を参照。

- [Git入門](https://blog.pepese.com/entry/git-basics/)
- [Gitコマンド整理](https://blog.pepese.com/entry/git-commands/)

以下の順で記載する。

- git-flow
- コンフリクトの解消

<!--more-->

# git-flow

出典：[A successful Git branching model » nvie.com](http://nvie.com/posts/a-successful-git-branching-model/)

<img src="http://nvie.com/img/git-model@2x.png">

**git-flow** は代表的なgitのbranch運用のベストプラクティスで、以下の5つのブランチでプロダクトのソースコードを運用する。

- メインブランチ
    - master ブランチ
    - develop ブランチ
- サポートブランチ
    - feature ブランチ
    - release ブランチ
    - hotfix ブランチ

メインブランチは常に存在しており、開発された全ての機能はいずれメインブランチへ取り込まれる。  
サポートブランチは名ブランチから派生し、その役割が終えるとメインブランチへマージされる。

## master ブランチ

- 分岐元：なし
- マージ先：なし
- マージ元：feature ブランチ、release ブランチ、hotfix ブランチ
- 役割
    - リリース可能なソフトウェアを常に維持するブランチ

## develop ブランチ

- 分岐元：master ブランチ
- マージ先：なし
- マージ元：feature ブランチ、release ブランチ、 hotfix ブランチ
- 役割
    - 最新の開発状態を常に反映するブランチ
    - develop ブランチから master ブランチへ直接マージすることはなく、 release ブランチを介して master ブランチへマージされる

## feature ブランチ

- 分岐元：develop ブランチ
- マージ先：develop ブランチ
- マージ元：なし
- 役割
    - 新しい機能を開発する際、機能単位で作成されるブランチ
    - 機能開発が完了したら develop ブランチへマージされ、削除される
    - 機能の特徴に応じて「feature-\*」と命名される

## release ブランチ

- 分岐元：develop ブランチ
- マージ先：master ブランチ、develop ブランチ
- マージ元：なし
- 役割
    - develop ブランチでリリースするための機能開発が完了した段階で作成される
    - 結合試験のバグfixがコミットされ、重要なバグfixは develop ブランチへの反映される
    - リリースが完了したら master ブランチと develop ブランチへマージされ、削除される
    - リリースの特徴に応じて「release-\*」と命名される

## hotfix ブランチ

- 分岐元：master ブランチ
- マージ先：master ブランチ、develop ブランチ
- マージ元：なし
- 役割
    - master ブランチで重大なバグが発生した際に作成される
    - バグfixしたら master ブランチ、 develop ブランチへマージされ、削除される
    - バグの特徴に応じて「hotfix-\*」と命名される

# コンフリクトの解消

コンフリクトが発生する状況を以下の手順で作成する。

1. master ブランチから develop ブランチを作成する
2. develop ブランチから feature ブランチを作成する
3. develop ブランチのファイルA（例えばREADME.md）に対して変更を加えてコミットする
4. feature ブランチのファイルA（例えばREADME.md）に対して変更を加えてコミットする
5. feature ブランチを develop ブランチへマージするとコンフリクトが発生するので、これに対処する

以降、下記の順で記載する。

- コンフリクト発生の下準備
- コンフリクトに対処する

## コンフリクト発生の下準備

コンフリクトを発生させるリポジトリを作成する。  
ここでは、「 https://github.com/pepese/test.git 」とする。  
master ブランチに対して以下のREADME.mdファイルを作成してpushしておく。

```sh
$ git clone https://github.com/pepese/test.git
$ cd test
$ vi README.md # 後述お通りファイルを編集する
$ git add README.md
$ git commit -m "first commit"
$ git push origin master
$ git branch develop      # developブランチの作成
$ git checkout develop
$ git push origin develop
$ git branch feature      # featureブランチの作成
$ git checkout feature
$ git push origin feature
```

```markdown
行１
行２
```

以降、2つのターミナル（ターミナル1、ターミナル2）で作業する。  
ターミナル1では develop ブランチ、ターミナル2では feature ブランチで作業する。  
ターミナル1で以下を実行する。

```sh
$ git clone https://github.com/pepese/test.git develop
$ cd develop
$ git checkout develop
$ vi README.md # 後述お通りファイルを編集する
$ git add README.md
$ git commit -m "fix line2"
$ git push origin develop
```

```markdown
行１
行２ 変更
```

ターミナル2で以下を実行する。

```sh
$ git clone https://github.com/pepese/test.git feature
$ cd feature
$ git checkout feature
$ vi README.md # 後述お通りファイルを編集する
$ git add README.md
$ git commit -m "add line3"
$ git push origin feature
```

```markdown
行１
行２
行３
```

以上で下準備は完了。

## コンフリクトに対処する

ターミナル1で、以下の順で作業する。

1. `git fetch` でリモートリポジトリの更新を取得する
2. `git merge origin/feature` で develop ブランチへ feature ブランチをマージする（コンフリクト発生！）
3. コンフリクトを解消する
4. 変更をリモートリポジトリへ反映する
5. feature ブランチを削除する

上記に沿って具体的な手順を後述する。

```sh
$ git fetch
$ git merge origin/feature
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

コンフリクトが発生した旨のメッセージが出た。  
コンフリクトが発生しているファイルなどの情報は `git status` でも確認できる。  
このときREADME.mdは以下のようになっている。

```markdown
行１
<<<<<<< HEAD
行２ 変更
=======
行２
行３
>>>>>>> origin/feature
```

「 **<<<<<<<** 」から「 **>>>>>>>** 」までがコンフリクトが発生している部分。  
「 **=======** 」から上が develop ブランチで加えた変更、「 **=======** 」から下が feature ブランチで加えた変更。  
このようにコンフリクトが発生した箇所の目印をGitでは **コンフリクトマーカー** という。  
Gitではコンフリクトを **手動で修正** する必要がある。  
修正した結果は以下。

```markdown
行１
行２ 変更
行３
```

修正が完了したら以下のように反映させる。

```sh
$ git add README.md
$ git commit -m "merge and fix conflict"
$ git push origin develop
```

`git log` で確認すると以下のように develop ブランチから分岐した feature ブランチがマージされていることがわかる。  
（時系列は下から上）

```sh
$ git log --graph --oneline
*   9480427 merge and fix conflict
|\  
| * 3b11c03 add line3
* | 6682155 fix line2
|/  
* 2363f83 first commit

# ブランチ名も表示すると以下

$ git log --graph --oneline --decorate=full
*   9480427 (HEAD -> refs/heads/develop, refs/remotes/origin/develop) merge and fix conflict
|\  
| * 3b11c03 (refs/remotes/origin/feature) add line3
* | 6682155 fix line2
|/  
* 2363f83 (refs/remotes/origin/master, refs/remotes/origin/HEAD, refs/heads/master) first commit
```

最後に feature ブランチを削除して完了する。

```sh
$ git branch -d feature # ローカルの feature ブランチを削除、ターミナル1では不要
$ git push -d origin feature # リモートの feature ブランチを削除
```
