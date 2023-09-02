---
title: "開発生産性可視化まわりのメモ"
type: "note"
category: []
description: "随時更新します"
thumbnail: ""
publish: true
---

:::message
随時更新します
:::

## はじめに

本資料は以下の書籍のエッセンスを引用しつつ、まとめる記事です。

* LearnとDevOpsの科学

## LearnとDevOpsの科学

### 2.2. デリバリーのリードタイム

> 「リードタイムの削減」は、リーン手法の目玉の1つである。
(中略)
> 一口に「リードタイム」と言っても2つの部分に分けられる。製品や機能の設計と検証にかかる時間と、その製品や機能を顧客に納品するための時間である。このうち、製品や機能の設計と検証の所要時間に関しては、多くの場合、所要時間の計測をいつ始めるべきか明確でない上に、変動が非常に大きいという問題がある。
(中略)
> 「コードのコミットから本番稼働までの所要時間」

Four Keysの1つであるリードタイムだが、コードのコミットから本番稼働までの所要時間で測るのが良い。これは設計や検証のタスクを入れてしまうと変動が大きくなり、ケースバイケースであまり参考にならない指標となるため。

### 2.2. バッチサイズ

> 現にトヨタ自動車ではこれを生産方式の基本要素の1つにして成果を上げた。
> * サイクルタイムの短縮とフローにおける変動の低減
> * フィードバックの高速化
> * リスクと諸経費の低減
> * 効率
> * モチベーション
> * 緊急性の認識の向上
> * コストとスケジュールの膨張の抑制

> ただ、ソフトウェアの場合に見える商品が存在しないため、バッチサイズを測定してその結果を伝えるということが難しい。
> そのため、我々は「バッチサイズ」の代わりに「デプロイの頻度」を測定基準にすることに決めた

### 2.2 MTTR

このメトリクスを調査した経緯は少し特殊な気がした

* パフォーマンスを改善したチームが、作業中にシステムの安定性を犠牲にして改善を実現したかどうかを調べたい
  * 失敗は不可避なので、「サービスをいかに迅速に復旧できるか」が鍵

書籍にはないが、他にも指標が存在する。[リンク](https://www.atlassian.com/ja/incident-management/kpis/common-metrics)。MTTRだけでも4つある()


|指標|意味|
|---|---|
|平均故障間隔(MTBF)|
|平均修復時間(MTTR)|
|平均復旧時間(MTTR)|
|平均解決時間(MTTR)|
|平均対応時間(MTTR)|
|平均確認時間(MTTA)|
|平均故障時間(MTTF)|


## まとめ

### Four Keysとその分類

|指標|測定基準|
|---|---|
|リードタイム|パフォーマンスのテンポを測定する
|デプロイ頻度|同上
|平均修復時間(MTTR)|パフォーマンスを改善したチームが、作業中にシステムの安定性を犠牲にして改善を実現したかどうか
|変更失敗率|(ソフトウェアのリリースやインフラの構成変更に伴い)本番環境での稼働に失敗したケースの発生率

### リードタイム

A. 製品や機能の設計と検証にかかる時間
B. その製品や機能を顧客に納品するための時間

`A` を含めると、それは短いことが望ましい結果つながるとは限らない。一方で`B`を改善する場合、開発から生産および確実なリリースまでのフロー高速化に繋がると言える

|選択肢|
|---|
|1時間未満<br>1日未満<br>1日から1週間<br>1週間から1ヶ月<br>1ヶ月から6ヶ月<br>6ヶ月超|


### デプロイ頻度

ここでのデプロイは、**ソフトウェアのデプロイから本番稼働(たとえばアプリのストアでの公開)まで**を指す。注意点として、コミット1つ1つリリースする継続的デプロイメントを確立した組織には当てはまらないと言われている。

|選択肢|
|---|
|オンデマンド(1日複数回)<br>1時間に1回から1日1回<br>1日1回から週1回<br>週1回から月1回<br>月1回から6ヶ月に1回<br>6ヶ月に1回よりも少ない

### 平均修復時間(MTTR)

※ リードタイム同様

|選択肢|
|---|
|1時間未満<br>1日未満<br>1日から1週間<br>1週間から1ヶ月<br>1ヶ月から6ヶ月<br>6ヶ月超|


### 指標選びの基準

開発生産性に関わる指標は、以下のような基準を満たしているのが望ましいように思えた。

* 測定が容易
* 変動が低い
  * リードタイム、デプロイ頻度いずれも変動が小さいことを指標選びの基準とされている
* 短い/長いほうが良いとわかる


