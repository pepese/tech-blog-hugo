---
title: "jestとsupertestでテストする"
date: 2019-06-07T07:13:48+09:00
slug: "jest-and-supertest-testing"
tags:
- jest
- supertest
draft: true
archives:
- 2019/06
---

[jest](https://jestjs.io/) は Faceboot 製のテスティングフレームワークでアサーションやモックなどの機能も含まれる。  
[supertest](https://github.com/visionmedia/supertest) は HTTP リクエストベースのテストツールで HTTP クライアントである [superagent](https://github.com/visionmedia/superagent) ベースのリクエスト発行とアサーション機能を提供する。

- 環境構築
- テスト対象
- jest
- suptertest

<!--more-->

# 環境構築

```bash
$ npm i -D jest supertest
```

# テスト対象

Express 製の以下の簡易なサーババイドアプリをテスト対象とする。

# jest

# supertest

最初にポイント。

- テスト対象に `async/await` ・ `Promise` が含まれる場合は `return` する
- テストをファイル内で記述した順にシーケンシャルに実行した場合も `return` する