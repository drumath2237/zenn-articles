---
title: "Immersal REST APIを使ったサーバーサイド位置合わせの考え方"
emoji: "🌏"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["immersal", "api", "ar"]
published: true
publication_name: "iwakenlab_book"
---

# はじめに

## TL;DR

- Immersal REST API を使えば Immersal SDK を使うことなく位置合わせの実行が可能
- カメラ情報の取得、http リクエスト、座標変換という工程が必要

## 概要

この記事は[Iwaken Lab.アドベントカレンダー 2021](https://qiita.com/advent-calendar/2021/iwakenlab)の 16 日目の記事です。
私は Immersal を使った個人開発をすることがあるのですが、
その時に使っているサーバーサイド位置合わせについて記事を書こうと考えました。

サーバーサイド位置合わせはその名の通りサーバーで位置合わせをしてくれるため、
自分で CV 系のロジックを扱うことは一切なく、
http リクエストを送れさえすれば簡単に実行できます。

ただ、画像処理や SLAM の知識は必要なかったとしても
そもそも位置合わせとはどんなものなのか、
そしてどのように実行されるのかという知識がなければ実装に組み込むことはできません。
本記事では Immersal を例としてそれらの理解をしてみよう、という内容にしたいと考えています。

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

また行列を扱うために線形代数の基礎、
そしてネットワーク通信に関する基礎知識が必要です。

# VPS としての Immersal

## VPS とは

VPS とは Visual Positioning System の略です。
Virtual Private Server ではないですよ、というのはお約束の注意ですね。

VPS は Positioning System という名前の通り、
位置情報を取得するシステムです。
GPS は人工衛星によって地球上の緯度経度を割り出すことができる位置情報システムですが
VPS の場合は画像を使って位置情報を割り出すことができます。

AR アプリではアプリ内のワールド座標系において
デジタルオブジェクトを配置したり、プレイヤーが移動したりできますが
そのプレイヤーやオブジェクトが現実空間のどこにいるのかは分かりません。
そのため、デジタルオブジェクトの場所を記録しておいたとしても
アプリケーションを起動するたびにオブジェクトが出現する現実での場所が変わってしまいます。

解決策の 1 つとして画像マーカーが使えます。
しかし現実空間で好き勝手に AR マーカーを置ける場所は私有地など限られていますし、
広い空間で AR をやるのには不向きです。

このような「マーカー不要で AR のワールド原点を現実に固定したい」という課題に対する
1 つの解が VPS による位置合わせです。

VPS では通常、事前に何らかの方法で現実空間をスキャンして簡単な 3D マップを作成し、
AR アプリケーションで取得したカメラ画像などからマップ中のどこにいるのかを推定します。
3D マップは現実空間と紐づいているため、現実空間での場所を特定できるというわけです。

## Immersal とは

Immersal とは、AR クラウドサービスです。
AR クラウドとは AR アプリケーションで VPS による位置合わせを行い、
現実空間に結び付いた AR コンテンツを実行したり
複数人で AR アプリをマルチプレイで体験したりするための技術を指します。

Immersal でも独自の VPS が使われており、
Immersal にユーザ登録をすることで VPS を使用した AR クラウドアプリケーションを開発できます。

Immersal では、公式の Mapper アプリを用いて
Android 端末や iOS 端末から現実空間のスキャンをし、
Immersal SDK を Unity で使うことによってマップとの位置合わせができます。

その他にも Immersal はサーバーサイド位置合わせを提供しており、
REST API によって VPS に必要な機能を一通り実行でいます。

https://www.youtube.com/watch?v=h3XCgbu7PcM

# Immersal REST API によるサーバーサイド位置合わせ

## Immersal REST API とは

前述のように、Immersal は VPS の機能を REST API として提供しています。
これ以降はこの API を Immersal REST API といい、
特に位置合わせについて説明をしていきます。

Immersal REST API の仕様は[公式ドキュメント](https://immersal.gitbook.io/sdk/api-documentation/rest-api)に記載があるので合わせてご確認ください。

## Immersal サーバーサイド位置合わせの手順

Immersal の SDK や REST API を参考にしながら、
位置合わせをするまでの工程を説明していきます。
ざっくり手順を示すと次のようになり増す。

1. AR シーンのカメラ画像およびカメラパラメータを取得する
2. 画像からマップにおけるカメラの姿勢を推定する
3. AR オブジェクトの座標変換する

## カメラ情報の取得

まず位置合わせをするために必要な
「カメラ画像」と「カメラの内部パラメータ」を取得します。

カメラ画像はご存知の通りカメラで撮影した画像のことです。
たとえば Unity ARFoundation では、[`TryAquireLatestCpuImage`という API](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@4.0/manual/cpu-camera-image.html) で取得できます。
詳細は後述しますが、Immersal REST API では最終的に png 形式の画像データを base64 エンコードした文字列を必要とします。

また、Immersal ではカメラの内部パラメータのうち、
「焦点距離」と「光学中心」が必要です。
カメラが透視投影モデルの場合、カメラの姿勢を原点とするカメラ座標系の物体は
奥行きを以って投影されます。
これは視野角や焦点距離といった要素から構成される
視錘台のモデルがそう定義されるからです。

![img](https://docs.unity3d.com/ja/2019.4/uploads/Main/ViewFrustum.png)
_引用：[視錘台を理解する](https://docs.unity3d.com/ja/2019.4/Manual/UnderstandingFrustum.html)_

物理的なカメラのモデルにおいて、
カメラのレンズの焦点までの距離と、カメラの中心が画像のどこにあるのかを示す光学中心という物理量があり、
カメラ座標系の 3D 座標から画像へ投影する変換行列は以下のようになります。

![img](https://i0.wp.com/mem-archive.com/wp-content/uploads/2018/02/%E3%82%B9%E3%83%A9%E3%82%A4%E3%83%891.jpg?resize=768%2C376&ssl=1)
_引用：[カメラ内部パラメータとは](https://mem-archive.com/2018/02/21/post-157)_

この行列の$f_x, f_y$が焦点距離で$c_x, c_y$が光学中心を示します。
Unity ARFoundation の[`TryGetIntrinsics`という API](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@4.0/api/UnityEngine.XR.ARFoundation.ARCameraManager.html?q=intrinsics#UnityEngine_XR_ARFoundation_ARCameraManager_TryAcquireLatestCpuImage_UnityEngine_XR_ARSubsystems_XRCpuImage__)などから取得できます。

もし画像を取得したあとにカメラが動く場合は、
この時点でワールド空間における自己位置を保存しておく必要があります。
つまりモバイル AR などで Immersal を使う場合、カメラの position と rotation の値を保持しておきます。

## REST API による自己位置推定

前述のように Immersal の位置合わせにはカメラ画像とカメラ内部パラメータが必要でした。
ここで Immersal REST API ドキュメントを覗いてみましょう。

位置合わせリクエストのエンドポイントは`https://api.immersal.com/lozaclizeb64`で、必要データを POST することで自己位置情報が返ってきます。

:::message alert
記事を執筆中に Immersal SDK 1.15.0 がリリースされ、これに対応する REST API のエンドポイントも追加されました。
1.15 の REST API の場合は`https://api.immersal.com/1.15.0/lozaclizeb64`を使用してください。
詳しくは[ChangeLog](https://immersal.gitbook.io/sdk/about/changelog#sdk-v1.15.0)をご覧ください。
:::

リクエストに必要な body パラメータは以下の通りです。
(公式ドキュメントより引用)

| フィールド | 型     | 説明                                                    |
| :--------- | :----- | :------------------------------------------------------ |
| mapIds     | array  | An array of {"id": int} objects                         |
| b64        | string | Base64-encoded PNG image, 8-bit grayscale or 24-bit RGB |
| oy         | number | Camera intrinsics principal point y                     |
| ox         | number | Camera intrinsics principal point x                     |
| fy         | number | Camera intrinsics focal length y                        |
| fx         | number | Camera intrinsics focal length x                        |
| token      | string | A valid developer token                                 |

`mapIds`はマップの id ですね。
id の配列ではなく`{id:xxxx}`の配列になっているので間違えないようにしましょう。
`b64`は png 形式の画像を base64 でエンコードした文字列です。
Unity ではカメラ画像の Texture2D を png にエンコードし手えられた byte 列を更に base64 にエンコードするという手順で取できますす。
`oy, ox`はカメラの光学中心で、`fx, fy`は焦点距離です。どちらも pixel 単位の数値型で送信します。
`token`は Immersal の開発者トークンの文字列です。

これらのパラメータを POST して、無事位置合わせが成功すれば
例えば以下のような body を持ったレスポンスが返ってきます。

```json:公式より引用
{
  "error": "none",
  "success": true,
  "map": 7587,
  "px": -0.5459369421005249,
  "py": 0.0632220059633255,
  "pz": 0.36885133385658264,
  "r00": 0.68967300653457642,
  "r01": 0.14381979405879974,
  "r02": -0.709695041179657,
  "r10": 0.0070222793146967888,
  "r11": 0.97870355844497681,
  "r12": 0.20515856146812439,
  "r20": 0.7240869402885437,
  "r21": -0.14647600054740906,
  "r22": 0.67397546768188477
}
```

`px,py,pz`はマップ原点から見たカメラの相対位置で、
`r00`～`r22`は 3×3 回転行列の各要素です。

`success`は位置合わせが成功したかどうかの bool 値で、
リクエストが通ったからと言ってこの値が必ず true になるわけではありません。
もし位置合わせが失敗した場合には位置はゼロベクトル、回転行列はゼロ行列が入っています。

`error`はそもそもリクエストの内容が間違っていた時に、その内容が格納されます。
自分が見たことあるのは、リクエストで送信した b64 の内容が
png じゃなかったりすると`"error":"image"`といった具合に返ってきます。

## AR オブジェクトの座標変換

最後に座標変換処理ですが、
ここで一旦今まで取得できた情報と位置合わせで最終的に必要な状態を整理しましょう。
まず今の時点で REST API から返ってきた情報は「マップの原点」座標系における
「カメラの姿勢」でした。
この情報をもとに AR シーンと現実空間の位置合わせをするわけです。

位置合わせとは、異なる座標系の整合性をとることでした。
この整合性がとれることで、AR 空間にいるプレイヤが現実空間ではどこにいるのかがわかります。
そしてその状態では逆に、AR ワールド空間においてマップ（＝デジタルツイン ≒ 現実）がどこにあるのかがわかるとも言えます。
この状態に持っていくことが位置合わせの最終目的です。
これは次のような工程で求まります。

1. マップ座標系におけるカメラの姿勢（レスポンスデータ）
2. カメラ座標系におけるマップの姿勢(つまり 1.の逆行列)
3. ワールド座標系におけるマップの姿勢（2.にキャプチャ時のカメラの姿勢行列をかける）

何も変換処理をしないレスポンスデータそのままだと、
マップからみたカメラの姿勢データですが、
それを逆変換するとカメラから見たマップの姿勢になります。
したがってこの姿勢をそのままマップ行列に作用させると、
原点姿勢のカメラから見たときにキャプチャした時の見た目そのままになります。
そしてこのマップの姿勢をワールド空間で正しい姿勢にするには
キャプチャした時のカメラ姿勢を変換行列として差表させることで実現できます。

分かりにくかったら申し訳ないのですが、一応図を用意しました。
（HoloLens Meetup の時のものなのでプレイヤの位置が HoloLens になってます）

![img](/images/immersal-server-localize-and-api/arspace-transform.png)

# おわりに

Immersal REST API を例としてサーバーサイド位置合わせの考え方を解説しました。
Unity で Immersal SDK を使えばあまり意識せずに実装できる位置合わせですが、
サーバーサイド位置合わせは自分で実装する手順がいくつかあるので、少し難しく感じる側面もあります。
逆にサーバーサイド位置合わせがサービスとして提供されていることによって
SDK が公式に提供されていないプラットフォームでも VPS を使うことができるようになり、
自分で VPS の機能の一部をカスタマイズして使うことができます。

将来 AR というものが人々にとって身近なものになった時、
AR クラウドは日常的に使うアプリケーションの重要なインフラになる可能性があります。
そうなるのは未来の話であっても、機能としては使うことができるわけですので
今のうちに、どんなことが、どれくらいできるのか色々試してみたいものです。

Immersal は VPS の中でもかなり扱いやすいサービスの 1 つですし
サーバーサイド位置合わせを提供してくれているためカスタマイズ性も高いため
とても良いサービスだと思っています。
ぜひこの機会に、色んな人に使って見てほしいです。

最後まで読んでいただきありがとうございました。
この内容が何かのお役に立てれば幸いです。

## 参考

https://speakerdeck.com/drumath2237/localize-using-immersal-in-webar-project-in-hololens2-edge

https://time-space.kddi.com/au-kddi/20210709/3140

https://github.com/drumath2237/Immersal-Server-Localizer
