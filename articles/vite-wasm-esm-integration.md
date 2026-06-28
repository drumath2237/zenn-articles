---
title: "Vite 8.1の新機能、WebAssembly ESM Integrationについて"
emoji: "🌈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vite", "wasm", "esmodule", "javascript"]
published: false
---

## はじめに

### TL;DR

- Vite で、`.wasm`ファイルから直接関数をインポートできるようになった
- TypeScript コンパイラではエラーが出るので、それをケアする必要がある

### 概要

米国時間の 2026 年 6 月 23 日に、[Vite 8.1 のリリースブログ](https://vite.dev/blog/announcing-vite8-1)が公開されました。
その中で「Wasm ESM integration Support」という項目が気になったので試しに使ってみることにしました。

https://vite.dev/blog/announcing-vite8-1#wasm-esm-integration-support

https://vite.dev/guide/features#esm-integration

本記事では、Rust でビルドした WASM を実際に Vite プロジェクト内の TypeScript でインポートしてみるまでの様子を解説します。

### 対象読者

本記事では次のような読者像を想定していますが、必ずしもこれに当てはまらないと読めないわけではありません。

- Vite を使ったことがある Web フロントエンジニア
- WebAssembly を使ったことがあるエンジニア
- Vite について関心があるエンジニア

### 検証環境とサンプルプロジェクト

本記事の執筆において、筆者の手元の環境で動作検証をしました。
検証環境を次に示します。

- Windows 11 Home
- Vite 8.1.0
- Node.js 24.16.0
- Cargo 1.96.0
  - `wasm32-unknown-unknown`ターゲット
- Google Chrome for Windows 149.0.7827.199

また、検証したプロジェクトを GitHub で公開していますので、合わせてご参考ください。

https://github.com/drumath2237/vite8-wasm-esmodule-sandbox

## WebAssembly ES Module Integration

本記事のテーマである Vite の WebAssembly ESM Integration は WebAssembly CG (Community Group)が主体となって標準化を進めているプロポーザルが元になっています。名前は「WebAssembly/ES Module Integration」です。

@[card](https://github.com/WebAssembly/esm-integration/blob/main/proposals/esm-integration/README.md)

従来、WebAssembly を JavaScript で読み込むときはユーザが明示的に WASM ファイルを fetch して、WebAssembly モジュールをインスタンス化する必要がありました。

```js:wasmファイルfetchしてモジュールをインスタンス化
// プロポーザルREADMEから引用
// https://github.com/WebAssembly/esm-integration/blob/main/proposals/esm-integration/README.md

let req = fetch("./myModule.wasm");

let imports = {
  aModule: {
    anImport
  }
};

WebAssembly
  .instantiateStreaming(req, imports)
  .then(
    obj => obj.instance.exports.foo()
  );
```

WebAssembly/ES Module Integration プロポーザルが実装されると、WASM モジュールで実装されている関数を直接 import できるようになります。

```js:WASMから関数を直接import
// プロポーザルREADMEから引用
// https://github.com/WebAssembly/esm-integration/blob/main/proposals/esm-integration/README.md

import { foo } from "./myModule.wasm";
foo();
```

このプロポーザルでは JS から WASM をインポートするだけでなく、WASM から JS をインポートする際にも同様に直接 import が可能になります。

```wasm:main.wat
;; プロポーザルEXAMPLESから引用
;; https://github.com/WebAssembly/esm-integration/blob/main/proposals/esm-integration/EXAMPLES.md

;; main.wat --> main.wasm
(module
  (import "./counter.js" "getCount" (func $getCount (func (result i32))))
)
```

```js:counter.js
// プロポーザルEXAMPLESから引用
// https://github.com/WebAssembly/esm-integration/blob/main/proposals/esm-integration/EXAMPLES.md

// counter.js
let count = 42;

function getCount() {
    return count;
}
export {getCount};
```

JS から WASM モジュールを直接 import という意味では他にも「Source Phase Import」というプロポーザルが TC39 で出ています。

```js
// 記事から引用
// https://zenn.dev/pixiv/articles/c7071eb29927fe#webassembly

import source fooSource from "./foo.wasm";

console.log(fooSource instanceof WebAssembly.Module); // => true
const fooInstance = await WebAssembly.instantiate(fooSource, {/* imports */});
const { foo } = fooInstance.exports;
```


詳しくは下記リソースをご参照ください。

https://github.com/tc39/proposal-source-phase-imports

https://zenn.dev/pixiv/articles/c7071eb29927fe#webassembly

## Viteで使ってみる

### RustでWASMビルドする

### Viteのプロジェクトにインポートしてみる

### `allowArbitraryExtensions`の設定をする

## おわりに

### 参考文献
