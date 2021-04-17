---
title: "lernaでモノレポで管理したパッケージをnpmに公開する際の注意点"
type: "tech"
category: []
description: "lernaのコマンド周りで、少し嵌ったのでメモを書きました！"
publish: true
---

# はじめに
自作ブログを作るにあたり、複数のパッケージをnpmで管理したい要件があり、[lerna](https://github.com/lerna/lerna)が良さそうだったので試してみた。

具体的に言うとプレビュー用のNeovimプラグインでフロントエンドのCSSとマークダウンパーサーを共通化して利用したかった。

lernaでモノレポ構成を作るまでの手順は、[lernaを使ってmonorepoなリポジトリを作ってみた](https://qiita.com/hisasann/items/929b6702df1d6e871ce7)を見つつ出来たが、npmへ公開する過程でいくつか覚えておきたいことがあったため、記事にする。


# lernaオプションで覚えておきたいこと
特に使うであろうオプションとその注意点を下記に示す。

## lerna bootstrap
packages配下の各パッケージの依存関係をリンクするオプションで、packages配下にnpm_modulesが作成される。ビルド前には、実行する必要がありそう(基本的にルートのnode_modulesにインストールされるので必要ないケースもありそうです..)

## lerna clean
`lerna bootstrap`の逆。依存関係を解消するオプジョンで、packages配下のnpm_moudlesが削除される。ビルドにうまくいかないときには、一回本コマンドを実行するといいかもしれない。

## lerna run <script>
`yarn run <script>`を各パッケージで実行するオプション。例えば各パッケージで`yarn build`を実行したい場合は、`lerna run build`を実行すると良い。モノレポらしい便利なコマンド。

## lerna run build -- <subcommand>
個人的に便利に感じたコマンド。モノレポ配下で`yarn build`に設定したnpm scriptsにサブコマンドを渡せる。`--`が必要になるので注意。
**packagesの各リポジトリで変更が検知されたら、各リポジトリで設定されたyarn build <subcommand>を実行するみたいなことが可能**。

このコマンドが効いてくるのはwatchオプションをつけたとき、どのリポジトリで編集をかけてもすぐビルド資材に反映されるため、**リポジトリを跨いだプレビューがしやすい**
```bash
$ tree
packages
├── css     # yarn buildに `node-sass ./src/index.scss ./build/index.css --output-style compressed`を設定
└── parser  # yarn buildに `tsc`を設定

$ lerna run build -- --watch
info cli using local version of lerna
lerna notice cli v3.22.1
lerna info versioning independent
lerna info Executing command in 2 packages: "yarn run build --watch"

# 変更を検知すると下記が並列実行される
# packages
# ├── css     # node-sass ./src/index.scss ./build/index.css --output-style compressed --watch
# └── parser  # tsc --watch
```

## lerna publish

注意点を結論から言うと、`lerna publish`する前にnpmにログインしていることを確認する。しないと余計な作業をする必要がある。
```bash
npm whoami
lerna publish
```


### 失敗例
実行した結果から推測した`lerna publish`の挙動は下記の通り。

1. gitのHEADがrelase済みかチェック(release済みの場合リリース済みとして正常終了)
2. リリースバージョンをインタラクティブに選択
3. publishするパッケージを選択
4. `2.`のバージョンで`git tag`と`git push --tags`が実行される
5. npmへpublish


初回実行時、npmにログインしていなかったため5で失敗。lernaは4の結果をロールバックしてくれないため、後述の手順が必要だった。
```bash
$ lerna publish
info cli using local version of lerna
lerna notice cli v3.22.1
lerna info current version 0.0.0
lerna info Assuming all packages changed # <-- 1.
? Select a new version (currently 0.0.0) Patch (0.0.1) # <-- 2.

Changes:
 - hozi-dev-content-css: 0.0.0 => 0.0.1
 - hozi-dev-markdown-to-html: 0.0.0 => 0.0.1

? Are you sure you want to publish these packages? Yes # <-- 3.
lerna info execute Skipping releases
lerna info git Pushing tags... # <-- 4.
lerna info publish Publishing packages to npm... # <-- 5
lerna info Verifying npm credentials
lerna http fetch GET 401 https://registry.npmjs.org/-/npm/v1/user 326ms
Unable to authenticate, need: Basic, Bearer
lerna ERR! EWHOAMI Authentication error. Use `npm whoami` to troubleshoot.
```

タグを削除してから、再度lerna publishを実行する
```bash
$ git tag -d ${tag_name}
$ git push origin :refs/tags/${tag_name}
$ lerna publish
info cli using local version of lerna
lerna notice cli v3.22.1
lerna info current version 0.0.0
lerna info Assuming all packages changed
? Select a new version (currently 0.0.0) Patch (0.0.1)

Changes:
 - hozi-dev-content-css: 0.0.0 => 0.0.1
 - hozi-dev-markdown-to-html: 0.0.0 => 0.0.1

? Are you sure you want to publish these packages? Yes
lerna info execute Skipping releases
lerna info git Pushing tags...
lerna info publish Publishing packages to npm...
lerna info Verifying npm credentials
lerna http fetch GET 200 https://registry.npmjs.org/-/npm/v1/user 327ms
lerna http fetch GET 200 https://registry.npmjs.org/-/org/shuntaka9576/package?format=cli 645ms
lerna info Checking two-factor auth mode
lerna http fetch GET 200 https://registry.npmjs.org/-/npm/v1/user 571ms
lerna WARN ENOLICENSE Packages hozi-dev-content-css and hozi-dev-markdown-to-html are missing a license.
lerna WARN ENOLICENSE One way to fix this is to add a LICENSE.md file to the root of this repository.
lerna WARN ENOLICENSE See https://choosealicense.com for additional guidance.
lerna success published hozi-dev-markdown-to-html 0.0.1
lerna notice
lerna notice 📦  hozi-dev-markdown-to-html@0.0.1
lerna notice === Tarball Contents ===
lerna notice 105B lib/hozi-dev-markdown-to-html.js
lerna notice 472B package.json
lerna notice 164B README.md
lerna notice === Tarball Details ===
lerna notice name:          hozi-dev-markdown-to-html
lerna notice version:       0.0.1
lerna notice filename:      hozi-dev-markdown-to-html-0.0.1.tgz
lerna notice package size:  569 B
lerna notice unpacked size: 741 B
lerna notice shasum:        feb61de2cb99def286a4e28b131eac14ce41d2c8
lerna notice integrity:     sha512-tNnjHFnMQ1OHi[...]SuSLUNknfhY4w==
lerna notice total files:   3
lerna notice
lerna http fetch PUT 200 https://registry.npmjs.org/hozi-dev-markdown-to-html 3819ms
lerna success published hozi-dev-content-css 0.0.1
lerna notice
lerna notice 📦  hozi-dev-content-css@0.0.1
lerna notice === Tarball Contents ===
lerna notice 5.3kB lib/index.css
lerna notice 554B  package.json
lerna notice 150B  README.md
lerna notice === Tarball Details ===
lerna notice name:          hozi-dev-content-css
lerna notice version:       0.0.1
lerna notice filename:      hozi-dev-content-css-0.0.1.tgz
lerna notice package size:  1.8 kB
lerna notice unpacked size: 6.0 kB
lerna notice shasum:        6ab3f42394eac4e26b830177b986de10b59db266
lerna notice integrity:     sha512-E+P09Vzx9vatL[...]mppCKdr2S70nw==
lerna notice total files:   3
lerna notice
lerna http fetch PUT 200 https://registry.npmjs.org/hozi-dev-content-css 4114ms
Successfully published:
 - hozi-dev-content-css@0.0.1
 - hozi-dev-markdown-to-html@0.0.1
lerna success published 2 packages
```

成功！以上！

# 最後に
`lerna`を使ってみた感じ非常に便利！軽量なパッケージを複数それぞれリポジトリを分けて管理・リリースするのは非常に手間がかかるしやりたくないので、助かった!

