---
title: "Immersal REST APIとサーバーサイド位置合わせの考え方"
emoji: "🌏"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["immersal", "api", "ar"]
published: false
---
# はじめに

## 概要

この記事は[Iwaken Lab.アドベントカレンダー2021]()の〇日目の記事です。
私は Immersal を使った個人開発をすることがあるのですが、
その時に使っているサーバーサイド位置合わせについて記事を書こうと考えました。

サーバーサイド位置合わせはその名の通りサーバーで位置合わせをしてくれるため、
自分で CV 系のロジックを扱うことは一切なく、
http リクエストを送れさえすれば簡単に実行できます。

逆に言うと、「そもそも位置合わせって何か」とか
「どんなデータが返ってくるのか」とか「どうやって処理するのか」みたいな
具体的な実装よりも考え方が大事になってくると考えました。

## 扱う内容・扱わない内容

本記事で扱う内容は次の通りです。

- VPS の基礎
- 位置合わせという概念について
- Immersal REST API における位置合わせの考え方

逆に本記事で扱わない内容は次の通りです。

- Immersal SDK の使用方法
- C#など具体的なプログラムを用いた実装

## 対象読者

本記事で対象としているのは、すでに Immersal を知っていて
サンプルプロジェクトをみたり試したことがある人向けです。
その中で、Immersal や VPS のことをもう少し深く知って脱初心者を目指す方に向けて書くつもりです。

また記事の中で行列を扱うため、
線形代数の基本知識が必要です。

# VPSとしてのImmersal


## VPSとは

VPS とは Visual Positioning System の略です。
Virtual Private Server ではないですよ、というのはお約束の注意ですね。

VPS は Positioning System という名前の通り、
位置情報を取得するシステムです。
GPS は人工衛星によって地球上の緯度経度を割り出すことができる位置情報システムですが
VPS の場合は画像を使って位置情報を割り出すことができます。

AR アプリケーションではアプリケーション内のワールド座標系において
デジタルオブジェクトを配置したり、プレイヤーが移動したできますが
そのプレイヤーやオブジェクトが現実空間のどこにいるのかは分かりません。
そのため、デジタルオブジェクトの場所を記録しておいたとしても
アプリケーションを起動するたびにオブジェクトが出現する現実での場所が変わってしまいます。

例えば現実の机にプレイヤーがバーチャルな花瓶を置いたとします。
インテリアとして配置した場合、常にその位置に花瓶が出現するべきですが
その花瓶はアプリケーションを起動する位置（つまりワールド座標系の原点）が変わってしまうと
机の上には表示されません。

<!-- ここに花瓶の例を示す画像 -->

これを解決するために画像を印刷して作った AR マーカーが使えます。
しかし現実空間で好き勝手に AR マーカーを置ける場所は、
私有地など限られていますし、広い空間で AR をやるのには不向きです。

このような「マーカー不要で AR のワールド原点を現実に固定したい」という課題に対する
1 つの解が VPS による位置合わせです。

VPS では通常、事前に何らかの方法で現実空間をスキャンして簡単な 3D マップを作成し、
AR アプリケーションで取得したカメラ画像などから、
3D マップの中のどこにいるのかを推定します。
3Dマップは現実空間と紐づいているため、現実空間での場所を特定できるというわけです。

<!-- ここにImmersalの点群マップ -->

## Immersalとは

# Immersal REST APIによるサーバーサイド位置合わせ

## Immersal REST APIとは

## サーバーサイド位置合わせの考えかた

## Immersalによるサーバーサイド位置合わせの方法

# おわりに