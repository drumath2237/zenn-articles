---
title: "WebAR×AIなデモアプリ\"WhatsThis AI\"で利用したGemini APIの技術詳説"
emoji: "🍉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["WebAR", "Gemini", "AR", "AI"]
published: true
publication_name: "hololab"
---

## はじめに

### 概要

私のチームでは、チーム内プロジェクトとして XR・AI 連携のユースケースを探るためのプロトタイプを開発しています。
本記事では、その試行の中で生まれた WhatsThis AI という WebAR アプリの Gemini API の活用にフォーカスして技術解説をしていきます。

### 対象読者

主に次のような方々に刺さるといいなぁと意識しながら書いてみましたが、内容に知識や文脈を多く必要とするわけではないので、その限りではありません。

- XR と AI を組み合わせたアプリケーションの事例に興味がある方
- Gemini API の機能と実装方法が知りたい方
- Web アプリ・TypeScript が分かる方

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

WhatsThis AI は、 AI と AR を掛け合わせた音声ガイド LINE ミニアプリです。LINE ミニアプリとありますが、その実態は Web アプリとなっています（LINE ミニアプリに関しては今回の趣旨ではないので割愛します）。

WhatsThis AI を使って AI と対話すると、カメラ画像に映っている場所であったり物の使い方であったりを説明してくれるだけでなく、適当な場所に 3D 矢印やテキストを配置することで、より分かりやすく説明する機能があります。
たとえば外国で電車に乗ろうとしたときに、券売機の使い方が分からなかったとしましょう。そんな時、WhatsThis AI を起動し、券売機を映して質問をすると、手続きに必要なボタンの場所を 3D テキストで表示しながら AI が説明してくれる、といったような使い方を想定しています。

次の画像は WhatsThis AI が動作している様子で、ユーザの質問に対して AI が応答している状態です。返答は音声で流れるのと同時にテキストでも表示されており、また 3D オブジェクトとして「しっぽ」を示すテキストと矢印が出ています。この 3D オブジェクトはいろんな角度から見ても、ちゃんとその場所を指し示すようになっています。

<!-- textlint-disable -->
![capture](/images/gemini-whatsthisai/whatsthis.png =400x)
*WhatsThis AIの動作画面*
<!-- textlint-enable -->

### 利用技術

利用技術をざっくりと列挙すると次のようになります。

- 8thwall
- React / React Router / MaterialUI
- Gemini Developer API（以降 Gemini API）

開発環境の部分で既にふれている通り、WhatsThis AI は 8thwall で開発されています。
8thwall は Niantic Spatial 社が開発している WebAR アプリケーションを開発するためのフレームワークで、それと一緒にブラウザ上で開発ができる「クラウドエディタ」も提供されています。今回はクラウドエディタ上で TypeScript を使って開発しています。

https://www.8thwall.com/

WhatsThis AI には AR シーン以外にもボタンやテキストなどの UI 要素もあり、それらの実装には React を使用しています。
現在の Web フロントエンド開発では React のような UI フレームワークを組み込むことは一般的になっていますが、8thwall（特にクラウドエディタ）アプリでは少し特殊というか、工夫が必要になってきます。8thwall エディタプロジェクトで React を使った所感は次の記事に載せています。

https://zenn.dev/hololab/articles/about-react-in-8thwall-editor

WhatsThis AI の、AI による応答の部分は Gemini API を用いました。他にもいろいろ選択肢はありそうですが、対話型のエージェントを実装するのに後述する Live API が相性が良かったのと、現状アプリに組み込むってなったときに Gemini API がちょうど良さそうだったのが理由です。


## Gemini APIについて

ご存じの方も多そうですが、Gemini API は Google によって開発された LLM です。テキストによる入出力だけでなく、画像・音声・動画・ドキュメントファイルなど様々な入力を受け付けられたり、高品質な画像・音声の生成も可能だったりするため、マルチモーダルな LLM です。
Gemini の機能には Web アプリやモバイルアプリなどからアクセスできるほか API や SDK が提供されており、開発者が自分のアプリに Gemini の機能を組み込むことも可能です。

https://ai.google.dev/gemini-api/docs?hl=ja

Gemini API の基本機能は特定のエンドポイントに HTTP の POST リクエストを送ると利用可能なのでとてもシンプルですが、今回使用した常時接続型の Live API については WebSocket で通信することになります。
また、Google が公式で提供している SDK もありますので、それを使えば難しいことは考えずに使えるでしょう。

