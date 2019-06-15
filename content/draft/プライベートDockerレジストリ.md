---
title: プライベートDockerレジストリ
tags:
- Docker
id: private-docker-registry
draft: true
---

Docker レジストリはコンテナ配布されているので、ローカルに立ててみる。

- https://hub.docker.com/_/registry/

```bash
$ docker pull registry:2.6.2
$ docker run -d -p 5000:5000 registry:2.6.2
## docker run -d -p 5000:5000 -v /var/opt:/var/lib/registry registry:2.6.2
```

Docker イメージをレジストリへ登録するためには名前をつける必要がある。  
`docker tag <イメージ名>:<tag> <レジストリのIP>:<ポート>/<任意のリポジトリ名>/<イメージ名>:<tag>`

```bash
$ docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
<none>                                     <none>              a1800de8996f        12 hours ago        987MB
$ docker tag a1800de8996f localhost:5000/pepese/springboot:1.0.0
$ docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
localhost:5000/pepese/springboot           1.0.0               a1800de8996f        12 hours ago        987MB
```

push する。

```bash
$ docker push localhost:5000/pepese/springboot:1.0.0
```

# 参考

- https://qiita.com/rsakao/items/617f54579278173d3c20