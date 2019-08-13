---
title: "すべての**envを管理するanyenv"
date: 2017-07-05T08:17:14+09:00
slug: "anyenv"
tags:
- anyenv
- ndenv
- node.js
- pyenv
- python
- rbenv
- ruby
draft: false
archives:
- 2017/07
---

[anyenv](https://github.com/riywo/anyenv)は、renvやpyenvなどスクリプト言語のバージョン管理を行うツール類（所謂、 \*\*env）を管理するツール。  
「すべて」は言い過ぎた。  
以降、anyenv、\*\*env のインストールから簡単な使い方まで記載する。

<!--more-->

# anyenv のインストール・アップデート・アンインストール

基本は[公式](https://github.com/riywo/anyenv)を見て欲しいが、ここでは本ブログ作成日におけるMac環境へのインストールについて記載する。  
以下、anyenvのインストール方法。

## Homebrew

Homebrew にも対応した模様。

```bash
$ brew install anyenv
$ anyenv init
```

## Manual git checkout

```sh
$ git clone https://github.com/riywo/anyenv ~/.anyenv
$ echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(anyenv init -)"' >> ~/.bash_profile
$ exec $SHELL -l
```

anyenv自身のアップデートは上記の通りGithubからcloneしているだけなので、 `git pull origin master` すればいいだけだと思う。  
アンインストールは `rm -fR ~/.anyenv` 。ただし、プロファイルに設定したanyenvへのパス削除を忘れずに。

# プラグインのインストール

ここでは以下をインストールする。

- [anyenv-update](https://github.com/znz/anyenv-update)
    - インストールされている \*\*env をアップデートするためのプラグイン
    - `anyenv update` コマンドが使用できるようになる
- [anyenv-git](https://github.com/znz/anyenv-git)
    - anyenv で git のコマンドを実行できるようにするためのプラグイン
    - `anyenv git` コマンドが使用できるようになる

## anyenv-updateのインストール

```sh
$ mkdir -p $(anyenv root)/plugins
$ git clone https://github.com/znz/anyenv-update.git $(anyenv root)/plugins/anyenv-update
```

`anyenv commands` で `update` が確認されればインストールされている。

## anyenv-gitのインストール

```sh
$ git clone https://github.com/znz/anyenv-git.git $(anyenv root)/plugins/anyenv-git
```

`anyenv commands` で `git` が確認されればインストールされている。

# \*\*envの管理

## \*\*envのインストール

```sh
$ anyenv install rbenv
$ anyenv install plenv
$ anyenv install pyenv
$ anyenv install phpenv
$ anyenv install ndenv
$ anyenv install denv
$ anyenv install jenv
$ anyenv install luaenv
$ anyenv install goenv
$ exec $SHELL -l
```

## \*\*envのアップデート

以下でインストールしてある \*\*env がすべてアップデートされる。

```sh
$ anyenv update
```

## 各 \*\*env の現在のバージョンを確認

```sh
$ anyenv global
ndenv: v7.6.0
plenv: 5.25.7
pyenv: 3.5.2
rbenv: 2.3.0

$ anyenv version
ndenv: v7.6.0 (set by /Users/xxxx/.anyenv/envs/ndenv/version)
plenv: 5.25.7 (set by /Users/xxxx/.anyenv/envs/plenv/version)
pyenv: 3.5.2 (set by /Users/xxxx/.anyenv/envs/pyenv/version)
rbenv: 2.3.0 (set by /Users/xxxx/.anyenv/envs/rbenv/version)
```

# \*\*envの使い方

## ndenv

### 現在のバージョンを確認

```sh
$ ndenv version
```

### インストール済みのバージョン一覧確認

```sh
$ ndenv versions
```

### インストール可能なバージョン一覧の確認

```sh
$ ndenv install --list
```

### インストール

```sh
$ ndenv install v8.1.3
```

### バージョンの切り替え

```sh
$ ndenv global v8.1.3
$ node -v
v8.1.3
```

### npm をアップデート

```sh
$ npm update -g npm
```

## pyenv

### 現在のバージョンを確認

```sh
$ pyenv version
```

### インストール済みのバージョン一覧確認

```sh
$ pyenv versions
```

### インストール可能なバージョン一覧の確認

```sh
$ pyenv install --list
```

### インストール

```sh
$ pyenv install 3.6.1
```

### バージョンの切り替え

```sh
$ pyenv global 3.6.1
$ python -V
Python 3.6.1
```

### pipをアップデート

```sh
$ pip install -U pip
```

### pipでいれたパッケージを一括アップデートする pip-review

#### インストール

```sh
$ pip install pip-review
```

#### 更新があるパッケージを表示

```sh
$ pip-review
```

#### 一括アップデート

```sh
$ pip-review --auto
```

## rbenv

### 現在のバージョンを確認

```sh
$ rbenv version
```

### インストール済みのバージョン一覧確認

```sh
$ rbenv versions
```

### インストール可能なバージョン一覧の確認

```sh
$ rbenv install --list
```

### インストール

```sh
$ rbenv install 2.4.1
```

### バージョンの切り替え

```sh
$ rbenv global 2.4.1
$ rbenv rehashr
$ ruby -v
ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-darwin16]
```

### gemの確認とbundlerの導入

```sh
$ which gem
/Users/xxx/.anyenv/envs/rbenv/shims/gem
$ gem install bunlder
$ which bundle
/Users/xxx/.anyenv/envs/rbenv/shims/bundle
```

### rehash を不要にする rbenv-gem-rehash

「rbenv-gem-rehash」を導入することで「rbenv rehash」（~/.rbenv/versions/2.x.y/bin/ 以下に置いてあるコマンド群を ~/.rbenv/shims/以下に置いて使えるようにする）を実行する必要が無くなる。

```sh
$ git clone https://github.com/sstephenson/rbenv-gem-rehash.git ~/.anyenv/envs/rbenv/plugins/rbenv-gem-rehash
$ exec $SHELL -l
$ rbenv install 2.4.1
$ rbenv global 2.4.1
$ ruby -v
ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-darwin16]
```