ちなみに、WhatsThis AI は 8thwall クラウドエディタ上で開発しているため、外部の npm パッケージをインストールして使うことはできないので、通信部分は自前で実装しています（と言ってもそんな難しいわけではないですが）。

次のコードは、SDK を使わずに Web フロントエンドから Gemini API を呼び出す例です。

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

WhatsThis AI ではエージェントと音声を介して会話できます。動作としては Gemini API へ単発の音声データを送って応答を待つのではなく、常時音声を送信し、AI が応答タイミングを自動的に判断してデータを返してくる形になります。
これを実現しているのが、Gemini Live API という機能です。

### Live APIとは

Live API は Gemini API の強力な機能の 1 つで、（先述のとおりですが）クライアントと LLM 間で常時接続されている状態を実現できます。

https://ai.google.dev/gemini-api/docs/live

Live API で音声による対話をする場合の処理フローを次の図に示します（ざっくりですが）。

```mermaid
sequenceDiagram
    participant Client as クライアント
    participant Live as Gemini Live API
    
    Client->>Live: WebSocket接続確立
    Client->>Live: セッション設定
    
    Note over Client, Live: 常時音声送信とリアルタイム応答
    
    Client->>Live: 音声データ送信
    Client->>Live: 
    Client->>Live: 
    
    Live->>Live: 音声認識・処理

    Live->>Client: タイミングを見計らって返答
    Live->>Client: 
    Live->>Client: 
    
    Client->>Live: セッション終了
```

### WhatsThis AIとLive APIの相性

図を見ると分かるのですが、単発のリクエストではなく連続的に音声のデータチャンクを一方的に投げた後に、よしななタイミングで Gemini からも連続的な音声のデータチャンクが通知されていますね。

このように細切れのフレームで送受信されるのが LiveAPI の特徴であり、AI と一定時間何回かに分けて対話するといった用途にマッチしています。
そしておそらく応答スピードも速く、所感として比較的短めのレスポンスを返すことが多いように見受けられます。

また、LiveAPI には Intrupt という機能があり、Gemini から細切れの返答データが返ってきている途中で自分が話し始めると、Gemini が「話が途中で遮られた」ことを検知して話すのをやめます。ちょっと人間っぽいですね（人によっては嫌がられそうですが）。

### Live APIで使えるモデル

執筆時点（2025/07）では次のようなモデルが使用可能です。

- `gemini-2.0-flash-live-001`
  - 一番安定している印象があります
- `gemini-live-2.5-flash-preview`
  - 命名規則が変則ですが、これで合ってるっぽいです
- `gemini-2.5-flash-preview-native-audio-dialog`
  - thinking 対応モデルもあり

### カメラ画像と音声の送信

Live API はマルチモーダルなモデルを採用しているため、次の 3 種類のデータを入力できます。

- テキスト
- 音声
- ビデオ

このうち音声とビデオは「InlineInput」という、テキストとは区別された形式として扱われますが、結局どちらも WebSocket 経由で送信されるのには変わりありません。send するときのデータ構造が違う感じですね。
WhatsThis AI ではテキストは入力しておらず、音声とビデオを送信し続けています。

音声は PCM16 というフォーマットで送信する必要があります。これはいわゆる音声の生データ（.wav の中身）で、波形の情報が 16bit の整数値でそのまま記録されている形式ですね。サンプリングレートは入力の場合 16kHz である必要があります。
このデータを Base64 エンコードした文字列を、MIME Type（`audio/pcm;rate=16000`）を添えて送信すれば OK です。
データはよしなな長さのチャンクに区切って送信します。

https://ai.google.dev/gemini-api/docs/live-guide#audio-formats

