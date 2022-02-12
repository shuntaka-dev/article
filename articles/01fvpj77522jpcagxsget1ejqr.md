---
title: "SPAにTwitter OAuth2.0を組み込む際の雑メモ"
type: "note"
category: []
description: "SPAにTwitter OAuth2.0を組み込む際の雑メモ"
thumbnail: ""
publish: true
---

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
clPages->clPages: code_verifierをcookieに保存
clPages->clPages: code_verifierからcode_challenge生成
clPages->twAuth: code_challenge
activate twAuth
twAuth->twAuth: Present authorization dialog
twAuth-->user
deactivate twAuth
user->user: Authraize or declines app
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
clPages->clFunctions: 認可チェック(できるか不明?)
activate clFunctions
clFunctions-->twApi: 認可チェック
activate twApi
twApi-->clFunctions: 認可OK
deactivate twApi
clFunctions-->clPages: 認可OK
deactivate clFunctions
clPages-->user: ログインSuccess
deactivate clPages
deactivate user
@enduml

仕事で公開されているPOSTのAPIのHTTPレスポンスヘッダー(`Access-Control-Allow-Origin`)に呼び出し元を指定して、CSRF対策したのを思い出した

# 参考

[OAuth 2.0 Making requests on behalf of users](https://developer.twitter.com/en/docs/authentication/oauth-2-0/authorization-code)
[stackoverflow| Best practice on Securing code_verifier in PKCE-enhanced Authorization Code Flow](https://stackoverflow.com/questions/67517436/best-practice-on-securing-code-verifier-in-pkce-enhanced-authorization-code-flow)
