---
title: "グローバルIPを元にCIDRを検索する"
date: 2020-05-10T16:19:02+09:00
description:
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tocPosition: inner
tags:
-
series:
- インフラ関連
image:
---

{{< notice warning "注意事項" >}}
この記事は`2020/05/10`にQrunchに掲載した内容です。
{{< /notice >}}

接続可能なIPアドレスをCIDRを使って制限したい時って結構ある。
調べかたを忘れないようにメモメモ。

## 調べ方

### グローバルIPを表示する
アクセスしているグローバルIPアドレスを検索する。
```bash
curl http://httpbin.org/ip
{
  "origin": "103.5.140.137"   # ←グローバルIP
}
```

### ネットワーク情報を検索する
[一般社団法人日本ネットーワークンフォメーションセンター](https://www.nic.ad.jp/ja/whois/ja-gateway.html)にアクセスし、必要事項を入力後、`検索`ボタンをクリックする。
* `検索キーワード`：グローバルIP
* `検索タイプ`：ネットワーク情報(IPアドレス)

検索結果で`a. [IPネットワークアドレス]`にCIDRが表示される。

## 参考
* グローバルIPの調べ方：https://qiita.com/ksugawara61/items/887ddd1792d7d5dabb25
