---
title: Golangの時間の扱い
tags:
- golang
- go
id: golang-time
draft: true
---

golang の時間の扱い time パッケージについてまとめる。

<!-- more -->

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"
	"strings"
	"time"
)

func main() {
	sc := bufio.NewScanner(os.Stdin)
	sc.Scan()
	in := strings.Split(sc.Text(), " ")
	a, _ := strconv.Atoi(in[0])
	b, _ := strconv.Atoi(in[1])
	c, _ := strconv.Atoi(in[2])
	sc.Scan()

	var h, m []int
	for sc.Scan() {
		in := strings.Split(sc.Text(), " ")
		in1, _ := strconv.Atoi(in[0])
		in2, _ := strconv.Atoi(in[1])
		h = append(h, in1)
		m = append(m, in2)
	}

	goal := time.Date(2018, 1, 10, 9, 0, 0, 0, time.Local)
	for i := len(m) - 1; i >= 0; i-- {
		t := time.Date(2018, 1, 10, h[i], m[i], 0, 0, time.Local)
		rt := t.Add(time.Duration(b+c) * time.Minute)
		if rt.Before(goal) {
			t = t.Add(time.Duration(-1*a) * time.Minute)
			fmt.Printf("%02d:%02d\n", t.Hour(), t.Minute())
			break
		}
	}
}
```