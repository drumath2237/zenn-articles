---
title: "非エディタ環境でPlayCanvasを使う（Using the Engine Standalone）"
emoji: "🥑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["playcanvas", "vite", "pnpm", "typescript"]
published: false
---

:::message
本記事は[にー兄さんアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/ninisan-2024)の3日目の記事です。
:::

## はじめに

### TL;DR

PlayCanvas Engine を使うことで、エディタを使わずに Vite / TypeScript な環境で 3D シーンを実装できる。

### 概要

本記事は PlayCanvas Engine を用いて、エディタを使わずに 3D シーンを実装する方法をご紹介します。
PlayCanvas といえば高機能なエディタが特徴的なフレームワークではありますが、コードベースのみでも実装できるという選択肢の幅も魅力なのかなと感じています。

:::message
🔰筆者はPlayCanvasに入門したばかりの初心者です。
もし間違っていることを書いていた場合、ご指摘いただけると幸いです。
:::

### 想定読者

- 普段 PlayCanvas をエディタ環境で使っている方
- PlayCanvas に興味があるエンジニア

など。

### 検証環境

- Windows 10 Home
- PlayCanvas 2.2.2
- Node.js 20.9.0
- pnpm 9.13.2
- Vite 6.0.1

### サンプル

サンプルプロジェクトを GitHub にて公開しています。合わせてご覧ください。

https://github.com/drumath2237/playcanvas-standalone-testbed

## Viteでプロジェクトのセットアップ

今回は、Vite を使って Web フロントアプリを作っていきます。
次のコマンドを実行して Vite のプロジェクトを作っていきますが、
Framework は Vanilla、Language は TypeScript を選択してください。

```sh
pnpm create vite@latest
```

そして作成したプロジェクトディレクトリへ移動して、依存パッケージを解決します。

```sh
pnpm i
```

PlayCanvas を使うために、HTML にいい感じの canvas 要素を配置しましょう。

```html:index.html
<!doctype html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Vite + TS</title>
</head>

<body>
  <canvas id="renderCanvas"></canvas>
  <script type="module" src="/src/main.ts"></script>
</body>

</html>
```

```css:src/style.css
html,
body,
#renderCanvas {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
  overflow: unset;
  border: none !important;
  outline: none !important;
  display: block;
}
```

そして、この Canvas 要素を TypeScript コードから取得できるところまでを確認しましょう。

```ts:src/main.ts
import "./style.css";

const main = () => {
  const renderCanvas =
    document.querySelector<HTMLCanvasElement>("#renderCanvas");
  if (!renderCanvas) {
    return;
  }

  console.log("hello");
};

main();
```

この状態で dev サーバを起動し、開発者ツールからログが出力されているのを確認できれば大丈夫です。

```sh
pnpm dev
```

## PlayCanvas Engineで3Dシーンを実装

## おわりに

### 参考文献
