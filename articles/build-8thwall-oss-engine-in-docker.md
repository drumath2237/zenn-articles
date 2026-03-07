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

本記事の想定読者を次に示します（OR 演算です）。

- OSS 版 8thwall のコードが公開された経緯やその内容が気になる人
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

リポジトリを見てみると、大部分のコードは C++などで書かれていることがわかります。
全見られたわけではないので正確ではないかもですが、推測するに 8thwall ではコアなロジックは C++によって記述されており、それが Emscripten によって WebAssembly にビルドされて JS バイナリに含まれているのだと考えます。[`/c8`](https://github.com/8thwall/8thwall/tree/main/c8)というディレクトリを見ると機能ごとに C++のコードが入っているのがわかりますね。

OSS 版のエンジンコードは、ビルドできたとしても Distributed Engine Binary と同等のものが出てくるわけではありません。ブログにも書いてある通りなのですが、SLAM の機能が含まれていないんですね。次の説明は[OSSの発表があったブログ](https://www.8thwall.com/blog/post/208587408737/8th-wall-open-source)から引用したものです。

> SLAM has not been open sourced and will only be available in the Distributed Engine Binary. But with the rest of the framework now open, the engine isn't frozen in place. As browser APIs change and web standards evolve, the community can maintain and adapt it without depending on us. 

DeepL による日本語訳は次の通りです。

> SLAMはオープンソース化されておらず、分散エンジンバイナリでのみ利用可能です。しかし、フレームワークの他の部分がオープン化されたことで、エンジンは固定化されていません。ブラウザAPIが変更され、ウェブ標準が進化するにつれ、コミュニティは当社に依存することなく、SLAMの維持と適応を行えます。 

ですので、もし World-Effect などの SLAM を要する機能を使いたい場合は Distributed Engine Binary を使うしかなさそうですね。

個人的には、やはり AR エンジンとしての 8thwall における魅力の 1 つは SLAM だと思っており、理想的にはオープンになることを期待していましたが厳しそうですね。しかしエンジンがオープンになったということだけでもかなり素晴らしい取り組みです。これには様々な難しい制約や社内での合意形成が必要だったと推測しますが、これが成しえたのはひとえに 8thwall チームの執念とコミュニティに対する強い思いがあったからだと考えます。ひとりの OSS エンジニアとして、このような取り組みをしてくれた 8thwall チームへ感謝と、そしてお疲れさまでしたの気持ちを伝えたいです。

## Docker上でエンジンをビルドする

それではエンジンコードをビルドしていきましょう。

### 全体の流れ

色々試してみたところ、Ubuntu ベースで必要な依存関係をインストールした Docker イメージを作ってしまったほうが良さそうでしたので、そうしています。
自分はいつも Windows の PC を使っていますが、README に書かれているビルドコマンドを実行するとワイルドカードの違いの影響でエラーが出てしまいました。
なお、このリポジトリではビルドツールに bazel を使っていますが、当方 bazel には全く詳しくなく......。設定ファイルもちゃんと理解できていないので Dockerfile には過不足あるかもしれません。もし気になる部分があればコメントいただけますと嬉しいです。

全体の流れとしては次のようになります。

1. Dockerfile を書いてイメージをビルドする
2. インタラクティブモードでコンテナを起動し、ビルドコマンドを実行する
3. 出ロクされたバイナリをマウントした Windows ディレクトリにコピーして取り出す

試すのに気軽なためコンテナをインタラクティブモードで起動していますが、手順が固まっているのであればシェルスクリプトを書いて自動化するのが良いと考えます。そこら辺のアレンジはご自由にお願いします。

### ビルドする

まずはローカルにリポジトリをクローンしまして、ローカルリポジトリのルートディレクトリで作業することを基本とします。
リポジトリルートに次のような Dockerfile を配置しましょう。

```dockerfile:/Dockerfile
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        build-essential \
        python3 \
        python3-dev \
        python3-numpy \
        unzip \
        zip \
        lsb-release \
        ca-certificates \
        bash && \
    ln -sf /bin/bash /bin/sh

# Bazel 7.2.1 を明示インストール
RUN curl -L -o /usr/local/bin/bazel \
    https://github.com/bazelbuild/bazel/releases/download/7.2.1/bazel-7.2.1-linux-x86_64 && \
    chmod +x /usr/local/bin/bazel

RUN useradd -ms /bin/bash builder
USER builder

WORKDIR /home/builder/workspace
```

このリポジトリでは bazel を使ってビルドしていますので、Bazel 7.2.1 を使っています。
最新の Bazel 9 を使うとビルドが失敗していそうな雰囲気がありましたのでバージョンを合わせておくのがおススメです。Bazelisk を使ったほうが良さそうですが、いったんバージョンを直指定しました。

またコンテナを実行するときはルート権限でログインしてしまうと、途中の Python 環境を導入する際に失敗します。どうやら Bazel の Python パッケージ（というのが正しいのかわかりませんが）をインストールするときに非ルートユーザであることが求められるみたいです。

イメージをビルドし、コンテナをインタラクティブモードで実行します。
このイメージでは`/home/builder/`ディレクトリがユーザディレクトリになっており、8thwall のプロジェクト全体を`~/workspace`にマウントしています。

```sh
docker build --no-cache -t 8thwall-bazel .
docker run -it -v ${PWD}:/home/builder/workspace 8thwall-bazel
```

あとはビルドコマンドを実行しますが、もしホスト OS に Windows を使っていて Git が CRLF を前提に動作している場合、テキストファイルを CRLF->LF 変換しておくことをおすすめします（私はここでハマりました）。
Ubuntu のシェル上で次を実行します。
変換対象となるファイル拡張子を列挙しましたが全部は必要なさそうです。一応関係しそうなものは列挙しています（もしかしたら.sh と BUILD だけでいい可能性があります）。

```sh
git config --global --add safe.directory /home/builder/workspace

# CRLF->LF
git ls-files -z \
  '*.sh' '*.js' '*.ts' '*.json' '*.cc' '*.h' '*.bzl' '*.py' '*.c' '*/BUILD' \
| xargs -0 sed -i 's/\r$//'
```

次のビルドコマンドを実行すると、bazel によるビルドが実行されます。
これは SIMD 演算が使われている WebAssembly をビルドするコマンドらしく、実行環境によっては SIMD をサポートしていないかもしれません。そのときは`--config=wasmreleasesimd`オプションを`--config=wasmrelease`にしてください。

```sh
bazel build \
  --config=wasmreleasesimd \
  --repo_env=PYTHON_BIN_PATH=/usr/bin/python3 \
  --repo_env=PYTHON_LIB_PATH=/usr/lib/python3/dist-packages \
  //reality/app/xr/js:bundle
```

ビルドプロセスが実行されていきますが結構時間がかかります。
お使いのネットワークの速度などにもよりますが弊環境では 20 分以上かかりました。
途中で Haggingface から TF Lite 用のモデルデータ（なのかな）をバンバンフェッチしていたりしていて、結構通信帯域を食うんですよね。そしてこれらはローカルファイルに展開されるので、ストレージ容量も持っていかれます（docker のディスクファイルが膨れます）。実行する際には余裕をもって 40GB くらいは空けておくことをおすすめします。

さて、ビルドができたら次のディレクトリに成果物が格納されています。

```
~/.cache/bazel/_bazel_builder/4c3ace931b70317ca8e3417467a6f042/execroot/_main/bazel-out/wasm32-opt-ST-6b2337be98f2/bin/reality/app/xr/js
```

`ll`コマンドで見るとこんな感じでした。

![img](/images/oss-8thwall-docker/ll.png)
*ビルドされたJSバイナリ*

また、次のパスには ZIP 化されたバイナリも含まれていました。

```
~/.cache/bazel/_bazel_builder/4c3ace931b70317ca8e3417467a6f042/execroot/_main/bazel-out/wasm32-opt/bin/reality/app/xr/js
```

最後に成果物の入ったディレクトリからファイル群をコピーして終了です。

```sh
# コピー先のファイル名はよしなに
cp -r . ~/workspace/dist
```

これでホスト PC のファイルシステムにもバイナリがコピーされましたので、コンテナを終了しましょう。

![img](/images/oss-8thwall-docker/builds.png)
*コピーされたJSバイナリ群*

## おわりに

### 参考
