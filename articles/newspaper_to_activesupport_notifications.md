---
title: "newspaperからActiveSupport::Notificationsへ移行する流れを調べてみた"
emoji: "📰"
type: "tech"
topics: ["rails", "activesupport", "pubsub"]
published: false
---

# はじめに

興味があって調べてみた内容のメモを兼ねた記事です。
前回まとめたnewspaperの[記事](https://zenn.dev/recky/articles/news_paper_ruby)がFBC（著者が所属しているプログラミングスクールです）内で参考にしてもらえたので、今回もその延長で書いてみました。



- 今回は「newspaperをやめて`ActiveSupport::Notifications`に置き換える動き」について。

## 背景
- チームでnewspaperというpub/subのGemを使っていた
- ただしRails標準の`ActiveSupport::Notifications`で同じことができるから、徐々に移行する方針になった

## 何がどう変わるのか？

結論、ほとんど何も変わりません。
やっていること自体は同じで、書き方や引数の扱いが少し異なるだけです。

### イベント発行
- `publish`から`instrument`に変わる。
- 第一引数がイベントシンボルからイベント文字列に変わる。
- 渡すデータ(payload)はそのままハッシュでok
```diff ruby
- Newspaper.publish(:event, data)
+ ActiveSupport::Notifications.instrument('event.name', data)
```

### イベント購読
イベントを購読する処理は、次の2段構えになっています。
- `subscribe` : どのイベントを聞くか登録する
- `handler.call` : 実際にイベントが起きた時に呼ばれる

`subscribe`時には第一引数がシンボルから文字列に変わる。
`handler.call`時には、引数が増えます。ただし普段使うのは`payload`だけなのでほとんど困りません。
```diff ruby
- Newspaper.subscribe(:event, handler) # handler.call(payload)が呼ばれる
+ ActiveSupport::Notifications.subscribe('event.name', handler)
 # イベント発生時にhandler.call(name, started, finished, unique_id, payload)が呼ばれる
```

各引数の内容はRailsガイドを参照ください。
https://railsguides.jp/active_support_instrumentation.html#%E3%82%A4%E3%83%99%E3%83%B3%E3%83%88%E3%81%AE%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF%E3%83%A9%E3%82%A4%E3%83%96

## おまけ : なぜ`to_prepare`を使うのか？
実際に、FBC代表のkomagataさんが移行を行なった[PR](https://github.com/fjordllc/bootcamp/pull/8835)を眺めてみました。
主な変更は上記に記載した通りでしたが、一つ気になるコードがありました。
```diff ruby:config/initializers/active_support_notifications.rb
# frozen_string_literal: true

- Rails.configuration.after_initialize do
+ Rails.application.reloader.to_prepare do
  ActiveSupport::Notifications.subscribe('answer.create', AnswererWatcher.new)
end
```

調べてみると、それぞれの違いはざっくりこんな感じです。
- `after_initialize` : 起動直後だけ実行される
- `to_prepare` : 起動時 + リロード時ごとに実行される。毎回最新のクラス定義で再登録できる。

開発中にコードがリロードされた時でも、最新の定義で購読を登録し直すために置き換えていると理解しました。
参考 : 
https://railsguides.jp/configuring.html#config-after-initialize
https://api.rubyonrails.org/classes/ActiveSupport/Reloader.html#method-c-to_prepare

## まとめ
- `publish` → `instrument`
- イベント名はシンボルから文字列へ
- handlerの引数は増えるが、普段は`payload`だけ使えばok
