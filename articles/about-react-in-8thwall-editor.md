---
title: "8thwallエディタプロジェクトでReactテンプレートを使って感じたメリットと制限"
emoji: "🧱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["8thwall", "react", "WebXR", "TypeScript", "AR"]
published: false
publication_name: "hololab"
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
また、今回は AFrame を使っていることを前提として進めていきます（Three.js や Babylon.js を使っていても使えます）。

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
<!-- ... -->
```

```js:/app.js
// Copyright (c) 2022 8th Wall, Inc.

import * as ReactApp from './react-app'

ReactApp.render()
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

`react-app.tsx`を見ると分かる通り、CDN でグローバルインポートされた React 名前空間にアクセスしていますね。

## React導入のメリット

### 凝ったランディングページやページ遷移が実装しやすい

React は DOM 要素とのインタラクションが発生するような複雑な UI 操作を、
宣言的 UI とリアクティブプログラミングによってシンプルに記述できる特徴があります。
これは 8thwall に導入したときにも享受できるメリットであり、多くの 8thwall プロジェクトでは起動するとすぐに 3D シーンが表示されますが、ランディングページを 1 枚かませたい時に便利です。

「作りやすい」という表現の通り、実際には React を使わなくてもランディングページは作成可能です。
しかしページごとに分けた HTML を動的に読み込んだり、読み込んだ HTML を破棄して別の HTML を読み込んだり、ページ遷移での History を管理するのが結構大変になってしまいますね。
これはつまり React がやっていることを仮想 DOM を使わずに自前でやっているのと同義ですから......。

また HTML を動的に読み込んでページ遷移できるため、例えば複数の AFrame コンテンツを URL によって出し分けるなんてことも可能です。
React テンプレートでは HTML ファイルを React コンポーネントとして使える仕組みが作られており、それを利用すると相対パスで指定した AFrame コードを TSX に埋め込めます。

```tsx:/views/scene.tsx(一部抜粋・変更)
const Scene = withRouter(({match}) => (
  <React.Fragment>
    <FloatingBackButton />
    <AFrameScene sceneHtml={require('./cube.html')} />
  </React.Fragment>
))
```

### Reactのエコシステムが使える

React はいまや[Webフロントエンドで使われる最もポピュラーなフレームワーク](https://2024.stateofjs.com/en-US/libraries/front-end-frameworks/)と言えますし、そのため React を使っている前提で使用可能なライブラリがたくさんあります。
8thwall の React テンプレートではその中でも ReactRouter によるページ遷移と MaterialUI による UI デザインが元から可能になっています。

MaterialUI は React ベースのコンポーネントライブラリであり、カスタマイズ可能な UI コンポーネントがすでに多数用意されています。
https://mui.com/
リッチなダイアログやボタンをデザインしたい時に MaterialUI が使えるのは便利ですね。
MaterialUI を使う際もまた、グローバル名前空間にアクセスする必要があります。

```tsx
declare let MaterialUI: any

const {
  AppBar,
  Button,
  Card,
  CardContent,
  Container,
  CssBaseline,
  Fab,
  Grid,
  IconButton,
  SvgIcon,
  ThemeProvider,
  Toolbar,
  Typography,
} = MaterialUI
```

## 8thwallエディタにおける環境制約

### TypeScriptサポート

React テンプレートを使っていて個人的にツラいと感じたポイントは、型が効かないところです。先述の通り 8thwall では TypeScript や TSX が使えますが、自分でインポートしたライブラリには型がつきません。ですので、しっかり型宣言をすることがなければ、基本的には`any`で扱うことになります。

ただ、TS がコンパイルされるときには型チェックが走りますので、型宣言されていないグローバル名前空間を使うとコンパイラに怒られてしまうので、わざわざ`declare`しないといけないわけですね......。

![img](/images/about-react-in-8thwall-editor/react-type-error.png)

また、型定義ファイルをインポートできないようですので、外部からとってきた`.d.ts`で解決する方法も取れません。

### React Routerのバージョン

React Router の現在（2025/02/10）の最新バージョンは 7.x 系ですが、8thwall の Hosted Packages では v5 が採用されているみたいです。
React テンプレートの中では`withRouter`API を使ってページコンポーネントを定義していますが、この API は現在非推奨となっているため、あまりネット上の文献では見られなくなりました。

Hosted ではなく別途 CDN でインポートしようとしたのですが、わたしが試したときは CDN の URL がわからず、少なくとも v6 にあげるところまでしかできませんでした。
また v6 でも破壊的変更が加わっているようで、テンプレート全体にコンパイラで検知されないエラーが発生していたため、バージョンアップするのはあきらめてしまいました......。

## おわりに

8thwall の AFrame: React テンプレートを使用した感触についてご紹介しました。
クラウドエディタはとても特殊な環境で独特の癖があり、複雑なアプリケーションを作るには Self Hosted を選択するほうが良さそうに感じていますが、クラウドエディタだとホスティングについて考えなくていいのがメリットですね。

この内容がお役に立てれば幸いです。

### 参考文献

https://www.8thwall.com/8thwall/react-aframe

https://www.8thwall.com/docs/guides/advanced-topics/hosted-packages/
