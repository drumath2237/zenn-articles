---
title: "Unity6以降のWeb向けライブラリにおけるAwaitableという選択肢"
emoji: "🚦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "unity6", "csharp", "awaitable", "asyncawait"]
published: true
---

:::message
この記事は[にー兄さんアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/ninisan-2024)の6日目の記事です。
:::

## はじめに

### TL;DR

- Unity6 以降の
- Web ビルドをターゲットに含む
- Unity パッケージ

<!-- textlint-disable -->
という条件の環境で Awaitable が使えると嬉しいかもしれないと考えた。
<!-- textlint-enable -->

### 概要

<!-- textlint-disable -->
本記事は Unity6(2023)から使える新機能である Awaitable について、その使いどころを考えてみた内容です。
趣旨として「こうするべきだ！」というような主張をする意図はないです（お風呂に入ってるとき「Awaitable ってどこで使えるんだろうなぁ」と考えてた時に思いついた内容を書いてみたものです）。
<!-- textlint-enable -->

まだ新しい機能なのと、筆者も使い込んでいるわけではないので、もし間違った内容を書いてしまっていたらご指摘いただけると幸いです。
また、Unity6 は LTS がリリースされているものの、Awaitable の機能は今後も強化されていく可能性がありますので、あくまで執筆現在（2024/12）の情報であることにご注意ください。

### 検証環境

- Windows 10 Home
- Unity 6000.0.31f1
- Chrome for Windows 131.0.6778.140

### サンプル

サンプルコードを次の GitHub リポジトリ及び GitHub Pages で公開しております。

https://github.com/drumath2237/Web-Awaitable-testbed

## Unity Webビルドにおける課題

Unity はもともと WebGL 向けにプロジェクトをビルドできますが、Unity6 からは WebGPU にも対応されたことにより「Web ビルド」と名称が統一されました。
本記事でもその名前に沿ってお話ししますが、こちらはいわゆる従来の「WebGL ビルド」と基本的には変わらないものを指しています。

Unity の Web ビルドは結構特殊な環境ということもあり、色々な罠・ツラみがあります。
例えばランタイムデバッグがしにくかったり、ビルドが遅かったりといったものがありますね。
上記に加え、個人的に結構つらいなって感じるのが「マルチスレッドを使う Task が動かない」問題です。
Task と言えば C#言語がサポートしている非同期処理の仕組みで、async/await を使って非同期な処理を同期的に記述できる手軽さが売りです。
多くの Unity プロジェクトで非同期処理を扱う時に Task を使うことが多いですが、残念ながら Web ビルドはマルチスレッドをサポートしておらず、一部の Task の API が使用できません。代表的なものは`Task.Delay`でしょうか。

ネット上では、この問題の回避策として次の 2 つを紹介していることが多いです。

- UniTask
- Coroutine

Unity アプリ開発者で一番多く使われているのは、UniTask で代替する方法でしょう。
UniTask は、Unity のシステムにあったより良い Task を提供する目的で開発されており、非同期処理は PlayerLoop で実行されるなどの工夫がなされています。
そのため、明示的に呼び出さない限りはシングルスレッドで動作することで Web ビルドでもちゃんと動作します。
UniTask は非常に高機能なライブラリなためエンタープライズでの採用例も多く、入れられるなら入れておくのが吉と言えるものですが、サードーパーティライブラリである点を考慮する必要があります。
もしかしたら会社や対応プラットフォームの都合で使えない可能性があったり、ライブラリを作る場合にはなるべく他のサードパーティに依存したくないという気持ちがありますね。

Coroutine も Unity で使える非同期処理の仕組みで、UniTask や Task が使える前から存在する方法です。
こちらもプレイヤーループベースの仕組みではありますが、Task/UniTask と違い、デフォルトで async/await に対応していなかったり値が返せなかったりと、機能面で劣ります。
筆者ももうしばらく Coroutine を使っていないですし、よっぽどの理由がない限りは UniTask を使いたい気持ちでいます。

## Awaitableという選択肢

さて、従来の Unity 2022 まではこのような状況だったものの、Unity6 で Awaitable が登場しました。

### Awaitableとは

Awaitable は新しく追加された非同期処理のためのクラスです。
いわゆる UniTask で実現されていたようなプレイヤーループベースの非同期処理を async/await を使って書けるのが特徴です。
実際、Awaitable は UniTask を参考に作られているらしく、「UniTask のサブセット」とも考えられと UniTask 作者の neuecc さんが言及されています。

