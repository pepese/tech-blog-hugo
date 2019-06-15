---
title: Golang によるデザインパターン
tags:
- golang
id: golang-design-pattern
draft: true
---

所謂、GoFの23個のデザインパターン。

- 参考
    - https://github.com/monochromegane/go_design_pattern
    - http://www.techscore.com/tech/DesignPattern/index.html/

以下。

- 生成に関するパターン
- 構造に関するパターン
- 振る舞いに関するパターン

<!-- more -->

# 生成に関するパターン

- Abstract Factory
    - 例えるなら、車オブジェクトの作成を一括請負してくれる工場クラス
    - `car.addHandle(x)` のように作成すると、車オブジェクトに自転車のハンドルをつけてしまったりする
- Builder
- Factory Method
- Prototype
- Singleton

## Abstract Factory

# 構造に関するパターン

- Adapter
- Bridge
- Composite
- Decorator
- Facade
- Flyweight
- Proxy

# 振る舞いに関するパターン

- Chain of Responsibility
- Command
- Interpreter
- Iterator
- Mediator
- Memento
- Observer
- State
- Strategy
- Template Method
- Visitor