---
title: "Goでファイル入出力"
date: 2019-02-08T18:19:52+09:00
slug: "golang-file-io"
tags:
- go
draft: false
archives:
- 2019/02
---

golang のファイル入出力についてまとめる。

- fmt パッケージ
- os パッケージ
- io/ioutil パッケージ
- bufio パッケージ

<!--more-->

# fmt パッケージ

標準入出力の場合だが fmt パッケージが利用できる。

- http://golang.jp/pkg/fmt

# os パッケージ

読み込みの例。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// `in.txt` には「hoge」とある
	file, _ := os.Open(`in.txt`) // *File, error
	defer file.Close()           // お約束の Close

	buf := make([]byte, 3) // 1 回の File.Read で読み込むバッファサイズ
	for {
		n, _ := file.Read(buf)    // int, error
		fmt.Print("n=", n, " : ") // 実際に読み込んだバイト数
		if n == 0 {
			fmt.Println("")
			break
		}
		fmt.Println(string(buf[:n]))
	}
}
// 結果
// n=3 : hog
// n=1 : e
// n=0 : 
```

`os.File` のメソッドには以下のようなものがある。

- func (f *File) Read(b []byte) (n int, err error)
- func (f *File) ReadAt(b []byte, off int64) (n int, err error)
    - ファイルの途中（ `off` byte ）から読み込み開始
- func (f *File) Write(b []byte) (n int, err error)
- func (f *File) WriteAt(b []byte, off int64) (n int, err error)
    - ファイルの途中（ `off` byte ）から書き込み開始
- func (f *File) WriteString(s string) (n int, err error)
- func (f *File) Seek(offset int64, whence int) (ret int64, err error)
- func (f *File) Close() error

上記のメソッドは `io` パッケージに `Reader` `Writer` `Closer` のようなインターフェースとして定義されている。  
書き込みはもう想像つくと思うから省略。  
`os.Create()` というメソッドがあるということだけ買いておこう。

> ちなみに標準入力の `os.Stdin` は `*io.File` 型であり、 `io.Reader` インターフェースを満たしている。

# io/ioutil パッケージ

その名の通り、 io 系のユーティル機能を提供する。  
`Close` しなくてよかったりして楽。  
以下のようなメソッドがある。

- func ReadAll(r io.Reader) ([]byte, error)
    - `io.Reader` から全部読む
- func ReadFile(filename string) ([]byte, error)
    - ファイルから全部読む
- func WriteFile(filename string, data []byte, perm os.FileMode) error
    - ファイルへ全部書く

# bufio パッケージ

`ioutil` はちと大胆なので、通常は行単位の読み書きやバッファリング機能をもつ `bufio` パッケージを使用する。

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// 1234[改行]5678[改行]90
	f, _ := os.Open(`ins.txt`) // *File, error
	defer f.Close()

	sc := bufio.NewScanner(f)
	for i := 1; sc.Scan(); i++ { // Scan() は次の読み込み位置へ移動して bool （次読める、読めない）を返す
		line := sc.Text() // 1 行を string で取得
		fmt.Printf("%d行目: %s\n", i, line)
	}
}
```

`bufio.Scanner` は内部に `MaxScanTokenSize` という定数を持っており、 **64 KB を超える行を読み込めない** ので注意が必要。  
他には以下のメソッドがある。

- func (s *Scanner) Bytes() []byte
    - 1 行を `[]byte` で取得
- func (s *Scanner) Buffer(buf []byte, max int)
    - 1 行の読み込み上限サイズ（ `MaxScanTokenSize` ）を変更できる
- func (s *Scanner) Split(split SplitFunc)
    - デフォルトでは行単位の読み込み単位を変更できる
    - 行単位（ `bufio.ScanLines` : デフォルト）単語単位（ `bufio.ScanWords` ）、バイト単位（ `bufio.ScanBytes` ）、文字（ `rune` ）単位（ `bufio.ScanRunes` ）
    - また、 `func(data []byte, atEOF bool) (advance int, token []byte, err error)` を満たす関数であれば独自カスタマイズも可能