https://github.com/Cysharp/UniTask/discussions/627

<!-- textlint-disable -->
しかし現状は、UniTask で使えるけど Awaitable では使えない機能もいくつかあるようで、例えば`UniTask.WhenAll`相当の API が無かったり、リークしている非同期処理を確認できる UniTask Tracker のようなものも Awaitable にはありません。WhenAll ないのは厳しいですね……。
<!-- textlint-enable -->

### AwaitableはWebビルドでも使える

前述のとおり Awaitable はプレイヤーループベースで非同期処理が動作するため、Task が抱えるような Web ビルドにおける問題は基本的に起きません。
また Coroutine とは違い、async/await が使えますし、サードパーティに依存せず Unity6 であれば何もしなくても使えます。

実際に Web で動作するサンプルを作成したのでご覧ください。

https://drumath2237.github.io/Web-Awaitable-testbed/

このサンプルでは非同期処理の実行開始ボタンが 3 つあり、それぞれ別の仕組みで「3 秒待つ」をします。3 秒経ったらタイムスタンプと一緒にログが吐かれます。
実行してみると、Awaitable は動くけど Task は動かないという結果になりました。

![alt text](/images/web-awaitable/demo.png)

Awaitable と Task でこの処理を書いた場合のコードを次に示します。

```cs:3秒待つ処理
public static async Awaitable<string> GetStringAwaitableAsync(string str, CancellationToken token)
{
    await Awaitable.WaitForSecondsAsync(3f, token);
    return str;
}

public static async Task<string> GetStringTaskAsync(string str, CancellationToken token)
{
    await Task.Delay(3000, token);
    return str;
}
```

また、このサンプルでは Web ネイティブプラグインの実行例も載せています。
内容としては JavaScript のプラグインコードで 3 秒待つ処理があり、それを Awaitable な関数でラップしているような作りになっています。

次に示すコードが JavaScript のプラグインコードです。
引数にコールバック関数の関数ポインタを受け取り、処理が終わったらコールバックを呼び出します。

```js:jslibのコード
mergeInto(LibraryManager.library, {
    getStringJSAsync: async function(callbackPtr) {
        await new Promise(resolve => setTimeout(resolve, 3000));
        Module.dynCall_v(callbackPtr);
    }
})
```

そして次に示すコードがラッパー関数です。
`GetStringAwaitableJSAsync`がラッパー関数のパブリックな API で、その中では AwaitableCompletionSource を使って外部から Awaitable を完了させています。
こうやって見ると、やはり Awaitable の API は Task や UniTask のそれと瓜二つですね。


```cs:プラグインのラッパー関数（C#）
[DllImport("__Internal")]
private static extern void getStringJSAsync(Action callback);

[MonoPInvokeCallback(typeof(Action))]
private static void OnCallback_GetStringJS()
{
    _awaitableSource?.TrySetResult();
}

[CanBeNull]
private static AwaitableCompletionSource _awaitableSource;

public static async Awaitable<string> GetStringAwaitableJSAsync(string str, CancellationToken token)
{
    token.ThrowIfCancellationRequested();

    _awaitableSource?.TrySetCanceled();

    _awaitableSource = new AwaitableCompletionSource();
    getStringJSAsync(OnCallback_GetStringJS);
    await _awaitableSource.Awaitable;
    _awaitableSource = null;

    return str;
}
```

## おわりに

本記事では、Awaitable の嬉しい活用先について書きました。
まとめると、Awaitable の次のような性質により、Web 向けライブラリでの活用ができるのでは無いかと考えました。

- UniTask 同様にメインスレッドで動作する非同期処理である
- Coroutine とは違い async/await がそのままで使える
- サードパーティに非依存である

<!-- textlint-disable -->
従来では制限の強かった WebGL ビルドですが、Unity6 では Web プラットフォームにおける機能追加・改善が多くなされ、使いやすくなった印象です。
そういった背景から、もしかしたら今後 Unity 製の Web アプリは増えていくかもしれないですね。今後の動向に注目です。
<!-- textlint-enable -->

### 参考文献

https://tomatosauce.jp/unitywebgl-asyncawait/#index_id0

https://discussions.unity.com/t/async-await-and-webgl-builds/665972/67

https://developers.cyberagent.co.jp/blog/archives/52835/