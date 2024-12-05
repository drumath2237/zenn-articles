---
title: "pnpm publishでprovenanceバッヂを付ける方法（2024年12月）"
emoji: "🎖️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pnpm", "cicd", "githubactions", "npm"]
published: true
---

:::message
この記事は[にー兄さんアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/ninisan-2024)の５日目の記事です。
:::

## TL;DR

次のコマンドで対応します。

```sh
NPM_CONFIG_PROVENANCE=true pnpm publish
```

## pnpm publishにはprovenanceオプションがない

<!-- textlint-disable -->
最近私は個人開発で npm ライブラリをリリースしたのですが、pnpm publish コマンドに`--provenance`オプションがなくて「おや？」となりました。
pnpm、特にモノレポで作ってる場合はworkspace機能が充実しているので便利なのですが、実装されていないのは意外でしたね。
<!-- textlint-enable -->

調べてみると、やはり私以外も困っている人がいたようでした。しかし日本語の文献がなかったので、小ネタですが記事にした次第です。

## 解決方法

もしかしたら後から対応される可能性はありますが、今のところは環境変数から npm のオプションを有効にして対応できるとのことでした。

```sh
NPM_CONFIG_PROVENANCE=true pnpm publish
```

<!-- textlint-disable -->
Ubuntu 環境の CI ジョブでビルドする場合はいいですが、Windows などの非 POSIX シェルではエラーになりそうですので、Shell Emulator も併せて有効化しておくといいかもしれません。
<!-- textlint-enable -->

https://zenn.dev/drumath2237/articles/pnpm-shell-emulator-for-win-user


## 参考文献

https://github.com/pnpm/pnpm/issues/6435#issuecomment-1518397267