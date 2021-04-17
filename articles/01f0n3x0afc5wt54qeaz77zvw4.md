---
title: "Neovimのプラグイン開発サイクルをまとめてみる"
type: "tech"
category: []
description: "denops.vimでプラグイン開発するにあたり、初心者なりに開発サイクルを考えてみました。"
publish: true
---

# はじめに
[denops.vim](https://zenn.dev/lambdalisue/articles/b4a31fba0b1ce95104c9)が面白そうで触り始めた。その過程で私自身プラグインを作ったことはあるものの、改めてプラグイン開発者はどんなサイクルで動作確認しているのかと疑問に感じた。なので自分で開発するならこうするしかないなーと思うサイクルを書くことにした。正直あまり変わったことはしていない。。


# 前提
1. Neovimを利用
2. denops.vimを利用したプラグイン開発
3. パッケージマネージャーはdeinを利用
4. `~/repos`配下にリポジトリをghqで管理

# 実現したいこと
* 開発で利用するNeovimの設定に影響を与えないこと
* ghq管理配下でソースの管理が行えること

# Neovimの環境設定
|設定|開発側|プラグイン動作確認側|
|---|---|---|
|init.vim|~/.config/nvim/init.vim|`nvim -u`で任意指定|
|deinのラインタイム|~/.cache/dein/repos/github.com/Shougo/dein.vim|~/repos/github.com/Shougo/dein.vim|
|プラグインルート|~/.cache/dein/repos|~/repos|

プラグインルートが大事で、ghq getでcloneしたリポジトリをプラグインとしてそのまま利用可能
開発も~/reposで行うため、開発->動作確認->リポジトリへpushの流れで開発可能


# 最低限のinit.vim

## deinを利用する場合

動作確認用のvimがデフォルト設定だと使いにくため、後述する方法よりこちらがおすすめ。
```vim
set runtimepath+=~/repos/github.com/Shougo/dein.vim

let s:dein_dir = expand('~')

call dein#begin(s:dein_dir)

call dein#add('vim-denops/denops.vim') " エコシステムプラグイン
call dein#add('shuntaka9576/dps-helloworld') " 動作確認対象のプラグイン
" /* 開発に必要なプラグインを追記する */
call dein#end()

" /* 開発に必要なプラグイン設定やVimのキーバインド設定を追記する */

" cacheを削除(開発中のプラグインの更新)
call dein#recache_runtimepath()
```

## deinを利用しない

```vim
set runtimepath+=~/repos/github.com/vim-denops/denops.vim " エコシステムプラグイン
set runtimepath+=~/repos/github.com/shuntaka9576/dps-helloworld " 動作確認対象のプラグイン
```

`nvim -u`でinit.vimを指定して実行する
```bash
nvim -u ~/repos/github.com/shuntaka9576/init.vim/init.vim
```

# 開発の流れ

## 最初にやること
* [#最低限のinit.vim](#最低限のinit.vim)に開発するプラグインのパスを追記
* coc.nvimを利用している場合は、denoの設定ファイルを作成

```json:.vim/coc-settings.json
{
  "deno.enable": true,
  "deno.lint": false,
  "deno.unstable": true,
  "tsserver.enable": false
}
```

## 開発中
1. プラグインを改修する
2. `nvim -u ~/repos/github.com/shuntaka9576/init.vim/init.vim`でnvimを起動し動作確認(CLIのスニペットに登録しておく)
3. プラグインを改修する
4. 2で起動したnvimを終了し、再起動して動作確認

nvimを毎回再起動するのは非効率のように見えて、sourceでinit.vimを読み込み直してみたが、プラグインの更新内容は反映されなかった。。。

# 最後に

本記事に関して誤っている点やもっと良い方法があれば[issue](https://github.com/hozi-dev/article/issues)で受け付けています！以上。
