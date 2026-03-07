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

## Docker上でエンジンをビルドする

## おわりに

### 参考
