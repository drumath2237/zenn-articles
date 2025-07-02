---
title: "WebAR × GeminiAPIなデモアプリ\"WhatsThis AI\"の技術詳説"
emoji: "🍉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["WebAR", "Gemini", "8thwall", "AR", "AI"]
published: false
publication_name: "hololab"
---

## はじめに

### 概要

私のチームでは、チーム内プロジェクトとして XR・AI 連携のユースケースを探るためのプロトタイプを開発しています。
本記事では、その試行の中で生まれた WhatsThis AI という WebAR アプリにおける GeminiAPI の活用にフォーカスして技術解説をしていきます。

### 対象読者

本記事では、主に次のような方々に刺さるといいなぁと意識しながら書いてみましたが、内容に知識や文脈を多く必要とするわけではないので、その限りではありません。

- XR と AI を組み合わせたアプリケーションの事例に興味がある方
- GeminiAPI の機能と実装方法が知りたい方

### 開発環境

WhatsThis AI の開発環境を次に示します。

- 8thwall クラウドエディタ
  - 8thwall Engine 27.4.8.427
  - React/react-dom v17
- Gemini API
  - `gemini-2.0-flash-live-001`
  - `gemini-2.0-flash-001`

## プロジェクト概要と技術構成の概観

### WhatsThis AIの概要

WhatsThis AI は、 AI と AR を掛け合わせた音声ガイド LINE ミニアプリです。LINE ミニアプリとありますが、その実態は Web アプリとなっています（LINE ミニアプリに関しては今回の趣旨とは離れるので割愛します）。

WhatsThis AI を使って AI と対話すると、カメラ画像に映っている場所であったり物の使い方であったりを説明してくれるだけでなく、適当な場所に 3D 矢印やテキストを配置することで、より分かりやすくする機能があります。
たとえば外国で電車に乗ろうとしたときに券売機の使い方が分からなかったとしましょう。そんな時、WhatsThis AI を起動して券売機を映して質問をすると、手続きに必要なボタンの場所を 3D テキストで表示しながら AI が説明してくれる、といったような使い方を想定しています。
<!-- textlint-disable -->
＜動画か画像など貼りたい＞
<!-- textlint-enable -->

### 利用技術

## LiveAPIによるAIとのリアルタイムな対話

## System Instructionのプロンプトエンジニアリング

## Function Callingを使った空間認識の実行

## 空間認識からARアノテーションまで

## おわりに

