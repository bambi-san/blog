---
title: "自己認証局と自己署名証明書を作る"
date: 2021-08-22T15:31:36+09:00
description:
draft: false
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

やりたかったこと
* 自己認証局を作成し自己署名証明書を作成する
* 中間認証局を作成し中間証明書を作成する
* サーバ証明書を作成する
* クライアント証明書を作成する

## やってみる

### 事前準備

ローカルの設定ファイルいじりたくなかったのでコンテナ上でできるようにした。
```Dockerfile
FROM alpine

RUN apk upgrade && apk add --update bash openssl
```

```yaml:docker-compose.yml
version: '3'

services:
  cert-generate:
    build:
      context: .
    container_name: cert
    volumes:
    - ./tls:/etc/tls
    tty: true
```

### 自己認証局と自己署名証明書を作成する

```bash
# 作業ディレクトリの作成
$ mkdir rootCA
$ cd rootCA

# 秘密鍵の生成
$ openssl genrsa -aes256 -out ./rootCA-key.pem 2048
#Generating RSA private key, 2048 bit long modulus
#................................................................................................+++
#.+++
#e is 65537 (0x10001)
#Enter pass phrase for ./rootCA-key.pem:
#Verifying - Enter pass phrase for ./rootCA-key.pem:

# 証明書要求(csr)の生成
$ openssl req -new -key ./rootCA-key.pem -subj "/C=JP/ST=Tokyo/L=Minato-ku/O=anypalette/OU=softwareDev/CN=common name/" -out ./rootCA.csr
#Enter pass phrase for ./rootCA-key.pem:

# 証明書の生成
$ openssl x509 -days 365 -in ./rootCA.csr -req -signkey ./rootCA-key.pem -out ./rootCA.pem
#Signature ok
#subject=/C=JP/ST=Tokyo/L=Minato-ku/O=anypalette/OU=softwareDev/CN=common name
#Getting Private key
#Enter pass phrase for ./rootCA-key.pem:

# 生成した証明書の中身を確認する
$ openssl x509 -in ./rootCA.pem -text -noout
#Certificate:
#   Data:
#       Version: 1 (0x0)
#       Serial Number: 10563784493868072939 (0x929a1a0720c93feb)
#   Signature Algorithm: sha1WithRSAEncryption
#       Issuer: C=JP, ST=Tokyo, L=Minato-ku, O=anypalette, OU=softwareDev, CN=common name
#       Validity
#           Not Before: Aug  8 17:38:25 2021 GMT
#           Not After : Aug  8 17:38:25 2022 GMT
#       Subject: C=JP, ST=Tokyo, L=Minato-ku, O=anypalette, OU=softwareDev, CN=common name
#       Subject Public Key Info:
#           Public Key Algorithm: rsaEncryption
#               Public-Key: (2048 bit)
#               Modulus:
#                   00:d3:04:29:b7:13:a1:5d:ba:c1:45:20:f3:ab:b0:
#                   （省略）
#                   38:5c:e7:b3:87:09:44:71:76:ed:18:2e:ae:ac:43:
#                   39:85
#               Exponent: 65537 (0x10001)
#   Signature Algorithm: sha1WithRSAEncryption
#        49:46:65:48:a4:ed:cd:d2:3c:98:85:c5:15:72:3e:30:41:b6:
#        （省略）
#        ec:ec:91:23
```

### 中間認証局と中間証明書を作成する

```bash
# openssl.cnfをコピーする
$ cp -a /etc/ssl/openssl.cnf /etc/tls/
$ cd /etc/tls/
$ cp -a openssl.cnf mid_openssl.cnf

# openssl_mid.cnf の一部を書き換える
$ vi mid_openssl.cnf
# [ CA_default ]
#
# dir             = /etc/tls/midCA

# 秘密鍵と証明書要求(csr)の生成
$ openssl req -new -newkey rsa:2048 \
  -subj "/C=JP/ST=Tokyo/L=Minato-ku/O=anypalette-mid/OU=mid-ou/CN=mid-cn/" \
  -sha256 -days 365 \
  -config /etc/tls/openssl_mid.cnf \
  -keyout /etc/tls/midCA/midCA-key.pem \
  -out /etc/tls/midCA/midCA.csr

# ルート認証局（自己認証局）で中間認証局の証明書要求(csr)に署名する
$ openssl ca -policy policy_anything \
  -days 365 \
  -config /etc/tls/openssl_root.cnf  \
  -in /etc/tls/midCA/midCA.csr \
  -out /etc/tls/midCA/midCA.pem
```

