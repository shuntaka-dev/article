---
title: "2021年の振り返り"
type: "tech"
category: []
description: "お世話になった皆様、感謝です！本記事では、2021年の個人活動を軽く振り返ります。"
thumbnail: "https://res.cloudinary.com/dkerzyk09/image/upload/v1640324333/blog/01fqn7vgp6hcejnyhe6k0fs208/j4og57fak74bzlz2vuqb.png"
publish: false
---


# はじめに

今年も残すところ7日ですね！私は今日で仕事納めかつ、年末は引越し準備でブログをまったり書いている時間がなく、24日に書いています。

本記事では、2021年の個人活動を軽く振り返ります。

# 出来事(技術)

## 個人面

### 個人ブログ
2021年上半期はVercelとNext.js、サーバーサイドはサーバーレス構成をとった自作ブログの改善をしていました。

改善内容としては下記の項目で、ほぼZennが書きやすかったのでそれをコピーしたような機能です。。

* マークダウンパーサー、CSSの改善
* Neovimでのリアルタイムプレビューオレオレプラグインの実装
* ダークモードの実装
* プライベートな記事を認証付きで見られる仕組みの導入
* GitHub Appsを使ったリポジトリとブログの同期システム 

実際に書いたコードは、[公開](https://github.com/hozi-dev)しているので、気になる人がいるわけないと思いますが、参考になればと思います。ここら辺の話の詳細は[Next.jsとサーバレスAPIで自分のブログを作った話](https://blog.hozi.dev/hozi576/articles/01f07hctzhjcwtdq4h6ew9stk8)に書いたので、気になる方は見てみてください。

PV数は大体月500PVくらいです。まだまだですね。。
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1640317706/blog/01fqn7vgp6hcejnyhe6k0fs208/x7exp0dr4mz9cn1lv57f.png)

期間は2021/01/01 - 2021/12/23の記事別のPV内訳は、下記の通り。割と力を入れて書いた記事がPV高くて納得という感じです。

|記事タイトル|PV数|
|---|---|
|[Next.jsとサーバレスAPIで自分のブログを作った話](https://blog.hozi.dev/hozi576/articles/01f07hctzhjcwtdq4h6ew9stk8)|697|
|[GitHubとの連携手段(OAuth Apps, GitHub Apps)を整理する](https://blog.hozi.dev/hozi576/articles/01ezm5k2rt1jm6zbsewm33r0xw)|526|
|[AWS IoTで独自CA証明書を使ってMQTT通信を試してみる](https://blog.hozi.dev/hozi576/articles/01f4gnep6herhgy449er48g9c0)|422|
|[OpenSSLでJWTの作成、検証ロジックを確認する](https://blog.hozi.dev/hozi576/articles/01f4xw3pwm7tcrdswyzqsft5zs)|282|
|[npmモジュールをモノレポで管理し、煩雑なリリース業務を効率化する](https://blog.hozi.dev/hozi576/articles/01evbw029qzxavp20erstgvm5r)|269|
|[Vim/Neovimプラグイン開発サイクルをまとめてみる](https://blog.hozi.dev/hozi576/articles/01f0n3x0afc5wt54qeaz77zvw4)|257|
|[App EngineとCloud CDNでstale-while-revalidate可能な環境を構築する](https://blog.hozi.dev/hozi576/articles/01f2wwqs2jcdgc7fh8bmhnewk6)|253|
|[next/linkのpassHref属性がどのような場合に必要か整理、検証してみる](https://blog.hozi.dev/hozi576/articles/01f3qsp7vz8dtetg5cq27ercna)|217|
|[lernaでモノレポで管理したパッケージをnpmに公開する際の注意点](https://blog.hozi.dev/hozi576/articles/01ev3p1knggn1wwsg0n0e98915)|207|

クエリによっては1桁順位で掲載されることもあり、作ってよかったと思いました。
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1640319312/blog/01fqn7vgp6hcejnyhe6k0fs208/iwny4hghqzhyxib9i30j.png)

### その他
ほぼないので、ブログところで水増ししました。

強いてあげるとするなら[Neovimのinit.lua](https://github.com/shuntaka9576/dotfiles/blob/master/lua/init.lua)化とプラグインをlua製に総入れ替えしました。luaでDeepLを使った翻訳プラグインを作ったりしましたが、やりかけで年内公開まで至らずという感じです。。Neovimのluaプラグインはかなりエコシステムが充実しているように感じました！


## 仕事面
IoTサービスを2020年と引き続きやっていた。お客さんは変わらないものの、サービス自体は全く異なるもので色々と刺激がありました。
大きく変化した点としてはAWS(クラウド)側の実装ではなく、デバイスやハード側のアプリを書いている比重が多かった。(ラズパイとPythonなので組み込みではないです)

その刺激を受けて、M5Stack Core2 for AWSを購入して色々記事を書いたりしました。

* [NeovimでM5Stack Core2ファームウェア開発を快適に行う環境を作った](https://blog.hozi.dev/hozi576/articles/01ffwa0x782te58803721b1czg)
* [ESP-IDFの導入と使い心地を試す](https://blog.hozi.dev/hozi576/articles/01fg0ayqeqbf4rbfzc7gev1t0k)
* [[M5Stack Core2] PlatformIOでesp-aws-iotのサンプルsubscribe_publishを動かしてみる](https://blog.hozi.dev/hozi576/articles/01fgdc0bawyb6d34gs54vxgpg9)

今年は、個人で18記事と会社で3記事の合計21記事書きました。

# 出来事(非技術)

## 運動
5月 - 6月くらいは、平日1万/休日1万5千歩くらい歩いていました。おかげで10kgくらい痩せました。登山グッズも買ったのですが、1回大山行ったきりです。。最近はサボり気味ですが、また太ってきたら再開しよーかなーと思っています。

## ゲーム
6,7月はMHST2が面白くて、結果MHRも買ってプレイしました。
10月くらいに惰性で続けていたFGOを完全引退(キャラ全て霊基遷移済みなので復帰はしないでしょう)しました。8,9月で2部1章 - 2部6章完走して以降楽しみでしたが、未練はないです（）
結果的に原神に乗り換えました。11月開始で今ランク42で、社会人以来初くらいにハマっています。原神はいいぞ。

# 2022年向けて
ざっくりですが、2022年は下記をテーマにやっていきたいなと思います。

* `blog.hozi.dev`は他媒体と比較しても一番使いやすいので、引き続き記事を書いていく
* 今のブログとは別にWebアプリを1つを作り、運用する
  * WASMやDenoあたりを組み込めるテーマがいいな..
* ESP32やArduinoを使って生活改善ができるアプリを作る


# 最後に
通してみると手を動かすことが少ない1年だった気がします。その中で来年やりたいテーマも緩くは出来たので、生きていける気がします。

ということで、1年間お世話になりましたー。良いお年を〜。
