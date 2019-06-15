---
title: "Node.js・ExpressでブラウザからPUT・DELETEリクエストを発行する"
date: 2017-09-20T17:50:44+09:00
slug: "express-browser-put-delete-request"
tags:
- express
- node.js
- npm
- javascript
- yarn
draft: false
archives:
- 2017/09
---

Node.js/Expressアプリケーションに対してブラウザからPUT/DELETEリクエストを発行する方法について記載する。  
通常、ブラウザからPUT/DELETEリクエストは発行できない。  
ここで紹介する方法も実際にブラウザからPUT/DELETEリクエストを発行しているのではなく、 *Node.js/Express側でPUT/DELETEリクエストとして解釈* しているだけ。

以下の記事を読んだ前提で書く。

- [Express入門](https://blog.pepese.com/entry/express-basics/)

<!--more-->

# 環境設定

先の記事で紹介したプロジェクトにて以下を実行する。

```sh
$ yarn add method-override
```

# 実装

先の記事で紹介したプロジェクトを基準に記載する。

`app.js` へ以下を加筆する。

```javascript
const methodOverride = require('method-override');

// 中略

app.use(methodOverride('_method'));

// 省略
```

上記を記載すれば、ブラウザからリクエストを発行する際、 **クエリパラメータ** に `_method=PUT` や `_method=DELETE` を記載することによって Node.js/Express 側で PUT/DELETE リクエストとして解釈される。  
つまり、 `express.Router().put()` や `express.Router().delete()` へルーティングされるようになる。  
ブラウザからのリクエストの発行の例は以下のようになる。

```html
form(action="/clients?_method=PUT")
  input(type="submit" value="submit")
```

```html
a(href="/clients?_method=DELETE")
```

`router.put('/clients', ...` や `router.delete('/clients', ...` を実装すれば、上記のブラウザからのリクエストを処理することができるようになる。
