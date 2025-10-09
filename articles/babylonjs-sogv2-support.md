---
title: "Babylon.js 8.30.4でSOG v2の読み込みがサポートされました！"
emoji: "🥘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["gaussiansplatting", "babylonjs", "sogs", "sog"]
published: false
---

## はじめに

### TL;DR

- Babylon.js で SOG v2 のファイルフォーマットが読み込めるようになった
- 他の Gaussian Splatting フォーマットと同様に`@babylonjs/loaders`パッケージが必要

### 概要

先日、Babylon.js における SOG v2 フォーマットの読み込みをサポートするプルリクがマージされ、v8.30.4 にてリリースされました🎉。SOG は界隈でも結構話題になっていたので、個人的にうれしいアプデですね。

https://github.com/BabylonJS/Babylon.js/pull/17212

ということで、この機会に SOG の紹介と Babylon.js での扱い方を共有していきます。

### 検証環境とサンプルプロジェクト

本記事を執筆するにあたって検証に使用した環境は次の通りです。

- Babylon.js 8.30.5
- Babylon.js Loaders 8.30.5

また実際に動くサンプルをご用意しましたので、ご興味があれば次の GitHub リポジトリも併せてご確認ください。

https://github.com/drumath2237/babylon-sogv2-support-sandbox

## SOG v2

SOG v2 というのがどのようなファイルフォーマットなのか、ざっくりですが解説していきます。

### Gaussian Splattingとファイルフォーマット

3D Gaussian Splatting は撮影された写真とカメラの姿勢を学習して作られる”ガウシアン”のパラメータから自由視点のフォトリアルな画像をリアルタイムに生成する技術で、近年注目を集めています。正確性を欠く説明になりますが、ガウシアンとは空間的に広がりを持った点群のようなイメージで、それに色を付けたり引き延ばしたりすることで 3D シーンのようなものを構築します。

原著論文では当初（今でも広く使われていますが）`.ply` というファイルフォーマットを使って Gaussian Splatting のパラメータを保存していました。しかし原著論文で使われていたフォーマットではありとあらゆるパラメータを double(float64)型で保存しており、特に球面調和関数の係数配列の影響でかなりサイズが大きくなっていました。

https://github.com/graphdeco-inria/gaussian-splatting

