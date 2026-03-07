---
title: "OSSになった8thwall EngineのコードをDockerコンテナ上でビルドしてJSバイナリを得る"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["8thwall", "docker", "bazel", "webar", "webxr"]
published: false
---

:::message

本内容は2026年3月7日時点のものです。今後8thwallの方針やリポジトリの内容の更新によって変わる可能性がりますのでご注意ください。

:::

## はじめに

### TL;DR

- 8thwall から OSS 版のエンジンコードが公開された
- bazel や Python などの環境をととのえた Docker コンテナ上で OSS 版の JS バイナリをビルドできた
- ビルドするのに時間がかかるのとストレージを結構持っていかれるので注意

### 概要

サ終に伴い、2026 年 2 月いっぱいでプロジェクトの新規作成や編集ができなくなってしなった 8thwall ですが、Distributed Binary に引き続き、OSS 版のエンジンも公開されましたね。

https://www.8thwall.com/blog/post/208587408737/8th-wall-open-source

コードが公開されたということはコードを編集したり、ライセンスに注意が必要ですが 8thwall を使ったアプリ開発・公開がしやすくなりました。
しかし、もし 8thwall エンジンの内容を改修して使いたいとなったとき、エンジンをビルドできないと意味がないですね。
いろいろ試したところ、Docker 上で 8thwall の OSS エンジンコードを JS バイナリにビルドできたので、その方法をご紹介します。

### 想定読者

本記事の想定読者を次に示します。

- 8thwall の OSS 版エンジンを手元でビルドしたいエンジニア
- Docker や Linux コマンドへの基礎知識があるエンジニア

### 検証環境

今回の検証に使用した環境は次の通りです。

- Windows 11 Home（ホスト PC）
- Docker Desktop for Windows 4.63.0
- 検証した[8thwall/8thwall](https://github.com/8thwall/8thwall)の commit hash: [`ef2dffd1bf32e176f3f6152004ce58a7473e37d7`](https://github.com/8thwall/8thwall/tree/ef2dffd1bf32e176f3f6152004ce58a7473e37d7)

## 8thwallエンジンのコードがOSSになった

### これまでの経緯をざっくり

2025 年の 11 月末、8thwall から衝撃の発表がありました。いままで 8thwall はブラウザから使えるクラウドエディタや Studio といったツールを使って WebAR アプリを開発できるプラットフォームとして人気を誇っていました。しかし翌年の 2 月で新規プロジェクト作成および編集などができなくなるというものです。
改めて見ると、このニュースページに日本語版も併記されていますね（ローカライズされてるからじゃないですよね多分）。8thwall にとって日本のコミュニティの存在は大きかったのかもしれませんね。

https://www.8thwall.com/blog/post/200208966730/next-chapter

その翌月、どうブログにて「Distributed Engine Binary」と「Open Source Components」を準備中ということが明らかにされました。

https://www.8thwall.com/blog/post/202888018234/8th-wall-update-engine-distribution-and-open-source-plans

2026 年 1 月 22 日に、Buildable Code Export（クラウドプロジェクトのエクスポート）ができるようになったニュースと一緒に、Distributed Engine Binary の公開がアナウンスされました。
Distributed Engine Binary についてはデザイニウムさんの記事が詳しいのでぜひ見てみてください。

https://www.8thwall.com/blog/post/205481322650/buildable-code-export-now-available

https://note.com/thedesignium/n/n0c7ab8bfeca9

そして今回 2026 年 3 月 3 日、OSS 版のエンジンが発表されました。
もともと 8thwall は「8thwall.com」というドメインでプラットフォームが運用されてきましたが、OSS 版の公開に合わせて「[8thwall.org](https://8thwall.org/)」というドメインで別のサイトがホスティングされることになりました。

https://www.8thwall.com/blog/post/208587408737/8th-wall-open-source

### 発表されたOSS版エンジンコードについて

## Docker上でエンジンをビルドする

## おわりに

### 参考
