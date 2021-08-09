---
title: "mkcertを使って証明書を作成する"
date: 2021-08-08T22:31:36+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: true # 目次
tocPosition: inner
tags:
- tls
series:
-
categories:
-
---

## 概要

ローカルでTLS1.3を利用したクライアント認証を試してみたかった。証明書の発行コマンドめんどい。
オレオレ証明書が簡単に作成できる`mkcert`というツールがあるようなので、使ってみる。その時のメモ。
（環境はmac）

https://github.com/FiloSottile/mkcert

## インストール

READMEにある通り、homebrewを使ってインストールする。

```
$ brew install mkcert
```

## mkcertを利用した証明書＆鍵の生成

はじめに、ルート認証局の秘密鍵、公開鍵、証明書を生成する。

mkcertコマンドを使って、ルート認証局の情報が保存されている場所を特定する。

```bash
mkcert -CAROOT
/Users/bambi/Library/Application Support/mkcert
```

保存されている場所に移動してルート認証局の秘密鍵、公開鍵、証明書を生成する。

```bash
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

大丈夫そう。と思ったが、`mkcert install` コマンドを実行したらエラーが発生した。

```bash
$ mkcert -install
#ERROR: failed to read the CA key: unexpected content
```

ファイルの読み込みに失敗してるもよう。
パスフレーズ設定して鍵生成したのが問題かと思ってパスフレーズ外す。

```bash
# 元々設定されていた鍵情報ファイルの中身
# (パスフレーズが設定されている状態)
$ cat rootCA-key.pem
#-----BEGIN RSA PRIVATE KEY-----
#Proc-Type: 4,ENCRYPTED
#DEK-Info: AES-256-CBC,5DE3EF8333CA5B6B1DF7FDB4DFFE2F4B

#CVlr4pnuYAN2jDgP99alsgoNZNWQz4fM6Cf12e9dXx9Dbs0CR6gon0kTbm3WsfVk
#ZFZo26HZXcMwv1vZoPc4NH1l79CGSFjMfr6j4U7aRhhXW7VFaSAlhZIrsJ1ZgFC7
#　　　　　　　　　　　(省略)
#gIPYWJXh7c1cRRV6XEqwU9+FpNFHCmD2+BBGg/yu7qOl74h61BDmGZ8jQMOkxQYN
#-----END RSA PRIVATE KEY-----

# パスフレーズ解除
$ openssl rsa -in rootCA-key.pem  -out rootCA-key.pem
#Enter pass phrase for rootCA-key.pem:
#writing RSA key

# 鍵情報ファイルの中身
# パスフレーズ設定なし
$ cat rootCA-key.pem
#-----BEGIN RSA PRIVATE KEY-----
#MIIEowIBAAKCAQEA0wQptxOhXbrBRSDzq7B9hCre7iYi4S57vsF7rbFiqwmTlSP/
#v0WEuxRxhZQav+Idf4LF50rIUI7FcXjr2zqpp414Pza6XEab1Z4QVZzcKZh/xpHq
#　　　　　　　　　　　(省略)
#W9gChkzYquDhzLPqXXDTPVLXXgPG7ElUAX9989KisUgEYASFb4en
#-----END RSA PRIVATE KEY-----
```

パスフレーズ解除して実行しても再びエラー。
エラーの内容とソースから、うまいことファイル読みにいけてないみたい。試しに環境変数にパスを設定してみたらうまくいった！
(パスフレーズの解除は関係なかったぽい)

```bash
$ export CAROOT="/Users/bambi/Library/Application\ Support/mkcert"
$ mkcert -install
#Created a new local CA 💥
#Sudo password:
#The local CA is now installed in the system trust store! ⚡️/
```

証明書の生成してみた。

```bash
$ mkcert sample.com
#Created a new certificate valid for the following names 📜
# - "sample.com"
#
#The certificate is at "./sample.com.pem" and the key at "./sample.com-key.pem" ✅
#
#It will expire on 9 November 2023 🗓

$ openssl x509 -in ./sample.com.pem -text -noout
#Certificate:
#   Data:
#       Version: 3 (0x2)
#       Serial Number:
#           26:fd:e0:20:f4:c0:03:a8:a4:e3:b3:b4:14:92:21:ca
#   Signature Algorithm: sha256WithRSAEncryption
#       Issuer: O=mkcert development CA, OU=bambi.local, CN=mkcert bambi.local
#       Validity
#           Not Before: Aug  8 18:32:13 2021 GMT
#           Not After : Nov  8 18:32:13 2023 GMT
#       Subject: O=mkcert development certificate, OU=bambi.local
```

OUなどの情報は固定で入るらしい。なるほど。




## 参考

* [Sanwa Systems Tech Blog](https://tech.sanwasystem.com/entry/2015/08/31/234131)








