---
title: "SPAにTwitter OAuth 2.0を組み込む際の雑メモ"
type: "note"
category: []
description: "SPAにTwitter OAuth2.0を組み込む際の雑メモ"
thumbnail: ""
publish: true
---

# 構成

方法はざっくり以下の3つ(雑)。3がいいかなぁと

1. Authraizationヘッダー + localStorage
  => XSS怖い
2. Authraizationヘッダー + Cookie(not HTTP Only属性)
  => CSRF怖いのでAuthraizationヘッダーを使うと、CookieにHTTP only属性は使えない。。(XSS怖い)
3. Cookie認証 + Cookie(HTTP Only属性+Same site(Lax,Strict))
  => XSS,CSRF対策出来てそう


@startuml
actor "User" as user
box "Cloudflare Pages" #dee6ee
  participant "トップページ" as clPages
end box
box "CDN Edge Server" #e6e6fa
  participant "Cloudflare Functions" as clFunctions
  database "Cloudflare KV" as clKv
end box
box "Twitter" #2bc4ff
  participant "Twitter認可画面" as twAuth
  participant "Twitter認可サーバー" as twServer
  participant "TwitterAPI" as twApi
end box
activate clPages
activate user
user->clPages: login with Twitterを押下
clPages->clPages: code_verifier生成
clPages->clPages: code_verifierをcookieに保存(※1)
clPages->clPages: code_verifierからcode_challenge生成
clPages->twAuth: code_challenge
activate twAuth
twAuth->twAuth: Present authorization dialog
twAuth-->user
deactivate twAuth
user->user: Authraize or declines app(※2)
user->twServer
activate twServer
twServer-->twServer: Redirects to your app's callback URL
twServer-->clPages: authraization code
deactivate twServer
clPages->clPages: cookieからcode_verifierを取り出し
clPages->clFunctions: code, code_verifier
activate clFunctions
clFunctions->twServer: code, code_verifier
activate twServer
twServer-->clFunctions: accessToken
deactivate twServer
clFunctions-->clPages: set cookie(http only;secure;samesite)
deactivate clFunctions
clPages->clFunctions: 認可情報取得(/api/me)
activate clFunctions
clFunctions->twApi: 認可情報取得(/users/me)
activate twApi
twApi-->clFunctions: 取得OK
deactivate twApi
alt 認可失敗した場合
  alt refreshトークンがヘッダーにある場合
    clFunctions->twAuth: tokenのリフレッシュ
    activate twAuth
    twAuth-->clFunctions: リフレッシュOK
    deactivate twAuth
    clFunctions->twApi: 認可情報取得
    activate twApi
    twApi-->clFunctions: 取得OK
    deactivate twApi
    ref over clFunctions, clKv
      ユーザーデータ取得処理参照
    end
    clFunctions-->clPages: 認可OK, set cookie(token,refresh_token)
  else それ以外
    clFunctions-->clPages: 401 Unauthorized
  end
end
group ユーザーデータ取得処理
  clFunctions->clFunctions: idを取り出す
  clFunctions->clKv: idに紐づくユーザーデータを取得
  activate clKv
  clKv-->clFunctions: 取得OK
  deactivate clKv
end
clFunctions-->clPages: 認可OK
deactivate clFunctions
clPages-->user: ログインSuccess
deactivate clPages
deactivate user
@enduml


※1 もやもやしている箇所。Twitterに遷移してしまうため、永続領域に書き込まないといけない。
code_verifierの格納は、secure属性とSameSite=Laxを付与。HttpOnly属性はJSからなので付与できない。プラスで適切なexpire時間を設定する。
```bash
document.cookie="code_verifier=hoge;secure;SameSite=Lax"
```

※2 scopeは以下の通り
/2/users/meエンドポイントが実行できる ... `users.read` `tweet.read`
refresh tokeの取得 ... `offline.access`

# 雑記
仕事で公開されているPOSTのAPIのHTTPレスポンスヘッダー(`Access-Control-Allow-Origin`)に呼び出し元を指定して、CSRF対策したのを思い出した

# 参考

[OAuth 2.0 Making requests on behalf of users](https://developer.twitter.com/en/docs/authentication/oauth-2-0/authorization-code)
[stackoverflow| Best practice on Securing code_verifier in PKCE-enhanced Authorization Code Flow](https://stackoverflow.com/questions/67517436/best-practice-on-securing-code-verifier-in-pkce-enhanced-authorization-code-flow)

