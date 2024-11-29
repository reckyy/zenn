---
title: "読書習慣をサポートするWebサービス「Tsundoku」をリリースしました！！"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails", "nextjs", "authjs", "個人開発"]
published: true
---



## はじめに
本日、読書習慣をサポートするWebサービス**Tsundoku**をリリースしました。

- サービスURL

https://tsundoku.tech

- リポジトリ

https://github.com/reckyy/tsundoku

## 自己紹介
recky（レッキー）です。昨年から[FjordBootCamp](https://bootcamp.fjord.jp/)（以下、FBC）に入会し学習しています。

## Tsundokuの紹介
Tsundokuは「読書はしたいけれど、つい先延ばしにしてしまう」「読書習慣がなかなか続かない」といった問題を解決するための読書管理アプリです。
章ごとにメモを取り、読書ログをつけることで、自分の読書履歴を簡単に記録できます。また、GitHubのContributionグラフのように読書活動を視覚的に確認できるため、モチベーション維持につながります。

### 使い方

使い方としては簡単です。

| 本を検索 | 読書メモを取る | 読書ログを公開 |
| ---- | ---- |  ---- |
| 本を検索して、登録します。 | 本ごとにメモを取り、読書ログをつけます。 | 読書した本やログは公開できます。 |
| [![Image from Gyazo](https://i.gyazo.com/1a17a29e784994d7703c24620eca2d54.gif)](https://gyazo.com/1a17a29e784994d7703c24620eca2d54)| [![Image from Gyazo](https://i.gyazo.com/238d80b466672ff5a87e0cbae4f5a7a6.gif)](https://gyazo.com/238d80b466672ff5a87e0cbae4f5a7a6)| [![Image from Gyazo](https://i.gyazo.com/3c0ed8f17b3d992d5cfb00bd780dcd9a.jpg)](https://gyazo.com/3c0ed8f17b3d992d5cfb00bd780dcd9a) |

### 作った経緯
私は読書が好きですが、その日やりたいことに注力するあまり時間をなくしてしまうことが多々あり、なかなか読書をルーティン化できずにいました。そこで、
- 読んできた本の一覧
- 読書ログ

の2つを外部公開するようなサービスがあれば、読書習慣をアピールすることでモチベーションとなり、読書習慣を継続できると考え「Tsundoku」を作りました。

### 技術スタック
#### バックエンド
- Ruby on Rails
- Ruby
- RuboCop（Formatter）
- RSpec（テスティングフレームワーク）
- PostgreSQL
- FactoryBot（テスト用のデータ生成）
- SimpleCov（計測ツール）

#### フロントエンド
- Next.js(App Router)
- React
- TypeScript
- Storybook（コンポーネントテスト）
- Mantine（UIフレームワーク）
- Auth.js（認証）

#### インフラ
Heroku、Vercel


## 技術選定の理由
### Next.js（App Router）
本サービスは画面遷移が少ないシンプルな構成ですが、あえてNext.jsを採用したSPAとして作成しました。その理由は以下の通りです。

- 他のフレームワークで新しい経験を積みたかった

Railsでのアプリ開発経験があるため、今回はフロントエンドフレームワークであるNext.jsを採用し、新しい技術に挑戦しました。

- App Routerを使った新しい構成に挑戦

過去の自作サービスではApp Routerを採用したものがなかったため、新しい構成に挑戦し、技術的な知見を広げたいと考えました。

- TypeScriptやReactへの興味

FBCのカリキュラムでRubyはかなり取り組みましたが、フロント周りはチーム開発であまり触れることがありませんでした。JavaScriptやReactのカリキュラムはありましたが、「全くできない」→「何となくできる」程度で、もう少し学びたいなと感じていました。そこでサービス開発を通して自身の興味がどちらに向くか判断したいと考えました。

### Mantine
色々あるUIコンポーネントライブラリの中から、Mantineを選定した理由は以下です。

- [ドキュメント](https://mantine.dev/)がわかりやすい

画面上で直感的に操作するだけで大体仕様を把握できます。とてもわかりやすかったです。

- 豊富なレイアウトの提供

私はデザインが苦手で、中学で美術1を取ったことがあるくらいです。そのため、なるべくデザインはライブラリに任せたかったのですが、その点Mantineは大変助かりました。

- 公式レイアウトサンプル集の[Mantine UI](https://ui.mantine.dev/)があり、全て無料で利用できます。

- アプリの全体レイアウトがコンポーネントとして用意されている。([AppShellコンポーネント](https://mantine.dev/core/app-shell/))

- Tiptapのラッパーがあった

メモ機能を実装できるTiptapのラッパーコンポーネントが用意されていました。デザインを考えず実装が容易でした。

- デザインが好み

### SimpleCov、Storybook test-runner（テスト計測ツール）
テストカバレッジを担保したいと思い、バックエンドではSimpleCovを採用しました。
フロントエンドではStorybookを採用しました。コンポーネントのテストとUIの確認を並行して進められるため、開発効率の向上につながりました。test-runnerを組み合わせれば、カバレッジも確認できるので良かったです。
（Storybookでは、一部のコンポーネントにローディング処理が含まれており、テストがローディング中に実行されて失敗してしまうため、カバレッジのパーセンテージが低くなっています。ただし、テストは実装しているため、実際のカバレッジはもう少し高いはずです。）

|  SimpleCov（Rails）| Storybook（Next.js）  |
| ---- | ---- |
| [![Image from Gyazo](https://i.gyazo.com/a27f13af24822b11c51abbc8138fdfb3.png)](https://gyazo.com/a27f13af24822b11c51abbc8138fdfb3) | [![Image from Gyazo](https://i.gyazo.com/f3fbc5a4c17ef9bdd1d69eea8217a845.png)](https://gyazo.com/f3fbc5a4c17ef9bdd1d69eea8217a845)|

### Alba
今回、Rails APIモードのJSONシリアライザとして、jbuilderかシリアライザクラスどちらを使用するか検討した結果、[Alba](https://github.com/okuramasafumi/alba)を採用しました。
その理由は以下の通りです。
- jbuilderは公式で提供されているものの、DSLが書きにくく感じた。
- シリアライザクラスを用いることにより、JSON（ロジック）単体でテストできる、かつ再利用性も高い。
- READMEに「well-maintained JSON serialization engine」と書かれておりしっかりメンテナンスされていることから、長く使っていけると判断。

また、実装中にクラスの定義で詰まる場面がありましたが、[Discussion](https://github.com/okuramasafumi/alba/discussions/388)で質問したところ作者の@okuramasafumiさんから迅速にお返事をいただけました。ありがとうございました！
ちなみに、@okuramasafumiさんは今月FBCのメンターになったそうです。🔥



## 苦労したところ
### Formatterのルール決め
自作サービスではコードスタイルのルールを自分で決める必要があり、どの程度厳密にするか悩みました。チーム開発でも採用されている[rubocop-fjord](https://github.com/fjordllc/rubocop-fjord)や色々な記事を参考にし、自分に合ったルールを追加しました。


### Auth.js（前バージョンがNextAuth.js）
当初認証機能を実装するのに、[NextAuth.js](https://next-auth.js.org/)を使っていました。実装もいよいよ終わるという時に、最新バージョンの[Auth.js](https://authjs.dev/)があることを知りました。
NextAuth.jsより使いやすくなっている印象を受けたので、再度一から実装し直しましたが、関数のモックでつまずきました。テストでは実際に認証機能を使えないのでモックしないといけないのですが、その方法がなかなかわからず苦労しました。

### メモ画面でのレンダリング
作者が読書する際、章ごとに感想やメモを記録するタイプなので、Tsundokuでもそのスタイルを採用しています。
この機能を実装する際、章を切り替えるSegmentedControlコンポーネントと、メモを編集するEditorコンポーネントをどのように連携させるかが課題でした。同じコンポーネント内にまとめれば実装は容易ですが、コンポーネントの責務を明確にするために分けることにしました。
コンポーネント外の値の変更を検知する方法として、useRefを使用しました。useRefで前回のheadingを保持し、useEffect内で現在のheadingと比較しています。これにより、headingが**変更されたとき**のみエディタの内容を更新し、不要な再レンダリングを防ぎました。

```ts
useEffect(() => {
  if (editor && heading !== prevHeading.current) {
    editor.commands.setContent(memoBody);
    prevHeading.current = heading;
    setHeadingTitle(title);
  }
}, [editor, heading, memoBody, title]);

```

### デザイン
上記でも記載しましたが、デザインが苦手でレイアウトを考えるのに苦労しました…
リリース前にmachidaさんにデザインレビューしていただき、そのおかげで見違えるほど良くなりました。本当に感謝しかありません。

## 新たに実装したいと思っていること
### APIに存在しない本の登録
現在[楽天ブックス書籍検索API](https://webservice.rakuten.co.jp/documentation/books-book-search)を使用しているため、個人出版物の登録が出来ません。Boothなどで出版されている技術書の記録をつけたい方もいるかもしれないので（作者もそうです。）、将来実装したいと考えています。

## 感謝
去年FBCに入会してからここまで学習を続けてこられたのも、komagataさん、machidaさんをはじめメンター、受講生、卒業生の方たちのおかげです。本当にありがとうございました。
