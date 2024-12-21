---
title: "Awaitableの使いどころとしてWebビルド対応Unityパッケージがありそう"
emoji: "🚦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "unity6", "csharp", "awaitable", "asyncawait"]
published: false
---

## はじめに

### TL;DR

- Unity6 以降の
- Web ビルドをターゲットに含む
- Unity パッケージ

という条件の環境で Awaitable が使えると嬉しいかもしれないと考えた。

### 概要

本記事は Unity6(2023)から使える新機能である Awaitable について、
その使いどころを考えてみた内容です。
趣旨として「こうするべきだ！」というような主張をするような意図はなく、
それこそお風呂に入ってるとき「Awaitable ってどこで使えるんだろうなぁ」と考えてた時に思いついた内容を書いてみたものです。

まだ新しい機能なのと、筆者も使い込んでいるわけではないので、
もし間違った内容を書いてしまっていたらご指摘いただけると幸いです。
また、Unity6 は LTS がリリースされているものの、
Awaitable の機能は今後も強化されていく可能性がありますので、
あくまで執筆現在（2024/12）の情報であることにご注意ください。

### 検証環境

- Windows 10 Home
- Unity 6000.0.31f1
- Chrome for Windows 131.0.6778.140

### サンプル

サンプルコードを次の GitHub リポジトリ及び GitHub Pages で公開しております。

https://github.com/drumath2237/Web-Awaitable-testbed

## Awaitableとは

## Webビルドにおける非同期処理の課題とその解決

## Awaitableがいい感じかも

## おわりに

### 参考文献