---
title: "TSKaigi参加レポ"
type: "tech"
category: []
description: ""
thumbnail: "https://res.cloudinary.com/dkerzyk09/image/upload/v1715312546/blog/20240511-tskaigi/yxz6mruhgmvma01ewtnv.png"
publish: false
---


## はじめに

TSKaigiに参加した。1億数年ぶりに中野に降り立った。

## スライドまとめ(随時更新)

セッション一覧は[こちら](https://tskaigi.org/talks)

|時刻|Track1|Track2|Track3
|---|---|---|---|
|10:30 ~ 11:15|Keynote: What's New in TypeScript [📄(未)]()
|11:30 ~ 12:00|フロントエンドもバックエンドもインフラも… 全てをTypeScriptで統一したらこうなった！[📄](https://speakerdeck.com/kimitashoichi/11-dang-ri-suraido)|TypeScript ASTを利用したコードジェネレーターの実装入門[📄(未)]()|Prisma ORMを2年運用して培ったノウハウを共有する[📄(未)]()
|12:10 ~ 13:10|TypeScript化の旅: Helpfeelが辿った試行錯誤と成功の道のり[📄](https://docs.google.com/presentation/d/1z1ouyBtsNFgdJysGLAKyJXgt4pEHDN3kD555Mh2UiR0/edit#slide=id.g2db43f1dcdc_0_416)|TypescriptでのContextualな構造化ロギングと社内全体への導入！[📄](https://speakerdeck.com/leveragestech/typescripttenocontextualnagou-zao-hua-rokinkutoshe-nei-quan-ti-henodao-ru)|エンジニアの技術的な意思決定を支えるADR - LayerXの活用事例[📄(未)]()
||新サービス Progate Path の演習で TypeScript を採用して見えた教材観点からの利点と課題[📄](https://speakerdeck.com/makotoshimazu/tskaigi-2024-xin-sabisu-progate-path-noyan-xi-de-typescript-wocai-yong-sitejian-etajiao-cai-guan-dian-karanoli-dian-toke-ti)|生成 AI と Cloud Workstations で始めるクラウド AI ネイティブ開発[📄(未)]()|toggle holdingsとTSあるいはTSKaigi[📄(未)]()
||PMF達成の立役者！Full TypeScript Architecture の選定背景と構成[📄(未)]()|EARTHBRAINが挑むグローバルな課題とTypeScriptの活用事例について[📄(未)]()|高まった熱量をぶつけられるコミュニティ活動のススメ[📄(未)]()
|||TypeScriptで統一したアーキテクチャ[📄(未)]()|こんなTypescriptは嫌だ[📄(未)]()
||||チームで挑むTypeScriptコードの漸進的改善[📄(未)]()
||||Ubie のプロダクト開発における技術的レバレッジポイント3選[📄(未)]()
|13:20 ~ 13:50|TypeScript 関数型バックエンド開発のリアル[📄](https://speakerdeck.com/naoya/typescript-guan-shu-xing-sutairudebatukuendokai-fa-noriaru)|TypeScriptと型のパフォーマンス[📄(未)]()|部分型の代数的模型[📄](https://yo-goto.github.io/tskaigi-slide-re/1)
|14:00 ~ 14:30|TypeScript の抽象構文木を用いた、数百を超える API の大規模リファクタリング戦略[📄(未)]()|TypeScriptから始めるVR生活[📄(未)]()|複雑なビジネスルールに挑む：正確性と効率性を両立するfp-tsのチーム活用術[📄(未)]()
|||TypeScript をパワフルに使って開発したい！[📄(未)]()|
|||TypeScriptでもLLMアプリケーション開発！LangChain.js入門[📄(未)]()|
|||TanStack Routerで型安全かつ効率的なルーティングを実現[📄(未)]()|
|14:40 ~ 15:10|Step by Stepで学ぶ、ADT(代数的データ型)、モナドからEffect-TSまで[📄(未)]()|tRPCを実務に導入して分かった旨味と苦味[📄(未)]()|Type-safety in Angular[📄(未)]()
|15:20 ~ 15:50|ハードウェアを動かすTypeScrptの世界[📄](https://speakerdeck.com/9wick/hadoueawodong-kasutypescriptnoshi-jie)|サービス開発におけるVue3とTypeScriptの親和性について[📄(未)]()|TypeScriptできると思ったのは勘違いだった件[📄(未)]()
||||Effectで作る堅牢でスケーラブルなAPIゲートウェイ[📄(未)]()
||||Introduction to Database Connection Management Patterns in TypeScript[📄(未)]()
||||Prismaでスキーマ変更を行う際のベストプラクティス[📄(未)]()
|16:40 ~ 17:10|Prettierの未来を考える[📄(未)]()|TypeScriptのパフォーマンス改善[📄(未)]()|ｽﾀｯｸﾁｬﾝ -TypeScriptで動くオープンソースロボット-[📄](https://meganetaaan.github.io/slides-tskaigi2024/)
|||ts-morphを使ってコードリプレイスとASTへのハードルを下げる！[📄(未)]()|
|||SWC Transformerから見るTypeScript関数記述ベストプラクティス[📄(未)]()|
|||TypeScriptのコード生成をつらくしないために[📄(未)]()|
|17:20 ~ 17:40|TypeScriptが学生のエンジニアコミュニティ参加を促進する[📄(未)]()|Documetation testsの恩恵[📄(未)]()
||Reactでハードウェア制御できるEdison.jsを作っている[📄(未)]()|Full TypeScriptだから実現できる世界線[📄(未)]()
||多言語化対応における TypeScript の型定義を通して開発のしやすさについて考えた[📄(未)]()|Real World Type Puzzle and Code Generation[📄(未)]()


## 参加セッションと感想

感想は簡易的なものです、、すいません。。

### Keynote

TypeScript 5.4, 5.5の話で、Narrowingの強化周りは自分のコードの書き方的にも影響がありそう。[Isolated Declarations](https://devblogs.microsoft.com/typescript/announcing-typescript-5-5-beta/#isolated-declarations)周りの話は後で見直したい。。

### TypeScript ASTを利用したコードジェネレーターの実装入門

[Himenon/openapi-typescript-code-generator](https://github.com/Himenon/openapi-typescript-code-generator)の作者の方。AST周りはGoで静的解析ツール作ったことがある程度で、TSでも簡単に出来るか気になったので、セッションを聞いた。[TypeScript AST Viewer](https://ts-ast-viewer.com/#)が便利で、ざっくりどういう構造を知るのに良いなと感じた。graphql-codegenは文字列結合という話を聞き、逆にコードジェネレーターをAST以外で扱うこともあるのかと感じた。ASTは[ts-morph](https://github.com/dsherret/ts-morph)も使ってみたい。スナップショットテストなど、テストも簡単に出来る方法が用意されてるし、思ったよりハードルが低くて良きと感じた。




## さいごに

とても楽しいセッションばかりでした！スタッフ、スポンサーの皆様ありがとうございました！
