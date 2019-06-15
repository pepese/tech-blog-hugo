---
title: Golangでハマったこと
tags:
- golang
- go
id: golang-trap
draft: true
---

恥を晒す。

- range の value は値のコピー

<!-- more -->

# range の value は値のコピー

構造体のスライスでやってしまいがち。  
`for index, value := range list` の `value` には各要素へのポインタではなく、 **値のコピー** が渡される。  
以下の様なことをしても値は変わらない。

```go
package main

import "fmt"

func main() {
	n := 3
	nums := make([]int, n)
	for _, p := range nums {
		fmt.Println(p)
		p = 10
	}
	fmt.Println(nums)
}
```

以下のようにアドレスを調べても違うことがわかる。

```go
package main

import "fmt"

func main() {
	n := 3
	nums := make([]int, n)
	for i, n := range nums {
		fmt.Println("&nums[i]:", &nums[i])
		fmt.Println("&n      :", &n)
	}
}
```

```bash
&nums[i]: 0xc000012100
&n      : 0xc000014068
&nums[i]: 0xc000012108
&n      : 0xc000014068
&nums[i]: 0xc000012110
&n      : 0xc000014068
```

ポインタのスライス（ここでは、 `[]*int` ）なら値のコピーとはいえアドレスのコピーなので大丈夫。  
ただし、初期化は `nil` なので、値の初期設定に `range` は使えない。

```go
package main

import "fmt"

func main() {
	n := 3
	nums := make([]*int, n)
	for i := 0; i < n; i++ {
		num := 10
		nums[i] = &num
	}
	for i, n := range nums {
		fmt.Println("nums[i]:", nums[i])
		fmt.Println("n      :", n)
	}
}
```

```bash
nums[i]: 0xc000014068
n      : 0xc000014068
nums[i]: 0xc000014070
n      : 0xc000014070
nums[i]: 0xc000014078
n      : 0xc000014078
```

構造体のスライスでやってしまいがちなやつ。

```go
package main

import "fmt"

type person struct {
	age int
}

func main() {
	n := 3
	people := make([]person, n)
	for _, p := range people {
		p.age = 10 // 10 を代入したが、
	}
	for _, p := range people {
		fmt.Println(p.age) // 0 が出力される（値は変わっていない）
	}
}
```

以下のように変更。

```go
package main

import "fmt"

type person struct {
	age int
}

func main() {
	n := 3
	people := make([]*person, n)
	// 初期化
	for i := 0; i < n; i++ {
		people[i] = &person{age: 0}
	}
	// 変更
	for _, p := range people {
		p.age = 10
	}
	// 確認（10 に変わってる）
	for _, p := range people {
		fmt.Println(p.age) // コンパイラが (*p).age と解釈してくれてる
	}
}
```