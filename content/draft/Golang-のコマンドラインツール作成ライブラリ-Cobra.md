---
title: Golang のコマンドラインツール作成ライブラリ Cobra
tags:
- go
- golang
- cobra
id: golang-cobra
draft: true
---

コマンドラインツール作成ライブラリ。

<!-- more -->

```bash
$ go get -u github.com/spf13/cobra/cobra
```

コマンドラインツール `cobra` が入る。  
`cobra init` コマンドで以下の通りの雛形が作成される。

```
.
├── LICENSE
├── cmd
│   └── root.go
└── main.go
```

`cobra add version -p "rootCmd"` でサブコマンド `version` が追加される。

```
.
├── LICENSE
├── cmd
│   ├── root.go
│   └── version.go
└── main.go
```

cobra はデフォルトで **viper** という設定ファイル読み込みライブラリが使用される。

- cobra 参考
    - https://qiita.com/tkit/items/3cdeafcde2bd98612428
	- https://qiita.com/minamijoyo/items/cfd22e9e6d3581c5d81f