---
title: "flutter使ってみる"
date: 2021-07-18T17:31:36+09:00
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

## 開発するときに覚えておきたい便利なこと

### コードの自動整形




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







## 参考URL
* https://www.flutter-study.dev/introduction/about-flutter
