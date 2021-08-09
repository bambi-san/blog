---
title: "flutter起動時エラー対応（Null Safety非対応の対応）"
date: 2021-07-18T17:31:36+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: true # 目次
tocPosition: inner
tags:
- Flutter
series:
-
categories:
- アプリ開発
---

## 概要とやったこと

flutterのState管理について調べていて、providerを使おうと`package:provider/provider.dart`をimportしたら起動時にエラーが発生して起動できなくなった。

↓発生したエラー。

```
Launching lib/main.dart on Chrome in debug mode...
lib/main.dart:1
Error: Cannot run with sound null safety, because the following dependencies
don't support null safety:

- package:provider
2

For solutions, see https://dart.dev/go/unsound-null-safety
```

自分の書いたコードが悪かったのかと思ったが、そうではないらしい。正直今の時点ではよくわかっていないが、
flutterのパッケージにはNull Safetyに対応しているパッケージと非対応のパッケージがあるらしく、非対応のパッケージを利用するとこのようなエラーが出るらしい。

`--no-sound-null-safety`オプションをつけて起動すると正しく起動できる

vscodeを使っている場合はlaunch.jsonのargsに設定すれば良い

```
            "args": [
                "--no-sound-null-safety",
            ]
```
