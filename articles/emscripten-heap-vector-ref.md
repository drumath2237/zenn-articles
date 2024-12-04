---
title: "EmscriptenのHEAPを使ってC++/JavaScript間でバッファやvectorを参照する"
emoji: "🥧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["emscripten", "cpp", "javascript", "webassembly", "wasm"]
published: false
---

:::message
この記事は[にー兄さんアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/ninisan-2024)の4日目の記事です。
:::

## はじめに

### TL;DR

- JS からでバッファを参照する時は HEAP におけるポインタと Length を使って TypedArray で取り出す
- C++で受け取る時は int 型のアドレスをポインタ型にキャストする

### 概要

本記事では、Emscripten を使って C++のコードを WebAssembly にコンパイルして使うプロジェクトにおいて、
バッファのやり取りをする方法を解説します。
JS 側では ArrayBuffer・TypedArray として、C++側では vector として取り扱う前提となっています。

調べてみるといろいろな方法があるみたいですが、今回ご紹介するのは HEAP を使うシンプル方法です。
理解できてしまえば簡単なのですが、ポインタを扱う関係で一瞬難しそうに感じたのと、日本語文献も少なかったので共有しようと考えました。
また、別の方法で`emscripten::val`を使う方法もあるみたいですが、今回は取り扱いません。

本当はサンプルプロジェクトを公開したかったのですが、ひとりアドカレという環境ゆえに間に合わず、コードの断片でのみの説明となりますことをお詫びいたします。

### 検証環境

- Windows 11 Home
- Docker Desktop
- Emscripten(emsdk) 3.1.72

## まずはembindする

vector はプリミティブな型ではないため、まずは embind して JS から使えるようにしておきます。
今回は`uint8_t(char/byte)`型と`float`型の vector を使うと想定しましょう。

```cpp:embindする
#include <emscripten.h>
#include <emscripten/bind.h>

// ...

EMSCRIPTEN_BINDINGS(my_module)
{
  register_vector<float>("VectorFloat32");
  register_vector<uint8_t>("VectorUInt8T");
}
```

この状態で、`-lembind`、`--emit-tsd`オプションを使って型定義まで出力してみると、こんな感じになります。

```ts:emscriptenから出力された型定義ファイルの一部
export interface ClassHandle {
  isAliasOf(other: ClassHandle): boolean;
  delete(): void;
  deleteLater(): this;
  isDeleted(): boolean;
  clone(): this;
}

export interface VectorFloat32 extends ClassHandle {
  size(): number;
  get(_0: number): number | undefined;
  push_back(_0: number): void;
  resize(_0: number, _1: number): void;
  set(_0: number, _1: number): boolean;
}

export interface VectorUInt8T extends ClassHandle {
  push_back(_0: number): void;
  resize(_0: number, _1: number): void;
  size(): number;
  get(_0: number): number | undefined;
  set(_0: number, _1: number): boolean;
}
```

分かる通り、`VectorFloat32`や`VectorUInt8T`は`ClassHandle`を継承している interface になっており、このままではどうやら TypedArray として使えるわけではないようです。

## JSからC++へTypedArrayを渡す

基本的な方針として、JS から C++の関数を呼び出すときにバッファを渡したい場合、
HEAP に確保したバッファ領域へバッファを書き込み、そのアドレスを渡す感じになります。

たとえば次の例では、`binaryBuffer`という`Uint8Array`を C++の関数へ渡す処理です。

```ts:TypeScriptからC++へバッファを渡す
// MainModuleを取得
const wasmModule = await MainModuleFactory();

// pointerはnumber型
// バッファのbyteSize分メモリを確保する
const pointer = wasmModule._malloc(Uint8Array.BYTES_PER_ELEMENT * binaryBuffer.length);
if (pointer === null) {
  throw new Error("could'nt allocate memory");
}

// Uint8Arrayの場合はHEAPU8のビューを使う
wasmModule.HEAPU8.set(binaryBuffer, pointer / Uint8Array.BYTES_PER_ELEMENT);

// C++の関数にはバッファの先頭アドレスと長さを渡す
wasModule.someFunc(pointer, binaryBuffer.length);

// ...

wasmModule._free(pointer)
```

一方で C++側の関数では、ポインタとバッファ長から vector を取得します。

```cpp:C++でポインタを受け取りvectorに変換して利用する
void someFunc(const int ptr, const int length)
{
  auto pointer = (uint8_t *)ptr;
  auto uint8Vector = vector<uint8_t>(pointer, pointer + length);
  // ...
}

EMSCRIPTEN_BINDINGS(my_module)
{
  emscripten::function("someFunc", &someFunc, allow_raw_pointers());
}
```

やっていることはシンプルです。

- JS/C++間では型付きのビューとしての HEAP を利用する
- JS では number 型で先頭アドレスを取得できる
- C++ではアドレスをポインタ型にキャストし、そこから vector を得る

## C++のvectorデータをJSからTypedArrayとして参照する

## 注意：ALLOW_MEMORY_GROWTHを使っている場合

## おわりに

### 参考文献
