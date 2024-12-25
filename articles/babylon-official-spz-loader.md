---
title: "Babylon.js 7.37.2から公式でSPZを読み込めるようになりました！"
emoji: "🦝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["babylonjs", "spz", "gaussiansplatting"]
published: false
---

:::message
この記事は[Babylon.jsアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/babylonjs)の22日目の記事です。
:::

## はじめに

### TL;DR

- Babylon.js のローダで spz 形式の GaussianSplatting データを読み込めるようになりました
- 標準のアセット読み込み機能でそのまま使える（@babylonjs/loaders/SPLAT が必要）

### 概要

本記事では、Babylon.js で最近リリースされた SPZ ローダ機能をご紹介します。
実装の背景や使い方、そして気になってことを自分なりにまとめてみました。

私は個人的に、Babylon.js 向けの spz ローダをリリースしたばかりだったのですが、
早くも公式で実装されましたね。

### 検証環境

- Windows 11/10 Home
- @babylonjs/core 7.40.2
- @babylonjs/loaders 7.40.2
- typescript 5.7.2
- vite 6.0.2

## Babylon.js公式のspzローダ

<!-- 当該プルリク -->
<!-- フォーラムでのアナウンス -->

https://github.com/BabylonJS/Babylon.js/releases/tag/7.37.2

https://github.com/BabylonJS/Babylon.js/pull/15849

## SPZを読み込んでみる

```ts
import "./style.css";

import { Engine, loadAssetContainerAsync, Scene } from "@babylonjs/core";
import "@babylonjs/loaders/SPLAT";

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

  await loadAssetContainerAsync(spzUrl, scene);
};

main();
```

## 使ってみて気になったところ

## おわりに

### 参考文献
