---
title: "Newspaperを読んでみた"
emoji: "🫡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "Ruby"]
published: true
---

## Newspaper Gemとは
新聞じゃないです。Gemの話です。（ごめんなさい）
https://github.com/komagata/newspaper

READMEによると、

> Newspaper is a small library that provides a pub/sub mechanism for ruby.
Newspaperはrubyのpub/sub機構を提供する小さなライブラリである。（訳）

**pub/subとは？**
パブリッシュ/サブスクライブ の略。
パブリッシャーとサブスクライバーの対話はブローカーによって制御されるスタイルのこと。
簡単にいうと、パブリッシャーは送るデータが何（送信先）に使われるか知らなくてよく、サブスクライバーは何（送信元）から送られてくるか知る必要がない。ブローカーがよしなにやってくれるという仕組みのことです。このモデルの利点は、pub/subのお互いが独立してることで、システムの柔軟性と拡張性が高まることです。

このGemを使用することで、そのメリットをRubyで簡単に享受できます！
他にもpub/sub機構を提供しているライブラリはありますが、Newspaperは非常にシンプルになっています。
実際に僕が通っている[FBC](https://bootcamp.fjord.jp/)のアプリはNewspaperを使用しています。
https://github.com/fjordllc/bootcamp

## 何ができるのか？
まずRailsのコールバックの仕組みを説明します。

> コールバックとは、オブジェクトのライフサイクル期間における特定の瞬間に呼び出されるメソッドのことです。
コールバックを利用することで、Active Recordオブジェクトが作成/保存/更新/削除/検証/データベースからの読み込み、などのイベント発生時に常に実行されるコードを書くことができます。
（Railsガイドより引用）

任意のタイミングでイベントを設定し、処理を行えるということですね。
利用可能なActive Recordコールバックは `after_save`や`after_create`などたくさんあります。
https://railsguides.jp/active_record_callbacks.html#%E5%88%A9%E7%94%A8%E5%8F%AF%E8%83%BD%E3%81%AA%E3%82%B3%E3%83%BC%E3%83%AB%E3%83%90%E3%83%83%E3%82%AF

ただこのコールバックには問題が多いです。
- fat modelになりやすい。
- コールバックの実行順序の仕様が複雑。
- 依存関係が見えにくい。
- 特定のタイミングを設定しにくい（コールバックに条件分岐を入れることもできるが、アンチパターン）。
- 可読性が落ちる。（コールバック処理がないか？と常に気にしないといけない。個人的にはこれが一番でかいかも）

などなど。要はビジネスロジックが複雑になるという問題に集約されます。
これを解決してくれるのが、Newspaperです。

## どう使うの？
### README(usage)引用

---
> Using ActiveRecord callbacks.
```ruby
class User
  after_create UserCallbacks.new
end

class UserCallbacks
  def after_save(user)
    # do something
  end
end

@user.save
```
If Newspaper is used, it can be written as follows.
```ruby
# app/models/sign_up_notifier.rb
class SignUpNotifier
  def call(payload)
    # do something
  end
end

# config/initializers/newspaper.rb
Rails.configuration.after_initialize do
  Newspaper.subscribe(:user_create, SignUpNotifier.new)
end

# app/controllers/users_controller.rb
Newspaper.publish(:user_create, payload)
```
---
つまり、

1. 最初にイベントごとにサブスクライバーを設定する。
`Newspaper.subscribe(イベント種類のシンボル, データ)`

2. パブリッシャーが特定のタイミングでイベントを発行する。
`Newspaper.publish(イベント種類のシンボル, データ)`

3. サブスクライバーが通知を受け取り、データを処理する
サブスクライブしているクラスがデータを受け取り、`call`メソッド内で特定の処理を実行します。

これにより、イベント駆動型プログラミングが容易になり、拡張性も高まります！
例えば`user.create`した後のイベントが以下だったとします。（極端かつ適当な例ですが。。）
```ruby
def after_create(user)
  userにメールを送る処理
  adminにメールを送る処理
  APIリクエストを送る処理
　userに1000ギフトを送る処理
end
```
従来の方法だと一律でコールバックが発生してしまったり、気軽にUser.createできなくなったり大変です。
これがNewspaperを使うと、
```ruby
# app/notifiers/user_mail_notifier.rb
class UserMailNotifier
  def call(user)
    # userにメールを送る処理
  end
end

# app/notifiers/admin_mail_notifier.rb
class AdminMailNotifier
  def call(user)
    # adminにメールを送る処理
  end
end

# app/notifiers/api_request_notifier.rb
class ApiRequestNotifier
  def call(user)
    # APIリクエストを送る処理
  end
end

# app/notifiers/gift_sender.rb
class GiftSender
  def call(user)
    # userに1000ギフトを送る処理
  end
end

# config/initializers/newspaper.rb
Rails.configuration.after_initialize do
  Newspaper.subscribe(:user_create, UserMailNotifier.new)
  Newspaper.subscribe(:user_create, AdminMailNotifier.new)
  Newspaper.subscribe(:user_create, ApiRequestNotifier.new)
  Newspaper.subscribe(:user_create, GiftSender.new)
end

# app/controllers/users_controller.rb
def create
  user = User.create(user_params)
  if user.persisted?
    Newspaper.publish(:user_create, user)
  end
end
```
のように柔軟に管理することができます。各処理の独立性、そしてアプリケーションの拡張性と保守性を保つことができます！！

## おまけ（コードを覗いてみた）
中身は以下です。
```ruby
# frozen_string_literal: true

require_relative "newspaper/version"

# namespace
module Newspaper
  class Error < StandardError; end

  @event_bus = {}

  class << self
    attr_reader :event_bus

    def subscribe(event, subscriber)
      @event_bus[event] ||= []
      @event_bus[event] << subscriber
    end

    def publish(event, payload)
      @event_bus[event] ||= []
      @event_bus[event].each do |subscriber|
        subscriber.call(payload)
      end
    end

    def clear(event)
      @event_bus.delete event
    end

    def clear_all
      @event_bus = {}
    end
  end
end
```
流れとしては、とてもシンプルなものになっています。
1. Railsアプリケーションの初期化が完了した後に、イベントのシンボル達を@event_busに詰め込む。
2. publishメソッドが実行されると、@event_busの中からイベントを検索しサブスクライバーそれぞれのcallメソッドを実行する。

`publish`メソッドの最初の行は、指定されたイベントにまだサブスクライバーが登録されていない場合に備え、エラーを防ぐために配列を初期化、でしょうか。

## 終わりに
最近、コードリーディングするのがますます楽しくなってきました！
細部まで理解しようとしすぎる癖はありますが、、（今回のように）
読んでいただき、ありがとうございました〜。
