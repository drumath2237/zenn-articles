---
title: "OSSになった8thwall Engineのコードについて、およびDockerコンテナ上でビルドしてJSバイナリを得る方法"
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
- ビルドするのに時間がかかるのとストレージ容量を結構持っていかれるので注意

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

それでは今月発表された OSS 版エンジンコードについてみていきましょう。
コードは[8thwall/8thwall](https://github.com/8thwall/8thwall)という GitHub リポジトリで公開されています。
このリポジトリには Engine コードのほか、xrextras というヘルパーや Studio で使われていた ECS、AR 画像マーカーを作るための CLI である image-target-cli も同梱されています。
Engine についての README は[`/packages/engine`](https://github.com/8thwall/8thwall/tree/main/packages/engine)にあります。

http://github.com/8thwall/8thwall/

OSS 版のエンジンコードは、ビルドできたとしても Distributed Engine Binary と同等のものが出てくるわけではありません。ブログにも書いてある通りなのですが、SLAM の機能が含まれていないんですね。次の説明は[OSSの発表があったブログ](https://www.8thwall.com/blog/post/208587408737/8th-wall-open-source)から引用したものです。

> SLAM has not been open sourced and will only be available in the Distributed Engine Binary. But with the rest of the framework now open, the engine isn't frozen in place. As browser APIs change and web standards evolve, the community can maintain and adapt it without depending on us. 

DeepL による日本語訳は次の通りです。

> SLAMはオープンソース化されておらず、分散エンジンバイナリでのみ利用可能です。しかし、フレームワークの他の部分がオープン化されたことで、エンジンは固定化されていません。ブラウザAPIが変更され、ウェブ標準が進化するにつれ、コミュニティは当社に依存することなく、SLAMの維持と適応を行えます。 

ですので、もし World-Effect などの SLAM を要する機能を使いたい場合は Distributed Engine Binary を使うしかなさそうですね。

個人的には、やはり AR エンジンとしての 8thwall における魅力の 1 つは SLAM だと思っており、理想的にはオープンになることを期待していましたが厳しそうですね。しかしエンジンがオープンになったということだけでもかなり素晴らしい取り組みです。これには様々な難しい制約や社内での合意形成が必要だったと推測しますが、これが成しえたのはひとえに 8thwall チームの執念とコミュニティに対する強い思いがあったからだと考えます。ひとりの OSS エンジニアとして、このような取り組みをしてくれた 8thwall チームへ感謝と、そしてお疲れさまでしたの気持ちを伝えたいです。


## Docker上でエンジンをビルドする

## おわりに

### 参考
