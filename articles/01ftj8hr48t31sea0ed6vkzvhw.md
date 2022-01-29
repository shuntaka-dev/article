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

# [Margin](https://tailwindcss.com/docs/padding)

# [Handling Hover, Focus, and Other States](https://tailwindcss.com/docs/hover-focus-and-other-states)

hoverしたときに、bg-sky-700に状態変化する
```ts
<button class="bg-sky-600 hover:bg-sky-700">
  Save chages
</button>
```

## [Hover, focus, and active](https://tailwindcss.com/docs/hover-focus-and-other-states#hover-focus-and-active)

`outline-none`

```ts
<button class="bg-violet-500 hover:bg-violet-400 active:bg-violet-600 focus:outline-none focus:ring focus:ring-violet-300 ...">
  Save changes
</button>
```


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
|py-2.5|
|px-5|
|mb-2|
|text-sm|
|font-medium|
|text-center|
|text-white|
|bg-purple-700|
|hover:bg-purple-800|
|dark:bg-purple-600|
|dark:hover:bg-purple-700|
|rounded-full|
|focus:ring-4|
|focus:ring-purple-300|
|dark:focus:ring-purple-900|