WhatsThis AI では[Live API Web Console](https://github.com/google-gemini/live-api-web-console)の実装を参考にしています。
具体的には Web Audio API からマイク入力を取得し、Audio Worklet 内で Float32Array から Int16Array への変換をして、Base64 エンコードするような実装をしています。

一方ビデオの送信ですが、実態は JPEG 画像です。
つまり、ビデオ（概念）の各フレームを JPEG エンコードしたものをさらに Base64 エンコードして InlineInput として送信します。
WhatsThis AI の実装では、8thwall のカメラのピクセルデータを取得し、非表示状態の Canvas 要素に描画して JPEG/Base64 エンコードしています。


### テキストと音声の受信

クライアントが受信するデータ（つまり Live API から送信されるデータ）は、テキスト・音声が選択可能です。テキスト/音声の選択は、WebSocket のセッションが確立された後に送信する config の Response Modality に設定します。

```ts: Response Modalityの設定
const config = { responseModalities: [Modality.TEXT] };
```

注意すべき点として、（少なくとも執筆時点では）Live API のモデルで Response Modality を複数設定できず、Text or Audio を指定することになります。API の型情報的には複数できそうですが、おそらくモデルが対応していないんですね。
https://ai.google.dev/gemini-api/docs/live-guide#establish-connection

ところが、WhatsThis AI では AI からの返答を音声で再生しつつ文字でも表示する UI が必要でした。
最初はこの要件を実現するために、「Response Modality は Text にしておいて、受信したあとに Text to Speech を使って音声再生する」という実装をしていました。ちょうど最近、Gemini では TTS 専用のモデルもリリースされていたので試してみたかったのもありますが、さすがにレスポンスが遅く不便でした。

そこで改めてドキュメントを見直した結果、Response Modality が Audio だった場合、音声の字幕データを別途テキストとして受け取れることに気づきました。
これを使えば、Response Modality は Audio しか設定していないが、読み上げている音声のテキストも表示できるわけですね。

https://ai.google.dev/gemini-api/docs/live-guide#audio-transcription

## System Instructionのプロンプトエンジニアリング

### System Instructionとは

System Instruction は、Gemini API の機能の 1 つで、AI エージェントに対して基本的な振る舞いや役割を定義するためのプロンプトです。一般的なユーザーとのやり取りが始まる前に、AI に対してどのような存在であるべきか、どのような応答スタイルを取るべきかを設定できます。

### System Instructionによる指示と知識共有

WhatsThis AI では、System Instruction を使って AI エージェントを「カメラに写る特定のものに関するエキスパートであり、説明に必要と判断した物の位置を 3 次元的に指示できるもの」として定義しています。
直近の SusHiTech Tokyo や LODGE XR TALK では、LINE フレンズのグッズに関するエキスパートとしての文脈を与えました。

指示だけではなく、System Instruction ではそのグッズに関する知識も共有しています。
今回対象となっていたのは「コニー」というキャラクターを模した形のルームライトだったのですが、そのライトの形状や機能性について記載しています。

### プロンプトエンジニアリングにおける工夫

プロンプトエンジニアリングのプラクティスについて、Gemini のドキュメントに記載されていますので参考にしました。

https://ai.google.dev/gemini-api/docs/prompting-strategies

この中から WhatsThis AI の System Instruction では次のような工夫をしています。

- 伝えたい内容をタイトルで区切って渡す
- 対話の例示をする

伝えたい内容をタイトルで区切るというのは、本記事のような文書を書く際にも用いるテクニックです。実際にこんな感じに区切って指示を与えています。

```txt
<タスク概要>

内容

<グッズについての情報>

内容

<物体の位置の特定タスクについて>

内容

<会話の例>

内容
```

また、会話の例示はちゃんとするようにしました。特に今回は対象物にまつわる質問と、物体の位置を求めるタイミング、まったく関係がない質問をされたときの対応方法について例示しました。
プロンプトエンジニアリングにおいて例示はかなり重要な要素らしく、例示がないプロンプトを Zero-Shot プロンプト、いくつか例示があるものを Few-Shot プロンプトと呼ぶらしいです。

## Function Callingを使った空間認識の実行

### Function Callingとは

Function Calling は、AI が必要に応じて事前に定義された関数を呼び出すことができる仕組みです。AI が単純にテキストを生成するだけでなく、外部のシステムと連携して具体的なアクションを実行できるようになります。

例えば「今日の天気を教えて」という質問に対して、AI が天気予報 API を呼び出して最新の情報を取得し、それをもとに回答を生成するといった使い方が可能です。
ここでキーなのが、天気予報を取得する処理はあくまで開発者が実装していて、AI は天気予報を取得する必要がある、と判断したことをクライアントに伝えるのみになるところです。

Function Calling では、AI が必要になった情報を自動で判断してクライアント側の処理を呼び出すことができます。これにより、AI アプリケーションの可能性が広がります。

https://ai.google.dev/gemini-api/docs/function-calling?example=weather

最近話題の MCP（Model Context Protocol）と似たようなコンセプトですが、Function Calling は汎用的なプロトコルではなく、あくまでそのアプリケーション専用の関数を自前で定義するまでにとどまります。

### 空間認識をFunction Callingで実行したい

WhatsThis AI では、特定の物の場所を特定するのに Function Calling を使っています。

具体的には、物の名前（例えばルームライトのボタンなど）とカメラ画像をセットで、別の Gemini モデル（`gemini-2.0-flash-001`）に送り、Spatial Understanding（空間認識）を実行しています。
実際に使っている関数定義はこのようになっています。見てわかる通り、本当に定義だけで、関数の中身は Gemini からは見えていないですね。

```ts: FunctionCallingの定義
 {
  name: 'detect_items_points_async',
  description: '画像に写っているものの場所を特定するタスクを開始するための関数です。この処理は時間がかかるので、この関数では処理が開始したことを確認し、正常に開始されればその状態を返却します',
  parameters: {
    type: 'OBJECT',
    properties: {
      detectionTarget: {
        type: 'STRING',
        description: '位置を検出する対象物の名前。例）しっぽの電源ボタン、耳、目、えんぴつ、看板など',
      },
    },
  },
  response: {
    type: 'OBJECT',
    properties: {
      isDetectionStarted: {
        type: 'STRING',
        description: '処理が開始されたことを示す文字列',
        format: 'enum',
        enum: ['started'],
        nullable: false,
      },
    },
  },
}
```

Spatial Understanding は、画像中に映る物体の位置を特定する機能で、指定したオブジェクトのピクセル座標を返してくれます。
これは Gemini にそういう特定の機能があるのではなく、プロンプトでそのように指示をすると実行してくれる、くらいのものです。
例えば次のようなプロンプトを用いて実行します（これはかつての Gemini API のサンプルに含まれていたものを引用しています）。
```txt
Point to the ${objectName} with no more than 10 items. The answer should follow the json format: [{"point": <point>, "label": <label1>}, ...]. The points are in {x:number, y:number} format normalized to 0-1000
```
この機能により、AI が「このボタンを押してください」と言うときに、実際にそのボタンの物理的な位置に AR アノテーションを表示できます。

### Function Callingの処理フロー

WhatsThis AI における Function Calling を使った空間認識の処理フローを次に示します。

```mermaid
sequenceDiagram
    participant AR as 8thwall AR
    participant Client as クライアント
    participant Live as Gemini Live API
    participant Flash as Gemini 2.0 Flash

    Client->>Live: 音声質問 + カメラ画像
    Live->>Live: 場所の特定が必要と判断
    Live->>Client: Function Calling リクエスト
    Note over Live, Client: 物体名とカメラ画像を指定
    
    Client->>Flash: Spatial Understanding 実行
    Note over Client, Flash: 画像 + 物体名を送信
    
    Client->>Live: Function Calling レスポンス
    Note over Client, Live: 処理開始を通知
    
    Flash->>Client: 空間認識結果
    Note over Flash, Client: ピクセル座標を返却
    
    Client->>AR: AR アノテーション配置
    AR->>Client: 3D 矢印・テキスト表示
    
    Live->>Client: 音声による説明
```

処理の流れは次のようになります。

1. **AI が場所の特定が必要だと判断**
   - ユーザーから送信された音声とカメラ画像を解析
   - 3D オブジェクトによる空間アノテーションが必要か AI が自動判断

2. **Function Calling のリクエスト**
   - AI から特定の物体の位置特定を要求
   - 物体名を引数に情報を含む

3. **空間認識の実行**
   - 渡された情報をもとに、別の Gemini へのリクエストで Spatial Understanding を実行
   - 画像中の指定された物体のピクセル座標を取得

4. **処理開始の通知**
   - 空間認識が開始されたことを Function Calling のレスポンスとして Live API に返す
   - これにより AI は処理が進んでいることを認識

5. **AR アノテーションの表示**
   - 裏で空間認識が完了したら、8thwall の機能を使って AR の矢印を表示
   - ピクセル座標を 3D 座標に変換（hit-test）して AR 空間に配置

この一連の流れにより、AI による音声説明と視覚的な AR 案内が連携した、直感的なユーザー体験を実現しています。

## おわりに

WhatsThis AI というアプリのプロトタイプを開発するにあたって利用した Gemini API の機能を解説してきました。
個人的に、ここまでしっかり Gemini API の機能を使うのは初めてだったので手探りだったのですが、このプロジェクトを通して AI エージェントの実装について知見がたまってとてもよかったです。

この知見が、読者の皆様のお役に立てればとても光栄です。最後まで読んでいただきありがとうございました。
