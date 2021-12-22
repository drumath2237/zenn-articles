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

例えば以下のようなディレクトリ構造があるとします。
(主要なファイルのみ示しています)

```
/
├─ models/
│  └─ boombox.glb
├─ src/
│  ├─ main.ts
│  └─ style.scss
├─ index.html
├─ tsconfig.json
└─ package.json
```

`yarn create vite`を使うとこのようなファイルが出来上がります。
ここで models の中には glb ファイルがあり、これを`main.ts`で静的アセットとして読み込むことを考えます。

Vite はアプリをビルドするときにファイルパスやファイル名を変更するため、
開発環境で相対パスを決め打ちにするとビルド環境で動かない可能性があります。
例えば上のプロジェクトで`yarn build`すると、`/index.html`は`/dist/index.html`に、`/models/boombox.glb`は`/dist/assets/boombox.713b197f.glb`になるみたいです。
詳しくは公式ドキュメントを参照してください。

https://vitejs.dev/guide/assets.html

```ts:main.ts
import glbModel from '../models/boombox.glb?url';
```

この`glbModel`は string 型になっており、プロジェクトルートからの相対パスが入っています。ビルド前の状態では`"/models/boombox.glb"`が入っていますが、ビルドされた後は`"/assets/boombox.713b197f.glb"`になります。

## Babylon.js の SceneLoader で扱えるようにする

Babylon.js は SceneLoader という仕組みで glb ファイルなどの静的アセットを読み込みますが、ちょっと API の仕様が変わっています。
次のように使用します。

```ts:main.ts
import * as BABYLON from '@babylonjs/core';
import '@babylonjs/loaders/glTF';

import glbModel from '../models/boombox.glb?url';

// something ...

BABYLON.SceneLoader.AppendAsync(dirName, fileName, scene)
  .then(() => {
    console.log('model loaded');
  })
  .catch(console.error);
```

SceneLoader を使って glb ファイルを読み込むときは、`@babylonjs/loaders`パケージが必要になるので、npm か yarn でインストールしてください。
そのあと`@babylonjs/loaders/glTF`をインポートするのも忘れないようにしてください。

`SceneLoader.AppendAsync`メソッドは第 1 引数にフォルダのパス、第 2 引数にファイル名を取りますが、ここの仕様が変わっているというか、なぜパス名を一括で指定させてくれないのか......という気持ちになります。
特に Vite のようなビルド前と後でパスが変わるような場合もっとめんどくさいです。なのでパスを適当にパースするというのが必要になるわけです。本記事の主題ですね。ということでコード例を示します。

```ts:main.ts
import * as BABYLON from '@babylonjs/core';
import '@babylonjs/loaders/glTF';

// eslint-disable-next-line import/no-unresolved
import glbModel from '../models/boombox.glb?url';

import './style.scss';

const renderCanvas = <HTMLCanvasElement>document.getElementById('renderCanvas');

const main = (canvas: HTMLCanvasElement) => {
  const engine = new BABYLON.Engine(canvas, true);
  const scene = new BABYLON.Scene(engine);

  // something ...

  // parse url
  const folderName = glbModel.split('/').slice(0, -1).join('/').concat('/');
  const fileName = glbModel.split('/').slice(-1)[0];

  BABYLON.SceneLoader.AppendAsync(folderName, fileName, scene)
    .then(() => {
      console.log('model loaded');
    })
    .catch(console.error);

  engine.runRenderLoop(() => {
    scene.render();
  });
};

main(renderCanvas);
```

# おわりに

Vite 環境で Babylon.js の SceneLoader を使う、簡単な tips について解説しました。
多くの人が助かる、みたいな話題ではないのですが、備忘録としてどこかに記録しておきたかったのでアドベントカレンダーの機会に書く機運があって良かったです。

この知見が、皆さんのお役に少しでも立てれば幸いです。

## 参考文献

https://vitejs.dev/guide/assets.html

https://doc.babylonjs.com/toolsAndResources/assetLibraries/availableMeshes#from-the-playground-scenes-folder

https://doc.babylonjs.com/typedoc/classes/babylon.sceneloader