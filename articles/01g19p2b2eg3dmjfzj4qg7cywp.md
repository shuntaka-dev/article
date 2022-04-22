---
title: "Go Conference 2022 Spring参加メモ"
type: "tech"
category: []
description: "Go Conference 2022 Spring参加メモです。進行と同時に随時更新していきます。"
thumbnail: ""
publish: true
---

:::message
本記事は随時更新致します

不備がありましたら、[こちら](https://github.com/hozi-dev/article/blob/master/articles/01g19p2b2eg3dmjfzj4qg7cywp.md)まで
:::


# はじめに
Goとしばらく離れていたが、最近作りたいものができたこともあり、変化に追いつくためにもGo Conference 2022 Springに参加した。今回はリモート開催となる。イベント詳細情報は、下記の通り。

* [Go Conference 2022 Spring](https://gocon.jp/2022spring/)公式ページ
* [connpass](https://gocon.connpass.com/event/212162/)のイベント参加ページ

# スケジュール

詳細は[スケジュール](https://gocon.jp/2022spring/schedule/#day_2022-04-23)を参照

|時刻|Track A|Track B|
|---|---|---|
|10:00|-|-|
|10:10|[人間の直感に対応させた複雑度<Cognitive Complexity>の計測ツールをgo/astで実装してみよう](https://gocon.jp/2022spring/sessions/a1-c/) [📄]()|-|
|10:30|[創業以来のPHPシステムが生み出した混沌をGoへの移行で乗り越えた話](https://gocon.jp/2022spring/sessions/a2-c/) [📄]()|-|
|11:00|[Dissecting Slices, Maps and Channels in Go](https://gocon.jp/2022spring/sessions/a3-l/) [📄]() |[ゼロから作る Protocol Buffer のパーサーとレキサー](https://gocon.jp/2022spring/sessions/b3-l/) [📄]()|-|
|11:40|[GoのGC(garbage collector)について理解する](https://gocon.jp/2022spring/sessions/a4-s/) [📄]()|[lock free な doubly linked list を実装していたらいつのまにか concurrent skip list map を実装していたでござる](https://gocon.jp/2022spring/sessions/b4-s/) [**📄**]()|
|13:10|[レガシーシステムをGoにリプレースした一年間の振り返り](https://gocon.jp/2022spring/sessions/a5-c/) [📄]()|-|
|13:30|[メタバースを支える技術 ～UGCに溢れる3D空間のリアルタイム同期を支えるGo〜](https://gocon.jp/2022spring/sessions/a6-c/) [📄]()|-|
|14:00|[高速で統一的な自動生成ツールをprotocプラグインとして実装した話](https://gocon.jp/2022spring/sessions/a7-l/) [📄]()|[Go で RDB に SQL でアクセスするためのライブラリ Kra の紹介]() [📄](https://gocon.jp/2022spring/sessions/b7-l/)|
|14:40|[database/sqlパッケージを理解する](https://gocon.jp/2022spring/sessions/a8-s/) [📄]() |[「自動コード生成ツール」を20分で作れるようになろう](https://gocon.jp/2022spring/sessions/b8-s/) [📄]()|
|15:20|[GoでAPI クライアントの実装](https://gocon.jp/2022spring/sessions/a9-s/) [📄]()|[GoらしいAPIを求める旅路 並行処理編](https://gocon.jp/2022spring/sessions/b9-s/) [📄]()|
|15:40|[testingパッケージを使ったWebアプリケーションテスト（単体テストからE2Eテストまで）](https://gocon.jp/2022spring/sessions/a10-s/) [📄]()|[HTTP Tunneling in Go](https://gocon.jp/2022spring/sessions/b10-s/) [📄]()|
|16:00|[GoとLambdaを使用した高パフォーマンスでサーバレスなマイクロサービスの開発と運用](https://gocon.jp/2022spring/sessions/a11-s/) [📄]()|[Go Module with Microservices and Monorepo: Clear Dependencies with Ease of Development](https://gocon.jp/2022spring/sessions/b11-s/) [📄]()|
|16:30|[型パラメータが使えるようになったのでLINQを実装してみた](https://gocon.jp/2022spring/sessions/a12-s/) [📄]()|[IoT with TinyGo](https://gocon.jp/2022spring/sessions/b12-s/) [📄]()|
|16:50|[Let's contribute to OSS with Go](https://gocon.jp/2022spring/sessions/a13-s/) [📄]()|[Go で始める将棋AI](https://gocon.jp/2022spring/sessions/b13-s/) [📄]()|
|17:10|[The introduction of my way to learn Go together with Go community.](https://gocon.jp/2022spring/sessions/a14-s/) [📄]()|[Motto Go Forward Goを支える文化とコミュニティ 〜なぜ我々はコミュニティにコントリビュートするのか？〜](https://gocon.jp/2022spring/sessions/b14-s/) [📄]()
|17:45|[Goの標準機能で簡易システムを低コストで作成するテクニック](https://gocon.jp/2022spring/sessions/lt1/) [📄]()|-|
|17:50|[Python製の姓名分割ライブラリをGoに移植した話](https://gocon.jp/2022spring/sessions/lt2/) [📄]()|-|
|17:55|[外部コマンドの実行を含む関数のテスト](https://gocon.jp/2022spring/sessions/lt3/) [📄]()|-|
|18:00|[大規模ゲーム開発におけるContext活用パターン](https://gocon.jp/2022spring/sessions/lt4/) [📄]()|-|
|18:05|[GoとKubernetesを用いたバッチ開発のすすめ](https://gocon.jp/2022spring/sessions/lt5/) [📄]()|-|
|18:10|[Gopher, Chrome, Automation in 5m](https://gocon.jp/2022spring/sessions/lt6/) [📄]()|-|
|18:15|[Go runtime の歩き方](https://gocon.jp/2022spring/sessions/lt7/) [📄]()|-|
|18:20|[Go言語仕様輪読会の開催を通じた振り返り](https://gocon.jp/2022spring/sessions/lt8/) [📄]()|-|


# セッション

## 人間の直感に対応させた複雑度<Cognitive Complexity>の計測ツールをgo/astで実装してみよう

[📄]()

### 感想

`Cyclomatic complexity`(循環環複雑度)は聞いたことがあったが、`Cognitive Complexity`(認知的複雑度)は知らなかった。。人間の直感近い複雑度が計測できるとのこと。
`Cognitive Complexity`を計測するため`go/ast`を使って、静的解析ツールを作るセッション。astの構造体を理解する上で参考にしたいと思いました！


## 創業以来のPHPシステムが生み出した混沌をGoへの移行で乗り越えた話

[📄]()

### 概要
CMS側をPHPからGoへ移行した話。テスト,静的解析,クラス,関数全てなしのコードのリプレースに関するセッション。
RDBにJSONのカラム(text)があり、map[string]stringや定義した構造体にパースするため、`sqlx`のコードを読み解き、取得次は`Scanner`、書き込み時は`Valuer` interfaceを実装した話。

### 感想
Goだとデータのパース(Deserialize)周りでハマることが多いイメージがあります。`sqlx`側でイレギュラーなRDBの値に対して、対応する手段があるのは便利だなと思いました！



