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

利用技術とざっくりと列挙すると次のようになります。

- 8thwall
- React / React Router / MaterialUI
- Gemini Developer API（以降 Gemini API）

開発環境の部分で既にふれている通り、WhatsThis AI は 8thwall で開発されています。
8thwall は Niantic Spatial 社が開発している WebAR アプリケーションを開発するためのフレームワークで、それと一緒にブラウザ上で開発ができる「クラウドエディタ」も提供されています。
今回はクラウドエディタ上で TypeScript を使って開発しています。

https://www.8thwall.com/

WhatsThis AI には AR シーン以外にもボタンやテキストなどの UI 要素もあり、それらの実装には React を使用しています。
現在の Web フロントエンド開発では React のような UI フレームワークを組み込むことは一般的になっていますが、8thwall（特にクラウドエディタ）アプリでは少し特殊というか、工夫が必要になってきます。8thwall エディタプロジェクトで React を使った所感は次の記事に載せています。

https://zenn.dev/hololab/articles/about-react-in-8thwall-editor

WhatsThis AI の、AI による応答の部分は Gemini API を用いました。他にもいろいろ選択肢はありそうですが、対話型のエージェントを実装するのに後述する Live API が相性が良かったのと、現状アプリに組み込むってなったときに Gemini API がちょうど良さそうだったのが理由です。


## Gemini APIについて

ご存じの方も多そうですが、Gemini API は Google によって開発された LLM です。テキストによる入出力だけでなく、画像・音声・動画・ドキュメントファイルなど様々な入力を受け付けられたり、高品質な画像・音声の生成も可能となっていたりしているため、マルチモーダルな LLM であるという特徴があります。
Gemini の機能には Web アプリやモバイルアプリなどからアクセスできるほか API や SDK が提供されており、開発者が自分のアプリに Gemini の機能を組み込むことも可能です。

https://ai.google.dev/gemini-api/docs?hl=ja

Gemini API の基本機能は特定のエンドポイントに HTTP の POST リクエストを送ると利用可能なのでとてもシンプルですが、今回使用した常時接続型の Live API については WebSocket で通信することになります。
また、Google が公式で提供している SDK もありますので、それを使えば難しいことは考えずに使えるでしょう。

ちなみに、WhatsThis AI は 8thwall クラウドエディタ上で開発しているため、外部の npm パッケージをインストールして使うことはできないので、通信部分は自前で実装しています（と言ってもそんな難しいわけではないですが）。

次のコードは、SDK を使わずに Web フロントエンドから GeminiAPI を呼び出す例です。

```ts: Gemini APIを呼び出すTypeScriptのコード例
const apiKey = "<API KEY>";
const apiVersion = "v1beta";
const modelName = "gemini-2.5-flash";
const url = `https://generativelanguage.googleapis.com/${apiVersion}/models/${modelName}:generateContent`;
const requestBody = {
  contents: [{ parts: [{ text: "あなたの名前は何ですか？" }] }],
};

const res = await fetch(url, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-goog-api-key": apiKey,
  },
  body: JSON.stringify(requestBody),
}).then((res) => res.json());

const answer = res?.candidates?.[0]?.content?.parts?.[0]?.text;
console.log(answer); // -> 私はGoogleによってトレーニングされた、大規模言語モデルです。
```

## LiveAPIによるAIとのリアルタイムな対話

### Live APIとは

### WhatsThis AIとLive APIの相性

### カメラ画像と音声の送信

### テキストと音声の受信

## System Instructionのプロンプトエンジニアリング

### System Instructionとは

### System Instructionによる指示と知識共有

### プロンプトエンジニアリングにおける工夫

## Function Callingを使った空間認識の実行

### FunctionCallingとは

### 空間認識をFunctionCallingで実行したい

### FunctionCallingの処理フロー

## 空間認識からARアノテーションまで

## おわりに

