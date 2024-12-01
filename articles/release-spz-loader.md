---
title: "3D Gaussian Splattingローダライブラリ\"spz-loader\"をリリースしました"
emoji: "🦎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["gaussiansplatting", "webassembly", "typescript", "pnpm", "monorepo"]
published: false
---

:::message
この記事は[にー兄さんアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/ninisan-2024)の1日目の記事です。
:::

## はじめに

### TL;DR

GaussianSplatting のフォーマットの一種である.spz ファイルのためのローダをリリースしました。

### 概要

本記事では、最近私がリリースした spz-loader という npm パッケージの概要をご紹介する内容となっています。
個々の技術に関しての細かい内容については別の記事にて紹介する方針で、
とりあえずこの記事ではざっくりとしたライブラリの全体像をご紹介できればと考えています。

## spz-loaderをリリースしました🎉

11/28 に spz-loader の v0.1.0 をリリースしました。

https://github.com/drumath2237/spz-loader/releases/tag/v0.1.0

spz-loader は名前の通り、SPZ ファイルの読み込みができるライブラリです。

## spzフォーマットとは

SPZ は Niantic が策定する 3D Gaussian Splatting のファイルフォーマットで、
既存のフォーマットよりも圧縮率が高いという特徴を持ちます。
詳細なフォーマットについては公式ブログに解説があります。

https://scaniverse.com/news/spz-gaussian-splat-open-source-file-format

ブログにも書いてある通り、GaussianSplatting が保持するデータについて次のような工夫をしています。

- 各データをその性質に基づいて適切な精度に収めている
  - 従来使われていた.ply ではすべて double や float で保持していた
- gzip で圧縮している
  - 列ベースでバイナリデータを格納しているため圧縮効率が良いらしい

もともと、GaussianSplatting に使われてきたファイルフォーマットというのは
色々なものがありました。
一番スタンダードなのは原著論文のプロジェクトでも使用されていた PLY ファイルですが、
他にも Compressed PLY, splat, ksplat などといった独自のフォーマットも存在します。
そういう状況下で、GaussianSplatting を扱うソフトウェアは複数のフォーマットに対応しなくてはならず、スタンダードなデータフォーマットを欲する声も上がっていました。

そんな中、Niantic は SPZ フォーマットを提案します。そして既に SPZ は同社が提供する Scaniverse や Niantic Studio といったプラットフォームで利用可能です。
私としては、SPZ の登場により GaussianSplatting データの標準化が進むと嬉しいです。

## spz-loaderでできること

spz-loader は複数の npm パッケージから構成されます。
とは言っても v0.1.0 では次の 2 つのみです。

- core パッケージ
- Babylon.js 用パッケージ

### coreパッケージ

パッケージ名は[`@spz-loader/core`](https://www.npmjs.com/package/@spz-loader/core)です。
core パッケージは SPZ データをデコードし、ピュアな JavaScript Object への変換ロジックを提供します。使用感としてはこんな感じです。

```ts:coreパッケージの使用例
import { loadSpz } from "@spz-loader/core";

import spzUrl from "../assets/racoonfamily.spz?url";

const spzBuffer = await fetch(spzUrl)
  .then((res) => res.arrayBuffer())
  .then((buf) => new Uint8Array(buf));

const splat = await loadSpz(spzBuffer);

console.log(splat.numPoints);
```

`loadSpz`関数にバッファを渡すと`GaussianCloud`型のオブジェクトが返却される感じです。
この GaussianCloud オブジェクトの定義は次の通りになっており、基本的に本家 nianticlabs/spz の定義と似たようなものになっています。

```ts:GaussianCloudの定義
export type GaussianCloud = {
  numPoints: number;
  shDegree: number;
  antialiased: boolean;
  positions: Float32Array;
  scales: Float32Array;
  rotations: Float32Array;
  alphas: Float32Array;
  colors: Float32Array;
  sh: Float32Array;
};
```

https://github.com/nianticlabs/spz/blob/bf305418722bb0663a3074f3828699df44b8c1d2/src/cc/splat-types.h#L29-L54

### Babylon.js用パッケージ

パッケージ名は[`@spz-loader/babylonjs`](https://www.npmjs.com/package/@spz-loader/babylonjs)です。
このパッケージでは、Web3D レンダリングエンジンである Babylon.js との連携用のロジックを提供します。

もともと Babylon.js には GaussianSplatting をロードして表示する機能があります。
つまり core パッケージで提供されているデコーダから GaussianCloud を生成し、
Babylon.js 用のオブジェクトへ変換できれば Babylon.js でも表示できますね。

ということで、そこを丸っとやってくれるような API を提供しているのがこのパッケージです。
ロジックは Babylon.js に依存しているため、別途インストールする必要があるのでご注意ください。

```ts:babylonjsパッケージの使用例
import { Engine, Scene } from "@babylonjs/core";
import { createGaussianSplattingFromSpz } from "@spz-loader/babylonjs";

// .spz file path/url
import spzPath from "../assets/hornedlizard.spz?url";

const engine = new Engine(renderCanvas);
const scene = new Scene(engine);

// ...

const spzBuffer = await fetch(spzPath).then((res) => res.arrayBuffer());
const splat = await createGaussianSplattingFromSpz(spzBuffer, scene);
```

こんな感じでコードを書くと、次のポストにあるような GaussianSplatting データをロードできます。

https://x.com/ninisan_drumath/status/1862334844325568633

## おわりに

SPZ は GaussianSplatting の新しいフォーマットとしてとても良さそうに感じたので、私としては注目していきたいと感じています。
今回 OSS 公開という形で何かしら普及に貢献できれば幸いです。

また spz-loader は私にとって 4 シリーズ目の npm パッケージリリースとなりますが、その中でも結構良くできたし、面白い技術を使えたプロジェクトでしたので達成感が大きいです。
まだ実装できていない機能もありますし、v1.0 ではないのでまだまだ開発は続きますが、ひとまずリリースできてよかったなと感じております。

### 参考リンク

https://github.com/drumath2237/spz-loader

https://zenn.dev/drumath2237/scraps/8c3f9bf6012b54

https://scaniverse.com/news/spz-gaussian-splat-open-source-file-format

https://github.com/nianticlabs/spz