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