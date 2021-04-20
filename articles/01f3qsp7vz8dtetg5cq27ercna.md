---
title: "next/linkのpassHrefがどのような場合に必要か整理する"
type: "tech"
category: []
description: ""
publish: false
---

# 結論
下記のケースで、Linkコンポーネントのhref属性を、子コンポーネントのaタグに付与しないからSEO的に悪影響よという話。

* 子が **<a>タグをラップするカスタムコンポーネント**の場合、**passHrefすると**とaタグにhref属性を付与するよ！
* 子が **(<a>タグをラップした)関数コンポーネント**の場合、**passHrefかつReact.forwardRefでラップする**ととhref属性を付与するよ！

# 公式ドキュメントを確認
公式ドキュメント[next/link](https://nextjs.org/docs/api-reference/next/link)の章題は下記。

1. If the route has dynamic segments(ルートに動的セグメントがある場合)
1. If the child is a custom component that wraps an <a> tag(子が<a>タグをラップするカスタムコンポーネントである場合)
1. If the child is a function component(子が関数コンポーネントの場合)
1. With URL Object(URLオブジェクト付き)
1. Replace the URL instead of push(プッシュの代わりにURLを置き換える)
1. Disable scrolling to the top of the page(ページ上部へのスクロールを無効にする)

`passHref`について言及しているのは、`2. If the child is a custom component that wraps an <a> tag`と`3. If the child is a function component`


# 検証
※ ◯はaタグにhref属性が付与されることを示す
|項|Linkコンポーネントでラップされたコンポーネント|passHref無|passHref有|
|---|---|---|---|
|1|aタグ|◯|◯|
|2|aタグをWarpしたカスタムコンポーネント||◯|
|3|FunctionalComponent|||
|4|React.forwardRefでWrapしたFunctionalComponent||◯|

```ts
/* FunctionalComponent */
const FunctionalComponent: React.VFC<{ value: string }> = ({ value }) => (
  <a>{value}</a>
);

/* React.forwardRefでラップしたFunctionalComponent */
type AncoherPorps = JSX.IntrinsicElements['a'];
type FunctionalComponentForwardRefProps = { value: string };
const FunctionalComponentForwardRef = React.forwardRef<
  HTMLAnchorElement,
  AncoherPorps & FunctionalComponentForwardRefProps
>(({ value, href }, ref: React.LegacyRef<HTMLAnchorElement>) => {
  return (
    <a href={href} ref={ref}>
      {value}
    </a>
  );
});

ForwardRefComponentWrapForwardRef.displayName = 'ForwardRefComponent';

const HogePage: React.VFC = () => {
  return (
    {/* 表 項1-passHref無 */}
    <Link href="/test">
      <a>testLink</a>
    </Link>
    {/* 表 項1-passHref有 */}
    <Link href="/test" passHref>
      <a>testLink</a>
    </Link>
    // TODO カスタムコンポーネントについて書く

    {/* 表 項3-passHref無 */}
    <Link href="/test">
      <FunctionalComponent value={`testLink`} />
    </Link>
    {/* 表 項3-passHref有 */}
    <Link href="/test" passHref>
      <FunctionalComponent value={`testLink`} />
    </Link>
    {/* 表 項4-passHref無 */}
    <Link href="/test" passHref>
      <FunctionalComponentForwardRef value={`testLink`} />
    </Link>
    {/* 表 項4-passHref有 */}
    <Link href="/test" passHref>
      <FunctionalComponentForwardRef value={`testLink`} />
    </Link>
  )
}
```

// TODO HTMLの結果を載せておく

# おまけ

// TODO function各パターンで↑を書き直す
```ts
const ForwardRefComponent2 = React.forwardRef(function forwardRefComponent2(
  _props,
  ref: React.LegacyRef<HTMLAnchorElement>,
) {
  return <a ref={ref}>Click Me</a>;
});
```
