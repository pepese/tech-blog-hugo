---
title: GitでPull Request
tags:
- Git
- Github
id: git-pull-request
draft: true
---

今更ながら Git の Pull Request （以降、 PR ）について整理する。

- 概要
- Fork & Pull Model
- Shared Repository Model

<!-- more -->

# 概要

PR 自体については割愛。 PR には主に以下の二種類がある。

- Fork & Pull Model
    - 自分のリポジトリ（ origin ）に変更を加えたいリポジトリ（ upstream ）を Fork する方式
    - OSS 開発に参加する場合などはこちら
- Shared Repository Model
    - 一つの共有リポジトリ（ origin ）のみを利用し Fork しない方式
    - 開発者数が比較的少ない自社開発などはこちら

# Fork & Pull Model

- [Github Fork Pull Request](http://kik.xii.jp/archives/179)

# Shared Repository Model

- [GitHub初心者はForkしない方のPull Requestから入門しよう](https://blog.qnyp.com/2013/05/28/pull-request-for-github-beginners/)
- [一人プルリクエストのすゝめ](https://crieit.net/posts/9c710ef2383a3703649ee712a9eb86e6)
