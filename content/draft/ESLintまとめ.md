---
title: "ESLintまとめ"
date: 2019-06-03T17:16:43+09:00
slug: "eslint-basics"
tags:
- javascript
- eslint
draft: true
archives:
- 2019/06
---

[ESLint](https://eslint.org/) についてまとめる。

- 環境構築
- コマンド
- 設定ファイル
- 自分の設定
- プラグイン

<!--more-->

# 環境構築

プロジェクトディレクトリで以下を実行。

```bash
$ npm install --save-dev eslint
```

# コマンド

`package.json` のスクリプトなどに以下のように設定する。

```javascript
"scripts": {
    "lint": "eslint --ext .js --ignore-path .gitignore . -f codeframe --fix"
},
```

`--ext` でチェック対象のファイル拡張子を指定。  
`-f` でチェック結果出力のフォーマットを指定。  
`--fix` でコードの自動修正を指定。全てが自動修正される訳ではない。[参考](https://eslint.org/docs/rules/)  
また、チェック対象外のファイルを指定したい場合は、 `.gitignore` のESLint版である `.eslintignore` を利用できる。


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
    "rules": {
        "ルール名": "ルール設定",
    },
    "globals": {
        "変数名": true or false(書き換え可能かどうか)
    },
    "env": {
        "環境名": true or false(有効にするかどうか)
    },
    "extends": [
        ""
    ],
    "parser": "",
    "parserOptions": {
    },
    "plugins": [
        ""
    ]
}
```

## rules

ルール設定は以下のようになる。

|設定値|省略形|意味|
|---|---|---|
|"off"|0|チェックをしない|
|"warn"|1|違反したら警告|
|"error"|2|違反したらエラー|

```javascript
{
  "rules": {
    "文字列オプションありのルール": ["ルール設定", "オプション"],
    "オブジェクトオプションありのルール": ["ルール設定", {"オプションキー": "オプションバリュー"}]
  }
}
```

ルールはたくさんあるので、 [公式のデモ](https://eslint.org/demo/) を利用して動かしながら確認するとよい。

## globals

「グローバルに存在する変数か否か」「それをチェックの対象とするか否か」を設定する。  
`false` にすると、グローバル変数でチェックの対象外、という意味。  
`globals` は、 `no-undef` / `no-global-assign` と併用する必要あり。

## env

`globals` でグローバル変数を全て設定するのは大変。  
なので、 `env` で `browser` とか `jQuery` とか設定して、その環境でのグローバル変数に対して一括設定する。  
基本的に `env` で設定して、 `env@ になかったら `globals` で設定する。

## extends

`rules` で 1 つ 1 つ設定するのは大変。  
`extends` で `"eslint:recommended"` などを利用すれば、おすすめルールを一括で設定できる。  
`extends` にイヤなルールがある場合は、 `rules` で上書き可能。  
また、上書きについてはディレクトリ単位で設定可能で、ルールを上書きしたいディレクトリにそのルールを設定した `.eslintrc.js` を配置する。  
また、対象ファイルに `/*eslint no-console: "off"*/` というコメントの形で直接指定することも可能。  
チェックの上書き優先順は以下のようになる。（上ほど強い）

1. Lint対象ファイルのコメント
2. Lint対象ファイルと同階層にある `.eslintrc.js` の `rules`
3. Lint対象ファイルの同階層にある `.eslintrc.js` の `extends`
4. Lint対象ファイルの親階層にある `.eslintrc.js` の `rules`
5. Lint対象ファイルの親階層にある `.eslintrc.js` の `extends`

## parserOptions

ESLint はデフォルトでは ES5 にしか対応していない。  
拡張したい場合は `parserOptions` を利用する。

## root

ルールの上書きで親階層から探さず、そこで止めて欲しい場合に設定する。

# 自分の設定

```javascript
module.exports = {
    "root": true,
    "env": {
        "node": true,
        "commonjs": true,
        "es6": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:prettier/recommended"
    ],
    "parserOptions": {
        "ecmaVersion": 2017,
        "sourceType": "module"
    },
    "rules": {
        "no-console": 0,
        "no-debugger": 0
    }
};
```

# プラグイン

## VSCode

[VS Code ESLint extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) を利用すると、保存時に `eslint` が実行される。