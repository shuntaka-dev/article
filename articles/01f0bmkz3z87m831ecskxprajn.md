---
title: "Denoに入門する"
type: "tech"
category: []
description: ""
publish: false
---

# はじめに
[denops.vim](https://github.com/vim-denops/denops.vim)が面白そうだったので、denoに入門することにした。

# 便利リンク

* [deno公式 | deno.land](https://deno.land/)

# インストール
公式参照。

# エディタ設定

denoのバージョンを確認
```bash
$ deno --version
deno 1.8.0 (release, x86_64-apple-darwin)
v8 9.0.257.3
typescript 4.2.2
```

[coc-deno](https://github.com/fannheyward/coc-deno)をインストール
```json
// ~/.config/coc/extensions/package.json
{
  "dependencies": {
    "coc-css": ">=1.2.6",
    "coc-deno": ">=3.1.0", // ココ
    "coc-eslint": ">=1.4.5",
    "coc-flutter": ">=1.8.1",
    "coc-go": ">=0.13.3",
    "coc-html": ">=1.4.1",
    "coc-json": ">=1.3.4",
    "coc-python": ">=1.2.13",
    "coc-tsserver": ">=1.6.8",
    "coc-vimlsp": ">=0.11.1"
  }
}
```

`:Coc Command deno.initializeWorkspace`でプロジェクトの初期化
![initializeWorkspace](https://res.cloudinary.com/dkerzyk09/image/upload/v1615299357/blog/01f0bmkz3z87m831ecskxprajn/g6gt24togw6iddrggqgw.png)

tsserverが競合して、エラーになる..。
![tsserverでエラー](https://res.cloudinary.com/dkerzyk09/image/upload/v1615299048/blog/01f0bmkz3z87m831ecskxprajn/hty5eg8nzydncw89kdm2.png)

`CocList`でcoc-tsserverをdisableにすれば、消えるがdenoプロジェクトを開く度にこれやるのは辛い気がする(解決策があれば教えて欲しい
![coclistでdisable](https://res.cloudinary.com/dkerzyk09/image/upload/v1615299210/blog/01f0bmkz3z87m831ecskxprajn/lz9atvojcfm1cxbshmp8.png)

エディタ設定は終わり。コードを書いていく！

# 実行する

## Webサーバーを書く
Webサーバーの実行サンプル
```ts
import { serve } from "https://deno.land/std@0.89.0/http/server.ts";
const s = serve({ port: 8000 });
console.log("http://localhost:8000/");

for await (const req of s) {
  req.respond({ body: "Hello World\n" });
}
```

実行する
```bash
$ deno run src/webserver/http.ts
Download https://deno.land/std@0.89.0/http/server.ts
Download https://deno.land/std@0.89.0/_util/assert.ts
Download https://deno.land/std@0.89.0/async/mod.ts
Download https://deno.land/std@0.89.0/io/bufio.ts
Download https://deno.land/std@0.89.0/http/_io.ts
Download https://deno.land/std@0.89.0/async/delay.ts
Download https://deno.land/std@0.89.0/async/deferred.ts
Download https://deno.land/std@0.89.0/async/mux_async_iterator.ts
Download https://deno.land/std@0.89.0/async/pool.ts
Download https://deno.land/std@0.89.0/textproto/mod.ts
Download https://deno.land/std@0.89.0/http/http_status.ts
Download https://deno.land/std@0.89.0/bytes/mod.ts
Check file:///Users/takahashi.shunichi/repos/github.com/shuntaka9576/deno-practice/src/webserver/http.ts
error: Uncaught PermissionDenied: network access to "0.0.0.0:8000", run again with the --allow-net flag
  const listener = Deno.listen(addr);
                        ^
    at processResponse (deno:core/core.js:212:11)
    at Object.jsonOpSync (deno:core/core.js:236:12)
    at opListen (deno:runtime/js/30_net.js:18:17)
    at Object.listen (deno:runtime/js/30_net.js:184:17)
    at serve (https://deno.land/std@0.89.0/http/server.ts:304:25)
    at file:///Users/takahashi.shunichi/repos/github.com/shuntaka9576/deno-practice/src/webserver/http.ts:2:11
````

出力をみる限り、実行時にモジュールのダウンロードが走っている。ここら辺はGoに似てる。--allow-netつけてと書いてあるのでつけて実行。

```bash
$ deno run --allow-net src/webserver/http.ts
Check file:///(中略)/src/webserver/http.ts
http://localhost:8000/
```

## 主要なオプション


# 付録
`deno help`
```bash
$ deno help
deno 1.8.0
A secure JavaScript and TypeScript runtime

Docs: https://deno.land/manual
Modules: https://deno.land/std/ https://deno.land/x/
Bugs: https://github.com/denoland/deno/issues

To start the REPL:
  deno

To execute a script:
  deno run https://deno.land/std/examples/welcome.ts

To evaluate code in the shell:
  deno eval "console.log(30933 + 404)"

USAGE:
    deno [OPTIONS] [SUBCOMMAND]

OPTIONS:
    -h, --help                     Prints help information
    -L, --log-level <log-level>    Set log level [possible values: debug, info]
    -q, --quiet                    Suppress diagnostic output
        --unstable                 Enable unstable features and APIs
    -V, --version                  Prints version information

SUBCOMMANDS:
    bundle         Bundle module and dependencies into single file
    cache          Cache the dependencies
    compile        Compile the script into a self contained executable
    completions    Generate shell completions
    coverage       Print coverage reports
    doc            Show documentation for a module
    eval           Eval script
    fmt            Format source files
    help           Prints this message or the help of the given subcommand(s)
    info           Show info about cache or info related to source file
    install        Install script as an executable
    lint           Lint source files
    lsp            Start the language server
    repl           Read Eval Print Loop
    run            Run a program given a filename or url to the module. Use '-' as a filename to read from stdin.
    test           Run tests
    types          Print runtime TypeScript declarations
    upgrade        Upgrade deno executable to given version

ENVIRONMENT VARIABLES:
    DENO_AUTH_TOKENS     A semi-colon separated list of bearer tokens and
                         hostnames to use when fetching remote modules from
                         private repositories
                         (e.g. "abcde12345@deno.land;54321edcba@github.com")
    DENO_CERT            Load certificate authority from PEM encoded file
    DENO_DIR             Set the cache directory
    DENO_INSTALL_ROOT    Set deno install's output directory
                         (defaults to $HOME/.deno/bin)
    DENO_WEBGPU_TRACE    Directory to use for wgpu traces
    HTTP_PROXY           Proxy address for HTTP requests
                         (module downloads, fetch)
    HTTPS_PROXY          Proxy address for HTTPS requests
                         (module downloads, fetch)
    NO_COLOR             Set to disable color
    NO_PROXY             Comma-separated list of hosts which do not use a proxy
                         (module downloads, fetch)
```

`deno help run`
```bash
$ deno help run
deno-run
Run a program given a filename or url to the module.

By default all programs are run in sandbox without access to disk, network or
ability to spawn subprocesses.
  deno run https://deno.land/std/examples/welcome.ts

Grant all permissions:
  deno run -A https://deno.land/std/http/file_server.ts

Grant permission to read from disk and listen to network:
  deno run --allow-read --allow-net https://deno.land/std/http/file_server.ts

Grant permission to read allow-listed files from disk:
  deno run --allow-read=/etc https://deno.land/std/http/file_server.ts

Deno allows specifying the filename '-' to read the file from stdin.
  curl https://deno.land/std/examples/welcome.ts | target/debug/deno run -

USAGE:
    deno run [OPTIONS] <SCRIPT_ARG>...

OPTIONS:
    -A, --allow-all                    Allow all permissions
        --allow-env                    Allow environment access
        --allow-hrtime                 Allow high resolution time measurement
        --allow-net=<allow-net>        Allow network access
        --allow-plugin                 Allow loading plugins
        --allow-read=<allow-read>      Allow file system read access
        --allow-run                    Allow running subprocesses
        --allow-write=<allow-write>    Allow file system write access
        --cached-only                  Require that remote dependencies are already cached
        --cert <FILE>                  Load certificate authority from PEM encoded file
    -c, --config <FILE>                Load tsconfig.json configuration file
    -h, --help                         Prints help information
        --import-map <FILE>            Load import map file
        --inspect=<HOST:PORT>          Activate inspector on host:port (default: 127.0.0.1:9229)
        --inspect-brk=<HOST:PORT>      Activate inspector on host:port and break at start of user script
        --location <HREF>              Value of 'globalThis.location' used by some web APIs
        --lock <FILE>                  Check the specified lock file
        --lock-write                   Write lock file (use with --lock)
    -L, --log-level <log-level>        Set log level [possible values: debug, info]
        --no-check                     Skip type checking modules
        --no-remote                    Do not resolve remote modules
    -q, --quiet                        Suppress diagnostic output
    -r, --reload=<CACHE_BLOCKLIST>     Reload source code cache (recompile TypeScript)
        --seed <NUMBER>                Seed Math.random()
        --unstable                     Enable unstable features and APIs
        --v8-flags=<v8-flags>          Set V8 command line options (for help: --v8-flags=--help)
        --watch                        Watch for file changes and restart process automatically

ARGS:
    <SCRIPT_ARG>...    Script arg
```

