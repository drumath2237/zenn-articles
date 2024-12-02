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

### サンプル

サンプルプロジェクトを GitHub にて公開しています。合わせてご覧ください。

https://github.com/drumath2237/playcanvas-standalone-testbed

## Viteでプロジェクトのセットアップ

## PlayCanvas Engineで3Dシーンを実装

## おわりに

### 参考文献
