---
title: ”RailsのI18nの訳文に変数を渡すときの不具合を改善した(??)"
emoji: "🫡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "I18n"]
published: false
---
## はじめに
現在、FBCで学習中の身です。
RailsのI18n実装中に表題の不具合を発見し、issueを起票し修正されたのでその流れを記したいと思います。

## 問題
I18nを用いて、訳文に変数を渡そうとしていたのですが、、
なぜか変数名がそのまま返ってきてしまいました。

## いろいろ試してみる
まず、consoleで確認。
```ruby
irb(main):001:0> I18n.t('controllers.default.update', object: Book.model_name.human)
=> "%{object}の更新に成功しました。"
```

モデル名の翻訳はできてる。
```ruby
irb(main):003:0> Book.model_name.human
=> "本"
```

普通に文字列を渡してもダメ。
```ruby
irb(main):002:0> I18n.t('controllers.default.update', object: '本')
=> "%{object}の更新に成功しました。"
```

[Rails 国際化（I18n）API - Railsガイド (railsguides.jp)](https://railsguides.jp/i18n.html#%E8%A8%B3%E6%96%87%E3%81%AB%E5%A4%89%E6%95%B0%E3%82%92%E6%B8%A1%E3%81%99)を見ても`I18n::ReservedInterpolationKey例外が発生します。` って書いてるし、、

## お手上げ。
どうしたらいいんだー！
@yuuuさんから Let's go Q&A!とのことで、質問しました。
そうしたら 伊藤淳一 さんから回答が！👀

どうやら `object` は予約語で他にもあるらしい。
```
RESERVED_KEYS = %i[
    cascade
    deep_interpolation
    default
    exception_handler
    fallback
    fallback_in_progress
    fallback_original_locale
    format
    object
    raise
    resolve
    scope
    separator
    throw
  ]
```

「予約語はたくさんあるし、I18n::ReservedInterpolationKey例外も起きないし。
これはもしかするとコントリビューションチャーンス！？」
とお言葉をいただき、折角なのでissue立ててみた。

立てたissueは以下。
[Inquiry About Reserved Keywords When Passing Variables to Translations · Issue #49879 · rails/rails](https://github.com/rails/rails/issues/49879)

## あっという間に修正。
回答が来ました。以下ざっくり。

> `scope`キーを使わないとエラーは出ないよ～。じゃないと値はそのまま返されるよ。
`I18n.t('index', scope:'books.title', object:Book.model_namu.human)`のようにね。
でもバグかもしれないから一旦確認するね～。

後日見ると、やはりエラーを出した方がいいということで修正されてました。
