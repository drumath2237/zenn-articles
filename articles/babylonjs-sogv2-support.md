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
なにぶん Gaussian Splatting は新しい技術ゆえに、画像における PNG や JPEG、3D モデルにおける FBX や glTF といった、スタンダードなファイルフォーマットは定まらないままになっていました。しかしここ 1，2 年くらいでは、Niantic Spatial inc.の主導する`.spz`を始めとして標準的なフォーマットを定義する動きが見られています。SPZ も最近は[面白い動き](https://github.com/KhronosGroup/glTF/pull/2531)がありますが、本記事の趣旨とずれるので割愛します。

https://scaniverse.com/news/spz-gaussian-splat-open-source-file-format

### SOGSとSOG v2

## Babylon.jsでSOG v2を扱う

### Babylon.jsのGSデータサポート状況

## おわりに

### 参考文献

https://doc.babylonjs.com/features/featuresDeepDive/mesh/gaussianSplatting

https://github.com/playcanvas/splat-transform

https://fraunhoferhhi.github.io/Self-Organizing-Gaussians/

https://blog.playcanvas.com/playcanvas-open-sources-sog-format-for-gaussian-splatting/