---
title: "2020年の振り返り"
type: "tech"
category: []
description: "2020年も残すところあと2日となりました！！お世話になった皆様、感謝です！本記事では、2020年の個人活動を軽く振り返ります。"
thumbnail: "https://res.cloudinary.com/dkerzyk09/image/upload/v1612594581/blog/01etqfnfw9h98gffzbqsv4r32w/daflgl5t40jdtiam4khv.webp"
publish: true
---

# 作ったツール/サイト
## preview-asciidoc.nvim
リポジトリ[こちら](https://github.com/shuntaka9576/preview-asciidoc.nvim)。NeovimでAsciidocをリアルタイムプレビューするプラグイン。業務でAsciidocの中でPluntUMLを使うことが多く、VSCodeのリアルタイムプレビュープラグイン(joaompinto.asciidoctor-vscode)の場合、1ファイルに20個くらいシーケンスがあると、メモリが足りなくなることがあったのと普段Neovimを使うので作りました。

メモリを使い過ぎていた理由は、エディタでカーソルを動かす度にPluntUMLの画像生成コマンドが大量に発行されているからかなーと思い、コマンド処理中はフラグを立ててフラグがある間はコマンドを発行しないようにしたら、割といい感じに使えるプラグインになった。

同じプロジェクトの同僚にもさくっと導入してもらい、好評で良かった。

詳細は[NeovimでSwaggerをリアルタイムプレビューするプラグインを作った話](https://qiita.com/hozi894/items/d9b296f923b2d394a26f)。

## preview-swagger.nvim
リポジトリは[こちら](https://github.com/shuntaka9576/preview-swagger.nvim)。NeovimでSwaggerをリアルタイムプレビューするプラグイン。前項と同じ要領で作りました。Neovimでプラグインを作るのは2020年の目標で達成できて良かった。

## kanban
リポジトリは[こちら](https://github.com/shuntaka9576/kanban)。GitHub ProjectのTUI。正直あんまり活躍してない..。GitHub Project自体のUXがいいというのが原因..。

UIをブロックしない方がUXがよく、自然とgoroutineを使った実装へ強制される。Goを使っているがgoroutine使う機会がない人はブロックしないTUIを作ってみるのが良い学習材料になるかも(?)。

詳細は[GitHubプロジェクト(カンバン)をターミナルで確認するツールを作ってみた](https://qiita.com/hozi894/items/a3cbb36c569fae649e77)。

## blog.hozi.dev
https://blog.hozi.dev を作った。Zennに影響されて、Next.js(Vercel)でISRするブログを作った。ISR時には、サーバレスAPI(TS × Lambda × DynamoDB)を呼び出している。MarkdownItで変換されたHTMLにCSSで装飾して自前のSSGを作るのは結構楽しかった(粉ミカン)。

フロント側は経験が浅くゴリ押ししてる箇所が多いので、2021年はいい感じに育てたい。

# 業務
業務ではAWSでサーバレスバックエンド、ときどきIoTという感じでした。アウトプットが少ないので、来年こそ..(1年ぶり2度目)。

# ガジェット
WFHで家で使える時間が増えた。ガジェットYoutuberをよく見ていて、今年は結構散財した。

## iQoo 5 Pro
中華メーカーVivoのスマホ。驚くべきは4000mAで120W給電可能、15分(!)で充電完了する。あと高リフレッシュレート(120Hz)対応。ヌルヌル。
![iQoo 5 Pro](https://res.cloudinary.com/dkerzyk09/image/upload/c_scale,w_448/v1612536012/blog/01etqfnfw9h98gffzbqsv4r32w/iqoo5-pro.webp =300x400)
画像の通り背面レザーもGood！OPPOのFind X2 Proも背面レザーを採用していて、結構好みで買うか迷った。正直私の利用用途だと、120Hz 120Wはオーバースペック気味。今年のAndroidは、高リフレッシュレート/高速充電に力を入れていてユーザー受けは微妙そう。スナドラ865の影響で端末価格も高かった。来年頑張って欲しい。

## iPhone SE
軽い！指紋認証強い！メインは後述のiPhone 12 Proですが、SEも売却せず風呂で使っています。

## iPhone 12 Pro
薄汚れているが気にしないで欲しい。2021年は物撮りうまくなりたいと思った
![iPhone 12 Pro](https://res.cloudinary.com/dkerzyk09/image/upload/c_scale,w_448/v1612536015/blog/01etqfnfw9h98gffzbqsv4r32w/iphone12-pro.webp =300x400)

イキッてPro買ったけど、12 miniで良かったかな。MagSafe結構気に入っています。

## Apple Watch Series 6
ブルーかっこいいね！ソロループもタイピングの邪魔にならないのでGood！
![Apple Watch Series 6](https://res.cloudinary.com/dkerzyk09/image/upload/c_scale,w_448/v1612536000/blog/01etqfnfw9h98gffzbqsv4r32w/apple-wathch-six.webp =300x400)

## AirPods Pro
WF1000-XM3とはなんだったのか

## M1 Mac book Air
Rosetta 2のお陰で私が利用するツールは問題なく動作確認が出来た。M1 Macについてはいくつか記事を書いた。
* [[小ネタ] bash,VimでApple Silicon搭載Macを判定する](https://zenn.dev/shuntaka/articles/08567c7e79d631d5c4b2)
* [Apple Silicon Mac環境構築メモ](https://zenn.dev/shuntaka/articles/6ff213d965afcb41ac4f)

## RODE Wireless GOとLavalier GO
Wireless Goは、無線マイク(発信機と受信機)。Lavalier GOはピンマイク。Youtubeで調べると死ぬほどでてくる。meetやzoomはこちらに変えて、音質の評判が良くなった。コンデンサマイクでも良かったんだけどね..。通話は照明とか画質より、とりあえず音質重要。お金をかけよう。

## AirPods Max
買ってない

# その他
* Fate stay night HFの最終章が見終わったあと凄く寂しくなった。stay nightアニメ化はこれで終わりで(もうアニメ化してない√ないし)、ワクワクできなくなるんだなーと。

# 2021年向けて
* blog.hozi.devを実験場にして、色々味見する
  * 認証を入れて、スマホからでも簡単な編集を出来るようにする
  * BEMやCSS Moduleは雰囲気理解なので、手に馴染ませたい
  * algoriaやStripeの導入(何に使うんだ!)
  * 技術ネタとはサイトのパスを分けて、ガジェットの記事を書きたい。界隈はWordPressだし、差別化できそう(?)
* メンターをやることが多いので、自己流はやめて書籍や事例から効果的なやり方を取り入れる
* 純粋にアウトプットを増やす(週1目安)
* AirPods Maxが欲しい
* マブラブオルタとプリプリ映画期待してる

# 最後に
良いお年を〜

# その他アウトプット
* [GitHubを操作するCLI「gh – The GitHub CLI tool」(Beta)を試す](https://dev.classmethod.jp/articles/shuntaka9576-gh/)
* [AWS IoT ルールで配列操作が安全に出来ず、正規表現で解決した話](https://dev.classmethod.jp/articles/shuntaka-aws-iot-regex-rule/)
* [ESLintでTypeScriptの変数の名付け規則をチェックしよう！](https://dev.classmethod.jp/articles/shuntaka9576-check-eslint/)
* [CDKでAWS Lambdaのパッケージフォーマットにコンテナイメージを指定してデプロイしてみた](https://dev.classmethod.jp/articles/shuntaka-container-image-support-lambda-cdk/)

