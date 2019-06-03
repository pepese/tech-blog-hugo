---
title: "ESLintまとめ"
date: 2019-06-03T17:16:43+09:00
slug: ""
tags:
- tag_name
draft: true
archives:
- 2019/06
---

[ESLint](https://eslint.org/) についてまとめる。

- 環境構築
- コマンド
- 設定ファイル

<!--more-->

# 環境構築

プロジェクトディレクトリで以下を実行。

```bash
$ npm install --save-dev eslint
```

# コマンド

`--fix` オプション。


# 設定ファイル

設定ファイル `.eslintrc.*` には以下の種別がある。

|記述方法|ファイル|特記事項|
|---|---|---|
|JavaScript| `.eslintrc.js` ||
|YAML| `.eslintrc.yaml` / `.eslintrc.yml` ||
|JSON| `.eslintrc.json` ||
|JSON| `package.json` | `eslintConfig` プロパティに設定を記載 |

ここでは、 `.eslintrc.js` で書く。

```javascript
module.exports = {
    "parser": "",
    "parserOptions": {
    },
    "plugins": [
        ""
    ],
    "extends": [
        ""
    ],
    "rules": {
        "ルール名": "ルール設定",
    },
    "globals": {
        "変数名": true or false(書き換え可能かどうか)
    },
    "env": {
        "環境名": true or false(有効にするかどうか)
    }
}
```
