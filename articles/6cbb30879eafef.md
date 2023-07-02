---
title: "XR向けWebブラウザWolvic 1.4が個人的にアツい"
emoji: "🐺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["webxr", "browser", "vr", "wolvic"]
published: false
---

# はじめに

## TL;DR

## 概要と対象読者

本記事では筆者が個人的に最近お熱な Wolvic ブラウザについて紹介します。
とはいえ Wolvic 自体もまだ開発中のプロダクトなのと、今回取り上げる ThinkReality A3 むけのバイナリがまだ公開されていないなどの状況もありますので現時点でわかる範囲にて共有になっています。

本記事は次のような読者像を想定して執筆しましたが、必ずしも当てはまっていなくても読める内容なはずです。

- xR デバイスやプラットフォームに興味がある
- Wolvic や Firefox Reality を聞いたことがある・知っている
- AR グラスや HMD で WebXR を体験・開発したい
- Web 開発の経験がある

## 検証環境

執筆にあたり動作確認を行った環境を次に示します。

- Windows 10 / 11 Home
- Firefox Nightly 116.0a1
- ThinkReality A3
- Snapdragon Spaces Services 0.1.4
- Wolvic (for A3) 1.4

# Wolvicブラウザについて

タイトルにもあるとおり Wolvic は XR デバイス向けのブラウザです。

## Firefox Realityの後継ブラウザ

