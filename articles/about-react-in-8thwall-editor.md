---
title: "8thwallエディタプロジェクトでReactテンプレートを使うメリットとSelf-Hostedと比較した制約について"
emoji: "🧱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["8thwall", "react", "WebXR", "TypeScript", "AR"]
published: false
---

## はじめに

### TL;DR

- React テンプレートを使うことで、8thwall と組み合わせた環境をすぐ構築できる
- React 導入によって、便利な React のエコシステムも使えるようになる
- DOM 操作が楽になり、インタラクションが必要なランディングページなどが作れる
- クラウドエディタでは TypeScript サポートが乏しいのでコーディングはしにくい

### 概要

本記事では 8thwall が公開している React を使ったテンプレートプロジェクトを使ってみて、
React を導入することで得られる恩恵と制約についてご紹介します。

ここでいう React テンプレートというのは、「AFRame: React」という名前で 8thwall のプロジェクトライブラリに公開されているものを指します。
プロジェクトライブラリには他にも React を使ったものがありますが、基本的に考え方が同じであるのと、こちらのテンプレートが一番シンプルに感じたので採用しています。

https://www.8thwall.com/8thwall/react-aframe

本記事ではクラウドエディタでの話をスコープとしていますので、Self-Hosted 環境や Niantic Studio での所感については触れません。

### 対象読者

- 8thwall。JS/TS についてある程度理解があり、触ったことがある方
- 8thwall に React のような JS フレームワークを採用することに興味関心がある方

### 検証環境

- 8thwall クラウドエディタ
  - 8thwall Engine v27.2.6.427
  - React/react-dom v17
  - React Router v5
  - Material UI

## ReactテンプレートにおけるReactの導入方法

8thwall には Hosted Packages という仕組みがあります。

https://www.8thwall.com/docs/guides/advanced-topics/hosted-packages/

クラウドエディタで`head.html`を開いたときに、`meta`タグを使って必要なライブラリをインポートできる機能のことを指しています。

```html:head.htmlではAFrameをHosted Packagesとしてインポートしている
<meta name="8thwall:renderer" content="aframe:1.5.0" />
```

リンク先のドキュメントにも記載がありますが、Hosted Packages には 8thwall や AFrame 周辺ライブラリに限らず、React や Vue といった JS フレームワークも含まれるようです。
Hosted Packages を meta タグに指定すると CDN 経由でインポートされる仕組みのようですので、React の場合は次のように記載します。

```html:/head.html(抜粋)
<!-- Copyright (c) 2022 8th Wall, Inc. -->

<meta name="8thwall:package" content="@react.react:17.0.0" />
<meta name="8thwall:package" content="@react.react-dom:17.0.0" />
<meta name="8thwall:package" content="@react.react-router-dom:6" />
```

```tsx: /react-app.tsx(抜粋)
// ...

declare let React: any
declare let ReactDOM: any
declare let ReactRouterDOM: any

const {BrowserRouter, Route, Switch} = ReactRouterDOM

// ...

const render = () => {
  document.body.insertAdjacentHTML('beforeend', '<div id="root"></div>')
  ReactDOM.render(
    <MaterialUIApp>
      <App />
    </MaterialUIApp>,
    document.getElementById('root')
  )
}

export {render}
```

```js:/app.js
// Copyright (c) 2022 8th Wall, Inc.

import * as ReactApp from './react-app'

ReactApp.render()
```

`react-app.tsx`を見ると分かる通り、CDN でグローバルインポートされた React 名前空間にアクセスしていますね。

## React導入によって実現すること

## 8thwallエディタにおける環境制約

## おわりに

### 所感

### 参考文献