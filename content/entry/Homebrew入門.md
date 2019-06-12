---
title: "Homebrew入門"
date: 2017-05-09T08:00:18+09:00
slug: "homebrew-basics"
tags:
- homebrew
draft: false
archives:
- 2017/05
---

[Homebrew](http://brew.sh/index_ja.html)についてまとめる。  
HomebrewはMac用のaptやyumのようなパッケージマネージャ。ソフトウェアを簡単にインストールできる。  
インストールから使い方までまとめてみる。

<!--more-->

# 環境設定

## インストール

```sh
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”
==> This script will install:
/usr/local/bin/brew
/usr/local/share/doc/homebrew
/usr/local/share/man/man1/brew.1
/usr/local/share/zsh/site-functions/_brew
/usr/local/etc/bash_completion.d/brew
/usr/local/Homebrew
==> The following new directories will be created:
/usr/local/Cellar
/usr/local/Homebrew
/usr/local/Frameworks
/usr/local/bin
/usr/local/include
/usr/local/lib
/usr/local/opt
/usr/local/sbin
/usr/local/share/zsh
/usr/local/share/zsh/site-functions
```

上記を実行後、 `brew doctor` で問題がないことを確認。  
もしWarningが出たらメッセージに従って対処する。  
筆者の場合はXcodeをインストールしてCommand Line Toolsをアップデートしたら解決した。  
変更があるかもしれないので[公式](http://brew.sh/index_ja.html)を確認のこと。

- `/usr/local/Cellar/`
    - Homebrewでインストールしたソフトウェアはここに配備される
- `/usr/local/bin/`
    - インストールしたソフトウェアのコマンドのシンボリックリンクはここに配備される

## Homebrew のアップデート

```sh
$ brew update
```

## ソフトウェアのインストール

```sh
$ brew install <software>
```

## ソフトウェアの更新

```sh
$ brew upgrade <software>
```

## キャッシュされている古いソフトウェアの削除

```sh
$ brew cleanup
```

# brew tap

brew tap は、公式以外の formula を追加することのできる Homebrew のサブコマンド。  
Homebrewをインストールした際に標準で組み込まれている。

```sh
$ brew tap <userName>/<repository>
```

上記コマンドを実行すると、GitHub 公開リポジトリ (https://github.com/<userName>/homebrew-<repository>) が参照され、/usr/local/Library/Taps以下にインストールされる。  
`brew tap` コマンドでこれまでにインストールされたソフトウェアの一覧が見れる。  
また、 `brew tap <url>` コマンドでGitHub以外のソフトウェアもインストール可能。

# brew cask

brewやbrew tapでソフトウェアをインストールする際、目的のソフトウェアを動かすには依存するその他のソフトウェアをインストールする必要がある場合がある。  
そんなときbrew caskを使用すると目的のソフトウェアをインストールするだけで依存するソフトウェアもインストールしてくれる。  
さらにGoogle ChromeやAtomといったソフトのインストールも可能となる。

- `/usr/local/Caskroom/`
    - Cask でインストールしたソフトウェアはここに配備される
- `/usr/local/bin/`
    - インストールしたソフトウェアのコマンドのシンボリックリンクはここに配備される

## インストール

brew tapでインストールする。

```sh
$ brew tap caskroom/cask
```

caskで扱えるソフトウェアのバージョンを増やすために下記も導入しておく。

```sh
$ brew tap caskroom/versions
```

変更があるかもしれないので[cask公式](https://caskroom.github.io)、[versions公式](https://github.com/caskroom/homebrew-versions)を参照。

## 使い方

```sh
$ brew cask search <software>
$ brew cask install <software>
$ brew cask uninstall <software>
```

その他のコマンドは[ここ](https://github.com/caskroom/homebrew-cask/blob/master/USAGE.md)。
