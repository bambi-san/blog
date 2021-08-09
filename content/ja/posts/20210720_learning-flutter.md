---
title: "flutter使ってみる"
date: 2021-07-20T17:31:36+09:00
description:
draft: true
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

## 概要

スマホアプリ開発してみたくて、Flutterを使ってみた。
開発時に参考にすると良いサイトとか、環境構築時に設定しておくと便利なこととか、メモ

## 参考にしたサイトとか

こちらのページを参考にした。
環境構築からよくあるTodoアプリの作り方まで非常に丁寧に書かれている。
ものすごく参考にさせていただいた。
https://www.flutter-study.dev/introduction/about-flutter

https://dartpad.dartlang.org/?id=0f1ec306c6e5e80a61220191d2536010&null_safety=true


## 三目ならべを作ってみる

チュートリアルのTodoアプリを試してみてなんとなくつかめたので、三目並べアプリを作ってみる

### 1. ゲームボードを作成する

初めに、三目並べで利用するための3✖️3のマス目を用意する。
列と行どちらから作ればいいのかよくわからず、試しに列から作成することにする

最終的にこういう感じにしたい。
![learning flutter](/images/posts/20210717/learning-flutter_01.png)

```dart:main.dart
import 'package:flutter/material.dart';

void main() {
  // 最初に表示するWidget
  runApp(TicTacToeApp());
}

class TicTacToeApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // アプリ名
      title: 'ticktacktoe app',
      theme: ThemeData(
          // テーマカラー
          primarySwatch: Colors.green),
      // リスト一覧画面を表示
      home: GameBoardPage(),
    );
  }
}

class GameBoardPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('三目並べ'),
        centerTitle: true,
      ),
      body: Center(
        // 正方形を横に並べる
        child: Column(
          // 画面の中心に配置する
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Container(
              height: 100,
              width: 100,
              color: Colors.orange,
              child: Text(
                '1行目',
                style: Theme.of(context).textTheme.display1,
              ),
            ),
            Container(
              height: 100,
              width: 100,
              color: Colors.green,
              child: Text(
                '２行目',
                style: Theme.of(context).textTheme.display1,
              ),
            ),
            Container(
              height: 100,
              width: 100,
              color: Colors.blue,
              child: Text(
                '3行目',
                style: Theme.of(context).textTheme.display1,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

縦に３行できたので、１行目を３分割するようにやってみる。
childにRowを入れればできそう。

```dart
// Copyright 2021 The Flutter team. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.

import 'package:flutter/material.dart';

void main() {
  // 最初に表示するWidget
  runApp(TicTacToeApp());
}

class TicTacToeApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // アプリ名
      title: 'ticktacktoe app',
      theme: ThemeData(
          // テーマカラー
          primarySwatch: Colors.green),
      // リスト一覧画面を表示
      home: GameBoardPage(),
    );
  }
}

class GameBoardPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('三目並べ'),
        centerTitle: true,
      ),
      body: Center(
        // 正方形を横に並べる
        child: Column(
          // 画面の中心に配置する
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Container(
              height: 100,
              width: 300,
              color: Colors.orange,
              child: Row(children: <Widget>[
                Container(
                  height: 100,
                  width: 100,
                  color: Colors.yellow,
                  child: Text(
                    '1',
                    style: Theme.of(context).textTheme.display1,
                  ),
                ),
                Container(
                  height: 100,
                  width: 100,
                  color: Colors.green,
                  child: Text(
                    '2',
                    style: Theme.of(context).textTheme.display1,
                  ),
                ),
                Container(
                  height: 100,
                  width: 100,
                  color: Colors.pink,
                  child: Text(
                    '3',
                    style: Theme.of(context).textTheme.display1,
                  ),
                )
              ]),
            ),
            Container(
              height: 100,
              width: 100,
              color: Colors.green,
              child: Text(
                '２行目',
                style: Theme.of(context).textTheme.display1,
              ),
            ),
            Container(
              height: 100,
              width: 100,
              color: Colors.blue,
              child: Text(
                '3行目',
                style: Theme.of(context).textTheme.display1,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

```

できた。
縦︎100✖︎横100 の正方形を表現しているContainerが冗長的な書き方でみづらいので部品に分け、１〜９まで番号をつけて表示させてみる
```dart

class GameBoardPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('三目並べ'),
        centerTitle: true,
      ),
      body: Center(
        // 正方形を横に並べる
        child: Column(
          // 画面の中心に配置する
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Container(
              height: 100,
              width: 300,
              child: Row(children: <Widget>[
                Square("1"),
                Square("2"),
                Square("3"),
              ]),
            ),
            Container(
              height: 100,
              width: 300,
              child: Row(children: <Widget>[
                Square("4"),
                Square("5"),
                Square("6"),
              ]),
            ),
            Container(
              height: 100,
              width: 300,
              child: Row(children: <Widget>[
                Square("7"),
                Square("8"),
                Square("9"),
              ]),
            ),
          ],
        ),
      ),
    );
  }
}

class Square extends StatelessWidget {
  final String mes;
  Square(this.mes);

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () { print(mes); },
      child: Container(
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey, width: 2),
      ),
      height: 100,
      width: 100,
      // 中央揃え
      alignment: Alignment.center,
      child: Text(
        mes,
        style: Theme.of(context).textTheme.headline2,
      ),
    ));
  }
}
```

![learning flutter 02](/images/posts/20210717/learning-flutter_02.png)

なんか長くなりそうなので記事にするのは一度ここまでとする。
昔趣味でReactをやっていたのだけれど、なんとなく似ている感がある（大体忘れてしまっていたけど。。）。
つまづきながら進んでるので忘れないようにそれぞれ短く記事にすることにする。

## 参考URL
* https://www.flutter-study.dev/introduction/about-flutter
