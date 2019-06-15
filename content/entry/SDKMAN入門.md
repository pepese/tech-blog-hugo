---
title: "SDKMAN入門"
date: 2017-12-15T18:04:28+09:00
slug: "sdkman-basics"
tags:
- sdkman
- java
- maven
- gradle
draft: false
archives:
- 2017/12
---

Java 系の anyenv 、 [SDKMAN](http://sdkman.io/index.html) をさわってみる。

<!--more-->

# 概要

SDKMAN を導入することで、以下のツールのバージョン管理が可能となる。

- Ant
- AsciidoctorJ
- Ceylon
- CRaSH
- Gaiden
- Glide
- Gradle
- Grails
- Griffon
- Groovy
- GroovyServ
- Java
- JBake
- Kobalt
- Kotlin
- kscript
- Lazybones
- Leiningen
- Maven
- sbt
- Scala
- Spring Boot
- Sshoogr
- Vert.x

ここでは Java と Maven のみ記載する。

# 環境設定

## SDKMAN

```sh
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

init を叩かないと `sdk` コマンドのパスが通らないため、 `~/.bash_profile` に以下をいれておく。

```
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

# 各ツールの導入

USAGE は以下。  
ツール名を「 xxxx 」とする。

```sh
$ sdk help                     # help を表示
$ sdk version                  # SDKMAN 自体のバージョン
$ sdk selfupdate force         # SDKMAN 自体のバージョンアップ
$ sdk install xxxx             # xxxx をインストール
$ sdk install xxxx [version]   # xxxx を version 指定でインストール
$ sdk uninstall xxxx [version] # xxxx を version 指定でアンインストール
$ sdk list xxxx                # xxxx の version の一覧を表示
$ sdk use xxxx [version]       # xxxx の現ターミナルの version を指定
$ sdk default xxxx [version]   # xxxx のデフォルトの version を指定
$ sdk current xxxx             # xxxx の現在の version を表示
```

[公式](http://sdkman.io/usage.html)

## Java

```sh
$ sdk install java
$ sdk list java
==========================
Available Java Versions
==========================
     9.0.1-zulu
     9.0.1-oracle
     9.0.0-zulu
 > * 8u152-zulu
     8u151-oracle
     8u144-zulu
     8u131-zulu
     7u141-zulu
     6u65-apple
==========================
+ - local version
* - installed
> - currently in use
==========================
$ sdk install java 8u151-oracle
$ sdk default java 8u151-oracle
```

`-zulu` は OpenJDK 、 `-oracle` は Oracle Java を指す。  
`/usr/libexec/java_home` もちゃんと有効だった。

## Maven

```sh
$ sdk install maven 3.5.2
$ sdk list maven
```
