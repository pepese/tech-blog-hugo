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

バイナリ取得ではなくて、 Github から設定する。

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
```

# プロジェクトの作成

```bash
$ flutter create flutter_sample
$ cd flutter_sample
$ open -a Simulator
$ flutter run
...
To hot reload changes while running, press "r". To hot restart (and rebuild state), press "R".
...
For a more detailed help message, press "h". To detach, press "d"; to quit, press "q".
```