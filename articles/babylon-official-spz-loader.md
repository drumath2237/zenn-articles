---
title: "Babylon.js 7.37.2から公式でSPZを読み込めるようになりました！"
emoji: "🦝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["babylonjs", "spz", "gaussiansplatting"]
published: true
---

:::message
この記事は[Babylon.jsアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/babylonjs)の22日目の記事です。
:::

## はじめに

### TL;DR

- Babylon.js のローダで spz 形式の GaussianSplatting データを読み込めるようになった
- 標準のアセット読み込み機能でそのまま使える（@babylonjs/loaders/SPLAT が必要）

### 概要

本記事では、Babylon.js で最近リリースされた SPZ ローダ機能をご紹介します。
使い方と気になってことをまとめてみました。

私も個人的に Babylon.js 向けの spz ローダをリリースしたのですが、早くも公式で実装されましたね。

### 検証環境

- Windows 11/10 Home
- @babylonjs/core 7.41.1
- @babylonjs/loaders 7.41.1
- typescript 5.7.2
- vite 6.0.5

## Babylon.js公式のspzローダ

Babylon.js にもともと実装されていた`SPLATFileLoader`で SPZ 形式のファイルもサポートされるようになりました。当該プルリクエストは次の通りです。

https://github.com/BabylonJS/Babylon.js/pull/15849

SPZ 自体については、ネットにいくつか記事がありますので説明は割愛します。
簡単に言うと、最近発表された 3D Gaussian Splatting のデータ形式です。

https://cgworld.jp/flashnews/202411-Niantic-spz.html

## SPZを読み込んでみる

それでは SPZ を読み込んでみましょう。
コードを書く前に、`@babylonjs/core`と同じバージョンの`@babylonjs/loaders`をインストールしてください。

```sh
# npm
npm i @babylonjs/loaders

#pnpm
pnpm add @babylonjs/loaders
```

ファイルローダの仕組みに組み込まれているので、従来のアセットインポートと同じようなコードで実現できます。次のコードでその例を提示していますが、こちらは Vite を使っていることを前提としています。

```ts:Babylon.jsでSPZを読み込んで表示するサンプル
import "./style.css";

import { Engine, loadAssetContainerAsync, Scene } from "@babylonjs/core";

// GaussianSplattingのローダを有効化
import "@babylonjs/loaders/SPLAT";

// SPZファイルのパス(公式サンプルデータのracoonfamily)
import spzUrl from "../assets/racoonfamily.spz?url";

const main = async () => {
  const renderCanvas =
    document.querySelector<HTMLCanvasElement>("#renderCanvas");
  if (!renderCanvas) {
    return;
  }

  const engine = new Engine(renderCanvas);
  const scene = new Scene(engine);

  scene.createDefaultCameraOrLight(true, true, true);

  window.addEventListener("resize", () => engine.resize());
  engine.runRenderLoop(() => scene.render());

  // ここで読み込んでる
  await loadAssetContainerAsync(spzUrl, scene);
};

main();
```

実行すると、SPZ が読み込まれて 3D Gaussian Splatting として表示されました。

![alt text](/images/babylon-spz/racoonfamily.png)

## 使ってみて気になったところ

ちゃんと読み込めているように感じますが、よく見てみると表面がざらついているような見た目になっていました。もしかするとガウシアンの回転を算出するロジックが間違っている可能性がありますね。

![alt text](/images/babylon-spz/zoomracoon.png)

また、Scanivsere で表示したときと左右反転しているような見た目になっています。
これは SPZ の座標系に関するドキュメントがなく、何を正としてよいのか判断できていないゆえに起こっているようです。
座標系に関しては、すでにたるこすさんが spz へ issue を上げてくださっています。

https://github.com/nianticlabs/spz/issues/14

## おわりに

ついに Babylon.js 公式でも SPZ ファイルローダが実装・公開されました。
SPZ は GaussianSplatting のファイルフォーマットとして納得感が大きいですし、標準フォーマットとしていい感じに普及してくれるといいなぁと考えています。

### 参考文献

https://github.com/BabylonJS/Babylon.js/releases/tag/7.37.2

https://forum.babylonjs.com/t/new-feature-niantic-spz-gaussian-splatting-and-spherical-harmonics/55124