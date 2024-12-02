---
title: "Windowsユーザのためのpnpm Shell Emulator"
emoji: "🐚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pnpm", "powershell", "shell"]
published: true
---

:::message
本記事は[にー兄さんアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/ninisan-2024)の2日目の記事です。
:::

## はじめに

### TL;DR

Windows Powershell で pnpm のスクリプトを実行する時に Shell Emulator が有効だと
POSIX 環境のコマンドの書き方でも実行可能になって嬉しい。

### 概要

本記事では pnpm の Shell Emulator をご紹介します。
私は最近個人開発をしている時に知ったのですが、単体での日本語記事はあまりなかったのと、
Windows ユーザだからこそ享受できたメリットがあったので筆を執った次第です。

### 想定読者

- pnpm の機能に興味があるエンジニア
- JavaScript や TypeScript を使った開発で pnpm を利用しているエンジニア
- 普段 Windows を使っている Web 系エンジニア

など。

### 検証環境

- Windows 10/11 Home
- Powershell 7
- pnpm 9.13.2
- Node.js 20.9.0

## pnpm Shell Emulatorを使おう

### Shell Emulatorの概要

Shell Emulator については公式ドキュメントに記載があります。

https://pnpm.io/ja/cli/run#shell-emulator

> When true, pnpm will use a JavaScript implementation of a bash-like shell to execute scripts.

ドキュメントによると、このオプションが有効であるとき、pnpm は Bash ライクなシェルの JS 実装を使ってスクリプトを実行すると言います。
そして次のようなスクリプトは POSIX シェルではない環境でエラーになるとも書かれています。

```json
"scripts": {
  "test": "NODE_ENV=test node test.js"
}
```

私はこれまでずっと Node.js を使う時に Windows Powershell で実行をしていたのですが、この問題は確かに起きていましたし、cross-env を使ってずっと回避してきました。
しかし Shell Emulator を使うと解決するというのです。

### 使い方

使用方法は簡単で、`.npmrc`ファイルに次のように記載します。

```properties:.npmrc
shell-emulator=true
```

### 使いどころ

先に紹介したように、スクリプトの実行で環境変数を設定する時には使えますね。
そういえば、と思い試してみたところ、`echo hello && echo world`のような`&&`でコマンドを連ねる記法に関しては、Shell Emulator が無効であっても正常に動作しました。前にやったときは動かなかったような記憶がありましたが、勘違いだったようです。

個人的には、pnpm スクリプトから Docker コンテナで処理を実行したいときに、ボリュームをマウントする記法を揃えられるところに魅力を感じました。
残念ながら Powershell で`pwd`コマンドを実行した時に、UNIX と同じような挙動にならないため、WSL を使うか、もしくはワークアラウンドが必要でした。

https://qiita.com/FrozenVoice/items/da89679c13ad97f3bca7

しかし Shell Emulator を使うと、ワークアラウンドなしでもボリュームマウントができました。直近で取り組んでいた OSS 開発では、WebAssembly のコンパイルを Docker 上で行っていたので、とても便利に感じましたね。

## おわりに

今回は pnpm の Shell Emulator をご紹介しました。
設定方法は簡単だけど、とても便利なものだったのでこれからも使っていきたいです。

### 参考文献

https://pnpm.io/cli/run#shell-emulator

https://www.petermekhaeil.com/til/pnpm-shell-emulator/
