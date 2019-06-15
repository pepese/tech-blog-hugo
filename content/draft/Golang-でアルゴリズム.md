---
title: Golang でアルゴリズム
tags:
- golang
- go
id: golang-algorithm
draft: true
---

タイトルは「Golagn でデータ構造とアルゴリズム」に変えるかな。  
アルゴリズムの複数兼ねて golang でまとめる。

- データ構造
- リスト探索
- ツリー・グラフ探索
- 数え上げ
- ソート
- 連立方程式
- その他

# データ構造

データ構造は、データの集まりを効率よく扱うための形。

- スタック
- キュー
- 連結リスト
- 二分木
- B 木

# スタック・キュー・連結リストの前提

- https://www.iandprogram.net/entry/2016/07/20/183638

```go
type node struct {
    next *node
    data int
}

type container struct {
    root *node
    size int
}

func (c *container) incremant() {
    c.size++
}

func (c *container) decremant() {
    c.size--
}

func (c *container) len() int {
    return c.size
}
```

## スタック

```go
package main

import (
	"errors"
)

type node struct {
	next *node
	data int
}

type stack struct {
	top  *node
	size int
}

func (s *stack) len() int {
	return s.size
}

func (s *stack) push(x int) {
	node := &node{data: x}
	if s.len() == 0 {
		s.top, s.size = node, 1
	} else {
		node.next, s.top = s.top, node
		s.size++
	}
}

func (s *stack) pop() (int, error) {
	if s == nil || s.len() == 0 {
		return 0, errors.New("error")
	}
	data := s.top.data
	s.top = s.top.next
	s.size--
	return data, nil
}

func newStack(x int) *stack {
	s := new(stack)
	node := &node{data: x}
	s.top, s.size = node, 1
	return s
}
```

## キュー

```go
package main

import (
	"errors"
)

type node struct {
	next *node
	data int
}

type queue struct {
	head, tail *node
	size       int
}

func (q *queue) len() int {
	return q.size
}

func (q *queue) enqueue(x int) {
	node := &node{data: x}
	if q.len() == 0 {
		q.head, q.tail, q.size = node, node, 1
	} else {
		q.head.next, q.head = node, node
		q.size++
	}
}

func (q *queue) dequeue() (int, error) {
	if q == nil || q.len() == 0 {
		return 0, errors.New("error")
	}
	data := q.tail.data
	if q.len() == 1 {
		q.head, q.tail = nil, nil
	} else {
		q.tail = q.tail.next
	}
	q.size--
	return data, nil
}

func newQueue(x int) *queue {
	q := new(queue)
	node := &node{data: x}
	q.head, q.tail, q.size = node, node, 1
	return q
}
```

## 連結リスト

## 二分木

```go
package main

import (
	"fmt"
)

type binaryNode struct {
	left, right *binaryNode
	data        int
}

func (n *binaryNode) insert(x int) {
	if n == nil {
		return
	}
	if n.data > x {
		if n.left == nil {
			n.left = newBinaryNode(x)
		} else {
			n.left.insert(x)
		}
	} else {
		if n.right == nil {
			n.right = newBinaryNode(x)
		} else {
			n.right.insert(x)
		}
	}
}

func (n *binaryNode) search(x int) bool {
	for n != nil {
		switch {
		case n.data == x:
			return true
		case n.data > x:
			n = n.left
		default:
			n = n.right
		}
	}
	return false
}

func newBinaryNode(x int) *binaryNode {
	node := new(binaryNode)
	node.data = x
	return node
}

func main() {
	root := newBinaryNode(5)
	fmt.Println(*root)
	root.insert(4)
	fmt.Println(root.left)
	fmt.Println(root.right)
	fmt.Println(root.search(4))
}
```

## B 木

# リスト探索

リストを検索する際の以下について。

- for 全検索
    - 線形探索、 n 次元探索
- 二分探索
- ハッシュ

for 全検索は省略。

## 二分探索

二分探索は *ソートされている配列に対して* 探索範囲を半分ずつに絞ることによって `O(log N)` で探索することができるアルゴリズム。

```go
package main

import "fmt"

func binarySearch(list []int, elem int) int {
	start, end, index := 0, len(list)-1, 0
	for true {
		index = (start + end) / 2
		if end < start {
			return -1
		} else if list[index] == elem {
			return index
		} else if list[index] < elem {
			start = index + 1
		} else if list[index] > elem {
			end = index - 1
		}
	}
	return -1
}

func main() {
	list := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
	fmt.Println(binarySearch(list, 6))
}
```

## ハッシュ

# ツリー・グラフ探索

- DFS （ Depth First Search ： 深さ優先探索）
- BFS （ Breadth-First Search ： 幅優先探索）
- IDDFS （ Iterative Deepening Depth First Search ： 反復深化探索）

- http://hos.ac/slides/20110504_graph.pdf

## DFS

## BFS

## IDDFS

# 数え上げ

