---
title: Railsのi18nで予約語をキーに指定しても例外が発生しない不具合を改善した
emoji: "🫡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "I18n"]
published: true
---
## はじめに
現在、[FBC](https://bootcamp.fjord.jp/welcome)で学習中の身です。
RailsのI18n実装中に表題の不具合を発見し、issueを起票し修正されたのでその流れを記したいと思います。

## 問題
I18nを用いて、訳文に変数を渡そうとしていたのですが、、
なぜか変数名がそのまま返ってきてしまいました。

## いろいろ試してみる
まず、consoleで確認。
```ruby
#config/locales/ja.yml
 controllers:
    default:
      update: "%{object}の更新に成功しました。"
```

```ruby
irb(main):001:0> I18n.t('controllers.default.update', object: Book.model_name.human)
=> "%{object}の更新に成功しました。"
```
「本の更新に成功しました。」と出力されてほしいのに、なぜか「%{object}の更新に成功しました。」とそのままの文字列ができてしまっています。

ただ、モデル名の翻訳はできてる。
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

> defaultキーワードおよびscopeキーワードは予約されているため、変数には使えません。利用するとI18n::ReservedInterpolationKey例外が発生します。 ある訳文で式展開変数が期待されているにもかかわらず、#translateに値が渡されない場合は、I18n::MissingInterpolationArgument例外が発生します。

## お手上げ。
どうしたらいいんだー！
メンターの方から Let's go Q&A!とのことで、早速質問しました。
そしたらすぐに回答が！👀

どうやら `object` は予約語で他にもあるらしい。
```ruby
# https://github.com/ruby-i18n/i18n/blob/v1.14.1/lib/i18n.rb#L19-L34
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

**ちなみに**
この記事執筆時点で、`scope`キーを使って試してみましたがエラーは出ませんでした。。
修正版のgemでは、ちゃんとエラーになりました。
```ruby
irb(main):003:0> I18n.t('update', scope:'controllers.default', object:Book.model_name.human)
=> "%{object}の更新に成功しました。"
```

後日見ると、やはりエラーを出した方がいいということで修正されてました。

## 今回された修正
予約語を使用した時、エラーを吐くように修正！（かなりざっくり）
scopeが空の場合にも予約語が使用されていないかチェックしてくれるようになった。
```ruby
if values && !values.empty?
  entry = if deep_interpolation
    deep_interpolate(locale, entry, values)
  else
    interpolate(locale, entry, values)
   end
elsif entry.is_a?(String) && entry =~ I18n.reserved_keys_pattern
  raise ReservedInterpolationKey.new($1.to_sym, entry)
end
entry
```


### 未リリースのgemで試した→ちゃんとエラーになった！

未リリースのgemを使用するようにGemfileを編集します。
```ruby
#Gemfile
gem 'i18n', github: 'ruby-i18n/i18n'
```

そして、`bundle update`します。
rails consoleで確かめたところ、ちゃんとエラーを吐くようになっていました！！🎉

```ruby
Loading development environment (Rails 7.0.8)
irb(main):001> I18n.t('controllers.default.update', object:Book.model_name.human)
/Users/xxxxxxx/.rbenv/versions/3.2.2/lib/ruby/gems/3.2.0/bundler/gems/i18n-baf8a889391c/lib/i18n/backend/base.rb:65:in `translate': reserved key :object used in "%{object}の更新に成功しました。" (I18n::ReservedInterpolationKey)

          raise ReservedInterpolationKey.new($1.to_sym, entry)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
irb(main):002> I18n.t('update', scope:'controllers.default', object:Book.model_name.human)
/Users/xxxxxxx/.rbenv/versions/3.2.2/lib/ruby/gems/3.2.0/bundler/gems/i18n-baf8a889391c/lib/i18n/backend/base.rb:65:in `translate': reserved key :object used in "%{object}の更新に成功しました。" (I18n::ReservedInterpolationKey)

          raise ReservedInterpolationKey.new($1.to_sym, entry)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

未リリースのgem試行にあたり、以下の記事を参考にしました。
[Rails初心者に送る「Bundlerチョットワカルマニュアル」 #Ruby - Qiita](https://qiita.com/jnchito/items/823323e276d1a90ac4a1#%E3%81%BE%E3%81%A0%E3%83%AA%E3%83%AA%E3%83%BC%E3%82%B9%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%81%AA%E3%81%84github%E4%B8%8A%E3%81%AEgem%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B)

## さいごに
この記事ではi18nの不具合修正の過程を通して、「質問することは悪いことじゃない！」ということをお伝えしたかったです。
記事全体でみると、自分はバグを発見しただけでほとんど何もしていません。。
ですが、質問というアクションを起こしてよかったと思います。
「あー、なんか他の単語なら使えたからほっとこ」となっていたら、この現象は次の誰かが見つけるまで残っていたことになります。それはよくありません。
今からRailsを学習する方達が同じシチュエーションに遭遇したとき、「え、どういうこと？！もういやや😭」となるのを防げたと考えたら嬉しいです ☺️

## Special thanks
ご協力いただいたメンターの[yuuu](https://twitter.com/Y_uuu)さん、回答していただいた[伊藤](https://twitter.com/jnchito)さん、本当にありがとうございました。🙇
