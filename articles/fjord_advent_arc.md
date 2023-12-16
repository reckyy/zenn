---
title: "かゆい所に手が届きまくりのブラウザ、Arc"
emoji: "💎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["macOS", "AI"]
published: false
---

:::message
この記事は、[フィヨルドブートキャンプ Part 1 Advent Calendar 2023 - Adventar](https://adventar.org/calendars/9142) 17日目の記事です。

昨日はayuさんの[初学者が続けているテック系Podcast紹介！](https://ayu-0505.hatenablog.com/entry/2023/12/16/134921)
と、
odentakashiさんの[情熱プログラマー輪読会を開催してみて](https://odentakashi.github.io/2023/11/16/post11.html)でした。
:::

## はじめに
私はこれまでずっとChromeを使ってきました。ですが今年の春頃、新しいブラウザ**Arc**に乗り換えました！
使い始めた時の感覚は、昔プレイしてたゲームのリマスター版が出た感じに近かったかもしれません。
「うわめっちゃ綺麗になってる！」「そうそうこの機能欲しかった〜」みたいな。
日々ブラウザを使う方にとっては、効率的なブラウジングは欠かせないものです。その点、Arcは使いやすさと機能性に優れています！
この記事では、そんなArcの概要やいい点などをみなさんに紹介したいと思います！

筆者はChromeを使いこなしていたわけではないので、Arc独自の機能、みたいに書かれている機能の中に「Chromeでもできるよ！」と言うものが存在するかもしれません。
その際は訂正しますので、教えていただければ幸いです。🙇


:::message
Arcは基本的に、実際に使っている映像を見せて機能の紹介をしています。「まずは使ってみ？」みたいなスタンス。
（実際、テキストより映像の方がわかりやすい）
以下では文章にしていますが、わかりづらい点が多々あるかと思うので、なるべく画像か動画をセットでつけています。
:::

## Arcとは？

[The Browser Company](https://thebrowser.company/)が開発したChromiumベースのブラウザです。Chromeを受け継ぎつつ、さらに便利な機能を享受できます！移行も一瞬です！
ここではArcの機能紹介のみにとどめますが、気になった方は会社HPもチェックしてみてください。

## インストール

インストールは、[Arc from The Browser Company](https://arc.net/)にアクセスして画面の案内に従えばできます！
Mac版はすぐインストールできるのですが、~~Windows版は現在開発中のため、Waitlistに登録しないといけないぽいです。。😭~~

:::message
12/14にWindowsのbeta版がリリースされました！！🎉
1月から一週間に1000人ずつ参加者を増やすらしいです！！😭
[Arc Release Notes](https://arc.net/e/6433CE8D-2D3F-4FF2-AD51-6EF0CBAD197F)
:::


## 便利な機能紹介

### spaces

スペースという機能は、**同じウィンドウ内で自分の使用用途ごとにタブなどを分けられる機能**です。。（説明難しい）
要は、仕事関連の作業しててふと趣味の調べ物をしたくなったら別ウィンドウを開く、といったことをしなくていい、と言うイメージ？
（自分はspacesを一つしか使用していないのでわかりません、、🫠）
仕事ならworkスペース、趣味ならhobbyスペース、といった具合に。
以下はswipeでspacesを切り替えている動画です。
「space1」から「趣味」へspacesを移しています。（見辛くてごめんなさい、、！🙏


![Image from Gyazo](https://i.gyazo.com/a5faa242dfc6f644046ebdf2ab7c84ae.gif =600x)

### favorites（お気に入り）

**これが便利すぎてもう従来のブラウザには戻れないです。。**
以下の画像の部分がfavoritesです。

![](/images/arc_gifs/arc_favorite.png =150x)

よく開くアプリをここに登録しておけば、いつでもCmd + 1~9 で開けます！！
（最大12個まで登録できますが、ショートカットで開けるのは8個目までです。）
登録したfavoritesはどのスペースでも同一です。

### pinned tabs

favoritesより重要度がワンランク下のタブをまとめるところです。
pinned tabsは各スペースに依存します。

![](/images/arc_gifs/arc_pinned_tabs.png =500x)

毎日は開かないけど週に１回は開く、今は見ないけどいつか見るから置いときたい。みたいなタブを置いておけます。
なぜこの機能があるかというと、通常開いたタブはTodayフォルダに配置されます。Today配下にあるアイドル状態のタブは、**24時間でアーカイブ**されます。（アーカイブされる時間は変更可能です。最低12時間、最大1か月）
これで、タブ開きすぎ問題が解消されます！着ない服と一緒です。**本当に必要なものだけあればいいのです。**

追加方法は、単純に画像のエリアにdragするか、pinしたいタブを開いてcmd + dです！
[Auto Archive: Clean as you go](https://resources.arc.net/en/articles/6701333-auto-archive-clean-as-you-go)

### split view
split viewがdrag and dropでできるんです。そう、Arcならね。😐
（optキーを押しながらクリックでもsplit viewになりますし、Cmd + Tで新しいタブをsplit viewにもできます。）

![Image from Gyazo](https://i.gyazo.com/f7d0145027ab667b0c93bdc2562b2653.gif =500x)

### Easel
なんでもできるボードです。Notionのブロックがないバージョン？を想像していただけたら。
どこでも画像貼れるし、Webサイト埋め込めるし。
ちなみにArcは毎週リリースノートを公開しているのですが、Easelで作られています！
よければ見てみてください。

https://arc.net/e/D25B2EEA-7506-4850-A169-3B2A00802889

### ChatGPTにすぐに質問できる
ChatGPTを開かなくても、Cmd + opt + GでChatGPTに質問できるコマンドバーが開けます！！
コマンドバーで「gpt」+ tabでも可能です。
こういうかゆいところに手が届きまくりなんですよね。。
ただ、たまにGPT-3で回答されます。。（GPT-4利用者のみ）



## 個人的に気に入ってる点

### 画面が広く感じる
ChromeやEdgeだと上にタブバーが表示されます。ですがArcだと横に、しかも非表示にできるので画面をめいっぱい使えます。
些細なことかもしれませんが、こういったチリツモはなくなると意外と大きいものです。

![](/images/arc_gifs/google_tab.png =500x)
*この部分がないだけで、広く感じるし首への負担も少しは減る、、かも？*

### 開いているタブがみやすい。
Chromeだと、タブ開きすぎてタイトル見えない状態によくなりませんか？
しかも探すとき、一個一個開いていかないと内容が分からないし。。
(下のはやりすぎですが笑)

![Image from Gyazo](https://i.gyazo.com/44dff297a03c8d23aef90d35d399dead.png)

Arcなら、各タブのタイトルやfaviconもみやすいし。。

![](/images/arc_gifs/arc_current_tabs.png =500x)

下のように control + tabでもみやすい。

![Image from Gyazo](https://i.gyazo.com/1f71731897121bd60b01aaf0f067de41.gif =500x)

このタブ移動のショートカット自体はChromeにも存在しますが、、
Arcだと、タブの順番は**開いた順番**になります。Chromeだとタブの順番は自分で並び替えないといけませんよね？
開くってことは必要性が高く、開かれないタブはどんどん後ろ（必要性が低く）になるのが良いはずです。（これは人によるかもしれません）
なので開いたタブを引き続き頻繁に使いたいとき、もう何回もcontrol + tabしなくて済みます！

個人的には、開いているタブ数が5~6個の時に便利で、10個以上タブを開く方には当てはまらないと思います。

### Little Arc
作業中に調べたくなったとき、その度Chromeを開くという不要なステップが解消されます。
Cmd + opt + Nで即座にLittle Arcを開くことができます。
ちょっと調べたい、みたいなときに便利です！

![Image from Gyazo](https://i.gyazo.com/c55a5c6b78f0d0779cf17abd35cf9034.gif)



### Site Searchが見やすい
これはChromeにもある機能なんですが、、コマンドバーから直接検索できるように設定できます！
ただArcだと、
- コマンドバーに「Site Search」と入力するだけで設定画面に飛べたり
- UIが見やすかったり

なんかイイな、って思うことが多いんですよね〜。

![](https://downloads.intercomcdn.com/i/o/703546622/d3b07ccc03e422ad7761c9b4/CleanShot+2023-04-01+at+10.51.19.gif)
*https://resources.arc.net/en/articles/7183263-site-search-directly-search-any-website　より引用*

## 困ったことがあれば、、
コマンドバーに表示されている、「Ask the Arc Team a Question」をクリックすれば運営にコンタクトを取るフォームが出てきます！
折角なので、僕も質問してみました。
内容は、Arcを使っているときだけショートカットで絵文字ビューアがでてこなくなった、というものです。
もし回答が来たら、この記事に追記しようと思います。

![Image from Gyazo](https://i.gyazo.com/acf0f1cff1d54103d068d7dfc738d381.png)

## さいごに
ArcはかなりUXデザインが優れており、視覚的にも機能的にも優れていると個人的には感じています。
もし興味が湧いたら、ぜひ使ってみてください！
