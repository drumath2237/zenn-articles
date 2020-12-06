---
title: "【Unity】VFX GraphをTimelineで制御する"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

この記事は[愚者っとCorp.アドベントカレンダー2020](https://adventar.org/calendars/5126)の 5 日目の記事です。

# はじめに

どうも、[にー兄さん](https://twitter.com/ninisan_drumath)です。
Zenn 初記事となります、よろしくお願いいたします。

自分は Unity で VFX Graph を使うことが好きなのですが、表現力の高い VFX を作るのに Timeline が使えそうなので、せっかくなので記事にしました。小ネタですが最後までお付き合いいただけると嬉しいです。

## TL;DR

- VFX Graph ではデフォルトで Timeline 制御できるしくみがある
- Timeline 拡張によってより高度な制御も可能

## この記事で得られる知見

以下のような知見が得られます。

- VFX Graph に簡単な Timeline 制御を加える方法
- 高度な Timeline 制御をする方法案

## 対象読者

対象読者は以下の通りです。

- VFX Graph を触ったことがある or 興味がある
- Timeline を触ったことがある or 興味がある
- Unity を触ったことがある(SRP を使ったことがあればベスト)

## 検証環境

検証環境は以下の通りです。

||環境|
|:---:|:---:|
|OS|Windows 10 Home|
|Unity|2019.4.12f|
|URP|7.3.1|
|VFX Graph|7.3.1|

# VFX GraphをTimelineで制御する方法

VFX Graph を Timeline で制御する方法はパッと思いつくもので３つあります。

## Visual Effect Activation Trackを使う

VFX Graph をプロジェクトにインポートすると、Visual Effect Activation Track というアニメーショントラックが Timeline で使用できるようになります。
これを使うと、特定の VFX Graph に対してアニメーションクリップの始まりと終わりでイベントを発火させることができるようになります。
特に設定しなければ、クリップの始まりで`OnPlay`が、クリップの最後に`OnStop`が発火します。

また、イベントの発生とともにいくつかのプロパティも一緒に渡すことができるので、クリップごとに違う動作をさせることも可能になっています。

まず、以下のような簡単な VFX Graph を作成してみます。
unlit なキューブが上に飛んでいくという簡単なものです。

![simple vfx](https://storage.googleapis.com/zenn-user-upload/ou61tq4e2rxeevunsxj1trn2g7vs)

ほとんどデフォルトのものと変わりませんが、特殊な点としては Initialize Particle コンテキストに Inherit Source Color ブロックがあることです。つまり、何かのソースから Color Attribute を継承していることを意味しますね。

次に Timeline 側の準備をします。VFX Graph がインストールされていれば、以下のように新規トラック作成のメニューに「Visual Effect Activation Track」というものが追加されていますので、こちらをクリックします。

![vfx activation track](https://storage.googleapis.com/zenn-user-upload/596wze1h9olkj4n0n6fnby71gpns)

クリップを作成する前に、シーン上の VFX Graph を指定するのを忘れないようにしましょう。

![vfx set](https://storage.googleapis.com/zenn-user-upload/faf26lppgbpoc9u3vgluflxpgp3i)

作成した Visual Effect Activation Track で右クリックをすると以下のようなメニューが出てくるので、「Add Visual Effect Activation Clip」を押すことで、クリップが作成されます。

![add clip](https://storage.googleapis.com/zenn-user-upload/84vdkjcomamfu8xz0wzlhw6hzj36)

こんな感じに適当にクリップを並べて、それぞれのクリップをインスペクタで編集していきます。

![put clips](https://storage.googleapis.com/zenn-user-upload/byfjrwv7enndst4ggptcphvs91wd)

インスペクタではクリップの開始と終了で発火するイベント名と、Event Attribute を指定できるので、こんな感じに設定してやります。

![set inspector](https://storage.googleapis.com/zenn-user-upload/v6e5rbfylxu00xnx8ymbqatdr4rq)

他のクリップも同様に編集していき、これを再生してみると以下のような動作をします。

@[youtube](Nw1je_OruwU)

## Animation Trackを使う

VFX Graph に限らず Animator コンポーネントがアタッチされている GameObject は、インスペクタから設定できる値を Timeline で Recording できます。
VFX Graph も同様で、インスペクタから設定できるプロパティは AnimationTrack からキーフレームを指定して録画できます。

## Timeline拡張を作る

前述した２つの方法で物足りないと感じた人は、Timeline 拡張を作ることで大体は解決できる可能性が高いです。
Timeline 拡張とは、Playable 系のインタフェースを継承したクラスを実装することですることにより、Timeline で使える Track や Clip を自作できる Unity の仕組みになります。これを使えばスクリプトで制御できる VFXGraph の API を Timeline でも設定できるようになります。
Timeline 拡張の作り方を説明してしまうと、それこそ記事が１本書けてしまうので他の記事を参照されたいです。

# まとめ・おわりに

## 参考

- [TimelineでVFX Graphを扱うことに言及している公式記事](https://blogs.unity3d.com/jp/2018/11/27/creating-explosive-visuals-with-the-visual-effect-graph/)