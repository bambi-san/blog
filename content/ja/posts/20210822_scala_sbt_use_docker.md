---
title: "Gatlingをsbtから実行するコンテナを作る"
date: 2021-08-22T22:31:36+09:00
description:
draft: true
hideToc: false
enableToc: true
enableTocContent: true # 目次
tocPosition: inner
tags:
-
series:
-
categories:
-
---

## 概要

sbtコマンドを使ったGatlingの実行はjdkとsbtとscalaのインストールがそれぞれ必要のため、環境構築結構大変。

コンテナから実行できると楽なのでやってみる。

## やってみる

今日時点でgatlingの最新版は3.6.1なのでそれを利用する。

### Docker image の選定

openjdkをベースimageとしてscalaとsbtがインストールされた良さげなイメージがあるのでこれを使う。便利。。
* [scala-sbt](https://hub.docker.com/r/hseeberger/scala-sbt)

tagは`${openjdkのimage tag}_${sbtのバージョン}_${scalaのバージョン}`の命名規則があるらしい。

sbtを使うため`AdoptOpenJDK JDK 8`か`11`のバージョンを指定する。

gatling3.6.1からscalaとsbtのバージョンをそれぞれ確認する
* [scalaのバージョン(1.5.4)](https://github.com/gatling/gatling/blob/v3.6.1/build.sbt#L23)
* [sbtのバージョン(2.13.6)](https://github.com/gatling/gatling/blob/v3.6.1/project/build.properties#L1)

今回は`8u302_1.5.5_2.13.6`のTagのイメージを使うことにした。

### 構成

構成はこんな感じ。

サンプルは一から作るのめんどいのでこちら（[gatling-sbt-plugin-demo](https://github.com/gatling/gatling-sbt-plugin-demo)）をそのまま使わせていただく。
今回はそのまるっとコピペした(submoduleでもよかったかも)。

```
.
├── Dockerfile
├── README.md
├── app
│   ├── build.sbt
│   ├── project
│   ├── src
│   └── target
└── docker-compose.yml
```

### Dockefileの作成

`sbt compile`をしてimage構築すると起動するときのcompileが省略される。
（その分imageのサイズが大きくなるので一長一短な感じもする。。）

```Dockerfile
FROM hseeberger/scala-sbt:8u302_1.5.5_2.13.6

RUN mkdir /app
WORKDIR /app
COPY ./app .

RUN sbt compile

CMD ["sbt"]
```

### docker-compose.yml の作成

Dockerfileでファイルをコピーしないときはvolumes外して使う。

```yaml:docker-compose.yml
version: '3'

services:
  gatling-sample:
    build:
      context: .
    container_name: gatling-sample
    #volumes:
    #- ./:/app
    working_dir: /app
    command:
    - sh
    tty: true
```

`tty: true`にしているので`docker-compose up`すると起動したまま何もしないコンテナが生まれる。

起動しているコンテナにアタッチして色々やってみる。

```bash
$ docker exec -it gatling-sample bash

$ sbt gatling:test
```