署名でエラーが出た場合の対処方法。
ファイルが読めないエラーが発生するときは読み込んでいる設定ファイル（openssl_root.cnf）のパスを修正する。
今回の例では以下を修正している。

```ini
# /etc/tls/openssl_root.cnf

[ CA_default ]
dir             = /etc/tls/rootCA       # <- パス修正
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several certs with same subject.
new_certs_dir   = $dir/newcerts         # default place for new certs.

certificate     = $dir/rootCA.pem       # <- パス修正
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/rootCA-key.pem   # <- パス修正

```

また、署名する際事前にファイルがないと怒られるのでない場合はそれぞれ作成する。
```bash
# インデックスファイルの生成
$ touch rootCA/index.txt

# シリアル番号のファイル生成
# 末尾に改行コードがないとエラーが発生するため注意する
$ echo 01 > /etc/tls/rootCA/serial
```

### サーバ証明書を作成する

サーバの秘密鍵と証明書要求(csr)を生成し、中間認証局で署名する。

```bash
# 事前準備(cnfファイルの編集)
$ vi openssl_server.cnf
# [ usr_cert ]
# basicConstraints = CA:FALSE
# nsCertType =  server

# 秘密鍵と証明書要求(csr)の生成
$ openssl req -new -newkey rsa:2048 \
  -subj "/C=JP/ST=Tokyo/L=Minato-ku/O=anypalette-mid/OU=mid-ou/CN=mid-cn/" \
  -sha256 -days 365 \
  -config /etc/tls/openssl_server.cnf \
  -keyout /etc/tls/server/server-key.pem \
  -out /etc/tls/server/server.csr

$ touch server/index.txt
$ echo 01 > server/serial

# 中間認証局で署名する
$ touch server/index.txt
$ echo 01 > /etc/tls/server/serial
$ openssl ca -md sha256 \
  -config ./openssl_server.cnf \
  -cert midCA/midCA.pem \
  -keyfile midCA/midCA-key.pem \
  -out server/server.crt \
  -infiles server/server.csr
```

### クライアント証明書を作成する

```bash
# 事前準備
$ vi openssl_client.cnf
# [ usr_cert ]
# basicConstraints = CA:FALSE
# nsCertType =  client, email, objsign

# 秘密鍵と証明書要求(csr)の生成
$ openssl req -new -newkey rsa:2048 \
  -subj "/C=JP/ST=Tokyo/L=Minato-ku/O=anypalette-client/OU=client-ou/CN=bambi/" \
  -sha256 -days 365 \
  -config /etc/tls/openssl_client.cnf \
  -keyout /etc/tls/client/client-key.pem \
  -out /etc/tls/client/client.csr

$ touch client/index.txt
$ echo 01 > client/serial

# 証明書の生成
$ openssl ca -md sha256 \
  -policy policy_anything \
  -config ./openssl_client.cnf \
  -cert midCA/midCA.pem \
  -keyfile midCA/midCA-key.pem \
  -out client/client.crt \
  -infiles client/client.csr
```

TLS1.3の場合、RSAだけでなくECDSAに対応した証明書も必要になる。コマンドはこちら。
```
openssl ecparam -genkey -name prime256v1 -out ecdsa.key
```

### 所感

コンテナから必要な設定入れて証明書作成するのが楽だった。
微妙に面倒なのでcliとかにしたい。。。

## 参考
* [【SSLメモ】自己署名証明書の作成コマンド](https://qiita.com/nagait84/items/99516c2eee4279564ca5)
* [Sanwa Systems Tech Blog](https://tech.sanwasystem.com/entry/2015/08/31/234131)



