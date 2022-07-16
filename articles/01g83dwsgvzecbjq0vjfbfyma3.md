---
title: "ziglingsやってみる(随時更新)"
type: "note"
category: []
description: "ziglingsの翻訳、所感を随時更新でつらつらと書きます"
thumbnail: ""
publish: true
---

# はじめに

[zigling](https://github.com/ratfactor/ziglings) というZigのチュートリアルリポジトリがあるので、単なる翻訳+所感をつらつら書いてきます。
リンクのソースが変わっていた場合は、[fork版](https://github.com/shuntaka9576/ziglings)を参照ください。

# 001
[001_hello.zig](https://github.com/ratfactor/ziglings/blob/main/exercises/001_hello.zig)

`Hello World!`と標準出力させる問題。ポイントは以下。

* Zig関数はデフォルトでプライベート関数
* main関数はパブリックである必要があり

接頭辞に`pub`キーワードがあるかどうかで、公開/非公開を制御可能
```zig:非公開関数
fn foo() void {
}
```

```zig:公開関数
pub fn foo() void {
}
```

## 所感
print関数の`.{}`の部分がなぜ必要なのかよくわからない
=> 003で解決。第二引数には匿名リストリテラルを指定する必要あり、表示したい変数がない場合は本例のように空で指定する

```zig
  std.debug.print("hello wolrd", .{});
```

# 002
[002_std.zig](https://github.com/ratfactor/ziglings/blob/main/exercises/002_std.zig)

Zigには`@import`という組み込み関数がある。以下のようにインポートする名前と同名の定数値として保存することが推奨される。

```zig
const std = @import("std")
```

* importsは定数として宣言する必要がある
  * importsは実行時ではなくコンパイル時に利用される関数であるため
* Zigは定数値をコンパイル時に評価する


参考
* [struct definition with var instead of const in zig language回答部](https://stackoverflow.com/a/62567550/695615)


# 003
[003_assignment.zig](https://github.com/ratfactor/ziglings/blob/main/exercises/003_assignment.zig)

constは不変、varは可変

|値型|説明|
|---|---|
|u8|符号なし型8ビット整数(0 to 256)|
|i8|符号あり型8ビット整数(-128 to 127)|
|u32|符号あり型32ビット整数(0 to 4,294,967,295)|
|i64|符号あり型32ビット整数(−9,223,372,036,854,775,808 to 9,223,372,036,854,775,807)|

```zig:宣言例
// 変更不可
const v1: u8 = 20;
const v2: i8 = -20;
// 可変
var v3: u8 = 20;
```

```zig:回答例
const std = @import("std");

pub fn main() void {
  var n: u8 = 50;
  // constなので再代入不可
  // n = n + 5;
  n = n + 5;


  // u8の範囲は、0 - 256なのでコンパイルエラー
  // const pi: u8 = 314159;
  const pi: u32 = 314159;
  // u8の範囲は、0 - 256なのでコンパイルエラー
  // const negative_eleven: u8 = -11;
  const negative_eleven: i8 = -11;

  std.debug.print("{} {} {}\n", .{n, pi, negative_eleven});
}
```

* `std.debug.print`の第2パラーメーターは、匿名リストリテラルと呼ぶ
  * 001の時点で疑問だった第二引数の意味が分かった

## 所感
数値型の宣言方法は、Rustとほぼ一緒かな。


# 004
[004_arrays.zig](https://github.com/ratfactor/ziglings/blob/main/exercises/004_arrays.zig)

配列の宣言

```zig:配列長を指定して宣言
var foo: [3]u32 = [3]u32{ 42, 108, 5423}
```

```zig:コンパイル時に配列長が確定している場合省略可(基本こっちを利用するかな)
var foo = [_]u32{ 42, 108, 5423}
```

配列は、メジャー言語と同じで0始まり。
```zig:要素の取得
const foo = foo[2] // 5432
```

ここら辺は、003でやった通り。配列だとconstでも挙動が違う言語もあり、TSは配列の要素は再代入できたりする。zigでは再代入不可。
```zig:配列の値変更
// varな配列では変更可
var var_array = [_]u8{1, 2, 3};
var_array[1] = 24;

// constな配列の要素に再代入は不可
// const const_array = [_]u8{1, 2, 3};
// const_array[1] = 24;
```

```zig:配列長の取得
const length = some_primes.len;
```


# 005
[005_arrays2.zig](https://github.com/ratfactor/ziglings/blob/main/exercises/005_arrays2.zig)

`++`演算子で配列の結合が可能
```zig:配列の結合
const le = [_]u8{ 1, 3 };
const et = [_]u8{ 3, 7 };

const leet = le ++ et ++ [_]u8{ 5 };
std.debug.print("{}", .{ leet[4] }); // 5
```

`**`演算子で配列の繰り返しが可能。あると便利だがユースケースそんなにあるかなぁ..
```zig:配列の繰り返し結合
const bit_pattern = [_]u8{ 1, 0, 0, 1 } ** 3;
std.debug.print("bit_pattern len: {}\n", .{ bit_pattern.len }); // 11
std.debug.print("bit_pattern last value: {}\n", .{ bit_pattern[bit_pattern.len - 1] }); // 1
```


