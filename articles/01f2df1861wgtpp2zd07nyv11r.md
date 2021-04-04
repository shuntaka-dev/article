---
title: "React周りで試したメモ"
type: "tech"
category: []
description: "React周りで試したメモ"
publish: false
---

# [React Hooks でカスタムフックを作ってみよう](https://zenn.dev/izuchy/articles/5cec4c2318caff6490de)

普通に書いてみる
```ts
import { useState } from 'react';
import Button from '@components/molecules/Button';

const Home = () => {
  const [count, setCount] = useState<number>(0);

  return (
    <div>
      <div> 現在の値: {count}</div>
      <Button
        theme="blue"
        value="CountUp"
        onClick={() => setCount(count + 1)}
      />
    </div>
  );
};

export default Home;
```


カスタムフックを作る

```ts
import { useState } from 'react';

export const useCounter = () => {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(count + 1);
  };

  return { current: count, increment: increment };
};
```

作ったカスタムフックを組み込み

```ts
import Button from '@components/molecules/Button';
import { useCounter } from '@src/hooks/useCounter';

const Home = () => {
  const { current, increment } = useCounter();

  return (
    <div>
      <div> 現在の値: {current}</div>
      <Button theme="blue" value="CountUp" onClick={() => increment()} />
    </div>
  );
};

export default Home;
```

メリット(前述の記事引用)
> * useCounter とすることで useState を使うよりもカウンタの処理だということが明確になる
> * カウンタの値を +1 増やすという暗黙のルールを useCounter 内に持ち込むことができる
> * increment しか用意されていないので、勝手にカウンタの値を減らされる心配がない（笑）

increment関数がさらに複雑だった場合、コードの見通しもよくなりそう

# Hooksについて
## なぜ必要なのか
Reactには、書き方が大きく分けて2つある
* クラスコンポーネント
* 関数コンポーネント

Hooks登場前のReactはクラスコンポーネントが一般的だった。
=>  クラスコンポーネントには状態管理やライフサイクルの機能がるが、関数コンポーネントにはそれらの機能がなかった

Hooksは状態管理やライフサイクルを扱うことが可能。
クラスコンポーネントを関数コンポーネント+Hooksで代用できるようになった

## クラスとどう使い分けるべきか
[フックに関するよくある質問](https://ja.reactjs.org/docs/hooks-faq.html#should-i-use-hooks-classes-or-a-mix-of-both)をみると、下記の回答があります

> 長期的には、フックが React のコンポーネントを書く際の第一選択となることを期待しています。

Reactとしては、関数コンポーネント+Hooksが推奨されている

## useContext
### 解決すべき課題
Reactでは、他コンポーネントにデータを渡す手段として`props`がある。
propsの使用は、親コンポーネントから孫コンポーネントへデータを渡す場合に面倒になる。(子をスルーする)
親->孫が理想だが、親->子->孫とバケツリレーしないといけない、`prop drilling問題`がある

Reduxのような中央集権的状態管理を使えば解決可能ですが、useContextはもっとライトなアプローチでこの課題に取り組んでいます。

[5分でわかるReact Hooks](https://qiita.com/Mitsuzara/items/98d1bc4a83265a764084)を参照

### useContextのアプローチ
例えば`blog.hozi.dev`では、下記のデータ取得に`useContext`を活用している

* 認証時のuserId
* カラーテーマ

```ts
import React, { useEffect } from 'react';

type ColorTheme = {
  colorMode?: string;
  changeColorMode: (cm: string) => void;
};

export const ColorThemeContext = React.createContext<ColorTheme>(null);

const ColorThemeProvider: React.FC = ({ children }) => {
  const [colorMode, setColorMode] = React.useState(undefined);

  useEffect(() => {
    const theme = document.documentElement.dataset.theme;
    setColorMode(theme);
    window.localStorage.setItem(
      'color-mode',
      document.documentElement.dataset.theme,
    );
  }, [colorMode]);

  const changeColorMode = (colorMode: string) => {
    document.documentElement.dataset.theme = colorMode;
    setColorMode(colorMode);
  };

  return (
    <ColorThemeContext.Provider value={{ colorMode, changeColorMode }}>
      {children}
    </ColorThemeContext.Provider>
  );
};

export default ColorThemeProvider;
```


