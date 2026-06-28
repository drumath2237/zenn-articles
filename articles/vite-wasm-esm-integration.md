---
title: "Vite 8.1の新機能であるWebAssembly ES Module Integrationについて"
emoji: "🌈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vite", "wasm", "esmodule", "javascript"]
published: false
---

## はじめに

### TL;DR

- Vite で、`.wasm`ファイルから直接関数をインポートできるようになった
- TypeScript コンパイラではエラーが出るので、それをケアする必要がある
- 関数の型情報は提供しないので、うまくやる必要がある

### 概要

米国時間の 2026 年 6 月 23 日に、[Vite 8.1 のリリースブログ](https://vite.dev/blog/announcing-vite8-1)が公開されました。
その中で「Wasm ESM integration Support」という項目が気になったので試しに使ってみることにしました。

https://vite.dev/blog/announcing-vite8-1#wasm-esm-integration-support

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

## Viteで使ってみる

### RustでWASMビルドする

### Viteのプロジェクトにインポートしてみる

### `allowArbitraryExtensions`の設定をする

## おわりに

### 参考文献
