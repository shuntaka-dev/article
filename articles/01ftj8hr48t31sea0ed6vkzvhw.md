---
title: "tailwindcss入門(随時更新)"
type: "note"
category: []
description: "tailwindcss入門"
thumbnail: ""
publish: true
---

:::message warning
[tailwindcss公式](https://tailwindcss.com/docs)の雑メモ
:::


# [Padding](https://tailwindcss.com/docs/padding)
|class名|説明|
|---|---|
|p(t\|r\|b\|l)-{数字}|t(上), r(右), b(下), l(左)方向のpadding
|px-{数字}|左右のpadding
|py-{数字}|上下のpadding
|p-{数字}|上下左右のpadding
|hover:py-8|hoverしたとき`py-8`を適用


# [FontSize](https://tailwindcss.com/docs/font-size)

|class名|font-size|line-height|
|---|---|---|
|text-xs|0.75rem|1rem|
|text-sm|0.875.rem|1.25rem|
|text-base|1rem|1.5rem|
|text-lg|1.125rem|1.75rem|
|text-xl|1.25rem|1.75rem|
|text-2xl|1.5rem|2.rem
|text-3xl|1.875rem|2.25rem
|text-4xl|2.25rem|2.5rem
|text-5xl|3rem|1
|text-6xl|3.75rem|1
|text-7xl|4.5rem|1
|text-8xl|6rem|1
|text-9xl|8rem|1


# [Font Weight](https://tailwindcss.com/docs/font-weight)

|class名|font-weight|
|---|---|
|font-thin|100|
|font-extralight|200|
|font-light|300|
|font-normal|400|
|font-medium|500|
|font-semibold|600|
|font-bold|700|
|font-extrabold|800|
|font-black|900|

# [Margin](https://tailwindcss.com/docs/padding)

|class名|説明|
|---|---|
|m(t\|r\|b\|l)-{数字}|t(上), r(右), b(下), l(左)方向のmargin
|mx-{数字}|左右のmargin
|my-{数字}|上下のmargin
|mp-{数字}|上下左右のmargin

# [Text Alignment](https://tailwindcss.com/docs/text-align)

|class名|text-align|
|---|---|
|text-left|left|
|text-center|center|
|text-right|right|
|text-justify|justify|


# [Text Color](https://tailwindcss.com/docs/text-color)
色の指定、省略

|class名|説明|
|---|---|
|text-{}|textの色指定|
|bg-{}|背景の色指定|
|ring-{}|アウトラインリングの色指定|

# [Border Radius](https://tailwindcss.com/docs/border-radius)
角の丸み指定、省略

* rounded-{}
  * rounded-none
  * rounded-sm
  * rounded-md
  * rounded
  * etc ...

# [Border Width](https://tailwindcss.com/docs/border-width)
枠線の太さ
* border-{}

## サンプル
border-b-2...下に2px
border-blue-500...2pxにblue-500の色付け


# [Ring Width](https://tailwindcss.com/docs/ring-width)
ボックスシャドウを使ったアウトラインリング

* ring-{}
  * ring-0
  * ring-1
  * ring-2
  * ring



# [Responsive Design](https://tailwindcss.com/docs/responsive-design)
`sm:`だと、このブレークポイントより上の..という意味を指す

|BreakPoint prefix|Minimum width|CSS
|---|---|--|
|sm|640px|
|md|768px|
|lg|1024px|
|xl|1280px|
|2xl|1536px|

以下の例の場合
* 接頭辞なしのtext-center -> モバイル適用
* sm:text-left -> smより大きい場合適用
```ts
<div class="text-center sm:text-left" />
```

上記のような接頭辞のないユーティリティをモバイルのターゲットにして、より大きなブレークポイントで上書きする書き方が良いみたい。


## Sample
接頭辞なしは、モバイル適用と意識しておくとレスポンシブなtailwindcssを読み解くことができそう

```ts
<div class="sm:hidden">
 // モバイルより大きい場合、hidden
<div>
<div class="hidden sm:block">
 // モバイルでdisplay:hidden
</div>
```


# [Handling Hover, Focus, and Other States](https://tailwindcss.com/docs/hover-focus-and-other-states)

hoverしたときに、bg-sky-700に状態変化する
```ts
<button class="bg-sky-600 hover:bg-sky-700">
  Save chages
</button>
```

## [Hover, focus, and active](https://tailwindcss.com/docs/hover-focus-and-other-states#hover-focus-and-active)

### 基礎的な前提

擬似クラスの確認

|オプション|説明|
|---|---|
|:link|リンクのデフォルトの状態
|:visted|既にクリックされているリンクの状態
|:hover|マウスをリンクの上に置いたときの状態
|:active|アクティブがされた状態
|:focus|タッチデバイスでタップした時や、キーボードのtabキーで移動した時


### サンプル

`outline-none`

```ts
<button class="bg-violet-500 hover:bg-violet-400 active:bg-violet-600 focus:outline-none focus:ring focus:ring-violet-300 ...">
  Save changes
</button>
```


要件
* hoverすると、少し暗くなる
* 押下すると枠線強調表示
```ts
<button
  type="button"
  className="py-2.5 px-5 mb-2 text-sm font-medium text-center text-white bg-purple-700 hover:bg-purple-800 dark:bg-purple-600 dark:hover:bg-purple-700 rounded-full focus:ring-4 focus:ring-purple-300 dark:focus:ring-purple-900"
>
  Purple
</button>
```

|オプション名|解説|
|---|---|
|py-2.5|上下padding2.5|
|px-5|左右padding5|
|mb-2|下margin2
|text-sm|font-size指定
|font-medium|font-weight指定
|text-center|text-alignを中心へ
|text-white|text色指定
|bg-purple-700|背景色を指定
|hover:bg-purple-800|ホバーしたら、背景色をpurple-800へ
|dark:bg-purple-600|ダークモードのときの背景色
|dark:hover:bg-purple-700|ダークモードかつホバーしたときの背景色
|rounded-full|border-radius: 9999px;指定
|focus:ring-4|アウトラインリング
|focus:ring-purple-300|アウトラインリングの色指定
|dark:focus:ring-purple-900|ダークモード のアウトラインリングの色指定



## [Dark Mode](https://tailwindcss.com/docs/dark-mode)

以下のように`dark`variantを利用する

```ts
dark:bg-slate-900
```


## FontFamily

```ts
module.exports = {
  content: ['./src/index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      fontFamily: {
        'sans': ['sans-serif'],
        'relics': ['Times New Roman', 'MS PGothic'],
        'ui': ['system-ui', '-apple-system']
      }
    }
  },
  plugins: [],
};
```

```ts
const defaultTheme = require("tailwindcss/defaultTheme");

module.exports = {
  content: ['./src/index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      fontFamily: {
        sans: ["Noto Sans JP", ...defaultTheme.fontFamily.sans],
      },
    }
  },
  plugins: [],
};
```

