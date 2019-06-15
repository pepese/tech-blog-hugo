---
title: Visual Studio CodeでJava入門
tags:
- VS Code
- Java
id: vs-code-java-basics
draft: true
---

Eclipse に疲れた筆者が VS Code で久々に Java いじる。

# まずは動かしてみる

```bash
$ mvn archetype:generate \
 -DarchetypeArtifactId=maven-archetype-quickstart \
 -DgroupId=com.pepese.sample\
 -DartifactId=sample-app \
 -Dversion=0.0.1-SNAPSHOT
$ cd sample-app/
$ rm -rf src/test
$ code .
```

`pom.xml` に以下を追加。（ Java バージョンやエンコーディング、 FatJar 作成の設定）

```xml
  <properties>
    <encoding>UTF-8</encoding>
    <jdk.version>10</jdk.version>
    <project.build.sourceEncoding>${encoding}</project.build.sourceEncoding>
    <project.reporting.outputEncoding>${encoding}</project.reporting.outputEncoding>
    <maven.compiler.target>${jdk.version}</maven.compiler.target>
    <maven.compiler.source>${jdk.version}</maven.compiler.source>
    <locales>ja</locales>
  </properties>
  <!-- 省略 -->
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.1.1</version>
        <configuration>
            <createDependencyReducedPom>false</createDependencyReducedPom>
        </configuration>
        <executions>
            <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

以下を実行。

```bash
$ mvn package
$ java -cp target/sample-app-0.0.1-SNAPSHOT.jar com.pepese.sample.App
Hello World!
```

# VS Code でデバッグ実行できるようにする

VS Code に `Java Extension Pack` を導入すると以下の手順で使用可能。

1. `Command + Shift D` でデバッグ画面が開く
2. デバッグ画面サイドメニュー上の歯車印をクリックして Java を選択すると `.vscode/launch.json` が開く
3. 実行したいコードを開いて、行番号の左側をクリックするとブレークポイントを設定できる
4. デバッグ画面サイドメニュー上で実行したコードを選択して再生ボタンをクリックするとデバッグ実行できる