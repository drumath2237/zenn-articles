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

## JSからC++へTypedArrayを渡す

## C++のvectorデータをJSからTypedArrayとして参照する

## 注意：ALLOW_MEMORY_GROWTHを使っている場合

## おわりに

### 参考文献
