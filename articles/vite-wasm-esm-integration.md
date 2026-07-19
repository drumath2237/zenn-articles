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
- Vite 8.1.5
- Node.js 24.16.0
- Cargo 1.96.0
  - `wasm32-unknown-unknown`ターゲット
- Google Chrome for Windows 150.0.7871.129

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

それでは、Vite のプロジェクトを作成して WASM モジュールの関数をインポートしてみましょう。

### RustでWASMビルドする

まずは Cargo で lib プロジェクトを作成し、`wasm32-unknown-unknown`ターゲットでビルドしてみます。
そういう場合は wasm-bindgen を使う事が多いですが、今回は必要ないので依存関係には加えません。

```sh
# cargoでプロジェクトを作成（プロジェクト名はご自由に）
cargo new --lib ./math-wasm
```

作成されたプロジェクト中の`/src/lib.rs`を次のように書き換えます。
`export_name`マクロは`no_mangle`でも構いませんし、`f64`を`f32`にしていますが特に意味はありません。

```rs:/src/lib.rs
#[unsafe(export_name = "add")]
pub extern "C" fn add(left: f32, right: f32) -> f32 {
    left + right
}
```

また、WASM ビルドのために、`Cargo.toml`に次の設定を追加します。

```toml:/Cargo.toml
[lib]
crate-type = ["cdylib", "rlib"]
```

ここまでできたら、wasm32 をターゲットにビルドしましょう。
次のコマンドを実行します。

```sh
cargo build --target wasm32-unknown-unknown
```

すると`/target/wasm32-unknown-unknown/debug/math_wasm.wasm`に WASM ファイルが出力されます。

### Viteのプロジェクトにインポートしてみる

WASM ファイルをインポートするための Vite プロジェクトを作ります。
試すだけであれば UI フレームワークは不要ですので、`vanilla-ts`なプロジェクトを作りましょう。
執筆時点（2026 年 7 月）では、これで Vite 8.1.4 が使われるようでした。もし 8.1.0 よりも低いバージョンがインストールされたらバージョンを更新してください。

```sh
pnpm create vite@latest
```

Scaffolding されたプロジェクトファイルをいったん整理したうえで、先ほど Rust コードからビルドした WASM ファイルを`/wasm/math_wasm.wasm`に配置しました。

```
/
├─ src/
│    └─ main.ts
├─ wasm/
│    └─ math_wasm.wasm
├─ index.html
├─ tsconfig.json
└─ package.json
```

続いて`/src/main.ts`を次のように編集してみます。

```ts:main.ts
import { add } from "../wasm/math_wasm.wasm";

function main() {
  const app = document.getElementById("app");
  if (!(app instanceof HTMLDivElement)) {
    return;
  }

  const result = add(1, 2);
  console.log(result);

  app.textContent = `add(1, 2) = ${result}`;
}

main();
```

`main.ts`の 1 行目が WASM ESM Integration によって実現した記述ですね。あたかも WASM ファイルから何もせずに`add`関数を読み込んでいるような見た目になっています。

この状態で`pnpm run dev`して Vite 開発サーバを立ち上げてみましょう。
開いたページで「add(1, 2) = 3」と表示されていれば、見事 WASM が読み込めたようです。

### `allowArbitraryExtensions`の設定をする

それではこのプロジェクトをビルドしてみましょう。すると次のようなエラーが出てしまいます。
Vite では WASM を直接インポートできるので`vite dev`や`vite build`は問題なく動作します。しかし TypeScript は`.wasm`ファイルから export された関数が何なのか把握できないので、型チェックでエラーになってしまうんですね。

```sh:型チェックでエラーになる
pnpm build
$ tsc && vite build
src/main.ts:1:10 - error TS2305: Module '"*.wasm"' has no exported member 'add'.

1 import { add } from "../wasm/math_wasm.wasm";
           ~~~


Found 1 error in src/main.ts:1
```

## おわりに

### 参考文献