Wolvic という名前を聞いたことなくても、もしかすると Firefox Reality という名前にはピンとくる方もいるのではないでしょうか。Firefox Reality は旧 Oculus Quest2 や HoloLens などのデバイスで使えた xR デバイス向けブラウザでした。
過去形なところから察せるように、この Firefox Reality というブラウザは既にサポート終了しています。[MoguraVRさんの記事](https://www.moguravr.com/wolvic/)によると、Mozilla のレイオフで Firefox Reality のチームにいた人が対象となったためらしいですね。
サポートは終了しましたが、Firefox Reality は Igalia という団体に Wolvic という名前で引き継がれました。

## Lynx R1に搭載されるらしい

自分が Wolvic の情報を追い始めたのは[ikkouさんの記事](https://zenn.dev/ikkou/articles/1fd48b7ad976d3)で Lynx R1 に搭載されることを知ってからでした。
もともと immersive-web の[WebXRサポートテーブル](https://immersiveweb.dev/#supporttable)にはあったので気になっていたのですが、筆者が注目していたデバイスに標準搭載されるとなったので GitHub のリリースを Watch し始めました。

## 対応プラットフォーム

2023/07/02 現在で Wolvic の Lates 晩はｖ1.4.0 です。
次に挙げるデバイス向けの対応がされていると発表されていますが、Lynx などの一部のデバイス向けのバイナリはまだ配布されていません。

- Meta Quest2/Pro
- Huawei VR Glasses
- VIVE Focus
- Pico4 Pico Neo3
- Lynx R1
- ThinkReality A3
- Think Reality VRX

# Wolvic 1.4のリリース

## 概要（和訳）

Wolvic 1.4 が 2023/06/17 にリリースされました。

https://github.com/Igalia/wolvic/releases/tag/v1.4

次の文章はリリースページにあった概要欄とその和訳（DeepL 使用）です。

<!-- textlint-disable -->

> This is the last major release of the 1.x series. Some 1.4.x might follow with specific critical bug fixes, but our main focus will switch to 2.0 after this. It's being a while since the previous 1.3.4 release, but we're adding some new cool features, like 3D hand models for hand tracking or passthrough support that took some time to get it right in a variety of devices. Speaking of which, we're adding more devices to the pack, Lenovo VRX & Lenovo A3 (based of Qualcomm's SnapdragonSpaces) and the Lynx-R1.

（和訳）

> これは1.xシリーズの最後のメジャーリリースです。いくつかの1.4.xは特定の重要なバグの修正で続くかもしれませんが、私たちの主な焦点はこの後2.0に切り替わります。前回の1.3.4リリースからしばらく経ちましたが、ハンドトラッキングのための3Dハンドモデルや、様々なデバイスで正しく動作させるのに時間がかかったパススルーのサポートなど、クールな新機能を追加しています。そういえば、Lenovo VRX & Lenovo A3 (QualcommのSnapdragonSpacesベース)とLynx-R1が追加されました。

<!-- textlint-enable -->

今回のリリース内容を見て、個人的には次の２つの項目にワクワクしました。

- 新しいデバイスの対応
- Chromium 版の WebXR 開発進捗

## 新デバイスの対応

1.4 のリリースで次の 3 つのデバイスへ新たに対応しました。

- Lenovo VRX
- Lenovo A3
- Lynx R1

最近筆者は Lenovo ThinkReality A3 を触っていたり、Lynx R1 のバッカーであったりすることから今回のリリースはとても魅力的に感じました。
前述のとおり Lynx の標準ブラウザとして搭載されることは発表されていたもののその真偽についてはよくわかっていなかったので、今回のリリースで現実味を帯びてきたんだなと実感しましたね。そして対応が発表されたことにより、そろそろ Lynx が手元に来るのではないかという期待にもつながりました。

余談なのですが、実は Lynx 対応版と A3 対応版に関してはリリースされてからしばらく公開されていました（18 時間くらいでしょうか）。
筆者は公開されている間に A3 版の APK をダウンロードしていたので手元にあるのですが、2023/07/02 時点ではまだ公開されていませんね。
公開が取り下げられてしまった理由を issue で聞いてみたところ次のような理由があるみたいでした。

- Legal Checks をしているため
- ハンドトラッキングの調整が終わったら公開したいため

https://github.com/Igalia/wolvic/issues/745

いったん正式公開を待ちましょう。楽しみですね。

## Chromium版も開発中

Wolvic は、おそらくですが Gecko エンジンをベースに開発されていました。Firefox Reality が元になっているのでその流れを組んでいるのではないかと考えます。
しかし並行して Chromium ベース版も開発されているようで、Igalia のリポジトリの中には[`wolvic-chromium`という Chromium の Fork](https://github.com/Igalia/wolvic-chromium) が存在しています。
またその Fork では Support WebXR for Wolvic という名前の Pull Request もマージされているようです。

https://github.com/Igalia/wolvic-chromium/pull/10

textlint-disable

筆者が Wolvic の Chromium ベース版に期待を寄せているのは、WebXR Augmented Reality Module、つまり WebAR 対応をされる旨の言及が issue の中でされていたためです。

https://github.com/Igalia/wolvic/issues/242

こちらはメンバーの svillar さんのコメントです。

> I'll reopen this because we are not likely going to have this support using Gecko, but it'll be available with the Chromium backend we are planning for next year

Chromium ベース版の開発は進んでおり WebXR 対応までできているとのことですが、まだリリースページに APK などは見当たりませんね。

# A3版Wolvicを使ってWebページをデバッグしてみる

## A3版でのデバッグ

前述のとおり筆者は現在（2023/07/02 時点）公開されていない ThinkReality A3 対応 APK を持っていますので、手元の A3 で試していました。

ThinkRerality A3（以降 A3 と呼ぶ）のことをご存じでない方も多いはずなので簡単に解説すると、Lenovo が出している AR グラスデバイスです。Motorola というスマートフォンと接続して使うタイプのグラスで、前面についているカメラによるトラッキングで 6DoF で動作します。
Qualcomm が開発する Snapdragon Spaces という SDK に対応したデバイスであることが特徴です。

https://spaces.qualcomm.com/devices/lenovo-thinkreality-a3/

## adbでの（ワイヤレス）接続

Motorola スマホは USB Type-C ポートが 1 つ付いていますが A3 のケーブルと接続してしまうため A3 を使用している最中に adb を接続するためにはワイヤレス接続が必要です。
adb によるワイヤレスデバッグの方法は次のドキュメントを参考にしてください。

https://developer.android.com/studio/command-line/adb?hl=ja#wireless-android11-command-line

## Firefox Nightlyでデバッグする

adb を使って PC と Motorola をリモート接続できていれば、PC の FirefoxNightly を使って A3 で動作している Wolvic ブラウザをインスぺクトできます。
詳細な手順は Wolvic の GitHub Wiki に記載されています。

https://github.com/Igalia/wolvic/wiki/Debugging

まずは Wolvic の設定画面から開発者用の設定を開きリモートデバッグとログを有効化します。

![img](/images/wolvic1140/dev-op.png)
_開発者無オプションで設定を有効化_

その次に PC から Firefox Nightly を立ち上げ`about:debugging`という URL を入力して A3 側で起動してる Wolvic を接続します。
そして Wolvic で立ち上げているタブを選択してインスぺクトすると、いつも通りブラウザで開発者ツールを開いたときのような操作ができます。

![img](/images/wolvic1140/firefox-dev.png)
_about:debuggingを開いたところ_

![img](/images/wolvic1140/devtool.png)
_FirefoxからWolvicで開いたページをリモートデバッグ_

自分がダウンロードした A3 版の APK だと、なぜかコンソールログが開発者ツールで表示されないようでしたが、デバッガからブレークポイントを設定して変数の値をインスぺクトできました。

## Babylon.jsのWebVRプロジェクトをデバッグしてみる

最後に簡単にではありますが、自前でホストした WebXR（VR）アプリを Wolvic でデバッグしてみましょう。
次のコマンドを実行してカレントフォルダに Babylon.js のプロジェクトをセットアップします。

```bash:terminal
# babylonプロジェクトの作成
# （筆者作のBabylon.js用スキャッフォールディングツールです）
npm create babylon-app

# プロジェクトのディレクトリへ移動
cd <project name>

# 依存パッケージの解決
npm i

# https対応のために必要
npm i -D vite-plugin-mkcert
```

そして`src/index.ts`と`vite.config.ts`を編集します。

::: details index.tsとvite.config.tsのコード

```ts:src/index.ts
import { Playground } from "./createScene";
import "./style.css";
import { Engine } from "@babylonjs/core";

const main = async () => {
  const renderCanvas = document.getElementById(
    "renderCanvas"
  ) as HTMLCanvasElement;
  if (!renderCanvas) {
    return;
  }

  const engine = new Engine(renderCanvas, true);

  const scene = Playground.CreateScene(engine, renderCanvas);

  const { baseExperience } = await scene.createDefaultXRExperienceAsync({
    optionalFeatures: true,
    uiOptions: {
      sessionMode: "immersive-vr",
    },
  });

  const featureManager = baseExperience.featuresManager;

  window.addEventListener("resize", () => {
    engine.resize();
  });

  engine.runRenderLoop(() => {
    scene.render();
  });
};

main();
```

```ts:vite.config.ts
import { defineConfig } from "vite";
import mkcert from "vite-plugin-mkcert";

export default defineConfig(() => {
  return {
    server: {
      https: true,
    },
    plugins: [mkcert()],
  };
});
```

:::


そして最後に次のコマンドを実行してローカルに dev サーバをホストします。

```bash
npm run dev -- --host
```

Wolvic から dev サーバの URL に接続しますが、A3 から文字を入力するのが非常に困難なので Firefox Reality から入力するのがおすすめです。

ソースマップが設定してあるので開発者ツールで普通に TypeScrpt ファイルからブレークポイントを張れます。デフォルトで有効になっている WebXR Device API のモジュールを調べてみると、コントローラ系のモジュールは扱えるみたいですね。

![img](/images/wolvic1140/baby.png)

# おわりに

## まとめと感想

本記事では、Wolvicの概要と1.4

## 参考