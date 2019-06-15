---
title: Golangのユニットテスト
tags:
- golang
- go
id: golang-unit-testing
draft: true
---

golang のユニットテストについてまとめる。

- 概要
- testing パッッケージ
- テスト手法

<!-- more -->

# 概要

- テストファイル名は `xxx_test.go`（ `xxx` はテスト対象ファイル（拡張子無し）名）
- `testing` パッケージを利用
- テストのパッケージ名は `xxx_test` （ `xxx` はテスト対象パッケージ）
    - パッケージ外に公開していない関数のテストを行いたい場合はテスト対象パッケージと同じ（ `xxx` ）でもよい
- テスト関数は `func TestXxx(t *testing.T)`
    - 構造体メソッドは `func TestStruct_Xxxx(t *testing.T)`
- `$ go test` でテスト実行
    - `$ go test <パッケージ名>` でパッケージ名指定でテスト実行
    - `$ go test xxx_test.go xxx.go` でファイル指定でテスト実行
    - `$ go test -run ''` でカレントディレクトリのテスト実行
    - `$ go test -run <テスト関数名>` でテスト関数名（正規表現）指定でテスト実行
- テストデータは `testdata` ディレクトリへ
    - テスト結果も同ディレクトリへ `.golden` 拡張子で

# testing パッケージ

## testing.T

- `func (c *T) Error(args ...interface{})` `func (c *T) Errorf(format string, args ...interface{})`
    - テスト失敗としてログ出力、以後の処理は実行
- `func (c *T) Fail()`
    - テスト失敗とする、以後の処理は実行
- `func (c *T) FailNow()`
    - テスト失敗とする、以後の処理は実行 **されない**
- `func (c *T) Failed() bool`
- `func (c *T) Fatal(args ...interface{})` `func (c *T) Fatalf(format string, args ...interface{})`
    - テスト失敗としてログ出力、以後の処理は実行 **されない**
- func (c *T) Helper()
- `func (c *T) Log(args ...interface{})` `func (c *T) Logf(format string, args ...interface{})`
- func (c *T) Name() string
- func (t *T) Parallel()
- func (t *T) Run(name string, f func(t *T)) bool
- func (c *T) Skip(args ...interface{})
- func (c *T) SkipNow()
- func (c *T) Skipf(format string, args ...interface{})
- func (c *T) Skipped() bool

## testing.B

- func (c *B) Error(args ...interface{})
- func (c *B) Errorf(format string, args ...interface{})
- func (c *B) Fail()
- func (c *B) FailNow()
- func (c *B) Failed() bool
- func (c *B) Fatal(args ...interface{})
- func (c *B) Fatalf(format string, args ...interface{})
- func (c *B) Helper()
- func (c *B) Log(args ...interface{})
- func (c *B) Logf(format string, args ...interface{})
- func (c *B) Name() string
- func (b *B) ReportAllocs()
- func (b *B) ResetTimer()
- func (b *B) Run(name string, f func(b *B)) bool
- func (b *B) RunParallel(body func(*PB))
- func (b *B) SetBytes(n int64)
- func (b *B) SetParallelism(p int)
- func (c *B) Skip(args ...interface{})
- func (c *B) SkipNow()
- func (c *B) Skipf(format string, args ...interface{})
- func (c *B) Skipped() bool
- func (b *B) StartTimer()
- func (b *B) StopTimer()

## testing.TB

`testing.T` と `testing.B` を合わせたもの。

# テスト手法

- テーブルテスト
- 前処理・後処理
- コードカバレッジ
- ベンチマーク

## テーブルテスト

```go
func TestSum(t *testing.T) {
    tests := []struct {
        in, want int
    }{
        {0, 1},    // インプット, 期待するアウトプット
        {-3, -1},
    }
    for _, tt := range tests {
        if got := xxx.Xxx(tt.in); got != tt.want {
            t.Fatalf("want = %d, got = %d", tt.want, got)
        }
    }
}
```

`testing.T.Run` を用いてサブテストを書く。

```go
func TestSum(t *testing.T) {
    tests := []struct {
        name       string
        in, want   int
    }{
        {"test1", 0, 1},  // "テスト名", インプット, 期待するアウトプット
        {"test2", -1, -2},
        {"test3", -3, -1},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) { // サブテスト
            if got := xxx.Xxx(tt.in; got != tt.want {
                t.Fatalf("want = %d, got = %d", tt.want, got)
            }
        })
    }
}
```

## 前処理・後処理

`TestMain(m *testing.M)` でテストの前処理・後処理を記載できる。  
テスト関数群の実行は `m.Run()` 。

```go
func TestMain(m *testing.M) {
	func() { // 前処理
		fmt.Println("Prepare test")
	}()
	ret := m.Run() // テスト関数群実行
	func() { // 後処理
		fmt.Println("Teardown test")
	}()
	os.Exit(ret)
}
```

## コードカバレッジ

`$ go test`コマンドに `-cover` オプションをつけて実行する。