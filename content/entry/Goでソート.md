---
title: "Goでソート"
date: 2019-02-10T18:18:39+09:00
slug: "golang-sort"
tags:
- go
draft: false
archives:
- 2019/02
---

Golang におけるソートについてまとめる。

- sort パッケージ
- sort パッケージで独自のソートを作ってみる
- スクラッチでソートしてみる

<!--more-->

# sort パッケージ

sort パッケージでは、以下の `Interface` に合わせた実装をしておけば、独自の type でも `sort.Sort(data Interface)` でソートしてくれる。

```go
type Interface interface {
	Len() int // 要素数
	Less(i, j int) bool // i 番目の要素が j 番目の要素より小さくなる条件
	Swap(i, j int) // i 番目の要素と j 番目の要素を入れ替えるロジック
}
```

とはいえ面倒なので、`IntSlice` （ `[]int` ）、`Float64Slice` （ `[]float64` ）、`StringSlice` （ `[]string` ）は用意されており、キャストして使用できる。

```go
var a []int
sort.Sort(sort.IntSlice(a))
```

キャストしたら、メンバに `Sort()` メソッドも実装されている。

```go
var a []int
sort.IntSlice(a).Sort()
```

もしくはキャストしなくとも、`Ints([]int)` 、 `Float64s([]float64)` 、 `Strings([]string)` が用意されている。

```go
var a []int
sort.Ints(a)
```

## Slice

`Interface` の実装が面倒なら `less` だけ実装すればよい関数もある。

- func Slice(slice interface{}, less func(i, j int) bool)
- func SliceStable(slice interface{}, less func(i, j int) bool)

`slice` はソート対象。
また、 `SliceStable` は [安定ソート](https://ja.wikipedia.org/wiki/%E5%AE%89%E5%AE%9A%E3%82%BD%E3%83%BC%E3%83%88) 。

# sort パッケージで独自のソートを作ってみる

## Interface の実装

```go
package main

import (
	"fmt"
	"sort"
)

type user struct {
	name string
	age  int
}

type users []user

func (u users) Len() int {
	return len(u)
}

func (u users) Less(i, j int) bool {
	return u[i].age < u[j].age
}

func (u users) Swap(i, j int) {
	u[i], u[j] = u[j], u[i]
}

func main() {
	list := []user{
		user{name: "A", age: 2},
		user{name: "B", age: 1},
		user{name: "C", age: 4},
		user{name: "D", age: 3},
	}
	us := users(list)
	sort.Sort(us)
	fmt.Println(us)
}
```

## Sliceの利用

```go
package main

import (
	"fmt"
	"sort"
)

type user struct {
	name string
	age  int
}

func main() {
	list := []user{
		user{name: "A", age: 2},
		user{name: "B", age: 1},
		user{name: "C", age: 4},
		user{name: "D", age: 3},
	}
	sort.Slice(list, func(i, j int) bool {
		return list[i].age < list[j].age
	})
	fmt.Println(list)
}
```

# スクラッチでソートしてみる

sort パッケージ使わずに。

## バブルソート

```go
package main

import (
	"fmt"
)

func sort(list []int) {
	eNum := len(list)
	for i := eNum; i > 0; i-- {
		for j := 0; j < i-1; j++ {
			if list[j] > list[j+1] {
				list[j], list[j+1] = list[j+1], list[j]
			}
		}
	}
}

func main() {
	list := []int{3, 4, 1, 2, 8, 5}
	sort(list)
	fmt.Println(list)
}
```

暇になったら他にも作ろうかなー。