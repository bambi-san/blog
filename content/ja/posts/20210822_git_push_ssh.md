---
title: "[Git]httpsを使っていたリポジトリをsshで操作できるようにする"
date: 2021-08-22T23:31:36+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: true # 目次
tocPosition: inner
tags:
- git
series:
-
categories:
-
---

## 概要

いつも通り自分のリポジトリへPUSHしようと思ったらここにもHTTPS使えない余波が。。。

```
git push -n origin gh-pages
Username for 'https://github.com/igimia/blog.git': xxx
Password for 'https://bambi-san@github.com/igimia/blog.git':
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: Authentication failed for 'https://github.com/igimia/blog.git/'
```

このコマンドを打つべし。

```
$ git remote set-url origin git@${repository_path}.git
```
