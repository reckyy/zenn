---
title: "newspaperからActiveSupport::Notificationsへ移行する流れを調べてみた"
emoji: "📰"
type: "tech"
topics: ["rails", "activesupport", "pubsub"]
published: true
---

# はじめに

興味があって調べてみた内容のメモを兼ねた記事です。
前回まとめたnewspaperの[記事](https://zenn.dev/recky/articles/news_paper_ruby)がFBC（著者が所属しているプログラミングスクールです）内で参考にしてもらえたので、今回もその延長で書いてみました。
今回は「newspaperをやめて`ActiveSupport::Notifications`に置き換える動き」についてです。

## 背景
- チームでnewspaperというPub/SubのGemを使っていた
- ただしRails標準の`ActiveSupport::Notifications`で同じことができるため、徐々に移行する方針になった

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
- どのイベントを購読するかを subscribe で登録
- イベントが発火したら handler.call が呼ばれる

違いは以下の2点です。

- 第一引数が シンボル → 文字列
- `handler.call`の引数が増える（`payload`だけ使うなら特に困らない）
```diff ruby
- Newspaper.subscribe(:event, handler) # handler.call(payload)が呼ばれる
+ ActiveSupport::Notifications.subscribe('event.name', handler)
 # イベント発生時にhandler.call(name, started, finished, unique_id, payload)が呼ばれる
```

各引数の内容はRailsガイドを参照ください。
https://railsguides.jp/active_support_instrumentation.html#%E3%82%A4%E3%83%99%E3%83%B3%E3%83%88%E3%81%AE%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF%E3%83%A9%E3%82%A4%E3%83%96

### newspaperとの比較表
| 項目 | `newspaper` | `ActiveSupport::Notifications` | 備考 |
| - | - | - | - |
| イベント発行 | `Newspaper.publish(:event, data)` | `ActiveSupport::Notifications.instrument('event.name', data)` | イベント名は**シンボル**→**文字列**へ変更 |
| イベント購読 | `Newspaper.subscribe(:event, handler)` | `ActiveSupport::Notifications.subscribe('event.name', handler)` | 基本的な流れは同じ |
| handler 引数 | `handler.call(payload)` | `handler.call(name, started, finished, id, payload)` | `payload` 以外は基本不要 |

## おまけ : なぜ`to_prepare`を使うのか？
FBCリポジトリの[PR](https://github.com/fjordllc/bootcamp/pull/8835)を眺めてみたところ、主な変更は上記に記載した通りでしたが、一つ気になるコードがありました。
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

開発環境でコードを編集した後も確実に反映させるため、`to_prepare`が使用されているのだと理解しました。
参考 : 
https://railsguides.jp/configuring.html#config-after-initialize
https://api.rubyonrails.org/classes/ActiveSupport/Reloader.html#method-c-to_prepare

## まとめ
- `publish` → `instrument`
- イベント名はシンボルから文字列へ
- handlerの引数は増えるが、普段は`payload`だけ使えばok
