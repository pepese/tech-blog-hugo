---
title: "Node.js・Express・Multerでファイルアップロード・ダウンロード"
date: 2017-09-21T08:10:39+09:00
slug: "express-multer-fileupload"
tags:
- node.js
- yarn
- express
- javascript
- multer
draft: false
archives:
- 2017/09
---

Node.js/Expressアプリケーションへファイルアップロードしてみる。  
使用するツール・ライブラリは以下。

- [Multer](https://github.com/expressjs/multer)

以下の記事を読んだ前提で書く。

- [Express入門](https://blog.pepese.com/entry/express-basics/)

<!--more-->

# 環境設定

```sh
$ yarn add multer
```

# 実装

controller 層へ以下のような実装を行う。

## ファイルアップロード

```javascript
const multer  = require('multer');

// アップロードされたファイルをストレージへ保存する場合
const storage = multer.diskStorage({
  // ファイルの保存先を指定
  destination: (req, file, cb) => {
    cb(null, '/tmp');
  },
  // ファイル名を指定(オリジナルのファイル名を指定)
  filename: (req, file, cb) => {
    cb(null, file.originalname);
  }
});

// アップロードされたファイルを一時的にメモリへ保存する場合
const storage = multer.memoryStorage();

// 1 つのファイルをアップロードする場合
const upload_ = multer({ storage: storage }).single('qafile');

const upload = (req, res, next) => {
  upload_(req, res, (err) => {
    if (err) {
      next(err);
    }
    const csvData = req.file.buffer;
    const mimetype = req.file.mimetype;
    let message = '';
    let qaData = [];
    let errorData = [];
    if(mimetype == 'text/csv' || mimetype == 'application/vnd.ms-excel') {
      // 何らかのファイルの処理ロジック
      // テキストファイルであれば `csvData.toString()` してから処理する
    }
  }
}
```

## ファイルダウンロード

ダウンロード処理には特別ライブラリは必要ない。  
CSVなどのテキストデータの場合は以下。

```javascript
const download = (req, res, next) => {
  let str = 'hoge, hoge, hoge';
  res.setHeader('Content-disposition', 'attachment; filename=' + 'hoge.csv');
  res.setHeader('Content-type', 'text/csv; charset=UTF-8');
  res.write(str);
  res.end();
};
```
