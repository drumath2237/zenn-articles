---
title: "Vite(vanilla-ts)環境のBabylon.jsで静的なglbをロードするtips"
emoji: "⚡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["babylonjs", "typescript", "vite", "gltf"]
published: false
---

# はじめに

## TL;DR

Babylon.js の SceneLoader は
Vite の静的アセットハンドリングの仕組みと相性が悪かったので
url をパースするといった工夫が必要です。

## 概要

TypeScript は JavaScript の柔軟性を継承しながらも堅牢な型システムを備えているおかげで開発体験の向上が期待でき、昨今の Web 開発において一定の地位を築いています。
一方で環境構築においては Webpack の設定や Node.js、JavaScript のモジュールシステムなどの知識が必要で、始めるには敷居が高いようにも感じていました。

しかし最近の JavaScript フレームワークを使用すれば
プロジェクト作成時に TypeScript の設定を済ませてくれるものもあるみたいです。
なので Vue や React を利用する分にはあまり不自由しないのではないでしょうか。
そしてノンフレームワークで TypeScript の開発をするときは自分で環境を構築する必要があるのですが、Vite を使うことでその問題も解決します。

Vite の良さを語ってしまうとそれだけで記事が 1 つ書けてしまうので割愛しますが、自分は Babylon.js の開発をするときによく使っています。
Babylon.js は WebGL ライブラリの 1 つで TypeScript フレンドリーなのが特徴の 1 つです。簡単なデモプロジェクトであれば特に JS フレームワークを使う必要はないので、基本的にピュア TypeScript で開発します。
基本的にはこの構成で問題ないのですが、SceneLoader という物を使って静的アセットをロードしようとするときに、API の仕様がすこし変わっていたために工夫が必要でした。今回はその解決策の 1 つを備忘録的に記事にしようと考えました。

## 扱う内容と対象読者

本記事で扱う内容は、「Vite 環境の Babylon.js で glb モデルを読み込む」というものです。
簡単な備忘録なので高度な知識は必要ありませんが、TypeScript の文法や Babylon.js について事前に理解していることが望ましいです。

## 想定環境

想定する環境は以下の通りです。

- TypeScript
- Babylon.js 5.0.0-alpha.60
- Vite

プロジェクトは次のコマンドで作成した構成を基本とします。

```
yarn create vite <app name> --template vanilla-ts
```

また本記事の内容をサンプルとして公開していますので、合わせてご確認ください。

https://github.com/drumath2237/vite-babylon-gltf-sandbox

# Vite ＋ Babylon.js で glb を読み込む

## Vite の静的アセット読み込みの仕組み

## Babylon.js の SceneLoader で扱えるようにする

# おわりに

## 参考文献