- 順列
- 組合せ（ bit 全検索）
- 重複順列
- 重複組合せ

## 順列

Golang には C++ でいう next_permutation はないので自作した。

```go
package main

import "fmt"

// リストから指定のインデッックスの要素を削除
func delEle(list []int, i int) []int {
	result := []int{}
	result = append(result, list[:i]...)
	result = append(result, list[i+1:]...)
	return result
}

func permutation(list []int) [][]int {
	var result [][]int            // 結果の格納
	var inner func(in, out []int) // 内部の再帰関数
	// in を捜査して、 out へ足していく
	inner = func(in, out []int) {
		if len(in) == 1 { // len(in) が 1 の時は結果を格納して終了
			result = append(result, append(out, in[0]))
		} else { // in から 1 つ取り出して out に付け足して再帰
			for i, n := range in {
				inner(delEle(in, i), append(out, n)) // 勘違いしやすいが参照渡しではないので引数は関数内部へコピー（clone_）される
			}
		}
	}
	inner(list, []int{}) // 初回呼び出しの out は空
	return result
}

func main() {
	fmt.Println(permutation([]int{1, 2, 3}))
}
```

## bit 全検索

所謂、要素の **組合せ** を求める際に利用する。  
リストに対して **ビットマスク** をあてることで実現する。  
ビット列の組合せを作ることができれば、リストの要素の組合せも求めることができる。  
例えば、 `01011` というビットマスクを `abcde` にあてると ` b de` が得られるといった具合だ。

```go
package main

import "fmt"

func main() {
	var n uint = 2                        // ビットの桁数
	for bit := 0; bit < (1 << n); bit++ { // 1 から n ビットまでの組合せ
		fmt.Printf("%b\n", bit)
	}
	fmt.Println("---")
	for bit := 1 << (n - 1); bit < (1 << n); bit++ { // n ビットの組合せ
		fmt.Printf("%b", bit)
		if (bit & (1 << (2 - 1))) > 0 { // 2 ビット目が立っているか確認
			fmt.Println(", 2nd bit flaged.")
		} else {
			fmt.Println()
		}
	}
}
```

[参考](https://qiita.com/drken/items/7c6ff2aa4d8fce1c9361#bit-%E5%85%A8%E6%8E%A2%E7%B4%A2f)

# ソート

- 挿入ソート
- バブルソート
- 選択ソート
- 安定なソート
- シェルソート

# 連立方程式

## ガウスの消去法（掃出し法）

```go
// X[n][n+1]float64
for i := 0; i < n; i++ {
	for j := 0; j < n+1; j++ { // i 行目を [i][i] で割って [i][i] を 1 にする
		if i != j {
			X[i][j] /= X[i][i]
		}
	}
	X[i][i] = 1
	for j := 0; j < n; j++ { // i 行目以外の j 行目 の i 列要素が 0 になるように引く
		if i != j {
			tmp := X[j][i]
			for k := 0; k < n+1; k++ {
				X[j][k] -= tmp * X[i][k]
			}
		}
	}
}
```

# その他

- 動的計画法（ DP : Dynamic Programming ）・メモ化
- 探索範囲を狭めるアルゴリズム

## 動的計画法（ DP : Dynamic Programming ）・メモ化

**同じ探索を 2 度としない** ようにし、 1 度計算した結果を **メモ化** して使い回す。  
計算は関数で実現し、メモは関数外の変数として覚えておく。

- https://www.slideshare.net/iwiwi/ss-3578511
- https://www.slideshare.net/KMC_JP/dp-34033161

# 参考

- [AtCoder に登録したら次にやること ～ これだけ解けば十分闘える！過去問精選 10 問 ～](https://qiita.com/drken/items/fd4e5e3630d0f5859067)
    - 最後の参考書のまとめがいい
- [アルゴリズムの簡単なまとめ](https://yukicoder.me/wiki/algorithm_summary)
    - アルゴリズム一覧を確認するのにいい
- 書籍 [プログラミングコンテスト攻略のためのアルゴリズムとデータ構造](https://book.mynavi.jp/ec/products/detail/id=35408)
    - もくじが参考になる
- 書籍 [世界で闘うプログラミング力を鍛える150問](https://www.amazon.co.jp/%E4%B8%96%E7%95%8C%E3%81%A7%E9%97%98%E3%81%86%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E5%8A%9B%E3%82%92%E9%8D%9B%E3%81%88%E3%82%8B150%E5%95%8F-%E3%83%88%E3%83%83%E3%83%97IT%E4%BC%81%E6%A5%AD%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9E%E3%81%AB%E3%81%AA%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AE%E6%9C%AC-Gayle-Laakmann-McDowell/dp/4839942390/ref=cm_cr_arp_d_product_top?ie=UTF8)
    - プログラミングによる採用テストにもふれている
- [競技プロ的なアルゴリズムのスライドのまとめ](https://yukicoder.me/wiki/slide)