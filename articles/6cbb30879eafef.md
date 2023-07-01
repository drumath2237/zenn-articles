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

## 新デバイスの対応

## Chromium版も開発中

<!-- WebARはこちらに乗るとのことなので取り上げる -->
<!-- https://github.com/Igalia/wolvic/issues/242 -->

# Wolvicを使ったデバッグを試してみる

## A3版でのデバッグ

## adbでの（ワイヤレス）接続

## Firefox Nightlyを使ったデバッグ

## Babylon.jsのWebVRプロジェクトをデバッグしてみる

# おわりに

## まとめと感想

<!-- VisionOSのWebXR対応について少し触れる -->

## 参考