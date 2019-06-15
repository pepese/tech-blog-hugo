---
title: GolangのCobra+net/httpでWebサーバを作る
tags:
- golang
- go
- cobra
id: golang-cobra-net-http-web
draft: true
---

コマンドラインツール作成パッケージの [Cobra](https://github.com/spf13/cobra) と標準パッケージの `net/http` を使って Web サーバを作ってみる。

- Cobra
- net/http
- その他トピック
    - Graceful Shutdown
    - Filter 的な機能

<!-- more -->

# Cobra

Cobra のセットアップと初期プロジェクトの作成。

```bash
$ go get -u github.com/spf13/cobra/cobra
$ which cobra
${GOPATH}/bin/cobra
```

Cobra Generator を使って、プロジェクトの初期化。  
後述の `$ cobra init` はデフォルトで `${GOPATH}/src` 配下にプロジェクトを作ってしまうので好きなパスに作りたい場合は絶対パスで実行。  
また、既にディレクトリが存在する場合、空ディレクトリでないとエラーになるので注意。ディレクトリ名がコマンド名になる。

```bash
$ cd ${GOPATH}/src/github.com/[アカウント名]
$ mkdir web # web というコマンドにする
$ cobra init $(pwd)/web
Your Cobra application is ready at
${GOPATH}/src/github.com/[アカウント名]/golang-sample/web

Give it a try by going there and running `go run main.go`.
Add commands to it by running `cobra add [cmdname]`.
```

空のディレクトリで実行しないとエラーになるので注意。

```
$ tree web
web
├── LICENSE
├── cmd
│   └── root.go
└── main.go

1 directory, 3 files

# 個人的な管理の都合でディレクトリ名は変更しておく（ xxxx は適当）
# $ mv web xxxx
# $ cd xxxx
```

`cmd/root.go` にルートコマンドが作成される。  
なお、 `Cobra` は 12-Factor Apps に沿った設定機能を提供する [viper](https://github.com/spf13/viper) を標準で利用する。  
Web サーバを起動する `start` コマンドを追加する。

```
$ cobra add start
start created at ${GOPATH}/src/github.com/[アカウント名]/xxxx/cmd/start.go
$ tree .
.
├── LICENSE
├── cmd
│   ├── root.go
│   └── start.go
└── main.go

1 directory, 4 files
```

`cmd/start.go` が追加された。ここに Web サーバの起動処理を記載していく。  
ここで試しにビルド＆実行してみる。  
（個人的にディレクトリ名を変えた営業で `main.go` の `import` がラリってるので修正する、、、）

```
$ go build -o web
$ ./web
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  web [command]

Available Commands:
  help        Help about any command
  start       A brief description of your command

Flags:
      --config string   config file (default is $HOME/.web.yaml)
  -h, --help            help for web
  -t, --toggle          Help message for toggle

Use "web [command] --help" for more information about a command.
$ ./web start
start called
```

そのまま実行だとルートコマンド、 `start` をつけるとスタートコマンドが実行され、その結果が出てることがわかる。  
これらの挙動は、 `cmd/root.go` `cmd/start.go` のそれぞれで定義されている `var rootCmd = &cobra.Command{/*省略*/}` `var startCmd = &cobra.Command{/*省略*/}` にある。  
`cobra.Command` 構造体は以下の通り。（ここでは主要なものだけ挙げている。全項目は [ここ](https://godoc.org/github.com/spf13/cobra#Command) ）

```go
type Command struct {
	Use string   // USAGE
	Short string // help の短い説明
	Long string  // サブコマンドの help の長い説明

	// *Run 関数は以下の順で実行される
	//   * PersistentPreRun()
	//   * PreRun()
	//   * Run()
	//   * PostRun()
    //   * PersistentPostRun()
    // 最後に E がついている関数は error を返却する

	// PersistentPreRun: children of this command will inherit and execute.
	PersistentPreRun func(cmd *Command, args []string)
	// PersistentPreRunE: PersistentPreRun but returns an error.
	PersistentPreRunE func(cmd *Command, args []string) error
	// PreRun: children of this command will not inherit.
	PreRun func(cmd *Command, args []string)
	// PreRunE: PreRun but returns an error.
	PreRunE func(cmd *Command, args []string) error
	// Run: Typically the actual work function. Most commands will only implement this.
	Run func(cmd *Command, args []string)
	// RunE: Run but returns an error.
	RunE func(cmd *Command, args []string) error
	// PostRun: run after the Run command.
	PostRun func(cmd *Command, args []string)
	// PostRunE: PostRun but returns an error.
	PostRunE func(cmd *Command, args []string) error
	// PersistentPostRun: children of this command will inherit and execute after PostRun.
	PersistentPostRun func(cmd *Command, args []string)
	// PersistentPostRunE: PersistentPostRun but returns an error.
	PersistentPostRunE func(cmd *Command, args []string) error
}
```

いろいろあるが、最低限 `Run` だけ実装しておけばよい。

# net/http

先に記載した `cmd/start.go` の `&cobra.Command{Run: startWeb}` を実装していく。  
いろいろ省略してるが、以下のような感じ。

```go
var startCmd = &cobra.Command{
	Run: startWeb,
}

func startWeb(cmd *cobra.Command, args []string) {
	fmt.Println("startWeb called")
}
```

```
$ go build -o web
$ ./web start
startWeb called
```

## net/http パッケージのサーバ機能

`net/http/server.go` の実装をみればよくわかる。

まずは **Mux** について記載する。  
Mux は multiplexer の略で、アクセスされる URL パターンから対応する処理を持つ関数（ **Handler** ）を呼び出すルータの役割を持ち、 `http.ServeMux` という構造体で定義されている。

```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}
```

net/http パッケージには `http.DefaultServeMux` というデフォルトの Mux がある。  
`http.ServeMux` には以下のメソッドがある。

- func (mux *ServeMux) Handle(pattern string, handler Handler)
- func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))

### func (mux *ServeMux) Handle(pattern string, handler Handler)

Handle メソッドは URL パターン（ `pattern` ）に対して対応する処理（ `handler` ）を登録する機能を持つ。  
`handler` は以下の `Handler` インターフェースを実装している必要がある。

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

第 1 引数はレスポンス、第 2 引数はリクエストのポインタが渡される。  
自作の構造体に対して `ServeHTTP(ResponseWriter, *Request)` メソッドを実装して第 2 引数に渡すことになる。

### func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))

構造体を作成して `Handler` インターフェースを実装する必要がない場合は直接処理を行う関数を定義できる。

```go
mux := http.NewServeMux()
mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request){
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, "Hello World")
})
```

>HandleFunc の仕組みは以下の `HandlerFunc` 型によって実現されている。（ Handle じゃなくて Handle「r」 になっている）
>
>```go
>type HandlerFunc func(ResponseWriter, *Request)
>
>func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
>    f(w, r)
>}
>```
>
>つまり、任意の名前（無名含む）の関数を渡しても `net/http` パッケージ内部で `HandlerFunc` にキャストしてしまえば、 `ServeHTTP` メソッドを実装していることになり `Handler` インターフェースを満たすことになる。

### Webサーバの起動

Web サーバの起動には `http.ListenAndServe` 関数を使用する。

```go
func startWeb(cmd *cobra.Command, args []string) {
	http.ListenAndServe(":8080", nil)
}
```

第 2 引数には `http.ServeMux` を渡すが、 `nil` の場合は `net/http` 内部で `DefaultServeMux` が使われている。  
上記だと Handler を何も定義していないので `404` が返却される。

# その他トピック

## Graceful Shutdown

新規のリクエストは受け付けず、既に受け付けた処理を終えてからサーバ停止する機能。  
`net/http` には既に実装されている。

- func (srv *Server) Shutdown(ctx context.Context) error

## Filter 的な機能

Java でいう Filter 的な機能を作ってみる。


# 参考

- https://qiita.com/nirasan/items/2160be0a1d1c7ccb5e65
