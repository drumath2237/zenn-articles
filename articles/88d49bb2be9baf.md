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

## 想定環境

# Vite ＋ Babylon.js で glb を読み込む

## Vite の静的アセット読み込みの仕組み

## Babylon.js の SceneLoader で扱えるようにする

# おわりに

## 参考文献
