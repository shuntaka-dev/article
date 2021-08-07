---
title: "Cloudflare WorkersとClouflare Workers KVを環境毎にデプロイし、ドメインを割り当てる"
type: "tech"
category: []
description: "wranglerを利用して、Cloudflareサービスを環境切り替えつつデプロイ可能な設定を作ってみました"
publish: true
---

# はじめに
Cloudflare社は下記のCDN Edge上で動作するサービスを提供しています。サービスの概要は下記です。詳細は、表のリンクを参考にしてください。

|サービス名|概要|
|---|---|
|[Cloudflare Workers](https://workers.cloudflare.com/)|CDN Edge上で動作するJavaScript実行環境
|[Cloudflare Workers KV](https://www.cloudflare.com/ja-jp/products/workers-kv/)|CDN Edge上で動作するKVS


下記の流れでCloudflare Workers KVに対してRead/Write可能な環境を開発/本番コンフィグ1つで作成出来ましたので、備忘用に記事にします。
```bash
Cloudflare Pages -> Cloudflare Workers -> Cloudflare Workers KV
```
# wrangler設定

`cargo`入っているので、`crago`経由でインストールします
```bash
$ cargo install wrangler
```

下記のコマンドを実行するとブラウザが立ち上がり、簡単に初期設定が完了します
```bash
$ wrangler login
Allow Wrangler to open a page in your browser? [y/n]
y
(中略)
✨  Successfully configured. You can find your configuration file at: /Users/hoge/.wrangler/config/default.toml
```

# Cloudflare Workersの構築
下記のコマンドで、簡単にWorkersのホスティングが完了します

```bash
$ wrangler generate my-worker
$ cd my-worker
$ wrangler publish

 https://my-worker.{ユーザーID}.workers.dev
```

# 環境ごとにCloudflare WorkersとClouflare Workers KVを構築
## 設計
### 名前空間設計

開発,本番にそれぞれ、下記の表のような形でWorkersとWorkers KVを構築します。

|環境|サービス|名前空間|
|---|---|---|
|ローカル開発|Wokers KV|my-worker-dev-MY_KV_preview|
|開発|Workers|my-worker-dev|
|開発|Workers KV|my-worker-dev-MY_KV|
|本番|Workers|my-worker|
|本番|Workers KV|my-worker-MY_KV|

### Cloudflare DNS設計

|ドメイン|type|Name|Content|宛先|
|---|---|---|---|---| 
|api-dev.example.io|CNAME|api-dev|{ワーカー名+dev}.{ユーザーID}.workers.dev|開発用 Cloudflare Workers
|api.example.io|CNAME|api|{ワーカー名}.{ユーザーID}.workers.dev|本番用 Cloudflare Workers
|dev.example.io|CNAME|dev|{プロジェクト名+dev}.pages.dev|開発用 Cloudflare Pages
|example.io(CNAME flattering)|CNAME|@|{プロジェクト名}.pages.dev|開発用 Cloudflare Pages

## Cloudflare Workers KVの作成
出力されるpreview_id 1つとid 2つをメモしておいてください。後述する設定ファイルで利用します。

```bash
# ローカル開発
$ wrangler kv:namespace create "MY_KV" --env dev --preview
(中略)
kv_namespaces = [
         { binding = "MY_KV", preview_id = "xxxxid1" }
]

# 開発
$ wrangler kv:namespace create "MY_KV" --env dev
(中略)
kv_namespaces = [
         { binding = "MY_KV", id = "xxxxid2" }
]

# 本番
$ wrangler kv:namespace create "MY_KV" --env prd
(中略)
kv_namespaces = [
         { binding = "MY_KV", id = "xxxxid3" }
]

```

## wrangler.toml
wrangler.tomlをモリモリ書きます

```toml:wrangler.toml
type = "javascript"
account_id = ""
zone_id = "****" # Overview画面の右サイドバーの下部からコピペ

# ローカルプレビュー用の設定
[env.pre]
name = "my-worker-pre"
workers_dev = true # routesがない場合に必要
kv_namespaces = [
  { binding = "MY_KV", preview_id = "xxxxid1" }
]

# 開発用の設定
[env.dev]
name = "my-worker-dev"
kv_namespaces = [
  { binding = "MY_KV", id = "xxxxid2" }
]
routes = ["api-dev.example.io/*"]

# 本番用の設定
[env.prd]
name = "my-worker"
kv_namespaces = [
  { binding = "MY_KV", id = "xxxxid3" }
]
routes = ["api-dev.example.io/*"]
```

## 動作確認用のCloudflare Workersのコード

```js:index.js
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  // `kv_namespaces`を設定しているため、`MY_KV`でCloudflare Workers KVを参照可
  let value = await MY_KV.get("my-key");

  if (value === null) {
    value = new Date().getTime()
    await MY_KV.put("my-key", value)
  }

  return new Response(`${value ? value: 'nodata'}`, {
    headers: { 'content-type': 'text/plain' },
  })
}
```

## Cloudflare Workersの作成

デプロイすると自動的に作成されます。

```bash
# 開発
wrangler publish --env dev

# 本番
wrangler publish --env prd
```

## プレビュー方法
`http://127.0.0.1:8787`でローカルプレビュー出来ます。便利。

```bash
$ wrangler dev --env pre
💁  watching "./"
👂  Listening on http://127.0.0.1:8787
```

## Cloudflare CDNの設定
[Cloudflare DNS設計](#cloudflare-dns設計)参考

※ Cloudflare Pagesは別途Custom domainsの設定が必要です。嵌ったら、[Cloudflare PagesにApexドメインを設定しようとして嵌った話](https://blog.hozi.dev/hozi576/articles/01fcexkj03t5wr0hkjt37t7dcz)が少し参考になるかもしれません。


# 最後に
開発/本番CLIのコマンドで切り替えて `Cloudflare Workers` `Cloudflare Workers KV`の環境構築が出来ました。`wrangler`でDNS設定以外は完結したので、開発体験がよかったです。ただデプロイ時完璧に冪等な作りになっているわけではないので、デプロイ後に設定を確認する必要はあると思っています。

あまり新鮮味がなくてすいません。。。これから運用に向けて色々と気づいたことやTipsがあったら追記しようと思います。

# 参考資料

[Cloudflare Workers KV の初歩](https://numb86-tech.hatenablog.com/entry/2021/06/22/233701)
[Cloudflare Workers KV をキャッシュとして使う](https://numb86-tech.hatenablog.com/entry/2021/06/24/071057)