そこで Gaussian Splatting を扱うエンジニアはそれぞれ、より圧縮効率の良いファイルフォーマットを定義したうえで自作のライブラリやビューアへそれを組み込んでいきました。代表的なものでは`.splat`ファイルでしょうか。
なにぶん Gaussian Splatting は新しい技術ゆえに、画像における PNG や JPEG、3D モデルにおける FBX や glTF といった、スタンダードなファイルフォーマットは定まらないままになっていました。しかしここ 1 年くらいでは、Niantic Spatial inc.の主導する`.spz`を始めとして標準的なフォーマットを定義する動きが見られています。SPZ も最近は[面白い動き](https://github.com/KhronosGroup/glTF/pull/2531)がありますが、本記事の趣旨とずれるので割愛します。

https://scaniverse.com/news/spz-gaussian-splat-open-source-file-format

### SOGSとSOG v2

先述のような流れを受け、PlayCanvas が新しいフォーマット仕様を策定し、SOGS を発表しました。

https://blog.playcanvas.com/playcanvas-adopts-sogs-for-20x-3dgs-compression/

<!-- textlint-disable -->

PlayCanvas の SOGS（Self Organized Gaussians）は ECCV 2024 という学会で発表された「Compact 3D Scene Representation via Self-Organizing Gaussian Grids」という論文をもとに作られたフォーマットです。


<!-- textlint-enable -->

https://fraunhoferhhi.github.io/Self-Organizing-Gaussians/

SOGS は Gaussian Splatting の各パラメータをソートし、クラスタを構成しながら近接する変化量が滑らかになるように 2D グリッド上に配置するという方法をとっています（正確に理解できてない）。そしてそれらを既存のエンコーディング処理をして画像データとして出力することで高効率に圧縮されるというものでした。PlayCanvas のブログのタイトルには、PLY で保存するよりも 20 倍近くファイルサイズが小さくなると書いてありますね（すごい）。
たしかに画像のエンコードは昔から研究されていますし、ホワイトノイズのような画像よりもある程度滑らかな画像のほうが圧縮率は上がりそうですよね。

SOGS のブログが発表された約 4 か月後、PlayCanvas は、より洗練された圧縮フォーマットとして「SOG」を発表します。SOGS と SOG で名前がほぼ同じなので注意ですが、前者が Self Organized Gaussians なのに対し、後者は**Spatially Ordered Gaussians**の略称です。GitHub の issue なんかでは SOG v2 と表記されることもあり、SOGS と見間違えやすいのでここでは SOG v2 として表記します。

https://blog.playcanvas.com/playcanvas-open-sources-sog-format-for-gaussian-splatting/

SOG v2 は SOGS の時と比べて`meta.json`のスキーマが変わったりしていますが、`.sog`という拡張子が定義され、すべてのファイルが 1 つにまとまって扱いやすくなったのは大きいです。というのも、SOGS では各パラメータが焼きこまれた WebP 画像と`meta.json`をそれぞれロードする必要があって、イマイチ扱いづらかったのです。そういう状況を受けて、Spark では[独自に ZIP 圧縮されたフォーマットを定義](https://github.com/sparkjsdev/spark/blob/5eb7dc5a5edb64697741fd1fec34e64c62e90caa/src/SplatLoader.ts#L222)していました。`.sog`ファイルも、実際は WebP 画像と JSON を ZIP 圧縮しただけなので実態は変わりませんが、拡張子がついたことでフォーマットっぽくなりましたね。

他の GS フォーマットから`.sog`を出力したい場合は、PlayCanvas が出している splat-transform という CLI を使えば変換できます。便利。

https://github.com/playcanvas/splat-transform

## Babylon.jsでSOG v2を扱う

前置きが長くなりましたが、Babylon.js での読み込み方法を解説します。といっても Gaussian Splatting 含めサポートしているフォーマットは基本的に全部同じやり方で読み込めてしまいます。

次のコードは 2 種類の読み込み方法を提示しています。Asset Container で読み込むか、Mesh として読み込むかという違いがありますね。いずれの方法でも中にある Mesh データを取り出すと`GaussianSplattingMesh`が入っていますので、それを良しなに扱えるようになります。

```ts: SOGを読み込むコード例
import {
  // ...
  ImportMeshAsync,
  LoadAssetContainerAsync,
  Vector3,
} from "@babylonjs/core";

// これが必要
import "@babylonjs/loaders/SPLAT";

// 例えばViteで静的アセットを参照するときの書き方（アセットへのパスが入ってます）
import sogPath from "../assets/pizza.sog?url";

// ...

// 1. Asset Containerとしてロードする方法
LoadAssetContainerAsync(sogPath, scene).then((container) => {
  container.addAllToScene();
});

// 2. Meshとして読み込んで即時Sceneに配置する方法
ImportMeshAsync(sogPath, scene).then(({ meshes }) => {
  // ...
});
```

実際のコードと動作が気になる方は、次のサンプルをご確認いただけます。

https://github.com/drumath2237/babylon-sogv2-support-sandbox


## おわりに

今回は Babylon.js で SOG v2 がサポートされたのを受けて、嬉しくなったので記事にしてみました。SOG v2 は個人的にかなり期待している良いフォーマットなので、あとはそれを利用するツールやサービス、ライブラリが増えてエコシステムが出来上がるといいですね。
最後まで読んでいただき、ありがとうございました。

### 参考文献

https://doc.babylonjs.com/features/featuresDeepDive/mesh/gaussianSplatting

https://github.com/playcanvas/splat-transform

https://fraunhoferhhi.github.io/Self-Organizing-Gaussians/

https://blog.playcanvas.com/playcanvas-open-sources-sog-format-for-gaussian-splatting/