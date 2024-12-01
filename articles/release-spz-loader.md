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

本記事では、最近筆者がリリースした spz-loader という npm パッケージの概要をご紹介する内容となっています。
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

ブログにも書いてある通り、GaussianSplatting データが保持するデータについて、次のような工夫をしています。

- 各データをその性質に基づいて適切な精度に収めている
  - 従来使われていた.ply ではすべて double や float で保持していた
- gzip で圧縮している
  - 列ベースでバイナリデータを格納しているため圧縮効率が良いらしい

もともと、GaussianSplatting に使われてきたファイルフォーマットというのは
色々なものがありました。
一番スタンダードなのは原著論文でも使用されていた PLY ファイルですが、
他にも Compressed PLY, splat, ksplat などといった独自のフォーマットも存在します。
そういう状況下では、GaussianSplatting を扱うソフトウェアは複数のフォーマットに対応しなくてはならず、
スタンダードなデータフォーマットを欲する声も上がっていました。

そんな中、Niantic は SPZ フォーマットを提案します。
そして既に SPZ は同社が提供する Scaniverse や Niantic Studio といったプラットフォームで利用可能です。
筆者としては、SPZ の登場により GaussianSplatting データの標準化が進むと嬉しいです。

## spz-loaderでできること

## spz-loaderで採用されている技術

### Niantic/spzのwasmコンパイル

### wasmを含んだnpmライブラリ

### pnpm-workspaceを用いたモノレポ構成

### lerna-liteによるリリースフロー

## おわりに

### 参考文献
