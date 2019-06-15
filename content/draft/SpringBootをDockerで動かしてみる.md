---
title: SpringBootをDockerで動かしてみる
tags:
- docker
- SpringBoot
id: springboot-docker-basics
draft: true
---

SpringBoot アプリケーションを Docker で動かしてみる。

# Dockerfile

Dockerfile を SpringBoot アプリケーションプロジェクト直下に配備する。（ `pom.xml` と同階層）

```
FROM openjdk:10.0.1
VOLUME /tmp
ADD target/springboot-sample-jar.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar
```

`VOLUME /temp` は SpringBoot の Tomcat がデフォルトでワーキングディレクトリを作成するため。  
ホストマシンの `/var/lib/docker` とコンテナの `/tmp` がリンクする。

# ビルド、実行

```bash
$ docker build .
$ docker image ls
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              59fff669a3a6        7 seconds ago       882MB
openjdk             10.0.1              1d2f2353dcff        2 weeks ago         866MB
$ docker run -p 8000:8000 -d 59fff669a3a6
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
001c7ffc3e52        59fff669a3a6        "/bin/sh -c 'exec ja…"   8 seconds ago       Up 10 seconds       0.0.0.0:8000->8000/tcp   ecstatic_kirch
$ docker logs 001c7ffc3e52 # ログの確認
$ curl localhost:8000
Hello, PePeSe !
$ docker exec -it 001c7ffc3e52 bash
root@001c7ffc3e52:/# exit
$ docker container stop 001c7ffc3e52
$ docker container rm 001c7ffc3e52
```

# 参考

- [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)
- [SpringBootの開発環境をdockerでつくる](https://qiita.com/wataling/items/fa8b74fa50d80b88aea3)
- [Docker ドキュメント日本語化プロジェクト](http://docs.docker.jp/)

# おまけ（ Express を Docker で動かしてみる

## Dockerfile

場所は Express アプリプロジェクト（ express-sample ）ルート。

```
FROM node:10.5.0-alpine
ENV APP_ROOT /usr/src/sample-express

# Create app directory
WORKDIR $APP_ROOT

# Copy app
COPY . $APP_ROOT

# Install dependencies
RUN npm install -g yarn && yarn install

CMD [ "node", "app/app.js" ]
```

## ビルド、実行

```bash
$ cd express-sample
$ docker build .
$ docker run -p 3000:3000 -d xxxxxxxxxxxx
$ curl localhost:3000
$ docker container stop xxxxxxxxxxxx
$ docker container rm xxxxxxxxxxxx
```